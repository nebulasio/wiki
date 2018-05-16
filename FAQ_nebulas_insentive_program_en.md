## Questions about Intensive Program

##### Q: A brief introduction about Nebula incentive program?

>  From 00:00 on May 7th, 2018 to 00:00 on July 2nd, 2018, all developers who successfully submit DApp on the main net of the Nebulas and corresponding promoters can obtain NAS bounties. Weekly Excellent Application Award 10,000 NAS, Monthly 20,000 NAS! Promoter Monthly Award 10,000 NAS! The total award amount is 460,000 NAS. See details
     Https://blog.nebulas.io/2018/05/02/official-interpretation-of-nebulas-incentive-program-implementation-details-season-1/

##### Q. Why start the Nebulas Incentive Program?

> A: Nebulas is committed to speeding up the development of a sustainable and healthy next-generation public blockchain ecosystem. To accomplish this, the Nebulas Foundation is encouraging developers to continue to develop more and better decentralized apps (DApps) on the Nebulas mainnet blockchain. In the spirit of letting everyone benefit from the fairness in decentralized collaboration, the Nebulas Foundation has initiated the first season of the Nebulas Incentive Program, welcoming global developers and promoters who recognized the power of blockchain and share similar values with Nebulas.

##### Q: More FAQ about Nebulas Incentive Program.
> A: Find it here: https://medium.com/nebulasio/nebulas-incentive-program-q-a-e48159815ab7

## Questions about setting up environment


##### Q:  How to start my node in local environment?

>A: You can refer to the official tutorial: https://github.com/nebulasio/wiki/tree/master/tutorials. If you encounter some problem, try to search in the github issues of `go-nebulas` and `wiki` and probably you will get new finding.

##### Q:  Do I need to start a local node to participate in the official incentive program?

> A:  Not necessary. You can use web-wallet (https://github.com/nebulasio/web-wallet ) to deploy the contract on testnet and mainnet.

##### Q: I am using a  mac. Why isn’t my library installed correctly?

> A: There are also some small partners in the community who have encountered similar problems. Most of the reason is that you need to run `brew update` to update brew version, and then the installed dependencies are compatible.

##### Q: I am using windows operating system. Can I start the node locally?

> A: You can install the virtual machine to start the node locally. 

##### Q: I am using Windows operating system. Can I deploy smart contracts to participate in this incentive plan?
> A: Yes, web-wallet and other components support running in the windows environment.

##### Q: I built a node in the local according to the official tutorial to test, is there any way to modify the default settings for faster debugging?

> A: Of course there are ways to change the time interval and the number of dynasties for each block. For details see:
https://github.com/nebulasio/go-nebulas/issues/109

##### Q: After the installation is complete, start Neb locally and report an error "Make build libnebulasv8.so cannot open shared object file: No such file or directory". What's going wrong?

> A: Usually due to missing v8 files. Please check carefully if you missed make deploy-v8 during installation. Or try solving the problem at https://github.com/nebulasio/go-nebulas/issues/93.

##### Q: Git clone too slow

> A: If the content to be downloaded is large, network jitter will cause the download to fail. You can try to configure the proxy. For details, refer to https://github.com/nebulasio/go-nebulas/issues/88.

##### Q: When sending a request to a node, occasionally "503 error ("error": "all SubConns are in TransientFailure")?

> A: There are two situations. One is that the number of network connections on the NEB node is too large, and the upper limit is reached. This situation can be retried later. The other is the local encounter after the first compile and start, this time you can try to restart the machine to solve. Since the second situation is more difficult to reproduce, the suspect may be that some of the resources in the system have not been released and the government will continue to look for better solutions.

##### Q: When sending a request to a node that is set up by itself, chrome shows 501 not implemented. The server does not support OPTIONS requests. Would you like to ask whether this is not supported by nebulas?

> A: modify local config
     rpc {
     ...
     http_module: ["api","admin"]
     # HTTP CORS allowed origins
     +  http_cors: ["*"]
     }


### Frequently used links:

Test Network: https://testnet.nebulas.io

Apply for NAS test currency: https://testnet.nebulas.io/claim

Main Website: https://mainnet.nebulas.io

Block Browser: https://explorer.nebulas.io

Help and teaching documentation:

Wiki: https://github.com/nebulasio/wiki

Tutorial: https://github.com/nebulasio/wiki/tree/master/tutorials

Tutorial website (English): http://nebulearn.com

Project examples:

How to develop a DApp in one hour

Related source code:

Main chain Code: https://github.com/nebulasio/go-nebulas

Browser code: https://github.com/nebulasio/explorer

Web wallet code: https://github.com/nebulasio/web-wallet

API:

Web SDK: https://github.com/nebulasio/neb.js