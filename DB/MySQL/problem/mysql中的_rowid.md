# [mysql中的_rowid](https://segmentfault.com/a/1190000019067459)

## 前言

在`Oracle`数据库的表中的每一行数据都有一个唯一的标识符，称为`rowid`，在`Oracle`内部通常就是使用它来访问数据的。

而在`MySQL`中也有一个类似的隐藏列`_rowid`来标记唯一的标识。但是需要注意`_rowid`并不是一个真实存在的列，其本质是一个**非空唯一列**的别名。

*PS：本文是基于`MySQL 5.7`进行研究的*

## _rowid到底是什么

在前文提到了`_rowid`并不是一个真实存在的列，其本质是一个**非空唯一列**的别名。为什么会这么说呢？

因为在某些情况下`_rowid`是不存在的，其只存在于以下情况：

1. 当表中存在一个**数字类型**的单列主键时，`_rowid`其实就是指的是这个主键列
2. 当表中**不存在主键**但存在一个**数字类型**的**非空唯一列**时，`_rowid`其实就是指的是对应**非空唯一列**。

需要注意以下情况是不存在`_rowid`的

1. **主键列**或者**非空唯一列**的类型不是**数字类型**
2. **主键**是联合主键
3. **唯一**列不是非空的。

详情可以参考`MySQL`[官方文档](https://link.segmentfault.com/?url=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fcreate-index.html)内容：

> If a table has a PRIMARY KEY or UNIQUE NOT NULL index that consists of a single column that has an integer type, you can use **_rowid** to refer to the indexed column in SELECT statements, as follows:
>
> - **_rowid** refers to the PRIMARY KEY column if there is a PRIMARY KEY consisting of a single integer column. If there is a PRIMARY KEY but it does not consist of a single integer column, **_rowid** cannot be used.
> - Otherwise, **_rowid** refers to the column in the first UNIQUE NOT NULL index if that index consists of a single integer column. If the first UNIQUE NOT NULL index does not consist of a single integer column, **_rowid** cannot be used.

## 参考资料

1. [13.1.14 CREATE INDEX Syntax](https://link.segmentfault.com/?url=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fcreate-index.html)
2. [Re: Oracle ROWID equivalent in MySQL](https://link.segmentfault.com/?url=https%3A%2F%2Fforums.mysql.com%2Fread.php%3F61%2C368131%2C379277)