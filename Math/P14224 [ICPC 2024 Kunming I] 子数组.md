唐题。

先考虑一个十分暴力的做法：对数组进行分治。对于一个分治区间 $[l,r]$：

+ 求出该区间内元素最大值 $mx$。
+ 找到数组 $X$ 表示该分治区间内所有值为 $mx$ 的位置。
+ 考虑计算 $mx$ 为最大值时，对答案的贡献。此时若一个区间 $[l',r']$ 满足 $[l',r']\subseteq[l,r]$ 且 $\sum\limits_{i=l'}^{r'}[a_i=mx]=k$，则该区间对 $res_k$ 做了 $1$ 的贡献。
+ 直接做时间复杂度过高，考虑只考虑值为 $mx$ 之间的位置。容易发现若设 $m=|X|$，则数组被划分为了 $m+1$ 段。设第 $i$ 段的长度为 $len_i$（不含值为 $mx$ 的端点），则该区间内 $mx$ 作为最大值对 $res_k$ 的贡献为：$\sum\limits_{i=1}^{m-k}(len_i+1)(len_{i+k}+1)$。
+ 剩下不包含 $mx$ 的区间，直接继续往下递归处理即可。

直接做时间复杂度显然超标。注意到对 $res$ 贡献的式子形如一个等差卷积，因此想到把 $len$ 翻转之后和自己做 NTT 卷积，此时时间复杂度为 $O(n\log^2n)$ 可以通过。

```cpp
#include <bits/stdc++.h>
#define int long long

using namespace std;

const int N = 1100010;
const int inf = 1e18;
const int mod = 998244353;
int f[N][20], lg[N];
int n, a[N], b[N];
int len[N], res[N];
vector<int> buc[N];
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
inline int inversion(int x)
{
	return power(x, mod - 2, mod);
}
inline int qry(int l, int r)
{
	int lgx = lg[r - l + 1];
	return max(f[l][lgx], f[r - (1 << lgx) + 1][lgx]);
}
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

int conv_len[N << 1], len2[N];
inline void cdq(int ql, int qr)
{
	if (ql > qr)
		return;
	int mx = qry(ql, qr);
//	cerr << "debug " << mx << ' ' << ql << ' ' << qr << '\n';
//	_sleep(20);
	assert(buc[mx].size());
	int l = 0, r = buc[mx].size() - 1, best = -1;
	while (l <= r)
	{
		int mid = l + r >> 1;
		if (buc[mx][mid] >= ql)
			best = mid, r = mid - 1;
		else
			l = mid + 1;
	}
	int pl = l;
	l = 0, r = buc[mx].size() - 1, best = -1;
	while (l <= r)
	{
		int mid = l + r >> 1;
		if (buc[mx][mid] <= qr)
			best = mid, l = mid + 1;
		else
			r = mid - 1;
	}
	int pr = r;
	int idx = 0;
//	memset(len, 0, sizeof len);
//	memset(len2, 0, sizeof len2);
	len[++idx] = buc[mx][pl] - ql + 1;
	for (int i = pl; i < pr; ++i)
		len[++idx] = buc[mx][i + 1] - buc[mx][i];
	len[++idx] = qr - buc[mx][pr] + 1;
	for (int i = 1; i <= idx; ++i)
		len2[i] = len[idx - i + 1];
//	memset(conv_len, 0, sizeof conv_len);
	Poly::convolution(len, idx, len2, idx, conv_len);
	// sum = idx - le + 1
	for (int le = 1; le < idx; ++le)
		res[le] = (res[le] + conv_len[idx - le + 1]) % mod;
//	for (int le = 1; le < idx; ++le)
//		for (int i = 1; i <= idx - le + 1; ++i)
//			res[le] = (res[le] + len[i] * len2[idx - i - le + 1] % mod) % mod;
//	for (int i = 1; i <= idx; ++i)
//		for (int j = i + 1; j <= idx; ++j)
//			res[j - i] = (res[j - i] + len[i] * len2[idx - j + 1] % mod) % mod;
//	cerr << "test\n";
	for (int i = 0; i <= idx + 1; ++i)
		len[i] = len2[i] = 0;
	for (int i = 0; i <= idx + idx + 1; ++i)
		conv_len[i] = 0;
	cdq(ql, buc[mx][pl] - 1);
	for (int i = pl; i < pr; ++i)
		cdq(buc[mx][i] + 1, buc[mx][i + 1] - 1);
	cdq(buc[mx][pr] + 1, qr);
}

signed main()
{
	cin.tie(0)->sync_with_stdio(false);
	int T;
	cin >> T;
	lg[0] = -1;
	for (int i = 1; i < N; ++i)
		lg[i] = lg[i >> 1] + 1;
	while (T--)
	{
		cin >> n;
		for (int i = 1; i <= n; ++i)
			cin >> a[i], f[i][0] = a[i], b[i] = a[i];
		sort(b + 1, b + n + 1);
		int _ = unique(b + 1, b + n + 1) - b - 1;
		for (int i = 1; i <= n; ++i)
			a[i] = lower_bound(b + 1, b + _ + 1, a[i]) - b, f[i][0] = a[i];
		for (int i = 1; i < 20; ++i)
			for (int j = 1; j <= n - (1 << i) + 1; ++j)
				f[j][i] = max(f[j][i - 1], f[j + (1 << (i - 1))][i - 1]);
		for (int i = 1; i <= n; ++i)
			res[i] = 0;
		for (int i = 1; i <= n; ++i)
			buc[i].clear();
		for (int i = 1; i <= n; ++i)
			buc[a[i]].emplace_back(i);
		cdq(1, n);
		int sum = 0;
		for (int i = 1; i <= n; ++i)
			sum = (sum + i * res[i] % mod * res[i] % mod) % mod;
		cout << sum << '\n';
	}
	return 0;
}
```