
### 一、数据模型介绍

在 mongodb 中，数据模型的 schema 定义是相对灵活的。不像是 mysql，需要在插入数据之前定义并声明好数据表的 schema。 mongodb 的 collection 并不关注 document 的结构。但是在实际应用中，一个 collection 中的 document 往往有着相同的 schema。

数据建模的关键是要平衡应用程序的需求，数据库引擎的性能特征和数据检索的模式。
当设计数据模型的时候，总要考虑应用程序对于数据的使用（例如查询、更新和数据处理）而不是继承数据自身的结构。


### 二、文档结构

为 mongodb 应用程序设计数据模型的关键围绕文档结构以及应用程序如何表示数据之间的关系。

> 引用

引用通过将链接或引用从一个文档包含到另一个文档来存储数据之间的关系。应用程序可以解析这些引用以访问相关数据。从广义上讲，这些是标准化数据模型。

![image](https://docs.mongodb.com/v3.6/_images/data-model-normalized.bakedsvg.svg)

> 嵌入数据

嵌入式文档通过在单个文档结构中存储相关数据来捕获数据之间的关系。 MongoDB文档可以将文档结构嵌入文档中的字段或数组中。这些非规范化数据模型允许应用程序在单个数据库操作中检索和操作相关数据。

![image](https://docs.mongodb.com/v3.6/_images/data-model-denormalized.bakedsvg.svg)


### 三、写操作的原子性

在 mongodb 中，文档级别的写操作是原子级别的。并且，没有单个的写操作可以原子的影响到超过一个文档或者多个集合。一个非规范化的数据模型将相关的数据内嵌到一个单个的文档中。这将有利于原子的写操作，因为单个写操作可以插入或者更新实体的数据。而规范化的数据会将数据拆分为多个集合，并且需要多个非原子的写入操作。

但是，嵌入文档的模式可能会限制应用程序可以使用数据的方式，或者可能限制修改应用程序的方法。


### 四、数据的使用和性能

当设计数据模型的时候，需要考虑应用程序将如何使用你的数据库。例如，如果你的应用程序仅仅使用最近添加的文档，可以考虑使用封顶集合。或者 如果你的应用程序需要对一个集合进行读操作，添加索引将有助于提高性能。