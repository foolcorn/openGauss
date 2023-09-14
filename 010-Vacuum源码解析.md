## 010-Vacuum源码解析



openGauss中采用最大堆二叉树结构来记录和管理astore堆表页面的空闲空间，该最大堆二叉树结构按照页面粒度进行与存储介质的读写操作，并单独储存于专门的空闲空间位图文件中（free space map，简称FSM）

![image-20230811173646515](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811173646515.png)

所有页面分为叶子节点页面和内部节点页面两种。两种页面的页面内部结构完全相同，区别在于：对于叶子节点页面，其页面中记录的二叉树的叶子节点对应堆/索引表页面的空闲空间程度；对于内部节点页面，其页面中记录的二叉树的叶子节点对应下层FSM页面的最大空闲空间程度。
使用FSM页面中的1个字节（即256档）来记录一个堆/索引页面的空闲空间程度。在FSM页面中不会记录任何堆/索引页面的页号信息，也不会记录任何根、子FSM节点页面的页号信息，这些信息主要通过以下的规则来计算得到：
（1） 在一个FSM页面内部，二叉树节点按照从上到下、从左到右逐层排布，即：第一个字节为根节点的空闲程度，第二个字节为第一层内部节点最左边节点的空闲程度，依次类推。
（2） 所有FSM页面在物理存储上采用深度优先顺序，即某个FSM页面之前所有的物理页面包括：该FSM页面所在子树的所有上层节点，加上该FSM页面所有左侧子树。
（3）所有FSM叶子节点页面中的二叉树的叶子节点，对应堆/索引表页面的空闲空间程度，且根据从左到右的顺序，分别对应第1个、第2个、….、第n个堆/索引表物理页面。
（4）除了（3）中这些FSM节点之外，其他FSM父节点保存子节点（子树）中空闲空间的最大值。
根据上述算法，可以高效地查询出具有足够空闲空间的堆/索引页面的页面号，并将待插入的数据插入其中。

此外，为了保证FSM信息的维护操作不会带来明显的开销，因此FSM的所有修改都是不记录日志的。同时，对于某个堆/索引页面对应的FSM信息，只在页面初始化和页面空闲空间整理（见本节后面介绍）两种场景下才会主动更新，除此之外，只有当新插入的数据发现该页面实际空间不足时才会被动更新该页面对应的FSM信息（也包括由于宕机导致的FSM页面损坏）。

空闲空间的管理难点在于空闲空间的回收。在openGauss中，对于astore存储格式，有3种回收空闲空间的方式。

##### 轻量级堆页面清理

当查询扫描到某个astore堆表页面时，会顺带尝试清理该页面上已经被删除的、足够老的元组（足够老是指在元组对于所有并发查询均为已经删除状态）。由于只是顺带清理该页面内容，因此只能删除元组内容本身，元组指针还需要保留，以免在索引侧造成空引用或空指针（可参见4.2.5 行存储索引机制）。一个比较特殊的情况是HOT场景。HOT场景是指对于该表上所有的索引更新前后的索引键值均没有发生变化，因此对于更新后的元组只需要插入堆表元组而不需要新插入索引元组。对于同一个页面内一条HOT链上的多个元组，如果它们都足够老了，那么在清理时可以额外删除所有中间的元组指针，只保留第一个版本的元组指针，并将其重定向到第一个不用被清理的元组版本的元组指针。

![image-20230811174200891](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811174200891.png)
轻量级堆页面清理的接口是heap_page_prune_opt函数，关键的数据结构是PruneState结构体：

![image-20230811174510277](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811174510277.png)

- new_prune_xid：用于记录页面上此次没有被清理的、但是已经被删除的元组的xmax，用于决定下次何时再次清理该页面；
- latestRemovedXid：记录该页面上被清理的元组的最大的xmax，用于判断热备上回放页面整理时是否需要等待只读查询；
- nredirected：重定向的元组数量
- ndead：死亡的元组数量
- nunused：回收的元组数量
- redirected：存储重定向的元组
- nowdead：存储死亡的元组
- nowunused：存储待回收的元组
- marked：bool数组，记录Page中每一个元组是否属于上面三个数组之一，如果属于则为true



##### 中量级堆页面清理和索引页清理

openGauss提供VACUUM语句来让用户主动执行对某个astore表（或某个库中所有的astore表）及其上的索引进行中量级清理。中量级清理过程中，不阻塞相关表的查询和DML操作。由于在astore表中，新、老版本元组是混合存储的，因此，与顺带执行的轻量级清理相比，astore表的中量级清理需要进行全表顺序（或索引）扫描，才能识别出所有待清理的老版本元组。对于扫描出来的确认要清理的元组，会首先清理索引中的元组，然后再清理堆表中的元组，从而可以避免出现索引空指针的问题。
中量级清理的对外接口是lazy_vacuum_rel函数，内部逐层调用lazy_scan_rel、lazy_scan_heap和heap_page_prune（同轻量级清理）来扫描和暂存几类待清理的元组。当待清理的元组积攒到一定数量之后（受maintenance_work_mem内存上限控制），先后调用lazy_vacuum_index接口和lazy_vacuum_heap接口来分别清理索引文件和堆表文件。其中，与堆表页面将元组指针置为UNUSED不同，索引页面直接删除被清理的元组指针，并进行页面重整。

![image-20230811175221679](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811175221679.png)

中量级清理的关键数据结构是LVRelStats结构体：

![image-20230811180353193](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811180353193.png)

- hasindex：bool值，代表是否启用两阶段锁策略
- old_rel_pages：之前的页面个数统计 
- rel_pages：当前的页面个数统计
- scanned_pages：已经扫描的页面个数
- scanned_tuples：已经扫描的元组个数 
- old_rel_tuples：之前的元组个数统计
- new_rel_tuples：当前的元组个数统计
- pages_removed：移除的页数
- tuples_deleted：移除的元组数
- nonempty_pages：最后一个非空页面的页面号加1
- num_dead_tuples：当前待清理的元组个数
- max_dead_tuples：单次最多可记录的待清理元组个数
- dead_tuples：待清理元组行号数组
- num_index_scans：扫描的索引数？
- latestRemovedXid：最新的移除事务号
- lock_waiter_detected：是否锁等待检测？
- new_idx_pages：新的索引页面数？
- new_idx_tuples：新的索引元组数？     
- idx_estimated：预估的索引数？
- currVacuumPartOid：当前的懒 清理 oid
- hasKeepInvisibleTuples：？



##### 重量级堆页面和索引页面清理

无论是轻量级清理，或是中量级清理，都只能局部清理astore页面中的死亡元组，无法真正实现对这些空闲空间的释放（被清理出的空间，仍然只能被该表使用）。因此，openGauss还提供了VACUUM FULL语句来让用户主动执行对某个astore表（或某个库中所有astore表）及其上的索引进行重量级清理。重量级清理将一个表中所有仍未死亡（但是可能已经被删除）的元组重新紧密插入到新的堆表文件中并在此基础上重新创建所有索引，从而实现对空闲空间的彻底回收。在重量级清理的主体流程中只允许用户执行只读查询操作，在重量级清理的提交流程中只读查询操作也会被阻塞。
为了尽可能提高重新创建的索引性能，如果用户堆表上有索引，那么上述全表扫描会采用索引扫描。
重量级清理的对外接口是cluster_rel函数，内部逐层调用rebuild_relation、copy_heap_data、tableam_relation_copy_for_cluster、heapam_relation_copy_for_cluster、copy_heap_data_internal、reform_and_rewrite_tuple、rewrite_heap_tuple。其中，“rewrite_heap_tuple”接口将每一条扫描的未死亡元组进行重构（去除被删除的字段）之后，插入到新的紧密排列的堆表中。在这个过程中，对原来多个元组之间的更新链关系采用两个哈希表来进行暂存。当一对更新元组的双方都扫描到之后，就进行新表的填充，并将更新后元组的新的TID（transaction ID，事务ID）保存到更新前的元组中。上述机制保证重量级清理过程中并发更新事务的执行机制不会受到破坏。

![image-20230811182316519](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811182316519.png)

重量级清理的关键数据结构是RewriteStateData结构体

![image-20230811182624712](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230811182624712.png)

- Relation rs_old_rel：源表
- Relation rs_new_rel：整理后的目标表
- Page rs_buffer：当前整理的源表页面
- BlockNumber rs_blockno：当前写入的目标表页面号
- bool rs_buffer_valid：当前缓冲区是否有效
- bool rs_use_wal：整理操作是否产生日志
- TransactionId rs_oldest_xmin：用于可见性判断的最老活跃事务号
- TransactionId rs_freeze_xid：用于元组冻结判断的事务号

- rs_cxt：哈希表内存上下文
- rs_unresolved_tups：未匹配的更新前元组版本
- rs_old_new_tid_map：未匹配的更新后元组版本
- rs_compressor：用于压缩元组
- rs_cmprBuffer：用于存储压缩元组
- rs_tupBuf：提供给压缩数组的缓存Page
- rs_size：清理的一批元组的Size
- rs_nTups：清理的一批元组的数量
- rs_doCmprFlag：是否需要压缩标志位
- rs_buffers_queue：adio 写队列？
- rs_buffers_queue_ptr：adio 写队列指针？
- rs_buffers_handler：adio 写缓存句柄？
- rs_buffers_handler_ptr：adio 写缓存句柄指针？
- rs_block_start：adio写的起始Page号？
- rs_block_count：adio写的Page数


##### AutoVacuum

手动清理主观因素较大，是否清理完全取决于数据库管理员的意志，有可能造成极大的空间浪费，因此使用autovacuum按需清理,以控制浪费的资源.数据库知道随着时间的推移产生了多少死元组(每个事务报告它删除和更新的元组数),因此当表累积一定数量的死元组时会触发清理(默认情况下这是20％的死元组)表,但是它会使用数据库更繁忙,而在大部份数据库空闲时间较少.

