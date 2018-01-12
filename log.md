# Logs

## Introduction

Nebulas provides two kinds of logs: console log & verbose log.

### Console Log

Console Log(CLog) is used to help you understand which job **Neb** is working on now, including start/stop components, receive new blocks on chain, do synchronization and so on.

- CLog will print all logs to stdout & log files both. You can check them in your standard output directly.

Nebulas console log statements

```
// log level can be `Info`,`Warning`,`Error`
logging.CLog().Info("")
```

##### Startup specifications
Nebulas start service should give a console log, the logs should before the service start. The log format just like this:

```
logging.CLog().Info("Starting xxx...")
```


##### Stopping specifications

Nebulas stop service should give a console log, the logs should before the service stoped. The log format just like this:

```
logging.CLog().Info("Stopping xxx...")
```

### Verbose Log

Verbose Log(VLog) is used to help you understant how **Neb** works on current job, including how to verifiy new blocks, how to discover new nodes, how to mint and so on.

- VLog will print logs to log files only. You can check them in your log folders if needed.

What'r more, you can set your concerned level to VLog to filter informations. The level filter follows the priority as **Debug < Info < Warn < Error < Fatal.**

## Hookers

By default, Function hookers & FileRotate hookers are added to CLog & VLog both.

### FunctionNameHooker

FunctionHooker will append current caller's function name & code line to the loggers. The result looks like this,

> time="2018-01-03T20:20:52+08:00" level=info msg="node init success" file=net_service.go **func=p2p.NewNetManager** **line=137** node.listen="[0.0.0.0:10001]"

### FileRotateHooker

FileRotateHooker will split logs into many smaller segments by time. By default, all logs will be rotated every 1 hour. The log folder looks like this,

> neb-2018010415.log 
> neb-2018010416.log
> neb.log -> /path/to/neb-2018010415.log

---

If you have any suggestions about logs, please feel free to submit issues on our [wiki](https://github.com/nebulasio/wiki) repo. Thanks!


