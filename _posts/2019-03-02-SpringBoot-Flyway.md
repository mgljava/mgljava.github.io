---
layout:     post
title:      SpringBoot集成Flyway
subtitle:   SpringBoot integration Flyway
date:       2019-03-01
author:     Monk
header-img: img/spring-boot-flyway.jpg
catalog: true
tags:
    - SpringBoot
    - Flyway
---
## SpringBoot 集成数据库迁移工具 Flyway（基于Gradle构建）

源代码：[Github](https://github.com/mgljava/java-tools)

由于在整合时用到了一些相关的数据库技术，就不一一详解了，具体如下：

> * flyway plugin: org.flywaydb.flyway
> * Druid: druid-spring-boot-starter
> * JPA：spring-boot-starter-data-jpa

### 1. 引入插件和依赖（build.gradle）
```
plugins {
    id 'java'
    id "org.springframework.boot" version '2.1.2.RELEASE'
    id "org.flywaydb.flyway" version "5.1.4"  // Flyway 插件
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // JPA
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    
    // flyway
    implementation "org.flywaydb:flyway-core:5.2.4"
    
    // 数据库
    implementation 'com.alibaba:druid-spring-boot-starter:1.1.10'
    runtimeOnly 'mysql:mysql-connector-java:8.0.12'
}
```

### 2. 编写配置文件（application.yml）
```
spring:
  flyway:
    url: jdbc:mysql://127.0.0.1:3306/flyway_test?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
    user: root
    password: 123456
    driver: com.mysql.cj.jdbc.Driver
    locations: classpath:db/migration # 指定sql文件存放的目录
  datasource: ## 其他参数就不配置了
    druid:
      url: jdbc:mysql://127.0.0.1:3306/flyway_test?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
```

### 3. 编写sql文件（V201903111310__CREATE_TABLE_USER_INFO.sql）,文件需要放在db/migration目录下,也可自定义目录，然后通过locations: classpath:custom/directory配置来指定
```sql
CREATE TABLE `user_info`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
  `age` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `name_index`(`name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 11 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

### 4. 建好数据库，然后运行，结果如下就代表集成成功了，所有放在配置目录下的sql将由flyway来管理和运行。
```
Init DruidDataSource
{dataSource-1} inited
Flyway Community Edition 5.2.4 by Boxfuse
Database: jdbc:mysql://127.0.0.1:3306/flyway_test (MySQL 5.
Successfully validated 1 migration (execution time 00:00.02
Current version of schema `flyway_test`: 201903111123
Schema `flyway_test` is up to date. No migration necessary.
```

### 5. Flyway的其他参数配置（整合而来）,所有的配置可以查看`org.springframework.boot.autoconfigure.flyway.FlywayProperties.java` 这个类提供的。
```
spring:
  flyway:
    baseline-version: 1 # 开始执行基准迁移时对现有的schema的版本打标签，默认值为1.
    baseline-description: first migration # 对执行迁移时基准版本的描述
    baseline-on-migrate: true # 当迁移时发现目标schema非空，而且带有没有元数据的表时，是否自动执行基准迁移，默认false.
    check-location: true # 检查迁移脚本的位置是否存在，默认false.
    clean-on-validation-error: false # 当发现校验错误时是否自动调用clean，默认false.
    encoding: UTF-8 # 设置迁移时的编码，默认UTF-8.
    ignore-future-migrations: true # 在读取模式历史记录表时是否忽略将来的迁移。
    init-sqls: # 当初始化好连接时要执行的SQL.
    out-of-order: false # 是否允许无序的迁移，默认false.
    placeholder-prefix: # 设置每个placeholder的前缀，默认${.
    placeholder-suffix: # 设置每个placeholder的后缀，默认}.
    schemas: # 设定需要flyway迁移的schema，大小写敏感，默认为连接默认的schema.
    sql-migration-prefix: # 迁移文件的前缀，默认为V.
    sql-migration-separator: # 迁移脚本的文件名分隔符，默认__
    sql-migration-suffixes: # 迁移脚本的后缀，默认为.sql
    table: # 使用的元数据表名，默认为schema_version
    validate-on-migrate: true # 迁移时是否校验，默认为true.
```