# Nebulas 101 - 05 Interacting with Nebulas by RPC API

[YouTube Tutorial](https://www.youtube.com/watch?v=to3tkwFjVXo)

Nebulas chain node can be accessed and controlled remotely through RPC. Nebulas chain provides a series of APIs to get node information, account balances, send transactions and deploy calls to smart contracts.

The remote access to the Nebulas chain is implemented by [gRPC](https://grpc.io), and also could be accessed by HTTP via the proxy ([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)). HTTP access is a interface implemented by RESTful, with the same parameters as the gRPC interface.

## API

We've implemented RPC server and HTTP sercer to provide API service in Go-Nebulas.

### Modules

All interfaces are divided into two modules: API and Admin.

- API: Provides interfaces that are not related to the user's private key.
- Admin: Provides interfaces that are related to the user's private key.

It's recommended for all Nebulas nodes to open API module for public users and Admin module for authorized users.
Node: (The admin api on Nebulas testnet & mainnet doesn't support remote call)

### Configuration

RPC server and HTTP server can be configured in the configuration file of each Nebulas node.

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

### Example

#### HTTP

Here is some examples to invoke HTTP interfaces using `curl`.

##### GetNebState

We can invoke `GetNebState` in API module to fetch the current state of local Nebulas node, including chain identity, tail block, protocl version and so on.

```bash
> curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nebstate

{"result":{"chain_id":100,"tail":"0aa1cceb7801b110fdd5217ba0a4356780c940133924d1c1a4eb60336934dab1","lib":"0000000000000000000000000000000000000000000000000000000000000000","height":"479","protocol_version":"/neb/1.0.0","synchronized":false,"version":"0.7.0"}}
```

##### UnlockAccount

We can invoke `UnlockAccount` in Admin module to unlock an account in memory. All unlocked accounts can be used to send transactions directly without passphrases.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz", "passphrase": "passphrase"}'

{"result":{"result":true}}
```

#### RPC

RPC server is implemented with [GRPC](https://grpc.io/). The serialization of GPRC is based on [Protocol Buffers](https://github.com/google/protobuf). You can find all rpc protobuf files in [Nebulas RPC Protobuf Folder](https://github.com/nebulasio/go-nebulas/tree/develop/rpc/pb).

Here is some examples to invoke rpc interfaces using `golang`.

##### GetNebState

We can invoke `GetNebState` in API module to fetch the current state of local Nebulas node.

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPC server connection address configuration
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// API interface to access node status information
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), & rpcpb.GetNebStateRequest {})
if err != nil {
    log.Println("GetNebState", "failed", err)
} else {
    log.Println("GetNebState tail", resp)
}
```

##### LockAccount

Account `n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz` has been unlocked after invoking `v1/admin/account/unlock` via HTTP request above. We can invoke `LockAccount` in Admin module to lock it again.

```go
import(
    "github.com/nebulasio/go-nebulas/rpc"
    "github.com/nebulasio/go-nebulas/rpc/pb"
)

// GRPC server connection address configuration
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
    log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// Admin interface to access, lock account address
admin := rpcpb.NewAdminServiceClient(conn)
from := "n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz"
resp, err = management.LockAccount(context.Background(), & rpcpb.LockAccountRequest {Address: from})
if err != nil {
    log.Println("LockAccount", from, "failed", err)
} else {
    log.Println("LockAccount", from, "result", resp)
}
```

### API List

For more interfaces, please refer to the official documentation:

- [API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md)
- [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md).

### Next

Nice job! Let's join official Testnet or Mainnet to enjoy Nebulas now!

 [Join to Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)
 [Join to Mainnet](https://github.com/nebulasio/wiki/blob/master/mainnet.md)
