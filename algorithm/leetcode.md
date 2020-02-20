---
typora-root-url: ../../shoppon.github.io
---

# leetcode题解

## 动态规划

#### [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

思路：对于所有的拐点，依次找出接水区间；接水区间可以描述为左右能找到的最大值。区间能接的水为两边的**较小值**减去区间点的值。

当前这个点能接的雨水量为从最左边到该点的最大值与从最右边到该点的最大值中较小值与其值的差。

#### [887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

鸡蛋足够的情况下，二分应该是最快的。

k=1, n=2时，ans=1

k=2, n=2时，ans=max((1,1), (2,1))+1=1

k=2, n=3时，ans=max((1,2),(2,2)+1=2

##### 总结

1. 能够拆分成子问题，并且能降低问题的规模
2. 子问题的解不影响父问题的解，无后向性

#### [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

```
[100, 4, 200, 1, 3, 2]
```

f(n)连续序列、长度、上下限

使用字典(hash)保存每个数字当前所在连续序列的**长度**。

遍历数组，对于每个元素，其最长序列的值为缓存中相邻最长续列的和加一，如果不存在则为0。同时该数字出现后有可能将其他序列连接起来了，因此需要更新左边序列的**下限**和右边序列的**上限**。

## 字符串

#### [140. 单词拆分 II](https://leetcode-cn.com/problems/word-break-ii/)

遍历单词，如果当前子串在字典中能找到，则从下一位置开始重新查找；如果找不到则继续添加子串，如果到最后还找不到则说明无解。

如何找出所有的解？

```python
s = "catsanddog"
wordDict = ["cat", "cats", "and", "sand", "dog"]
```

## 数组

#### [4.寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

要求算法的时间复杂度为 O(log(m + n))

找出前(m+n)/2最小的数

每次比较两个数组的前两个数，POP出较小的数，直到达到(m+n)/2个。

## 图

### 概念

#### 欧拉图

#### 环

### 图的表示

<img src="/imgs/dag.png" alt="dag" style="zoom:67%;" />

邻接矩阵：

```python
[
  [0, 1, 0],
  [1, 0, 0],
  [0, 0, 0]
]
```

邻接表：使用字典保存每个起点所在边的终点集合。

```python
graph = {
  'a': ['b', 'c']
  'd': ['a', 'c']
}
```

逆邻接表：使用字典保存每个终点所在边的起点集合。

```python
graph = {
  'a': ['d'],
  'b': ['a'],
  'c': ['a', 'd']
}
```

### 练习

#### [329. 矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)

将二维矩阵转换成有向图，目标比当前点的值大则有路径，否则不可达无路径。

暴力：从最小的节点进行深搜，获取最大值，直到所有的节点均被遍历

#### 深搜非递归实现步骤：

1. 使用栈保存待访问的节点，使用集合保存已访问节点
2. 从栈中POP出下一个要访问的节点
3. 当栈非空时，依次访问当前节点的邻居，如果存在可访问的邻居节点，则将邻居加入到栈中

#### 解决超时

1. 使用记忆数组保存遍历结果？？为什么下一次访问时可以用上一次结果？

#### [207. 课程表](https://leetcode-cn.com/problems/course-schedule/)

如果一个课程不被其他课程依赖则将其移除出去。如果一个顶点没有入度，则移除。

使用bfs进行拓扑排序

#### [753. 破解保险箱](https://leetcode-cn.com/problems/cracking-the-safe/)

密码位数n，范围0~k-1，共有k^n种可能，需要将这k^n种可能放到一个长度最小的字符串中。

n=3,k=3

0001002001101112---

每次添加1位、2位形成一个未保存的新解，直到所有解都添加。

动态规划？？

n=3,k=2，共8种可能

所有的解：000001010011100101110111

如何压缩：00010100111

n=2,k=3，共9种可能性

所有的解：000102101112202122

可能的解：00112021022

如何压缩：

n=2,k=2

所有的解：000110

压缩后：00110

全排列去重。

## 回溯

#### [46. 全排列](https://leetcode-cn.com/problems/permutations/)

两个元素时，全排列为交换位置。

三个元素时，3*两个全排列

四个元素时，4*三个全排列

。。。

## 待做列表

- [827. 最大人工岛](https://leetcode-cn.com/problems/making-a-large-island/)

- [212. 单词搜索 II](https://leetcode-cn.com/problems/word-search-ii/)

- [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

- [621. 任务调度器](https://leetcode-cn.com/problems/task-scheduler/)


