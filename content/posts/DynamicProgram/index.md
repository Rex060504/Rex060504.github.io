---
title: '动态规划'
date: '2026-09-24T21:28:22+08:00'
draft: false
author: 'rex060504'
tags: ['博客']
categories: ['学习']
summary: '关于动态规划的学习梳理'
toc: true
---

## 动态规划的核心三步

1.状态定义 —— 明确dp[i]代表什么

2.边界条件 —— 初始的基础状态决定

3.状态转移方程 —— 阶段之间的逻辑推导

## 三大特征

空间换时间 —— 利用数组或者哈希表存储中间的heritage结果

记忆化搜索 —— 避免自顶向下的重复字问题计算

无后效性 —— 过去的状态只能通过当前状态影响未来

## 解题步骤

1.确定状态表示

2.找到状态转移方程

3.确定边界和遍历顺序

## 入门题

1.[爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

2.[打家劫舍](https://leetcode.cn/problems/Gu0c2T/)

3.[打家劫舍 II](https://leetcode.cn/problems/PzWKhm/)

## 线性DP

1.[最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

2.[最大子数组和](https://leetcode.cn/problems/maximum-subarray/description/)

3.[乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)


#### 以最长递增子序列为例

题目描述：
给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

解题步骤：
1.确定状态表示：dp[i] 表示：以索引 i 结尾的（必须包含 nums[i]）最长上升子序列的长度。

2.确定边界和遍历顺序：因为每个元素本身就是一个递增子序列，所以初始化dp[i]=1。从下往上遍历，计算上层的dp[]需要借助下层的dp。

3.找到状态转移方程：
我们去检查 i 前面的每一个元素 nums[j]（其中 0 <= j < i）。
如果 nums[i] > nums[j]：说明 nums[i] 可以接到以 nums[j] 结尾的上升子序列后面，形成一个新的更长序列,此时长度变为 dp[j] + 1。

```python
dp=[1]*2505
    for i in range(n):
        for j in range(i):
            if nums[j]<nums[i]:
                if dp[j]+1>dp[i]:
                    dp[i]=dp[j]+1
    max_len=0
    for i in range(n):
        if dp[i]>max_len:
            max_len=dp[i]
```

变体：要求不仅输出长度，还要输出具体的最长子序列是哪些元素。

只需要再定义一个prev数组记录当前以nums[i]结尾的子序列中接在nums[i]前面的那个元素的下标j：prev[i]=j

```python
dp=[1]*2505
    prev=[-1]*2505
    for i in range(n):
        for j in range(i):
            if nums[j]<nums[i]:
                if dp[j]+1>dp[i]:
                    dp[i]=dp[j]+1
                    prev[i]=j
    max_len=0
    idx=0
    for i in range(n):
        if dp[i]>max_len:
            max_len=dp[i]
            idx=i
    res=[]
    while idx!=-1:
        res.append(nums[idx])
        idx=prev[idx]
    res.reverse()
```

## 二维DP

一维 DP的核心思想是“在一条线上记录状态”。
而二维 DP则是将状态扩展到“网格/矩阵”或者“两个序列/两个维度”上。

二维 DP 绝大多数题目都可以归为以下两大类：

1.网格矩阵型（Grid / Matrix）在 m * n 的棋盘或网格上移动（通常只能向下或向右），求路径数、最小/最大路径和等。状态定义： dp[i][j] 表示从起点到达网格坐标 (i, j) 的最优解/方案数。状态转移： dp[i][j] 通常由上方 dp[i-1][j] 和左方 dp[i][j-1]$ 递推而来。

2.双序列 / 串匹配型（Two Sequences / Strings）给定两个字符串/数组 S_1 和 S_2，求它们的重合、匹配、变幻关系。状态定义： dp[i][j] 表示 S_1 前 i 个字符和 S_2 前 j 个字符的匹配结果。状态转移： 比较 S_1[i-1] 和 S_2[j-1] 是否相等，决定是继承 dp[i-1][j-1] 还是从 dp[i-1][j]、dp[i][j-1] 转移。

### 练习题

1.[不同路径](https://leetcode.cn/problems/unique-paths/description/)

2.[不同路径 II](https://leetcode.cn/problems/unique-paths-ii/description/)

3.[最小路径和](https://leetcode.cn/problems/minimum-path-sum/description/)

4.[最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

5.[编辑距离](https://leetcode.cn/problems/edit-distance/)

#### 以最长公共子序列为例

题目描述：
给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。  
一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。  
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。  
两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。  

解题思路：
dp[i][j]表示text1的前i项与text2前j项中最长公共子序列的长度

当 text1[i-1] == text2[j-1] 时，dp[i][j] = dp[i-1][j-1].  
当 text1[i-1] != text2[j-1] 时,此时最长公共子序列的长度只能来源于两种可能：忽略 text1 的当前字符 和 忽略 text2 的当前字符.  
此时：dp[i][j] = max(dp[i-1][j], dp[i][j-1]).   

因存在下标为i-1和j-1，所以初始化时增加一层虚拟边界以防越界.    
遍历顺序自底向上.  

```python
    m=len(text1)
    n=len(text2)
    dp=[[0 for _ in range(n+1)] for _ in range(m+1)]
    for i in range(1,m+1):
        for j in range(1,n+1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1]+1
            else:
                dp[i][j] = max(dp[i][j-1], dp[i-1][j])
```

#### 以编辑距离为例

题目描述：
给你两个单词 word1 和 word2， 请返回将 word1 转换成 word2 所使用的最少操作数  。
你可以对一个单词进行如下三种操作：
插入一个字符. 
删除一个字符. 
替换一个字符. 

解题思路：
dp[i][j]表示word1的前i项要变成word2的前i项最少需要的操作数

word1[i-1] == word2[j-1] 时，dp[i][j] = dp[i-1][j-1]
word1[i-1] != word2[j-1] 时，我们需要从三种基本操作中选一个花费最小的，然后 +1（加上本次操作消耗）：  
1.删除word1的当前项: dp[i-1][j]+1.  
2.在word1当前项的前面插入word2的当前项: dp[i][j-1]+1.  
3.替换word1当前项为word2当前项: dp[i-1][j-1]+1.  
所以: dp[i][j]=min(dp[i-1][j],dp[i][j-1],dp[i-1][j-1])+1.  

初始化虚拟边界, dp[i][0]=i , dp[0][j]=j.  

```python
m, n = len(word1), len(word2)
        dp = [[0 for _ in range(n+1)] for _ in range(m+1)]
        for i in range(m+1):
            dp[i][0]=i
        for j in range(n+1):
            dp[0][j]=j
        for i in range(1,m+1):
            for j in range(1,n+1):
                if word1[i-1]==word2[j-1]:
                    dp[i][j]=dp[i-1][j-1]
                else:
                    dp[i][j]=min(dp[i-1][j],dp[i][j-1],dp[i-1][j-1])+1
```

## 0-1背包问题

### 问题模型与“0-1”的含义

#### 经典问题描述

有一个背包，最多只能承受重量 W。  
现在有 N 件物品，每件物品有两个属性：  
重量weight[i]   
价值value[i]    
问：在不超过背包容量限制的前提下，能装入物品的最大总价值是多少？  

#### 为什么叫 “0-1” 背包？

因为每件物品只有一种选择:    
选（1）： 把物品装进背包。    
不选（0）： 放弃该物品。    
物品不能切割，且每件物品只能用一次！（这就是它和“完全背包”的区别）。  

### 二维 DP 推导过程

1. 状态定义定义二维数组 dp[i][j]：前 i 件物品 中挑选（即物品索引为 0 ~ i-1），在背包容量为 j 时，能获得的最大总价值。

2. 状态转移方程（选 or 不选）面对第 i 件物品（重量为 w = weight[i-1]，价值为 v = value[i-1]），只有两种可能：

当前背包装不下第 i 件物品（即 j < w）：没办法，只能不选。直接继承不装这件物品时的价值：dp[i][j] = dp[i-1][j]

当前背包装得下第 i 件物品（即 j >= w ）：我们需要在“不选”和“选”之间做决策，取价值更大的那个：

不选： 保持原样 -> dp[i-1][j]

选： 留出 w 的容量装第 i 件物品，拿到价值 v，再加上剩余容量 j - w 在前 i-1 件物品中的最大价值 -> dp[i-1][j - w] + v

综合状态转移方程：dp[i][j] = max(dp[i-1][j], dp[i-1][j - weight[i-1]] + value[i-1])

```python
def knapsack_2d(weights: list[int], values: list[int], W: int) -> int:
    n = len(weights)
    # dp[i][j] 表示前 i 个物品在容量为 j 时的最大价值
    dp = [[0] * (W + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        w = weights[i - 1]
        v = values[i - 1]
        for j in range(1, W + 1):
            if j < w:
                # 容量不够，不能选
                dp[i][j] = dp[i - 1][j]
            else:
                # 抉择：不选 vs 选
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - w] + v)
                
    return dp[n][W]

# 测试用例
weights = [2, 3, 4, 5]
values = [3, 4, 5, 6]
W = 8
print("二维 DP 最大价值:", knapsack_2d(weights, values, W))  
```
时间复杂度：O(W*N)  空间复杂度:O(W*N)

### 优化：一维滚动数组（空间降维）

观察上面的二维方程：dp[i][j] = max(dp[i-1][j], dp[i-1][j - w] + v) 会发现：计算第 i 行时，只依赖于第 i-1 行的数据.   
这意味着我们根本不需要保留所有旧行，完全可以用一个一维数组 dp[j] 重复覆盖。

如果压成一维 dp[j]，转移方程变成：dp[j] = max(dp[j], dp[j - w] + v)此时容量 $j$ 必须从大到小（倒序）遍历

因为计算 dp[j] 需要用到旧的（即上一步 i-1 的）dp[j - w]。如果正序遍历，dp[j - w] 会先被更新成“包含当前物品 i”的新值，导致第 i 件物品被重复累加（变成了完全背包）。而倒序遍历能保证 dp[j - w] 依然是未经更新的旧值（只使用过一次当前物品）。

```python
def knapsack_1d(weights: list[int], values: list[int], W: int) -> int:
    n = len(weights)
    # dp[j] 表示容量为 j 时的最大价值
    dp = [0] * (W + 1)
    
    for i in range(n):
        w = weights[i]
        v = values[i]
        # 容量从大到小倒序遍历，保证每件物品只被选择一次！
        for j in range(W, w - 1, -1):
            dp[j] = max(dp[j], dp[j - w] + v)
            
    return dp[W]

# 测试
print("一维 DP 最大价值:", knapsack_1d(weights, values, W))  # 输出: 10
```
时间复杂度：O(W*N)  空间复杂度:O(W)

若j从小到大遍历，必然会有情况：j-w的值等于前面遍历过的j，此时对于当前第i物品的价值被多次累加计算（前面遍历过的dp[j]已经是加上第i个物品的价值，现在的dp[j-w]+v 使得第i个物品的价值被重复计算）。相较之下二维dp[i][j]数组是通过第一个下标i来标识，使得前面赋值过的dp[i][j]与dp[i-1][j-w]区分开.

### 练习题

1.[切割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)

## 完全背包

完全背包与01背包的不同点是物品可以重复选，故一维DP遍历时需要正序遍历（01背包需要倒序遍历）。

```python
# 0-1 背包（倒序遍历）
for w, v in items:
    for j in range(W, w - 1, -1):
        dp[j] = max(dp[j], dp[j - w] + v)

# 完全背包（正序遍历）
for w, v in items:
    for j in range(w, W + 1):  
        dp[j] = max(dp[j], dp[j - w] + v)
```

### 练习题

1.[零钱兑换](https://leetcode.cn/problems/coin-change/)

2.[零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)

#### 以零钱兑换II为例

题目描述：给你一个整数数组 coins 表示不同面额的硬币，另给一个整数 amount 表示总金额。    
请你计算并返回可以凑成总金额的硬币组合数。如果任何硬币组合都无法凑出总金额，返回 0 。   
假设每一种面额的硬币有无限个。   
题目数据 保证 最终 结果符合 32 位 带符号整数。  

示例 1：   
输入：amount = 5, coins = [1, 2, 5]
输出：4
解释：有四种方式可以凑成总金额：
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1

解：
1.dp[i][j]表示前i个硬币 凑出j金额的组合数

2.初始化dp[0——n][0]=1 ，dp[0][1——amount]=0，自底向上计算

3.当j < w 时：dp[i][j] = dp[i-1][j] 此时不选第i个元素，dp继承上一层i-1的值

当j >= w 时：dp[i][j] = dp[i-1][j] + dp[i][j-w] 此时dp[i][j]等于不选第i个元素的组合数加上至少选一个当前元素的组合数

4.将状态压缩到一维：通过for遍历的第几层控制i 

```python
        # dp[j] 表示凑成金额 j 的组合数
        dp = [0] * (amount + 1)
        dp[0] = 1 
        
        # 先遍历硬币，再正序遍历金额（求组合数）
        for coin in coins:
            for j in range(coin, amount + 1): 
                dp[j] += dp[j - coin]
                
        return dp[amount]
```
时间复杂度：O(n*amount).  空间复杂度：O(amount)

## 区间DP

 区间 DP 的核心特征是：状态定义在一个区间 [i, j] 上，我们通过从小区间（短区间）的最优解，一步步合并推出大区间（长区间）的最优解。

 dp[i][j] 表示区间 [i, j] 上的最优解。   
 我们要在这个区间里寻找一个分割点 $k$（其中 i <= k < j），把大区间分成 [i, k] 和 [k+1, j] 两部分：    
 dp[i][j] = min (dp[i][k] + dp[k+1][j] + {合并两个区间的代价})

 ### 练习题

 1.[最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

2.[戳气球](https://leetcode.cn/problems/burst-balloons/description/)

#### 以戳气球为例

题目描述：有 n 个气球，编号为0 到 n - 1，每个气球上都标有一个数字，这些数字存在数组 nums 中。    
现在要求你戳破所有的气球。戳破第 i 个气球，你可以获得 nums[i - 1] * nums[i] * nums[i + 1] 枚硬币。 这里的 i - 1 和 i + 1 代表和 i 相邻的两个气球的序号。如果 i - 1或 i + 1 超出了数组的边界，那么就当它是一个数字为 1 的气球。   
求所能获得硬币的最大数量。

示例 1：  
输入：nums = [3,1,5,8]   
输出：167   
解释：  
nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> []    
coins =  3*1*5    +   3*5*8   +  1*3*8  + 1*8*1 = 167   

解：  
如果 i - 1或 i + 1 超出了数组的边界，那么就当它是一个数字为 1 的气球。故我们用虚拟边界处理边界情况：在数组两端加上1

反向思考 哪一个元素是最后被戳破的。一个区间上的问题分解为：戳破最后一个元素的代价 这个元素的左边区间 右边区间

设dp[i][j]表示在开区间(i,j)上戳破所有元素（不包含i j）获得的最大硬币数。i < k < j，k为最后戳破的元素。此时区间被划分为五部分：i (i,k) k (k,j) j。

因为(i,k) (k,j) 事实上已经早被全部戳破 所以戳破k能得到的硬币数是：val[i]*val[k]*val[j]

对于(i,k) (k,j)区间能得到的最大硬币数分别用dp[i][k] dp[k][j]表示

状态转移方程：dp[i][j] = max(val[i]*val[k]*val[j] + dp[i][k] + dp[k][j])

初始化dp数组全为0 （显然当i-j+1 < 3时 在开区间内能得到的最大硬币数是0） 自底向上遍历

```python
    val=[1]+nums+[1]
    n=len(val)
    dp=[[0]*(n) for i in range(n)]
    for L in range(3,n+1):
        for i in range(n-L+1):
            j=i+L-1
            for k in range(i+1,j):
                coins=val[k]*val[j]*val[i]
                dp[i][j]=max(dp[i][j],dp[i][k]+dp[k][j]+coins)
```