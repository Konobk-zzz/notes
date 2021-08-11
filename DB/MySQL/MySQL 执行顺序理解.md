# 在 MySQL 中如何执行查询？

这篇文章适用于对 MySQL 数据库有一些基本概念的测试人员。我想提出一些我在深入研究 MySQL 时遇到的事情。以下是在 MySQL 服务器内执行查询的顺序的简要概述。

------

执行 SQL 查询时，执行 SQL 指令的顺序是：

- FROM 子句
- WHERE 子句
- GROUP BY 子句
- HAVING 子句
- SELECT 子句
- ORDER BY 子句

但是，根据查询中指定的顺序，HAVING 和 GROUP BY 子句可以出现在 SELECT 之后。

数据库引擎在后端如何执行查询？让我们为每个子句举一个例子并理解其顺序。

```
SELECT  *  FROM order_details WHERE category =  'produce' ;
```

执行的第一个子句是 FROM 子句，用于列出查询所需的表和任何连接。正是通过这个子句，我们可以缩小可能的记录集大小。上面的查询是直接的，没有任何连接。

让我们再举一个在查询中使用 JOIN 的例子

```
选择order_details 。order_id ，客户。customer_name FROM customers INNER  JOIN order_details ON customers 。customer_id = order_details 。客户ID;
```

在上面的查询中，JOIN 条件在第一步中被评估。JOIN 操作的顺序由查询优化器在构建查询计划时动态确定。ON 条件是决定从每个表中连接哪些行的标准。FROM 子句的结果是一个临时结果（如临时表），由满足所有连接条件的组合行组成。在上面的例子中，MySQL 将返回来自 customers 和 order_details 表的所有行，其中在 customers 和 order_details 表中都有匹配的 customer_id 值。

接下来是 WHERE 子句。如果您没有在语句中指定 WHERE 子句，优化器将从临时结果中检索所有行。在带有 WHERE 子句的查询中，临时结果中的每一行都根据 WHERE 条件进行评估，并被丢弃或保留。

接下来是 GROUP BY 子句，它是 SELECT 语句的可选部分。如果有 GROUP BY 子句，则临时结果现在被分成多个组，GROUP BY 子句中的列中的每个值组合对应一个组。当您对表执行 GROUP BY 时，它将检索该组中的第一行。下面的 GROUP BY 示例使用 COUNT 函数返回产品和产品类别中的订单数量（针对该产品）。

```
SELECT product ,  COUNT ( * )  AS  "订单数量"   FROM order_details  WHERE category =  'produce'  GROUP  BY product;
```

现在是 HAVING 子句。HAVING 子句使您能够指定过滤哪些组结果出现在最终结果中的条件。它在每个组上运行一次，并消除组中不满足 HAVING 子句的所有行。在下面的查询中，在组装了整个临时结果表后，优化器将过滤结果，以便只返回订单数超过 20 的产品。HAVING 子句过滤完组后，会产生一个新的临时结果集，在这个新的临时结果中，每个组只有一行。

```
SELECT product ,  COUNT ( * )  AS  "订单数量"  FROM order_details WHERE category =  'produce'  GROUP  BY product HAVING  COUNT ( * )  >  20 ;
```

MySQL HAVING 子句与 GROUP BY 子句结合使用，以将返回的行组限制为仅那些条件为 TRUE 的行。

接下来是 SELECT 子句。从 GROUP BY 和 HAVING 子句产生的新临时结果的行中，SELECT 现在组装它需要的列。

最后，最后一步是 ORDER BY 子句。ORDER BY 子句用于对结果集中的记录进行排序。在同时使用 GROUP BY 和 ORDER BY 子句的查询中，您只能引用 ORDER BY 中的列，前提是它们位于分组过程生成的新临时结果中，即 GROUP BY 或聚合函数中的列。

```
SELECT  *  FROM  （SELECT产品， COUNT （* ） AS  “订单数量”  FROM ORDER_DETAILS WHERE supplier_name =  '微软'  GROUP  BY产品） AS temp_table ORDER  BY supplier_city DESC ;
```

在上面的示例中，将首先执行 GROUP BY，然后执行 ORDER BY 子句。在带有 GROUP BY 子句的 SELECT 中使用非聚合列是非标准的。MySQL 通常会返回它找到的第一行的值并丢弃其余的。任何 ORDER BY 子句仅适用于返回的列值，而不适用于丢弃的列值。

------

查询执行并没有那么复杂。MySQL 只是按照它的计划，按顺序从每个表中获取行并根据相关列进行连接。在此过程中，可能需要创建一个临时表来存储结果。一旦所有行都可用，它会将它们发送到客户端。