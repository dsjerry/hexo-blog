---
title: PHP 面向对象-类与对象的使用
date: 2020-06-25 21:26:07
tags: php
categories: 后端学习
toc: true
---

和面向对象编程语言有所不同，PHP不是一种纯面向对象语言。但随着发展，PHP也向着这方面发展

### 面向对象的特征

- 封装性：将对象的属性和行为封装起来，不需要让外界知道具体的细节
- 继承性：主要描述类与类之间的关系；不需要重写原有类的代码，对原有类的功能进行扩展，提高代码的复用性、增强开发效率
- 多态性：指同一操作作用于不同对象，会产生不同的结果。大概就是一个类可以产生不同的对象

### 类的定义和实例化<!-- more -->

```php
class Animal{
    public $name;
    public function shout(){
        echo "哈哈哈";
    }
}
// 实例化（创建对象）
$person = new Animal();
```

### 对象的基本使用

#### 成员操作

创建对象之后，使用 `对象 -> 成员` 来访问属性和方法

- 成员属性

  ```php
  // 访问并赋值
  $person->name = "张三";
  
  // 删除属性值
  unset($person->name);
  
  // 判断属性是否设置值
  isset($person->name);
  
  // 判断属性是否被定义
  property_exists('Animal', 'name');
  ```

  > 属性值为 **null** 或使用 **unset( )** 删除后的 *isset()* 结果都是 false。unset() 删除的只是属性值，属性还在的

  

- 成员方法

  ```php
  $person->shout();
  ```

  特殊变量 **$this** 代表当前对象，用于完成对象内部成员之间的访问。只能在类定义的方法内使用，不能在类外使用
  
  ```php
  class Human{
      public $name = "张三";
      public function say($val){
          echo $this->name.$val."哈哈哈";
      }
  }
  $person = new Human();
  $person->say("说");
  ```
  
  > 这里的 *$this* 代表的是 *$person* 对象
  
  
  
- 可变类和可变成员

   与可变变量和可变函数类似，就是取个别名的意思

   声明一个新的变量来接收值，这值是类名、属性名、方法名的字符串形式

  ```php
  $humanbeing = 'Hunman';
  
  // 实例化对象
  $person = new $humanbeing();
  // 访问属性
  $beCall = 'name';
  echo $person -> $beCall;
  // 调用方法
  $shout = 'say';
  $person -> $shout("okok");
  ```

  > 在访问的时候 `->` 后面跟的是`$变量名称`，是要加上 *$* 的

#### 链式调用

一个函数的返回值是一个对象的时候，可以在前一个调用的后面继续调用其返回的对象中的方法

```php
class Goods{
    public $name = "电脑";
    public function start(){
        echo "开机";
        return $this;
    }
    public function open(){
        echo "打开应用";
        return $this;
    }
}
$computer = new Goods();
echo $computer -> name -> start() -> open();
```



#### 对象的特殊操作符

- 判断两个对象是否相等，使用 `==` 或 `===` 

  - == ：同一个类的实例，且属性和属性值相等

  - === ：必须是同一个实例

    ```php
    class Test{
        public $flag;
    }
    $a = $b = new Test();
    $c = new Test();
    var_dump( $a == $b );  // 输出结果：bool(true)
    var_dump( $a == $c );  // true
    var_dump( $a == $c );  // true
    var_dump( $a === $b );	// true
    var_dump( $a === $c );	// false
    ```

- 使用 `instanceof` 关键字判断对象是否是某个对象的实例

  ```php
  var_dump( $a instanceof Test );	 // true
  var_dump( $a instanceof other);  // false
  ```

#### 对象克隆

```php
class Test{
    public $flag = 1;
}
$a = new Test();
$b = $a;
$a -> flag = 3;
var_dump( $a -> flag );  // int(3)
var_dump( $b -> flag );  // int(3)
```

修改 $a 的值，$b 也会跟着改变，要想的到多个 *全等(===)* 对象并且它们的值不相互影响，使用 `clone` 关键字

```php
$b = clone $a;
var_dump( $a -> flag );  // int(3)
var_dump( $b -> flag );  // int(1)
```

在对象克隆的时候，如果想对新的对象的某些属性进行初始化，可以使用 *魔术方法 __clone*

> 在调用了 `clone` 的时候就会自动执行 `__clone`

- 魔术方法：不需要手动调用，在某些时刻自动执行
- PHP中所有的魔术方法都是以 `__` （两个下划线）开头。

在 Test 类中加入

```php
public function __clone(){
    $this -> flag = 'xixi';
}
```

**所以**

- 如果不改变 a 对象的值，它就是原本的值
- 给 a 赋了新的值，a 的值就会改变
- b 要是不想随着 a 改变，就要使用 clone 关键字
- 使用 clone 之后的 b，它的值就是原本的值 1
- 如果要想自定义使用 clone 后 b 的值，就要使用 __clone 



### 构造方法和析构方法

#### 构造方法

- 每一个类都有一个构造方法，在创建对象时自动调用。主要用于在创建对象的时候初始化功能

- 如果在类中没有显式的声明它，PHP会自动生成一个什么都没有的构造方法

  ```php
  function __construct(){
      // 初始化操作
  }
  ```

  默认情况下是 public ，可以省略

  ```php
  class Person{
      public $name;
      public function __construct($name = "xxx"){
          $this -> name = $name;
      }
      public function show(){
          echo $this -> name."正在吃饭";
      }
  }
  $p1 = new Person("李四");  // 张三正在吃饭
  $p2 = new Person();    // xxx在吃饭
  ```

  > 使用构造函数就可以实现传递不同的参数来实例不同的对象，这样就不用建立好多个类了

#### 析构方法

- 在对象被销毁之前自动调用，可以用作关闭文件、释放结果集等

- 在使用 *unset()* 的时候 或 PHP脚本执行结束自动释放对象时，自动执行

  ```php
  class Student{
      public function __destruct(){
          echo '正在执行析构方法';
      }
  }
  $xiaoming = new Student();
  unset( $xioaming );    // 这时候就会执行，输出结果
  ```

  不使用 *unset* 的话，例如在函数中实例化对象

  ```php
  function test(){
      $xioaming = new Student();
  }
  test();   // 函数执行结束，自动执行析构方法
  ```

  如果不希望对象在函数结束的手就被销毁，可以使用 *返回值* 接收对象

### 匿名类

......