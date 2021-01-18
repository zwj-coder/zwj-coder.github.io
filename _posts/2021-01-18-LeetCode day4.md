# Day 4: LeetCode

## LeetCode 22

https://leetcode-cn.com/problems/generate-parentheses/

维护两个变量：可用的`(`数目m和剩余的`)`数目n

初始$(0, 0)$

对于状态$(m, n)$ 如果$n=0$, 所有`)`使用完了，添加到结果中。

```cpp
class Solution {
public:
    vector<string> res;
    string s;
    void dfs(int n, int l, int r) {
        if (r == n) {
            res.push_back(s);
        }
        if (l == 0) {
            s += '(';
            dfs(n, l+1, r);
            s.erase(s.end()-1);
        } else {
            if (l + r < n) {
                s += '(';
                dfs(n, l+1, r);
                s.erase(s.end()-1);
            }
            s += ')';
            dfs(n, l-1, r+1);
            s.erase(s.end()-1);
        } 
    }
    vector<string> generateParenthesis(int n) {
        dfs(n, 0, 0);
        return res;
    }
};
```

## LeetCode 23

https://leetcode-cn.com/problems/merge-k-sorted-lists/

递归很容易解决。

将链表数组分成两个部分，分别递归计算前一个部分和后一个部分的值，再调用两个链表的归并排序函数返回最终结果

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* merge(ListNode* left, ListNode* right) {
        if (left == nullptr) return right;
        if (right == nullptr) return left;
        if (left->val < right->val) {
            left->next = merge(left->next, right);
            return left;
        } else {
            right->next = merge(left, right->next);
            return right;
        }
    }
    ListNode* mergeK(vector<ListNode*>& lists, int l, int r) {
        if (l == r) return lists[l];
        int m = (l + r) / 2;
        ListNode* left = mergeK(lists, l, m);
        ListNode* right = mergeK(lists, m+1, r);
        return merge(left, right);
    }
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if (lists.size() == 0) return nullptr;
        return mergeK(lists, 0, lists.size()-1);
    }
};
```

## LeetCode 25

https://leetcode-cn.com/problems/reverse-nodes-in-k-group/

小技巧，在链表头加入一个dump头节点。

对需要反转的部分用头插法翻转

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode* dump = new ListNode();
        dump->next = head;
        head = dump;
        int n = 0;
        ListNode *p = head;
        while (p->next != nullptr) {
            p = p->next;
            n++;
        }
        p = head;
        while (n >= k) {
            ListNode* q = p;
            int t = k;
            while (t > 0) {
                q = q->next;
                t--;
            }
            auto *ne = q->next;
            auto *tmp = p->next;
            auto *f = p->next;
            while (f != ne) {
                ListNode *next = f->next;
                f->next = p->next;
                p->next = f;
                f = next;
            }
            tmp->next = ne;
            p = tmp;
            n -= k;
        }
        return head->next;
    }
};
```

## LeetCode 30

https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/

字符串hash加滑动窗口

这里hash不是常用的滚动hash。

```cpp
class Solution {
public:
    using ll = long long;
    map<ll, vector<int>> hashs;
    int m, p;
    vector<ll> p_pow;
    vector<ll> hs;
    void compute_hash(const string& s, vector<string>& words) {
        m = 1e9 + 7;
        p = 31;
        int n = s.size();
        p_pow.resize(n);
        p_pow[0] = 1;
        for (int i = 1; i < n; i++) {
            p_pow[i] = (p_pow[i-1] * p) % m;
        }
        for (int i = 0; i < words.size(); i++) {
            const string& pp = words[i];
            ll hp = 0;
            for (int i = 0; i < pp.size(); i++) {
                hp = (hp + (pp[i]-'a'+1) * p_pow[i]) % m;
            } 
            hashs[hp].push_back(i);
        }
        hs.resize(s.size()+1);
        for (int i = 0; i < s.size(); i++) {
            hs[i+1] = (hs[i] + (s[i]-'a'+1) * p_pow[i]) % m;
        }
    }
    vector<int> findSubstring(string s, vector<string>& words) {
        int n = words.size(); 
        if (n == 0) return {};
        int len = words[0].size();
        if (len > s.size()) {
            return {};
        }
        vector<int> res;
        compute_hash(s, words);
        int l = 0;
        int r = n * len;
        for (; r <= s.size(); l++, r++) {
            map<ll, vector<int>> tmp(hashs);
            int i = l;
            while (i < r) {
                ll pre = (hs[i+len] + m - hs[i]) % m;
                bool flag = false;
                auto it = tmp.begin();
                for (; it != tmp.end(); ++it) {
                    if (pre == it->first * p_pow[i] % m) {
                        flag = true;
                        break;
                    }
                }
                if (flag) {
                    it->second.pop_back();
                    if (it->second.empty()) {
                        tmp.erase(it);
                    }
                    i += len;
                } else {
                    break;
                }
            }
            if (i == r) {
                res.push_back(l);
            }
        }
        return res;
    }
};
```

## LeetCode 31

https://leetcode-cn.com/problems/next-permutation/

下一个较大的最小排列，在当前排列中将一个较小的元素和较大的元素交换，然后将交换后，较大元素位置后面的位置进行排序。为了保证这个排列尽量下，要保证较小的元素尽量靠右，较大的元素尽量的小，因为较小元素跟末尾构成了一个递减排列。

1. 从后往前找一个降序对(a, b), a < b， 这个时候a就是最靠右的较小元素,如果找不到说明这个排列已经是最大的了，下一个排列时最小排列。
2. 从后往前找到最小的元素满足b > a， 这个时候，这个b就是最小的较大元素。
3. 然后交换a，b两个元素，然后将[a+1, end]排列反转

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int i = nums.size() - 2;
        while (i >= 0 && nums[i] >= nums[i+1]) {
            i--;
        }
        if (i >= 0) {
            int j = nums.size() - 1;
            while (nums[j] <= nums[i]) {
                j--;
            }
            swap(nums[i], nums[j]);
        }
        reverse(nums.begin()+i+1, nums.end());
    }
};
```

## LeetCode 33

二分搜索

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int l = 0, r = nums.size()-1;
        if (l == r) {
            if (nums[l] == target) return 0;
            return -1;
        }
        while (l <= r) {
            int m = (l + r) / 2;
            if (nums[m] >= nums[l]) {
                if (target == nums[m]) {
                    return m;
                } else if (target < nums[m] && target >= nums[l]) {
                    r = m-1;
                } else {
                    l = m+1;
                }
            } else {
                if (target == nums[m]) {
                    return m;
                } else if (target > nums[m] && target <= nums[r]) {
                    l = m+1;
                } else {
                    r = m-1;
                }
            }
        }
        return -1;
    }
};
```

## LeetCode 34

二分法，找到

$max\ a[i] \leq target$

$min\ a[i]>=target$

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        ios::sync_with_stdio(false);
        if (nums.size() == 0) return {-1, -1};
        if (nums.size() == 1) {
            if (nums[0] == target) return {0, 0};
            return {-1, -1};
        }
        int l = -1, r = nums.size();
        int max_i, min_i;
        while (l + 1 < r) {
            int m = (l + r) / 2;
            if (nums[m] <= target) {
                l = m;
            } else {
                r = m;
            }
        }
        max_i = l;
        l = -1;
        r = nums.size();
        while (l + 1 < r) {
            int m = (l + r) / 2;
            if (nums[m] >= target) {
                r = m;
            } else {
                l = m;
            }
        }
        min_i = r;
        if (max_i >= 0 && max_i < nums.size() && nums[max_i] == target) {
            return {min_i, max_i};
        } else {
            return {-1, -1};
        }
    }
};
```
