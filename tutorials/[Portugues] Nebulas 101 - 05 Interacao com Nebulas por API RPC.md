# Nebulas 101 - 05 Interação com Nebulas através do API RPC

[Tutorial YouTube](https://www.youtube.com/watch?v=to3tkwFjVXo)

A chain de nós Nebulas pode ser acedida e controlada remotamente através de RPC. A chain de Nebulas fornece uma série de APIs para obter informação dos nós, balanço de contas, envio de transações, e implementação de chamadas de smart contracts.

O acesso remoto à chain de Nebulas é implementado por [gRPC](https://grpc.io), e pode também ser acedido através de HTTP por um proxy ([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)). Acesso HTTP é uma interface implementada por RESTful, com os mesmos parâmetros de uma interface gRPC.

## API

Implentamos um servidor RPC e HTTP para fornecer um serviço API em Go-Nebulas.

### Módulos

Todas as interfaces estão divididas em dois módulos: API e Admin.

- API: Fornece interfaces que não têm relação com a chave privada do utilizador.
- Admin: Fornece interfaces que estão relacionadas com a chave privada do utilizador.

É recomendado para todos os nós de Nebulas que abram o módulo API para utilizadores públicos, e o módulo Admin para utilizadores autorizados.
Nó: (O API Admin na testnet & mainnet da Nebulas não suporta chamadas remotas)

### Configuração

Servidor RPC e HTTP pode ser configurado no ficheiro de configuração de cada nó de Nebulas.

```protobuf
rpc {
 # Porta do serviço API gRPC
 rpc_listen: ["127.0.0.1:8684"]
 # Porta do serviço API HTTP
 http_listen: ["127.0.0.1:8685"]
 # Módulo aberto que fornece serviço http para fora
 http_module: ["api", "admin"]
}
```

### Exemplo

#### HTTP

Aqui estão alguns exemplos para invocar as interfaces HTTP usando `curl`.

##### GetNebState

Pode invocar `GetNebState` através do módulo API para obter o estado corrente do nó local de Nebulas, incluíndo a identidade da chain, bloco da cauda (tail block), versões do protocolo, entre outros.

```bash
> curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nebstate

{"result":{"chain_id":100,"tail":"0aa1cceb7801b110fdd5217ba0a4356780c940133924d1c1a4eb60336934dab1","lib":"0000000000000000000000000000000000000000000000000000000000000000","height":"479","protocol_version":"/neb/1.0.0","synchronized":false,"version":"0.7.0"}}
```

##### UnlockAccount

Pode invocar `UnlockAccount` através do módulo Admin para desbloquear uma conta na me memória. Todas as contas desbloqueadas podem ser utilizadas para enviar transações directamente sem senhas.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz", "passphrase": "passphrase"}'

{"result":{"result":true}}
```

#### RPC

Servidor RPC foi implementado com [GRPC](https://grpc.io/). A serialização do GPRC é baseada em [Protocol Buffers](https://github.com/google/protobuf). Pode encontrar todos os ficheiros RPC protobuf no [Directório Nebulas RPC Protobuf](https://github.com/nebulasio/go-nebulas/tree/develop/rpc/pb).

Aqui estão alguns exemplos da invocação de interfaces RPC usando `golang`.

##### GetNebState

Podemos invocar `GetNebState` através do módulo API para obter o estado corrente do nó local de Nebulas.

```go
import(
 "github.com/nebulasio/go-nebulas/rpc"
 "github.com/nebulasio/go-nebulas/rpc/pb"
)

// Configuração do endereço de ligação ao servidor GRPC
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
 log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// Interface API para aceder a informação do estado do nó
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), & rpcpb.GetNebStateRequest {})
if err != nil {
 log.Println("GetNebState", "failed", err)
} else {
 log.Println("GetNebState tail", resp)
}
```

##### LockAccount

Conta `n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz` foi desbloqueada após invocar `v1/admin/account/unlock` através do pedido HTTP acima. Pode invocar `LockAccount` através do módulo Admin para a voltar a bloquear.

```go
import(
 "github.com/nebulasio/go-nebulas/rpc"
 "github.com/nebulasio/go-nebulas/rpc/pb"
)

// Configuração do endereço de ligação ao servidor GRPC
addr := fmt.Sprintf("127.0.0.1:%d",uint32(8684))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
 log.Warn("rpc.Dial() failed:", err)
}
defer conn.Close()

// Interface Admin para aceder, bloquear endereço de conta
admin := rpcpb.NewAdminServiceClient(conn)
from := "n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz"
resp, err = management.LockAccount(context.Background(), & rpcpb.LockAccountRequest {Address: from})
if err != nil {
 log.Println("LockAccount", from, "failed", err)
} else {
 log.Println("LockAccount", from, "result", resp)
}
```

### Lista API

Para mais interfaces, refira-se à documentação oficial:

- [Mõdulo API](https://github.com/nebulasio/wiki/blob/master/rpc.md)
- [Módulo Admin](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md).

### Próximo

Bom trabalho! Vamos juntar-nos à Testnet ou Mainnet oficial para desfrutar de Nebulas agora!

 [Join to Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)
 [Join to Mainnet](https://github.com/nebulasio/wiki/blob/master/mainnet.md)
