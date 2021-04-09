# innodb和myisam存储引擎的区别
在MySQL 5.5之前，MyISAM是mysql的默认数据库引擎，其由早期的ISAM（Indexed Sequential Access Method：有索引的顺序访问方法）所改良。虽然MyISAM性能极佳，但却有一个显著的缺点：不支持事务处理。不过，MySQL也导入了另一种数据库引擎InnoDB，以强化参考完整性与并发违规处理机制，后来就逐渐取代MyISAM。

## 区别
**存储结构：**
- MyISAM表是独立于操作系统的，这说明可以轻松地将其从Windows服务器移植到Linux服务器；每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表明。例如，我建立了一个MyISAM引擎的test表，那么就会生成以下三个文件：
  - test.frm，存储表定义；
  - test.MYD，存储数据；
  - test.MYI，存储索引。==**索引文件中的索引指向数据文件中的数据**==
- InnoDB ==**使用表空间存储数据和索引。**== 可以选择使用共享表空间还是独占表空间，默认是共享表。除了表空间，InnoDB也会生成.frm存储表定义。
  - 共享表空间以及独占表空间都是针对数据的从物理意义上来讲
  - 共享表空间:  会把表集中存储在一个系统表空间里。即每一个数据库的所有表的数据，索引文件全部放在一个文件中。该文件目录默认的是服务器的数据目录。 默认的文件名为:ibdata1  初始化为10M。
  - 独占表空间:  每一个表分别创建一个表空间，这时。在对应的数据库目录里每一个表都有.ibd文件(这个文件包括了单独一个表的数据内容以及索引内容)。
  - 总共生产2个文件：
    - test.frm
    - test.ibd

**存储空间：**
- MyISAM可被压缩，占据的存储空间较小，支持静态表、动态表、压缩表三种不同的存储格式。
  - 静态表：比如char(20)，那么就会实打实的存20个字符，如果不足20个字符，用空格填充满。查询速度快因为知道下一个字段一定从21个字符开始。但是需要更多的存储空间。
  - 动态表：比如char(20)，实际有多少存多少。用更少的存储空间，但是难以维护。
  - 压缩表：只读表，每条记录分开压缩，所以不能同时访问
- InnoDB需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。（使用LRU最近最少使用的置换算法）

**可移植性、备份及恢复：**
- MyISAM的数据是以文件的形式存储 ==**是独立于操作系统的**== ，所以在跨平台的数据转移中会很方便，同时在备份和恢复时也可单独针对某个表进行操作。
- InnoDB备份的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了。

**事务支持：**
- MyISAM强调的是性能，每次查询具有原子性，其执行数度比InnoDB类型更快，但是不提供事务支持。
- InnoDB提供事务、外键等高级数据库功能，具有事务提交、回滚和崩溃修复能力。

**AUTO_INCREMENT：**
- InnoDB的auto_increment字段必须是索引。如果是组合索引，必须为组合索引的第一列。
  - ```sql
    create table autoincrement_demo_inno(
    id1 int not null auto_increment,
    id2 int not null,
    name varchar(10),
    index(id1, id2)
    ) engine=InnoDB
    ```
  - 这里插入几条数据，id1就会增加多少

- 在MyISAM中，auto_increment字段也必须是索引，但如果是组合索引，可以不是组合索引的第一列
  - ```sql
    create table autoincrement_demo_inno(
    id1 int not null auto_increment,
    id2 int not null,
    name varchar(10),
    index(id2, id1)
    ) engine=MyISAM
    ```
  - 这里自增的是id1，但是索引第一个是id2.结果如下
  - ![](https://gitee.com/super-jimwang/img/raw/master/img/20210221152113.png)
  - id1的自增在id2相同的情况下才会进行。

**表锁差异：**
- MyISAM只支持表级锁，用户在操作MyISAM表时，select、update、delete和insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。
- InnoDB支持事务和行级锁。行锁大幅度提高了多用户并发操作的新能，但是==**InnoDB的行锁，只是在唯一索引是有效的**==，非唯一索引的WHERE都会锁全表的。

表主键：
- MyISAM允许没有任何索引和主键的表存在，==**索引保存的是行的地址，不含数据**==。
- 对于InnoDB，如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)，==**InnoDB数据库中的数据和主键节点保存在一起，所有其他索引节点中保存的是主键索引的值。**==

**表的具体行数：**
- MyISAM保存表的总行数，select count() from table;会直接取出出该值；
- 而InnoDB没有保存表的总行数，如果使用select count() from table；就会遍历整个表，消耗相当大，但是在加了wehre条件后，myisam和innodb处理的方式都一样。

**CURD操作：**
- 在MyISAM中，如果执行大量的SELECT，MyISAM是更好的选择。
  - ==**为什么MyISAM查询更快？**== 看myisam与innodb的索引细节
    - 查询的时候，由于innodb支持事务，所以会有mvvc的一个比较。这个过程会损耗性能。
    - 查询的时候，如果走了索引，由于innodb是聚簇索引，会有一个回表的过程，即：先去非聚簇索引树中查询数据，找到数据对应的key之后，再通过key回表到聚簇索引树，最后找到需要的数据。而myisam直接就是非簇集索引，查询的时候查到的最后结果不是聚簇索引树的key，而是磁盘地址，所以会直接去查询磁盘。
    - 锁的一个损耗，innodb锁支持行锁，在检查锁的时候不仅检查表锁，还要看行锁。
- 对于InnoDB，如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表（并发的时候，myisam是表锁，而innodb可以做到行锁，所以innodb更快）。DELETE从性能上InnoDB更优，但DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除，在innodb上如果要清空保存有大量数据的表，最好使用truncate table这个命令。

**外键：**
MyISAM不支持外键，而InnoDB支持外键。

## 总结
MyISAM更适合读多，并发少的场景。

> 如何通过索引找到记录

通过索引遍历树，到达叶子节点的页。叶子结点的关键字多的话会导致一页装不下。

遍历这一页，如果该页没有，就接着去下一页找，前一页有指向后一页的指针。