---
title: [CF2242F] Summer Vacation 胡乱写点
date: 2026-08-28 23:32:11
categories:
  - 草稿
tags:
  - 算法
  - 学习
---

越来越浮躁而无所事事了，尝试静下心来输出一些 Human Hallucination。

> 给定序列 $a_1\sim a_n$。$1\le a_i\le n\le 10^5$
>
> 对每个 $i$ 求解如下过程的最终答案：
>
> ``` python
> let x = a[i]
> for j in [i, n]
> 	if x < a[j] then
>         x <- x + a[j]
>     else
>         x <- x - a[j]
> return x
> ```

这是一个非常简单的模型，我们有很多操作空间。

最简单的手法是直接在序列上模拟，这样是 $\mathcal O(n^2)$ 的。人们开发出的序列操作手法并不多，套几个试一试基本就到头了。

为了提高信息的利用率，我们往往需要引入其它维度进行联动。

引入值域，进行简单的数值分析，发现第 1 种操作不会连续做 $\log n$ 次，但这个性质只能说明每个段很短，并不能说明段很少（恰恰相反）。

说得朦胧一点就是，并不能有效地把序列上的操作数控制在一个合理的范围内。



好在我们有别的手法，把值域看成一条线段，要做的无非是切分和平移。简单分析一下，线段数量每次最多 $\times 2$​，定期重构并把信息传递回序列就能够很好地控制。

代码上的细节是定期重构对这段时间内的信息非常不友好，好在这些东西暴力算起来特别方便。显式地建立起分块结构可能更有利于写代码。每 $B$ 下重构一次，$\mathcal O(\frac{n^2}B + \frac{n}B 2^B)$，取 $B = \log n$ 得 $\mathcal O(\frac{n^2}{\log n})$​。

能让这个东西跑过去也是种本事。不要用 vector，我被卡成狗了。

我在说啥我草。

``` cpp
#include <bits/stdc++.h>
#define pb emplace_back
#define fir first
#define sec second

using i64 = long long;
using pii = std::pair<int, int>;

constexpr int B = 18;

int main() {
    std::cin.tie(nullptr)->sync_with_stdio(false);
    int n;
    std::cin >> n;

    std::vector<int> a(n), f(2 * n), b(n, 0);
    for (auto& x : a) std::cin >> x;
    for (int l = 0; l < n; l += B) {
        int r = std::min(n, l + B);
        auto solve = [&](auto&& solve, int x, int L, int R, int delta) -> void {
            if (x >= r) {
                for (int i = L; i < R; ++i) {
                    f[i] = i + delta;
                }
                return;
            }
            if (R + delta <= a[x]) {
                solve(solve, x + 1, L, R, delta + a[x]);
            } else if (L + delta >= a[x]) {
                solve(solve, x + 1, L, R, delta - a[x]);
            } else {
                solve(solve, x + 1, L, a[x] - delta, delta + a[x]);
                solve(solve, x + 1, a[x] - delta, R, delta - a[x]);
            }
        };
        solve(solve, l, 0, 2 * n, 0);
        for (int i = 0; i < l; ++i) b[i] = f[b[i]];
        for (int i = l; i < r; ++i)
            for (int j = i; j < r; ++j)
                b[i] = b[i] >= a[j] ? b[i] - a[j] : b[i] + a[j];
    }
    std::reverse(b.begin(), b.end());
    for (auto& x : b) std::cout << x << ' ';
    std::cout << '\n';
    return 0;
}
```

