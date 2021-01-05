# Codeforces D.Good Substrings

https://codeforces.com/problemset/problem/271/D

题目如下：

给定一个字符串`s`，包括一些小写的英文字符。有一些字符是good,有一些字符是bad。

然后对于一个字符子串，如果在这个子串中，bad字符的数目小于等于`k`, 那么这个字符子串就是good，统计所有不同的字符子串的数目

例子：

```
s = "ababab"
t = "01000000000000000000000000"
1
```

`t`表示第i个字符如果是1，则`'a'+1`是好字符，其他的都是bad

首先可以想到的是我们可以计算所有子串的hash来统计一个字符串有多少个不同的子串。

但是题目只需要good的子串，由于我们不关心bad字符是哪些，只关心bad字符有多少个。所以把`s`字符串中的所有坏字符统一修改为`'{'`, 这是修改后的字符串为`"{b{b{b"`。然后我们使用rabin-karp字符串匹配算法，根据bad字符串`"{"`在修改后的字符串上进行匹配。返回一个所有匹配的index数组，对于这个例子，返回的数组就是`{0,2,4}`。

那么剩下的工作就很简单了，对于一个子串`s[l,...,r]` 如果`l到r`中的bad字符数小于等于k，则统计，反之则不统计

为了方便，将返回的数组进一步转换成一个prefix数组, `prefix[i]`表示前i个字符中有多少个坏字符。

那么子串`s[l,...,r]`中的坏字符数量就是`prefix[r+1] - prefix[l]`

代码如下：

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

vector<int> rabin_karp(string const& s, string const& t) {
    int ns = s.size(), nt = t.size();
    const int p = 31;
    const int m = 1e9 + 9;
    vector<ll> p_pow(max(ns, nt));
    p_pow[0] = 1;
    for (int i = 1; i < (int)p_pow.size(); i++)
        p_pow[i] = (p_pow[i-1] * p) % m;

    vector<long long> h(nt + 1, 0);
    for (int i = 0; i < nt; i++)
        h[i+1] = (h[i] + (t[i] - 'a' + 1) * p_pow[i]) % m;
    long long h_s = 0;
    for (int i = 0; i < ns; i++)
        h_s = (h_s + (s[i] - 'a' + 1) * p_pow[i]) % m;

    vector<int> occurences;
    for (int i = 0; i + ns - 1 < nt; i++) {
        long long cur_h = (h[i+ns] + m - h[i]) % m;
        if (cur_h == h_s * p_pow[i] % m)
            occurences.push_back(i);
    }
    return occurences;
}

void solve() {
    ios::sync_with_stdio(false);
    string s, t;
    cin >> s >> t;
    int k;
    cin >> k;
    string tmp(s);
    set<char> good_ones;
    for (int i = 0; i < t.size(); i++) {
        if (t[i] == '1') {
            good_ones.insert('a' + i);
        }
    }
    for (char& c: tmp) {
        if (!good_ones.count(c)) {
            c = 'a' + 26;
        }
    }
    int n = s.size();
    const int p = 31;
    const int m = 1e9 + 9;
    vector<ll> p_pow(n);
    p_pow[0] = 1;
    for (int i = 1; i < (int)p_pow.size(); i++)
        p_pow[i] = (p_pow[i-1] * p) % m;

    vector<long long> h(n + 1, 0);
    for (int i = 0; i < n; i++)
        h[i+1] = (h[i] + (s[i] - 'a' + 1) * p_pow[i]) % m;

    string bad = "{";
    vector<int> occur = rabin_karp(bad, tmp);
    vector<int> pos(n);
    for (int i = 0; i < occur.size(); i++) {
        pos[occur[i]]++;
    }
    vector<int> prefix(n+1, 0);
    for (int i = 1; i <= n; i++) {
        prefix[i] = prefix[i-1] + pos[i-1];
    }
    int cnt = 0;
    for (int l = 1; l <= n; l++) {
        set<ll> hs;
        for (int i = 0; i <= n-l; i++) {
            int a = i+1, b = i + l;
            if (prefix[b] - prefix[a-1] <= k) {
                ll cur_p = (h[i+l] + m - h[i]) % m;
                cur_p = (cur_p * p_pow[n-i-1]) % m;
                hs.insert(cur_p);
            }
        }
        cnt += hs.size();
    }
    cout << cnt << "\n";

}

int main() {
    solve();
    return 0;
}

```
