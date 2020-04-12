MySQL体系结构与存储引擎

## MySQL体系结构

先看 MySQL 数据库的体系结构，如下图所示。      



<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/MySQL%E6%9E%B6%E6%9E%84.png" style="zoom: 80%;" />



MySQL体系结构由ClientConnectors层、MySQLServer层及存储引擎层组成。

**ClientConnectors层**

负责处理客户端的连接请求，与客户端创建连接。目前 MySQL 几乎支持所有的连接类型，例如常见的 JDBC、Python、Go 等。

**MySQL Server 层**

MySQLServer层主要包括ConnectionPool、Service&utilities、SQLinterface、Parser解析器、Optimizer查询优化器、Caches缓存等模块。

1. ConnectionPool，负责处理和存储数据库与客户端创建的连接，线程池资源管理，一个线程负责管理一个连接。还包括了用户认证模块，就是用户登录身份的认证和鉴权及安全管理，也就是用户执行操作权限校验。

2. Service & utilities 是管理服务&工具集，包括备份恢复、安全管理、集群管理服务和工具。
3. SQL interface，负责接收客户端发送的各种 SQL 语句，比如 DML、DDL 和存储过程等。
4. Parser 解析器会对 SQL 语句进行语法解析生成解析树。
5. Optimizer 查询优化器会根据解析树生成执行计划，并选择合适的索引，然后按照执行计划执行 SQL 语言并与各个存储引擎交互。
6. Caches缓存包括各个存储引擎的缓存部分，比如：InnoDB存储的BufferPool，Caches中也会缓存一些权限，也包括一些 Session 级别的缓存。

**存储引擎层**

存储引擎包括MyISAM、InnoDB，以及支持归档的Archive和内存的Memory等。MySQL是插件式的存储引擎，只要正确定义与MySQLServer交互的接口，任何引擎都可以访问MySQL。

**物理存储层**

存储引擎底部是物理存储层，是文件的物理存储层，包括二进制日志、数据文件、错误日志、慢查询日志、全日志、redo/undo 日志等。



下面是一条SQL SELECT语句的执行过程：

<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/SQL%E8%AF%AD%E5%8F%A5%E7%9A%84%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png" style="zoom: 80%;" />

## 存储引擎

存储引擎是 MySQL 中具体与文件打交道的子系统，它是根据 MySQL AB 公司提供的文件访问层的抽象接口，定制的一种文件访问机制，这种机制就叫作存储引擎。

InnoDB 存储引擎的具体架构如下图所示。上半部分是实例层（计算层），位于内存中，下半部分是物理层，位于文件系统中。





### 实例层

实例层分为线程和内存。

InnoDB 重要的线程有 Master Thread，Master Thread 是 InnoDB 的主线程，负责调度其他各线程。

* MasterThread的优先级最高,其内部包含几个循环：主循环（loop）、后台循环（backgroundloop）、刷新循环（flushloop）、暂停循环（suspendloop）。Master Thread 会根据其内部运行的相关状态在各循环间进行切换。

  大部分操作在主循环（loop）中完成，其包含 1s 和 10s 两种操作。

* buf_dump_thread 负责将 buffer pool 中的内容 dump 到物理文件中，以便再次启动 MySQL 时，可以快速加热数据。

* page_cleaner_thread负责将bufferpool中的脏页刷新到磁盘，在5.6版本之前没有这个线程，刷新操作都是由主线程完成的，所以在刷新脏页时会非常影响MySQL的处理能力，在5.7 版本之后可以通过参数设置开启多个 page_cleaner_thread。

* purge_thread 负责将不再使用的 Undo 日志进行回收。

* read_thread 处理用户的读请求，并负责将数据页从磁盘上读取出来，可以通过参数设置线程数量。

* write_thread 负责将数据页从缓冲区写入磁盘，也可以通过参数设置线程数量，page_cleaner 线程发起刷脏页操作后 write_thread 就开始工作了。

* redo_log_thread 负责把日志缓冲中的内容刷新到 Redo log 文件中。

* insert_buffer_thread 负责把 Insert Buffer 中的内容刷新到磁盘。

实例层的内存部分主要包含InnoDBBufferPool，这里包含InnoDB最重要的缓存内容。数据和索引页、undo页、insertbuffer页、自适应Hash索引页、数据字典页和锁信息等。additionalmemorypool后续已不再使用。Redobuffer里存储数据修改所产生的Redolog。doublewritebuffer是 double write 所需的 buffer，主要解决由于宕机引起的物理写入操作中断，数据页不完整的问题。

### 物理层

物理层在逻辑上分为系统表空间、用户表空间和 Redo日志。

系统表空间里有 ibdata 文件和一些 Undo，ibdata 文件里有 insert buffer 段、double write段、回滚段、索引段、数据字典段和 Undo 信息段。

用户表空间是指以 .ibd 为后缀的文件，文件中包含 insert buffer 的 bitmap 页、叶子页（这里存储真正的用户数据）、非叶子页。

Redo日志中包括多个Redo文件，这些文件循环使用，当达到一定存储阈值时会触发checkpoint刷脏页操作，同时也会在MySQL实例异常宕机后重启，InnoDB表数据自动还原恢复过程中使用。

### 内存和物理结构

内存和物理结构，如下图所示。

<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/%E5%86%85%E5%AD%98%E5%92%8C%E7%89%A9%E7%90%86%E7%BB%93%E6%9E%84.png" style="zoom: 80%;" />

**BufferPool**

用户读取或者写入的最新数据都存储在BufferPool中，如果BufferPool中没有找到则会读取物理文件进行查找，之后存储到BufferPool中并返回给MySQLServe。Buffer Pool 采用LRU 机制。

BufferPool决定了一个SQL执行的速度快慢，如果查询结果页都在内存中则返回结果速度很快，否则会产生物理读（磁盘读），返回结果时间变长。但我们又不能将所有数据页都存储到BufferPool中。在单机单实例情况下，我们可以配置BufferPool为物理内存的60%~80%，剩余内存用于session产生的sort和join等，以及运维管理使用。如果是单机多实例，所有实例的bufferpool总量也不要超过物理内存的80%。开始时我们可以根据经验设置一个BufferPool的经验值，比如16GB，之后业务在MySQL运行一段时间后可以根据show global status like'%buffer_pool_wait%' 的值来看是否需要调整 Buffer Pool 的大小。

**Redolog**

Redolog是一个循环复用的文件集，**负责记录InnoDB中所有对BufferPool的物理修改日志**，当Redolog文件空间中，检查点位置的LSN和最新写入的LSN差值（checkpoint_age）达到Redolog文件总空间的75%后，InnoDB会进行异步刷新操作，直到降至75%以下，并释放Redolog的空间；当checkpoint_age达到文件总量大小的 90% 后，会触发同步刷新，此时 InnoDB 处于挂起状态无法操作。

补充：

* 日志序号 (LSN:Log sequence number) 标识特定日志文件记录在日志文件中的位置。 
* checkpoint_age：检查点，将缓冲池中的脏页刷回到磁盘。当缓冲池不够用时，根据LRU算法溢出的页，若此页为脏页，那么需要强制执行Checkpoint，将脏页也就是页的新版本刷回磁盘。 

每个页有LSN，重做日志中也有LSN，Checkpoint也有LSN。可以通过命令SHOW ENGINE INNODB STATUS来观察 。

这样我们就看到**Redolog的大小直接影响了数据库的处理能力**，如果设置太小会导致强行checkpoint操作频繁刷新脏页，那我们就需要将Redolog设置的大一些，5.6版本之前Redo log 设置的大一些，5.6 版本之前 Redo log 总大小不能超过 3.8GB，5.7 版本之后放开了这个限制。

事务提交时 log buffer 会刷新到 Redo log 文件中，具体刷新机制由参数控制。

### Myisam和InnoDB的区别

* **是否支持行级锁** : MyISAM 只有表级锁，而InnoDB 支持行级锁和表级锁,默认为行级锁，适合高并发操作。 
* **是否支持外键**： MyISAM不支持，而InnoDB支持 
* **是否支持事务**：MyISAM不支持，而InnoDB支持 
* **缓存**：MyISAM只缓存索引，InnoDB缓存索引和真实数据，所以对内存要求高
* **崩溃恢复**：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。 
* InnoDB 支持 MVCC，MyISAM 不支持；


InnoDB 表最大还可以支持 64TB，支持聚簇索引、支持压缩数据存储，支持数据加密，支持查询/索引/数据高速缓存，支持自适应hash索引、空间索引，支持热备份和恢复等

## InnoDB 核心要点

<img src="https://github.com/lvminghui/Java-Notes/tree/master/docs/typora-user-images" style="zoom: 80%;" />



ARIES 三原则

WriteAheadLogging（WAL）。

* 先写日志后写磁盘，日志成功写入后事务就不会丢失，后续由checkpoint机制来保证磁盘物理文件与Redo日志达到一致性；
* 利用Redo 记录变更后的数据，即 Redo 记录事务数据变更后的值；
* 利用 Undo 记录变更前的数据，即 Undo 记录事务数据变更前的值，用于回滚和其他事务多版本读。

show engine innodb status\G 的结果里面有详细的 InnoDB 运行态信息，分段记录的，包括内存、线程、信号、锁、事务。

# 深入理解事务与锁机制

## 事务及其特性

一个逻辑工作单元要成为事务，在关系型数据库管理系统中，必须满足 4 个特性，即所谓的 ACID：原子性、一致性、隔离性和持久性。

Atomicity（原子性）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行 

Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。

Isolation（隔离性）：同一时间，只允许一个事务操作同一数据，不同的事务之间彼此没有任何干扰。 

Durability（持久性）：事务处理结束后，对数据的修改是永久的。

### 一致性

一致性其实包括两部分内容，分别是约束一致性和数据一致性。

* 约束一致性：数据库创建表结构时所制定的外键，唯一索引等约束。
* 数据一致性：是一个综合性的规定，或者说是一个把握全局的规定。因为它是由原子性、持久性、隔离性共同保证的结果，而不是单单依赖于某一种技术。

### 原子性

原子性就是前面提到的两个“要么”，即要么改了，要么没改。也就是说用户感受不到一个正在改的状态。MySQL 是通过 WAL（Write Ahead Log）技术来实现这种效果的。

举例来讲，如果事务提交了，那改了的数据就生效了，如果此时BufferPool的脏页没有刷盘，如何来保证改了的数据生效呢？就需要使用Redo日志恢复出来的数据就需要使用Redo日志恢复出来的数据。而如果事务没有提交，且BufferPool的脏页被刷盘了，那这个本不应该存在的数据如何消失呢？就需要通过 Undo 来实现了，Undo 又是通过 Redo 来保证的，所以最终原子性的保证还是靠 Redo 的 WAL 机制实现的。

### 持久性

所谓持久性，就是指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的，接下来的操作或故障不应该对其有任何影响。持久性是如何保证的呢？一旦事务提交，通过原子性，即便是遇到宕机，也可以从逻辑上将数据找回来后再次写入物理存储空间，这样就从逻辑和物理两个方面保证了数据不会丢失，即保证了数据不会丢失，即保证了数据库的持久性。

### 隔离性

所谓隔离性，指的是一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对其他的并发事务是隔离的。锁和多版本控制就符合隔离性。

## 并发事务控制

### 单版本控制-锁

锁用独占的方式来保证在只有一个版本的情况下事务之间相互隔离，所以锁可以理解为单版本控制。在MySQL事务中，锁的实现与隔离级别有关系，在RR（RepeatableRead）隔离级别下，MySQL为了解决幻读的问题，以牺牲并行度为代价，通过Gap锁来防止数据的写入，而这种锁，因为其并行度不够，冲突很多，经常会引起死锁。现在流行的Row模式可以避免很多冲突甚至死锁问题，所以推荐默认使用 Row + RC（Read Committed）模式的隔离级别，可以很大程度上提高数据库的读写并行度。

补充： 在row level模式下，bin-log中可以不记录执行的sql语句的上下文相关的信息，仅仅只需要记录那一条被修改。 

### 多版本控制-MVCC

 MVCC，是指在数据库中，为了实现高并发的数据访问，对数据进行多版本处理，并通过事务的可见性来保证事务能看到自己应该看到的数据版本。

那个多版本是如何生成的呢？每一次对数据库的修改，都会在Undo日志中记录当前修改记录的事务号及修改前数据状态的存储地址（即ROLL_PTR），以便在必要的时候可以回滚到老的数据版本。例如，一个读事务查询到当前记录，而最新的事务还未提交，根据原子性，读事务看不到最新数据，但可以去回滚段中找到老版本的数据，这样就生成了多个版本。多版本控制很巧妙地将稀缺资源的独占互斥转换为并发，大大提高了数据库的吞吐量及读写性能。

## 技术原理

### 原子性技术原理

每一个写事务，都会修改BufferPool，从而产生相应的Redo日志，这些日志信息会被记录到ib_logfiles文件中。因为Redo日志是遵循WriteAheadLog的方式写的，所以事务是顺序被记录的。在MySQL中，任何BufferPool中的页被刷到磁盘之前，都会先写入到日志文件中，这样做有两方面的保证。如果BufferPool中的这个页没有刷成功，此时数据库挂了，那在数据库再次启动之后，可以通过 Redo 日志将其恢复出来，以保证脏页写下去的数据不会丢失，所以必须要保证 Redo 先写。

为 Buffer Pool 的空间是有限的，要载入新页时，需要从 LRU 链表中淘汰一些页，而这些页必须要刷盘之后，才可以重新使用，那这时的刷盘，就需要保证对应的 LSN 的日志也要提前写到 ib_logfiles 中，如果没有写的话，恰巧这个事务又没有提交，数据库挂了，在数据库启动之后，这个事务就没法回滚了。所以如果不写日志的话，这些数据对应的回滚日志可能就不存在，导致未提交的事务回滚不了，从而不能保证原子性，所以原子性就是通过 WAL 来保证的。

### 持久性技术原理

通过原子性可以保证逻辑上的持久性，通过存储引擎的数据刷盘可以保证物理上的持久性。这个过程与前面提到的Redo日志、事务状态、数据库恢复、参数innodb_flush_log_at_trx_commit 有关，还与 binlog 有关。

### 隔离性技术原理

InnoDB 支持的隔离性有 4 种，隔离性从低到高分别为：读未提交、读提交、可重复读、可串行化。

具体说到隔离性的实现方式，我们通常用ReadView表示一个事务的可见性。RC级别的事务可见性比较高，它可以看到已提交的事务的所有修改。而RR级别的事务，则没有这个功能，一个读事务中，不管其他事务对这些数据做了什么修改，以及是否提交，只要自己不提交，查询的数据结果就不会变。

随着时间的推移，读提交每一条读操作语句都会获取一次 Read View，每次更新之后，都会获取数据库中最新的事务提交状态，也就可以看到最新提交的事务了，即每条语句执行都会更新其可见性视图。

而反观可重复读，这个可见性视图，只有在自己当前事务提交之后，才去更新，所以与其他事务是没有关系的。

**在 RR 级别下，长时间未提交的事务会影响数据库的 PURGE 操作，从而影响数据库的性能，所以可以对这样的事务添加一个监控**。

### 一致性技术原理

数据的完整性是通过其他三个特性来保证的，为了保证数据的完整性，提出来三个特性，这三个特性又是由同一个技术来实现的，所以理解 Redo/Undo 才能理解数据库的本质。

### MVCC 实现原理

MVCC最大的好处是读不加锁，读写不冲突。在读多写少的OLTP（On-LineTransactionProcessing）应用中，读写不冲突是非常重要的，极大的提高了系统的并发性能。

在 MVCC 并发控制中，读操作可以分为两类: 快照读（Snapshot Read）与当前读 （Current Read）。

* 快照读：读取的是记录的可见版本（有可能是历史版本），不用加锁。
* 当前读：读取的是记录的最新版本，并且当前读返回的记录，都会加锁，保证其他事务不会再并发修改这条记录。 

如何区分快照读和当前读呢？ 可以简单的理解为：

* 快照读：简单的 select 操作，属于快照读，不需要加锁。 
* 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。 

下面用一个事务对某行记录更新的过程来说明MVCC中多版本的实现。

假设 F1～F6 是表中字段的名字，1～6 是其对应的数据。后面三个隐含字段分别对应该行的隐含ID、事务号和回滚指针，如下图所示。

<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/MVCC%E5%AE%9E%E7%8E%B0.png" alt="image-20200412113401177" style="zoom:80%;" />

* 隐含ID（DB_ROW_ID），6个字节，当由InnoDB自动产生聚集索引时，聚集索引包括这个DB_ROW_ID的值。
* 事务号（DB_TRX_ID），6个字节，标记了最新更新这条行记录的TransactionID，每处理一个事务，其值自动+1。
* 回滚指针（DB_ROLL_PT），7个字节，指向当前记录项的RollbackSegment的Undolog记录，通过这个指针才能查找之前版本的数据。

首先，假如这条数据是刚 INSERT 的，可以认为 ID 为 1，其他两个字段为空。

然后，当事务 1 更改该行的数据值时，会进行如下操作，如下图所示。

<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/MVCC%E5%AE%9E%E7%8E%B02.png" style="zoom:80%;" />

* 用排他锁锁定该行；记录 Redo log
* 把该行修改前的值复制到 Undo log，即图中下面的行
* 修改当前行的值，填写事务编号，使回滚指针指向 Undo log 中修改前的行

如果数据继续执行，此时Undolog中有两行记录，并且通过回滚指针连在一起。因此，如果Undolog一直不删除，则会通过当前记录的回滚指针回溯到该行创建时的初始内容，所幸的是在InnoDB中存在purge 线程，它会查询那些比现在最老的活动事务还早的 Undo log，并删除它们，从而保证 Undo log 文件不会无限增长，如下图所示。

<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/MVCC3.png" style="zoom:80%;" />

## 并发事务问题及解决方案

脏读 ：表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。

不可重复读 ：是指在一个事务内，多次读同一数据数据发生了变化。

幻读 ：指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。

产生的这些问题，MySQL 数据库是通过事务隔离级别来解决的。值得一提的是，InnoDB通过Gap锁解决了幻读的问题。

不可重复读重点在于 UPDATA 和 DELETE，而幻读的重点在于 INSERT。它们之间最大的区别是如何通过锁机制来解决它们产生的问题。



## InnoDB 的锁

InnoDB 的锁分为行锁和表锁。

其中行锁包括两种：

* 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。
* 排他锁（X）：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是表锁。表锁又分为三种。

* 意向共享锁（IS）：事务计划给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁
* 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的的 IX 锁。
* 自增锁（AUTO-INC Locks）：特殊表锁，自增长计数器通过该“锁”来获得子增长计数器最大的计数值。

在加行锁之前必须先获得表级意向锁，否则等待 innodb_lock_wait_timeout 超时后根据innodb_rollback_on_timeout 决定是否回滚事务。

### InnoDB自增锁

在MySQLInnoDB存储引擎中，我们在设计表结构的时候，通常会建议添加一列作为自增主键。这里就会涉及一个特殊的锁：自增锁（即：AUTO-INCLocks），它属于表锁的一种，在 INSERT 结束后立即释放。我们可以执行 **show engine innodb status\G** 来查看自增锁的状态信息。

InnoDB 锁关系矩阵如下图，其中：+ 表示兼容，- 表示不兼容。



<img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/typora-user-images/InnoDB%20%E9%94%81%E5%85%B3%E7%B3%BB%E7%9F%A9%E9%98%B5.png"  style="zoom:80%;" />

### InnoDB 行锁

InnoDB 行锁是通过对索引数据页上的记录（record）加锁实现的。主要实现算法有 3 种：Record Lock、Gap Lock 和 Next-key Lock。

* RecordLock锁：单个行记录的锁（锁数据，不锁Gap）。
* GapLock锁：间隙锁，锁定一个范围，不包括记录本身（不锁数据，仅仅锁数据前面的Gap）。
* Next-keyLock 锁：同时锁住数据，并且锁住数据前面的 Gap。

**排查 InnoDB 锁问题**

排查InnoDB锁问题通常有2种方法。打开innodb_lock_monitor表，注意使用后记得关闭，否则会影响性能。在MySQL5.5版本之后，可以通过查看information_schema 库下面的 innodb_locks、innodb_lock_waits、innodb_trx 三个视图排查 InnoDB 的锁问题。

### InnoDB死锁

在MySQL中死锁不会发生在MyISAM存储引擎中，但会发生在InnoDB存储引擎中，因为InnoDB是逐行加锁的，极容易产生死锁。那么死锁产生的四个条件是什么呢？

* 互斥条件：一个资源每次只能被一个进程使用；
* 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放；
* 不剥夺条件：进程已获得的资源，在没使用完之前，不能强行剥夺；
* 循环等待条件：多个进程之间形成的一种互相循环等待资源的关系。

在发生死锁时，InnoDB存储引擎会自动检测，并且会自动回滚代价较小的事务来解决死锁问题。但很多时候一旦发生死锁，InnoDB存储引擎的处理的效率是很低下的或者有时候根本解决不了问题，需要人为手动去解决。

既然死锁问题会导致严重的后果，那么在开发或者使用数据库的过程中，如何避免死锁的产生呢？这里给出一些建议：

* 加锁顺序一致；
* 尽量基于primary或uniquekey更新数据。
* 单次操作数据量不宜过多，涉及表尽量少。
* 减少表上索引，减少锁定资源。
* 相关工具：pt-deadlock-logger

查看MySQL数据库中死锁的相关信息，可以执行showengineinnodbstatus\G来进行查看，重点关注“LATESTDETECTEDDEADLOCK”部分。

一些开发建议来避免线上业务因死锁造成的不必要的影响。

* 更新SQL的where条件时尽量用索引；
* 加锁索引准确，缩小锁定范围；
* 减少范围更新，尤其非主键/非唯一索引上的范围更新。
* 控制事务大小，减少锁定数据量和锁定时间长度 （innodb_row_lock_time_avg）。
* 加锁顺序一致，尽可能一次性锁定所有所需的数据行。

