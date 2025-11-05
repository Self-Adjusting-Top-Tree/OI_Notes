我怎么不会这个啊（）

先考虑一个（~~并非~~）简单的情况：若参数序列 $q$ 满足 $q_1<q_2<\ldots<q_Q$ 即 $q$ 序列单调递增：

利用经典结论：对于任意正整数 $x,y$，下面两个情况中至少有一个成立：

+ $x\bmod y=x$
+ $x\bmod y\le \frac x2$

~~其实就是辗转相除的时间复杂度证明~~

于是考虑对于相邻两个参数 $q_i,q_{i+1}$，考虑特殊处理掉前面 $\lfloor\frac{q_{i+1}}{q_i}\rfloor$ 个完整的段，剩余长为 $q_{i+1}\bmod q_i$ 的段直接暴力递归处理，二分找出第一个会更新的位置，根据上面的分析可以知道每次操作长度至少减半，总时间复杂度不会超过对数量级。因此总时间复杂度为 $O(n\log^2n)$，满足题目给出的限制。

---

对于 $q$ 序列不单调递增的情况，容易发现对于两个相邻的位置 $q_i,q_{i+1}$ 若有 $q_i\ge q_{i+1}$ 那么前一个操作一定没用，用单调栈处理即可得到 $q$ 单调递增的情况。因此总时间复杂度仍为 $O(n\log^2n)$ 可以通过。

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
int n, q, top;
int a[N], rep[N], diff[N];

inline void dfs(int x, int y)
{
    if (!x)
        return;
    int l = 1, r = top, best = top + 1;
    while (l <= r)
    {
        int mid = l + r >> 1;
        if (a[mid] > x)
            best = mid, r = mid - 1;
        else
            l = mid + 1;
    }
    --best;
    if (!best)
        diff[1] += y, diff[x + 1] -= y;
    else
    {
        rep[best] += x / a[best] * y;
        dfs(x % a[best], y);
    }
}

signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    cout << fixed << setprecision(15);
    cin >> n >> q;
    a[top = 1] = n;
    for (int i = 1; i <= q; ++i)
    {
        int x;
        cin >> x;
        while (top && a[top] >= x)
            --top;
        a[++top] = x;
    }
    rep[top] = 1;
    for (int i = top - 1; i; --i)
    {
        rep[i] += a[i + 1] / a[i] * rep[i + 1];
        dfs(a[i + 1] % a[i], rep[i + 1]);
    }
    diff[1] += rep[1];
    diff[a[1] + 1] -= rep[1];
    for (int i = 1; i <= n; ++i)
        cout << (diff[i] += diff[i - 1]) << '\n';
    return 0;
}

```