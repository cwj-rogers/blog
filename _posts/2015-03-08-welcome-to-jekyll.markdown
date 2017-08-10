---
layout: post
title:  yii 查询构建器
date:   2016-07-18 22:21:49
categories: Yii
---
[TOC]
###批量插入：
```php
Yii::$app->db->createCommand()->batchInsert(UserModel::tableName(), ['user_id','username'], [
    ['1','test1'],
    ['2','test2'],
    ['3','test3'],
])->execute();
```
