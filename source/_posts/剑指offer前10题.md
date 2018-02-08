---
title: 剑指offer前10题
date: 2017-06-16 21:25:21
tags: [剑指向offer,算法,笔试题]
categories: [算法]
author: kaishun
id: 72
permalink: offer-harvester
---


## 二维数组中的查找
问题:  
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数
<!-- more -->
```
	/*思路一：
	 * 把每一行看成有序递增的数组，
	          利用二分查找，
	          通过遍历每一行得到答案，
	           时间复杂度是nlogn
	 */
	public boolean find(int [][] array,int target) {
        for(int i=0;i<array.length;i++){
        	int low=0;
        	int high=array[i].length-1;
        	while(low<=high){
        		int mid=(low+high)/2;
        		if(target>array[i][mid]){
        			low=mid+1;       			
        		}else if(target<array[i][mid]){
        			high=mid-1;
        		}else{
        			return true;
        		}
        	}
        }
        return false;
 
    }
	/*
	 * 思路二:
	 *  利用二维数组由上到下，由左到右递增的规律，
		那么选取右上角或者左下角的元素a[row][col]与target进行比较，
		当target小于元素a[row][col]时，那么target必定在元素a所在行的左边,
		即col--；
		当target大于元素a[row][col]时，那么target必定在元素a所在列的下边,
		即row++；
	 */
	public boolean find2(int [][] array,int target){
		int rows=0;
		int cols=array[0].length-1;
		while(rows<=array.length-1&&cols>=0){
			if(target==array[rows][cols]){
				return true;
			}else if(target<array[rows][cols]){
				cols--;
			}else{
				rows++;
			}
		}
		return false;
	}
```  

## 替换空格
请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
```java
//方法1
public String replaceSpace(StringBuffer str) {
    	
		return str.toString().replace(" ", "%20");
		
    }
//方法2  肯定是不能用replace函数了
    /*
     * 思路：从后往前替换；
     	（1）统计str中空格的个数
     	（2）定义两个指针变量：原始str的结束索引indexOld  和  替换后str的结束索引indexNew
     	（3）从后向前循环替换，直到indexOld与indexNew碰头为止
		代码如下（java）：
     */
public String replaceSpace2(StringBuffer str) {
        
        //遍历可变字符序列，来统计blank的个数
        int count =0;
        for (int i = 0; i < str.length(); i++) {
            if(str.charAt(i) == ' ')
                count++;
        }
        //定义两个指针变量：原始字符串的结束索引indexOld，替换后str的结束索引indexNew
        int indexOld = str.length()-1;
        int indexNew = indexOld + 2*count;
        //扩大StringBuffer的长度
        str.setLength(indexNew+1);
        //循环
        while(indexOld >=0 && indexNew > indexOld){
             
            if(str.charAt(indexOld) == ' '){
                str.setCharAt(indexNew--, '0');
                str.setCharAt(indexNew--, '2');
                str.setCharAt(indexNew--, '%');
                //注意：替换完后indexOld需要减1
                indexOld--;
            }else
                str.setCharAt(indexNew--, str.charAt(indexOld--));
        }
        return str.toString(); 
         
    }

```
## 输入一个链表，从尾到头打印链表每个节点的值。
```java
public class Solution {
    ArrayList<Integer> arrayList=new ArrayList<Integer>();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode!=null){
            this.printListFromTailToHead(listNode.next);
            arrayList.add(listNode.val);
        }
        return arrayList;
    }
}  
```
## 重建二叉树
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。  
[参考文章](http://www.cr173.com/html/18891_1.html)

## 旋转数组中的最小数字
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
```
 public int Fibonacci(int n) {
		 if(n==0){
			 return 0;
		 }else if(n==1){
			 return 1;
		 }
		 //不能用迭代，会内存溢出
//		 return Fibonacci(n-1)+Fibonacci(n-2);
		 int recode = 0; //记录上n-2的值
		 int result=1;   //记录n-1的值
		 int trans=0;  //为了中转而已
		 for(int i=2;i<=n;i++){
			 trans=result;
			 result= recode+result;
			 recode=trans;
		 }
		 return result; 
	 }

```

## 跳台阶
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
```java
/*
	 * 对于本题,前提只有 一次 1阶或者2阶的跳法。
		a.如果两种跳法，1阶或者2阶，那么假定第一次跳的是一阶，那么剩下的是n-1个台阶，跳法是f(n-1);
		b.假定第一次跳的是2阶，那么剩下的是n-2个台阶，跳法是f(n-2)
		c.由a\b假设可以得出总跳法为: f(n) = f(n-1) + f(n-2) 
		d.然后通过实际的情况可以得出：只有一阶的时候 f(1) = 1 ,只有两阶的时候可以有 f(2) = 2
		e.可以发现最终得出的是一个斐波那契数列：
	 */
	 public int JumpFloor(int target) {
		 if(target==1){
			 return 1;
		 }else if(target==2){
			 return 2;
		 }
		 //用迭代或者下面的方法
//		 return JumpFloor(n-1)+JumpFloor(n-2);
		 int recode = 1; //记录上n-2的值
		 int result=2;   //记录n-1的值
		 int trans=0;  //为了中转而已
		 for(int i=3;i<=target;i++){
			 trans=result;
			 result= recode+result;
			 recode=trans;
		 }
		 return result; 
	 }
```
## 变态跳台阶
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

分析:
f(1) = 1  
f(2) = f(2-1) + f(2-2)       
f(3) = f(3-1) + f(3-2) + f(3-3)   
...  
f(n) = f(n-1) + f(n-2) + f(n-3) + ... + f(n-(n-1)) + f(n-n) 

所以f(n)= f(n) = f(0) + f(1) + f(2) + f(3) + ... + f(n-2) + f(n-1) = f(n-1) + f(n-1)=2*f(n-1) 
所以 fn=除了0之后，其他是等比数列
```
public int JumpFloorII(int target) {
	        if(target==0){
	        	return 1;
	        }
	        return 1<<(target-1);
	    }
```  
## 矩形覆盖
我们可以用2\*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2\*1的小矩形无重叠地覆盖一个2\*n的大矩形，总共有多少种方法？
```java
//这里就直接用递归了，其实最好别用递归，万一n很大会内存溢出
public class Solution {
    public int RectCover(int target) {
        if (target < 1) {
            return 0;
        } else if (target == 1 || target == 2) {
            return target;
        } else {
            return RectCover(target-1) + RectCover(target-2);
        }
    }
}
```  
## 二进制中1的个数
输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。  
分析:  
如果一个整数不为0，那么这个整数至少有一位是1。如果我们把这个整数减1，那么原来处在整数最右边的1就会变为0，原来在1后面的所有的0都会变成1(如果最右边的1后面还有0的话)。其余所有位将不会受到影响。  负数也是一样的，因为题目中说了用补码表示
```java
public class Solution {
    public int NumberOf1(int n) {
        int count = 0;
        while(n!= 0){
            count++;
            n = n & (n - 1);
         }
        return count;
    }
}
```  
## 数值的整数次方
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。   
分析：这个题目有多种解法，有递归，累乘，都比较容易，这里介绍累乘的方法:
```java
public static double power(double base, int exponent) {
        double result = 1.0;
        for (int i = 0; i < Math.abs(exponent); i++) {
            result *= base;
        }
        if (exponent >= 0)
            return result;
        else
            return 1 / result;
    }
```
## 调整数组顺序使奇数位于偶数前面
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
