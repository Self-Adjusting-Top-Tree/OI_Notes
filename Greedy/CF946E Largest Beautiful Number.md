比较简单的贪心。为了让构造的数 $t$ 的值尽量大，容易想到尽可能的让 $s,t$ 的公共前缀最大。而一个数 $x$ 是美丽数的充要条件是没有出现次数为奇数的数字。因此枚举这个最长的公共前缀位置 $k$，用前缀和判断 $\text{pre}(s,t)\ge k$ 是否可行。找到最大的满足条件的 $k$ 后，贪心的把出现次数为奇数的数从大到小添到 $t$ 的末尾部分，剩余部分全填 $9$ 即可。

总时间复杂度为 $O(n)$，可以轻松通过。

```cpp
#pragma GCC optimize(3, "Ofast", "inline", "unroll-loops")
#include <iostream>
#include <cstring>
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

char s[N];
int pre[N][10];

signed main()
{
    // freopen("1.in", "r", stdin);
    // freopen("1.out", "w", stdout);
    // cin.tie(0)->sync_with_stdio(false);
    int T;
    cin >> T;
    while (T--)
    {
        scanf("%s", s + 1);
        int n = strlen(s + 1), idx = n - 1;
        for (int i = 1; i <= n; ++i)
        {
            for (int j = 0; j < 10; ++j)
                pre[i][j] = pre[i - 1][j];
            pre[i][s[i] & 15] ^= 1;
        }
        for (int i = n; i; --i)
            for (int j = (s[i] & 15) - 1; ~j; --j)
                if (i != 1 || j)
                {
                    int cnt = 0;
                    for (int k = 0; k < 10; ++k)
                    {
                        if (j == k)
                            cnt += 1 ^ pre[i - 1][k];
                        else
                            cnt += pre[i - 1][k];
                    }
                    if (cnt <= n - i)
                    {
                        for (int k = 1; k < i; ++k)
                            cout << s[k];
                        cout << j;
                        for (int k = i + 1; k <= n - cnt; ++k)
                            cout << 9;
                        for (int k = 9; ~k; --k)
                        {
                            if (j == k)
                            {
                                if (!pre[i - 1][k])
                                    cout << k;
                            }
                            else
                            {
                                if (pre[i - 1][k])
                                    cout << k;
                            }
                        }
                        goto label;
                    }
                }
        while (idx & 1)
            --idx;
        for (int i = 1; i <= idx; ++i)
            cout << 9;
        label:;
        cout << '\n';
    }
    return 0;
}

```