# 复合集合与索引

大多数 线性规划 (linear program) 模型都涉及对来自多个不同集合的成员组合进行索引。实际上，索引集合之间的交互往往是模型中最复杂的部分；一旦确定了集合的排列方式，模型的其余部分就可以清晰简洁地编写。

除了最简单的模型外，所有模型都使用复合集合，其成员是对象的二元组、三元组、四元组，甚至是更长的“元组” (tuple)。本章从有序对集合的声明和使用开始。我们首先关注来自两个集合的所有对的集合，然后转向所有对的子集以及通过成对集合的“切片” (slice)。后续章节将探讨更长元组的集合，以及 AMPL 的集合运算符和索引表达式在元组集合上的扩展。

本章的最后一节介绍了按其他集合索引声明的集合。索引集合的集合通常与元组集合扮演类似的角色，但它代表了一种略有不同的建模思路。每种集合在某些情况下都是合适的，我们提供了一些选择它们之间的指导原则。

## 6.1 有序对集合

有序对的对象，无论是数字还是字符串，都写成用逗号分隔并用括号括起来的形式：

("PITT", "STL") ("bands", 5) (3, 101)

正如“有序” (ordered) 这个词所暗示的，哪个对象在前是有区别的；("STL", "PITT") 与 ("PITT", "STL") 不同。同一个对象可以同时出现在第一和第二位置，例如 ("PITT", "PITT")。

对可以像单个对象一样收集到集合中。逗号分隔的对列表可以用花括号括起来，表示一个有序对的字面量集合：

$$
\begin{array}{rl}
& \{\mathrm{("PITT","STL"),("PITT","FRE"),("PITT","DET"),("CLEV","FRE")}\} \\
& \{\mathrm{(1,1),(1,2),(1,3),(2,1),(2,2),(2,3),(3,1),(3,2),(3,3)}\}
\end{array}
$$

然而，由于有序对集合通常较大且容易变化，它们很少在 AMPL 模型中显式出现。相反，它们以各种方式被符号化地描述。

在我们的示例中，经常出现来自两个给定集合的所有有序对的集合。例如，在图 3-1a 的运输模型中，所有起始地-目的地对的集合可以写成以下任一形式：

$$
\begin{array}{rl} & \{\mathtt{ORIG},\mathtt{DEST}\} \\ & \{\mathtt{i}\mathtt{in}\mathtt{ORIG},\mathtt{j}\mathtt{in}\mathtt{DEST}\} \end{array}
$$

这取决于上下文是否需要虚拟索引 i 和 j。图 4-4 中的多周期生产模型使用了一个由字符串集合（代表产品）和数字集合（代表周）组成的全部配对集合：

$$
\begin{array}{rl} & \{\mathtt{PROD},\mathtt{1}.\mathtt{T}\} \\ & \{\mathtt{p}\mathtt{in}\mathtt{PROD},\mathtt{t}\mathtt{in}\mathtt{1}.\mathtt{T}\} \end{array}
$$

模型组件的各种集合，如参数 `revenue` 和变量 `Sell`，都在这个集合上建立索引。当在模型中引用个别组件时，它们必须有两个下标，如 `revenue[p,t]` 或 `Sell[p,t]`。下标的顺序总是与配对中对象的顺序相同；在这种情况下，第一个下标必须引用 `PROD` 中的字符串，第二个下标必须引用 `1..T` 中的数字。

像 $\{\mathtt{p}\mathtt{in}\mathtt{PROD},\mathtt{t}\mathtt{in}\mathtt{1}.\mathtt{T}\}$ 这样的索引表达式是代数符号中“对于所有 $p$ 属于 $P$，$t = 1,\ldots ,T$”这类短语的 AMPL 转写。在这种情况下，没有特别的理由要以有序对的方式来思考，实际上我们在第 4 章介绍多周期生产模型时并未提及有序对。另一方面，我们可以通过显式定义配对集合，来修改图 3-1a 中的运输模型，以强调产地-目的地配对作为城市间“链接 (LINKS)”的作用：

$$
\mathtt{set}\mathtt{LINKS} = \{\mathtt{ORIG},\mathtt{DEST}\} ;
$$

然后可以在链接上对运输成本和运输量建立索引：

$$
\begin{array}{rl} & \mathtt{param cost}\{\mathtt{LINKS}\} > = 0;\\ & \mathtt{var Trans}\{\mathtt{LINKS}\} > = 0; \end{array}
$$

在目标函数中，所有运输成本的总和可以这样写：

`minimize Total Cost: sum {(i,j) in LINKS} cost[i,j] * Trans[i,j];`

注意，当虚拟索引在一个像 `LINKS` 这样的配对集合上运行时，它们必须以 `(i,j)` 这样的配对形式定义。如果写成 `{k in LINKS}` 来求和就会出错。完整模型如图 6-1 所示，应与图 3-1a 进行比较。数据的指定可以与图 3-1b 相同。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/2c7501196145490692514f8414873ddf5b5302f3564747b3e3777b6cf186d231.jpg)  
图 6-1：包含所有配对的运输模型 (transp2.mod)。

# 6.2 有序对的子集和切片

在许多应用中，我们只关心来自两个集合的所有有序对的一个子集。例如，在运输模型中，可能并非每个产地都能向每个目的地发货。单位运输成本可能仅针对可用的产地-目的地配对提供，因此希望仅在这些配对上对成本和变量建立索引。用 AMPL 的术语来说，我们希望上面定义的集合 `LINKS` 仅包含数据中给出的配对子集，而不是来自 `ORIG` 和 `DEST` 的所有配对。

声明集合 `LINKS` 是不够的，因为这样只声明了一个单成员的集合。至少，我们需要写成 `set LINKS dimen 2;` 以表明数据必须由“维度”为二的成员组成——即成对的元素。更好的做法是，我们可以说 `LINKS` 是 `ORIG` 和 `DEST` 所有配对组合构成的集合的一个子集：

`set LINKS within {ORIG,DEST};`

这样做的优点在于使模型的意图更加清晰；同时也有助于发现数据中的错误。随后对参数 `cost`、变量 `Trans` 以及目标函数的声明与图 6-1 中的保持一致。但现在 `cost[i,j]` 和 `Trans[i,j]` 的定义仅限于数据中作为 `LINKS` 成员给出的那些配对组合，并且表达式 `sum {(i,j) in LINKS} cost[i,j] * Trans[i,j]` 将只表示对这些指定配对的求和。

约束条件如何编写？在原始运输模型中，供应上限约束是：

subject to Supply {i in ORIG}: sum {j in DEST} Trans[i,j] $=$ supply[i];

当 LINKS 是配对的子集时，这种写法将不再适用，因为对于每个 i in ORIG，它试图对 DEST 中的所有 j 求和 Trans[i,j]，而 Trans[i,j] 只对 LINKS 中的配对 (i,j) 有定义。如果我们尝试这样做，将会收到如下错误信息：

error processing constraint Supply['GARY']: invalid subscript Trans['GARY','FRA']

我们真正想表达的是，对于每一个起始地 i，求和应覆盖所有满足 (i,j) 是允许链接的目的地 j。这一语句可以直接转换为 AMPL 语法，方法是在 sum 后的索引表达式中添加一个条件：

subject to Supply {i in ORIG}: sum {j in DEST: (i,j) in LINKS} Trans[i,j] $=$ supply[i];

不过，为了避免这种略显笨拙的形式，AMPL 允许我们从索引表达式中省略 j in DEST，从而产生如下更为简洁的约束：

subject to Supply {i in ORIG}: sum {(i,j) in LINKS} Trans[i,j] $=$ supply[i];

由于 {(i,j) in LINKS} 出现在 i 已经被定义的上下文中，AMPL 将此索引表达式解释为所有使得 (i,j) 属于 LINKS 的 j 的集合。需求约束的处理方式类似，模型的完整修订版本如图 6-2a 所示。该模型的一小部分代表性数据如图 6-2b 所示；AMPL 提供了多种方便的方法来指定复合集合的成员资格及其索引数据，详见第 9 章。

从图 6-2a 可以看出，索引表达式 {(i,j) in LINKS} 在三个出现的地方具有不同的含义。其成员可以通过如下表格来理解：

<table><tr><td></td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td></tr><tr><td>GARY</td><td></td><td>x</td><td>x</td><td></td><td></td><td></td><td></td></tr><tr><td>CLEV</td><td>x</td><td>x</td><td>x</td><td>x</td><td>x</td><td></td><td>x</td></tr><tr><td>PITT</td><td>x</td><td></td><td>x</td><td>x</td><td>x</td><td></td><td></td></tr></table>

行表示起始地，列表示目的地，集合中的每一对用 x 标记。对于 {ORIG, DEST} 的表格将完全填充 x，而所示表格描述了由图 6-2b 中数据定义的“稀疏”对子集的 {LINKS}。

在 i 和 j 当前未定义的点上，例如在目标函数 minimize Total_Cost: sum {(i,j) in LINKS} cost[i,j] * Trans[i,j]; 中

set ORIG; # 起始地集合  
set DEST; # 目的地集合  
set LINKS within {ORIG,DEST}; # 链接集合  
param supply {ORIG} $>= 0$; # 起始地可用数量  
param demand {DEST} $>= 0$; # 目的地需求数量  
check: sum {i in ORIG} supply[i] $=$ sum {j in DEST} demand[j];  
param cost {LINKS} $>= 0$; # 每单位运输成本  
var Trans {LINKS} $>= 0$; # 待运输单位数  
minimize Total_Cost: sum {(i,j) in LINKS} cost[i,j] * Trans[i,j];  
subject to Supply {i in ORIG}: sum {(i,j) in LINKS} Trans[i,j] $=$ supply[i];  
subject to Demand {j in DEST}: sum {(i,j) in LINKS} Trans[i,j] $=$ demand[j];

图 6-2a：具有选定对的运输模型 (transp3.mod)。

param: ORIG: supply :=  
GARY 1400  
CLEV 2600  
PITT 2900;  

param: DEST: demand :=  
FRA 900  
DET 1200  
LAN 600  
WIN 400  
STL 1700  
FRE 1100  
LAF 1000;  

param: LINKS: cost :=  
GARY DET 14  
GARY LAN 11  
GARY STL 16  
GARY LAF 8  
CLEV FRA 27  
CLEV DET 9  
CLEV LAN 12  
CLEV WIN 9  
CLEV STL 26  
CLEV LAF 17  
PITT FRA 24  
PITT WIN 13  
PITT STL 28  
PITT FRE 99;

索引表达式 $\{(i,j)$ in LINKS} 表示此表中的所有对。但在 i 已经定义的点上，例如在供应约束 Supply {i in ORIG}: sum {(i,j) in LINKS} Trans[i,j] $=$ supply[i]; 中，表达式 $\{(i,j)$ in LINKS} 仅与表中对应于 i 的行相关。你可以将其视为在已定义的第一个分量对应的行中对表进行一维“切片”。虽然在此情况下第一个分量是先前定义的虚拟索引，但当第一个分量是任何可计算为有效集合对象的表达式时，也适用相同约定；例如，我们可以写 来表示表中第一行的对。

同样，在 j 已经定义的地方，例如在需求约束 Demand {j in DEST}: sum {(i,j) in LINKS} Trans[i,j] $=$ demand[j]; 中，表达式 $\{(i,j)$ in LINKS} 从表中对应于 j 的列中选择对。表中第三列的对可以通过 $\{(i, "LAN")$ in LINKS} 指定。

# 6.3 更长元组的集合

AMPL 中表示有序对的记法可以自然地扩展到三元组、四元组或任意长度的有序列表。集合中的所有元组必须具有相同的维度。例如，一个集合不能同时包含有序对和三元组，也不能根据数据中的某些值来确定该集合包含的是有序对还是三元组。

图 4-1 中的多产品 (Multicommodity) 运输模型 (transportation model) 提供了一些如何使用有序三元组（以及更长的元组）的示例。在该模型的原始版本中，成本 (cost) 和运输量 (Trans) 的索引都是基于起始地-目的地-产品 (ORIG-DEST-PROD) 三元组：

```ampl
param cost {ORIG, DEST, PROD} >= 0;
var Trans {ORIG, DEST, PROD} >= 0;
```

在目标函数中，cost 和 Trans 都带有三个下标，总成本 (Total Cost) 通过对所有三元组求和得到：

```ampl
minimize Total_Cost:
    sum {i in ORIG, j in DEST, p in PROD} cost[i,j,p] * Trans[i,j,p];
```

索引表达式与之前类似，只是它们列出的是三个集合而非两个。若索引表达式列出了 $k$ 个集合，则相应地表示一个由 $k$ 元组组成的集合。

如果我们像图 6-2a 中那样定义 LINKS，则多产品的声明如下所示：

```ampl
set LINKS within {ORIG, DEST};
param cost {LINKS, PROD} >= 0;
var Trans {LINKS, PROD} >= 0;
minimize Total_Cost:
    sum {(i,j) in LINKS, p in PROD} cost[i,j,p] * Trans[i,j,p];
```

这里展示了如何将一个三元组集合指定为由一个有序对集合 (LINKS) 和一个单元素集合 (PROD) 组合而成。由于 cost 和 Trans 的索引是 {LINKS, PROD}，因此它们的前两个下标必须来自 LINKS 中的一个有序对，第三个下标则来自 PROD 中的一个成员。类似地，也可以构建更长的元组集合。

最后一种可能的情况是，只有某些起始地、目的地和产品的组合是可行的。此时，我们可以定义一个仅包含允许组合的三元组的集合：

```ampl
set ROUTES within {ORIG, DEST, PROD};
```

成本和运输量都基于这个集合进行索引：

```ampl
param cost {ROUTES} >= 0;
var Trans {ROUTES} >= 0;
```

在目标函数中，总成本是对该集合中所有三元组求和得到的：

```ampl
minimize Total_Cost:
    sum {(i,j,p) in ROUTES} cost[i,j,p] * Trans[i,j,p];
```

单个三元组以类似于有序对的方式书写，即用括号括起并以逗号分隔的列表 (i,j,p)。更长的列表则表示更长的元组。

在该模型的三个约束条件中，求和必须在集合 ROUTES 的三个不同切片上进行：

```ampl
subject to Supply {i in ORIG, p in PROD}:
    sum {(i,j,p) in ROUTES} Trans[i,j,p] = supply[i,p];

subject to Demand {j in DEST, p in PROD}:
    sum {(i,j,p) in ROUTES} Trans[i,j,p] = demand[j,p];

subject to Multi {i in ORIG, j in DEST}:
    sum {(i,j,p) in ROUTES} Trans[i,j,p] <= limit[i,j];
```

例如，在供应约束 (Supply constraint) 中，索引 $i$ 和 $p$ 是在求和符号之前定义的，因此 {(i,j,p) in ROUTES} 表示所有使得 (i,j,p) 是 ROUTES 中一个三元组的 $j$。AMPL 允许对任意维度和任意坐标组合的元组集合进行类似的切片操作。

当你声明一个高维集合（如 ROUTES）时，像 within {ORIG,DEST,PROD} 这样的短语可能表示一个包含大量成员的集合。例如，若有 10 个起始地 (origins)、100 个目的地 (destinations) 和 100 种产品 (products)，则该集合潜在地包含 100,000 个成员。幸运的是，AMPL 在处理声明时并不会创建这个集合，而只是检查 ROUTES 数据中的每个元组，确保其第一个分量属于 ORIG、第二个分量属于 DEST、第三个分量属于 PROD。因此，只要集合 ROUTES 本身不包含大量的三元组，就可以高效地处理它。

在其他上下文中使用高维集合时，你可能需要更加小心，以避免无意中迫使 AMPL 生成一个庞大的元组集合。例如，考虑如何约束从每个起始地运出的所有产品的总量不超过某个值。你可以写成：

```ampl
subject to Supply_All {i in ORIG}:  
sum {j in DEST, p in PROD: (i,j,p) in ROUTES} Trans[i,j,p] <= supply_all[i];
```

或者使用更紧凑的切片表示法：

```ampl
subject to Supply_All {i in ORIG}:  
sum {(i,j,p) in ROUTES} Trans[i,j,p] <= supply_all[i];
```

在第一种情况下，AMPL 显式地生成集合 $\{j$ in DEST, $p$ in PROD$\}$ 并检查 $(i,j,p)$ 是否属于 ROUTES；而在第二种情况下，它能够使用一种更高效的方法来查找 ROUTES 中具有给定 $i$ 的所有 $(i,j,p)$。在我们的小例子中，这可能看起来并不重要，但对于具有实际规模的问题，切片版本可能是唯一能够在合理的时间和空间内处理的版本。

# 6.4 元组集合上的操作

对于复合集合的操作，尽可能与第 5 章中为简单集合引入的操作相同。由二元组、三元组或更长元组组成的集合可以进行并集 (union)、交集 (inter)、差集 (diff) 和对称差集 (symdiff) 运算；可以用 in 和 within 进行测试；并可以用 card 进行计数。操作数的维度必须适当地匹配，例如你不能将一个由二元组组成的集合与一个由三元组组成的集合取并集。此外，AMPL 中的复合集合不能被声明为有序 (ordered) 或循环 (circular)，因此也不能作为 first 和 next 等仅接受有序集合作为参数的函数的输入。

另一个集合运算符 cross 用于生成其参数的所有二元组合——即笛卡尔积 (cross or Cartesian product)。因此，集合表达式

ORIG cross DEST

表示与索引表达式 $\{\text{ORIG}, \text{DEST}\}$ 相同的集合，而

ORIG cross DEST cross PROD

则与 $\{\text{ORIG}, \text{DEST}, \text{PROD}\}$ 相同。

到目前为止，我们的示例都是这样构建的：每个复合集合都在先前指定的简单集合的笛卡尔积 (cross product) 中有一个定义域；例如，LINKS 位于 ORIG 与 DEST 的笛卡尔积中，而 ROUTES 位于 ORIG、DEST 和 PROD 的笛卡尔积中。这种做法有助于生成清晰且正确的模型。然而，如果你发现将定义域作为数据的一部分来指定不太方便，你也可以在模型内部来定义它们。为此，AMPL 提供了一个迭代集合算子 (iterated set operator)，如下例所示：

```
set ROUTES dimen 3;
set PROD = setof {(i,j,p) in ROUTES} p;
set LINKS = setof {(i,j,p) in ROUTES} (i,j);
```

类似于迭代求和算子，`setof` 后面跟着一个索引表达式和一个参数，该参数可以是任何能够计算出合法集合成员的表达式。该参数对索引集合的每个成员进行计算，然后将结果合并成一个新的集合，并由该算子返回。重复的成员会被忽略。因此，这些用于 PROD 和 LINKS 的表达式将给出所有对象 $\mathbb{P}$ 和所有对 $(i,j)$ 组成的集合，使得存在某个成员 $(i,j,p)$ 属于 ROUTES。

与简单集合一样，复合集合中的成员资格也可以通过在索引表达式的末尾添加逻辑条件来加以限制。例如，多商品运输模型 (multicommodity transportation model) 可以定义：

$$
\mathtt{set~DEMAND} = \{\mathtt{j~in~DEST},\mathtt{p~in~PROD}: \mathtt{demand}[\mathtt{j},\mathtt{p}] > 0\} ;
$$

这样，`DEMAND` 只包含那些在目的地 j 对产品 p 有正需求量 (positive demand) 的对 (j,p)。再举一个例子，假设我们还希望对产品从一个起始地转移到另一个起始地进行建模。我们可以简单地定义：

$$
\mathtt{set~TRANSF} = \{\mathtt{ORIG},\mathtt{ORIG}\};
$$

以指定由 `ORIG` 中所有成员组成的对的集合。但这个集合将包括像 ("PITT","PITT") 这样的对；为了指定 `ORIG` 中所有不同成员组成的对的集合，必须添加一个条件：

$$
\mathtt{set~TRANSF} = \{\mathtt{i1~in~ORIG},\mathtt{i2~in~ORIG}: \mathtt{i1} <> \mathtt{i2}\};
$$

这是另一个需要定义两个不同的虚拟索引 (dummy indices) i1 和 i2 来遍历同一集合的情况；该条件选择那些 i1 不等于 i2 的对。

如果一个集合是有序的，那么索引表达式内的条件也可以引用该顺序。我们可以声明集合 `ORIG` 是有序的：

```
set ORIG ordered;
set TRANSF = {i1 in ORIG, i2 in ORIG: ord(i1) < ord(i2)};
```

以定义一个来自 `ORIG` 的“三角形”对集合，其中不包含任何一对及其反向对。例如，根据 `ORIG` 中哪个排在前面，`TRANSF` 将包含 ("PITT","CLEV") 或 ("CLEV","PITT") 其中之一，但不会同时包含两者。

数字集合可以类似地处理，因为它们本身具有自然的顺序。假设我们希望在图 4-4 的多周期生产模型 (multiperiod production model) 中通过如下声明来容纳不同年限的库存：

集合 `PROD`; # 产品  
参数 `T > 0`; # 周数  
参数 `A > 0`; # 库存最大年限  
变量 `Inv {PROD, 0..T, 0..A} >= 0`; # 库存吨数  

根据初始库存的处理方式，我们可能需要添加一个约束条件，即在周期 t 内的库存不能超过 t 周龄：

```
subject to Too_Old {p in PROD, t in 1..T, a in 1..A: a > t}: Inv[p,t,a] = 0;
```

在这种情况下，有一种更简单的写法来表示索引表达式：

```
subject to Too_Old {p in PROD, t in 1..T, a in t+1..A}: Inv[p,t,a] = 0;
```

这里由 `t in 1..T` 定义的虚拟索引立即用于短语 `a in t+1..A` 中。在本例及其他指定两个或多个集合的索引表达式中，逗号分隔的短语从左到右依次求值。在一个短语中定义的任何虚拟索引都可以在所有后续短语中使用。

# 6.5 索引集合族

虽然在 AMPL 模型中最常见的是单个集合的声明，但集合也可以按其他集合进行索引形成集合族。其原理与参数、变量或约束的索引集合族非常相似。

为了说明索引集合族的用途，让我们扩展图 4-4 中的多周期生产模型，以识别每种产品的不同市场区域。我们首先声明：

```
set PROD;
set AREA {PROD};
```

这表示对于 `PROD` 的每个成员 p，都存在一个集合 `AREA[p]`；其成员将表示产品 p 销售的市场区域。

市场需求量、预期销售收入和销售量也应按区域、产品和周进行索引：

param market {p in PROD, a in AREA[p], 1..T} >= 0;  
param revenue {p in PROD, a in AREA[p], 1..T} >= 0;  
var Sell {p in PROD, a in AREA[p], t in 1..T} >= 0, <= market[p,a,t];

在 market 和 revenue 的声明中，我们只定义了指定集合 AREA[p] 所需的虚拟索引 (p, a)，但对于 Sell 变量，我们需要定义所有的虚拟索引，以便可以用来指定上界 market[p,a,t]。这是另一个索引表达式的一个短语中定义的索引被后续短语使用的例子；对于来自集合 PROD 的每个 p，a 都遍历一个不同的集合 AREA[p]。

在目标函数中，图 4-4 中的表达式 revenue[p,t] * Sell[p,t] 必须替换为对产品 p 所有区域收入的求和：

maximize Total_Profit:  
sum {p in PROD, t in 1..T}  
(sum {a in AREA[p]} revenue[p,a,t]*Sell[p,a,t] - producost[p]*Make[p,t] - invcost[p]*Inv[p,t]);

唯一其他的变化是在 Balance 约束中，Sell[p,t] 同样被替换为求和形式：

subject to Balance {p in PROD, t in 1..T}:  
Make[p,t] + Inv[p,t-1] = sum {a in AREA[p]} Sell[p,a,t] + Inv[p,t];

集合 PROD：# 产品  
集合 AREA {PROD}；# 每个产品的市场区域  
参数 T > 0；# 周数  
参数 rate {PROD} > 0；# 每小时生产的吨数  
参数 inv0 {PROD} >= 0；# 初始库存  
参数 avail {1..T} >= 0；# 每周可用的小时数  
参数 market {p in PROD, a in AREA[p], 1..T} >= 0；# 每周销售吨数的限制  
参数 producost {PROD} >= 0；# 每吨生产的成本  
参数 invcost {PROD} >= 0；# 每吨库存的成本  
参数 revenue {p in PROD, a in AREA[p], 1..T} >= 0；# 每吨销售的收入  

变量 Make {PROD,1..T} >= 0；# 生产的吨数  
变量 Inv {PROD,0..T} >= 0；# 库存吨数  
变量 Sell {p in PROD, a in AREA[p], t in 1..T} >= 0, <= market[p,a,t]；# 销售吨数  

目标函数最大化 Total_Profit:  
$$
\sum_{p \in PROD, t \in 1..T} \left( \sum_{a \in AREA[p]} revenue[p,a,t] \cdot Sell[p,a,t] - producost[p] \cdot Make[p,t] - invcost[p] \cdot Inv[p,t] \right);
$$
# 所有产品所有周的总收入减去成本  

约束条件 Time {t in 1..T}:  
$$
\sum_{p \in PROD} \left( \frac{1}{rate[p]} \right) \cdot Make[p,t] \leq avail[t];
$$
# 所有产品使用的总小时数不能超过每周可用的小时数  

约束条件 Init_Inv {p in PROD}:  
$$
Inv[p,0] = inv0[p];
$$
# 初始库存必须等于给定值  

约束条件 Balance {p in PROD, t in 1..T}:  
$$
Make[p,t] + Inv[p,t-1] = \sum_{a \in AREA[p]} Sell[p,a,t] + Inv[p,t];
$$
# 生产的吨数和从库存中取出的数量必须等于销售的数量和放入库存的数量  

完整模型如图 6-3 所示。

在此模型的数据中，索引集合 AREA 中的每个集都像普通集合一样指定：

```
set PROD := bands coils;
set AREA[bands] := east north;
set AREA[coils] := east west export;
```

参数 `revenue` 和 `market` 现在是三个集合的索引，因此它们的数据值以一系列表格的形式指定。由于索引是对不同集合 `AREA[p]` 的每个产品 `p`，值最方便地安排为每个产品一张表，如图 6-4 所示。（第 9 章解释了这种排列背后的一般规则。）

我们可以用一个集合 `PRODAREA` 来代替这个模型，该集合包含一对 `(p,a)`，使得产品 `p` 将在区域 `a` 中销售当且仅当 `(p,a)` 是 `PRODAREA` 的成员。然而，我们基于 `PROD` 和 `AREA[p]` 的公式似乎更可取，因为它强调了产品与区域之间的层次关系。尽管模型在许多地方必须引用销售一个产品的所有区域的集合，但它从未引用在一个区域中销售的所有产品的集合。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/b9aec6ee43a6ffccb8ce910751795eb2e3fc2e0c466b6b2d5b56a0fd84ee27ff.jpg)  
图 6-4：带索引集合的多周期生产数据 (steelT3.dat)。

作为对比示例，我们可以考虑多产品运输模型 (multicommodity transportation model) 如何使用索引集合 (indexed collections of sets)。如图 6-5 所示，对于每种产品 (product)，我们定义了一个供应该产品的起始地 (origins) 集合、一个需求该产品的目的地 (destinations) 集合，以及一个表示该产品可能运输路径的连接 (links) 集合：

```
set ORIG;     # 起始地集合
set DEST;     # 目的地集合
set PROD;     # 产品集合

set orig {PROD} within ORIG;
set dest {PROD} within DEST;
set links {p in PROD} = orig[p] cross dest[p];

param supply {p in PROD, orig[p]} >= 0;  # 在起始地可用的供应量
param demand {p in PROD, dest[p]} >= 0;  # 在目的地所需的需求量

check {p in PROD}: sum {i in orig[p]} supply[p,i] = sum {j in dest[p]} demand[p,j];

param limit {ORIG, DEST} >= 0;
param cost {p in PROD, links[p]} >= 0;   # 每单位运输量的成本
var Trans {p in PROD, links[p]} >= 0;    # 待运输的单位数量

minimize Total_Cost:
    sum {p in PROD, (i,j) in links[p]} cost[p,i,j] * Trans[p,i,j];

subject to Supply {p in PROD, i in orig[p]}:
    sum {j in dest[p]} Trans[p,i,j] = supply[p,i];

subject to Demand {p in PROD, j in dest[p]}:
    sum {i in orig[p]} Trans[p,i,j] = demand[p,j];

subject to Multi {i in ORIG, j in DEST}:
    sum {p in PROD: (i,j) in links[p]} Trans[p,i,j] <= limit[i,j];
```

图 6-5：使用索引集合的多产品运输模型 (multic.mod)。

```
set orig {PROD} within ORIG;
set dest {PROD} within DEST;
set links {p in PROD} = orig[p] cross dest[p];
```

`links` 的声明表明，可以拥有一个复合集合 (compound sets) 的索引集合，并且可以通过其他索引集合的集合运算来定义索引集合。除了前面提到的运算之外，还有适用于集合的迭代并集 (iterated union) 和交集 (iterated intersection) 运算符，其作用方式类似于数字的迭代求和。例如，表达式

```
union {p in PROD} orig[p]
inter {p in PROD} orig[p]
```

分别表示至少供应一种产品的起始地子集，以及供应所有产品的起始地子集。

图 6-3 中观察到的基于产品的层次关系 (hierarchical relationship) 在图 6-5 的大部分内容中也可见。模型反复处理与特定产品相关的所有起始地、目的地和连接的集合。唯一的例外出现在最后一个约束中，其中求和必须针对通过特定连接运输的所有产品：

```
subject to Multi {i in ORIG, j in DEST}:
    sum {p in PROD: (i,j) in links[p]} Trans[p,i,j] <= limit[i,j];
```

在这里，有必要在 `sum` 后面使用一个略显别扭的索引表达式，来描述一个与层次结构不匹配的集合。

通常来说，几乎所有能够用索引集合编写的模型，也可以用元组集合来编写。正如我们的示例所示，索引集合最适合用于具有层次关系的实体，如产品和区域。另一方面，在处理像起始地和目的地这样对称关联的实体时，元组集合更为合适。

# 练习

6-1. 回到图 4-6 和图 4-7 中的生产与运输模型。使用 display 命令，并结合 6.4 节中演示的索引表达式，你可以确定各种复合集合的成员；例如，你可以使用

```python
ampl: display {j in DEST, p in PROD: demand[j,p] > 500};
set {j in DEST, p in PROD: demand[j,p] > 500} :=
  (DET,coils) (STL,bands) (STL,coils) (FRE,coils);
```

来显示所有需求量大于 500 的产品与目的地的组合集合。

(a) 使用 display 确定以下仅依赖于数据的集合成员：

- 所有生产速率大于每小时 150 吨的起始地与产品的组合。
- 所有运输成本小于等于每吨 10 美元的起始地、目的地和产品的组合。
- 所有线圈运输成本小于等于每吨 10 美元的起始地与目的地的组合。
- 所有每小时生产成本小于 30,000 美元的起始地与产品的组合。
- 所有运输成本超过生产成本 15% 的起始地、目的地和产品的组合。
- 所有运输成本超过生产成本 15% 但低于 25% 的起始地、目的地和产品的组合。

(b) 使用 display 确定以下依赖于最优解以及数据的集合成员：

- 所有生产量至少为 1000 吨的起始地与产品的组合。
- 所有运输量非零的起始地、目的地和产品的组合。
- 所有生产使用时间超过 10 小时的起始地与产品的组合。
- 所有产品占用起始地可用时间超过 25% 的起始地与产品的组合。
- 所有从起始地运输的产品总量至少为 1000 吨的起始地与产品的组合。

6-2. 本练习与上一题类似，但涉及图 6-2 中运输模型的有序对版本。

(a) 使用 display 和索引表达式确定以下集合的成员：

- 起始地-目的地链接的运输成本小于 $10$ 美元/吨。  
- 可由 GARY 服务的目的地。可服务 FRE 的起始地。  
- 在最优解中用于运输的链接。  
- 在最优解中从 CLEV 出发用于运输的链接。  
- 从所有起始地出发，运输总成本超过 $20,000$ 美元的目的地。

(b) 使用 display 命令和 setof 运算符来确定以下集合的成员：

- 从任意起始地 (ORIG) 出发，运输成本超过 20 的目的地 (DEST)。  
- 所有目的地-起始地对 $(j,i)$，使得从 i 到 j 的链路 (LINKS) 在最优解中被使用。

6-3. 使用 display 和适当的集合表达式，从图 6-3 和 6-4 的多周期 (T) 生产模型中确定以下集合的成员：

- 所有被任意产品 (PROD) 服务的市场区域。  
- 所有产品、区域和周的组合，使得在最优解中实际销售量等于最大可销售量。  
- 所有产品和周的组合，使得在所有区域中的总销售量大于或等于 6000 吨。

6-4. 为尝试以下实验，首先输入这些声明：

$\mathsf{ampl}:\mathsf{set}~Q = \{1\ldots 10,1\ldots 10,1\ldots 10,1\ldots 10,1\ldots 10,1\ldots 10\} ;$  
$\mathsf{ampl}:\mathsf{set}~S~\text{within}~Q;$  
$\mathsf{ampl}:\mathsf{data};$  
$\mathsf{ampl}:\mathsf{set}~S\coloneqq 123345234456345567456789;$

(a) 现在尝试以下两个命令：

这两个命令中的表达式表示相同的集合，但你在 AMPL 中是否得到了相同的响应速度？解释造成差异的原因。

(b) 预测命令 display Q 的结果。

6-5. 本练习要求你使用复合集合以多种方式重新表述图 2-1 中的饮食模型。

(a) 重新表述饮食模型，使其使用声明 set GIVE within {NUTR,FOOD}；

以定义营养成分 (NUTR) i 可在食物 (FOOD) j 中找到的对 $(i,j)$ 的子集。

(b) 重新表述饮食模型，使其使用声明

set FN {NUTR} within FOOD;

为每个营养成分 i 定义可提供该营养的所有食物集合 FN[i]。

(c) 重新表述饮食模型，使其使用声明

set NF {FOOD} within NUTR;

为每个食物 j 定义该食物所提供所有营养成分的集合 NF[j]。解释你为何认为这种表述比 (b) 中的更自然和方便。

6-6. 重新阅读第 6.3 节中的建议，并完成以下对多商品运输模型的重新表述：

(a) 使用起始地-目的地对的子集 LINKS。  
(b) 使用起始地-目的地-产品三元组的子集 ROUTES。  
(c) 使用目的地-产品对的子集 MARKETS，其特性为产品 p 可在目的地 j 销售当且仅当 $(j,p)$ 在该子集中。

6-7. 对于图 4-1 中的多商品运输问题 (multicommodity transportation problem)，按照第 6.4 节中的以下两个建议进行增强。

(a) 添加声明

$$
\mathtt{set~DEMAND} = \{\mathtt{j~in~DEST},\mathtt{p~in~PROD}: \mathtt{demand}[\mathtt{j},\mathtt{p}] > 0\} ;
$$

并使变量 (variables) 的索引范围为 {ORIG,DEMAND}，这样变量仅在可能需要满足需求的地方定义。对模型的其余部分做出所有必要的修改，以使用该集合。

(b) 添加声明

set LINKS within {ORIG,DEST}; set TRANSF = {i1 in ORIG, i2 in ORIG: i1 <> i2};

在 `LINKS` 上定义变量以表示运送到目的地的运输量 (shipments)，在 `TRANSF` 上定义变量以表示起始地之间的运输量。现在每个起始地的约束 (constraint) 必须说明总运输量（包括运送到其他起始地以及目的地的运输量）必须等于供应量 (supply) 加上从其他起始地运入的运输量。完成此情形下的模型构建。

6-8. 重新构建练习 3-3(b) 中的模型，使其使用一个允许的工厂-轧钢厂运输对集合 `LINK1`，以及一个允许的轧钢厂-工厂运输对集合 `LINK2`。

6-9. 作为知名科学会议程序委员会的主席，你必须将提交的论文分配给志愿评审员。为了以最有效的方式完成这项工作，你可以构建一个线性规划 (LP) 模型，其思路类似于第 3 章中讨论的指派模型 (assignment model)，但有一些额外的变化。

在浏览了论文和评审员名单后，你可以整理出以下数据：

```ampl
set Papers; 
set Referees; 
set Categories; 
set PaperKind within {Papers, Categories}; 
set Willing within {Referees, Categories};
```

前两个集合的内容是不言自明的，而第三个集合包含论文可能被分类的主题类别。如果论文 $p$ 属于类别 $c$，则集合 `PaperKind` 包含一对 $(p, c)$；通常，一篇论文可以属于多个类别。如果评审员 $r$ 愿意处理类别 $c$ 的论文，则集合 `Willing` 包含一对 $(r, c)$。

(a) 集合 

$$
\{(r, c) \in \text{Willing}, (p, c) \in \text{PaperKind}\}
$$

的维度是多少？该集合中包含的元组有何意义？

(b) 基于对 (a) 的回答，解释为什么声明

```ampl
set CanHandle = setof {(r, c) in Willing, (p, c) in PaperKind} (r, p);
```

给出了集合 $(r, p)$，使得评审员 $r$ 可以被指派论文 $p$。

你的模型可以使用参数 `ppref` 和变量 `Review`，它们均以 `CanHandle` 为索引；`ppref[r, p]` 表示审稿人 $r$ 对论文 $p$ 的偏好值，而 `Review[r, p]` 若审稿人 $r$ 被分配审阅论文 $p$ 则取值为 `1`，否则为 `0`。假设偏好值越高越好，请写出这些组件以及目标函数的声明，目标函数用于最大化所有分配的偏好值总和。

(c) 不幸的是，你还未获得审稿人对具体论文的偏好，因为他们尚未看到任何论文。你所拥有的是他们对不同类别的偏好：

```ampl
param cpref {Willing} integer >= 0, <= 5;
```

解释为何在目标函数中用以下表达式替换 `ppref[r, p]` 是合理的：

```ampl
max {(r, C) in Willing, (p, C) in PaperKind} cpref[r, C]
```

(d) 最后，你必须定义以下参数以表明所需完成的工作量：

```ampl
param nreferees integer > 0;  # 每篇论文所需的审稿人数
param minwork integer > 0;    # 每位审稿人至少需审阅的论文数
param maxwork integer >= minwork;  # 每位审稿人最多可审阅的论文数
```

制定适当的分配约束。通过制定约束条件完成模型，确保每篇论文都获得所需数量的审稿人，且每位审稿人都被分配到合理数量的论文。