---
title: 欧拉回路
toc: true
date: 2024-11-06 10:09:17
tags:
  - 图论
  - 欧拉回路
---

网络上关于求欧拉回路的线性算法的资料普遍缺少证明。本文将通过分析欧拉回路的性质正向推导出这一算法。

## 算法流程

基本的定义可以参考 [Alex_Wei 的博客](https://www.cnblogs.com/alex-wei/p/basic_graph_theory.html)，本文不再赘述。

算法流程部分仅推导求无向图欧拉回路的算法，求有向图欧拉回路的算法的推导过程是类似的，更改一些对应术语即可。

显然图中的孤立点与欧拉回路无关，本小节假设要么图中不存在孤立点，要么整张图就是一个孤立点。

**性质 1**：无向图 $G$ 是欧拉图，当且仅当图 $G$ 连通，且 $\forall u \in G$，$2 \mid d(u)$。

>   证明：必要性显然，下证充分性。
>
>   考虑归纳法。$|E| = 0$ 时结论显然成立。假设 $|E| < m$ 时结论成立，下证 $|E| = m$ 时结论成立。
>
>   显然图中至少存在一个环（否则图将会是一棵树，树的叶子节点度数为 $1$，矛盾）。删除环上的边后图可能分裂成若干个连通块，但每个连通块中至少存在一个环上的点，并且仍然满足 $\forall u \in G$，$2 \mid d(u)$。由归纳假设，每个连通块都存在欧拉回路，用环把这些欧拉回路串起来即可得到原图的欧拉回路。

**性质 2**：若无向图 $G$ 是欧拉图，则 $G$ 是边双连通图。

>   证明：性质 1 的证明直接导出了这一结论。

**性质 3**：无向图 $G$ 是半欧拉图，当且仅当图 $G$ 连通，且 $\sum_{u \in G} [2 \nmid d(u)] = 2$。并且，设满足 $2 \nmid d(u)$ 的两个点分别为 $s, t$，则 $G$ 的欧拉迹的两个端点一定为 $s, t$。

>   证明：必要性显然，下证充分性。
>
>   在 $s, t$ 之间再连一条边，则图会变成一张欧拉图。把新图的欧拉回路中 $(s, t)$ 这条边删掉即可得到原图的一条欧拉迹。

现在考虑构造一张欧拉图的欧拉回路或一张半欧拉图的欧拉迹。考虑递归构造，定义函数 $dfs(s)$ 返回 $s$ 所在的连通块的生成子图中一条 $s \rightsquigarrow s$ 的欧拉回路或一条 $s \rightsquigarrow t$ 的欧拉迹。

若整张图就是一个孤立点，则可以返回一条空途径 $s$。否则任取 $s$ 的一条邻边 $(s, v)$ 并删除，有以下两种情况：

1.  图变得不连通，则 $(s, v)$ 是原图的一条割边。由性质 2，我们要求的一定是一条欧拉迹。并且，$t$ 一定在 $v$ 所在的连通块，因为假设在原图加入 $(s, t)$ 这条边则原图会变得连通。因此可以返回 $dfs(s) + dfs(v)$。
2.  图仍然连通。可以返回 $s + dfs(v)$。

这样我们就得到了一个 $O(m (n + m))$ 的算法，瓶颈在于判断图是否连通。

我们希望避免判断图是否连通，即合并两种情况。实际上这是非常简单的。可以先调用 $dfs(v)$，设结果为 $a$，再调用 $dfs(s)$，设结果为 $b$，最后返回 $b + a$。这是因为如果图不连通，则 $dfs(s)$ 和 $dfs(v)$ 的过程互不干扰，否则 $dfs(v)$ 时会把图删空，$dfs(s)$ 会直接返回 $s$。

这样我们就得到了一个 $O(n + m)$ 的算法，不过实现时有一些需要注意的地方：

-   删边可以用打标记 + 当前弧优化代替。
-   $dfs$ 并不需要真的返回一条途径，只需要不断向构造的途径里 prepend 新的点。这可以通过在 $dfs$ 过程中执行 append，最后执行一次 reverse 解决。
-   不断 $dfs(s)$ 其实很蠢，这相当于枚举 $s$ 的每条出边并执行 $dfs(v)$，回溯前再 append $s$。

这样就得到了我们平时实现的算法。

## 参考实现

模板题：[UOJ117 欧拉回路](https://uoj.ac/problem/117)。本题要求输出边的顺序，只需要把回溯前 append $s$ 这一步改成每次调用 $dfs(v)$ 之后 append $(s, v)$ 这条边即可。

```c++
#include <bits/stdc++.h>

using namespace std;

#define all(x) begin(x), end(x)
#define len(x) int((x).size())

using ll = long long;
using db = long double;

template <typename T>
inline bool ckmin(T &x, const T &y) {
    return y < x and (x = y, true);
}

template <typename T>
inline bool ckmax(T &x, const T &y) {
    return x < y and (x = y, true);
}

int n, m;

namespace undirected {
constexpr int N = 1e5 + 5, M = 4e5 + 5;

int tot = 1, hd[N], nxt[M], to[M], d[N];
bool vis[M];
vector<int> p;

void ae(int u, int v) {
    ++d[u], ++d[v];
    nxt[++tot] = hd[u], hd[u] = tot, to[tot] = v;
    nxt[++tot] = hd[v], hd[v] = tot, to[tot] = u;
}

void dfs(int u) {
    for (int &i = hd[u]; i > 0; i = nxt[i]) {
        if (vis[i / 2]) {
            continue;
        }
        vis[i / 2] = true;
        int tmp = i % 2 == 1 ? -(i / 2) : i / 2;
        dfs(to[i]);
        p.push_back(tmp);
    }
}

void proc() {
    for (int i = 1; i <= m; ++i) {
        int u, v;
        cin >> u >> v;
        ae(u, v);
    }
    int s = 0;
    for (int i = 1; i <= n; ++i) {
        if (d[i] % 2 == 1) {
            cout << "NO\n";
            return;
        }
        if (d[i] > 0) {
            s = i;
        }
    }
    if (s != 0) {
        dfs(s);
    }
    if (len(p) < m) {
        cout << "NO\n";
        return;
    }
    cout << "YES\n";
    for (int i = m; --i >= 0;) {
        cout << p[i] << " \n"[i == 0];
    }
}
}

namespace directed {
constexpr int N = 1e5 + 5, M = 2e5 + 5;

int tot, hd[N], nxt[M], to[M], inD[N], outD[N];
vector<int> p;

void ae(int u, int v) {
    ++inD[v], ++outD[u];
    nxt[++tot] = hd[u], hd[u] = tot, to[tot] = v;
}

void dfs(int u) {
    int &i = hd[u];
    while (i > 0) {
        int tmp = i;
        i = nxt[i];
        dfs(to[tmp]);
        p.push_back(tmp);
    }
}

void proc() {
    for (int i = 1; i <= m; ++i) {
        int u, v;
        cin >> u >> v;
        ae(u, v);
    }
    int s = 0;
    for (int i = 1; i <= n; ++i) {
        if (inD[i] != outD[i]) {
            cout << "NO\n";
            return;
        }
        if (inD[i] > 0) {
            s = i;
        }
    }
    if (s != 0) {
        dfs(s);
    }
    if (len(p) < m) {
        cout << "NO\n";
        return;
    }
    cout << "YES\n";
    for (int i = m; --i >= 0;) {
        cout << p[i] << " \n"[i == 0];
    }
}
}

int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    cin.exceptions(cin.failbit);

    int tc;
    cin >> tc >> n >> m;
    if (tc == 1) {
        undirected::proc();
    } else {
        directed::proc();
    }
}
```
