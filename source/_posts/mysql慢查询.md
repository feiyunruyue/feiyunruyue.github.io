title: mysql慢查询
date: 2018-08-17 10:48:26
categories: 编程
tags: mysql
---
分享两个生产环境遇到的慢查询，两个查询以前都挺正常的，直到有一天，数据量突然上来了，问题就来了。

<!-- more -->

## 慢查询一
先说下第一个查询，背景是这样的，每个用户有多条记录，每个记录有个金额。

A 1000
A 2000
B 1000
。。。


统计完后是这样的
A 3000
B 1000

```
SELECT userId, sum(amount) FROM t1 WHERE status=1 GROUP BY userId limit #{offset}, 1000
```

每天大约1万个用户，直到有一天，突然增加了80万用户，sql找到offset的时候，相当于要查出来80万条数据，然后把这些数据丢弃掉，这样相当耗时，执行explain可以发现，使用了 "Using where; Using temporary; Using filesort"，使用了临时表，还有文件排序（无法用索引文成的排序）。


解决方案当时提了两种，最后采用了下面这种。

```
SELECT userId, SUM(amount) FROM t1 WHERE userId> #{userId} and status=1 GROUP BY userId
 order by userId asc limit 1000
```

这样可以利用userId上的索引，速度一下子提上来了。

另外一种方案的思路是分两次查询，先查询出一个范围来，再根据范围去查询。


## 慢查询二

```
SELECT * FROM t2 WHERE id > ? and interest_date=? order by id asc limit 1000
```

数据量有一天超过了2000万，首次查询的时候特别慢，首次查询id>0。

使用explain发现走的主键这个索引，没有走interest_date。改成强制走interest_date索引，使用force index，好了(这个方案还可以优化)。

```
SELECT * FROM t2 force index(idx_date) WHERE id > ? and interest_date=? order by id asc limit 1000
```






