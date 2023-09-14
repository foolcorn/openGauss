## 009-缓冲区buffer源码解析



对于一个基于磁盘的数据库而言，如果每次查询都要从磁盘中读取元组实在太慢了，所以在内存中设立了共享的高速缓冲区，用来缓存一些经常被访问的页面，其设计架构如图所示。

![在这里插入图片描述](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/9fd469ea441549acade3d7f1c910c9a3.png)



最上层的是一个hash表，key的类型是BufferTag，映射的value是bufferid

BufferTag可以唯一标记一个磁盘上文件的Page，bufferid则对应于缓冲区buffer数组的下标。

#####  BufferTag类

![image-20230810153153582](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230810153153582.png)

- rnode：行存物理文件名的主体命名
- forkNum：行存物理文件名的后缀命名
- blockNum：文件中的页面号



在buffer设计上，openGauss将bufffer数组拆成两个部分，一个数组记录了buffer的描述信息，另一个数组则是纯粹的存储单元，大小和Page对应。比较反人类的设计是两个数组的下标差1，即存储buffer的下标从1开始而不是从0开始，因此存储id=buffer desc id + 1。

##### BufferDesc类

![image-20230810154913145](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230810154913145.png)

- tag：即buffer对应的BufferTag（唯一Page标识）
- state：对应buffer的状态位（排他锁标志，脏页标志，有效页面标志，有效tag标志，页面I/O状态标志，被pin的次数usage_count）
- buf_id：就是buffer id（从0开始）
- wait_backend_pid：等待页面unpin的线程的线程号
- io_in_progress_lock：用于管理页面并发I/O操作（从磁盘加载和写入磁盘）的轻量级锁。
- content_lock：管理页面内容并发读写操作的轻量级锁。
- rec_lsn：上次写入磁盘之后该页面第一次修改操作的日志lsn值
- dirty_queue_loc：该页面在全局脏页队列数组中的（取模）下标。

关于pin和unpin，当某个线程准备访问buffer时，需要pin住这个buffer，代表这个buffer正在被该线程使用，这样这个buffer无法被其他线程淘汰。当不访问buffer时，就需要解pin。同时buffer也可以被多个线程同时pin住，通过引用计数记录，usage_count。当pin了一次时，这个buffer对应的引用计数就加一。

##### Buffer数组

遗憾的是我没找到相关源码，目前还是疑惑状态，具体形式是什么，理论上应该是一个bufpage的数组。

Buffer之下是Bgwriter模块，即后台写线程，负责将脏页写入到磁盘中，首先buffer会平均分配给每个bgwriter线程，线程会循环扫描自己负责的buffer的desc信息，如果state标记为脏，则会收集起来批量写入到双写文件中，若后续刷盘后，将脏页标记为非脏。

##### BgWriterProc类

该类存储一个bgwriter线程的相关信息

![image-20230810171348638](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230810171348638.png)

- proc：？
- dirty_buf_list：存储每个线程收集的脏页信息数组
- dirty_list_size：脏页信息数组长度
- cand_buf_list：存储刷盘后非脏页面（包括一直非脏的页面空闲后也会存入该数组）
- cand_list_size：空闲数组长度
- buf_id_start：该bgwriter负责的bufferid范围的起始id
- head：空闲数组的头下标
- tail：空闲数组的尾下标
- need_flush：是否需要刷盘
- thrd_dw_cxt：双写线程事务？？？
- thread_last_flush：？
- next_scan_loc：上次bgwriter循环扫描停止处的bufferid，下次收集脏页从该位置开始



##### CkptSortItem类

暂时没弄懂这个数据结构的意义是什么

![image-20230810173503407](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230810173503407.png)

- tsId：？
- relNode：?
- bucketNode：？
- forkNum：
- blockNum：
- buf_id：

与bgwriter同时进行的还有pagewriter线程，分为主线程和子线程。主pagewriter线程只有一个，负责从全局脏页队列数组中批量获取脏页面、将这些脏页批量写入双写文件、推进整个数据库的检查点（故障恢复点）、分发脏页给各个pagewriter子线程，以及将分发给自己的脏页写入文件系统。子线程只负责将主线程分发给自己的脏页写入文件系统。

注意从全局脏页队列数组和Buffer中的脏页并不冲突，所有Buffer中的脏页会同步记录到全局的脏页队列中，Buffer的脏页由对应负责的bgwriter写入到双写文件中，全局脏页队列里的脏页由pagewriter写入到双写文件中，盲猜双写文件应该会自动解决重复问题，而且如果刷盘出问题数据损坏，还可以用全局脏页队列的数据做补救，而且pagewriter的主要目的不是写脏页，只是顺带，主要是为了推进checkpoint。

pagewriter线程的信息保存在PageWriterProc结构体中。

##### PageWriterProc类

![image-20230810182912517](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230810182912517.png)

- proc：？
- start_loc：？
- end_loc：？
- need_flush：？
- actual_flush_num：？