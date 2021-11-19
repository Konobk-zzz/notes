# InnoDB锁实践

## 建表语句

```sql
CREATE TABLE `test`.`practice`  (
  `id` int NOT NULL,
  `col` int NULL,
  PRIMARY KEY (`id`)
);
```

**本次实践中会话均设置 `autocommit = 0`**



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

根据监视器的信息，我们可以看到`id=2298900`的事务正在等待`lock_practice`表`PRIMARY`索引上的行锁（非间隙锁）。会话一，在插入记录时，会先在表上加入IX锁，接着获取了插入的这行的X记录锁。此时会话二执行了相同的插入语句，同样能够获取到IX锁，因为不冲突嘛，接着他理应在该行设置X记录锁，但是可以从上面的日志中看到会话二请求的是S锁。通过文档得知，如果发生重复键错误，则会在重复索引记录上设置共享锁（疑惑1：InnoDB在加锁前识别到了重复，个人猜测可能是因为在插入的位置识别到了会话一插入的记录，加锁只是为了插入后保护该记录不被其他会话修改）。

**这里会话二请求的两个锁可能是 IX S。被观点没有经过验证。**理由是INSERT 会先请求IX锁，然后发现冲突了会在记录上放置S锁，但是无法解释此时放置S锁为什么没有请求IS锁。难道是发现冲突之后放弃IX锁了，转而请求IS S锁。

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

