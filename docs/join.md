---
date: 2020-10-15T20:52:34+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "MySQL JOIN 方法理解"  # 文章标题
url:  "posts/mysql/docs/join"  # 设置网页永久链接
tags: [ "sql", "mysql"]  # 标签
series: [ "MySQL 学习笔记"]  # 系列
categories: [ "学习笔记"]  # 分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## JOIN 简介

- INNER JOIN：如果表中有至少一个匹配，则返回行
- LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行
- RIGHT JOIN：即使左表中没有匹配，也从右表返回所有的行
- FULL JOIN：只要其中一个表中存在匹配，则返回行

## INNER JOIN

> 交集

```sql
SELECT websites.id, websites.name, access_log.count, access_log.date
FROM websites
         INNER JOIN access_log
                    ON websites.id = access_log.site_id;
```

## LEFT JOIN

> 右边表-左边表

```sql
SELECT websites.name, access_log.count, access_log.date
FROM websites
         LEFT JOIN access_log
                   ON websites.id = access_log.site_id
ORDER BY access_log.count DESC;
```

## RIGHT JOIN

> 左边表-右边表

```sql
SELECT websites.name, access_log.count, access_log.date
FROM websites
         RIGHT JOIN access_log
                    ON websites.id = access_log.site_id
ORDER BY access_log.count DESC;
```

## UNION

> 操作符合并两个或多个 SELECT 语句的结果，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

### 只会选取不同的值

```sql
SELECT country
FROM websites
UNION
SELECT country
FROM apps
ORDER BY country;
```

### 选取所有的值

```sql
SELECT country, name
FROM websites
WHERE country = 'cn'
UNION ALL
SELECT country, app_name
FROM apps
WHERE country = 'cn'
ORDER BY country;
```

## 从一个表复制信息到另一个表

> 语句从一个表复制数据，然后把数据插入到一个新创建的表中

### CREATE TABLE

```sql
CREATE TABLE websites_backup
SELECT *
FROM websites;
```

### INSERT INTO SELECT

> 语句从一个表复制数据，然后把数据插入到一个已存在的表中

```sql
INSERT INTO websites (name, country)
SELECT app_name, country
FROM apps;

INSERT INTO websites (name, country)
SELECT app_name, country
FROM apps
WHERE id = 1;
```