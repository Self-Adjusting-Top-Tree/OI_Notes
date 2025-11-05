【模板】树上 Exchange-Argument，建议先做：[AGC023F](https://www.luogu.com.cn/problem/AT_agc023_f)，[CF277D](https://www.luogu.com.cn/problem/CF277D)，~~CBC020F~~

考虑在某一时刻，已经确定了两个块内部的顺序，现在要给这两个块排序。设第一个块内有 $c_1$ 个元素，其值分别为  $a_1,a_2,\ldots,a_{c_1}$；第二个块内有 $c_2$ 个元素，其值分别为 $b_1,b_2,\ldots,b_{c_2}$。在此之前已经放了 $T$ 个数，那么：

+ 若把第一个块放在第二个块之前，则这两个块对答案的贡献为 $\sum\limits_{i=1}^{c_1}a_i\times(i+T)+\sum\limits_{i=1}^{c_2}b_i\times(i+c_1+T)$。
+ 若把第二个块放在第一个块之前，则这两个块对答案的贡献为 $\sum\limits_{i=1}^{c_1}a_i\times(i+c_2+T)+\sum\limits_{i=1}^{c_2}b_i\times(i+T)$。

然后比较上面的两个式子，即：$\sum\limits_{i=1}^{c_1}a_i\times(i+T)+\sum\limits_{i=1}^{c_2}b_i\times(i+c_1+T)<\sum\limits_{i=1}^{c_1}a_i\times(i+c_2+T)+\sum\limits_{i=1}^{c_2}b_i\times(i+T)$。

容易发现此时 $T$ 不会对不等式的结果产生影响，因此删去 $T$ 的部分，不等式变为：$\sum\limits_{i=1}^{c_1}a_i\times i+\sum\limits_{i=1}^{c_2}b_i\times(i+c_1)<\sum\limits_{i=1}^{c_1}a_i\times(i+c_2)+\sum\limits_{i=1}^{c_2}b_i\times i$。

然后发现不等式左右两侧的 $a_i\times i,b_i\times i$ 部分可以约去，因此不等式变为：$\sum\limits_{i=1}^{c_2}b_ic_1<\sum\limits_{i=1}^{c_1}a_ic_2$，即 $c_1\sum\limits_{i=1}^{c_2}b_i<c_2\sum\limits_{i=1}^{c_1}a_i$。

套路的把 $c_1$ 和 $a_i$，$c_2$ 和 $b_i$ 放在一起，原不等式变为：$\frac{\sum\limits_{i=1}^{c_1}a_i}{c_1}>\frac{\sum\limits_{i=1}^{c_2}b_i}{c_2}$。此时上面部分只和 $a,b$ 数组所有元素的和有关，因此设 $s_1=\sum\limits_{i=1}^{c_1}a_i,s_2=\sum\limits_{I=1}^{c_2}b_i$，不等式变为 $\frac{s_1}{c_1}>\frac{s_2}{c_2}$。

然后把这个东西上树，由于有依赖关系，因此想到用并查集和堆来维护：堆内每个元素维护信息 $(cnt,sum,ave,id)$ 分别表示当前确定顺序的块内的元素数量 / 元素的和 / 元素的平均值 / 块内的代表元素（即深度最浅的元素），合并两个块的时候 $cnt,sum$ 是容易维护的，$ave$ 直接用 $cnt,sum$ 来算即可。每一次找到堆内 $ave$ 最大的元素所在位置 $u$，然后找到其父亲结点 $fa$，用并查集找出当前 $fa$ 所在连通块内位置最浅的节点，把 $u$ 的信息合并到 $fa$ 上即可。

这样做有一个问题是堆不能很好的支持删除位置，可能会导致一个位置被多次用来更新。但是注意到每次合并操作之后，块内的价值一定会变得更大，因此一个位置每一次用过之后把代表元素打一次标记，后面访问到该块时直接跳过即可。

总时间复杂度为 $O(\sum n\log n)$ 可以通过。

~~所以为啥这个题 $n$ 只有 $1000$ 啊，这个不是开 $10^6$ 也能做吗~~

```cpp line-numbers
#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <iostream>
#include <queue>
#include <numeric>
#include <bits/stl_algo.h>
#define int long long
using namespace std;
const int N = 500010;
const int inf = 1e18;
const int mod = 998244353;

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
int a[N], fa[N], up[N], vis[N];
struct DaSongSong { int cnt, sum; } song[N];
inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }

signed main()
{
    // freopen("1.in", "r", stdin);
    // freopen("1.out", "w", stdout);
    cin.tie(0)->sync_with_stdio(false);
    int n, R;
    while (cin >> n >> R, n || R)
    {
        for (int i = 1; i <= n; ++i)
            cin >> a[i], song[i] = {1, a[i]}, adj[i].clear();
        for (int i = 1; i < n; ++i)
        {
            int a, b;
            cin >> a >> b;
            adj[a].emplace_back(b), up[b] = a;
        }
        iota(fa + 1, fa + n + 1, 1);
        priority_queue<pair<double, int>> q;
        for (int i = 1; i <= n; ++i)
            if (i != R)
                q.push({a[i], i});
        int sum = 0;
        for (int i = 1; i <= n; ++i)
            sum += a[i], vis[i] = 0;
        while (q.size())
        {
            int id = q.top().second;
            q.pop();
            if (!vis[id])
            {
                vis[id] = 1;
                int pos = find(up[id]);
                sum += song[pos].cnt * song[id].sum;
                song[pos].cnt += song[id].cnt, song[pos].sum += song[id].sum;
                if (pos != R)
                    q.push({1. * song[pos].sum / song[pos].cnt, pos});
                fa[id] = pos;
            }
        }
        cout << sum << '\n';
    }
    return 0;
}
```