# drop、truncate、delete区别
- drop 直接删除表；
- truncate 删除表中数据，再插入时自增长id又从1开始 ；
- delete 删除表中数据，可以加where字句。
 
drop table：
- 属于DDL（Data Definition Language，数据库定义语言）
- 不可回滚
- 不可带 where
- 删除表内容和结构
- 删除速度快


truncate table：
- 属于DDL（Data Definition Language，数据库定义语言）
- 不可回滚
- 不可带 where
- 删除表内容
- 删除速度快
 
delete from：
- 属于DML
- 可回滚
- 可带where
- 表结构在，表内容要看where执行的情况
- 删除速度慢,需要逐行删除
 
使用简要说明：
- 不再需要一张表的时候，用drop
- 想删除部分数据行时候，用delete，并且带上where子句
- 保留表而删除所有数据的时候用truncate


---
部分转载自：https://www.yuque.com/fanzhengxu/tba6b8/dx0hvw#oWBVa