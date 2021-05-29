title: Leetcode --- 字符串相乘
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 19:01:00
---
### 题目
[字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)<br/>
给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。
示例1:
>
输入: num1 = "2", num2 = "3"
输出: "6"

示例2:
>
输入: num1 = "123", num2 = "456"
输出: "56088"

说明:
>
num1 和 num2 的长度小于110。<br/>
num1 和 num2 只包含数字 0-9。<br/>
num1 和 num2 均不以零开头，除非是数字 0 本身。<br/>
不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。<br/>

<!-- more -->

### 题解

感觉做起来饶了一大圈的样子，还是python的整数舒服，都不需要在意溢出的问题。
```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'


class Solution:
    def multiply(self, num1: str, num2: str) -> str:
        alist = list(num1)
        blist = list(num2)
        result = list()

        # 得到乘法的中间过程
        """
                1 2 3
            *   4 5 6
            --------------    中间过程
                7 3 8
              6 1 5 0
            4 9 2 0 0
            ---------------   结果
            5 6 0 8 8
        """
        # 逆序相乘，从个位开始计算
        # i是从右数第几个位置，方便添加0
        # 此步做乘法
        for i, m in enumerate(alist[::-1]):
            # 低位补充0
            temp = ['0'] * i
            # 进位数，例如 9 * 9 = 81，那么p=8
            p = 0
            for n in blist[::-1]:
                # 计算当前位置，并加上进位数
                x = int(m) * int(n) + p
                # 将个位数字保存
                temp.insert(0, str(x % 10))
                # 十位数进位
                p = x // 10
            # 不要忘记最后的进位数要加上
            if p > 0:
                temp.insert(0, str(p))
            # 保存中间过程
            # 逆序保存，方便之后的加法运算
            result.append(temp[::-1])
        print(result)

        # 与上面一样，不过此步是做加法，将中间过程的所有数相加
        end = list()
        p = 0
        index = 0
        while True:
            # index处所有数相加的临时结果
            tmp_result = 0
            # 用来判断是否所有的数都加上过了
            flag = False
            for r in result:
                # 如果中间过程的数中，此位置上还有数
                if len(r) > index:
                    flag = True
                    tmp_result += int(r[index])

            tmp_result += p
            end.insert(0, str(tmp_result % 10))
            p = tmp_result // 10

            # 说明所有的数都已经计算完毕
            if not flag:
                break
            # 位置往前挪
            index += 1

        return str(int(''.join(end)))


s = Solution()
n1 = "123"
n2 = "456"
print(s.multiply(n1, n2))

```