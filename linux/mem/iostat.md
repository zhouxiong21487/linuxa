# iostat

## 描述

通过iostat方便查看CPU、网卡、tty设备、磁盘、CD-ROM 等等设备的活动情况, 负载信息

## 语法格式

```shell
用法: iostat [ 选项 ] [ <时间间隔> [ <次数> ] ]
选项:
[ -c ] [ -d ] [ -h ] [ -k | -m ] [ -N ] [ -t ] [ -V ] [ -x ] [ -y ] [ -z ]
[ -j { ID | LABEL | PATH | UUID | ... } ]
[ [ -T ] -g <用户组名> ] [ -p [ <设备> [,...] | ALL ] ]
[ <设备> [...] | ALL ]
```

## 常用参数

```shell
-c		显示CPU使用率报告
-d		显示设备利用率报告
-g 		group_name { device [...] | ALL }
        显示一组设备的统计信息，iostat命令报告列表中每个单独设备的统计信息，然后报告组dis-的一行全局统		计信息，充当group_name并由列表中的所有设备组成。ALL关键字表示系统定义的所有块设备都应包括在该		 组中
-h		使设备使用情况报告更易于人阅读
-j 		{ ID | LABEL | PATH | UUID | ... } [ device [...] | ALL ]
        显示永久设备名称，选项ID，LABEL等指定持久名称的类型。这些选项不受限制，仅前提是该目录具有
        必需的永久名称在/dev/disk中。（可选）可以在所选的持久名称类型中指定多个设备。因为永久设备名		    称是通常很长，此选项会隐式启用-h选项
-k		以每秒千字节为单位显示统计信息，默认
-m		以每秒兆字节为单位显示统计信息
-N		显示任何设备映射器设备的注册设备映射器名称。用于查看LVM2统计信息
-p 		[ { device [,...] | ALL } ]
        显示系统使用的块设备及其所有分区的统计信息。如果在命令行上输入了设备名称，则统计信息
        为此，将显示所有分区。最后，ALL关键字指示必须显示所有由块定义的块设备和分区的统计信息。
-T		此选项必须与选项-g一起使用，并指示仅显示该组的全局统计信息，而不显示该组中各个设备的统计信息
-t		打印显示的每个报告的时间
-V		打印版本号，然后退出
-x		显示扩展统计信息
-y		如果以给定的时间间隔显示多个记录，则自系统启动以来忽略带有统计信息的第一个报告
-z		告诉iostat忽略在采样期间没有任何活动的任何设备的输出

```

## 实例

### 1. 显示所有设备负载情况

```shell
[root@localhost ~]# iostat 
Linux 3.10.0-514.el7.x86_64 (fanqq) 	2020年04月15日 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          12.34    0.00    0.70    0.02    0.00   86.93

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.24         9.21         2.84    1277988     394545

```

- **tps**：该设备每秒的传输次数，`一次传输`意思是`一次I/O请求`，多个逻辑请求可能会被合并为`一次I/O请求`,`一次传输`请求的大小是未知的
- **kB_read/s**：每秒从设备读取的数据量
- **kB_wrtn/s**：每秒从设备写入的数据量
- **kB_read **：读取的总数据量
- **kB_wrtn**：写入的总数据量

> 如果`%iowait`的值过高，表示硬盘存在I/O瓶颈，`%idle`值高，表示CPU较空闲，如果`%idle`值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。`%idle`值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU

### 2. 查看设备使用率和响应时间

```shell
[root@localhost ~]# iostat -x -d 
Linux 3.10.0-514.el7.x86_64 (fanqq) 	2020年04月15日 	_x86_64_	(1 CPU)

Device: rrqm/s   wrqm/s  r/s   w/s  rkB/s  wkB/s avgrq-sz avgqu-sz await r_await w_await  svctm  %util
sda     0.00     0.01    0.11  0.13 9.15   2.83  101.58    0.02    68.81  138.42    7.45    5.85   0.14

```

- **rrqm/s**：每秒这个设备相关的读取请求有多少被合并Merge了，（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）
- **wrqm/s**：每秒这个设备相关的写入请求有多少被合并Merge了
- **r/s**：每秒读取的扇区数
- **w/s**：每秒写入的扇区数
- **rkB/s**：每秒读K字节数
- **wkB/s**：每秒写K字节数
- **avgrq-sz**：平均请求扇区的大小
- **avgqu-sz**：是平均请求队列的长度。毫无疑问，队列长度越短越好
- **await**： 平均每个IO所需要的时间，包括在队列等待的时间，也包括磁盘控制器处理本次请求的有效时间
- **r_await**：每个读操作平均所需要的时间，不仅包括硬盘设备读操作的时间，也包括在内核队列中的时间
- **w_await**： 每个写操平均所需要的时间，不仅包括硬盘设备写操作的时间，也包括在队列中等待的时间
- **svctm**：表面看是每个IO请求的服务时间，不包括等待时间，但是实际上，这个指标已经废弃。实际上，iostat工具没有任何一输出项表示的是硬盘设备平均每次IO的时间
- **%util**：工作时间或者繁忙时间占总时间的百分比

> 如果` %util `接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 `svctm `比较接近 `await`，说明 I/O 几乎没有等待时间；如果 `await` 远大于` svctm`，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果`avgqu-sz`比较大，也表示有当量io在等待

### 3. 获取部分cpu状态值

```shell
[root@localhost ~]# iostat -c
Linux 3.10.0-514.el7.x86_64 (fanqq) 	2020年04月15日 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          11.46    0.00    0.66    0.02    0.00   87.86

```

### 4. 查看TPS和吞吐量

```shell
[root@localhost ~]# iostat -d
Linux 3.10.0-514.el7.x86_64 (localhost.localdomain) 	2020年04月15日 	_x86_64_	(1 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.22         8.55         2.66    1278192     397753
```

- **tps**：该设备每秒的传输次数，`一次传输`意思是`一次I/O请求`，多个逻辑请求可能会被合并为`一次I/O请求`,`一次传输`请求的大小是未知的
- **kB_read/s**：每秒从设备读取的数据量
- **kB_wrtn/s**：每秒从设备写入的数据量
- **kB_read **：读取的总数据量
- **kB_wrtn**：写入的总数据量