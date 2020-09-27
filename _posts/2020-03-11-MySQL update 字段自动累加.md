---
layout:     post
title:      MySQL问题
subtitle:   update 字段自动累加
date:       2020-03-11
author:     dm
header-img: img/post-bg-mma-2.jpg
catalog: true
tags:
    - MySQL




---

## 修改时字段自动累加
```mysql
set @num=0;
update equipment_market_goods_catagory
SET sort_no = (@num := @num + 1);

```

