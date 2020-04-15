# MySQL优化

## 1 MySQL优化概述

### 1.1 优化的哲学

- 优化有风险，涉足需谨慎

#### 1.1.1 优化可能带来的问题

- 优化不总是对一个单纯的环境进行，还很可能是一个复杂的已投产的系统。
- 优化手段本来就有很大的风险，只不过你没能力意识到和预见到！
- 任何的技术可以解决一个问题，但必然存在带来一个问题的风险！
- 对于优化来说解决问题而带来的问题,控制在可接受的范围内才是有成果。
- 保持现状或出现更差的情况都是失败！

#### 1.1.2 优化的需求

- 稳定性和业务可持续性,通常比性能更重要！
- 优化不可避免涉及到变更，变更就有风险！
- 优化使性能变好，维持和变差是等概率事件！
- 切记优化,应该是各部门协同，共同参与的工作，任何单一部门都不能对数据库进行优化！
- 所以优化工作,是由业务需要驱使的！！！

#### 1.1.3 优化由谁参与

- 在进行数据库优化时，应由数据库管理员、业务部门代表、应用程序架构师、应用程序设计人员、应用程序开发人员、硬件及系统管理员、存储管理员等，业务相关人员共同参与。 

### 1.2 优化思路

#### 1.2.1 优化什么

- 在数据库优化上有两个主要方面：即安全与性能。
	- 安全 ---> 数据可持续性
	- 性能 ---> 数据的高性能访问

#### 1.2.2 优化的范围有哪些

- 存储、主机和操作系统方面:
	- 主机架构稳定性
	- I/O规划及配置
	- Swap交换分区
	- OS内核参数和网络问题

- 应用程序方面:
	- 应用程序稳定性
	- SQL语句性能
	- 串行访问资源
	- 性能欠佳会话管理
	- 这个应用适不适合用MySQL

- 数据库优化方面:
	- 内存
	- 数据库结构(物理&逻辑)
	- 实例配置

说明：不管是在，设计系统，定位问题还是优化，都可以按照这个顺序执行。

#### 1.2.3 优化维度

- 数据库优化维度有四个:
  - 硬件、系统配置、数据库表结构、SQL及索引

- 优化选择
  - 优化成本:硬件>系统配置>数据库表结构>SQL及索引
  - 优化效果:硬件<系统配置<数据库表结构<SQL及索引

### 1.3 优化工具有啥？

#### 1.3.1 数据库层面

- 检查问题常用工具

``` shell
mysql
msyqladmin                                 mysql客户端，可进行管理操作
mysqlshow                                  功能强大的查看shell命令
show [SESSION | GLOBAL] variables          查看数据库参数信息
SHOW [SESSION | GLOBAL] STATUS             查看数据库的状态信息
information_schema                         获取元数据的方法
SHOW ENGINE INNODB STATUS                  Innodb引擎的所有状态
SHOW PROCESSLIST                           查看当前所有连接session状态
explain                                    获取查询语句的执行计划
show index                                 查看表的索引信息
slow-log                                   记录慢查询语句
mysqldumpslow                              分析slowlog文件的
```

- 不常用但好用的工具

``` shell
zabbix                  监控主机、系统、数据库（部署zabbix监控平台）
pt-query-digest         分析慢日志
mysqlslap               分析慢日志
sysbench                压力测试工具
mysql profiling         统计数据库整体状态工具    
Performance Schema      mysql性能状态统计的数据
workbench               管理、备份、监控、分析、优化工具（比较费资源）
```

#### 1.3.2 数据库层面问题解决思路

- 一般应急调优的思路：
	- 针对突然的业务办理卡顿，无法进行正常的业务处理！需要立马解决的场景！

``` sql
1、show processlist
2、explain  select id ,name from stu where name='clsn'; # ALL  id name age  sex
    select id,name from stu  where id=2-1 函数 结果集>30;
    show index from table;
3、通过执行计划判断，索引问题（有没有、合不合理）或者语句本身问题
4、show status  like '%lock%';    # 查询锁状态
    kill SESSION_ID;   # 杀掉有问题的session
```


- 常规调优思路：
	- 针对业务周期性的卡顿，例如在每天10-11点业务特别慢，但是还能够使用，过了这段时间就好了。

``` sql
1、查看slowlog，分析slowlog，分析出查询慢的语句。
2、按照一定优先级，进行一个一个的排查所有慢语句。
3、分析top sql，进行explain调试，查看语句执行时间。
4、调整索引或语句本身。
```

#### 1.3.3 系统层面

``` shell
1.cpu方面
vmstat、sar top、htop、nmon、mpstat
2.内存
free 、ps -aux 、
3.IO设备（磁盘、网络）
iostat 、 ss  、 netstat 、 iptraf、iftop、lsof、

vmstat 命令说明：
Procs：r显示有多少进程正在等待CPU时间。b显示处于不可中断的休眠的进程数量。在等待I/O
Memory：swpd显示被交换到磁盘的数据块的数量。未被使用的数据块，用户缓冲数据块，用于操作系统的数据块的数量
Swap：操作系统每秒从磁盘上交换到内存和从内存交换到磁盘的数据块的数量。s1和s0最好是0
Io：每秒从设备中读入b1的写入到设备b0的数据块的数量。反映了磁盘I/O
System：显示了每秒发生中断的数量(in)和上下文交换(cs)的数量
Cpu：显示用于运行用户代码，系统代码，空闲，等待I/O的CPU时间

iostat命令说明
实例命令：iostat -dk 1 5
　　　　    iostat -d -k -x 5 （查看设备使用率（%util）和响应时间（await））
tps：该设备每秒的传输次数。“一次传输”意思是“一次I/O请求”。多个逻辑请求可能会被合并为“一次I/O请求”。
iops ：硬件出厂的时候，厂家定义的一个每秒最大的IO次数
"一次传输"请求的大小是未知的。
kB_read/s：每秒从设备（drive expressed）读取的数据量；
KB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。
```

#### 1.3.4 系统层面问题解决办法

- 你认为到底负载高好，还是低好呢？
	- 在实际的生产中，一般认为 cpu只要不超过90%都没什么问题 。

- 当然不排除下面这些特殊情况：

``` shell
问题一：cpu负载高，IO负载低
内存不够
磁盘性能差
SQL问题  ------>去数据库层，进一步排查sql问题
IO出问题了（磁盘到临界了、raid设计不好、raid降级、锁、在单位时间内tps过高）
tps过高: 大量的小数据IO、大量的全表扫描
```

- 问题二：IO负载高，cpu负载低

``` 
大量小的IO 写操作：
　　autocommit   ，产生大量小IO
　　IO/PS,磁盘的一个定值，硬件出厂的时候，厂家定义的一个每秒最大的IO次数。
大量大的IO 写操作
　　SQL问题的几率比较大
```

- 问题三：IO和cpu负载都很高

```
硬件不够了或sql存在问题
```

### 1.4 基础优化

#### 1.4.1 优化思路

- 定位问题点吮吸
  - 硬件 --> 系统 --> 应用 --> 数据库(参数,sql,索引) --> 架构（高可用、读写分离、分库分表）

- 处理方向
	- 明确优化目标、性能和安全的折中、防患未然

#### 1.4.2 硬件优化

- 主机方面：
  - 根据数据库类型，主机CPU选择、内存容量选择、磁盘选择
  - 平衡内存和磁盘资源
  - 随机的I/O和顺序的I/O
  - 主机 RAID卡的BBU(Battery Backup Unit)关闭

- cpu的选择：
	- cpu的两个关键因素：核数、主频
	- 根据不同的业务类型进行选择：
	- cpu密集型：计算比较多，OLTP     主频很高的cpu、核数还要多
	- IO密集型：查询比较，OLAP         核数要多，主频不一定高的

- 内存的选择：
	- OLAP类型数据库，需要更多内存，和数据获取量级有关。
	- OLTP类型数据一般内存是cpu核心数量的2倍到4倍，没有最佳实践。

- 存储方面：
	- 根据存储数据种类的不同，选择不同的存储设备
	- 配置合理的RAID级别(raid5、raid10、热备盘)
	- 对与操作系统来讲，不需要太特殊的选择，最好做好冗余（raid1）（ssd、sas 、- sata）
	- raid卡：主机raid卡选择：
		　　实现操作系统磁盘的冗余（raid1）
	　　　平衡内存和磁盘资源
	　　　随机的I/O和顺序的I/O
	　　　主机 RAID卡的BBU(Battery Backup Unit)要关闭。

- 网络设备方面：
	- 使用流量支持更高的网络设备（交换机、路由器、网线、网卡、HBA卡）

注意：以上这些规划应该在初始设计系统时就应该考虑好。

#### 1.4.3 服务器硬件优化

- 1、物理状态灯：
- 2、自带管理设备：远程控制卡（FENCE设备：ipmi ilo idarc），开关机、硬件监控。
- 3、第三方的监控软件、设备（snmp、agent）对物理设施进行监控
- 4、存储设备：自带的监控平台。EMC2（hp收购了）， 日立（hds），IBM低端OEM hds，高端存储是自己技术，华为存储

#### 1.4.4 系统优化

- Cpu：
	- 基本不需要调整，在硬件选择方面下功夫即可。

- 内存：
	- 基本不需要调整，在硬件选择方面下功夫即可。

- SWAP：
	- MySQL尽量避免使用swap。
	- 阿里云的服务器中默认swap为0

- IO ：
	- raid、no lvm、 ext4或xfs、ssd、IO调度策略

- Swap调整(不使用swap分区)

``` shell
/proc/sys/vm/swappiness的内容改成0（临时），/etc/sysctl.conf上添加vm.swappiness=0（永久）
这个参数决定了Linux是倾向于使用swap，还是倾向于释放文件系统cache。在内存紧张的情况下，数值越低越倾向于释放文件系统cache。
当然，这个参数只能减少使用swap的概率，并不能避免Linux使用swap。
修改MySQL的配置参数innodb_flush_method，开启O_DIRECT模式。
这种情况下，InnoDB的buffer pool会直接绕过文件系统cache来访问磁盘，但是redo log依旧会使用文件系统cache。
值得注意的是，Redo log是覆写模式的，即使使用了文件系统的cache，也不会占用太多
```

- IO调度策略

``` shell
echo deadline>/sys/block/sda/queue/scheduler   临时修改为deadline
永久修改
vi /boot/grub/grub.conf
更改到如下内容:
kernel /boot/vmlinuz-2.6.18-8.el5 ro root=LABEL=/ elevator=deadline rhgb quiet
```

#### 1.4.5 系统参数调整

- Linux系统内核参数优化

``` shell
vim /etc/sysctl.conf
   net.ipv4.ip_local_port_range = 1024 65535   # 用户端口范围
   net.ipv4.tcp_max_syn_backlog = 4096 
   net.ipv4.tcp_fin_timeout = 30 
   fs.file-max=65535          # 系统最大文件句柄，控制的是能打开文件最大数量
```

- 用户限制参数（mysql可以不设置以下配置）

``` shell
vim    /etc/security/limits.conf 
   * soft nproc 65535
   * hard nproc 65535
   * soft nofile 65535
   * hard nofile 65535
```

#### 1.4.6 应用优化

- 业务应用和数据库应用独立,
- 防火墙：iptables、selinux等其他无用服务(关闭)：

```
chkconfig --level 23456 acpid off
   chkconfig --level 23456 anacron off
   chkconfig --level 23456 autofs off
   chkconfig --level 23456 avahi-daemon off
   chkconfig --level 23456 bluetooth off
   chkconfig --level 23456 cups off
   chkconfig --level 23456 firstboot off
   chkconfig --level 23456 haldaemon off
   chkconfig --level 23456 hplip off
   chkconfig --level 23456 ip6tables off
   chkconfig --level 23456 iptables  off
   chkconfig --level 23456 isdn off
   chkconfig --level 23456 pcscd off
   chkconfig --level 23456 sendmail  off
   chkconfig --level 23456 yum-updatesd  off
```

- 安装图形界面的服务器不要启动图形界面  runlevel 3 
- 另外，思考将来我们的业务是否真的需要MySQL，还是使用其他种类的数据库。用数据库的最高境界就是不用数据库。

### 1.5 数据库优化

- SQL优化方向：
	- 执行计划、索引、SQL改写

- 架构优化方向：
    - 高可用架构、高性能架构、分库分表

### 1.5.1 数据库参数优化

调整：
- 实例整体（高级优化，扩展）：

``` shell

thread_concurrency=64

#thread_concurrency的值的正确与否, 对mysql的性能影响很大, 在多个cpu(或多核)的情#况下，错误设置了thread_concurrency的值, 会导致mysql不能充分利用多cpu(或多核), #出现同一时刻只能一个cpu(或核)在工作的情况。
#
#thread_concurrency应设为CPU核数的2倍. 比如有一个双核的CPU, 那#thread_concurrency  的应该为4; 2个双核的cpu, thread_concurrency的值应为8.
#
#比如：根据上面介绍我们目前系统的配置，可知道为4个CPU,每个CPU为8核，按照上面的计算规则，这儿应为:4*8*2=64
#
#查看系统当前thread_concurrency默认配置命令：
#
# show variables like 'thread_concurrency';

sort_buffer_size = 4M

#默认为256KB

# Sort_Buffer_Size 是一个connection级参数，在每个connection（session）第一次需要使用这个buffer的时候，一次性分配设置的内存。

#Sort_Buffer_Size 并不是越大越好，由于是connection级的参数，过大的设置+高并发可能会耗尽系统内存资源。例如：500个连接将会消耗 500*sort_buffer_size(8M)=4G内存

#Sort_Buffer_Size 超过2KB的时候，就会使用mmap() 而不是 malloc() 来进行内存分配，导致效率降低。

join_buffer_size = 2M

#默认为256K

#用于表间关联缓存的大小，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。

#如果应用中，很少出现join语句，则可以不用太在乎join_buffer_size参数的设置大小。

#如果join语句不是很少的话，个人建议可以适当增大join_buffer_size到1MB左右，如果内存充足可以设置为2MB


key_buffer_size=400M

#     key_buffer_size是用于索引块的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，对MyISAM(MySQL表存储的一种类型，可以百度等查看详情)表性能影响最大的一个参数。如果你使它太大，系统将开始换页并且真的变慢了。严格说是它决定了数据库索引处理的速度，尤其是索引读的速度。对于内存在4GB左右的服务器该参数可设置为256M或384M.

# 怎么才能知道key_buffer_size的设置是否合理呢，一般可以检查状态值Key_read_requests和Key_reads   ，比例key_reads / key_read_requests应该尽可能的低，比如1:100，1:1000 ，1:10000。其值可以用以下命令查得：show status like 'key_read%';

# 比如查看系统当前key_read和key_read_request值为：

# +-------------------+-------+

# | Variable_name     | Value |

# +-------------------+-------+

# | Key_read_requests | 28535 |

# | Key_reads         | 269   |

# +-------------------+-------+

# 可知道有28535个请求，有269个请求在内存中没有找到直接从硬盘读取索引.

# 未命中缓存的概率为：0.94%=269/28535*100%.  一般未命中概率在0.1之下比较好。目前已远远大于0.1，证明效果不好。若命中率在0.01以下，则建议适当的修改key_buffer_size值。


read_buffer_size=4M #（默认值：2097144即2M）

#        read_buffer_size 是MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。read_buffer_size变量控制这一

# 缓冲区的大小。如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能.


read_rnd_buffer_size=8M # (默认值：8388608即8M)

# read_rnd_buffer_size=8M

# read_rnd_buffer_size 是MySql的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开

# 销过大。

tmp_table_size=16M

#    tmp_table_size是MySql的heap （堆积）表缓冲大小。所有联合在一个DML指令内完成，并且大多数联合甚至可以不用临时表即可以完成。大多数临时表是基于内

# 存的(HEAP)表。具有大的记录长度的临时表 (所有列的长度的和)或包含BLOB列的表存储在硬盘上。如果某个内部heap（堆积）表大小超过tmp_table_size，MySQL可以根据需要自

# 动将内存中的heap表改为基于硬盘的MyISAM表。还可以通过设置tmp_table_size选项来增加临时表的大小。也就是说，如果调高该值，MySql同时将增加heap表的大小，可达到提高

# 联接查询速度的效果。

record_buffer = 128K

#   record_buffer每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，你可能想要增加该值。默认数值是131072

# (128K)

table_open_cache = 1024 (默认值：512)

 

# TABLE_CACHE(5.1.3及以后版本又名TABLE_OPEN_CACHE)

# table_cache指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。通过检查峰值时间的状态值Open_tables和Opened_tables，可以决定是否需要增加table_cache的值。如果你发现open_tables等于table_cache，并且opened_tables在不断增长，那么你就需要增加table_cache的值了（上述状态值可以使用SHOW STATUS LIKE ‘Open%tables’获得）。注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。

# SHOW STATUS LIKE 'Open%tables';

# +---------------+-------+

# | Variable_name | Value |

# +---------------+-------+

# | Open_tables   | 356   |

# | Opened_tables | 0     |

# +---------------+-------+

# 2 rows in set (0.00 sec)

# open_tables表示当前打开的表缓存数，如果执行flush tables操作，则此系统会关闭一些当前没有使用的表缓存而使得此状态值减小；

# opend_tables表示曾经打开的表缓存数，会一直进行累加，如果执行flush tables操作，值不会减小。

# 在mysql默认安装情况下，table_cache的值在2G内存以下的机器中的值默认时256到512，如果机器有4G内存,则默认这个值 是2048，但这决意味着机器内存越大，这个值应该越大，因为table_cache加大后，使得mysql对SQL响应的速度更快了，不可避免的会产生 更多的死锁（dead lock），这样反而使得数据库整个一套操作慢了下来，严重影响性能。所以平时维护中还是要根据库的实际情况去作出判断，找到最适合你维护的库的 table_cache值。

# 由于MySQL是多线程的机制,为了提高性能,每个线程都是独自打开自己需要的表的文件描 述符,而不是通过共享已经打开的.针对不同存储引擎处理的方法当然也不一样

# 在myisam表引擎中,数据文件的描述符 (descriptor)是不共享的,但是索引文件的描述符却是所有线程共享的.Innodb中和使用表空间类型有关,假如是共享表空间那么实际就一个数 据文件,当然占用的数据文件描述符就会比独立表空间少.
# mysql手册上给的建议大小 是:table_cache=max_connections*n

# n表示查询语句中最大表数, 还需要为临时表和文件保留一些额外的文件描述符。

# 这个数据遭到很多质疑,table_cache够用就好,检查 Opened_tables值,如果这个值很大,或增长很快那么你就得考虑加大table_cache了.

#   table_cache：所有线程打开的表的数目。增大该值可以增加mysqld需要的文件描述符的数量。默认值是64.

   thread_cache_size=64     # 服务器线程缓存 (1G—>8, 2G—>16, 3G—>32, >3G—>64)

# 默认的thread_cache_size=8，但是看到好多配置的样例里的值一般是32，64，甚至是128，感觉这个参数对优化应该有帮助，于是查了下：
# 根据调查发现以上服务器线程缓存thread_cache_size没有进行设置，或者设置过小,这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时如果缓存中还有空间,那么客户端的线程将被放到缓存中,如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用。(–>表示要调整的值)   根据物理内存设置规则如下：
# 1G —> 8
# 2G —> 16
# 3G —> 32     >3G —> 64

#   mysql> show status like 'thread%';
# +——————-+——-+
# | Variable_name     | Value |
# +——————-+——-+
# | Threads_cached    | 0     |  <—当前被缓存的空闲线程的数量
# | Threads_connected | 1     |  <—正在使用（处于连接状态）的线程
# | Threads_created   | 1498  |  <—服务启动以来，创建了多少个线程
# | Threads_running   | 1     |  <—正在忙的线程（正在查询数据，传输数据等等操作）
# +——————-+——-+

# 查看开机起来数据库被连接了多少次？

# mysql> show status like '%connection%';
# +———————-+——-+
# | Variable_name        | Value |
# +———————-+——-+
# | Connections          | 1504  |          –>服务启动以来，历史连接数
# | Max_used_connections | 2     |
# +———————-+——-+

# 通过连接线程池的命中率来判断设置值是否合适？命中率超过90%以上,设定合理。

#  (Connections -  Threads_created) / Connections * 100 %
```

- 连接层（基础优化）
	- 设置合理的连接客户和连接方式

``` shell

    max_connections=3000
# 修改max_connections参数值，由默认的151，修改为3000（750M）。

# max_connections是指MySql的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySql会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。可以过'conn%'通配符查看当前状态的连接数量，以定夺该值的大小。

# MySQL服务器允许的最大连接数16384；

# 查看系统当前最大连接数：

# show variables like 'max_connections';

   connect_timeout=10           # 连接超时采用默认10s
     max_user_connections=800 # 最大用户连接数

#  修改max_user_connections值，由默认的0，修改为800
#  max_user_connections是指每个数据库用户的最大连接

# 针对某一个账号的所有客户端并行连接到MYSQL服务的最大并行连接数。简单说是指同一个账号能够同时连接到mysql服务的最大连接数。设置为0表示不限制。

# 目前默认值为：0不受限制。

# 这儿顺便介绍下Max_used_connections:它是指从这次mysql服务启动到现在，同一时刻并行连接数的最大值。它不是指当前的连接情况，而是一个比较值。如果在过去某一个时刻，MYSQL服务同时有1000个请求连接过来，而之后再也没有出现这么大的并发请求时，则Max_used_connections=1000.请注意与show variables 里的max_user_connections的区别。默认为0表示无限大。

# 查看max_user_connections值

# show variables like 'max_user_connections';
   skip-name-resolve         # 跳过域名解析
# 添加skip-name-resolve，默认被注释掉，没有该参数。
# skip-name-resolve

# skip-name-resolve：禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求！
   wait_timeout=1800 #（单位为秒）
   interactive_timeout=1800
#    修改wait_timeout参数值，由默认的8小时，修改为30分钟。(本次不用)

# 我对wait-timeout这个参数的理解：MySQL客户端的数据库连接闲置最大时间值。

# 说得比较通俗一点，就是当你的MySQL连接闲置超过一定时间后将会被强行关闭。MySQL默认的wait-timeout  值为8个小时，可以通过命令show variables like 'wait_timeout'查看结果值;。

# 设置这个值是非常有意义的，比如你的网站有大量的MySQL链接请求（每个MySQL连接都是要内存资源开销的 ），由于你的程序的原因有大量的连接请求空闲啥事也不干，白白占用内存资源，或者导致MySQL超过最大连接数从来无法新建连接导致“Too many connections”的错误。在设置之前你可以查看一下你的MYSQL的状态（可用show processlist)，如果经常发现MYSQL中有大量的Sleep进程，则需要 修改wait-timeout值了。

# interactive_timeout：服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql_real_connect()中使用CLIENT_INTERACTIVE选项的客户端。

# wait_timeout:服务器关闭非交互连接之前等待活动的秒数。在线程启动时，根据全局wait_timeout值或全局 interactive_timeout值初始化会话wait_timeout值，取决于客户端类型(由mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义).

# 这两个参数必须配合使用。否则单独设置wait_timeout无效
   back_log=500                  # 可以在堆栈中的连接数量

#    修改back_log参数值:由默认的50修改为500.（每个连接256kb,占用：125M）


#     back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果MySql的连接数据达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。将会报：unauthenticated user | xxx.xxx.xxx.xxx | NULL | Connect | NULL | login | NULL 的待连接进程时.

# back_log值不能超过TCP/IP连接的侦听队列的大小。若超过则无效，查看当前系统的TCP/IP连接的侦听队列的大小命令：cat /proc/sys/net/ipv4/tcp_max_syn_backlog目前系统为1024。对于Linux系统推荐设置为小于512的整数。

# 修改系统内核参数，

# 查看mysql 当前系统默认back_log值，命令：

# show variables like 'back_log'; 查看当前数量
```

- SQL层（基础优化）

``` shell
query_cache_size=0 # ：查询缓存  默认关闭
# >>>  
# OLAP类型数据库,需要重点加大此内存缓存，
# 但是一般不会超过GB
# 对于经常被修改的数据，缓存会立马失效。
# 我们可以实用内存数据库（redis、memecache），替代他的功能。
```

### 1.5.2 存储引擎层（innodb基础优化参数）

``` shell
default-storage-engine= InnoDB #(设置InnoDB类型，另外还可以设置MyISAM类型)
innodb_thread_concurrency=32
# 表示SQL经过解析后，允许同时有32个线程去innodb引擎取数据，如果超过32个，则需要排队；
# 值太大会产生热点数据，global锁争用严重，影响性能
innodb_buffer_pool_size       # 没有固定大小，50%测试值，看看情况再微调。但是尽量设置不要超过物理内存70%

  innodb_additional_mem_pool_size=20M # (默认8M)

#      innodb_additional_mem_pool_size 设置了InnoDB存储引擎用来存放数据字典信息以及一些内部数据结构的内存空间大小，所以当我们一个MySQL Instance中的数据库对象非常多的时候，是需要适当调整该参数的大小以确保所有数据都能存放在内存中提高访问效率的。

# 这个参数大小是否足够还是比较容易知道的，因为当过小的时候，MySQL会记录Warning信息到数据库的error log中，这时候你就知道该调整这个参数大小了。

# 查看当前系统mysql的error日志  cat  /var/lib/mysql/机器名.error 发现有很多waring警告。所以要调大为20M.

# 根据MySQL手册，对于2G内存的机器，推荐值是20M。

    # 32G内存的 100M
innodb_io_capacity=20000
# 每秒后台进程处理IO数据的上限，一般为IO QPS总能力的75%
# 比如SSD是3W QPS，75%大概是2W，双实例减半，为1W，几个实例除以几
innodb_file_per_table=on
# 一表一文件，可以避免共享表空间的IO竞争
innodb_flush_log_at_trx_commit=1 
# 0，不管有没有提交，每秒钟都写到binlog日志里
# 1，每次提交事务，都会把log buffer的内容写到磁盘里去，对日志文件做到磁盘刷新，安全最好
# 2，每次提交事务，都写到操作系统缓存，由OS刷新到磁盘，性能最好
innodb_flush_method=O_DIRECT
# SSD直接写硬盘，不写硬盘cache，也就是绕过fsync()刷硬盘
innodb_flush_neighbors=0
# SSD设置为0，SAS打开刷新相邻块，随机访问转换为顺序访问
innodb_page_size=4k
# 默认是16K，这里是SSD，写SSD前要擦除，擦除单位是extent，一个extent有128个page组成，16128 > 4128 ，效率会更高
innodb_log_files_in_group=4
# 几个innodb redo log日志组
innodb_log_file_size=1000M
# redo log日志循化写，生产必须大于1G，
# 如果太小，那么innodb_buffer_pool_size的数据有可能不能及时写入redo log造成halt等待；查看是否够用？如果value大于0，则提高改参数或者增加日志组
# root@master 12:51: [(none)]> show global status like '%log_wait%'; +------------------+-------+ | Variable_name | Value | +------------------+-------+ | Innodb_log_waits | 0 | +------------------+-------+ 1 row in set (0.00 sec) root@master 12:54: [(none)]> show global status like '%Innodb_os_log_written%'; +-----------------------+-------+ | Variable_name | Value | +-----------------------+-------+ | Innodb_os_log_written | 1024 | +-----------------------+-------+ 1 row in set (0.00 sec) #此参数大小可作为设置日志文件size大小参考值
innodb_max_dirty_pages_pct   # 默认达到百分之75的时候刷写 内存脏页到磁盘。

log_bin # 开启
sync_binlog=1
# 0，事务提交后，mysql不做fsync之类的刷盘，由文件系统来决定什么落盘
# n，多少次提交，每n次提交持久化磁盘
# 生产设为1
```

## 2 MySQL优化深入

### 2.1 mysql优化原理

页面静态化，memcache/redis是通过减少对mysql 操作来提升访问速度，但是一个网站总是要操作数据库，我们如何提升对mysql的操作速度。
方针：
- ① 存储层：数据表”存储引擎”选取、字段类型选取、逆范式(3范式)
- ② 设计层：索引、分区/分表、存储过程，sql语句的优化
- ③ 架构层：分布式部署(集群)(读写分离)，需要增加硬件
- ④ sql语句层：结果一样的情况下，要选择效率高、速度快、节省资源的sql语句执行

### 2.2 存储引擎的选择

#### 2.2.1 存储引擎介绍

熟悉的存储引擎：Myisam、innodb  memory

- （1）什么是存储引擎？

```
数据表存储数据的一种格式。
数据存储在不同的格式里边，该格式体现的特性也是不一样的。例如innodb存储引擎的特性有支持事务、支持行级锁，mysiam支持的特性有压缩机制等。

MySQL中的数据是通过各种不同的技术(格式)存储在文件（或者内存）中的。技术和本身的特性就称为"存储引擎"。
```

- （2）存储引擎的理解：


```
现实生活中，楼房、平房就是具体存储人的存储引擎，楼房、平房有自己独特的技术特性
例如楼房有楼梯、电梯、平房可以自己打井喝水等。
```


- （3）存储引擎所处的位置：

```
存储引擎，处于MySql服务器的最底层，直接存储数据，导致上层的操作，依赖于存储引擎的选择。
客户端-》网络连接层-》业务逻辑层（编译，优化，执行SQL）-》存储引擎层

查看当前mysql支持的存储引擎列表：show engines
```

- （4）常用存储引擎：

```
① Myisam：表锁，全文索引
② Innodb：行(记录)锁，事务（回滚），外键
③ Memory：内存存储引擎，速度快、数据容易丢失
```

#### 2.2.2 innodb存储引擎


\>=5.5 版本中默认的存储引擎，MySql推荐使用的存储引擎。提供事务，行级锁定，存储引擎。
事务安全型存储引擎，更加注重数据的完整性和安全性。

-（1）存储格式：

``` sql
innodb存储引擎  每个数据表有单独的“结构文件”  *.frm
数据，索引集中存储，存储于同一个表空间文件中ibdata1。
ibdata1就是InnoDB表的共享存储空间，默认innodb所有表的数据都在一个ibdata1里。
创建innodb表后，存在文件如下：
create table t1(id int,name varchar(32)) engine innodb charset utf8;

.frm表结构文件。

innodb表空间文件：存储innodb的数据和索引。
ibdata1

默认，所有的 innodb表的数据和索引在同一个表空间文件中，
通过配置可以达到每个innodb的表对应一个表空间文件。
show  variables   like ‘innodb_file_per_table%’

开启该配置：
set  global  innodb_file_per_table=1;


创建一个innodbd的表进行测试使用。

查看表对应的文件自己独立的“数据/索引”文件

系统配置参数innodb_file_per_table后期无论发生任何变化，t2都有自己独立的“数据/索引”文件。
注意：相比较之下，使用独占表空间的效率以及性能会更高一点。
注意：innodb数据表不能直接进行文件的复制/粘贴进行备份还原，可以使用如下指令：

备份数据库的指令
> mysqldump  -uroot -p密码  数据库名字 > f:/文件名称.sql  [备份]


> mysql -uroot  -p密码 数据库   <  f:/文件名称.sql  [还原]
```

- （2）数据是按照主键顺序存储。

```
该innodb数据表，数据的写入顺序 与 存储的顺序不一致，需要按照主键的顺序把记录摆放到对应的位置上去，速度比Myisam的要稍慢。
create table t3(
id int primary key auto_increment,
name varchar(32) not null
)engine innodb charset utf8;
insert into t3 values(223,'刘备'),(12,'张飞'),(162,'张聊'),(1892,'网飞');
给innodb数据表写入4条记录信息(主键id值顺序不同)
插入时做排序工作，效率低。
```

- （3）并发处理：

```
擅长处理并发。
行级锁定(row-level locking)，实现了行级锁定，在一定情况下，可以选择行级锁来提升并发性，也支持表级锁定，innodb根据操作选择。
锁机制：
当客户端操作表（记录）时，为了保证操作的隔离性（多个客户端操作不能相互影响），通过加锁来处理。
操作方面：
读锁：读操作时增加的锁，也叫共享锁，S-lock。特征是所有人都只可以读，只有释放锁之后才可以写。
写锁：写操作时增加的锁，也叫独占锁或排他锁，X-lock。特征，只有锁表的客户可以操作（读写）这个表，其他客户读都不能读。
办公室开会锁上门。
锁定粒度（范围）
表级锁：开销小，加锁快，发生锁冲突的概率最高，并发度最低。myisam和innodb都支持。
行级锁：开销大，加锁慢，发生锁冲突的概率最低，并发度也最高。innodb支持 
```

#### 2.2.3、MyISAM存储引擎
<=5.5mysql默认的存储引擎。
（ISAM——索引顺序访问方法）是Indexed Sequential Access Method(索引顺序存取方法)的缩写 
它是一种索引机制，用于高效访问文件中的数据行，擅长与处理高速读与写。

- （1）存储方式：
  
```
数据，索引，结构分别存储于不同的文件中。
create table t4(id int,name varchar(32)) engine myisam charset utf8;

mysiam存储引擎数据表，每个数据表都有三个文件*.frm（结构文件） *.MYD(数据文件)  *.MYI(索引文件)
这三个文件支持物理复制、粘贴操作(直接备份还原)。
```

- （2）数据的存储顺序为插入顺序。

``` sql
create table t5(
id int primary key auto_increment,
name varchar(32) not null
)engine myisam  charset utf8;
insert into t5 values(2223,'刘备'),(12,'张飞'),(162,'张聊'),(1892,'网飞');
数据查询的顺序，与写入的顺序一致。
数据写入时候，没有按照主键id值给予排序存储，该特点导致数据写入的速度非常快。
```

- （3）并发性

```
mysiam的并发性较比innodb要稍逊色（mysiam不支持事务）
因为数据表是“表锁”
myisam和innodb的取舍
如果表对事务的要求不高，同时是以查询和添加为主，我们考虑使用MyISAM存储引擎，比如bbs中的发帖表，回复表。
INNODB存储引擎：
对事务要求高，保存的数据都是重要数据，我们建议使用INNODB，比如订单表，库存表，商品表，账号表等等。
购买成功了库存 -1，
产生订单，操作表
```

#### 2.2.4、memory

内存存储引擎，	

```sql
特点：内部数据运行速度非常快，临时存储一些信息
缺点：服务器如果断电，重启，就会清空该存储引擎的全部数据
create table t6(id int,name varchar(32)) engine memory charset utf8;
mysql服务，重启后，数据丢失。
```

### 2.3 查找需要优化语句

#### 2.3.1、慢查询日志
是一种mysql提供的日志，记录所有执行时间超过某个时间界限的sql的语句。这个时间界限，我们可以指定。在mysql中默认没有开启慢查询，即使开启了，只会记录执行的sql语句超过10秒的语句。

- 方式一、临时启动慢查询记录日志

``` SQL
（1）{mysql程序所在的目录}>bin/mysqld.exe   --safe-mode  --slow-query-log 
注意：先把mysql关闭后，再执行以上指令启动。

进入cmd开始启动；
>bin/mysqld.exe   --safe-mode  --slow-query-log

通过慢查询日志定位执行效率较低的SQL语句。慢查询日志记录了所有执行时间超过long_query_time所设置的SQL语句。

（2）在默认情况下，慢查询日志是存储到data目录下面的。根据配置文件里面的配置，找到data的存储路径。

（3）可以通过命令查看慢查询日志的时间
show variables like ‘long_query_time’;

修改慢查询日志时间：set  long_query_time=0.5;

（4）测试查询：
查看慢查询日志 
benchmark(count,expr)函数可以测试执行count次expr操作需要的时间。
select benchmark(10000,90000000*4)



（5）一般情况下，一个sql语句执行比较慢，原因是没有索引
添加索引之前,索引文件大小如下；
没有添加索引之前查询时间如下：



添加索引之后：alter  table  emp add index(empno)

添加索引后，索引文件变大。

添加索引之后需要的时间；

结论：创建完索引后，索引文件会变大，添加索引会明显的提高查询速度。
```

- 方式二：通过修改配置文件，添加如下语句

``` SQL
在配置文件中指定：（1）开启（2）时间界限

log-slow-queries="d:/slow-log"
慢查询日志文件存储的路径，当前是把慢查询日志存储到d:盘下面，文件名为slow-log
long_query_time=1
指定慢查询的时间，默认是10秒，我们自定义成1或0.5秒，也就是说当一个sql语句的执行速度超过1秒时，会把该语句添加到慢查询日志里面，
通过配置文件是永远的开启慢查询日志
```

### 2.3.2、精确记录查询时间

``` SQL
使用mysql提供profile机制完成。
profile记录每次执行的sql语句的具体时间，精确时间到小数点8位
（1）开启profile机制:
set profiling = 1;

执行需要分析的sql语句（自动记录该sql的时间）
（2）查看记录sql语句的执行时间：
show profiles;

注意：不需要分析时，最好将其关闭。
set profiling=0;
```

### 2.4 索引讲解

#### 2.4.1、索引的基本介绍

利用关键字，就是记录的部分数据（某个字段，某些字段，某个字段的一部分），建立与记录位置的对应关系，就是索引。
索引的作用：是用于快速定位实际数据位置的一种机制。
例如：
字典的   检索
写字楼   导航
索引在mysql中，是独立于数据的一种特殊的数据结构。
索引一定有顺序（排好序的快速查找结构），记录则不一定。

测试添加索引前后，对比执行时间。

#### 2.4.2、索引的类型：
4种类型：
主键索引，唯一索引，普通索引，全文索引。
无论任何类型，都是通过建立关键字与位置的对应的关系来实现的。
以上类型的差异，是对关键字的要求不同。
关键字：记录的部分数据（某个字段，某些字段，某个字段的一部分）
`普通索引`：对关键字没有要求。
`唯一索引`：要求关键字不能重复，同时增加唯一约束。
`主键索引`：要求关键字不能重复，也不能为NULL。同时增加主键约束。
`全文索引`：关键字的来源不是所有字段的数据，而是从字段中提取的特别关键词。
关键词的来源：可以是某个字段，也可以是某些字段（`复合索引`）。如果一个索引通过在多个字段上提取的关键字，称之为复合索引。
比如：alter table emp add   index (field1,field2)

#### 2.4.3、索引管理语法
- （1）创建索引：

``` sql
建表时：
注意：索引可以起名字，但是主索引不能起名字，因为一个表仅仅可以有一个主键索引，其他索引可以出现多个。名字可以省略，mysql会默认生成，通常使用字段名来充当。
show create table index1;

更新表结构
alter table index2 add unique key (name),add index(age),add fulltext index(intro),add index(name,age);

注意：
第一点：如果表中存在数据，数据符合唯一或主键约束才可能创建成功。
第二点：auto_increment属性，依赖于一个KEY（主键或唯一）。
```

- （2）删除索引


``` sql
修改表结构时完成：
删除主键索引：alter table  table_name drop primary key 
主键索引的删除，如果没有auto_increment 属性则使用 alter table  表名 drop primary key
如果在删除主键索引时，该字段中有auto_increment则先去掉该属性再删除。
去除主键的auto_inrement属性：
alter table 表名 modify id int unsigned not null comment '主键'

如有主键中有auto_incrments属性时，删除主键索引，则报如图提示。

去除主键的auto_inrement属性：
alter table index1 modify id int unsigned;

删除普通索引，唯一索引，全文索引，复合索引；
语法：
alter table 表名  drop index 索引的名称；

如果没有指定索引名，则可以通过查看索引的方法得到索引名（一般依赖于索引字段的名字）
```

- （3）查看索引

``` sql
show indexes from table_name;
show index from table_name\G
show create table table_name
show keys from table_name
desc table_name
```

- （4）创建索引注意事项：

``` sql
第一：较频繁的作为查询条件字段应该创建索引
	select * from emp where empno = 1
第二：唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件
	select * from emp where sex = '男‘
第三：更新非常频繁的字段不适合创建索引
	select * from emp where logincount = 1
第四：不会出现在WHERE子句中字段不该创建索引
```


### 2.5、执行计划

主要用于分析sql语句的执行情况（并不执行sql语句）得到sql语句是否使用了索引，使用了哪些索引。 

``` sql
语法：explain  sql语句\G   或 desc sql语句\G 
```

添加索引进行查看
删除索引时，在看执行计划



### 2.6、索引的数据结构

查看索引的类型：

``` sql
show keys from 表名;
```


### 2.6.1、myisam的存储引擎索引结构：

索引的节点中存储的是数据的物理地址（磁道和扇区）
在查找数据时，查找到索引后，根据索引节点中记录的物理地址，查找到具体的数据内容。


### 2.6.2、innodb的存储引擎的索引结构

innodb的主键索引文件上 直接存放该行数据,称为聚簇索引,非主索引指向对主键的引用（非主键索引的节点存储是主键的id）

比如要通过name创建的索引，查询name=’采臣’的,先根据name建立的索引，找出该条记录的主键id，再根据主键的id通过主键索引找出该条记录。

innodb的主索引文件上 直接存放该行数据,称为聚簇索引,非主索引指向对主键的引用
myisam中, 主索引和非主索引,都指向物理行(磁盘位置).
注意: innodb来说, 
1: 主键索引 既存储索引值,又在叶子中存储行的数据
2: 如果没有主键, 则会Unique key做主键 
3: 如果没有unique,则系统生成一个内部的rowid做主键.
4: 像innodb中,主键的索引结构中,既存储了主键值,又存储了行数据,这种结构称为”聚簇索引”
聚簇索引 
优势: 根据主键查询条目比较少时,不用回行(数据就在主键节点下)
劣势: 如果碰到不规则数据插入时,造成频繁的页分裂（索引的节点移动）.

### 2.6.3 、索引覆盖

- 索引覆盖是指：如果查询的列恰好是索引的一部分，那么查询只需要在索引区上进行，不需要到数据区再找数据，这种查询速度非常快，称为“索引覆盖”

- 索引覆盖就是，我要在书里 查找一个内容，由于目录写的很详细，我在目录中就获取到了，不需要再翻到该页查看。
``` sql
典型情况如下：
学生表：
共30个字段。
select id, name, height,gender from student where name=’XXX’;
建立索引：
alter table student add index (name);
alter table student add index (name, id, height, gender, class_id);
select name, id, height, gender, class_id from student
负面影响，增加了索引的尺寸。
保证该索引的使用率尽可能高，索引覆盖才有意义。
```

#### 2.6.8、索引的使用原则
- 1、列独立

``` sql
只有参与条件表达式的字段独立在关系运算符的一侧，该字段才可能使用到索引。
“独立的列”是指索引列不能是表达式的一部分，也不能是函数的参数。
```

- 2、like查询


``` sql
在使用like(模糊匹配)的时候，在左边没有通配符的情况下，才可以使用索引。
在mysql里，以%开头的like查询，用不到索引。

如果select的字段正好就是索引，那么会用到索引即索引覆盖。

如果该表改为innodb引擎，因为非主键索引中存储的是id,select的字段是id因此用到了索引覆盖。
比如如下把表改成了 innodb的引擎，对name建立了索引，如下查询，就用到了索引覆盖。
如果是innodb的表，可以如上使用。
```

- 3、OR运算都具有索引

``` sql
如果出现OR(或者)运算，要求所有参与运算的字段都存在索引，才会使用到索引。
如下：name有索引，classid没有索引

如下：id有索引，name有索引
```

- 4、复合索引使用

``` sql

最左原则：
对于创建的多列(复合)索引，只要查询条件使用了最左边的列，索引一般就会被使用。 


注意：在多列索引里面，如果有多个查询条件，要想查询效率比较高，比如如下建立的索引，
index(a,b,c,d)  要保证最左边的列用到索引。
```

- 5、mysql 智能选择

``` sql
如果mysql认为，全表扫描不会慢于使用索引，则mysql会选择放弃索引，直接使用全表扫描。一般当取出的数据量超过表中数据的20%，优化器就不会使用索引，而是全表扫描。
```

- 6、优化group by语句。 

``` sql
默认情况下， mysql对所有的group by col1,col2进行排序。这与在查询中指定order by col1,col2类型，如果查询中包括group by 但用户想要避免排序结果的消耗，则可以使用order by null禁止排序。 
输出班级的id,classid

根据classid分组，自动根据classid进行 了排序，
select classid,sum(age) from user group by classid;

如果不想根据classid排序，则可以在后面使用order by nulll
select classid,sum(age) from user group by classid order by null;
```

### 2.7、mysql中锁机制

#### 2.7.1、应用场合：

比如有如下操作：

``` sql
（1）从数据库中取出id的值(比如id=100) 
（2）把这个值-1(id=100-1)
（3）再把该值存回到数据库(id=99)
假如id=1
有两个进程（用户）同时操作，
使用锁机制来完成，
同时操作时，只有一个进程获得锁，其他进程就等待，
进程1
添加锁
id =100
id=100-1
id=99
释放锁
进程2
wating等待    
wating等待
wating等待
id =100
id=100-1
id=99
```
### 2.7.2、mysql里面的锁的几种形式

- 锁机制： 
当客户端操作表（记录）时，为了保证操作的隔离性（多个客户端操作不能相互影响），通过加锁来处理。
- 操作方面： 
  - 读锁：读操作时增加的锁，也叫共享锁，S-lock。特征是所有人都只可以读，只有释放锁之后才可以写。
  - 写锁：写操作时增加的锁，也叫独占锁或排他锁，X-lock。特征，只有锁表的客户可以操作（读写）这个表，其他客户读都不能读。
办公室开会锁上门。
- 锁定粒度（范围） 
  - 表级锁：开销小，加锁快，发生锁冲突的概率最高，并发度最低。
  myisam引擎的表支持表锁，
  - 行级锁：开销大，加锁慢，发生锁冲突的概率最低，并发度也最高。
  innodb引擎的表支持行锁与表锁。

### 2.7.3、表锁的演示，
``` sql
建立测试表，并添加测试数据：
create table user(
    id int primary key auto_increment,
    name varchar(32) not null default '',
    age tinyint unsigned not null default 0,
    email varchar(32) not null default '',
    classid int not null default 1
)engine myisam charset utf8;
insert into user values(null,'xiaogang',12,'gang@sohu.com',4),
(null,'xiaohong',13,'hong@sohu.com',2),
(null,'xiaolong',31,'long@sohu.com',2),
(null,'xiaofeng',22,'feng@sohu.com',3),
(null,'xiaogui',42,'gui@sohu.com',3);

添加锁的语法： lock table table_name1 read|write,table_name2 read|write
释放锁的语法：unlock tables
```

- （1）添加读锁

``` sql
语法：lock  table_name   read|write;


另外一个用户登录后，不能执行修改操作，可以执行查询操作。


注意：添加读锁后，自己和其他的进程（用户）只能对该表查询操作，自己也不能执行修改操作。


注意：添加表的锁定后，针对锁表的用户，只能操作锁定的表，不能操作没有锁定的表。


执行释放锁，

释放锁之后，另外的一个进程，可以执行修改的操作了。
```

- （2）添加写锁，

```
只有锁表的客户可以操作（读写）这个表，其他客户读都不能读。

查看另外的一个用户，是否可以操作该表，其他的用户，读都不能读，
```

#### 2.7.4、行锁的演示
innodb存储引擎是通过给索引上的索引项加锁来实现的，这就意味着：只有通过索引条件(主键)检索数据，innodb才会使用行级锁，否则，innodb使用表锁。
 
``` sql
语法： 
begin;
执行语句； 
commit; 
当前用户添加行锁，
另外的一个用户登录，进行操作。
```

#### 2.7.5、通过php代码来实现锁机制。

在apache里面有一个bin目录 下面有一个ab.exe工具，该工具可以模拟多个并发测试。

``` shell
语法：
ab.exe  –n 总的请求数  -c 并发数  url地址；
```

```
使用mysql里面锁机制缺点：就是阻塞，假如有一张goods表，goods表里面有一个库存的字段，当前下订单时，如果锁定了goods表，还能执行查询goods表吗？
会阻塞拖慢整个网站的速度，一但锁定goods表（添加写锁，要更改库存），则其他进程就无法查询goods表。
可以使用文件锁，
```

``` php
文件锁分为两种方式：

【一】.阻塞模式：(如果其他进程已经加锁文件,当前进程会一直等其他进程解锁文件后继续执行)

<?php
//连接数据库
$con=mysqli_connect("192.168.2.186","root","root","test");
//查询商品数量是否大于0,大于0才能下单,并减少库存
$fp = fopen("lock.txt", "r");
//加锁
if(flock($fp,LOCK_EX))
{
	$res=mysqli_fetch_assoc(mysqli_query($con,'SELECT total FROM shop WHERE id=1 LIMIT 1'));
	if($res['total']>0){mysqli_query($con,'UPDATE shop SET total=total-1  WHERE id=1');}
	//执行完成解锁
	flock($fp,LOCK_UN);
}
//关闭文件
fclose($fp);
unset($res);
mysqli_close($con);
?>
这种情况若是其他进程已经加锁文件，那么所有进程都会等他执行完并解锁文件后才会执行

【二】.非阻塞模式：(如果其他进程已经加锁文件,当前进程不会等其他进程解锁文件,而是走else)
<?php
//连接数据库
$con=mysqli_connect("192.168.2.186","root","root","test");

//查询商品数量是否大于0,大于0才能下单,并减少库存

$fp = fopen("lock.txt", "r");
//加锁
if(flock($fp,LOCK_EX | LOCK_NB))
{
	$res=mysqli_fetch_assoc(mysqli_query($con,'SELECT total FROM shop WHERE id=1 LIMIT 1'));
	if($res['total']>0){mysqli_query($con,'UPDATE shop SET total=total-1  WHERE id=1');}
	//执行完成解锁
	flock($fp,LOCK_UN);
}else{
　　echo "locked file failed\n";
}
unset($res);
mysqli_close($con);
?>
这种情况就会直接走else返回提示信息
```

## 3 Mysql架构优化

### 3.1 查询缓存

#### 3.1.1、具体使用

什么是查询缓存？
mysql服务器提供的，用于缓存select语句结果的一种内部内存缓存系统。
如果开启了查询缓存，将所有的查询结果，都缓存起来，使用同样的select语句，再次查询时，直接返回缓存的结果即可
查看缓存设置情况，并给缓存空间设置大小：

``` sql
> show variables  like ‘query_cache%’;   //查看缓存使用情况

query_cache_size:缓存空间大小
query_cache_type:是否有开启缓存
如何开启查询缓存，并设置缓存空间大小？
在my.ini中对上边的两个变量进行配置：



query_cache_size=134217728
query_cache_type=1

配置完成，之后需要重启mysql，

查看缓存开启成功：show  variables like ‘query_cache%’;

sql语句第一次执行没有缓存，之后就有缓存了：
```

#### 3.1.2、无缓存

- （1）缓存失效

``` sql
数据表的数据(数据有修改)有变化 或者 数据表结构(字段的增、减)有变化，则会清空全部的缓存数据，即缓存失效。
update emp set job=’123456’ where empno=123456;


执行了一个update语句，导致之前存在缓存(empno=1234567)被清空了
```

- （2）不使用缓存

``` sql
sql语句有变化表达式，则不会生成/使用缓存。
例如有 时间信息、随机数等
select ename,job,now() from emp where empno=123456;

在sql语句中有“时间”变化的表达式，则不使用缓存
select * from emp order by rand() limit 4;

sql语句中有“随机数”的表达式，不给使用缓存
```

（3）生成多个缓存

``` sql
生成缓存的sql语句对“空格”、“大小写”比较敏感
相同结果的sql语句，由于空格、大小写问题就会分别生成多个缓存。

注意：相同结果的sql语句，由于大小写问题会分别生成缓存：
```

（4）禁用缓存

``` sql
sql_no_cache 不进行缓存
select  sql_no_cache  * from emp where empno=123456;
意思是当前查询结果不使用查询缓存；
```

#### 3.1.3、查看缓存空间使用情况

``` sql
 show status like ‘Qcache%’;  //查看缓存使用情况
```

>生产环境不使用mysql自带的缓存，使用redis缓存代替

### 3.2 分区技术

#### 3.2.1、分区介绍

基本概念，把一个表，从逻辑上分成多个区域，便于存储数据。
采用分区的前提，数据量非常大。
如果数据表的记录非常多，比如达到上亿条，数据表的活性就大大降低，数据表的运行速度就比较慢、效率低下，影响mysql数据库的整体性能，就可以采用分区解决，分区是mysql本身就支持的技术。

``` sql
查看当前mysql软件是否支持分区；
show variables like '%partition%';

以上的结构，在创建（修改）表时，可以指定表，可以被分成几个区域。
利用表选项：partition 完成。 
create table  table_name(
	字段信息，
    索引，
)engine myisam charser utf8
partition by 分区算法（分区字段）（
	分区选项
）；
分区算法：
条件分区：list (列表) range(范围)  取模轮询（hash,key）
```

#### 3.2.2、分区算法
- （1）list分区


``` sql
list :条件值为一个数据列表。
通过预定义的列表的值来对数据进行分割
例子：假如你创建一个如下的一个表，该表保存有全国20家分公司的职员记录，这20家分公司的编号从1到20.而这20家分公司分布在全国4个区域，如下表所示：
职员表：emp
id  name   store_id(分公司的id)
12   小宝     1
14   二宝     6
北部    1,4,5,6,17,18
南部    2,7,9,10,11,13
东部    3,12,19,20
西部    8,14,15,16

insert into emp values(12,’xiaobao’,14)
insert into emp values(15,’二bao’,17)


create table p_list(
    id int,
    name varchar(32),
    store_id int
)engine myisam charset utf8
partition by list (store_id)(
    partition p_north values in (1,4,5,6,17,18),
    partition p_east values in(2,7,9,10,11,13),
    partition p_south values in(3,12,19,20),
    partition p_west values in(8,14,15,16)
);

创建分区表后查看文件，

添加几条数据，测试是否用到了分区：

explain partitions select * from p_list where store_id=20\G
注意：在使用分区时，where后面的字段必须是分区字段，才能使用到分区。

没有分区条件，则会到所有的分区里面去查找，即便如此，查询效率也要比单表查询高。
```

- （2）Range（范围）

``` sql
这种模式允许将数据划分不同范围。例如可以将一个表通过月份划分成若干个分区
create table p_range(
    id int,
    name varchar(32),
    birthday date
)engine myisam charset utf8
partition by range (month(birthday))(
    partition p_1 values less than (4),
    partition p_2 values less than(7),
    partition p_3 values less than(10),
    partition p_4 values less than MAXVALUE
);
less than   小于;
MAXVALUE 可能的最大值 

insert into p_range values(1,’xiaobao’,’2016-09-09’);
insert into p_range values(1,’xiaobao’,’2016-11-09’);
插入的数据如下；

分区的效果如下；
```

- （3）Hash（哈希）

``` sql
这种模式允许通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区。例如可以建立一个对表主键进行分区的表。
create table  p_hash(
	id  int,
	name varchar(20),
	birthday date
)engine myisam charset utf8 
partition by hash(month(birthday)) partitions 5;
分区效果如下；
```


- （4）Key（键值）

``` sql
上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。 
create table p_key(
    id int,
    name varchar(32),
    birthday date
)engine myisam charset utf8
partition by key (id) partitions 5;
```

#### 3.2.3、 分区管理

具体就是对已经存在的分区进行增加、减少操作。
- （1）删除分区

``` sql
删除分区： 
① 在key/hash领域不会造成数据丢失(删除分区后数据会重新整合到剩余的分区去)
② 在range/list领域会造成数据丢失

求余方式(key/hash):
>alter table 表名 coalesce partition 数量;
范围方式(range/list):
>alter table 表名 drop partition 分区名称;

1）删除hash类型分区
删除分区之前，数据如下


执行删除分区的操作：alter table p_hash  coalesce partition 4

上图，把5个分表中的4个都删除，只剩下一个
剩余一个分表效果：

并且，数据没有减少：
剩余唯一一个分区的时候，就禁止删除了，但是可以drop掉整个数据表，如下图：
alter table p_hash  coalesce  partition 1;

2）删除list类型分表(数据有对应丢失)
alter table p_list drop partition   p_north;
```

- （2）增加分区

``` sql
求余方式： key/hash
> alter table 表名  add  partition partitions  数量;
范围方式： range/list
>  alter table 表名 add partition(
           partition 名称 values  less than (常量)
           或
           partition 名称 values  in (n,n,n)
       );

1) 给p_hash 增加hash分表

alter table p_hash  add partition partitions 6;


增加后，一共有7个分表体现：

分表增加好后，又把数据平均地分配给各个分表存储。

特别注意；
create table p_range2(
    id int primary key auto_increment,
    name varchar(32),
    birthday date
)engine myisam charset utf8
partition by range (month(birthday))(
    partition p_1 values less than (4),
    partition p_2 values less than(7),
    partition p_3 values less than(10),
    partition p_4 values less than MAXVALUE
);


注意：创建分区的字段必须是主键或唯一索引的一部分
primary key(id,birthday)
不等价于如下量行代码；
primary key(id)
primary key(birthday)
create table p_range2(
	id  int auto_increment,
	name varchar(32),
	birthday date,
	primary key(id,birthday)
)engine myisam charset utf8
partition by range (month(birthday))(
	partition p_1 values less than (4),
	partition p_2 values less than (7),
	partition p_3 values less than(10),
	partition p_4 values less than MAXVALUE
);



create table p_range3(
	id  int auto_increment,
	name varchar(32),
	birthday date,
	unique key(id,birthday)
)engine myisam charset utf8
partition by range (month(birthday))(
	partition p_1 values less than (3),
	partition p_2 values less than (6),
	partition p_3 values less than(9),
	partition p_4 values less than MAXVALUE
);
```

### 3.3、分表技术
#### 3.3.1. 分表设计

物理方式分表设计
自己手动创建多个数据表出来
php程序需要考虑分表算法：数据往哪个表写，从哪个表读  

``` php
QQ的登录表。假设QQ的用户有10亿，如果只有一张表，每个用户登录的时候数据库都要从这10亿中查找，会很慢很慢。如果将这一张表分成100份，每张表有1000万条，就小了很多，比如qq0,qq1,qq1...qq99表。
用户登录的时候，可以将用户的id%100，那么会得到0-99的数，查询表的时候，将表名qq跟取模的数连接起来，就构建了表名。比如123456789用户，取模的89，那么就到qq89表查询，查询的时间将会大大缩短。
注册时，如何存储到多张表里面？
登录时，如何知道查询那张表？
注册时
$user_id = $redis->incr(‘user_id’);
表单提交过来的内容；
$username = ‘大宝’;
假如我们要分四张表来存储；
$user_id%4 = 获取余数   
假如$user_id=8了  余数是0，那我们就存储到user_0表里面了，
user_0表里面的字段  id   $user_id    ‘大宝’
$redis->set($username_register_name,$user_id)
登录时，我们用名称来登录；
$username = ‘大宝’,如何知道该名称在那张表里面呢？  

$username->user_id->通过user_id算出存储的表；
```

#### 3.3.2. 垂直分表（比较常用）

水平分表：是把一个表的全部记录信息分别存储到不同的分表之中。
垂直分表：是把一个表的全部字段分别存储到不同的表里边。

有的时候，一个数据表设计好了，里边有许多字段，但是这些字段有的是经常使用的，有的是不常用的。在进行正常数据表操作的时候，不常用的字段也会占据一定的资源，对整体操作的性能造成一定的干扰、影响。
为了减少资源的开销、提升运行效率，就可以把不常用的字段给创建到一个专门的辅表中去。
同一个业务表的不同字段分别存储到不同数据表的过程就是“垂直分表”。

``` sql
例如：  
会员数据表有如下字段：
会员表： user_id  登录名  密码  邮箱  手机号码  身高  体重  性别  家庭地址  身份证号码

以上表，红色是常用的，蓝色的是不常用的
为了使得常用字段运行速度更快、效率更高，把常用字段给调出来,因此数据表做以下垂直分表设计：
会员表(主)user字段：user_id  登录名  密码  邮箱  手机号码
会员表(辅)user_fu字段：user_id  身高  体重  性别  家庭地址  身份证号码

以上把会员表根据字段是否常用给分为两个表的过程就是垂直分表。
存储文章
经常查询的数据     title（标题）   author（作者）    
```

### 3.4、数据碎片与维护
在长期的数据更改过程中，索引文件和数据文件，都将产生空洞，形成碎片，我们可以通过一个操作（不产生对数据实质影响的操作）来修改表，

``` sql
建表语句：
create table t1(id int)engine myisam;
insert into t1 values(1),(2),(3)
insert into t1 select * from t1;

表的原始大小：

删除了一部分数据，应该表的容量会减少一部分，但是没有减掉，


开始整理：
optimize    table  表名;

整理后的结果，容量减少了一部分。

比如：表的引擎为innodb,可以alter table xxx engine innodb 

optimize table 表名，也可以修复。
注意：修复表的数据及索引碎片，就会把所有的数据文件重新整理一遍，使之对齐，这个过程，如果表的行数比较大，也是比较耗费资源的操作，所以，不能频繁的修复。
如果表的update，delete操作很频繁，可以按周月来修复。
```


### 3.5、范式讲解

- 第一范式：

```
我们的表要满足两个条件：
（1）表的属性（列）要具有原子性（不可分割）
（2）表不能有重复的列，
只要是关系型数据库，就天然的满足第一范式。
关系型数据库:有行和列的概念，即为二维表格，常见的有mysql , sql server, oracle , informix ， db2, postgresql
非关系型数据库：面向对象和集合，没有行和列的概念。
```

- 第二范式

```
表要满足：不能存在完全相同的两条记录，通常是通过设置一个主键来实现，主键一般是非业务逻辑主键。
```

- 第三范式

```
表中不能存在冗余数据，表中列的值，如果可以通过推导出来，则就不应该设置该列。
```

- 反三范式（逆范式）
  
```
有的时候基于性能考虑，需要有意违反 三范式，适度的冗余，以达到提高查询效率的目的。
```



### 3.6、视图

#### 3.6.1、视图的定义 

视图的定义：
视图是由查询结果形成的一张虚拟表，是表通过某种运算得到的一个投影。


```sql
创建视图的语法： 
create   view   view_name   as   select 语句
说明：
（1）视图名跟表名是一个级别的名字，隶属于数据库；
（2）该语句的含义可以理解为：就是将该select命名为该名字（视图名）；
（3）视图也可以设定自己的字段名，而不是select语句本身的字段名——通常不设置。
（4）视图的使用，几乎跟表一样！
```

#### 3.6.2、视图的作用

准备测试数据；goods表和category表；

- （1）可以简化查询。

``` sql
案例1：查询平均价格前3高的栏目。
传统的sql语句写法
select cat_id,avg(shop_price)  pj   from goods  group by cat_id order by pj  desc limit 3;

创建一个视图
create   view   ecs_goods_v1  as select cat_id,avg(shop_price) pj from ecs_goods group by cat_id;


创建好了视图，再次查询平均价格前3高的栏目时，我们就可以直接查询视图
select * from ecs_goods_v1  order by pj desc limit 3;

案例2:查询出商品表，以及所在的栏目名称；
传统的写法
select goods_id,goods_name,b.cat_name from ecs_goods a left join ecs_category b on a.cat_id=b.cat_id;

创建一个视图
create view  ecs_goods_v2  as  select goods_id,goods_name,b.cat_name from ecs_goods a left join ecs_category b on a.cat_id=b.cat_id;
查询视图；
select * from ecs_goods_v2;
```

- （2）可以进行权限控制，

把表的权限封闭，但是开放相应的视图权限，视图里只开放部分数据，比如某张表，用户表为例，2个网站搞合作，可以查询对方网站的用户，需要向对方开放用户表的权限，但是呢，又不想开放用户表中的密码字段。
再比如一个goods表，两个网站搞合作，可以相互查询对方的商品表，比如进货价格字段不能让对方查看。

``` sql
案例：
（1）创建一个goods表，添加几条数据
给测试的goods表添加一个in_price(进货价格)字段；


（2）创建一个视图
create view  goods_v1 as select id,goods_name,shop_price from goods;

（3）授权一个账号
grant  权限  on 数据库名称.视图名或表名  to  ‘用户名称’@’%’  identified by ‘密码’’

grant  select  on php.goods_v1  to  ‘xiaolei’@’%’  identified by ‘1234’’

以上语句，表示创建了一个 dahei的用户，密码是123456，权限时再php69库下面的goods_v1视图具有查询的权限；
```

#### 3.6.3、查询视图 

``` sql
语法：select * from 视图名 [where 条件] 
视图和表一样，可以添加where 条件
```

#### 3.6.4、修改视图 

``` sql
alter   view   view_name  as   select   XXXX 
``` 

#### 3.6.5、删除视图 

``` sql
drop  view  视图名称
```

#### 3.6.6、查看视图结构 
和表一样的，语法，desc 视图名称

#### 3.6.7、查看所有视图 

``` sql
和表一样，语法：show   tables;
注意：没有show views语句；
```

#### 3.6.8、视图与表的关系 
视图是表的查询结果，自然表的数据改变了，影响视图的结果。

（1）视图的数据与表的数据一一对应时，可以修改。

（2）视图增删该也会影响表，但是视图并不是总是能增删该的。
``` sql
create view lmj as select cat_id,max(shop_price) as lmj from goods group by cat_id; 
mysql> update lmj set lmj=1000 where cat_id=4; 
ERROR 1288 (HY000): The target table lmj of the UPDATE is not updatable 
```

（3）对于视图insert还应注意，视图必须包含表中没有默认值的列。



注意：向视图里面插入数据时，视图必须包含表中没有默认值的列，才能插入成功，否则就插入失败。
注意：在实际的开发中，不要对视图进行增删改。


### 3.7、SQL 编程

 
#### 3.7.1、变量声明 
- （1）会话变量

``` sql
定义形式：
set   @变量名  =  值；
说明：
1，跟php类似，第一次给其赋值，就算定义了
2，它可以在编程环境和非编程环境中使用！
3，使用的任何场合也都带该“@”符号。 
```

-（2）普通变量

``` sql
定义形式：
declare  变量名   类型   【default  默认值】；
说明：
1、它必须先声明（即定义），此时也可以赋值；
2、赋值跟会话变量一样：  set  变量名 = 值；
3、它只能在编程环境中使用！！！
说明：什么是编程环境？
编程环境是指  （1）存储过程 （2）函数 （3）触发器。
```

-（3）变量赋值形式 

``` sql
语法1：
set 变量名 = 表达式;#此语法中的变量必须先使用declare声明，在编程环境中使用
语法2：
set @变量名=表达式；
#此方式可以无需declare语法声明，而是直接赋值，类似php定义变量并赋值。

语法3：
select @变量名:=表达式；
#此语句会给该变量赋值，同时还会作为一个select语句输出‘结果集’。 

语法4：
select 表达式 into @变量名;#此语句虽然看起来是select语句，但其实并不输出‘结果集’，而是给变量赋值。
```

#### 3.7.2、运算符 

``` sql
（1）算术运算符
+、-、*、/、% 
注意：mysql没有++和—运算符 
（2）关系运算符
>、>=、<、<=、=（等于）、<>（不等于） !=（不等于）
（3）逻辑运算符
and（与）、or（或）、not（非）
```

#### 3.7.3、语句块包含符 

``` sql
所谓语句块包含符，在js或php中，以及绝大部分的其他语言中，都是大括号：{} 
它用在很多场合：if， switch,  for,  function 
而mysql编程中的语句块包含符是begin   end结构。
```


#### 3.7.4、if判断 
MySQL支持两种判断，第一个是if判断，第二个 case判断 

``` sql
if语法 
单分支
if 条件 then 
	//代码
end if; 
双分支
if 条件 then 
	代码1 
else 
	代码2 
end if; 
多分支
if 条件 then 
	代码1 
elseif 条件 then 
	代码2 
else 
	代码3
end if; 
通过存储过程 来体验 if语句的结构 ，
创建存储过程语法：
create procedure 存储过程名(参数1,参数2,…) 
begin 
	//代码;
end 
案例：接收4个数字，
如果输入1则输出春天，2=》夏天 3=》秋天  4 =》冬天 其他数字=》出错
注意：通常情况下，“；“表示SQL语句结束，同时向服务器提交并执行。但是存储过程中有很多SQL语句，每一句都要以分号隔开，这时候我们就需要使用其他符号来代替向服务器提交的命令。通过delimiter命令更改语句结束符。 

delimiter $
create procedure  p1(num int)
begin
	if num=1 then
		select '春天' as '季节';
	elseif num=2 then
		select '夏天' as '季节';
	elseif num=3 then
		select '秋天' as '季节';
	elseif num=4 then
		select '冬天' as  '季节';
	else
		select '无法无天' as '季节';
	end if;
end$
delimiter ;
```

#### 3.7.5、case判断 

``` sql
语法：
case  变量
when 值  then     语句; 
when 值  then     语句; 
else  语句; 
end case ; 

案例：接收4个数字，
如果输入1则输出春天，2=》夏天 3=》秋天  4 =》冬天 其他数字=》出错
create procedure  p2(num int)
begin
	case num
	when 1 then  select '春天' as '季节';
	when 2 then  select '夏天' as '季节';
	when 3 then  select '秋天' as '季节';
	when 4 then  select '冬天' as '季节';
	else  select '无法无天' as '季节';
	end case;
end$
```

#### 3.7.6、循环 
MySQL支持的循环有loop、while、repeat循环

``` sql
（1）loop循环 
语法：
标签名:loop 
	leave 标签名    --退出循环
end loop; 
#案例：创建一个存储过程，完成计算1到n的和。
create procedure  p3(n int)
begin
	declare i int default 1;
	declare s int default 0;
	aa:loop
		set s=s+i;
		set i=i+1;
		if i>n then
			leave aa;
		end if;
	end loop;
	select s;
end$

（2）while循环
 [标签:]while 条件 do 
	//代码
end while; 
#案例：创建一个存储过程，完成计算1到n的和。
create procedure  p4(n int)
begin
	declare i int default 1;
	declare s int default 0;
	while  i<=n  do 
		set s=s+i;
		set i=i+1;
	end while;
	select s;
end$
```

### 3.8 、存储过程

#### 3.8.1、概念 
存储过程(procedure) 
概念类似于函数，就是把一段代码封装起来，当要执行这一段代码的时候，可以通过调用该存储过程来实现。在封装的语句体里面，可以同if/else ,case,while等控制结构。
可以进行sql编程。
``` sql
查看现有的存储过程。
show procedure status 
```

#### 3.8.2、存储过程的优点

存储过程（Stored Procedure）是在大型数据库系统中，一组为了完成特定功能的SQL 语句集，存储在数据库中，经过第一次编译后再次调用不需要再次编译，用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。存储过程是数据库中的一个重要对象，任何一个设计良好的数据库应用程序都应该用到存储过程。
（1）存储过程只在创造时进行编译，以后每次执行存储过程都不需再重新编译，而一般SQL语句每执行一次就编译一次,所以使用存储过程可提高数据库执行速度。
（2）当对数据库进行复杂操作时(如对多个表进行Update,Insert,Query,Delete时)，可将此复杂操作用存储过程封装起来与数据库提供的事务处理结合一起使用。
（3）存储过程可以重复使用,可减少数据库开发人员的工作量
（4）安全性高,可设定只有某些用户才具有对指定存储过程的使用权

#### 3.8.3、创建存储过程 

``` sql
语法：
create procedure 存储过程名(参数1,参数2,…) 
begin 
	//代码
end 
参数的类型：
in（输入参数）： 表示该形参只能接受实参的数据——这是默认值，不写就是in；
out（输出参数）：表示该形参其实是用于将内部的数据“传出”到外部给实参；
inout（输入输出参数）：具有上述2个功能。
案例1：查询一个表里面某些语句 
create procedure p6(goods_id int)
begin
	select * from goods;
end$
call p6()

案例2：第二个存储过程体会参数，使用参数
比如我们取出某个id的数据
create procedure p8(price float)
begin
select * from goods where shop_price>price;
end$


说明：
（1）存储过程中，可有各种编程元素：变量，流程控制，函数调用；
（2）还可以有：增删改查等各种mysql语句；
（3）其中select（或show，或desc）会作为存储过程执行后的“结果集”返回；
（4）形参可以设定数据的“进出方向”：
（5）存储过程是属于数据库，在哪个数据库里面定义的，就在哪个数据库里面调用。
```

#### 3.8.4、调用存储过程 

``` sql
语法：
call 存储过程名称(参数)
在php里面如何调用，
mysql_query(‘call p7(5)’);
```

#### 3.8.5、创建复杂的存储过程 

``` sql
案例1，体会“控制结构”;
定义一个存储过程，有两个参数，第一个参数是价格，第二个参数是一个字符串，
如果该字符串等于’h’ 则就取出大于该价格（第一个参数）商品数据，其他则输出小于该价格的商品；
create procedure p8(price float,str char(1))
begin
	if str='h' then
	select  id,goods_name,shop_price from goods  where shop_price>=price;
	else
	select  id,goods_name,shop_price from goods  where shop_price<price;
	end if;
end$


案例2：带有输出参数的存储过程
create procedure p9(in num int,out res int)
begin
	set  res = num*num;
end$
注意：在调用具有输出参数的存储过程时，要使用一个变量来接收。

call p9(8,@res);select @res;

案例3：带有输入输出参数的存储过程
create procedure p10(inout num int)
begin
	set   num=num*num;
end$
注意：在调用时先创建一个变量，调用存储过程时，使用该变量接收。
set @a = 10;call p10(@a);select @a$
```

#### 3.8.6、删除存储过程


``` sql
语法：drop    procedure    存储过程的名称
```

### 3.9、存储函数
存储函数就是函数

#### 3.9.1、自定义函数 
-（1）定义语法 

``` sql
create function 函数名(参数) returns 返回值类型
begin 
	//代码
end 

说明：
（1）函数内部可以有各种编程语言的元素：变量，流程控制，函数调用；
（2）函数内部可以有增删改等语句！
（3）但：函数内部不可以有select（或show或desc）这种返回结果集的语句！
（2）调用 
跟系统函数调用一样：任何需要数据的位置，都可以调用该函数。
案例1：返回两个数的和
create function sumhe(num1 int,num2 int) returns int
begin
	return num1+num2;
end$

案例2：定义一个函数，返回1到n的和。
create function nhe(n int)  returns int
begin
	declare i int default 1;
	declare s int default 0;
	while i<=n do 
		set s=s+i;
		set i=i+1;
	end while;
	return s;
end$

注意点：创建的函数，是隶属于数据库的，只能在创建函数的数据库中使用。
select sumhe(345,34435)$
```

#### 3.9.2、系统函数 


``` sql
（1）数字类
mysql> select rand();//返回0到1间的随机数
mysql>select * from it_goods  order by rand() limit 2;//随机取出2件商品

mysql>select  floor(3.9)//输出3 
mysql>select  ceil(3.1)//输出4 
mysql>select  round(3.5)//输出4四舍五入
select goods_name,round(shop_price) from goods limit 10;

（2）大小写转换
mysql> select ucase('I am a boy!') //    --转成大写
mysql> select lcase('I am a boy!') //    --转成小写

3）截取字符串
mysql> select left('abcde',3)//			--从左边截取
mysql> select right('abcde',3) //        --从右边截取
mysql> select substring('abcde',2,3)//   --从第二个位置开始，截取3个，位置从1开始
select left(goods_name,1),round(shop_price) from goods limit 10

mysql> select concat(10,':锄禾日当午')//   --字符串相连


select concat(left(goods_name,1),'...'),round(shop_price) from goods limit 10;
mysql> select coalesce(null,123); 
coalesce(str1,str2)：如果第str1为null，就显示str2 
select goods_name,coalesce(goods_thumb,'无图')from goods limit 10;


mysql> select length('锄禾日当午') //   输出10   显示字节的个数
mysql> select char_length('锄禾日当午') // 输出5   显示字符的个数
mysql> select length(trim('  abc  ')) //  trim用来去字符串两边空格
mysql> select replace('abc','bc','pache')//   将bc替换成pache 

（4）时间类
mysql> select unix_timestamp()//      --时间戳

mysql> select from_unixtime(unix_timestamp()) //  --将时间戳转成日期格式 
from_unixtime(unix_timestamp(),'%Y-%m-%d-%h-%i-%d')

mysql>select  curdate();返回今天的时间日期： 
mysql> select now()//					--取出当前时间




案例1：比如一个电影网站，求出今天添加的电影；在添加电影时，有一个添加的时间戳。
select  id,title  from dede_archives where   (from_unixtime(添加时间,'%Y-%m-%d')=curdate());

案例2：比如一个电影网站，求出昨天添加的电影；在添加电影时，有一个添加的时间戳。
扩展，如何取出昨天或者指定某个时间的电影：
date_sub
基本用法：
date_sub(时间日期时间,interval 数字  时间单位) 
说明：
（1）时间单位：可以是year month day hour minute second 
（2）数字：可以是正数和负数。
比如：取出昨天的日期：
mysql> select date_sub(curdate(),interval 1 day);
比如：取出上一个月日期：
mysql> select date_sub(curdate(),interval 1 month);
如下案例是：求出前第2天添加的电影数据
select  id,title,from_unixtime(add_time,'%Y-%m-%d') from movie where   (from_unixtime(add_time,'%Y-%m-%d'))=date_sub(curdate(),interval 2 day);
```

### 3.10、触发器


#### 3.10.1、简介 
（1）触发器是一个特殊的存储过程，它是MySQL在insert、update、delete的时候自动执行的代码块。
（2）触发器必须定义在特定的表上。
（3）自动执行，不能直接调用，
作用：监视某种情况并触发某种操作。 
触发器的思路：
监视it_order表，如果it_order表里面有增删改的操作，则自动触发it_goods里面里面增删该的操作。
比如新添加一个订单，则it_goods表，就自动减少对应商品的库存。
比如取消一个订单，则it_goods表，就自动增加对应商品的库存减少的库存。


#### 3.10.2、触发器四要素


监视地点：就是设置监视的表
监视事件；设置监视的那张表的insert ,update,delete操作；
触发时间：设置触发时间，监视表的操作之前，还是之后；
触发事件：满足条件了，设置的触发的操作；
准备测试数据；

#### 3.10.3、创建触发器 

``` sql
创建触发器的语法：
create  trigger  trigger_name 
after/before  insert /update/delete  on   表名
for each row 
begin 
sql语句：（触发的语句一句或多句）
end 
案例1：第一个触发器，购买一头猪，减少1个库存。 
分析：
监视地点：it_order表
监视事件：it_order表的insert 操作；
触发时间：it_order表的insert 操作之后
触发事件：it_goods表猪的库存减1操作；
create  trigger  t1
after   insert  on  it_order
for each row
begin
update it_goods set goods_number=goods_number-1 where id=1;
end$



注意：以上触发器是有问题的， 无论买谁，都是减少的猪的数量，而且数量是1，
案例2：购买商品，减少对应库存 
create  trigger  t1
after   insert  on it_order
for each row
begin
update it_goods set goods_number=goods_number-new.much where id=new.goods_id;
end$
注意：如果在触发器中引用行的值。
对于insert 而言，新增的行用new来表示，行中的每一列的值，用  new.列名 来表示。


特别注意：


案例3：取消订单时，减掉的库存要添加回来 
分析：
监视地点：it_order表
监视事件：it_order表的delete操作；
触发时间：it_order表的delete操作之后
触发事件：it_goods表减掉库存再加回来；
注意：
对于delete而言，it_order表删除的行用old来表示，行中的每一列的值，用 old.列名 来表示。
create  trigger  t2
after   delete  on  it_order
for each row
begin
update it_goods set goods_number=goods_number+old.much where id=old.goods_id;
end$



案例4：修改订单时，库存也要做对应修改(修改的数据，有商品的数量，类型)
分析：

注意：
对于update而言，修改之前行用old来表示，行中的每一列的值，用 old.列名 来表示。
修改之后，用new来表示，行中的每一列的值，用 new.列名 来表示
思路：如何完成修改订单，触发it_goods表的操作，
（1）取消订单
（2）重新下单
create  trigger  t3
after   update  on  it_order
for each row
begin
update it_goods set goods_number=goods_number+old.much where id=old.goods_id;
update it_goods set goods_number=goods_number-new.much where id=new.goods_id;
end$
```

#### 3.10.4、删除触发器 
``` sql
语法：drop trigger 触发器的名称
```

#### 3.10.5、查看触发器 

``` sql
语法： show triggers 
```

#### 3.10.6、before和after的区别 

after是先完成数据的增删改，再触发，触发器中的语句晚于监视的增删改，无法影响前面的增删该动作。
就类似于先吃饭，再付钱。
before是先完成触发，再增删改，触发的语句先于监视的增删改发生，我们有机会判断修改即将发生的操作。
就类似于先付钱，再吃饭

``` sql
典型案例：对于已下的订单，进行判断，如果订单的数量>5，就认为是恶意订单，强制把所定的商品数量改成5 
分析：
监视的表 :it_order
监视的事件：it_order表的insert操作
触发的时间：it_order表的insert操作之前
触发的事件：如果订单数量大于5，则改成5
create  trigger  t4
before   insert  on  it_order
for each row
begin
	if new.much>5 then
	set new.much=5;
	end if;
end$
```


### 3.11、事务操作

MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
事务用来管理 insert,update,delete 语句
事务特点：
（1）原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
（2）一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。
（3）隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。
（4）持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。


```sql
语法：
BEGIN 开始一个事务
ROLLBACK 事务回滚
COMMIT 事务确认
```


``` php
$pdo  = new PDO('mysql:host=localhost;dbname=php70','xiaolong','123456');
$pdo->exec('set names utf8');
测试数据如下；

$pdo->beginTransaction();//启动事务
$pdo->commit()提交事务
$pdo->rollback()回滚事务
$pdo  = new PDO('mysql:host=localhost;dbname=php69','root','root');
$pdo->exec('set names utf8');
$pdo->beginTransaction();
$res1 = $pdo->exec("insert into user values(null,'name3',12,'email1',1)");
$res2 = $pdo->exec("insert into user values(null,'name4',12,'email2',2))");
if(!$res1 || !$res2 ){
	$pdo->rollback();
}else {
	$pdo->commit();
}
echo 'ok';
```

### 3.12、读写分离（主从复制）

#### 3.12.1、什么是主从复制
至少两台数据库服务器，可以分别设置主服务器和从服务器，对主服务器的任何操作都会同步到从服务器上。

主要作用：
（1）分担压力
（2）备份数据
#### 3.12.2、实现原理
mysql中有一种日志，叫做bin日志（二进制日志），会记录下所有修改过数据库的sql语句。主从复制的原理实际是多台服务器都开启bin日志，然后主服务器会把执行过的sql语句记录到bin日志中，之后从服务器读取该日志，在从服务器再把bin日志中记录的sql语句同样的执行一遍。这样从服务器上的数据就和主服务器相同了。

实现读写分离使用的知识点
（1）主从都要开启bin日志
（2）主服务器需要授权用户
（3）具体的配置过程；

#### 3.12.3、账号(用户)管理

``` sql
（1）添加账号（用户）
语法：grant  权限  on 数据库.数据表  to  ‘用户名’@’ip地址’   identified  by ‘密码’
比如：
grant  all  on   *.*   to ‘xiaohei’@’%’  identified  by ‘1234’
注意：创建的用户信息，是保存在mysql库下面的user表里面的。
select user,host from mysql.user;

案例：
第一步：在window的虚拟主机里面，添加一个账号如下；

注意：mysql里面用户信息是存储到mysql库下面user表中，


第二步：在linu中里面使用window中新建的用户(xiaoqian)登录window中 mysql服务器。
mysql  -h192.168.1.10 –uxiaomei –p123456
（2）删除账号
语法：drop  user  ‘用户名’@’ip地址’;
```

#### 3.12.4、bin-log开启操作

``` sql
（1）开启bin-log日志，
打开mysql的配置文件，
window下面    my.ini
linux下面      my.cnf

log-bin=mysql-bin
server-id=1
注意：修改完成mysql的配置后，要重启mysql服务，切记不需要重启apache,
注意：mysql里面数据表的存储位置，要看配置文件，

开启配置后，产生的二进制日志文件如下；

（2）与log-bin日志相关的函数，
flush logs
执行该命令，就会产生一个新的log-bin日志

产生的文件如下；

reset master;
清空所有的log-bin日志，并产生一个新的log-bin日志

show master status
查看最后（新）的一个log-bin日志 

（3）查看log-bin日志里面的内容 
新建一张表，测试log-bin日志是否记录增删改的sql语句


注意：使用mysql安装目录下面的bin目录下面mysqlbinlog命令，来查看日志内容。


语法：mysqlbinlog    --no-defaults     二进制日志的名称（全路径）
MySQL中二进制文件所在目录
/var/lib/mysql


注意：end_log_pos的理解，用于记录上一个 sql语句的结束，下一个sql语句 的开始位置



通过show master status命令，能查看到二进制文件里面最后一个pos位置。
```


#### 3.12.5、具体的配置步骤


``` sql
实验规划，需要两台主机
第一台主机：ip地址  192.168.1.69      配置为master服务器
第二台主机：ip地址  192.168.1.70      配置为slave 服务器
1、配置主服务器
（1）开启二进制日志。
（2）要设置一个server-id（作为一个服务器的编号，是唯一）  该值不能和从服务器相同。


注意：在my.cnf配置文件里面，配置的区域在[mysqld]与[mysql]之间配置；
注意：配置完成后，要重启mysql服务
（3）授权一个账号，让从服务器通过该账号读取log-bin日志里面的内容
grant  replication slave  on *.*  to 'xiongda'@'%' identified by '123456'

赋予从库权限账号，允许用户在主库上读取日志，也就是Slave机器读取File权限，
grant FILE on *.* to 'xiongda'@'%' identified by '123456';




（4）记录主服务器里面的最新的二进制的名称和pos位置

注意：此时，就禁止对主服务器执行增删改的操作，一直到从服务器配置成功。
2、配置从服务器
（1）开启二进制日志。
（2）要设置一个server-id  该值不能和主服务器的相同。

注意：配置好配置文件后，要重启mysql服务器；
（3）停止从服务器
执行，stop   slave  指令即可。

（4）开始配置，
配置的语法：
change master to master_host=”主服务器的ip地址”,master_user=”授权用户的名称”,master_password=”授权用户的密码”,master_log_file=”二进制日志文件的名称”,master_log_pos=记录的pos位置；

change master to master_host=”192.168.1.10”,master_user=’xiaogang’,master_password=’123456’,master_log_file=”mysql-bin.000001”,master_log_pos=611

（5）开启从服务器
执行start slave指令即可。

（6）查看是否配置成功
执行show slave status;



Slave_IO_Running:Yes 
此进程负责从服务器从主服务器上读取binlog 日志，并写入从服务器上的中继日志。 
Slave_SQL_Running:Yes 
此进程负责读取并且执行中继日志中的binlog日志， 
注：以上两个都为yes则表明成功，只要其中一个进程的状态是no，则表示复制进程停止，错误原因可以从”last_error”字段的值中看到。

3、测试主从复制
在主服务器创建一个新库，并添加一张新表，并插入新数据，

在从服务器上面查看是否有该库，该表，该记录。

4、撤销从服务器
在从服务器上执行如下两个指令。
（1）stop slave
（2）reset slave all
```


#### 3.12.6、实现读写分离


``` php
1、通过业务逻辑来实现读写分离
class  mysql{
	$dbm=主服务器 
	$dbs1=从服务器 
	$dbs2=从服务器 
	public function query(){
	   在query里面进行语句判断，分析连接不同的mysql服务器。 
	}
} 
2、TP框架里面实现读写分离
具体的步骤：
（1）通过mysql授权账号，
注意：授权的账号，是给php代码连接的。
主服务器：192.168.1.69的主机授权账号如下：

从服务器：192.168.1.70的主机授权账号如下：

（2）步骤TP框架里面配置文件



（3）在TP里面，测试读写分离的配置，
db()->query('select * from user');
db()->execute('insert into user values(3,"xiaolong")')
```

## 4 高并发Mysql 33条军规


### 4.1、基础规范

<label style="color:red">（1）必须使用InnoDB存储引擎

解读：支持事务、行级锁、并发性能更好、CPU及内存缓存页优化使得资源利用率更高



<label style="color:red">（2）必须使用utf8mb4字符集

解读：万国码，无需转码，无乱码风险，可存储表情文字emoji

 

<label style="color:red">（3）数据表、数据字段必须加入中文注释

解读：N年后谁tm知道这个r1,r2,r3字段是干嘛的


<label style="color:red">（4）禁止使用存储过程、视图、触发器、Event

解读：高并发大数据的互联网业务，架构设计思路是“解放数据库CPU，将计算转移到服务层”，并发量大的情况下，这些功能很可能将数据库拖死，业务逻辑放到服务层具备更好的扩展性，能够轻易实现“增机器就加性能”。数据库擅长存储与索引，CPU计算还是上移吧

 
<label style="color:red">（5）禁止存储大文件或者大照片

解读：为何要让数据库做它不擅长的事情？大文件和照片存储在文件系统，数据库里存URI多好

 
### 4.2、命名规范

<label style="color:red">（6）只允许使用内网域名，而不是ip连接数据库

 

<label style="color:red">（7）线上环境、开发环境、测试环境数据库内网域名遵循命名规范

业务名称：xxx

线上环境：tp.xxx.db

开发环境：tp.xxx.rdb

测试环境：tp.xxx.tdb

从库在名称后加-s标识，备库在名称后加-ss标识

线上从库：tp.xxx-s.db

线上备库：tp.xxx-sss.db

 

<label style="color:red">（8）库名、表名、字段名：小写，下划线风格，不超过32个字符，必须见名知意，禁止拼音英文混用


<label style="color:red">（9）表名t_xxx，非唯一索引名idx_xxx，唯一索引名uniq_xxx


### 4.5、表设计规范

<label style="color:red">（10）单实例表数目必须小于500



<label style="color:red">（11）单表列数目必须小于30

 

<label style="color:red">（12）表必须有主键，例如自增主键

解读：

a）主键递增，数据行写入可以提高插入性能，可以避免page分裂，减少表碎片提升空间和内存的使用

b）主键要选择较短的数据类型， Innodb引擎普通索引都会保存主键的值，较短的数据类型可以有效的减少索引的磁盘空间，提高索引的缓存效率

c） 无主键的表删除，在row模式的主从架构，会导致备库夯住

 

<label style="color:red">（13）禁止使用外键，如果有外键完整性约束，需要应用程序控制

解读：外键会导致表与表之间耦合，update与delete操作都会涉及相关联的表，十分影响sql 的性能，甚至会造成死锁。高并发情况下容易造成数据库性能，大数据高并发业务场景数据库使用以性能优先

 

### 4.4、字段设计规范

<label style="color:red">（14）必须把字段定义为NOT NULL并且提供默认值

解读：

a）null的列使索引/索引统计/值比较都更加复杂，对MySQL来说更难优化

b）null 这种类型MySQL内部需要进行特殊处理，增加数据库处理记录的复杂性；同等条件下，表中有较多空字段的时候，数据库的处理性能会降低很多

c）null值需要更多的存储空，无论是表还是索引中每行中的null的列都需要额外的空间来标识

d）对null 的处理时候，只能采用is null或is not null，而不能采用=、in、<、<>、!=、not in这些操作符号。如：where name!=’shenjian’，如果存在name为null值的记录，查询结果就不会包含name为null值的记录

 

<label style="color:red">（15）禁止使用TEXT、BLOB类型

解读：会浪费更多的磁盘和内存空间，非必要的大量的大字段查询会淘汰掉热数据，导致内存命中率急剧降低，影响数据库性能

 

<label style="color:red">（16）禁止使用小数存储货币

解读：使用整数吧，小数容易导致钱对不上
解决方案：使用“分”作为单位，这样数据库里就是整数了。

 

<label style="color:red">（17）必须使用varchar(20)存储手机号

解读：

a）涉及到区号或者国家代号，可能出现+-()

b）手机号会去做数学运算么？

c）varchar可以支持模糊查询，例如：like“138%”

 

<label style="color:red">（18）禁止使用ENUM，可使用TINYINT代替

解读：

a）增加新的ENUM值要做DDL操作

b）ENUM的内部实际存储就是整数，你以为自己定义的是字符串？

 

### 4.5、索引设计规范

<label style="color:red">（19）单表索引建议控制在5个以内

 

<label style="color:red">（20）单索引字段数不允许超过5个

解读：字段超过5个时，实际已经起不到有效过滤数据的作用了

 

<label style="color:red">（21）禁止在更新十分频繁、区分度不高的属性上建立索引

解读：

a）更新会变更B+树，更新频繁的字段建立索引会大大降低数据库性能

b）“性别”这种区分度不大的属性，建立索引是没有什么意义的，不能有效过滤数据，性能与全表扫描类似

 

<label style="color:red">（22）建立组合索引，必须把区分度高的字段放在前面

解读：能够更加有效的过滤数据

 

### 4.6、SQL使用规范

<label style="color:red">（23）禁止使用SELECT *，只获取必要的字段，需要显示说明列属性

解读：

a）读取不需要的列会增加CPU、IO、NET消耗

b）不能有效的利用覆盖索引

c）使用SELECT *容易在增加或者删除字段后出现程序BUG

 

<label style="color:red">（24）禁止使用INSERT INTO t_xxx VALUES(xxx)，必须显示指定插入的列属性

解读：容易在增加或者删除字段后出现程序BUG

 

<label style="color:red">（25）禁止使用属性隐式转换

解读：SELECT uid FROM t_user WHERE phone=13812345678 会导致全表扫描，而不能命中phone索引，猜猜为什么？（这个线上问题不止出现过一次）

 

<label style="color:red">（26）禁止在WHERE条件的属性上使用函数或者表达式

解读：SELECT uid FROM t_user WHERE from_unixtime(day)>='2017-02-15' 会导致全表扫描

正确的写法是：SELECT uid FROM t_user WHERE day>= unix_timestamp('2017-02-15 00:00:00')

 

<label style="color:red">（27）禁止负向查询，以及%开头的模糊查询

解读：

a）负向查询条件：NOT、!=、<>、!<、!>、NOT IN、NOT LIKE等，会导致全表扫描

b）%开头的模糊查询，会导致全表扫描

 

<label style="color:red">（28）禁止大表使用JOIN查询，禁止大表使用子查询, 超过三张表禁止JOIN

解读：会产生临时表，消耗较多内存与CPU，极大影响数据库性能

 

<label style="color:red">（29）禁止使用OR条件，必须改为IN查询

解读：旧版本Mysql的OR查询是不能命中索引的，即使能命中索引，为何要让数据库耗费更多的CPU帮助实施查询优化呢？

 

<label style="color:red">（30）应用程序必须捕获SQL异常，并有相应处理

### 4.7、行为规范

<label style="color:red">（31）禁止非DBA对线上数据库进行写操作，修改线上数据需要提交工单，由DBA执行，提交的SQL语句必须经过测试

<label style="color:red">（32）分配非DBA以只读帐号，必须通过VPN+跳板机访问授权的从库

<label style="color:red">（33）开发、测试、线上环境隔离
