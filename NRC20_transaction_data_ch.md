## 星云NRC20交易数据说明


###交易数据

eg:

```
curl -i -H 'Content-Type: application/json' -X POST https://mainnet.nebulas.io/v1/user/getTransactionReceipt -d '{"hash":"61b5c6e3e006989739427a234e04eb4875e40c09328bffa7d54583eba5261a3b"}'
```

返回数据

```json
{
    "result":{
        "hash":"61b5c6e3e006989739427a234e04eb4875e40c09328bffa7d54583eba5261a3b",
        "chainId":1,
        "from":"n1VvDG4tvkKUEAnQ4yDoMR4nXFyWvLQo6pR",
        "to":"n1zUNqeBPvsyrw5zxp9mKcDdLTjuaEL7s39",
        "value":"0",
        "nonce":"8",
        "timestamp":"1559717188",
        "type":"call",
        "data":"eyJGdW5jdGlvbiI6InRyYW5zZmVyIiwiQXJncyI6IltcIm4xYmppczRMRm9STDFEZ3J2R2pvMnhzZlJGQVNYR0NaMmZaXCIsXCIxNzc1NzAwMDAwMDAwMDAwMDAwMDAwMFwiXSJ9",
        "gas_price":"20000000000",
        "gas_limit":"330000",
        "contract_address":"",
        "status":1,
        "gas_used":"21015",
        "execute_error":"",
        "execute_result":"""",
        "block_height":"2482518"
    }
}
```

data数据base64解析

```json
{
    "Function": "transfer",
    "Args": "[\"n1bjis4LFoRL1DgrvGjo2xsfRFASXGCZ2fZ\",\"17757000000000000000000\"]"
}
```

NRC20转账

- Func: transfer
- Args: 交易参数，数组json序列化字符串
	- addr: 交易转账接受地址
	- value: 交易金额，目前接受参数只支持数据字符串。

####数据说明

- type: 交易类型
 - binary: 普通类型，可在data中附加binary数据。to地址为合约地址时默认调用合约的accept方法。（合约未写accept方法时交易会执行失败）
	- deploy: 合约部署类型。
	- call: 合约调用类型。
	- protocol: nbre部署code类型。
	- dip: DIP奖励类型。nebulas定期提交，普通用户无法提交。
- chainId: 交易链id。
- from: 交易发起地址。
- to: 交易接受地址。
- value: 交易的NAS金额，从from转账到to地址，值为非负整数字符串，单位为wei。eg: “0”,”100”
- nonce: from发起的交易序列号。从1开始。
- data: 交易数据，接口查询为base64编码，decode后为json字符串。
	- deploy: 
		- Source: 合约代码
		- SourceType: 合约代码类型，参数为js（合约为JavaScript）和ts（合约代码为typeScript）。
		- Args: 合约初始化参数，为空字符串或数组的json序列化数据。
	- call:
		- Func: 合约调用方法
		- Args: 合约初始化参数，为空字符串或数组的json序列化数据。
	- protocol: 
		- Data: 协议代码binary数据	
- timestamp: 交易时间戳。
- status: 交易状态。0失败，1成功，2pending状态（交易未上链，打包过程中）
- execute_error: 交易失败时的失败原因
- execute_result: 交易的执行结果，交易类型为deploy或call合约调用时，合约的执行结果
- block_height: 区块高度。

**NRC20交易类型为call，调用NRC20合约的转账方法。**

###event数据

 event记录交易的执行结果。

```
curl -i -H 'Content-Type: application/json' -X POST https://mainnet.nebulas.io/v1/user/getEventsByHash -d '{"hash":"61b5c6e3e006989739427a234e04eb4875e40c09328bffa7d54583eba5261a3b"}'
```

返回数据

```json
{
    "result":{
        "events":[
            {
                "topic":"chain.contract.Atlas Protocol",
                "data":"{"Status":true,"Transfer":{"from":"n1VvDG4tvkKUEAnQ4yDoMR4nXFyWvLQo6pR","to":"n1bjis4LFoRL1DgrvGjo2xsfRFASXGCZ2fZ","value":"1.7757e+22"}}"
            },
            {
                "topic":"chain.transactionResult",
                "data":"{"hash":"61b5c6e3e006989739427a234e04eb4875e40c09328bffa7d54583eba5261a3b","status":1,"gas_used":"21015","error":"","execute_result":"\"\""}"
            }
        ]
    }
}
```

#### 交易topic
	
- chain.transactionResult: 交易的最终执行结果event，星云主网记录
	- hash: 交易哈希
	- status: 交易执行的结果。0失败，1成功
	- gas_used: 交易执行消耗的gas
	- execute_result: 交易执行结果
	- error: 交易执行错误
- chain.contract.***: 合约记录的event，合约记录
	- eg: chain.contract.Atlas Protocol
		- Status: ATP执行结果
		- Transfer: 交易数据
			- from:交易发起地址
			- to:交易接受地址
			- value: 交易金额 ，参数类型为字符串，可以为科学记数法。
					eg: “0”,”1.7757e+22”
					
					
## 判断NRC20转账成功的方法

- 交易状态`status`为1
- 交易金额查询
	- tx data数据解析第二个参数
- 查询地址NRC20余额
	- 调用合约的`balanceOf`方法查询

**[附NRC20标准说明](https://github.com/nebulasio/wiki/blob/master/NRC20.md)**
 