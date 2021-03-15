---
title: Mongodb 常用命令
categories: blog
---

## 远程登录到 mongodb 实例

```sh
mongo --host xxxxx --authenticationDatabase admin -u root -p
```

## 选择数据库和显示所有集合

```sh
> show databases;
admin                      0.000GB
local                      0.000GB
production                 19.478GB
> use production;
> show tables;
```

## 获取集合的统计信息

```sh
> db.logs.stats();
{
	"ns" : "production.logs",
	"size" : 16613917285,  # 集合大小
	"count" : 122419823,  # 文档总数
	"avgObjSize" : 135,
	"storageSize" : 6713712640,
	"capped" : false,
```


## 对索引的操作

```sh
# 获取 clogs 的所有索引
db.logs.getIndexes();

# 删除某个索引
db.logs.dropIndex();

# 添加某个索引
db.logs.createIndex();
```

## 查询分析

```sh
# MongoDB运行查询优化器对当前的查询进行评估并选择一个最佳的查询计划
db.logs.find({ "to" : "5b0d4602eb0a1952811a6d22", "action" : "SHARE" }).explain()

#mongoDB运行查询优化器对当前的查询进行评估并选择一个最佳的查询计划进行执行，在执行完毕后返回这个最佳执行计划执行完成时的相关统计信息
db.logs.find({ "to" : "5b0d4602eb0a1952811a6d22", "action" : "SHARE" }).explain( "executionStats" )
```

## 终止操作

如果有特别慢的查询或者很耗时的加索引操作导致系统出现异常，可以通过 mongodb 提供的命令终止该操作，以便于系统尽快恢复

```sh
# 查找当前正在运行的操作
db.currentOp()

# 根据 operation id 终止对应的操作
db.killOp()

# 找出创建索引的操作
db.currentOp(
    {
      $or: [
        { op: "command", "query.createIndexes": { $exists: true } },
        { op: "none", ns: /\.system\.indexes\b/ }
      ]
    }
)
```

> [killOp](https://docs.mongodb.com/manual/reference/method/db.killOp/)

> [currentop-index-creation](https://docs.mongodb.com/manual/reference/method/db.currentOp/#currentop-index-creation)
