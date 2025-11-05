其实很简单（）

前置知识：FFT / NTT

### 001. P6800 【模板】Chirp Z-Transform

> 给定一个 $n$ 项多项式 $P(x)$ 以及 $c, m$，请计算 $P(c^0),P(c^1),\dots,P(c^{m-1})$。所有答案都对 $998244353$ 取模。

题解：

$P(c^i)$ 的值为 $\sum\limits_{j=0}^{n-1}a_jc^{ij}$，因此要求的东西其实就是：

$\forall i\in[0,m),F_i=\sum\limits_{j=0}^{n-1}a_jc^{ij}\bmod 998244353$

这是一个二维卷积的形式，考虑使用 Chirp-Z Transform 优化。有核心公式：

$$
ij=\binom{i+j}2-\binom i2-\binom j2
$$

用这个东西来替换上式的 $ij$，可得：

$$
\begin{aligned}
F_i&=\sum\limits_{j=0}^{n-1}a_jc^{ij}\\
   &=\sum\limits_{j=0}^{n-1}a^jc^{\binom{i+j}2-\binom i2-\binom j2}\\
   &=c^{-\binom i2}\sum\limits_{j=0}^{n-1}a^jc^{\binom{i+j}2-\binom j2}\\
   &={\large{c^{-\binom i2}}}\sum\limits_{j=0}^{n-1}\left(\left(a^jc^{-\binom j2}\right)\left(c^\binom{i+j}2\right)\right)
\end{aligned}
$$

设 $A_i=a_ic^{-\binom i2},B_i=c^{\binom i2}$，那么上述式子形如一个等和卷积，翻转 $B$ 得到 $B'$，设 $C=A\odot B$ 即 $A$ 和 $B$ 的卷积，即可求得后半部分的和式。而前面的 $c^{-\binom i2}$ 可以单 $\log$ 计算也可以线性递推。总时间复杂度为 $O(n\log n)$，瓶颈在于 NTT。

我的实现细节处理的不怎么好，但是也能过（）

```cpp line-numbers
namespace Luminescent
{
    struct DSU
    {
        int fa[N];
        inline DSU() { iota(fa, fa + N, 0); }
        inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
        inline int merge(int x, int y)
        {
            x = find(x), y = find(y);
            if (x != y)
                return fa[x] = y, 1;
            return 0;
        }
    };
    inline void add(int &x, int v) { x = (x + v) % mod; }
    inline void sub(int &x, int v) { x = (x - v + mod) % mod; }
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
    inline int varphi(int x)
    {
        int phi = 1;
        for (int i = 2; i * i <= x; ++i)
            if (x % i == 0)
            {
                phi *= (i - 1);
                x /= i;
                while (x % i == 0)
                    phi *= i, x /= i;
            }
        if (x > 1)
            phi *= (x - 1);
        return phi;
    }
} using namespace Luminescent;

namespace Poly
{
    const int g = 3;
    int rev[N * 8];
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
        // cerr << "n = " << n << '\n';
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

namespace Loyalty
{
    int a[N], A[N * 8], B[N * 8], C[N * 8];
    inline void main([[maybe_unused]] int _ca, [[maybe_unused]] int atc)
    {
        int n, c, m;
        cin >> n >> c >> m;
        for (int i = 0; i < n; ++i)
            cin >> a[i];
        for (int i = 0; i < max(n, m); ++i)
            A[i] = a[i] * inversion(power(c, i * (i - 1) / 2, mod)) % mod;
        for (int i = 0; i < max(n, m) + max(n, m); ++i)
            B[max(n, m) + max(n, m) - i - 1] = power(c, i * (i - 1) / 2, mod);
        // cerr << "???\n";
        Poly::convolution(A, max(n, m), B, max(n, m) + max(n, m), C);
        // cerr << "!!!\n";
        for (int j = 0; j < m; ++j)
        {
            int sum = 0;
            sum = C[max(n, m) + max(n, m) - j - 1];
            // for (int i = 0; i < n; ++i)
            //     sum = (sum + A[i] * B[n + n - i - j - 1] % mod) % mod;
            cout << sum * inversion(power(c, j * (j - 1) / 2, mod)) % mod << ' ';
        }
        cout << '\n';
    }
}

signed main()
{
    // freopen("b.in", "r", stdin);
    // freopen("b.oo", "w", stdout);
    cin.tie(0)->sync_with_stdio(false);
    cout << fixed << setprecision(15);
    int T = 1;
    // cin >> T;
    for (int ca = 1; ca <= T; ++ca)
        Loyalty::main(ca, T);
    return 0;
}
```

### 002. CBC024G

> 对于整数 $n$，有一张 $n$ 个点的竞赛图，图上每一条边 $(u,v)$（$u<v$）的方向有 $\frac12$ 的概率是由 $u$ 连向 $v$ 的，另外 $\frac12$ 的概率是由 $v$ 连向 $u$ 的。
>
> 现在给定 $m$，对于每个 $n=1\ldots m$，你需要求出 $n$ 个点的竞赛图上强连通分量的数目的期望，答案对 $998244353$ 取模。

~~这么会夹带私货~~

首先先思考，假设给定了 $n$ 的值，怎么快速计算答案。

容易证明若 $A,B$ 两个点集之间，不存在任何一条边 $u\to v$ 使得 $u\in B,v\in A$，那么 $A,B$ 之间就形成了一个新的 SCC。

考虑直接计数这样被分割的 SCC 数量。考虑枚举集合 $A$ 的大小 $i$，那么有 $\binom ni$ 种选择 $A$ 的方法。定义 $B$ 集合是 $A$ 在全集下的补集，则若不存在 $B$ 连向 $A$ 的边，则两个点集之间的全部 $2^{|A|\times|B|}$ 条边都必须由 $A$ 集合连向 $B$ 集合即已被定向，因此答案即为 $1+\sum\limits_{i=1}^{n-1}\binom ni2^{-i(n-i)}$，简单细节处理或使用 $O(\sqrt n)-O(1)$ 的光速幂均可做到 $O(n)$ 计算答案。

然后考虑优化到快速计算 $1\sim n$ 的答案。记竞赛图大小为 $n$ 时的期望为 $E_n$，则有：

$$E_n=1+\sum\limits_{i=1}^{n-1}\binom ni2^{-i(n-i)}=\sum\limits_{i=1}^n\binom ni2^{-i(n-i)}$$

给这个东西搞一个 EGF 然后只保留系数，可以得到：

$$\frac{E_n}{n!}=\sum\limits_{i=1}^n\frac{2^{-i(n-i)}}{i!(n-i)!}$$

记 $p\equiv 2^{-1}\pmod{998244353}$，则上式可以改写为：

$$\frac{E_n}{n!}=\sum\limits_{i=1}^n\frac{p^{i(n-i)}}{i!(n-i)!}$$

然后特殊处理掉边界上 $i=0$ 的部分：

$$\frac{E_n}{n!}+\frac1{n!}=\sum\limits_{i=0}^n\frac{p^{i(n-i)}}{i!(n-i)!}$$

这是一个二维卷积，可以用 Chirp-Z 变换和 NTT 将其优化至 $O(n\log n)$ 求解。

具体而言：考虑经典套路，有 $i(n-i)=\binom n2-\binom i2-\binom{n-i}2$，于是开始推柿子：

$$
\begin{aligned}
 &\sum\limits_{i=0}^n\frac{p^{i(n-i)}}{i!(n-i)!}\\
=&\sum\limits_{i=0}^n\frac{p^{\binom n2-\binom i2-\binom{n-i}2}}{i!(n-i)!}\\
=&\sum\limits_{i=0}^n\frac{p^{\binom n2}\times p^{-\binom i2}\times p^{-\binom{n-i}2}}{i!(n-i)!}\\
=&p^{\binom n2}\sum\limits_{i=0}^n\left(\frac{p^{-\binom i2}}{i!}\times\frac{p^{-\binom{n-i}2}}{(n-i)!}\right)\\
\end{aligned}
$$

设 $A_i=\dfrac{p^{-\binom i2}}{i!}$，则设 $C=A\odot A$ 即自己和自己的卷积，上述表达式的值可以记作 $p^{\binom n2}C_n$。因为 $998244353$ 是 NTT 友好模数，所以可以 $O(n\log n)$ 求解上述问题。

```cpp line-numbers
namespace Luminescent
{
    struct DSU
    {
        int fa[N];
        inline DSU() { iota(fa, fa + N, 0); }
        inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
        inline int merge(int x, int y)
        {
            x = find(x), y = find(y);
            if (x != y)
                return fa[x] = y, 1;
            return 0;
        }
    };
    inline void add(int &x, int v) { x = (x + v) % mod; }
    inline void sub(int &x, int v) { x = (x - v + mod) % mod; }
    inline int power(int a, long long b, int c)
    {
        int sum = 1;
        while (b)
        {
            if (b & 1)
                sum = 1ll * sum * a % c;
            a = 1ll * a * a % c, b >>= 1;
        }
        return sum;
    }
    inline int inversion(int x) { return power(x, mod - 2, mod); }
    inline int varphi(int x)
    {
        int phi = 1;
        for (int i = 2; i * i <= x; ++i)
            if (x % i == 0)
            {
                phi *= (i - 1);
                x /= i;
                while (x % i == 0)
                    phi *= i, x /= i;
            }
        if (x > 1)
            phi *= (x - 1);
        return phi;
    }
} using namespace Luminescent;

namespace Poly
{
    const int g = 3;
    int rev[N * 4];
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
        // cerr << "n = " << n << '\n';
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

namespace Loyalty
{
    int fac[N], inv[N], ifac[N];
    int A[N << 2], B[N << 2], C[N << 2];
    inline void main([[maybe_unused]] int _ca, [[maybe_unused]] int atc)
    {
        int m;
        cin >> m;
        for (int i = 0; i < 2; ++i)
            fac[i] = inv[i] = ifac[i] = 1;
        for (int i = 2; i < N; ++i)
        {
            fac[i] = fac[i - 1] * i % mod;
            inv[i] = mod - inv[mod % i] * (mod / i) % mod;
            ifac[i] = ifac[i - 1] * inv[i] % mod;
        }
        const int inv_2 = mod + 1 >> 1;
        for (int i = 0; i <= m; ++i)
            A[i] = B[i] = inversion(power(inv_2, i * (i - 1) / 2, mod)) * ifac[i] % mod;
        Poly::convolution(A, m, B, m, C);
        for (int i = 1; i <= m; ++i)
            cout << (C[i] * power(inv_2, i * (i - 1) / 2, mod) % mod * fac[i] - 1 + mod) % mod << '\n';
    }
}
```

### 003. P5293 [HNOI2019] 白兔之舞

前置知识：矩阵优化 dp，单位根反演

先考虑 $L$ 很小怎么做。容易想到枚举一共走了 $i$ 步，后面的方案数形如组合数乘以矩阵的形式。设转移矩阵为 $M$，则对于每个 $t\in[0,k)\cap\mathbb N$，答案可以表示为：

$$
\begin{aligned}
 &\sum\limits_{i\equiv t\pmod k}\binom Li(M_i)_{x,y}\\
=&\sum\limits_i\binom Li(M_i)_{x,y}[i\equiv t\bmod k]\\
=&\sum\limits_i\binom Li(M_i)_{x,y}[k\mid i-t]\\
=&\sum\limits_i[k\mid i-t]\binom Li(M_i)_{x,y}\\
=&\sum\limits_i\frac1k\sum\limits_{j=0}^{k-1}\omega_k^{j(i-t)}\binom Li(M_i)_{x,y}\\
=&\frac1k\sum\limits_{i}\sum\limits_{j=0}^{k-1}\omega_k^{j(i-t)}\binom Li(M_i)_{x,y}\\
=&\frac1k\sum\limits_i\sum\limits_{j=0}^{k-1}\omega_k^{ij}\times\omega_k^{-tj}\binom Li(M_i)_{x,y}\\
=&\frac1k\sum\limits_{j=0}^{k-1}\omega_k^{-tj}\sum\limits_{i}\omega_k^{ij}\binom Li(M_i)_{x,y}\\
=&\frac1k\sum\limits_{j=0}^{k-1}\omega_k^{-tj}\sum\limits_i\binom Li\textrm{I}^{L-i}\omega_k^{ij}(M_i)_{x,y}\\
=&\frac 1k\sum\limits_{j=0}^{k-1}\omega_k^{-tj}((\textrm{I}+\omega_k^j\times M)^L)_{x,y}
\end{aligned}
$$

把上述式子写成关于 $\omega_k^j$ 的函数，即设 $f(x)=\frac 1k\sum\limits_{j=0}^{k-1}x^j((\mathrm{I}+\omega_k^j\times M)^L)_{x,y}$，问题即为对每个 $t\in[0,k)\cap\mathbb N$ 求 $f(\omega_k^{-t})$ 的值。注意到 $f(x)$ 是一个多项式函数，而要求的值形如二维卷积的形式，因此可以使用 Chirp-Z Transform 优化。

因为这个题模数不固定，所以需要使用 MTT，这里使用了拆系数的 FFT 来计算答案（拆系数是因为卡精度）。因为这个题太卡精度了，所以这里使用了 `long double`。

```cpp line-numbers
namespace Luminescent
{
    const double pi = acos(-1);
    const ld pi_l = acosl(-1);
    struct DSU
    {
        int fa[N];
        inline DSU() { iota(fa, fa + N, 0); }
        inline int find(int x) { return x == fa[x] ? x : fa[x] = find(fa[x]); }
        inline int merge(int x, int y)
        {
            x = find(x), y = find(y);
            if (x != y)
                return fa[x] = y, 1;
            return 0;
        }
    };
    inline void add(int &x, int v) { x = (x + v) % mod; }
    inline void sub(int &x, int v) { x = (x - v + mod) % mod; }
    inline int power(int a, int b, int c)
    {
        int sum = 1;
        while (b)
        {
            if (b & 1)
                sum = 1ll * sum * a % c;
            a = 1ll * a * a % c, b >>= 1;
        }
        return sum;
    }
    inline int inversion(int x, int mod) { return power(x, mod - 2, mod); }
    inline int varphi(int x)
    {
        int phi = 1;
        for (int i = 2; i * i <= x; ++i)
            if (x % i == 0)
            {
                phi *= (i - 1);
                x /= i;
                while (x % i == 0)
                    phi *= i, x /= i;
            }
        if (x > 1)
            phi *= (x - 1);
        return phi;
    }
} using namespace Luminescent;

int n;

namespace LA_Matrix
{
    struct Matrix
    {
        int a[3][3];
        inline Matrix() { memset(a, 0, sizeof a); }
        inline Matrix(int val)
        {
            for (int i = 0; i < n; ++i)
                for (int j = 0; j < n; ++j)
                    a[i][j] = val;
        }
    };
    const inline Matrix I()
    {
        Matrix o;
        o.a[0][0] = o.a[1][1] = o.a[2][2] = 1;
        return o;
    }
    inline Matrix operator+(const Matrix &l, const Matrix &r)
    {
        Matrix res;
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                res.a[i][j] = (l.a[i][j] + r.a[i][j]) % mod;
        return res;
    }
    inline Matrix operator*(const Matrix &l, const Matrix &r)
    {
        Matrix res;
        for (int k = 0; k < n; ++k)
            for (int i = 0; i < n; ++i)
                for (int j = 0; j < n; ++j)
                    res.a[i][j] = (res.a[i][j] + l.a[i][k] * r.a[k][j] % mod) % mod;
        return res;
    }
    inline Matrix operator*(const Matrix &l, int r)
    {
        r %= mod;
        Matrix res;
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                res.a[i][j] = l.a[i][j] * r % mod;
        return res;
    }
    inline Matrix power(Matrix a, int b)
    {
        if (!b)
            return I();
        Matrix res = power(a, b >> 1);
        res = res * res;
        if (b & 1)
            res = res * a;
        return res;
    }
}

namespace Poly
{
    int rev[N];
    using cauto = complex<ld>;
    inline void fft(cauto *a, int n, int mode)
    {
        int bit = 1;
        while ((1 << bit) < n)
            ++bit;
        for (int i = 0; i < n; ++i)
            if (i < rev[i])
                swap(a[i], a[rev[i]]);
        for (int len = 2; len <= n; len <<= 1)
        {
            cauto x = cauto(cosl(pi_l * 2. / len), sinl(pi_l * 2. / len));
            if (mode == 1)
                x = cauto(cosl(pi_l * 2. / len), -sinl(pi_l * 2. / len));
            for (int i = 0; i < n; i += len)
            {
                cauto v = cauto(1, 0);
                for (int j = 0; j < len / 2; ++j, v = v * x)
                {
                    cauto v1 = a[i + j], v2 = a[i + j + len / 2] * v;
                    a[i + j] = v1 + v2, a[i + j + len / 2] = v1 - v2;
                }
            }
        }
    }
    cauto A1[N], B1[N], A2[N], B2[N], C1[N], C2[N], C3[N];
    inline void convolution(int *a, int *b, int len, int *c)
    {
        int n = len << 1;
        while (n & (n - 1))
            ++n;
        int bit = 1;
        while ((1 << bit) < n)
            ++bit;
        for (int i = 0; i < n; ++i)
            rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bit - 1));
        for (int i = 0; i < n; ++i)
            A1[i].real(a[i] >> 15), A2[i].real(a[i] & 32767);
        for (int i = 0; i < n; ++i)
            B1[i].real(b[i] >> 15), B2[i].real(b[i] & 32767);
        fft(A1, n, 1), fft(A2, n, 1), fft(B1, n, 1), fft(B2, n, 1);
        for (int i = 0; i < n; ++i)
            C1[i] = A2[i] * B2[i], C2[i] = A1[i] * B2[i] + A2[i] * B1[i], C3[i] = A1[i] * B1[i];
        fft(C1, n, -1), fft(C2, n, -1), fft(C3, n, -1);
        for (int i = 0; i < n; ++i)
            C1[i].real(C1[i].real() / n), C2[i].real(C2[i].real() / n), C3[i].real(C3[i].real() / n);
        for (int i = 0; i < n; ++i)
        {
            int v0 = (int)(C1[i].real() + 0.5) % mod;
            int v1 = (int)(C2[i].real() + 0.5) % mod;
            int v2 = (int)(C3[i].real() + 0.5) % mod;
            c[i] = (v2 * (1ll << 30) % mod + v1 * (1ll << 15) % mod + v0) % mod;
        }
    }
}

namespace Loyalty
{
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
        for (int i = 2; ; ++i)
            if (chk(i, p))
                return i;
        return -1;
    }
    int k, L, x, y;
    int m1[N], m2[N], m3[N];
    inline void main([[maybe_unused]] int _ca, [[maybe_unused]] int atc)
    {
        cin >> n >> k >> L >> x >> y >> mod;
        int _ = mod - 1;
        for (int i = 2; i * i <= _; ++i)
            if (_ % i == 0)
            {
                fj.emplace_back(i);
                while (_ % i == 0)
                    _ /= i;
            }
        if (_ > 1)
            fj.emplace_back(_);
        const int g = find_g(mod), omega = power(g, (mod - 1) / k, mod), inv_k = inversion(k, mod);
        // cerr << omega << '\n';
        LA_Matrix::Matrix M_map;
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                cin >> M_map.a[i][j];
        // cerr << "m1: ";
        for (int i = 0; i < k; ++i)
            m1[i] = (LA_Matrix::power(LA_Matrix::I() + M_map * power(omega, i, mod), L)).a[x - 1][y - 1] * power(omega, i * (i - 1) / 2, mod) % mod;//, cerr << m1[i] << ' ';
        // cerr << '\n';
        // return;
        for (int i = 0; i < k + k; ++i)
            m2[k + k - i - 1] = power(omega, mod - i * (i - 1) / 2 % (mod - 1) - 1, mod);//, cerr << m2[k + k - i - 1] << ' ';
        // return;
        // cerr << '\n';
        // for (int i = 0; i < k; ++i)
        //     for (int j = 0; j < k + k; ++j)
        //         m3[i + j] = (m3[i + j] + m1[i] * m2[j] % mod) % mod;
        Poly::convolution(m1, m2, k + k, m3);
        // cerr << "m3: ";
        // for (int i = 0; i < k + k; ++i)
        //     cerr << m3[i] << ' ';
        // cerr << '\n';
        for (int i = 0; i < k; ++i)
            cout << m3[k + k - i - 1] * power(omega, i * (i - 1) / 2, mod) % mod * inv_k % mod << '\n';
    }
}

signed main()
{
    // freopen("4.in", "r", stdin);
    // freopen("test.out1", "w", stdout);
    cin.tie(0)->sync_with_stdio(false);
    cout << fixed << setprecision(15);
    int T = 1;
    // cin >> T;
    for (int ca = 1; ca <= T; ++ca)
        Loyalty::main(ca, T);
    return 0;
}

```

### 004. CF1054H Epic Convolution

咕咕咕