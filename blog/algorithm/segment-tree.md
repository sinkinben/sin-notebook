## 线段树 (Segment Tree)

预备知识：[树状数组](https://www.cnblogs.com/sinkinben/p/14282913.html) 。

与树状数组 (Binary Index Tree, BIT, aka "二叉索引树") 类似，线段树适用于以下场景：

> 给定数组 `a[n]`， 并且要求 `w` 次修改数组，现有 `q` 次区间查询，每次区间查询包括 `[l, r]` 2 个参数，要求返回 `sum(a[l, r])` 的值。

如果没有「修改元素」的要求，显然用前缀和是最好的。既然树状数组能解决上述场景，那么线段树比树状数组好在哪里呢？

- 线段树可以在 $O(\log{n})$ 的时间复杂度内实现单点修改、**区间修改**、区间查询（区间求和，求区间最大值，求区间最小值）等操作。
- 树状数组适用于单点修改和区间查询。

显然，线段树比树状数组能力更强，树状数组能做到的，线段树同样能做到。



## 结构

线段树的本质是一棵基于数组表示的二叉树。

假设有一个大小为 5 的数组 $a[5] = \{10, 11, 12, 13, 14\}$ ，记线段树为 $d[n]$ ，**下标均从 1 开始计数**。那么该线段树的形态如下：

```text
                  [1, 5]
               /          \
          [1,3]             [4,5]
         /     \           /     \
      [1,2]    [3,3]   [4,4]     [5,5]
     /     \
[1,1]       [2,2]
// 线段树的性质：叶子节点是数组元素
```

每个节点表示一个区间（亦即所谓的「线段」），线段树 $d_i$ 记录的是这一区间的和：

```text
d[1] = sum([1, 5]) = 60
d[2] = sum([1, 3]) = 33
d[3] = sum([4, 5]) = 27
d[4] = sum([1, 2]) = 21
d[5] = sum([3, 3]) = 12
d[6] = sum([4, 4]) = 13
d[7] = sum([5, 5]) = 14
d[8] = sum([1, 1]) = 10
d[9] = sum([2, 2]) = 11
```

前面提到，线段树的本质是基于数组表示的二叉树，那么节点 $d_i$ 的左右孩子节点分别为 $d_{2i}, d_{2i+1}$, 且根据线段树的形态，那么我们有：
$$
d_i = d_{2i} + d_{2i+1}
$$
根据这一递推公式，显然可以得知，线段树建立和修改是「自底向上」的。



## 建树

建立线段树的第一个问题：给定数组 $a[n]$ ，那么线段树需要多大的空间？

> **分析**
>
> - 线段树是一棵完全二叉树，其高度为 $h = \lceil \log{n} \rceil$，高度从 0 开始计数，这意味着该二叉树有 $h+1$ 层。
> - 考虑最坏情况，线段树是一棵满二叉树，那么有 $2^{h+1} - 1$ 个节点。
>
> $$
> 2^{h+1} - 1 \le 2^{h+1} \le 2^{\log{n} + 2} \le 2^{\log{4n}} = 4n
> $$
>
> - 因此，线段树的空间一般开 $4n$ 大小。当然，调数学库精确算一下也是可以的 (if you want) 。

假设我们当前节点 $d_i$ 表示的是区间 $[s, t]$ 之和，那么其左节点 $d_{2i}$ 表示的是区间 $[s,\frac{s+t}{2}]$ ，其右节点 $d_{2i+1}$ 表示的是区间 $[\frac{s+t}{2}+1, t]$ .

建树操作如下：

```cpp
const int n = 5;
int nums[n + 1] = {0, 10, 11, 12, 13, 14};
vector<int> d;
void build(int s, int t, int idx)
{
    if (s == t)
    {
        d[idx] = nums[s];
        return;
    }
    int m = s + (t - s) / 2, l = 2 * idx, r = 2 * idx + 1;
    build(s, m, l), build(m + 1, t, r);
    d[idx] = d[l] + d[r];
}
int main()
{
    d.resize(4 * n, 0);
    build(1, n, 1);
}
```



## 区间查询

区间查询，即求出区间 $[l,r]$ 之和，或者求出区间的最大值、最小值等操作。

依旧以这个树为例子：

```text
                  [1, 5]
               /          \
          [1,3]             [4,5]
         /     \           /     \
      [1,2]    [3,3]   [4,4]     [5,5]
     /     \
[1,1]       [2,2]
```

如果区间 $[l, r]$ 是线段树上的一个节点，那么这种查询是简单的，直接获取 $d_1 = 60$ 即可。那如果要查询的区间不是对应于某个节点（即横跨若干节点），查询是怎么做到的呢？以查询区间 $[3, 5]$ 为例，可以拆分为 $[3,3]$ 和 $[4, 5]$ 这 2 个节点。

一般地，查询区间为 $[l, r]$ ，根据线段树的性质，最多可拆分为 $\log{n}$ 个子区间（这些子区间要求是一个极大的子区间，即 $[4,5]$ 不用再进一步拆分为 $[4,4]$ 和 $[5,5]$ ）。

```cpp
// [l, r] 为查询区间, [s, t] 为当前节点包含的区间
// idx 代表线段树的节点
int getsum(int l, int r, int s, int t, int idx)
{
    if (l <= s && t <= r)
        return d[idx];
    int sum = 0;
    int m = s + (t - s) / 2;
    if (l <= m) // 如果 [s, m] 与目标区间 [l, r] 有交集
        sum += getsum(l, r, s, m, 2 * idx);
    if (m < r)  // 如果 [m+1, t] 与目标区间 [l, r] 有交集
        sum += getsum(l, r, m + 1, t, 2 * idx + 1);
    return sum;
}
```



## 区间修改

如果要对 `nums[i]` 的值修改，那么线段树需要做出调整，任何包含 `nums[i]` 的区间都需要修改，复杂度是 $O(\log{n})$ .

如果要对区间 $[a, b]$ 内数组元素做修改，显然，在线段树中，任何包括了 `nums[a ... b]` 中某一元素的区间都要修改一次，这时候复杂度为 $O((b-a+1) \cdot \log{n})$ ，这个复杂度属实有点蚌埠住 😅 。

因此需要引入「惰性标记」，简单来说，就是空间换时间。

> 懒惰标记，简单来说，就是通过**延迟对节点信息的更改**，从而减少可能不必要的操作次数。每次执行修改时，我们通过打标记的方法表明该节点对应的区间在某一次操作中被更改，但**不更新该节点的子节点**的信息。实质性的修改则在下一次访问带有标记的节点时才进行。

如下图所示，给线段树的每个节点都加入一个标记值 `t[i]` ，表示这一区间的值的变化。

<img src = "https://gitee.com/sinkinben/pic-go/raw/master/img/20210816210434.png" style="width:60%; border-radius:0px;" />

如果想要给区间 $[3,5]$ 的每个数都加上一个值 `val = 5` ，那么会找到子区间 $[3,3]$ 和 $[4, 5]$ ，修改它们的标记值。注意，下图的线段树 `d[i]` 修改为节点的值（即区间之和）。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210821193401.png" style="width:60%; border-radius:0px;" />

<p style="text-align: center; font-size: 12px;"><span style="color: red">* </span>注：此处应为 <code style="background: none; font-size: 12px !important;">d[5] = 12 + 5 = 17, d[3] = 27 + 2 * 5 = 37</code></p>

虽然节点 `d[3]` 节点被修改，但它的孩子节点并没有修改（所谓的「延迟更改」）。同时需要注意，`d[3]` 真正的变化值为 `5 * 2 = 10` .

接下来，如果我们需要查找区间 $[4,4]$ 之和，那么会遍历到 $d_3 = [3,4]$ 这一区间。当发现这一节点存在不为 0 的标记值时，此时需要真正地更新其孩子，将标记值「下放」，并重置标记值为 0 。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210821194010.png" style="width:60%; border-radius:0px;" />

<p style="text-align: center; font-size: 12px;"><span style="color: red">* </span>注：与上同理，此处应为 <code style="background: none; font-size: 12px !important;">d[5] = 17, d[3] = 37, d[6] = 18, d[7] = 19</code></p>

**带惰性标记的区间修改**

```cpp
// [l, r] 为修改的目标区间, [s, t] 为当前节点包含的区间
// idx 代表线段树的节点
// val 为变化值
void update(int l, int r, int s, int t, int idx, int val)
{
    // [s, t] 为修改区间 [l, r] 的子集时
    // 直接修改当前节点的值, 然后打标记
    if (l <= s && t <= r)
    {
        d[idx] += (t - s + 1) * val;
        vals[idx] += val;
        return;
    }
    int m = s + (t - s) / 2;
    int lchild = 2 * idx, rchild = 2 * idx + 1;
    // 如果不是叶子节点, 并且存在标记
    if (vals[idx] && s != t)
    {
        // 标记下放并重置
        // 注意标记下放使用 +=, 因为左右孩子可能存在非零标记
        d[lchild] += vals[idx] * (m - s + 1), vals[lchild] += vals[idx];
        d[rchild] += vals[idx] * (t - m), vals[rchild] += vals[idx];
        vals[idx] = 0;
    }
    if (l <= m) update(l, r, s, m, lchild, val);
    if (r > m)  update(l, r, m + 1, t, rchild, val);
}
```



**带惰性标记的区间查询**

```cpp
// [l, r] 为查询区间, [s, t] 为当前节点包含的区间
// idx 代表线段树的节点
int getsum(int l, int r, int s, int t, int idx)
{
    if (l <= s && t <= r)
        return d[idx];
    int sum = 0;
    int m = s + (t - s) / 2;
    int lchild = 2 * idx, rchild = 2 * idx + 1;
    // 如果存在非零标记
    if (vals[idx])
    {
        d[lchild] += (m - s + 1) * vals[idx], vals[lchild] += vals[idx];
        d[rchild] += (t - m) * vals[idx], vals[rchild] += vals[idx];
        vals[idx] = 0;
    }
    if (l <= m) // 如果 [s, m] 与目标区间 [l, r] 有交集
        sum += getsum(l, r, s, m, 2 * idx);
    if (m < r) // 如果 [m+1, t] 与目标区间 [l, r] 有交集
        sum += getsum(l, r, m + 1, t, 2 * idx + 1);
    return sum;
}
```



模版例题：

- [洛谷 P3372](https://www.luogu.com.cn/problem/P3372)
- [洛谷 P3373](https://www.luogu.com.cn/problem/P3373)



## 参考

- [1] https://oi-wiki.org/ds/seg/
- [2] https://www.cnblogs.com/sinkinben/p/14282913.html