## Perf Guide
perf only exist on linux

it can
* trace system call
* profile program in any langauge
* trace almost any kernel event

#### perf top
top shows cpu usage of programs, perf top shows cpu usage of functions. when a program is running at 100% CPU, use perf top to find which function is using up the most cpu

perf top will give both kernel functions and user space functions. busy kernel should show multiple kernel functions are using CPU. Need to figure out if those kernel functions are in a call chain

#### perf record
perf record allow saving of the data for analysis. it saves to a file `perf.data` in the current directory.

4 main ways of using perf record:
1. perf record [Command] #start and profile the command until it exits
2. perf record [PID] #start and profile the PID until crtl-c
3. perf record -a #profile every process until crtl-c
4. perf record -p [PID] [Command] #if both PID and command is specified, it will profile the PID until command exits, e.g. perf record -p 8999 sleep -5, so you can profile a PID for K seconds

perf collect profiling data of function call by sampling. perf can also record different kind of events. when it record events, it doesn't sample. if it records system call, it will record every system call. example of events:
* system call
* sending network packets
* reading from block device
* context switch/page fault
* can make any kernel function into a event (kprob)

perf record will record the stack trace up to the system call when recording a event. this means you can trace the program that is calling the system function. e.g.

```
perf record -e syscall:sys_enter_connect -ag # g means collect stack trace
```

#### Analyzing perf record using perf report
perf report creates interactive report on the record

#### perf annotate for perf record
perf annotate will show which assembly instruction is being executed the most during the execution.

be careful, this can be off by one instruction

#### perf script for perf record
perf script prints out all the samples perf has collected as text for external analysis tool. such as flamegraph.

```
perf script | stackcollapse-perf.pl | flamegraph > file.svg
```

#### perf list
list every event

### Perf with Java/Node.js
normally when running perf on interpreted languages like node.js or java, perf tell you which interpreter function is running not which java or javascript function is running. but JVM and Node can communicate with perf to provide information regarding the function. 

in Node.js:
```
node --perf-basic-prof program.js
```

in java, use perf-map-agent
1. find PID of process
2. create-java-perf-map.sh $PID

##### Kernel Function in Stack Trace
sometimes kernel function can be seen in stack trace from perf. this is either because the function made a system call, or it created a page fault, requiring a system call.

#### Perf options
`-F` sets sampling frequency
`-g` record stack traces
`-e` choose events to record
`-a` record entire system
`-p` specify pid

### Perf stat
show CPU counter statistics for command.

to show detailed CPU counter statistics:
```
perf stat -ddd [command]
```

to show various basic CPU statistics
```
perf stat -e cycles,instructions,cache-misses -a
```

to count system calls for PID, until crtl-c
```
perf stat -e 'syscalls:sys_enter_*' -p [PID]
```

to count block device IO event for the entire system for 10 seconds
```
perf stat -e 'block: ' -a sleep 10
```

for high performance CPU/hardware level events, you'll be interested in L1 cache hit/miss, branch prediction miss, instructions per cycle, CPU cycles, page faults, TLB misses. all these can be viewed using stat with `-ddd`, which will show hardware counters. e.g.

```
sudo perf stat -ddd ls -R /
```

to count events. e.g.

```
sudo perf stat -e context-switches ls -R /
sudo perf stat -e 'syscalls:sys_enter_*' ls -R /
```

it could add additional overhead if every system event is counted

#### perf trace
trace system calls system wide
```
perf trace
```

trace system call for 1 PID
```
perf trace -p [PID]
```

perf trace is similar to strace but has way less overhead. however, sometimes it drops system calls, and it won't show you the strings that's been written/read

### Adding new trace events
adding a trace point for kernel function e.g. tcp_sendmsg:
```
perf probe 'tcp_sendmsg'
```

then trace the previously created probe:
```
perf record -e -a probe:tcp_sendmsg
```

add a trace point for user function myfunc return, and return retval as a string:
```
perf probe 'myfunc%return +0($retval):string'
```

trace previous probe when size > 0 and state is not TCP_ESTABLISHED(1): (require kernel debug info)
```
perf record -e -a probe:tcp_sendmsg --filter 'size > 0 && skc_state != 1' -a 
```

add a tracepoint for do_sys_open() with filename as a string: (require kernel debug info)
```
perf probe 'do_sys_open filename:string'
```

### Inner Working
the perf system is split into 2 parts:
* program in userspace called perf
* a system in linux kernel

the user space program calls the kernel perf system to obtain the stats.

this also means the perf running in the user space has to match in version to the kernel perf system.

perf gives the symbols from the program from the program's symbol table. so perf can't give this if symbol table for the program is stripped. 

how perf works basically:
1. perf calls perf_event_open system call
2. kernel write events to a ring buffer in user space
3. perf read events off the ring buffer and display them

when new events are written faster than perf can read, events will be dropped from perf.
