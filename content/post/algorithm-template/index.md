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

## 拓扑排序(Topological Sort) (To do)

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
## 树状数组(Fenwick Tree)(To do)
> Reference: https://www.cnblogs.com/xenny/p/9739600.html
```cpp

```
## 线段树(Segment Tree)(To do)
> Reference: 
```cpp

```


> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
