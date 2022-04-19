---
title: mysql学习
tags: 
date: 2022-04-17 23:30:00
categories: 学习总结
---

记录本周学习过的关于 MySQL 的面试题

## 一、MySQL的执行计划

执行计划：一条sql语句的执行过程

sql语句前加上 `explain`

通过 explain 命令获取 select 语句的执行计划,可以知道以下信息：

- 表的读取顺序，
- 数据读取操作的类型，
- 哪些索引可以使用，
- 哪些索引实际使用了，
- 表之间的引用，
- 每张表有多少行被优化器查询等信息。

## 二、索引是什么？

mysql组成：
![aa323f68e1024c8e5928cac960af795c.png](https://runcoderhang.github.io/thumbnails/d3db892942604dd4898d281adedd8677.png)
其中存储引擎包括：InnoDB（默认） 、MyISAM

在mysql文件中：

> DataBaseTable.frm
> DataBaseTable.ibd :存储索引和数据（Innodb存储引擎）
> DataBaseTable.MYD :存储的数据（MyISAM存储引擎）
> DataBaseTable.MYI :索引

InnoDB引擎特性：

- 插入缓冲（Insert Buffer）
- 两次写（Double Write）
- 自适应哈希索引（Adaptive Hash Index）
- 预读（Read Ahead）
    - 线性预读（linear read-ahead）
    - 随机预读（randomread-ahead）

**索引是什么？**

索引是为了加速对表中数据行的检索而创建的一种分散的存储结构。索引是针对表而建立的，它是由数据页面以外的索引页面组成的，每个索引页面中的行都会含有逻辑指针，以便加速检索物理数据。

> - 索引 --> 用途 --> 提高查询效率（索引和实际的数据都是存储在磁盘的，只不过在进行数据读取的时候会优先把索引加载到内存中）

索引存储数据的格式

> - 数据存储格式 --\> K-V --> 数据结构 --> hash表 --> B+树 --> 为什么是B+树
>     
> - 面试问题：给8G/16G的文件 --> 分块读取 --> 分而治之
>     
> - IO问题 --> 减少io量 / 减少io次数
>     
> 
> 操作系统
> 
> - 局部性原理：
>     - 时间局部性：之前被访问过的数据很有可能再次被访问
>     - 空间局部性：数据和程序都有聚集成群的倾向
> - 磁盘预读：
>     内存跟磁盘在进行交互的时候有一个最小的逻辑单位，这个单位称之为页，或者datapage，一般是4k或者8k，由操作系统决定。我们在进行数据读取的时候，一般会读取页的整数倍，也就是 4k，8k，16k，innodb存储引擎在进行数据加载的时候读取的是16kb的数据
> 
> **注意** 上面的分块读取就是这里的 页

## 三、为什么选择B+树？

> - hash表
>     ![97d304726b4cc7c080cb817d07213394.png](https://runcoderhang.github.io/thumbnails/6ac995032b534d70b67348de7460a5e7.png)
>     需要比较好的hash算法，如果算法不好的话，会导致hash碰撞，hash冲突，导致数据散列不均匀
>     当需要进行范围查找的时候需要挨个遍历，效率比较低
> 
> memory的存储引擎支持的就是hash索引，同时注意innodb存储引擎支持自适应hash（即mysql的server会判断数据存储时使用hash还是B+树 ，这是人为干预不了的）
> 
二叉树 、 BST(二叉搜索树) 、 AVL 、 红黑树
![2dcbc7ed308bc0e4a8d1cf6039d56e1f.png](https://runcoderhang.github.io/thumbnails/38eaeb5c196b4aadbec845df09341c11.png)
这些树共同的劣势：当需要向这些树中插入更多的数据的时候，会导致当前树变得非常高，加大读取的次数，影响查询的效率（归根原因是仅有两个节点）
    
B树：
![fb8ae6d9aaf0684108728dffe32df19e.png](https://runcoderhang.github.io/thumbnails/f6e29480d0824a358340adbbb823698d.png)
![26477ed327186f2ee24a34492fcc4418.png](https://runcoderhang.github.io/thumbnails/445da1b87e444c6180a1eae314bdc6f4.png)
![f59aaf20b03780cd85a8ff69a1507d2d.png](https://runcoderhang.github.io/thumbnails/8a6de21bdbd64ddfa08330252436018a.png)
B树劣势：
- 每个节点都有key，同时也包含data，而每个页存储空间是有限的，如果data比较大的话会导致每个节点存储的key数量变小
- 当存储的数据量很大的时候会导致深度较大，增大查询时磁盘io次数，进而影响查询的性能

B+树：
![3c3d79491d0d1aa3b202a750c1a38849.png](https://runcoderhang.github.io/thumbnails/467bfea0073a4526a0c48d3078d9a1b0.png)
![2b37977cd3ee235d3c59a8e174f85830.png](https://runcoderhang.github.io/thumbnails/05136411325a4b41a430a71d073351a7.png)
变化如下：
1. B+ Tree每个节点可以包含更多的几点，这么做的原因有两个：
    - 为了降低树的高度
    - 将数据范围变为多个区间，区间越多，数据检索越快
2. 非叶子节点存储key，叶子节点存储key和数据
3. 叶子节点两两指针互相连接（符合磁盘的预读特性），顺序查询性能更高

注意：在 B+ Tree 上有两个头指针，一个指向根结点，另一个指向关键字最小的叶子节点，而且所有叶子节点（即数据节点）之间是一种链式环结构。因此可以对 B+ Tree 进行两种查找运算：
1. 一种是对于主键的范围查找和分页查找
2. 另一种是从根节点开始，进行随机查找

**一般情况下 3到4层的 B+ Tree 足以支持千万级别的数据存储。**

选择索引的时候，选择 `int` 还是 `varchar` ？ int占4字节，varchar是根据（）中自己定义，小于 4 字节选择 varchar，大于 4 字节，选择 int
 
在进行索引设计的时候，尽可能让 key 占用较少的存储空间。例如让某一列的前一部分充当我们的索引（前缀索引）


索引的选择性：例如选择截取前几个字符作为我们的索引

```sql
SELECT COUNT(*) AS cnt, city FROM citydemo GROUP BY city ORDER BY cnt DESC LIMIT 10;

# 截取前三个字符
SELECT COUNT(*) AS cnt, LEFT(city, 3) AS pref FROM citydemo GROUP BY pref ORDER BY cnt DESC LIMIT 10;

# 截取前五个字符索引
SELECT COUNT(*) AS cnt, LEFT(city, 5) AS pref FROM citydemo GROUP BY pref ORDER BY cnt DESC LIMIT 10;
```


## 四、聚簇索引和非聚簇索引

数据跟索引存储在一起的叫做聚簇索引，没有存储在一起的叫做非聚簇索引

InnoDB存储引擎在进行数据插入的时候，数据必须要跟某一个索引列存储在一起，这个索引列可以是主键，如果没有主键，选择唯一键，如果没有唯一键，选择6字节的rowid来进行存储

数据必定是跟某一个索引绑定在一起的，绑定数据的索引叫做聚簇索引。其他索引的叶子节点中存储的数据不再是整行的记录，而是聚簇索引的id值。

> 举例：
> Table: id(主键), name(普通索引), age gender
> id是聚簇索引，name对应的索引的B+树上的叶子节点存储的就是id值

InnoDB中既有聚簇索引，也有非聚簇索引 (.ibd)
MyISAM中只有非聚簇索引 (.MYD & .MYI)

- 拓展：

InnoDB: **主键索引就是聚簇索引，只能有一个聚簇索引，但是可以有很多非聚簇索引**

> ```sql
> CREATE TABLE table(id(主键), name(普通索引), age, gender);
> ```
> InnoDB存储引擎在进行数据插入的时候，数据必须要跟某一个索引列存储在一起，这个索引列可以是主键，如果没有主键，选择唯一键，如果没有唯一键，选择6字节的rowid来进行存储

此时，插入数据后 InnoDB 上面是索引，下面是数据。因为聚簇索引数据跟索引放到一起，如果有多个聚簇索引就会产生数据冗余。

![8aed97938f1e7aa2c0bcde9cda4d8026.png](https://runcoderhang.github.io/thumbnails/2870cd4237214157b4fc3fc5701458e5.png)

> 假设再给 name 字段加个索引，就是非聚簇索引，那么存储方式如下：

![09ed3019dcee775d9594fe29fd75d946.png](https://runcoderhang.github.io/thumbnails/b98d73a737c243fa96dfc207f7ba0d42.png)

此时，name 下面就不能存放数据，会造成数据冗余。 **应该放数据所在索引的 key 的值，所有非聚簇索引都指向聚簇索引，不然找不到数据。**


## 五、回表、索引覆盖、

### 回表：

```sql
CREATE TABLE table(id(主键), name(普通索引), age, gender)
SELECT * FROM table WHERE name='zhangsan';
```

检索过程：先根据 name B+树匹配到对应的叶子节点，查询到对应记录的id值，再根据id去id的B+ 数中检索整行记录，这个过程就称之为 **回表** 。

由此可见回表效率低，要尽量避免回表操作（实际是基于非主键索引的查询需要多扫描一棵索引树， 在应用中应该尽量使用主键查询）


### 索引覆盖：

常见的方法是：将被查询的字段，建立到联合索引里去。

```sql
CREATE TABLE table(id(主键), name(普通索引), age, gender)
SELECT id,name FROM table WHERE name='zhangsan';  # 索引覆盖

SELECT id,name,gender, FROM table WHERE name='zhangsan';  # 需要回表
```

检索过程：根据 name 的值去 name B+树检索对应的记录，能获取到id的属性值，索引的叶子节点中包含了查询的所有列，此时不需要回表，这个过程叫做 **索引覆盖** ，`Extra` 字段为 `using index` 的提示信息（ **推荐使用** ）

在某些场景中，可以考虑将要查询的所有列都变成组合索引，此时会使用索引覆盖，加快查询效率。

> **注意** 第二个 sql 语句需要回表。

检索过程：能够根据 name 值去 name B+树获取到 id 的属性值。但索引的叶子节点存储了主键 id，没有存储 gender，gender 字段必须回表查询才能获取到，不符合索引覆盖。

但如果吧 `name` 单列索引升级为联合索引 `(name, gender)` 

```sql
CREATE TABLE table1(id, name, sex, index(name, sex))ENGINE=innodb;
SELECT id, name, gender FROM table1 WHERE name='zhangsan';
```
单列索引升级为联合索引 `(name, gender)` 后，索引叶子节点存储了主键 id, name, gender 都能够命中索引覆盖，无需回表。


### 最左匹配

创建索引的时候可以选择多个列来共同组成索引，此时叫做组合索引或者联合索引。要遵循最左匹配原则

```sql
CREATE TABLE user(id(主键), index(name, age), gender);
SELECT * FROM user WHERE name='zhangsan' AND age=12;  # 可以利用组合索引
SELECT * FROM user WHERE name='zhangsan';  # 可以
SELECT * FROM user WHERE age=12;  # 不可以
SELECT * FROM user WHERE age=12 AND name='zhangsan';  # 可以组合索引，mysql内有查询优化器可以调整

# 再扩展
CREATE TABLE user(id(主键), index(name, age, gender));
SELECT * FROM user WHERE name='zhangsan' AND age=12 AND gender='男';  # 可以利用组合索引
SELECT * FROM user WHERE name='zhangsan' AND age=12 ;  # 可以
SELECT * FROM user WHERE age=12 AND name='zhangsan';  # 可以组合索引，mysql内有查询优化器可以调整
SELECT * FROM user WHERE name='zhangsan'; # 可以

SELECT * FROM user WHERE age=12 AND gender='男'; # 不可以
SELECT * FROM user WHERE age=12;  # 不可以
SELECT * FROM user WHERE name='zhangsan' AND gender='男'; # 可以利用组合作用，但只用上 name索引，age gender 索引用不到
````

**注意** ：当遇到范围查询(>、<、between、like)就会停止匹配
```sql
SELECT * FROM user WHERE name='zhangsan' AND age>12 AND gender='男'; # name，age可以用到组合索引，但 gender 用不到
```

索引的底层是一颗 B+ 树，那么联合索引当然还是一颗 B+ 树，只不过联合索引的健值数量不是一个，而是多个。构建一颗 B+ 树只能根据一个值来构建，因此数据库依据联合索引最左的字段来构建 B+ 树。

> 形如(a,b,c)联合索引的 b+ 树, 非叶子节点存储的是第一个关键字的索引 a，而叶子节点存储的是三个关键字的数据。这里可以看出 a 是有序的，而 b，c 都是无序的。但是当在 a 相同的时候，b 是有序的，b 相同的时候，c 又是有序的。
> 
> 当查询到 b 的值以后（这是一个范围值），c 是无序的。所以就不能根据联合索引来确定到低该取哪一行。


### 索引下推

```sql
SELECT * FROM user WHERE name='zhangsan' AND age=12;
```

上面语句在没有索引下推之前：先根据name从存储引擎中拉取数据到server层，然后在server层中对age进行数据过滤

有了索引下推之后：根据name和age两个条件来做数据筛选，将筛选之后的结果返回给server层

> 索引下推：是把原来在server层进行的条件过滤下推到了存储引擎层

## 如何做SQL优化？

1. 最大化利用索引
2. 尽可能避免全表扫描
3. 减少无效数据的查询

**避免不走索引的场景**

1. 尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描。
2. 尽量避免使用in 和not in，会导致引擎走全表扫描。如下：
3. 尽量避免使用 or，会导致数据库引擎放弃索引进行全表扫描。如下：
4. 尽量避免在where条件中等号的左侧进行表达式、函数操作，会导致数据库引擎放弃索引进行全表扫描。
5. 当数据量大时，避免使用where 1=1的条件。通常为了方便拼装查询条件，我们会默认使用该条件，数据库引擎会放弃索引进行全表扫描。如下：
6. 查询条件不能用 <> 或者 !=

**SELECT语句其他优化**

1. 避免出现select *，需要哪些字段就返回哪些字段。
2. 多表关联查询时，小表在前，大表在后。
    在MySQL中，执行 from 后的表关联查询是从左往右执行的（Oracle相反），第一张表会涉及到全表扫描，所以将小表放在前面，先扫小表，扫描快效率较高，再扫描后面的大表，或许只扫描大表的前100行就符合返回条件并return了。
3. 使用表的别名
    当在SQL语句中连接多个表时，请使用表的别名并把别名前缀于每个列名上。这样就可以减少解析的时间并减少哪些由于列名歧义引起的语法错误。
4. 用 where 字句替换 HAVING 字句
    避免使用HAVING字句，因为HAVING只会在检索出所有记录之后才对结果集进行过滤，而where则是在聚合前筛选记录，如果能通过where字句限制记录的数目，那就能减少这方面的开销。HAVING中的条件一般用于聚合函数的过滤，除此之外，应该将条件写在where字句中。
    where和having的区别：where后面不能使用组函数
5. 调整Where字句中的连接顺序
    MySQL采用从左往右，自上而下的顺序解析where子句。根据这个原理，应将过滤数据多的条件往前放，最快速度缩小结果集。


**查询条件优化**

1. 对于复杂的查询，可以使用中间临时表 暂存数据；
2. 优化group by语句
    默认情况下，MySQL 会对GROUP BY分组的所有值进行排序，如 “GROUP BY col1，col2，....;” 查询的方法如同在查询中指定 “ORDER BY col1，col2，...;” 如果显式包括一个包含相同的列的 ORDER BY子句，MySQL 可以毫不减速地对它进行优化，尽管仍然进行排序。
    因此，如果查询包括 GROUP BY 但你并不想对分组的值进行排序，你可以指定 ORDER BY NULL禁止排序
3. 优化union查询
    MySQL通过创建并填充临时表的方式来执行union查询。
    除非确实要消除重复的行，否则建议使用union all。
    原因在于如果没有all这个关键词，MySQL会给临时表加上distinct选项，这会导致对整个临时表的数据做唯一性校验，这样做的消耗相当高。
4. 使用 truncate 代替 delete

**建表优化**

在表中建立索引时，优先考虑where、order by使用到的字段。
