$nm\le 10^4$，所以考虑十分暴力的做法。

把所有在前门的人按照体力从小到大排序，让她们走到自己能走到的离后门最远的位置，然后把后门的人按照体力从小到大排序，判断是否每个人都能找到一个自己能走到的位置即可。

贪心正确性十分显然。直接模拟时间复杂度为 $O(n^2m^2)$，精细实现可以做到 $O(nm\log nm)$ 但是没有必要。

```cpp
// Author: 美丽好 rua 的大宋宋

#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <iostream>
#include <vector>
#include <bits/stl_algo.h>
#define int long long
using namespace std;
const int N = 300010;
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

int a[N], b[N];
inline int dist(int x1, int y1, int x2, int y2) { return abs(x1 - x2) + abs(y1 - y2); }

signed main()
{
    // freopen("1.in", "r", stdin);
    // freopen("1.out", "w", stdout);
    cin.tie(0)->sync_with_stdio(false);
    int n, m, k;
    cin >> n >> m >> k;
    for (int i = 1; i <= k; ++i)
        cin >> a[i];
    int kk;
    cin >> kk;
    for (int i = 1; i <= kk; ++i)
        cin >> b[i];
    sort(a + 1, a + k + 1);
    sort(b + 1, b + kk + 1);
    vector<vector<int>> vis;
    vis.resize(n + 2);
    for (int i = 0; i < n + 2; ++i)
        vis[i].resize(m + 2);
    for (int i = 1; i <= k; ++i)
    {
        int mx = -1, ix = -1, iy = -1;
        for (int j = 1; j <= n; ++j)
            for (int k = 1; k <= m; ++k)
                if (j + k <= a[i] && !vis[j][k])
                {
                    int new_dist = dist(j, k, 0, m + 1);
                    if (new_dist > mx)
                        mx = new_dist, ix = j, iy = k;
                }
        if (!~mx)
        {
            cout << "NO\n";
            return 0;
        }
        vis[ix][iy] = 1;
    }
    for (int i = 1; i <= kk; ++i)
    {
        int mx = -1, ix = -1, iy = -1;
        for (int j = 1; j <= n; ++j)
            for (int k = 1; k <= m; ++k)
                if (dist(j, k, 0, m + 1) <= b[i] && !vis[j][k])
                {
                    int new_dist = dist(j, k, 0, m + 1);
                    if (new_dist > mx)
                        mx = new_dist, ix = j, iy = k;
                }
        if (!~mx)
        {
            cout << "NO\n";
            return 0;
        }
        vis[ix][iy] = 1;
    }
    cout << "YES\n";
    return 0;
}

```