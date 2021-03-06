# 附2 阿里巴巴手册（泰山版）解读-MySQL数据库

## 1. 建表规约

- 【强制】varchar 是可变长字符串，不预先分配存储空间，长度不要超过 5000，如果存储长度 大于此值，**定义字段类型为 text，独立出来一张表，用主键来对应，避免影响其它字段索引效率**。

> 批量取出数据的场景下，通常不需要text字段的内容，text字段通常在 `getById` 这类场景中取出。如果未将text字段独立，则在扫描主表的某一行数据时，会严重降低性能。
>
> 详细原因参考：[B+树](./01-Mysql-index-data-structure.md#B+Tree(B-Tree变种))



- 【推荐】单表行数超过 500 万行或者单表容量超过 2GB，才推荐进行分库分表。

> 500万行是阿里的标准，不是适用于所有场景的固定阈值，需要根据具体的情况来分析。默认B+树的主索引容量为2000万个，一般不能超过这个值。通常建议分表阈值不超过1000万行。



## 索引规约

- 【强制】业务上具有唯一特性的字段，即使是组合字段，也必须建成唯一索引。

  说明：不要以为唯一索引影响了 insert 速度，这个速度损耗可以忽略，但提高查找速度是明显的；另外， 即使在应用层做了非常完善的校验控制，只要没有唯一索引，根据墨菲定律，必然有脏数据产生。

> 数据库层面，通过索引控制唯一性是非常必要的，这样可以更好的避免在代码开发和维护时的错误导致的脏数据。



- 【强制】超过三个表禁止 join。需要 join 的字段，数据类型保持绝对一致；多表关联查询时， 保证被关联的字段需要有索引。

  说明：即使**双表 join 也要注意表索引、SQL 性能**。

> 双表join时，需要尽量在大表一方的关联条件上建立索引，这样可以使用 [NLJ算法](./05-Sql-Optimize-02.md#嵌套循环连接Nested-Loop Join(NLJ) 算法) 提升关联查询的性能。



- 【强制】在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据

   实际文本区分度决定索引长度。

> 通常建立前20个字符的索引就有足够的区分度了。



- 【强制】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。

> 不允许 `like '%abc'` 或者 `like '%'`，因为这种条件会进行全表扫描，性能极差，而 `like 'abc%'` 可以走索引，是允许的。



- 【推荐】如果有 order by 的场景，请注意利用索引的有序性。order by 最后的字段是组合索

  引的一部分，并且放在索引组合顺序的最后，避免出现 file_sort 的情况，影响查询性能。

  正例：where a=? and b=? order by c; 索引：a_b_c

  反例：索引如果存在范围查询，那么索引有序性无法利用，如：WHERE a>10 ORDER BY b; 索引 a_b 无

  法排序。

> 参考  [Order by与Group by优化](./04-Sql-Optimize-01.md#Order by与Group by优化) 



- 【推荐】利用覆盖索引来进行查询操作，避免回表。

  说明：如果一本书需要知道第 11 章是什么标题，会翻开第 11 章对应的那一页吗？目录浏览一下就好，这 个目录就是起到覆盖索引的作用。

  正例：能够建立索引的种类分为主键索引、唯一索引、普通索引三种，而覆盖索引只是一种查询的一种效 果，用 explain 的结果，extra 列会出现：using index。

> 覆盖索引是指对于常用的查询语句，如果只覆盖少量字段，就基于这几个字段建立联合索引，这样在检索时只会查询这个联合索引表，而无需再回表进行关联，也无需对完整的数据行进行扫描，可以有效提高查询性能。
>
> 参考 [索引最左前缀原理](./01-Mysql-index-data-structure.md#索引最左前缀原理) 中的联合索引示意图



- 【推荐】利用延迟关联或者子查询优化超多分页场景。

  说明：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回 N 行，那当 offset 特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 SQL 改写。

  正例：先快速定位需要获取的 id 段，然后再关联：

  ```sql
  SELECT a.* FROM 表1 a, (select id from 表1 where 条件 LIMIT 100000,20) b where a.id=b.id
  ```

> 其中的b表查询走的是索引，只需要读一次磁盘就可以将索引载入内存，然后在内存中检索到所需的20个id，再与a表进行关联，使用 [NLJ算法](./05-Sql-Optimize-02.md#嵌套循环连接Nested-Loop Join(NLJ) 算法) 可以很快得到查询结果，只需要扫描a表的20行数据即可。
>
> 参考 [常见的分页场景优化技巧](./05-Sql-Optimize-02.md#常见的分页场景优化技巧) 



- 【推荐】SQL 性能优化的目标：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。

  说明：

   1） consts 单表中最多只有一个匹配行（主键或者唯一索引），在优化阶段即可读取到数据。

   2） ref 指的是使用普通的索引（normal index）。

   3） range 对索引进行范围检索。

  反例：explain 表的结果，type=index，索引物理文件全扫描，速度非常慢，这个 index 级别比较 range 还低，与全表扫描是小巫见大巫。

> 参考 [Explain详解与索引最佳实践](./02-Explain-Index-Actual.md) 



- 【推荐】建组合索引的时候，区分度最高的在最左边。

  正例：如果 where a=? and b=?，a 列的几乎接近于唯一值，那么只需要单建 idx_a 索引即可。

  说明：存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：where c>? and d=?

  那么即使 c 的区分度更高，也必须把 d 放在索引的最前列，即建立组合索引 idx_d_c。

> 通常，where条件中，出现范围检索的字段，如果需要创建联合索引，需要放在索引的最后。
>
> 如果建立的索引为 idx_c_d，当c使用范围检索时，mysql会认为数据量太大，回表开销大，而选择不走索引 idx_c_d，直接进行全表扫描。



## SQL语句

- 【强制】不要使用 count(列名)或 count(常量)来替代 count(\*)，count(\*)是 SQL92 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。

  说明：count(\*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。

> 参考  [count(*)查询优化](./05-Sql-Optimize-02.md#53-count查询优化) 



## ORM映射