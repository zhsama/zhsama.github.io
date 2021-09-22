---
title: LeetCode刷题指南
date: 2021-09-22 22:09:54
updated: 2021-09-22 22:09:56
tags: ['leetcode','算法']
categories: leetcode
description: 本人LeetCode刷题近400道后，总结出来的算法小抄。包括了回溯、双指针，滑动窗口和动态规划等常见算法的做题思路和模板。
---

# 回溯


如何判断该使用回溯：感觉如果不穷举一下就没法知道答案。


一般回溯的问题有三种：


1.  Find a path to success 有没有解
1.  Find all paths to success 求所有解
1.  求所有解的个数
1.  求所有解的具体信息
3.  Find the best path to success 求最优解



```python
def backtrack(新区域, res, path):
    if 结束:# 一般是最小值最大值
        res.add(path) # 
        return
    for 选择 in 新区域:
        if 符合要求:
            path.add(当前选择)
            backtrack(新区域, res, path)
            path.pop()
```


# 滑动窗口


### 思路1：


1.  定义两个指针 left 和 right 分别指向区间的开头和结尾，注意是闭区间；定义 sums 用来统计该区间内的各个字符出现次数；
1.  第一重 while 循环是为了判断 right 指针的位置是否超出了数组边界；当 right 每次到了新位置，需要增加 right 指针的求和/计数；
1.  第二重 while 循环是让 left 指针向右移动到 [left, right] 区间符合题意的位置；
    当 left 每次移动到了新位置，需要减少 left 指针的求和/计数；
1.  在第二重 while 循环之后，成功找到了一个符合题意的 [left, right] 区间，题目要求
    最大的区间长度，因此更新 res 为 max(res, 当前区间的长度) 。
1.  right 指针每次向右移动一步，开始探索新的区间。



```python
def findSubArray(nums):
    N = len(nums) # 数组/字符串长度
    left, right = 0, 0 # 双指针，表示当前遍历的区间[left, right]，闭区间
    sums = 0 # 用于统计 子数组/子区间 是否有效，根据题目可能会改成求和/计数
    res = 0 # 保存最大的满足题目要求的 子数组/子串 长度
    while right < N: # 当右边的指针没有搜索到 数组/字符串 的结尾
        sums += nums[right] # 增加当前右边指针的数字/字符的求和/计数
        while 区间[left, right]不符合题意：# 此时需要一直移动左指针，直至找到一个符合题意的区间
            sums -= nums[left] # 移动左指针前需要从counter中减少left位置字符的求和/计数
            left += 1 # 真正的移动左指针，注意不能跟上面一行代码写反
        # 到 while 结束时，我们找到了一个符合题意要求的 子数组/子串
        res = max(res, right - left + 1) # 需要更新结果
        right += 1 # 移动右指针，去探索新的区间
    return res
```


### 思路2：


1.  当移动 right 扩大窗口，即加入字符时，应该更新哪些数据？
1.  什么条件下，窗口应该暂停扩大，开始移动 left 缩小窗口？
1.  当移动 left 缩小窗口，即移出字符时，应该更新哪些数据？
1.  我们要的结果应该在扩大窗口时还是缩小窗口时进行更新？



```python
# s为匹配串，t为模式串
def slideWindow(s, t):
    needs, window = collections.defaultdict(int), collections.defaultdict(int)
    # 计录模式串中每个字符分别有多少个
    for i in t:
        needs[i] += 1

    left, right = 0, 0
    # 记录窗口中满足模式串字符的个数
    valid = 0
    while right < len(s):
        # 进入窗口的字符
        c = s[right]
        # 更新窗口内数据
        do something

        # debug 位置
        print(left, right)

        # 窗口收缩操作
        while left need to shrink:
            d = s[left]
            left += 1
            # 更新窗口内数据
            do something
        
        # 窗口右移
        right += 1
```


# BFS


问题的本质就是让你在一幅「图」中找到从起点 _start_ 到终点 _target_ 的最近距离


```python
def bfs(start, target):
    queue = collections.deque()
    # 记录是否经过该节点
    visited = set()
    step = 0
    # 初始化第一个节点
    queue.apped(start)
    visited.add(start)
    while queue:
        # 从当前节点向四周扩散
        for i in range(len(queue)):
            temp = queue.popleft()
            # 是否到达终点
            if temp == target:
                return step
            # 添加相邻节点
            for node in temp.相邻节点:
                if node not in visited:
                    queue.apped(node)
                    visited.add(node)
        step += 1
```


# 二分查找


二分查找场景：寻找一个数、寻找左侧边界、寻找右侧边界。


```python
def binarySeaech(nums):
    left, right = 0, len(nums)
    
    while condition:
        mid = lefg + (right -left) / 2
        if nums[mid] == target:
            do something
        elif nums[mid] < target:
            left = something
        elif nums[mid] > target:
            right = something
    return something
```


# 背包问题：


定义 dp 数组的作用十分关键！！！


必须明确dp[i][j]，i，j，分别代表什么，并为dp数组正确初始化。


然后是定义状态转移方程，之后就可以愉快地套模板了~


### 通用转移方程


1. 最值问题:



```python
dp[i] = max/min(dp[i], dp[i - nums] + 1) 或者 dp[i] = max/min(dp[i], dp[i - num] + num)
```


2. 存在问题(bool)：



```python
dp[i] = dp[i] or dp[i - num]
```


3. 组合问题：



```python
dp[i] += dp[i - num]
```


### 二维dp


```python
def bags(nums, target):
    n = len(nums)
    dp = [[0] * (target + 1) for _ in range(n + 1)]

    for i in range(n + 1):
        # 设定所需的初始状态
        dp[i][0] = [1]
    
    for i in range(1, n + 1):
        for j in range(target + 1):
            if condition:
                转移方程1
            else:
                转移方程2
    
    return dp[-1][-1]
```


### 一维dp


1. 如果是0-1背包，即数组中的元素不可重复使用，nums放在外循环，target在内循环，且内循环倒序；



```python
for num in nums:
    for i in range(target, num - 1, -1):
```


2. 如果是完全背包，即数组中的元素可重复使用，nums放在外循环，target在内循环。且内循环正序。



```python
for num in nums:
    for i in range(num, target + 1):
```


3. 如果组合问题需考虑元素之间的顺序，需将target放在外循环，将nums放在内循环。



```python
for i in range(1, target+1):
    for num in nums:
```


整体代码如下


```python
def bags(nums, target):
    n = len(nums)
    dp = [0] * (target + 1)

    # 设定所需的初始状态
    dp[0] = 1
    
    for num in nums:
        for j in range(target, num - 1, -1):
            转移方程
    
    return dp[-1]
```


# 手撸LRU


1. 新加入的节点放在链表末尾，`addNodeToLast(node)`
1. 若容量达到上限，去除最久未使用的数据，`removeHeadNode(head.next)`
1. 若数据新被访问过，比如被`get`了或被`put`了新值，把该节点挪到链表末尾，`moveNodeToLast(node)`



下图为lru的基本数据结构，用hashmap存储`{value: node}`


![](https://cdn.nlark.com/yuque/0/2021/jpeg/1431305/1628497262413-769ddb04-b2b2-4d59-95ad-2e79bcc8d57f.jpeg#height=256&id=LiL9J&originHeight=511&originWidth=997&originalType=binary&ratio=1&status=done&style=none&width=499)

`put`操作的逻辑

![](https://cdn.nlark.com/yuque/0/2021/jpeg/1431305/1628497265751-aa1e3cc8-aefa-49ef-914b-d4094dd429a9.jpeg#height=406&id=BFC0S&originHeight=811&originWidth=955&originalType=binary&ratio=1&status=done&style=none&width=478)


```python
class TreeNode:

    def __init__(self, key=0, val=0):
        self.pre = None
        self.next = None
        self.key = key
        self.val = val


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.hashmap = {}
        self.head = TreeNode()
        self.tail = TreeNode()
        self.head.next = self.tail
        self.tail.pre = self.head

    def remove_node(self, node):
        node.pre.next = node.next
        node.next.pre = node.pre

    def add_node(self, node):
        self.tail.pre.next = node
        node.pre = self.tail.pre
        node.next = self.tail
        self.tail.pre = node

    def move_node_to_last(self, node):
        self.remove_node(node)
        self.add_node(node)

    def get(self, key: int) -> int:
        if key not in self.hashmap:
            return -1
        node = self.hashmap[key]
        self.move_node_to_last(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.hashmap:
            node = self.hashmap[key]
            node.val = value
            self.move_node_to_last(node)
            return
        if len(self.hashmap) == self.capacity:
            self.hashmap.pop(self.head.next.key)
            self.remove_node(self.head.next)
        node = TreeNode(key, value)
        self.add_node(node)
        self.hashmap[key] = node
```

