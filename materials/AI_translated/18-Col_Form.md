
# 列式建模

由于优化问题的基本思想是在变量的约束条件下最小化或最大化某个函数，因此 AMPL 倾向于对变量和约束进行显式描述。这就是为什么 var 声明通常出现在最前面，随后是使用这些变量的 minimize 或 maximize 以及 subject to 声明。本书中的示例表明，这种方法可以用于建立各种各样的优化问题模型。

然而，对于某些类型的线性规划 (linear program)，在声明变量之前先声明目标函数和约束 (constraint) 可能更为合适。通常在这些情况下，单个变量在所有约束中的系数所呈现出的模式或解释，要比单个约束在所有变量中的系数更为简单。在线性规划的专业术语中，相比于按“行” (row-wise) 描述约束系数矩阵，按“列” (columnwise) 描述更为容易。因此，通过首先声明约束和目标函数，然后在变量的声明中列出非零系数，可以简化模型的构建。

第 15 章中描述的 网络线性规划 (network linear program) 是这种现象的一个例子。每个 变量 (variable) 在 约束 (constraint) 中最多只有两个非零系数，即一个 $+1$ 和一个 $-1$。与其尝试用代数方式描述这些 约束 (constraint)，你可能会发现，在每个 变量 (variable) 的声明中直接指定该 变量 (variable) 所涉及的一个或两个 约束 (constraint) 更为简便。实际上，这正是你在 15.3 节中引入的特殊 节点 (node) 和 弧 (arc) 声明中所做的事情。节点声明首先出现，用于描述网络节点处的 约束 (constraint) 特性。然后，弧声明通过使用 from 和 to 短语，定义网络流 变量 (variable) 并确定它们在节点 约束 (constraint) 中的非零系数位置。这种方法特别具有吸引力，因为它直接对应于大多数人思考网络流问题的方式。

对于 AMPL 来说，为每一种你可能希望按列而非按行声明的 线性规划 (linear program) 提供特殊的声明和短语是不切实际的。相反，通过为 var 和 subject to 声明增加额外选项，AMPL 允许任何 线性规划 (linear program) 采用按列声明的方式。本章将通过两个对比鲜明的例子——投入产出生产模型和排班调度模型——介绍 AMPL 的按列建模特性，并以可用于按列建模的语言扩展的总结作为结尾。

## 16.1 投入产出模型

在第 1 章中的简单最大利润生产模型中，生产的商品与消耗的资源是明确区分的，因此总体生产显然受到可用资源的限制。然而，在更真实的复杂操作（如 钢铁厂 (steel mill) 或炼油厂）模型中，生产是在一系列单元中进行的；因此，某些生产单元的输入可能是其他单元的输出。在这种情况下，我们需要一个更通用的模型，能够处理既可能是输入也可能是输出的 物料 (MAT)，以及涉及多个输入和输出的 活动 (ACT)。

我们首先以通常的按行（或按 约束 (constraint)）方式开发一个 AMPL 模型。然后，我们解释按列（或按 变量 (variable)）的替代方法，并讨论模型的改进。

### 按约束建模

我们的模型定义从一组 物料 (MAT) 和一组 活动 (ACT) 开始：

```ampl
set MAT;
set ACT;
```

关键数据是所有 物料 (MAT)-活动 (ACT) 组合的投入-产出系数：

```ampl
param io {MAT,ACT};
```

如果 $\mathrm{io}[i,j] > 0$，则解释为 活动 (ACT) $j$ 的一个单位产生的 物料 (MAT) $i$ 的数量（作为产出）。另一方面，如果 $\mathrm{io}[i,j] < 0$，则表示 活动 (ACT) $j$ 的一个单位消耗的 物料 (MAT) $i$ 的数量的负值（作为投入）。例如，值 10 表示每单位 $j$ 产生 10 个单位的 $i$，而值 $-10$ 表示每单位 $j$ 消耗 10 个单位的 $i$。当然，我们可以预期对于许多 $i$ 和 $j$ 的组合，将有 $\mathrm{io}[i,j] = 0$，表示 物料 (MAT) $i$ 根本不参与 活动 (ACT) $j$。

要了解为什么我们要以这种方式解释 $\mathrm{io}[i,j]$，假设我们定义 Run[j] 为我们运行 活动 (ACT) $j$ 的水平：

param act_min $\{\mathrm{ACT}\} > = 0$。param act_max {j in ACT} >= act_min[j]；var Run {j in ACT} >= act_min[j], $\epsilon =$ act_max[j]；

那么 $\mathrm{i}\circ [\mathrm{i},\mathrm{j}]^{\star}\mathrm{Run}[\mathrm{j}]$ 是活动 j 产生的物料 i 的总量（如果 $\mathrm{i}\circ [\mathrm{i},\mathrm{j}] > 0$）或消耗的物料 i 的总量的负值（如果 $\mathrm{i}\circ [\mathrm{i},\mathrm{j}]< 0$）。对所有活动求和，我们看到

set MAT; # 物料
set ACT; # 活动
param io {MAT,ACT}; # 投入-产出系数
param revenue {ACT};
param act_min {ACT} >= 0;
param act_max {j in ACT} >= act_min[j];
var Run {j in ACT} >= act_min[j], <= act_max[j];
maximize Net_Profit: sum {j in ACT} revenue[j] * Run[j];
subject to Balance {i in MAT}: sum {j in ACT} io[i,j] * Run[j] = 0;

图 16-1：按行排列的投入-产出模型 (iorow.mod)。

sum {j in ACT} io[i,j] * Run[j]

表示运营中产生的物料 i 的数量减去消耗的数量。这些数量必须平衡，如下约束所表达：

subject to Balance {i in MAT}: sum {j in ACT} io[i,j] * Run[j] = 0;

那么资源的可用性或成品的需求呢？这些可以通过代表物料采购或销售的额外活动轻松建模。物料 i 的采购活动没有投入，只有 i 作为产出；Run[i] 的上限表示此资源的可用数量。类似地，物料 i 的销售活动没有产出，只有 i 作为投入，Run[i] 的下限表示必须生产用于销售的此商品的数量。

我们通过将单位收入 (unit revenues) 与活动相关联来完成模型。销售活动必然产生正向收入，而采购和生产活动则产生负向收入 —— 也就是成本。单位收入与活动水平的乘积之和即为运营的总净利润：

```ampl
param revenue {ACT};
maximize Net_Profit: sum {j in ACT} revenue[j] * Run[j];
```

完整的模型如图 16-1 所示。

# 一种按列组织的模型公式 (A columnwise formulation)

正如我们对采购和销售活动的讨论所提示的，本模型中的所有内容都可以按活动 (activity) 来组织。具体来说，对于每个活动 $j$，我们有一个决策变量 Run[j]、一个由 revenue[j] 表示的成本或收入、上下限 act_min[j] 和 act_max[j]，以及一组输入-输出系数 io[i,j]。诸如提高某个装置的收率或获得新的供应来源等变化，都可以通过添加一个活动或修改现有活动的数据来实现。

在按行组织的模型公式 (formulation by rows) 中，活动在该模型中的重要性被部分掩盖了。虽然 act_min[j] 和 act_max[j] 出现在变量的声明中，revenue[j] 出现在目标函数中，而 io[i,j] 的值出现在约束声明中。而按列组织的替代方案通过在 var 声明中添加 obj 和 coeff 子句，将所有这些信息整合在一起：

```ampl
var Run {j in ACT} >= act_min[j], <= act_max[j],
    obj Net_Profit revenue[j],
    coeff {i in MAT} Balance[i] io[i,j];
```

obj 子句表示，在名为 Net_Profit 的目标函数中，变量 Run[j] 的系数为 revenue[j]；也就是说，应当将项 revenue[j] * Run[j] 加入目标函数。coeff 子句稍微复杂一些，因为它被一个集合索引。它表示对于每种物料 $i$，在约束 Balance[i] 中，变量 Run[j] 应当具有系数 io[i,j]，因此项 io[i,j] * Run[j] 应当被加入。这些子句一起描述了线性规划中所有变量的所有系数。

由于我们将所有系数都放在了 var 声明中，因此必须将它们从其他声明中移除：

```ampl
maximize Net_Profit;
subject to Balance {i in MAT}: to_come = 0;
```

关键字 to_come 指示由 var 声明生成的项 io[i,j] * Run[j] 应当被“加入”到哪里。你可以将 to_come = 0 视为一个约束的模板，当系数被声明时，该模板将被填充。然而，在本例中，由于目标函数仅仅是项 revenue[j] * Run[j] 的总和，因此不需要模板。如下面第 16.3 节所示，模板可以用有限的几种方式编写。

由于 obj 和 coeff 子句引用了 Net_Profit 和 Balance，因此在按列组织的模型公式中，var 声明必须出现在 maximize 和 subject to 声明之后。完整模型如图 16-2 所示。

set MAT; # 物料  
set ACT; # 活动  
param io {MAT,ACT}; # 投入-产出系数  
param revenue {ACT};  
param act_min {ACT} >= 0;  
param act_max {i in ACT} >= act_min[j];  
maximize Net_Profit;  
subject to Balance {i in MAT}: to_come = 0;  
var Run {j in ACT} >= act_min[j], <= act_max[j],  
obj Net_Profit revenue[j],  
coeff {i in MAT} Balance[i] io[i,j];  

图 16-2：按列建模 (iocoll.mod)。

# 按列建模的改进

随着模型变得更加复杂，按列建模方法的优势更加明显。例如，如果我们希望引入单独的变量来表示成品物料的销售情况，可以声明一个可销售物料的子集，并用它来索引新的边界、收入和变量集合：

set MATF within MAT; # 成品物料  
param revenue {MATF} $>= 0$.  
param sell_min {MATF} $>= 0$.  
param sell_max {i in MATF} $>=$ sell_min[i];  
var Sell {i in MATF} $>=$ sell_min[i], $<=$ sell_max[i];  

我们现在可以取消之前描述的特殊销售活动。由于 ACT 的其余成员代表采购或生产活动，我们可以引入一个与之相关的非负参数 cost：

$$
\mathtt{param}\ \mathtt{cost}\{\mathtt{ACT}\} >= 0;
$$

在按行建模方法中，新的目标函数写为：

maximize Net_Profit: sum {i in MATF} revenue[i] * Sell[i] - sum {j in ACT} cost[j] * Run[j];  

表示总销售收入减去原材料和生产成本。

到目前为止，我们似乎已经改进了图 16-1 中的模型。净利润的构成更清晰地建模，销售仅限于明确指定的成品物料；同时，通过 display Sell 等命令，可以更容易地单独检查最优销售量与其他变量。接下来需要完善约束条件。我们希望表达物料 i 来自所有活动的净产出，如图 16-1 中所示：

$$
\mathtt{sum}\{\mathtt{j}\ \mathtt{in}\ \mathtt{ACT}\}\ \mathtt{io}[\mathtt{i},\mathtt{j}] * \mathtt{Run}[\mathtt{j}]
$$

必须与销售量平衡 —— 如果 i 是成品物料 (MATF)，则为 Sell[i]，否则为 0。因此，约束声明应写为：

subject to Balance {i in MAT} sum {j in ACT} io[i,j] * Run[j] $=$ if i in MATF then Sell[i] else 0;

不幸的是，由于 if-then-else 表达式引入的复杂性，这一约束似乎不如我们最初的约束清晰。

在按列建模的替代方案中，目标函数和约束与图 16-2 中相同，而所有变化都反映在变量声明中：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/afa8eb2f666ad90f9e275b4590237047105e4507fd5d91ccfa074d3646304e62.jpg)  
图 16-3：按列建模，包含销售活动 (iocol2.mod)。

var Run {j in ACT} >= act_min[j], <= act_max[j],  
obj Net_Profit - cost[j],  
coeff {i in MAT} Balance[i] io[i,j];  

var Sell {i in MAT} >= sell_min[i], <= sell_max[i],  
obj Net_Profit revenue[i],  
coeff Balance[i] -1;  

在这种观点下，变量 Sell[i] 表示我们之前描述的销售活动类型，仅以物料 i 作为输入，没有物料作为输出 —— 因此在约束 Balance[i] 中只有一个 -1 的系数。我们不需要为 Sell[i] 指定所有零系数；在声明中的 coeff 短语里未明确引用的任何约束，默认系数为零。整个模型如图 16-3 所示。

这个例子表明，列式方法特别适用于区分不同活动类型的投入产出 (input-output) 模型的改进。例如，可以很容易地添加另一组变量来表示原材料的采购。

另一方面，涉及大量特殊约束的投入产出模型版本则更适合按行进行建模。

# 16.2 一个调度模型

在第 2.4 节中，我们观察到混合模型的一般形式适用于某些调度问题。这里我们描述一个相关的调度模型，该模型特别适合采用列式方法。

假设一个工厂下周的生产被划分为固定的时间段，或称为班次 (shift)。你希望将员工分配到各个班次，使得每个班次都有所需数量的人员在工作。然而，你不能独立地安排每个班次，因为只有某些每周的排班表 (schedule) 是允许的；例如，一个人不能连续工作五个班次。因此，你的问题更恰当地被视为将员工分配到排班表的问题，使得每个班次都被覆盖，并且整体分配是最经济的。

我们可以方便地用班次的索引子集集合来表示这个问题的排班表：

set SHIFTS; # 班次  
param Nsched; # 排班表数量;  
set SCHEDS = 1..Nsched; # 排班表集合  
set SHIFT_LIST {SCHEDS} within SHIFTS;  

对于集合 SCHEDS 中的每个排班表 j，一个人在排班表 j 上工作的班次包含在集合 SHIFT_LIST[j] 中。我们还为每个排班表指定每人薪资率，以及每个班次所需的人员数量：

param rate {SCHEDS} >= 0; param required {SHIFTS} >= 0;  

我们令变量 Work[j] 表示分配到排班表 j 上工作的人数，并最小化所有排班表上 rate[j] * Work[j] 的总和：

var Work {SCHEDS} >= 0; minimize Total_Cost: sum {j in SCHEDS} rate[j] * Work[j];

最后，我们的约束条件说明，分配到每个班次 $i$ 的员工总数必须至少满足所需人数：

```ampl
subject to Shift_Needs {i in SHIFTS}: 
    sum {j in SCHEDS: i in SHIFT_LIST[j]} Work[j] >= required[i];
```

在左侧，我们对所有满足 $i \in \text{SHIFT\_LIST}[j]$ 的排班表 $j$ 求 $\text{Work}[j]$ 的总和。这个总和表示被分配到包含班次 $i$ 的排班表上的员工总数，因此等于覆盖班次 $i$ 的员工总数。

这种表述方式中的约束条件比较别扭，这促使我们尝试按列（columnwise）的方式来建模。和前面的例子一样，我们首先声明目标函数和约束条件，但先不声明变量：

最小化 总成本；满足  
$\text{Shift\_Needs}\ \{i \in \text{SHIFTS}\}: \text{to\_come} \geq \text{required}[i];$

$\text{Work}[j]$ 的系数出现在它的 var 声明中。在目标函数中，它的系数是 $\text{rate}[j]$。在约束条件中，$\text{SHIFT\_LIST}[j]$ 的成员关系准确地告诉了我们需要知道的信息：对于 $\text{SHIFT\_LIST}[j]$ 中的每个 $i$，$\text{Work}[j]$ 在约束 $\text{Shift\_Needs}[i]$ 中的系数为 1，而在其他约束中的系数为 0。这引导我们得到如下简洁的声明：

```ampl
var Work {j in SCHEDS} >= 0,
    obj Total_Cost rate[j],
    coeff {i in SHIFT_LIST[j]} Shift_Needs[i] 1;
```

完整模型如图 16-4 所示。

作为该模型的一个具体实例，假设你从周一到周五每天有三个班次，周六有两个班次。每天在第一、二、三班次分别需要 100、78 和 52 名员工。为了简化问题，假设无论安排哪种班表，每人成本相同，因此你可以通过将 $\text{rate}[j]$ 设置为 1 来最小化员工总数。

至于班表安排，一个合理的排班规则可能是每位员工一周工作五个班次，但在任何 24 小时内最多只能工作一个班次。数据文件的一部分如图 16-5 所示；我们没有展示完整文件，因为满足该规则的班表共有 126 种！由此产生的含有 126 个变量的 线性规划 (linear program) 并不难求解：

```ampl
ampl: model sched.mod; data sched.dat; solve;
MINDS 5.5: optimal solution found.
19 iterations, objective 265.6
ampl: option display_eps .000001;
ampl: option omit_zero_rows 1;
ampl: option display_1col 0, display_width 60;
ampl: display Work;
Work [*] :=
 10  28.8   30  14.4   71  35.6  106  23.2  123  35.6
 18   7.6   35   6.8   73  28.0  109  14.4   24   6.8
 66  35.6   87  14.4  113  14.4 ;
```

如你所见，这个 最优解 (optimal solution) 使用了 13 种班表，其中一些是分数数量。（这个问题还存在许多其他的最优解，所以你得到的结果可能有所不同。）如果你将这个解中的每个分数都向上取整到下一个最大值，你会得到一个使用 271 名员工的相当不错的可行解。然而，要确定这是否是最好的整数解，则需要使用整数规划技术，这将在第 20 章中讨论。

在这种情况下，列式表述的便利性直接源于我们选择表示数据的方式。我们设想建模者会从时间表的角度进行思考，并希望尝试增加、删除或修改不同的时间表，以查看可以获得哪些解决方案。子集 `SHIFT_LIST[j]` 提供了一种方便且简洁的方式来维护数据中的时间表。由于数据是按时间表组织的，且每个时间表都有一个对应的变量，因此这种组织方式被证明是更简单且在更大规模问题中更高效的，可以通过变量来指定系数。

```ampl
set SHIFTS; # 班次
param Nsched; # 时间表数量
set SCHEDS = 1..Nsched; # 时间表集合
set SHIFT_LIST {SCHEDS} within SHIFTS;
param rate {SCHEDS} >= 0;
param required {SHIFTS} >= 0;
minimize Total_Cost;
subject to Shift_Needs {i in SHIFTS}: 
    sum {j in SCHEDS: i in SHIFT_LIST[j]} Work[j] >= required[i];
var Work {j in SCHEDS} >= 0,
    obj Total_Cost rate[j],
    coeff {i in SHIFT_LIST[j]} Shift_Needs[i] 1;
```

图 16-4：列式调度模型 (sched.mod)

```ampl
set SHIFTS := Mon1 Tue1 Wed1 Thu1 Fri1 Sat1
              Mon2 Tue2 Wed2 Thu2 Fri2 Sat2
              Mon3 Tue3 Wed3 Thu3 Fri3;

param Nsched := 126;

set SHIFT_LIST[1] := Mon1 Tue1 Wed1 Thu1 Fri1;
set SHIFT_LIST[2] := Mon1 Tue1 Wed1 Thu1 Fri2;
set SHIFT_LIST[3] := Mon1 Tue1 Wed1 Thu1 Fri3;
set SHIFT_LIST[4] := Mon1 Tue1 Wed1 Thu1 Sat1;
set SHIFT_LIST[5] := Mon1 Tue1 Wed1 Thu1 Sat2;

# （省略 117 行）

set SHIFT_LIST[123] := Tue1 Wed1 Thu1 Fri2 Sat2;
set SHIFT_LIST[124] := Tue1 Wed1 Thu2 Fri2 Sat2;
set SHIFT_LIST[125] := Tue1 Wed2 Thu2 Fri2 Sat2;
set SHIFT_LIST[126] := Tue2 Wed2 Thu2 Fri2 Sat2;

param rate default 1;

param required := Mon1 100 Mon2 78 Mon3 52
                 Tue1 100 Tue2 78 Tue3 52
                 Wed1 100 Wed2 78 Wed3 52
                 Thu1 100 Thu2 78 Thu3 52
                 Fri1 100 Fri2 78 Fri3 52
                 Sat1 100 Sat2 78;
```

这类模型用于各种调度问题。为了方便起见，可以使用关键字 `cover`（类似于网络中的 `from` 和 `to`）来指定系数为 1：

```ampl
var Work {j in SCHEDS} >= 0,
    obj Total_Cost rate[j],
    cover {i in SHIFT_LIST[j]} Shift_Needs[i];
```

一些最著名且规模最大的例子出现在航空公司的机组调度中，其中变量可能代表机组的分配而不是个人的分配，班次变为航班，而要求是每个航班配备一个机组。这就形成了所谓的集合覆盖问题 (set covering problem)，其目标是以最经济的方式用代表机组时间表的子集来覆盖所有航班的集合。

## 16.3 列式表述的规则

AMPL 约束的代数描述可以写成以下任意形式：

`arith-expr` $\leq$ `arith-expr`  
`arith-expr` $=$ `arith-expr`  
`arith-expr` $\geq$ `arith-expr`  
`const-expr` $\leq$ `arith-expr` $\leq$ `const-expr`  
`const-expr` $\geq$ `arith-expr` $\geq$ `const-expr`

每个 `const-expr` 必须是一个不包含变量的算术表达式，而 `arith-expr` 可以是任何有效的算术表达式——但如果结果是一个 线性规划 (linear program)，则它必须在变量上是线性的（第 8.2 节）。为了允许列式建模，其中一个 `arith-expr` 可以写成如下形式之一：

`to_come` `to_come + arith-expr` `arith-expr + to_come`

通常，这种类型的“模板”约束（如我们的例子中所示）由 `to_come`、一个关系运算符和一个 `const-expr` 组成；约束中的线性项全部在后续的 `var` 声明中提供，`to_come` 显示了它们应该插入的位置。如果模板约束确实包含变量，则这些变量必须来自之前的 `var` 声明，此时模型就成为一种行式与列式建模的混合形式。

目标函数的表达式也可以按上述方式之一包含 `to_come`。如果目标函数是通过后续 `var` 声明完全指定的线性项之和（如我们的例子中所示），则目标函数的表达式仅为 `to_come`，可以省略。

在 `var` 声明中，约束系数可以通过一个或多个短语指定，每个短语由关键字 `coeff`、可选的索引表达式、约束名称和一个 `arith-expr` 构成。如果存在索引表达式，则为索引集合中的每个成员生成一个系数；否则只生成一个系数。索引表达式也可以采用特殊形式 `{if logical-expr}`，如第 8.4 或 15.3 节中所示，在这种情况下，仅当 `logical-expr` 的值为真时才生成系数。我们的简单示例中每个 `var` 声明只需要一个 `coeff` 短语，

```ampl
set CITIES; 
set LINKS within (CITIES cross CITIES); 
set PRODS; 
param supply {CITIES,PRODS} >= 0;      # 城市 i 中产品 p 的 供应量 (supply)
param demand {CITIES,PRODS} >= 0;      # 城市 j 中产品 p 的 需求量 (demand)
check {p in PRODS}: 
   sum {i in CITIES} supply[i,p] = sum {j in CITIES} demand[j,p];
param cost {LINKS,PRODS} >= 0;         # 运输成本/每 1000 件产品
param capacity {LINKS,PRODS} >= 0;     # 每条线路最大运输件数
param cap_joint {LINKS} >= 0;          # 每条线路最大总运输件数
minimize Total_Cost;
node Balance {k in CITIES, p in PRODS}: net_in = demand[k,p] - supply[k,p];
subject to Multi {(i,j) in LINKS}: 
   to_come <= cap_joint[i,j];
arc Ship {(i,j) in LINKS, p in PRODS} >= 0, <= capacity[i,j,p],
   from Balance[i,p], to Balance[j,p],
   coeff Multi[i,j] 1.0,
   obj Total_Cost cost[i,j,p];
```

但一般来说，对于变量出现的每个不同的带索引的约束集合，都需要一个单独的 `coeff` 短语。

目标函数系数可以用相同的方式指定，只是此时使用关键字 `obj` 代替 `coeff`。

在网络的弧声明中使用的 `obj` 短语与在 `var` 声明中使用的 `obj` 短语相同（第 15.4 节）。由弧声明定义的网络变量的约束系数通常在 `from` 和 `to` 短语中给出，但也可能存在 `coeff` 短语；如果希望以列方式描述除流平衡约束之外的“附加”约束，它们可能会很有用。例如，图 16-6 展示了如何使用 `coeff` 短语以完全列方式重写图 15-13 中的多商品流模型。

# 参考文献

Bibliography Gerald Kahan, "Walking Through a Columnar Approach to Linear Programming of a Business." Interfaces 12, 3 (1982) pp. 32- 39. 对线性规划的列方式方法的简要介绍，并附有一个小例子。

# 练习

16-1. (a) 构造图 2-1 中饮食模型的列式表述。

(b) 构造图 5-1 中饮食模型的列式表述。由于此饮食模型中存在两个独立的约束集合，因此你需要在 `var` 声明中使用两个 `coeff` 短语。

16-2. 扩展图 16-3 中的列式生产模型，以包含表示原材料购买量的变量 `Buy[i]`。

16-3. 使用第 4.2 节中引入的方法，为图 16-3 中的模型构建一个多周期版本：首先在一组周上复制该模型，然后引入库存变量将各周连接起来。对所有变量（包括库存变量）仅使用列式声明。

16-4. “卷材修边”或“下料”问题与本章描述的调度问题有许多共同之处。回顾练习 2-6 中对卷材修边问题的描述，并用它来回答以下问题。

(a) 卷材修边问题中的可用切割模式与线性规划表述中变量的系数之间有何关系？  
(b) 构造一个仅使用变量列式声明的卷材修边问题的 AMPL 模型。  
(c) 对练习 2-6 中给出的数据求解卷材修边问题。作为对你的模型的测试，证明它给出的最优值与模型的行式表述相同。

16-5. 在第 16.2 节末尾提到的集合覆盖问题可以一般性地表述如下。给定一个集合 $S$ 和 $S$ 的某些子集 $T_{1},T_{2},\ldots ,T_{n}$，每个子集都关联一个成本。如果所选的某些子集 $T_{j}$ 的并集包含 $S$ 中的每个元素，则称该选择覆盖了 $S$。例如，如果

$$
\texttt{S} = \{1,2,3,4\}
$$

且

$$
\begin{array}{rlrlrlrlrl}{\mathrm{T1}}&{=\{1,2,4\}}&{\mathrm{T2}}&{=\{2,3\}}&{\mathrm{T3}}&{=\{1\}}&{\mathrm{T4}}&{=\{3,4\}}&{\mathrm{T5}}&{=\{1,3\}}\end{array}
$$

选择 (T1,T2) 和 (T2,T4,T5) 覆盖了 S，但选择 (T3,T4,T5) 没有覆盖。集合覆盖问题 (set covering problem) 的目标是找到成本最低的子集选择，使得这些子集能够覆盖 S。请为任意给定集合和子集构造一个按列的线性规划 (columnwise linear program)，用于解决该问题。
