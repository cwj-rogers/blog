---
layout: post
title:  yii behaviors 行为
date:   2017-07-10 22:21:49
categories: Yii
---
[behaviors使用参考手册](http://www.digpage.com/behavior.html)

使用行为可以在不修改现有类的情况下，对类的功能进行扩充。通过将行为绑定到一个类，可以使类具有行为本身所定义的属性和方法，`就好像类本来就有这些属性和方法一样`。 而且不需要写一个新的类去继承或包含现有类。

### 行为绑定小结
- 绑定的动作从Component发起；
- 静态绑定通过重载 yii\base\Componet::behaviors() 实现；
- 动态绑定通过调用 yii\base\Component::attachBehaviors() 实现；
- 行为还可以通过为 Component 配置 as 配置项进行绑定；
- 行为有匿名行为和命名行为之分，区别在于绑定时是否给出命名。 命名行为可以通过其命名进行标识，从而有针对性地进行解除等操作；
- 绑定过程中，后绑定的行为会取代已经绑定的同名行为；
- 绑定的意义有两点，一是为行为设置 $owner 。二是将行为中拟响应的事件的handler绑定到类中去。

### 跨域设置
![使用behaviors解决跨域限制](./QQ图片20170607171223.png)

### 行为里设置事件
```php
<?php
namespace app\Components;
use yii\db\ActiveRecord;
use yii\base\Behavior;
class MyBehavior extends Behavior
{
    // 重载events() 使得在事件触发时，调用行为中的一些方法
    public function events()
    {
        // 在EVENT_BEFORE_VALIDATE事件触发时，调用成员函数 beforeValidate
        return [
            ActiveRecord::EVENT_BEFORE_VALIDATE => 'beforeValidate',
        ];
    }

    // 注意beforeValidate 是行为的成员函数，而不是绑定的类的成员函数。
    // 还要注意，这个函数的签名，要满足事件handler的要求。
    public function beforeValidate($event)
    {
        // ...
    }
}
?>
```

### 静态方法绑定行为
```php
<?php
namespace app\models;
use yii\db\ActiveRecord;
use app\Components\MyBehavior;
class User extends ActiveRecord
{
    public function behaviors()
    {
        return [
            // 匿名的行为，仅直接给出行为的类名称
            MyBehavior::className(),
            // 名为myBehavior2的行为，也是仅给出行为的类名称
            'myBehavior2' => MyBehavior::className(),

            // 匿名行为，给出了MyBehavior类的配置数组
            [
                'class' => MyBehavior::className(),
                'prop1' => 'value1',
                'prop3' => 'value3',
            ],
            // 名为myBehavior4的行为，也是给出了MyBehavior类的配置数组
            'myBehavior4' => [
                'class' => MyBehavior::className(),
                'prop1' => 'value1',
                'prop3' => 'value3',
            ]
        ];
    }
}

//配置文件中绑定
[
    'as myBehavior2' => MyBehavior::className(),
    'as myBehavior3' => [
        'class' => MyBehavior::className(),
        'prop1' => 'value1',
        'prop3' => 'value3',
    ],
]
?>
```

### 动态方法绑定行为
```php
<?php
$Component->attachBehaviors([
    'myBehavior1' => new MyBehavior,  // 这是一个命名行为
    MyBehavior::className(),          // 这是一个匿名行为
]);
?>
```
