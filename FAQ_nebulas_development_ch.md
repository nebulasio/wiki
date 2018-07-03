## 环境搭建相关问题

### 基础篇
请问如何在本地启动我的节点呢？

A：可以参考官方的tutorial： https://github.com/nebulasio/wiki/tree/master/tutorials 。 遇到了问题可以在issue中搜索一下是不是有其他小伙伴也遇到啦。

参加官方激励计划必须要在本地搭建节点嘛？

A：完全不用，可以使用web-wallet (https://github.com/nebulasio/web-wallet ) 将合约部署在官方的testnet及mainnet上。

哪里能看到钱包的基础教程呢？

A: 可以使用官方发布的web-wallet 。教程参见官方发布的功能介绍 https://blog.nebulas.io/2018/04/12/creating-a-nas-wallet/ 

我是mac操作环境，为什么我照教程安装的库(go, rocksDB)版本不对呢？

A：在社区也有一些小伙伴遇到了类似的问题。大多数的原因是brew update更新brew自身的版本，然后安装的依赖就能兼容啦。

我是windows操作环境，可以在本地启动节点吗？

A：可以在本地安装虚拟机启动节点。星云链暂时不支持在window下直接启动，后续会提供支持。如果有热心积极的小伙伴提供技术支持一定会拿到官方丰厚的奖励。

我是windows环境，能够部署智能合约参加这次的激励计划嘛？
A：可以的，web-wallet等组件支持在windows环境下运行。

###进阶篇

我在本地按照官方的教程搭建了节点进行测试，有没有什么办法修改默认设置从而更快的调试呢？

A：当然有办法啦，可以修改每个出block的时间间隔及朝代数目。详情参见：
https://github.com/nebulasio/go-nebulas/issues/109

2.  安装完成之后，在本地启动neb，报错make build libnebulasv8.so cannot open shared object file: No such file or directory 是什么情况?

A：通常是由于缺少v8文件所致。请仔细检查是否在安装的过程中漏掉了make deploy-v8。或尝试按https://github.com/nebulasio/go-nebulas/issues/93 的方案解决问题。

3. Git clone的时候代码下载过慢肿么办？

A：需要下载的内容较大时网络抖动会造成下载失败。可以尝试配置代理，具体参考 https://github.com/nebulasio/go-nebulas/issues/88 。

4. 在往节点发送请求时，偶尔会出现 “503 error ("error": "all SubConns are in TransientFailure”） 是什么情况？

A：有两种情况。一种是neb节点的网络连接数过多，触及了上限，此种情况稍后重试即可。另一种是本地首次编译启动后遇见，此时可以尝试重启机器解决。由于第二种情况问题较难复现，怀疑原因可能是系统部分资源没有释放，官方会继续寻找更好的解决方案。

5. 在往自己搭建的节点发送请求时，chrome后台显示501 not implemented 服务器不支持OPTIONS请求，想问下这个是不是nebulas本身不支持呢？

A：通常问题是因为在本地的配置文件中没有开启跨域配置，修改本地default.conf：
rpc {
...
http_module: ["api","admin"]
# HTTP CORS allowed origins
+  http_cors: ["*"]
}

##开发相关问题：

### 基础篇
1. 怎么才能快速上手开发DApp呢？ 

官方推出教程循序渐进地为星云爱好者们介绍如何从零开始在星云链上开发自己的DApp。参见官方博客：
https://blog.nebulas.io/2018/05/04/how-to-build-a-dapp-on-nebulas-part-1/ 
https://blog.nebulas.io/2018/05/05/how-to-build-a-dapp-on-nebulas-part-2/ 

2. 怎么使用钱包与星云链交互呢？

A: 基础教程参见官方发布的功能介绍 https://blog.nebulas.io/2018/04/12/creating-a-nas-wallet/ 

社区也有小伙伴积极地整理了部署合约的注意事项：
https://mp.weixin.qq.com/s?__biz=MzI3NzExODg4OA==&mid=2650822384&idx=1&sn=e985d2b5bbdb88638e21959568dbdb0a&scene=21#wechat_redirect

3. 部署的合约内容如何查询呢？

A: 可以通过部署合约返回的transaction hash在 web-wallet 或https://explorer.nebulas.io/#/ 查询完成状态。

4. 已部署的合约可以直接升级吗?或者支持数据迁移到新的合约吗?

A: 不支持合约升级，只能重新部署合约。也暂不支持合约数据迁移。

5. 发送合约需要消耗gas嘛，上限下限是多少呢？

A: Gas是发布合约所必须消耗的。在星云链上开发消耗的gas很低，gaslimit的范围为gaslimit [20000, 50000000000]。gasprice默认为10^wei。注意1NAS=1^18wei哦，实际消耗的是很低的。

6. 怎么查询我的账户的交易记录？

A: 在https://explorer.nebulas.io/#/输入你的账户地址即可

7. 为什么我发送的交易一直处于pending状态呢？

A: 通常原因是你在发送交易时输入了一个过大的nonce。nonce是在执行时按顺序执行的。可以查询账户当前状态（getAccontState）来确保你发送的nonce是连续的。

8. 为什么我查不到我部署的合约呢？

A：通常有两种可能。第一种是部署时失败，可以通过返回的transaction hash在 web-wallet 或https://explorer.nebulas.io/#/ 查询完成状态。第二种情况是你不小心设置错了网络，弄错了testnet/mainnet

9. 能否能通过合约地址查询合约内容？

A：目前查询合约状态是通过transaction hash来查询的，官方已经于今日完成了通过合约地址查询的功能，将很快上线，敬请期待。

10. 如果我的合约有个bug，想修补，怎么办？

A：重新部署一个合约。将你的DApp界面请求指向新的合约。

11. 在测试网测试时，怎么申请nas代币呢？

A: https://testnet.nebulas.io/claim/ 

12. 智能合约的console.log会打印在哪里？

A：目前打印在neb节点的后台日志中，不会在返回中返回。如果有的小伙伴有打印日志的需求，可以参见文档在本地启动节点。

13. 考虑到用户上传私钥会有顾虑，那么如何在web上方便的开发DApp呢？
A：可以使用nebPay来处理支付信息，nebpay会调用浏览器插件或钱包app Nas Nano 来完成支付.
社区热心人士开发了Chrome插件钱包(https://github.com/ChengOrangeJu/WebExtensionWallet)。该钱包提供发送交易/导入钱包/调用合约方法/查询交易状态等功能，能够显著的减少web开发者开发DApp时的工作量。

###进阶篇

1.  如何在合约内部输出一些信息呢?

A：可以在合约内部发布一些event，支持订阅查询，详细参见：https://github.com/nebulasio/wiki/blob/master/smart_contract.md#event 

2. 为什么我调用了很多次合约都成功了，查询数据却迟迟没有变化呢？

A：一个可能是你在程序中用的是api.call的方法而不是sendtransaction的方法。call方法是在节点上模拟执行，所得结果并不会上链，也不消耗gas。

3. 如何为每个用户动态创建一个map呢？

A：可以在智能合约的LocalStorageMap内使用该用户的地址当做key，value定义为一个Object。

4. 能在智能合约中使用随机数嘛？

A：在1.0.2版本中，智能合约的执行环境支持随机数生成，在链上添加随机数生成机制，保证随机性并在执行时可验证。使得Nvm能为更多应用场景，如数字娱乐、游戏提供功能支持。已经在测试网及主网上线。


5. 怎么样才能更快的调试我的智能合约呢？

A：可以参加如何在本地启动节点的教程，在本地节点进行调试。

6. 现在Nebulas的智能合约支持先知机的使用吗？

A: 非常好的问题，在星云链近期的周报中，已经将不同智能合约之间进行数据的访问列为已完成，很快将在测试网和主网陆续上线。当合约调用的功能发布之后，相信很快会有先知机出现。敬请期待。

7. 怎么遍历LocalStorageMap的全部内容呢？
A：详细解决方案参见wiki：
https://github.com/nebulasio/wiki/blob/master/tutorials/%5B%E4%B8%AD%E6%96%87%5D%20Nebulas%20101%20-%2004%20%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E5%AD%98%E5%82%A8%E5%8C%BA.md#map%E6%95%B0%E6%8D%AE%E9%81%8D%E5%8E%86


8. 星云的合约是像以太一样吗？可以给这个合约打NAS，然后创建合约的owner可以调用合约methodwithdrawNAS到自己的NAS账户里面？

A：星云链智能合约内部可以调用Blockchain.transfer进行nas的转账。

9. 怎么在智能合约内部查询账户余额？

A：目前没有最直接的方式查询，将于近期支持。替代方案可以是可以在智能合约函数内维护一个变量，追踪blockchain.transaction.value的累计变化。

10. 我想做一个密码管理的合约，内容会不会被别人看到呢？

A：你在调用合约的过程中发送的所有数据都会被其他人查看到。如果要求私密性，可以将参数加密后再调用合约函数方法。

11. 我想在星云链上存储图片，怎么破？

A：一次transaction的数据长度限制为128k，如果您需要在链上存储数据可以考虑将图片数据拆成多个分片，分多次transaction完成上传。另一种替代方案是您可以将图片存储在别的地方，将图片的hash校验存储在星云链上保证数据的一致性。

12. 如何在vscode中调试星云链的代码呢？

A：社区热情小伙伴整理了教学，参考http://www.nasforce.io/index.php?c=read&id=53&page=1 

13.  在合约内使用 Blockchain.transfer 向钱包地址转账时，可以查到该转账的交易记录吗？如果没有交易记录，怎么向钱包用户证明合约的这笔转账呢？

A：使用  Blockchain.transfer 完成的转账是没有转账记录的，但是会有event记录，可以通过 GetEventsByHash 查询本次合约调用的交易hash来获取这些events。也可以直接使用 nebulas explorer 查询该交易hash来查看这些转账的events。

14. 当发送合约调用的交易时，可以得到调用的合约函数的返回值吗？

A: 可以的。具体操作为：使用API GetTransactionReceipt 通过交易hash 得到 receipt，receipt中的 execute_result 和 execute_error 分别是函数的返回值和本次交易执行是的错误信息。


## 开发辅助工具篇

DApp如何获取用户的钱包地址呢?
目前社区办的浏览器插件支持获取用户的wallet address，可以参考issue：https://github.com/ChengOrangeJu/WebExtensionWallet/issues/6

用户使用web-wallet调用合约时的常见错误： execute_err: "contract check failed”
A：这是新手们频繁遇到的错误，这时候需要检查使用的合约地址是否正确，是否是部署合约时返回的合约地址。

Web-wallet或nebpay调用合约时的参数格式错误。
A：新手们经常遇到的错误，调用合约的参数格式是参数数组的JSON字符串，可以先创建一个参数数组，然后用JSON.stringify()得到正确的参数格式。也可以使用JSOM.parse()检查参数是否有错。参见issue：https://github.com/nebulasio/nebPay/issues/2, https://github.com/nebulasio/web-wallet/issues/38

使用NebPay的queryPayInfo查询不到交易，“payId **** does not exist“”。
A： 需要注意options参数中的网络选择，详见NebPay的文档。


## 附录常用链接

官网：https://nebulas.io

非技术白皮书：https://nebulas.io/docs/NebulasWhitepaperZh.pdf

技术白皮书：https://nebulas.io/docs/NebulasTechnicalWhitepaperZh.pdf

Wiki: https://github.com/nebulasio/wiki

Tutorials: https://github.com/nebulasio/wiki/tree/master/tutorials

主链代码: https://github.com/nebulasio/go-nebulas

浏览器代码: https://github.com/nebulasio/explorer

网页钱包代码: https://github.com/nebulasio/web-wallet

Web SDK: https://github.com/nebulasio/neb.js

NebPay 源码:  https://github.com/nebulasio/nebPay

NebPay 官方文档: https://github.com/nebulasio/nebPay/blob/master/doc/

NebPay 使用教程:

英文版:  https://medium.com/nebulasio/how-to-use-nebpay-in-your-dapp-8e785e560fbb

中文版: https://blog.nebulas.io/how-to-use-nebpay-in-your-dapp/

测试网：https://github.com/nebulasio/wiki/blob/master/testnet.md

主网：https://github.com/nebulasio/wiki/blob/master/mainnet.md
区块浏览器: https://explorer.nebulas.io
领取测试NAS：https://testnet.nebulas.io/claim/ 
星云NRC20官方说明：https://github.com/nebulasio/wiki/blob/master/NRC20.md 
激励计划页面：https://incentive.nebulas.io/ 