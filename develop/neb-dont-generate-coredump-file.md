## OverView
In Testing, neb may be crash, but don't generate coredump file.  We can find the crash log  In /var/log/apport.log
```
"called for pid 10110, signal 11, core limit 0, dump mode 1 "
```
The coredump file is very very importance,  it  can serve as useful debugging aids in several situations , help us to debug quickly . We should make neb to generate coredump file .
 
We can use  `ulimit -a` command to show core file size , if size is zero , we should change the option, temporary use `ulimit -c unlimited` , permanently we can change `/etc/security/limits.conf`  , after reboot or `sysctl -p`

```
<domain>    <type>    <item>        <value>
 * soft    core            unlimited
```

But  these ways are't work , neb can't generate coredump file and `cat /proc/$pid/limits` always "Max core file size  0"

## Why? Why? Why?  It's not Work

1. setting is wrong ? so try a c++ programe build, run it can generate coredump 

2. Neb is started by supervisord,  is supervisord cause？
3. Try to start neb without supervisord， the neb coredump  
4. Yes supervisord , google supervisord+coredump to solve 

## Solution

Supervisord only set RLIMIT_NOFILE, RLIMIT_NOPROC by set_rlimits , others are seted default 0 
  1. modify supervisord code options.py in 1293 line 
```
vim /usr/lib/python2.6/site-packages/supervisor/options.py

soft, hard = resource.getrlimit(resource.RLIMIT_CORE)
resource.setrlimit(resource.RLIMIT_CORE, (-1, hard))
```
  2. restart supervisord and it works .

## Other 
change coredump file  name and path

```
echo "/neb/app/core-%e-%p-%t" > /proc/sys/kernel/core_pattern

%p: pid
%: '%' is dropped
%%: output one '%'
%u: uid
%g: gid
%s: signal number
%t: UNIX time of dump
%h: hostname
%e: executable filename
%: both are dropped
```
## References

* [supervisord coredump](https://www.jianshu.com/p/f5920842b27b)
* [core_pattern](https://sigquit.wordpress.com/tag/core_pattern/)
