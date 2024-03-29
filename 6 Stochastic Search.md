# 6 Stochastic Search

**随机搜索**

随机合成方法学习假设空间中程序的一个分布(distribution)，该分布以规范为条件，然后从分布中抽样程序来学习一个一致的程序。这些技术的目标是<u>使用习得的分布来指导程序的搜索</u>，这更有可能带来符合规范的程序。学习程序分布的一些常见方法包括Metropolis-Hastings算法、genetic programming遗传编程和 machine learning机器学习。

##### **目标：**

学习假设空间里的程序分布来找程序

##### 概念：

**分析树（parse tree），也称具体语法树**（concrete syntax tree），是一个反映某种[形式语言](https://zh.wikipedia.org/wiki/形式语言)[字符串](https://zh.wikipedia.org/wiki/字符串)的[语法](https://zh.wikipedia.org/wiki/语法)关系的有根有序[树](https://zh.wikipedia.org/wiki/树_(图论))。分析树一般按照两种相反的法则生成，一种是[依存语法](https://zh.wikipedia.org/w/index.php?title=依存语法&action=edit&redlink=1),一种是[短语结构语法](https://zh.wikipedia.org/w/index.php?title=短语结构语法&action=edit&redlink=1)。分析树和[抽象语法树](https://zh.wikipedia.org/wiki/抽象語法樹)是不同的。

## 6.1 Metropolis-Hastings Algorithm for Sampling Expressions

(The stochastic SyGuS solver)随机SyGuS求解器：使用**Metropolis-Hastings算法**从给定语法中抽取表达式，以学习符合规范的表达式。该求解器的灵感来自于使用基于马尔可夫链蒙特卡罗(MCMC)采样技术超优化无环二进制汇编代码的工作。

搜索算法的核心思想是首先定义一个Score函数，该函数在所有可能的程序的假设空间中为每个程序分配一个代价(cost)，它定义了程序域内的概率分布。由于代价函数的目标是建模一个高度不规则和高维的搜索空间，它通常是复杂的和非连续的，这使得直接采样技术很难从这种分布中抽样程序。因此，我们使用Metropolis-Hastings算法对期望的程序进行抽样。

假设空间中所有可能的程序(具有一定的固定长度)的空间可以用一个大的**密集图**( large dense graph)来表示，其中节点表示不同的部分表达式，从节点n1(对应表达式e1)到节点n2(对应表达式e2)的一条边表示从e1得到表达式e2的单编辑变换(single-edit transformation)。<u>Metropolis-Hastings算法在图上执行从一个随机节点开始的概率遍历(probabilistic walk)，到达满足规范的期望节点。</u>让Score(e)表示与表示表达式e的节点n相关的代价函数，该表达式捕获e满足规格φ的程度的概念。将概率分布P赋给概率P(e)∝Score(e)，我们增加了算法达到最佳分数的期望表达式的机会。该算法沿增加表达式分数的方向进行概率游走。然而，由于代价函数是非连续的，算法也可以移动到分数较低的图中的表达式，具有一定的低概率。

(The stochastic SyGuS solver)随机SyGuS求解器定义**表达式e的score为：**

![image-20210507191726065](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210507191726065.png)

β是一个设为0.5的常数，C(e)表示e不正确的例子数。当C(e)较小时，Score(e)的值较大，说明e表达式在大多数规范示例上是正确的。当C(e) = 0时，表达式e是期望的表达式。

假设程序的大小固定为常数k，并且图由假设空间中所有大小为k的程序组成。求解器随机选择图中的第一个表达式e。然后，它在e的分析树(parse tree)中随机选择一个节点ν。让eν表示根于此节点ν的子表达式。然后，该算法均匀地从邻近区域中选择另一个大小相等的表达式e'ν。设e'表示用e'ν代替e中的子表达式eν而得到的新表达式。

给定原表达式e和变异表达式e’，Metropolis-Hastings算法选择将变异表达式作为新的候选表达式，接受率(acceptance ratio)如下:

α(e, e') = min(1, Score(e)/Score(e'))

从直观上看，如果新的表达式得到更高的分数值，算法接受变异(i.e. α(e, e0 ) = 1)。否则，它选择接受概率等于α(e, e‘) < 1的新表达式。

给定规范φ时，所期望的表达式的大小通常是未知的。随机求解器从程序规模k = 1开始，在每次Metropolis-Hastings算法迭代后使用一些概率pm( ? )搜索规模k + 1的程序。

让我们假设随机求解器目前正在寻找大小为k = 6的表达式。语法中所有大小为6的表达式的解析树的形状如图(a)所示。这样的表达总共有768个。图(b)显示了解析树中不同节点的终端的可能值。

![image-20210507201310437](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210507201310437.png)

让我们假设求解器选择表达式e = if(x≤0,y, x)作为当前候选表达式(其可能性为1/768)，并且求解器选择布尔判断x≤0进行变异。

![image-20210507203602255](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210507203602255.png)

由于有48种不同的谓词选择，使谓词x≤0变为y≤0的概率为1/6 × 1/48 = 1/288, e' = if(y≤0,y, x)。对于给定的一组输入-输出示例{(−1,4)，(−3,−1)，(−1,−2)，(1,2)，(3,1)，(6,2)}，表达式得分为Score(e) = e−2β， Score(e') = e−3β。求解器然后选择将表达式e突变为e'，概率为e−β。需要注意的是，如果算法考虑变异得到表达式e" = if(x≤y, y, x)，则得分为score (e") = 1，而e"将是算法以1的概率选择的新的候选表达式。

##### 读后问题

需要查找相关资料深入理解Metropolis-Hastings算法

## 6.2 Genetic Programming

遗传编程可以看作是遗传算法在计算机程序领域的扩展，在这个过程中，一群程序不断进化，并利用由达尔文的自然选择理论和包括交叉、突变、复制和删除等自然遗传操作启发的原理，转化为新一代的程序。与传统遗传算法相比，遗传编程的演化结构是动态变化形状和大小的层次计算机程序，其中一组函数和终端符号定义了可能程序的完整假设空间。遗传编程算法维持一个个体程序的种群，并使用变异和交叉操作生成程序变体。变异操作对程序执行随机变化，而交叉操作允许在不同的程序之间重用有用的子程序。适应度函数对应于程序满足规范的程度的概念，用于不同程序变体的适用性。在演进过程中满足规范的程序将作为期望的程序返回。

遗传编程算法从一组适用于给定域的终端和函数开始。该集合定义了算法所考虑的程序的完整搜索空间。假设空间有两个关键要求:i)需要合成的程序存在于搜索空间中，ii)每个函数应该能够接受任何其他函数返回的值或终端值作为其参数。第一种要求保证了程序的存在性，而第二种要求保证了程序在演化过程中通过变异和交叉所获得的程序的格式良好。

遗传编程算法的设计主要有四个步骤。首先，需要定义一组终端符号和函数符号。其次，需要定义与程序的生存能力相对应的**适应度度量**，即程序满足规范的程度。第三，我们需要定义一些**搜索参数**，如种群的大小，程序中表达式的数量，交叉的概率，突变，繁殖，删除等。这些参数指导着进化过程。最后，我们需要确定结束演化过程并返回结果程序的**终止条件**。

遗传算法首先创建一个程序的随机群体，其中群体中的每个程序都是使用终端符号和函数符号随机生成的。由于对终端符号和函数符号集的要求，填充中的每个程序在语法上是正确的。然后测量种群中每个程序的适应度。**适应度度量通常是特定于领域和问题的。适应度度量的一些例子包括正确的输入-输出例子的数量，程序输出和期望输出之间的偏差，基于程序执行的属性(如时间、能量等)，或结合多个标准的多目标成本函数。**由随机程序组成的初始代通常具有非常差的适应度，但一些个体的适应度得分高于其他个体。适应度得分之间的差异用于执行交叉和变异操作。

**交叉操作**根据适应度分数从总体中随机选择两个程序(称为父母程序)。然后从两个程序中**随机选择一个节点(在相应的分析树中)并交换它们**。我们现在得到两个子代程序。这个操作重复多次以获得一组新的子代程序。一个交叉操作示例如图6.3所示。**变异操作根据适应度评分随机选择一个程序并选择一个节点。然后，它删除在选定节点上生根的子树，并使用终端符号和函数符号随机生长一棵新的子树。**图6.4显示了一个突变操作示例。繁殖操作根据适应度分数随机选择一个程序，并将其复制到新的种群中。

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210507211312679.png" alt="image-20210507211312679" style="zoom: 67%;" />

遗传编程算法利用这些操作迭代地进化程序的种群，直到满足终止准则。终止后，获取进化过程中基于适应度评分产生的最佳单程序作为结果返回。如果返回的程序与所需的程序相对应，则将遗传算法的运行指定为成功运行。

基于遗传编程的系统的成功关键取决于良好的**适应度函数**的设计，这需要非凡的洞察力和创造力。遗传编程已被用于发现新的互斥算法和修复程序中的错误。最近也有一些有趣的工作将遗传规划技术与逻辑推理技术结合起来。

## 6.3 Machine Learning

现在已经发展了一些基于机器学习的技术来指导从假设空间学习正确程序的搜索过程。这些技术的主要思想是**学习受规范(例如输入-输出示例)制约的程序空间的概率分布**。然后，这个概率分布被用来抽样程序，更有可能与给定的规范一致。

Menon等人提出了一个基于实例编程的机器学习框架，**假设空间由上下文无关语法(CFG)定义，系统给出一组输入-输出示例，学习语法中不同规则的权重(称为线索 *clues*)，从而获得相应的*概率上下文无关语法probabilistic context-free grammar* (PCFG)**。然后使用PCFG来指导枚举搜索，以找到所需的程序。利用输入输出样例训练语料库和相应的程序，对输入输出样例的不同语法规则的权重进行训练。

##### 概念：

*<u>probabilistic context-free grammar</u>：PCFG扩展了上下文无关语法，类似于隐藏马尔科夫模型扩展常规语法的方式。每个产品都有一个概率。一个派生(解析)的概率是该派生中使用的产品的概率的乘积。这些概率可以被视为模型的参数，对于大的问题，通过机器学习很方便地学习这些参数。概率语法的有效性受其训练数据集上下文的限制。*

该框架实例化在文本处理语言上，它在FlashFill DSL之上添加了文本函数(text functions)。然后，领域专家手动定义某些特征，这些特征可以帮助学习算法在输入-输出例子中识别正确的程序。一些特性包括检查输出字符串集合中是否有重复字符串的dedup_cue，检查输出字符串是否被排序的sort_cue等。学习过程利用了这样一个事实，即规则用于一组输入-输出对(x¯, y¯)所需程序的机会很大程度上取决于x¯和y¯结构中的某些特征，即特征。这种方法在数量级上比枚举baseline具有更好的性能。

这个框架的一个缺点是手工设计有用的线索(特性)是一项耗时且困难的任务，并且需要大量的领域专业知识。此外，由于学习到的权重结果在PCFG中，框架不能使用搜索过程中生成的部分树的上下文信息来更有效地指导探索。

## 6.4 Neural Program Synthesis

程序合成和程序归纳法的问题最近引起了深度学习社区的极大兴趣。提出的方法可分为两类:一是程序归纳，二是程序合成。执行程序归纳的神经结构学习一个能够复制所需程序行为的网络，而执行程序合成的神经系统返回一个符合所需规范行为的可解释程序。

程序归纳技术开发了新的神经结构，可以学习与给定输入-输出示例集一致的程序行为。这些方法中的许多都是受到计算模块(如CPU、GPU)和数据结构(如栈)的启发。**这些方法的关键思想是开发网络原子操作的连续表示，然后使用神经控制器的端到端训练或强化学习来学习程序行为。**虽然这个程序归纳的工作已经导致了一些有前途的神经结构，但也有一些缺点。这些技术的一个缺点是不能生成已学习程序的可解释模型，而且通常需要大量的计算资源和每个合成任务数千个输入输出示例。

最近提出的一些神经系统确实可以学习可解释程序。神经FlashFill系统开发两个神经结构: 1)交叉相关的 I/O 编码器产生的连续表示的一组输入-输出的例子, 和2)Recursive-Reverse-Recursive神经网络(R3NN)逐步合成在连续表示示例的DSL中的程序。Neural-RAM学习由一组给定的模块组成的回路，这些模块与一组输入输出示例相一致。它首先开发所有模块的连续表示，然后学习一个定义不同模块之间如何相互连接的控制器。DeepCoder学习整数输入输出示例的嵌入，以学习可能对示例上所需转换有用的函数空间的分布。然后，它使用学习到的函数分布来指导类似于§4.1.1所示的基于深度优先的自顶向下枚举算法。最后，也有一些最近的概率编程语言，如Terpret和Forth，使程序员能够编写部分程序，并使用基于梯度下降的技术来完成它们，使完成的程序满足输入输出示例。

对于FlashFill DSL，神经系统FlashFill系统会根据输入-输出示例学习DSL中的程序生成模型。生成模型是端到端使用DSL中的大型训练程序集以及相应的输入-输出示例进行训练的。训练集是通过从DSL中采样程序自动生成的，然后使用基于规则的方法生成符合要求的输入-输出示例。该系统开发了两个关键的神经结构:  i) i/O编码器  和  ii) R3NN网络。系统从语法的起始符号开始，使用R3NN增量地构造所需要的程序。R3NN取一个部分程序树(语法中的派生)，并决定树中的哪个非终结节点要展开，以及使用哪个展开规则。生成过程受I/O编码向量的制约。

I/O编码器生成输入-输出示例集的连续表示。互相关编码器(The cross correlation encoder)首先在输入和输出字符串上运行双向lstm，然后计算从lstm得到的两个输出张量之间的互相关。洞察的关键思想是了解输出字符串的哪些部分来自输入字符串的哪些部分。