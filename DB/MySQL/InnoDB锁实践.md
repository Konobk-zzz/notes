# InnoDB锁实践

## 建表语句

```sql
CREATE TABLE `test`.`practice`  (
  `id` int NOT NULL,
  `col` int NULL,
  PRIMARY KEY (`id`)
);
```

**本次实践中会话均设置 `autocommit = 0`，事务隔离级别为RR，MySQL版本为 5.6.29**

## 如何获取引擎运行信息

```sql
-- 展示InnoDB引擎状态
SHOW ENGINE INNODB STATUS;
-- 当前运行的所有事务
SELECT * FROM information_schema.INNODB_TRX;
-- 当前出现的锁
SELECT * FROM information_schema.InnoDB_LOCKS;
-- 锁等待的对应关系
SELECT * FROM information_schema.INNODB_LOCK_WAITS;
```



## 表锁

表级锁是 MySQL 锁中粒度最大的一种锁，表示当前的操作对整张表加锁，**资源开销比行锁少，不会出现死锁的情况，但是发生锁冲突的概率很大**。被大部分的mysql引擎支持，MyISAM和InnoDB都支持表级锁，但是InnoDB默认的是行级锁。

表锁由 MySQL Server 实现，一般在执行 DDL 语句时会对整个表进行加锁，比如说 ALTER TABLE 等操作。在执行 SQL 语句时，也可以明确指定对某个表进行加锁。

表锁使用的是一次性锁技术，也就是说，在会话开始的地方使用 lock 命令将后续需要用到的表都加上锁，在表释放前，只能访问这些加锁的表，不能访问其他表，直到最后通过 unlock tables 释放所有表锁。

除了使用 unlock tables 显示释放锁之外，会话持有其他表锁时执行lock table 语句会释放会话之前持有的锁；会话持有其他表锁时执行 start transaction 或者 begin 开启事务时，也会释放之前持有的锁。

**共享锁用法**：

```sql
LOCK TABLE table_name [ AS alias_name ] READ
复制代码
```

**排它锁用法**：

```sql
LOCK TABLE table_name [AS alias_name][ LOW_PRIORITY ] WRITE

复制代码
```

**解锁用法**：

```sql
unlock tables;
```



## 记录锁 和 意图锁

当前记录状态

```sql
mysql> select * from lock_practice;
Empty set
```

我们先在第一个会话中插入一条记录：

```sql
mysql> insert into lock_practice values (1,1);
Query OK, 1 row affected (0.02 sec)
```

然后在第二个会话也执行同样的语句，此时语句会被阻塞。

我们查询

```sql
mysql> SHOW ENGINE INNODB STATUS;
```

得到

```text
------------
TRANSACTIONS
------------
---TRANSACTION 2298900, ACTIVE 872 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 7, OS thread handle 0xcbac, query id 446 localhost ::1 root update
insert into lock_practice values (1,1)
------- TRX HAS BEEN WAITING 3 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298900 lock mode S locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000231413; asc    #  ;;
 2: len 7; hex 8e000001410110; asc     A  ;;
 3: len 4; hex 80000001; asc     ;;

------------------
---TRANSACTION 2298899, ACTIVE 1565 sec
2 lock struct(s), heap size 360, 1 row lock(s), undo log entries 1
MySQL thread id 6, OS thread handle 0xbb5c, query id 402 localhost ::1 root cleaning up
```

根据监视器的信息，我们可以看到`id=2298900`的事务正在等待`lock_practice`表`PRIMARY`索引上的行锁（非间隙锁）。会话一，在插入记录时，会先在表上加入IX锁，接着获取这个间隙的插入意向锁。此时会话二执行了相同的插入语句，同样能够获取到IX锁，因为不冲突嘛，接着他也获取了这个间隙的插入意向锁。在该行设置X记录锁之前检测到该页上存在活跃事务，帮助该事务建立锁对象。通过文档得知，如果发生重复键错误，则会在重复索引记录上设置共享锁，等待会话一结束。

此时，有一个死锁场景：如果再加入一个会话三，执行相同的插入语句。会话二、会话三 都会在该记录上加上S锁，然后等待 会话一 在该记录上的X锁释放。当会话一 回滚 释放锁时，会话二 会话三 都会请求该记录的X锁，此时就发生死锁了。

```text
------------
TRANSACTIONS
------------
---TRANSACTION 2298901, ACTIVE 45 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 10, OS thread handle 0x88c, query id 461 localhost ::1 root update
insert into lock_practice values (1,1)
------- TRX HAS BEEN WAITING 45 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298901 lock mode S locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000231413; asc    #  ;;
 2: len 7; hex 8e000001410110; asc     A  ;;
 3: len 4; hex 80000001; asc     ;;

------------------
---TRANSACTION 2298900, ACTIVE 3268 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1184, 1 row lock(s)
MySQL thread id 7, OS thread handle 0xcbac, query id 460 localhost ::1 root update
insert into lock_practice values (1,1)
------- TRX HAS BEEN WAITING 47 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298900 lock mode S locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000231413; asc    #  ;;
 2: len 7; hex 8e000001410110; asc     A  ;;
 3: len 4; hex 80000001; asc     ;;

------------------
---TRANSACTION 2298899, ACTIVE 3961 sec
2 lock struct(s), heap size 360, 1 row lock(s), undo log entries 1
MySQL thread id 6, OS thread handle 0xbb5c, query id 402 localhost ::1 root cleaning up
```

此时如果会话一提交事务，会话二、会话三都会提示：

```sql
1062 - Duplicate entry '1' for key 'PRIMARY'
```

并不会发生死锁。

但是如果会话一是回滚事务，会话二、会话三就会发生死锁，最终一个会话插入成功，另一个会话失败。

会话二：

```sql
mysql> insert into lock_practice values (1,1);
Query OK, 1 row affected (5.85 sec)
```

会话三：

```sql
mysql> insert into lock_practice values (1,1);
1213 - Deadlock found when trying to get lock; try restarting transaction
```

此时我们再来看看监视器：

```text
------------------------
LATEST DETECTED DEADLOCK
------------------------
2021-11-19 18:19:54 88c
*** (1) TRANSACTION:
TRANSACTION 2298912, ACTIVE 6 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 7, OS thread handle 0xcbac, query id 514 localhost ::1 root update
insert into lock_practice values (1,1)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298912 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) TRANSACTION:
TRANSACTION 2298913, ACTIVE 4 sec inserting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 10, OS thread handle 0x88c, query id 515 localhost ::1 root update
insert into lock_practice values (1,1)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298913 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 72 index `PRIMARY` of table `test`.`lock_practice` trx id 2298913 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** WE ROLL BACK TRANSACTION (2)
```

我们可以看到最近一次死锁记录已经将刚刚发生的死锁记录下来了。`id = 2298912`的事务想要请求 `lock_practice`表的`PRIMARY`索引上 48页第3号 上的X锁，可以看到 `id = 2298913` 的事务同样想要请求该位置上的X锁，同时他持有该位置上的S锁。`id = 2298912`的事务活动时间为6s，`id = 2298913`的事务活动时间为4s，于是InnoDB决定回滚占用资源最少的事务，也就是 `id = 2298913`的事务。

**这里请求需要四个锁，个人猜测是 IS S IX X 这四个锁，在最后的X锁发生了竞争。本条看法没有经过验证。**

## 猜想验证

### IS 和 表级X锁冲突而不是行级X锁冲突

会话一先执行

```sql
mysql> select * from lock_practice where id between 1 and 10;
+----+-------+
| id | col   |
+----+-------+
|  1 |   111 |
|  4 |   444 |
| 10 | 10111 |
+----+-------+
3 rows in set (0.11 sec)
```

会话二再执行

```mysql
mysql> insert into lock_practice values(6,666);
Query OK, 1 row affected (0.01 sec)
```

会话三再执行

```sql
mysql> lock tables lock_practice Write;
```

发生阻塞

```mysql
---TRANSACTION 2426535, ACTIVE 76 sec
1 lock struct(s), heap size 360, 0 row lock(s), undo log entries 1
MySQL thread id 5222, OS thread handle 0xf700, query id 338448 localhost ::1 root cleaning up
---TRANSACTION 2426143, ACTIVE 538 sec
MySQL thread id 5220, OS thread handle 0x12d18, query id 337467 localhost ::1 root cleaning up
Trx read view will not see trx with id >= 2426144, sees < 2426144
```

可以看到没有发生锁冲突，此时再来一个会话四执行

```mysql
mysql> insert into lock_practice values(6,666);
```

会发生阻塞，似乎在等待会话三的表锁加锁成功

```mysql
---TRANSACTION 2426535, not started
MySQL thread id 5222, OS thread handle 0xf700, query id 338968 localhost ::1 root Waiting for table metadata lock
insert into lock_practice values(6,666)
---TRANSACTION 2426143, ACTIVE 727 sec
MySQL thread id 5220, OS thread handle 0x12d18, query id 337467 localhost ::1 root cleaning up
Trx read view will not see trx with id >= 2426144, sees < 2426144
```

可以看到正在等待表的元数据锁。（这里的会话四实际上是会话二回滚了后再插入，所以会话四的id和会话二的id是一致的）

### 首次INSERT的时候只持有一个锁结构，冲突时会再获得一个锁结构

当前记录情况

```mysql
select * from lock_practice;
+-----+--------+
| id  | col    |
+-----+--------+
|   1 |    111 |
|   4 |    444 |
|  10 |  10111 |
| 100 | 100111 |
+-----+--------+
4 rows in set (0.15 sec)
```

会话一首先执行：

```mysql
mysql> insert into lock_practice values(2,222);
Query OK, 1 row affected (0.02 sec)
```

可以看到只持有一个锁结构：

```mysql
---TRANSACTION 2427548, ACTIVE 24 sec
1 lock struct(s), heap size 360, 0 row lock(s), undo log entries 1
MySQL thread id 5220, OS thread handle 0x12d18, query id 341276 localhost ::1 root cleaning up
```

会话二再执行：

```mysql
mysql> insert into lock_practice values(3,333);
Query OK, 1 row affected (0.03 sec)
```

看到各会话都持有一个锁结构：

```mysql
---TRANSACTION 2429103, ACTIVE 137 sec
1 lock struct(s), heap size 360, 0 row lock(s), undo log entries 1
MySQL thread id 5257, OS thread handle 0x7ee4, query id 345410 localhost ::1 root cleaning up
---TRANSACTION 2427548, ACTIVE 1980 sec
1 lock struct(s), heap size 360, 0 row lock(s), undo log entries 1
MySQL thread id 5220, OS thread handle 0x12d18, query id 341276 localhost ::1 root cleaning up
```

会话三执行：

```mysql
mysql> insert into lock_practice values(2,222);
```

会话三和会话一冲突，现在看到双方都持有两个锁结构

```mysql
---TRANSACTION 2430997, ACTIVE 4 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 5222, OS thread handle 0xf700, query id 350397 localhost ::1 root update
insert into lock_practice values(2,222)
------- TRX HAS BEEN WAITING 4 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 80 index `PRIMARY` of table `test`.`lock_practice` trx id 2430997 lock mode S locks rec but not gap waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 000000250a9c; asc    %  ;;
 2: len 7; hex b0000001660110; asc     f  ;;
 3: len 4; hex 800000de; asc     ;;

------------------
---TRANSACTION 2429103, ACTIVE 2280 sec
1 lock struct(s), heap size 360, 0 row lock(s), undo log entries 1
MySQL thread id 5257, OS thread handle 0x7ee4, query id 345410 localhost ::1 root cleaning up
---TRANSACTION 2427548, ACTIVE 4123 sec
2 lock struct(s), heap size 360, 1 row lock(s), undo log entries 1
MySQL thread id 5220, OS thread handle 0x12d18, query id 341276 localhost ::1 root cleaning up
```

insert插入之前会先获得记录间隙上的插入意向间隙锁。猜测：没有冲突时，插入完成只持有记录上的插入意向间隙锁。一旦发生冲突，再补上X记录锁。

**实验记录**

会话一：

```mysql
mysql> insert into lock_practice values(2,222);
Query OK, 1 row affected (0.00 sec)

mysql> insert into lock_practice values(3,333);
Query OK, 1 row affected (0.00 sec)
```

会话二：

```mysql
mysql> insert into lock_practice values(2,222);
```

锁状态：

```mysql
---TRANSACTION 2432196, ACTIVE 3 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 5257, OS thread handle 0x7ee4, query id 353685 localhost ::1 root update
insert into lock_practice values(2,222)
------- TRX HAS BEEN WAITING 3 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 80 index `PRIMARY` of table `test`.`lock_practice` trx id 2432196 lock mode S locks rec but not gap waiting
Record lock, heap no 7 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 000000251cbc; asc    %  ;;
 2: len 7; hex d4000001ba0110; asc        ;;
 3: len 4; hex 800000de; asc     ;;

------------------
---TRANSACTION 2432188, ACTIVE 10 sec
2 lock struct(s), heap size 360, 1 row lock(s), undo log entries 2
MySQL thread id 5220, OS thread handle 0x12d18, query id 353675 localhost ::1 root cleaning up
```

会话三：

```mysql
mysql> insert into lock_practice values(3,333);
```

锁状态：

```mysql
---TRANSACTION 2431822, ACTIVE 4 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 5222, OS thread handle 0xf700, query id 352649 localhost ::1 root update
insert into lock_practice values(3,333)
------- TRX HAS BEEN WAITING 4 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 80 index `PRIMARY` of table `test`.`lock_practice` trx id 2431822 lock mode S locks rec but not gap waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000003; asc     ;;
 1: len 6; hex 000000251ac1; asc    %  ;;
 2: len 7; hex fe000001a2011c; asc        ;;
 3: len 4; hex 8000014d; asc    M;;

------------------
---TRANSACTION 2431818, ACTIVE 10 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 360, 1 row lock(s)
MySQL thread id 5257, OS thread handle 0x7ee4, query id 352632 localhost ::1 root update
insert into lock_practice values(2,222)
------- TRX HAS BEEN WAITING 10 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 48 page no 3 n bits 80 index `PRIMARY` of table `test`.`lock_practice` trx id 2431818 lock mode S locks rec but not gap waiting
Record lock, heap no 7 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 000000251ac1; asc    %  ;;
 2: len 7; hex fe000001a20110; asc        ;;
 3: len 4; hex 800000de; asc     ;;

------------------
---TRANSACTION 2431681, ACTIVE 165 sec
2 lock struct(s), heap size 360, 2 row lock(s), undo log entries 2
MySQL thread id 5220, OS thread handle 0x12d18, query id 352260 localhost ::1 root cleaning up
```

会话二、会话三超时后锁状态：

```mysql
---TRANSACTION 2431822, ACTIVE 67 sec
1 lock struct(s), heap size 360, 0 row lock(s)
MySQL thread id 5222, OS thread handle 0xf700, query id 352649 localhost ::1 root cleaning up
---TRANSACTION 2431818, ACTIVE 73 sec
1 lock struct(s), heap size 360, 0 row lock(s)
MySQL thread id 5257, OS thread handle 0x7ee4, query id 352632 localhost ::1 root cleaning up
---TRANSACTION 2431681, ACTIVE 228 sec
2 lock struct(s), heap size 360, 2 row lock(s), undo log entries 2
MySQL thread id 5220, OS thread handle 0x12d18, query id 352260 localhost ::1 root cleaning up
```

会话二、会话三回滚后：

```mysql
---TRANSACTION 2431681, ACTIVE 274 sec
2 lock struct(s), heap size 360, 2 row lock(s), undo log entries 2
MySQL thread id 5220, OS thread handle 0x12d18, query id 352260 localhost ::1 root cleaning up
```

从上述的记录可以观测到的时，会话一插入完成时持有一个锁结构，但并没有锁定行记录，猜测只有插入意向锁。在产生冲突以后多了一个锁结构，并且锁定的行记录相应增加。在会话一后续插入的冲突会话超时、回滚以后锁并没有释放，表明事务过程中获取的锁会持续到事务结束才释放。

通过查阅资料发现insert实际上使用的是隐式锁

https://juejin.cn/post/7031754419167297549

查看源码会发现，插入操作的前后分别有一行代码：`mtr_start()` 和 `mtr_commit()`。这被称为 **迷你事务（mini-transaction）**。mini-transaction 也可以包含子事务，实际上在 `insert` 的执行过程中就会加多个 mini-transaction。

每个 mini-transaction 会遵守下面的几个规则：

- 修改一个页需要获得该页的 X-LATCH；
- 访问一个页需要获得该页的 S-LATCH 或 X-LATCH；
- 持有该页的 LATCH 直到修改或者访问该页的操作完成。

`insert`会在检查锁冲突和写数据之前，会对记录所在的页加一个 RW-X-LATCH 锁，执行完写数据之后再释放该锁（实际上写数据的操作就是写 redo log（重做日志），将脏页加入 flush list) 。这个锁的释放非常快，但是这个锁足以保证在插入数据的过程中其他事务无法访问记录所在的页。

所以最终RR级别下插入的流程为：

1. 对要操作的页加 `RW-X-LATCH` 
2. 判断是否有和插入意向锁冲突的锁，如果有加插入意向锁，进入锁等待；如果没有，进入步骤3；
3. 直接写数据，不加任何锁
4. 释放 `RW-X-LATCH`；

隐式锁的转换：

InnoDb 在插入记录时，是不加锁的。如果事务 A 插入记录且未提交，这时事务 B 尝试对这条记录加锁，事务 B 会先去判断记录上保存的事务 id 是否活跃，如果活跃的话，那么就帮助事务 A 去建立一个锁对象，然后自身进入等待事务 A 状态，这就是所谓的隐式锁转换为显式锁。
