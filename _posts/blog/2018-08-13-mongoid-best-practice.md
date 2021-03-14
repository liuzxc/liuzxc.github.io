---
title: Mongoid 最佳实践
categories:
  - mongodb
---


# 一、默认添加 `Mongoid::Timestamps`

对于新建表，请默认加上 `Mongoid::Timestamps`，对于日志类的数据，至少要加上 `Mongoid::Timestamps::Created`

```ruby
class User
  include Mongoid::Document
  include Mongoid::Timestamps
```

**好处：**

* 便于根据时间范围查数据
* 查找问题，解决 bug 的需要

# 二、对于 `boolean` 字段，需要设置默认值

```ruby
field :reviewed, type: Boolean, default: false
```

如果一开始没有加默认值，后期再添加上，会存在一个潜在的坑，旧的数据在数据库里面可能是 `NULL`，但是 `mongoid` 会把 `NULL` 转化为 `false`，所以判断一个布尔字段不为 `true` 查询语句就会成这样：

```ruby
reviewed: { "$in" => [false, nil]}
```

> 如果给一个未加默认字段的字段添加默认值，尽量将已产生的数据更新

# 三、对于新建的表要考虑加索引

新建的表虽然当前没有性能问题，但是随着数据的增多，慢查询造成的影响会越来越大，最终成为系统的不稳定因素。

> 一般情况下，数据量超过1万，就应该建立索引，一定是根据你的查询语句建立索引

# 四、谨慎使用 `default_scope`

`default_scope` 一般用于软删除或者是过滤业务上几乎不会用到的数据，滥用 `default_scope` 将会导致编码效率降低，问题的排查也会变得很困难。

# 五、控制 `scope` 的数量

只有针对频繁的查询才应该设置 `scope`，滥用 `scope` 将会导致又臭又长的链式调用。

针对同一字段设置多个 `scope` 尤其需要注意，存在潜在的坑：

如果 `User model` 存在以下 `scope`

```ruby
scope :not_guest, -> { where(:category.ne => 'guest') }
scope :not_special, -> { where(:category.ne => 'special') }
```

如果你想查询既不是 guest 用户也不是特殊用户课程数量，你也许或这样做：

```ruby
User.not_guest.not_special.count
```

实际生成的查询语句却是这样：

```log
{"count"=>"users", "query"=>{"deleted_at"=>nil, "category"=>{"$ne"=>"special"}}}
```

`not_extra` 被忽略掉了，这是 `mysql` 和 `mongodb` 的不同之处。

# 六、不要同时使用 `order` 和 `distinct`

例如：

```ruby
User.order(updated_at: 'desc').distinct(:open_id)
```

对应的查询语句：

```log
{"distinct"=>"users", "key"=>"open_id", "query"=>{"deleted_at"=>nil}}
```

排序已经被丢弃了，可以这样做

```ruby
User.order(updated_at: 'desc').pluck(:open_id).uniq
```

# 七、使用 Mongoid::Enum 的时候应该设置默认值

枚举类型通常用在表示不同的类别或者状态，使用的时候一定要加默认值，否则将会默认使用列表里面的第一个元素作为默认值，这也许不是你想要的

```ruby
# 如果不加 default，:manager 将作为默认值
enum :roles, [:manager, :administrator], default: :administrator

# 如默认值不在列表当中，需要指定 required 为 false
enum :roles, [:manager, :administrator], {default: nil, required: false}
```

# 八、更改索引时需要删除旧的索引

使用 mongoid 更改已存在的索引时，mongoid 会新建索引而不会删除旧的索引，需要手动删除旧的索引

```sh
#创建索引的命令
rake db:mongoid:create_indexes

#删除所有的索引
rake db:mongoid:remove_indexes

#删除未定义的索引
rake db:mongoid:remove_undefined_indexes
```

> [mongoid indexing](https://mongoid.github.io/old/en/mongoid/docs/indexing.html)

# 九、按需加载所需要的数据

对查询语句执行 explain 后，返回的额信息里会带有 responseLength，该值过大会影响性能，所以查询语句不仅要尽量匹配索引，还要限制返回文档的
大小，尽量多的使用 only 和 limit，不要一次性的拉取全部的数据，尽量按需加载。

>[system.profile.responseLength](https://docs.mongodb.com/manual/reference/database-profiler/#system.profile.responseLength)

# 十、distinct 性能优于 pluck

根据 benchmark 的数据显示，distinct 的速度是 pluck 的两倍。
