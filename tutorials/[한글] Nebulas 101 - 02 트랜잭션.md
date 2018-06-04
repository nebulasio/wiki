# Nebulas 101 - 02 네뷸러스에서 트랜잭션 전송하기

[유튜브 튜토리얼](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> For this portion of the tutorial we will pick up where we left off in the [Installation tutorial](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).
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

Here we will use the coinbase account in the `conf/example/miner.conf`, which is `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` as the sender. As the miner's coinbase account, it will receive some tokens as the mining reward. Then we could send these tokens to another account later.
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

Firstly, start a seed node as the first node in local private chain.

```bash
./neb -c conf/default/config.conf
```

### Start Miner Node

Secondly, start a miner node connecting to the seed node. This node will generate new blocks in local private chain.

```bash
./neb -c conf/example/miner.conf
``` 

> **How long a new block will be minted?**
> 
> In Nebulas, DPoS is chosen as the temporary consensus algorithm before Proof-of-Devotion(PoD, described in [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) is ready. In this consensus algorithm, each miner will mint new block one by one every 15 seconds.
> 
> In current context, we have to wait for 315(=15*21) seconds to get a new block because there is only one miner among 21 miners defined in `conf/default/genesis.conf` working now.

Once a new block minted by the miner, the mining reward will be added to the coinbase wallet address used in `conf/example/miner.conf` which is `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## Interact with Nodes

Nebulas provides developers with HTTP API, gRPC API and CLI to interact with the running nodes. Here, we will share how to send a transaction in three methods with HTTP API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 

> The Nebulas HTTP Lisenter is defined in the node configuration. The default port of our seed node is `8685`.

At first, check the sender's balance before sending a transaction.

### Check Account State

Fetch the state of sender's account `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` with `/v1/user/accountstate` in API Module using `curl`.

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

> **Note**
> Type is used to check if this account is a smart contract account. `88` represents smart contract account and `87` means a non-contract account.

As we see, the receiver has been rewarded some tokens for mining new blocks.

Then let's check the receiver's account state.

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

The new account doesn't have tokens as expected.

### Send a Transaction

Now let’s send a transaction in three methods to transfer some tokens from the sender to the receiver!

#### Sign & Send

In this way, we can sign a transaction in an offline environment and then submit it to another online node. This is the safest method for everyone to submit a transaction without exposing your own private key to the Internet. 

First, sign the transaction to get raw data.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Note**
> Nonce is an very important attribute in a transaction. It's designed to prevent [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). For a given account, only after its transaction with nonce N is accepted, will its transaction with nonce N+1 be processed. Thus, we have to check the latest nonce of the account on chain before preparing a new transaction.

Then, send the raw data to an online Nebulas node.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}⏎
```

#### Send with Passphrase

If you trust a Nebulas node so much that you can delegate your keystore files to it, the second method is a good fit for you.

First, upload your keystore files to the keydir folders in the trusted Nebulas node.

Then, send the transaction with your passphrase.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Note**
> Because we have sent a transaction with nonce 1 from the account `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, new transaction with same `from` should be increased by 1, namely 2.

#### Unlock & Send

This is the most dangerous method. You probably shouldn’t use it unless you have complete trust in the receiving Nebulas node.

First, upload your keystore files to the keydir folders in the trusted Nebulas node.

Then unlock your accounts with your passphrase for a given duration in the node.
The unit of the duration is nano seconds (300000000000=300s).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

After unlocking the account, everyone is able to send any transaction directly within the duration in that node without your authorization.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}⏎
```

## Transaction Receipt

We'll get a `txhash` in three methods after sending a transaction successfully. The `txhash` value can be used to query the transaction status.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}⏎
```

The `status` fields may be 0, 1 or 2.

- **0: Failed.** It means the transaction has been submitted on chain but its execution failed.
- **1: Successful.** It means the transaction has been submitted on chain and its execution successeed.
- **2: Pending.** It means the transaction hasn't been packed into a block.

### Double Check

Let's double check the receiver's balance. 

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

Here you should see a balance that is the total of all the successful transfers that you executed.

### Next step: Tutorial 3

 [Write and run a smart contract with JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
