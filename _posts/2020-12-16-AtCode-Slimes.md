# AtCode Slimes

### Problem Statement

There are $N$ slimes lining up in a row. Initially, the $i$-th slime from the left has a size of aiai.

Taro is trying to combine all the slimes into a larger slime. He will perform the following operation repeatedly until there is only one slime:

- Choose two adjacent slimes, and combine them into a new slime. The new slime has a size of $x+y$, where $x$ and $y$ are the sizes of the slimes before combining them. Here, a cost of $x+y$ is incurred. The positional relationship of the slimes does not change while combining slimes.

Find the minimum possible total cost incurred.

$dp[l][r]$ 表示将[l, r]组合起来的最小cost

base case $dp[i][i] = 0$ 因为此时只有一个值，所以最小cost为0

$dp[l][r] = min(dp[l][k]+dp[k+1][r]) + a[l] + \dots + a[r]$

上面这个式子是因为，每次除了加上两个子range，还要加上整个range的和。例如[1, 2, 3, 4], 对于子range[1,2] = 3和[3,4] = 7， 整个range就为3 + 7 + 1 + 2 + 3 + 4 = 20

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

void solve() {
	ios::sync_with_stdio(false);
	int n;
	cin >> n;
	vector<ll> a(n+1);
	vector<ll> prefix(n+1, 0);
	for (int i = 1; i <= n; i++) {
		cin >> a[i];
		prefix[i] = prefix[i-1] + a[i];
	}
	// dp[l][r] 表示 minimum cost of combining the [l,r]
	vector<vector<ll>> dp(n+1, vector<ll>(n+1));
	for (int i = 1; i <= n; i++) {
		dp[i][i] = 0;
	}
	for (int len = 2; len <= n; len++) {
		for (int l = 1; l + len - 1 <= n; l++) {
			int r = l + len - 1;
			ll m = LLONG_MAX;
			for (int k = l; k < r; k++) {
				m = min(m, dp[l][k] + dp[k+1][r]);	
			}
			dp[l][r] = m + prefix[r] - prefix[l-1];
		}
	}
	cout << dp[1][n] << endl;
}

int main() {
	solve();
}
```

