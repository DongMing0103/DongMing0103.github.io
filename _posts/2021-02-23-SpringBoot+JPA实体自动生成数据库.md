---
layout:     post
title:      SpringBoot+JPA实体自动生成数据库
subtitle:   自动生成数据库
date:       2021-02-26
author:     dm
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - SpringData







---

## POM

```pom
<!-- MySQL + MyBatis依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.13</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.2</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.1</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```



## 连接数据库

```yaml
spring:
    datasource:
        url: jdbc:mysql://xxx.xxx.xxx.xxx:3306/xxx?createDatabaseIfNotExist=true&serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
        username: xxx
        password: xxxx
        driver-class-name: com.mysql.jdbc.Driver
    jpa:
        properties:
            hibernate:
                hbm2ddl:
                    auto: update  # create每次运行都删除原有表创建新表,update不用每次创建新表
        database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
        show-sql: true # 打印SQL语句
```

- MySQL5InnoDBDialect数据库引擎
- ddl-auto
  - validate 加载 Hibernate 时，验证创建数据库表结构
  - create 每次加载 Hibernate ，重新创建数据库表结构
  - create-drop 加载 Hibernate 时创建，sessionFactory关闭退出时删除表结构
  - update 加载 Hibernate 自动更新数据库结构

**启动类开启**

```java
@SpringBootApplication
//自动填充或更新实体中的CreateDate、CreatedBy
@EnableJpaAuditing
public class Startup {
    public static void main(String[] args) {
        SpringApplication.run(Startup.class, args);
    }
}
```

**公共实体**

```java
/** @Author dongm @Description: 公共实体 @Date 2021/2/25 */
@Data
// 该注解标注的类不会映射到数据库中单独的表，该类所拥有的属性都将映射到其子类
@MappedSuperclass
public class CommonEntity extends BaseEntity {

  @Id private String id;

  @Field
  @CreatedBy
  @Column(name = "create_user_id", columnDefinition = "varchar(32) COMMENT '创建人ID'")
  private String createUserId;

  @Field
  @LastModifiedBy
  @Column(name = "update_user_id", columnDefinition = "varchar(32) COMMENT '更新人ID'")
  private String updateUserId;

  @Field
  @CreatedDate
  @Column(
      name = "create_time",
      columnDefinition = "timestamp default CURRENT_TIMESTAMP COMMENT '创建时间'")
  private LocalDateTime createTime;

  @Field
  @LastModifiedDate
  @Column(
      name = "update_time",
      columnDefinition =
          " timestamp default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'")
  private LocalDateTime updateTime;

  /** 渠道编号 */
  @Column(name = "channel_no", columnDefinition = "varchar(255) COMMENT '渠道编号'")
  private String channelNo;

  /** 应用编号 */
  @Column(name = "app_no", columnDefinition = "varchar(255) COMMENT '应用编号'")
  private String appNo;

  /** 流水号 */
  @Column(name = "serial_no", columnDefinition = "varchar(255) COMMENT '流水号'")
  private String serialNo;

  /** 签名 */
  @Column(name = "sign_data", columnDefinition = "varchar(200) COMMENT '签名'")
  private String signData;
}
```

- 超类，公共实体
- Column定义字段、Id定义唯一标识
- columnDefinition定义字段类型、长度、默认值、注释
- CURRENT_TIMESTAMP定义默认的创建和修改时间

```java
/** @Author dongm @Description: @Date 2021/2/22 */
@Data
@Entity
@Table(appliesTo = "send_verify_message", comment = "发送验证码短信表")
public class SendVerifyMessage extends CommonEntity {

  @Id
  @Column(name = "id", columnDefinition = "varchar(32) COMMENT '主键id'")
  private String id;

  /** 交易码 */
  @Column(name = "trade_no", columnDefinition = "varchar(255) COMMENT '交易码'")
  private String tradeNo;

  /** 手机号码 */
  @Column(name = "mobile", columnDefinition = "varchar(11) COMMENT '手机号码'")
  private String mobile;

  /** 模板编号，固定编号“VerifyCode” */
  @Column(name = "model_no", columnDefinition = "varchar(11) COMMENT '模板编号，固定编号“VerifyCode”'")
  private String modelNo;

  /** 操作日期（YYYY-MM-DD） */
  @Column(name = "trans_date", columnDefinition = "date COMMENT '操作日期（YYYY-MM-DD）'")
  @Temporal(TemporalType.DATE)
  private Date transDate;

  /** 操作时间（YYYY-MM-DD hh:mm:ss） */
  @Column(name = "trans_tradetime", columnDefinition = "date COMMENT '操作时间（YYYY-MM-DD hh:mm:ss）'")
  @Temporal(TemporalType.TIMESTAMP)
  private Date transTradetime;
}
```

- 数据库定义
- 表定义、注释，Table为org.hibernate.annotations.Table
- 索引定义
- 唯一约束