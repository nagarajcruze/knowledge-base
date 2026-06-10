# Linux Performance Troubleshooting Tools Guide (2026)

* * *

# 1\. System Overview & CPU Pressure

These commands help identify:

- system load
- CPU saturation
- scheduler pressure
- process behavior
- memory pressure

* * *

# uptime

## What it does

Shows:

- current time
- uptime
- logged-in users
- system load average

## Common Usage

```bash
uptime
```

## Example Output

```bash
10:30:01 up 5 days, 3:22, 2 users, load average: 0.52, 1.10, 1.30
```

## Fields

| Field | Meaning |
| --- | --- |
| current time | Current system time |
| up 5 days | System uptime |
| 2 users | Logged in users |
| 0.52 | 1-minute load average |
| 1.10 | 5-minute load average |
| 1.30 | 15-minute load average |

## Important Understanding

Load average is NOT CPU percentage.

It represents:

- runnable tasks
- tasks waiting for CPU
- tasks blocked in uninterruptible IO

## Real Usage

### 4-core system

```bash
load average: 8
```

Means:

- system overloaded
- more runnable tasks than available CPU cores

* * *

# vmstat

## What it does

Shows:

- CPU usage
- scheduler pressure
- memory
- swap
- IO
- interrupts

One of the most important Linux troubleshooting commands.

## Common Usage

```bash
vmstat 1
```

Refresh every second.

## Example Output

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 500000  20000 300000    0    0    10    20  100  200 20  5 70  5  0
```

## Important Columns

### Process Columns

| Column | Meaning |
| --- | --- |
| r   | Runnable tasks waiting for CPU |
| b   | Blocked tasks waiting on IO |

### Memory Columns

| Column | Meaning |
| --- | --- |
| swpd | Swap used |
| free | Free RAM |
| buff | Buffer memory |
| cache | Filesystem cache |

### Swap Columns

| Column | Meaning |
| --- | --- |
| si  | Swap in |
| so  | Swap out |

### IO Columns

| Column | Meaning |
| --- | --- |
| bi  | Blocks read |
| bo  | Blocks written |

### CPU Columns

| Column | Meaning |
| --- | --- |
| us  | User CPU |
| sy  | Kernel CPU |
| id  | Idle CPU |
| wa  | IO wait |
| st  | Stolen CPU in VM |

## Real Usage

### High CPU pressure

```bash
r = 20
```

On a 4-core system:

- heavy CPU contention

### High IO wait

```bash
wa = 40
```

Usually indicates:

- slow disk
- overloaded storage

### Active swapping

```bash
si/so continuously increasing
```

Means:

- memory pressure
- system slowdown

* * *

# mpstat

## What it does

Shows per-CPU/core statistics.

Useful for:

- single-thread bottlenecks
- interrupt imbalance
- uneven CPU utilization

## Common Usage

```bash
mpstat -P ALL 1
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-P ALL | Show all CPU cores |
| 1   | Refresh every second |

## Example Output

```bash
CPU   %usr %sys %idle %iowait
all    20    5    70      5
0      90    5     5      0
1       5    2    93      0
```

## Columns

| Column | Meaning |
| --- | --- |
| CPU | Core number |
| %usr | User CPU usage |
| %sys | Kernel CPU usage |
| %idle | Idle percentage |
| %iowait | Waiting for IO |

## Real Usage

### Single-thread bottleneck

```bash
CPU0 = 100%
Others mostly idle
```

Usually means:

- single-threaded application
- Python GIL limitation
- CPU affinity issue

* * *

# pidstat

## What it does

Shows process-level and thread-level resource usage.

Useful for:

- identifying heavy processes
- identifying hot threads
- tracking CPU/memory/IO per process

## Common Usage

### Process statistics

```bash
pidstat 1
```

### Thread statistics

```bash
pidstat -t 1
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-t | Show threads |
| 1   | Refresh every second |

## Example Output

```bash
PID   %usr %system %CPU CPU Command
2200   40      10   50   1 python
3300    5       2    7   0 nginx
```

## Columns

| Column | Meaning |
| --- | --- |
| PID | Process ID |
| %usr | User CPU |
| %system | Kernel CPU |
| %CPU | Total CPU |
| CPU | CPU core |
| Command | Process name |

## Thread Example

```bash
TGID  TID  %CPU Command
2200  2201  80  java
2200  2202   5  java
```

| Column | Meaning |
| --- | --- |
| TGID | Process ID |
| TID | Thread ID |
| %CPU | Thread CPU usage |

## Real Usage

### One hot thread

```bash
One thread = 100%
Other threads mostly idle
```

Could indicate:

- lock contention
- single-thread bottleneck
- GC issue

* * *

# top

## What it does

Real-time system monitor.

Shows:

- CPU
- memory
- load
- processes

## Common Usage

```bash
top
```

## Important Interactive Keys

| Key | Meaning |
| --- | --- |
| P   | Sort by CPU |
| M   | Sort by memory |
| H   | Show threads |
| q   | Quit |

## Example Output

```bash
PID USER  %CPU %MEM COMMAND
2200 app   80   10  java
3300 root  20    2  nginx
```

## Important Header Example

```bash
%Cpu(s): 20 us, 10 sy, 60 id, 10 wa
```

| Field | Meaning |
| --- | --- |
| us  | User CPU |
| sy  | Kernel CPU |
| id  | Idle CPU |
| wa  | IO wait |

## Real Usage

### High IO wait

```bash
wa = 30+
```

Usually storage bottleneck.

### One process using 400%

Means:

- process using 4 CPU cores

* * *

# 2\. Memory & Swap Tools

* * *

# free

## What it does

Shows memory and swap usage.

## Common Usage

```bash
free -m
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-m | Show values in MB |
| \-g | Show values in GB |
| \-h | Human readable |

## Example Output

```bash
              total   used   free  shared  buff/cache available
Mem:           8000   3000   1000     200       4000      4500
Swap:          2000      0   2000
```

## Important Columns

| Column | Meaning |
| --- | --- |
| total | Total RAM |
| used | Used memory |
| free | Completely unused RAM |
| buff/cache | Filesystem cache |
| available | Estimated reusable memory |

## Important Understanding

Linux intentionally uses RAM as filesystem cache.

Low "free" memory alone is NOT a problem.

Focus on:

```bash
available
```

## Real Usage

### Healthy system

```bash
free = low
available = high
```

System healthy.

### Memory pressure

```bash
available low
swap growing
vmstat si/so active
```

System under memory pressure.

* * *

# swapon

## What it does

Shows active swap devices/files.

## Common Usage

```bash
swapon --show
```

## Example Output

```bash
NAME      TYPE SIZE USED PRIO
/swapfile file 2G   0B   -2
```

## Columns

| Column | Meaning |
| --- | --- |
| NAME | Swap device/file |
| TYPE | File or partition |
| SIZE | Total swap size |
| USED | Swap currently used |
| PRIO | Swap priority |

## Real Usage

Heavy active swap usage together with:

```bash
vmstat si/so
```

Usually indicates:

- memory pressure
- application slowdown

* * *

# 3\. Disk & Storage Tools

* * *

# iostat

## What it does

Shows disk IO performance.

Useful for:

- storage bottlenecks
- latency analysis
- throughput analysis

## Common Usage

```bash
iostat -xz 1
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-x | Extended statistics |
| \-z | Hide inactive devices |
| 1   | Refresh every second |

## Example Output

```bash
Device:  r/s   w/s rkB/s wkB/s await svctm %util
sda      20    10 1024   512   5.2    1.0   30
nvme0n1 200   100 8000  4000  25.0    0.5   95
```

## Important Columns

| Column | Meaning |
| --- | --- |
| r/s | Reads per second |
| w/s | Writes per second |
| rkB/s | Read throughput |
| wkB/s | Write throughput |
| await | Average request wait time |
| svctm | Device service time |
| %util | Disk utilization |

## Real Usage

### Disk saturation

```bash
%util = 100
```

Disk fully busy.

### High latency

```bash
await = 50ms+
```

Usually:

- slow disk
- overloaded storage backend

* * *

# 4\. Networking Tools

* * *

# sar

## What it does

Collects and displays historical system activity.

Network mode useful for:

- bandwidth
- packet rates
- TCP retransmissions

## Common Usage

### Interface statistics

```bash
sar -n DEV 1
```

### TCP statistics

```bash
sar -n TCP,ETCP 1
```

### Combined statistics

```bash
sar -n TCP,ETCP,DEV 1
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-n DEV | Network device statistics |
| \-n TCP | TCP statistics |
| \-n ETCP | Extended TCP statistics |
| 1   | Refresh every second |

## DEV Example Output

```bash
IFACE rxpck/s txpck/s rxkB/s txkB/s
eth0     100      80    500    300
```

## DEV Columns

| Column | Meaning |
| --- | --- |
| IFACE | Network interface |
| rxpck/s | Packets received/sec |
| txpck/s | Packets sent/sec |
| rxkB/s | Receive throughput |
| txkB/s | Transmit throughput |

## TCP Example Output

```bash
active/s passive/s retrans/s
10        5          2
```

## TCP Columns

| Column | Meaning |
| --- | --- |
| active/s | Outgoing TCP connects/sec |
| passive/s | Incoming connects/sec |
| retrans/s | TCP retransmissions/sec |

## Real Usage

### High retransmissions

```bash
retrans/s very high
```

Possible causes:

- packet loss
- congestion
- unstable network

* * *

# netstat

## What it does

Shows:

- connections
- listening ports
- socket states

Modern replacement:

```bash
ss
```

## Common Usage

```bash
netstat -tulnp
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-t | TCP |
| \-u | UDP |
| \-l | Listening sockets |
| \-n | Numeric output |
| \-p | Show process |

## Example Output

```bash
Proto Local Address   Foreign Address State   PID/Program
tcp   0.0.0.0:22      0.0.0.0:*       LISTEN  1200/sshd
```

## Important TCP States

| State | Meaning |
| --- | --- |
| LISTEN | Waiting for connections |
| ESTABLISHED | Active connection |
| TIME_WAIT | Recently closed |
| CLOSE_WAIT | Remote side closed |

## Real Usage

### Too many CLOSE_WAIT sockets

Usually indicates:

- application socket leak
- improper connection cleanup

* * *

# tcpdump

## What it does

Captures network packets.

Useful for:

- DNS debugging
- API troubleshooting
- TLS issues
- packet loss analysis

## Common Usage

### Capture packets

```bash
tcpdump -i eth0
```

### Capture specific port

```bash
tcpdump -i eth0 port 443
```

### Save capture

```bash
tcpdump -w capture.pcap
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-i | Interface |
| port | Filter by port |
| \-w | Write capture file |

## Example Output

```bash
10:00:01 IP 192.168.1.10.443 > 192.168.1.20.50000: TCP
```

## Important Fields

| Field | Meaning |
| --- | --- |
| source IP.port | Packet sender |
| destination IP.port | Packet receiver |
| TCP/UDP | Protocol |

## Real Usage

### Repeated SYN packets

Usually indicates:

- firewall issue
- unreachable server
- dropped packets

* * *

# nicstat

## What it does

Shows NIC/interface utilization.

Useful for:

- NIC saturation
- bandwidth analysis
- packet queue buildup

## Common Usage

```bash
nicstat 1
```

## Example Output

```bash
Int   rKB/s  wKB/s %Util Sat
eth0   1000   500   20    0
```

## Columns

| Column | Meaning |
| --- | --- |
| Int | Interface |
| rKB/s | Receive throughput |
| wKB/s | Transmit throughput |
| %Util | NIC utilization |
| Sat | NIC saturation |

## Real Usage

### High NIC utilization

May indicate:

- interface saturation
- backup traffic
- replication traffic

* * *

# lsof

## What it does

Lists open files and sockets.

Useful for:

- identifying process using port
- identifying active TCP connections
- finding deleted open files

## Common Usage

### Find process using port

```bash
lsof -i :8080
```

### Show established TCP connections

```bash
lsof -iTCP -sTCP:ESTABLISHED
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-i | Network files/sockets |
| \-iTCP | TCP sockets |
| \-sTCP:ESTABLISHED | Established connections only |

## Example Output

```bash
COMMAND PID USER FD TYPE DEVICE NODE NAME
nginx   220 root 10u IPv4 12345 TCP server:443->client:50000 (ESTABLISHED)
```

## Important Columns

| Column | Meaning |
| --- | --- |
| COMMAND | Process name |
| PID | Process ID |
| USER | Owner |
| FD  | File descriptor |
| NAME | Connection details |

## Real Usage

### Port already in use

```bash
lsof -i :8080
```

Find which process owns port.

* * *

# 5\. Process & System Call Tracing

* * *

# ps

## What it does

Shows process information.

## Common Usage

```bash
ps -ef f
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-e | All processes |
| \-f | Full format |
| f   | Process tree |

## Example Output

```bash
UID   PID  PPID  C STIME TTY   TIME CMD
root    1     0  0 10:00 ?     00:00:03 systemd
root  820     1  0 10:01 ?     00:00:00 sshd
user 1450   820  0 10:05 pts/0 00:00:00  \_ bash
```

## Important Columns

| Column | Meaning |
| --- | --- |
| UID | Owner |
| PID | Process ID |
| PPID | Parent process |
| TIME | CPU time used |
| CMD | Command |

## Real Usage

Useful for:

- identifying parent-child processes
- process tree debugging
- Jenkins subprocess debugging

* * *

# strace

## What it does

Traces system calls made by a process.

Useful for:

- diagnosing hangs
- identifying blocking operations
- tracing file/network activity

## Common Usage

### Attach to process

```bash
strace -p 2200
```

### Trace command

```bash
strace ls
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-p | Attach to process |
| \-f | Follow child processes |
| \-o | Write output to file |

## Example Output

```bash
read(3, "hello", 5) = 5
write(1, "ok", 2) = 2
open("/tmp/file", O_RDONLY) = 4
```

## Common System Calls

| Call | Meaning |
| --- | --- |
| read | Read from file/socket |
| write | Write operation |
| open | Open file |
| connect | Network connection |
| futex | Thread synchronization |

## Real Usage

### Application hanging on futex

Usually:

- thread contention
- locking issue

### Hanging on connect

Usually:

- network issue
- unreachable dependency

* * *

# 6\. Logging & Kernel Diagnostics

* * *

# dmesg

## What it does

Shows kernel ring buffer messages.

Useful for:

- hardware issues
- disk failures
- OOM events
- driver problems

## Common Usage

```bash
dmesg | tail
```

### Human-readable timestamps

```bash
dmesg -T
```

## Example Output

```bash
[12345.678] Out of memory: Killed process 2200
[12346.100] eth0: Link is Down
```

## Real Usage

### OOM killer

```bash
Out of memory: Killed process
```

System ran out of RAM.

### Disk errors

```bash
EXT4-fs error
```

Possible filesystem/storage issue.

* * *

# journalctl

## What it does

Shows logs from systemd journal.

Modern Linux logging tool.

## Common Usage

### Service logs

```bash
journalctl -u nginx
```

### Follow logs live

```bash
journalctl -u nginx -f
```

### Current boot logs

```bash
journalctl -b
```

### Kernel logs

```bash
journalctl -k
```

## Important Arguments

| Argument | Meaning |
| --- | --- |
| \-u | Service unit |
| \-f | Follow logs |
| \-b | Current boot |
| \-k | Kernel logs |

## Real Usage

### Service startup failures

```bash
journalctl -u docker
```

Useful for:

- crash investigation
- startup failures
- permission issues

* * *

# Typical Production Troubleshooting Flow

## Step 1: Check overall pressure

```bash
uptime
vmstat 1
```

## Step 2: Check CPU distribution

```bash
mpstat -P ALL 1
pidstat 1
```

## Step 3: Check storage bottleneck

```bash
iostat -xz 1
```

## Step 4: Check network

```bash
sar -n DEV,TCP,ETCP 1
```

## Step 5: Identify exact process behavior

```bash
strace -p PID
```

## Step 6: Check logs

```bash
journalctl -u SERVICE
```

---

# 8. Resource Saturation & Queue Lengths

Understanding **Capacity** vs. **Saturation** is key to diagnosing SRE alerts.

- **Capacity**: How much of a resource is currently active (e.g. CPU at 100%, RAM at 95%, Network utilizing 10Gbps).
- **Saturation**: The amount of overflow work that cannot be processed immediately and is queued waiting for resources. Saturation leads to latency spikes and application timeouts.

Here is how to check capacity, saturation, and queue lengths across subsystems:

### A. CPU Saturation
- **Concept**: CPU cores are fully occupied, and runnable threads must wait in a scheduler run queue.
- **How to detect it**:
  - **Uptime Load Average**: Run `uptime` or `top`. Compare the 1, 5, and 15-minute load averages to the system's logical CPU core count. If the load average is significantly higher than the core count, the CPU scheduler is saturated.
  - **Run Queue Length**: Run `vmstat 1`. The **`r`** (runnable) column displays the number of tasks waiting for CPU execution. If this number is consistently larger than the CPU core count, tasks are queueing.
  - **Context Switches**: Run `vmstat 1`. High values in the **`cs`** (context switches) column indicate that the CPU is spending excessive time switching between threads instead of running application code.

### B. Memory Saturation
- **Concept**: The physical RAM is full, forcing the kernel to reclaim cache or swap active pages to disk.
- **How to detect it**:
  - **Active Swapping**: Run `vmstat 1`. Inspect the **`si`** (swap in) and **`so`** (swap out) columns. If `si`/`so` are continuously non-zero, the system is actively swapping memory pages to disk, causing severe latency.
  - **OOM Events**: Check the kernel logs using `dmesg -T | grep -i oom` or checking `/var/log/syslog`. If you see `Out of memory: Killed process <PID>`, memory was fully saturated and the kernel killed a process to prevent a system crash.
  - **Pressure Stall Information (PSI)**: Inspect `/proc/pressure/memory`. It shows the percentage of time that tasks are delayed due to memory resource unavailability (e.g. `some avg10=2.50` means tasks were delayed 2.5% of the last 10 seconds).

### C. Storage I/O Saturation
- **Concept**: The disk controller is processing requests at maximum throughput/IOPS, causing read/write operations to queue.
- **How to detect it**:
  - **Disk Active Time**: Run `iostat -xz 1`. The **`%util`** column shows the percentage of time the disk was busy. If it is close to 100%, the drive is fully utilized.
  - **I/O Queue Length**: Run `iostat -xz 1`. The **`aqu-sz`** (average queue size) column displays the average number of read or write requests that were queued to the storage device. A high value (e.g., > 2 per disk) indicates saturation.
  - **Wait Latency**: The **`await`** (average wait time in ms) column shows the time taken from issuing an I/O request to its completion. Latencies above 10ms-20ms on SSDs indicate severe drive saturation.
  - **Blocked Processes**: Run `vmstat 1`. The **`b`** column displays the number of processes blocked in uninterruptible sleep (usually waiting for disk I/O).

### D. Network Saturation
- **Concept**: The network interface card (NIC) is processing maximum bandwidth, causing packet queue buffer dropouts or TCP retransmissions.
- **How to detect it**:
  - **TCP Retransmissions**: Run `sar -n ETCP 1`. The **`retrans/s`** column shows the number of TCP segments retransmitted per second. High retransmission rates indicate packet drops due to network congestion/saturation.
  - **Interface Dropped Packets**: Run `ip -s link show eth0` or `ifconfig eth0`. Inspect the `drop` or `overrun` counters under RX (receive) and TX (transmit). Non-zero or growing values indicate buffer overflows at the NIC level.