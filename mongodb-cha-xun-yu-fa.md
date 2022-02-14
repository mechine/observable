# mongodb 查询语法

### 过滤：

`{"username":"张三"}  = where username = '张三'`

`{"age":{$lt:30}} = where age < 30`&#x20;

`{"age":{$lte:30}} = where age <= 30`

`{"age":{$gt:30}} = where age > 30`

`{"age":{$gte:30}} = where age >= 30`

#### `按照类型过滤：`

({"title" : {$type : 'string'\}})

### `正则：`

MongoDB使用$regex操作符来设置匹配字符串的正则表达式，使用PCRE(Pert Compatible Regular Expression)作为正则表达式语言。

regex操作符 {:{$regex:/pattern/，$options:’’\}} {:{$regex:’pattern’，$options:’’\}} {:{$regex:/pattern/\}} 正则表达式对象 {: /pattern/} $regex与正则表达式对象的区别:

在$in操作符中只能使用正则表达式对象，例如:{name:{$in:\[/^joe/i,/^jack/\}} 在使用隐式的$and操作符中，只能使用$regex，例如:{name:{$regex:/^jo/i, $nin:\['john']\}} 当option选项中包含X或S选项时，只能使用$regex，例如:{name:{$regex:/m.\*line/,$options:"si"\}} $regex操作符的使用

$regex操作符中的option选项可以改变正则匹配的默认行为，它包括i, m, x以及S四个选项，其含义如下

i 忽略大小写，\{{$regex/pattern/i\}}，设置i选项后，模式中的字母会进行大小写不敏感匹配。 m 多行匹配模式，\{{$regex/pattern/,$options:'m'}，m选项会更改^和$元字符的默认行为，分别使用与行的开头和结尾匹配，而不是与输入字符串的开头和结尾匹配。 x 忽略非转义的空白字符，{:{$regex:/pattern/,$options:'m'}，设置x选项后，正则表达式中的非转义的空白字符将被忽略，同时井号(#)被解释为注释的开头注，只能显式位于option选项中。 s 单行匹配模式{:{$regex:/pattern/,$options:'s'}，设置s选项后，会改变模式中的点号(.)元字符的默认行为，它会匹配所有字符，包括换行符(\n)，只能显式位于option选项中。 使用$regex操作符时，需要注意下面几个问题:

i，m，x，s可以组合使用，例如:{name:{$regex:/j\*k/,$options:"si"\}} 在设置索弓}的字段上进行正则匹配可以提高查询速度，而且当正则表达式使用的是前缀表达式时，查询速度会进一步提高，例如:{name:{$regex: /^joe/}

`包含`

{post\_text:{$regex:"runoob"\}}

{post\_text:/runoob/}

{post\_text:{$regex:"runoob",$options:"$i"\}}

{tags:{$regex:"run"\}}



### 组合：

#### and

`{"username":"张三","age":30}`

#### **or**

{$or: \[`{"username":"张三"},{"username":"李四"}}`

#### 组合查询

`{"age": {$gt:18}, $or: [{"username": "张三"},{"username": "李四"}]}`

### 筛选排序：

db.COLLECTION\_NAME.find().limit(NUMBER)

db.COLLECTION\_NAME.find().limit(NUMBER).skip(NUMBER)

db.COLLECTION\_NAME.find().sort({KEY:1})



### 聚合：

$sum 计算总和。&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", num\_tutorial : {$sum : "$likes"\}}}])&#x20;

$avg 计算平均值&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", num\_tutorial : {$avg : "$likes"\}}}])&#x20;

$min 获取集合中所有文档对应值得最小值。

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", num\_tutorial : {$min : "$likes"\}}}])&#x20;

$max 获取集合中所有文档对应值得最大值。&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", num\_tutorial : {$max : "$likes"\}}}])&#x20;

$push 将值加入一个数组中，不会判断是否有重复的值。&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", url : {$push: "$url"\}}}])&#x20;

$addToSet 将值加入一个数组中，会判断是否有重复的值，若相同的值在数组中已经存在了，则不加入。 db.mycol.aggregate(\[{$group : {\_id : "$by\_user", url : {$addToSet : "$url"\}}}])&#x20;

$first 根据资源文档的排序获取第一个文档数据。&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", first\_url : {$first : "$url"\}}}])&#x20;

$last 根据资源文档的排序获取最后一个文档数据&#x20;

db.mycol.aggregate(\[{$group : {\_id : "$by\_user", last\_url : {$last : "$url"\}}}])

#### 管道

$project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。&#x20;

$match：用于过滤数据，只输出符合条件的文档。

$match使用MongoDB的标准查询操作。&#x20;

$limit：用来限制MongoDB聚合管道返回的文档数。&#x20;

$skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。&#x20;

$unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。&#x20;

$group：将集合中的文档分组，可用于统计结果。&#x20;

$sort：将输入文档排序后输出。&#x20;

$geoNear：输出接近某一地理位置的有序文档。

`db.articles.aggregate( [ { $match : { score : { $gt : 70, $lte : 90 } } }, { $group: { _id: null, count: { $sum: 1 } } } ] );`

### 新增：

```
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)
```



```
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

### 更新：

query : update的查询条件，类似sql update查询内where后面的。&#x20;

update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的 upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。&#x20;

multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。&#x20;

writeConcern :可选，抛出异常的级别。

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

举例：

```
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
```

### 删除：

query :（可选）删除的文档的条件。&#x20;

justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。&#x20;

writeConcern :（可选）抛出异常的级别。

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```



