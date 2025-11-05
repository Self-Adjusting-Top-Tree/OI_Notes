$\text{lcm}$ 为 $n$ 的最大集合就是 $n$ 的所有正因子组成的集合。因此答案就是：$$\sum\limits_{i=2}^n\sum\limits_{j\mid i}j$$。

然后套路的推柿子：

$$
\begin
{aligned}
 &\sum\limits_{i=2}^n\sum\limits_{j\mid i}j\\
=&(\sum\limits_{j=1}^n\sum\limits_{i=1}^{\lfloor\frac ni\rfloor}j)-1\\
=&(\sum\limits_{j=1}^nj\sum\limits_{i=1}^{\lfloor\frac ni\rfloor}1)-1\\
=&\sum\limits_{j=1}^n(j\lfloor\frac ni\rfloor)-1\\
\end{aligned}
$$

这是一个整除分块的形式，可以做到 $O(Tn^{\frac12})$ 时间复杂度解决。

```cpp
signed main()
{
    cin.tie(0)->sync_with_stdio(false);
    int n;
    while (cin >> n, n)
    {
        int cnt = -1;
        for (int l = 1, r; l <= n; l = r + 1)
            r = n / (n / l), cnt += (n / l) * (r - l + 1) * (r + l) / 2;
        cout << cnt << '\n';
    }
    return 0;
}
```