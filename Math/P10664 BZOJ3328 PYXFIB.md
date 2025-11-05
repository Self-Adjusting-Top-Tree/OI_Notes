【模板】单位根反演（~~其实不太模板~~）

单位根反演的关键结论是：$n[n\mid k]=\sum\limits_{i=0}^{n-1}{\omega_{n}^{ik}}$，其中 $\omega_{n}$ 是在模意义下的单位根。具体来说有 $\omega\equiv g^{\frac{p-1}{n}}\pmod p$，其中 $p$ 是模数，$g$ 是 $p$ 的一个原根。

然后考虑用上面的结论来推这个题：

$$
\begin{aligned}
 &\sum\limits_{i=0}^n[k\mid i]\binom niF_i\\
=&\frac1k\sum\limits_{i=0}^n(\sum\limits_{j=0}^{k-1}\omega^{ij})\binom niF_i\\
=&\frac1k\sum\limits_{j=0}^{k-1}\sum\limits_{i=0}^n\omega^{ij}\binom niF_i\\
=&\frac1k\sum\limits_{j=0}^{k-1}\sum\limits_{i=0}^n(\omega^j)^i\binom niF_i\\
\end{aligned}
$$

套路的把 $F$ 即斐波那契数列用矩阵的形式表示出来：

$$
\begin{aligned}
 &\frac1k\sum\limits_{j=0}^{k-1}\sum\limits_{i=0}^n(\omega^j)^i\binom niF_i\\
=&\frac 1k\sum\limits_{j=0}^{k-1}\sum\limits_{i=0}^n\binom ni(\begin{bmatrix}&1&1&\\&1&0&\\\end{bmatrix}\omega^j)^i\\
=&\frac 1k\sum\limits_{j=0}^{k-1}\sum\limits_{i=0}^n\binom ni(\textbf{I}^{n-i})(\begin{bmatrix}&1&1&\\&1&0&\\\end{bmatrix}\omega^j)^i\\
=&\frac 1k\sum\limits_{j=0}^{k-1}(\textbf{I}+\begin{bmatrix}&1&1&\\&1&0&\\\end{bmatrix}\omega^j)^n\\
\end{aligned}
$$

将 $\omega=g^{\frac{p-1}n}$ 带入原式，用矩阵快速幂优化上述式子的计算，除去找原根 $g$ 所需的时间复杂度，总时间复杂度为 $O(Tk\log n\omega^3)$，其中在这里 $\omega$ 定义为矩阵的大小，本题中取 $\omega=2$。

实现起来稍微有点卡常，一个比较好的卡常方法是展开矩阵乘法：

```cpp
// Author: 美丽好 rua 的大宋宋
// #pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#define int long long
using namespace std;
using i64 = long long;
const int N = 1000010;
const int inf = 1e18;
const int mod = 998244353;

inline int power(int a, int b, int c)
{
    int ans = 1;
    while (b)
    {
        if (b & 1)
            ans = 1ll * ans * a % c;
        a = 1ll * a * a % c, b >>= 1;
    }
    return ans;
}
inline int inversion(int x) { return power(x, mod - 2, mod); }

using i128 = __int128;
inline void chkmin(int &x, int y)
{
    if (x > y)
        x = y;
}
inline void chkmax(int &x, int y)
{
    if (x < y)
        x = y;
}

vector<int> fj;
inline int chk(int x, int p)
{
    for (int &i : fj)
        if (power(x, (p - 1) / i, p) == 1)
            return 0;
    return 1;
}

inline int find_g(int p)
{
    for (int i = 2;; ++i)
        if (chk(i, p))
            return i;
    return -1;
}

int n, k, p, g, omega;

struct Matrix
{
    int a[2][2];
    inline Matrix() { memset(a, 0, sizeof a); }
} ;
inline Matrix operator*(const Matrix &l, const Matrix &r)
{
    Matrix o;
    o.a[0][0] = (l.a[0][0] * r.a[0][0] + l.a[0][1] * r.a[1][0]) % p;
    o.a[0][1] = (l.a[0][0] * r.a[0][1] + l.a[0][1] * r.a[1][1]) % p;
    o.a[1][0] = (l.a[1][0] * r.a[0][0] + l.a[1][1] * r.a[1][0]) % p;
    o.a[1][1] = (l.a[1][0] * r.a[0][1] + l.a[1][1] * r.a[1][1]) % p;
    return o;
}
const inline Matrix I()
{
    Matrix o;
    o.a[0][0] = o.a[1][1] = 1;
    o.a[1][0] = o.a[0][1] = 0;
    return o;
}
inline Matrix power(Matrix a, int b)
{
    if (!b)
        return I();
    auto o = power(a, b >> 1);
    o = o * o;
    if (b & 1)
        o = o * a;
    return o;
}

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    int T;
    cin >> T;
    while (T--)
    {
        cin >> n >> k >> p;
        fj.clear();
        int _ = p - 1;
        for (int i = 2; i * i <= _; ++i)
            if (_ % i == 0)
            {
                fj.emplace_back(i);
                while (_ % i == 0)
                    _ /= i;
            }
        if (_ > 1)
            fj.emplace_back(_);
        g = find_g(p);
        omega = power(g, (p - 1) / k, p);
        const int inv_k = power(k % p, p - 2, p);
        const auto mat_i = I();
        Matrix mat_base;
        mat_base.a[0][0] = mat_base.a[0][1] = mat_base.a[1][0] = 1;
        mat_base.a[1][1] = 0;
        // cerr << "wa " << g << ' ' << omega << '\n';
        int omega_pwr = 1, sum = 0;
        for (int j = 0; j < k; ++j)
        {
            Matrix mat_inner;
            for (int x = 0; x < 2; ++x)
                for (int y = 0; y < 2; ++y)
                    mat_inner.a[x][y] = (mat_base.a[x][y] * omega_pwr + mat_i.a[x][y]) % p;
            Matrix mat_pwr = power(mat_inner, n);
            sum = (sum + mat_pwr.a[0][0]) % p;
            // cerr << j << ": " << mat_inner.a[0][0] << '\n';
            omega_pwr = omega_pwr * omega % p;
            // cerr << j << ": " << mat_pwr.a[0][0] << '\n';
        }
        sum = sum * inv_k % p;
        cout << sum << '\n';
    }
    return 0;
}

/*

1
10 9 37

*/

/*

33

*/

```