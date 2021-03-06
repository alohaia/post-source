---
title: 基于 R 的数学建模
comments: true
mathjax: false
date: 2021-09-07 17:08:09
tags:
    - R
    - 数学建模
categories:
    - [Learning, Computer, R]
    - [Projects, Contests, 数学建模大赛]
---

主要参考薛毅编著的《数学建模‧基于 R》。

<!-- more -->

# 概率统计模型

# 线性规划

R 提供了许多可以用来解决线性规划问题的包，这里我们使用 lpSolve。



## 普通线性规划问题

使用 lpSolve 提供的 `lp()` 函数可求解一般线性规划问题：

```r
lp (direction = "min", objective.in, const.mat, const.dir, const.rhs,
    transpose.constraints = TRUE, int.vec, presolve=0, compute.sens=0,
    binary.vec, all.int=FALSE, all.bin=FALSE, scale = 196, dense.const,
    num.bin.solns=1, use.rw=FALSE)
```

## 运输问题和最优指派问题

运输问题的目标一般可表示为

$$\min(\max)\quad z=\sum\limits_{i=1}^n\sum\limits_{j=1}^mc_{ij}x_{ij}
\,.$$

可以用这一点来决定一个问题是否为运输问题。

### 运输问题

使用 `lp.transport()` 函数可求解一般运输问题：

```r
lp.transport (cost.mat, direction="min", row.signs, row.rhs, col.signs,
              col.rhs, presolve=0, compute.sens=0, integers = 1:(nc*nr) )
```

{% note primary 示例 %}

```r
z <- matrix(
    c(
        5, 6, 7, 8,
        1e30, 5, 6, 7,
        1e30, 1e30, 6, 7,
        1e30, 1e30, 1e30, 6
    ),
    nrow = 4, ncol = 4, byrow = TRUE
)
t.row <- c(14, 15, 15, 13)
t.col <- c(10, 14, 20, 8)
trans_sol <- lp.transport(
    cost.mat = z,
    row.signs = rep("<=", 4), row.rhs = t.row,
    col.signs = rep("==", 4), col.rhs = t.col,
    integers = 0
)
trans_sol
trans_sol$solution
```

```
#> Success: the objective function is 292 
#>      [,1] [,2] [,3] [,4]
#> [1,]   10    0    4    0
#> [2,]    0   14    1    0
#> [3,]    0    0   15    0
#> [4,]    0    0    0    8
```
{% endnote %}

### 最优指派问题

最优指派问题是运输问题的特殊情况，可使用 `lo.assign()` 函数求解：

```r
lp.assign (cost.mat, direction = "min", presolve = 0, compute.sens = 0)
```

# 最优化模型

这里讨论的最优化模型本质上是非显性最优化问题，即非线性规划问题。

一般形式的**约束最优化问题**的数学模型为：

$$\begin{aligned}
\min\quad  & f({\bf x}),\ {\bf x}=(x_1,x_2,\cdots,x_n)^T\in{\bf R}^n\,.\\
s.\,t\quad & c_i({\bf x})=0,\ i\in{\rm E}=\{1,2,\cdots,l\}\,,\\
            & c_i({\bf x})\ge0,\ i\in{\rm I}=\{l+1,l+2,\cdots,l+m\}\,.
\end{aligned}$$

将后面的约束条件去掉，即为**无约束最优化问题**的数学模型。

## 最优化问题的求解

R 中求解最优化问题的可用函数。

### 一元函数求极值

R 中的 `optimize()`（或 `optimise()` 函数）为一元函数极值函数，可求

$$\min(\max)\quad f(x),\ x\in[a,b]
\,.$$

使用格式：

```r
optimize(f, interval, ..., lower = min(interval), upper = max(interval),
         maximum = FALSE,
         tol = .Machine$double.eps^0.25)
```

`interval` 指定寻找极小值的范围，如 `c(-10,9)`，`lower` 和 `upper` 默认由 `interval` 提供。`maximum` 指明是否是求最大值，默认为 `FALSE`（求最小值）。

{% note primary 示例 %}

```r
f <- function(x) x^3 - 2 * x - 5
optimize(f, interval = c(0, 2))
```

```
#> $minimum
#> [1] 0.8165
#> 
#> $objective
#> [1] -6.089
```
{% endnote %}

### 多元无约束问题

#### `nlm()` 函数

“nlm” 是 Non-Linear Minimization 的缩写，即非线性函数的极小值

$$\min\quad f({\bf x}),\ {\bf x}\in{\bf R}^n
\,.$$

使用格式：

```r
nlm(f, p, ..., hessian = FALSE, typsize = rep(1, length(p)),
    fscale = 1, print.level = 0, ndigit = 12, gradtol = 1e-6,
    stepmax = max(1000 * sqrt(sum((p/typsize)^2)), 1000),
    steptol = 1e-6, iterlim = 100, check.analyticals = TRUE)
```

{% tablecaption %}nlm() 函数的返回值{% endtablecaption %}

| 元素         | 说明                                                            |
|--------------|-----------------------------------------------------------------|
| `minimum`    | 极小值                                                          |
| `estimate`   | 极小点的估计值                                                  |
| `gradient`   | 极小点处的梯度值                                                |
| `hessian`    | 最优点处的 Hesse 矩阵，需要参数 `hessian = TRUE`                |
| `code`       | 编码，简单说，1、2 表示结果为最优点，3、4、5 表示可能不是最优点 |
| `iterations` | 求解所需的迭代次数                                              |

{% note primary 示例 %}

求 Rosenbrock 函数的极小点：


```r
obj <- function(x) {
    f <- c(10 * (x[2] - x[1]^2), 1 - x[1])
    sum(f^2)
}
nlm(obj, c(-1.2, 1))
```

```
#> $minimum
#> [1] 3.974e-12
#> 
#> $estimate
#> [1] 1 1
#> 
#> $gradient
#> [1] -6.539e-07  3.336e-07
#> 
#> $code
#> [1] 1
#> 
#> $iterations
#> [1] 23
```

在目标函数中使用 `attr()` 添加梯度函数：


```r
obj <- function(x) {
    f <- c(10 * (x[2] - x[1]^2), 1 - x[1])
    g <- function(x, f) {
        j <- matrix(c(-20 * x[1], 10, -1, 0), 2, 2, byrow = TRUE)
        return(2 * t(j) %*% f)
    }
    result <- t(f) %*% f
    attr(result, "gradient") <- g(x, f)
    return(result)
}
nlm(obj, c(-1.2, 1))
```

```
#> $minimum
#> [1] 1.182e-20
#> 
#> $estimate
#> [1] 1 1
#> 
#> $gradient
#> [1]  2.584e-09 -1.201e-09
#> 
#> $code
#> [1] 1
#> 
#> $iterations
#> [1] 24
```
{% endnote %}

# 图论与网格模型

使用 `igraph` 包画图。



{% note primary 示例 %}

1. 柯尼斯堡七桥问题：

```r
e <- c(1, 2, 1, 2, 1, 4, 1, 4, 1, 3, 2, 3, 4, 3)
g1 <- graph(e, directed = FALSE) %>%
    set_vertex_attr("name", value = LETTERS[1:4])
e_names <- paste("e", 1:7, sep = "")
plot(g1, layout = layout.circle, edge.label = e_names)
```

{% asset_img "柯尼斯堡七桥问题-1.png" "柯尼斯堡七桥问题'柯尼斯堡七桥问题'" %}
2. 使用 `graph.formula()` 构建完全图 K~6~：

```r
plot(graph.formula(A:B:C:D:E:F:G:H:I:J:K:L:M:N:O:P:Q:R:S:T -
    A:B:C:D:E:F:G:H:I:J:K:L:M:N:O:P:Q:R:S:T), layout = layout.circle)
```

{% asset_img "完全图 K20-1.png" "完全图 K20'完全图 K20'" %}
3. 完全偶图：

```r
plot(graph.full.bipartite(10, 10), layout = layout.circle)
```

{% asset_img "完全偶图-1.png" "完全偶图'完全偶图'" %}
{% endnote %}

使用 `graph.adjacency()`（或 `graph_from_adjacency_matrix()`）可以构建赋权图：

```r
graph_from_adjacency_matrix(
    adjmatrix,
    mode = c("directed", "undirected", "max", "min", "upper", "lower", "plus"),
    weighted = NULL,
    diag = TRUE,
    add.colnames = NULL,
    add.rownames = NA
)
```

其他函数：
1. `degree()` 计算图的**度**序列
2. `graph.strength()` 计算图的顶点**强度**（连接顶点各边的权和）序列
3. `is.degree.sequence()` 或 `is.graphical.degree.sequence()` 判断是否存在顶点度符合给定序列的图
4. `is.connected()` 检查图是否连通
5. `clusters()` 将图分为连通的子图（聚类）

## 最短路问题


```r
edges <- c(
    1, 2, 1, 3, 2, 4, 2, 5, 3, 5, 3, 8, 3, 9, 4, 5, 4, 6,
    5, 6, 6, 7, 7, 8, 7, 9, 7, 10, 9, 10
)
weigets <- c(
    122, 359, 345, 167, 180, 195, 246, 443, 415,
    92, 213, 79, 199, 163, 215
)
g <- graph.empty(directed = F) + vertices(1:10) + edges(edges)
sp <- shortest.paths(g, v = 3, weights = weigets)
gs <- get.shortest.paths(g, from = 3, weights = weigets)
```

## 最优连线问题

`igraph` 中，`minimum.spanning.tree()` 可以求最优生成树，其使用格式为：

```r
minimum.spanning.tree(graph, weights = NULL, algorithm = NULL, ...)
```

