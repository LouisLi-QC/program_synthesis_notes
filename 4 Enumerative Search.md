# 4 Enumerative Search  枚举搜索算法

基于枚举搜索的合成技术已被证明是在<u>丰富复杂假设空间</u>中合成小程序最有效的技术之一。枚举技术的核心思想是首先利用程序的大小、复杂度等指标来构造假设空间，然后对空间中的程序通过修剪进行枚举，从而有效地搜索满足规范要求的程序。枚举算法通常被设计成对无限假设空间具有<u>半可定性</u>，但它们通常适用于几乎所有类型的假设空间和规范约束。

## 4.1 Enumerative Search

介绍一种简单的假设空间枚举合成算法，该算法使用CFG(类似于SyGuS)定义，但该算法可以很容易地扩展到其他形式的假设空间，如 partial programs, context-sensitive languages等。

定义一个条件线性整数算术表达式的空间：

![image-20210501123232884](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210501123232884.png)

让假设空间被定义为a context-free grammar G = (V,Σ(R, S),  其中

V表示the set of non-terminals in the grammar  比如上图中的 S 和 B 就是non-terminals

Σ表示a finite set of terminals including constants and program variables  所有的常量和变量

R表示a finite relation from V to (V ∪ Σ)∗ corresponding to the grammar production rules  关系

S表示the start symbol  开始符

grammar G中的推导(程序)可以以自顶向下或自底向上的方式枚举，以找到与给定规范一致的推导derivation

### 4.1.1 Top-down Tree search

**一个简单的自上而下的枚举算法：**

该算法在语法中枚举推导程序，并从起始符号s开始，以自上而下的方式保持部分推导P~的有序列表。

1. 在每次迭代中，使用RemoveFirst()函数从集合P~中选择第一个推导p，并检查p是否满足给定的规范φ。如果是，算法返回p作为所需程序。否则, 算法计算部分推导中所有non-terminals的集合节点α~，然后使用一个预定义RankNonTerminal函数来对α~进行扩大和排序。

2. 然后以有序的方式考虑每个non-terminals α，并计算扩大α的可能产生规则集合β~。该算法以排序的方式(使用RankProductionRule函数)再次搜索产生规则集，以计算展扩大导数p '，其中non-terminals α被β取代。

3. 最后，如果p‘已经包含在（P~V）中的某个程序中，该算法通过避免添加新的扩展程序p’来精简搜索空间。

算法一直迭代直到找到符合描述的程序p。该算法通常还为程序推导的最大大小提供了一个附加的界限，用于终止搜索。

![image-20210428212455828](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210428212455828.png)

##### 读后问题：

具体算法还要再看，这个图还是挺清晰的。

问题在于如何定义出这个条件线性整数算术表达式的空间。

### 4.1.2 Bottom-up Tree-search

自底向上树搜索方法缓存在搜索过程中构造的中间程序（intermediate programs）的结果，以修剪冗余的程序族。具体来说，如果两个派生(程序)p和p '相对于规范φ是等价的，那么在搜索过程中只需要考虑其中的一个。对于自顶向下算法，这个检查是在Subsumed函数内执行的，以避免在派生集合p~中添加一个派生p '。这种优化并没有在很大程度上减少自顶向下枚举策略的搜索空间，由于只有完整的推导才能求等价。但是，它可以对下面描述的自底向上搜索技术产生巨大的影响。

**一个简单的自底向上的枚举算法：**

1. 该算法首先在grammar G中按照叶表达式的大小 progSize 的顺序构建一组叶表达式。
2. 然后，它使用较小的表达式逐步构建候选表达式E~，以找到满足规范的表达式。该算法的关键思想是维护一组语义唯一的表达（E~中的两个表达式e和e’相对于规格φ没有功能上的等价）。这种修剪允许自底向上枚举算法显著减少在需要考虑的表达式空间。

## 4.2 Bidirectional Enumerative Search  双向枚举搜索

在某些情况下，双向搜索可以更有效地执行搜索，其中来自输入状态的前向搜索与来自期望输出状态的后向搜索相结合。

**一个简单的双向枚举算法：**

给定一个规格φ≡(φpre， φpost)，其中 φpre 指定输入状态集， φpost 指输出状态集，算法保持F~和B~两组表达式。

1. 集合F~从输入态 φpre 开始执行正向枚举搜索得到的表达式组成，而集合B~从输出态 φpost 开始执行反向搜索得到的表达式组成。

2. 它按照程序大小的递增顺序迭代构建这两个集合，直到找到f∈F~和b∈B~的表达式，使f和b对应的状态能够匹配。所得到的程序是由这两个表达式组合而成的。

双向枚举搜索已被用于许多综合领域，包括学习程序来解决几何结构和缩放汇编代码的超优化。

## 4.3 Offline Exhaustive Enumeration and Composition  离线详尽枚举和组合

另一种有趣但资源密集的枚举技术是对给定大小范围内的所有程序执行离线详尽枚举。然后对程序进行大量预定义输入(pre-defined inputs)的计算，以获得从程序到输入-输出对的<u>对应映射函数</u>。最后，给出一组输入输出示例，使用映射函数检索程序。

**例子：The Unagi system**

给定一个在比赛中使用的位向量程序的DSL, Unagi系统首先离线详尽地枚举给定大小(他们用的是15)的DSL中的所有表达式。在枚举过程中，采用了几种剪枝技术来合并等价程序。

执行枚举后，每个程序对256个预定义的输入位向量进行计算，以创建从程序到输入-输出示例的映射。在比赛中，对于**大小为15的小程序**，Unagi系统使用直接映射来快速找到相应的程序。

为了枚举**更大的程序**，Unagi使用了一种不完整但有效的基于输入-输出等价(input-output equivalence)的剪枝策略。它修剪掉了在256个输入上产生相同输出的表达式，以构造更大的程序。此外，Unagi还利用这些训练问题在大型程序的空间中学习了几个语法先验(syntactic priors)，例如，它学习到了表达式树在训练数据中趋于不平衡，折叠问题的初始累加器值始终为零。它使用这些语法先验来进一步指导详尽枚举。

最后，为了学习**大于50的更大的程序**，它使用统一的策略将输入-输出示例分解为不同的集合。每一组输入-输出示例都可以用一个较小的程序求解，然后使用if条件将这些集合粘在一起。Unagi系统使用了32个线程运行独立的搜索策略，并在亚马逊的EC2云上使用了3000小时的计算时间。值得注意的是，这种统一小表达式来学习大程序的技术也被赢得2016年SyGuS竞赛的枚举综合系统所使用。