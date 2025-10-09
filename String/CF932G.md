十分经典的题（同样也是 PAM 的入门题之一），很多内容参考了 [这篇文章](https://www.luogu.com.cn/article/aytabs1n)。

先考虑这样的一个问题（P14185 回文子串划分计数）：

> 给定一个长度为 $n$ 的字符串 $S$。问有多少种划分该字符串的方法，使得划分的每一个子串都是回文串。
>
> 定义长度为 $n$ 的字符串 $S$ 的划分形如：选择若干个区间 $[l_1,r_1],[l_2,r_2],\ldots,[l_k,r_k]$，满足 $l_1=1,r_k=n,\forall i\in[2,k]\cap\mathbb{N_+},l_i=r_{i-1}+1,\forall i\in[1,k]\cap\mathbb{N_+},l_i\le r_i$。此时字符串 $S$ 被划分为 $k$ 个子串，第 $i$ 个子串是 $S_{[l_i:r_i]}$。
>
> Data Range: $1\le n\le 10^6,\forall i\in[1,n]\cap\mathbb{N_+},S_i$ 是小写英文字母。

Sol：

前置知识：[回文自动机 PAM](https://www.luogu.com.cn/problem/P5496)。

一些定义：

> **回文串**：定义一个长度为 $n$ 的串 $S$ 是回文的，当且仅当 $\forall i\in[1,n],S_i=S_{n-i+1}$。
>
> **Border**：一个长度为 $m$ 的串 $T$ 是长度为 $n$ 的串 $S$ 的 Border，当且仅当 $S_{[1:m]}=S_{[n-m+1:n]}=T$。
>
> **最长 Border**：对于长度为 $n$ 的串 $S$，其最长 Border 定义为最大的正整数 $x<n$ 使得存在一个长度为 $x$ 的串 $T$ 是串 $S$ 的 Border。
>
> **周期**：$m$ 是一个长度为 $n$ 的串 $S$ 的周期，当且仅当 $n\ge m$ 且 $\forall i\in[1,n-m+1],S_i=S_{i+m}$。
>
> **最小正周期**：即最小的正整数 $m$ 使得 $m$ 是串 $S$ 的一个周期。

然后是一些引理（比较重要的已被加粗）：

+ 对于一个长度为 $n$ 的串 $S$，若 $S$ 是回文串，则若长度为 $m$ 的串 $T$ 是 $S$ 的后缀，$T$ 必然是 $S$ 的一个 Border。
+ 对于一个长度为 $n$ 的串 $S$，若长度为 $m$ 的串 $T$ 是 $S$ 的 Border，则 $|S|-m$ 必然是 $S$ 的一个周期。
+ **若有长度为 $n$ 的串 $S$ 和长度为 $m$ 的串 $T$，满足 $n\le m\le 2n$，则 $S$ 串在 $T$ 串上的所有匹配位置下标构成等差数列。**
+ **在上一条的基础上，若该等差数列的长度 $\ge 3$，设其公差为 $d$，则有串 $S$ 的最小正周期为 $d$。**
+ 对于一个长度为 $n$ 的串 $S$，其所有长度不小于 $\frac n2$ 的 Border 的不同长度构成一个等差数列。

考虑扩展 Border 的定义，原有的 Border 可以看作是串 $S$ 在和自己匹配，现在考虑重定义 Border，于是可以类比 Border，定义 $\text{Border}^+(S)$ 是长度为 $n$ 的串 $S$ 长度不小于 $\frac n2$ 的 Border 长度组成的集合。更形式化的说：

$$
\text{Border}^+(S)=\lbrace x\mid x\ge \frac n2,S_{[1:x]}=S_{[n-x+1:n]}\rbrace
$$

根据上面的定义，可知对于任意长度为 $n$ 的串 $S$，均有 $\text{Border}^+(S)$ 是一个等差数列。

然后容易得到关键结论：

+ **一个长度为 $n$ 的字符串 $S$，Border 可以划分为不超过 $O(\log n)$ 个等差数列。**

证明的话就是考虑倍增的去划 $\text{Border}^+$，根据前面被加粗的第三条引理可知倍增划分后每一段都形如一个等差数列。而倍增的性质就是划分为 $O(\log n)$ 的部分处理，因此上述关键结论得证。

然后考虑类比 PAM 上的 fail 指针，额外再维护 slink 指针。记回文树上点 $x$ 的等差值 $\text{diff}(x)=\text{len}(x)-\text{len}(\text{fail}(x))$，则 $\text{slink}(x)$ 对应的点为 fail 指针形成的 fail 树上，距离 $x$ 点最近的祖先结点 $y$ 满足 $\text{diff}(x)\neq\text{diff}(y)$。根据上面的结论，可以知道对于回文树上任意一个结点 $x$，都有从 $x$ 开始跳 slink 指针直到到达根结点为止，最多只会跳 $O(\log n)$ 次。

---

现在回到最初的问题。先考虑一个朴素的 dp 解法：

设 $f_i$ 表示 $S_{[1:i]}$ 这个前缀划分为若干回文子段的方案数，那么考虑枚举 $i$ 所在回文子串的开始位置 $j$，有转移：

+ $f_i=\sum\limits_{1\le j\le i}[s_{j:i}\text{ is a palindrome string}]f_{j-1}$

初始条件为 $f_0=1$。

直接暴力转移时间复杂度为 $O(n^3)$，用哈希等手段快速判断区间是否为回文串，时间复杂度可优化到 $O(n^2)$。

考虑到问题和回文串有关，因此想到 PAM。考虑一边建 PAM 一边利用 PAM 的性质和 fail，slink 指针快速维护出 $f$ 数组的值。考虑到前面提到的性质，回文串的回文后缀一定是该回文串的 Border，所以考虑从 $f_{i-\text{len}(\text{slink}(i))-\text{diff}(i)}$ 转移到 $f_i$。而每次在 PAM 中插入字符 $c$ 时，都从 PAM 中 $c$ 字符对应的结点 $x$ 开始向根跳 slink 指针，根据前面提到的性质，slink 指针跳到根结点的期望不超过 $O(\log n)$ 次，因此直接暴力模拟上述过程转移，时间复杂度为 $O(n\log n)$。

下面给出 P14185 的代码实现：

```c
#include <stdio.h>
#include <string.h>
#define N 1000010
#define mod 998244353

char s[N], t[N];
int idx, la;
typedef struct
{
    int ne[26], cnt, len, fail;
    int diff, slink;
} Node;
Node tree[N];
int f[N], g[N];

void init();
int getfail(int, int);
void ins(char, int);
long long max(long long, long long);

int main()
{
    scanf("%s", s + 1);
    int n = strlen(s + 1);
    init();
    f[0] = 1;
    for (int i = 1; i <= n; ++i)
    {
        ins(s[i], i);
        for (int j = la; j > 1; j = tree[j].slink)
        {
            g[j] = f[i - tree[tree[j].slink].len - tree[j].diff];
            if (tree[j].diff == tree[tree[j].fail].diff)
                g[j] = (g[j] + g[tree[j].fail]) % mod;
            f[i] = (f[i] + g[j]) % mod;
        }
    }
    printf("%d\n", f[n]);
    return 0;
}

long long max(long long a, long long b)
{
    return a > b ? a : b;
}

void init()
{
    idx = 1, la = 0, tree[1].len = -1, tree[0].fail = 1;
}

int getfail(int x, int i)
{
    while (s[i] != s[i - tree[x].len - 1])
        x = tree[x].fail;
    return x;
}

void ins(char o, int i)
{
    int x = getfail(la, i);
    if (!tree[x].ne[o - 'a'])
    {
        tree[++idx].len = tree[x].len + 2;
        tree[idx].fail = tree[getfail(tree[x].fail, i)].ne[o - 'a'];
        tree[x].ne[o - 'a'] = idx;
        tree[idx].diff = tree[idx].len - tree[tree[idx].fail].len;
        if (tree[idx].diff == tree[tree[idx].fail].diff)
            tree[idx].slink = tree[tree[idx].fail].slink;
        else
            tree[idx].slink = tree[idx].fail;
    }
    ++tree[la = tree[x].ne[o - 'a']].cnt;
}
```

---

然后回到本题。考虑做一个小小的转化：

> 对于给定的长度为 $n$ 的串 $S$，构造另一个长度为 $n$ 的串 $T$，使得串 $T$ 满足：$T_1=S_1,T_2=S_n,T_3=S_2,T_4=S_{n-1},T_5=S_3,\ldots$。

此时问题变为求 $T$ 的回文划分数，**且必须保证 $T$ 中划分的每一个回文串长度均为偶数。** 

在前面提到的问题的基础上修改：设 $f_i$ 表示 $S_{[1:i]}$ 这个前缀划分为若干**长度为偶数的**回文子段的方案数。这个修改是简单的，只需在原有 dp 的基础上，只更新所有满足 $2\mid i$ 的下标 $i$ 的 dp 值即可。时间复杂度仍为 $O(n\log n)$ 可以通过该题。

代码：

```c
#include <stdio.h>
#include <string.h>
#define N 1000010
#define mod 1000000007

char s[N], t[N];
int idx, la;
typedef struct
{
    int ne[26], cnt, len, fail;
    int diff, slink;
} Node;
Node tree[N];
int f[N], g[N];

void init();
int getfail(int, int);
void ins(char, int);
long long max(long long, long long);

int main()
{
    scanf("%s", s + 1);
    int n = strlen(s + 1), idx = 0;
    for (int l = 1, r = n; l <= r; ++l, --r)
    {
        t[++idx] = s[l];
        if (l != r)
            t[++idx] = s[r];
    }
    for (int i = 1; i <= n; ++i)
        s[i] = t[i];
    init();
    f[0] = 1;
    for (int i = 1; i <= n; ++i)
    {
        ins(s[i], i);
        for (int j = la; j > 1; j = tree[j].slink)
        {
            g[j] = f[i - tree[tree[j].slink].len - tree[j].diff];
            if (tree[j].diff == tree[tree[j].fail].diff)
                g[j] = (g[j] + g[tree[j].fail]) % mod;
            if (~i & 1)
                f[i] = (f[i] + g[j]) % mod;
        }
    }
    printf("%d\n", f[n]);
    return 0;
}

long long max(long long a, long long b)
{
    return a > b ? a : b;
}

void init()
{
    idx = 1, la = 0, tree[1].len = -1, tree[0].fail = 1;
}

int getfail(int x, int i)
{
    while (s[i] != s[i - tree[x].len - 1])
        x = tree[x].fail;
    return x;
}

void ins(char o, int i)
{
    int x = getfail(la, i);
    if (!tree[x].ne[o - 'a'])
    {
        tree[++idx].len = tree[x].len + 2;
        tree[idx].fail = tree[getfail(tree[x].fail, i)].ne[o - 'a'];
        tree[x].ne[o - 'a'] = idx;
        tree[idx].diff = tree[idx].len - tree[tree[idx].fail].len;
        if (tree[idx].diff == tree[tree[idx].fail].diff)
            tree[idx].slink = tree[tree[idx].fail].slink;
        else
            tree[idx].slink = tree[idx].fail;
    }
    ++tree[la = tree[x].ne[o - 'a']].cnt;
}
```