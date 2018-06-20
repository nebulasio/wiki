# Nebulas 101 - 05 RPC APIを通じてNebulasに接続

[YouTube チュートリアル](https://www.youtube.com/watch?v=to3tkwFjVXo)

NebulasチェーンノードがRPCを通じてリモートで訪問またはコントロールすることができる。Nebulasチェーンは一連のAPIを提供してスマートコントラクトにノード情報、アカウントバランス、情報を発信すると呼び出しを配備することを取得する。

Nebulasチェーンのリモートアクセスは[gRPC](https://grpc.io)に実装されている。([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway))代理を通じてHTTPでアクセスもできる。HTTPアクセスはRESTfulで実装されている一つのインターフェース、gRPCインターフェースと同じパラメーターである。

## API

RPCサーバーとHTTPサーバーを実装してGo-NebulasでAPIサービスを提供する。

### モジュール

全てのインタフェースが二つのモジュールに二分される: APIと管理者。

- API: ユーザーのプライベートキーに関連のないインターフェースを提供する。
- Admin: ユーザーのプライベートキーに関連のあるインターフェースを提供する

全てのNebulasノードはAPIモジュールを一般のユーザーに、管理者モジュールを認証したユーザーに開放することを薦める。
注意: (Nebulasテストネット & メインネットでの管理者apiはリモート呼び出しを支持しない)

### コンフィグレーション

Nebulasノード毎にの配置ファイルでRPCサーバーとHTTPサーバーを配置することができる。

```protobuf
rpc {
    # gRPC API service port
    rpc_listen: ["127.0.0.1:8684"]
    # HTTP API service port
    http_listen: ["127.0.0.1:8685"]
    # Open module that can provide http service to outside
    http_module: ["api", "admin"]
}
```

### 実例

#### HTTP

ここは`curl`を使用してHTTPインターフェースを呼び出すの例。

##### GetNebState

APIモジュールで `GetNebState` を呼び出してローカルNebulasノードの現在の状態をフェッチする。チェーンの識別、テールブロック、プロトコルバージョンなどが含まれている。

```bash
> curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nebstate

{"result":{"chain_id":100,"tail":"0aa1cceb7801b110fdd5217ba0a4356780c940133924d1c1a4eb60336934dab1","lib":"0000000000000000000000000000000000000000000000000000000000000000","height":"479","protocol_version":"/neb/1.0.0","synchronized":false,"version":"0.7.0"}}
```

##### UnlockAccount

管理者モジュールで `UnlockAccount` を呼び出してメモリにのアカウントをロック解除する。全てのロック解除されたアカウントがパスフレーズなしで直接に交易を発信することに使用できる。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz", "passphrase": "passphrase"}'

{"result":{"result":true}}
```

#### RPC

RPCサーバーは [GRPC](https://grpc.io/) で実装されている。GPRCのシリアル化は [Protocol Buffers](https://github.com/google/protobuf) に基づく。全てのrpcプロトバフファイルは [Nebulas RPC Protobuf Folder](https://github.com/nebulasio/go-nebulas/tree/develop/rpc/pb) にある。

ここは`golang`を使用してrpcインターフェースを呼び出すの例。

##### GetNebState

APIモジュールで `GetNebState` を呼び出してローカルNebulasノードの現在の状態をフェッチする。

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPCサーバー連結アドレスの配置
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// ノード状態情報にアクセスためのAPIインターフェース
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), & rpcpb.GetNebStateRequest {})
if err != nil {
    log.Println("GetNebState", "failed", err)
} else {
    log.Println("GetNebState tail", resp)
}
```

##### LockAccount

上のHTTPリクエストを通じて`v1/admin/account/unlock`を呼び出したあと、アカウント `n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz`のロックが解除される。管理者モジュールで `LockAccount` を呼び出して再びロックすることができる。

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPCサーバー連結アドレスの配置
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// 管理者インターフェースでアカウントアドレスをアクセス、ロックする。
admin := rpcpb.NewAdminServiceClient(conn)
from := "n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz"
resp, err = management.LockAccount(context.Background(), & rpcpb.LockAccountRequest {Address: from})
if err != nil {
    log.Println("LockAccount", from, "failed", err)
} else {
    log.Println("LockAccount", from, "result", resp)
}
```

### APIリスト

より多くのインターフェースについては、公式ドキュメントを参照してください:

- [API モジュール](https://github.com/nebulasio/wiki/blob/master/rpc.md)
- [Admin モジュール](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md).

### 次は

よくできた! 公式テストネットやメインネットに参加してNebulasをお楽しみに!

 [テストネットに参加](https://github.com/nebulasio/wiki/blob/master/testnet.md)
 [メインネットに参加](https://github.com/nebulasio/wiki/blob/master/mainnet.md)
