---
layout: post
title:  yii 查询构建器
date:   2016-07-18 22:21:49
categories: Yii
---
## 简单查询：
```php
<?php
one(): 根据查询结果返回查询的第一条记录。
all(): 根据查询结果返回所有记录。
count(): 返回记录的数量。
sum(): 返回指定列的总数。
average(): 返回指定列的平均值。
min(): 返回指定列的最小值。
max(): 返回指定列的最大值。
scalar(): 返回查询结果的第一行中的第一列的值。
column(): 返回查询结果中的第一列的值。
exists(): 返回一个值，该值指示查询结果是否有数据。
where(): 添加查询条件
with(): 该查询应执行的关系列表。
indexBy(): 根据索引的列的名称查询结果。
asArray(): 以数组的形式返回每条记录。
//Customer::find()->asArray()->one();    以数组形式返回一条数据；
?>
```
## 关联查询：
```php
<?php
ActiveRecord::hasOne()：返回对应关系的单条记录
ActiveRecord::hasMany()：返回对应关系的多条记录

//应用实例
//客户表Model：CustomerModel
//订单表Model：OrdersModel
//国家表Model：CountrysModel
//首先要建立表与表之间的关系
//在CustomerModel中添加与订单的关系

Class CustomerModel extends yiidbActiveRecord
{
    ...

    public function getOrders()
    {
        //客户和订单是一对多的关系所以用hasMany
        //此处OrdersModel在CustomerModel顶部别忘了加对应的命名空间
        //id对应的是OrdersModel的id字段，order_id对应CustomerModel的order_id字段
        return $this->hasMany(OrdersModel::className(), ['id'=>'order_id']);
    }

    public function getCountry()
    {
        //客户和国家是一对一的关系所以用hasOne
        return $this->hasOne(CountrysModel::className(), ['id'=>'Country_id']);
    }
    ....
}

// 查询客户与他们的订单和国家
CustomerModel::find()->with('orders', 'country')->all();

// 查询客户与他们的订单和订单的发货地址
CustomerModel::find()->with('orders.address')->all();

// 查询客户与他们的国家和状态为1的订单
CustomerModel::find()->with([
    'orders' => function ($query) {
        $query->andWhere('status = 1');
        },
        'country',
])->all();
?>
```

## 条件查询：
### [[and]]:将不同的条件组合在一起，用法举例：
```php
<?php
//SQL:`id=1 AND id=2`
$cond = ['and', 'id=1', 'id=2']

//SQL:`type=1 AND (id=1 OR id=2)`
$cond = ['and', 'type=1', ['or', 'id=1', 'id=2']]
?>
```
### [[or]]:
```php
<?php
//SQL:`(type IN (7, 8, 9) OR (id IN (1, 2, 3)))`
$cond = ['or', ['type' => [7, 8, 9]], ['id' => [1, 2, 3]] ]
?>
```
### [[not]]:
```php
<?php
//SQL:`NOT (attribute IS NULL)`
$cond = ['not', ['attribute' => null]]
?>
```
### [[between]]: not between 用法相同
```php
<?php
//SQL:`id BETWEEN 1 AND 10`
$cond = ['between', 'id', 1, 10]
?>
```
### [[in]]: not in 用法类似
```php
<?php
//SQL:`id IN (1, 2, 3)`
$cond = ['in', 'id', [1, 2, 3]]

//IN条件也适用于多字段
$cond = ['in', ['id', 'name'], [['id' => 1, 'name' => 'foo'], ['id' => 2, 'name' => 'bar']]]

//也适用于内嵌sql语句
$cond = ['in', 'user_id', (new Query())->select('id')->from('users')->where(['active' => 1])]
?>
```
### [[like]]:
```php
<?php
//SQL:`name LIKE '%tester%'`
$cond = ['like', 'name', 'tester']

//SQL:`name LIKE '%test%' AND name LIKE '%sample%'`
$cond = ['like', 'name', ['test', 'sample']]

//SQL:`name LIKE '%tester'`
$cond = ['like', 'name', '%tester', false]
?>
```
### [[exists]]: not exists用法类似
```php
<?php
//SQL:EXISTS (SELECT "id" FROM "users" WHERE "active"=1)
$cond = ['exists', (new Query())->select('id')->from('users')->where(['active' => 1])]
?>
```
### 运算符:
```php
<?php
//SQL:`id >= 10`
$cond = ['>=', 'id', 10]

//SQL:`id != 10`
$cond = ['!=', 'id', 10]
?>
```

## 封装条件
### findOne()和findAll():
```php
<?php
// 查询key值为10的客户
$customer = Customer::findOne(10);
$customer = Customer::find()->where(['id' => 10])->one();

// 查询年龄为30，状态值为1的客户
$customer = Customer::findOne(['age' => 30, 'status' => 1]);
$customer = Customer::find()->where(['age' => 30, 'status' => 1])->one();

// 查询key值为10的所有客户
$customers = Customer::findAll(10);
$customers = Customer::find()->where(['id' => 10])->all();

// 查询key值为10，11,12的客户
$customers = Customer::findAll([10, 11, 12]);
$customers = Customer::find()->where(['id' => [10, 11, 12]])->all();

// 查询年龄为30，状态值为1的所有客户
$customers = Customer::findAll(['age' => 30, 'status' => 1]);
$customers = Customer::find()->where(['age' => 30, 'status' => 1])->all();
?>
```
### 更新：
```php
<?php
//update();
//runValidation boolen 是否通过validate()校验字段 默认为true
//attributeNames array 需要更新的字段
$model->update($runValidation , $attributeNames);

//updateAll();
//update customer set status = 1 where status = 2
Customer::updateAll(['status' => 1], 'status = 2');

//update customer set status = 1 where status = 2 and uid = 1;
Customer::updateAll(['status' => 1], ['status'=> '2','uid'=>'1']);
?>
```
### 删除：
```php
<?php
$model = Customer::findOne($id);
$model->delete();

$model->deleteAll(['id'=>1]);
?>
```

## 批量插入：
```php
<?php
Yii::$app->db->createCommand()->batchInsert(UserModel::tableName(), ['user_id','username'], [
    ['1','test1'],
    ['2','test2'],
    ['3','test3'],
])->execute();
?>
```
## 查看执行sql
```php
<?php
$query = UserModel::find()->where(['status'=>1]);
echo $query->createCommand()->getRawSql();
?>
```
