【模板】Exchange-Argument，设当前有两个组，其中第一个组内有 $x$ 个 $0$ 和 $y$ 个 $1$，第二个组内有 $y$ 个 $0$ 和 $x$ 个 $1$。则此时若把第一组放到第二组前面则两组之间的贡献为 $a\times y$，否则两组之间的贡献为 $b\times x$，因此当且仅当 $a\times y<b\times x$ 即 $\frac yx<\frac ba$ 时第一组在第二组前面。

按这个维护一个优先队列存储当前所有的组，然后用树上并查集来维护组之间的关系即可。总时间复杂度为 $O(n\log n)$，瓶颈在堆上。

```cpp
#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#define int long long
using namespace std;
const int N = 200010;
const int inf = 1e18;
const int mod = 1e9 + 7;

inline int power(int a, int b, int c)
{
    int ans = 1;
    while (b)
    {
        if (b & 1)
            ans = ans * a % c;
        a = a * a % c, b >>= 1;
    }
    return ans;
}
inline int inversion(int x) { return power(x, mod - 2, mod); }

vector<int> adj[N];
int f[N][2], a[N], up[N], res;

inline void dfs(int u, int fa)
{
    f[u][0] = 1 ^ a[u], f[u][1] = a[u], up[u] = fa;
    for (int &v : adj[u])
        if (v != fa)
            dfs(v, u);//, f[u][0] += f[v][0], f[u][1] += f[v][1];
}

struct DSU
{
    int fa[N];
    inline DSU() { iota(fa, fa + N, 0); }
    inline void init(int maxn) { iota(fa, fa + maxn + 1, 0); }
    inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
    inline int merge(int x, int y)
    {
        x = find(x), y = find(y);
        if (x != y)
            return fa[x] = y, 1;
        return 0;
    }
} dsu;

int vis[N];

using ld = long double;

struct Node
{
    ld val;
    int id;
    inline bool operator<(const Node &r) const
    {
        return val > r.val;
    }
};

// count(0) = x ; count(1) = y
inline ld calc(int x, int y)
{
    return x ? 1.L * y / x : 1e100;
}

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    int n;
    cin >> n;
    for (int i = 2; i <= n; ++i)
    {
        int x;
        cin >> x;
        adj[i].emplace_back(x);
        adj[x].emplace_back(i);
    }
    for (int i = 1; i <= n; ++i)
        cin >> a[i];
    dfs(1, 0);
    priority_queue<Node> q;
    for (int i = 1; i <= n; ++i)
        q.push({calc(f[i][0], f[i][1]), i});
    while (q.size())
    {
        int o = q.top().id;
        q.pop();
        if (vis[o])
            continue;
        vis[o] = 1;
        if (up[o])
        {
            int x = dsu.find(up[o]);
            res += f[o][0] * f[x][1];
            f[x][0] += f[o][0], f[x][1] += f[o][1];
            dsu.fa[o] = x;
            q.push({calc(f[x][0], f[x][1]), x});
        }
    }
    cout << res << '\n';
    return 0;
}
```

