# Day2 LeetCode

今日回学校happy，晚上回来只做了一题，调bug还挑了很久，确实是自己傻逼了，比较复杂一点的逻辑还是会漏掉一些细节，导致出错。

## LeetCode 460 LFU缓存

之前做过LRU缓存，当缓存满的时候，删除最久未使用的项，为新项腾空间。

LFU类似，但是更复杂一点，当缓存满的时候，删除最不经常使用的项，为新项腾空间。那么这里就要维护每个键值对的访问次数，如果访问次数最小的项不止一个，那么就根据最久未使用原则来删除对应项。

思路：

显然同LRU一样，用一个map维护key -> ListNode的映射，用于在$O(1)$时间内快速的定位值。

同样需要一个map维护key -> frequency的映射，用于记录每个键的访问次数。

同时对于每个frequency，其对应的key可能不知一个，所以使用一个双链表维护最久未使用原则，每次访问就把该项移动到首端，每次缓存满就将尾部的换出。

### 实现

首先就是数据结构:

定义一个双链表

```cpp
class Node {
public:
    Node *prev;
    Node *next;
    int val;
    int key;
    Node(int k, int v): key(k), val(v), prev(nullptr), next(nullptr) {}
};
```

```cpp
class LFUCache {
public:
	// key -> Node映射
    map<int, Node*> key_val;
    // key -> frequency映射
    map<int, int> key_freq;
    // key -> 一个双链表的头节点
    map<int, Node*> freq_time; 
    int size;
    int cap;
    LFUCache(int capacity): size(0), cap(capacity) {
    }
```

get操作比较简单，如果有对应key的话，要增加这个key的访问次数，同时在freq_time映射中，我们要将对应key的节点从原来的链表中删除，移动到增加后的freq_time的链表中

```cpp
int get(int key) {
       	if (cap == 0) return -1;
        if (size == 0) return -1;
        // 如果不存在
        if (!key_val.count(key)) return -1;
        // p为对应的节点
        auto *p = key_val[key];
        int old_freq = key_freq[key]++;
        int val = p->val;
        key_val.erase(key);
        remove(p);
        if (freq_time[old_freq]->next == freq_time[old_freq]) {
            freq_time.erase(old_freq);
        }
        // head 为新freq对应的头
        if (freq_time[key_freq[key]] == nullptr) {
            Node *head = new Node(INT_MAX, INT_MAX);
            head->next = head;
            head->prev = head;
            freq_time[key_freq[key]] = head;
        }
        auto *head = freq_time[key_freq[key]];
        Node *np = new Node(key, val);
        insert(head, np);
        key_val.insert({key, np});
        return val;
    }
```

set操作稍微复杂一点，如果包括这个key，那么操作跟get差不多，都需要将对应的节点从原来的链表中删除，然后移动到增加后的访问次数对应的双链表的头部，这里不同是，插入的value是put对应的value. 如果不包括这个key且还有空间，那么创建一个新节点，递增对应的访问次数，插入访问次数对应的双链表中。如果缓存已满，那么在访问次数最小的对应的双链表中删除尾部节点，然后再调用put(key, value)，由于这时有了空间，所以会增加成功

```cpp
void put(int key, int value) {
        if (cap == 0) return;
        if (key_val.count(key)) {
            auto *p = key_val[key];
            int old_freq = key_freq[key]++;           
            key_val.erase(key);
            remove(p);
            if (freq_time[old_freq]->next == freq_time[old_freq]) {
                freq_time.erase(old_freq);
            }
            if (freq_time[key_freq[key]] == nullptr) {
                Node *head = new Node(INT_MAX, INT_MAX);
                head->next = head;
                head->prev = head;
                freq_time[key_freq[key]] = head;
            }
            auto *head = freq_time[key_freq[key]];
            Node *np = new Node(key, value);
            insert(head, np);
            key_val.insert({key, np});
        } else {
            if (size < cap) {
                Node *np = new Node(key, value);
                key_freq[key]++;
                if (freq_time[key_freq[key]] == nullptr) {
                    Node *head = new Node(INT_MAX, INT_MAX);
                    head->next = head;
                    head->prev = head;
                    freq_time[key_freq[key]] = head;
                }
                Node *head = freq_time[key_freq[key]];
                insert(head, np);
                key_val.insert({key, np});
                size++;
            } else {
                auto pair = freq_time.begin();
                int freq = pair->first;
                Node *head = pair->second;
                key_val.erase(head->prev->key);
                key_freq.erase(head->prev->key);
                removeLast(head);
                if (head->next == head)
                    freq_time.erase(freq_time.begin());
                size--;
                put(key, value);
            }
        }
    }
```

全部代码：

```cpp
class Node {
public:
    Node *prev;
    Node *next;
    int val;
    int key;
    Node(int k, int v): key(k), val(v), prev(nullptr), next(nullptr) {}
};
class LFUCache {
public:
    map<int, Node*> key_val;
    map<int, int> key_freq;
    map<int, Node*> freq_time; 
    int size;
    int cap;
    LFUCache(int capacity): size(0), cap(capacity) {
    }
    
    void remove(Node *p) {
        p->prev->next = p->next;
        p->next->prev = p->prev;
        delete p;
        p = nullptr;        
    }

    void insert(Node *head, Node *p) {
        p->next = head->next;
        p->prev = head;
        head->next->prev = p;
        head->next = p;
    }

    void removeLast(Node *head) {
        assert(head->next != head);
        Node *last = head->prev;
        remove(last);
    }

    int get(int key) {
        if (cap == 0) return -1;
        if (size == 0) return -1;
        // 如果不存在
        if (!key_val.count(key)) return -1;
        // p为对应的节点
        auto *p = key_val[key];
        int old_freq = key_freq[key]++;
        int val = p->val;
        key_val.erase(key);
        remove(p);
        if (freq_time[old_freq]->next == freq_time[old_freq]) {
            freq_time.erase(old_freq);
        }
        // head 为新freq对应的头
        if (freq_time[key_freq[key]] == nullptr) {
            Node *head = new Node(INT_MAX, INT_MAX);
            head->next = head;
            head->prev = head;
            freq_time[key_freq[key]] = head;
        }
        auto *head = freq_time[key_freq[key]];
        Node *np = new Node(key, val);
        insert(head, np);
        key_val.insert({key, np});
        return val;
    }
    
    
};
```
