# explain执行计划

**XPLAIN简介**

　　EXPLAIN 命令是查看查询优化器如何决定执行查询的主要方法,使用EXPLAIN,只需要在查询中的SELECT关键字之前增加EXPLAIN这个词即可,MYSQL会在查询上设置一个标记,当执行查询时,这个标记会使其返回关于在执行计划中每一步的信息,而不是执行它,它会返回一行或多行信息,显示出执行计划中的每一部分和执行的次序,从而可以从分析结果中找到查询语句或是表结构的性能瓶颈。

**EXPLAIN能干嘛**

1. 分析出表的读取顺序
2. 数据读取操作的操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 表之间的引用
6. 每张表有多少行被优化器查询  

**EXPLAIN如何用**

Explain + SQL语句即可,如下:

```
explain select * from tbl_dept;
```

- 1执行结果如下:

*![这里写图片描述](https://img-blog.csdn.net/20180521155012377?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poMTU3MzI2MjE2Nzk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)*

**EXPLAIN结果参数含义**

1.id: id代表执行select子句或操作表的顺序,例如,上述的执行结果代表只有一次执行而且执行顺序是第一(因为只有一个id为1的执行结果),id分别有三种不同的执行结果,分别如下:

- id相同,执行顺序由上至下

![img](https://img2018.cnblogs.com/blog/885859/201904/885859-20190419152720756-229903142.png)

- id不同,如果是子查询,id的序号会递增,id值越大,优先级越高,越先被执行

![img](https://img2018.cnblogs.com/blog/885859/201904/885859-20190419152812436-124187623.png)

- id相同和不同,同时存在,遵从优先级高的优先执行,优先级相同的按照由上至下的顺序执行

![img](https://img2018.cnblogs.com/blog/885859/201904/885859-20190419152835637-1233280197.png)

**2.select_type**
　　查询的类型,主要用于区别普通查询,联合查询,子查询等复杂查询

- simple: 简单的select查询,查询中不包含子查询或union查询
- primary: 查询中若包含任何复杂的子部分,最外层查询则被标记为primary
- subquery 在select 或where 列表中包含了子查询
- derived 在from列表中包含的子查询被标记为derived,mysql会递归这些子查询,把结果放在临时表里
- union 做第二个select出现在union之后,则被标记为union,若union包含在from子句的子查询中,外层select将被标记为derived
- union result 从union表获取结果的select

**3.table**
　　显示一行的数据时关于哪张表的
**4.type**
　　查询类型从最好到最差依次是:system>const>eq_ref>ref>range>index>All,一般情况下,得至少保证达到range级别,最好能达到ref

- **system**:表只有一行记录,这是const类型的特例,平时不会出现
- **const**:表示通过索引一次就找到了,const即常量,它用于比较primary key或unique索引,因为只匹配一行数据,所以效率很快,如将主键置于where条件中,mysql就能将该查询转换为一个常量 

![img](https://img2018.cnblogs.com/blog/885859/201904/885859-20190419153049676-1370075656.png)

- **eq_ref**:唯一性索引扫描,对于每个索引键,表中只有一条记录与之匹配,常见于主键或唯一索引扫描
- **ref**:非唯一性索引扫描,返回匹配某个单独值的行,它可能会找到多个符合条件的行,所以他应该属于查找和扫描的混合体
- **range**:只检索给定范围的行,使用一个索引来选择行,如where语句中出现了between,<,>,in等查询,这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。
- **index**:index类型只遍历索引树,这通常比All快,因为索引文件通常比数据文件小,index是从索引中读取,all从硬盘中读取
- **all**:全表扫描,是最差的一种查询类型

**5.possible_keys**
　　显示可能应用在这张表中的索引,一个或多个,查询到的索引不一定是真正被用到的

**6.key**
　　实际使用的索引,如果为null,则没有使用索引,因此会出现possible_keys列有可能被用到的索引,但是key列为null,表示实际没用索引。

**7.key_len**
　　表示索引中使用的字节数,而通过该列计算查询中使用的 索引长度,在不损失精确性的情况下,长度越短越好,key_len显示的值为索引字段的最大可能长度,并非实际使用长度,即,key_len是根据表定义计算而得么不是通过表内检索出的

**8.ref**
　　显示索引的哪一列被使用了,如果可能的话是一个常数,哪些列或常量被用于查找索引列上的值

**9.rows**
　　根据表统计信息及索引选用情况,大只估算出找到所需的记录所需要读取的行数

**10.Extra**

- **Using filesort**:说明mysql会对数据使用一个外部的索引排序,而不是按照表内的索引顺序进行读取,mysql中无法利用索引完成的排序操作称为"文件排序"
- **Using temporary** :使用了临时表保存中间结果,mysql在对查询结果排序时使用临时表,常见于order by和分组查询group by
- **Using index**:表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。 其中的覆盖索引含义是所查询的列是和建立的索引字段和个数是一一对应的
- **Using where**:表明使用了where过滤
- **Using join buffer**:表明使用了连接缓存,如在查询的时候会有多次join,则可能会产生临时表
- **impossible where**:表示where子句的值总是false,不能用来获取任何元祖。如下例：

```
select * from t1 where id='1' and id='2';
```

- select tables optimized away

在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

- distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作，即一旦MySQL找到了与行相联合匹配的行，就不再搜索了。

**重点：**

　　**type**：访问类型，查看SQL到底是以何种类型访问数据的。

　　**key**：使用的索引，MySQL用了哪个索引，有时候MySQL用的索引不是最好的，需要force index()。

　　**rows**：最大扫描的列数。

　　**extra**：重要的额外信息，特别注意损耗性能的两个情况，using filesort和using temporary。

 

附出处：https://blog.csdn.net/zh15732621679/article/details/80394790#commentBox