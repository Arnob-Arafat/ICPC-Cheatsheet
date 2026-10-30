# ICPC C++ Cheatsheet

Fast reference for contest day: boilerplate, data types, STL, and common patterns.

## Table of Contents

1. [Boilerplate](#1-boilerplate)
2. [Primitive Data Types](#2-primitive-data-types)
3. [Strings](#3-strings)
4. [STL Containers](#4-stl-containers)
5. [Custom Sorting and Comparators](#5-custom-sorting-and-comparators)
6. [STL Algorithms](#6-stl-algorithms)
7. [Bit Manipulation](#7-bit-manipulation)
8. [Math and Number Theory](#8-math-and-number-theory-snippets)
9. [Graph and DS Templates](#9-graph-and-ds-templates)
10. [Which Structure Should I Use?](#10-which-structure-should-i-use)
11. [Common Pitfalls](#11-common-pitfalls)
12. [Contest-Day Strategy](#12-contest-day-strategy)

---

## 1. Boilerplate

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef unsigned long long ull;
typedef long double ld;
typedef pair<int,int> pii;
typedef pair<ll,ll> pll;
typedef vector<int> vi;
typedef vector<ll> vll;

#define pb push_back
#define all(x) (x).begin(), (x).end()
#define sz(x) (int)(x).size()
#define F first
#define S second

const int MOD = 1e9 + 7;      // or 998244353
const int INF = 1e9;
const ll LINF = 4e18;
const int N = 2e5 + 5;

void solve() {
    // code here
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);
    int t = 1;
    // cin >> t;
    while (t--) solve();
    return 0;
}
```

**Notes**

- Use `'\n'` instead of `endl` (endl flushes and is slow).
- Do not mix `scanf/printf` with `cin/cout` after `sync_with_stdio(false)`.
- Check the problem statement: some sets use stdin/stdout, some use files. For files: `freopen("in.txt","r",stdin);`
- Reset global arrays between test cases.

## 2. Primitive Data Types

| Type | Size | Range | Use when |
|---|---|---|---|
| `bool` | 1 B | true/false | flags, visited arrays |
| `char` | 1 B | -128..127 | single characters, ASCII math (`c - 'a'`) |
| `int` | 4 B | about ±2.147e9 (2^31-1) | default for counts, indices, values under 2e9 |
| `unsigned int` | 4 B | 0..4.29e9 | bit tricks, hashing |
| `long long` | 8 B | about ±9.22e18 | sums/products, n up to 1e9 squared, modular multiplication |
| `unsigned long long` | 8 B | 0..1.8e19 | hashing, overflow-wrap needed |
| `__int128` | 16 B | about ±1.7e38 | products of two `long long`, big modmul (no direct cin/cout) |
| `float` | 4 B | ~7 digits | almost never; avoid |
| `double` | 8 B | ~15 digits | geometry, probabilities, general real numbers |
| `long double` | 12/16 B | ~18 digits | when `double` precision fails |

**Rules of thumb**

- If the answer or any intermediate value may exceed 2e9, use `ll`.
- `1 << 40` overflows. Write `1LL << 40`.
- `int * int` overflows before assignment to `ll`. Write `(ll)a * b`.
- Integer division truncates toward zero: `-7 / 2 = -3`, and `-7 % 2 = -1`. Fix modulo: `((a % m) + m) % m`.
- Compare doubles with epsilon: `fabs(a - b) < 1e-9`.
- Print doubles: `cout << fixed << setprecision(9) << x;`
- Print `__int128` by converting to a string manually.

### Limits quick table

| Constraint n | Typical complexity needed |
|---|---|
| n <= 10 | O(n!) |
| n <= 20 | O(2^n * n) bitmask |
| n <= 500 | O(n^3) |
| n <= 5000 | O(n^2) |
| n <= 1e5 to 2e5 | O(n log n) |
| n <= 1e6 to 1e7 | O(n), or O(n log n) with a small constant |
| n up to 1e18 | O(log n) or O(1) |

Roughly 1e8 simple operations per second.

## 3. Strings

```cpp
string s; cin >> s;              // read a word
getline(cin, s);                 // read a whole line (use cin.ignore() after cin >> n)
s.size(); s.empty();
s += "abc"; s.push_back('x'); s.pop_back();
s.substr(pos, len);              // O(len)
s.find("ab");                    // index or string::npos
s.rfind('a');
s.insert(pos, "xyz"); s.erase(pos, len);
reverse(all(s));
sort(all(s));
stoi(s); stoll(s); stod(s);      // string to number
to_string(123);                  // number to string
s == t; s < t;                   // lexicographic compare
isdigit(c); isalpha(c); islower(c); isupper(c);
toupper(c); tolower(c);
int idx = c - 'a';               // letter to 0..25
```

**stringstream (split / parse)**

```cpp
stringstream ss(line);
string word;
while (ss >> word) { /* each token */ }
```

Use `string` for text. For many frequency counts of lowercase letters, use `int cnt[26]`.

## 4. STL Containers

| Container | Behavior | Insert / erase / find | Use when |
|---|---|---|---|
| `array<T,N>` | fixed size | O(1) access | fixed-size small arrays |
| `vector<T>` | dynamic array | push_back O(1) amortized, access O(1), insert middle O(n) | default container |
| `deque<T>` | double-ended | push/pop both ends O(1), access O(1) | sliding window, 0-1 BFS |
| `list<T>` | doubly linked | O(1) insert at known iterator | rare; splice operations |
| `stack<T>` | LIFO | O(1) | bracket matching, DFS, monotonic stack |
| `queue<T>` | FIFO | O(1) | BFS |
| `priority_queue<T>` | max-heap | push/pop O(log n), top O(1) | Dijkstra, greedy, k-th largest |
| `set<T>` | sorted unique | O(log n) | ordered unique values, lower_bound |
| `multiset<T>` | sorted, duplicates | O(log n) | sorted with duplicates, sliding-window median |
| `map<K,V>` | sorted key-value | O(log n) | ordered counting, coordinate lookups |
| `multimap<K,V>` | sorted, duplicate keys | O(log n) | rare |
| `unordered_set` | hash | O(1) average | fast membership tests |
| `unordered_map` | hash | O(1) average | fast counting when order does not matter |
| `bitset<N>` | bits | O(N/64) per bulk op | subset-sum DP, sieve, bit-parallel tricks |
| `pair<A,B>` | two values | compares lexicographically | edges, coordinates |
| `tuple<...>` | n values | `get<i>(t)` | triple data, `tie` for unpacking |

### vector

```cpp
vector<int> v(n);              // n zeros
vector<int> v(n, -1);          // n copies of -1
vector<vector<int>> g(n);      // adjacency list
vector<vector<int>> dp(n, vector<int>(m, 0));  // 2D
v.push_back(x); v.pop_back(); v.back(); v.front();
v.size(); v.empty(); v.clear(); v.resize(k);
v.insert(v.begin() + i, x); v.erase(v.begin() + i);
v.erase(unique(all(v)), v.end());   // dedupe (sort first)
for (auto &x : v) x++;         // use & to modify
for (auto [a, b] : vp) {}      // structured binding (C++17)
```

### stack / queue / priority_queue

```cpp
stack<int> st;   st.push(x); st.top(); st.pop();
queue<int> q;    q.push(x);  q.front(); q.pop();
priority_queue<int> mx;                             // max-heap
priority_queue<int, vector<int>, greater<int>> mn;  // min-heap
priority_queue<pair<ll,int>, vector<pair<ll,int>>, greater<>> pq; // Dijkstra
```

### set / multiset

```cpp
set<int> s;
s.insert(x); s.erase(x); s.count(x); s.find(x);
s.lower_bound(x);   // first element >= x
s.upper_bound(x);   // first element > x
*s.begin(); *s.rbegin();
auto it = s.lower_bound(x);
if (it != s.begin()) { --it; /* largest element < x */ }

multiset<int> ms;
ms.erase(ms.find(x));   // erase ONE copy (ms.erase(x) removes ALL copies)
```

### map / unordered_map

```cpp
map<string,int> m;
m[key]++;                       // creates key with 0 if missing
m.count(key); m.find(key);
m.erase(key);
for (auto &[k, v] : m) {}       // sorted order
m.lower_bound(k);
m.begin()->first; m.rbegin()->first;
```

`unordered_map` can be slow or hacked with bad hashing. A safe pattern:

```cpp
struct custom_hash {
    size_t operator()(ull x) const {
        x += chrono::steady_clock::now().time_since_epoch().count();
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
};
unordered_map<ll,int,custom_hash> um;
```

### pair / tuple

```cpp
pair<int,int> p = {1, 2};  p.first; p.second;
make_pair(a, b);
tuple<int,int,int> t = {1, 2, 3};
auto [a, b, c] = t;
int x, y; tie(x, y) = make_pair(5, 6);
```

### bitset

```cpp
bitset<100005> b;
b.set(i); b.reset(i); b.flip(i); b[i]; b.count(); b.any(); b.none();
b |= (b << x);          // subset-sum DP
```

## 5. Custom Sorting and Comparators

```cpp
sort(all(v));                               // ascending
sort(all(v), greater<int>());               // descending
sort(all(v), [](const pii &a, const pii &b) {
    if (a.first != b.first) return a.first < b.first;
    return a.second > b.second;
});

struct Item {
    int a, b;
    bool operator<(const Item &o) const { return a < o.a; }
};
```

The comparator must be a strict weak ordering. Use `<`, never `<=`, or you risk a runtime error.

## 6. STL Algorithms

| Function | What it does |
|---|---|
| `sort(a, a+n)` / `sort(all(v))` | O(n log n) sort |
| `stable_sort` | keeps order of equal elements |
| `reverse(all(v))` | reverse |
| `min_element / max_element(all(v))` | iterator to min / max |
| `*max_element(all(v))` | max value |
| `accumulate(all(v), 0LL)` | sum (use `0LL` for long long!) |
| `count(all(v), x)` / `count_if` | count matches |
| `find(all(v), x)` | iterator or end() |
| `binary_search(all(v), x)` | exists? (sorted) |
| `lower_bound(all(v), x)` | first >= x (sorted) |
| `upper_bound(all(v), x)` | first > x (sorted) |
| `unique(all(v))` | remove consecutive duplicates |
| `fill(all(v), x)` / `memset(a, 0, sizeof a)` | fill (memset only reliable for 0 and -1) |
| `iota(all(v), 0)` | fill 0, 1, 2, ... |
| `next_permutation(all(v))` | next lexicographic permutation (false at end) |
| `prev_permutation` | previous permutation |
| `rotate(v.begin(), v.begin()+k, v.end())` | left rotate by k |
| `swap(a, b)` | swap |
| `min(a, b)`, `max(a, b)` | types must match: `max(1LL, x)` |
| `__gcd(a, b)` / `gcd(a, b)` | gcd (C++17: `lcm` too) |
| `partial_sum(all(v), pre.begin())` | prefix sums |
| `adjacent_difference` | difference array |
| `all_of / any_of / none_of` | predicate checks |
| `is_sorted(all(v))` | sorted check |
| `nth_element(v.begin(), v.begin()+k, v.end())` | k-th smallest in O(n) |
| `merge`, `set_union`, `set_intersection` | on sorted ranges |
| `shuffle(all(v), rng)` | shuffle |

**Binary search index tricks**

```cpp
int pos = lower_bound(all(v), x) - v.begin();    // count of elements < x
int cnt = upper_bound(all(v), x) - lower_bound(all(v), x);  // occurrences of x
```

**Random**

```cpp
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
int r = rng() % n;
shuffle(all(v), rng);
```

## 7. Bit Manipulation

```cpp
x & 1                   // odd?
x >> i & 1              // i-th bit
x | (1 << i)            // set bit
x & ~(1 << i)           // clear bit
x ^ (1 << i)            // toggle bit
x & (x - 1)             // remove lowest set bit
x & -x                  // lowest set bit
__builtin_popcount(x)   // number of set bits (popcountll for long long)
__builtin_clz(x)        // leading zeros
__builtin_ctz(x)        // trailing zeros
31 - __builtin_clz(x)   // floor(log2(x))

// iterate all subsets of n items
for (int mask = 0; mask < (1 << n); mask++) {}
// iterate submasks of mask
for (int sub = mask; sub; sub = (sub - 1) & mask) {}
```

## 8. Math and Number Theory Snippets

```cpp
ll power(ll b, ll e, ll m = MOD) {
    ll r = 1; b %= m;
    while (e > 0) {
        if (e & 1) r = r * b % m;
        b = b * b % m;
        e >>= 1;
    }
    return r;
}
ll inv(ll a) { return power(a, MOD - 2); }   // MOD must be prime

// Sieve
vector<bool> isp(N, true);
void sieve() {
    isp[0] = isp[1] = false;
    for (int i = 2; (ll)i * i < N; i++)
        if (isp[i]) for (int j = i * i; j < N; j += i) isp[j] = false;
}

// nCr mod prime
ll fact[N], ifact[N];
void init() {
    fact[0] = 1;
    for (int i = 1; i < N; i++) fact[i] = fact[i-1] * i % MOD;
    ifact[N-1] = inv(fact[N-1]);
    for (int i = N - 1; i > 0; i--) ifact[i-1] = ifact[i] * i % MOD;
}
ll nCr(int n, int r) {
    if (r < 0 || r > n) return 0;
    return fact[n] * ifact[r] % MOD * ifact[n-r] % MOD;
}

// Modular add / mul / sub
ll add(ll a, ll b) { return (a + b) % MOD; }
ll mul(ll a, ll b) { return a * b % MOD; }
ll sub(ll a, ll b) { return ((a - b) % MOD + MOD) % MOD; }
```

Other handy bits: `sqrtl`, `cbrt`, `pow` (double; avoid for exact integers), ceil division for integers `(a + b - 1) / b`, `acos(-1)` = pi, `hypot(x, y)`, `atan2(y, x)`.

## 9. Graph and DS Templates

### Adjacency list and BFS

```cpp
vector<vector<int>> g(n + 1);
g[u].pb(v); g[v].pb(u);          // undirected

vector<int> dist(n + 1, -1);
queue<int> q; q.push(s); dist[s] = 0;
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : g[u]) if (dist[v] == -1) {
        dist[v] = dist[u] + 1; q.push(v);
    }
}
```

### Dijkstra

```cpp
vector<vector<pair<int,ll>>> g(n + 1);   // {to, weight}
vector<ll> d(n + 1, LINF);
priority_queue<pair<ll,int>, vector<pair<ll,int>>, greater<>> pq;
d[s] = 0; pq.push({0, s});
while (!pq.empty()) {
    auto [du, u] = pq.top(); pq.pop();
    if (du > d[u]) continue;
    for (auto [v, w] : g[u])
        if (d[u] + w < d[v]) { d[v] = d[u] + w; pq.push({d[v], v}); }
}
```

### DSU (Union-Find)

```cpp
int par[N], rnk[N];
void init(int n) { for (int i = 0; i <= n; i++) par[i] = i, rnk[i] = 0; }
int find(int x) { return par[x] == x ? x : par[x] = find(par[x]); }
bool unite(int a, int b) {
    a = find(a); b = find(b);
    if (a == b) return false;
    if (rnk[a] < rnk[b]) swap(a, b);
    par[b] = a;
    if (rnk[a] == rnk[b]) rnk[a]++;
    return true;
}
```

### Fenwick Tree (BIT)

```cpp
ll bit[N];
void upd(int i, ll v) { for (; i < N; i += i & -i) bit[i] += v; }
ll qry(int i) { ll s = 0; for (; i > 0; i -= i & -i) s += bit[i]; return s; }
// range sum [l, r] = qry(r) - qry(l - 1)
```

### Prefix sums and 2D prefix

```cpp
pre[i] = pre[i-1] + a[i];                       // 1-indexed
sum(l, r) = pre[r] - pre[l-1];
p[i][j] = a[i][j] + p[i-1][j] + p[i][j-1] - p[i-1][j-1];
```

### Direction arrays (grids)

```cpp
int dx[] = {1, -1, 0, 0}, dy[] = {0, 0, 1, -1};
// 8 directions: dx[] = {1,1,1,0,0,-1,-1,-1}, dy[] = {1,0,-1,1,-1,1,0,-1}
```

## 10. Which Structure Should I Use?

| Need | Use |
|---|---|
| Count frequencies | `map` / `unordered_map` / array if range is small |
| Sorted unique values + neighbors | `set` + `lower_bound` |
| Repeated min or max extraction | `priority_queue` |
| Remove arbitrary elements and still get min/max | `multiset` |
| Shortest path, unweighted | BFS with `queue` |
| Shortest path, weighted non-negative | Dijkstra |
| Shortest path, 0/1 weights | 0-1 BFS with `deque` |
| Connectivity / components / Kruskal | DSU |
| Range sum with point updates | Fenwick tree |
| Range min/max static | sparse table; with updates, segment tree |
| Next greater / smaller element | monotonic `stack` |
| Sliding window min/max | monotonic `deque` |
| Subset sums with large n | `bitset` |
| Pairs or edges stored together | `pair`, `tuple`, or a small struct |

## 11. Common Pitfalls

- **Overflow**: use `long long` for sums and products. `(ll)a * b`, `1LL << k`, `accumulate(..., 0LL)`.
- **Array size**: declare globals a bit larger than the max constraint. Big arrays go global, not inside `main` (stack overflow).
- **Multiple test cases**: clear vectors, maps and arrays each test. `memset` over 1e5-sized arrays per test with large t gets TLE, so reset only up to n.
- **0-indexed vs 1-indexed**: stay consistent.
- **`size()` is unsigned**: `v.size() - 1` underflows when empty. Use `(int)v.size()` or `sz(v)`.
- **Erase in a loop** invalidates iterators. Use `it = s.erase(it)`.
- **`map[key]` inserts** the key if it is missing. Use `count` / `find` to test membership.
- **Recursion depth** around 1e5 or more may crash. Use iterative DFS or increase the stack if allowed.
- **`INF` addition**: `INF + INF` overflows `int`. Use `LINF` with `ll`, or guard checks.
- **Mod negatives**: add MOD before taking `%` after subtraction.
- **Double output**: use `fixed << setprecision(k)` and check the required format.
- **Output format**: exact spacing, case (`YES`/`Yes`), and trailing newline match the statement.
- **Reading until EOF**: `while (cin >> x) {}`.
- **Debug cheaply**: `#define dbg(x) cerr << #x << " = " << x << '\n';`

## 12. Contest-Day Strategy

- Read all problems first; start with the easiest ones (check the scoreboard for solve counts).
- Work out a brute force mentally and verify with small cases before coding the optimal solution.
- Test sample input and edge cases (n = 1, all equal, max values) before submitting; each wrong answer costs 20 penalty minutes.
- Share the single computer wisely: think and write on paper while a teammate codes.
- When stuck for 20+ minutes, switch problems or swap with a teammate.
- Keep your template in the team notebook if printed material is allowed.
