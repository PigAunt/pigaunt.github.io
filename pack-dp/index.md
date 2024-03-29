# 背包问题


## 01背包

现在有一个固定容量的背包，和若干体积一定、价值一定的物品，要将一些物品放入背包中，并获得最大的价值。

如果我们已经有了一个装有最大价值物品的背包，要对它进行扩容，并装入体积为$v_i$，价值为$w_i$的物品，此时最大价值就是原来的价值加上新增物品的价值。如果用$f_{i, j}$表示选到第$i$个物品，总体积为$j$，依照这个思路写出的转移方程如下：

$$
f_{i, j} = f_{i - 1, j} \\
f_{i, j} = \max(f_{i - 1, j - v_i} + w_i)
$$

### 恰好和不超过

其实认真想想，不管$j$是指“恰好”还是“不超过”，转移方程是不变的。但要注意在初始化的时候，“恰好”型需要将$f_0$外的$f$数组初始化为$-\infty$

### 空间优化

可以发现，$f_{i, j}$由$f_{i - 1, j}$和$f_{i - 1, j - v_i} + w_i$递推而来，只跟上一个状态有关，因此可以将$O(NM)$的空间复杂度优化至$O(M)$，只需要控制倒序循环（先更新大的，保证不会让新的去更新新的）即可。

### 时间优化

因为该问题不受后面信息的影响，可以在读入时直接开始递推。

模板代码如下：

```cpp
#include <iostream>

const int N = 1005, V = 1005;

int n, v;
int itemSpace[N], itemWeight[N];

int SinglePack()
{
	int d[N][V];
	for (int i = 0; i < N; i++)
		for (int j = 0; j < V; j++)
			d[i][j] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int i = 1; i <= n; i++)
	{
		for (int j = 0; j <= v; j++)
		{
			d[i][j] = d[i - 1][j];
			if (itemSpace[i] <= j)
				d[i][j] = std::max(d[i][j], 
					d[i - 1][j - itemSpace[i]] + itemWeight[i]);
		}
	}
	
	return d[n][v];
}

int SinglePackOptimized()
{
	int d[V];
	for (int i = 0; i < V; i++)
			d[i] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int i = 1; i <= n; i++)
	{
		for (int j = v; j >= itemSpace[i]; j--)
		{
			d[j] = std::max(d[j], d[j - itemSpace[i]] + itemWeight[i]);
		}
	}
	
	return d[v];
}

int main()
{
	std::cin >> n >> v;
	for (int i = 1; i <= n; i++)
		std::cin >> itemSpace[i] >> itemWeight[i];
	
	std::cout << SinglePack() << std::endl;
	return 0;
}
```

## 完全背包

和01背包类似，只是物品的数量变成了无限个。

由于物品由无限个，此时当前状态可以用于更新当前状态，因此将循环顺序改为从小到大即可。

### 为什么不需要考虑物品的数量

显然，考虑物品的数量可以正确解决问题，但时间复杂度升至$O(NM^2)$，实在是太慢了。

如果第$i$个物品可以装$s_i$个，那么$f_{i, j}$和$f_{i, j - v_i}$可以这样转移:

$$
f_{i, j} = \max(f_{i - 1, j}, f_{i - 1, j - v_i} + w_i, f_{i - 1, j - 2v_i} + 2w_i, \dots, f_{i - 1, j - s_i v_i} + s_i w_i)
$$
$$
f_{i, j - v_i} = \max(f_{i - 1, j - v_i}, f_{i - 1, j - 2v_i} + w_i, f_{i - 1, j - 3v_i} + 2w_i, \dots, f_{i - 1, j - s_i v_i} + (s_i - 1)w_i)
$$

发现$f_{i, j - v_i} + w_i$其实就是$f_{i, j}$转移中$f_{i - 1, j}$后面所有项的最大值，因此转移方程简化如下：

$$
f_{i, j} = \max(f_{i - 1, j}, f_{i, j - v_i} + w_i)
$$

再应用上面提到的空间优化，得到下面的代码

```cpp
#include <iostream>

const int N = 1005, V = 1005;

int n, v;
int itemSpace[N], itemWeight[N];

int FullPack()
{
	int d[N][V];
	for (int i = 0; i < N; i++)
		for (int j = 0; j < V; j++)
			d[i][j] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int i = 1; i <= n; i++)
	{
		for (int j = itemSpace[i]; j <= v; j++)
		{
			for (int k = 0; k * itemSpace[i] <= j; k++)
			{
				d[i][j] = std::max(d[i][j], 
					d[i][j - k * itemSpace[i]] + k * itemWeight[i]);
			}
		}
	}
	
	return d[n][v];
}

int FullPackOptimized()
{
	int d[N][V];
	for (int i = 0; i < N; i++)
		for (int j = 0; j < V; j++)
			d[i][j] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int i = 1; i <= n; i++)
	{
		for (int j = 0; j <= v; j++)
		{
			d[i][j] = d[i - 1][j];
			if (itemSpace[i] <= j)
				d[i][j] = std::max(d[i][j], 
					d[i][j - itemSpace[i]] + itemWeight[i]);
		}
	}
	
	return d[n][v];
}

int FullPackFullOptimized()
{
	int d[V];
	for (int i = 0; i < V; i++)
			d[i] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int i = 1; i <= n; i++)
	{
		for (int j = itemSpace[i]; j <= v; j++)
		{
			d[j] = std::max(d[j], d[j - itemSpace[i]] + itemWeight[i]);
		}
	}
	
	return d[v];
}

int main()
{
	std::cin >> n >> v;
	for (int i = 1; i <= n; i++)
		std::cin >> itemSpace[i] >> itemWeight[i];
	
	std::cout << FullPackOptimized() << std::endl;
	return 0;
}
```

## 多重背包

多重背包与完全背包类似，但限定了每种物品的数量。

首先这个问题可以用完全背包优化前的算法，但时间复杂度太高，我们可以考虑用二进制拼凑思想优化，即把多个物品合并成由2的非负整数次幂的几个物品组，然后做01背包，时间复杂度$O(M \log N)$

可以用优化完全背包的方式优化多重背包吗？我们尝试写出$f_{i, j}$到$f_{i, j - s_i v_ i}$的转移方程：

$$
f_{i, j} = \max(f_{i - 1, j}, f_{i - 1, j - v} + w, \dots, f_{i - 1, j - s_i v_i} + s_i w_i)
$$
$$
f_{i, j - v} = \max(f_{i - 1, j - v}, f_{i - 1, j - 2v} + w, \dots, f_{i - 1, j - (s_i + 1)v_i} + s_i w_i)
$$

发现$f_{i, j - v} + w$并不能像完全背包一样对齐，而是少了$f_{i - 1, j}$，多出了$f_{i - 1, j - (s_i + 1)v_i} + (s_i + 1)w_i$。以此类推，在不出边界的情况下，之后的$f_{i - 1, j - 2v}$直到$f_{i - 1, j - s_i v_i}$都是如此，就像是一个长度为$s$的窗口滑出。因此，要求每一个状态的最大值，只需要求它前面滑动窗口中状态的最大值。

要求滑动窗口的最大值，可以用单调队列。

## 几个问题优化方式的联系

我们发现，完全背包和多重背包优化的关键都在于把一类数的特征由一个数或一个更快找出这个特征的数据结构中的数等价替换。

## 背包问题简单变形

### 多维费用

[CH0206-4978-宠物小精灵之收服](http://noi.openjudge.cn/ch0206/4978/)

这个题就是典型的二维费用的背包问题，状态定义只需要增加一维表示第二种费用。但本题要注意，“当皮卡丘的体力小于等于0时，小智就必须结束狩猎（因为他需要给皮卡丘疗伤），而使得皮卡丘体力小于等于0的野生小精灵也不会被小智收服”，所以答案不是$f_{n, m}$而是$f_{n, m - 1}$，有些题库，包括我校校内题库，没有考虑这一点，导致正解不能AC。

AC代码：

```cpp
#include <iostream>

const int N = 1005, M = 505;

int n, m, k;
int f[N][M];

int main()
{
    std::cin >> n >> m >> k;
    for (int i = 1; i <= k; i++)
    {
        int ball, hp;
        std::cin >> ball >> hp;
        for (int j = n; j >= ball; j--)
            for (int t = m; t >= hp; t--)
                f[j][t] = std::max(f[j][t], f[j - ball][t - hp] + 1);
    }
    
    std::cout << f[n][m - 1] << " ";
    for (int i = 0; i <= m; i++)
        if (f[n][i] == f[n][m - 1])
        {
            std::cout << m - i << std::endl;
            break;
        }
    return 0;
}
```

### 多维价值

这个很少见，如果有要么考虑各种价值的优先级，要么考虑将多种价值转换为一种。

## 分组背包

在01背包的基础上，将物品划为多个组，要求每组至多取一个物品，总价值的最大值。

要解决分组背包问题，只需要在状态定义中加一维，表示组数即可，但要注意控制循环层级，第一层为组数（可以看作物品，因为每组只选一个），第二层体积，第三层组中物品。

示例代码：

```cpp
#include <iostream>

const int N = 105, V = 105, S = 105;

int n, v, s[N];
int itemSpace[S][N], itemWeight[S][N];

int GroupPackOptimized()
{
	int d[V];
	for (int i = 0; i < V; i++)
			d[i] = 0;
	
	// subsets of d[i, j]: a. has i; b. without i (d[i - 1, j]).
	for (int g = 1; g <= n; g++)
	{
		for (int j = v; j >= 0; j--)
		{
			for (int i = 0; i < s[g]; i++)
			{
				if (itemSpace[g][i] <= j)
				d[j] = std::max(d[j], d[j - itemSpace[g][i]] + itemWeight[g][i]);
			}
		}
	}
	
	return d[v];
}

int main()
{
	std::cin >> n >> v;
	for (int i = 1; i <= n; i++)
	{
		std::cin >> s[i];
		for (int j = 0; j < s[i]; j++)
		{
			std::cin >> itemSpace[i][j] >> itemWeight[i][j];
		}
	}
	
	std::cout << GroupPackOptimized() << std::endl;
	return 0;
}
```

## 有依赖关系的背包问题

这类问题有些可以通过转化成为分组背包问题，有一些只能通过树形动态规划解决。下面这一题可以转化为分组背包问题：

[洛谷-P1064-NOIP2006提高组 金明的预算方案](https://www.luogu.com.cn/problem/P1064)

由于本题附件最多有两个，可以拆分成不同选法，放在一个组里。AC代码：

```cpp
#include <iostream>
#include <vector>

const int N = 65, M = 32005;

struct Item
{
    Item(int v, int w) : v(v), w(w) {}
    int v;
    int w;
};

int m, n;
std::vector<Item> items[N];
int f[M];

int main()
{
    std::cin >> m >> n;
    int v, w, dep;
    for (int i = 1; i <= n; i++)
    {
        std::cin >> v >> w >> dep;

        int len = items[dep].size();
        if (dep == 0)
            items[i].push_back(Item(v, v * w));
        else
            for (int j = 0; j < len; j++)
                items[dep].push_back(Item(items[dep][j].v + v, items[dep][j].w + v * w));
    }

    for (int i = 1; i <= n; i++)
        for (int j = m; j >= 0; j--)
        {
            int len = items[i].size();
            for (int k = 0; k < len; k++)
                if (j - items[i][k].v >= 0)
                    f[j] = std::max(f[j], f[j - items[i][k].v] + items[i][k].w);
        }

    std::cout << f[m] << std::endl;
    return 0;
}
```

更复杂的情况需要考虑树形动态规划。
