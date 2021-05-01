# undo log和redo log
## redo log
redo log是InnoDB存储引擎层的日志，又称重做日志文件，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来。在实例和介质失败（media failure）时，redo log文件就能派上用场，如数据库掉电，InnoDB存储引擎会使用redo log恢复到掉电前的时刻，以此来保证数据的完整性。==有了redo log，当数据库发生宕机重启后，可通过redo log将未落盘的数据恢复，即保证已经提交的事务记录不会丢失。==

==因为磁盘是有缓存的，只有当缓存被淘汰，或者通过操作系统的系统调用才会被落盘，那么在落盘前，数据库的数据是有可能会丢失的==

在一条更新语句进行执行的时候，InnoDB引擎会把更新记录写到redo log日志中，然后更新内存，此时算是语句执行完了，然后在空闲的时候或者是按照设定的更新策略将redo log中的内容更新到磁盘中，这里涉及到WAL即Write Ahead logging技术，他的关键点是先写日志，再写磁盘。

有了redo log日志，那么在数据库进行异常重启的时候，可以根据redo log日志进行恢复，也就达到了crash-safe。

==redo log是循环写，可以配置为1组4个文件，每个文件的大小是1GB==

**==redo log通常是物理日志==，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。**

## undo log
undo log用于回滚用，当进行操作时，undo log会写反操作。比如mysql进行insert，则undo log记录delete

**undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。记录操作**

## binlog
binlog是属于MySQL Server层面的，又称为归档日志，属于逻辑日志，是以二进制的形式记录的是这个语句的原始逻辑，依靠binlog是没有crash-safe能力的。主要是用于主从复制的时候做同步用的。直接把binlog给从机，从机按照binlog执行语句就行了。

### redo log和binlog区别
redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。

==**redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
redo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。**== 

==binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。==

> undo log为何要持久化，如何持久化

undo log是存储更新前的旧值的，也是逻辑log。为了防止断电，而事务的数据已经插入的情况，需要回滚。

一有更新语句，直接写入磁盘。

> redolog和binlog的两阶段提交说一下

先redo log进入prepare阶段，然后binlog落盘，然后redo log提交commit

1.prepare阶段，redo log落盘前，mysqld crash，回滚

2.prepare阶段，redo log落盘后，binlog落盘前，mysqld crash，回滚

3.commit阶段，binlog落盘后，mysqld crash，提交redo log

当MySQL写完redolog并将它标记为prepare状态时，并且会在redolog中记录一个XID，它全局唯一的标识着这个事务。只要这个XID和binlog中记录的XID是一致的，MySQL就会认为binlog和redolog逻辑上一致，就会commit。而如果仅仅是rodolog中记录了XID，binlog中没有，MySQL就会RollBack

> redo log什么时候落盘？

redo log会先写到redo log buffer中，落盘时机如下：

（1）如果写入redo log buffer的日志已经占据了redo log buffer总容量的一半了，也就是超过了8MB的redo log在缓冲里了，此时就会把他们刷入到磁盘文件里去

（2）一个事务提交的时候，必须把他的那些redo log所在的redo log block都刷入到磁盘文件里去，只有这样，当事务提交之后，他修改的数据绝对不会丢失，因为redo log里有重做日志，随时可以恢复事务做的修改
（PS：当然，之前最早最早的时候，我们讲过，这个redo log哪怕事务提交的时候写入磁盘文件，也是先进入os cache的，进入os的文件缓冲区里，所以是否提交事务就强行把redo log刷入物理磁盘文件中，这个需要设置对应的参数)

（3）后台线程定时刷新，有一个后台线程每隔1秒就会把redo log buffer里的redo log block刷到磁盘文件里去

（4）MySQL关闭的时候，redo log block都会刷入到磁盘里去

> 主从复制有几种模式

基于语句(Statement-based replication)和基于行(Row-based replication)(以下简称SBR，RBR)

**基于语句：**

实现方式：当leader收到客户端的请求，也就是执行语句，然后将每一个insert，update,delete语句发送给slave节点；

优点：因为直接发送执行语句，所以会产生比较少的日志数量；并且当机器因为故障停机需要备份时，可以很快的完成数据的恢复；

缺点：sql语句含有不确定的函数时，比如Now()或者Rand(),会使每一个slave节点产生不同的值，造成主从不一致；

sql表定义中auto_incrementde列或者依赖已存在的数据的语句，比如update ...where ..condition...，需要每个slave角色节点与master节点的执行顺序抑制，否则也会造成主从不一致的现象。

**基于行：**

实现方式：

插入：对于插入，日志中会包含表定义所有列的值。

删除：删除会包含足够的信息标识需要删除的行，通常情况下是表中的主键；如果表中没有主键，日志会记录需要删除行的旧值；

更新：更新操作会包含信息标识需要删除的行，并包含更新列的新值；

优点：因为日志中记录的是表数据修改的逻辑日志，对于主从复制模式没有数据不一致的现象出现；

相比SBR模式的复制模式，对于insert，udpate 和update语句会减少锁住行的数量，相应地提高数据库的并发。

缺点：RBR模式相比SBR模式会产生更多的日志文件；对于数据修改语句(DML)比如update、delete会把每一行的数据修改都会产生一条日志；

在日志中，不能看到用户执行的sql语句，只能看到每一行数据列的变化；

对于BLOB等类型大的数据类型，会产生较大的主从复制延迟
