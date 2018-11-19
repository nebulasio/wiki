# NBRE Design Doc

NBRE (Nebulas Runtiome Environment) is the Nebulas chain execution environment.
Its framework is shown as follows.

![](https://github.com/nebulasio/wiki/blob/50d4c53753b24b37611bacb94b5eedd49d03afdd/resources/NBRE-Overview.png)

NBRE contains two main processes, which provide the methods how to update algorithms and how to execute algorithms.

The updating process provides how to upload algorithms and core protocols.
It includes the following steps:

A. The algorithms are implemented with the languages supported by LLVM. Then, their codes are handled by the NASIR tool, which are translated to bitcode.

B. The bitcode streams are coded with base64, which are translated to payload of transaction data. The transaction data is uploaded to the online chain.

C. After that, the transaction data will be packed and varified. Then, the related bitcode will stored into the RocksDB.

The execution process exhibits the processes from request to results.
The corresponding details are as follows.

1. User appries for algorithm call requests with the forms of RPC or RESful API.

2. After receiving the request, the core NEB forward it to NBRE.

3. NBRE starts JIT and loads the algorithm code into JIT.

4. The JIT executes the algorithm with specified parameters and the invoking method, and returns the execution result.

5. NBRE returns the execution result to NEB through IPC.

6. NEB returns the result to the user.
