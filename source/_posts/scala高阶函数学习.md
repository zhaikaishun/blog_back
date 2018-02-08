---
title: scala高阶函数学习
date: 2017-05-01 21:25:21
tags: [scala]
categories: [programme]
author: kaishun
id: 53
permalink: scala-higher-order-functions
---

参考文章   
http://www.cnblogs.com/wzm-xu/p/4063814.html  
http://www.cnblogs.com/wzm-xu/p/4064389.html  

# scala高阶函数上
高阶函数是函数式编程里面一个非常重要的特色，所谓的高阶函数，就是以其它函数作为参数的函数。

下面以一个小例子演示Scala的高阶函数特性，非常有意思，也非常强大。

首先看这么一个程序：
<!-- more -->
**code1**：

```scala
object higherorderfuntion{
   def sum1(a:Int,b:Int):Int=
     if(a>b) 0
     else a+sum1(a+1,b)

   def sum2(a:Int,b:Int):Int=
     if(a>b) 0
     else cube(a)+sum2(a+1,b)

   def sum3(a:Int,b:Int):Int=
     if(a>b) 0
     else fac(a)+sum3(a+1,b)

   def cube(a:Int):Int=a*a*a

   def fac(a:Int):Int=
   if (a==0) 1
   else a*fac(a-1)

   def main(args:Array[String])={
     println(sum1(1,3))
     println(sum2(1,3))
     println(sum3(1,3))
    }
}
```
上面这个例子“没有”用到高阶函数，sum1是计算a+(a+1)+(a+2)+...+(b),  sum2是计算a\^3+(a+1)\^3+(a+2)\^3+...+b\^3,

sum3是计算a!+(a+1)!+(a+2)!+...+b!。分析sum1,sum2,sum3的代码，很容易发现这三个函数有着相似的“Pattern”，

抛开函数名不论，这三个函数唯一的区别在于，在 else语句中对a的处理：a, cube(a) , fac(a).  那么来看，在函数式

编程里面，是如何非常精彩的利用这个“Pattern”来使得代码更加精简的：

**code2**：
```scala
object higherorderfuntion{
   def sum(f:Int=>Int,a:Int,b:Int):Int=
     if(a>b) 0
     else f(a)+sum(f,a+1,b)

   def sumInt(a:Int,b:Int):Int = sum(id,a,b)
   def sumCube(a:Int,b:Int):Int = sum(cube,a,b)
   def sumFac(a:Int,b:Int):Int = sum(fac,a,b)

   def id(a:Int) = a
   def cube(a:Int):Int=a*a*a
   def fac(a:Int):Int=
     if (a==0) 1
     else a*fac(a-1)

   def main(args:Array[String])={
     println(sumInt(1,3))
     println(sumCube(1,3))
     println(sumFac(1,3))
   }

}
```  
重头戏来了，我们来看sum函数的实现：
```scala
def sum(f:Int=>Int,a:Int,b:Int):Int=
     if(a>b) 0
     else f(a)+sum(f,a+1,b)
```  
sum函数接收三个参数，第二个和第三个参数分别是： a:Int 和 b:Int, 这和第一个例子中是一样的。

比较令人费解的是sum函数的第一个参数：f:Int=>Int  

这个是什么意思呢？意思是sum函数接收一个名字叫做 “f” 的函数作为参数，而 Int=>Int 是对f的说明：

=>左边的Int是说：函数f接收一个Int类型的参数，

=>右边的Int是说：函数f的返回值是Int类型的。

好了，那么既然sum函数接收函数f作为一个参数，那么sum就可以利用f了，事实也是这样的，看sum

函数的else语句就知道了：**f(a)**

然后再看sumInt、sumCube和sumFac三个函数的定义：
```scala
def sumInt(a:Int,b:Int):Int = sum(id,a,b)
def sumCube(a:Int,b:Int):Int = sum(cube,a,b)
def sumFac(a:Int,b:Int):Int = sum(fac,a,b)
```
以sumCube函数为例吧，它在定义的时候，把cube函数作为参数，传递给sum函数，

而我们看cube函数的定义：
```scala
def cube(a:Int):Int=a*a*a
```
发现，cube函数接收一个Int作为参数，并且返回一个Int，也就是说cube函数是符合sum函数的第一个

参数:   f:Int=>Int  的

是不是很精彩呢？

事实上上述代码还可以进一步简化，因为我们观察到id函数和cube函数的功能非常简单，不需要单独作为

一个函数出现，进一步简化后的代码如下：    
**code3**
```scala
object higherorderfuntion{
   def sum(f:Int=>Int,a:Int,b:Int):Int=
     if(a>b) 0
     else f(a)+sum(f,a+1,b)

   def sumInt(a:Int,b:Int):Int = sum(x=>x,a,b)
   def sumCube(a:Int,b:Int):Int = sum(x=>x*x*x,a,b)
   def sumFac(a:Int,b:Int):Int = sum(fac,a,b)

   def fac(a:Int):Int=
     if (a==0) 1
     else a*fac(a-1)

   def main(args:Array[String])={
     println(sumInt(1,3))
     println(sumCube(1,3))
     println(sumFac(1,3))
   }
}
```

code3和code2相比，去掉了id函数和cube这两个函数，并且sumInt和sumCube函数的声明也发生了

一点变化：
```scala
def sumInt(a:Int,b:Int):Int = sum(x=>x,a,b)
def sumCube(a:Int,b:Int):Int = sum(x=>x*x*x,a,b)
```
像： x=>x   和     x=>x*x*x   这样的东西是什么呢？在函数式编程里面，这被叫做“function literal”,又称“匿名函数”，

说白了，这也是函数的一种表达方式，只是这个函数没有名字罢了。

code3其实也还有点小毛病，那就是sum函数和fac函数都不是“尾递归”，所以呢，把它们改成尾递归如下：

**code4：**
```
object higherorderfuntion{
   def sum(f:Int=>Int,a:Int,b:Int):Int={
     def loop(a:Int,acc:Int):Int=
       if(a>b) acc
       else loop(a+1,f(a)+acc)
     loop(a,0)
   }
   def sumInt(a:Int,b:Int):Int = sum(x=>x,a,b)
   def sumCube(a:Int,b:Int):Int = sum(x=>x*x*x,a,b)
   def sumFac(a:Int,b:Int):Int = sum(fac,a,b)

   def fac(a:Int):Int=
   {
     def loop(a:Int,acc:Int):Int=
       if(a==0) acc
       else loop(a-1,a*acc)
     loop(a,1)
   }
   def main(args:Array[String])={
     println(sumInt(1,4))
     println(sumCube(1,4))
     println(sumFac(1,4))
   }

}
```  

# scala高阶函数下
在scala高阶函数上我们演示了如何把一个函数作为参数传递给另外一个函数。

在本文里面，我们来演示函数式编程另外一个重要的特性：返回一个函数。首先来看这么一段代码：

**code piece 1：**
```scala
def sum(f:Int=>Int):(Int,Int)=>Int={
       def sumF(a:Int,b:Int):Int=
         if(a>b) 0
         else f(a)+sumF(a+1,b)
       sumF
   }
```
一点点来看，f:Int=>Int 是sum函数接收的参数，该参数是一个函数。 

":" 号后面的 (Int,Int) => Int 是sum函数的返回值，又(Int,Int) => Int是一个函数的类型：接收两个Int型的数，

返回一个Int的数。也就是说调用sum函数，其返回的值是一个函数。  这一点对于已经习惯C、C++、Java等编程语言的

程序员来说有一点难以理解。

继续看例子吧，如果执行下面的一行代码会发生什么呢？
```scala
sum(x=>x*x)(1,4)
```
结果会返回30。为什么是30呢？ 30= 1^2+ 2^2 + 3^2 + 4^2.

首先，sum(x=>x*x) 是一个函数，并且sum(x=>x*x)的类型是(Int,Int)=>Int，正因为sum(x=>x*x)是一个函数，

所以它才可以继续接收参数(1,4).

好吧，我可能没有把这个事儿说清楚，实在是太抽象了。但是，读者应该有了一个基本的印象，那就是：

在函数式编程里面，函数f可以接收函数g作为参数，也可以返回函数h。

到这里还没完。。。。。。

Scala里面可以继续对code piece1 进行简化，如下：

**code piece2**：
```scala
object higherorderfuntion{
   def sum(f:Int=>Int)(a:Int,b:Int):Int={
      if(a>b) 0  else f(a)+sum(f)(a+1,b)
   }

   def fac(a:Int):Int=
   {
     def loop(a:Int,acc:Int):Int=
       if(a==0) acc
       else loop(a-1,a*acc)
     loop(a,1)
   }
   def main(args:Array[String])={
     println(sum(x=>x)(1,4))
     println(sum(x=>x*x*x)(1,4))
     println(sum(fac)(1,4))
   }

}
```  
这里我们需要特别注意，这里sum函数的参数列表变成了两个：(f:Int=>Int)和(a:Int,b:Int)

在scala里面，函数可以有多个参数列表。比如下面的函数定义：  
```
def f(args_1)(args_2)...(args_n)=E
```
E代表一个函数体，且n>1

那么上述定义等价于：  
```scala
def f(args_1)(args_2)...(args_n-1)={def g(args_n)=E;g}
```
还是以code piece2中的sum函数为例：
```
def sum(f:Int=>Int)(a:Int,b:Int):Int={
      if(a>b) 0  else f(a)+sum(f)(a+1,b)
   }
```
等价于：
```
def sum(f:Int=>Int):(Int,Int)=>Int={
     def g(a:Int,b:Int):Int = 
      if(a>b) 0  else f(a)+sum(f)(a+1,b)
     g
   }
```
这个定义和code piece1中的sum函数的定义是一致的。
---

---

---

# 扩展： 什么是尾递归 
参考https://www.zhihu.com/question/20761771/answer/19996299  
这个虽然是python的，但是也能让我们理解什么是尾递归
尾递归和一般的递归不同在对内存的占用，普通递归创建stack累积而后计算收缩，尾递归只会占用恒量的内存（和迭代一样）。SICP中描述了一个内存占用曲线，用以上答案中的Python代码为例（普通递归）：
```python
def recsum(x):
  if x == 1:
    return x
  else:
    return x + recsum(x - 1)
```
当调用recsum(5)，Python调试器中发生如下状况：
```python
recsum(5)
5 + recsum(4)
5 + (4 + recsum(3))
5 + (4 + (3 + recsum(2)))
5 + (4 + (3 + (2 + recsum(1))))
5 + (4 + (3 + (2 + 1)))
5 + (4 + (3 + 3))
5 + (4 + 6)
5 + 10
15

```
这个曲线就代表内存占用大小的峰值，从左到右，达到顶峰，再从右到左收缩。而我们通常不希望这样的事情发生，所以使用迭代，只占据常量stack space(更新这个栈！而非扩展他)。
---------------------
**（一个替代方案：迭代）**  
```python
for i in range(6):
  sum += i
```
因为Python，Java，Pascal等等无法在语言中实现尾递归优化(Tail Call Optimization, TCO)，所以采用了for, while, goto等特殊结构代替recursive的表述。Scheme则不需要这样曲折地表达，一旦写成尾递归形式，就可以进行尾递归优化。---------------------Python中可以写（尾递归）：

```python
def tailrecsum(x, running_total=0):
  if x == 0:
    return running_total
  else:
    return tailrecsum(x - 1, running_total + x)

```
理论上类似上面：  
```
tailrecsum(5, 0)
tailrecsum(4, 5)
tailrecsum(3, 9)
tailrecsum(2, 12)
tailrecsum(1, 14)
tailrecsum(0, 15)
15
```

观察到，tailrecsum(x, y)中形式变量y的实际变量值是不断更新的，对比普通递归就很清楚，后者每个recsum()调用中y值不变，仅在层级上加深。所以，尾递归是把变化的参数传递给递归函数的变量了。怎么写尾递归？形式上只要最后一个return语句是单纯函数就可以。如：
```
return tailrec(x+1);
```  
而
```
return tailrec(x+1) + x;
```
则不可以，因为无法更新tailrec()函数内的实际变量，只是新建一个栈。但Python不能尾递归优化（Java不行，C可以，我不知道为什么），这里是用它做个例子。====================================如何优化尾递归：在编译器处理过程中生成中间代码（通常是三地址代码），用编译器优化。


另外一个的回答
尾递归就是操作的最后一步是调用自身的递归。

这是尾递归：
```
function f(x) {
   if (x === 1) return 1;
   return f(x-1);
}
```  
（这个程序没什么意义，仅作为理解辅助之用）。

这不是尾递归：
```
function f(x) {
   if (x === 1) return 1;
   return 1 + f(x-1);
}
```  
后者不是尾递归，是因为该函数的最后一步操作是用1加上f(x-1)的返回结果，因此，最后一步操作不是调用自身。注意，后面这段代码的递归也发生在函数末尾，但它不是尾递归。尾递归的判断标准是函数运行最后一步是否调用自身，而不是是否在函数的最后一行调用自身。因此阶乘n!的递归求法，尽管看起来递归发生在函数末尾，其实也不是尾递归：
```python
function factorial(n) {
   if (n === 1) return 1;
   return n * factorial(n-1); // 最后一步不是调用自身，因此不是尾递归
}
```
使用尾递归可以带来一个好处：因为进入最后一步后不再需要参考外层函数（caller）的信息，因此没必要保存外层函数的stack，递归需要用的stack只有目前这层函数的，因此避免了栈溢出风险。一些语言提供了尾递归优化，当识别出使用了尾递归时，会相应地只保留当前层函数的stack，节省内存。所以，在没有尾递归优化的语言，如java, python中，鼓励用迭代iteration来改写尾递归；在有尾递归优化的语言如Erlang中，鼓励用尾递归来改写其他形式的递归。谢谢 @jamesr 的指正~关于如何改写，参考[：尾调用优化 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)  
知乎尾递归参考:  https://www.zhihu.com/question/20761771

