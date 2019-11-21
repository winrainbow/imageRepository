# MySql 笔记[锁]

**数据库锁初衷：处理并发问题**

根据加锁的范围，MySQL的锁大致分为：

1. 全局锁
2. 表级锁
3. 行锁

## 全局锁

*3个问题：是什么？为什么？实现？*

> 全局锁是对整个数据库实例加锁，加锁命令：Flush tables with read lock(FTWRL)，加锁后整个数据库是只读状态，之后其他线程的以下语句会被阻塞：更新语句，数据定义语句（建表、修改表结构等）和更新事务的提交语句
>
> 取消锁的命令：unlock tables;

使用场景：整个库做备份

MySQL官方自带的逻辑备份工具是mysqldump,当mysqldump使用参数-single-transaction时，导出数据之前就会启动一个事务，来确保存拿到一致性视图，由于MVCC（多版本并发控制）的支持，这个过程中的数据是可以正常更新的。

为什么需要FTWRL?

如果存储引擎不支持事务引擎，就破坏了备份的一致性，这时就需要FTWRL命令；所以，single-transaction 方法只适用于所有的表使用事物引擎的库。

为什么不使用 set global readonly = true?

1. 修改global变量影响范围更大：如在有些系统中，readonly的值会被用来做其他逻辑（判断一个库是主库还是从库）
2. 异常处理机制不同：执行FTWRL命令后如果连接异常断开，MySQL会自动释放锁，整个库可以正常更新；而设置readonly后，必须重新设置为false，状态才会更新

## 表级锁

> 分两种：表锁、元数据锁（meta data lock,MDL）

### 表锁：针对某个表进行锁定只读，或者锁定只写，

命令：lock tables [t1] read, [t2] write; 使用unlock tables 解除

注：锁住的是其他线程，也限定了本线程

write:本线程可读可写，其他线程不可读写

read:本线程可读不可写，共他线程可读不可写

如：线程1（连接1）执行 lock tables account read;

线程1可执行的操作：

	1. select * from account;

线程2可执行的操作：

	1. select * from account;
 	2. update ,insert操作会阻塞，直到线程1 unlock tables; 后才会有执行结果；

使用场景：在未出现更细粒度的锁之前，表锁是最常用的处理并发的方式，而对于InnoDB这种支持行锁的引擎，一般不使用lock tables命令来控制并发，因锁住整个表的影响面太大

### 元数据锁（metaDataLock MDL）

> 不需要显示使用，在访问一个表时会被 自动加上，在MySQL5.5版本引入MDL,当对一个表进行增删改查时，加入MDL读锁，当要对表结构变更时，加入MDL写锁

注：事务中的MDL锁是在语句开始执行时申请，但是在语句结束后并不会马上释放，而会等到整个事务提交后再释放

### 如何给小表加字段：

1. 解决长事务问题，如果有长事务在执行，则先暂停DDL或者kill掉事务
2. 如果这个表查询频繁，kill掉未必好用，因新的请求马上就来了，这时可以先执行 alter table 

```sql
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```

## 行锁

> 锁住表中的某一行，当一个事务在更新一行时，如果另一个事务也要更新同一行，需等待事务A的操作执行完才可以

| 事务A                                                        | 事务B                                         |
| ------------------------------------------------------------ | --------------------------------------------- |
| begin:<br />update t set k=k+1 where id=1;<br />update t set k=k+1 where id =2 |                                               |
|                                                              | begin:<br />update t set k = k+2 where id =1; |
| commit;                                                      |                                               |

**行锁是在需要时加上，但是在事务结束时才释放，而不是不需要时 立刻释放**

事务中的语句顺序合理，可以减少行锁的占用时间，从而提升并发度

## 死锁和死锁检测

| 事务A                                     | 事务B                          |
| ----------------------------------------- | ------------------------------ |
| begin:<br/>update t set k=k+1 where id=1; | begin:                         |
|                                           | update t set k=k+1 where id=2; |
| update t set k=k+1 where id=2;            |                                |
|                                           | update t set k=k+1 where id=1; |

其中：事务A在等待事务B释放id=2的行锁，事务B在等待事务A释放id=1的行锁，从而死锁

解决死锁方式：

1. 设置超时，如果超时则事务回滚  innodb_lock_wait_timeout，默认50s,太短会误伤，太长会影响并发
2. 发起死锁检测：如果有死锁，则回滚其中的一个事务，其他的事务得以执行，将参数 innodb_deadlock_detect 设置为 on

正常情况使用第二种方式，而且 innodb_deadlock_detect默认是打开的

存在的问题：每当一个事务被锁时，就要看看它所依赖的线程有没有被别人锁住，如此循环，最后判断是否出现了循环等待。如果所有事务更新同一行，则会有性能问题。

解决上述方式：1，关闭死锁检测（确定不会出现死锁）2，控制并发，在数据库服务端作。



**两阶段锁：加锁阶段和解锁阶段，解锁阶段不可以在执行加锁 ** 





