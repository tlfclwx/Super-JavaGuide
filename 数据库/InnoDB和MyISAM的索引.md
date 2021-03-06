# InnoDB和MyISAM索引
![](https://gitee.com/super-jimwang/img/raw/master/img/20210221154039.png)

## InnoDB和MyISAM所使用的索引
两种引擎都是使用B+树。

MyISAM引擎，B+树的数据结构中存储的内容实际上是实际数据的地址值。也就是说它的索引和实际数据是分开的，只不过使用索引指向了实际数据。这种索引的模式被称为非聚集索引。

Innodb引擎的索引的数据结构也是B+树，只不过数据结构中存储的都是实际的数据，这种索引有被称为聚集索引。

### MyISAM的索引实现原理
![](https://gitee.com/super-jimwang/img/raw/master/img/20210221165602.png)
![](https://gitee.com/super-jimwang/img/raw/master/img/20210221165528.png)
==**对于非聚簇索引，不管是主索引还是辅助索引，都只需要检索一遍就行了**==

MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。 


### InnoDB的索引实现原理
![](https://gitee.com/super-jimwang/img/raw/master/img/20210221165234.png)
虽然InnoDB也使用B+Tree作为索引结构，但具体实现方式却与MyISAM截然不同。

第一个重大区别是InnoDB的数据文件本身就是索引文件。从上文知道，MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。而在InnoDB中，==**表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录**==。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。

上图是InnoDB主索引（同时也是数据文件）的示意图，可以看到叶节点包含了完整的数据记录。这种索引叫做聚集索引。因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

第二个与MyISAM索引的不同是InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，==**InnoDB的所有辅助索引都引用主键作为data域**==。例如，下图为定义在Col3上的一个辅助索引：

![](https://gitee.com/super-jimwang/img/raw/master/img/20210221165350.png)

聚集索引这种实现方式使得按主键的搜索十分高效，但是 ==**辅助索引（也就是非主键建立的索引）搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。**==


==**为什么辅助索引记录的是主键，而不是一整行的数据，或者指向主键的指针？**==
- 不记录一整行数据是为了节省空间。
- 而不使用指针是因为当行移动时，不用维护辅助索引。


### 相关问题
> 为什么innodb不建议使用过长的字段作为主键？

因为主键会作为辅助索引的存储叶子节点，所以太长会导致占用空间大

> 为什么innodb建议使用单调的主键

因为索引是一颗B+树，非单调的在插入记录时会发生频繁的调整。

写入位置：如果不自增，需要调整叶子节点中每一页的数据顺序，就跟数组插入一样，改数据后面的数据都需要往后移动。