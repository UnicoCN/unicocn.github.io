---
title: 常见数据结构 & 算法の模板
description: Data Structures and Algorithm Template (C++)
slug: Data-Structures-and-Algorithm-Template
date: 2023-07-09 01:00:00+0000
image: cover.png
categories:
    - Study
tags:
    - C++
    - Algorithm
    - Template
---

## 快速排序(Quick Sort)

### My Version

```cpp
void Qsort(vector<int>& a, int l, int r) {
    int i = l, j = r;
    int mid = a[(i+j)>>1];
    do {
        while (a[i] < mid) ++i;
        while (a[j] > mid) --j;
        if (i <= j) {
            swap(a[i], a[j]);
            ++i;
            --j;
        }
    } while (i < j);
    if (l < j) Qsort(a,l,j);
    if (i < r) Qsort(a,i,r);
}
```

### ChatGPT version
```cpp
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;

    for (int j = low; j <= high - 1; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i+1], arr[high]);
    return i+1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

## 归并排序(Merge Sort)
### 递归版
```cpp
void MergeSort(vector<int>& a, int l, int r) {
    if (l == r) return;
    int mid = l + ((r - l) >> 1);
    MergeSort(a,l,mid);
    MergeSort(a,mid+1,r);
    int tmp[a.size()]; // 用vector太慢
    int i = l, j = mid+1, k = l;
    while (i <= mid && j <= r)
    tmp[k++] = (a[i] < a[j])?a[i++]:a[j++];
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++];
    for (int i = l; i <= r; ++i)
    a[i] = tmp[i];
    return;
}
```
### 迭代版
```cpp
void merge(vector<int>& arr, int l, int mid, int r) {
    int* temp = new int[r - l + 1];
    int i = l, j = mid + 1, k = 0;
    while (i <= mid && j <= r) {
        if (arr[i] < arr[j]) {
            temp[k++] = arr[i++];
        } else {
            temp[k++] = arr[j++];
        }
    }
    while (i <= mid) {
        temp[k++] = arr[i++];
    }
    while (j <= r) {
        temp[k++] = arr[j++];
    }
    for (int i = l, k = 0; i <= r; i++, k++) {
        arr[i] = temp[k];
    }
    delete[] temp;
}

void MergeSort(vector<int>& arr, int n) {
	for (int step = 1; step < n; step <<= 1) {
        for (int i = 0; i < n - step; i += (step << 1)) {
            int l = i, mid = i + step - 1, r = min(i + (step << 1) - 1, n - 1);
            merge(arr, l, mid, r);
        }
    }
}

```
### 多路归并
```cpp
class Solution {
public:

    ListNode* merge(ListNode* head1, ListNode* head2) {
        ListNode* dummy_head = new ListNode();
        ListNode* pre = dummy_head;

        ListNode* p1 = head1;
        ListNode* p2 = head2;
        while (p1 != nullptr && p2 != nullptr) {
            if (p1->val < p2->val) {
                pre->next = p1;
                pre = pre->next;
                p1 = p1->next;
            } else {
                pre->next = p2;
                pre = pre->next;
                p2 = p2->next;
            }
        }
        if (p1 != nullptr) pre->next = p1;
        if (p2 != nullptr) pre->next = p2;
        
        return dummy_head->next;
    }

    ListNode* mergeKLists(vector<ListNode*>& lists, int l, int r) {
        if (l > r) return nullptr; 
        if (l == r) return lists[l];
        return merge(mergeKLists(lists, l, (l + r) >> 1), mergeKLists(lists, ((l + r) >> 1) + 1, r));
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return mergeKLists(lists, 0, lists.size() - 1);
    }
};
```
## 并查集(Union-Find Set)
### 路径压缩
```cpp
int main() {

    int n;
    cin >> n;

    vector<int> f(n);
    for (int i = 0; i < n; ++i)
        f[i] = i;

    function<int(int)> Find = [&](int x) {
    return (f[x] == x)?x:f[x] = Find(f[x]);
    };

    function<void(int,int)> Union = [&](int x, int y) {
    f[Find(x)] = y;
    };

    for (int i = 0; i < n; ++i) {
        int a, b;
        cin >> a >> b;
        Union(a,b);
    }

    return 0;
}
```
### 路径压缩+按秩合并(To do)

## 拓扑排序(Topological Sort)
```cpp
void topo(int n, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(n);
    vector<int> ind(n, 0);
    for (auto &p : prerequisites) {
        int x = p[0], y = p[1];
        adj[y].push_back(x);
        ind[x] ++;
    }
    queue<int> Q;
    vector<bool> vis(n, false);
    for (int p = 0; p < n; ++p)
        if (ind[p] == 0) {
            Q.push(p);
            vis[p] = true;
        }

    while (!Q.empty()) {
        auto cur = Q.front();
        Q.pop();
        for (auto &nxt : adj[cur])
            if (!vis[nxt]) {
                if (--ind[nxt] == 0) {
                    Q.push(nxt);
                    vis[nxt] = true;
                }
            }
    }
}
```

## 欧拉筛法求素数
```cpp
vector<int> getPrimes(int n) {
    vector<int> primes;
    vector<bool> isPrime(n + 1, true);

    for (int i = 2; i <= n; ++i) {
        if (isPrime[i]) {
            primes.push_back(i);
        }

        for (int j = 0; j < primes.size() && i * primes[j] <= n; ++j) {
            isPrime[i * primes[j]] = false;
            if (i % primes[j] == 0) break;
        }
    }

    return primes;
}
```

## 模数逆元

### 费马小定理求逆元
```cpp
/* a^b模mod的快速幂 */
int64_t modpow(int64_t a, int64_t b, int64_t mod) {
    int64_t res = 1;
    while (b) {
        if (b & 1) res = res * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return res;
}
/* 求a模mod下的逆元 */
int64_t modinv(int64_t a, int64_t mod) {
    return modpow(a, mod - 2, mod);
}
```
### 线性逆元求法
#### 阶乘逆元(求0-n阶乘模p的逆元)
```cpp
void exPower( int b, int p, int & a, int & k ) { // 扩展欧几里得算法求逆元
    if( p == 0 ) {
        a = 1; k = 0;
        return;
    }
    exPower( p, b % p, k, a );
    k -= b / p * a;
    return;
}
int inv( int b, int p ) {
    int a, k;
    exPower( b, p, a, k );
    if( a < 0 ) a += p;
    return a;
}
void init( int n ) {
    Fact[ 0 ] = 1;
    for( int i = 1; i <= n; ++i ) Fact[ i ] = Fact[ i - 1 ] * i % Mod;
    INV[ n ] = inv( Fact[ n ], Mod ); // 先求出n!的逆元，也可以用费马小定理求
    for( int i = n - 1; i >= 0; --i ) INV[ i ] = INV[ i + 1 ] * ( i + 1 ) % Mod;
    return;
}
```
#### 连续数字逆元(求1-n模p的逆元)
```cpp

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);

    int n;
    int p;
    cin >> n >> p;
    vector<int> ans(n + 1);
    ans[1] = 1;
    for (int i = 2; i <= n; ++i)
        ans[i] = static_cast<int>(1LL * (p - p / i) * ans[p % i] % p);
    for (int i = 1; i <= n; ++i)
        cout << ans[i] << '\n';    

    return 0;
}
```

## 矩阵快速幂
```cpp
#include<bits/stdc++.h>

using namespace std;

using ll = long long;

constexpr ll MOD = 1e9 + 7;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);

    int n;
    ll k;
    cin >> n >> k;
    vector<vector<int>> a(n, vector<int>(n));
    vector<vector<int>> res(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            cin >> a[i][j];
    for (int i = 0; i < n; ++i)
        res[i][i] = 1;
    auto getMul = [&](vector<vector<int>> &a, vector<vector<int>> &b) {
        int tmp[n][n];
        memset(tmp, 0, sizeof(tmp));

        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                for (int k = 0; k < n; ++k)
                    tmp[i][j] = (int)(1LL * tmp[i][j] + 1LL * a[i][k] * b[k][j] % MOD) % MOD;

        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                a[i][j] = tmp[i][j];        
    };

    while (k) {
        if (k & 1) getMul(res, a);
        getMul(a, a);
        k >>= 1;
    }

    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            cout << res[i][j] << " \n"[j == n-1];

    return 0;
}
```
## KMP
### 下标从0开始
```cpp
void KMP(std::string& s, std::string& p) {
    int n = s.size(), m = p.size();
    std::vector<int> nxt(m+1);
    std::vector<int> f(n+1);
    nxt[1] = 0;
    int j = 0;
    for (int i = 2; i <= m; ++i) {
        while (j > 0 && p[j] != p[i-1])
            j = nxt[j];
        if (p[j] == p[i-1]) j++;
        nxt[i] = j;
    }

    j = 0;
    for (int i = 1; i <= n; ++i) {
        while ((j == m) || (j > 0 && p[j] != s[i-1]))
            j = nxt[j];
        if (p[j] == s[i-1]) j ++;
        f[i] = j;
        if (f[i] == m) std::cout << i - m + 1 << '\n';
    }
    for (int i = 1; i <= m; ++i)
        std::cout << nxt[i] << " \n"[i == m];
}
```
### 下标从1开始
```cpp
#include <iostream>
#include <vector>
#include <cstring>

char s[1000002];
char p[1000002]; // 下标从1开始+末尾终结符

void KMP(char* s, char* p) {
    int n = strlen(s+1);
    int m = strlen(p+1);

    std::vector<int> nxt(m+1);
    std::vector<int> f(n+1);

    nxt[1] = 0;
    int j = 0;
    for (int i = 2; i <= m; ++i) {
        while (j > 0 && p[j+1] != p[i])
            j = nxt[j];
        if (p[j+1] == p[i]) j++;
        nxt[i] = j;
    }
    j = 0;
    for (int i = 1; i <= n; ++i) {
        while ((j == m) || (j > 0 && p[j+1] != s[i]))
            j = nxt[j];
        if (p[j+1] == s[i]) j ++;
        f[i] = j;
        if (f[i] == m) std::cout << i - m + 1 << '\n';
    }
    for (int i = 1; i <= m; ++i)
        std::cout << nxt[i] << " \n"[i == m];
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);    

    scanf("%s", s+1);
    scanf("%s", p+1);

    KMP(s, p);
}
```
## 树状数组(Fenwick Tree)
> Reference: https://www.cnblogs.com/xenny/p/9739600.html
### 单点修改 & 区间求和
```cpp
/**
 * https://www.luogu.com.cn/problem/P3374
*/
#include<bits/stdc++.h>

using namespace std;

const int MAXN = 5e5 + 10;
int n, m;
long long Tree[MAXN];


int lowbit(int x) {
    return -x&x;
}

void add(int x, int v) {
    while (x <= n) {
        Tree[x] += v;
        x += lowbit(x);
    }
}

long long query(int x) {
    long long sum = 0;
    while (x > 0) {
        sum += Tree[x];
        x -= lowbit(x);
    }
    return sum;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);

    cin >> n >> m;

    vector<int> a(n);
    memset(Tree, 0, sizeof(Tree));

    for (int i = 1; i <= n; ++i) {
        int num;
        cin >> num;
        add(i, num);
    }

    for (int i = 0; i < m; ++i) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 1) {
            add(x, y);
        } else {
            cout << query(y) - query(x-1) << endl;
        }
    }

    return 0;
}
```
### 区间修改 & 区间求和(To Do)

### 单点修改 & 区间最值(To Do)

## 线段树(Segment Tree)
> Reference: https://www.bilibili.com/video/BV1qY411n7Qs
### 不带标记线段树
```cpp
/**
 * https://www.luogu.com.cn/problem/P3374
*/
#include<bits/stdc++.h>

using namespace std;

int n, m;
int a[500010], f[2000010];

inline void BuildTree(int k, int l, int r) {
    if (l == r) {
        f[k] = a[l];
        return;
    }
    int m = (l + r) >> 1;
    BuildTree(k + k, l, m);
    BuildTree(k + k + 1, m+1, r);
    f[k] = f[k + k] + f[k + k + 1];
    return;
}

inline void add(int k, int l, int r, int p, int v) {
    f[k] += v;
    if (l == r) return;
    int m = (l + r) >> 1;
    if (p <= m)
        add(k+k,l,m,p,v);
    else 
        add(k+k+1,m+1,r,p,v);
}

inline int query(int k, int l, int r, int h, int t) {
    if (l == h && r == t) return f[k];
    int m = (l + r) >> 1;
    if (t <= m)
        return query(k+k,l,m,h,t);
    else if (h > m)
        return query(k+k+1,m+1,r,h,t);
    else return query(k+k,l,m,h,m) + query(k+k+1,m+1,r,m+1,t);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);

    cin >> n >> m;
    for (int i = 1; i <= n; ++i)
        cin >> a[i];
    BuildTree(1,1,n);

    for (int i = 1; i <= m; ++i) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 1) {
            add(1,1,n,x,y);
        } else {
            cout << query(1,1,n,x,y) << endl;
        }
    }

    return 0;
}
```
### 带标记线段树
```cpp
/**
 * https://www.luogu.com.cn/problem/P3372
*/
#include<bits/stdc++.h>

using namespace std;

int n, m;
long long a[100010], f[400010], ex[400010];

inline void BuildTree(int k, int l, int r) {
    ex[k] = 0;
    if (l == r) {
        f[k] = a[l];
        return;
    }
    int m = (l + r) >> 1;
    BuildTree(k + k, l, m);
    BuildTree(k + k + 1, m+1, r);
    f[k] = f[k + k] + f[k + k + 1];
    return;
}
/* 单点修改 */
inline void add(int k, int l, int r, int p, int v) {
        f[k] += v;
    if (l == r) return;
    int m = (l + r) >> 1;
    if (p <= m)
        add(k+k,l,m,p,v);
    else 
        add(k+k+1,m+1,r,p,v);
}
/* 区间修改 */
inline void add2(int k, int l ,int r, int h, int t, int v) {
    if (l == h && r == t) {
        ex[k] += v;
        return;
    }
    f[k] += (t-h+1) * v;
    int m = (l + r) >> 1;
    if (t <= m)
        add2(k+k,l,m,h,t,v);
    else if (h > m)
        add2(k+k+1,m+1,r,h,t,v);
    else {
        add2(k+k,l,m,h,m,v);
        add2(k+k+1,m+1,r,m+1,t,v);
    }
}

inline long long query(int k, int l, int r, int h, int t) {
    if (l == h && r == t) return f[k];
    int m = (l + r) >> 1;
    if (t <= m)
        return query(k+k,l,m,h,t);
    else if (h > m)
        return query(k+k+1,m+1,r,h,t);
    else return query(k+k,l,m,h,m) + query(k+k+1,m+1,r,m+1,t);
}

inline long long query2(int k, int l, int r, int h, int t, long long p) {
    p += ex[k];
    if (l == h && r == t) return f[k] +  p * (r - l + 1);
    int m = (l + r) >> 1;
    if (t <= m)
        return query2(k+k,l,m,h,t,p);
    else if (h > m)
        return query2(k+k+1,m+1,r,h,t,p);
    else return query2(k+k,l,m,h,m,p) + query2(k+k+1,m+1,r,m+1,t,p);
}

int main() {

    scanf("%d%d",&n,&m);
    for (int i = 1; i <= n; ++i)
        scanf("%lld",&a[i]);
    BuildTree(1,1,n);

    for (int i = 1; i <= m; ++i) {
        int op;
        scanf("%d",&op);
        if (op == 1) {
            int x,  y,  v;
            scanf("%d%d%d",&x,&y,&v);
            add2(1,1,n,x,y,v);
        } else {
            int x, y;
            scanf("%d%d",&x,&y);
            printf("%lld\n", query2(1,1,n,x,y,0));
        }
    }

    return 0;
}
```



> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
