# AgentSmith-HIDS

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;--The name of this project was inspired by the movie - The Matrix

[![License](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://github.com/DianrongSecurity/AgentSmith-HIDS/blob/master/LICENSE) [![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

English | [简体中文](README-zh_CN.md)



# THIS REPO IS OLD 
# RELEASE VERSION: https://github.com/bytedance/Elkeid


### About AgentSmith-HIDS

Technically, AgentSmith-HIDS is not a Host-based Intrusion Detection System (HIDS) due to lack of rule engine and detection function. However, it can be used as a high performance 'Host Information Collect Agent' as part of your own HIDS solution.
The comprehensiveness of information which can be collected by this agent was one of the most important metrics during developing this project, hence it was built to function in the kernel stack and achieve huge advantage comparing to those function in user stack, such as:

* **Better performance**, Information needed are collected in kernel stack to avoid additional supplement actions such as traversal of '/proc'; and to enhance the performance of data transportation, data collected is transferred via shared ram instead of netlink.
* **Hard to be bypassed**, Information collection was powered by specifically designed kernel drive, makes it almost impossible to bypass the detection for malicious software like rootkit, which can deliberately hide themselves.
* **Easy to be integrated**，The AgentSmith-HIDS was built to integrate with other applications and can be used not only as security tool but also a good monitoring tool, or even a good detector of your assets. The agent is capable of collecting the users, files, processes and internet connections for you, so let's imagine when you integrate it with CMDB, you could get a comprehensive map consists of your network, host, container and business (even dependencies). What if you also have a Database audit tool at hand? The map can be extended to contain the relationship between your DB, DB User, tables, fields, applications, network, host and containers etc. Thinking of the possibility of integration with network intrusion detection system and/or threat intelligence etc., higher traceability could also be achieved. It just never gets old.
* **Kernel stack + User stack**，AgentSmith-HIDS also provide user stack module, to further extend the functionality when working with kernel stack module.



### Major abilities of AgentSmith-HIDS：

* Kernel stack module hooks **execve, connect, process inject, create file, DNS query, load LKM** behaviors via Kprobe，and is also capable of monitoring containers by being compatible with Linux namespace.
* User stack module utilize built in detection functions including queries of **User List**，**Listening ports list**，**System RPM list**，**Schedule jobs**
* **AntiRootkit**，From: [Tyton](https://github.com/nbulischeck/tyton) ,for now add **PROC_FILE_HOOK**，**SYSCALL_HOOK**，**LKM_HIDDEN**，**INTERRUPTS_HOOK** feature，but only wark on Kernel > 3.10.
* Cred Change monitoring (sudo/su/sshd except)
* User Login monitoring


### Usage scenarios/methods of AgentSmith-HIDS (to be added)

* [How to detect reverse shell by AgentSmith HIDS](doc/How-to-use-AgentSmith-HIDS-to-detect-reverse-shell/How-to-detect-reverse-shell-by-AgentSmith-HIDS.md)


### About the compatibility with Kernel version

* Kernel > 2.6.25
* AntiRootKit > 3.10


### About the compatibility with Containers

| Source | Nodename       |
| ------ | -------------- |
| Host   | hostname       |
| Docker | container name |
| k8s    | pod name       |




### Composition of AgentSmith-HIDS

* **Kernel stack module (LKM)**
    Hook key functions via Kprobe to capture information needed.
* **User stack module** 
    Collect data capatured by kernel stack module, perform necessary process and send it to Kafka; 
    Keep sending heartbeat packet to server so process integrity can be identitied; 
    Execute commands received from server.
* **Agent Server**(Optional)
    Send commands to user stack module and monitoring the status of user stack module.

### Execve Hook

Achieved by hooking **sys_execve()/sys_execveat()/compat_sys_execve()/compat_sys_execveat()**, example:

```json
{
    "uid":"0",
    "data_type":"59",
    "run_path":"/tmp",
    "exe":"/opt/ltp/testcases/bin/growfiles",
    "argv":"growfiles -W gf26 -D 0 -b -i 0 -L 60 -u -B 1000b -e 1 -r 128-32768:128 -R 512-64000 -T 4 -f gfsmallio-35861 -d /tmp/ltp-Ujxl8kKsKY ",
    "pid":"35861",
    "ppid":"35711",
    "pgid":"35861",
    "tgid":"35861",
    "comm":"growfiles",
    "nodename":"test",
    "stdin":"/dev/pts/1",
    "stdout":"/dev/pts/1",
    "sessionid":"3",
    "sip":"192.168.165.1",
    "sport":"61726",
    "dip":"192.168.165.128",
    "dport":"22",
    "sa_family":"1",
    "pid_tree":"1(systemd)->1384(sshd)->2175(sshd)->2177(bash)->2193(fish)->35552(runltp)->35711(ltp-pan)->35861(growfiles)",
    "tty_name":"pts1",
    "socket_process_pid":"2175",
    "socket_process_exe":"/usr/sbin/sshd",
    "SSH_CONNECTION":"192.168.165.1 61726 192.168.165.128 22",
    "LD_PRELOAD":"/root/ldpreload/test.so",
    "user":"root",
    "time":"1579575429143",
    "local_ip":"192.168.165.128",
    "hostname":"test",
    "exe_md5":"01272152d4901fd3c2efacab5c0e38e5",
    "socket_process_exe_md5":"686cd72b4339da33bfb6fe8fb94a301f"
}
```

### Bind Hook

Achieved by hooking **sys_bind()**, example:

```json
{
    "uid":"0",
    "data_type":"49",
    "sa_family":"2",
    "exe":"/usr/bin/python2.7",
    "pid":"109640",
    "ppid":"215496",
    "pgid":"109640",
    "tgid":"109640",
    "comm":"python",
    "nodename":"n225-117-018",
    "sip":"0.0.0.0",
    "sport":"8000",
    "res":"0",
    "sessionid":"30",
    "user":"root",
    "time":"1587540231936",
    "local_ip_str":"10.225.117.18",
    "hostname_str":"n225-117-018",
    "exe_md5":"4f458165a2129ba549f1b6605ee87e74"
}
```

### Connect Hook

Achieved by hooking **tcp_v4_connect()/tcp_v6_connect()/ip4_datagram_connect()/ip6_datagram_connect()**, example:

```json
{
    "uid":"0",
    "data_type":"42",
    "sa_family":"2",
    "connect_type":"4",
    "dport":"1025",
    "dip":"180.101.49.11",
    "exe":"/usr/bin/ping",
    "pid":"6294",
    "ppid":"1941",
    "pgid":"6294",
    "tgid":"6294",
    "comm":"ping",
    "nodename":"test",
    "sip":"192.168.165.153",
    "sport":"45524",
    "res":"0",
    "sessionid":"1",
    "user":"root",
    "time":"1575721921240",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"735ae70b4ceb8707acc40bc5a3d06e04"
}
```



### DNS Query Hook

Achieved by hooking **udp_recvmsg()/udpv6_recvmsg()**, example:

```json
{
    "uid":"0",
    "data_type":"601",
    "sa_family":"2",
    "dport":"53",
    "dip":"192.168.165.2",
    "exe":"/usr/bin/ping",
    "pid":"6294",
    "ppid":"1941",
    "pgid":"6294",
    "tgid":"6294",
    "comm":"ping",
    "nodename":"test",
    "sip":"192.168.165.153",
    "sport":"53178",
    "qr":"1",
    "opcode":"0",
    "rcode":"0",
    "query":"www.baidu.com",
    "sessionid":"1",
    "user":"root",
    "time":"1575721921240",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"39c45487a85e26ce5755a893f7e88293"
}
```



### Create File Hook

Achieved by hooking **security_inode_create()**, example:

```json
{
    "uid":"0",
    "data_type":"602",
    "exe":"/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre/bin/java",
    "file_path":"/tmp/kafka-logs/replication-offset-checkpoint.tmp",
    "pid":"3341",
    "ppid":"1",
    "pgid":"2657",
    "tgid":"2659",
    "comm":"kafka-scheduler",
    "nodename":"test",
    "sessionid":"3",
    "user":"root",
    "time":"1575721984257",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"215be70a38c3a2e14e09d637c85d5311",
    "create_file_md5":"d41d8cd98f00b204e9800998ecf8427e"
}
```



### Process Inject Hook

Achieved by hooking **sys_ptrace()**, example:

```json
{
    "uid":"0",
    "data_type":"101",
    "ptrace_request":"4",
    "target_pid":"7402",
    "addr":"00007ffe13011ee6",
    "data":"-a",
    "exe":"/root/ptrace/ptrace",
    "pid":"7401",
    "ppid":"1941",
    "pgid":"7401",
    "tgid":"7401",
    "comm":"ptrace",
    "nodename":"test",
    "sessionid":"1",
    "user":"root",
    "time":"1575722717065",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"863293f9fcf1af7afe5797a4b6b7aa0a"
}
```


### Load LKM File Hook

Achieved by hooking **load_module()**, example:

```json
{
    "uid":"0",
    "data_type":"603",
    "exe":"/usr/bin/kmod",
    "lkm_file":"/root/ptrace/ptrace",
    "pid":"29461",
    "ppid":"9766",
    "pgid":"29461",
    "tgid":"29461",
    "comm":"insmod",
    "nodename":"test",
    "sessionid":"13",
    "user":"root",
    "time":"1577212873791",
    "local_ip":"192.168.165.152",
    "hostname":"test",
    "exe_md5":"0010433ab9105d666b044779f36d6d1e",
    "load_file_md5":"863293f9fcf1af7afe5797a4b6b7aa0a"
}
```


### Cred Change Hook

Achieved by Hook **commit_creds()**，example：

```json
{
    "uid":"0",
    "data_type":"604",
    "exe":"/tmp/tt",
    "pid":"27737",
    "ppid":"26865",
    "pgid":"27737",
    "tgid":"27737",
    "comm":"tt",
    "old_uid":"1000",
    "nodename":"test",
    "sessionid":"42",
    "user":"root",
    "time":"1578396197131",
    "local_ip":"192.168.165.152",
    "hostname":"test",
    "exe_md5":"d99a695d2dc4b5099383f30964689c55"
}
```


### User Login Alert
```json
{
    "data_type":"1001",
    "status":"Failed",
    "type":"password",
    "user_exsit":"false",
    "user":"sad",
    "from_ip":"192.168.165.1",
    "port":"63089",
    "processor":"ssh2",
    "time":"1578405483119",
    "local_ip":"192.168.165.128",
    "hostname":"localhost.localdomain"
}
```


### PROC File Hook Alert
```json
{
    "uid":"-1",
    "data_type":"700",
    "module_name":"autoipv6",
    "hidden":"0",
    "time":"1578384987766",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### Syscall Hook Alert
```json
{
    "uid":"-1",
    "data_type":"701",
    "module_name":"diamorphine",
    "hidden":"1",
    "syscall_number":"78",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### LKM Hidden Alert
```json
{
    "uid":"-1",
    "data_type":"702",
    "module_name":"diamorphine",
    "hidden":"1",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### Interrupts Hook Alert
```json
{
    "uid":"-1",
    "data_type":"703",
    "module_name":"syshook",
    "hidden":"1",
    "interrupt_number":"2",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### About Performance of AgentSmith-HIDS

Testing Environment(VM):

| CPU       |  Intel(R) Xeon(R) Platinum 8260 CPU @ 2.40GHz    4Core |
| --------- | ------------------------------------------------ |
| RAM       | 8GB                                              |
| OS/Kernel | Debian9  /  4.14.81.bm.19-amd64          |

Testing Load：

`ltp -f syscalls`

Testing Result(1min):

| Hook Handler           | Average Delay(us) |  TP99(us) |   TP95(us) |   TP90(us) |
| ---------------------- | ----------------- | ----|----|----|
|   connect_entry_handler| 0.2914          |6.7627|0.355|0.3012|
|   connect_handler      |   2.1406        |18.3801|12.102|7.832|
|   execve_entry_handler |   5.9320        |13.7034|9.908|8.334|
|   execve_handler       |   6.8826        |26.0584|15.9976|12.6260|
|   security_inode_create_entry_handler|   1.9963|9.3042|6.7730|4.6816|
|   security_inode_create_handler|   4.2114|13.2165|8.83775|6.534|

Original Testing Data:

[Benchmark Data](https://github.com/EBWi11/AgentSmith-HIDS/tree/master/benchmark_data)


**cyclictest testing**

`cyclictest -p 90 - m -c 0 -i 200 -n -h 100 -q -l 1000000`

Uninstall Smith：
```
# Total: 000999485
# Min Latencies: 00002
# Avg Latencies: 00007
# Max Latencies: 13905
# Histogram Overflows: 00515
```

install Smith：
```
# Total: 000999519
# Min Latencies: 00002
# Avg Latencies: 00007
# Max Latencies: 15216
# Histogram Overflows: 00481
```


**time -v /opt/ltp/testcases/bin/execve05 -n 30000**

10 times

Install Smith：

| Average User Time(s) |  Average System Time(s) |
| ---------------------- | ----------------- |
|22.329|14.885|

Uninstall Smith：

| Average User Time(s) |  Average System Time(s) |
| ---------------------- | ----------------- |
|22.271|14.395|

### Documents for deployment and testing purpose:

[Quick Start](https://github.com/EBWi11/AgentSmith-HIDS/blob/master/doc/AgentSmith-HIDS-Quick-Start.md)




### Special Thanks(Not in order)

[yuzunzhi](https://github.com/yuzunzhi)

[hapood](https://github.com/hapood)

[HF-Daniel](https://github.com/HF-Daniel)

[smcdef](https://github.com/smcdef)


### Wechat of developer

<img src="doc/wechat.jpg" width="50%" height="50%"/>


### Wechat channel of '灾难控制局'

We would constantly provide information about the functionalities of AgentSmith-HIDS via this channel, a good place to receive the most updated news:)

<img src="doc/SecDamageControl.jpg" width="50%" height="50%"/>


## License

AgentSmith-HIDS kernel module are distributed under the GNU GPLv2 license.
