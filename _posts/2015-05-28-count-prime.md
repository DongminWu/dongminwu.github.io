---
layout: post
title: leetcode, count prime数素数
date: 2015-05-28
categories: leetcode
---

在看之前没写完的一题

>Count Primes 

>Description:
>Count the number of prime numbers less than a non-negative number, n.

>Credits:
>Special thanks to @mithmatt for adding this problem and creating all test cases.

自己先先写了一个解法

```python
def countPrimes(self, n):
        i=0
        flag=0
        for val in range(0,n):
            if self.is_prime(val):
                i=i+1
        return i;
    def is_prime(self,n):
        for i in range(3, n):
            if n % i == 0:
                return False
        return True
```

这样的话会超时, 因为时间复杂度是O(n2)
 ```
 Submission Result: Time Limit Exceeded
        Last executed input:
                	499979
 ```

 听了同事的意见, 用求根号来解决

 >如果n不是素数 n=a*b (n>a>1 n>b>1)
那么 a 和 b一定有一个不超过根号n [否则 n=a*b>(根号n)*(根号n)=n,矛盾]
于是只要除到根号n就可以判断是否是素数

后来google了一个更加棒的

(http://bookshadow.com/weblog/2015/04/27/leetcode-count-primes/)

```python
class Solution:
    # @param {integer} n
    # @return {integer}
    def countPrimes(self, n):
        isPrime = [True] * max(n, 2)
        isPrime[0], isPrime[1] = False, False
        x = 2
        while x * x < n:
            if isPrime[x]:
                p = x * x
                while p < n:
                    isPrime[p] = False
                    p += x
            x += 1
        return sum(isPrime)
```

他结合了根号 和 那个筛选法, 

```python

while x*x < n:
```

这一句限制了他在根号内取值, 因为最多x取到 sqrt(n)

然后用一个列表 存储了 筛选的结果



