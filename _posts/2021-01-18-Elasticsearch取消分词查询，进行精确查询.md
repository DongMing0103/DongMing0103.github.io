---
layout:     post
title:      Elasticsearch
subtitle:   Elasticsearch取消分词查询
date:       2021-01-18
author:     dm
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Elasticsearch





---



*问题背景：通过ES查询数据“沙”，期望查询结果为空，实际查询出数据“黄沙”、“河底沙”。原因：ES使用分词查询，非SQL中 “=” 查询。故取消ES默认分词查询。*



---

# 1. 取消分词

## 1.1 添加注解

`在需要取消默认分词字段上添加注解 @Field(index = FieldIndex.not_analyzed, type = FieldType.String)`

```java
package com.xcb.common.service.dto;

import com.xcb.common.entity.XcbDictItem;
import lombok.Data;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldIndex;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

@Document(indexName = XcbDictItemDTO.index)
@Data
public class XcbDictItemDTO implements Serializable {
  public static final String index = XcbDictItem.collName + "_query";

  private String id;
	// 上级编码
  private String parentId; 
	// 字典名称
  @Field(index = FieldIndex.not_analyzed, type = FieldType.String)
  private String name; 
	// 字典值
  private String value; 
	// 字典描述
  private String desc; 
	 // 排序
  private int order = 0;
	// 是否隐藏 0否 1隐藏
  private Integer show; 

  private List<XcbDictItemDTO> childItems = Collections.emptyList();

  private LocalDateTime createTime;

  @Override
  public boolean equals(Object o) {
    if (this == o) {
      return true;
    }
    if (o == null || getClass() != o.getClass()) {
      return false;
    }
    if (!super.equals(o)) {
      return false;
    }
    XcbDictItemDTO itemDTO = (XcbDictItemDTO) o;
    return Objects.equals(id, itemDTO.id);
  }

  @Override
  public int hashCode() {
    return Objects.hash(super.hashCode(), id);
  }
}
```



## 1.2. 删除Elasticsearch中数据

## 1.3. 重新启动项目 



# 2. 处理其他方法需使用分词查询问题

## 2.1 修改查询语句

`WildCardQueryAppender.newInstance(qb).append("name", request.getName());`

```java
@Override
public Page<XcbDictItemDTO> getByParentCode(DictItemRequest request, Pageable pageable) {
  // 如果没有传sort，默认创建时间降序排列
  pageable = SortUtil.getDefaultSortForArticle(pageable);
  BoolQueryBuilder qb = QueryBuilders.boolQuery();
  if (StringUtils.isNotBlank(request.getParentCode())) {
    Optional.ofNullable(dictSerivice.getByCode(request.getParentCode()))
        .map(dict -> qb.must(QueryBuilders.termQuery("parentId", dict.getId())));
  }
  if (StringUtils.isNotBlank(request.getName())) {
    WildCardQueryAppender.newInstance(qb).append("name", request.getName());
  }
  /** 默认查询条件 */
  BoolQueryBuilderImpl.getDefaultBoolQueryBuilder(qb, request);
  return qb.hasClauses()
      ? xcbDictItemSearchRepository.search(qb, pageable)
      : xcbDictItemSearchRepository.findAll(pageable);
}
```

## 2.2 封装WildCardQueryAppender方法

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;

public class WildCardQueryAppender {
    private BoolQueryBuilder qb;

    public static WildCardQueryAppender newInstance(BoolQueryBuilder qb) {
        return new WildCardQueryAppender(qb);
    }

    private WildCardQueryAppender(BoolQueryBuilder qb) {
        this.qb = qb;
    }

    public WildCardQueryAppender append(String name, String value) {
        if (StringUtils.isNotBlank(value)) {
            this.qb.must(QueryBuilders.wildcardQuery(name, "*" + value + "*"));
        }

        return this;
    }

    public BoolQueryBuilder getResult() {
        return this.qb;
    }
}
```

