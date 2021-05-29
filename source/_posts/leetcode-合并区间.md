title: Leetcode --- 合并区间
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-02-02 00:36:00
---

### 题目
[合并区间](https://leetcode-cn.com/problems/merge-intervals/)

以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。


示例 1：
```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```
示例 2：
```
输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
 ```

提示：
```
1 <= intervals.length <= 10^4
intervals[i].length == 2
0 <= starti <= endi <= 10^4
```

### 解答
```python
class Solution(object):
    def merge(self, intervals):
        """
        :type intervals: List[List[int]]
        :rtype: List[List[int]]
        """
        intervals = sorted(intervals, key=lambda x: x[0])
        current = intervals[0]
        rv = list()
        for inter in intervals[1:]:
            # 如果无法合并的话, 那么将前一个区间放入列表
            # current移动到后一个区间上
            if current[1] < inter[0]:
                rv.append(current)
                current = inter
            # 前一个区间的较大值, 在后一个区间范围内
            # 合并两个区间, 但此时不将current放入列表, 需要继续合并
            elif inter[0] <= current[1] <= inter[1]:
                current = [current[0], inter[1]]
            # 前一个区间包括了后一个区间, 不做处理
            else:
                pass

        rv.append(current)
        return rv


if __name__ == "__main__":
    i = [[1, 3], [2, 6], [8, 10], [15, 18]]
    s = Solution()
    print(s.merge(i))

```

