---
layout: post
title: leetcode, 二分求幂实现pow函数
date: 2015-06-01
categories: leetcode
---

###实现二分求幂

题目:

> Pow(x, n) 

>Implement pow(x, n).


简单粗暴的只用实现求幂函数pow(x,n)

其实网上有一篇挺好的文章:

[http://blog.jobbole.com/74468/]

英文版:

[http://videlalvaro.github.io/2014/03/the-power-algorithm.html]


A过的代码如下:

```python

def myPow(self, x, n):
		y=1
		if n < 0:
			index = -n
		else:
			index = n
		while (1):
			ret = index % 2 
			if ret == 1:
				y *= x
			index /= 2
			if index==0:
				break;
			x *= x
		if n < 0:
			return 1/y
		else:
			return y

```

...

刚刚躺床上玩了半个小时手机归来.


试图自己阐述一遍思路:

原理是这样的. 比如2的6次方, 相当于3个2的2次方相乘, 或者是一个2的4次方乘以2的平方.

所以就可以以平方 为单位把求幂的过程分解

假设求x的y次方, 
用`index` 表示 y/2 之后的整数部分的值
用`mod` 表示y/2 的余数

那么一开始的算式是:
 
	result = x^y

提取一个平方

	result = x^2*(x^(y/2)*x^mod) = x^2*x^(y/2)*(x^2)*x^mod

再次提取一个平方
	
	result = x^2*x^2*(x^(y/(2*2))*x^mod)*(x^2)*x^mod
	       = (x^2)^2*x^(y/(2^2))*(x^2)^2*x^mod*x^2*x^mod

这里面就有递归迭代的意思了
, 所以呢. 无节操引用个别人的用递归实现的方法

```cpp
 class Solution 
 {
    public:
    double pow(double x, int n) 
    {
        int index = n;
        if (n == 0)
            return 1;
        if (n == 1)
            return x;
        if (n < 0)
            index = -n;
        double rev = index%2==0 ? pow(x*x, index>>1) : pow(x*x, index>>1)*x;
        if (n < 0)
            return 1 / rev;
        else
            return rev;
    }
 };

```

另外既然是二分查找, 就可以看到二进制数和二分查找的相关性, 

其实这个查找过程就和求二进制数是一样的

不断的除以二然后记录余数...直到商为0

所以可以用遍历幂的二进制形式的方法来解答, 继续盗代码

[http://blog.csdn.net/fengbingyang/article/details/12236121]

```cpp
double my_pow(double x, int n)
{
	if(n==0)
        	return 1.0;
	if(n<0)
		return 1.0 / pow(x,-n);
	double ans = 1.0 ;
	for(; n>0; x *= x, n>>=1)
	{
		if(n&1>0)
			ans *= x;
	}
	return ans;
}
```

恩关于应用呢.

在第一篇引用的文章里面有写, 可以做方便的repeat函数用, 因为求幂运算就是不断重复一个

```python

def function_foo(a,b):
	return a*b

```

的运算, 只不过传进去的是 `function_foo(x,x)`

所以这样的算法可以降低所有repeat函数的时间复杂度到O(log(n))
