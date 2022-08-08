---
layout:     post
title:      Elasticsearch
subtitle:   Elasticsearch查询语句
date:       2022-08-08
author:     dm
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Elasticsearch





---

#### 统计数量

``` elm
GET 25a6ab008e4346032a4b769d91388891/_count
{
  "query": {
    "bool": {

      "must": [
        {
          "term": {
            "year": {
              "": "VALUE"
            }
          }
        },
        {
          "match": {
            "archivesType": 1
          }
        }
      ]
    }
  }
}


GET 25a6ab008e4346032a4b769d91388891/_count
{
  "query": {
    "match": {
      "xxx": "123"
    }
  }
}
```

#### 修改语句

```elm

POST 25a6ab008e4346032a4b769d91388891/_update_by_query
{
  "query": {
    
    "bool": {
      
      "must": [
        {
          "match_phrase": {
            "category": "出生"
          }
        },
        {
          "match": {
            "categoryCode.keyword": "出生"
          }
        }
      ]
    }
  },
    "script": {
        "inline": "ctx._source['categoryCode'] ='CSYX'"
    }
}
```

#### 查询映射关系

``` elm
GET 25a6ab008e4346032a4b769d91388891/_mapping
```

