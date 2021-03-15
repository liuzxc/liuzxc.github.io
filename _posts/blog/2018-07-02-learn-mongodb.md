---
title: Mongodb 常见问题处理
categories: blog
---

## 1. How to kill slow query?

```bash
db.currentOp()
# find opid
db.killOp(opid)
```

## 2. Mongo::Error::OperationFailure: Cursor not found

### 什么是 cursor？

cursor 并不是查询结果，可以理解成是数据在遍历过程中的内部指针，其返回的是一个资源或者说是数据读取的接口.
`db.collection.find()` 会返回一个 cursor，为了获取文档，你需要去遍历这个 cursor.

> [iterate-a-cursor](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/)

Mongodb server 会批量返回查询结果，其数据量不能超过 `maximum BSON document size`(默认 16 M).
`find()` 和 `aggregate()` 操作会默认批量的返回 101 个文档。

> [cursor-batches](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/#cursor-batches)

### 为什么 cursor 会找不到？

Mongodb server 会自动关闭10分钟后不活跃的 cursor，或者是客户处理完的所有的 cursor。所以如果一个 batch 内的
文档在10分钟内没有处理完，当前 cursor 会被关闭，之后处理完了，再用同一个 cursor 向服务器获取下一个 batch 的
cursor 时，这个 cursor 已经过期了，所以会报 cursor 找不到的错误。

> [closure-of-inactive-cursors](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/#closure-of-inactive-cursors)

### 解决方案

1. 使用 `cursor.noCursorTimeout()` 方法，去掉时间限制;
2. 使用 `batch_size()` 方法，减少批量返回的数据，避免单个 batch 处理时间过长;

## 3. Sort operation used more than the maximum 33554432 bytes of RAM.

如果 mongodb 需要排序获取文档，但是又没有用到索引，那么排序操作的所有文档的大小再加上少量的开销必须小于 32M，不然就会报错。

>If MongoDB cannot use an index to get documents in the requested sort order, the combined size of all documents in the sort operation, plus a small overhead, must be less than 32 megabytes.

有两种解决方案：

1. 创建适当的索引，让排序的字段可以用到索引；
2. 增加 sort 操作的内存限制

```sh
# 将 sort 操作可用内存增加到 64 M
db.adminCommand({setParameter:1, internalQueryExecMaxBlockingSortBytes: 67108864})
```

推荐使用第一种方案，第二种方案对服务器的内存有要求，如果内存资源本身很紧张，增加内存限制是比较危险的。

> [sort operations](https://docs.mongodb.com/manual/reference/limits/#Sort-Operations)
