## OverView
Last week we found a lot of “Failed to update latest irreversible block.”  in neb log with Leon. The  reference code  (nebulasio/go-nebulas/core/blockchain.go updateLatestIrreversibleBlock ) ， in the code we found the cur variable is not equal to the tail variable , why?  to  find the cause, we try to use tool to dynamically display variable information and facilitate single-step debugging. 

## Goroutines 
In c++ program we often use gbd to debug, so we think why not to  use gdb to debug golang program . First we try to look up the BlockChain loop goroutine state  and print the variables .

In c++ we all use `info  threads` and thread x to show thread info but in the golang program ，we should use  `info goroutines` and `goroutine xx bt` to displays the current list of running goroutines.

(gdb) `info goroutines`
Undefined info command: "goroutines".  Try "help info". 
(gdb) `source` /usr/local/go/src/runtime/runtime-gdb.py 
Loading Go Runtime support.
(gdb) `info goroutines`

```
1 waiting  runtime.gopark
2 waiting  runtime.gopark
3 waiting  runtime.gopark
4 waiting  runtime.gopark
5 syscall  runtime.notetsleepg
6 syscall  runtime.notetsleepg
7 waiting  runtime.gopark
... ... 
```

(gdb) goroutine 84 bt

```
 #0 runtime.gopark (unlockf={void (struct runtime.g , void , bool *)} 0xc420c57c80, lock=0x0, reason="select", traceEv=24 '\030', traceskip=1) at /data/packages/go/src/runtime/proc.go:288
 #1 0x0000000000440fd9 in runtime.selectgo (sel=0xc420c57f48, ~r1=842353656960) at /data/packages/go/src/runtime/select.go:395
 #2 0x0000000000ad2d73 in github.com/nebulasio/go-nebulas/core.(*BlockChain).loop (bc=0xc4202c6320)at /neb/golang/src/github.com/nebulasio/go-nebulas/core/blockchain.go:184
 #3 0x0000000000460421 in runtime.goexit () at /data/packages/go/src/runtime/asm_amd64.s:2337
 #4 .....
```
But neb has too many goroutines, we don’t kown which one ,  we give up

## BreakPoints
Second  we try to set break point to debug 
(gdb) `b blockchain.go:381`
Breakpoint 2 at 0xad4373: file /neb/golang/src/github.com/nebulasio/go-nebulas/core/blockchain.go, line 381.
(gdb) `b core/blockchain.go:390`
Breakpoint 3 at 0xad44c6: file /neb/golang/src/github.com/nebulasio/go-nebulas/core/blockchain.go, line 390.
(gdb) `info breakpoints` // show all breakpoints 
(gdb) `d 2` //delete No 2 breakpoint
Now let the neb continue its execution until the next breakpoint, enter the c command:
(gdb) `c`
Continuing
```
Thread 6 "neb" hit Breakpoint 2, github.com/nebulasio/go-nebulas/core.(*BlockChain).updateLatestIrreversibleBlock (bc=0xc4202c6320, tail=0xc4244198c0)
at /neb/golang/src/github.com/nebulasio/go-nebulas/core/blockchain.go:382
382        miners := make(map[string
```
now we can use p(print) to print variables value 
(gdb) `p cur`
$2 = (struct github.com/nebulasio/go-nebulas/core.Block *) 0xc420716f90
(gdb) `p cur.height`
$3 = 0
(gdb) `p bc`
$4 = (struct github.com/nebulasio/go-nebulas/core.BlockChain *) 0xc4202c6320
(gdb) `p bc.latestIrreversibleBlock`
$5 = (struct github.com/nebulasio/go-nebulas/core.Block *) 0xc4240bbb00
(gdb) `p bc.latestIrreversibleBlock.height`
$6 = 51743
(gdb) `p tail`
$7 = (struct github.com/nebulasio/go-nebulas/core.Block *) 0xc4244198c0
(gdb) `p tail.height`
$8 = 51749
now we can use `info goroutines` again, to find current goroutine. info goroutines with the * indicating the current execution, so we find the current goroutine nunmber quickly.

the next breakpoint we can use `c` command , so we found the cur and lib is not equal, because of  length of the miners is less than ConsensusSize， In the loop the cur change to the parent block . 

## Other
When compiling Go programs, the following points require particular attention:
* Using -ldflags "-s" will prevent the standard debugging information from being printed
* Using -gcflags "-N-l" will prevent Go from performing some of its automated optimizations -optimizations of aggregate variables, functions, etc. These optimizations can make it very difficult for GDB to do its job, so it's best to disable them at compile time using these flags.

## References
* [Debugging with GDB](https://astaxie.gitbooks.io/build-web-application-with-golang/en/11.2.html)
* [GDB调试GO程序](http://blog.studygolang.com/2012/12/gdb%E8%B0%83%E8%AF%95go%E7%A8%8B%E5%BA%8F/)





