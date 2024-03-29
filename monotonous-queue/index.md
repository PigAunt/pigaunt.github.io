# 单调队列


## 滑动窗口问题

[洛谷-P1886-滑动窗口/【模板】单调队列](https://www.luogu.com.cn/problem/P1886)

有一个长为$n$的序列$a$，以及一个大小为$k$的窗口。现在这个从左边开始向右滑动，每次滑动一个单位，求出每次滑动后窗口中的最大值和最小值。

如果每次都遍历整个窗口，时间复杂度为$O(nk)$，相当高。但其中我们进行了很多无效的比较，比如已知一个数在窗口中小于另一个，进入下一个窗口时还在进行比较。

为了解决这个问题，我们引入一种数据结构————单调队列。

## 单调队列的原理和性质

既然叫“单调”队列，单调队列中的元素体现了以下的单调性：

1. 元素排列具有单调性（增/减/其他）
2. 取出元素的位置组成的序列具有单调性（不会倒回去取）

注意：单调队列只能同时体现一种单调性。

如何实现上面提到的单调性？单调队列采用了如下的方法：

1. 如果新加入的元素满足单调性，从尾部入队
2. 如果新加入的元素比队列中所有元素都大/小/其他，清空队列，把它请进来
3. （满足滑动窗口问题的条件）如果元素过期（在窗口外），从头部出队

由于严格保证了单调性，最值可以直接在队头取。

示例代码：

```cpp
#include <iostream>
#include <queue>

const int N = 1e6 + 5;

int n, k, a[N];

void GetMin()
{
    std::deque<int> q;
    for (int i = 1; i <= n; i++)
    {
        if (!q.empty() && i - k >= q.front())
            q.pop_front();
        while (!q.empty() && a[i] <= a[q.back()])
            q.pop_back();
        q.push_back(i);
        
        if (i >= k)
            std::cout << a[q.front()] << " ";
    }
    std::cout << std::endl;
}

void GetMax()
{
    std::deque<int> q;
    for (int i = 1; i <= n; i++)
    {
        if (!q.empty() && i - k >= q.front())
            q.pop_front();
        while (!q.empty() && a[i] >= a[q.back()])
            q.pop_back();
        q.push_back(i);
        
        if (i >= k)
            std::cout << a[q.front()] << " ";
    }
    std::cout << std::endl;
}

int main()
{
    std::cin >> n >> k;
    for (int i = 1; i <= n; i++)
        std::cin >> a[i];
    
    GetMin();
    GetMax();
    return 0;
}
```
