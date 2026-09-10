# 第 02 章　C++ 语法速成

> **本章目标**：把写算法题会用到的 C++ 语法一次讲完。
>
> ⚠️ 本章是「**够用就停**」的速成路线：只讲竞赛用得到的部分，
> 不讲类、模板、异常、智能指针这些工程内容。
> 看完能写题，但**不足以**去做软件开发。

---

## 2.1　变量与数据类型

**变量**就是给内存里的一块空间起个名字。

```cpp
int a = 5;              // 整数
long long b = 10000000000LL;  // 大整数
double c = 3.14;        // 小数
char d = 'A';           // 单个字符
bool e = true;          // 布尔（真/假）
string f = "hello";     // 字符串
```

### 最关键的：数据范围（溢出是头号 bug）

| 类型 | 占多少字节 | 范围 | 什么时候用 |
|---|:---:|---|---|
| `int` | 4 | 约 `-2.1e9 ~ 2.1e9` | 一般整数 |
| `long long` | 8 | 约 `-9.2e18 ~ 9.2e18` | **答案可能超 2e9 时** |
| `unsigned int` | 4 | `0 ~ 4.2e9` | 少用，容易踩坑 |
| `double` | 8 | 约 ±1.7e308，15 位精度 | 小数、几何、概率 |
| `char` | 1 | `-128 ~ 127` | 字符 |
| `bool` | 1 | `0 / 1` | 判断 |

**估算技巧**：`2^30 ≈ 1e9`，`int` 上限约 `2×1e9`。

### 什么时候必须用 `long long`？

只要**两个数相乘**，或者**要累加很多个数**，就换成 `long long`。

```cpp
int n = 100000;
int ans = n * n;        // ❌ 1e10 已经超过 int 上限，溢出成随机数
long long ans = 1LL * n * n;   // ✅ 正确
```

> 🎯 `1LL * n * n` 里的 `1LL` 是**类型提升**技巧：
> 先把 `n` 提升到 `long long`，再做乘法，避免中途溢出。
> 类似地还有 `1.0 *`（提升为 double）。

**赛题里的信号**：
- 出现 `n ≤ 1e5`，答案要取模 → 中间乘法必须 `long long`
- 出现 `n ≤ 1e9` 或 `2^63` → 全程 `long long`
- 出现「答案对 998244353 取模」→ 乘法全用 `long long`

### 两个省事写法

```cpp
using ll = long long;      // 之后可以用 ll 代替 long long
using namespace std;
```

有些人的板子里会写 `#define int long long`，
这样所有 `int` 都变 `long long`——**虽然方便，但会导致**
1. `main` 必须写成 `signed main()`（因为 `main` 不能返回 `long long`）
2. 数组占内存翻倍，开大数组可能 **MLE**
3. 运算变慢约 2~3 倍

**新手建议：不用这个宏，该写 `ll` 就写 `ll`。**

### 常量

```cpp
const int MOD = 998244353;      // 取模常用
const int INF = 0x3f3f3f3f;     // 无穷大（两个相加不溢出）
const ll  LLINF = 1e18;         // long long 版无穷大
```

---

## 2.2　运算符

### 算术

```cpp
int a = 7, b = 3;
a + b    // 10
a - b    // 4
a * b    // 21
a / b    // 2   ← 整数除法，直接截断小数！
a % b    // 1   ← 取余数
```

> ⚠️ **整数除法陷阱**：`7 / 2` 是 `3` 不是 `3.5`。
> 想得到小数必须写成 `7.0 / 2` 或 `(double)a / b`。

### 向上取整的写法

计算 `ceil(a / b)`（a、b 都是正整数）时，**不要用 `ceil`**（浮点有误差），用：

```cpp
int ans = (a + b - 1) / b;      // 整数意义下的向上取整
```

### 取模与负数（重点）

```cpp
7 % 3     // 1
-7 % 3    // -1   ← C++ 里负数取模结果是负数！（数学上期望是 2）
```

**为什么重要**：做减法后取模，结果可能变负，导致答案错误。

```cpp
// ❌ 错误写法
ans = (a - b) % MOD;                  // 可能得到负数

// ✅ 正确写法
ans = ((a - b) % MOD + MOD) % MOD;    // 先加 MOD 变正，再取模
```

**另外，位运算不能替代取模！**
```cpp
x % 2   // ✅ 等价于 x & 1（仅当 x >= 0）
x % 4   // ✅ 等价于 x & 3（仅当 2 的幂，且 x >= 0）
x % 3   // ❌ 没有位运算写法
```

### 自增自减

```cpp
int i = 5;
++i;    // i 变成 6（前置：先加再用）
i++;    // i 变成 7（后置：先用再加）
```

**在循环里 `i++` 和 `++i` 效果一样**，竞赛里随便写。

### 比较与逻辑

```cpp
a == b    // 相等（注意是两个等号！）
a != b    // 不等
a > b     // 大于
a <= b    // 小于等于
!flag     // 逻辑非
x && y    // 逻辑与（都为真才为真）
x || y    // 逻辑或（有一个真就为真）
```

> 🎯 **新手第一大坑**：`if (a = b)` 和 `if (a == b)` 完全不同！
> 前者是**赋值**，会把 `b` 赋给 `a`，而且几乎永远为真。
> 这个错误**编译器不会报错**（只给个警告），务必打开 `-Wall`。

### 位运算（竞赛高频）

先把整数想成二进制，例如 `13 = 1101₂`。

| 运算 | 符号 | 例子 | 含义 |
|---|:---:|---|---|
| 与 | `&` | `13 & 10 = 8` | 两位都为 1 才是 1 |
| 或 | `\|` | `13 \| 10 = 15` | 有一位为 1 就是 1 |
| 异或 | `^` | `13 ^ 10 = 7` | 两位不同才是 1 |
| 取反 | `~` | `~13 = -14` | 0/1 互换 |
| 左移 | `<<` | `1 << 4 = 16` | 左移 n 位 = 乘 `2^n` |
| 右移 | `>>` | `16 >> 4 = 1` | 右移 n 位 = 除以 `2^n` |

**三个必背技巧：**

```cpp
x << 1              // x * 2
x >> 1              // x / 2
x & 1               // x 的奇偶性：1 为奇，0 为偶
x & (x - 1)         // 把最低位的 1 变成 0
x & (-x)            // 取出最低位的 1（树状数组的核心 lowbit）
1 << k              // 第 k 位为 1（表示「第 k 个元素被选中」）
int(x >> k) & 1     // 取出 x 的第 k 位
```

**异或的性质（后面 DP / 线性基常用）：**

```cpp
x ^ x == 0          // 自己异或自己是 0
x ^ 0 == x          // 异或 0 不变
x ^ y == y ^ x      // 交换律
a ^ b ^ b == a      // 可以「抵消」
```

> ⚠️ **运算优先级陷阱**：`&`、`|`、`^` 的优先级**低于** `==`！
> ```cpp
> if (x & 1 == 0)      // ❌ 会被理解成 x & (1 == 0)
> if ((x & 1) == 0)    // ✅ 加括号
> ```
> **规则：位运算一律加括号。**

---

## 2.3　分支与循环

### if / else

```cpp
if (score >= 90) {
    cout << "优秀\n";
} else if (score >= 60) {
    cout << "及格\n";
} else {
    cout << "不及格\n";
}
```

**三元运算符**（简单的二选一可以缩写）：

```cpp
int mx = (a > b) ? a : b;      // 如果 a > b 取 a，否则取 b
```

### 三种循环

```cpp
// ① for：次数已知时用
for (int i = 0; i < n; i++) {
    sum += a[i];
}

// ② while：条件满足就一直做
while (x > 0) {
    x /= 10;
    cnt++;
}

// ③ do-while：至少执行一次
do {
    x--;
} while (x > 0);
```

### ⭐ 范围 for（C++11 起，写题最常用）

```cpp
vector<int> v = {1, 2, 3, 4};

// ① 只读遍历（不修改）
for (int x : v) {
    cout << x << ' ';
}

// ② 引用遍历（可以修改原值）
for (int &x : v) {
    x *= 2;             // v 变成 {2, 4, 6, 8}
}

// ③ 只读引用（避免拷贝大对象，推荐）
for (const int &x : v) {
    cout << x << ' ';
}
```

**什么时候用 `&`？**
- 要**修改**元素 → 必须 `int &x`
- 元素是**大对象**（如 `string`、`vector`）→ 用 `const auto &x` 省时间
- 元素是小整数 → 随便

### `break` 和 `continue`

```cpp
for (int i = 0; i < n; i++) {
    if (a[i] == x) { found = true; break; }    // 跳出整个循环
    if (a[i] < 0) continue;                     // 跳过本次，进入下一轮
    sum += a[i];
}
```

> ⚠️ `break` 只跳出**最内层**循环。嵌套循环想一起跳出，可以用一个 `flag` 变量。

---

## 2.4　数组与 `vector`

### 定长数组

```cpp
int a[10];                    // 下标 0..9
int b[105] = {0};             // 全部初始化为 0（推荐）
int c[5] = {1, 2, 3};         // 前三个是 1,2,3，后面自动补 0

// 二维数组
int mp[105][105] = {0};
```

**竞赛常用做法**：全局开大数组（默认全 0，且在堆上，不会爆栈）

```cpp
const int N = 100005;
int a[N];                     // 全局，自动初始化为 0
```

> ⚠️ **局部数组在栈上**，开太大（如 `int a[1000000]`）会**栈溢出崩溃**。
> 大数组一律**放全局**。

### `vector`：可变长数组（竞赛主力）

```cpp
vector<int> v;                // 空
vector<int> v2(5);            // 5 个 0
vector<int> v3(5, 3);         // 5 个 3
vector<int> v4 = {1, 2, 3};   // 初始化列表

v.push_back(10);              // 末尾添加
v.pop_back();                 // 删除末尾
v.size();                     // 元素个数（返回 unsigned，注意！）
v.empty();                    // 是否为空
v.clear();                    // 清空
v.front();                    // 第一个元素
v.back();                     // 最后一个元素
v[0];                         // 下标访问（不检查越界！）
```

**二维 vector（图的邻接表常用）：**

```cpp
vector<vector<int>> g(n + 1);   // n+1 个空的 vector
g[1].push_back(2);              // 1 号点的邻居加一个 2
```

**带权图的邻接表（pair）：**

```cpp
vector<vector<pair<int,int>>> g(n + 1);  // {邻居, 边权}
g[u].push_back({v, w});
g[v].push_back({u, w});                  // 无向图要加两次
```

> ⚠️ **`v.size()` 返回无符号整数**，和 `int` 比较时要小心：
> ```cpp
> for (int i = 0; i < v.size() - 1; i++)   // ❌ v 为空时，size()-1 = 巨大正数！
> for (int i = 0; i + 1 < (int)v.size(); i++)  // ✅ 强制转 int
> ```
> **习惯：和 `size()` 比较时一律 `(int)v.size()`。**

---

## 2.5　函数与递归

### 函数

```cpp
// 返回类型 函数名(参数) { 函数体 }
int add(int a, int b) {
    return a + b;
}

int gcd(int a, int b) {          // 最大公约数（辗转相除）
    return b == 0 ? a : gcd(a, b % a);
}

void printHello() {              // void 表示不返回
    cout << "Hello\n";
}

int main() {
    cout << add(3, 5) << '\n';    // 8
    cout << gcd(12, 18) << '\n';  // 6
}
```

**引用传参（重要）：**

```cpp
void modify(int x)  { x = 100; }     // ❌ 传值：改的是副本，外面不变
void modify2(int &x) { x = 100; }    // ✅ 传引用：真的改了外面的

void printBig(const vector<int> &v) {   // ✅ const 引用：不拷贝、也不改
    for (int x : v) cout << x << ' ';
}
```

**规则**：参数是 `vector` / `string` 等大对象时，**一定加 `&`**，否则每次调用都复制一遍，慢到超时。

### 递归

**递归 = 函数调用自己**。关键是找到两件事：

1. **递归边界**（什么时候停）
2. **递归式**（大问题怎么变小事）

```cpp
// 阶乘
long long fact(int n) {
    if (n <= 1) return 1;         // 边界
    return n * fact(n - 1);       // 递归式
}

// 汉诺塔
void hanoi(int n, char from, char via, char to) {
    if (n == 0) return;
    hanoi(n - 1, from, to, via);
    cout << from << " -> " << to << '\n';
    hanoi(n - 1, via, from, to);
}
```

> ⚠️ **递归深度**：默认栈深度约几万层。递归 `1e6` 层会**栈溢出**，
> 这时要么改写成迭代（自己开 `stack`），要么减少递归深度。
> 有些平台的栈更小，注意看题。

**递归的思维模型**：不要试图在脑子里跟踪每一层！
只相信「**子问题的答案是对的**」，然后想「怎么用子问题的答案拼出原问题的答案」。

---

## 2.6　结构体、排序与自定义比较

### 结构体

把几个相关的数据打包成一个整体：

```cpp
struct Student {
    string name;
    int score;
    int id;
};

Student s;
s.name = "Tom";
s.score = 90;

// 直接初始化
Student t = {"Jerry", 85, 2};
vector<Student> v = {{"Tom", 90, 1}, {"Jerry", 85, 2}};
```

### `sort`：排序（写题第一高频函数）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> a = {5, 2, 8, 1, 9};

    sort(a.begin(), a.end());                 // 升序：1 2 5 8 9
    sort(a.begin(), a.end(), greater<int>()); // 降序：9 8 5 2 1

    // 数组写法
    int b[5] = {5, 2, 8, 1, 9};
    sort(b, b + 5);                           // 升序
}
```

复杂度 `O(n log n)`，默认**升序**。

### 自定义排序规则（重点）

**写法一：`cmp` 函数**（最传统）

```cpp
struct Edge { int u, v, w; };

// 按边权从小到大
bool cmp(const Edge &a, const Edge &b) {
    return a.w < b.w;
}

sort(edges.begin(), edges.end(), cmp);
```

**写法二：Lambda**（现代写法，推荐）

```cpp
sort(v.begin(), v.end(), [](const Edge &a, const Edge &b) {
    return a.w < b.w;
});
```

**多关键字排序**（分数降序，分数相同按 id 升序）：

```cpp
sort(v.begin(), v.end(), [](const Student &a, const Student &b) {
    if (a.score != b.score) return a.score > b.score;   // 先比分数，降序
    return a.id < b.id;                                 // 再比 id，升序
});
```

> ⚠️ **`cmp` 必须满足「严格弱序」**：
> - `cmp(a, a)` 必须为 `false`（不能写成 `<=`！）
> - 写成 `return a.x <= b.x;` 会导致 `sort` **内存越界、程序崩溃**
>
> **口诀：`cmp` 里永远用 `<` 或 `>`，绝不用 `<=` 或 `>=`。**

**为什么？** 因为 `sort` 用 `cmp(a,b) == false && cmp(b,a) == false` 判断「等价」，
`<=` 会让 `a <= a` 为真，逻辑自相矛盾，`sort` 的实现就跑飞了。

### `pair`（成对数据）

```cpp
pair<int, int> p = {3, 5};
p.first    // 3
p.second   // 5

// pair 自带比较：先比 first，再比 second
vector<pair<int,int>> vp = {{2, 9}, {1, 5}, {2, 3}};
sort(vp.begin(), vp.end());
// 结果：{1,5}, {2,3}, {2,9}

// 结构化绑定（C++17，超好用）
auto [x, y] = p;
for (auto [u, v] : vp) cout << u << ' ' << v << '\n';
```

`pair` 默认按 `first` 升序、`first` 相同时按 `second` 升序。
想降序就把元素取负，或者写 `cmp`。

---

## 2.7　常用 STL 容器

### 一览表

| 容器 | 底层 | 能干什么 | 单次操作 |
|---|---|---|---|
| `vector` | 动态数组 | 随机访问、尾部增删 | `O(1)` / 尾 `O(1)` |
| `string` | 字符数组 | 字符串处理 | 类似 `vector` |
| `pair` | 结构体 | 打包两个值 | — |
| `stack` | 栈 | 后进先出 | `O(1)` |
| `queue` | 队列 | 先进先出 | `O(1)` |
| `deque` | 双端队列 | 两头都能进出 | `O(1)` |
| `priority_queue` | 堆 | 每次取最大/最小 | `O(log n)` |
| `set` | 红黑树 | 有序、去重 | `O(log n)` |
| `multiset` | 红黑树 | 有序、可重复 | `O(log n)` |
| `map` | 红黑树 | 键值对、键有序 | `O(log n)` |
| `unordered_map` | 哈希表 | 键值对、平均更快 | 平均 `O(1)` |
| `bitset` | 位数组 | 位运算加速 | `O(n/64)` |

**怎么选？**
- 只是存一串数 → `vector`
- 要**去重 + 有序 + 查某个数在不在** → `set`
- 要**统计出现次数** → `map`（键有序）或 `unordered_map`（更快，但可能被卡）
- 要**每次取最大值** → `priority_queue`
- 要**先进先出** → `queue`（如 BFS）
- 要**后进先出** → `stack`（如 DFS）

### `stack` 栈

```cpp
stack<int> st;
st.push(1); st.push(2); st.push(3);
st.top();      // 3
st.pop();      // 弹出 3
st.size();     // 2
st.empty();    // false
```

> ⚠️ `pop()` **不返回**被弹出的值！要先 `top()` 再 `pop()`。

### `queue` 队列

```cpp
queue<int> q;
q.push(1); q.push(2);
q.front();     // 1（队首）
q.back();      // 2（队尾）
q.pop();       // 弹出 1
q.empty();
```

### `priority_queue` 优先队列（堆）

```cpp
// 大根堆（默认，堆顶最大）
priority_queue<int> pq;
pq.push(3); pq.push(1); pq.push(5);
pq.top();      // 5

// 小根堆（堆顶最小）—— 必须写全这个模板
priority_queue<int, vector<int>, greater<int>> pq2;

// pair 的堆：按 first 排序
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq3;
pq3.push({1, 5});      // {1,5} 在堆顶

// 自定义结构体
struct Node { int d, u; };
bool operator<(const Node &a, const Node &b) {
    return a.d > b.d;      // 注意：这里是反的！见下方说明
}
priority_queue<Node> pq4;
```

> 🎯 **最反直觉的点**：`priority_queue` 用 `<` 判断「谁排在后面」。
> 所以 `operator<` 里写 `a.d > b.d` 得到的是**小根堆**。
> **记不住就别用结构体，改用 `pair`+`greater<>`。**

**Dijkstra 里的典型用法**：`priority_queue<pair<ll,int>, vector<...>, greater<>>`，
`first` 是距离，`second` 是点编号——距离小的优先。

### `set` / `multiset`

```cpp
set<int> s;                 // 自动排序 + 自动去重
s.insert(3); s.insert(1); s.insert(3);
// s = {1, 3}（3 只出现一次）

s.count(3);                 // 1（存在）
s.find(3) != s.end();       // true
s.erase(3);                 // 删除

// 二分查找（O(log n)，红黑树内部二分）
auto it = s.lower_bound(5); // 第一个 >= 5 的迭代器
auto it2 = s.upper_bound(5);// 第一个 > 5 的迭代器
s.begin();                  // 最小值
*prev(s.end());             // 最大值

multiset<int> ms;           // 允许重复
ms.insert(3); ms.insert(3);
ms.count(3);                // 2
ms.erase(ms.find(3));       // 只删一个（erase(3) 会删掉全部！）
```

> ⚠️ **`multiset::erase(x)` 会删掉所有等于 x 的元素**。
> 只删一个要用 `erase(find(x))`。

> ⚠️ `set` 里的元素**不能修改**（会破坏排序），要改就删了再插。

### `map` / `unordered_map`

```cpp
map<string, int> cnt;       // 键是有序的
cnt["apple"]++;             // 不存在时自动创建并置 0，然后 +1
cnt["banana"] += 2;

// 遍历（按 key 升序）
for (auto &[key, val] : cnt) {
    cout << key << " -> " << val << '\n';
}

// 查找（不要用 cnt[key] 查询！）
if (cnt.find("apple") != cnt.end()) {
    cout << cnt["apple"] << '\n';
}
```

> ⚠️ **`mp[key]` 在 key 不存在时会「插入一个默认值」**。
> 用它做「查询」会悄悄改变容器大小，多组数据时容易出鬼。
> **查询一律用 `find()` 或 `count()`。**

`unordered_map` 用法一样，平均更快，但：
- 元素**无序**（不能按 key 遍历排序）
- 出题人可以**构造数据把哈希卡成 `O(n)`**（防卡技巧见第 13 章）

### `bitset`（位运算加速）

```cpp
bitset<100> bs;             // 100 位，默认全 0
bs[0] = 1;                  // 第 0 位置 1
bs.set(3);                  // 第 3 位置 1
bs.reset(3);                // 第 3 位置 0
bs.count();                 // 1 的个数
bs.set();                   // 全部置 1

bitset<8> a(string("10101010"));
bitset<8> b(string("11001100"));
(a & b).to_string();        // "10001000"（& | ^ 都支持，一次算 8 位）
```

**用途**：布尔数组的批量操作、`O(n/64)` 的可行性 DP、KMP 匹配加速。

---

## 2.8　常用库函数速查

### 查找

```cpp
vector<int> a = {1, 3, 5, 7, 9};        // 必须是有序的！

// 第一个 >= 5 的位置（返回迭代器）
auto it = lower_bound(a.begin(), a.end(), 5);
int idx = it - a.begin();               // 2

// 第一个 > 5 的位置
auto it2 = upper_bound(a.begin(), a.end(), 5);
int idx2 = it2 - a.begin();             // 3

// 数组版
int b[] = {1, 3, 5, 7, 9};
int p = lower_bound(b, b + 5, 5) - b;   // 2

// 降序数组要先传 greater
sort(a.begin(), a.end(), greater<int>());
auto it3 = lower_bound(a.begin(), a.end(), 5, greater<int>());
```

> 🎯 **`lower_bound` 是二分查找，前提是数组有序**！忘了排序 → 结果随机。
> 复杂度 `O(log n)`。

### 去重

```cpp
vector<int> a = {1, 1, 2, 3, 3, 3, 5};

sort(a.begin(), a.end());                    // ① 先排序（必须）
a.erase(unique(a.begin(), a.end()), a.end()); // ② 再去重
// a = {1, 2, 3, 5}
```

`unique` 只是把重复元素**移到后面**并返回「新末尾」，
所以**必须配合 `erase`** 才能真正删除。

### 翻转与旋转

```cpp
reverse(a.begin(), a.end());                        // 整个翻转
reverse(a.begin(), a.begin() + 3);                  // 只翻转前 3 个
rotate(a.begin(), a.begin() + 1, a.end());          // 循环左移 1 位
```

### 最值与累加

```cpp
int mx = *max_element(a.begin(), a.end());
int mn = *min_element(a.begin(), a.end());
int i  = max_element(a.begin(), a.end()) - a.begin();   // 最大值下标

long long sum = accumulate(a.begin(), a.end(), 0LL);    // ⚠️ 初值要给 0LL！
```

> ⚠️ `accumulate` 第三个参数是**初值**，写成 `0` 会按 `int` 累加 → **溢出**。
> **一律写 `0LL`。**

### 全排列

```cpp
sort(a.begin(), a.end());           // 必须先排序（从最小排列开始）
do {
    for (int x : a) cout << x << ' ';
    cout << '\n';
} while (next_permutation(a.begin(), a.end()));
```

`n ≤ 8` 左右可以用它暴力枚举所有排列（`n!` 增长极快）。

### 填充与拷贝

```cpp
fill(a.begin(), a.end(), 0);        // 全部填 0
memset(arr, 0, sizeof(arr));        // 只能填 0 / -1 / 0x3f
copy(a.begin(), a.end(), b.begin()); // 拷贝
iota(a.begin(), a.end(), 1);        // 从 1 开始递增填充：1,2,3,...
```

> ⚠️ `memset` 按**字节**填充。填 `0x3f` 得到的是 `0x3f3f3f3f`（约 1e9），
> 这在竞赛里被当成「很大的数」；但填 `1` 会得到 `0x01010101 = 16843009`，
> **不是你想要的值**。**要用 `memset` 就用 `0`、`-1`、`0x3f`。**

### 内置位运算函数（GCC 专用，极快）

```cpp
__builtin_popcount(x)      // x 的二进制里 1 的个数（int 版）
__builtin_popcountll(x)    // long long 版
__builtin_clz(x)           // 前导 0 的个数（int 版）
__builtin_ctz(x)           // 末尾 0 的个数（int 版）= 最低位 1 的下标
__builtin_parity(x)        // 1 的个数的奇偶性
```

**直接拿来算「最高位」和「最低位」：**

```cpp
int lowbit = x & (-x);                 // 最低位的 1（如 x=12(1100) → 4）
int highbit = 1 << (31 - __builtin_clz(x));   // 最高位
```

> ⚠️ `__builtin_clz(0)` 和 `__builtin_ctz(0)` 是**未定义行为**，用前判 0。

### 数学函数

```cpp
sqrt(x)        // 平方根（double，有精度误差！）
pow(a, b)      // a^b（double，慢且不准，整数幂请用快速幂）
abs(x)         // 绝对值（int 版；long long 用 llabs 或 std::abs）
fabs(x)        // double 绝对值
floor(x) / ceil(x)   // 向下 / 向上取整（double 版）
```

> ⚠️ **整数开方不要直接用 `sqrt`**：
> ```cpp
> long long n = 1e18;
> long long r = sqrt(n);        // ❌ 浮点误差，可能差 1
>
> // ✅ 安全写法
> long long r = sqrt(n);
> while ((r + 1) * (r + 1) <= n) r++;
> while (r * r > n) r--;
> ```

### 输出控制

```cpp
cout << fixed << setprecision(10) << ans << '\n';   // 保留 10 位小数
printf("%.10f\n", ans);                             // 同上
```

浮点答案一般要求**误差不超过 `1e-6`**，所以输出 **10~12 位小数**比较保险。

---

## 2.9　新手必踩的 10 个坑

| # | 坑 | 正确做法 |
|:---:|---|---|
| 1 | `if (a = b)`（单等号） | 判断相等用 `==` |
| 2 | `int` 溢出 | 有乘法/大累加就换 `long long`，用 `1LL *` 提升 |
| 3 | `cmp` 里用了 `<=` | 只用 `<` / `>` |
| 4 | 数组开太小 | 全局开 `N + 5`，永远多留 5 个 |
| 5 | 局部开大数组爆栈 | 大数组放全局 |
| 6 | `(a - b) % MOD` 得到负数 | 写 `((a - b) % MOD + MOD) % MOD` |
| 7 | `cout << endl` 导致超时 | 改成 `'\n'` |
| 8 | `mp[key]` 当查询用 | 用 `find()` / `count()` |
| 9 | `v.size() - 1` 出负数变巨大 | 写 `(int)v.size() - 1` |
| 10 | 忘了 `sort` 就 `lower_bound` | 二分前必须有序 |

**额外两个：**
- 调试输出忘了删 → 判成 `Wrong Answer`
- 多组数据忘了清空全局数组 / 容器 → 上一组的数据污染这一组

---

## 2.10　本章练习

**基础（必须做）**
1. 读入 n 和 n 个整数，从小到大输出，再输出最大值、最小值、和。
2. 读入一个字符串，统计每个字符出现次数。
3. 读入 n 个 `{姓名, 分数}`，按分数从高到低输出；分数相同按姓名排序。
4. 用递归求斐波那契第 n 项（n ≤ 30）。
5. 用 `next_permutation` 输出 `1..n` 的所有排列（n ≤ 7）。

**进阶**
6. 读入 n 个数，去重后输出（用 `sort` + `unique`）。
7. 用 `set` 维护一个集合，支持插入、删除、查询「比 x 小的最大元素」。
8. 用 `priority_queue` 实现：不断加入数字，每次输出当前中位数
   （提示：用两个堆）。
9. 用 `bitset` 求出 1~100 内所有素数（埃氏筛的位运算版）。
10. 写一个 `cmp`，把 `vector<pair<int,int>>` 按 `second` 降序排序，
    `second` 相同时按 `first` 升序。

> 答案思路都在本章里。**敲不出来就回去看对应小节，不要直接搜答案。**

---

## 本章小结

到这一步你应该能：

- ✅ 选择合适的类型，并知道什么时候必须 `long long`
- ✅ 用 `vector` / `map` / `set` / `priority_queue` / `queue` / `stack` 解决「存、查、排序、取最值」
- ✅ 写 `cmp` 排序结构体
- ✅ 用 `sort` / `lower_bound` / `unique` / `reverse` / `accumulate` 等库函数省事
- ✅ 背下那 10 个坑

→ 下一章：[第 03 章 复杂度与基础算法](03-复杂度与基础算法.md)


