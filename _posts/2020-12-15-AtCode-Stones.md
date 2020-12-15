# K-Stones

给定一个正整数的集合$A={a_1, a_2, \dots, a_n}$, Taro和Jiro按照如下规则进行游戏。

初始我们有一堆数目为K的石头，两个玩家依次进行如下操作：

​	选择A中的一个元素$x$，从石头堆中移除$x$个石头。

当一个玩家无法移除对应石头时，这个玩家就输了。

假定两个玩家都以最优的策略进行游戏。回答哪个玩家取得最后的胜利，

taro走先手。



``` 
定义dp(i)表示如果有i个石头，我们能赢吗？
dp(i) = 如果对于A中的任意一个元素x，如果移除x个石头后，dp(i-x)为false，那么dp(i)=true;

base case:
dp(0) = false 因为如果开始时没有石头，我们无法移除，则直接失败。
对于A中的任何一个元素x，dp(0 + x)都是成功的，因为我们一直采取最优策略，所以移除x后，对手无法进行移除，所以我们胜利。

```

```cpp
#include <bits/stdc++.h>
using namespace std;

void solve() {
	ios::sync_with_stdio(false);
	int n, k;
	cin >> n >> k;
	set<int> s;
	vector<bool> dp(k+1, false);
	for (int i = 1; i <= n; i++) {
		int a;
		cin >> a;
		s.insert(a);
	}
	dp[0] = false;;
	for (int i = 1; i <= k; i++) {
		bool flag = false;
		for (auto x: s) {
			if (i-x >= 0 && dp[i-x] == false) {
				flag = true;	
			}	
		}
		dp[i] = flag;
	}
	if (dp[k]) {
		cout << "First" << "\n";
	} else {
		cout << "Second" << "\n";
	}
}

int main() {
	solve();
}
```

