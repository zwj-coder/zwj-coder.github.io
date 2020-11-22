# Graph Theory

## Graph representation

邻接表和邻接矩阵

![image-20201122095642906](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122095642906.png)

![image-20201122095722777](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122095722777.png)

邻接表的空间复杂度: $O(V+E)$

邻接矩阵的空间复杂度：$O(V^2)$

图表示的选用一般跟图是否稀疏，和要使用的图算法有关。

## BFS

![image-20201122100201155](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122100201155.png)

BFS explore graph level by level from s

1. level 0 = {s}
2. level i = vertices reachable by path of i edges but not fewer
3. build level i > 0 from level i-1 by trying all outgoing edges, but ignoring vertices from previous level

```python
def bfs(s, graph):
    parent = {}
    d = {}
    parent[s] = None
    d[s] = 0
    q = Queue()
    q.put(s)
    while not q.empty():
        u = q.get()
        if graph.get(u, -1) != -1:
            for v in graph[u]:
                if v not in parent:
                    parent[v] = u
                    d[v] = d[u]+1
                    q.put(v)
    return parent, d

```

时间复杂度：$O(V+E)$

## DFS

```python
def dfs(graph):
    parent = {}
    d = {}
    discover = {}
    finish = {}
    counter = 0
    def helper(s):
        nonlocal counter
        counter += 1
        discover[s] = counter
        for v in graph.get(s, []):
            if v not in parent:
                parent[v] = s
                d[v] = d[s]+1
                helper(v)
        counter += 1
        finish[s] = counter
    for s in graph.keys():
        if s not in parent:
            parent[s] = None 
            d[s] = 0
            helper(s)
    return parent, d, discover, finish
```

dfs中可以记录两个时间戳: Discover和Finish

Discover时间戳表示节点第一次被访问的时刻

Finish时间戳表示节点的所有outgoing edges全部被访问过后的时刻。

### Parenthesis theorem

$u.d$ 表示u节点的discover时间戳

$u.f$表示u节点的finish时间戳

1. 如果$[u.d,u.f]$和$[v.d,v.f]$不重合，那么$u$和$v$互不为子关系。
2. 如果$[u.d,u.f]$完全包括在$[v.d,v.f]$中，那么u是v的子

![image-20201122101047432](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122101047432.png)

### Classification of edges

Undirected graph

1. Tree edges
2. Back edges
3. Forward edges
4. Cross edges

Directed graph

1. Tree edges
2. Back edges

## Topological sort

A topological sort of a dag G = (V, E) is a linear ordering of all its vertices such that if G contains an edge (u, v), then u appears before v in the ordering

Topological-Sort(G):

1. Call DFS(G) to compute the finish time v.f for each vertex v
2. as each vertex is finished, insert it onto the front of a linked list
3. return the linked list of vertices(the decreasing ordering of finish time)

```python
def topological_sort(graph):
    _, _, topo_discover, topo_finish = dfs(graph)
    return [k for k, v in sorted(topo_finish.items(), key=lambda item: -item[1])]
```

## Strongly Connected Component

Strongly-Connected-Components(G):

1. Call DFS(G) to compute the finishing time of every vertex
2. Compute $G^T$
3. Call DFS($G^T$), but in the main loop of DFS, consider the  vertices in order of descreasing finshing time
4. output the vertices of each tree in DFS as seperated strongly connected components

```python
def transpose(graph):
    graph_t = {}
    for key in graph.keys():
        for value in graph.get(key):
            if graph_t.get(value, -1) == -1:
                graph_t[value] = []
            graph_t[value].append(key)
    return graph_t

def strongly_connected_component(graph):
    topo = topological_sort(graph)
    graph_t = transpose(graph)
    res = {}
    parent = {}
    def dfs(s):
        nonlocal component
        nonlocal key
        for v in graph_t.get(s, []):
            if v not in parent.keys():
                component[s] = v
                parent[s] = v
                key = key + str(v)
                dfs(v)
        res[key] = component
    for u in topo:
        if u not in parent.keys():
            key = str(u)
            parent[u] = None
            component = {}
            dfs(u)
    return res, parent
```

### 算法本质

在第一步计算的finish time的基础上，在第三部按finish time降序处理tranpose graph中的节点。

假设一个简单的图包括3个connected connected components

A->B, A->C

假设原图中的完成时间最大的集合就是A,B,C

那么将原图转置后就变成了

A<-B, A<-C

由于集合A中的点没有出边到其他集合，DFS只能访问A集合中的所有点，所有最终输出component A, 这是A中的所有点都已经finish了，然后处理B，由于B的出边只有A，但是A中所有的点都被访问过了，所以B也只能访问集合内的点，输出Component B, 集合C的处理类似。

 ## Minimum Spanning Trees

一种通用策略是，假定A是某个最小生成树的子集，那么每次选择一条safe edge (u, v)，并加入到子集中，同样得到最小生成树的子集，直到最终形成最小生成树

```
Generic-MST(G, w)：
	A=empty
	while A does not form a spanning tree
		find a edge (u, v) that is safe for A
		A = A union {(u,v)}
	return A
```

两种常用算法Kruskal和Prim都是采用的贪心策略

```
def MST-KRUSKAL(G, w):
    A = empty set
    for each vertex v in G.V:
        MAKE-SET(v)
    sort the edges of G.E into nondecreasing order by weight w
    for each edge (u, v) taken in taken in nondecreasing order by weight w:
        if FIND-SET(u) !=  FIND-SET(v):
            A = A UNION {(u, v)}
            UNION(u, v)
    return 
```

```
def MST-PRIM(G, w, r):
    for each vertex u in G.V:
        u.key = inf
        u.parent = Nil
    r.key = 0
    Q = G.V
    while Q is not empty:
        u = EXTRACT-MIN(Q)
        for each v in G.adj[u]:
            if v in Q and w(u, v) < v.key:
                v.parent = u
                v.key = w(u, v)
```

## 单源最短路径

一条最重要的性质

![graph核心](C:\Users\zhangwenjum\Desktop\graph核心.PNG)

最重要的操作：松弛

只有松弛操作能够更新点的最短距离和父亲节点。

```python
def Relax(u, v, w):
	if v.d > u.d + w(u, v):
		v.d = u.d + w(u, v)
		u.parent = u
```

### Bellman-Ford算法

对图中的所有边优化|V|-1次，因为形成的最短路径一定没有环，因为如果有正环，去掉正环可以得到一个更短的路径，如果有负环就无解。所以构成的Path Tree最多含有|V|-1条表，如果一次松弛所有的边，然后这样松弛|V|-1一次，最终的效果一定和按照$(v_0,v_1),(v_1,v_2),\dots,(v_{k-1},v_k)$效果相同

![image-20201122105854636](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122105854636.png)

```cpp
const int inf = 1e8;
bool bellman_ford(int s, std::vector<std::vector<std::pair<int, int>>> &adj, std::vector<int>& d, std::vector<int>& p) {
    /*
    * def BELLMAN-FORD(G, w, s):
    *   for i = 1 to |G.V| - 1:
    *       for each edge (u,v) in G.E:
    *           RELAX(u, v, w)
    *   for each edge (u, v) in G.E:
    *       if v.d > u.d + w(u, v):
    *           return false; 
    *   return true;
    * 
    */

    int n = adj.size();
    int v = 0;
    for (auto& vs: adj) {
        v += vs.size();
    }
    d.assign(n, inf);
    p.assign(n, -1);
    d[s] = 0;
    for (int i = 0; i < v-1; ++i) {
        for (int u = 0; u < adj.size(); ++u) {
            for (auto& edge: adj[u]) {
                int v = edge.first;
                int w = edge.second;
                if (d[v] > w + d[u]) {
                    d[v] = w + d[u];
                    p[v] = u;
                }
            }
        } 
    }
    for (int u = 0; u < adj.size(); ++u) {
        for (auto& edge: adj[u]) {
            if (d[edge.first] > edge.second + d[u]) {
                return false;
            }
        }
    }
    return true;
}
```

## dag 最短路径

Dag是一类特殊的图，对dag进行拓扑排序后得到的图，任何边(u,v)，u点一定排在v的前面

所以对于任何一条最短路径$(v_0,v_1),(v_1,v_2),\dots,(v_{k-1},v_k)$, 如果我们按照拓扑排序的顺序对边进行松弛，那么最终的效果一定相同。因为对这个顺序中的任意边$(v_i, v_j)$， $v_i$一定在拓扑排序中出现在$v_j$的前面

![image-20201122110540075](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122110540075.png)

### Dijkstra算法

算法的核心在于维护两个集合A和B，A集合为以及形成最短路径树的集合，B集合为还没有形成最短路径树的集合，每次选一条A到B的最小cross edge，并进行松弛操作

![image-20201122110556581](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122110556581.png)

```cpp
void dijkstra(int s, std::vector<std::vector<std::pair<int, int>>> &adj, std::vector<int> &d, std::vector<int> &p) {
    /*
    * DIJKSTRA(G, w, s)
    * S = empty
    * Q = G.V
    * while Q is not empty:
    *   u = EXTRACT-MIN(Q)
    *   S = S union {u}
    *   for each vertex v in G.adj[u]:
    *       RELAX(u, v, w)
    */
    
    int n = adj.size();
    int u, minimum;
    d.assign(n, inf);
    p.assign(n, -1);
    d[s] = 0;
    using pii = std::pair<int,int>;
    std::priority_queue<pii, std::vector<pii>, std::greater<pii>> q;
    q.push({0, s});
    while (!q.empty()) {
        std::tie(minimum, u) = q.top();
        q.pop();
        if (minimum != d[u]) {
            continue;
        }
        for (auto& edge: adj[u]) {
            int v, w;
            std::tie(v, w) = edge;
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
                p[v] = u;
                q.push({d[v], v});
            }
        }
    }
}
```

## Floyd-Warshall algorithm

基于动态规划的策略

简单来说就是算法考虑最短路径的中间节点，一个中间节点指的是，在一条路径$p=<v_1, v_2, \dots, v_l>$中，除了$v_1$和$v_l$的任意一个节点。

首先考虑节点中的一个k子集$\{1,2,\dots,k\}$

对于任意一对节点i, j, 考虑所有中间节点都在k子集中的所有路径，假定p为其中的一条最短路径。那么floyd-warshall算法考虑两种情况，取其中的最小值：

1. 如果k不是p的中间节点，那么所有的中间节点来自于k-1子集，那么从i到j的所有中间节点在k-1子集的最短路径同时也是从i到j的所有中间节点在k子集的最短路径。满足递推关系

2. 如果k是p的中间节点，那么就可以将p分为两个部分

   i->k->j，前面一段路径是i到k的所有中间节点在k-1子集的最短路径，后面一段路径是k到j的所有中间节点在k-1子集的最短路径。

递推表达式:

$d^k_{ij}$表示在i到j的所有中间节点在k子集的最短路径的距离

那么当k=0是，最短距离就是`d[i][j]`的值，如果i到j有边就是边的权值，没有就是无穷。

如果k>1，那么就执行松弛操作

这里等号后面的第一项就是k不是所有中间节点在k子集的最短路径中的点，所以在k子集的最短距离就等于在k-1子集中的最短距离。第二项表示k是k子集的最短距离的点，那么就可以分为两段路径，每一段路径都是一个k-1子集的最短距离点。

$d^k_{ij}=min(d^{k-1}_{ij}, d^{k-1}_{ik}+d^{k-1}_{kj})$

所以算法就是：

![image-20201122112241641](C:\Users\zhangwenjum\AppData\Roaming\Typora\typora-user-images\image-20201122112241641.png)

当k等于n是，得到的最短路径p就是所用中间节点在$\{1,2,\dots,n\}$的路径。就是一个全局的最短路径。
