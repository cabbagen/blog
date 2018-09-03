
#### 一、指定验证规则
当进行插入和更新操作的时候，mongodb 提供了 schema 校验的功能。而验证规则基于每一个集合。

* 当创建一个新的集合时，可以通过 [db.createCollection()](https://docs.mongodb.com/v3.6/reference/method/db.createCollection/#db.createCollection) 创建一个带有校验的集合。

* 当对一个存在的集合添加校验规则时，可以使用 [collMod](https://docs.mongodb.com/v3.6/reference/command/collMod/#dbcmd.collMod) 命令修改 validator 选项。 

* mongodb 也提供了下面两个相关的选项
    
    * validationLevel 选项：在对已存在的文档更新时，校验的严格程度
    
    * valadationAction 选项：当文档校验失败时，是应该抛出错误拒绝还是允许非法的文档。


#### 二、JSON Schema

在 mongodb3.6 版本中新增： [$jsonSchema](https://docs.mongodb.com/v3.6/reference/operator/query/jsonSchema/#op._S_jsonSchema) 运算符匹配根据给定的JSON模式验证的文档

````
db.createCollection("students", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "name", "year", "major", "gpa" ],
         properties: {
            name: {
               bsonType: "string",
               description: "must be a string and is required"
            },
            gender: {
               bsonType: "string",
               description: "must be a string and is not required"
            },
            year: {
               bsonType: "int",
               minimum: 2017,
               maximum: 3017,
               exclusiveMaximum: false,
               description: "must be an integer in [ 2017, 3017 ] and is required"
            },
            major: {
               enum: [ "Math", "English", "Computer Science", "History", null ],
               description: "can only be one of the enum values and is required"
            },
            gpa: {
               bsonType: [ "double" ],
               minimum: 0,
               description: "must be a double and is required"
            }
         }
      }
   }
})
````

#### 三、Query Expressions （查询表达式）

除了 JSON Schema 验证，mongodb 也支持使用查询筛选表达式进行验证。例如使用 [$near](https://docs.mongodb.com/v3.6/reference/operator/query/near/#op._S_near) 、[$where](https://docs.mongodb.com/v3.6/reference/operator/query/where/#op._S_where) 等。 

````
db.createCollection( "contacts",
   { validator: { $or:
      [
         { phone: { $type: "string" } },
         { email: { $regex: /@mongodb\.com$/ } },
         { status: { $in: [ "Unknown", "Incomplete" ] } }
      ]
   }
} )
````

#### 四、Bahavior （行为）

验证出现在更新文档和插入文档的过程中。对于之前已经存在于集合中的文档，只有在更新他们的时候，才会验证。

> Existing documents （已经存在的文档）

**validationLevel** 参数决定了 mongodb 的验证规则。

* 如果 **validationLevel** 是 **strict（default）**，mongodb 会在所有的插入和更新上调用验证规则。

* 如果 **validationLevel** 是 **moderate**，mongodb 会对在已满足验证规则的现有文档的插入和更新上调用验证规则。 对于**moderate**，不检查不符合验证标准的现有文档的更新是否有效

````
// 下面是一个例子

db.contacts.insert([
   { "_id": 1, "name": "Anne", "phone": "+1 555 123 456", "city": "London", "status": "Complete" },
   { "_id": 2, "name": "Ivan", "city": "Vancouver" }
])

db.runCommand( {
   collMod: "contacts",
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone", "name" ],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         name: {
            bsonType: "string",
            description: "must be a string and is required"
         }
      }
   } },
   validationLevel: "moderate"
} )

// contacts 集合现在有 moderate 级别的验证级别
// 如果更新 _id = 1, 文档符合 验证所定义的结构，所以会经过验证规则
// 如果更新 _id = 2, 文档不符合 验证所定义的结构，所以不会经过验证规则
````

* 如果需要禁掉验证，设置 **validationLevel** 为 **off**

> Accept or Reject Invalid Documents （接受 或者 拒绝 非法的文档）

**validationAction** 决定了 mongodb 如何处理不符合验证规则的文档

* 如果 **validationAction** 为 **error（default）**，mongodb 会拒绝插入或者更新不符合验证规则的文档。

* 如果 **validationAction** 为 **warn**，mongodb 会记录下违规，但是允许插入和更新文档。

#### 五、Restrictions （限制）

* 不能指定验证对于在 **admin**、**local**、**config** 数据库中的集合。

* 不能指定验证对于 **system.*** 集合

 

