# AtCoder L-Deque

https://atcoder.jp/contests/dp/tasks/dp_l

大意是给定一个序列$a = (a_1, a_2, \dots, a_N)$， 两个玩家不断进行如下操作，直到序列为空

​	移除开头或结尾的元素$x$, 该玩家会得到$x$分

记$X$和$Y$为Taro和Jiro的全部得分，找到最后$X-Y$的结果，每次Taro都会尝试最大$X-Y$，而Jiro尝试最小化$X-Y$

做法为区间dp

状态转移为:

```
记dp[l][r]为在l到r区间最后得到X-Y的结果
那么得到dp[l][r]有两种可能的路径
1. 移除头部的元素, 获得a[l]分 dp[l][r] = a[l] - dp[l-1][r]
dp[l-1][r]是上一步对手的得分差，那么计算这一步就要取相反数
2. 移除尾部的元素，获得a[r]分，dp[l][r] = a[r] - dp[l][r-1]
如果是1，那么
```

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
 
void solve() {
	ios::sync_with_stdio(false);
	int n;
	cin >> n;
	vector<int> a(n+1);
	for (int i = 1; i <= n; i++) {
		cin >> a[i];
	}
	vector<vector<ll>> dp(n+1, vector<ll>(n+1,0));
	for (int i = 1; i <= n; i++) {
		dp[i][i] = a[i];
	}
	for (int len = 2; len <= n; len++) {
		for (int l = 1; l + len - 1 <= n; l++) {
			int  r = l + len - 1;
			dp[l][r] = max(a[l] - dp[l+1][r], a[r] - dp[l][r-1]);
		}
	}
	cout << dp[1][n] << "\n";
}
 
int main() {
	solve();
}
```
