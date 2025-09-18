# 简单集合与索引

本书接下来的四章将全面介绍 AMPL 在线性规划 (linear program) 方面的功能。这里的组织方式是按照语言特性进行的，而不是像前四章教程那样按照模型类型进行组织。由于 AMPL 的基本特性之间密切相关，我们不会试图孤立地解释某一特性。相反，我们假设读者已经具备了第 1 至第 4 章所介绍的 AMPL 基础知识。

我们从集合 (set) 开始介绍，集合是 AMPL 模型中最基本的组成部分。在典型的模型中，几乎所有参数 (parameter)、变量 (variable) 和约束 (constraint) 都是基于集合进行索引的，许多表达式也包含对集合的操作（通常是求和）。集合索引 (set indexing) 这一特性使得简洁的模型能够描述大型数学规划问题。

由于集合如此基础，AMPL 提供了多种类型的集合和操作。集合的成员可以是字符串或数字，可以是有序的或无序的；它们可以单独出现，也可以作为有序对、三元组或更长的“元组” (tuple) 出现。可以通过显式列出或计算成员、对其他集合进行并集 (union) 和交集 (intersection) 等操作，或者指定任意算术或逻辑条件来定义集合。

任何模型组件或迭代操作都可以使用标准形式的索引表达式 ( indexing expression ) 对任何集合进行索引。甚至集合本身也可以声明为基于其他集合索引的集合。

本章将介绍较简单的集合类型，以及集合操作和索引表达式；最后将讨论有序集合。第 6 章将展示这些概念如何扩展到复合集合，包括成对和三元组的集合，以及索引集合族。第 7 章专门讨论参数和表达式，第 8 章则讨论构成线性规划的变量、目标函数和约束。

## 5.1 无序集合

最基本的 AMPL 集合是一种无序的字符串集合。通常，集合中的所有字符串都用于表示同一类型的实例——例如原材料、产品、工厂或城市。通常这些字符串会被选择为具有可识别意义的名称（如 coils、FISH、New_York），但它们同样可以是仅为建模者所知的代码（如 23RPFG、486/33C）。在 AMPL 模型中出现的字符串必须用引号括起来，可以是单引号（如 'A&P'）或双引号（如 "Bell+ Howell"）。在所有上下文中，大写字母和小写字母是有区别的，因此例如 "fish"、"Fish" 和 "FISH" 表示不同的集合成员。

集合的声明只需要包含关键字 set 和一个名称。例如，一个模型可以声明

```
set PROD;
```

以表明在模型的其余部分将通过名称 PROD 来引用某个特定集合。名称可以是任意由字母、数字和下划线（_）组成的序列，只要它不是一个合法的数字即可。有一些名称在 AMPL 中具有特殊含义，只能用于特定目的；而更多的名称具有预定义的含义，如果以其他方式使用则可以更改其含义。例如，sum 被保留用于迭代加法运算符；而 prod 仅预定义为迭代乘法运算符，因此你可以将 prod 重新定义为一个产品集合：

```
set prod;
```

保留字列表见 A.1 节。

声明的集合成员通常在模型的数据部分中指定，其方式将在第 9 章中描述；对于大多数数学规划应用，建议将模型和数据分开。然而，有时在模型中引用特定的字符串集合是必要的。这种字面量集合通过在花括号内列出其成员来指定：

$$
\{\mathrm{"bands", "coils", "plate"}\}
$$

该表达式可以在任何集合有效的地方使用，例如在模型语句中为集合 PROD 指定固定的成员：

$$
\mathsf{set\ PROD} = \{\mathrm{"bands", "coils", "plate"}\};
$$

这种声明最好仅限于集合成员较少、是模型的基本组成部分或不常更改的情况。尽管如此，我们会发现 = 语句在集合声明中通常很有用，其目的是通过其他集合和参数来定义一个集合。运算符 = 可以替换为 default，用于初始化集合，同时允许其值被数据语句覆盖或通过后续赋值更改。不过这些选项对于参数更为重要，因此我们将在 7.5 节中更详细地讨论它们。

请注意，AMPL 区分诸如 `bands` 的字符串和仅包含一个字符串的集合 `{ "bands" }`。没有成员的集合（空集）记为 $\{\}$。

## 5.2 数字集合

集合成员也可以是数字。实际上，一个集合的成员可以是数字和字符串的混合体，尽管这种情况很少见。在 AMPL 模型中，字面数字按惯例写成一串数字，可选地前面加一个符号，包含一个可选的小数点，并可选地后面跟一个指数；指数由 d、D、e 或 E 组成，可选地跟一个符号和一串数字。数字 (1) 和相应的字符串 ("1") 是不同的；相比之下，表示相同数字的不同形式，如 100 和 $\pm \mathbb{E} + \mathbb{Z}$，代表相同的集合成员。

一组数字通常是一个序列，对应于所建模情况中的某种进展，例如一系列周或年。就像字符串一样，集合中的数字可以作为数据的一部分指定，也可以在模型中以大括号之间的列表形式指定，例如 $\{1,2,3,4,5,6\}$。这种集合可以通过符号 1 . . 6 更简洁地描述。可以使用附加的 by 子句指定数字之间的间隔而不是 1；例如，

1990 . . 2020 by 5

表示集合

$$
\{1990,1995,2000,2005,2010,2015,2020\}
$$

这种表达式可以在任何适合集合的地方使用，特别是在集合声明的赋值短语中：

set YEARS $=$ 1990 . . 2020 by 5;

通过给集合一个简短而有意义的名称，此声明可以帮助使模型的其余部分更易读。

除非这些数字的值对模型至关重要或很少更改，否则不建议通过像 2020 和 5 这样的字面量指定 . . 表达式中的所有数字。在图 4-4 和 4-5 的多期生产示例中可以看到更好的安排，其中声明了一个参数 T 来表示周期数，并使用表达式 1 . . T 和 0 . . T 来表示参数、变量、约束和求和索引的周期集。T 的值在数据中指定，因此可以轻松地从一次运行更改为下一次运行。作为一个更复杂的例子，我们可以写成

param start integer; param end $\gimel$ start integer; param interval $\gimel$ 0 integer; set YEARS $=$ start . . end by interval;

如果随后我们给出数据为

param start : $=$ 1990; param end : $=$ 2020; param interval : $=$ 5;

那么 YEARS 将与前一个示例中的集合相同（如果 end 是 2023 也是如此）。您可以使用任何算术表达式来表示 . . 表达式中的任何值。

数字集合的成员具有与其他任何数字相同的属性，因此可以在算术表达式中使用。图 4-4 中可以看到一个简单的例子，其中物料平衡约束声明为 subject to Balance {p in PROD, t in 1 . . T}: Make[p,t] + Inv[p,t-1] = Sell[p,t] + Inv[p,t]:

由于 t 遍历集合 1. .T，我们可以写 Inv[p,t- 1] 来表示前一周末的库存。如果 t 遍历的是一个字符串集合，则表达式 t- 1 将会因为错误而被拒绝。

集合成员不必是整数。AMPL 会尝试将每个数值型集合成员存储为最接近的可表示浮点数。你可以通过以下实验来查看在你的计算机上这是如何工作的：

```ampl
ampl: option display_width 50;
ampl: display { -5/3 .. 5/3 by 1/3 };
set -5/3 .. 5/3 by 1/3 :=
-1.6666666666666667  -1.3333333333333335  -1  -0.666666666666667  -0.333333333333335
-2.220446049250313e-16
0.3333333333333326   0.666666666666663    0.9999999999999998  1.333333333333333
1.666666666666663;
```

你可能期望 0 和 1 是这个集合的成员，但由于浮点运算中的舍入误差，情况并非如此。如果你的模型依赖于集合成员具有精确值，那么在集合中使用分数是不明智的。对于合理大小的整数成员，应该没有类似的问题；对于 IEEE 标准算术，整数在 $2^{53}$ (约 $10^{16}$) 以内可以精确表示，而对于当前使用的几乎所有计算机，整数在 $2^{47}$ (约 $10^{14}$) 以内可以精确表示。

## 5.3 集合运算

AMPL 有四个运算符可以从现有集合构造新集合：

- `A union B` 并集：在 A 或 B 中  
- `A inter B` 交集：在 A 和 B 中  
- `A diff B` 差集：在 A 中但不在 B 中  
- `A symdiff B` 对称差集：在 A 或 B 中但不同时在两者中  

以下 AMPL 会话摘录展示了这些运算符的工作方式：

```ampl
ampl: set Y1 := 1990 .. 2020 by 5;
ampl: set Y2 := 2000 .. 2025 by 5;
ampl: display Y1 union Y2, Y1 inter Y2;
set Y1 union Y2 := 1990 1995 2000 2005 2010 2015 2020 2025;
set Y1 inter Y2 := 2000 2005 2010 2015 2020;

ampl: display Y1 diff Y2, Y1 symdiff Y2;
set Y1 diff Y2 := 1990 1995;
set Y1 symdiff Y2 := 1990 1995 2025;
```

集合运算符的操作数可以是其他集合表达式，从而允许构建更复杂的表达式：

```ampl
ampl: display Y1 symdiff (Y1 symdiff Y2);
set Y1 symdiff (Y1 symdiff Y2) := 2000 2005 2010 2015 2020 2025;

ampl: display (Y1 union {2025,2035,2045}) diff Y2;
set (Y1 union {2025,2035,2045}) diff Y2 := 1990 1995 2035 2045;

ampl: display (2000 .. 2040 by 5) symdiff (Y1 union Y2);
set (2000 .. 2040 by 5) symdiff (Y1 union Y2) := 2030 2035 2040 1990 1995;
```

然而，操作数必须始终表示集合，因此例如你必须写 `Y1 union {2025}`，而不是 `Y1 union 2025`。

除非使用括号指示其他方式，集合运算符从左到右结合。`union`、`diff` 和 `symdiff` 运算符具有相同的优先级，仅低于 `inter` 运算符。因此，例如，

`A union B inter C diff D` 被解析为  
`(A union (B inter C)) diff D`

所有 AMPL 运算符的优先级层次结构见 A.4 节的表 A-1。

集合运算经常用于集合声明的赋值语句中，以便根据已声明的集合来定义新集合。一个简单的例子是对图 2-1 中饮食模型的变体。与其为每种营养素的数量指定下限和上限，不如指定一组具有下限的营养素和一组具有上限的营养素（每种营养素都属于其中一个集合；某些营养素可能同时属于两个集合）。你可以声明：

```ampl
set MINREQ;  # 具有最小需求 (minimum requirements) 的营养素
set MAXREQ;  # 具有最大需求 (maximum requirements) 的营养素
set NUTR;    # 所有营养素 (DUBIOUS)
```

但这样一来，你将依赖模型的使用者确保 `NUTR` 恰好包含 `MINREQ` 和 `MAXREQ` 的所有成员。最好的情况下这是不必要的工作，最坏的情况下则可能出错。相反，你可以将 `NUTR` 定义为其并集：

```ampl
set NUTR = MINREQ union MAXREQ;
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/bdc500da88abbfd3adeb01b86660bcc37a4e103f0cb45974717ee6c29a68bba7.jpg)  
图 5-1：使用并集运算符的饮食模型 (dietu.mod)

需要这三个集合，因为营养素的最小值和最大值分别以 `MINREQ` 和 `MAXREQ` 为索引，

$$
\begin{array}{rl}
& {\tt param~min~\{MINREQ\}~>=~0;} \\
& {\tt param~max~\{MAXREQ\}~>=~0;}
\end{array}
$$

而食物中营养素的含量则以 `NUTR` 为索引：

$$
\mathtt{param~amt~\{NUTR,FOOD\}~>=~0};
$$

模型其余部分的修改是直接的；结果如图 5-1 所示。

作为一个一般原则，让模型需要提供冗余信息是一个坏主意。相反，应选择在数据中提供最小必要的集合集合，而其他相关集合则在模型中通过表达式定义。

# 5.4 集合成员关系运算和函数

另外两个 AMPL 运算符 `in` 和 `within` 用于测试集合的成员关系。例如，表达式

```ampl
"B2" in NUTR
```

当且仅当字符串 `"B2"` 是集合 `NUTR` 的成员时为真。表达式

```ampl
MINREQ within NUTR
```

当集合 MINREQ 的所有成员同时也是集合 NUTR 的成员时（即 MINREQ 是 NUTR 的子集，或与 NUTR 相同），该表达式为真。运算符 in 和 within 在 AMPL 中分别对应传统代数记号中的 $\in$ 和 $\subseteq$。在这里，成员与集合之间的区别尤为重要：in 的左操作数必须是一个求值为字符串或数字的表达式，而 within 的左操作数必须是一个求值为集合的表达式。

AMPL 还提供了 not in 和 not within 运算符，它们会将结果的真值取反。

你可以直接将 within 应用于正在声明的集合，以说明它必须是某个其他集合的子集。回到饮食模型的例子，如果所有营养成分都有最小需求，但只有部分营养成分有最大需求，那么可以这样声明集合：

set NUTR; set MAXREQ within NUTR;

如果为 MAXREQ 指定的任何成员不属于 NUTR，AMPL 将拒绝该模型的数据。

内置函数 card 用于计算集合中的成员数量（即集合的基数）；例如，card(NUTR) 表示集合 NUTR 中成员的数量。card 函数的参数可以是任何求值为集合的表达式。

# 5.5 索引表达式

在代数记号中，集合的使用通常通过诸如“对所有 $i \in P$”、“对 $t = 1, \ldots ,T$”或“对所有满足 $c_{j} > 0$ 的 $j \in R$”这样的短语非正式地表示。在 AMPL 中，与之对应的是索引表达式 (indexing expression)，它出现在几乎所有示例的花括号 $\{\ldots \}$ 中。当我们指定模型组件所基于的索引集合，或指定求和运算所涉及的集合时，都会使用索引表达式。由于索引表达式定义了一个集合，因此可以在任何需要集合的地方使用它。

索引表达式的最简单形式就是括号内直接写一个集合名称或表达式。我们在图 4-4 的多周期生产模型中的参数声明中已经见过这种用法：

$$
\begin{array}{rl} 
& {\tt param~rate~\{PROD\} > 0;} \\ 
& {\tt param~avail~\{1..T\} >= 0;} 
\end{array}
$$

在模型的后续部分，对这些参数的引用通过单个集合成员进行下标，例如 avail[t] 和 rate[p]。变量也可以以完全相同的方式声明和使用，只是关键字 var 替代了 param。

在我们模型的下标及其他表达式中出现的名称，如 t 和 i，是索引表达式定义的哑索引 (dummy indices) 的例子。实际上，任何索引表达式都可以选择性地定义一个在其指定集合上运行的哑索引。哑索引在指定参数的边界时非常方便：

param f_min {FOOD} >= 0; param f_max {j in FOOD} >= f_min[j];

以及变量的边界：

$$
\mathtt{var~Buy~\{j~in~FOOD\}~>=~f\_min[j],~<=~f\_max[j]~;}
$$

它们在指定约束条件所依赖的集合以及求和运算所涉及的集合时也至关重要。我们经常看到这些用途结合在一起，例如在以下声明中：

subject to Time {t in 1..T}: sum {p in PROD} (1/rate[p]) * Make[p,t] <= avail[t];

以及

subject to Diet_Min {i in MINREQ}: sum {j in FOOD} amt[i,j] * Buy[j] >= n_min[i];

一个索引表达式包括一个索引名称、关键字 in 以及一个集合表达式，与之前一致。我们一直使用单个字母作为索引名称，但这并不是必须的；索引名称可以是任何不是有效数字的字母、数字和下划线序列，就像模型组件的名称一样。

虽然由模型组件声明所定义的名称在整个模型后续的所有语句中都是已知的，但虚拟索引名称 (dummy index name) 的定义仅在其所属的索引表达式范围内有效。通常，这个范围可以从上下文中明显看出。例如，在上面的 Diet_Min 声明中，{i in MINREQ} 的范围延伸到语句末尾，因此 i 可以在约束条件描述的任何地方使用。另一方面，{j in FOOD} 的范围仅覆盖被加数 amt[i,j] * Buy[j]。关于求和及其他迭代运算符的索引表达式范围，将在第 7 章中进一步讨论。

一旦索引表达式的范围结束，其虚拟索引就会变为未定义。因此，同一个索引名称可以在模型中反复定义，实际上，使用相对较少的不同索引名称是一种良好的实践。一种常见的约定是将某些索引名称与某些集合关联起来，例如 i 总是遍历 NUTR，j 总是遍历 FOOD。然而，这仅仅是一种约定，而不是 AMPL 所施加的限制。事实上，当我们修改饮食模型，使其包含 NUTR 的一个子集 MINREQ 时，我们曾使用 i 来遍历 MINREQ 以及 NUTR。相反的情况也会出现，例如，如果我们想指定一个约束条件，即饮食中每种食物 j 的数量至少是饮食中食物总量的某个比例 min_frac[j]：

subject to Food_Ratio {j in FOOD}: Buy[j] >= min_frac[j] * sum {jj in FOOD} Buy[jj];

由于 j in FOOD 的范围延伸到声明末尾，因此在约束条件内部的求和运算中，定义了一个不同的索引 jj 来遍历集合 FOOD。

作为最后一种选择，索引表达式中的集合后面可以跟一个冒号 (:) 和一个逻辑条件。该索引表达式则仅表示满足该条件的成员子集。例如，

$$
\{\texttt{j in FOOD: f\_max[j] - f\_min[j] < l}\}
$$

描述了所有最小和最大数量几乎相同的食品集合，以及

$$
\{\texttt{i in NUTR: i in MAXREQ or n\_min[i] > 0}\}
$$

描述了位于 MAXREQ 中或 n_min 为正的营养素集合。诸如 or 和 $<$ 等运算符用于构成逻辑条件的用法将在第 7 章中详细解释。

通过指定一个条件，索引表达式定义了一个新集合。你不仅可以在带索引的声明和求和中使用该索引表达式来表示此集合，在其他任何可以出现集合表达式的地方也可以使用。例如，你可以使用以下任一方式

```ampl
set NUTREQ = {i in NUTR: i in MAXREQ or n_min[i] > 0};
set NUTREQ = MAXREQ union {i in MINREQ: n_min[i] > 0};
```

来定义 NUTREQ，以表示我们前面示例中的集合表达式；你也可以使用以下任一方式

```ampl
set BOTHREQ = {i in MINREQ: i in MAXREQ};
set BOTHREQ = MINREQ inter MAXREQ;
```

来定义 BOTHREQ 为同时具有最小和最大需求的所有营养素的集合。对于某些复杂的集合，通常会发现有多种描述方式，这取决于你如何组合集合操作和索引表达式的条件。当然，有些方式比其他方式更易读，因此值得花些精力找出最易读的表达方式。在第 6 章中，我们还将讨论在指定复合集合时，出于效率考虑有时会使某种选择优于其他选择。

除了在模型内部有用之外，索引表达式在 display 语句中也很有用，可以用来总结数据或解的特征。以下示例基于图 5-1 的模型和图 5-2 的数据：

```ampl
ampl: model dietu.mod;
ampl: data dietu.dat;
ampl: display MAXREQ union {i in MINREQ: n_min[i] > 0};
set MAXREQ union {i in MINREQ: n_min[i] > 0} := A NA CAL C;

ampl: solve;
CPLEX 8.0.0: optimal solution; objective 74.27382022
2 dual simplex iterations (0 in phase I)

ampl: display {j in FOOD: Buy[j] > f_min[j]};
set {j in FOOD: Buy[j] > f_min[j]} := CHK MTL SPG;

ampl: display {i in MINREQ: Diet_Min[i].slack = 0};
set {i in MINREQ: (Diet_Min[i].slack) == 0} := C CAL;
```

AMPL 的交互式命令允许在索引表达式的条件部分引用变量和约束，如上述最后两个 display 语句所示。然而，在模型内部，任何索引表达式中只能提及集合、参数和虚拟索引。

上述集合 BOTHREQ 很可能为空集，例如当每个营养素在数据中要么具有最小需求，要么具有最大需求，但不会同时具有两者时。对空集进行索引并不是错误。当模型中的某个组件被声明为在某个最终为空的集合上进行索引时，AMPL 会简单地跳过生成该组件。对空集求和的结果为零，其他在空集上的迭代运算符也有明显的解释（见 A.4 节）。

# 5.6 有序集合

任何一组数字都有自然的排序，因此数字经常被用来表示实体，例如时间周期，其排序对于模型的规范至关重要。为了描述本周库存与上周库存之间的差异，例如，我们需要对周进行排序，以便“上一个”周总是明确定义的。

一个 AMPL 模型也可以通过在集合的声明中添加关键字 ordered 或 circular，为任何一组数字或字符串定义自己的排序。你在模型或数据中给出集合成员的顺序，就是 AMPL 处理它们的顺序。在声明为 circular 的集合中，第一个成员被认为跟在最后一个成员之后，而最后一个成员被认为在第一个成员之前；在 ordered 集合中，第一个成员没有前驱，最后一个成员没有后继。

字符串的有序集合通常比数字集合为模型的数据提供更好的文档。回到图 4-4 的多周期生产模型，我们观察到，从数据中无法判断数字 1 到 T 指的是哪些周，甚至无法判断它们是周而不是天或月。假设我们让周由一个有序集合表示，该集合包含，例如，27sep、04oct、11oct 和 18oct。T 的声明被替换为

set WEEKS ordered;

并且所有后续出现的 1..T 都被替换为 WEEKS。在 Balance 约束中，表达式 t-1 被替换为 prev(t)，它选择集合排序中 t 之前的成员：

subject to Balance {p in PROD, t in WEEKS}: Make[p,t] + Inv[p,prev(t)] = Sell[p,t] + Inv[p,t] # 错误

然而，这并不完全正确，因为当 t 是 WEEKS 中的第一个周时，成员 prev(t) 未定义。当你尝试解决这个问题时，你会收到如下错误消息：

error processing constraint Balance['bands','27sep']
        can't compute prev('27sep', WEEKS) - '27sep' is the first member

解决此问题的一种方法是为第一个周期提供一个单独的平衡 (Balance) 约束，其中 `Inv[p,prev(t)]` 被替换为初始库存 `inv0[p]`：

```ampl
subject to Balance0 {p in PROD}:
   Make[p,first(WEEKS)] + inv0[p] = Sell[p,first(WEEKS)] + Inv[p,first(WEEKS)];
```

常规的平衡约束仅限于剩余的周期：

```ampl
subject to Balance {p in PROD, t in WEEKS: ord(t) > 1}:
   Make[p,t] + Inv[p,prev(t)] = Sell[p,t] + Inv[p,t];
```

完整的模型和数据如图 5-3 和图 5-4 所示。作为对更有意义的周期名称的权衡，我们必须编写一个稍微复杂一点的模型。

正如我们的示例所示，AMPL 提供了多种专门应用于有序集合的函数。这些函数有三种基本类型。

集合

```ampl
set PROD;            # 产品 (Product) 集合
set WEEKS ordered;   # 周数

param rate {PROD} > 0;      # 每小时生产的吨数
param inv0 {PROD} >= 0;     # 初始库存
param avail {WEEKS} >= 0;   # 每周可用小时数
param market {PROD,WEEKS} >= 0;  # 每周销售吨数限制
param procost {PROD} >= 0;       # 每吨生产成本
param invcost {PROD} >= 0;       # 每吨库存成本
param revenue {PROD,WEEKS} >= 0; # 每吨销售收入

var Make {PROD,WEEKS} >= 0;      # 生产的吨数
var Inv {PROD,WEEKS} >= 0;       # 库存吨数
var Sell {p in PROD, t in WEEKS} >= 0, <= market[p,t]; # 销售吨数
```

目标函数

```ampl
maximize Total_Profit:
   sum {p in PROD, t in WEEKS}
      (revenue[p,t]*Sell[p,t] - procost[p]*Make[p,t] - invcost[p]*Inv[p,t]);
   # 目标：所有周期的总收入减去成本
```

约束条件

```ampl
subject to Time {t in WEEKS}:
   sum {p in PROD} (1/rate[p]) * Make[p,t] <= avail[t];
   # 所有产品使用的总小时数
   # 不得超过每周可用小时数

subject to Balance0 {p in PROD}:
   Make[p,first(WEEKS)] + inv0[p] = Sell[p,first(WEEKS)] + Inv[p,first(WEEKS)];

subject to Balance {p in PROD, t in WEEKS: ord(t) > 1}:
   Make[p,t] + Inv[p,prev(t)] = Sell[p,t] + Inv[p,t];
   # 生产的吨数和从库存中取出的吨数
   # 必须等于销售的吨数和放入库存的吨数
```

首先，有一些函数可以从集合的某个绝对位置返回一个成员。你可以写 `first(WEEKS)` 和 `last(WEEKS)` 来表示有序集合 `WEEKS` 的第一个和最后一个成员。要挑选其他成员，可以使用 `member(5, WEEKS)` 来表示 `WEEKS` 的第 5 个成员。这些函数的参数必须计算为一个有序集合，除了 `member` 的第一个参数，它可以是任何计算为正整数的表达式。

第二种函数是从相对于另一个成员的位置返回一个成员。因此你可以写 `prev(t, WEEKS)` 表示 `WEEKS` 中 `t` 之前的成员，`next(t, WEEKS)` 表示 `t` 之后的成员。更一般地，表达式如 `prev(t, WEEKS, 5)` 和 `next(t, WEEKS, 3)` 分别表示 `WEEKS` 中 `t` 之前第 5 个成员和之后第 3 个成员。还有“循环”版本 `prevw` 和 `nextw`，它们的工作方式相同，只是将集合的末尾视为环绕到开头；实际上，它们将所有有序集合视为其声明是循环的。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/e77080ca38a63bb2fa5ff5a9b61e7edea49a5613a7e74ce031bbcc839a853024.jpg)  
图 5-4：生产模型数据 (steelT2.dat)。

在所有这些函数中，第一个参数必须计算为一个数字或字符串，第二个参数为一个有序集合，第三个参数为一个整数。通常这个整数是正数，但零和负值也以一致的方式解释；例如，next(t, WEEKS, 0) 与 t 相同，而 next(t, WEEKS, -5) 与 prev(t, WEEKS, 5) 相同。

最后，有一些函数可以返回成员在集合中的位置。表达式 `ord(t, WEEKS)` 返回 `t` 在集合 `WEEKS` 中的数值位置，如果 `t` 不是 `WEEKS` 的成员，则会给出错误信息。另一种形式 `ord0(t, WEEKS)` 与其类似，不同之处在于当 `t` 不是 `WEEKS` 的成员时，它返回 `0`。对于这些函数，第一个参数必须求值为一个正整数，第二个参数必须为一个有序集合。

如果 `next`、`nextw`、`prev`、`prevw` 或 `ord` 的第一个参数是一个在有序集合上运行的虚拟索引，并且第二个参数未指定集合，则假定其关联的索引集合。因此，在如下约束中：

```
subject to Balance {p in PROD, t in WEEKS: ord(t) > 1};
Make[p,t] + Inv[p,prev(t)] = Sell[p,t] + Inv[p,t];
```

函数 `ord(t)` 和 `prev(t)` 被解释为如果写成 `ord(t, WEEKS)` 和 `prev(t, WEEKS)` 一样。

有序集合也可以与任何适用于一般集合的 AMPL 运算符和函数一起使用。`diff` 运算的结果保留左操作数的顺序，因此在我们的示例中，物料平衡约束可以写成：

```
subject to Balance {p in PROD, t in WEEKS diff {first(weeks)}}:
Make[p,t] + Inv[p,prev(t)] = Sell[p,t] + Inv[p,t];
```

然而，对于 `union`、`inter` 和 `symdiff`，结果的顺序是未定义的；AMPL 将结果视为无序集合。

对于包含在有序集合中的集合，AMPL 提供了一种方式来表示应继承排序。例如，假设你想尝试使用不同长度的时间范围来运行多周期生产模型。在以下声明中，有序集合 `ALL_WEEKS` 和参数 `T` 在数据中给出，而子集 `WEEKS` 则通过索引表达式定义，仅包括前 `T` 周：

```
set ALL_WEEKS ordered;
param T > 0 integer;
set WEEKS = {t in ALL_WEEKS: ord(t) <= T} ordered by ALL_WEEKS;
```

我们指定 `ordered by ALL_WEEKS`，以便 `WEEKS` 成为一个有序集合，其成员与在 `ALL_WEEKS` 中的顺序相同。`ordered by` 和 `circular by` 短语与第 5.4 节中的 `within` 短语加上 `ordered` 或 `circular` 具有相同的效果，不同之处在于它们还会使声明的集合继承包含集合的排序。还有 `ordered by reversed` 和 `circular by reversed` 短语，它们使声明集合的排序与包含集合的排序相反。所有这些短语既可以用于数据中提供的子集，也可以用于通过表达式定义的子集，如上面的示例所示。

# 预定义集合和区间表达式

AMPL 为某些常见的区间和其他集合（这些集合要么是无限的，要么可能非常大）提供了特殊名称和表达式。索引表达式不能在这些集合上迭代，但它们在指定集合和参数声明中的条件短语时非常方便。

AMPL 区间是包含两个边界之间所有数字的集合。有实数 (浮点数) 和整数的区间，分别由关键字 `interval` 和 `integer` 引入。它们可以被指定为闭区间、开区间或半开区间，遵循标准数学符号，

$$
\begin{array}{rl}
& \texttt{interval} [a,b]\equiv \{x\colon a\leq x\leq b\} ,\\
& \texttt{interval} (a,b]\equiv \{x\colon a< x\leq b\} ,\\
& \texttt{interval} [a,b)\equiv \{x\colon a\leq x< b\} ,\\
& \texttt{interval} (a,b)\equiv \{x\colon a< x< b\} ,\\
& \texttt{integer} [a,b]\equiv \{x\in I\colon a\leq x\leq b\} ,\\
& \texttt{integer} (a,b]\equiv \{x\in I\colon a< x\leq b\} ,\\
& \texttt{integer} [a,b)\equiv \{x\in I\colon a\leq x< b\} ,\\
& \texttt{integer} (a,b)\equiv \{x\in I\colon a< x< b\}
\end{array}
$$

其中 a 和 b 是任意算术表达式，$I$ 表示整数集。在声明短语中 `interval within interval ordered by [reversed ] interval circular by [reversed ] interval` 关键字 `interval` 可以省略。

例如，在声明第 1 章的参数 `rate` 时，你可以声明 `param rate {PROD} in interval (0,maxrate];` 表示生产率必须大于零且不超过某个先前定义的参数 `maxrate`；你可以更简洁地写成 `param rate {PROD} in (0,maxrate];` 或等价地写成 `param rate {PROD} >0, <= maxrate;`

可以通过使用预定义的 AMPL 参数 `Infinity` 作为右边界，或 `-Infinity` 作为左边界来指定无界区间，因此 `param rate {PROD} in (0,Infinity];` 与 `param rate {PROD} >0;` 在图 1-4a 中表示完全相同的意思。一般来说，区间不会让你在集合或参数声明中说出任何新内容；它们只是给你提供了另一种表达方式。（它们在定义导入函数时有更重要的作用，如 A.22 节所述。）

预定义的无限集 `Reals` 和 `Integers` 分别是所有浮点数和整数的集合，按数值顺序排列。预定义的无限集 `ASCII`、`EBCDIC` 和 `Display` 都表示字符串和数字的全集，任何一维集合的成员都从中抽取。`ASCII` 和 `EBCDIC` 分别按 ASCII 和 EBCDIC 排序序列排序。`Display` 使用 AMPL 的 `display` 命令中的排序（A.16 节）：数字在文字之前并按数值排序；文字按 ASCII 排序序列排序。

例如，你可以声明 `set PROD ordered by ASCII;` 使 `PROD` 成员的 AMPL 排序为字母顺序，无论它们在数据中的排序如何。这种对 `PROD` 成员的重新排序不会影响图 1-4a 中模型的解，但它使 AMPL 列出的大多数以 `PROD` 为索引的实体按相同顺序出现（见 A.6.2）。

# 练习

5-1. (a) 显示集合

- 5/3.5/3 by 1/3 0.1 by .1

解释你的计算机算术中舍入误差的任何证据。

(b) 在你的计算机上尝试以下来自第 5.2 节和第 5.4 节的命令：

```ampl
ampl: set HUGE = 1..1e7;
ampl: display card(HUGE);
```

当 AMPL 内存不足时，它会显示有多少字节可用？（如果你的计算机确实有足够的内存，请尝试 `1..1e8`。）通过实验看看你的计算机在不耗尽内存的情况下最多能容纳多大的集合 HUGE。

5-2. 修改练习 1-6 的模型，使其使用两个不同的属性集合：一个具有下限的属性集合和一个具有上限的属性集合。使用与图 5-1 中相同的方法。

5-3. 使用 `display` 命令，并结合索引表达式（如第 5.5 节所示），确定与图 5-1 和图 5-2 中饮食模型相关的以下集合：

- 单位成本大于 \$2.00 的食物表。
- 钠 (NA) 含量超过 1000 的食物表。
- 在最优解中对总成本贡献超过 \$10 的食物表。
- 在最优解中购买量超过最低水平但低于最高水平的食物表。
- 最优饮食中供给量恰好为最低允许量的营养素。
- 最优饮食中供给量恰好为最高允许量的营养素。
- 最优饮食中供给量超过最低允许量但低于最高允许量的营养素。

5-4. 本练习涉及图 4-4 中的多周期生产模型。

(a) 假设我们定义两个额外的标量参数，

```ampl
param Tbegin integer >= 1;
param Tend integer >= Tbegin, <= T;
```

我们只想求解覆盖从 Tbegin 到 Tend 周的线性规划。但我们仍然希望参数使用索引 `1..T`，这样我们就不需要在每次尝试不同的 Tbegin 或 Tend 值时更改数据表。

首先，我们可以将变量、目标函数和约束声明中的每个 `1..T` 更改为 `Tbegin..Tend`。通过进行这些及其他必要的更改，创建一个正确覆盖所需周数的模型。

(b) 现在假设我们定义一个不同的标量参数，

```ampl
param Tagg integer >= 1;
```

我们想要“聚合”模型，使得我们的线性规划中的一个“周期”变为 Tagg 周长，而不是一周。如果我们有一年的每周数据，这将是合适的，因为这会产生一个太大而不便于分析的线性规划。

为了正确聚合，我们必须将每个周期中的工时可用性定义为该周期内所有周的可用性之和：

```ampl
param avail_agg {t in 1..T by Tagg} = sum {u in t..t+Tagg-1} avail[u];
```

参数 `market` 和 `revenue` 也必须类似地求和。对图 4-4 中的模型进行所有必要的更改，使得生成的线性规划被正确聚合。

(c) 重新编写 (a) 和 (b) 中的模型，使其使用有序的字符串集合来表示周期，如图 5-3 所示。

5-5. 将图 3-1a 中的运输模型扩展为多周期版本，其中周期为月份，用有序的字符串集合表示，例如 "Jan"、"Feb" 等。使用起始地的库存来连接各个周期。

5-6. 修改图 5-3 中的模型，将 Balance0 和 Balance 约束合并，如图 4-4 所示。提示：0..T 和 1..T 类似于

```ampl
set WEEKS0 ordered;
set WEEKS = {i in WEEKS0: ord(i) > 1} ordered by WEEKS0;
```