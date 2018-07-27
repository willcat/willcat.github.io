---
title: redis笔记1
date: 2018-07-25 09:35:50
tags: [Redis,读书笔记, Redis实战, NoSql]
categories: [NoSql]
---
### redis数据结构概览，及部分常用命令
#### 字符串
|存储的值 | 操作 | 说明 |
|----|----|
|字符串、整数、浮点数| set get del| 支持对字符串全部或部分的操作、支持对整数和浮点数的自增、自减操作|

#### 列表
|命令 | 行为 |
|----|----|
|RPUSH| 将给定值push到列表最右边 |
|LPUSH| 将给定值push到列表最左边 |
|LINDEX| 获取列表对应位置上的单个元素,左起为0 |
|LPOP| 从列表最左边弹出一个元素，并且返回该元素的值|

#### 集合(set)
|命令 | 行为 |
|----|----|
|SADD| 添加元素到集合，重复元素不会添加|
|SREM| 从集合中删除元素 |
|SISMEMBER| 查看元素是否在集合中 |
|SMEMBERS| 查看集合中所有元素，如果元素多会很慢,**慎用**|

#### 散列(hash)
|命令 | 行为 |
|----|----|
|HSET| 添加元素到hash,如果key重复则覆盖旧的value|
|HDEL| 从hash中删除元素 |
|HGET| 读取指定元素的值 |
|HGETALL| 查看集合中所有元素|

#### 有序集合(zset)
有序集合类似散列也是键值对，不过值是`score`，而且必须为**浮点数**

|命令 | 行为 |示例|
|----|----|
|ZADD| 添加元素到| ZADD key score member |
|ZREM| 删除元素 | |
|ZRANGE | 读取指定位置范围的member | ZRANGE key start stop [WITHSCORES] ,位置都是闭合的|
|ZRANGEBYSCORE | 读取指定score范围的member | ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]|

### 数据结构操作进阶
#### 字符串
1. valued的整数类型的取值范围与系统的长整数一致，32位系统上就是32位有符号整数，在64位系统上就是64位有符号整数。
#### 发布和订阅
`PUBLISH`
`SUBSCRIBE`
#### Sort
能同时适用三种数据的操作
#### 基本的redis事务
用法:
`
MULTI
cmd1
cmd2
..
EXEC
`
#### 过期
1. redis不能为键里单个元素设置过期时间。
|命令 | 行为 |示例|
|----|----|
|PESIST|移除键的过期时间|
|TTL|查看键还有多久过期时间，单位是秒|
|PTTL|查看键还有多久过期时间,单位是毫秒|
