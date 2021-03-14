---
title: Mongodb 性能优化
categories:
  - mongodb
  - database
---

## mongodb 创建索引和查询需要遵循的原则

### 创建索引需要遵循的规则


### 查询语句需要遵循的规则

1. 对于同一字段的查询，in 查询要优于 or 查询；
2. mongodb 在一次查询中只能使用一个索引，但 or 查询是个例外，or 可以对每个子句都使用索引，因为 or 查询实际上是将两次查询的结果合并，然后去除重复的文档；
3. 对于正则匹配的查询语句，前缀匹配加上大小写敏感的性能是最优的；

## explain command

同 mysql 类似，mongodb 也提供了一个 explain 的命令，用于帮助我们优化查询的性能

```sql
db.collection.find().explain()
db.keys.find( { x : { $in : [ 3, 4, 50, 74, 75, 90 ] } } ).explain( "executionStats" )
```

>[mongodb explain command](https://docs.mongodb.com/manual/reference/explain-results/)


## explain 返回信息

* queryPlanner（查询计划）：查询优化选择的计划细节和被拒绝的计划。其可能包括以下值：
* queryPlanner.namespace－一个字符串，运行查询的指定命名空间
* queryPlanner.indexFilterSet－一个布尔什，表示MongoDB在查询中是否使用索引过滤
* queryPlanner.winningPlan－由查询优化选择的计划文档
* winningPlan.stage－表示查询阶段的字符串
* winningPlan.inputStage－表示子过程的文档
* winningPlan.inputStages－表示子过程的文档数组
* queryPlanner.rejectedPlans－被查询优化备选并被拒绝的计划数组
* executionStats,（执行状态）：被选中执行计划和被拒绝执行计划的详细说明：
* queryPlanner.nReturned－匹配查询条件的文档数
* queryPlanner.executionTimeMillis－计划选择和查询执行所需的总时间（毫秒数）
* queryPlanner.totalKeysExamined－扫描的索引总数
* queryPlanner.totalDocsExamined－扫描的文档总数
* queryPlanner.totalDocsExamined－扫描的文档总数
* queryPlanner.executionStages－显示执行成功细节的查询阶段树
* executionStages.works－指定查询执行阶段执行的“工作单元”的数量
* executionStages.advanced－返回的中间结果数
* executionStages.needTime－未将中间结果推进到其父级的工作周期数
* executionStages.needYield－存储层要求查询系统产生的锁的次数
* executionStages.isEOF－指定执行阶段是否已到达流结束
* queryPlanner.allPlansExecution－包含在计划选择阶段期间捕获的部分执行信息，包括选择计划和拒绝计划
* serverInfo,（服务器信息）：MongoDB实例的相关信息：
* serverInfo.winningPlan－使用的执行计划
* winningPlan.shards－包括每个访问片的queryPlanner和serverInfo的文档数组
* responseLength: 返回的文档的大小，该值过大会影响性能


## 什么是 Query Plans?

mongodb 自带查询优化器，它会基于已有的索引去执行查询和选择最有效率的查询计划。

![query plan logic](https://docs.mongodb.com/manual/_images/query-planner-diagram.bakedsvg.svg)

## Stage 类型

* COLLSCAN: 全表扫描
* IXSCAN: 索引扫描
* FETCH: 根据索引去检索指定document
* SHARD_MERGE: 各个分片返回数据进行merge
* SORT: 表明在内存中进行了排序
* SORT_MERGE: 表明在内存中进行了排序后再合并
* LIMIT: 使用limit限制返回数
* SKIP: 使用skip进行跳过
* IDHACK: 针对_id进行查询
* SHARDING_FILTER: 通过mongos对分片数据进行查询
* COUNT: 利用db.coll.count()之类进行count运算
* COUNTSCAN: count不使用用Index进行count时的stage返回
* COUNT_SCAN: count使用了Index进行count时的stage返回
* SUBPLA: 未使用到索引的$or查询的stage返回
* TEXT: 使用全文索引进行查询时候的stage返回

> [mongodb indexes](https://docs.mongodb.com/manual/indexes/)

## 索引优化的例子

假如数据库有一个叫 coupon 的表

```ruby
class Coupon
	field :user_id, type: string
	field :type, type: Integer
	field :created_at, type: DateTime
	...
end
```

有如下的查询语句：

```ruby
Coupon.where(type: 1)
      .order_by(:created_at => 'desc')
      .where(:user_id.ne => nil)
      .limit(200)
```

应该如何设计索引才更有效呢？

先查看该表一共有多少数据：

```sql
mgset-8091301:PRIMARY> db.coupons.stats()
{
	"ns" : "readio_server_production.coupons",
	"size" : 16967382,
	"count" : 123419,
	...
```


一共有12万左右的数据，基于已有的索引，查询性能是怎么样的呢？

```sql
db.coupons.find({type: 1, user_id: {$ne: null}}).sort({created_at: 1}).explain("executionStats")

	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 547,
		"executionTimeMillis" : 217,
		"totalKeysExamined" : 111335,
		"totalDocsExamined" : 547,
		"executionStages" : {
			"stage" : "SORT",
			"nReturned" : 547,
			"executionTimeMillisEstimate" : 201,
			"works" : 111885,
			"advanced" : 547,
			"needTime" : 111337,
			"needYield" : 0,
			"saveState" : 874,
			"restoreState" : 874,
			"isEOF" : 1,
			"invalidates" : 0,
			"sortPattern" : {
				"created_at" : 1
			},
			"memUsage" : 110550,
			"memLimit" : 33554432,
```

从两项指标可以看出该查询语句的性能是很差的，totalKeysExamined 超过了11万，stage 是 SORT，表明在内存中进行了排序，
原因是 created_at 字段上没有索引。

先尝试给 type 字段加上索引

```sql
db.coupons.createIndex({type: 1})

	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 547,
		"executionTimeMillis" : 45,
		"totalKeysExamined" : 11825,
		"totalDocsExamined" : 11825,
		"executionStages" : {
			"stage" : "SORT",
			"nReturned" : 547,
			"executionTimeMillisEstimate" : 20,
			"works" : 12375,
			"advanced" : 547,
			"needTime" : 11827,
			"needYield" : 0,
			"saveState" : 190,
			"restoreState" : 190,
			"isEOF" : 1,
			"invalidates" : 0,
			"sortPattern" : {
				"created_at" : 1
			},
```

给 type 字段加上索引还是有效果的，`totalKeysExamined` 降低到了1万行左右，执行的时间也变为
原来的五分之一，但是因为created_at没有索引，依然会在内存中排序。

为了避免在内存中排序，考虑加上 `type + created_at` 的组合索引，

```sql
db.coupons.createIndex({type: 1, created_at: -1})

	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 547,
		"executionTimeMillis" : 21,
		"totalKeysExamined" : 11825,
		"totalDocsExamined" : 11825,
		"executionStages" : {
			"stage" : "FETCH",
```

扫描索引的行数虽然没变，但是 stage 已经变为 FETCH，说明已经不会在内存中排序了。

如果加上 `type + user_id + created_at` 的组合索引又会怎么样呢？

```sql
db.coupons.createIndex({type: 1, user_id: 1, created_at: -1});

	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 547,
		"executionTimeMillis" : 6,
		"totalKeysExamined" : 548,
		"totalDocsExamined" : 547,
		"executionStages" : {
			"stage" : "SORT",
			"nReturned" : 547,
			"executionTimeMillisEstimate" : 0,
			"works" : 1098,
			"advanced" : 547,
			"needTime" : 550,
			"needYield" : 0,
			"saveState" : 14,
			"restoreState" : 14,
			"isEOF" : 1,
			"invalidates" : 0,
			"sortPattern" : {
				"created_at" : 1
			},
			"memUsage" : 110550,
```

扫描索引的行数变为 500 行左右，查询时间又减半，但是 stage 却变为了 SORT，created_at 不是加上了索引么，
为什么还会在内存中排序呢？

>The inequality operator $ne is not very selective since it often matches a large portion of the index. As a result, in many cases, a $ne query with an index may perform no better than a $ne query that must scan all documents in a collection

$ne 查询通常不能很有效的利用索引，user_id 用了 $ne 查询条件后，这个查询就不能有效的使用这个索引，所以还会在内存中排序。既然 created_at 没有效果，我们可以尝试在这个组合字段中去掉 created_at，然后再看下查询的效率：

```sql
db.coupons.createIndex({type: 1, user_id: 1});

"executionStats" : {
	"executionSuccess" : true,
	"nReturned" : 547,
	"executionTimeMillis" : 5,
	"totalKeysExamined" : 548,
	"totalDocsExamined" : 547,
	"executionStages" : {
		"stage" : "SORT",
		"nReturned" : 547,
		"executionTimeMillisEstimate" : 0,
		"works" : 1098,
		"advanced" : 547,
		"needTime" : 550,
		"needYield" : 0,
		"saveState" : 14,
		"restoreState" : 14,
		"isEOF" : 1,
		"invalidates" : 0,
		"sortPattern" : {
			"created_at" : 1
		},
```

从结果上看，和之前加了 created_at 的组合索引的效率是一样的，所以我们可以看到，这个查询条件的最佳索引就是 `type + user_id`
