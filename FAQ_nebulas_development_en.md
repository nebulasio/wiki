
## Quick start tutorial:

#### How to Compile and Install Nebulas
> A: Follow this instruction step by step on [this page](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md)

#### How to Send Transactions on Nebulas
> Nebulas provides three methods to send transactions：
>(1) Sign & Send
>(2) Send with Passphrase
>(3) Unlock & Send

> [Detailed introduction](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md)

#### How to Write and run a smart contract
> Through this [tutorial](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
 we will learn how to write, deploy, and execute smart contracts in Nebulas.

#### How to use Smart Contract Storage
  
> We introduce in [detail](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md)
 the storage of the smart contract. Nebulas smart contracts provide on-chain data storage capabilities. Similar to the traditional key-value storage system (eg: redis), smart contracts can be stored on the Nebulas by paying with (gas).


#### How to Interacting with Nebulas by RPC API
> Nebulas chain node can be accessed and controlled remotely through RPC. Nebulas chain provides a series of APIs to get node information, account balances, send transactions and deploy calls to smart contracts.
> The remote access to the Nebulas chain is implemented by gRPC, and also could be accessed by HTTP via the proxy (grpc-gateway). HTTP access is a interface implemented by RESTful, with the same parameters as the gRPC interface.
> [See detail](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md)


## Advanced questions

### Questions about setting up environment

#### Q:  Do I have to start a local node to deploy and call a smart contract?

> A:  Not necessary. You can use web-wallet (https://github.com/nebulasio/web-wallet ) to deploy the contract on testnet and mainnet.

#### Q: I am using a mac. Why isn’t my library installed correctly?

> A: There are also people from the community who have encountered similar problems. Most of the reason is that you need to run `brew update` to update brew version, and then the installed dependencies are compatible.

#### Q: I am using windows operating system. Can I start the node locally?
> A: You can install the virtual machine to start the node locally. 

#### Q: I am using Windows operating system. Can I deploy smart contracts to participate in this incentive plan?
> A: Yes, web-wallet and other components is runnable under the windows operation system.

#### Q: I built a node in the local according to the official tutorial to test, is there any way to modify the default settings for faster debugging?
> A: Of course there are ways to change the time interval and the number of dynasties for each block. For details see:
https://github.com/nebulasio/go-nebulas/issues/109

#### Q: After the installation is complete, start Neb locally and report an error "Make build libnebulasv8.so cannot open shared object file: No such file or directory". What's going wrong?
> A: Usually due to missing v8 files. Please check carefully if you missed make deploy-v8 during installation. Or try solving the problem at https://github.com/nebulasio/go-nebulas/issues/93.


#### Q: Git clone too slow
> A: Unstable network environment will cause the failure of download. You can try to configure the proxy. For details, refer to https://github.com/nebulasio/go-nebulas/issues/88.

#### Q: When I try to send a request to a node, occasionally "503 error ("error": "all SubConns are in TransientFailure")?
> A: There are two situations. One is that the the upper limit of network connections on a Neb node is reached. You can retry later under this situation.
> The other situation is that after the first `make build`, this time you can try to restart your computer to solve the problem. Since the second situation is more difficult to reproduce, we suspect  that some of the resources in the system may have not been released and we will continue to look for better solutions.

#### Q: When sending a request to a node that is set up by itself, chrome shows 501 not implemented. The server does not support OPTIONS requests. Would you like to ask whether this is not supported by nebulas?
> A: modify local config
>     rpc {
>     ...
>     http_module: ["api","admin"]
>     # HTTP CORS allowed origins
>     +  http_cors: ["*"]
>     }


### Questions about Developing on Nebulas

#### Q: Where to get a quick start of develop a DApp

> The official launch of the tutorial  provide a step-by-step guide for the nebula fans on how to develop their own DApp on the Nebula. See official blog:

> https://medium.com/nebulasio/how-to-build-a-dapp-on-nebulas-part-1-da4eaf9399bc
> https://medium.com/nebulasio/how-to-build-a-dapp-on-nebulas-part-2-5424789f7417

#### Q: How to use wallet to interact with Nebulas?
> A: For basic tutorials, see the official feature release introduction 
> https://github.com/nebulasio/web-wallet 

#### Q: How to query the content of the deployed contract?

> A: The  status can be found in web-wallet or https://explorer.nebulas.io/#/ via the transaction hash returned when deploying contract.

#### Q: Can the deployed contract be upgraded directly or support data migration to a new contract?

> A: No, you need to redeploy the smart contract.

#### Q: Does the contract need to consume gas, and what is the lower limit?

> A: Gas must be consumed when the contract is deployed or called. The consumption of gas in the development of the Nebulas is not expensive, and the gaslimit range is [20000, 50000000000]. Gasprice defaults to 10^wei. Note that 1NAS=1^18wei, the actual consumption is very low.

#### Q: How to check the transaction history of my account?
> A: Enter your account address at https://explorer.nebulas.io/#/

#### Q: Why does the transaction I send have been pending?
> A: The usual reason is that you have entered too big nonce when sending transactions. Nonce is executed sequentially. You can query the current state of the account (getAccontState) to ensure that the nonce you send is continuous.

#### Q: Why can't I find out the contract I deployed?
> A: There are usually two possibilities. The first is a deployment failure. You can query the completion status in web-wallet or https://explorer.nebulas.io/#/ by returning the transaction hash. The second situation is that you accidentally set up the wrong network and mistakenly testnet/mainnet

#### Q: Can I inquire about the content of the contract through the contract address?
> A: The query status of the current contract is queried by the transaction hash. The official has completed the function of querying through the contract address today and will be online soon. Please look forward to it.

#### Q: What if my contract has a bug and I want to fix it?
> A: Redeploy a contract. Point your DApp interface request to the new contract.

#### Q: How to apply nas tokens when testing on the test network?
> A: https://testnet.nebulas.io/claim/

#### Q: Where will console.log be printed in smart contract?
> A: Currently printed in the background log of the neb node. If there is a need to print logs for local test, please check the documentation to start the node locally.

#### Q: Considering that users may have concerns about uploading private keys, how can you easily develop DApps on the web?
> A: A Chrome-Extension wallet from community enthusiasts could be helpful (https://github.com/ChengOrangeJu/WebExtensionWallet). The wallet provides functions such as send transactions / import wallet / call contract method / query transaction status, which would reduce the workload of web developers significantly when developing DApps.


## Frequently used links:

Test Network: https://testnet.nebulas.io

Apply for NAS test currency: https://testnet.nebulas.io/claim

Main Website: https://mainnet.nebulas.io

Block Browser: https://explorer.nebulas.io

Help and teaching documentation:

Wiki: https://github.com/nebulasio/wiki

Tutorial: https://github.com/nebulasio/wiki/tree/master/tutorials

Tutorial website (English): http://nebulearn.com

Related source code:

Main chain Code: https://github.com/nebulasio/go-nebulas

Browser code: https://github.com/nebulasio/explorer

Web wallet code: https://github.com/nebulasio/web-wallet

API:

Web SDK: https://github.com/nebulasio/neb.js