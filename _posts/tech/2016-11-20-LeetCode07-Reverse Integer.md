---
layout: post
title:  "LeetCode_07"
category: tech
tags: [c++,算法,LeetCode]
description: LeetCode
---  

#### 第七题：  reverse integer
>##### 题目概述：
>**English：** Reverse digits of an integer.   
 **中文意思：** 将一个整型变量倒置，如将123变为321，将-123变为-321.
 **Example1:** x = 123, return 321 
 **Example2:** x = -123, return -321

------
>##### 题目求解：
>此题的关键不在于算法的设计，倒置相信很多同学都会写，此题关键在于溢出处理，这是很多同学容易忽略的。
* 首先对于补码的学习
  **补码范围**在数字显示中，如8位，显示的范围是 **-128 - 127**.  其补码显示为 **0x80   -   0x7F**。***1000 0000  ~  0111 1111***。
* 补码的优点：         
   (1) 减法运算可以用加法来实现，即用求和来代替求差。
   (2) 数的符号位可以同数值部分作为一个整体参与运算。
   (3) 两数的补码之和（差）=两数和（差）的补码。

---
>##### 题目补充：
  此题，没有给出的条件是，判断溢出，如果超过了返回0，有可能有同学因为没有这个判断而无法通过，加入此条件此题就不难写了。
         
----

>##### 程序代码：

~~~ c++
class Solution {
public:
    int reverse(int x) {
        if (x == 0) 
			return 0;

		int max = 0x7fffffff;        //补码表示32位int变量最大值
		int min = 0x80000000;  //补码表示32位int变量最小值
		int sign = -1;
		if (x > 0)
			sign = 1;

		x = abs(x);
		double answer = 0;
		int temp = 0;
		for (; x != 0;)
		{
			temp = x % 10;
			x = x / 10;
			answer = answer * 10 + temp;

		}
		answer = answer * sign;
		if (answer > max || answer < min)
			return 0;
		return answer;
    }
};
~~~

----
>#####Run Time
运行时间：9ms



