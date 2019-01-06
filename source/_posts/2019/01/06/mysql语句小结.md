---
title: mysql语句小结
date: 2019-01-06 09:01:37
tags: [mysql]
categories: 基础杂谈
type:
---

> 纸上得来终夜寝啊，据知此事要躬行

## 前言

说起数据库学习历程，启蒙应该是毕业前完成的学习了慕课网的一门课程两遍吧，还自我感觉尚可，接下来就是入职以来在公司内部奥特曼系统写sql吧，算上真正操练了，对联合查询子查询等高级使用有个进阶，然后就是现在写node项目直连数据库啦，更多的是考虑了大量数据下查询性能以及安全权限方面的问题。

<!--more-->

## 常用全局命令

删除表数据：

1. `drop table tb`：暴力，删除内容和定义，释放空间，直接完全删除
2. `truncate table tb`：意义上和drop相同，区别是只是清空表数据而已, 比较温柔。
3. `delete table tb`：也是删除整个表的数据,但是过程是痛苦的(系统一行一行地删,效率较truncate低)

查看sql配置项： `show global variables like '%secure_file_priv%';`

## 批量插入记录(重复更新)

海量数据常常面临高效入库的过程，如果一次条记录入库，反复新增断开数据库连接，开销是很惊人的，常用的方法有两种，replace和insert，均是需要存在唯一索引，具体写法如下。

1. `REPLACE INTO T (id, c1, c2) VALUES (1, 1, 1), (2, 2, 2) ON DUPLICATE KEY UPDATE c1 = c1 + VALUES(c1), c2 = c2 + VALUES(c2);`插入数据可以编写函数**拼接sql**语句，项目中我们就是这种方式 

   ```js
   let sql = `replace into ${tableName}(${columns.join(', ')}) values `;
   let str = [];
   let _push = function(row) {
       let values = columns.map(column => {
           let value = row[column];
           return typeof value === 'number' || value === 'NOW()' ? value : `'${value}'`;
       });
       str.push(`(${values.join(', ')})`);
   };
   rowsList.forEach(rows => {
       if (Array.isArray(rows)) {
           rows.forEach(row => {
               _push(row);
           });
       } else if (typeof rows === 'object') {
           _push(rows);
       }
   });
   sql += str.join(',') + ';';
   ```

2. `INSERT`其实和replace类似，只是insert可以仅仅更新个别字段，而replace要全部更新前面插入的字段

## 批量更新

批量更新其实前面讲的那两种方法也可以做到，只是我们有时仅仅想想大量更新数据某个字段用不到前面的**两种牛刀**，那么`update`就要出场了。

```mysql
UPDATE T
SET c1 = c1 + CASE id 
WHEN 1 THEN 1 
WHEN 2 THEN 2 
END, 
c2 = c2 + CASE id 
WHEN 1 THEN 1 
WHEN 2 THEN 2 
END
WHERE id IN (1, 2);
```

当然where条件也可以不加的。

## 随机查询指定数量数据

之前因为对sql rand()函数原理不了解，查询我们一个千万级的数据库时使用了rand()函数，结果可想而知了。

1. `Order by Rand()`：rand() 函数会先为每一行数据生成一个 1~0之间的随机数，然后在根据这个数字，进行**排序**再选出最小的N行数据（N取决于limit N), 如果数据海量排序比较速度可想而知了，一般来说超过万行的数据就不推荐使用这种方式了。
2. `rand()改进1` ：`SELECT id FROM users, (SELECT ((1/COUNT(*))*100) as n FROM users) as x WHERE RAND()<=x.n LIMIT 1;`首先使用了一个**子查询**，计算出你想要随机出的记录所**在总记录的百分比**，然后再乘上100（防止比例过小）再使用这个小数，去和随机数比较，取出小于或等于这个小数的记录。举个例子 你想从一百万条记录中随机取10条记录，那么算式就是 10/1_000_000 * 100 = 0.001 查询语句就是：`SELECT id FROM users WHERE RAND()<=0.001 LIMIT 10;`。其实也可以首先子查询查询出数据总量，根据总量偏移进行正常查询**不使用rand函数**。
3. `rand()改进2`: `SELECT id FROM users, (SELECT ((1/COUNT(*))*100) as n FROM users) as x WHERE RAND()<=x.n ORDER BY RAND() LIMIT 1;`改进方法1中达到了快速数据的目的，但是它的随机性不好，那么改进方法2就是使用一定的性能去换取随机分布率.
4. `inner join`: `SELECT * FROM users as u JOIN (SELECT ROUND(RAND() * (SELECT MAX(id) FROM users)) AS id ) AS u2 WHERE u.id >= u2.id ORDER BY u.id DESC LIMIT 1;`巧妙的使用了自增长的ID主键，取其最大值，然后再乘上随机函数 的到一个 随机的ID，这样你就可以根据想要得到的随机记录数，决定使用 >= 或是 = 运算符去筛选结果了( = 仅用于随机一条记录的情况)。被查询的表必须是**连续自增**的主键表，很多表其实不是连续自增的。