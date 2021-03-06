# 924.尽量减少恶意软件的传播

https://leetcode-cn.com/problems/minimize-malware-spread

## 题目描述

```
在节点网络中，只有当 graph[i][j] = 1 时，每个节点 i 能够直接连接到另一个节点 j。

一些节点 initial 最初被恶意软件感染。只要两个节点直接连接，且其中至少一个节点受到恶意软件的感染，那么两个节点都将被恶意软件感染。这种恶意软件的传播将继续，直到没有更多的节点可以被这种方式感染。

假设 M(initial) 是在恶意软件停止传播之后，整个网络中感染恶意软件的最终节点数。

我们可以从初始列表中删除一个节点。如果移除这一节点将最小化 M(initial)， 则返回该节点。如果有多个节点满足条件，就返回索引最小的节点。

请注意，如果某个节点已从受感染节点的列表 initial 中删除，它以后可能仍然因恶意软件传播而受到感染。



示例 1：

输入：graph = [[1,1,0],[1,1,0],[0,0,1]], initial = [0,1]
输出：0
示例 2：

输入：graph = [[1,0,0],[0,1,0],[0,0,1]], initial = [0,2]
输出：0
示例 3：

输入：graph = [[1,1,1],[1,1,1],[1,1,1]], initial = [1,2]
输出：1
提示：

1 < graph.length = graph[0].length <= 300
0 <= graph[i][j] == graph[j][i] <= 1
graph[i][i] == 1
1 <= initial.length < graph.length
0 <= initial[i] < graph.length
来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/minimize-malware-spread
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

## 思路

**前置知识**

-   并查集

**入门学习资料**

-   [Using Disjoint Set (Union-Find) To Build A Maze Generator](https://coderscat.com/using-disjoint-set-union-find-to-create-maze)
-   [Disjoint Set Data Structure](https://www.topcoder.com/community/competitive-programming/tutorials/disjoint-set-data-structures/)
-   [个人笔记](https://github.com/suukii/Articles/blob/master/articles/dsa_union_find.md)

**思路 1**

-   先构建并查集，并记录每个不交集的节点数。
-   依次从 initial 中拿掉一个节点，然后分别计算感染节点数，记录最小感染数和对应的节点索引。

**思路 2**

-   构建并查集
-   找到一个满足以下条件的不交集：
    1.  集合中只有一个感染节点
    2.  集合的节点数量最大
-   如果不存在上述集合，那就返回 `initial` 中的最小索引

## 代码

TypeScript Code

UnionFind

```ts
class UnionFind {
    private parents: Array<number>
    private sizes: Array<number>

    constructor(size: number) {
        this.parents = Array(size)
            .fill(0)
            .map((_, i) => i)
        this.sizes = Array(size).fill(1)
    }

    getSizeOfSet(x: number): number {
        const px = this.findSet(x)
        return this.sizes[px]
    }

    findSet(x: number): number {
        if (x !== this.parents[x]) {
            this.parents[x] = this.findSet(this.parents[x])
        }
        return this.parents[x]
    }

    unionSet(x: number, y: number): void {
        const px: number = this.findSet(x)
        const py: number = this.findSet(y)
        if (px === py) return
        if (this.sizes[px] > this.sizes[py]) {
            this.parents[py] = px
            this.sizes[px] += this.sizes[py]
        } else {
            this.parents[px] = py
            this.sizes[py] += this.sizes[px]
        }
    }
}
```

思路 1：

```ts
function minMalwareSpread(graph: number[][], initial: number[]): number {
    // 构建并查集
    const len: number = graph.length
    const uf: UnionFind = new UnionFind(len)
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j < len; j++) {
            graph[i][j] === 1 && uf.unionSet(i, j)
        }
    }

    // 计算最终感染节点数
    const countInfected = (initial: number[]): number => {
        const parents: number[] = []
        for (let i = 0; i < initial.length; i++) {
            const p = uf.findSet(initial[i])
            parents.find(e => e === p) || parents.push(p)
        }
        return parents.reduce(
            (res: number, p: number): number => res + uf.getSizeOfSet(p),
            0,
        )
    }

    let ans: number = 0
    let leastInfected: number = Infinity

    for (let i = 0; i < initial.length; i++) {
        // 把第 i 个节点拿掉，然后计算感染节点数
        const infected: number = countInfected([
            ...initial.slice(0, i),
            ...initial.slice(i + 1),
        ])
        // 如果感染数更少，更新 ans
        if (infected < leastInfected) {
            leastInfected = infected
            ans = initial[i]
        }
        // 如果感染数一样少，取索引值较小的那个
        else if (infected === leastInfected) {
            initial[i] < ans && (ans = initial[i])
        }
    }
    return ans
}
```

思路 2：

```ts
function minMalwareSpread(graph: number[][], initial: number[]): number {
    // 构建并查集
    const len: number = graph.length
    const uf: UnionFind = new UnionFind(len)
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j < len; j++) {
            graph[i][j] === 1 && uf.unionSet(i, j)
        }
    }

    // 计算每个不交集中分别有多少个感染节点
    const malwares: number[] = Array(len).fill(0)
    initial.forEach(i => malwares[uf.findSet(i)]++)

    // ans: [集合的节点数总数，感染节点索引]
    let ans: [number, number] = [1, Infinity]
    for (let i of initial) {
        // 如果某个集合中只有一个感染节点
        if (malwares[uf.findSet(i)] === 1) {
            const count = uf.getSizeOfSet(i)
            // 如果这个集合的节点数更多，更新 ans
            if (count > ans[0]) {
                ans = [count, i]
            }
            // 如果有多个满足条件的集合，取索引值更小的那个节点
            else if (count === ans[0]) {
                ans = [count, Math.min(ans[1], i)]
            }
        }
    }
    // 如果不存在满足条件的集合，返回感染节点中索引值最小的那个节点
    return ans[1] === Infinity ? Math.min(...initial) : ans[1]
}
```
