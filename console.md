# REPL console

Nebulas provide an interactive javascript console, which can invoke all API and management RPC methods. Some management methods may require passphrase. Start console using the command:

```bash
./neb console
```

We have API and admin two schemes to access the console cmds. Users can quickly enter instructions using the TAB key.

```javascript
> api.
api.call                    api.getBlockByHash          api.getNebState             api.subscribe
api.estimateGas             api.getBlockByHeight        api.getTransactionReceipt
api.gasPrice                api.getDynasty              api.latestIrreversibleBlock
api.getAccountState         api.getEventsByHash         api.sendRawTransaction

> admin.
admin.accounts                      admin.nodeInfo                      admin.signHash
admin.getConfig                     admin.sendTransaction               admin.signTransactionWithPassphrase
admin.lockAccount                   admin.sendTransactionWithPassphrase admin.startPprof
admin.newAccount                    admin.setHost                       admin.unlockAccount

```

```javascript
> admin.unlockAccount("n1FF***dptE", "passphrase")
Unlock account n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE
Passphrase:
{
    "result": {
        "result": true
    }
}
```

The command parameters of the command line are consistent with the parameters of the RPC interface. [NEB RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) and [NEB RPC_Admin](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md).