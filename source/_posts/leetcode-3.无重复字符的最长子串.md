title: leetcode --- 3.无重复字符的最长子串
author: _Tao
tags: []
categories: []
date: 2021-03-26 00:44:00
---

### 题目

[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1
>输入: s = "abcabcbb"	<br/>
输出: 3 	<br/>
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。<br/>


示例 2
> 输入: s = "bbbbb"<br/>
输出: 1	<br/>
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。	<br/>


示例 3
> 输入: s = "pwwkew"<br/>
输出: 3	<br/>
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。<br/>
     
     
示例 4:
> 输入: s = "" <br/>
输出: 0


### 解法
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        temp = ""
        length = 0
        for ss in s:
            index = temp.find(ss)
            if index == -1:
                temp += ss
            else:
                temp = temp[index + 1:] + ss
            if len(temp) > length:
                length = len(temp)

        return length


string = "aabaab!bb"
s = Solution()
print(s.lengthOfLongestSubstring(string))

```