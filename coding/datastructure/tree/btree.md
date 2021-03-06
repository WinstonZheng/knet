# B树
# 概述
B树，是二叉查找树的扩展，子节点的数量可以多余两个。具体的划分为两种：B(B-)树和B+树。

## B(B-)树
一个m阶的B树满足性质如下：
1. 每个结点至多拥有m棵子树；
2. 根结点至少拥有两颗子树（存在子树的情况下）；
3. 除了根结点以外，其余每个分支结点至少拥有 m/2 棵子树；
4. 所有的叶结点都在同一层上；
5. 有 k 棵子树的分支结点则存在 k-1 个关键码，关键码按照递增次序进行排列；
6. 关键字数量需要满足ceil(m/2)-1 <= n <= m-1；

### 操作
具体操作类似于B+树中的分裂与合并操作。

## B+树
1. 数据项存储在叶子节点上；
2. 非叶子节点存储直到M-1关键字以指示搜索方向，关键字i代表子树i+1总最小的关键字；
3. 树的根或者一片树叶，或者其儿子数在2-M之间；
4. 除根外，所有非树叶节点的儿子数在(M+1)/2（取上界）和M之间；
5. 所有的树叶都在相同的深度上并由(L+1)/2(上界)和L之间个数据项。

### 操作
- 添加项<br>
如果子节点满，分裂叶子节点，在父节点中添加新的关键字；如果父节点也满了，则在祖父节点添加新的关键字...（递归向上），根节点满了，分裂根节点，新建新的节点作为根，分裂的旧的根节点作为子树，高度增加。

> 其他方式处理儿子过多，其中一种是领养机制，将多出的儿子节点交给相邻节点进行领养，并修改关键字。（可以使节点更饱满）

- 删除项<br>
删除某个叶子节点的数据，如果该叶子节点存储的容量达到最小项的要求，则通过领养机制从相邻节点中未达到最小项数的叶子节点领取数据，如果相邻节点达到最小值，则通过合并机制将叶子节点进行合并操作。


## B+/-树比较
- 结构上
    - B树中关键字集合分布在整棵树中，叶节点中不包含任何关键字信息，而B+树关键字集合分布在叶子结点中，非叶节点只是叶子结点中关键字的索引；
    - B树中任何一个关键字只出现在一个结点中，而B+树中的关键字必须出现在叶节点中，也可能在非叶结点中重复出现；
    
- 性能上，为什么B+树更适合作为索引（操作系统和数据库）
    - 不同于B树只适合随机检索，B+树同时支持随机检索和顺序检索；（B+树数据在叶子节点顺序存储，可以通过链接的方式将叶子连起来）
    - B+树的磁盘读写代价更低。B+树的内部结点并没有指向关键字具体信息（存储的数据项）的指针，其内部结点比B树小，盘块能容纳的结点中关键字数量更多，一次性读入内存中可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素。
    - B+树的查询效率更加稳定。B树搜索有可能会在非叶子结点结束，越靠近根节点的记录查找时间越短，只要找到关键字即可确定记录的存在，其性能等价于在关键字全集内做一次二分查找。而在B+树中，顺序检索比较明显，随机检索时，任何关键字的查找都必须走一条从根节点到叶节点的路，所有关键字的查找路径长度相同，导致每一个关键字的查询效率相当。
    - （数据库索引采用B+树的主要原因是，）B-树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。B+树的叶子节点使用指针顺序连接在一起，只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作（或者说效率太低）。

## 应用


# Reference
- 《数据结构与算法描述 Java语言描述》
- [B树和B+树的总结](https://www.cnblogs.com/George1994/p/7008732.html)
- [B+树比B树更适合做文件索引的原因](https://blog.csdn.net/mine_song/article/details/63251546)