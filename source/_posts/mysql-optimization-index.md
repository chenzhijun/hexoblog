---
title: 数据库索引优化实例
copyright: true
date: 2018-03-23 14:47:25
tags: MySQL
categories: 数据库
---

# 数据库索引优化实例

索引对于查找小表或者需要处理表中大部分或全部行的大型表的查询不太重要，适合的才是最好的。

## 数据库索引的操作

1. 快速找到匹配where条件的行结果；
2. 如果有多个索引，MySQL通常会使用能检索除最少结果行的索引；
3. 如果有多列索引，优化器可以使用索引的最左边的前缀来查找行；比如有col1,col2,col3三列索引，那么你就可以使用(col1)，（col1,col2）,(col1,col2,col3)三个索引；
4. 如果有多表连接join的语句，mysql会将varchar(10)和char(10)看成同一个类型并且有效的使用索引；两个表使用的字符要一样比如utf8,和latin，就不行；
5. 在查找最小值和最大值的时候，优化器会先优化你是否在where条件中有常量值；
6. 索引排序会使用最左边的索引列前缀；
7. 在某些情况下，可以优化查询来检索值，而不必咨询数据行

## 主键与外键优化

主键索引的查询优化得益于主键不能为NULL值。表的数据在物理存储上就被组织为基于主键列或多列进行快速查找和排序。如果你的表很大又非常重要，然而却没有声明主键，或者组合主键，那么你需要新建一个新的列使用auto-increment来作为主键。

外键优化，如果有一个大表，有许多列，可以考虑将表的列进行拆分成小表，并且用外键关联。每个小表都有主键，这样可以让查找数据更加快速，而需要数据的时候也可以是使用join关联查询。由于数据是分散的，查询可以有更少的I/O,并且会使用更少的缓存。因为相关的列在磁盘中存储在一块。

## 组合索引

建立一个表：

```sql
CREATE TABLE test (
    id         INT NOT NULL,
    last_name  CHAR(30) NOT NULL,
    first_name CHAR(30) NOT NULL,
    PRIMARY KEY (id),
    INDEX name (last_name,first_name)
);
```

我们有组合索引name，在索引查询中使用最左列查询是会使用索引的。也就是如下面的列子：

```sql
SELECT * FROM test WHERE last_name='Widenius';

SELECT * FROM test
  WHERE last_name='Widenius' AND first_name='Michael';

SELECT * FROM test
  WHERE last_name='Widenius'
  AND (first_name='Michael' OR first_name='Monty');

SELECT * FROM test
  WHERE last_name='Widenius'
  AND first_name >='M' AND first_name < 'N';
```

即只要有最左侧的索引列，那么索引就会生效。但是如果是单独使用的`first_name`，那么索引将不会生效，如下面的示例：

<!--more-->

```sql
SELECT * FROM test WHERE first_name='Michael';

SELECT * FROM test
  WHERE last_name='Widenius' OR first_name='Michael';
```

如果是first_name,last_name两个列都建立索引，那么查询的时候如果两列都有建立单独的索引，那么优化器可能会使用能排除最大结果的那个索引。如果是联合索引，那么只要带有最左侧的索引列，那么优化器就会工作。比如如果有索引(col1,col2,col3)，下面的语句只会有前面两个可以用索引下面两个不会用到索引：

```sql
SELECT * FROM tbl_name WHERE col1=val1;
SELECT * FROM tbl_name WHERE col1=val1 AND col2=val2;

SELECT * FROM tbl_name WHERE col2=val2;
SELECT * FROM tbl_name WHERE col2=val2 AND col3=val3;
```

### 索引扩展的使用

如果表的主键是一个组合索引，并且还声明了另一个索引。那么innodb会自动建立一个次索引，这个次索引包含了两个组合主键。

```sql
CREATE TABLE t1 (
  i1 INT NOT NULL DEFAULT 0,
  i2 INT NOT NULL DEFAULT 0,
  d DATE DEFAULT NULL,
  PRIMARY KEY (i1, i2),
  INDEX k_d (d)
) ENGINE = InnoDB;
```

主键是一个组合主键，并且还有一个索引`k_d`；数据库还会在内部生成一个次索引（d,i1,i2）；优化器可以扩展次索引为ref,range,index_merge，可以在join和排序优化，和最大最小值优化；
The optimizer can use extended secondary indexes for ref, range, and index_merge index access, for loose index scans, for join and sorting optimization, and for MIN()/MAX() optimization.

考虑下面一个示例：

```sql
INSERT INTO t1 VALUES
(1, 1, '1998-01-01'), (1, 2, '1999-01-01'),
(1, 3, '2000-01-01'), (1, 4, '2001-01-01'),
(1, 5, '2002-01-01'), (2, 1, '1998-01-01'),
(2, 2, '1999-01-01'), (2, 3, '2000-01-01'),
(2, 4, '2001-01-01'), (2, 5, '2002-01-01'),
(3, 1, '1998-01-01'), (3, 2, '1999-01-01'),
(3, 3, '2000-01-01'), (3, 4, '2001-01-01'),
(3, 5, '2002-01-01'), (4, 1, '1998-01-01'),
(4, 2, '1999-01-01'), (4, 3, '2000-01-01'),
(4, 4, '2001-01-01'), (4, 5, '2002-01-01'),
(5, 1, '1998-01-01'), (5, 2, '1999-01-01'),
(5, 3, '2000-01-01'), (5, 4, '2001-01-01'),
(5, 5, '2002-01-01');
```

执行执行计划：

```sql
EXPLAIN SELECT COUNT(*) FROM t1 WHERE i1 = 3 AND d = '2000-01-01'
```

![2018-03-23-18-54-12](/images/qiniu/2018-03-23-18-54-12.png)


如果是多表查询就像下面这样：

![2018-03-23-19-30-04](/images/qiniu/2018-03-23-19-30-04.png)

如果要关闭优化器：

```sql
SET optimizer_switch = 'use_index_extensions=off';
```

### 执行计划 - EXPLAIN

`EXPLAIN`语句提供你MySQL是怎么执行语句的过程。EXPLAIN可以在`SELECT`,`DELETE`,`INSERT`,`REPLACE`,`UPDATE`语句上使用，返回他们的执行计划。也就是这些语句是按照何种方式执行会返回表记录。EXPLAIN的执行计划可以使用`SHOW WARNINGS`来返回更多的扩展信息。

#### 执行计划输出的列信息

|      列名       |                    意义                    |
| :-----------: | :--------------------------------------: |
|      id       |                SELECT标识码                 |
|  select_type  |                 SELECT类型                 |
|     table     |                    表明                    |
|  partitions   |                   匹配分区                   |
|     type      |            获取类型（access_type）             |
| possible_keys |                  可选择的索引                  |
|      key      |              实际执行语句时候选择的索引               |
|    key_len    |                选择的key的长度                 |
|      ref      |    The columns compared to the index     |
|     rows      |               预计会有多少行被查询到                |
|   filtered    | Percentage of rows filtered by table condition |
|     extra     |                   扩展信息                   |

其中`select_type`可以是下面这些值：

|    select_type 值     |                    意义                    |
| :------------------: | :--------------------------------------: |
|        SIMPLE        |           简单查询（不使用union或者子查询）            |
|       PRIMARY        |                 最常用的查询类型                 |
|        UNION         | 联合查询(Second or later SELECT statement in a UNION) |
|   DEPENDENT_UNION    | 依赖外部的联合查询（Second or later SELECT statement in a UNION, dependent on outer query） |
|     UNION RESULT     |                 UNION的结果                 |
|       SUBQUERY       |                   子查询                    |
|  DEPENDENT SUBQUERY  |                 依赖外部的子查询                 |
|       DERIVED        | Derived table SELECT (subquery in FROM clause) |
|     MATERIALIZED     |          Materialized subquery           |
| UNCACHEABLE SUBQUERY | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
|  UNCACHEABLE UNION   | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |

其中`type` 可以是下面这些值，如果执行计划是全表扫描，那么最好是进行优化，以下表格从最优到最慢排序：

|      type值      |                    意义                    |
| :-------------: | :--------------------------------------: |
|     system      |         表只有一行数据（系统表），特殊的const类型          |
|      const      |      表查询出来的结果最多只有一行，适用于主键查询和唯一索引查询       |
|     eq_ref      | 唯一索引行，常用于join查询中主键查询和非null唯一索引。第二个表只会匹配第一个表中的一行数据，一对一的关系 |
|       ref       |       一对多关系，前一个表的一行数据可以匹配另一个表的多行数据       |
|    fulltext     |                  全文本扫描                   |
|   ref_or_null   |             类似ref，不过会处理null值             |
|   index_merge   | This join type indicates that the Index Merge optimization is used. In this case, the `key` column in the output row contains a list of indexes used, and `key_len` contains a list of the longest key parts for the indexes used. |
| unique_subquery | 替换eq_ref在in的子查询中的类型（value IN (SELECT primary_key FROM single_table WHERE some_expr)） |
| index_subquery  | 类似unique_subquery，替换在in查询中，但是它是使用在非唯一索引的子查询中(value IN (SELECT key_column FROM single_table WHERE some_expr)) |
|      range      | 范围搜索，使用索引查找行结果。在这个查询中key列是指那个索引被用到，key_len是指被用到的最长的key。ref列是NULL |
|      index      | 同ALL一样会扫描全表不过是以索引扫描。在这个类型中只有单索引会出现这种类型。另外注意两点注意:1.If the index is a covering index for the queries and can be used to satisfy all data required from the table, only the index tree is scanned. In this case, the Extra column says Using index. An index-only scan usually is faster than ALL because the size of the index usually is smaller than the table data. 2. A full table scan is performed using reads from the index to look up data rows in index order. `Uses index` does not appear in the `Extra` column |
|       ALL       |                全表扫描，需要避免                 |

其中`ref`的含义为：

ref列显示哪些列或常量与key列中命名的索引相比较，以便从表中选择数据。

## 实例

外部银行卡表，除了追歼，没有其它索引。

![2018-03-23-20-20-21](/images/qiniu/2018-03-23-20-20-21.png)

第一次使用查询语句执行计划

![2018-03-23-20-22-25](/images/qiniu/2018-03-23-20-22-25.png)

增加索引：

```sql
ALTER TABLE `cncsen`.`bank_card_info` 
ADD UNIQUE INDEX `card_no_UNIQUE` (`card_no` ASC);
```

![2018-03-23-20-23-59](/images/qiniu/2018-03-23-20-23-59.png)

再次执行刚刚的执行计划：

![2018-03-23-20-25-31](/images/qiniu/2018-03-23-20-25-31.png)

加入索引后很明显我们由全表扫描变成了范围扫描，试想如果该表数据很大，那么这种优化是有意义的。所以Explain还是很有作用的

## 参考文档

[How MySQL Uses Indexes](https://dev.mysql.com/doc/refman/5.7/en/mysql-indexes.html)

[EXPLAIN Output Format](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)