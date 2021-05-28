---
title: MySQL调优-参数配置
date: 2019-7-24 20:38:29
tags: [数据库, MySQL, 参数配置]
---
# MySQL调优-参数配置
## 设置参数的方法

以下方法优先级递减，第一种关闭当前session后失效，第二种当前服务进程关闭后失效

- set [session] 参数名 = 参数值; #设置当前会话（连接）参数
- set global 参数名 = 参数值; #设置全局参数，部分设置需要新建会话才生效
- 设置应用[配置文件](https://blog.51cto.com/moerjinrong/2092791)
  - Windows下将配置文件my.ini存放到应用程序根目录
  - Linux保存到/etc/my.cnf文件

## 常用参数

### connection连接参数

- max_connections，最大连接数，`show VARIABLES like 'max_connections';`

  max_connections 代表数据库同时允许的最大允许连接数

  连接有两种常见状态:sleep / query 

  sleep 代表连接处于闲置状态

  query 代表连接正处于处理任务的状态

  sleep + query 连接的总量不能超过max_connections的设置值

  否则会出现经典错误:"ERROR 1040：Too many connetcions"

- Thread%，查看连接状态，`show status like 'Threads%';`

  Threads_connected 代表当前已经有多少连接(Sleep+Query)

  Threads_created 代表历史总共创建过多少个数据库连接 

  Threads_running 代表有几个连接正处于"工作"状态，也是目前的并发数

  Threads_cached 共缓存过多少连接.如果我们在MySQL服务器配置文件中设置了thread_cache_size，当客户端断开之后，服务器处理此客户的线程将会缓存起来以响应下一个客户而不是销毁(前提是缓存数未达上限)。

  例：`set global thread_cache_size=80;`（缓存80个数据库缓存连接）

  查询MySQL历史运行过程中最大连接数的数量及时点，可根据该值去估算更改缓存大小

  `show status like 'Max_used_connections%';`

- back_log，设置保存多少数据库请求到堆栈（缓冲区）中，`show VARIABLES LIKE 'back_log'`

  如果MySql的连接数达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。将会报：
  unauthenticated user | xxx.xxx.xxx.xxx | NULL | Connect | NULL | login | NULL 的待连接进程时

- wait_timeout和interactive_timeout，这两个参数都是至超过一段时间后，数据库连接自动关闭(默认28800秒，即8小时)

  interactive_timeout针对交互式连接，wait_timeout针对非交互式连接。

  说得直白一点，通过mysql客户端连接数据库是交互式连接，通过jdbc连接数据库是非交互式连接。

  查看当前数据库连接详细状况：`show processlist;`

### 查询缓存（QC）参数

将结果放入到内存缓存区，之后使用相同（需完全相同，包括字符和大小写）的select语句，将直接从缓存区读取

使用`show status like 'Qcache%';`查看当前缓存使用情况

- Qcache_free_memory:Query Cache 中目前剩余的内存大小。通过这个参数我们可以较为准确的观察当前系统中的Query Cache内存大小是否足够，是需要增多还是过多了。

- Qcache_lowmen_prunes：多少条Query因为内存不足而被清除出Query Cache，通过Qcache_lowmem_prunes和Qcache_free_memory 相互结合，能够更清楚的了解到我们系统中Query Cache的内存大小是否真的足够，是否非常频繁的出现因为内存不足而有Query被换出。这个数字最好是长时间来看，如果这个数字在不断增长，就表示可能碎片化非常严重，或者内存很少。

- Qcache_total_blocks：当前Query Cache中block的数量

- Qcache_free_blocks：缓存中相邻内存块的个数。如果该值显示过大，则说明Query Cache中的内存碎片较多了。

  查询缓存碎片率：Qcache_free_block/Qcache_total_block*100%。如果查询缓存碎片率超过20%，可以用flush query cache整理缓存碎片。Block默认是 4KB ，设置值大对大数据查询有好处，但是如果你查询的都是小数据查询，就容易造成内存碎片和浪费。

- Qcache_hits：表示有多少次命中缓存。我们主要可以通过该值来验证我们的查询能缓存的效果。数字越大缓存效果越理想。


- Qcache_inserts：表示多少次未命中而插入，意思是新来的SQL请求在缓存中未找到，不得不执行查询处理，执行查询处理后把结果insert带查询缓存中。这样的情况次数越多，表示查询缓存 应用到的比较少，效果也就不理想。


- Qcache_queries_in_cache:当前缓存中缓存的查询数量。 


- Qcache_not_cached 未进入查询缓存的select个数

缓存参数设置

- query_cache_size，缓存大小。QC存储的单位最小是1024byte，所以如果你设定的一个不是1024的倍数的值。这个值会被四舍五入到最接近当前值的等于1024的倍数的值

- query_cache_limit，超出此大小的查询将不被缓存

- query_cache_type：缓存类型，决定缓存什么样子的查询，注意这个值不能随便设置必须设置为数字，**5.7中默认禁用QC，无法使用set命令设置，需要在my.ini中进行开启，**，可选值以及说明如下：

  -  0：OFF 相当于禁用了
  - 1：ON 将缓存所有结果，除非你的select语句使用了SQL_NO_CACHE禁用了查询缓存
  - 2：DENAND  则只缓存select语句中通过SQL_CACHE指定需要缓存的查询。

- query_cache_min_res_unit：缓存块的最小大小，query_cache_min_res_unit的配置是一柄双刃剑，默认是 4KB ，设置值大对大数据查询有好处，但是如果你查询的都是小数据查询，就容易造成内存碎片和浪费。

  每个查询占用缓存的平均值=（query_cache_size - Qcache_free_memory）/ Qcache_queries_in_cache

  查询缓存利用率：（query_cache_size - Qcache_free_memory) / query_cache_size * 100%，查询缓存利用率在25%以下的话说明query_cache_size 设置过大，可以适当减小：查询缓存利用率在80%以上而且Qcache_lowmem_prunes > 50的话说明query_cache_size可能有点小，要不就是碎片太多

  查询缓存命中率：Qcache_hits/(Qcache_hits+Qcache_inserts)*100%

- sort_buffer_size，排序缓冲区

  每个需要排序的线程分配该大小的一个缓冲区。增加这值加速ORDER BY 或 GROUP BY操作 sort_buffer_size是一个connection级的参数，在每个 connection（session）第一次需要使用这个buffer的时候，一次性分配设置的内存。sort_buffer_size并不是越大越好，由于是connection级的参数，过大的设置+高并发可能会耗尽系统的内存资源。例如：500个连接将会消耗500*sort_buffer_size(2M)=1G

### InooDB参数设置

- innodb_buffer_pool_size，缓存池大小设置

  该值设置可参考以下三个参数

  - Innodb已使用的缓存"页Page"数量

    show global status like 'Innodb_buffer_pool_pages_data';

  - Innodb全部缓存页数量

    show global status like 'Innodb_buffer_pool_pages_total';

  - Innodb每页的长度

    show global status like 'Innodb_page_size';

  使用这三个参数计算页面使用率：

  result = Innodb_buffer_pool_pages_data / Innodb_buffer_pool_pages_total * 100%；

  val > 95% 则考虑增大 innodb_buffer_pool_size， 建议使用物理内存的75%
  val < 95% 则考虑减小 innodb_buffer_pool_size， 建议设置为：Innodb_buffer_pool_pages_data * Innodb_page_size * 1.05 / (1024*1024*1024)

- innodb_flush_log_at_trx_commit，在事务控制中，存在"事务区"来保证事务完整性，在事务提交以后，这些事务区的数据会写入到硬盘上，同时事务操作日志（log）也需要向硬盘中写入。这个参数就是用来控制何时写日志数据的，可以设置以下三个值

  - 0：log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作。
  - 1：每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush（刷到磁盘）中去，该模式为系统默认。
  - 2：每次事务提交时MySQL都会把log buffer的数据写入log file，但是flush（刷到磁盘）操作并不会同时进行。该模式下，MySQL会每秒执行一次 flush（刷到磁盘）操作。

  优劣性比较

  - 当设置为0，该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
  - 当设置为1，该模式是最安全的，但也是最慢的一种方式。在mysqld 服务崩溃或者服务器主机crash的情况下，binary log 只有可能丢失最多一个语句或者一个事务。。
  - 当设置为2，该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。
  - 实际测试发现，该值对插入数据的速度影响非常大，设置为2时插入10000条记录只需要两秒，设置为0时只需要一秒，设置为1时，则需要229秒。因此，MySQL手册也建议尽量将插入操作合并成一个事务，这样可以大幅度提高速度。

- innodb_doublewrite，双写操作，同一份数据写入两次，保证数据存在一个副本，预防数据因为介质问题产生丢失，默认开启

- innodb_file_per_table，表空间存储模式

  - 0，使用统一的系统表空间
  - 1，使用独立的表空间

- innodb_thread_concurrency，设置innodb线程的并发数，默认值为0表示不被限制，若要设置则与服务器的CPU核心数相同或是CPU的核心数的2倍。