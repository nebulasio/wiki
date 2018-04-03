# REPL console

Nebulas provide an interactive javascript console, which can invoke all API and management RPC methods. The console is connected to the local node by default without specifying host.

## start console
Start console using the command:

```bash
./neb console
```
In the case of not specifying the configuration file, the terminal's startup defaults to the configuration of `conf/default/config.conf`. If the local configuration file is not available or you want to specify the configuration file, the terminal starts like this:

```bash
./neb -c <config file> console
```

#### console interaction

The console can use the `admin.setHost` interface to specify the nodes that are connected. When the console is started or the host is not specified, the terminal is interacting with the local node. **Therefore, you need to start a local node before starting the console.**

```javascript
> admin.setHost("https://testnet.nebulas.io")
```
*Tips: The Testnet only starts the RPC interface of the API, so only the api scheme is available.*

## console usage
We have API and admin two schemes to access the console cmds. Users can quickly enter instructions using the `TAB` key.

```javascript
> api.
api.call                    api.getBlockByHash          api.getNebState             api.subscribe
api.estimateGas             api.getBlockByHeight        api.getTransactionReceipt
api.gasPrice                api.getDynasty              api.latestIrreversibleBlock
api.getAccountState         api.getEventsByHash         api.sendRawTransaction

```

```javascript
> admin.
admin.accounts                      admin.nodeInfo                      admin.signHash
admin.getConfig                     admin.sendTransaction               admin.signTransactionWithPassphrase
admin.lockAccount                   admin.sendTransactionWithPassphrase admin.startPprof
admin.newAccount                    admin.setHost                       admin.unlockAccount
```
Some management methods may require passphrase. The user can pass in the password when the interface is called, or the console prompts the user for input when the password is not entered. **We recommend using a console prompt to enter your password because it is not visible.**

Enter the password directly:

```javascript
> admin.unlockAccount("n1UWZa8yuvRgePRPgp8a2jX4J9UwGXfHp6i", "passphrase")
{
    "result": {
        "result": true
    }
}
```
Use terminal prompt:

```javascript
> admin.unlockAccount("n1UWZa8yuvRgePRPgp8a2jX4J9UwGXfHp6i")
Unlock account n1UWZa8yuvRgePRPgp8a2jX4J9UwGXfHp6i
Passphrase:
{
    "result": {
        "result": true
    }
}
```
The interfaces with passphrase prompt:

```javascript
admin.newAccount
admin.unlockAccount
admin.signHash
admin.signTransactionWithPassphrase
admin.sendTransactionWithPassphrase
```
The command parameters of the command line are consistent with the parameters of the RPC interface. [NEB RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) and [NEB RPC_Admin](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md).

## console exit
The console can exit with the `ctrl-C` or `exit` command.