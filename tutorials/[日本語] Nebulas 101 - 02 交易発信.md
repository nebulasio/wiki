# Nebulas 101 - 02 Nebulasで交易を発信

[Youtube チュートリアル](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> このチュートリアルの部分には、上の章[コンパイルとインストール](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B%E6%97%A5%E6%9C%AC%E8%AA%9E%5D%20Nebulas%20101%20-%2001%20%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%AB%E3%81%A8%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB.md)に続く。

Nebulasは三つの交易発信方法を提供する:

1. サイン & センド
2. パスフレーズで発信する
3. アンロック & センド

ここはNebulasで上の三つの方法を通じて交易発信するのを紹介して、交易が成功するかどうかを検証する。

## アカウントの準備

Nebulasで、アドレスごとにユニークなアカウントを意味する。

二つのアカウントを準備: 一つはトークンを発送する（発送側のアドレスは"from"と呼ぶ）、もう一つはトークンを受け取る（受け取るのアドレスは"to"と呼ぶ）。

### 発信側

ここは`conf/example/miner.conf`にコインベースのアカウントを使用して、`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`を発信者として設置する。マイナーのコインベースアカウントとして、いくつかのトークンが発掘報酬として受け取っている。その後は別のアカウントにこれらのトークンを発送することができる。

### 受信側

新しいワーレットを作って、トークンを受け取る。

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> このコマンドを運行するとき、`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`とは別のワーレットアドレスが生成する。生成したアドレスを受信アドレスとして使用してください。

新しいワーレットのキーストアファイルは `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`にある。

## ノードを起動

### シードノードを起動

はじめは、ローカルプライベートチェーンにシードノードを最初のノードとして起動する。

```bash
./neb -c conf/default/config.conf
```

### マイナーノードを起動

次いで、マイナーノードを起動してシードノードに連結する。このノードはローカルプライベートチェーンに新たなブロックを生成する。

```bash
./neb -c conf/example/miner.conf
``` 

> **新しいブロックが作成されるまでの期間は？**
> 
> Nebulasで、Proof-of-Devotion(PoD, [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)に記載している)が準備完了以前に、DPoSが一時的なコンセンサスアルゴリズムとして選ばれた。このコンセンサスアルゴリズムに、それぞれのマイナーは15秒毎に新たなブロックを発掘する。
> 
> 今のコンテキストに、`conf/default/genesis.conf`にある21のマイナーの中で、ただ一つのマイナーが作業しているため、新しいブロックを生成するの間に、315(=15*21)秒の時間を待ってなければならない。

一度新しいブロックがマイナーによって発掘されると、発掘報酬が`conf/example/miner.conf`に使用するコインベースのアドレスに追加される、ここは`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`を意味する。

## ノードに対話

Nebulasは開発者が運行しているノードに対話するために、HTTP API、gRPC APIとCLIを提供する。ここは、どう交易を送信するかをHTTP
API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)) で三つの方法の使用を共有する。　

> NubelasのHTTPリスナーはノード配置に定義されている。シードノードのデフォルトポートは`8685`。

はじめに、交易を発信する前に、発信者のバランスをチェックする。

### アカウントの状態をチェック

`curl`を使用して、APIモジュールにある`/v1/user/accountstate`で発信者のアカウント`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`の状態をフェッチする。

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

> **注意**
> タイプはスマートコントラクトアカウントかどうかをチェックする。`88`はスマートコントラクトアカウントで、`87`はスマートコントラクトアカウントではない。

見ている通り、受信者は新たなブロックを発掘するために、いくつかのトークンに報われる。

次いで、受信者のアカウント状態をチェックする。

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

新しいアカウントは予想通り何のトークンも持っていない。

### 交易を発信

三つの方法で交易を発信して発信者から受信者にいくつかのトークンを転送しましょう。

#### サイン & センド

この形で、一つの交易をオフライン環境でサインして、別のオンラインノードに差し出すことができる。これは誰もが自分のプライベートキーをインターネットに公開することなく交易を差し出すことで最も安全的な方法である。

はじめに、交易をサインして生データを取得する。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **注意**
> ノンスは交易には貴重的な属性を持っている。それは [replay attacks](https://en.wikipedia.org/wiki/Replay_attack) に防止するために設計する。特定のアカウントにすれば、ノンスNの交易がアクセプトしたことに限って、ノンスN+1の交易がプロセスされる。したがって、新しい交易を準備する前に、チェーンでアカウントの最新ノンスをチェックしなければならない。

次いで、生データをオンラインNebulasノードに発送する。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}
```

#### パスフレーズで発信

Nebulasノードを信頼するならば、キーストアファイルをそれに委託することができる。そうであれば二番目の方法は適している。

はじめに、信頼のNebulasノードでキーストアファイルをキーディレクターフォルダにアップロードする。

次いで、パスフレーズで交易を発信する。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **注意**
> このアカウント`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`からはノンス1で一つの交易を発信することがあるので、同じ`from`で新しい交易が1で増加しているはず、すなわち、2となる。

#### アンロック & センド

これは最も危険な方法である。受信のNebulasノードを完全的に信頼していないかぎり、おそらくそれを使用すべきではない。

はじめに、信頼のNebulasノードでキーストアファイルをキーディレクターフォルダにアップロードする。

次いで、ノードに与えられた期限のために、パスフレーズでアカウントをアンロックする。期限の単位はナノセカンド(300000000000=300s)。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

このアカウントをアンロックした後、誰もが期限のなかに許可なしで直接にあのノードに任意の交易を発信することができる。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}
```

## 交易レシート

交易を成功に発信するあと、三つの方法で`txhash`を取得する。この`txhash`の値は交易状態にクエリすることができる。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}
```

`status`フィールドは0か1か、または2である。

- **0: 失敗。** この交易はチェーンに差し出した、でも実行失敗することを意味する。
- **1: 成功。** この交易はチェーンに差し出した、そして実行成功することを意味する。
- **2: ペンディング。** この交易はブロックにパックされていないと意味する。

### ダブルチェック

受信者のバランスをダブルチェックしましょう。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

ここでは実行した全ての成功の転送合計であるバランスが表示されている。

### 次の章: チュートリアル 3

 [JavaScriptでスマートコントラクトを作って走る](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B%E6%97%A5%E6%9C%AC%E8%AA%9E%5D%20Nebulas%20101%20-%2003%20%E3%82%B9%E3%83%9E%E3%83%BC%E3%83%88%E3%83%BB%E3%82%B3%E3%83%B3%E3%83%88%E3%83%A9%E3%82%AF%E3%83%88%E5%89%B5%E4%BD%9C.md)
