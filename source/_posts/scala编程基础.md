---
title: scala编程基础
date: 2017-04-28 21:25:21
tags: [scala]
categories: [programme]
author: kaishun
id: 51
permalink: scala-basic
---

###  多行字符串的表示方法
多行字符串用三个双引号来表示分隔符，格式为：""" ... """。
实例如下：
```scala
val foo = """菜鸟教程
www.runoob.com
www.w3cschool.cc
www.runnoob.com
以上三个地址都能访问"""
```
<!-- more -->
## 变量
### 变量声明
```
var VariableName : DataType [=  Initial Value]
或
val VariableName : DataType [=  Initial Value]

变量声明不一定需要初始值，以下也是正确的：
var myVar :Int;
val myVal :String;

```  
例如
```
var myVar : String = "Foo"
var myVar : String = "Too"

var myVar :Int;
val myVal :String;
```

### 变量类型引用
在 Scala 中声明变量和常量不一定要指明数据类型，在没有指明数据类型的情况下，其数据类型是通过变量或常量的初始值推断出来的。
所以，如果在没有指明数据类型的情况下声明变量或常量必须要给出其初始值，否则将会报错。  
```
var myVar = 10;
val myVal = "Hello, Scala!";
``` 

## 访问修饰符
Scala 访问修饰符基本和Java的一样，分别有：private，protected，public。
如果没有指定访问修饰符符，默认情况下，Scala对象的访问级别都是 public。
**Scala 中的 private 限定符，比 Java 更严格，在嵌套类情况下，外层类甚至不能访问被嵌套类的私有成员。**
```
class Outer{
    class Inner{
    private def f(){println("f")}
    class InnerMost{
        f() // 正确
        }
    }
    (new Inner).f() //错误
}
``` 
### 作用域保护
Scala中，访问修饰符可以通过使用限定词强调。格式为:
```
private[x] 
或 
protected[x]
```
这里的x指代某个所属的包、类或单例对象。如果写成private[x],读作"这个成员除了对[…]中的类或[…]中的包中的类及它们的伴生对像可见外，对其它所有类都是private。
这种技巧在横跨了若干包的大型项目中非常有用，它允许你定义一些在你项目的若干子包中可见但对于项目外部的客户却始终不可见的东西。
```
package bobsrocckets{
    package navigation{
        private[bobsrockets] class Navigator{
         protected[navigation] def useStarChart(){}
         class LegOfJourney{
             private[Navigator] val distance = 100
             }
            private[this] var speed = 200
            }
        }
        package launch{
        import navigation._
        object Vehicle{
        private[launch] val guide = new Navigator
        }
    }
}
``` 
上述例子中，类Navigator被标记为private[bobsrockets]就是说这个类对包含在bobsrockets包里的所有的类和对象可见。
比如说，从Vehicle对象里对Navigator的访问是被允许的，因为对象Vehicle包含在包launch中，而launch包在bobsrockets中，相反，所有在包bobsrockets之外的代码都不能访问类Navigator。

## 运算符
运算符和java的基本类似
### 位运算符
```
A = 0011 1100

B = 0000 1101

-------位运算----------

A&B = 0000 1100

A|B = 0011 1101

A^B = 0011 0001

~A  = 1100 0011
```

```
object Test {
   def main(args: Array[String]) {
      var a = 60;           /* 60 = 0011 1100 */  
      var b = 13;           /* 13 = 0000 1101 */
      var c = 0;

      c = a & b;            /* 12 = 0000 1100 */ 
      println("a & b = " + c );

      c = a | b;            /* 61 = 0011 1101 */
      println("a | b = " + c );

      c = a ^ b;            /* 49 = 0011 0001 */
      println("a ^ b = " + c );

      c = ~a;               /* -61 = 1100 0011 */
      println("~a = " + c );

      c = a << 2;           /* 240 = 1111 0000 */
      println("a << 2 = " + c );

      c = a >> 2;           /* 215 = 1111 */
      println("a >> 2  = " + c );

      c = a >>> 2;          /* 215 = 0000 1111 */
      println("a >>> 2 = " + c );
   }
} 
```
执行以上代码结果为
```
$ scalac Test.scala 
$ scala Test
a & b = 12
a | b = 61
a ^ b = 49
~a = -61
a << 2 = 240
a >> 2  = 15
a >>> 2 = 15

```


## 循环
### for循环

Scala 语言中 for 循环的语法：
```scala
for( var x <- Range ){
   statement(s);
}
```
以上语法中，Range 可以是一个数字区间表示 i to j ，或者 i until j。左箭头 <\- 用于为变量 x 赋值。 

**实例**

```scala
object Test {
   def main(args: Array[String]) {
      var a = 0;
      // for 循环
      for( a <- 1 to 10){
         println( "Value of a: " + a );
      }
   }
}
```
输出
```scala
value of a: 1
value of a: 2
value of a: 3
value of a: 4
value of a: 5
value of a: 6
value of a: 7
value of a: 8
value of a: 9
value of a: 10
```
以下是使用了 i until j 
```scala
object Test {
   def main(args: Array[String]) {
      var a = 0;
      // for 循环
      for( a <- 1 until 10){
         println( "Value of a: " + a );
      }
   }
}
```
输出
```
value of a: 1
value of a: 2
value of a: 3
value of a: 4
value of a: 5
value of a: 6
value of a: 7
value of a: 8
value of a: 9
```
在 for 循环 中你可以使用分号 (;) 来设置多个区间，它将迭代给定区间所有的可能值。以下实例演示了两个区间的循环实例：
```
object Test {
   def main(args: Array[String]) {
      var a = 0;
      var b = 0;
      // for 循环
      for( a <- 1 to 3; b <- 1 to 3){
         println( "Value of a: " + a );
         println( "Value of b: " + b );
      }
   }
}
```

#### for 循环集合
```
object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6);

      // for 循环
      for( a <- numList ){
         println( "Value of a: " + a );
      }
   }
}
------------------------输出
value of a: 1
value of a: 2
value of a: 3
value of a: 4
value of a: 5
value of a: 6

```
#### for 循环过滤
Scala 可以使用一个或多个 if 语句来过滤一些元素。
以下是在 for 循环中使用过滤器的语法。 你可以使用分号(;)来为表达式添加一个或多个的过滤条件。
```
object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6,7,8,9,10);

      // for 循环
      for( a <- numList
           if a != 3; if a < 8 ){
         println( "Value of a: " + a );
      }
   }
}
------------------输出-----------
value of a: 1
value of a: 2
value of a: 4
value of a: 5
value of a: 6
value of a: 7
```
#### for 使用 yield
你可以将 for 循环的返回值作为一个变量存储。语法格式如下：
```
var retVal = for{ var x <- List
     if condition1; if condition2...
}yield x
```
实例
以下实例演示了 for 循环中使用 yield：
```
object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6,7,8,9,10);

      // for 循环
      var retVal = for{ a <- numList 
                        if a != 3; if a < 8
                      }yield a

      // 输出返回值
      for( a <- retVal){
         println( "Value of a: " + a );
      }
   }
}
--------------------输出--------------
value of a: 1
value of a: 2
value of a: 4
value of a: 5
value of a: 6
value of a: 7

```











## Scala 函数
Scala 有函数和方法，二者在语义上的区别很小。Scala 方法是类的一部分，而函数是一个对象可以赋值给一个变量。换句话来说在类中定义的函数即是方法。
我们可以在任何地方定义函数，甚至可以在函数内定义函数（内嵌函数）。更重要的一点是 Scala 函数名可以由以下特殊字符：+, ++, ~, &,-, -- , \, /, : 等。

### 函数声明
```
def functionName ([参数列表]) : [return type]
``` 

### 函数定义
方法定义由一个def 关键字开始，紧接着是可选的参数列表，一个冒号"：" 和方法的返回类型，一个等于号"="，最后是方法的主体。
Scala 函数定义格式如下：
```
def functionName ([参数列表]) : [return type] = {
   function body
   return [expr]
}
``` 
以上代码中 return type 可以是任意合法的 Scala 数据类型。参数列表中的参数可以使用逗号分隔。
以下函数的功能是将两个传入的参数相加并求和：
```
object add{
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```  
**如果函数没有返回值，可以返回为 Unit，这个类似于 Java 的 void**, 实例如下：
```
object Hello{
   def printMe( ) : Unit = {
      println("Hello, Scala!")
   }
}
``` 
### 函数调用
```
functionName( 参数列表 )
```
如果函数使用了实例的对象来调用，我们可以使用类似java的格式 (使用 . 号)：
```
[instance.]functionName( 参数列表 )
```
以下实例演示了定义与调用函数的实例:
```
object Test {
   def main(args: Array[String]) {
        println( "Returned Value : " + addInt(5,7) );
   }
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
``` 
**++Scala也是一种函数式语言++，所以函数是 Scala 语言的核心。以下一些函数概念有助于我们更好的理解 Scala 编程：**

### 函数传名调用
传值调用（call-by-value）：先计算参数表达式的值，再应用到函数内部；  
传名调用（call-by-name）：将未计算的参数表达式直接应用到函数内部
看例子 
```scala
package com.doggie  
  
object Add {  
  def addByName(a: Int, b: => Int) = a + b   
  def addByValue(a: Int, b: Int) = a + b   
}  
------------------------
addByName(2, 2 + 2)  
->2 + (2 + 2)  
->2 + 4  
->6  
  
addByValue(2, 2 + 2)  
->addByValue(2, 4)  
->2 + 4  
->6  
```
**addByName是传名调用，addByValue是传值调用。语法上可以看出，使用传名调用时，在参数名称和参数类型中间有一个=>符号。**  
例子： 酒鬼喝酒
```scala
package first.example

/**
  * Created by Administrator on 2017/3/28.
  */
object CallByName {
  //最开始拥有的软妹币
  var money = 10
  //每天喝掉一个软妹币
  def drink(): Unit = {
    money -= 1
  }
  //数钱时要算上被喝掉的软妹币
  def count(): Int = {
    drink()
    return money
  }
  //每天都数钱
  def printByName(x: => Int): Unit = {
    for(i <- 0 until 5)
      println("每天算一算，酒鬼还剩" + x + "块钱！")
  }
  //第一天数一下记墙上，以后每天看墙上的余额
  def printByValue(x: Int): Unit = {
    for(i <- 0 until 5)
      println("只算第一天，酒鬼还剩" + x + "块钱！")
  }

  def main(args: Array[String]) = {
    printByName(count())
    printByValue(count())
  }
}

------------------输出-----------------
每天算一算，酒鬼还剩9块钱！
每天算一算，酒鬼还剩8块钱！
每天算一算，酒鬼还剩7块钱！
每天算一算，酒鬼还剩6块钱！
每天算一算，酒鬼还剩5块钱！
只算第一天，酒鬼还剩4块钱！
只算第一天，酒鬼还剩4块钱！
只算第一天，酒鬼还剩4块钱！
只算第一天，酒鬼还剩4块钱！
只算第一天，酒鬼还剩4块钱！
```
### Scala 指定函数参数名
一般情况下函数调用参数，就按照函数定义时的参数顺序一个个传递。但是我们也可以通过指定函数参数名，并且不需要按照顺序向函数传递参数，实例如下
```scala
object Test {
   def main(args: Array[String]) {
        printInt(b=5, a=7);
   }
   def printInt( a:Int, b:Int ) = {
      println("Value of a : " + a );
      println("Value of b : " + b );
   }
}
-------------输出--------------------
Value of a :  7
Value of b :  5
```
### Scala 函数 - 可变参数
Scala 允许你指明函数的最后一个参数可以是重复的，即我们不需要指定函数参数的个数，可以向函数传入可变长度参数列表。
Scala 通过在参数的类型之后放一个星号来设置可变参数(可重复的参数)。例如：
```scala
object Test {
   def main(args: Array[String]) {
        printStrings("Runoob", "Scala", "Python");
   }
   def printStrings( args:String* ) = {
      var i : Int = 0;
      for( arg <- args ){
         println("Arg value[" + i + "] = " + arg );
         i = i + 1;
      }
   }
}
-------------输出-------------
Arg value[0] = Runoob
Arg value[1] = Scala
Arg value[2] = Python
```
### Scala 递归函数
例如计算阶乘
```scala
object Test {
   def main(args: Array[String]) {
      for (i <- 1 to 10)
         println(i + " 的阶乘为: = " + factorial(i) )
   }
   
   def factorial(n: BigInt): BigInt = {  
      if (n <= 1)
         1  
      else    
      n * factorial(n - 1)
   }
}
----------------输出-------------------
1 的阶乘为: = 1
2 的阶乘为: = 2
3 的阶乘为: = 6
4 的阶乘为: = 24
5 的阶乘为: = 120
6 的阶乘为: = 720
7 的阶乘为: = 5040
8 的阶乘为: = 40320
9 的阶乘为: = 362880
10 的阶乘为: = 3628800
```
### Scala 函数 - 默认参数值
Scala 可以为函数参数指定默认参数值，使用了默认参数，你在调用函数的过程中可以不需要传递参数，这时函数就会调用它的默认参数值，如果传递了参数，则传递值会取代默认值。实例如下：
```scala
object Test {
   def main(args: Array[String]) {
        println( "返回值 : " + addInt() );
   }
   def addInt( a:Int=5, b:Int=7 ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
-----------------------------输出-----------------------
$ scalac Test.scala
$ scala Test
返回值 : 12
```

### scala函数嵌套
TODO


### Scala 偏应用函数  

Scala 偏应用函数是一种表达式，你不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。
如下实例，我们打印日志信息：  

实例中，log() 方法接收两个参数：date 和 message。我们在程序执行时调用了三次，参数 date 值都相同，message 不同。
我们可以使用偏应用函数优化以上方法，绑定第一个 date 参数，第二个参数使用下划线(_)替换缺失的参数列表，并把这个新的函数值的索引的赋给变量。实例修改如下：
```
  def main(args: Array[String]) {
    val date = new Date
    val logWithDateBound = log(date, _ : String)

    logWithDateBound("message1" )
    Thread.sleep(1000)
    logWithDateBound("message2" )
    Thread.sleep(1000)
    logWithDateBound("message3" )
  }

  def log(date: Date, message: String)  = {
    println(date + "----" + message)
  } 
```
### Scala匿名函数
Scala 中定义匿名函数的语法很简单，箭头左边是参数列表，右边是函数体。
使用匿名函数后，我们的代码变得更简洁了。
下面的表达式就定义了一个接受一个Int类型输入参数的匿名函数:  ,这里可能不好理解，其实可以先暂时放下，先看我另外一篇高阶函数的文章，再过来看匿名函数
```scala
var inc = (x:Int) => x+1
```  
上述定义的匿名函数，其实是下面这种写法的简写：
```scala
def add2 = new Function1[Int,Int]{  
	def apply(x:Int):Int = x+1;  
} 
----------更多参考http://www.runoob.com/scala/anonymous-functions.html----------
```
### Scala 高阶函数
下一篇文章