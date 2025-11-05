一个非常简单的证法：有经典结论 $\gcd(F_n,F_m)=F_{\gcd(n,m)}$。因此若正整数 $n$ 存在 $\ge 3$ 的质因子（即 $n>4$ 且 $n$ 为合数）则 $F_n$ 一定不为斐波那契质数。若 $n>4$ 且 $n$ 为质数，则不存在 $m<n$ 使得 $\gcd(n,m)>2$，$F_n$ 一定为斐波那契质数。

剩下 $n\le 4$ 的情况，直接手算，可得 $n=2$ 或 $n=3$ 时 $F_n$ 为斐波那契质数，否则 $F_n$ 不为斐波那契质数。

```cpp
#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
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
    i64 ans = 1;
    while (b)
    {
        if (b & 1)
            ans = 1ll * ans * a % c;
        a = 1ll * a * a % c, b >>= 1;
    }
    return ans;
}
inline int inversion(int x) { return power(x, mod - 2, mod); }

inline int chk(int x)
{
    for (int i = 2; i * i <= x; ++i)
        if (x % i == 0)
            return 0;
    return 1;
}

using ld = long double;
ld f[N];

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    int n;
    vector<int> v;
    for (int i = 2; v.size() <= 22001; ++i)
        if (chk(i))
            v.emplace_back(i);
    f[1] = 1, f[2] = 1;
    for (int i = 3; i < N; ++i)
    {
        f[i] = f[i - 1] + f[i - 2];
        while (f[i] > 1e12)
            f[i] /= 10, f[i - 1] /= 10;
    }
    while (cin >> n)
    {
        if (n <= 2)
            cout << n + 1 << '\n';
        else
        {
            ld oo = f[v[n - 1]];
            while (oo > 1e9)
                oo /= 10;
            cout << (int)oo << '\n';
        }
    }
    return 0;
}
```