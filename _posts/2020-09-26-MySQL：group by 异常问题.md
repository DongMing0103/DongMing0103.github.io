layout:     post
title:      MySQL问题
subtitle:   group by 异常问题
date:       2020-09-26
author:     dm
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:

​	-MySQL

---

## 报错问题
```
Expression #2 of SELECT list is not in GROUP BY clause and contains
nonaggregated column ‘sss.month_id’ which is not functionally
dependent on columns in GROUP BY clause; this is incompatible with
sql_mode=only_full_group_by

```

## 原因
```
查看mysql版本命令：select version();

查看sql_model参数命令：

SELECT @@GLOBAL.sql_mode;

SELECT @@SESSION.sql_mode;

```

```

MySQL 5.7.5及以上功能依赖检测功能。如果启用了ONLY_FULL_GROUP_BY SQL模式（默认情况下），MySQL将拒绝选择列表，HAVING条件或ORDER BY列表的查询引用在GROUP BY子句中既未命名的非集合列，也不在功能上依赖于它们。（5.7.5之前，MySQL没有检测到功能依赖关系，默认情况下不启用ONLY_FULL_GROUP_BY。有关5.7.5之前的行为的说明，请参见“MySQL 5.6参考手册”。）

```

## 解决方法
### 1.只选择出现在`group by`后面的列，或者给列增加聚合函数；

### 2.命令行输入：

```mysql
set sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
 
如果上面设置完成后依然报错，这里执行下面的SQL，将全局设置更改，然后必须关闭现有的连接，重新打开连接后查询即可
 
set global sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

```

默认关掉ONLY_FULL_GROUP_BY！

更改完之后可以查看`sql_mode:`

`SELECT @@GLOBAL.sql_mode;`

`SELECT @@SESSION.sql_mode;`



