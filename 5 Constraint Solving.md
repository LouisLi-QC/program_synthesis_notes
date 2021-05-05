# 5 Constraint Solving

许多程序合成的成功应用都依赖于一些约束求解技术(constraint solving)。一般来说，约束求解指的是在一个公式(一个模型)中找到一个自由变量的实例(an instantiation of free variables)，从而使公式成立。将约束求解应用于程序合成的<u>关键思想是将规范和语法程序限制(syntactic program restrictions)都编码在一个公式中，这样任何真实的模型都对应于一个正确的程序。</u>编码应该将公式中的变量映射到所需程序中子表达式的选择。

例子：一个8-bit变量x的按位操作(bitwise operations)的DSL

![image-20210429211536999](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210429211536999.png)

该DSL中的每个程序都包含一个最顶层的二进制操作(a single top-most binary operation)，每个操作数要么是输入，要么是某个8-bit常量。

##### theory of bitvectors TBV

Φ(hP , hE1, hE2, c1, c2) encodes any program P in this DSL that satisfies the spec φ: ∀x P(x) ≥ 0 (as a signed integer).

## 5.1 Component-Based Synthesis

### 5.1.1 End-to-end SMT Encoding

第一个基于约束的程序合成方法将语言和规范直接编码到SAT/SMT约束中，类似于上面的示例。

这种编码的一个经典例子是**基于组件的合成 component-based synthesis**。我们在§3.2.1中介绍了这一工作，以提出区分输入的概念。现在，我们将讨论其整体编码的合成问题。

在基于组件的合成中，我们的语法偏差是一个<u>被允许的组件</u>(allowed components)的库。每个组件都是一个特定于领域(domain-specific function)的函数，可以在所需的程序中使用。通常，它们的组合没有语法限制(例如，没有语法):任何类型安全的、格式良好的组件组合在语言中构成了一个有效的程序。在不失一般性的前提下，我们假设每个程序使用所有提供的组件，每个恰好一次(可以通过在库中包含多个副本来实现组件的多次使用)。

Jha等人首先将他们的编码应用到位操作程序领域。在其中，只有一种类型(固定长度的位向量)，因此所有组件用法都是先验类型安全的。为了解决一个合成问题，他们至少需要编码所需的程序：

(a) is well-formed
(b) 在不违反各自规范的情况下使用所有组件
(c) 与所提供的规范一致(例如：输入-输出示例)。
此外，为了应用distinguishing inputs技术(见§3.2.1)，他们在一个单独的查询程序中编码
(d) 在一个新的输入上与其他一些一致的程序不一致。
我们现在描述基于组件的合成约束(a)-(c)。

#### Well-formedness

如果一个无循环程序能够形成一个由组件及其连接的有向无环图(DAG)(即<u>一个组件的输出作为其他组件的输入</u>)，那么该程序就是结构良好(well-formed)的。此外，它必须只使用库中的所有组件一次。

意思就是每个组件都用一次，而且组建可以连接成图（单向流程图）

well-formed的SMT编码将程序中的每个变量与定义和赋值的单行号相关联。

使用这种表示法，编码格式良好性降低为:

(a) 确保所有的行号在各自的域范围内(即程序输入在程序的第一行被赋值，所有库组件在接下来的行被调用)。
(b) 确保一致性:所有输出变量(及其对应的行号)应不同。
(c) 确保无循环性:组成部分应只接受早先分配的输入变量

#### Component usage

所有这些规范都被附加到合成约束中，它们的输入和输出映射到程序中相应的变量。为了进一步确保程序的数据流是一致的，我们添加了另一个约束，以确保输出变量作为另一个组件的输入时保持相同的值。

#### Specification

Jha等人使用输入-输出示例作为规范，这些示例很容易在附加到合成约束的附加子句中编码。一般来说，只要能够在底层SMT逻辑中编码(或已经表示为SMT逻辑)，任何规范形式就足够了。然而，复杂的逻辑规范可能会使综合约束更加难以解决，因此Jha等人所使用的输入-输出示例和外部验证oracle的组合对于高效的程序综合更为有利。

##### 读后问题

需要深入学习SAT/SMT —— 命题逻辑公式(propositional logical formula)的可满足性判定问题 / 可满足性模理论(satisfiability modulo theories)

### 5.1.2 Sketch generation and completion

Jha等人的合成方法对整个合成问题进行了完整的SMT编码，给出了组件库和所需的规范。虽然功能强大，但这种编码很难设计和实现。另一种解决方案是将合成过程拆分为sketch生成和sketch实现(Sketch generation and completion)。如§3.3.1所介绍的sketching技术，假定存在一个sketch——一个部分程序，由求解器用子表达式填充漏洞。通常，这些sketch是由人编写的，但是也可以自动生成它们。

## 5.2 Solver-Aided Programming

开发特定于问题的SAT/SMT编码并生成相应的公式可能非常麻烦和耗时。出于这个原因，社区构建了几个框架来表示合成问题(和一些相关问题)，这些框架将高级编程语言中的表示转换为相应的幕后公式。更一般地说，这样的框架实现了<u>求解器辅助编程</u>(solver-aided programming)的思想:它们用新的结构增强了高级编程语言，这些结构要求约束求解被具体化。这样的构造可以将二级子问题嵌入到宿主语言的常规程序中。

## 5.3 Inductive Logic Programming

归纳逻辑编程(ILP)是一个从实例中自动推理逻辑程序的领域。与本研究中的其他综合方法相比，ILP侧重于<u>合成一阶规则和关系</u>，而不是函数表达式或程序。然而，它在方法论上类似于基于约束的合成——应用逻辑推理来推导所需的关系。由于ILP中的主要规范类型是一组示例，因此ILP也与programming by examples密切相关，将在第7章中讨论。

