[toc]



---



# 需求

有这么一条数据A，他有一个字段tag_ids，用来存储tag id列表。

1149265184814718977,26,33

需要查找的数据表T。

| id   | name  | tag_ids                     |
| ---- | ----- | --------------------------- |
| 1    | 测试1 | 1149265184814718977,3333    |
| 2    | 测试2 | 123213,123333,33            |
| 3    | 测试3 | 1149265184814718977,26,1111 |

我需要将数据A的tag_ids分别放到数据表T的tag_ids里面进行查询。

我们知道find_in_set特别适合这种查询场景，我们可以轻松的用如下SQL，获取数据表T的符合条件的数据：

```sql
select * from T where find_in_set("33",tag_ids);
```

这样就会查询出符合条件tag_ids中包含33的数据出来了。

那我们直接将数据A的tag_ids放进去替换33呢？

```sql
select * from T where find_in_set("1149265184814718977,26,33",tag_ids);
```

执行结果显而易见，查询不出任何东西。那我们应该怎么解决这个问题呢？



# 解决方法

1. 首先我们需要创建一张序列表

```sql
CREATE TABLE `sequence` (
  `id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

2. 填入1-100的数据，具体可以根据需要增加。

3. 利用substring_index和sequence切割字符串

   ```sql
   SELECT
   	substring_index( substring_index( t.tag_ids, ',', b.id ), ',',- 1 ) AS id,b.id 
   FROM
   	A AS a 
   	left JOIN sequence AS b ON b.id between 1 and ( length( a.tag_ids ) - length( REPLACE ( a.tag_ids, ',', '' )) + 1 ) 
   ```

   这会将数据表A转化成这样一下格式。

   | tag_ids             |
   | ------------------- |
   | 1149265184814718977 |
   | 26                  |
   | 33                  |

4. 结合find_in_set查询数据表T。

   ```sql
   SELECT
   	b.NAME,
   	b.tag_ids 
   FROM
   	(
   	SELECT
   		substring_index( substring_index( a.tag_ids, ',', b.id ), ',',- 1 ) AS id 
   	FROM
   		A AS a
   		LEFT JOIN sequence AS b ON b.id BETWEEN 1 
   		AND ( length( a.tag_ids ) - length( REPLACE ( a.tag_ids, ',', '' )) + 1 ) 
   	) AS a
   	JOIN T AS b ON FIND_IN_SET(
   	a.id,
   	b.tag_ids)
   ```

   






