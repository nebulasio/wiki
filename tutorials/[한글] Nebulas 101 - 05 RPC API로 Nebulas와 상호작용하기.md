# Nebulas 101 - 05 RPC API로 Nebulas와 상호작용하기

[유튜브 튜토리얼](https://www.youtube.com/watch?v=to3tkwFjVXo)

네뷸러스 체인 노드는 RPC를 통해 원격으로 접근하고 제어할 수 있습니다. 네뷸러스 체인은 노드 정보와 주소 잔액을 얻고, 트랜잭션을 전송하고 스마트 컨트랙트를 배포하기 위한 일련의 API를 제공합니다.

[gRPC](https://grpc.io)에 의해 네뷸러스 체인의 원격 접근이 구현됩니다. 또한 프록시 ([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway))를 통해 HTTP에 의해 접근될 수 있습니다. HTTP 접근은 gRPC 인터페이스와 같은 매개변수를 가진 RESTful에 의해 구현된 인터페이스입니다.

## API

Go-Nebulas에서 API 서비스를 제공하기 위해 RPC 서버와 HTTP 서버를 구현했습니다.

### 모듈

모든 인터페이스는 두 모듈로 나눠집니다: API, Admin.

- API: 유저의 프라이빗 키와 관계 없는 인터페이스를 제공합니다.
- Admin: 유저의 프라이빗 키와 관련 있는 인터페이스를 제공합니다.

모든 네뷸러스 노드가 퍼블릭 유저를 위해 API 모듈을 열고 승인된 유저를 위해 Admin 모듈을 열 것을 권장합니다.
주의사항: (네뷸러스 테스트넷과 메인넷 위의 Admin API는 원격 호출을 지원하지 않습니다)

### 설정

RPC 서버와 HTTP 서버는 각 네뷸러스 노드의 구성 파일에서 구성될 수 있습니다.

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

### 예제

#### HTTP

`curl`을 사용하여 HTTP 인터페이스를 호출하는 예제입니다.

##### GetNebState

로컬 네뷸러스 노드의 체인 정체성, 꼬리 블럭, 프로토콜 버전 등을 포함하여 현재 상태를 가져오기 위해 API 모듈에서 `GetNebState`를 호출할 수 있습니다.

```bash
> curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nebstate

{"result":{"chain_id":100,"tail":"0aa1cceb7801b110fdd5217ba0a4356780c940133924d1c1a4eb60336934dab1","lib":"0000000000000000000000000000000000000000000000000000000000000000","height":"479","protocol_version":"/neb/1.0.0","synchronized":false,"version":"0.7.0"}}
```

##### UnlockAccount

메모리에서 주소를 언락하기 위해 Admin 모듈에서 `UnlockAccount`를 호출할 수 있습니다. 모든 언락된 주소들은 비밀번호 없이 직접 트랜잭션을 전송하는데 사용될 수 있습니다.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz", "passphrase": "passphrase"}'

{"result":{"result":true}}
```

#### RPC

RPC 서버는 [GRPC](https://grpc.io/)로 구현되었습니다. GRPC의 직렬화는 [Protocol Buffers](https://github.com/google/protobuf)에 기반합니다. [Nebulas RPC Protobuf Folder](https://github.com/nebulasio/go-nebulas/tree/develop/rpc/pb)에서 모든 rpc protobuf 파일들을 찾을 수 있습니다.

`golang`을 사용하여 rpc 인터페이스를 호출하는 예제입니다.

##### GetNebState

로컬 네뷸러스 노드의 현재 상태를 가져오기 위해 API 모듈에서 `GetNebState`를 호출할 수 있습니다.

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPC 서버 연결 주소 구성
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// API 노드 상태 정보에 접근하기 위한 인터페이스
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), & rpcpb.GetNebStateRequest {})
if err != nil {
    log.Println("GetNebState", "failed", err)
} else {
    log.Println("GetNebState tail", resp)
}
```

##### LockAccount

주소 `n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz`는 위 HTTP 요청을 통해 `v1/admin/account/unlock`을 호출한 후 언락됩니다. 다시 잠금하기 위해 Admin 모듈에서 `LockAccount`를 호출할 수 있습니다.

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPC 서버 연결 주소 구성
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// Admin 지갑 주소에 접근하고 잠금하기 위한 인터페이스
admin := rpcpb.NewAdminServiceClient(conn)
from := "n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz"
resp, err = management.LockAccount(context.Background(), & rpcpb.LockAccountRequest {Address: from})
if err != nil {
    log.Println("LockAccount", from, "failed", err)
} else {
    log.Println("LockAccount", from, "result", resp)
}
```

### API 목록

더 많은 인터페이스를 보고 싶으면 공식 문서를 참조해주세요:

- [API 모듈](https://github.com/nebulasio/wiki/blob/master/rpc.md)
- [Admin 모듈](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)

### 다음

수고하셨습니다! 네뷸러스를 즐기기 위해 공식 테스트넷 혹은 메인넷에 참여해주세요!

 [테스트넷 참여](https://github.com/nebulasio/wiki/blob/master/testnet.md)
 [메인넷 참여](https://github.com/nebulasio/wiki/blob/master/mainnet.md)
