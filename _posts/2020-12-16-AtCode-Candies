# AtCode Candies

有N个小朋友，以1,2, ..., N编号

他们决定分享K颗糖，对于小朋友$i(1 \leq i \leq N)$, 小朋友i必须接受$[0, a_i]$颗糖，同时要求没有糖剩余。

找到分享糖的所有方法

这个题目如果直接dp会超时，如下图

```cpp
#include <bits/stdc++.h>
using namespace std;
 
const int mod = 1e9 + 7;
 
void solve() {
	ios::sync_with_stdio(false);
	int N, K;
	cin >> N >> K;
	vector<int> a(N+1); 
	for (int i = 1; i <= N; i++) {
		cin >> a[i];
	}
	// dp[i][j]表示i个人取j颗糖的方法数
	vector<vector<int>> dp(N+1, vector<int>(K+1));
	//base case 
	dp[0][0] = 1;
	for (int i = 1; i <= K; i++) {
		dp[0][i] = 0;
	}
	for (int i = 1; i <= N; i++) {
		dp[i][0] = 1;
	}
	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= K; j++) {
			for (int k = 0; k <= a[i]; k++) {
				if (j - k >= 0) 
					dp[i][j] = (dp[i][j] + dp[i-1][j-k]) % mod;
				else {
					break;
				}
			}
		}
	}
	cout << dp[N][K] << "\n";
 
}
 
int main() {
	solve();
}
```

```
分析：
转移关系dp[i][j]表示i个小朋友，分享j个糖的所有方法总数。
base case:
如果只有一个小朋友，那么dp[1][j] = 1 (0<=j<=a[i])
inductive steps:
对于i个小朋友，j颗糖，所有的方法总数应该为：
dp[i][j] += dp[i-1][j-k] (0<=k<=a[i] && j - k >= 0)
但是这种方法需要3重循环，时间复杂度太高，把上式展开，那么实际上
dp[i][j] = dp[i-1][j-a[i]] + dp[i-1][1] + ... + dp[i-1][j]
所以只用在第一层i循环内使用前缀和的思路，可以把3重循环降成两重
prefix[j] = sum(dp[i-1][0:j])
dp[i][j] = prefix[j] - prefix[j-a[i]-1] 如果j-a[i] > 0
dp[i][j] = prefix[j] 否则
```

```cpp
#include <bits/stdc++.h>
using namespace std;

const int mod = 1e9 + 7;

void solve() {
	ios::sync_with_stdio(false);
	int N, K;
	cin >> N >> K;
	vector<int> a(N+1); 
	for (int i = 1; i <= N; i++) {
		cin >> a[i];
	}
	// dp[i][j]表示i个人取j颗糖的方法数
	vector<vector<int>> dp(N+1, vector<int>(K+1));
	//base case 
	for (int i = 0; i <= a[1]; i++) {
		dp[1][i] = 1;
	}
	for (int i = 2; i <= N; i++) {
		vector<int> prefix(K+1);
		prefix[0] = dp[i-1][0];
		for (int j = 1; j <= K; j++) {
			prefix[j] = (prefix[j-1] + dp[i-1][j]) %mod;
		}
		for (int j = 0; j <= K; j++) {
			if (j - a[i] > 0) 
				dp[i][j] = (prefix[j] - prefix[j-a[i]-1] + mod) % mod;
			else {
				dp[i][j] = prefix[j];
			}
		}
	}
	cout << dp[N][K] << "\n";

}

int main() {
	solve();
}
```
