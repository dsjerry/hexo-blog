---
title: PHP面向对象进阶
date: 2020-06-26 21:56:03
tags: php
categories: 后端学习
toc: true
---

这篇包括类常量、静态成员、封装、继承、接口

### 类常量与静态成员

如果希望类中定义的成员被所有对象共享，可以使用类常量或静态成员来实现

#### 类常量<!-- more -->

- 定义后的类常量值不变，在类中使用 `const` 关键字定义类常量

  ```php
  const 类常量名 = "常量值";
  ```

  在访问类常量的时候，使用格式 **类名::常量名称** ，**::** 称为范围解析操作符

  ```php
  class Student{
      const SCHOOL = "火星学院";
  }
  echo Student::SCHOOL;
  ```

  - 在类内也可以访问类常量，使用 `self` 关键字代替类名
  - 如 `self::SCHOOL` ，这样避免了修改类名之后要修改类中的代码的麻烦

#### 静态成员

- 使用 `static` 关键字来修饰静态成员，属于类的成员
- 通过类名直接访问，不需要实例化对象

```php
class Student{
    public static $msg;
    public static function show(){
        echo '信息：' . self::$msg;	// 类内访问静态属性
    }
    public static function test(){
        self::show();	// 类内调用静态方法
    }
}
Student::$msg = 'PHP学习';	// 类外访问静态属性
Student::show();	// 类外调用静态方法
```



### 继承和封装

为了保护数据不被调用者随意修改、防止重复定义

#### 继承

- 在现有的类的基础上去构造一个新的类。这个新类叫作 *子类*，原有的类叫作 *父类*
- 使用 `extends` 关键字
- 只允许单继承，即是每个子类只能有一个父类

```php
class Animal{
    public $name;
    public function shout(){
        echo $this -> name . '发出叫声';
    }
}
class Cat extends Animal{
    public function __construct($name){
        $this -> name = $name;
    }
}
$cat = new Cat("Tom");
$cat -> shout();	// Tom发出叫声
```

- 当子类有和父类相同名字的成员的时候，子类成员会覆盖父类成员

##### traits 关键字

- 使得PHP可以自由复用成员属性和方法，减少单继承的限制

  ```php
  class Animal{}
  trait Cat{
      public function shout(){
          echo "喵喵";
      }
  }
  class TomCat extends Animal{
      use Cat;
      public function __construct(){
          $this -> shout();
      }
  }
  $tom = new TomCat();	// 输出结果：喵喵
  ```

  > 当子类、父类、traits有相同名称的成员的时候，优先级是：子类 > traits > 父类

  可以给 *traits* 中同名的成员取别名，例如有 Cat 和 Dog 两个Traits 中有相同的 shout()

  ```php
  use Cat, Dog{
      Cat::shout insteadof Dog;  // 将左边的 Trait 指定的成员代替给右边
      Dog::shout as cry;	//  将右边指定的成员名代替左边的
  }
  ```

  - 当执行 cry()，的时候，实际上是执行 Dog中的 shout()

#### 封装

- 隐蔽程序内部的细节，仅对外开放接口
- 类的封装通过访问控制修饰符实现的
  - public：公有修饰符（同一类内、子类、类外 可访问）
  - protected：保护成员修饰符（同一类内、子类 可访问）
  - private：私有修饰符（同一类内 可访问）

在PHP 4 中的所有属性都是用 `var` 声明的，效果和 `public` 一样，以后的版本也兼容，但是会转换为 public

```php
class User
{
    public $name = 'jerry';
    protected $tel = "16888";
    private $age = 18;
}
```

要想访问到 *protected* 和 *private* 的成员，有两种方法：公有方法，魔术方法

- 公有方法

  在一个类中声明一个公有方法，然后通过这个公有方法访问。例如：在上面 User 类中加入

  ```php
  public function test(){
      return $this -> age;
  }
  ```

  然后调用

  ```php
  $user = new User();
  echo $user -> age;	// 无法访问到私有属性
  echo $user -> test();	// 可以访问到age
  ```

- 魔术方法

  待整理......



#### 方法重写

- 重写方法时，要保持参数数量一致
- 子类中方法的访问级别等于或者小于父类中被重写的方法的访问级别

##### 非静态方法重写

```php
class Person{
    public function introduce(){
        echo __CLASS__;
    }
}
class Student extends Person{
    public function introduce(){
        echo __CLASS__;
    }
}
$s1 = new Student();
$s1 -> introduce();		// 输出结果：Student
```

- 魔术常量 `__CLASS__` 用于返回当前被调用的类名

- > 私有成员只能在本类内访问，所以父类的私有属性成员不能被重写

##### 静态方法重写

对静态成员的调用除了可以使用类名，还可以使用self、parent、static 关键字代替

- self：获取当前方法调用时所在的类

- parent：获取当前类的父类

- static：获取实际运行时方法所在的类，也称为后期静态绑定

  ```php
  class Person{
      public function show(){
          self::introduce();		// 优先访问父类方法
          static::introduce();	// 优先访问子类方法
      }
      public static function introduce(){
          echo '[Person]';
      }
  }
  class Student extends Person{
      public function show(){
          parent::show();		// 子类调用父类方法
      }
      public static function introduce(){
          echo '[Student]';
      }
  }
  $s1 = new Student();
  $s1 -> show();		// 输出结果： [Person][Student]
  ```

#### final 关键字

被 `final` 关键字修饰的类和成员方法不能被修改，final 类不能被继承，只能被实例化。

```php
class Person{
    protected final function show(){
        // final 方法不能被子类重写
    }
}
final class Student extends Person{
    // final 类不能被继承，只能被实例化
}
```

- 在代码上告诉了别人，在这里已经结束了，在代码层面上限制了类的使用方式，从而减少不必要的沟通



### 抽象类和接口

在项目来发中，经常需要定义方法来描述类的一些行为特征，但是这些行为特征在不同的情况之下又有不同的特点。所以在这种无法确定的情况下，就要使用抽象类和接口

#### 抽象类与抽象方法

抽象类用来定义某种行为，但是具体的实现需要 *子类* 来完成。使用 `abstract` 关键字来修饰抽象类

比如跑步这个行为，有恢复跑、基础跑、长跑等多种跑步方式

```php
abstract class 类名{		// 定义抽象类
    public abstract function 方法名();		// 定义抽象方法
}
```

在使用的时候需要注意：

- 抽象方法是只有方法声明而没有方法体的特殊方法
- 含有抽象方法的类必须被定义为抽象类
- 抽象类中可以有非抽象方法、成员属性和常量
- 抽象类不能被实例化，只能被继承
- 子类继承抽象类时必须实现抽象方法，否则也必须定义成抽象方法有下一个继承实现

抽象类中的抽象方法被声明为 *protected*，那么子类中实现的方法可以声明为 *protected* 或者是 *public*，而不能是 *private*。就是说范围变大了

```php
abstract class sport{
    public abstract function type();
}
class Run extends sport{
    public function type(){
        echo "长跑";
    }
}
$running = new Run();
$running -> type();		// 输出结果：长跑
```

- 在使用继承抽象类的子类中，还可以添加其他东西。。但是在实现的时候，参数必须和定义抽象方法的参数一样
- 抽象实现了，当然就不用抽象啦

#### 接口

如果抽象类中的 *所有* 方法都是 *抽象方法*，那这个类就叫做**接口**，关键字`interface` 

在接口中，所有的方法只能是公有的，不能使用 final 关键字来修饰

```php
interface 接口名{
    public function 方法名();
}
```

- 记得！没有具体的函数体！
- 因为接口所有方法都是抽象的，所以省略掉 abstract 关键字
- 接口方法体没有具体实现，所以需要通过某个类使用 `implements` 关键字来实现接口

```php
interface ComInterface{
    public function connect();
    public function transfer();
    public function disconnect();
}
class MobilePhone implements ComInterface{
    public function connect(){
        echo "连接";
    }
    public function transfer(){
        echo "传输";
    }
    public function dsiconnect(){
        echo "断开连接";
    }
}
```

- *MobilePhone* 类中必须实现 *ComInterface* 接口中定义的所有方法

- 一个类可以实现多个接口，用逗号隔开，但是接口中的方法不能重命名

- 接口中可以定义常量，与类常量用法相同，但是不能被子类或子接口覆盖

- 类也可以在继承的时候实现接口

  ```php
  class MobilePhone extends Phone implements ComInterface{
      
  }
  ```

#### extends 和 implements 的区别

- extends：继承一个类来创建子类
- implements：一个类通过这个关键字声明自己使用一个或多个接口，要通过重写才能实现

继承之后使用多个接口：

```php
class A extends B implements C,D,E
```

> 我感觉就是，extends 使用在不是全是抽象类的地方，可以实现普通继承，也可以实现接口继承，不过就是还不在接口的范畴之内，若果这个类里面都是抽象方法，那就是接口了，所以就要使用 implements

#### 多态与类型约束

- 多态：实现同一操作作用于不同的对象，产生不同的执行效果
- 类型约束：在成程序实现多态时限制传入的参数必须是某个类或接口

获取商品价格

```php
function price(Goods &g){
    return $g -> getName() . '的价格是' . $g -> getPrice();
}
```

- $g 表示用户传入的具体商品对象
- 为了保证每一个传入的对象必须含有 getName() 方法和 getPrice() 方法，将参数 $g 的类型指定为 Goods 接口

定义接口

```php
interface Goods{
    public function getName();
    public function getPrice();
}
```

实现接口

```php
// 定义 Phone 类实现 Goods接口
class Phone implements Goods{
    public function getName(){
        return '手机';
    }
    public function getPrice(){
        return '2000';
    }
}
// 定义 Computer 类实现 Goods 接口
class Computer implements Goods{
    public function getName(){
        return '电脑';
    }
    public function getPrice(){
        return '6000';
    }
}
// 等等。。。。实现Goods接口
```

向 price() 里面传入不同的 *商品对象* ，就可以得到对应商品的价格

```php
// 实例化商品类
$goods = new Phone();
// 获取对应的商品价格，输出结果：手机的价格是2000
price( $goods );
```

总的来说就是：有接口，实现接口，实例化对象，通过函数使用