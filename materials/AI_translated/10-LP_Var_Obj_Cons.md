# 线性规划：变量、目标和约束

最著名的优化模型（也是我们迄今为止所有示例的基础）是线性规划 (linear program)。线性规划的变量取值来自某个连续范围；目标函数和约束必须仅使用变量的线性函数。前几章已经非正式或隐含地描述了这些要求；在这里我们将更加具体。

线性规划 (linear program) 特别重要，因为它准确地表示了许多优化的实际应用。线性函数的简单性使得线性模型易于构建、解释和分析。它们也易于求解；如果你能将问题表示为线性规划，即使有数千个约束 (constraint) 和变量 (variable)，你也可以确信能够准确快速地找到最优解。

本章描述了如何声明变量，定义 AMPL 识别为变量线性表达式的表达式，并给出声明线性目标 (objective) 和约束 (constraint) 的规则。关于变量、目标和约束的大部分材料也适用于其他 AMPL 模型，并将在后续章节中使用。

由于 AMPL 本质上是一种代数建模语言 (algebraic model)，本章重点介绍如何使用代数目标函数和约束 (algebraic objectives and constraints) 来表达线性规划 (linear program) 的相关特性。对于具有某些特殊结构（例如网络结构）的线性规划，AMPL 提供了替代的表示方法，这可以使模型更易于编写、阅读和求解。这些特殊结构也是第 15 章至第 17 章的主题之一。

# 8.1 变量 (Variables)

线性规划中的变量 (variable) 与其数值参数 (parameter) 有很多共同之处。二者都是代表数字的符号，并且都可以用于算术表达式中。参数值由建模者提供或通过其他值计算得出，而变量值则由优化算法（由我们称为求解器 (solver) 的软件包实现）确定。

从语法上看，变量声明与第 7 章中定义的参数声明相同，只是以关键字 var 而不是 param 开头。然而，当这些修饰语应用于变量而不是参数时，其含义可能有所不同。

以 $>= $ 或 $<= $ 开头的修饰语在声明线性规划变量时最为常见。在我们所有的例子中都出现了这种用法，最早见于图 1-4 中的生产模型 (production model)：

$$
\mathtt{var~Make~\{p~in~PROD\}~>=~0,~<=~market[p]~;}
$$

此声明为集合 PROD 的每个成员 $\mathbb{P}$ 创建了一个索引变量 Make[p]；在这方面，其规则与参数完全相同。这两个修饰语的作用是对变量的允许取值施加限制或约束 (constraint)。具体来说，$>= 0$ 表示所有变量 Make[p] 必须被优化算法赋予非负值，而 $<= $ market[p] 则表示对于每个产品 $\mathbb{P}$ ，赋予 Make[p] 的值不得超过参数 market[p] 的值。

一般来说，$>= $ 或 $<= $ 后面可以跟任何由先前定义的集合和参数以及当前定义的虚拟索引 (dummy index) 组成的算术表达式。大多数线性规划的建模方式都要求每个变量必须非负；在 AMPL 的变量声明中可以通过 $>= 0$ 直接指定非负性，也可以如图 5-1 中的饮食模型 (diet model) 那样间接指定：

$$
\begin{array}{rl} & {\tt param~f\_min~\{FOOD\}~>=~0;}\\ & {\tt param~f\_max~\{j~in~FOOD\}~>=~f\_min[j];}\\ & {\tt var~Buy~\{j~in~FOOD\}~>=~f\_min[j],~<=~f\_max[j];} \end{array}
$$

跟在 $\geq$ 和 $\leq$ 后面的值是变量的 下界 (lower bound) 和 上界 (upper bound)。由于这些边界表示一种 约束 (constraint)，它们也可以通过本章后面描述的约束声明来施加。然而，将边界放在 var 声明中，可能会使模型更短或更清晰，尽管不会使 最优解 (optimal solution) 有所不同或更容易找到。一些 求解器 (solver) 确实会特别处理边界以加速其算法，但在 AMPL 中，无论边界在模型中如何表达，都会被自动识别。

变量声明中不能使用比较运算符 $<, >$ 或 $<>$ 来限定短语。在线性规划 (linear program) 中，限制变量为（例如）$< 3$ 是没有意义的，因为它总是可以选择为 2.99999……或无限接近 3 的值。

变量声明中的 $=$ 短语会产生一个定义，就像参数声明一样。然而，由于这是在声明一个变量，因此 $=$ 运算符右侧的表达式可以包含先前声明的变量以及集合和参数。例如，你可以不用写出图 6-3（steelT3.mod）中的多周期生产模型的复杂目标函数：

$$\max \text{Total\_Profit: } \sum_{p \in \text{PROD}, t \in 1..T} \left( \sum_{a \in \text{AREA}[p]} \text{revenue}[p,a,t] \cdot \text{Sell}[p,a,t] - \text{producost}[p] \cdot \text{Make}[p,t] - \text{invcost}[p] \cdot \text{Inv}[p,t] \right);$$

而是可以定义变量来表示 总收入 (Total Revenue)、生产成本 (Production Cost) 和 库存成本 (Inventory Cost)：

$$\text{var Total\_Revenue} = \sum_{p \in \text{PROD}, t \in 1..T} \sum_{a \in \text{AREA}[p]} \text{revenue}[p,a,t] \cdot \text{Sell}[p,a,t];$$  
$$\text{var Total\_Prod\_Cost} = \sum_{p \in \text{PROD}, t \in 1..T} \text{producost}[p] \cdot \text{Make}[p,t];$$  
$$\text{var Total\_Inv\_Cost} = \sum_{p \in \text{PROD}, t \in 1..T} \text{invcost}[p] \cdot \text{Inv}[p,t];$$

然后目标函数就是这三个定义变量的总和：

$$\max \text{Total\_Profit: Total\_Revenue} - \text{Total\_Prod\_Cost} - \text{Total\_Inv\_Cost};$$

这样目标函数的结构更加清晰。此外，这些定义的变量可以方便地用于 display 语句，以显示利润的三个主要组成部分的对比情况：

```
ampl: display Total_Revenue, Total_Prod_Cost, Total_Inv_Cost;
Total_Revenue = 801385
Total_Prod_Cost = 285643
Total_Inv_Cost = 1221
```

像这样的定义变量声明不会在生成的问题实例中引入额外的约束。相反，$=$ 右侧的线性表达式会在目标函数和约束中每一次出现该定义变量时被代入。定义变量在线性规划中更为有用，因为代入可能是隐式的，因此我们将在第 18 章再次讨论这个主题。

如果 $=$ 运算符右侧的表达式不包含任何变量，则你只是在定义变量，并将其固定为数据所给定的值。在这种情况下，你应该改用 param 声明。另一方面，如果你只是想在开发或分析模型时临时固定某些变量，则应保持声明不变，而应使用第 11.4 节中描述的 fix 命令来固定它们。

变量声明中的 $\coloneqq$ 或 default 短语为指定的变量提供初始值。未通过 $\coloneqq$ 赋予初始值的变量也可以从数据文件中赋予初始值。当调用求解器时，变量的初始值通常会被更改——理想情况下是更改为最优值。因此，变量初始值的主要目的是为求解器提供一个良好的起始解。然而，线性规划的求解器很少能有效利用起始解，所以我们将在第 18 章关于非线性规划的内容中进一步讨论这一主题。

最后，变量可以声明为整数，以便在任何最优解中必须取整数值，或者声明为二进制，以便只能取 0 和 1。包含此类变量的模型称为整数规划 (integer program)，相关内容将在第 20 章中讨论。

# 8.2 线性表达式

对于给定变量，如果该变量每增加或减少一个单位，表达式的值都相应地增加或减少某个固定量，则称该算术表达式在此变量上是线性的。如果一个表达式在其所有变量上都是线性的，则称其为线性表达式 (linear expression)。（严格来说，这些是仿射表达式 (affine expression)，而线性表达式是指常数项为零的仿射表达式。为简便起见，我们将忽略这一区别。）

AMPL 将以下形式的项的总和识别为线性表达式：

constant-expr variable-ref  
(constant-expr) * variable-ref

其中每个 constant-expr 是一个不包含变量的算术表达式，而 variable-ref 是对变量（可能带有下标）的引用。如果根据运算符优先级规则（见表 A-1），省略 constant-expr 周围的括号不会改变结果，则可以省略括号。以下示例均来自图 6-3 中多周期生产模型的约束条件，根据上述定义，它们都是线性表达式：

```r
avail[t]
Make[p,t] + Inv[p,t-1]
sum {p in PROD} (1/rate[p]) * Make[p,t]
sum {a in AREA[p]} Sell[p,a,t] + Inv[p,t]
```

该模型的目标函数，

$$
\sum_{p \in PROD, t \in 1..T} \left( 
\sum_{a \in AREA[p]} revenue[p,a,t] \cdot Sell[p,a,t] 
- producost[p] \cdot Make[p,t] 
- invcost[p] \cdot Inv[p,t] 
\right)
$$

也是线性的，因为减去一项等于加上该项的负值，而求和的求和本身仍是一个求和。

各种形式的表达式等价于上述形式的项之和，并且也被 AMPL 识别为线性表达式。除以一个算术表达式等价于乘以其倒数，因此在线性规划中可以写作 Make[p,t] / rate[p]。

乘法的顺序无关紧要，因此变量引用 (variable-ref) 不必出现在项的末尾；例如，revenue[p,a,t] * Sell[p,a,t] 等价于 Sell[p,a,t] * revenue[p,a,t]。

结合这些原则的一个例子是：假设 revenue[p,a,t] 的单位是美元每公吨，而 Sell 的单位仍然是吨。如果我们定义转换因子：

param mt_t = 0.90718474; # 公吨每吨  
param t_mt = 1 / mt_t; # 吨每公吨

那么以下两个表达式都是表示总收入的线性表达式：

\(\sum_{a \in AREA[p]} mt_t \times revenue[p,a,t] \times Sell[p,a,t]\)  
以及  
\(\sum_{a \in AREA[p]} \frac{revenue[p,a,t] \times Sell[p,a,t]}{t\_mt}\)  

继续我们的例子，如果成本也以美元每公吨计，目标函数可以写成：

\[ mt_t \times \sum_{p \in PROD, t \in 1..T} \left( \sum_{a \in AREA[p]} revenue[p,a,t] \times Sell[p,a,t] - prodcost[p] \times Make[p,t] - invcost[p] \times Inv[p,t] \right) \]

或者写成：

\[ \sum_{p \in PROD, t \in 1..T} \frac{ \sum_{a \in AREA[p]} revenue[p,a,t] \times Sell[p,a,t] - prodcost[p] \times Make[p,t] - invcost[p] \times Inv[p,t] }{t\_mt} \]

乘法和除法对任何求和式进行分配，从而得到一个等价的线性项之和。注意在第一种形式中，mt_t 乘以整个求和式 \(\{p \in PROD, t \in 1..T\}\)，而在第二种形式中，t_mt 只除紧跟在 \(\sum \{p \in PROD, t \in 1..T\}\) 后面的被加项，因为 / 运算符比 sum 运算符具有更高的优先级。不过在这些例子中效果是相同的。

最后，if-then-else 运算符在 then 和 else 后面的表达式都是线性表达式，且在 if 和 else 之间的逻辑表达式中不包含任何变量时，将产生线性结果。以下例子出现在第 7.3 节的一个约束中：

\[ Make[j,t] + \left( if\ t = first(WEEKS)\ then\ inv0[j]\ else\ Inv[j,prev[t]] \right) \]

在线性表达式中的变量不能作为任何其他运算符的操作数，也不能作为任何函数的参数。此规则适用于像 max、min、abs、forall 和 exists 这样的迭代运算符，以及 ^ 和标准数值函数如 sqrt、log 和 cos。

总结来说，线性表达式可以是以下形式项的任意和：

- constant-expr var-ref  
- (constant-expr) * (linear-expr)  
- (linear-expr) * (constant-expr)  
- (linear-expr) / (constant-expr)  
- if logical-expr then linear-expr else linear-expr

其中 constant-expr 是任何不包含变量引用的算术表达式，linear-expr 是任何其他（更简单的）线性表达式。如果根据表 A-1 中的运算符优先规则结果相同，则可以省略括号。AMPL 会自动执行将任何此类表达式转换为简单线性项之和的变换。

# 8.3 目标函数

目标函数的声明由关键字 minimize 或 maximize 之一、一个名称、一个冒号以及一个先前定义的集合、参数和变量的线性表达式组成。我们已经看到了如下示例：  
`minimize Total_Cost: sum {j in FOOD} cost[j] * Buy[j];`  
和  
`maximize Total_Profit: sum {p in PROD, t in 1..T} (sum {a in AREA[p]} revenue[p,a,t] * Sell[p,a,t] - prodcost[p] * Make[p,t] - invcost[p] * Inv[p,t]);`

除了在第 15 和第 16 章中将要介绍的某些“按列” (columnwise) 声明外，目标函数的名称在模型中不再起其他作用。在 AMPL 命令中，目标函数的名称表示其值。例如，在求解图 2-1 饮食模型的一个可行实例后，我们可以发出以下命令：

```
ampl: display {j in FOOD} 100 * cost[j] * Buy[j] / Total_Cost;
100*cost[j]*Buy[j]/Total_Cost [*] :=
BEEF     14.4845
CHK       4.38762
FISH      3.8794
HAM      24.4792
MCH      16.0089
MTL      16.8559
SPG      15.6862
TUR       4.21822
```

以显示每种食物所花费的总成本百分比。

尽管特定的线性规划必须有一个目标函数，但一个模型可以包含多个目标函数声明。此外，任何 minimize 或 maximize 声明都可以通过在目标名称后包含一个索引表达式来定义一个目标函数的索引集合。在这些情况下，你可以在输入 solve 命令之前发出 objective 命令，以指示要优化的目标。

作为一个示例，请回忆当我们尝试使用图 2-2 的数据求解图 2-1 的模型时，发现没有解能满足所有约束；随后我们将钠 (NA) 的上限增加到 50000，以使可行解成为可能。合理的问题是：为了让可行解成为可能，钠上限究竟需要增加多少？为此，我们可以引入一个新的目标函数，使其等于饮食中的总钠含量：

```
minimize Total_NA: sum {j in FOOD} amt["NA",j] * Buy[j];
```

（我们仅为钠创建这个目标函数，因为我们没有理由去最小化大多数其他营养成分。）我们可以像以前一样求解总成本的线性规划，因为 AMPL 默认选择模型的第一个目标函数：

```
ampl: model diet.mod;
ampl: data diet2a.dat;
ampl: display n_max["NA"];
n_max['NA'] = 50000
ampl: minimize Total_NA: sum {j in FOOD} amt["NA",j] * Buy[j];
ampl: solve;
MINOS 5.5: optimal solution found.
13 iterations, objective 118.0594032
Objective = Total_Cost
```

求解器告诉我们最低成本，我们也可以使用 display 查看总钠含量，即使当前并未对其进行最小化：

```
ampl: display Total_NA;
Total_NA = 50000
```

接下来，我们可以使用 objective 命令将目标切换为最小化总钠含量。solve 命令随后会使用这个替代目标重新优化，我们显示 Total_Cost 以确定由此产生的成本：

ampl: objective Total_NA;  
ampl: solve;  
MINOS 5.5: optimal solution found.  
1 iterations, objective 48186  
ampl: display Total_Cost;  
Total_Cost = 123.627  

我们看到钠含量可以降低约 1800，尽管成本因此被迫提高了约 $5.50。（一般来说，更健康的饮食更加昂贵，因为它们迫使解偏离最小化成本的解。）

作为另一个例子，以下是我们如何对图 3-2 的办公室分配问题的不同最优解进行实验。首先我们求解原始问题：

ampl: model transp.mod; data assign.dat; solve;  
CPLEX 8.0.0: 最优解;  
目标函数值 28  
24 次对偶单纯形迭代（第 I 阶段 0 次）  
ampl: option display_1col 1000, omit_zero_rows 1;  
ampl: option display_eps .000001;  
ampl: display Total Cost, ampl? {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];  
Total Cost = 28  
cost[i,j]*Trans[i,j] :=  
Coullard C118 6  
Daskin D241 4  
Hazen C246 1  
Hopp D237 1  
Iravani C138 2  
Linetsky C250 3  
Mehrotra D239 2  
Nelson C140 4  
Smilowitz M233 1  
Tamhane C251 3  
White M239 1  

为了在实验过程中保持目标函数值处于该最优水平，我们添加一个约束，将目标函数表达式固定为当前值 28：

ampl: subject to Stay_Optimal;  
ampl? sum {i in ORIG, j in DEST}  
ampl? cost[i,j] * Trans[i,j] = 28;

接下来，回想一下 cost[i,j] 是个体 i 对办公室 j 的评分，而 Trans[i,j] 在最优解中若将个体 i 分配到办公室 j 则为 1，否则为 0。因此，

sum {j in DEST} cost[i,j] * Trans[i,j]

始终等于个体 i 被分配的办公室的评分。我们使用该表达式来声明一个新的目标函数：

ampl: minimize Pref_of {i in ORIG}:  
ampl? sum {j in DEST} cost[i,j] * Trans[i,j];

该语句为每个个体 i 创建一个目标函数 Pref_of[i]，以最小化个体 i 被分配的房间的评分。然后我们可以选择任意一个个体，并优化他或她在分配中的评分：

ampl: objective Pref_of["Coullard"];  
ampl: solve;  
CPLEX 8.0.0: 最优解;  
目标函数值 3  
3 次单纯形迭代（第 I 阶段 0 次）

查看新的分配方案，我们发现原始目标函数值未变，且所选个体的情况确实得到了改善，当然这是以其他人的代价为前提的：

ampl: display Total Cost,  
ampl? {i in ORIG, j in DEsT} cost[i,j] \* Trans[i,j];  
Total Cost $= 28$  
cost[i,j]\*Trans[i,j] :=  
Cullard D241 3  
Daskin D237 1  
Hazen C246 1  
Hopp C251 5  
Iravani C138 2  
Linetsky C250 3  
Mehrotra D239 2  
Nelson C140 4  
Smilowitz M233 1  
Tamhane C118 5  
White M239 1  

我们能够做出这一改变，是因为原始总评分目标函数存在多个最优解。求解器任意返回其中一个，但通过使用第二个目标函数，我们可以将其推向其他解。

# 8.4 约束 (Constraints)

最简单的 约束 (constraint) 声明以关键字 subject to、一个名称和一个冒号开始。 甚至 subject to 也是可选的；AMPL 假定任何不以关键字开头的声明都是一个 约束。 冒号后面是 约束 的代数描述，用先前定义的集合、参数和 变量 (variable) 表示。 因此，在图 1-4 中介绍的 生产 (Make) 模型中，我们有以下由有限 处理 时间施加的 约束：

subject to Time: sum {p in PROD} (1/rate[p]) * Make[p] <= avail;

约束 的名称，像目标函数的名称一样，在 代数模型 (algebraic model) 中的其他地方不会被使用，尽管它出现在替代的“按列”公式中（第 16 章），并且在 AMPL 命令环境中用于指定 约束 的对偶值和其他相关量（第 14 章）。

大型 线性规划 (linear program) 模型中的大多数 约束 都被定义为索引集合，方法是在 约束 名称后给出一个索引表达式。例如，约束 Time 在后续示例中被推广，以说明 生产时间 (production time) 不得超过每个处理阶段 s 中可用的时间（图 1-6a）：

```ampl
subject to Time {s in STAGE}: sum {p in PROD} (1/rate[p,s]) * Make[p] <= avail[s];
```

或在每周 t 中（图 4-4）：

```ampl
subject to Time {t in 1..T}: sum {p in PROD} (1/rate[p]) * Make[p,t] <= avail[t];
```

后一个示例中的另一个 约束 表明，对于每周 t 中的每种 产品 (product) p，生产 (Make)、销售 (Sell) 和 库存 (Inv) 必须平衡：

```ampl
subject to Balance {p in PROD, t in 1..T}: Make[p,t] + Inv[p,t-1] = Sell[p,t] + Inv[p,t];
```

约束 声明可以指定任何有效的索引表达式，该表达式定义一个集合（如第 5 和第 6 章所述）；对于该集合的每个成员，都有一个 约束。约束 名称可以带下标，因此 Time[1] 或 Balance[p,t+1] 指的是索引集合中的特定 约束。

约束 声明中的索引表达式应该为索引集合的每个维度指定一个虚拟索引（如前面示例中的 s、t 和 p）。然后，当 AMPL 处理与特定索引集合成员对应的 约束 时，虚拟索引从该成员中获取其值。这种虚拟索引的使用使得单个 约束 表达式能够表示许多 约束；索引表达式是 AMPL 对模型的代数陈述中可能出现的诸如“对于所有产品 p 和周 t = 1 到 T”这样的短语的翻译。

通过使用更复杂的索引表达式，您可以更精确地指定要包含在模型中的 约束。例如，考虑以下 生产时间 约束 的变体：

```ampl
subject to Time {t in 1..T: avail[t] > 0}: sum {p in PROD} (1/rate[p]) * Make[p,t] <= avail[t];
```

这表示如果在数据中将 `avail[t]` 指定为零，则应将其解释为“第 t 周的时间无限制”，而不是“第 t 周的时间限制为零”。在更简单的情况下，如果只有一个不按周索引的 `Time` 约束，你可以指定如下类似的条件定义：

```ampl
subject to Time {if avail > 0}: sum {p in PROD} (1/rate[p]) * Make[p] <= avail;
```

伪索引表达式 `{if avail > 0}` 表示如果条件 `avail > 0` 为真，则生成一个名为 `Time` 的约束；如果条件为假，则不生成任何约束。（同样的符号也可用于条件定义其他模型组件。）

在 AMPL 中，约束的代数描述可以是任意两个由等号或不等号分隔的线性表达式：

```
linear-expr <= linear-expr
linear-expr = linear-expr
linear-expr >= linear-expr
```

尽管在线性规划的数学描述中通常将包含变量的所有项放在运算符左侧，其他项放在右侧（如 `Time` 约束所示），但 AMPL 并不要求如此（如 `Balance` 约束所示）。

应根据方便性和可读性来决定将哪些项放在运算符的哪一侧。AMPL 会自动处理约束的规范化，例如合并涉及相同变量的线性项，或将变量从约束的一侧移到另一侧。第 1.4 节中描述的 `expand` 命令可以显示约束的规范形式。

AMPL 还允许如下所示的双重不等式约束，例如来自图 2-1 的饮食模型：

```ampl
subject to Diet {i in NUTR}: 
    n_min[i] <= sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i];
```

这表示中间表达式（即所有食物提供的营养 i 的总量）必须大于等于 `n_min[i]`，同时小于等于 `n_max[i]`。此类约束的允许形式为：

```
const-expr <= linear-expr <= const-expr
const-expr >= linear-expr >= const-expr
```

其中每个 `const-expr` 必须不包含变量。其效果是对 `linear-expr` 的值施加上下界。如果模型需要在左侧或右侧的 `const-expr` 中包含变量，则必须在单独的声明中定义两个不同的约束。

在线性规划的大多数应用中，你无需担心约束的形式。如果你只是以最方便的方式编写约束，它们将根据本章的规则被识别为适当的线性约束。然而，确实存在一些情况，你的建模选择将决定 AMPL 是否将你的模型识别为线性模型。假设我们希望进一步约束生产模型，使得任何产品 p 的生产量不得超过总生产量的某个比例。我们定义一个参数 max_frac 来表示这个限制比例；约束则表示为：产品 p 的生产量除以总生产量必须小于或等于 max_frac：

subject to Limit {p in PROD}: Make[p] / sum {q in PROD} Make[q] <= max_frac;

对 AMPL 来说，这不是一个线性约束，因为其左侧表达式包含对变量之和的除法运算。但如果我们将其重写为：

subject to Limit {p in PROD}: Make[p] <= max_frac * sum {q in PROD} Make[q];

那么 AMPL 就会将其识别为线性约束。

AMPL 在准备将模型和数据传递给求解器时会对约束进行简化。例如，它可能会消除固定值的变量，将单变量约束与变量的简单边界合并，或删除被其他约束所蕴含的约束。通常你可以忽略这个预处理阶段，但有办法观察其效果并修改其行为，如第 14.1 节所述。

# 练习

8-1. 在图 5-1 的饮食模型中，向变量声明添加一个 $\coloneqq$ 短语（如第 8.1 节所述），以将每个变量初始化为其上下界之间的中点值。

将此模型连同图 5-2 中的数据一起读入 AMPL。使用 display 命令确定初始解未能满足哪些约束（如果有的话），并计算此解的总成本。总成本比最优总成本高还是低？

8-2. 本练习要求你重新表述各种类型的约束，使其成为线性约束。

(a) 下列 约束 (constraint) 表示产品 $\mathbb{P}$ 在任何 周期 (period) t 的 库存 (inventory) Inv[p,t] 不得超过该产品单 周期 (period) 生产量 (production) Make[p,t] 的最小值：

subject to Inv_Limit {p in PROD, t in 1..T}: Inv[p,t] $\leq$ min {tt in 1..T} Make[p,tt];

由于此 约束 (constraint) 对 变量 (variable) 应用了 min 运算符，因此 AMPL 不将其识别为线性 约束 。请制定一个具有相同效果的线性 约束 。

(b) 下列 约束 表示从一个 周期 (period) 到下一个 周期 总 库存 (inventory) 的变化不得超过某个参数 max_change：

subject to Max_Change {t in 1..T}: abs(sum {p in PROD} Inv[p,t-1] - sum {p in PROD} Inv[p,t]) $\leq$ max_change;

由于此 约束 对包含 变量 (variable) 的表达式应用了 abs 函数，因此它不是线性的。请制定一个具有相同效果的线性 约束 。

(c) 下面的 约束 表示某一期中总 生产量 (production) 与总 库存 (inventory) 量的比值不得超过 max_inv_ratio：

subject to Max_Inv_Ratio {t in 1..T}: (sum {p in PROD} Inv[p,t]) / (sum {p in PROD} Make[p,t]) $\leq$ max_inv_ratio;

该 约束 不是线性的，因为它将一个 变量 (variable) 的总和除以另一个 变量 的总和。请提出一个具有相同效果的线性 约束 。

(d) 对于以下情况，关于替代线性 约束 的表述，你能说些什么？

- 在 (a) 中，min 被 max 替换。
- 在 (b) 中，$\leq$ max_change 被 $\geq$ min_change 替换。
- 在 (c) 中，参数 max_inv_ratio 被一个新 变量 Ratio[t] 替换。

8-3. 本练习探讨在饮食模型中使用多个 目标函数 (objective function) 的一些可能性。这里我们考虑 图 5-1 的模型，以及 图 5-2 的数据。假设 成本 (cost) 不仅按 食物 (food) 索引，还按 商店 (store) 索引：

set STORE: param cost {STORE,FOOD} > 0;

然后可以为每个 商店 (store) 定义一个单独的 目标函数 (objective function)：

minimize Total_Cost {s in STORE}: sum {j in FOOD} cost[s,j] * Buy[j];

考虑以下三家 商店 (store) 的数据：

<table><tr><td colspan="11">set STORE := "A&P" JEWEL VONS ;</td></tr><tr><td>param cost:</td><td>BEEF</td><td>CHK</td><td>FISH</td><td>HAM</td><td>MCH</td><td>MTL</td><td>SPG</td><td>TUR</td><td>: =</td><td></td></tr><tr><td>"A&P"</td><td>3.19</td><td>2.59</td><td>2.29</td><td>2.89</td><td>1.89</td><td>1.99</td><td>1.99</td><td>2.49</td><td></td><td></td></tr><tr><td>JEWEL</td><td>3.09</td><td>2.79</td><td>2.29</td><td>2.59</td><td>1.59</td><td>1.99</td><td>2.09</td><td>2.30</td><td></td><td></td></tr><tr><td>VONS</td><td>2.59</td><td>2.99</td><td>2.49</td><td>2.69</td><td>1.99</td><td>2.29</td><td>2.00</td><td>2.69</td><td>;</td><td></td></tr></table>

使用 objective 命令，找出每家 商店 (store) 的最低 成本 (cost) 饮食方案。哪家 商店 (store) 提供最低的总 成本 (cost) ？

现在考虑一个额外的 目标函数 (objective function)，表示购买的总包数，而不考虑 成本 (cost)：

minimize Total_Number: sum {j in FOOD} Buy[j];

该 目标函数 (objective function) 的最小值是多少？当该 目标函数 最小时，三家 商店 (store) 的 成本 (cost) 分别是多少？解释为什么你会预期这些 成本 (cost) 高于 (a) 中计算的 成本 (cost) 。

8-4. 本练习与第 8.3 节的指派示例有关。

(a) 在总排名保持在最优值 28 的前提下，你能为每个人指派的最佳排名办公室是什么？似乎有多少种不同的最优指派方案，哪些人在不同指派方案中会得到不同的办公室？

(b) 修改指派示例，使其能够找出在总排名可以从 28 增加但不得超过 30 的前提下，你能为每个人指派的最佳排名办公室。

(c) 在进行 (b) 中建议的修改后，负责分配办公室的人员再次尝试最小化目标函数 Pref_of["Couillard"]。这次报告的解如下：

amp1：display Total Cost, amp1? {i in ORIG, j in DEST} cost[i,j]*Trans[i,j]; 总成本 $= 30$ cost[i,j]\*Trans[i,j] :=  
Coullard M239 1  
Daskin D241 4  
Hazen C246 1  
Hopp C251 2.5  
Hopp D237 0.5  
Irawani C138 2  
Linetsky C250 3  
Mehrotra D239 2  
Nelson C140 4  
SmiLowitz M233 1  
Tamlane C118 5  
White C251 2.5  
White D237 1.5

现在 Coulard 被分配到了她的第一选择，但整个解存在什么问题？为什么它不能为我们所陈述的分配问题提供一个有用的解决方案？

8-5. 回到运输模型的分配版本，见图 3-1a 和图 3-2。

(a) 对于每个 i in ORIG，添加参数 worst[i]，并添加约束，规定对于每一个 i in ORIG 和 j in DEST 的组合，若 cost[i, j] 大于 worst[i]，则 Trans[i, j] 必须等于 0。（参见第 8.4 节中的约束 Time 以获得类似示例。）在该模型的分配解释中，这些新约束意味着什么？

(b) 使用 (a) 中的模型，展示存在一个目标函数值为 28 的最优解，在该解中，没有人被分配到比其第五选择更差的办公室。

(c) 使用 (a) 中的模型，证明至少有一个人必须被分配到比其第四选择更差的办公室。

(d) 使用 (a) 中的模型，证明如果在不对其他人选择施加任何限制的情况下，给 Nelson 分配他的第一选择，则目标函数值不能小于 31。类似地，确定如果给其他每个人分配其第一选择，目标函数值可以达到的最小值。
