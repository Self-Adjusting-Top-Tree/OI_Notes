和另一篇带代码的题解实现好像不太一样（）

把问题差分，答案为 $f(r)-f(l-1)$。问题在于求 $f(n)$ 的值：

+ $n<10^7$：直接筛出来即可。
+ $n\in[10^7,10^8)$：此时有 $d(n)=8$，考虑分类讨论：
  + $n=a^7$：此时合法的质数 $a$ 的数量很少，直接暴力枚举即可。
  + $n=a^3\times b$：此时 $a^3<10^8$，合法的质数 $a$ 的数量仍然比较少。$a$ 最小取 $2$，所以 $b$ 最大取到 $1.25\times 10^7$。在筛的时候处理出 $p_i$ 表示 $1\sim i$ 中质数的数量，然后枚举 $a$ 的过程中直接求出 $b$ 可以取值的区间，用 $p$ 数组算出区间内质数数量即可。
  + $n=a\times b\times c$：钦定 $a<b<c$，然后枚举质数，在枚举 $a,b,c$ 的过程中不枚举显然不可能合法的组合即可。
+ $n\in[10^8,10^9)$：此时有 $d(n)=9$，继续分类讨论：
  + $n=a^8$：合法的质数 $a$ 的数量还是很少，直接暴力枚举即可。
  + $n=a^2b^2$：钦定 $a<b$，此时 $a^2\le \sqrt n$ 即 $a\le\sqrt[4]n$，合法的质数 $a$ 的数量很少，然后在此基础上继续枚举质数 $b$，此时合法的质数 $b$ 数量也不多，直接暴力枚举即可。

实现的一些细节是在枚举 $n=a^3\times b$ 时，若对于一个给定的 $a$，$b$ 值的可行区间为 $[l,r]$，此时若有 $l\le a\le r$ 那么需要特判掉 $a=b$ 的情况，把答案减 $1$。

```cpp
// Author: 美丽好 rua 的大宋宋
// g++ complete_numbers.cpp -o a -g -std=c++2a -Ofast -Wall -Wextra

// Start coding at 10/14/2025 22:15:27
// Finish coding at 10/14/2025 23:47:36
// #pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/rope>
// #define int long long
using i64 = long long;
using i128 = __int128;
using ull = unsigned long long;
const int N = 25000010;
const int inf = 1e18;
const int mod = 924844033;

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

int lg10[N];
std::bitset<N> isp;
int pr[N / 8], idx, d_cnt[N], num[N], pre[N], zpre[N];
inline void sieve()
{
    lg10[0] = -1;
    for (int i = 1; i < N; ++i)
        lg10[i] = lg10[i / 10] + 1;
    isp[1] = d_cnt[1] = 1;
    for (int i = 2; i < N; ++i)
    {
        if (!isp[i])
            d_cnt[i] = 2, num[i] = 1, pr[++idx] = i;
        for (int j = 1; j <= idx && i * pr[j] < N; ++j)
        {
            isp[i * pr[j]] = 1;
            if (i % pr[j] == 0)
            {
                d_cnt[i * pr[j]] = d_cnt[i] / (num[i] + 1) * (num[i] + 2);
                num[i * pr[j]] = num[i] + 1;
                break;
            }
            d_cnt[i * pr[j]] = d_cnt[i] << 1;
            num[i * pr[j]] = 1;
        }
    }
    // std::cerr << "d: ";
    // for (int i = 1; i <= 40; ++i)
    //     std::cerr << d_cnt[i] << ' ';
    // std::cerr << '\n';
    for (int i = 1; i < N; ++i)
    {
        pre[i] = pre[i - 1];
        if (lg10[i] + 1 == d_cnt[i])
            ++pre[i];
        zpre[i] = zpre[i - 1] + (1 ^ isp[i]);
    }
}
inline int solve_a_b_c(int l, int r)
{
    int cnt = 0;
    for (int i = 1; i <= idx && pr[i] <= 1000; ++i)
        for (int j = i + 1; j <= idx && pr[i] * pr[j] <= 1000000; ++j)
        {
            int val = pr[i] * pr[j];
            int _l = l / val, _r = r / val;
            if (val * _l < l)
                ++_l;
            _l = std::max(_l, pr[j] + 1);
            if (_l > _r)
                continue;
            cnt += zpre[_r] - zpre[_l - 1];
        }
    return cnt;
}
inline int solve(int x)
{
    if (x <= 9999999)
        return pre[x];
    else
    {
        int cnt = pre[9999999];
        // d(x) = 8
        // 1. x = a^7
        // 2. x = a^3 * b
        // 3. x = a * b * c
        int lim = std::min(99999999, x);
        for (int i = 1; i * i * i * i * i * i * i <= lim; ++i)
            if (i * i * i * i * i * i * i >= 10000000 && !isp[i])
                ++cnt;
        for (int i = 1; i <= idx && pr[i] * pr[i] * pr[i] <= lim; ++i)
        {
            int l = 10000000 / (pr[i] * pr[i] * pr[i]), r = lim / (pr[i] * pr[i] * pr[i]);
            if (l * pr[i] * pr[i] * pr[i] < 10000000)
                ++l;
            if (l <= pr[i] && pr[i] <= r)
                --cnt;
            cnt += zpre[r] - zpre[l - 1];
        }
        cnt += solve_a_b_c(10000000, lim);
        if (x < 100000000)
            return cnt;
        // d(x) = 9
        // 1. x = a^8
        // 2. x = a^2 * b^2
        for (int i = 1; 1ll * i * i * i * i * i * i * i * i <= x; ++i)
            if (1ll * i * i * i * i * i * i * i * i >= 100000000 && !isp[i])
                ++cnt;
        for (int i = 1; i <= idx && pr[i] * pr[i] <= 32000; ++i)
            for (int j = i + 1; j <= idx && 1ll * pr[i] * pr[i] * pr[j] * pr[j] <= x; ++j)
                if (1ll * pr[i] * pr[i] * pr[j] * pr[j] >= 100000000)
                    ++cnt;
        return cnt;
    }
}

signed main()
{
    std::cin.tie(0)->sync_with_stdio(false);
    std::cout << std::fixed << std::setprecision(15);
    
    int T;
    std::cin >> T;

    sieve();

    // std::cerr << "debug: " << solve(9999999) << ' ' << solve(10000000) << '\n';
    while (T--)
    {
        int l, r;
        std::cin >> l >> r;
        if (l == 1e9)
        {
            std::cout << 0 << '\n';
            continue;
        }
        if (r == 1e9)
            --r;
        std::cout << solve(r) - solve(l - 1) << '\n';
    }
    return 0;
}

```