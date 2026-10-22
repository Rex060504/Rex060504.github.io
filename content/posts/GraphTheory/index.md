---
title: '图论初步'
date: '2026-10-05T10:49:19+08:00'
draft: false
author: 'rex060504'
tags: ['博客']
categories: ['学习']
summary: '图论的初步学习'
toc: false
---

## 图的常见存储方式

最常用的存储方式有两种：邻接矩阵 与 邻接表。

### 邻接矩阵（Adjacency Matrix）

grid[i][j] = 1（或边的权重）：表示从节点 i 到节点 j 有一条边。   
grid[i][j] = 0（或 无穷大）：表示节点 i 和 j 之间没有边。

```python
# 假设有 4 个节点（0 到 3），无向图
n = 4
edges = [[0, 1], [0, 2], [1, 3]]

matrix = [[0] * n for _ in range(n)]
for u, v in edges:
    matrix[u][v] = 1
    matrix[v][u] = 1  
    # 无向图需要双向建边
```
优点：查找节点 u 到 v 是否有边非常快（O(1)）。

缺点：空间复杂度为 O(N^2)。当节点很多但边很少（稀疏图）时，会浪费大量内存；遍历某个节点的所有邻居需要检查整行（O(N)）。

### 邻接表（Adjacency List）

用哈希表 / 列表存储每个节点的邻居列表。Python 中通常用 collections.defaultdict(list) 或列表套列表 [[] for _ in range(n)] 实现。

```python
from collections import defaultdict

n = 4
edges = [[0, 1], [0, 2], [1, 3]]

graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)  
    # 无向图双向添加；如果是有向图，只加 graph[u].append(v)
```
建好后的 graph 结构如下：
```python
{
    0: [1, 2],
    1: [0, 3],
    2: [0],
    3: [1]
}
```

优点：空间复杂度为 O(V + E)（V 为点数，E 为边数），省空间；遍历某个节点的所有邻居只需要看它的列表长度，效率极高。

缺点：判断节点 u 和 v 是否直接相连需要遍历 graph[u]。

## DFS和BFS

DFS 依赖栈（Stack / 递归系统栈），具有“后进先出”的特性，所以会一条路走到底；

BFS 依赖队列（Queue），具有“先进先出”的特性，所以会像水波纹一样逐层向外扩散。

```python
def dfs(node, visited, graph):
    # 1. 递归基 / 终止条件：节点为空或已访问
    if not node or node in visited:
        return
    
    # 2. 标记当前节点为已访问，并执行相应逻辑
    visited.add(node)
    print(f"访问节点: {node}")
    
    # 3. 递归遍历所有相邻节点
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, visited, graph)
            
    # 4. （可选）回溯逻辑：如果需要寻找所有路径而非仅仅遍历，在此处撤销标记
    # visited.remove(node)
```

BFS 需要显式维护一个队列 collections.deque（双端队列，保证O(1)的出队效率）。

```python
from collections import deque

def bfs(start_node, graph):
    visited = set([start_node]) # 1. 初始节点入集
    queue = deque([start_node]) # 2. 初始节点入队
    
    while queue:
        # 3. 取出队首节点
        curr = queue.popleft()
        print(f"访问节点: {curr}")
        
        # 4. 将相邻未访问节点扩展入队
        for neighbor in graph[curr]:
            if neighbor not in visited:
                visited.add(neighbor) # 注意：入队时就要标记 visited，防止重复入队！
                queue.append(neighbor)
```

在无权图（或边权全部相同）中求最短路径，选 BFS   
BFS 是按照距离起点的步数（1 步、2 步、3 步……）逐层向外扩展的。  
第 1 次扩展到的节点，距离起点都是 1；   
第 2 次扩展到的节点，距离起点都是 2；   
第 k 次第一次到达目标节点时，所经过的步数 $k$ 就必然是全局最短路径。一旦找到目标，可以立刻中断搜索并返回结果。

### 练习题

leetcode：

1.[寻找图中是否存在路径](https://leetcode.cn/problems/find-if-path-exists-in-graph/)

2.[钥匙和房间](https://leetcode.cn/problems/keys-and-rooms/)

3.[二进制矩阵中的最短路径](https://leetcode.cn/problems/shortest-path-in-binary-matrix/description/)

kemacoder：

1.[可达路径](https://kamacoder.com/problempage.php?pid=1170)

2.[计数孤岛](https://kamacoder.com/problempage.php?pid=1171)

3.[最大岛屿的面积](https://kamacoder.com/problempage.php?pid=1172)

4.[孤岛的总面积](https://kamacoder.com/problempage.php?pid=1173)

5.[沉没孤岛](https://kamacoder.com/problempage.php?pid=1174)

6.[高山流水](https://kamacoder.com/problempage.php?pid=1175)

7.[建造最大岛屿](https://kamacoder.com/problempage.php?pid=1176)

8.[字符串迁移](https://kamacoder.com/problempage.php?pid=1183)

#### 以二进制矩阵中的最短路径为例

尝试不光输出路径长度 还要输出最短路径。于是我们需要一个parent映射某一个节点的前驱节点,还需要记录搜索的层数.  
要准确统计 BFS 的步数/层数，通常有两种做法：   

方法 A：将步数 step 随坐标一起存入队列.   
直接让队列存储三元组 (x, y, step)，这样每个节点都自带“从起点到当前节点走了多少步”的信息
```python
    while queue:
        x, y, step = queue.popleft()
        
        if x == n - 1 and y == n - 1:
            is_arrival = True
            final_step = step
            break
            
        for nx, ny in get_neighbors(x, y):
            parent[(nx, ny)] = (x, y)
            queue.append((nx, ny, step + 1))  # 扩展出的新节点步数 +1
```

方法 B：按层循环（size = len(queue)）.  
如果一定要维护一个全局 count，必须在 while 循环内部用 for _ in range(len(queue)) 把这一层的节点一次性全部处理完，然后再对 count 进行 +1。如代码所示


```python
    n = len(grid)
    if grid[0][0] == 1 or grid[n-1][n-1] == 1:
        print(-1)
        exit(0)

    parent={(0,0):None}
    queue=deque()
    queue.append((0,0))
    count=1
    is_arrival=False

    def func(x, y ):
        direction=[(0,1),(0,-1),(-1,0),(1,0),(1,1),(-1,-1),(1,-1),(-1,1)]
        ret=[]
        for dx,dy in direction:
            nx,ny=x+dx,y+dy
            if nx<0 or nx>=n or ny<0 or ny>=n:
                continue
            if grid[nx][ny]==1:
                continue
            if (nx,ny) in parent:
                continue
            ret.append((nx,ny))
        return  ret

    while queue:
        size=len(queue)
        for _ in range(size):
            x,y=queue.popleft()
            if x==n-1 and y==n-1:
                is_arrival=True
                break
            next_level=func(x,y)
            for nx,ny in next_level:
                parent[(nx,ny)]=(x,y)
                queue.append((nx,ny))
        if is_arrival:
            break
        count+=1
    if is_arrival:
        print(count)
        cur=(n-1,n-1)
        path=[]
        while cur:
            path.append(cur)
            cur=parent[cur]
        print(path[::-1])
    else:
        print(-1)
        exit(0)
```

## 并查集

并查集功能：   
快速判断两个元素是否属于同一个集合    
快速合并两个集合    
图论里，它最常用来维护连通性：把一个连通块看成一个集合

### 并查集解决什么问题？

假设有 n 个点，初始时每个点各自独立。

现在不断加入边，比如：

1 -- 2   
3 -- 4   
2 -- 3   
我们想知道：

1 和 4 是否连通？    
当前一共有几个连通块？    
再加入一条边 1 -- 4，会不会形成环？   

并查集支持两种操作：

find(x)	查找 x 所在集合的代表元素，也就是“根”

union(a, b)	把 a 和 b 所在的两个集合合并

### 核心思想：用一棵树表示一个集合

我们用一个数组：fa[x] 表示节点 x 的父亲。

如果 fa[x] == x，说明 x 是它所在集合的根。

同一个集合里的所有节点，最终都会指向同一个根。

比如集合 {1, 2, 3, 4} 可以表示成

        1  
      /   \  
     2     3  
          /  
         4  

其中 fa[1] = 1，fa[2] = 1，fa[3] = 1，fa[4] = 3。

虽然 4 不是直接指向根 1，但通过 4 -> 3 -> 1 可以找到根。

所以，find(x) 的任务，就是从 x 出发一直往上找，直到找到根。

### 初始化

一开始每个点都是独立集合，所以每个点都是自己的根：
```python
for i in range(n):
    fa[i]=i
    sz[i]=1 // 集合大小，用于按大小合并
```
sz[i] 表示以 i 为根的集合大小。只有根节点的 sz 有意义。

### find：查找根节点 + 路径压缩

最朴素的写法：
```python
def find(x):
    if fa[x]==x:
        return x
    return find(fa[x])
```

但如果树退化成一条链：1 -> 2 -> 3 -> 4 -> 5 我的递归的深度大 速度越来越慢

于是我们想能不能把这个树的高度变低 同一个树上的所有节点的根节点root都相同 可以把子树直接插到根节点root上去 甚至最好能把同一个集合中的元素每个都直接插入根节点

状态压缩
```python
def find(x):
    if fa[x]==x:
        return x
    fa[x]=find(fa[x])
    return fa[x]
``` 
找到根之后，把沿途所有节点的父亲直接改成根。

### union：合并两个集合

合并时要注意：必须合并两个根，不能随便改某个节点的父亲。

```python
def union(a,b):
    ra=find(a)
    rb=find(b)
    if ra==rb:
        return False

    // 按大小合并：小树挂到大树下面
    if sz[a]<sz[b]:
        ra,rb=rb,ra
    fa[rb]=ra
    sz[ra]+=sz[rb]
    return True
```
为什么按大小合并？

如果每次都随便挂，树可能越来越高。    
按大小合并可以保证树高增长很慢。

### 时间复杂度

并查集的复杂度非常优秀：

只用路径压缩：均摊 O(log n).  
路径压缩 + 按大小/按秩合并：均摊 O(α(n)).  
其中 α(n) 是反阿克曼函数，增长极慢。   

在实际范围内：
α(n) <= 4

所以可以近似看成：
每次操作几乎是 O(1)

### 练习题

kamacoder:

1.[寻找存在的路线](https://kamacoder.com/problempage.php?pid=1179)

2.[多余的边](https://kamacoder.com/problempage.php?pid=1181)

## 最小生成树

### 基本概念

1. 生成树

对于一个连通无向图 G = (V, E)，它的生成树是：

一个包含图中全部 n 个顶点，但只有 n - 1 条边的连通子图。

生成树有几个关键性质：

恰好有 n - 1 条边；   
连通且无环；   
去掉任意一条边就不连通，加任意一条边就成环。    

2. 最小生成树

每条边有权值 w。最小生成树就是：

所有生成树中，边权之和最小的那一棵。   
注意：最小生成树不一定唯一，但最小权值和唯一。    

3. 什么图有最小生成树？

图必须连通，否则不存在生成树；    
不连通时，可以对每个连通块求最小生成树，得到最小生成森林。     

### Prim算法

维护一个dist[i]数组 表示i到最小生成树（已选集合）的最小距离（最小边权）

每次将离最小生成树最近的非生成树节点加入已选集合中

更新非生成树节点到生成树节点的最小距离

用优先队列（最小堆）实现每次将离最小生成树最近的非生成树节点加入已选集合中

示例代码 无向有权图

朴素版
```python
    # 邻接矩阵
    graph = [[INF] * (n + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        graph[i][i] = 0
    for _ in range(m):
        u, v, w = map(int,input().split())
        # 处理重边：保留最小的
        if w < graph[u][v]:
            graph[u][v] = w
            graph[v][u] = w
    dist = [INF] * (n + 1)
    visited=set()
    start=1
    dist[start] = 0
    ans = 0
    for _ in range(n):
        min_dist=INF
        index=0
        for i in range(1,n+1):
            if i not in visited and min_dist>dist[i]:
                min_dist=dist[i]
                index=i
        if index==0 :
            print("impossible")
            sys.exit()
        visited.add(index)
        ans+=dist[index]
        for v in range(1,n+1):
            if v not in visited and graph[index][v]<dist[v]:
                dist[v]=graph[index][v]
    print(ans) 
```

优先队列
```python
#graph[u]=[(v,w)],graph=[v]=[(u,w)]

dist=[float('inf') for _ in range(i)]
visited=set()
start=0
visited.add(start)
heap=[(0,start)]
cnt=0
total_weight=0
while heap and cnt < n:
    d,u=heapq.heappop(heap)
    if u in visited:
        continue
    visited.add(u)
    total_weight+=d
    cnt+=1
    for v,w in graph[u]:
        if v not in visited and w < dist[v]:  
            dist[v]=w
            heapq.push(heap,(w,v))
if n==cnt:
    print(total_weight)
else:
    print(impossible) 
```
时间复杂度：优先队列下 O((M+N)*log N)

### Kruskal算法  

把所有边按权值从小到大排序，然后依次考虑：

如果这条边的两个端点已经连通，加了会成环，跳过；

如果还没连通，就选它，并把两个端点合并。

直到选够 n - 1 条边为止

并查集实现

```python
# edges=[(w,u,v)] 按照w升序排序

ans = 0
cnt = 0
for w, u, v in edges:
    if union(u, v):
        ans += w
        cnt += 1
        if cnt == n - 1:
            break

if cnt == n - 1:
    print(ans)
else:
    print("impossible") 
```

排序：O(M log M) 并查集操作：O(M α(N)) 总复杂度：O(M log M).   
适合稀疏图

### 练习题

luogu:

1. [繁忙都市](https://www.luogu.com.cn/problem/P2330)

leetcode:

1. [连接所有点的最小费用](https://leetcode.cn/problems/min-cost-to-connect-all-points/description/)

2. [最小体力消耗路径](https://leetcode.cn/problems/path-with-minimum-effort/description/)

#### 以最小体力消耗路径为例

题目描述：你准备参加一场远足活动。给你一个二维 rows x columns 的地图 heights ，其中 heights[row][col] 表示格子 (row, col) 的高度。一开始你在最左上角的格子 (0, 0) ，且你希望去最右下角的格子 (rows-1, columns-1) （注意下标从 0 开始编号）。你每次可以往 上，下，左，右 四个方向之一移动，你想要找到耗费 体力 最小的一条路径。

一条路径耗费的 体力值 是路径上相邻格子之间 高度差绝对值 的 最大值 决定的。

请你返回从左上角走到右下角的最小 体力消耗值 。

思路
1. Kruskal 思想

把网格所有相邻格子之间的边按权值从小到大排序，然后依次加入并查集。

第一次出现起点和终点连通时，当前这条边的权值就是答案。

复杂度

边数 E ≈ 2 × rows × cols；  
排序 O(E log E)；   
并查集 O(E α)；   
总 O(rows × cols × log(rows × cols))    

```python
def kruskal(heights,n,m):
    fa = [i for i in range(m * n)]
    sz = [0 for i in range(m * n)]
    directions = [(1, 0), (0, 1)]
    start = 0
    destination = n * m - 1
    edges = []
    for i in range(n):
        for j in range(m):
            for di, dy in directions:
                ni = i + di
                nj = j + dy
                if ni >= n or nj >= m:
                    continue
                u, v = m * i + j, m * ni + nj
                w = abs(heights[ni][nj] - heights[i][j])
                edges.append((w, u, v))
    edges.sort()
    def find(x):
        if fa[x] == x:
            return x
        fa[x] = find(fa[x])
        return fa[x]
    def union(a, b):
        ra = find(a)
        rb = find(b)
        if ra == rb:
            return False
        if sz[ra] < sz[rb]:
            ra, rb = rb, ra
        sz[ra] += sz[rb]
        fa[rb] = ra
        return True
    for w, u, v in edges:
        if union(u, v):
            if find(start) == find(destination):
                return w
    return 0
```
2. 二分答案 + 连通性判断

“存在一条路径，最大边权 ≤ X” 这个条件，对 X 是单调的：

X 越大，保留的边越多，越容易连通；    
X 越小，保留的边越少，越难连通。

答案一定是某条边的边权，边权范围是 [0, max_diff]。

我们可以二分体力 X：

只保留边权 ≤ X 的边；   
判断起点和终点是否连通。  
如果连通，说明 X 可行，尝试更小；否则 X 太小，尝试更大。   

复杂度

二分 O(log(max_diff)) 次；
每次 BFS O(rows × cols)；
总 O(rows × cols × log(max_diff))。

```python
# 1 <= heights[i][j] <= 10e6
def bin_BFS(heights,n,m):
    def check(B):
        visited = [[False]*m for _ in range(n)]
        queue=deque()
        queue.append((0,0))
        directions = [(1, 0), (0, 1), (0, -1), (-1, 0)]
        visited[0][0]=True
        while queue:
            x,y = queue.popleft()
            if x==n-1 and y==m-1:
                return True
            for dx, dy in directions:
                nx = x + dx
                ny = y + dy
                if 0<=nx<n and 0<=ny<m and not visited[nx][ny]:
                    w=abs(heights[nx][ny] - heights[x][y])
                    if w<=B:
                        visited[nx][ny] = True
                        queue.append((nx,ny))
        return False
    lo,hi=0,10**6
    while lo<=hi:
        mid = lo+(hi-lo)//2
        if check(mid):
            hi = mid-1
        else:
            lo = mid+1
    return lo
```

3. Dijkstra 算法 

后面内容介绍

## "拓扑排序"与"环检测"

### 什么是拓扑排序（Topological Sort）

用一句话概括：将一个有向无环图（Directed Acyclic Graph DAG）的所有节点排成一个线性序列，使得对于图中的任意一条有向边(u, v)，节点 u 在序列中都出现在节点 v 的前面

直观理解假设你在大学选课：课程 A（高数）是 课程 B（线性代数）的先修课：A ——> B.   
课程 B 是 课程 C（机器学习）的先修课：B ——> C.   
那么一个合法的上课顺序就是 [A, B, C]。这种确定依赖关系执行顺序的过程，就是拓扑排序。

#### 重要结论：

1.只有“有向无环图”才能进行拓扑排序。   
2.如果图中有环（比如 A ——> B ——> C ——> A），就会陷入死循环（互相依赖），此时无法进行拓扑排序。    
3.因此，拓扑排序的过程本身就可以用来检测图中是否存在环。   

### 算法实现：Kahn 算法（基于 BFS 的入度数组法）

核心概念：入度（Indegree）

入度：指向某个节点的边数。   
物理意义：入度为 0 表示该节点没有任何前置依赖（比如没有先修课要求，随时可以选）。

Kahn 算法步骤：

1.统计入度：计算每个节点的入度，并构建邻接表。

2.队列初始化：将所有入度为 0 的节点加入队列 queue。

3.BFS 扩展：出队一个节点 u，将其加入拓扑序列 topo_order。遍历 u 的所有邻居 v，将 v 的入度减 1（代表指向 v 的前置依赖已经完成）。如果 v 的入度减到了 0，说明 v 的所有依赖均已满足，将 v 入队。

4.环检测：
    如果最后生成的 topo_order 节点数量等于图中的总节点数 N，说明拓扑排序成功（无环）。    
    如果节点数量小于 N，说明图中存在环（剩余节点的入度永远无法降到 0）。

### 练习题

1.[课程表](https://leetcode.cn/problems/course-schedule/)

```python
        graph=[[] for _ in range(numCourses)]
        indegree=[0 for _ in range(numCourses)]
        for a,b in prerequisites:
            graph[b].append(a)
            indegree[a] += 1
        queue = deque()
        topo_order=[]
        for i,x in enumerate(indegree):
            if x==0:
                queue.append(i)
        while queue:
            cur=queue.popleft()
            topo_order.append(cur)
            for u in graph[cur]:
                indegree[u] -= 1
                if indegree[u]==0:
                    queue.append(u)
        return len(topo_order)==numCourses
```
补充：三色标记法（基于 DFS 的环检测）

除了 BFS 的 Kahn 算法，也可以用 DFS 来检测有向图中是否存在环，这就需要用到三色标记法（状态机思想）：

状态 0（白色 / Unvisited）：节点尚未被访问。

状态 1（灰色 / Visiting）：节点正在被递归访问中（在当前的递归栈里）。

状态 2（黑色 / Visited）：节点及其所有子孙节点均已搜索完毕。

检测逻辑：
在 DFS 过程中，如果遍历到了一个状态为 1（灰色）的节点，说明在当前的递归路径上遇到了自己之前的祖先节点，即找到了一个环

```python
    visited = [0 for _ in range(numCourses)]
    topo_order = []
    has_cycle=False
    def dfs(i):
        nonlocal has_cycle
        visited[i] = 1

        for u in graph[i]:
            if visited[u] == 0:
                dfs(u)
                if has_cycle:
                    return
            elif visited[u] == 1:
                has_cycle = True
                return
        visited[i] = 2
        topo_order.append(i)
    for i in range(numCourses):
        if visited[i]==0:
            dfs(i)
            if has_cycle:
                return []
    return topo_order[::-1]
```

## Dijkstra 算法

### Dijkstra算法解决什么问题

#### 问题定义

给定一个**带权图**，每条边有一个**非负权值 w ≥ 0**。

求：从起点 s 到**所有其他点的最短路径长度**。

这叫**单源最短路**（Single Source Shortest Path）。

注意关键词：__单源、非负边权__

### 核心思想

每次从未确定的点中，选出当前距离起点最近的点，用它去更新邻居，然后把它“确定”下来。

维护两个数组：

dist[i] 从起点s 到点i 当前的最短距离

visited[i] i的最短路径是否已经被确定

```python
#初始化
dist[s] = 0
dist[i] = INF   #（i ≠ s）
visited[i] = False
```

每一轮：

在未确定的点中，找 dist 最小的点 u；   
把 u 标记为已确定；   
用 u 去松弛它的所有邻居 v：   

```python
if dist[u] + w(u, v) < dist[v]:
    dist[v] = dist[u] + w(u, v)
```
重复 n 轮，所有点的 dist 就都是最终答案

#### 为什么选出来的 u，它的 dist[u] 就一定是最终答案？

因为边权非负。

假设 u 是当前未确定点中 dist 最小的。如果存在另一条更短的路径到 u，那条路径一定经过某个未确定的点 x，并且：

dist[x] + 某段非负路径 ≥ dist[x] ≥ dist[u]

但 u 已经是未确定点里最小的了，所以不可能存在更短的路径。

这也是 Dijkstra 不能处理负权边的根本原因：负权会让“已经确定的点”被后来的路径推翻

### 朴素版 Dijkstra

#### 算法流程
1. 初始化 dist[s] = 0，其余 INF
2. 重复 n 次：
     a. 在未 visited 的点中，找 dist 最小的点 u
     b. 如果 dist[u] == INF，说明剩下的点都不可达，break
     c. visited[u] = True
     d. 对 u 的每个邻居 v：
           if dist[u] + w(u,v) < dist[v]:
               dist[v] = dist[u] + w(u,v)
3. 输出 dist 数组

#### 示例代码

```python
#邻接矩阵
def dijkstra(n,graph,s):
    INF=float('inf')
    dist=[INF]*n
    dist[s]=0
    visited=[False]*n
    for _ in range(n):
        min_dist=INF
        u= -1
        for i in range(n):
            if not visited[i] and (u==-1 or dist[u]>dist[i]):
                u=i
        if u==-1 or dist[u]==INF:
            break
        visited[u]=True
        for i in range(n):
            if not visited[i] and graph[u][i]!=INF:
                if dist[u]+graph[u][i]<dist[i]:
                    dist[i]=dist[u]+graph[u][i]
    return dist 
```
复杂度

每轮找最小值：O(n)，共 n 轮 → O(n²)。 
每轮松弛邻居：O(n)（邻接矩阵），共 n 轮 → O(n²)    
总复杂度：O(n²)    
空间：O(n²)（邻接矩阵）。

适合稠密图，和 Prim 朴素版一样。

### 堆优化版 Dijkstra

朴素版每轮都要扫描所有点找 dist 最小的，这一步是 O(n)。n 轮就是 O(n²)。

但如果我们用优先队列（小根堆）维护“当前所有未确定点的 dist”，每次取最小值就是 O(log n)。

#### 改进

不再每轮扫描全图，而是：

把 (dist, u) 放进小根堆；   
每次从堆顶取 dist 最小的点；   
如果这个点已经确定过，跳过（因为堆里可能有旧数据）；   
否则确定它，并用它松弛邻居，把新的 (新dist, v) 压入堆。   

#### 为什么堆里会有旧数据

因为一个点的 dist 可能被多次更新。

比如 v 先被更新成 dist[v] = 10，压入堆 (10, v)；后来又被更新成 dist[v] = 5，再压入 (5, v)。

此时堆里同时有 (10, v) 和 (5, v)。

当我们弹出 (10, v) 时，发现 v 已经被确定过了（或者 10 > dist[v]），直接跳过。

这叫懒删除：不主动从堆里删旧数据，弹出时判断一下即可。

#### 算法流程

1. dist[s] = 0，堆中放入 (0, s)
2. while 堆非空：
     a. 弹出 (d, u)
     b. 如果 visited[u]，跳过
     c. visited[u] = True
     d. 对 u 的每个邻居 (v, w)：
           if dist[u] + w < dist[v]:
               dist[v] = dist[u] + w
               堆中放入 (dist[v], v)

#### 示例代码

```python
#邻接表
import heapq
def dijkstra_heap(n,graph,s):
    INF=float('inf')
    dist=[INF]*n
    dist[s]=0
    visited=[False]*n
    heap=[(0,s)]
    while heap:
        d,u=heapq.heappop(heap)

        if visited[u]:
            continue
        visited[u]=True
        for v,w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap,(dist[v],v))
    return dist
```
复杂度

每条边最多导致一次 push：O(E log E).   
每个点最多弹出一次有效数据，旧数据也最多 E 次：O(E log E).   
总复杂度：O((V + E) log V)，常简写为 O(E log V).  
空间：O(V + E)。

适合稀疏图。

E 接近 V² → 朴素版；   
E 接近 V → 堆优化版

### 练习题

leetcode:

1. [网络延迟时间](https://leetcode.cn/problems/network-delay-time/description/)

2. [最小体力消耗路径](https://leetcode.cn/problems/path-with-minimum-effort/description/)

#### 最小体力消耗路径的Dijkstra算法

把 Dijkstra 的松弛条件改一下：

普通：dist[v] = min(dist[v], dist[u] + w)    
本题：dist[v] = min(dist[v], max(dist[u], w))   
其中 dist[u] 表示“从起点到 u 的路径上最大边权的最小值”。  

```python
def dijkstra(heights,n,m):
    INF=float('inf')
    dist=[INF]*(m*n)
    visited=[False]*(m*n)
    directions=[(1, 0),(-1,0) ,(0, 1),(0,-1)]
    dist[0]=0
    source=0
    heap=[(0,source)]
    while heap:
        d,u=heapq.heappop(heap)
        if visited[u]:
            continue
        visited[u]=True
        x,y=u//m,u%m
        for dx,dy in directions:
            nx=x+dx
            ny=y+dy
            if 0<=nx<n and 0<=ny<m:
                v=nx*m+ny
                w=abs(heights[nx][ny] - heights[x][y])
                nd=max(dist[u],w)
                if not visited[v] and nd<dist[v]:
                    dist[v]=nd
                    heapq.heappush(heap,(nd,v))
    return dist[n*m-1]
```

## Bellman-Ford算法

#### Dijkstra的局限

Dijkstra 的核心是“贪心”：每次选当前距离最小的点，然后确定它。这个做法要求边权不能为负。因为负权边可能让“已经确定”的最短路变得更短，贪心就失效了。

Bellman-Ford 解决的是：

可以有负权边的单源最短路；    
还能检测负环。

注意：如果存在从源点可达的负环，最短路没有意义，因为可以绕负环无限变小。Bellman-Ford 不能求出这种图的最短路，但能告诉你“有负环”。

### 核心思想：反复“松弛”

先定义：   
dist[v]：当前从源点 s 到 v 的最短距离估计值；   
一条边 (u, v, w) 表示 u -> v 权值为 w。    

松弛操作就是：   
如果 dist[u] + w < dist[v]
    就更新 dist[v] = dist[u] + w

对所有边做松弛，重复 n - 1 轮。   
然后再做一轮，如果还能松弛，说明存在负环。

### 为什么是n-1轮

这是理解 Bellman-Ford 的关键。

一条最短路最多包含 n - 1 条边。   
因为如果一条最短路有 n 条边，而图只有 n 个点，那么根据抽屉原理，它一定重复经过了某个点，也就是有环。   
如果这个环是正环或零环，去掉它不会更差；如果是负环，最短路就不存在。   

所以：

第 1 轮松弛后，至少能确定“最多 1 条边”的最短路；
第 2 轮后，至少能确定“最多 2 条边”的最短路；
……
第 n - 1 轮后，所有最短路都能确定。   
可以把它理解成：每一轮让最短路信息沿着边走一步，最多走 n - 1 步   

### 算法步骤

给定图 G，源点 s：

初始化：   
dist[s] = 0   
其他 dist[v] = INF   

重复 n - 1 轮：    
遍历每一条边 (u, v, w)：   
    如果 dist[u] != INF 且 dist[v] > dist[u] + w：   
        dist[v] = dist[u] + w    

负环检测：   
再遍历一遍所有边：   
    如果还能松弛，说明存在从源点可达的负环
输出 dist。   

#### 代码模板

```python
def bellman_ford(n,egdes,s):
    INF=float('inf')
    dist=[INF]*(n+1)
    dist[s]=0

    for i in range(n-1):
        updated=False
        for u,v,w in edges:
            if dist[u]!=INF and w+dist[u]<dist[v]:
                dist[v]=w+dist[u]
                update=True
        # 如果这一轮没有任何更新，后面也不会更新了
        if not updated:
            break
    for u, v, w in edges:
        if dist[u] != INF and dist[v] > dist[u] + w:
            return None  # 存在从源点可达的负环
    return dist
```

## SPFA （队列优化 Bellman-Ford）

举个极端例子：图里有 10000 条边，但这一轮只有 1 个点的距离被更新了。Bellman-Ford 仍然要把 10000 条边全扫一遍，其中 9999 条边的松弛都是无效的。

队列优化（SPFA）的核心：
只有距离被更新过的点，才值得拿它的出边去松弛别人。   
所以不再“每轮扫全图”，而是维护一个队列，里面放“距离刚刚变小、还没拿它去松弛别人”的点。每次取出一个点，只松弛它的出边；如果某条出边让邻居的距离变小了，就把邻居也扔进队列。

### 算法流程

维护：   
dist[v]：当前最短路估计；   
in_queue[v]：v 是否已经在队列里，防止重复入队；   
cnt[v]：当前从源点 s 到 v 的最短路径上，经过的边数。   
一个队列 q。   

步骤：

初始化 dist[s] = 0，其余 INF；s 入队，in_queue[s] = True。   
队列非空时，取出队首 u，in_queue[u] = False。   
遍历 u 的每一条出边 (u, v, w)：   

如果 dist[u] + w < dist[v]，更新 dist[v],cnt[v] = cnt[u]+1；
如果 v 不在队列里，把 v 入队，in_queue[v] = True.    
如果 cnt[v] >= n，说明有负环，直接返回。
队列空了，dist 就是答案。

注意：一个点可能被多次入队、多次出队。每次出队时，它拿的是当前最新的 dist[u] 去松弛，所以不会漏。

### 为什么是这样

队列里放的是“距离刚刚被改小、但还没用它松弛过出边”的点。   
初始时，只有源点 dist 被设为 0，它“刚被更新”，入队。   
每次取出 u，用它松弛出边。如果 v 被更新，说明 v 也“刚被更新”，需要入队等待处理。   
如果 v 已经在队列里，说明它还没被处理，等它出队时会用最新值，不用重复入队。
所以整个过程中，任何一次有效的距离更新，都会导致被更新的点最终被处理一次。

### cnt的理解

cnt[v]：当前从源点 s 到 v 的最短路径上，经过的边数。 

如果存在负权回路 则此回路上的某点 v 处的dist[v]就被一直松弛（减小），边数也会一直增加，永无止境。显然此时从s到v的最短路径的边数大于n-1。

在n个节点的图中（保证源点和目标点联通，无负权回路）源点到目标点最多有n-1条边相连（他们之间有n-2个节点）。如果一个权和最短的路径的边数大于n-1，则一定存在负权回路。所以我们通过从源点到某一点的最小权和路径的边数就能判断存不存在负权回路

一个点到另一个点的最小权路径显然是只有一种情况（即便有多条权和相等的路径，我们只需要任意的选择一个，只要保证权和最小即可）

假设我们此时拿出一个节点u，v是u的后驱节点。如果v被松弛，说明u->v是源点s到v的最短路径上的最后一部分。我们若想求cnt[v] 则cnt[v]=cnt[u]+1

### 复杂度

平均 / 实际：O(kM)，k 通常是个小常数（2~3 左右），稀疏图上非常快。   
最坏：仍然是 O(NM)。SPFA 在特定构造的图上会被卡到和 Bellman-Ford 一样慢，甚至更慢（因为多了队列操作）。  

### 代码模板

```python
from collections import deque
#邻接表
def spfa(n,graph,s):
    INF=float('inf')
    dist=[INF]*(n+1)
    dist[s]=0
    in_queue=[False]*(n+1)
    queue=deque([s])
    cnt=[0]*(n+1)
    in_queue[s]=True
    while queue:
        u = queue.popleft()
        in_queue[u]=False
        for v,w in graph[u]:
            if dist[u]+w < dist[v]:
                dist[v]=dist[u]+w
                cnt[v]=cnt[u]+1
                if cnt[v]>=n:
                        return None
                if not in_queue[v]:
                    queue.append(v)
                    in_queue[v]=True
    return dist
```

### 练习题

1. [城市间货物运输 I](https://kamacoder.com/problempage.php?pid=1152)

2. [城市间货物运输 II](https://kamacoder.com/problempage.php?pid=1153)

3. [城市间货物运输 III](https://kamacoder.com/problempage.php?pid=1154)

#### 以 城市间货物运输 III 为例

题目描述：点击超链接查看

计算在最多经过 k 个城市的条件下，从城市 src 到城市 dst 的最低运输成本。   
此题的关键：理解 最多经过k个城市。  
最多经过k个城市 -> src到dst的路径最多有k+1条边 - 求src可达dst所有的路径（边数小于等于k+1）中的最小权值和，如果不能达到dst则输出特殊字符。

此题有负权回路，但因路径边数的限制不会一直在回路打转。

想象一种情况：k=5 路径中最多有6条边 src->dst 在6条边的路径的最小权值和是1 相较之下4条边的路径的权值和是2。 如果按照普通的代码模板我们dist数组只会记录dist[dst]=1 但是也许在4条边的路径的情况下 其有负权回路使得在剩下2条边时可以用这2条边去走负权回路使得其权值和从当前的2变为更小。也就是说我们不能只用一维dist记录从src到dst的最小权和路径，还需要一个step记录这是在有step条边的情况下src到dst的路径的最小权和，故用二维数组dist[u][step]记录。

当前节点v的每一次松弛必然从前驱节点u转移 v的step = u的step + 1 
```python
    for v,w,in graph[u]:
        if dist[u][step] + w < dist[v][stp+1]:
            dist[v][step+1] = dist[u][step] + w
```
如果取出来的step>=k+1，说明这条路径此时已经达到了最大的边数，不能再加边了，故忽略这种情况。
```python
    if step>=k+1:
        continue
```
由于会有重复入队的情况，为了降低操作次数，我们维护一个inqueue[u][step]数组

完整代码：
```python
def func(n, graph, s,k):
    INF = float('inf')
    dist = [[INF] * (k + 2) for _ in range(n + 1)]
    dist[s][0] = 0
    queue = deque()
    queue.append((s,0))
    in_queue = [[False] * (k + 2) for _ in range(n + 1)]
    in_queue[s][0]=True
    while queue:
        u,step=queue.popleft()
        in_queue[u][step]=False
        if step>=k+1:
            continue
        for v,w in graph[u]:
            if dist[u][step]+w<dist[v][step+1]:
                dist[v][step+1]=dist[u][step]+w
                if not in_queue[v][step+1]:
                    queue.append((v, step+1))
                    in_queue[v][step+1]=True
    return dist
```