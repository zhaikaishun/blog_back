---
title: scala集合
date: 2017-04-29 21:25:21
tags: [scala]
categories: [programme]
author: kaishun
id: 52
permalink: scala-collect
blogexcerpt: 本文参考至scala编程，菜鸟教程，然后将自己的判断以及重要方法的提取， 解释， 合并。在 Scala 中， 字符串的类型实际上是 Java String，它本身没有 String 类。在 Scala 中，String 是一个不可变的对象，所以该对象不可被修改。这就意味着你如果修改字符串就会产生一个新的字符串对象
---

本文参考至scala编程，菜鸟教程，然后将自己的判断以及重要方法的提取，解释，合并
# **字符串**
在 Scala 中，字符串的类型实际上是 Java String，它本身没有 String 类。在 Scala 中，String 是一个不可变的对象，所以该对象不可被修改。这就意味着你如果修改字符串就会产生一个新的字符串对象。  

 String 对象是不可变的，如果你需要创建一个可以修改的字符串，可以使用 String Builder 类，如下实例:  
```scala
 object Test {
   def main(args: Array[String]) {
      val buf = new StringBuilder;
      buf += 'a'
      buf ++= "bcdef"
      println( "buf is : " + buf.toString );
   }
}
```
 
同java一样，scala的字符串用过length()方法得到长度，String 类中使用string1.concat(string2); 方法来连接两个字符串
也可以直接用+号连接字符串
java.lang.String的所有方法，在scala中也可以使用, 这里不仔细介绍

# **数组**
## **声明数组**
```
var z:Array[String] = new Array[String](3)

或

var z = new Array[String](3)
```
## **赋值**
```scala
z(0) = "Runoob"; z(1) = "Baidu"; z(4/2) = "Google"
```
也可以这样定义一个数组
```scala
var z = Array("Runoob", "Baidu", "Google")
```
## **处理数组**
```scala
object Test {
   def main(args: Array[String]) {
      var myList = Array(1.9, 2.9, 3.4, 3.5)
      
      // 输出所有数组元素
      for ( x <- myList ) {
         println( x )
      }

      // 计算数组所有元素的总和
      var total = 0.0;
      for ( i <- 0 to (myList.length - 1)) {
         total += myList(i);
      }
      println("总和为 " + total);

      // 查找数组中的最大元素
      var max = myList(0);
      for ( i <- 1 to (myList.length - 1) ) {
         if (myList(i) > max) max = myList(i);
      }
      println("最大值为 " + max);
    
   }
}

-------输出-------
1.9
2.9
3.4
3.5
总和为 11.7
最大值为 3.5
```
## **多维数组**
```
def main(args: Array[String]) {
      var myMatrix = ofDim[Int](3,3)
      
      // 创建矩阵
      for (i <- 0 to 2) {
         for ( j <- 0 to 2) {
            myMatrix(i)(j) = j;
         }
      }
      
      // 打印二维阵列
      for (i <- 0 to 2) {
         for ( j <- 0 to 2) {
            print(" " + myMatrix(i)(j));
         }
         println();
      }
    
   }
```
## **合并数组**
使用concat() 方法来合并两个数组
```scala
def main(args: Array[String]) {
      var myList1 = Array(1, 2, 3, 4,)
      var myList2 = Array(5, 6, 7, 8)

      var myList3 =  concat( myList1, myList2)
      
      // 输出所有数组元素
      for ( x <- myList3 ) {
         println( x )
      }
   }
   
----------输出-----------
1
2
3
4
5
6
7
8

```
## **创建区间数组**
以下实例中，我们使用了 range() 方法来生成一个区间范围内的数组。range() 方法最后一个参数为步长，默认为 1：
```scala
def main(args: Array[String]) {
      var myList1 = range(10, 20, 2)
      var myList2 = range(10,20)

      // 输出所有数组元素
      for ( x <- myList1 ) {
         print( " " + x )
      }
      println()
      for ( x <- myList2 ) {
         print( " " + x )
      }
   }
   -----------------输出---------------
10 12 14 16 18
10 11 12 13 14 15 16 17 18 19
```

# **Scala 集合List**
Scala 列表类似于数组，它们所有元素的类型都相同，但是它们也有所不同：列表是不可变的，值一旦被定义了就不能改变，其次列表 具有递归的结构（也就是链接表结构）而数组不是。。
列表的元素类型 T 可以写成 List[T]。例如，以下列出了多种类型的列表：
## **列表构造方式**
### **构造列表方式一**
```scala
// 字符串列表
val site: List[String] = List("Runoob", "Google", "Baidu")

// 整型列表
val nums: List[Int] = List(1, 2, 3, 4)

// 空列表
val empty: List[Nothing] = List()

// 二维列表
val dim: List[List[Int]] =
   List(
      List(1, 0, 0),
      List(0, 1, 0),
      List(0, 0, 1)
   )
```
### **构造列表方式二**
构造列表的两个基本单位是 Nil 和 ::(发音为cons) Nil代表空列表，中缀符号:: 表示列表从前端扩展。也就是说x::xs代表了一个元素为x，后面紧贴着xs的列表，因此以上实例我们可以写成
```scala

// 字符串列表
val site = "Runoob" :: ("Google" :: ("Baidu" :: Nil))

// 整型列表
val nums = 1 :: (2 :: (3 :: (4 :: Nil)))

// 空列表
val empty = Nil

// 二维列表
val dim = (1 :: (0 :: (0 :: Nil))) ::
          (0 :: (1 :: (0 :: Nil))) ::
          (0 :: (0 :: (1 :: Nil))) :: Nil
```
### **方式二的简化**
由于以::结尾，::遵循右结合的规则，A::B::C 等价于A::(B::C), 因此，前面定义用到的括号可以去掉
```scala
val names = 1::2::3::4::Nil
```
与前面的names定义一致

## **列表的基本操作**
- Scala列表有三个基本操作：
- head 返回列表第一个元素
- tail 返回一个列表，包含除了第一元素之外的其他元素
- isEmpty 在列表为空时返回true  
对于Scala列表的任何操作都可以使用这三个基本操作来表达。实例如下:
```scala
object Test {
   def main(args: Array[String]) {
      val site = "Runoob" :: ("Google" :: ("Baidu" :: Nil))
      val nums = Nil

      println( "第一网站是 : " + site.head )
      println( "最后一个网站是 : " + site.tail )
      println( "查看列表 site 是否为空 : " + site.isEmpty )
      println( "查看 nums 是否为空 : " + nums.isEmpty )
   }
}
-----------输出--------
第一网站是 : Runoob
最后一个网站是 : List(Google, Baidu)
查看列表 site 是否为空 : false
查看 nums 是否为空 : true
```
## **List类的一阶方法**
### **连接列表**
你可以使用 ::: 运算符或 List.:::() 方法或 List.concat() 方法来连接两个或多个列表。实例如下:
```scala
object Test {
   def main(args: Array[String]) {
      val site1 = "Runoob" :: ("Google" :: ("Baidu" :: Nil))
      val site2 = "Facebook" :: ("Taobao" :: Nil)

      // 使用 ::: 运算符
      var fruit = site1 ::: site2
      println( "site1 ::: site2 : " + fruit )
      
      // 使用 Set.:::() 方法
      fruit = site1.:::(site2)
      println( "site1.:::(site2) : " + fruit )

      // 使用 concat 方法
      fruit = List.concat(site1, site2)
      println( "List.concat(site1, site2) : " + fruit  )
      

   }
}
```
### **List.fill()**
我们可以使用 List.fill() 方法来创建一个指定重复数量的元素列表：
```scala
object Test {
   def main(args: Array[String]) {
      val site = List.fill(3)("Runoob") // 重复 Runoob 3次
      println( "site : " + site  )

      val num = List.fill(10)(2)         // 重复元素 2, 10 次
      println( "num : " + num  )
   }
}
---------输出--------
site : List(Runoob, Runoob, Runoob)
num : List(2, 2, 2, 2, 2, 2, 2, 2, 2, 2)
```
### **List.tabulate()**
List.tabulate() 方法是通过给定的函数来创建列表。
方法的第一个参数为元素的数量，可以是二维的，第二个参数为指定的函数，我们通过指定的函数计算结果并返回值插入到列表中，起始值为 0，实例如下：
```scala
object Test {
   def main(args: Array[String]) {
      // 通过给定的函数创建 5 个元素
      val squares = List.tabulate(6)(n => n * n)
      println( "一维 : " + squares  )

      // 创建二维列表
      val mul = List.tabulate( 4,5 )( _ * _ )      
      println( "多维 : " + mul  )
   }
}
----------------输出---------
一维 : List(0, 1, 4, 9, 16, 25)
多维 : List(List(0, 0, 0, 0, 0), List(0, 1, 2, 3, 4), List(0, 2, 4, 6, 8), List(0, 3, 6, 9, 12))
```

### **列表反转**
List.reverse 用于将列表的顺序反转，实例如下：
```scala
object Test {
   def main(args: Array[String]) {
      val site = "Runoob" :: ("Google" :: ("Baidu" :: Nil))
      println( "site 反转前 : " + site )

      println( "site 反转前 : " + site.reverse )
   }
}
------------输出---------------
$ vim Test.scala 
$ scala Test.scala 
site 反转前 : List(Runoob, Google, Baidu)
site 反转前 : List(Baidu, Google, Runoob)
```

### **前缀与后缀 drop take splitAt**
```scala
def main(args: Array[String]) {
    var ls = "google"::"baidu"::"tenxun"::"alibaba"::"apples"::Nil
    var takeTest =ls.take(2)
    var dropTest = ls.drop(2)
    var splitTest = ls.splitAt(3)
    println("takeTest: "+takeTest)
    println("dropTest: "+dropTest)
    println("splitTest: "+splitTest)
  }
  
--------------输出-------------
takeTest: List(google, baidu)
dropTest: List(tenxun, alibaba, apples)
splitTest: (List(google, baidu, tenxun),List(alibaba, apples))
```
### **元素选择 apply方法和indices方法**
apply方法是获取第几个值，list.apply(2)和list(2)是一样的  
indices是获取所有的列表的Range,暂时不知道有多大用处
```scala
def main(args: Array[String]) {
    var ls = "google"::"baidu"::"tenxun"::"alibaba"::"apples"::"stackoverflow"::Nil
    var applyTest = ls.apply(2)
    var indicesTest =  ls.indices
    println("applyTest: "+applyTest)
    println("applyTest2: "+ls(2))
    println("indicesTest: "+indicesTest)
    for(i<-indicesTest){
      println(ls(i))
    }
  }
-------------输出--------
applyTest: tenxun
applyTest2: tenxun
indicesTest: Range(0, 1, 2, 3, 4, 5)
google
baidu
tenxun
alibaba
apples
stackoverflow


```
### **toString和mkString**
toString和java的一样
mkString  mkString(start: String,sep: String,end: String): String  
第一个参数是以什么开始，第二个参数是以什么分割，第三个参数是以什么结尾，函数返回一个String  
```scala
def main(args: Array[String]) {
    var ls = "google" :: "baidu" :: "tenxun" :: "alibaba" :: "apples" :: "stackoverflow" :: Nil
    println("toString: "+ls.toString())
    println("mkString: "+ls.mkString("{",";","}"))
  }
---------输出----------
toString: List(google, baidu, tenxun, alibaba, apples, stackoverflow)
mkString: {google;baidu;tenxun;alibaba;apples;stackoverflow{
```
### **转换列表 toArray element iterator**
想要在数组Array和列表list之间转换，可以使用List的toArray和Array的toList  
例子
```
scala> var ls = "google" :: "baidu" :: "tenxun" :: "alibaba" :: "apples" :: "stackoverflow" :: Nil
ls: List[String] = List(google, baidu, tenxun, alibaba, apples, stackoverflow)

scala> var arrays = ls.toArray
arrays: Array[String] = Array(google, baidu, tenxun, alibaba, apples, stackoverflow)

scala> var lists = arrays.toList
lists: List[String] = List(google, baidu, tenxun, alibaba, apples, stackoverflow)
```
element在很旧的版本有，现在已经过时不用了。如果要用枚举器访问列表元素，可以使用iterator 
```scala
def main(args: Array[String]) {
    var ls = "google" :: "baidu" :: "tenxun" :: "alibaba" :: "apples" :: "stackoverflow" :: Nil
     val it =ls.iterator
      while(it.hasNext){
        print(it.next()+",")
      }
  }
--------------输出------------
google, baidu, tenxun, alibaba, apples, stackoverflow, 
```
## **list类的高阶用法**
在java中，若要提取出满足特点条件的元素，或者检查所有元素是否满足某种性质，或者用某种方式转变列表的所有元素，这样的需求一般都需要使用for或者while循环的固定表达式，在scala中，可以通过使用List的一些高阶方法（函数）来更为简介的实现  
### **列表间映射：map、flatMap和foreach**
1. xs map f 操作返回**把函数f应用在xs的每个列表元素之后**由此**组成的新列表**。如：
```
scala> List(1,2,3).map(_ +1)
res1: List[Int] = List(2, 3, 4)
```
```scala
scala> val words = List("zks","zhaikaishun","kaishun","kai","xiaozhai")
words: List[String] = List(zks, zhaikaishun, kaishun, kai, xiaozhai)

scala> words.map(_.length)
res2: List[Int] = List(3, 11, 7, 3, 8)
```
2. flatMap操作符与map类似，不过它的右操作元是能够返回元素列表的函数。它对列表的每个元素调用该方法，**然后连接所有方法**的结果并返回。map与flatMap的差异举例说明如下：
```scala
scala> words.map(_.toList)
res3: List[List[Char]] = List(List(z, k, s), List(z, h, a, i, k, a, i, s, h, u, n), List(k, a, i, s, h, u, n), List(k, a, i), List(x, i, a, o, z, h, a, i))

scala> words.flatMap(_.toList)
res4: List[Char] = List(z, k, s, z, h, a, i, k, a, i, s, h, u, n, k, a, i, s, h, u, n, k, a, i, x, i, a, o, z, h, a, i)
```
map与flatMap的差异和协作可以用下面的例子体会
```scala
scala> List.range(1, 5).flatMap(i => List.range(1, i).map(j => (i, j)))
res9: List[(Int, Int)] = List((2,1), (3,1), (3,2), (4,1), (4,2), (4,3))
```
解释 : List.range(1,5)生成了List(1,2,3,4),注意没有5  
.flatMap对内部的每一个元素进行操作, 后面有个.map是对内部List.range(1, i)的每一个元素进行操作，最后flatMap返回的还是一个List  
上述例子也可以用for循环+yield来完成
```scala
scala>  for (i <- List.range(1, 5); j <- List.range(1, i)) yield (i,j)
res10: List[(Int, Int)] = List((2,1), (3,1), (3,2), (4,1), (4,2), (4,3))
```
3. foreach是第三种与映射类似的操作。它的右操作元是过程（返回Unit的函数）。它只是对每个列表元素都调用一遍过程。操作的结果仍然是Unit，不会产生结果列表。例如：
```scala
def main(args: Array[String]) {
    var sum =0
    List(1, 2, 3, 4, 5) foreach (sum += _)
    println(sum)
  }
-----输出----
15
```
### **列表过滤：filter、partition、find、takeWhile、dropWhile和span**
1.xs filter p操作产生xs中符合p（x）为true的所有元素组成的列表。如：
```scala
scala> List (1, 2, 3, 4, 5) filter (_ % 2 == 0)
res10: List[Int] = List(2, 4)

scala> words filter (_.length == 3)
res11: List[String] = List(the, fox)
```
2.partition方法与filter类似，不过返回的是列表对。其中一个包含所有论断为真的元素，另一个包含所有论断为假的元素。
**xs partition p  等价于 (xs filter p, xs filter (!p()))  **
举例如下：
```scala
scala> List(1, 2, 3, 4, 5) partition (_ % 2 ==0)
res12: (List[Int], List[Int]) = (List(2, 4),List(1, 3, 5))
```
3.find方法同样与filter方法类似，不过返回的是第一个满足给定论断的元素，而并不是全部。xs find p 操作以列表xs和论断p为操作元。返回可选值。如果xs中存在元素x使得p（x）为真，Some（x）将返回。否则，若p对所有元素都不成立，None将返回。举例如下：
```scala
scala> List(1, 2, 3, 4, 5) find (_ % 2 == 0)
res13: Option[Int] = Some(2)

scala> List(1, 2, 3, 4, 5) find (_  <= 0)
res15: Option[Int] = None
```
4. xs takeWhile p操作返回列表xs中最长的能够满足p的前缀。例如：
```scala
scala> List(1, 2, 3, -4, 5) takeWhile (_ > 0)
res16: List[Int] = List(1, 2, 3)
```
5.xs dropWhile p操作移除最长能够满足p的前缀。举例如下：
```scala
scala> val words = List("the", "quick", "brown", "fox")
words: List[String] = List(the, quick, brown, fox)

scala> words dropWhile (_ startsWith "t")
res11: List[String] = List(quick, brown, fox)
```
6.span方法把takeWhile和dropWhile组合成一个操作。它返回一对列表，定义与下列等式一致：
xs span p 等价于 （xs takeWhile p， xs dropWhile p） 
```scala
scala> List(1, 2, 3, -4, 5) span (_ >0)
res18: (List[Int], List[Int]) = (List(1, 2, 3),List(-4, 5))
```
###  **列表的论断：forall和exists**
1. xs forall p 如果列表的所有元素满足p则返回true
2. xs exists p 如果列表中有一个值满足p就返回true
```scala
  def hasZeroRow(m: List[List[Int]]) = m.exists(row => row forall (_ == 0))
  def main(args: Array[String]) {
    val m= List(List(3,0,0), List(0,3,0), List(0,0,3))
    var flag :Boolean= hasZeroRow(m)
    println(flag)
  }
----------输出--------
false
```
### **折叠操作**
如果我们把集合看成是一张纸条，每一小段代表一个元素，那么reduceLeft就将这张纸条从左向右”折叠”，最前面的两个元素会首先“重叠”在一起，这时会使用传给reduceLeft的参数函数进行计算，返回的结果就像是已经折叠在一起的两段纸条，它们已经是一个叠加的状态了，所以它，也就是上次重叠的结果会继续做为一个单一的值和下一个元素继续“叠加”，直到折叠到集合的最后一个元素
1. **reduceLeft**
```scala
scala>  List.range(1, 5).reduceLeft(_+_)
res12: Int = 10
```
2. **reduceRight**
和reduceLeft相似，但是是从右向左折叠,**注意**:==它的操作方向是从右到左，但是参数的顺序却并不是，而是依然第一参数是左边的元素，第二参数是右边的元素==
```scala
scala> List.range(1, 4) reduceRight(_ - _)
res15: Int = 2
// 2-3 = -1
// 1-(-1)=2
```
3. **foldLeft**
类似于reduceLeft, 不过开始折叠的第一个元素不是集合中的第一个元素，而是传入的一个元素，有点类似于先将这个参数放入的集合中的首位，然后在reduceLeft
```scala
scala> List.range(1, 5).foldLeft(1)(_+_)
res1: Int = 11
```
4. **foldRight**
类似于foldLeft，不过是从右向左，不再举例
```scala
scala> List.range(1, 4).foldRight(3)(_-_)
res1: Int = -1
```
# **scala 集合Set**
Set和List基本相似，只是所有的元素都是唯一的
默认的Set是不可变的,默认引用的是 scala.collection.immutable.Set
```scala
val set = Set(1,2,3)
println(set.getClass.getName) // 

println(set.exists(_ % 2 == 0)) //true
println(set.drop(1)) //Set(2,3)
```
如果需要使用可变集合需要引入 scala.collection.mutable.Set：
```scala
import scala.collection.mutable.Set // 可以在任何地方引入 可变集合

val mutableSet = Set(1,2,3)
println(mutableSet.getClass.getName) // scala.collection.mutable.HashSet

mutableSet.add(4)
mutableSet.remove(1)
mutableSet += 5
mutableSet -= 2

println(mutableSet) // Set(5, 3, 4)

val another = mutableSet.toSet
println(another.getClass.getName) // scala.collection.immutable.Set
```

## **连接集合**
```scala
 var site = site1 ++ site2
```
## **查找集合中最大与最小元素**
Set.min  
Set.max

## **交集**
Set.intersect
```scala
 set1.intersect(set2)
```

# **map**
ap(映射)是一种可迭代的键值对（key/value）结构。
所有的值都可以通过键来获取。
Map 中的键都是唯一的。
Map 也叫哈希表（Hash tables）。
Map 有两种类型，可变与不可变，区别在于可变对象可以修改它，而不可变对象不可以。
默认情况下 Scala 使用不可变 Map。如果你需要使用可变集合，你需要显式的引入 import scala.collection.mutable.Map 类
在 Scala 中 你可以同时使用可变与不可变 Map，不可变的直接使用 Map，可变的使用 mutable.Map。以下实例演示了不可变 Map 的应用：
不可变map
```scala
scala> val colors = Map("red" -> "#FF0000", "azure" -> "#F0FFFF")
colors: scala.collection.immutable.Map[String,String] = Map(red -> #FF0000, azure -> #F0FFFF)
```  
## **Map的赋值**  
如果需要添加 key-value 对，可以使用 + 号，如下所示：
```scala
// 空哈希表，键为字符串，值为整型
var A:Map[Char,Int] = Map()
A += ('I' -> 1)
A += ('J' -> 5)
A += ('K' -> 10)
A += ('L' -> 100)
println（A）
-------输出--------
Map(I -> 1, J -> 5, K -> 10, L -> 100)
```
## **Map的基本操作**
- keys      返回 Map 所有的键(key)
- values	返回 Map 所有的值(value)
- isEmpty	在 Map 为空时返回true  
例子
```
object Test {
   def main(args: Array[String]) {
      val colors = Map("red" -> "#FF0000",
                       "azure" -> "#F0FFFF",
                       "peru" -> "#CD853F")

      val nums: Map[Int, Int] = Map()

      println( "colors 中的键为 : " + colors.keys )
      println( "colors 中的值为 : " + colors.values )
      println( "检测 colors 是否为空 : " + colors.isEmpty )
      println( "检测 nums 是否为空 : " + nums.isEmpty )
   }
}
---------输出---------
colors 中的键为 : Set(red, azure, peru)
colors 中的值为 : MapLike(#FF0000, #F0FFFF, #CD853F)
检测 colors 是否为空 : false
检测 nums 是否为空 : true
```
## **Map 合并**
++ 运算符或 Map.++() 方法来连接两个 Map, Map 合并时会移除重复的 key。
```scala
val colors1 = Map("red" -> "#FF0000",
      "azure" -> "#F0FFFF",
      "peru" -> "#CD853F")
    val colors2 = Map("blue" -> "#0033FF",
      "yellow" -> "#FFFF00",
      "red" -> "#FF0001")

    //  ++ 作为运算符
    var colors = colors1 ++ colors2
    println( "colors1 ++ colors2 : " + colors )

    //  ++ 作为方法
    colors = colors1.++(colors2)
    println( "colors1.++(colors2)) : " + colors )
--------输出--------------------
colors1 ++ colors2 : Map(blue -> #0033FF, azure -> #F0FFFF, peru -> #CD853F, yellow -> #FFFF00, red -> #FF0001)
colors1.++(colors2)) : Map(blue -> #0033FF, azure -> #F0FFFF, peru -> #CD853F, yellow -> #FFFF00, red -> #FF0001)
```  
## **输出map和key**
```scala
def main(args: Array[String]): Unit = {
    val sites = Map("runoob" -> "http://www.runoob.com",
      "baidu" -> "http://www.baidu.com",
      "taobao" -> "http://www.taobao.com")
    sites.keys.foreach{
      i=>print("key: "+i)
        println("value: "+sites(i))
    }
  }
----------输出-----
key: runoobvalue: http://www.runoob.com
key: baiduvalue: http://www.baidu.com
key: taobaovalue: http://www.taobao.com
```  
##  **Map.contains查看是否存在指定的key**

## **元组**
元组可以把固定数量的条目组合在一起以便于整体的传送，不像数组或者列表，元祖可以保存不同类型的对象    
元组的值是通过将单个的值包含在圆括号中构成的。例如
```scala
val t = (1, 3.14, "Fred") 
```
以上实例在元组中定义了三个元素，对应的类型分别为[Int, Double, java.lang.String]。
此外我们也可以使用以下方式来定义：
```scala
val t = new Tuple3(1, 3.14, "Fred")
```  
### **定义与取值**
元组的实际类型取决于它的元素的类型，比如 (99, "runoob") 是 Tuple2[Int, String]。 ('u', 'r', "the", 1, 4, "me") 为 Tuple6[Char, Char, String, Int, Int, String]。  
目前 Scala 支持的元组最大长度为 22。对于更大长度你可以使用集合，或者扩展元组。
访问元组的元素可以通过数字索引，如下一个元组：  
我们可以使用 t.\_1 访问第一个元素， t.\_2 访问第二个元素，如下所示：  
```scala
  def main(args: Array[String]) {
    val t = (4,3,2,1)

    val sum = t._1 + t._2 + t._3 + t._4
    var secondTuple=t._2
    println("第二个元素为: "+secondTuple)
    println( "元素之和为: "  + sum )
  }
----输出----------
第二个元素为: 3
元素之和为: 10

```
### **迭代元组**
你可以使用 Tuple.productIterator() 方法来迭代输出元组的所有元素：
```scala
  def main(args: Array[String]) {
    val t = (4,3,2,1)
    t.productIterator.foreach(i=>println("value: "+i))
  }
--------输出--------------
value: 4
value: 3
value: 2
value: 1
```  
### **元组转为字符串**
Tuple.toString()  
### **元素交换**
Tuple.swap 方法来交换元组的元素  
```scala
  def main(args: Array[String]) {
      val t = new Tuple2("www.google.com", "http://blog.csdn.net/t1dmzks")
      println("交换后的元组: " + t )
    }
-------输出-----------
交换后的元组: (www.google.com,www.runoob.com)
```  
# **Scala Option**
TODO

# **Scala Iterator**
Scala Iterator（迭代器）不是一个集合，它是一种用于访问集合的方法。
迭代器 it 的两个基本操作是 next 和 hasNext。
调用 it.next() 会返回迭代器的下一个元素，并且更新迭代器的状态。
调用 it.hasNext() 用于检测集合中是否还有元素。
让迭代器 it 逐个返回所有元素最简单的方法是使用 while 循环：
```scala
object Test {
   def main(args: Array[String]) {
      val it = Iterator("Baidu", "Google", "Runoob", "Taobao")
      
      while (it.hasNext){
         println(it.next())
      }
   }
}
```
## **查找最大与最小元素**
 it.min 和 it.max 方法

## **获取迭代器的长度**
it.size 或 it.length 
