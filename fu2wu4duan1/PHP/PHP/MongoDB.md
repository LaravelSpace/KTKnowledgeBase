## PHP手册--MongoDB

首先需要注意这篇文档使用的PHP版本是PHP7，如果是别的版本的PHP不一定适用。

### 安装

MongoDB的扩展是需要自己额外安装的，这里推荐使用pecl来安装：`pecl install mongodb`。安装完成后在php.ini配置文件里添加MongoDB扩展`extension=mongodb.so`。保存后使用`php -i`命令检查一下扩展是否启用了。

如果觉得原生的扩展不好用，可以使用mongodb官方提供的composer扩展包。关于扩展包的文档可以在mongodb官方网站的[Drivers](https://docs.mongodb.com/drivers/)标签下找到，Drivers目录下面有官方提供的不同语言的扩展包。在安装完PHP的MongoDB扩展之后，就可以直接通过composer命令`composer require mongodb/mongodb`安装了。这里需要注意一下PHP的MongoDB扩展的版本和composer扩展包版本的依赖关系。

### 相关类名

| 类名                        | 用途                   |
| --------------------------- | ---------------------- |
| MongoDB\Driver\Manager      | 连接数据库             |
| MongoDB\BSON\ObjectId       | 生成\_id               |
| MongoDB\Driver\BulkWrite    | 插入，修改，删除       |
| MongoDB\Driver\WriteConcern | 配置                   |
| MongoDB\Driver\WriteResult  | 插入，修改，删除的结果 |
| MongoDB\Driver\Query        | 查询                   |
| MongoDB\Driver\Cursor       | 查询的结果             |

### 连接

```php
$manager = new Manager('mongodb://homestead:secret@localhost:27017');
```

这样就创建了一个连接，创建连接涉及到Manager。在Manager的构造函数中，可以传入一个MongoDB的连接url，数据库访问地址是localhost，端口是27017，用户名是homestead，用户密码是secret。

### 辅助方法

```php
public function getCollectionName()
{
    return $this->database . '.' . $this->collection;
}
```

### 插入

插入需要用到BulkWrite，WriteConcern，WriteResult。BulkWrite类用于处理写入的信息，可以同时插入一条或者多条记录。WriteResult类里记录了插入操作处理的结果。

进行插入操作的时候使用的是BulkWrite的insert()方法。

#### 单条插入

```php
$insertField = ['_id' => new ObjectId(), 'name' => '111'];
$bulkWrite = new BulkWrite();
$insertId = $bulkWrite->insert($insertField);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite('db_name.collection_name', $bulk, $writeConcern);
```

BulkWrite类的insert()方法会处理要插入的数据，如果插入的数据没有`_id`字段的话就会生成一个，这个方法返回一个ObjectId对象，包含`_id`字段的信息。var_dump()出来就是下面这样的数据，可以使用\_\_toString()方法转换成文本。关于`_id`字段，详细的可以看MongoDB的相关信息。

```php
object(MongoDB\BSON\ObjectId)#124 (1) {
  ["oid"]=>
  string(24) "5fd6fda1df5c274bc51e1751"
}
```

BulkWrite处理完成的数据交给Manager的executeBulkWrite()方法处理，executeBulkWrite()方法的第一个参数是一个字符串`'db_name.collection_name'`，db_name表示数据库名，collection_name表示集合名。拼字符串的操作后续会被辅助方法getCollectionName()代替。executeBulkWrite()方法返回一个WriteResult对象。里面记录本次操作的信息，比如这里nInserted=1就表示刚才插入了一条数据。

```php
object(MongoDB\Driver\WriteResult)#133 (9) {
  ["nInserted"]=>
  int(1)
  ["nMatched"]=>
  int(0)
  ["nModified"]=>
  int(0)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#127 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

#### 多条插入

```php
$insertFieldBulk = [
    ['_id' => new ObjectId(), 'name' => '222', 'age' => 20, 'status' => 1],
    ['_id' => new ObjectId(), 'name' => '333', 'age' => 30, 'status' => 1],
    ['_id' => new ObjectId(), 'name' => '444', 'age' => 40, 'status' => 0],
    ['_id' => new ObjectId(), 'name' => '555', 'age' => 40, 'status' => 0],
];

$bulkWrite = new BulkWrite();
$insertIdList = [];
foreach ($insertFieldBulk as $index => $itemInsertField) {
    $insertIdList[$index] = $bulkWrite->insert($itemInsertField);
}

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

$insertIdList和$writeResult的输出如下，这里插入了4条数据。

```php
// $insertIdList
array(4) {
  [0]=>
  object(MongoDB\BSON\ObjectId)#133 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb741"
  }
  [1]=>
  object(MongoDB\BSON\ObjectId)#134 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb742"
  }
  [2]=>
  object(MongoDB\BSON\ObjectId)#135 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb743"
  }
  [3]=>
  object(MongoDB\BSON\ObjectId)#136 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb744"
  }
}
// $writeResult
object(MongoDB\Driver\WriteResult)#139 (9) {
  ["nInserted"]=>
  int(4)
  ["nMatched"]=>
  int(0)
  ["nModified"]=>
  int(0)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#131 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

### 查询

查询需要用到Query，Cursor。Query类用于处理查询条件，Cursor类用于记录查询的结果。

Query类的$filter参数用于设置查询的筛选条件，$options参数可以设置limit，skip，sort等条件。关于查询语句的写法可以参考mongodb官方提供的[查询文档](https://docs.mongodb.com/manual/tutorial/query-documents/)，还有[操作符文档](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)。当查询条件为空时，就会查询全表。

#### 通过id查询

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$filter = ['_id' => new ObjectId($objectId)];
$options = [];
$query = new Query($filter, $options);

$manager = new Manager($this->uri);
$cursor = $manager->executeQuery($this->getCollectionName(), $query);
```

Manager的executeQuery()方法执行查询并返回一个Cursor对象。可以调用Cursor类的toArray()方法，将Cursor对象转换成数组，数组中就是查询的结果，结果以对象的形式呈现。

```php
array(1) {
  [0]=>
  object(stdClass)#133 (4) {
    ["_id"]=>
    object(MongoDB\BSON\ObjectId)#132 (1) {
      ["oid"]=>
      string(24) "5fd6fda1df5c274bc51e1751"
    }
    ["name"]=>
    string(3) "222"
    ["age"]=>
    int(10)
    ["status"]=>
    int(1)
  }
}
```

#### 通过条件查询

单条件和操作符：

```php
// 查询name=222的记录
$filter = ['name' => '222'];
// 查询age大于20的记录，$gt表示大于
// 类似的，$lt表示小于，$gte表示大于等于
$filter = ['age' => ['$gt' => 20]];
```

多条件之间的与和或的关系：

```php
// age>20 and status=0
$filter = [
    'age'    => ['$gt' => 20],
    'status' => 0
];
// age>20 or status=0
$filter = [
    '$or' => [
        'age'    => ['$gt' => 20],
        'status' => 0
    ]
];
// name='222' and (age>20 or status=0)
$filter = [
    'name' => '222'
    '$or' => [
        'age'    => ['$gt' => 20],
        'status' => 0
    ]
];
```

### 更新

更新需要用到BulkWrite，WriteConcern，WriteResult。BulkWrite类用于处理更新的信息，可以同时修改一条或者多条记录。WriteResult类里记录了更新操作处理的结果。

进行更新操作的时候使用的是BulkWrite的update()方法。更新这里需要注意一下$options参数的multi和upsert参数。multi=false时只会更新符合条件的第一条记录，multi=true时会更新全部符合条件的记录。upsert=false时找不到匹配的记录不会新建一条记录，upsert=true时找不到匹配的记录会新建一条记录。

更新同时涉及查询和写入操作，查询条件的写法和上文中单纯的查询操作的写法是一样的，但是，写入操作的写法和单纯的插入操作是不一样的，写入操作也是有类似查询条件的操作符一样的东西，详细的看mongodb官方提供的[更新文档](https://docs.mongodb.com/manual/reference/command/update/#update-with-an-aggregation-pipeline)。

#### 通过id更新

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$bulkWrite = new BulkWrite;
$filter = ['_id' => new ObjectId($objectId)];
$newObj = ['$set' => ['name' => '222222']];
$options = ['multi' => true, 'upsert' => false];
$bulkWrite->update($filter, $newObj, $options);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

$newObj参数就是操作数据的参数，样例里的`$set`操作符表示追加或者修改字段，常用的还有`$unset`操作符表示移除字段。样例操作的结果$writeResult如下，nMatched表示匹配到几条数据，nModified表示修改了几条数据。

```php
object(MongoDB\Driver\WriteResult)#132 (9) {
  ["nInserted"]=>
  int(0)
  ["nMatched"]=>
  int(1)
  ["nModified"]=>
  int(1)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#129 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

#### 通过条件更新

```php
$bulkWrite = new BulkWrite;
// update set name='222' where name='111'
$filter = ['name'=>'111'];
$newObj = ['$set' => ['name' => '222']];
// 更新第一条
// $options = ['multi' => false, 'upsert' => false];
// 更新全部
$options = ['multi' => true, 'upsert' => false];
$bulkWrite->update($filter, $newObj, $options);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

### 删除

更新需要用到BulkWrite，WriteConcern，WriteResult。BulkWrite类用于处理删除的信息，可以同时删除一条或者多条记录。WriteResult类里记录了删除操作处理的结果。

进行更新操作的时候使用的是BulkWrite的delete()方法。这里需要注意一下$options参数的limit参数。limit=false时会删除全部符合条件的记录，limit=true时只会删除符合条件的第一条记录。

删除操作的查询条件的写法和上文中单纯的查询操作的写法是一样的，但是这里特别要注意，如果删除操作的查询条件时空的，会删除全部的数据。

#### 通过id删除

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$bulkWrite = new BulkWrite;
$filter = ['_id' => new ObjectId($objectId)];
$deleteOptions = ['limit' => false];
$bulkWrite->delete($filter, $deleteOptions);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

#### 通过条件删除

```php
$bulkWrite = new BulkWrite;
$filter = ['name'=>'111'];
$deleteOptions = ['limit' => false];
$bulkWrite->delete($filter, $deleteOptions);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

