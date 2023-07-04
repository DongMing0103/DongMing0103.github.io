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

## ES操作手册

**说明：**

`1. 文档中data_index为实际情况索引`
`2. 所有操作均在kibana操作`

#### 1、ES查询突破1W条限制（分页10条/页时 1000页以后报错）

```elm
PUT _all/_settings
{         
	"index.max_result_window":200000
}
```

#### 2、查询es内的所有索引信息

```elm
GET _cat/indices
```

![查询ES内所有索引](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%9F%A5%E8%AF%A2ES%E5%86%85%E6%89%80%E6%9C%89%E7%B4%A2%E5%BC%95.png)

#### 3、查询索引字段信息

![查询索引字段信息](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%9F%A5%E8%AF%A2%E7%B4%A2%E5%BC%95%E5%AD%97%E6%AE%B5%E4%BF%A1%E6%81%AF.png)

#### 4、无条件查询指定索引数据量

```elm
GET data_index/_count
```

![无条件查询指定索引数据量](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%97%A0%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E6%8C%87%E5%AE%9A%E7%B4%A2%E5%BC%95%E6%95%B0%E6%8D%AE%E9%87%8F.png)

#### 5、有条件查询指定索引数据量（实际情况可自由变通）

```
GET data_index/_count
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "archivingState": 1
          }
        },
        {
          "match_phrase": {
            "archivalCode": "Z017"
          }
        },
        {
          "range": {
            "pageNumber": {
              "gte": 1,
              "lte": 1000
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "status": 1
          }
        }
      ]
    }
  }
}
```

![有条件查询指定索引数据量](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%9C%89%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E6%8C%87%E5%AE%9A%E7%B4%A2%E5%BC%95%E6%95%B0%E6%8D%AE%E9%87%8F.png)

#### 6、无条件查询指定索引数据

```elm
GET data_index/_search
```

![无条件查询指定索引数据](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%97%A0%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E6%8C%87%E5%AE%9A%E7%B4%A2%E5%BC%95%E6%95%B0%E6%8D%AE.png)

#### 7、有条件查询指定索引数据（条件同查询数量）

```elm
GET data_index/_search
```

![有条件查询指定索引数据](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%9C%89%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E6%8C%87%E5%AE%9A%E7%B4%A2%E5%BC%95%E6%95%B0%E6%8D%AE.png)

#### 8、删除指定索引所有数据（谨慎操作）

```
POST data_index/_delete_by_query
{
  "query": {
    "match_all": {}
	}
}
```

#### 9、根据条件删除指定索引数据（谨慎操作）

```elm
POST data_index/_delete_by_query
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "archivingState": 1
          }
        },
        {
          "match_phrase": {
            "archivalCode": "Z017"
          }
        },
        {
          "range": {
            "pageNumber": {
              "gte": 1,
              "lte": 1000
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "status": 1
          }
        }
      ]
    }
  }
}
```

#### 10、删除索引（谨慎操作）

```elm
DELETE data_index
```

#### 11、根据条件修改数据（条件和修改内容可根据需要自行增删）（谨慎操作）

```elm
POST data_index/_update_by_query
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "archivingState": 1
          }
        },
        {
          "match_phrase": {
            "archivalCode": "Z017"
          }
        },
        {
          "range": {
            "pageNumber": {
              "gte": 1,
              "lte": 1000
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "status": 1
          }
        }
      ]
    }
  },
  "script": {
    "inline": "ctx._source['archivingState'] = 1;ctx._source['docketKey'] = ctx._source['archivalCode'].substring(0,ctx._source['archivalCode'].lastIndexOf('-'));ctx._source['archivalCode']=ctx._source['archivalCode'].replace('Z017','Z016')"
  }
}
```

![根据条件修改数据（条件和修改内容可根据需要自行增删）](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E6%A0%B9%E6%8D%AE%E6%9D%A1%E4%BB%B6%E4%BF%AE%E6%94%B9%E6%95%B0%E6%8D%AE%EF%BC%88%E6%9D%A1%E4%BB%B6%E5%92%8C%E4%BF%AE%E6%94%B9%E5%86%85%E5%AE%B9%E5%8F%AF%E6%A0%B9%E6%8D%AE%E9%9C%80%E8%A6%81%E8%87%AA%E8%A1%8C%E5%A2%9E%E5%88%A0%EF%BC%89.png)

#### 12、ES备份

`第一步：查询原索引字段映射,复制查询结果mapping{...}`

```elm
GET f219ec6de3ccb79360b4ecf13d6c9c95/_mapping
```

`第二步：把复制的内容，复制到要新建的索引下，并执行：`

```elm
PUT f219ec6de3ccb79360b4ecf13d6c9c95_new
{
 "mappings" : {......}     
}
```

`第三步：把原索引数据复制到新索引上`

```elm
POST _reindex
{
  "source": {
    "index": "f219ec6de3ccb79360b4ecf13d6c9c95"
  },
  "dest": {
    "index": "f219ec6de3ccb79360b4ecf13d6c9c95_new"
  }
}
```
`第四步：查询原索引与新索引数量是否一致`

```elm
GET f219ec6de3ccb79360b4ecf13d6c9c95_new/_count
GET f219ec6de3ccb79360b4ecf13d6c9c95/_count
```

#### 13、避免请求超时
![避免请求超时](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/work/%E9%81%BF%E5%85%8D%E8%AF%B7%E6%B1%82%E8%B6%85%E6%97%B6.png)
`GET _tasks/xxxxxx：信息由请求后出现`



