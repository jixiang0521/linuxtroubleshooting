# linux 性能优化

## CPU

### 进程和CPU的原理

- 进程和线程

	- 进程是系统拥有自愿的最基本单位

		- fork()子进程
		- 进程4中状态：D /R /S/Z

	- 线程是任务调度的最基本单位/线程间共享内存

		- 底层clone()-->偏上层pthead_create()

- CPU调度

	- 处理内核态和处理用户态  ring0-ring4

- 中断系统

	- 中断调度的作用是 cpu及时处理硬件响应/目的：提高CPU使用率 /异步处理机制

		- /proc/interupts

- CPU缓存
- CPU 架构

	- 不同的CPU架构  有无CPU 获取内存的自留地

### 性能指标

- 平均负载

	- 正在运行/等待运行/不可中断状态的平均进程数

- CPU使用率

	- 用户CPU
	- 系统CPU
	- IOWAIT
	- 软中断

		- 精细化处理硬终端后续步骤

			- /proc/softirqs

				- 常见导致软中断CPU问题分析方向--》网络

	- 硬终端

		- 硬件调度CPU

			- /proc/interrupts

	- gt 用户CPU

		- 虚拟化宿主机 

	- si 窃取CPU

		- 虚拟化镜像--》没有隔离镜像内部到看外部

- 上下文切换

	- 自愿上下文

		- 进程无法获取自愿 而被切换

	- 非自愿上下文

		- 进程运行的时间片到了 而被迫 切换

### 性能分析工具

- top/ps
- vmstat
- pidstat 

	- 进程分析工具

- strace

	- 分析进程

- perf

	- 从CPU 角度分析 内核和程序性能
	- perf top / perf report
	- 结合火焰图分析内核性能

- /proc 文件系统 内核态和用户态交互的 文件层
- pstree 进程树 /线程
- execsnoop 分析进程（重点：可以查看 生命周期较短的进程和父进程的使用情况）

### 调优的方式

- 绑定CPU 减少CPU切换的时间
- 进程CPU的限制
- 进程的优先级

	-  （用户进程）nice 越低 优先级越高，CPU优先调度高优先级，优先调度中断（内核）  优先调度 内核   最后调度低优先级 用户进程

- numa 优化
- 中断负载均衡

## 内存

### 内存原理

- 虚拟内存

	- 虚拟内存与通过页管理机制（MMV）与物理内存关联，进程分配的均为 虚拟内存
	- 虚拟内存远远大于物理内存
	- 进程内存各段分布（代码段/数据段/堆（memleak）/栈（局部变量）/文件映射（memleak）

- 地址空间

	- 用户空间（X86  32(3G)/64）
	- 内核空间 (X86  32(1G)/64）

- 内存分配与回收

	- 分配：缺页异常机制

		- 进程获取内存malloc()

	- 回收

		- 回收缓存（slab 管理机制）
		- SWAP  内存回收机制 （不常访问的内存）

			- 不常用文件页（LRU）
			- 脏页--》先刷到磁盘再回收

		- 杀死进程 OOM （oom_score  设置）用户调整触发oom 的阈值

- buffer/cache

	- buffer

		- 不通过文件系统直接写磁盘（裸I/o）一般20MB

	- cache

		- 通过文件系统与硬盘读写交互

			-  slab机制 管理小块内存作为缓存，目的：1.减少频繁回收内存带来性能影响

	- 作用：作为慢速磁盘和快速CPU之间的桥梁

- SWAP 机制

	- 作用：内存回收
	- /proc/sys/vm/swappiness 调整活跃度

		- swappiness 低 ->回收文件页（写文件释放cache 回收） 反之 高--》回收匿名页 （代码中内存 没有实际文件）（swap 刷硬盘 吃性能）

			- 现在环境中 建议直接关闭swap

### 性能指标

- 系统内存使用
- 进程内存使用
- 缓存和缓冲命中率

	- 缓存命中率高--》 性能好

- SWAP指标

### 性能分析

- top/free

	- VIRT  虚拟内存
	- RES 常驻内存
	- SHR 共享内存

- vmstat

	- bi/bo 块设备的读取和写入

- /proc/meminfo
- cachetop/cachestat

### 调优的方法

- 利用缓存
- 减少SWAP
- 限制进程内存资源
- 使用HugePage

## 网络

### 网络原理

- 网络配置
- TCP/IP协议

	- TCP 11种状态机

- 网络收发流程

	- 应用程序--》系统调用--》socket-->tcp/udp-->ip-->链路层-->网卡（硬终断、软中断）

- 路由

	- route -n  网关/目的地址

- 网络防火墙

	- ip_tables(内核)  nat/fileter     input /prerouting/forward/postrouting/output

- 网络QoS

### 性能指标

- 吞吐
- 延迟

	- TTL

- 丢包率
- 重传率（TCP）

### 性能分析

- netstat/ss/ifconfg
- ping
- telnet
- hping --> 可以基于syn ping
- tcpdump/wireshark

	- tcpdump --nn udp port 53 or host   200.200.200.1

- sar

	- sar -n DEV 1

### 性能调优

- 内核调优

	- NAT调优
	- 功能卸载
	- 负载均衡

## 文件系统

### 文件系统的原理

- VFS

	- 存储接口层，对接不同的文件系统，task只与VFS对接即可

- 文件系统I/O栈

	- 直接I/O

		- 跳过页缓存

	- 非直接I/O

		- 经过页缓存

	- 阻塞

		- 等待直到获得结果

	- 非阻塞

		- 执行i/o后 继续执行其他任务，通过轮询或者通知的机制 获取结果（selecl/epoll）

	- 同步i/o
	- 异步i/o
	- 通用块层

		- 文件系统与块设备的中间层 有i/o队列---为整个系统最慢一环

			- 快速向低速读写-缓存队列

	- 设备层

		- 驱动上层

- 文件系统缓存
- 文件系统的种类

### 性能指标

- 容量
- IOPS

	- 小文件类型
	- 大文件类型

### 性能剖析

- df
- sar
- vmstat
- proc文件系统

### 调优方法

- 文件系统选型
- 利用文件系统缓存

## 应用程序

## 磁盘I/O

### 磁盘原理

- 磁盘管理

	- 软硬RAID

- 磁盘类型

	- HDD、SDD

- 磁盘的接口

	- SATA
	- SAS
	- 光纤
	- IDE

- 磁盘i/o栈

	- 磁盘有序读写、无序读写

### 性能指标

- 使用率
- IOPS
- 吞吐量
- IOWAIT

### 性能剖析

- pidstat 进程

	- pidstat -p -d

- iostat  整体

	- /proc/diskstats

		- r/s w/s-->IOPS
		- rKB/s wKB/s--> 吞吐量
		- X_wait --> 响应时间
		- %util 磁盘使用率

- sar

	- sar -d  整体

### 调优方法

- 系统调用
- I/o资源控制
- Raid
- 利用缓存

## linux 内核

### 内核原理

- 内核态

	- 内存管理
	- 任务调度
	- 进程管理
	- 文件管理
	- 中断门

### 性能分析

- perf
- /proc

### 性能调优

- 内核选项

## 架构设计

*XMind - Trial Version*