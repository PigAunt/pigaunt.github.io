# 状态压缩的动态规划


## 分类

基本有两种：
+ 棋盘式的动态规划（基于联通性）
+ 集合式的动态规划

## 状态压缩

利用某些手段（以二进制压缩为主）将复杂的状态处理后存储。

## 分类

### 棋盘式的动态规划

给你一个棋盘，要在棋盘上摆放一些东西，但又有一些限制，求方案数/最多摆放数量。这类问题一般将每一行（自上而下递推）或每一列（自左而右递推）的摆放情况压缩成一个二进制数，每一位对应一个位置的摆放情况，然后根据题目限制设计递推式。

### 集合式的动态规划

其实和上面类似，也是将一组数二进制压缩成一个数，只不过这一组数不表示棋盘上的一行或一列。

## 例题

[LOJ-10170-国王](https://loj.ac/p/10170) / [LOJ-2153-「SCOI2005」互不侵犯](https://loj.ac/p/2153)

本题属于**棋盘式的动态规划**。

可以发现，每一行的摆放情况只与上一行有关。并且每一行的摆放情况可以用一个二进制数表示。因此，我们用$f_{i, k, state}$表示放完$i$行，至多放$k$个，且第$i$行的摆放情况是$state$的最多摆放数量。

若用$s_i$、$s_j$分别表示相邻的两行，那么由于国王不能上下相邻，所以$s_i \land s_j$必须为$0$，又因为国王间至少隔2格，那么必须满足$s_i$、$s_j$、$s_i \lor s_j$没有相邻的两位都是$1$。

排除了错误的摆放情况，我们可以用一个`std::vector<int>`存储所有可能的情况，再用一个`std::vector<int>`的数组表示状态间的可能的转移。

为了减少计算，我们还可以用一个数组`count[state]`表示特定状态的国王个数。

**小技巧：最后不必枚举最后一行的状态再相加，可以递推到原有行数的下一行，用$f_{n + 1, m, 0}$表示答案。**

```cpp
#include <iostream>
#include <vector>

const int N = 12, M = 1024, K = 105;

int n, m, count[M];
long long f[N][K][M];
std::vector<int> s;
std::vector<int> to[M];

bool Check(int state)
{
    for (int i = 0; i < n; i++)
        if (((state >> i) & 1) && ((state >> (i + 1)) & 1))
            return false;
    return true;
}

int CountKing(int state)
{
    int res = 0;
    for (int i = 0; i < n; i++)
        if ((state >> i) & 1)
            res++;
    return res;
}

int main()
{
    std::cin >> n >> m;
    
    for (int i = 0; i < (1 << n); i++)
        if (Check(i))
        {
            s.push_back(i);
            count[i] = CountKing(i);
        }
    
    for (int i = 0; i < s.size(); i++)
        for (int j = 0; j < s.size(); j++)
        {
            int prev = s[i], next = s[j];
            if (((prev & next) == 0) && Check(prev | next))
                to[i].push_back(j);
        }
    
    f[0][0][0] = 1;
    for (int i = 1; i <= n + 1; i++)
        for (int j = 0; j <= m; j++)
            for (int prev = 0; prev < s.size(); prev++)
                for (int next : to[prev])
                {
                    int c = count[s[prev]];
                    if (j >= c)
                        f[i][j][next] += f[i - 1][j - c][prev];
                }
    
    std::cout << f[n + 1][m][0] << std::endl;
    return 0;
}
```

[AcWing-327-玉米田](https://www.acwing.com/problem/content/329/)

与上一题类似，但要注意有的土地不能种。要排除那些种植情况，我们可以用一个数组`g[line]`表示该行不能种的情况（将不能种的位定为$1$），如果一个种植情况$state$和`g[line]`有公共的位置（即两者相与不为$0$），这个情况就要排除。

```cpp
#include <iostream>
#include <vector>

const int N = 14, M = 4096, MOD = 1e8;

int n, m;
int g[N];
std::vector<int> s;
std::vector<int> to[M];
int f[N][M];

bool Check(int state)
{
    for (int i = 0; i < m; i++)
        if (((state >> i) & 1) && ((state >> (i + 1)) & 1))
            return false;
    return true;
}

int main()
{
    std::cin >> n >> m;
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < m; j++)
        {
            bool t;
            std::cin >> t;
            g[i] += (!t) << j;
        }
    
    for (int i = 0; i < (1 << m); i++)
        if (Check(i))
            s.push_back(i);
    
    for (int i = 0; i < s.size(); i++)
        for (int j = 0; j < s.size(); j++)
        {
            int prev = s[i], next = s[j];
            if ((prev & next) == 0)
                to[i].push_back(j);
        }
        
    f[0][0] = 1;
    for (int i = 1; i <= n + 1; i++)
        for (int prev = 0; prev < s.size(); prev++)
            for (int next : to[prev])
            {
                if (g[i] & s[next])
                    continue;
                f[i][next] = (f[i][next] + f[i - 1][prev]) % MOD;
            }
    
    std::cout << f[n + 1][0] << std::endl;
    return 0;
}
```

[洛谷-P2704-[NOI2001] 炮兵阵地](https://www.luogu.com.cn/problem/P2704)

与上一题类似，本题有的地方不能放炮兵部队。但本题与前面的区别在于，影响的范围多了一格，因此第$i$行的摆放情况与第$i - 1$行、第$i - 2$行有关。用$f_{i, s1, s2}$表示放完$i$行，第$i$行状态是$s1$、第$i - 1$行状态是$s2$。

为压缩空间，这里用了滚动数组。

```cpp
#include <iostream>
#include <vector>

const int N = 105, M = 11, K = 1024;

int n, m;
int g[N], count[K];
std::vector<int> s;
int f[2][K][K];

bool Check(int state)
{
    for (int i = 0; i < m; i++)
        if (((state >> i) & 1) && (((state >> (i + 1)) & 1) || ((state >> (i + 2)) & 1)))
            return false;
    return true;
}

int GetCount(int state)
{
    int res = 0;
    for (int i = 0; i < m; i++)
        if ((state >> i) & 1)
            res++;
    return res;
}

int main()
{
    std::cin >> n >> m;
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < m; j++)
        {
            char c;
            std::cin >> c;
            if (c == 'H')
                g[i] += 1 << j;
        }
    
    for (int i = 0; i < (1 << m); i++)
        if (Check(i))
        {
            s.push_back(i);
            count[i] = GetCount(i);
        }
    
    for (int i = 1; i <= n + 2; i++)
        for (int a = 0; a < s.size(); a++) // i
            for (int b = 0; b < s.size(); b++) // i - 1
                for (int c = 0; c < s.size(); c++) // i - 2
                {
                    int p = s[a], q = s[b], r = s[c];
                    if ((p & q) | (q & r) | (p & r))
                        continue;
                    if ((g[i] & p) | (g[i - 1] & q))
                        continue;
                    f[i & 1][a][b] = std::max(f[i & 1][a][b], f[(i - 1) & 1][b][c] + count[p]);
                }
    
    std::cout << f[(n + 2) & 1][0][0] << std::endl;
    return 0;
}
```

[洛谷-P831-[NOIP2016 提高组] 愤怒的小鸟](https://www.luogu.com.cn/problem/P2831)

本题属于**集合式的动态规划**。

可以把一条抛物线覆盖的猪按猪的编号二进制压缩成一个数，用$f_{state}$表示达到这一状态的最少抛物线数量。由于本题中的抛物线一定过原点，所以只要找到另外两个点就可以确定一条抛物线，我们用$path_{i, j}$表示有一条抛物线经过$i$、$j$的覆盖情况。

处理完$path_{i, j}$后，我们枚举状态$state$并扫描它，找到第一个猪没有被覆盖的位置并标记，并用与它构成的$path$来更新$state$。转移方程如下：

$$
f_{state \lor path_{from, to}} = \min(f_{to} + 1)
$$

时间复杂度$O(T(n ^ 3 + 2 ^ n n))$。

```cpp
#include <iostream>
#include <cmath>
#include <vector>

const int N = 20, M = (1 << 18) + 5, PosInf = 1e9;
const double EPS = 1e-6;

struct Pig
{
    double x, y;
} pigs[N];

int n, m;
int f[M], path[N][N];

int main()
{
    int T;
    std::cin >> T;
    for (int test = 1; test <= T; test++)
    {
        std::cin >> n >> m;
        for (int i = 1; i <= n; i++)
            std::cin >> pigs[i].x >> pigs[i].y;

        for (int i = 1; i <= (1 << n) - 1; i++)
            f[i] = PosInf;
        f[0] = 0;

        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
            {
                path[i][j] = 0;
                if (i == j)
                {
                    path[i][j] = (1 << (i - 1));
                    continue;
                }

                Pig &u = pigs[i];
                Pig &v = pigs[j];
                if (fabs(u.x - v.x) < EPS)
                    continue;

                // x1 * x1 * a + b * x1 = y1
                // x2 * x2 * a + b * x2 = y2
                double a = (u.y * v.x - v.y * u.x) / (u.x * u.x * v.x - v.x * v.x * u.x);
                if (fabs(a - 0) < EPS || a > 0)
                    continue;
                double b = (u.y - a * u.x * u.x) / u.x;

                int state = (1 << (i - 1)) | (1 << (j - 1));

                for (int k = 1; k <= n; k++)
                {
                    if (k == i || k == j)
                        continue;

                    Pig &w = pigs[k];
                    if (fabs(a * w.x * w.x + b * w.x - w.y) < EPS)
                        state = state | (1 << (k - 1));
                }

                path[i][j] = state;
            }

        for (int i = 0; i <= (1 << n) - 1; i++)
        {
            int x = 0;
            for (int j = 1; j <= n; j++)
                if (!((i >> (j - 1)) & 1))
                {
                    x = j;
                    break;
                }

            for (int j = 1; j <= n; j++)
                f[i | path[x][j]] = std::min(f[i | path[x][j]], f[i] + 1);
        }

        std::cout << f[(1 << n) - 1] << std::endl;
    }
    return 0;
}
```
