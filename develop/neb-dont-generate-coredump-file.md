## OverView
During Testing, neb may be crash, and we want to get the coredump file which could help us to find the reason. However, neb don't generate coredump file by default.  We can find the crash log in /var/log/apport.log when a crash occurred:

```
"called for pid 10110, signal 11, core limit 0, dump mode 1 "
```
The coredump file is very very important, it can serve as useful debugging aids in several situations, and help us to debug quickly. Therefore we should make neb to generate coredump file.

## Set the core file size
 
We can use  `ulimit -a` command to show core file size. If it's size is zero, which means coredump file is disabled, then we should set a value for core file size. for temporarily change we can use `ulimit -c unlimited` , and for permanently change we can edit `/etc/security/limits.conf` file, it will take effect after reboot or command `sysctl -p`.

```
<domain>    <type>    <item>        <value>
 * soft    core            unlimited
```

But these ways are't work, neb still can't generate coredump file and `cat /proc/$pid/limits` always "Max core file size  0"

## Why? Why? Why?  It doesn't Work

1. If the setting is wrong? Just try a c++ programe build, run it and we can find that it can generate coredump.

2. Neb is started by supervisord,  is it caused by supervisord？
3. Try to start neb without supervisord, then the neb coredump is generated!
4. Yes, the reason is supervisord, then we can google "supervisord+coredump" to solve it.

## Solution

Supervisord only set RLIMIT_NOFILE, RLIMIT_NOPROC by set_rlimits , others are seted default 0 
  1. modify supervisord code options.py in 1293 line 
```
vim /usr/lib/python2.6/site-packages/supervisor/options.py

soft, hard = resource.getrlimit(resource.RLIMIT_CORE)
resource.setrlimit(resource.RLIMIT_CORE, (-1, hard))
```
  2. restart supervisord and it works .

## Other seetings
You can also change the name and path of coredump file by changing file `/proc/sys/kernel/core_pattern`:

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
