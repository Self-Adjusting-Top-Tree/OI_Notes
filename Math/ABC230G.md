直接做比较困难，考虑容斥。于是得到下面的式子：

$$
\def\sl{\sum\limits_{1\le i\le j\le n}}
(\sl 1)-(\sl[\gcd(i,j)=1])-(\sl[\gcd(p_i,p_j)=1])+(\sl[\gcd(i,j)=1]\land[\gcd(p_i,p_j)=1])
$$

考虑对四个部分分别计算：

$\def\sl{\sum\limits_{1\le i\le j\le n}}\sl 1$：显然答案为 $\binom n2$。

$\def\sl{\sum\limits_{1\le i\le j\le n}}\sl[\gcd(i,j)=1]$：【模板】欧拉反演，有 $\varphi(i)=\sum\limits_{1\le j\le i}[\gcd(i,j)=1]$，因此原式的值即为 $\sum\limits_{1\le i\le n}\varphi(i)$。

$\def\sl{\sum\limits_{1\le i\le j\le n}}\sl[\gcd(p_i,p_j)]=1$：因为 $p$ 是一个 $1\sim n$ 的排列，所以 $\lbrace(i,j)\rbrace$ 和 $\lbrace(p_i,p_j)\rbrace$ 形成双射关系即两两对应，答案也为 $\def\sl{\sum\limits_{1\le i\le j\le n}}\sum\limits_{1\le i\le n}\varphi(i)$。

$\def\sl{\sum\limits_{1\le i\le j\le n}}\sl[\gcd(i,j)=1]\land[\gcd(p_i,p_j)=1]$：~~笑点解析：想了一个小时图论建模，给 $i$ 向 $p_i$ 连有向边然后在基环树上搞事。~~ 看上去很能莫反，于是开始推式子：

$$
\begin{aligned}
 &\sum\limits_{1\le i\le j\le n}[\gcd(i,j)=1]\land[\gcd(p_i,p_j)=1]\\
=&\sum\limits_{1\le i\le j\le n}(\sum\limits_{g\mid\gcd(i,j)}\mu(g))(\sum\limits_{g\mid\gcd(p_i,p_j)}\mu(g))\\
=&\sum\limits_{1\le i\le j\le n}(\sum\limits_{g\mid i}\sum\limits_{g\mid j}\mu(g))(\sum\limits_{g\mid p_i}\sum\limits_{g\mid p_j}\mu(g))\\
=&\sum\limits_{1\le i\le j\le n}(\sum\limits_{g_1\mid i}\sum\limits_{g_1\mid j}\mu(g_1))(\sum\limits_{g_2\mid p_i}\sum\limits_{g_2\mid p_j}\mu(g_2))\\
=&\sum\limits_{1\le i\le j\le n}(\sum\limits_{g_1}[g_1\mid i][g_1\mid j]\mu(g_1))(\sum\limits_{g_2}[g_2\mid p_i][g_2\mid p_j]\mu(g_2))\\
=&\sum\limits_{g_1}\sum\limits_{g_2}\mu(g_1)\mu(g_2)\sum\limits_{1\le i\le j\le n}[g_1\mid i][g_1\mid j][g_2\mid p_i][g_2\mid p_j]\\
=&\sum\limits_{g_1}\sum\limits_{g_2}\mu(g_1)\mu(g_2)\sum\limits_{1\le i\le j\le n}[(g_1\mid i)\land(g_2\mid p_i)][(g_1\mid j)\land(g_2\mid p_j)]\\
\end{aligned}
$$

然后设 $f(i,j)=\sum\limits_{k=1}^n[i\mid k][j\mid p_k]$，则原式可以简化为：

$$
\begin{aligned}
 &\sum\limits_{g_1}\sum\limits_{g_2}\mu(g_1)\mu(g_2)\sum\limits_{1\le i\le j\le n}[(g_1\mid i)\land(g_2\mid p_i)][(g_1\mid j)\land(g_2\mid p_j)]\\
=&\sum\limits_{g_1}\sum\limits_{g_2}\mu(g_1)\mu(g_2)\binom{f(g_1,g_2)+1}{2}
\end{aligned}
$$

设集合 $D_i$：$D_i=\lbrace j\mid (j\mid i,j\in D_i)\rbrace$ 即 $D_i$ 内存储 $i$ 的所有正因数，显然可以通过类似埃筛的方法 $O(n\log n)$ 求解。然后借助 $D$ 可以直接三重循环暴力求出 $f(g_1,g_2)$ 部分。而最外层的两重 $\sum$ 因为 $f(g_1,g_2)>0$ 的 $(g_1,g_2)$ 数对数量不多，所以可以直接暴力处理。

然后来算一下这个东西的上界：用调整法可以发现当且仅当 $\forall i\in[1,n],p_i=i$ 时 $\sum[f(g_1,g_2)>0]$ 的值最大，有 $\sum[f(g_1,g_2)>0]=\sum\limits_{i=1}^n(d(i)d(p_i))=\sum\limits_{i=1}^n(d(i)^2)$。容易算出该值在 $n=2\times 10^5$ 时不超过 $10^8$。

然后是一些实现时的细节：因为 $g_1,g_2$ 都是 $1\sim n$ 范围内的，所以 $(g_1,g_2)$ 数对的范围是 $O(n^2)$ 级别的。而开 `map` 时间复杂度多一只 `log` 无法接受。此时 `C++` 提供了下面的几个哈希表：

+ `std::unordered_map`
+ `__gnu_cxx::hash_map`
+ `__gnu_pbds::gp_hash_table`
+ `__gnu_pbds::cc_hash_table`

其中前两种哈希表常数过大，会 TLE。而后两种哈希表 `gp_hash_table` 使用的查探法性能略优于 `cc_hash_table` 使用的拉链法。但是在内存上，`cc_hash_table` 要略优于 `gp_hash_table`（虽然她们 `sizeof` 出来大小都是 `104`）。

![image](https://s21.ax1x.com/2025/10/08/pVHCQcd.png)

~~__gnu_cxx::hash_map 太慢了，这里就不测了~~

代码：

```cpp
#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#define int long long
using namespace std;
const int N = 200010;
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
const int i9 = inversion(9);

int a[N], phi[N], mu[N], isp[N], idx, pr[N];
inline void sieve()
{
    isp[1] = 1, mu[1] = 1, phi[1] = 1;
    for (int i = 2; i < N; ++i)
    {
        if (!isp[i])
            pr[++idx] = i, mu[i] = -1, phi[i] = i - 1;
        for (int j = 1; j <= idx && i * pr[j] < N; ++j)
        {
            isp[i * pr[j]] = 1;
            if (i % pr[j] == 0)
            {
                phi[i * pr[j]] = phi[i] * pr[j], mu[i * pr[j]] = 0;
                break;
            }
            phi[i * pr[j]] = phi[i] * phi[pr[j]], mu[i * pr[j]] = -mu[i];
        }
    }
}
inline int harsh(int i, int j) { return i * INT_MAX + j; }
pair<int, int> dududu(int i, int j) { return {i / INT_MAX, i % INT_MAX}; }
struct myhash
{
    static uint64_t rand(uint64_t x)
    {
        x ^= x << 13;
        x ^= x >> 7;
        return x ^= x << 17;
    }
    size_t operator()(size_t x) const
    {
        static const uint64_t f_rand = chrono::steady_clock::now().time_since_epoch().count();
        return rand(x + f_rand);
    }
};
__gnu_pbds::cc_hash_table<int, int, myhash> mp[N];
vector<int> divs[N];

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    int n;
    cin >> n;
    for (int i = 1; i <= n; ++i)
        cin >> a[i];
    sieve();
    for (int i = 1; i <= n; ++i)
        for (int j = i; j <= n; j += i)
            divs[j].emplace_back(i);
    int cnt = 0;
    for (int i = 1; i <= n; ++i)
    {
        for (int &j : divs[i])
            for (int &k : divs[a[i]])
                // ++mp[harsh(j, k)];
                ++mp[j][k];
    }
    for (int i = 1; i <= n; ++i)
        for (auto &j : mp[i])
            cnt += mu[i] * mu[j.first] * 1ll * j.second * (j.second + 1) / 2;
    int cnt2 = 0;
    for (int i = 1; i <= n; ++i)
        cnt2 += 2 * phi[i];
    cout << n * (n + 1) / 2 - cnt2 + cnt << '\n';
    return 0;
}
```