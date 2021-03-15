---
layout: post
title: Java 基础
excerpt: java
categories: blog
comments: true
share: true
---

* [变量](#variables)
* [控制语句](#statement)
* [Java API](#java_api)
* [类](#class)
* [继承](#inherit)
* [抽象](#abstract)
* [接口](#interface)
* [构造器](#constructor)
* [静态](#static)
* [异常处理](#exception)
* [线程](#thread)

#### 变量
{: #variables}

Java 有两种变量：

1. primitive主数据类型（boolean, char, integer, byte, short, int, long, float, double)
2. 引用

* 变量命名规则： 必须以字母，下划线`_`或`$`符号开头；
* 实例变量有默认值，局部变量没有默认值；

变量的比较：

* `==`用于比较两个primitive主数据类型货判断两个引用是否引用同一个对象；
* `equals()`来判断两个对象是否在意义上相等（例如两个String对象是否带有相同的字节组合）;

> Integer 和 int 的区别: int 是 Java 提供的原始数据类型，而 Integer 是 Java 为 int 提供的封装类，它提供了很多整数相关的
操作方法；int 的默认值是0，而 Integer 的默认值是 null

#### 控制语句
{: #statement}

##### if...esle

1. if语句

```java
  int score = 90;
  if(score > 100){
    System.out.println(".....");
  }
  //以上可以简写为：
  if(score > 100)
    System.out.println(".....");
```

2. if...else 语句

```java
  int score = 90;
  if(score > 100){
    System.out.println(".....");
  } else {
    System.out.println(".....");
  }
```

3. if...else if...else

```java
  int score = 90;
  if (score > 100) {
    System.out.println(".....");
  } else if (score > 90) {
    System.out.println(".....");
  } else {
    ....
  }
```

##### switch

* switch语句中的条件表达式的值只能是int，string或枚举类型

```java
 int score = 90;
 switch(score){
 case 90 :
     System.out.println("A");
     break;
 case 50 :
     System.out.println("B");
     break;
 default:
     System.out.println("C");
     break;
 }
```

##### 循环

```java
int[] arr = {1,2,3,4,5};

for(int i=0;i<arr.length;i++) {...}

for(int i : arr) {...}

while(i < arr.length) {...}
```

可以通过 break 关键字跳出当前循环，如果要跳出嵌套循环该如何做呢？

1. 使用标号

```java
 ok:
 for(int i=0; i<=10 && !bo; i++){
    System.out.println("i:"+i);
    for(int j=0;j<=9;j++){
       System.out.println("j:"+j);
       if(j==8){
          break ok;
       }
    }
 }
```

2. 控制外层循环表达式

```java
boolean con = false;
for(int i=0; i<=10 && !con; i++){
System.out.println("i:"+i);
	for(int j=0;j<=9;j++){
	   System.out.println("j:"+j);
	   if(j==8){
	      con = true;
	      break ok;
	   }
	}
}
```

#### Java API
{: #java_api}

要使用API中的类，你必须要知道它被放在哪个包中，有两种方式指定 Java 使用的类：

1. import 把 import 放在源文件的最前面；

```java
import java.util.ArrayList;
```

2. 在程序中打出全名；

```java
java.util.ArrayList<String> myList = new java.util.ArrayList<String>();
```

ArrayList: ArrayList 不同于普通数组，创建时不需要指定大小，并且可以调用其提供的大量方法；

```java
import java.util.ArrayList;
class Array {
  public static void main(String[] args){
	  ArrayList<String> myList = new ArrayList<String>();
	  String a = new String("liu");
	  myList.add(a);
	  String b = "xing";
	  myList.add(b);
	  System.out.println("the array list length is: " + myList.size());
	  String c = myList.get(1);
	  System.out.println(c);
	  boolean isIn = myList.contains(b);
	  int i = myList.indexOf(a);
	  boolean isEmpty = myList.isEmpty();
  }
}
```

Math:

```java
Math.ceil(11.3) => 12 向上取整
Math.floor(11.6) => 11 向下取整
Math.round(11.5) => 12 四舍五入向下取整，算法为Math.floor(x+0.5)
```

#### 类
{: #class}

`public class Test{...}`

* 类名必须和文件名相同
* 一个文件可以有多个类，但是只能有一个是public的

##### 内部类

```ruby
class OuterClass {

  class InnerClass {
    void go() {...}
  }
}
```

1. 内部类可以使用外部类的所有方法和变量
2. 内部类的实例一定会绑在外部类的实例上

####继承
{: #inherit}

```java
public class <子类> extends <父类> {}
```

继承下来的方法可以被覆盖掉，但是实例变量不能被覆盖掉

##### 4种访问权限：

1. private: private类型的成员不会被继承；
2. default: 只有在同一包中的默认事物可以存取；
3. protected: 允许不在相同包的子类继承受保护的部分；
4. public: 任何程序代码都可以存取的公开事物；public类型的成员会被继承；

|  Modifier   |  Class  |  Package  |  Subclass  |  World  |
|-------------|---------|-----------|------------|---------|
|  public     |    Y    |     Y     |     Y      |    Y    |
|  protected  |    Y    |     Y     |     Y      |    N    |
|  no modifier|    Y    |     Y     |     N      |    N    |
|  private    |    Y    |     N     |     N      |    N    |

##### 方法的覆盖：

1. 参数必须一样，且返回类型必须兼容；
2. 不能降低方法的存取权限；

##### 方法的重载：

重载的意义是方法的名称相同，但参数不同，重载与多态毫无关系。

1. 返回类型可以不同，但不可以只改变返回类型；
2. 可以更改存取权限；

```java
public class Animal {
 public void beFriendly() {
   System.out.println("father befriendly");
 }
 public int age() {
   return 5;
 }
}

public class Cat extends Animal {
  int weight = 15;
  public void beFriendly() {
    super.beFriendly(); //可以用super调用父类的方法
    System.out.println("son befriendly");
  }

  public int age() {
    return 6;
  }

  public static void main(String[] args) {
    Cat c = new Cat();
    c.beFriendly();
    System.out.println(c.age());
    System.out.println(c.weight);
  }
}
```

#### 抽象
{: #abstract}

抽象类：抽象类代表没有人能创建出该类的实例，抽象类除了被继承过之外，没有用途，没有值，没有目的。

```java
abstract class Animal {}
```

抽象方法：抽象方法代表此方法一定被覆盖过。

* 抽象方法没有实体

```java
public abstract void eat();
```

* 如果声明一个抽象的方法，必须将类也标记为抽象的，不能在非抽象类中拥有抽象方法，但抽象类中可以有非抽象方法。

#### 接口
{: #interface}

接口`interface`可以用来解决多重继承问题,接口的方法一定是抽象的。

接口的定义：

```java
public interface Pet {....}
```

接口的实现：

```java
public class Dog extends Canine implements Pet {....}
//类可以实现多个接口
public class Dog extends Canine implements Pet, Saveable, Paintable {....}
```

Pet.java:

```java
public interface Pet {
  //接口的方法必须是抽象的，所以它们没有内容，必须以分号结尾
  abstract void beFriendly();
  abstract void play();
}
```

Dog.java:

```java
public class Dog implements Pet {
 //Dog类必须实现Pet的方法
 public void beFriendly() {
   System.out.println("befriendly");
 }
 public void play() {
   System.out.println("play");
 }

 public static void main(String[] args){
   Dog d = new Dog();
   d.beFriendly();
   d.play();
 }
}
```

#### 构造器
{: #constructor}

##### 构造函数

构造函数带有你在初始化对象时会执行的程序代码，也就是新建一个对象时会被执行，如果你没有写构造函数，编译器会帮你写。
但是需要注意，编译器只会在你完全没有设定构造函数时才会帮你写构造函数。如果你已经写了一个有参数的构造函数，并且你还
需要一个没有参数的构造函数，你必须自己手动写。

```java
public class Duck{
  //构造函数没有返回类型，而且要与类的名称相同
  public Duck(){
    System.out.println("duck initlalize");
  }

  public Duck(int i){
    System.out.println("initlalize:" + i);
  }

  public static void main(String[] args){
    Duck d = new Duck();
  }
}
```

> 在 Ruby 中构造函数名为 intialize，但是 Ruby 和 Java 不同的是，Ruby 并不支持函数重载，它通过设置默认参数来实现带有
不同个数参数的构造函数

```ruby
class Duck
   def initialize(name = nil)
     puts "init object"
   end
end

d = Duck.new
```

在创建新对象时，所有继承下来的构造函数都会执行，先执行父类的，再执行子类的

```java
public class Duck1 extends Duck{
    public Duck1(){
       System.out.println("duck1 initialize");
    }
    public static void main(String[] args){
      Duck1 d = new Duck1();
   }
}
➜

=> duck initlalize
   duck1 initlalize
```

> Ruby 在创建新的对象时，并不会自动执行来自父类的构造函数

```ruby
class Duck
   def initialize
     puts "duck init"
   end
end

class Duck1 < Duck

  def initialize
     #可以使用super调用父类的构造函数
     #super
     puts "duck1 init"
  end

end

d1 = Duck1.new => "duck1 init"
```

super() VS this()

1. super() 用于调用父类的构造函数
2. this() 用于从某个构造函数调用同一个类的另外一个构造函数，this() 只能用在构造函数中，且必须是第一行语句
3. 每个构造函数可以选择调用 super() 或 this()，但不能同时调用

```java
public class Duck1 extends Duck{
    public Duck1(){
       System.out.println("duck1 initialize");
    }
    public Duck1(int i){
       this(); //调用上面的构造函数Duck1()
       System.out.println("duck1 initialize" + i);
    }
    public static void main(String[] args){
      Duck1 d = new Duck1(100);
   }
}

=> duck initlalize
   duck1 initialize
   duck1 initialize100
```

#### 静态
{: #static}

##### 静态方法

用static关键字标记方法是静态方法

1. 静态的方法不能调用非静态的变量
2. 静态方法也不能调用非静态的方法

```ruby
class Test{
   public static void test(){
     System.out.println("static method");
   }

   public static void main(String[] args){
      Test.test();
      Test t = new Test();
      t.test(); // Java 类的实例可以调用静态方法， 而 Ruby 不可以
   }
}
```

> Java 中的静态方法类似于 Ruby 的类方法，但不同的是 Java 的静态方法可以被类的实例所调用，
而 Ruby 类的实例无法调用类方法

```ruby
class Test
    def self.test
       puts "class method"
    end
end

Test.test
```

##### 静态变量

1. 静态变量被同类的所有实例所共享
2. 静态变量会在该类的任何静态方法执行之前就初始化

```java
public class Test{
   public static int count = 0;
   public Test(){
    count++;
    System.out.println("count: "+ count);
   }

   public static void main(String[] args){
     Test t = new Test();
     Test t1 = new Test();
   }
}

count: 1
count: 2
```

> Java 的静态变量类似于 Ruby 的类变量

```ruby
class Test
    @@count = 0
    def initialize
      @@count += 1
      puts "@@count: #{@@count}"
    end
end

t = Test.new
t1 = Test.new

@@count: 1
@@count: 2
```

##### 静态final变量

一个被标记为 final 的变量到表它一旦被初始化之后就不会改动，也就是说类加载之后静态 fianl 变量就一直会维持原值

1. 常量的名称应该是大写字母
2. Java 中的常量是把变量同时标记为 static 和 final 的

```java
public class Test{
   //常量的名称应该是大写字母
   public static final int SIZE = 100;

   public static void main(String[] args){
     System.out.println(Test.SIZE);
   }
}
```

####### final的其他用途

1. final 的 method 不能被覆盖；
2. final 的类不能被继承；


#### 异常处理
{: #exception}

异常是一种 Exception 类型的对象
编译器不会注意 RuntimeException 类型的异常
方法可以用 throws 关键字抛出异常对象

```java
try {
	//危险动作
} catch(Exception ex) {
	//尝试恢复
} finally {
    //不管有没有异常都得执行的程序
}
```

```java
public class Test{
   public void divide() throws Exception {
     for(int i=10;i>=0;i--){
        System.out.println(100/i);
     }
   }
   public static void main(String[] args){
     Test t = new Test();
     try{
       t.divide();
       System.out.println("test1"); //如果divide()不出异常，该句不会被执行
     } catch(Exception ex) {
       System.out.println("exception found");
     } finally {
       System.out.println("finally");
     }
   }
}
```

> java的异常处理机制与ruby的非常相似

```ruby
begin
  ...
rescue(exception ex)
  ...
ensure
  ...
end
```

######处理多种异常

```java
public void play() throws Exception1, Exception2 {....}

try {
    ...play()
} catch(Exception1 ex1) {
	...
} catch(Exception2 ex2) {
	...
}
```

#### 线程
{: #thread}

##### Java 有两种方法来创建线程：

* 从 Thread 类继承一个新的线程类，重载它的 run() 方法

```java
class Thread1 extends Thread{
    public void run(){
        play();
    }

    public void play(){
       System.out.println(Thread.currentThread().getName());
    }

   public static void main(String[] args){
       Thread1 t = new Thread1();
       t.setName("liu");
       t.start();
       System.out.println("main");
   }
}
```

* 实现 Runnable 接口，覆盖它的 run() 方法

实现 Runnable 接口来建立给 thread 运行的任务

Runnable 接口只有一个方法：public void run()，且没有参数

```java
public class MyRunnable implements Runnable {
    public void run() {
      doMore();
    }

    public void doMore(){
      System.out.println("MyRunnable");
    }
}
```

```java
public class ThreadTester {
    public static void main(String[] args){
        Runnable threadJob = new MyRunnable();
        //将Runnable的实例传给Thread的构造函数
        Thread myThread = new Thread(threadJob);

        //调用start()才会让线程开始执行
        myThread.start();

        System.out.println("back main");
    }
}
```

##### 线程的状态

<figure>
    <img src="/images/java01.png">
    <figcaption>Java 线程状态图</figcaption>
</figure>

1. 新建状态（New）：新创建了一个线程对象
2. 就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的 start() 方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3. 运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4. 阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
  * 等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
  * 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
  * 其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。
5. 死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期

##### 线程的优先级

可以调用 Thread 类的方法 getPriority() 和 setPriority() 来存取线程的优先级, 线程的优先级是1-10之间的正整数

  * 最低优先级： 1
  * 最高优先级：10
  * 默认优先级： 5

##### synchronized

synchronized 关键字来修饰方法使它每次只能被单一的线程存取。

`private synchronized void play(){...}`

```java
public class Test implements Runnable{
   public void run() {
     for(int i=5;i>0;i--){
        System.out.println(Thread.currentThread().getName() + ": " + 10/i);
     }
   }
   public static void main(String[] args){
     Runnable job = new Test();
     Thread one = new Thread(job);
     Thread two = new Thread(job);
     one.setName("one");
     two.setName("two");
     one.start();
     two.start();
     }
 }

 =>

one: 2
one: 2
one: 3
two: 2
two: 2
two: 3
two: 5
two: 10
one: 5
one: 10
```

加上 synchronized 关键字后：

```java
public class Test implements Runnable{
   public synchronized void run() {
     for(int i=5;i>0;i--){
        System.out.println(Thread.currentThread().getName() + ": " + 10/i);
     }
   }
   public static void main(String[] args){
     Runnable job = new Test();
     Thread one = new Thread(job);
     Thread two = new Thread(job);
     one.setName("one");
     two.setName("two");
     one.start();
     two.start();
     }
 }

=>

one: 2
one: 2
one: 3
one: 5
one: 10
two: 2
two: 2
two: 3
two: 5
two: 10
 ```
