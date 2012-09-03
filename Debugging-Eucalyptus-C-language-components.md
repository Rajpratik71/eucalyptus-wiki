# Debugging Eucalyptus C-language components (CC and NC)

CC and NC can be debugged using gdb, which can be:

* used to analyze a core dump,
* attached to a live Apache process hosting CC or NC, 
* used to start CC or NC under a debugger from the very beginning.

Each approach will be discussed in turn. 

> The commands below assume that `$EUCALYPTUS` is set to the root of Eucalyptus installation: typically just `/` for package-based installs and often `/opt/eucalyptus` for from-source installations.

## Core dump

Core dumps are useful when a SEGFAULT is difficult to trigger manually, especially on CC, which does a lot of forking.  You know your CC or NC is segfaulting when `httpd-[cc|nc]_error_log` contains lines similar to:

```bash
[Wed Aug 29 14:41:07 2012] [notice] child pid 22520 exit signal Segmentation fault (11)
[Wed Aug 29 14:41:13 2012] [notice] child pid 22555 exit signal Segmentation fault (11)
[Wed Aug 29 14:41:19 2012] [notice] child pid 22579 exit signal Segmentation fault (11)
```

To ensure that CC produces a core dump, you'll need to add the following line

```bash
echo "CoreDumpDirectory /tmp" >>$EUCALYPTUS/etc/eucalyptus/httpd-cc.conf
```

at the end of `create_httpd_config()` function in `$EUCALYPTUS/etc/init.d/eucalyptus-cc`. For NC do the same with 'nc' instead of 'cc' in the paths above. For the changes to take effect, stop the component, increase the core limit (in case it is too low), and start the component again. 

```bash
$EUCALYPTUS/etc/init.d/eucalyptus-cc stop
ulimit -c unlimited
$EUCALYPTUS/etc/init.d/eucalyptus-cc start
```
After that the error in the log should change to:

```bash
[Wed Aug 29 15:39:53 2012] [notice] child pid 6926 exit signal Segmentation fault (11), possible coredump in /tmp
```
And the `/tmp` directory should contain the core dump that can be brought up in `gdb`:

```bash
gdb /usr/sbin/httpd /tmp/core.9895
....
Core was generated by `/usr/sbin/httpd -f /opt/eucalyptus/etc/eucalyptus/httpd-nc.conf'.
Program terminated with signal 11, Segmentation fault.
#0  0x00007f73bf7a357c in vfprintf () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install httpd-2.2.15-15.el6.centos.1.x86_64
(gdb)
```

## Attach gdb to a Eucalyptus component

Attaching to a running instance of the component is often sufficient to examine its memory state or to catch a reproducible SEGFAULT with the debugger attached. 

The main difficulty has to do with deciding which process to attach to and how to ensure the debugger follows the forks you want. Component log files `cc.log` and `nc.log` may reveal to you the PID of the thread of control that you are looking for. 

NC is easier to debug as in steady state it only consists of two heavyweight processes: the core of Apache daemon (running as `root`) and the Apache deamon with the Eucalyptus shared library loaded (running as `eucalyptus`):

```bash
# ps aux | grep eucalyptus/httpd
root     22526  0.0  0.0  55168  1452 ?        Ss   16:00   0:00 /usr/sbin/httpd -f /opt/eucalyptus/etc/eucalyptus/httpd-nc.conf
500      22528  0.2  1.4 2105452 114548 ?      Sl   16:00   0:00 /usr/sbin/httpd -f /opt/eucalyptus/etc/eucalyptus/httpd-nc.conf
```

Attaching `gdb` to the latter will allow one to pause its execution, possibly set a breakpoint or inspect state of threads, and to either detach or let it run under the debugger (it is important not to pause the component for too long, since eventually network request timeouts on the upstream component may turn the system into an unusual state):

```bash
# gdb --pid=22528
....
(gdb) info thread
  3 Thread 0x7f601f224700 (LWP 22537)  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
  2 Thread 0x7f6018116700 (LWP 22541)  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
* 1 Thread 0x7f6093de77e0 (LWP 22528)  0x00007f6092423fff in accept4 () from /lib64/libc.so.6
(gdb) cont
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x00007f6092423fff in accept4 () from /lib64/libc.so.6
(gdb) detach
Detaching from program: /usr/sbin/httpd, process 22528
(gdb) quit
```

NC uses multiple threads, which can be examined interactively to identify them:

```bash
(gdb) info thread
  3 Thread 0x7f601f224700 (LWP 22537)  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
  2 Thread 0x7f6018116700 (LWP 22541)  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
* 1 Thread 0x7f6093de77e0 (LWP 22528)  0x00007f6092423fff in accept4 () from /lib64/libc.so.6
(gdb) thread 2
[Switching to thread 2 (Thread 0x7f6018116700 (LWP 22541))]#0  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
(gdb) bt
#0  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
#1  0x00007f60923e6fd0 in sleep () from /lib64/libc.so.6
#2  0x00007f608e0494c6 in monitoring_thread (arg=0x7f608e304520) at handlers.c:620
#3  0x00007f60926d37f1 in start_thread () from /lib64/libpthread.so.0
#4  0x00007f6092421ccd in clone () from /lib64/libc.so.6
(gdb) thread 3
[Switching to thread 3 (Thread 0x7f601f224700 (LWP 22537))]#0  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
(gdb) bt
#0  0x00007f60923e715d in nanosleep () from /lib64/libc.so.6
#1  0x00007f609241b124 in usleep () from /lib64/libc.so.6
#2  0x00007f608e07fe40 in sensor_bottom_half () at sensor.c:54
#3  0x00007f608e07fecb in sensor_thread (arg=0x0) at sensor.c:76
#4  0x00007f60926d37f1 in start_thread () from /lib64/libpthread.so.0
#5  0x00007f6092421ccd in clone () from /lib64/libc.so.6
(gdb)
```
One can discern from the above that **thread 2** is the `monitoring_thread` and **thread 3** is the `sensor_thread`. If there were instances in the process of being started up or rebooted or bundled, you would also see `startup_thread` or `rebooting_thread` or `bundling_thread` in the list.

## Run Eucalyptus component under gdb

Several environment variables must be set when starting a Eucalyptus component under `gdb` from the beginning:

```bash
export EUCALYPTUS=/opt/eucalyptus
export AXIS2C_HOME=/opt/eucalyptus/packages/axis2c-src-1.6.0/
export LD_LIBRARY_PATH=$AXIS2C_HOME/lib:$AXIS2C_HOME/modules/rampart
export PATH=$PATH:$EUCALYPTUS/usr/lib/eucalyptus
```
The first two are critical for any invocation, the last two may be needed, depending on the execution path of the component. Any running instance of the component must be shut down before invoking the component under the debugger. Depending on the distribution, the Apache binary may be called `httpd` or `apache2`:

```bash
# gdb /usr/sbin/httpd
...
Reading symbols from /usr/sbin/httpd...(no debugging symbols found)...done.
Missing separate debuginfos, use: debuginfo-install httpd-2.2.15-15.el6.centos.1.x86_64
(gdb) break monitoring_thread
Function "monitoring_thread" not defined.
Make breakpoint pending on future shared library load? (y or [n]) y
Breakpoint 1 (monitoring_thread) pending.
(gdb) run -X -f $EUCALYPTUS/etc/eucalyptus/httpd-nc.conf >/dev/null
Starting program: /usr/sbin/httpd -X -f $EUCALYPTUS/etc/eucalyptus/httpd-nc.conf >/dev/null
[Thread debugging using libthread_db enabled]
[New Thread 0x7fff833d4700 (LWP 382)]
Detaching after fork from child process 383.
Detaching after fork from child process 385.
[New Thread 0x7fff7c2c6700 (LWP 386)]
[Switching to Thread 0x7fff7c2c6700 (LWP 386)]

Breakpoint 1, monitoring_thread (arg=0x7ffff24b4520) at handlers.c:498
498         logprintfl (EUCADEBUG, "spawning monitoring thread\n");
(gdb) cont
Continuing.
```

Note how setting breakpoints before the Eucalyptus component shared library is loaded results in 'not defined' error. Take care to type in the breakpoint information accurately. For NC, the default policy of debugger staying with the parent process is sufficient. For CC, which uses forks extensively, you may be able to reach the desired process by setting `set follow-fork-mode child` option on the `gdb` prompt.