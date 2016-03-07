---
title: 数据结构
layout: page
date: 2016-02-23
---
[TOC]

## 栈
###相关应用
1. 括号匹配。
2. 中缀转后缀表达式

        进行中缀转后缀操作，使用符号栈存储遍历过程中遇到的符号。
        从左到右遍历中缀表达式，遇到数字直接输出，遇到符号准备将其放入符号栈：
        若符号栈为空，直接放入。
        若当前符号大于栈顶符号优先级，直接放入。
        如果小于等于栈顶符号优先级，将栈顶符号弹出，直到栈顶符号小于当前符号，再将当前符号入栈。
        如果当前符号为“（”，直接入栈。
        如果当前符号为“）”，依次将符号栈的符号弹出，直到找到“（”。
        按此规则进行遍历，最后如果符号栈仍有符号，弹出即可。

3. 后缀表达式计算

        遇到数字入栈
        遇到符号：弹出两个数字，计算后把结果存入栈

## 树
### 基本概念
- **path**：node n1 to nk is defined as a sequence of nodes n1, n2, . . . , nk such that ni is the parent of ni+1 for 1 ≤ i < k.
- **length**：The length of this path is the number of edges on the path, namely k − 1.
- **depth**：For any node ni, the depth of ni is the length of the unique path from the root to ni. The depth of a tree is equal to the depth of the deepest leaf.
- **height**：The height of ni is the length of the longest path from ni to a leaf. Thus all leaves are at height 0. The height of a tree is equal to the height of the root.
- Conventionally, an empty tree (tree with no nodes, if such are allowed) has depth and height −1.

上面的定义来自《Data Structures and Algorithm Analysis》，下面的定义来自国内教材。
树的深度是从根节点开始（其深度为1）自顶向下逐层累加的，而高度是从叶节点开始（其高度为1）自底向上逐层累加的。虽然树的深度和高度一样，但是具体到树的某个节点，其深度和高度是不一样的。

也就是说根节点的深度，叶子节点的高度有两种不同的初始值，有些定义为0，有些定义为1。

### 二叉树
- **满二叉树**：所有节点都有0或者2个孩子。
- **完全二叉树**：除了最后一层从左到右填充，其他层都是完全满的。
    完全二叉树的性质：n = 2^h+1 - 1 (此时叶子节点的高度定义为0)

#### 应用
- 表达式树：通过栈构建表达式树

### 哈夫曼树

    1. 为每个符号建立一个叶子节点，并加上其相应的发生频率
    2. 当有一个以上的节点存在时，进行下列循环:
        1. 把这些节点作为带权值的二叉树的根节点，左右子树为空
        2. 选择两棵根结点权值最小的树作为左右子树构造一棵新的二叉树，且至新的二叉树的根结点的权值为其左右子树上根结点的权值之和。
        3. 把权值最小的两个根节点移除
        4. 将新的二叉树加入队列中.
    3. 最后剩下的节点暨为根节点，此时二叉树已经完成。

### 二叉搜索树
左子树小于右子树.
二叉查找树的一般操作的执行时间为O(lgn)。但二叉查找树若退化成了一棵具有n个结点的线性链后，则这些操作最坏情况运行时间为O(n)。
### AVL 树
即自平衡的二叉搜索树。AVL树中任何节点的两个子树的高度最大差别为 1，这样可以使二叉搜索的深度保持 log(n) 的深度，这样节点分布相对均匀，可以使搜索的时间复杂度是 log(n)

#### 旋转
有四种情况导致 AVL 树失衡：

1. An insertion into the left subtree of the left child of α. (LL)
2. An insertion into the right subtree of the left child of α. (RL)
3. An insertion into the left subtree of the right child of α. (LR)
4. An insertion into the right subtree of the right child of α. (RR)

其中，LL，RR只需要一次旋转，LR，RL 需要两次旋转

### 红黑树
红黑树，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。
通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出俩倍，因而是接近平衡的。可以在O(log n)时间内做查找，插入和删除。

红黑树是 2-3 树的一种等同。换句话说，对于每个 2-3 树，都存在至少一个数据元素是同样次序的红黑树。在 2-3 树上的插入和删除操作也等同于在红黑树中颜色翻转和旋转。这使得 2-3 树成为理解红黑树背后的逻辑的重要工具，这也是很多介绍算法的教科书在红黑树之前介绍 2-3 树的原因，尽管 2-3 树在实践中不经常使用。

红黑树与 2-3 树的等价定义：
- Red links lean left.
- No node has two red links connected to it.
- The tree has perfect black balance : every path from the root to a null link has the
same number of black links

1. 如果插入一个node引起了树的不平衡，AVL和RB-Tree都是最多只需要2次旋转操作，即两者都是O(1)；但是在删除node引起树的不平衡时，最坏情况下，AVL需要维护从被删node到root这条路径上所有node的平衡性，因此需要旋转的量级O(logN)，而RB-Tree最多只需3次旋转，只需要O(1)的复杂度。
2. 其次，AVL的结构相较RB-Tree来说更为平衡，在插入和删除node更容易引起Tree的unbalance，因此在大量数据需要插入或者删除时，AVL需要rebalance的频率会更高。因此，RB-Tree在需要大量插入和删除node的场景下，效率更高。自然，由于AVL高度平衡，因此AVL的search效率更高。
3. map的实现只是折衷了两者在search、insert以及delete下的效率。总体来说，RB-tree的统计性能是高于AVL的。

另一种红黑树定义：
性质1. 节点是红色或黑色。
性质2. 根是黑色。
性质3. 所有叶子都是黑色（叶子是NIL节点）。
性质4. 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
性质5. 从任一节点到其每个叶子的所有简单路径 都包含相同数目的黑色节点。

### B-tree
According to Knuth's definition, a B-tree of order m is a tree which satisfies the following properties:

- Every node has at most m children.
- Every non-leaf node (except root) has at least [m/2] children.
- The root has at least two children if it is not a leaf node.
- A non-leaf node with k children contains k−1 keys.
- All leaves appear in the same level

### B+-tree
B+-tree ：是应文件系统所需而产生的一种B-tree的变形树。

A B+ tree can be viewed as a B-tree in which each node contains only keys (not key-value pairs), and to which an additional level is added at the bottom with linked leaves.

**为什么说B+-tree比B 树更适合实际应用中操作系统的文件索引和数据库索引？**

1. B+-tree的磁盘读写代价更低

    B+-tree的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。
    举个例子，假设磁盘中的一个盘块容纳16bytes，而一个关键字2bytes，一个关键字具体信息指针2bytes。一棵9阶B-tree(一个结点最多8个关键字)的内部结点需要2个盘快。而B+ 树内部结点只需要1个盘快。当需要把内部结点读入内存中的时候，B 树就比B+ 树多一次盘块查找时间(在磁盘中就是盘片旋转的时间)。

2. B+-tree的查询效率更加稳定

    由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

**B+-tree的应用:** VSAM(虚拟存储存取法)文件


## 哈希

### 冲突解决
1. 链接法（开散列）：使用链表存储相同哈希值的值
2. 开发定址法（闭散列）
    1. 线性探测散列：消除冲突，产生一次聚集
    2. 二次探测散列：消除一次聚集，产生二次聚集
    3. 双重哈希

## 优先队列
### 二叉堆（基于数组）
从 1 下标开始，对于每个 i，左孩子是 2i，右孩子是 2i+1，父节点是 i/2。

插入通过上滤调整堆，删除通过下滤调整堆。

### d 叉堆

### 左式堆

## 图
实现：邻接链表，邻接矩阵

### 拓扑排序
- Kahn算法

    1. 从 DAG 图中选择一个 没有前驱（即入度为0）的顶点并输出。
    2. 从图中删除该顶点和所有以它为起点的有向边。
    3. 重复 1 和 2 直到当前的 DAG 图为空或当前图中不存在无前驱的顶点为止。后一种情况说明有向图中必然存在环。

- DFS

        L ← Empty list that will contain the sorted nodes
        S ← Set of all nodes with no outgoing edges
        for each node n in S do
            visit(n)
        function visit(node n)
            if n has not been visited yet then
                mark n as visited
                for each node m with an edge from m to ndo
                    visit(m)
                add n to L

### 最短路径
- Dijkstra算法

    dijkstra算法用来计算从一个点到其他所有点的最短路径的算法，复杂度O(N2)。

- Floyd算法

    floyd算法是最简单的最短路径算法，可以计算图中任意两点间的最短路径  folyd算法的时间复杂度是O(N3),如果是一个没有边权的图，把相连的两点  间的距离设为dist[i][j] = 1，不相连的两点设为无穷大，用 floyd算法可以判断i,j两点是否有路径相连。

### 网络流问题

### 最小生成树
- Prim 算法

    1. 清空生成树，任取一个顶点加入生成树
    2. 在那些一个端点在生成树里，另一个端点不在生成树里的边中，选取一条权最小的边，将它和另一个端点加进生成树
    3. 重复步骤2，直到所有的顶点都进入了生成树为止，此时的生成树就是最小生成树

- Kruskal算法

    构造一个只含n个顶点，而边集为空的子图，若将该子图中各个顶点看成是各棵树的根节点，则它是一个含有n棵树的森林 。之后，从网的边集中选取一条权值最小的边，若该边的两个顶点分属不同的树 ，则将其加入子图，也就是这两个顶点分别所在的 两棵树合成一棵树；反之，若该边的两个顶点已落在同一棵树上，则不可取，而应该取下一条权值最小的边再试之。依次类推，直至森林只有一棵树。