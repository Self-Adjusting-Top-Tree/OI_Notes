[弱化版：P13382 [GCJ 2011 Finals] Runs](https://www.luogu.com.cn/problem/P13382)

考虑连续段 dp。为了方便处理，这里把字符串 $s$ 离散化，记 $w$ 表示 $s$ 中最大的字符。

设 $f_{i,j}$ 表示当前用了前 $i$ 个字符，当前有恰好 $j$ 个连续段的方案数。因为 $s$ 串被离散化了，所以初始条件为 $f_{1,1}=1$。

转移时考虑插入一个新的**连续段**（注意：不是单个字符），考虑将其插入对答案造成的影响：

+ 插入到两个连续段之间：新增一个连续段
+ 插入到一个连续段内：新增两个连续段

因此想到一个简单的转移：若当前用到第 $i$ 个字符，当前恰有 $j$ 个连续段，那么枚举插入到一个连续段内的连续段数 $k$，插入到两个连续段之间的连续段数 $p$（需满足 $k\neq 0\lor p\neq 0$ 且 $j+2k+p\le cnt$，其中 $cnt$ 是原始串 $s$ 的连续段数量），转移方程即为：

$$
f_{i+1,j+2k+p}\leftarrow f_{i,j}\binom{buc_i-1}{k+p-1}\binom{j+1}{p}\binom{pre_i-j}{k}
$$

其中 $buc_i$ 为串 $s$ 中字符 $i$ 出现的次数，$pre_i$ 为串 $s$ 中字符 $1$ 到字符 $i$ 出现次数的总和（$s$ 为离散化后的串）。

直接暴力转移时间复杂度为 $O(\sum n^3\omega)$，其中 $\omega$ 为字符集大小，这里取 $\omega=26$。

在 $O(\sum n^3\omega)$ 的 dp 做法上进行优化：直接做是困难的，所以可以想到去搞一些容斥之类的事。

改设 $f_{i,j}$ 表示当前用到第前 $i$ 个字符，**当前最多有 $j$ 个连续段的预留端点**的方案数。

此时转移的时候枚举最后 $i$ 字符形成了 $k$ 个连续段，此时 $k$ 个连续段会占用 $j$ 个连续段的预留端点，因为其和顺序无关，所以方案数为 $\binom jk$。

然后还需要把 $buc_i$ 个字符拆成 $k$ 个连续段，这是经典问题，方案数为 $\binom{buc_i-1}{k-1}$。

最后剩余的 $j-k$ 个分段点分给前 $i-1$ 个字符，发现这个东西其实就是 $f_{i,j}$ 的一个子问题，可以递归解决，方案数即为 $f_{i-1,j-k}$。

显然可以通过加乘原理来完成整体的转移方程式：

$$f_{i,j}=\sum\limits_{k=1}^{\min(j,buc_i)}\binom jk\binom{buc_i-1}{k-1}f_{i-1,j-k}$$

算答案部分可以直接套用二项式反演公式，对于每个 $m\in[1,n]$，答案为 $\sum\limits_{i=1}^m(-1)^{m-i}\binom{n-i}{m-i}f_{w,i}$。预处理阶乘和阶乘逆，因为有 $\sum buc_i=n$ 所以总时间复杂度为 $O(\sum n^2\omega)$，其中 $\omega$ 取字符集大小，这里有 $\omega=26$。

这个时间复杂度仍然无法通过，考虑继续优化。先优化 $f$ 这个 dp 数组的求值过程。套路的拆组合数，得到：

$$
\begin{aligned}
f_{i,j}
&=\sum\limits_{k=1}^{\min(j,buc_i)}\frac{j!}{k!(j-k)!}\times\frac{(buc_i-1)!}{(k-1)!(buc_i-k)!}f_{i-1,j-k}\\
&=j!(buc_i-1)!\sum\limits_{k=1}^{\min(j,buc_i)}\frac{1}{k!(j-k)!(k-1)!(buc_i-k)!}f_{i-1,j-k}\\
&=j!(buc_i-1)!\red{\sum\limits_{k=1}^{\min(j,buc_i)}\frac{1}{k!(k-1)!(buc_i-k)!}\times\frac{f_{i-1,j-k}}{(j-k)!}}
\end{aligned}
$$

此时我们惊喜的发现式子后面的红色部分是一个等和卷积，可以用 NTT 进行优化，在该部分做到时间复杂度 $O(n\omega\log n)$。

然后再考虑优化算答案部分。对于每个 $m\in[1,n]$，答案的形式形如：

$$\sum\limits_{i=1}^m(-1)^{m-i}\binom{n-i}{m-i}f_{w,i}$$

仍然是套路的拆组合数，得到：

$$
\begin{aligned}
\text{res}_m
&=\sum\limits_{i=1}^m(-1)^{m-i}\binom{n-i}{m-i}f_{w,i}\\
&=\sum\limits_{i=1}^m(-1)^{m-i}\frac{(n-i)!}{(n-m)!(m-i)!}f_{w,i}\\
&=\frac1{(n-m)!}\red{\sum\limits_{i=1}^m(\frac{(-1)^{m-i}}{(m-i)!}\times(f_{w,i}\times(n-i)!))}
\end{aligned}
$$

后面红色部分的答案又是一个等和卷积的形式，同样可以使用 NTT 优化。因此本题做完了，总时间复杂度为 $O(\sum n\omega\log n)$ 可以通过。

----

$O(\sum n^2\omega)$ 的代码（但是只能拿 $30$ 分，经过一些卡常后可以拿 $50$ 分）：

```cpp line-numbers
// #pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <iostream>
#include <string.h>
#include <bits/stl_algo.h>
#define int long long
using namespace std;
const int N = 1000010;
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

using ull = unsigned long long;
using i128 = __int128;

int f[30][1010];
char s[N], t[N];
int fac[N], inv[N], ifac[N];
inline int binom(int a, int b)
{
    if (b < 0 || a < b)
        return 0;
    return fac[a] * ifac[b] % mod * ifac[a - b] % mod;
}
inline void add(int &x, int v) { x += v; if (x >= mod) x -= mod; }

signed main()
{
    // freopen("1.in", "r", stdin);
    // freopen("1.out", "w", stdout);
    // cin.tie(0)->sync_with_stdio(false);
    for (int i = 0; i < 2; ++i)
        fac[i] = inv[i] = ifac[i] = 1;
    for (int i = 2; i < N; ++i)
    {
        fac[i] = fac[i - 1] * i % mod;
        inv[i] = mod - inv[mod % i] * (mod / i) % mod;
        ifac[i] = ifac[i - 1] * inv[i] % mod;
    }
    scanf("%s", s + 1);
    int n = strlen(s + 1), buc[30] = {0};
    for (int i = 1; i <= n; ++i)
        t[i] = s[i];
    sort(t + 1, t + n + 1);
    int _ = unique(t + 1, t + n + 1) - t - 1;
    for (int i = 1; i <= n; ++i)
        s[i] = lower_bound(t + 1, t + _ + 1, s[i]) - t + 'a' - 1;
    for (int i = 1; i <= n; ++i)
        ++buc[s[i] - 'a' + 1];
    for (int m = 1; m <= n; ++m)
    {
        memset(f, 0, sizeof f);
        int now = 0;
        f[0][0] = 1;
        for (int i = 1; i <= _; ++i)
        {
            for (int j = 0; j <= m; ++j)
            {
                f[i][j] = 0;
                for (int k = 1; k <= min(j, buc[i]); ++k)
                    f[i][j] = (f[i][j] + binom(j, k) * binom(buc[i] - 1, k - 1) % mod * f[i - 1][j - k] % mod) % mod;
            }
            // cerr << "debug for " << i << ": ";
            // for (int j = 0; j <= min(now, m); ++j)
            //     cerr << f[i][j] << ' ';
            // cerr << '\n';
            now += buc[i];
        }
        int sum = 0;
        for (int i = 1; i <= m; ++i)
        {
            int coef = ((m - i) & 1) ? (mod - 1) : 1;
            sum = (sum + coef * binom(n - i, m - i) % mod * f[_][i]) % mod;
        }
        cout << sum << ' ';
    }
    cout << '\n';
    return 0;
}

```

$O(\sum n\omega\log n)$ 的代码：

```cpp line-numbers
// #pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <iostream>
#include <string.h>
#include <bits/stl_algo.h>
#define int long long
using namespace std;
const int N = 1000010;
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

using ull = unsigned long long;
using i128 = __int128;

int f[27][100010];
char s[N], t[N];
int fac[N], inv[N], ifac[N];
inline int binom(int a, int b)
{
    if (b < 0 || a < b)
        return 0;
    return fac[a] * ifac[b] % mod * ifac[a - b] % mod;
}
inline void add(int &x, int v) { x += v; if (x >= mod) x -= mod; }

namespace Poly
{
    const int g = 3;
    int rev[N];
    void ntt(int *a, int n, int mode)
    {
        int bit = 1;
        while ((1 << bit) < n)
            ++bit;
        for (int i = 0; i < n; ++i)
        {
            rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bit - 1));
            if (i < rev[i])
                swap(a[i], a[rev[i]]);
        }
        for (int l = 2; l <= n; l <<= 1)
        {
            int x = power(g, (mod - 1) / l, mod);
            if (mode == 1)
                x = inversion(x);
            for (int i = 0; i < n; i += l)
            {
                int v = 1;
                for (int j = 0; j < l / 2; ++j, v = v * x % mod)
                {
                    int v1 = a[i + j], v2 = a[i + j + l / 2] * v % mod;
                    a[i + j] = (v1 + v2) % mod, a[i + j + l / 2] = (v1 - v2 + mod) % mod;
                }
            }
        }
    }
    // calc convolution: c[i] = \sum\limits_{j=0}^i (a[j] * b[i - j])
    void convolution(int *a, int n, int *b, int m, int *c)
    {
        int tn = n, tm = m;
        n = n + m + 2;
        while (__builtin_popcount(n) > 1)
            ++n;
        for (int i = tn + 1; i <= n + 1; ++i)
            a[i] = 0;
        for (int i = tm + 1; i <= n + 1; ++i)
            b[i] = 0;
        ntt(a, n, 0), ntt(b, n, 0);
        for (int i = 0; i < n; ++i)
            c[i] = a[i] * b[i] % mod;
        ntt(c, n, 1);
        const int inv_n = inversion(n);
        for (int i = 0; i <= n + m; ++i)
            c[i] = c[i] * inv_n % mod;
    }
}

int A[N], B[N], C[N];

signed main()
{
    // freopen("1.in", "r", stdin);
    // freopen("1.out", "w", stdout);
    // cin.tie(0)->sync_with_stdio(false);
    for (int i = 0; i < 2; ++i)
        fac[i] = inv[i] = ifac[i] = 1;
    for (int i = 2; i < N; ++i)
    {
        fac[i] = fac[i - 1] * i % mod;
        inv[i] = mod - inv[mod % i] * (mod / i) % mod;
        ifac[i] = ifac[i - 1] * inv[i] % mod;
    }
    scanf("%s", s + 1);
    int n = strlen(s + 1), buc[30] = {0};
    for (int i = 1; i <= n; ++i)
        t[i] = s[i];
    sort(t + 1, t + n + 1);
    int _ = unique(t + 1, t + n + 1) - t - 1;
    for (int i = 1; i <= n; ++i)
        s[i] = lower_bound(t + 1, t + _ + 1, s[i]) - t + 'a' - 1;
    for (int i = 1; i <= n; ++i)
        ++buc[s[i] - 'a' + 1];
    memset(f, 0, sizeof f);
    int now = 0;
    f[0][0] = 1;
    for (int i = 1; i <= _; ++i)
    {
        A[0] = 0;
        for (int k = 1; k <= buc[i]; ++k)
            A[k] = ifac[k] * ifac[k - 1] % mod * ifac[buc[i] - k] % mod;
        for (int k = 0; k <= now; ++k)
            B[k] = ifac[k] * f[i - 1][k] % mod;
        Poly::convolution(A, buc[i], B, now, C);
        for (int j = 0; j <= now + buc[i]; ++j)
            f[i][j] = C[j] * fac[j] % mod * fac[buc[i] - 1] % mod;
        now += buc[i];
    }
    memset(A, 0, sizeof A);
    memset(B, 0, sizeof B);
    for (int i = 0; i <= n; ++i)
        A[i] = fac[n - i] * f[_][i] % mod;
    for (int i = 0; i <= n; ++i)
    {
        int coef = (i & 1) ? (mod - 1) : 1;
        B[i] = ifac[i] * coef % mod;
    }
    Poly::convolution(A, n, B, n, C);
    for (int m = 1; m <= n; ++m)
        cout << C[m] * ifac[n - m] % mod << ' ';
    cout << '\n';
    return 0;
}

```