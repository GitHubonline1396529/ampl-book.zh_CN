# 参数 (Parameter) 与表达式

一个大型优化模型通常会使用许多数值。正如我们之前所解释的，只需要在 AMPL 模型中对这些数值进行简洁的符号描述，而具体的数值则在单独的数据语句中给出，相关内容将在第 9 章中介绍。

在 AMPL 中，单个命名的数值称为参数 (parameter)。尽管某些参数被定义为单独的标量值，但大多数参数以向量、矩阵或其他集合的形式出现，这些集合通常以集合作为索引。因此，在语义清晰的情况下，我们有时会将索引化的参数集合统称为“一个参数”。本章开始的 7.1 节将介绍在 AMPL 模型中声明参数以及引用参数的规则。

参数及其他数值是构成模型目标函数与约束表达式的构建基础。第 7.2 和 7.3 节分别介绍了算术表达式（具有数值）和逻辑表达式（结果为真或假）。除了传统代数符号中的标准一元和二元运算符之外，AMPL 还提供了诸如 `sum` 和 `prod` 的迭代运算符，以及条件运算符（if-then-else），它根据第三个表达式的真假在两个表达式之间进行选择。

目标函数与约束中的表达式必然涉及变量，其声明与使用将在第 8 章中讨论。然而，有一些常见的用途是仅涉及集合与参数的表达式。7.4 节描述了如何使用逻辑表达式来检验数据的有效性，可以直接在参数声明中使用，也可以在独立的 check 语句中使用。7.5 节介绍了通过先前声明的参数与集合中的算术表达式来定义新参数的功能，而 7.6 节则描述了随机生成参数的方法。

尽管参数的主要目的是表示数值，但它们也可以表示逻辑值或任意字符串。这些可能性分别在 7.7 节与 7.8 节中介绍。AMPL 提供了一系列用于字符串的运算符，但由于它们最常用于 AMPL 命令和编程而非模型中，因此我们将它们的介绍推迟到 13.7 节。

# 7.1 参数 (Parameter) 声明

参数声明描述了模型所需的某些数据，并指出模型在后续表达式中将如何引用数据值。

最简单的参数声明由关键字 param 和一个名称组成：

param T;

在此声明之后的任意位置，T 都可用于引用一个数值。

更常见的情况是，参数声明中的名称后跟随一个索引表达式：

param avail {1..T};  
param demand {DEST, PROD};  
param revenue {p in PROD, AREA[p], 1..T};



对于索引表达式所指定集合的每个成员，都会定义一个参数。因此，一个参数由其名称及其关联的集合成员唯一确定；在模型的其余部分中，通过书写名称和带括号的“下标”来引用该参数：

avail[i]  
demand[j,p]  
revenue[p,a,t]

如果索引是针对第 5 章中描述的简单对象集合，则有一个下标。如果索引是针对第 6 章中描述的配对、三元组或更长的元组集合，则必须有相应数量的下标，以逗号分隔。下标可以是任意表达式，只要它们的计算结果属于基础索引集合的成员即可。

一个未索引的参数是一个标量值，而一个在简单集合上索引的参数则具有向量或数组的特征；当索引是在整数序列上时，例如 `param avail {1..T};`，各个带下标的参数是 `avail[1]`、`avail[2]`、...、`avail[T]`，这与线性代数中的向量或 Fortran、C 等编程语言中的数组有明显的相似性。然而，AMPL 的向量概念更为通用，因为参数也可以在字符串集合上进行索引，而这些字符串甚至不需要有序。在字符串集合上的索引最适合用于那些对应于地点、产品和其他实体的参数，这些实体没有特别自然的编号方式。而在数字序列上的索引则更适合用于那些对应于周 (week)、阶段 (stage) 等本质上有序且有编号的参数；即使对于这些情况，你可能更倾向于使用第 5.6 节中所述的有序字符串集合。

一个在成对集合上索引的参数类似于一个二维数组或矩阵。如果索引是来自两个集合的所有成对组合，例如

```
set ORIG; set DEST; param cost {ORIG,DEST};
```

那么对于每个来自 ORIG 的 i 和来自 DEST 的 j 的组合，都有一个参数 `cost[i,j]`，此时与矩阵的类比最为贴切——尽管再次强调，下标更可能是字符串而不是数字。然而，如果索引是在成对集合的一个子集上：

```
set ORIG; set DEST; set LINKS within {ORIG,DEST}; param cost {LINKS};
```

那么 `cost[i,j]` 只对那些来自 ORIG 的 i 和来自 DEST 的 j 且 `(i,j)` 属于 LINKS 的组合存在。在这种情况下，你可以将 `cost` 视为一个“稀疏”矩阵。

类似的评论也适用于在三元组和更长元组上索引的参数，它们类似于编程语言中的高维数组。

# 7.2 算术表达式

AMPL 中的算术表达式与其他计算机语言中的表达式大致相同。字面量数字由可选的符号后跟一串数字组成，这些数字可以包含也可以不包含小数点（例如，-17 或 2.71828 或 +.3）。在字面量末尾还可以有一个指数，由字母 d、D、e 或 E 和可选的符号后跟数字组成（如 1e30 或 7.66439D-07）。

字面量、参数和变量通过标准的加法 `(+)`、减法 `(-)`、乘法 `(*)`、除法 `(/)` 和幂运算 `(^)` 运算符合并为表达式。熟悉的算术约定适用。幂运算具有比乘法和除法更高的优先级，乘法和除法又比加法和减法具有更高的优先级；相同优先级的连续运算从左到右结合，但幂运算从右到左结合。可以使用括号来改变求值顺序。

算术表达式还可以使用 `div` 运算符，该运算符在左操作数除以右操作数时返回截断后的商；`mod` 运算符，用于计算余数；以及 `less` 运算符，如果结果为正，则返回左操作数减去右操作数的值，否则返回零。在优先级和分组方面，AMPL 将 `div` 和 `mod` 视为与除法相同，而将 `less` 视为与减法相同。

表 7-1 中列出了算术运算符（以及稍后将介绍的逻辑运算符）。在选择运算符符号时，AMPL 尽可能遵循常见的编程语言，例如用 `*` 表示乘法，用 `/` 表示除法。然而，有时存在不止一种标准，比如指数运算，有些语言使用 `^`，而另一些语言则使用 `**`。在这种及其他情况下，AMPL 提供了替代形式。表 7-1 中将更常见的形式列在左侧，替代形式（如果有）列在右侧。

<table><tr><td>常用形式</td><td>替代形式</td><td>操作数类型</td><td>结果类型</td></tr><tr><td>if-then-else</td><td></td><td>逻辑型, 算术型</td><td>算术型</td></tr><tr><td>or</td><td>||</td><td>逻辑型</td><td>逻辑型</td></tr><tr><td>exists forall</td><td></td><td>逻辑型</td><td>逻辑型</td></tr><tr><td>and</td><td>&amp;&amp;</td><td>逻辑型</td><td>逻辑型</td></tr><tr><td>not (一元)</td><td>!</td><td>逻辑型</td><td>逻辑型</td></tr><tr><td>&lt; &lt;= = &lt;&gt; &gt; &gt;=</td><td>&lt; &lt;= = = != &gt; &gt;=</td><td>算术型</td><td>逻辑型</td></tr><tr><td>in not in</td><td></td><td>对象, 集合</td><td>逻辑型</td></tr><tr><td>+ - less</td><td></td><td>算术型</td><td>算术型</td></tr><tr><td>sum prod min max</td><td></td><td>算术型</td><td>算术型</td></tr><tr><td>* / div mod</td><td></td><td>算术型</td><td>算术型</td></tr><tr><td>+ - (一元)</td><td></td><td>算术型</td><td>算术型</td></tr><tr><td>^</td><td>**</td><td>算术型</td><td>算术型</td></tr></table>

指数运算和 `if-then-else` 是右结合的；其他运算符是左结合的。`if-then-else` 的逻辑操作数出现在 `if` 后面，而算术操作数出现在 `then` 和（可选的）`else` 后面。

表 7-1：按优先级递增顺序排列的算术和逻辑运算符。

你可以根据喜好混合使用这些运算符，但如果你在选择上保持一致，模型将更容易阅读和理解。

构建算术表达式的另一种方法是对其他表达式应用函数。函数引用由一个名称后跟括号中的参数或逗号分隔的参数列表组成；算术参数可以是任何算术表达式。以下是一些示例，分别计算其参数的最小值、绝对值和平方根：

```ampl
min(T,20)
abs( sum {i in ORIG} supply[i] - sum {j in DEST} demand[j])
sqrt((tan[j]- tan[k])^2)
```

表 7-2 列出了模型中常用的内置算术函数。除了 `min` 和 `max` 之外，这些函数的名称可以被重新定义，但其原始含义将变得不可访问。例如，一个模型可以像上面最后一个例子中那样声明一个名为 `tan` 的参数，但之后就不能再引用函数 `tan`。

第 5 章中描述的集合函数 `card` 和 `ord` 同样会产生算术结果。此外，AMPL 还提供了几种“舍入”函数（第 11.3 节）和各种随机数函数（见下文第 7.6 节）。附录 A.22 中描述了一种“导入”由你自己程序定义的函数的机制。

<table><tr><td>abs(x)</td><td>绝对值，|x|</td></tr><tr><td>acos(x)</td><td>反余弦，cos⁻¹(x)</td></tr><tr><td>acosh(x)</td><td>反双曲余弦，cosh⁻¹(x)</td></tr><tr><td>asin(x)</td><td>反正弦，sin⁻¹(x)</td></tr><tr><td>asinh(x)</td><td>反双曲正弦，sinh⁻¹(x)</td></tr><tr><td>atan(x)</td><td>反正切，tan⁻¹(x)</td></tr><tr><td>atan2(y,x)</td><td>反正切，tan⁻¹(y/x)</td></tr><tr><td>atanh(x)</td><td>反双曲正切，tanh⁻¹(x)</td></tr><tr><td>cos(x)</td><td>余弦</td></tr><tr><td>cosh(x)</td><td>双曲余弦</td></tr><tr><td>exp(x)</td><td>指数，eˣ</td></tr><tr><td>log(x)</td><td>自然对数，logₑ(x)</td></tr><tr><td>log10(x)</td><td>常用对数，log₁₀(x)</td></tr><tr><td>max(x,y,...)</td><td>最大值（2 个或多个参数）</td></tr><tr><td>min(x,y,...)</td><td>最小值（2 个或多个参数）</td></tr><tr><td>sin(x)</td><td>正弦</td></tr><tr><td>sinh(x)</td><td>双曲正弦</td></tr><tr><td>sqrt(x)</td><td>平方根</td></tr><tr><td>tan(x)</td><td>正切</td></tr><tr><td>tanh(x)</td><td>双曲正切</td></tr></table>

表 7-2：模型中使用的内置算术函数。

最后，代数符号中的索引运算符如 $\Sigma$ 和 $\Pi$ 在 AMPL 中通过在集合上迭代运算的表达式进行推广。特别是，大多数大规模线性规划模型都包含迭代求和：

```ampl
sum {i in ORIG} supply[i]
```

关键字 `sum` 后面可以跟任何索引表达式。后续的算术表达式对索引集合的每个成员计算一次，然后将所有结果值相加。因此，上述求和（来自图 3-1a 的运输模型）表示所有起始地的总供应量。`sum` 运算符的优先级低于 `*`，因此同一模型的目标函数可以写成表示所有起始地和目的地组合的成本 $cost[i,j] \times Trans[i,j]$ 的总和。然而，`sum` 的优先级高于 `+` 或 `-`，因此对于图 6-3 中多周期生产模型的目标函数，我们必须写成：

```ampl
sum {p in PROD, t in 1..T} 
    (revenue[p,t] - cost[p,t]) * Make[p,t] - 
    sum {p in PROD, t in 1..T-1} 
        (holdcost[p,t] * Inv[p,t])
```

$$\sum_{p \in PROD, t \in 1..T} \left( \sum_{a \in AREA[p]} revenue[p,a,t] \cdot Sell[p,a,t] - producost[p] \cdot Make[p,t] - invcost[p] \cdot Inv[p,t] \right);$$

外层求和作用于其后的整个括号表达式，而内层求和仅作用于项 $revenue[p,a,t] \cdot Sell[p,a,t]$。

其他迭代算术运算符包括用于乘法的 $prod$、用于最小值的 $min$ 和用于最大值的 $max$。例如，我们可以使用 $\max_{i \in ORIG} supply[i]$

来描述在任何起始地可获得的最大供应量。

请记住，尽管 AMPL 算术函数或运算符可以应用于变量以及参数或集合的数值成员，但对变量的大多数运算是非线性的。第 8.2 节描述了 AMPL 在线性规划中对算术表达式的要求。第 18 章讨论了某些求解器可以处理的一些变量的非线性函数。

# 7.3 逻辑和条件表达式

算术表达式的值可以通过比较运算符相互测试：

= 等于 <> 不等于 < 小于 <= 小于或等于 > 大于 >= 大于或等于

比较的结果是 "true" 或 "false"。因此，如果参数 T 的值大于 1，则 T > 1 为真，否则为假；当且仅当总供应量等于总需求量时为真。

比较是 AMPL 逻辑表达式的一个例子，逻辑表达式的结果为真或假。第 5.4 节中描述的使用 $in$ 和 $within$ 的集合成员测试是另一个例子。更复杂的逻辑表达式可以通过逻辑运算符构建。$and$ 运算符当且仅当其两个操作数都为真时返回真，而 $or$ 当且仅当其至少一个操作数为真时返回真；一元运算符 $not$ 对真返回假，对假返回真。因此表达式 $T \geq 0\ and\ T \leq 10$

仅当 T 位于区间 $[0, 10]$ 内时为真，而第 5.5 节中的以下表达式，$i \in MAXREQ\ or\ n\_min[i] > 0$

如果 $i$ 是 $MAXREQ$ 的成员，或者 $n\_min[i]$ 为正数，或者两者都满足时为真。当多个运算符一起使用时，任何比较、成员关系或算术运算符的优先级都高于逻辑运算符；$and$ 的优先级高于 $or$，而 $not$ 的优先级高于两者。因此表达式 $not\ i \in MAXREQ\ or\ n\_min[i] > 0\ and\ n\_min[i] \leq 10$

被解释为

$(not\ (i \in MAXREQ))\ or\ (n\_min[i] > 0)\ and\ (n\_min[i] \leq 10)$

或者，可以使用 $not\ in$ 运算符：

$i\ not\ in\ MAXREQ\ or\ n\_min[i] > 0\ and\ n\_min[i] \leq 10$

优先级总结在表 7-1 中，该表还给出了替代形式。

像 $+$ 和 $*$ 一样，$or$ 和 $and$ 运算符也有迭代版本。迭代 $or$ 用 $exists$ 表示，迭代 $and$ 用 $forall$ 表示。例如，表达式 $exists\ \{i \in ORIG\}\ demand[i] > 10$ 当且仅当至少一个起始地的需求大于 10 时为真，而 $forall\ \{i \in ORIG\}\ demand[i] > 10$ 当且仅当每个起始地的需求都大于 10 时为真。

逻辑表达式的另一个用途是作为条件运算符或 $if-then-else$ 运算符的操作数，该运算符根据逻辑表达式为真或假返回两个不同的算术值之一。考虑图 5-3 多周期生产模型中的两组库存平衡 (Inv) 约束：

subject to Balance0 {p in PROD}: Make[p,first(WEEKS)] $+$ inv0[p] $=$ Sell[p,first(WEEKS)] $+$ Inv[p,first(WEEKS)];

受限于平衡 {p in PROD, t in WEEKS: ord(t) > 1}: Make[p,t] $^+$ Inv[p,prev(t)] $=$ Sell[p,t] $^+$ Inv[p,t];

Balance0 约束本质上是将 t 设为 first(WEEKS) 的 Balance 约束。唯一的区别在于第二项，该项表示前一周的库存；在第一周时它被指定为 inv0[p]（在 Balance0 约束中），而在后续周中则由变量 Inv[p, prev(t)] 表示（在 Balance 约束中）。我们希望将这些约束合并为一个声明，通过引入一个在 t 为第一周时取值为 inv0[p]，否则取值为 Inv[p, prev(t)] 的项。这样的项在 AMPL 中写作：

if t $=$ first(WEEKS) then inv0[p] else Inv[p,prev(t)]

将此表达式放入约束声明中，我们可以写成 subject to Balance {p in PROD, t in WEEKS}: Make[p,t] $^+$ (if t $=$ first(WEEKS) then inv0[p] else Inv[p,prev(t)]) $=$ Sell[p,t] $^+$ Inv[p,t];

这种形式比两个独立的声明更简洁、更直接地表达了库存平衡约束。

条件表达式的一般形式为 if a then $b$ else $c$

其中 $a$ 是一个逻辑表达式。若 $a$ 的值为真，则条件表达式取 $b$ 的值；若 $a$ 为假，则表达式取 $c$ 的值。若 $c$ 为零，则 else $c$ 部分可以省略。大多数情况下 $b$ 和 $c$ 是算术表达式，但它们也可以是字符串或集合表达式，只要二者属于同一类型即可。由于 then 和 else 的优先级低于其他所有运算符，除非条件表达式出现在语句末尾，否则需要加上括号（如上例所示）。

AMPL 还提供了一个用于编程的 if-then-else 结构；与许多编程语言中的条件语句类似，它根据某个逻辑表达式的真假来执行不同的语句块。我们将在第 13 章中与其他 AMPL 编程特性一起介绍它。这里描述的 if-then-else 不是一个语句，而是一个值有条件确定的表达式。因此它应出现在声明内部，即通常对表达式求值的位置。

# 7.4 参数的限制

如果 $\mathbb{T}$ 用于表示多周期模型中的周数，则它应为一个大于 1 的整数。通过在 T 的声明中包含这些条件，

param $\mathrm{T} > 1$ integer;

如果你不小心将 $\mathbb{T}$ 设置为 1，你会指示 AMPL 拒绝你的数据：

error processing param T: failed check: param $\mathrm{T} = 1$ is not > 1;

或者设置为 2.5：

error processing param T: failed check: param $\mathrm{T} = 2.5$ is not an integer;

只要存在此类错误，AMPL 就不会将你的问题实例发送给求解器。

在参数的索引集合声明中，诸如 integer 或 $>= 0$ 的简单限制适用于每一个被定义的参数。我们的示例经常使用此选项来指定向量和数组是非负的：

param demand $\{\mathrm{DEST},\mathrm{PROD}\} >= 0;$

但如果你在索引表达式中包含虚拟索引，则可以使用它们为每个参数指定不同的限制：

param f_min $\{\mathrm{FOOD}\} >= 0;$ param f_max {j in FOOD} >= f_min[j];

这些声明的效果是为集合 `FOOD` 中的每一个 `j` 定义一对参数 `f_max[j] >= f_min[j]`。

参数声明的限制短语可以是单词 `integer` 或 `binary`，或比较运算符后跟一个算术表达式。其中 `integer` 将参数限制为整数值（整数），而 `binary` 将其限制为 `0` 或 `1`。算术表达式可以引用模型中先前定义的集合和参数，以及当前声明中定义的虚拟索引。同一声明中可以有多个限制短语，在这种情况下，它们之间可以用逗号可选地分隔。

在特殊情况下，限制短语甚至可以引用其所在声明中的参数。例如，某些多周期生产模型是根据参数 `cumulative_market[p,t]` 定义的，该参数表示产品 $\mathbb{P}$ 在第 1 周到第 `t` 周的累计需求量。由于累计需求不会减少，你可能会尝试编写如下限制短语：

```ampl
param cumulative_market {p in PROD, t in 1..T} >= cumulative_market[p,t-1]; # ERROR
```

然而，对于参数 `cumulative_market[p,1]`，该限制短语将引用未定义的 `cumulative_market[p,0]`；`AMPL` 会以错误消息拒绝该声明。在这种情况下，你再次需要一个条件表达式来特别处理第一个周期：

```ampl
param cumulative_market {p in PROD, t in 1..T} >= if t = 1 then 0 else cumulative_market[p,t-1];
```

同样的内容可以写得更简洁一些，因为默认“`else 0`”。几乎总是需要某种形式的 `if-then-else` 表达式才能实现这种自引用。

正如你从最后一个示例中可能猜到的那样，有时希望对模型数据施加比在声明中通过限制短语所能表达的更复杂的限制。这就是 `check` 语句的目的。例如，在图 3-1a 的运输模型中，总供应量必须等于总需求量：

多商品版本，如图 4-1 所示，使用带索引的 `check` 语句来表示每种产品 (`product`) 的总供应量 (`supply`) 必须等于总需求量 (`demand`)：

此处的限制条件针对 `PROD` 的每个成员 $\mathbb{P}$ 测试一次。如果对任何成员测试失败，`AMPL` 将打印错误消息并拒绝所有数据。

你可以将 `check` 语句视为一种约束 (`constraint`) 的指定方式，但仅作用于数据。`restriction` 子句是一个逻辑表达式，可以使用任何先前定义的集合 (`set`) 和参数 (`parameter`)，以及在该语句的索引表达式中定义的虚拟索引 (`dummy index`)。在读取数据值之后，该逻辑表达式必须计算结果为真；如果指定了索引表达式，则逻辑表达式将针对虚拟索引的每个集合成员赋值分别计算，并且每个结果都必须为真。

我们强烈建议使用 restriction 短语和 check 语句来验证模型的数据。这些功能将帮助你在早期阶段发现数据错误，此时错误易于修复。未被发现的数据错误最多会导致变量 (variable) 和约束生成错误，从而让你从 AMPL 得到某种错误消息。在其他情况下，数据错误会导致生成不正确的线性规划 (linear program, LP)。如果你幸运的话，不正确的 LP 将产生一个无意义的最优解 (optimal solution)，因此——可能在付出相当大的努力之后——你能够反向找出数据中的错误。最坏的情况是，不正确的 LP 将产生一个看似合理的解，而错误未被发现。

# 7.5 计算参数

很少能安排模型可用的数据值恰好是目标函数和约束所需的系数值。甚至在图 1-4 的简单生产模型中，我们将约束写为

``` ampl
sum {p in PROD} (1/rate[p]) * Make[p] <= avail;
```

因为生产速率是以吨/小时给出的，而 Make[p] 的系数必须以小时/吨表示。在约束和目标函数中可以使用任何参数表达式，但最好保持表达式简单。当需要更复杂的表达式时，通常通过定义新的计算参数（computed parameter）来表示数据参数会使模型更易于理解。

计算参数 (computed parameter) 的声明包含一个赋值短语 (assignment phrase)，它与上一节中描述的约束短语 (restriction phrase) 类似，但使用 = 运算符表示该参数被设置为等于某个表达式，而不是仅仅受到不等式的约束。作为第一个例子，假设提供给图 4-1 多商品运输模型 (multicommodity transportation model) 的数据值包括每种产品的总需求量 (total demand)，以及每个目的地 (destination) 的需求份额 (share of demand)。目的地的份额是介于零和 100 之间的百分比，但由于四舍五入和近似，它们在所有目的地上的总和可能不完全等于 100%。因此，我们声明数据参数来表示这些份额，并声明一个计算参数等于它们的总和：

``` ampl
param share {DEST} >= 0, <= 100;
param tot_sh = sum {j in DEST} share[j];
```

然后，我们可以声明一个数据参数来表示总需求量，并声明一个计算参数等于每个目的地的需求量：

``` ampl
param tot_dem {PROD} >= 0;
param demand {j in DEST, p in PROD}
   = share[j] * tot_dem[p] / tot_sh;
```

除以 tot_sh 的作用是作为一个校正因子，用于总和不等于 $100\%$ 的情况。一旦以这种方式定义了需求量，模型就可以像图 4-1 中那样使用它：

``` ampl
subject to Demand {j in DEST, p in PROD}:
   sum {i in ORIG} Trans[i,j,p] = demand[j,p];
```

我们可以通过将 tot_sh 和 demand[j,p] 的公式直接代入此约束来避免使用计算参数：

``` ampl
subject to Demand {j in DEST, p in PROD}:
   sum {i in ORIG} Trans[i,j,p] =
      share[j] * tot_dem[p] / sum {k in DEST} share[k];
```

这种替代方法使模型稍微短一些，但需求量的计算和约束的结构都更难理解。

作为另一个例子，考虑多周期生产模型 (multiperiod production model)（图 4-4）的一种情景，其中需要计算最低库存量 (minimum inventories)。具体来说，假设产品 p 在第 t 周的库存量至少必须是接下来一周可销售的最大数量 market[p,t+1] 的某个分数 (fraction)。因此，我们使用以下声明来提供数据：

``` ampl
param frac > 0;  param market {PROD,1..T+1} >= 0;  and then declare
```

然后声明

``` ampl
param mininv {p in PROD, t in 0..T} = frac * market[p,t+1];  var Inv {p in PROD, t in 0..T} >= mininv[p,t];
```

以定义和使用表示产品 p 在第 t 周最低库存量的参数 mininv[p,t]。在整个会话过程中，AMPL 会保持所有参数的 = 定义是最新的。因此，例如，如果您更改 frac 的值，所有 mininv 参数的值会自动相应地改变。

如果您像上面的例子中那样定义了一个计算参数，那么您就不能再为它指定一个数据值。尝试这样做将导致错误消息：

```
mininv was defined in the model  context: param >>> mininv << := bands 2 3000
```

但是，有一种替代方法可以为参数定义初始值，但允许稍后更改它。

如果你使用 default 运算符代替 = 来定义一个参数，那么该参数会被初始化而不是被定义。其值取自 default 运算符右侧表达式的值，但如果该表达式的值随后发生变化，参数值不会随之改变。初始值可以通过 data 语句覆盖，也可以通过后续的赋值语句更改。这一特性在编写重复更新某些值的 AMPL 脚本时非常有用，如 13.2 节所示。

如果你使用 default 运算符代替 $=$ 来定义一个参数，那么你可以在 data 语句中指定值，以覆盖原本会被计算出的值。例如，通过声明

```cpp
param mininv {p in PROD, t in 0..T} default frac * market[p,t+1];
```

你可以允许在模型数据中指定一些特殊的最小库存值，可以是列表形式：

```cpp
param mininv :=
    bands 2 3000
    coils 2 2000
    coils 3 2000;
```

或表格形式：

```cpp
param market: 1 2 3 4 :=
    bands . 3000 .
    coils . 2000 2000 . ;
```

（AMPL 在 data 语句中使用 "." 表示省略的条目，如第 9 章和 A.12.2 节所述。）

参数默认值的表达式仅在首次需要该参数值时才会被求值，例如当 solve 命令处理使用该参数的目标函数或约束时。

在大多数 $=$ 和 default 语句中，运算符后面跟着一个由先前定义的集合和参数（但不包括变量）以及当前定义的虚拟索引组成的算术表达式。然而，索引集合中的一些参数可以基于同一集合中的其他参数来赋予计算值或默认值。例如，你可以通过如下方式定义 mininv 参数为一个移动平均值，以平滑最小库存中的一些波动：

```cpp
param mininv {p in PROD, t in 0..T} =
    if t = 0 then inv0[p]
    else 0.5 * (mininv[p,t-1] + frac * market[p,t+1]);
```

第 0 周的 `mininv` 值被显式设置为初始库存，而后续每周 $t$ 的值则根据前一周的值来定义。AMPL 允许这种“递归”定义，但如果检测到导致参数值直接或间接依赖于自身的循环引用，它将报错。

你可以将本节中介绍的语句与上一节中的限制语句结合使用，以进一步检查计算出的值。例如，声明

```cpp
param mininv {p in PROD, t in 0..T} = frac * market[p,t+1], >= 0;
```

将在任何 `mininv` 参数的计算值为负时触发错误。每当 AMPL 会话出于任何目的使用 `mininv` 时，都会触发此检查。

# 7.6 随机生成的参数

在测试模型时，尤其是在开发的早期阶段，你可能会发现使用随机生成的数据来代替后续才能获得的实际数据会比较方便。随机生成的参数在试验替代模型公式或求解器 (solver) 时也很有用。

随机生成的参数类似于前一节中介绍的计算参数，只不过它们的定义表达式通过使用 AMPL 内置的随机数生成函数（见表 A-3）而变得具有随机性。以最简单的情况为例，表示可用小时数的单个参数 `avail`（在 steel.mod 中）可以定义为等于一个随机函数：

```python
param avail_mean > 0;
param avail_variance > 0, < avail_mean / 2;
param avail = max(Normal(avail_mean, avail_variance), 0);
```

添加一些索引即可得到该模型的多阶段版本：

```python
param avail {STAGE} = max(Normal(avail_mean, avail_variance), 0);
```

对于每个阶段 `s`，这会从相同的随机分布中为 `avail[s]` 赋予一个不同的随机值。若要指定依赖于阶段的随机分布，你也需要为均值和方差参数添加索引：

```python
param avail_mean {STAGE} > 0;
param avail_variance {s in STAGE} > 0, < avail_mean[s] / 2;
param avail {s in STAGE} = max(Normal(avail_mean[s], avail_variance[s]), 0);
```

表达式 `max(..., 0)` 的作用是处理正态分布中罕见的返回负值的情况（即使均值为正）。

从前一节的示例中自然可以引出更一般的随机计算参数的方法。在多商品运输问题 (multicommodity transportation problem) 中，你可以定义随机的需求份额：

```python
param share {DEST} = Uniform(0, 100);
param tot_sh = sum {j in DEST} share[j];
param tot_dem {PROD} >= 0;
param demand {j in DEST, p in PROD} = share[j] * tot_dem[p] / tot_sh;
```

由于参数 `tot_sh` 和 `demand` 是根据随机参数定义的，因此它们也成为随机的。在多时期生产模型 (multiperiod production model) 中，你可以通过初始值和每期随机增加量来定义需求量 `market[p,t]`：

```python
param market1 {PROD} >= 0;
param max_incr {PROD} >= 0;
param market {p in PROD, t in 1..T+1} =
    if t = 1 then market1[p]
    else Uniform(0, max_incr[p]) * market[p, t-1];
```

这种递归定义提供了一种随时间生成简单随机过程的方法。

所有 AMPL 随机函数都基于一个周期非常长的均匀随机数生成器。然而，当你启动 AMPL 或执行 reset 命令时，生成器会被重置，"随机"值也会与之前相同。你可以通过将 AMPL 选项 randseed 更改为除默认值 1 以外的其他整数来请求不同的值；为此，命令是：

```python
option randseed n;
```

其中 $n$ 是某个整数值。非零值会给出在每次 AMPL 重置时重复的序列。值为 0 时请求 AMPL 根据系统时钟的当前值选择种子，从而（在实际应用中）在每次重置时得到不同的种子。

当 AMPL 的 reset data 命令应用于随机计算的参数时，也会导致确定新的随机值样本。该命令的使用在第 11.3 节中讨论。

# 7.7 逻辑参数

尽管参数通常表示数值，但也可以选择用于表示真/假值或字符字符串。

当前版本的 AMPL 不支持完全意义上的 "逻辑" 类型参数（即只表示真和假值），但可以使用二进制类型的参数达到相同效果。作为示例，我们描述前述库存示例在消费品中的应用。每周某些产品可能会进行特别促销，在这种情况下它们需要更高的库存比例。使用二进制类型的参数，我们可以通过以下声明表示这种情况：

param fr_reg  $>0$  # 常规库存比例
param fr_pro  $\gimel$  fr_reg; # 促销商品的比例
param promote {PROD,1. .T+1} binary;
param market {PROD,1. .T+1} >= 0;

当没有促销时，二进制参数 promote[p,t] 为 0，有促销时为 1。因此我们可以通过 if-then-else 表达式定义最小库存参数如下：

```cpp
param mininv {p in PROD, t in 0. .T} = (if promote[p,t] = 1 then fr_pro else fr_reg) * market[p,t+1];
```

我们也可以更简洁地说：

```cpp
param mininv {p in PROD, t in 0. .T} = (if promote[p,t] then fr_pro else fr_reg) * market[p,t+1];
```

当像 promote[p,t] 这样的算术表达式出现在需要逻辑表达式的地方时，AMPL 将任何非零值解释为真，零解释为假。您需要稍微小心以避免被这种隐式转换绊倒。例如，在第 7.4 节中我们使用了表达式，如果您不小心写成，这完全合法，但并不表示您想要的意思。

# 7.8 符号参数

您可以通过在声明中包含关键字 symbolic 来允许参数表示字符字符串值。符号参数的值可以是字符串或数字，就像集合的成员一样，但字符串值不能参与算术运算。

符号参数的主要用途是指定要特殊处理的单个集合成员。例如，在交通流模型中，有一个交叉口集合，其中两个成员被指定为入口和出口。符号参数可用于表示这两个成员：

```cpp
set INTER;
param entr symbolic in INTER;
param exit symbolic in INTER, <> entr;
```

在数据语句中，每个符号参数都被赋予一个适当的字符串：

集合 INTER : $=$ a b c d e f g ; 参数 entr : $=$ a ; 参数 exit : $=$ g ;

这些参数随后用于定义目标函数和约束；完整模型在第 15.2 节中开发。

符号参数的另一种用途是将描述性字符串与集合成员关联起来。例如，考虑图 3-1a 中运输模型的 “起始地” 集合。当我们在第 3 章开头引入这个集合时，我们通过一个 4 字符字符串和一个更长的描述性字符串来描述每个起始城市。短字符串成为 AMPL 集合 ORIG 的成员，而长字符串不再起作用。为了使两者都可用，我们可以声明 set ORIG; param orig_name {ORIG} symbolic; param supply {ORIG} >= 0;

然后在数据中我们可以指定 param: ORIG: orig_name supply := GARY "Gary, Indiana" 1400 CLEV "Cleveland, Ohio" 2600 PITT "Pittsburgh, Pennsylvania" 2900;

由于长字符串不具备 AMPL 名称的形式，因此需要加引号。它们在模型或生成的线性规划中仍然不起作用，但可以通过第 12 章中描述的 display 和 printf 命令为文档目的检索它们。

正如存在算术和逻辑运算符及函数一样，也有 AMPL 字符串运算符和函数用于处理字符串值。这些功能主要用于 AMPL 命令脚本而不是模型中，因此我们将它们的描述推迟到第 13.7 节。

# 练习

7-1. 展示如何修改图 4-1 中的多商品运输模型，使其应用以下数据限制。使用集合或参数声明中的限制短语，或 check 语句，视情况而定。

- 没有城市同时属于 ORIG 和 DEST。  
- DEST 中的城市数量必须大于 ORIG 中的数量。  
- DEST 中任何城市的需求数量不得超过 1000。  
- 每种产品在所有起始地的总供应量必须等于该产品在所有目的地的总需求量。  
- 所有产品在所有起始地的总供应量必须等于所有产品在所有目的地的总需求量。  
- 某一起始地所有产品的总供应量不得超过从该起始地出发的所有运输的总容量。  
- 某一目的地所有产品的总需求量不得超过运送到该目的地的所有运输的总容量。

7-2. 展示如何修改图 4-4 中的多期生产模型，使其应用以下数据限制。

- 周数是一个大于 1 的正整数。  
- 产品的初始库存不超过该产品在所有周期内的市场需求总量。  
- 任何一周内，产品的库存成本都不超过该产品当周预期收入的 $10\%$。  
- 每周的工作小时数介于 24 至 40 小时之间，且相邻周之间的变化不超过 8 小时。  
- 对于每种产品，其预期收入从一周到下一周不会减少。

7-3. 下列练习的解涉及使用 if-then-else 运算符来构建约束。

(a) 在 7.3 节中关于 Balance 约束的例子中，我们使用了一个表达式开头如下：

if t $=$ first(weeks) then

请找出一个使用函数 ord(t) 的等价表达式。

(b) 将图 5-1 饮食模型中的 Diet_Min 和 Diet_Max 约束合并为一个约束声明。

(c) 在图 4-1 的多商品运输模型中，假设目的地的需求超过了起始地所能提供的供应量。为了弥补这一差距，可以在某些起始地购买（而非生产）有限数量的额外吨位用于运输。

为建模这种情况，假设我们声明了起始地的一个子集，

```AMPL
set BUY_ORIG within ORIG;
```

表示可以在这些起始地购买额外吨位。相关的数据值和决策变量可以基于该子集进行索引：

```AMPL
param buy_supply {BUY_ORIG,PROD} >= 0; # 可购买的数量  
param buy_cost {BUY_ORIG,PROD} > 0;    # 每吨购买成本  
var Buy {i in BUY_ORIG, p in PROD} >= 0, <= buy_supply[i,p]; # 购买量
```

修改目标函数以包含购买成本。修改供应约束（Supply constraints），使其表示对于每个起始地和每种产品，运出的总吨位必须等于生产供应量加上（如果适用）购买的吨位。

(d) 构造与 (c) 中相同的模型，但 `BUY_ORIG` 是由产品 $\mathbb{P}$ 可在起始地 i 购买的序对 (i,p) 组成的集合。

7-4. 本练习关注图 4-1 中的以下集合和参数：

```AMPL
set ORIG;   # 起始地  
set DEST;   # 目的地  
set PROD;   # 产品  
param supply {ORIG,PROD} >= 0;  
param demand {DEST,PROD} >= 0;
```

(a) 使用 `=` 运算符编写参数声明，以计算具有以下定义的参数：

- `prod_supply[p]` 是产品 $\mathbb{P}$ 在所有起始地的总供应量。
- `dest_demand[j]` 是目的地 j 所有产品的总需求量 (demand)。
- `true_limit[i,j,p]` 是可以从 i 运送到 j 的产品 $\mathbb{P}$ 的最大数量，也就是说，不超过 `limit[i,j]`、i 处 $\mathbb{P}$ 的供应量 (supply) 或 j 处 $\mathbb{P}$ 的需求量 (demand) 的最大值。
- `max_supply[p]` 是任何起始地 (origins) 可提供的产品 $\mathbb{P}$ 的最大供应量 (supply)。
- `max_dif[p]` 是所有起始地和目的地组合中，产品 $\mathbb{P}$ 的供应量 (supply) 与需求量 (demand) 之间的最大差值。

(b) 使用 `=` 运算符编写集合声明，以计算以下集合：

- 在某些目的地 j 处需求量 (demand) 至少为 500 的产品 $\mathbb{P}$。
- 在所有目的地 j 处需求量 (demand) 至少为 250 的产品 $\mathbb{P}$。
- 在某些目的地 j 处需求量 (demand) 等于 500 的产品 $\mathbb{P}$。

7-5. AMPL 参数可以被定义为包含多种序列，特别是通过使用递归定义。例如，我们可以通过以下方式使 $\mathbb{S}[\mathbf{j}]$ 等于从 1 到某个给定限制 N 的前 j 个整数之和：

$$
\begin{array}{rl}
& \mathtt{param}\ \mathtt{N}; \\
& \mathtt{param}\ \mathtt{s}\{\mathtt{j} \in 1..N\} = \mathtt{sum}\{\mathtt{jj} \in 1..j\} \mathtt{jj};
\end{array}
$$

或者，使用求和公式，

$$
\mathtt{param}\ \mathtt{s}\{\mathtt{j} \in 1..N\} = \mathtt{j} \times (\mathtt{j} + 1) / 2;
$$

或者，使用递归定义，

$$
\mathtt{param}\ \mathtt{s}\{\mathtt{j}\ \mathtt{in}\ \mathtt{L}.\mathtt{N}\} = \mathtt{if}\ \mathtt{j} = 1\ \mathtt{then}\ 1\ \mathtt{else}\ \mathtt{s}[\mathtt{j} - 1] + \mathtt{j};
$$

本练习要求你尝试一些其他的可能性。

(a) 定义 $\mathtt{fact}[\mathtt{n}]$ 为 n 的阶乘，即前 n 个正整数的乘积。给出如上所示的递归和非递归定义。

(b) 斐波那契数列 (Fibonacci number) 在数学上定义为 $f_{0} = f_{1} = 1$ 和 $f_{n} = f_{n - 1} + f_{n - 2}$。使用递归声明，在 AMPL 中定义 $\mathtt{fib}[\mathtt{n}]$ 以等于第 n 个斐波那契数。

使用另一个 AMPL 声明验证第 $n$ 个斐波那契数等于最接近 $(\frac{1}{2} + \frac{1}{2}\sqrt{5})^{n} / \sqrt{5}$ 的整数。

(c) 这是另一个递归定义，称为阿克曼函数 (Ackermann function)，用于正整数 $i$ 和 $j$：

$$
\begin{array}{c}
A(i,0)=i+1 \\
A(0,j+1)=A(1,j) \\
A(i+1,j+1)=A(A(i,j+1),j)
\end{array}
$$

使用递归声明，在 AMPL 中定义 $\mathtt{ack}[\mathtt{i},\mathtt{j}]$，使其等于 $A(i,j)$。使用 display 打印 $\mathtt{ack}[0,0]$、$\mathtt{ack}[1,1]$、$\mathtt{ack}[2,2]$ 等。你遇到了什么困难？

(d) 由以下 odd 声明定义的值 $\mathtt{odd}[\mathtt{i}]$ 是什么？

param odd {i in 1..N} = if i = 1 then 3 else min {j in odd[i-1]+2..odd[i-1]*2 by 2: not exists {k in 1..i-1} j mod odd[k] = 0} j;

一旦你弄明白了，请创建一个更简单且高效的声明，以给出这些数字的集合而不是数组。

(e) 一棵“树”由一组节点组成，其中一个我们指定为“根”。除根节点外，每个节点在树中都有一个唯一的前驱节点，这样如果你从一个节点向后追溯到它的前驱，再追溯到前驱的前驱，依此类推，你最终总会到达根节点。树可以像这样绘制，根节点在左侧，每个节点向其后继节点画箭头：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/896f45db0e79fbe823eda56455abd8a799169b9b23771991bbb5c4866616b968.jpg)

我们可以用 AMPL 的集合和参数来存储树的结构，如下所示：

set NODES; param Root symbolic in NODES; param pred {i in NODES diff {Root}} symbolic in NODES diff {i};

每个节点 i（除 Root 外）都有一个前驱 pred[i]。

节点的深度是从该节点回溯到根节点所遇到的前驱节点数量；根节点的深度为 0。给出一个 AMPL 定义 depth[i]，正确计算每个节点 i 的深度。要检查你的答案，请将你的定义应用于上述树的 AMPL 数据；读入数据后，使用 display 查看参数 depth。数据中的错误可能导致树加上一个不连通的环，如下所示：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a9d1d62181e35e462467fadd5267135b4eb81ad4e8c6bc53541e348e24f49484.jpg)

如果你输入了这样的数据，当你尝试显示 depth 时会发生什么？