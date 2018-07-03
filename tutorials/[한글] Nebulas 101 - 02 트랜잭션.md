# Nebulas 101 - 02 네뷸러스에서 트랜잭션 전송하기

[유튜브 튜토리얼](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> 튜토리얼의 이 부분에서는 우리가 [설치 튜토리얼](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md)에서 멈췄던 부분부터 살펴보겠습니다.

네뷸러스는 트랜잭션을 전송하는 방법 3가지를 제공합니다:

1. 서명 & 전송
2. 비밀번호와 함께 전송
3. 잠금해제 & 전송

위 3가지 방법을 통해 네뷸러스에서 트랜잭션을 전송하고, 트랜잭션이 성공적이었는지 확인하는 내용입니다.

## 주소 준비하기

네뷸러스에서 각 주소는 고유한 계정을 나타냅니다.

두 주소를 준비하세요: 토큰을 보낼 주소("from"이라고 불리는 보내는 주소)와 토큰을 받을 주소("to"라고 불리는 받는 주소)

### 보내는 사람

여기서 우리는 `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`를 전송자 주소로써 `conf/example/miner.conf`에 있는 코인베이스 계정을 사용할 것입니다. 채굴자의 코인베이스 계정으로써, 채굴 보상으로써 토큰을 받을 것입니다. 그 때, 우리는 이 토큰을 다른 주소로 보낼 수 있습니다.

### 받는 사람

토큰을 받을 새 주소를 생성하세요.

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> 이 명령어를 실행할 때, `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`와 다른 지갑 주소를 얻을 것입니다. 생성된 주소를 받는 주소로써 사용해주세요.

새 주소의 키스토어 파일은 `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`에 있습니다.

## 노드 시작하기

### 시드노드 시작하기

먼저, 로컬 체인에 첫 번째 노드인 시드 노드를 시작합니다.

```bash
./neb -c conf/default/config.conf
```

### 마이너노드 시작하기

다음으로, 시드노드와 연결되어 있는 마이너노드를 시작합니다. 이 노드는 로컬 체인에서 새 블록을 생성할 것입니다.

```bash
./neb -c conf/example/miner.conf
``` 

> **새 블록이 채굴되는데 얼마나 걸릴까?**
> 
> 네뷸러스에서, DPoS가 기여도 증명 합의 알고리즘([기술백서](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)에서 설명되어 있는 PoD)이 준비되기 전에 임시 합의 알고리즘으로 채택되었습니다. 이 합의 알고리즘에서, 각 채굴자들은 매 15초마다 한 블록을 채굴합니다.
> 
> 현재 맥락에서, 우리는 새 블록을 얻기까지 315(=15*21)초를 기다려야 합니다. 왜냐하면 지금 작동하고 있는 `conf/default/genesis.conf`에서 정의된 21명의 채굴자 중에 오직 한 명의 마이너만 존재하기 때문입니다.

새 블록이 마이너에 의해 채굴되면, 채굴 보상은 `conf/example/miner.conf`에서 사용되고 있는 코인베이스 지갑 주소 `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`에 지급될 것입니다.

## 노드와 상호작용하기

네뷸러스는 작동하고 있는 노드와 상호작용하게 하기 위해 개발자들에게 HTTP API, gRPC API 그리고 CLI를 제공합니다. 여기, 우리는 HTTP API를 사용하여 트랜잭션을 전송하는 방법 세 가지를 볼 수 있습니다 ([API 모듈](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin 모듈](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)).

> 네뷸러스 HTTP 리스너는 노드 설정에 정의되어있습니다. 시드노드의 기본 포트는 `8685`입니다.

먼저, 트랜잭션을 전송하기 전에 전송하는 사람의 잔액을 체크합니다.

### 주소 상태 체크

`curl`을 사용하여 API 모듈에 있는 `/v1/user/accountstate`로 전송자의 주소인 `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`의 상태를 가져옵니다.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'

{
    "result": {
        "balance": "67066180000000000000",
        "nonce": "0",
        "type": 87
    }
}
```

> **주의사항**
> 타입은 주소가 스마트 컨트랙트 주소인지 체크할 때 사용합니다. `88`은 스마트 컨트랙트 주소를 나타내고, `87`은 일반 주소를 나타냅니다.

우리가 보았듯이, 받는 사람은 새 블록을 채굴함으로써 일정량의 토큰을 보상받습니다.

그럼 받는 사람의 주소 상태를 체크하겠습니다.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'


{
    "result": {
        "balance": "0",
        "nonce": "0",
        "type": 87
    }
}
```

새 주소는 예상한대로 토큰을 가지고 있지 않습니다.

### 트랜잭션 전송하기

세 가지 방법으로 일정량의 토큰을 전송하는 트랜잭션을 전송해보겠습니다.

#### 서명 & 전송

이 방법은, 오프라인 환경에서 트랜잭션에 서명할 수 있고 또다른 온라인 노드에 그 트랜잭션을 제출할 수 있습니다. 이것은 인터넷에 자신의 프라이빗 키를 노출시키지 않고 트랜잭션을 보내는 안전한 방법입니다.

먼저, 정제되지 않은 데이터를 얻기 위해 트랜잭션에 서명합니다.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **주의사항**
논스는 트랜잭션에서 매우 중요한 속성입니다. 이것은 [리플레이 어택](https://en.wikipedia.org/wiki/Replay_attack)을 막기 위해 설계되었습니다. 주어진 주소에 대해서, 논스 N을 가지고 있는 트랜잭션이 승인된 후에, 논스 N+1이 된 트랜잭션이 진행될 것입니다. 그러므로, 새 트랜잭션을 준비하기 전에 체인에서 주소의 가장 최신 논스를 체크해야 합니다.

그 때, 온라인 네뷸러스 노드에 정제되지 않은 데이터를 보냅니다.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}⏎
```

#### 비밀번호와 함께 전송하기

키스토어 파일을 위임할 수 있을 정도로 네뷸러스 노드를 신뢰한다면, 두 번째 방법이 좋은 방법이 될 것입니다.

첫 번째로, 키스토어 파일을 믿을 수 있는 네뷸러스 노드에 있는 keydir 폴더에 업로드합니다.

그 때, 비밀번호와 함께 트랜잭션을 전송합니다.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **주의사항**
> 주소 `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`로부터 논스 1의 트랜잭션을 전송했기 때문에, 같은 `from`의 새로운 트랜잭션은 1이 증가한 2가 됩니다.

#### 잠금해제 & 전송

이것은 가장 위험한 방법입니다. 네뷸러스 노드를 받음에 있어서 신뢰를 완성시키지 않았다면 이 방법을 사용하지 말아야할 것입니다.

먼저, 신뢰된 네뷸러스 노드의 keydir 폴더에 키스토어 파일을 업로드합니다.

그 때, 노드에서 주어진 시간동안 비밀번호와 함께 주소는 언락합니다.
시간의 단위는 나노세컨드입니다 (300000000000=300s).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

주소를 잠금해제한 후에, 아무나 권한 없이 그 노드에서 일정기간동안 직접적으로 어떤 트랜잭션이라도 전송할 수 있다.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}⏎
```

## 트랜잭션 영수증

세 가지 방법으로 성공적으로 트랜잭션을 전송한 후에 `txhash`를 얻을 것입니다. `txhash` 값은 트랜잭션 상태를 쿼리하는데 사용될 수 있습니다.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}⏎
```

`status` 필드는 0, 1 혹은 2가 될 수 있습니다.

- **0: 실패.** 트랜잭션이 체인에 제출되었지만 실행이 실패했음을 의미합니다.
- **1: 성공.** 트랜잭션이 체인에 제출되었고 실행이 성공했음을 의미합니다.
- **2: 대기.** 트랜잭션이 블록에 제출되지 않았음을 의미합니다.

### 이중 체크

받은 사람의 잔액을 이중 체크합니다.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

여기에서 실행한 모든 성공적인 전송의 합계를 볼 수 있습니다.

### 다음 단계: 튜토리얼 3

 [자바스크립트로 스마트 컨트랙트 작성 및 실행하기](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
