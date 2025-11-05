【模板】极角排序

把所有向量的起点平移到 $(0,0)$ 处，然后按 `atan` 值（即 C++ 中 cmath 库里的 `atan2` 函数）将所有向量的中点排序。此时所有向量按照夹角度数从小到大排序。

此时有经典结论：最后的答案必然存在一条直线 $l$，使得不存在向量和 $l$ 共线，且选取 $l$ 一侧的所有向量并统计答案可以得到最优解。

后面部分是简单的，直接破环为链倍长数组然后双指针扫描即可，总时间复杂度为 $O(n\log n)$ 瓶颈在于对向量做极角排序。

代码，写的有点丑（）

```cpp
// #pragma GCC optimze(3, "Ofast", "inline", "unroll-loops")
#include <bits/stdc++.h>
#define int long long
using namespace std;
using i64 = long long;

const int mod = 998244353;
const int G = 3;

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
using ld = long double;
inline void chkmax(ld &x, ld y)
{
    if (x < y)
        x = y;
}

int addmod(int a, int b)
{
    a += b;
    if (a >= mod)
        a -= mod;
    return a;
}
int submod(int a, int b)
{
    a -= b;
    if (a < 0)
        a += mod;
    return a;
}
int mulmod(int a, int b) { return (int)((i64)a * b % mod); }

const int N = 1000010;
struct Vector
{
    ld x, y;
    inline bool operator<(const Vector &r) const
    {
        return atan2l(y, x) < atan2l(r.y, r.x);
    }
} vec[N];

int n, idx;
const ld pi = acosl(-1);
inline ld luminescent(Vector x, Vector y, int l, int r)
{
    ld o = atan2l(y.y, y.x) - atan2l(x.y, x.x);
    if (l <= n && r > n)
        o += pi * 2;
    return o;
}

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    cout << fixed << setprecision(3);
    cin >> n;
    for (int i = 1; i <= n; ++i)
    {
        double x, y;
        cin >> x >> y;
        if (x || y)
            vec[++idx] = {x, y};
    }
    n = idx;
    sort(vec + 1, vec + n + 1);
    for (int i = 1; i <= n; ++i)
        vec[i + n] = vec[i];
    ld vx = 0, vy = 0;
    ld res = vec[1].x * vec[1].x + vec[1].y * vec[1].y;
    for (int l = 1, r = 1; l <= n + n; )
    {
        while (r <= n + n && luminescent(vec[l], vec[r], l, r) < pi + 1e-10)
            vx += vec[r].x, vy += vec[r].y, chkmax(res, vx * vx + vy * vy), ++r;
        // cerr << "debug " << l << ' ' << r << '\n';
        vx -= vec[l].x, vy -= vec[l].y, ++l, chkmax(res, vx * vx + vy * vy);
    }
    cout << (int)res << '\n';
    return 0;
}

```