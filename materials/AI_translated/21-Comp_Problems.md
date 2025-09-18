# 互补问题 (Complementarity Problems)

许多物理和经济现象最自然的建模方式是通过说明某些不等式约束必须是互补的，即至少其中一个必须以等式形式成立。原则上，这些条件可以伴随一个目标函数，但更常见的是用于构建寻找可行解的互补问题。实际上，优化可以被看作是互补性的一种特殊情况，因为线性和平滑非线性优化的标准最优性条件本身就是互补问题。然而，其他类型的互补问题并非源自优化，或者无法方便地作为优化问题进行建模或求解。

AMPL 运算符 `complements` 允许在约束声明中直接指定互补条件。因此，可以以一种自然的方式构建互补模型，并且此类模型的实例可以轻松发送给专门用于互补问题的求解器。

为了说明 `complements` 的语法，我们首先描述如何使用它来建模一些简单的经济均衡问题，其中一些等价于线性规划，而另一些则不然。然后，我们给出 `complements` 运算符的一般定义，适用于不等式对以及通过双重不等式的更一般的“混合”互补条件。在这些部分适当的地方，我们还评论了 AMPL 与用于“平方”混合互补问题的 PATH 求解器的接口。在最后一节中，我们描述了互补约束如何适应 AMPL 的几个现有功能，包括预处理、约束名称后缀和约束的通用同义词。

# 19.1 互补性的来源

经济均衡是互补条件最著名的应用之一。在本节中，我们首先展示之前生产经济学中的线性规划示例如何具有等价的互补模型形式，以及如何

```
set PROD; # 产品集合
set ACT; # 活动集合
param cost {ACT} > 0; # 每种活动的单位成本
param demand {PROD} >= 0; # 每种产品的需求量
param io {PROD,ACT} >= 0; # 每种活动产生每种产品的单位数量
var Level {j in ACT} >= 0;
minimize Total Cost: sum {j in ACT} cost[j] * Level[j];
subject to Demand {i in PROD}: sum {j in ACT} io[i,j] * Level[j] >= demand[i];
```

有界变量通过互补概念的扩展来处理。然后我们描述了对价格依赖需求的进一步扩展，这并非由优化驱动或等价于任何线性规划。最后我们简要描述了其他互补模型和应用。

# 生产经济学的互补模型

在 2.4 节中，我们观察到饮食模型的形式同样适用于生产经济学模型。此时决策变量可以视为生产活动的水平，目标函数即为总生产成本，最小化总成本：
$$\min \sum_{j \in ACT} cost[j] \times Level[j]$$
其中 $cost[j]$ 和 $Level[j]$ 分别表示活动 $j$ 的单位成本和活动水平。约束条件表明，各产品的总产出必须至少满足产品的需求量：

$$\text{subject to } Demand\ \{i \in PROD\}: \sum_{j \in ACT} io[i,j] \times Level[j] \geq demand[i];$$
其中 $io[i,j]$ 表示每单位活动 $j$ 所生产的产品 $i$ 的数量，$demand[i]$ 表示产品 $i$ 的总需求量。图 19-1 和图 19-2 展示了这一“经济”模型及其部分数据。

通过线性规划 求解器 可以轻松计算出最低 成本 的 生产 水平 ：

```ampl: model econmin.mod; ampl: data econ.dat; ampl: solve; CPLEX 8.0.0: optimal solution; objective 6808640.553; 3 dual simplex iterations (0 in phase I)```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/c2813c517192ab059d92407a78fc8fa72c406becd33da1ee6836a9cedb2c682d.jpg)  
图 19-2：生产 模型 的 数据 (econ.dat)。

ampl: display Level; Level [\*]:= P1 0 Pla 1555.3 P2 0 P2a 0 P2b 0 P3 147.465 P3c 1889.4 P4 0

回想 一下 （ 见 第 12.5 节 ） ， 每个 约束 还 对应 有 对偶 值 或 边际 值 —— 即 “ 价格 ” ：

ampl: display Demand.dual; Demand.dual [\*]:= AA1 16.7051 AC1 5.44585 BC1 57.818 BC2 0 NA2 0 NA3 0

在 线性 规划 的 传统 解释 中 ， 在 足够 小 的 范围 内 ， 约束 \(i\) 对应 的 价格 表示 产品 \(i\) 的 需求 每 增加 一 个 单位 时 总 成本 的 变化 量 。

现在 考虑 生产 经济学 问题 的 另 一 种 视角 ， 其中 我们 定义 变量 \(Price[i]\) 以及 \(Level[j]\) ， 并 寻求 一 种 均衡 解 而 非 最优 解 。 均衡 解 必须 满足 两 个 条件 。

首先 ， 对于 每 种 产品 ， 总 产出 必须 满足 需求 且 价格 必须 非负 ， 此外 这 两 个 关系 之间 必须 满足 互补 性 ： 当 产量 超过 需求 时 价格 必须 为 零 ； 或者 等价 地 ， 当 价格 为 正 时 产量 必须 等于 需求 。 这种 关系 在 AMPL 中 通过 互补 算子 （ complements operator ） 表示 ：

$$\text{subject to } Pri\_Compl\ \{i \in PROD\}: Price[i] \geq 0\quad \text{complements}\quad \sum_{j \in ACT} io[i,j] \times Level[j] \geq demand[i];$$

当 两 个 不等式 通过 complements 连接 时 ， 它们 都 必须 成立 ， 且 至少 有 一 个 必须 取 等号 。 由于 我们 的 示例 是 基于 集合 PROD 索引 的 ， 因此 它 为 每 种 产品 都 建立 了 一 个 这样 的 关系 。

其次 ， 对于 每 项 活动 ， 还 存在 另 一 种 可能 最初 不太 明显 的 关系 。 考虑 对于 每 单位 活动 j ， 所 产生 的 产品 i 在 模型 价格 下 的 价值 为 Price[i] * io[i,j] 。 因此 ， 每 单位 活动 j 所有 产出 的 总 价值 为 在 均衡 价格 下 ， 该 总 价值 不能 超过 该 活动 的 单位 成本 cost[j] 。 此外 ， 这种 关系 与 活动 j 的 水平 之间 存在 互补 性 ： 当 成本 超过 总 价值 时 ， 该 活动 必须 为 零 ； 或者 等价 地 ， 当 活动 为 正值 时 ， 总 价值 必须 等于 成本 。 同样 ， 这种 关系 可以 用 AMPL 中 的 互补 算子 表示 ：

subject to Lev_Compl {j in ACT}: Level[j] >= 0 complements sum {i in PROD} Price[i] * io[i,j] <= cost[j];

此处 约束 以 ACT 为 索引 ， 因此 我们 对 每 项 活动 都 有 一 个 互补 关系 。

将 这 两 组 互补 约束 结合 起来 ， 我们 得到 如 图 19-3 所示 的 线性 互补 问题 。 变量 的 数量 与 互补 关系 的 数量 相等 （ 等于 活动 数 加 产品 数 ） ， 使 其 成为 一 个 “ 方形 ” 互补 问题 ， 适用 于 某些 求解 技术 ， 尽管 这些 技术 不同 于 线性 规划 所用 的 技术 。

例如 ， 应用 PATH 求解 器 ， 可以 看出 该 互补 问题 与 相关 的 最小 成本 生产 问题 具有 相同 的 解 ：

集合 PROD; # 产品  
集合 ACT; # 活动  
参数 cost {ACT} > 0; # 每项活动的单位成本  
参数 demand {PROD} >= 0; # 每种产品的需求量  
参数 io {PROD, ACT} >= 0; # 每单位活动产生的每种产品数量  
变量 Price {i in PROD};  
变量 Level {i in ACT};  
约束 Pri_Compl {i in PROD}: Price[i] >= 0 complements sum {j in ACT} io[i,j] * Level[j] >= demand[i];  
约束 Lev_Compl {j in ACT}: Level[j] >= 0 complements sum {i in PROD} Price[i] * io[i,j] <= cost[j];

ampl: model econ.mod;  
ampl: data econ.dat;  
ampl: option solver path;  
ampl: solve;  
Path v4.5: Solution found.  
7 iterations (0 for crash); 33 pivots.  
20 function, 8 gradient evaluations.

ampl: display sum {j in ACT} cost[j] * Level[j];  
sum{j in ACT} cost[j]*Level[j] = 6808640

进一步使用 display 命令显示，Level 与生产经济学 LP 中的值相同，且 Price 与 LP 中 Demand.dual 的值相同。

# 有界变量的互补性

现在假设我们通过为活动水平设置边界来扩展模型：level_min[j] $ \leq $ Level[j] $ \leq $ level_max[j]。只要将活动的互补关系推广到“混合”形式，优化问题与平方互补问题之间的等价性仍然可以保持。当某项活动的单位成本大于其总价值时，该项活动的水平必须处于其下界（与之前类似）。当某项活动的水平处于其上下界之间时，其成本必须等于其总价值。此外，当某项活动的水平处于其上界时，其成本也可以小于其总价值。这三种关系通过互补算子的另一种形式进行总结：

约束 Lev_Compl {j in ACT}: level_min[j] $ \leq $ Level[j] $ \leq $ level_max[j] complements cost[j] - sum {i in PROD} Price[i] * io[i,j];

集合 PROD; # 产品集合  
集合 ACT; # 活动集合  
参数 cost {ACT} > 0; # 每项活动的单位成本  
参数 demand {PROD} >= 0; # 每种产品的需求量  
参数 io {PROD, ACT} >= 0; # 每项活动产生每种产品的单位数量  
参数 level_min {ACT} > 0; # 每项活动的最小允许水平  
参数 level_max {ACT} > 0; # 每项活动的最大允许水平  
变量 Price {i in PROD};  
变量 Level {j in ACT};  
约束 Pri_Compl {i in PROD}: Price[i] $ \geq $ 0 complements sum {j in ACT} io[i,j] * Level[j] $ \geq $ demand[i];  
约束 Lev_Compl {j in ACT}: level_min[j] $ \leq $ Level[j] $ \leq $ level_max[j] complements cost[j] - sum {i in PROD} Price[i] * io[i,j];

图 19-4：生产均衡模型的有界版本 (econ2.mod)。

当一个双重不等式通过 complements 与一个表达式相连时，不等式必须成立，并且要么该表达式为零，要么下界不等式取等号且表达式非负，要么上界不等式取等号且表达式非正。

图 19-4 展示了我们互补性示例的一个有界版本。PATH 求解器也可以应用于该模型：

```ampl
model econ2.mod;
data econ2.dat;
option solver path;
solve;
```

```ampl
display level_min, Level, level_max;
```

结果与通过在变量中添加上述边界从我们之前的示例（图 19-1）推导出的线性规划 (linear program, LP) 相同。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/f8ddbfe33d58f076749bcd086c4bff7af2914f6537b96cac86b0c627b584c80e.jpg)  
图 19-5：价格依赖型需求 (econnl.mod)。

# 价格依赖型需求的互补性

如果互补问题仅仅来源于线性规划 (linear program, LP)，那么它们将只有非常有限的应用价值。然而，经济均衡 (economic equilibrium) 的概念可以推广到那些没有 LP 等价形式的问题中。例如，与其将需求量 (demand) 视为固定的，更合理的方式是将每种产品的需求量视为其价格的递减函数。

最简单的情形是线性递减需求函数，它在 AMPL 中可以表示为：

```
demzero[i] - demrate[i] * Price[i]
```

其中 `demzero[i]` 和 `demrate[i]` 是非负参数。由此产生的互补问题只需将该表达式代入 `demand[i]`，如图 19-5 所示。该互补问题仍然是方阵形式，因此仍可由 PATH 求解器 (solver) 求解，但结果显然不同：

```
ampl: model econnl.mod;
ampl: data econnl.dat;
ampl: option solver path;
ampl: solve;
```

```
Path v4.5: Solution found.
11 iterations (3 for crash); 11 pivots.
12 function, 12 gradient evaluations.
```

```
ampl: display Level;
```

现在，需求量和价格之间的平衡倾向于降低均衡生产水平。

由于在该模型中变量 `Price[i]` 出现在互补算子 (complements operator) 的两边，因此不存在等价的线性规划问题。虽然存在一个等价的非线性优化模型，但该模型不易推导，且求解难度也可能更大。

# 其他互补模型与应用

这一基本示例可以扩展为更复杂的经济均衡模型。活动变量与价格变量及其对应的互补约束 (complementarity constraint) 可以由多个索引集合组成，成本函数与价格函数也可以是非线性的。只要问题在变量数量与互补约束数量上保持相等（或可容易地转换为这种形式，如下一节所述），像 PATH 这样的求解器便能处理所有这些扩展。

更复杂的模型可能会加入目标函数，并以任意比例混合等式、不等式和互补约束。这类所谓 MPEC（mathematical programs with equilibrium constraints，带均衡约束的数学规划）问题的求解技术目前仍处于相对实验阶段。

互补问题也出现在物理系统中，可用于建模力之间的平衡条件。例如，一个 互补约束 (complementarity constraint) 可以表示两个物体之间关系的离散化形式。在 互补算子 (complements operator) 一侧的关系在物体接触点处以等式成立，而在另一侧的关系则在物体未接触处以等式成立。

博弈论提供了另一类例子。双矩阵博弈 (bimatrix game) 的 纳什均衡 (Nash equilibrium) 由互补条件刻画，例如，其中 变量 (variable) 是两个玩家做出其可选行动的概率。对于每个行动，要么其概率为零，要么相关的 等式约束 (equality constraint) 成立，以确保通过增加或减少其概率无法获得任何收益。

本章末尾的参考文献中引用了一些详细描述各种互补问题的综述。

# 19.2 互补约束的形式

一个 AMPL 互补约束 (complementarity constraint) 由两个表达式或 约束 (constraint) 通过 互补算子 (complements operator) 分隔组成。总是存在两个不等式，它们的位置决定了约束如何被解释。

如果在 互补算子 (complements operator) 两侧各有一个不等式，则约束具有一般形式：单不等式 complements 单不等式；其中单不等式是任何有效的普通 约束 (constraint) —— 线性或非线性 —— 包含一个 $\geq$ 或 $\leq$ 算子。当两个单不等式关系都满足，且至少有一个以等式满足时，这种类型的 约束 (constraint) 即被满足。

如果两个不等式都在 互补算子 (complements operator) 的同一侧，则约束具有以下形式之一：双不等式 complements 表达式；表达式 complements 双不等式；其中双不等式是任何包含两个 $\geq$ 或两个 $\leq$ 算子的普通 AMPL 约束 (constraint)，表达式是任何数值表达式。变量 (variable) 可以非线性地出现在双不等式或表达式（或两者）中。这种类型 约束 (constraint) 被满足的条件如下：

如果双不等式的左侧 $\leq$ 或右侧 $\geq$ 以等式成立，则表达式大于或等于 $0$；如果双不等式的右侧 $\leq$ 或左侧 $\geq$ 以等式成立，则表达式小于或等于 $0$；如果双不等式的两边都不以等式成立，则表达式等于 $0$。

在双不等式形如 $0 \leq body \leq \infty$ 的特殊情况下，这些条件简化为一对单不等式的互补条件。

为了完整性，当双不等式左侧等于右侧的特殊情况可以用以下形式之一书写：等式 complements 表达式；表达式 complements 等式；

这种类型的 约束 (constraint) 等价于仅由等式组成的普通 约束 (constraint)；它对表达式不施加任何限制。

对于需要“平方”互补系统的 求解器 (solver)，当模型实例中 变量 (variable) 的数量等于 互补约束 (complementarity constraint) 的数量加上 等式约束 (equality constraint) 的数量时，AMPL 会将其转换为平方形式。可以有任意数量的附加不等式 约束 (constraint)，但不能有任何目标函数。每个等式都如前所述被简单地转化为互补条件；每个新增的不等式都与一个新的、除此之外未使用的 变量 (variable) 形成 complements 关系，从而保持问题整体的平方性。

# 19.3 处理互补约束

AMPL 中所有用于普通等式和不等式的功能都可以直接扩展到互补约束 (complementarity constraints)。本节涵盖三个方面的扩展：相关解值的表达式、预处理 (presolve) 及其相关问题统计信息显示 (display) 的影响，以及约束的通用同义词。

# 相关解值

AMPL 内置的与问题及其解相关的值的后缀 (suffix) 也扩展到了互补约束，但包含两组后缀 —— 形如 cname.Lsuf 和 cname.Rsuf —— 分别对应互补运算的左操作数和右操作数。例如，在求解 econ2.mod（图 19-4）之后，我们可以使用以下 display 命令查看与约束 Lev_Compl 相关的值：

ampl: display Lev_Compl.Llb, Lev_Compl.Lbody, Lev_Compl.Rbody, Lev_Compl.Rslack;

<table><tr><td>Lev_Compl.Llb</td><td>Lev_Compl.Lbody</td><td>Lev_Compl.Rbody</td><td>Lev_Compl.Rslack</td></tr><tr><td>P1</td><td>240</td><td>240</td><td>1392.86</td></tr><tr><td>P1a</td><td>270</td><td>1000</td><td>-824.286</td></tr><tr><td>P2</td><td>220</td><td>220</td><td>264.286</td></tr><tr><td>P2a</td><td>260</td><td>680</td><td>5.00222e-12</td></tr><tr><td>P2b</td><td>200</td><td>200</td><td>564.286</td></tr><tr><td>P3</td><td>260</td><td>260</td><td>614.286</td></tr><tr><td>P3c</td><td>220</td><td>1000</td><td>-801.429</td></tr><tr><td>P4</td><td>240</td><td>240</td><td>584.286</td></tr><tr><td>;</td><td></td><td></td><td></td></tr></table>

由于 Lev_Compl 的右操作数是一个表达式，它被视为一个具有无限上下界的“约束”，因此其松弛值 (slack) 为无穷大。

对于互补约束，也定义了形如 cname.slack 的后缀。对于互补的单个不等式对，它等于 cname.Lslack 和 cname.Rslack 中的较小者。因此，当且仅当两个不等式均满足时，该值为非负数；当互补约束严格成立时，该值为零。对于如下形式的互补双重不等式：

expr complements lbound \( \leq \) body \( \leq \) ubound  
lbound \( \leq \) body \( \leq \) ubound complements expr

cname.slack 被定义为：

\[
\begin{cases}
\min(\text{expr}, \text{body} - \text{lbound}) & \text{if } \text{body} \leq \text{lbound} \\
\min(-\text{expr}, \text{ubound} - \text{body}) & \text{if } \text{body} \geq \text{ubound} \\
-\text{abs}(\text{expr}) & \text{otherwise}
\end{cases}
\]

因此，在这种情况下，它总是非正的，并且当互补约束恰好满足时，其值为零。

如果一个互补约束 cname 在表达式中以无后缀形式出现，则将其解释为代表 cname.slack。

# 预处理

如第 14.1 节所述，AMPL 包含一个预处理阶段，可以显著简化某些线性规划问题。在存在互补约束的情况下，几种新的简化方式成为可能。

例如，给定形式如下的约束：

$$
expr_{1} \geq 0 \text{ complement } expr_{2} \geq 0
$$

如果预处理可以推断出对于所有可行点 $expr_{1}$ 严格为正——换句话说，它具有一个正的下界 (lower bound)——则可以用 $expr_{2} = 0$ 替换该约束。

类似地，对于形式如下的 约束 (constraint)：

lbound $\leq$ body $\leq$ ubound complements expr，有多种可能性，包括以下几种：

如果 预处理 (presolve) 可以推断出对于所有可行点，body $<$ ubound，则 lbound $\leq$ body complements expr $\geq 0$ 可替换为 lbound $<$ body $<$ ubound，expr $= 0$，expr $< 0$，body $=$ ubound。

除非使用 选项 (option) presolve 0 关闭 预处理 (presolve) 阶段，否则这类变换会自动执行。与普通 约束 (constraint) 一样，结果以原始模型的形式报告。

通过 显示 (display) 一些预定义参数：

_ncons 预处理前普通 约束 (constraint) 的数量  
_nccons 预处理前 互补性条件 (complementarity conditions) 的数量  
_sncons 预处理后普通 约束 (constraint) 的数量  
_snccons 预处理后 互补性条件 (complementarity conditions) 的数量  

或者通过设置 选项 (option) show_stats 1，可以获得一些关于 预处理 (presolve) 所做简化变换数量的信息：

ampl: model econ2.mod; data econ2.dat;  
ampl: option solver path;  
ampl: option show_stats 1;  
ampl: solve;

Presolve eliminates 16 constraints and 2 variables.  
Presolve resolves 2 of 14 complementarity conditions.  
Adjusted problem:  
12 variables, all linear  
12 constraints, all linear; 62 nonzeros  
12 complementarity conditions among the constraints:  
12 linear, 0 nonlinear.  
0 objectives.

Path v4.5: Solution found.  
7 iterations (1 for crash); 30 pivots.  
8 function, 8 gradient evaluations.

ampl: display _ncons, _nccons, _sncons, _snccons;  
_ncons = 28  
_nccons = 14  
_sncons = 12  
_snccons = 12

在首次实例化问题时，AMPL 将每个 互补约束 (complementarity constraint) 计为两个普通 约束 (constraint)（complements 的两个参数），同时也计为一个 互补性条件 (complementarity condition)。因此，_ncons 等于预处理前 互补约束 (complementarity constraint) 的数量，而 _ncons 等于 _nccons 的两倍加上预处理前任何非互补 约束 (constraint) 的数量。show_stats 输出开头的 预处理 (presolve) 信息表明 预处理 (presolve) 在多大程度上减少了这些数量。

在这种情况下，通过将每种 产品 (product) 的 需求量 (demand) 与其最低可能产出（即每个 Level[j] 被设为 level_min[j] 时的产出量）进行比较，可以发现减产的原因：

``` ampl
impl: display {i in PROD} 
      appl? (sum{j in ACT} io[i,j]*level_min[j], demand[i]);
      : sum{j in ACT} io[i,j]*level_min[j]   demand[i] =
AA1           69820                         70000
AC1           45800                         80000
BC1           37300                         90000
BC2           29700                         70000
NA2         1854920                       4e+05
NA3          843700                       8e+05
;
```

可以看到，对于 产品 (product) NA2 和 NA3，即使在最低 活动 (activity) 水平下，总产出也超过了 需求量 (demand)。因此，在如下 约束 (constraint) 中：

``` ampl
subject to Pri_Compl {i in PROD}:
   Price[i] >= 0 complements
   sum {j in ACT} io[i,j] * Level[j] >= demand[i];
```

对于 NA2 或 NA3，complements 的右侧参数永远不会以等式形式成立。因此，presolve 得出结论：Price["NA2"] 和 Price["NA3"] 可以固定为零，并将它们从后续问题中移除。

# 通用同义词

AMPL 为 约束 (constraints) 提供的通用同义词（第 12.6 节）也适用于 互补性条件 (complementarity conditions)，主要是通过在同义词名称中用 ccon 替代 con 来实现的。

从建模者的视角来看（即 presolve 之前），普通的 约束 (constraint) 同义词仍然包括：

- _ncons：presolve 之前的普通约束数量  
- _conname：presolve 之前的普通约束名称  
- _con：presolve 之前的普通约束同义词  

互补性约束的同义词则包括：

- _ncons：presolve 之前的互补性约束数量  
- _cconname：presolve 之前的互补性约束名称  
- _ccon：presolve 之前的互补性约束同义词  

由于每个互补性约束还会生成两个普通约束（如前文 presolve 讨论中所述），在 _conname 中对应每个 _cconname 条目都有两个条目：

``` ampl
ampl: display {i in 1..6} (_conname[i], _cconname[i]);
: _conname[i]              _cconname[i] :=
1 "Pri_Compl['AA1'].L"     "Pri_Compl['AA1']"
2 "Pri_Compl['AA1'].R"     "Pri_Compl['AC1']"
3 "Pri_Compl['AC1'].L"     "Pri_Compl['BC1']"
4 "Pri_Compl['AC1'].R"     "Pri_Compl['BC2']"
5 "Pri_Compl['BC1'].L"     "Pri_Compl['NA2']"
6 "Pri_Compl['BC1'].R"     "Pri_Compl['NA3']"
;
```

对于每个互补性约束 cname，complements 运算符的左侧和右侧参数分别是名为 cname.L 和 cname.R 的普通约束。这可以通过使用同义词术语展开示例中的互补性约束 Pri_Compl['AA1'] 及其对应的两个普通约束来验证：

ampl: expand Pri_Compl['AA1'];
subject to Pri_Compl['AA1']:
	Price['AA1'] >= 0 complements
	60*Level['P1'] + 8*Level['Pla'] + 8*Level['P2'] + 40*Level['P2a'] + 15*Level['P2b'] + 70*Level['P3'] + 25*Level['P3c'] + 60*Level['P4'] >= 70000;

ampl: expand _con[1], _con[2];
subject to Pri_Compl.L['AA1']:
	Price['AA1'] >= 0;
subject to Pri_Compl.R['AA1']:
	60*Level['P1'] + 8*Level['Pla'] + 8*Level['P2'] + 40*Level['P2a'] + 15*Level['P2b'] + 70*Level['P3'] + 25*Level['P3c'] + 60*Level['P4'] >= 70000;

从 求解器 (solver) 的 视角 (经过 presolve 之后)，定义 了 一个 更加 有限 的 同义词 集合：

- _sncons：presolve 之后 所有 约束 的 数量  
- _snccons：presolve 之后 互补 约束 的 数量  
- _sconname：presolve 之后 所有 约束 的 名称  
- _scon：presolve 之后 所有 约束 的 同义词  

必然地，_snccons 小于 或 等于 _sncons，仅当 所有 约束 均为 互补 约束 时 才 相等。

为 简化 发送给 求解器 的 问题 描述，AMPL 将 每个 互补 约束 转换 为 以下 规范 形式 之一：

expr complements lbound $\leq$ var $\leq$ ubound  
expr $\leq 0$ complements var $\leq$ ubound  
expr $\geq 0$ complements lbound $\leq$ var

其中 `var` 是每个约束对应的不同的变量名称。(当 `complements` 两侧出现比单个变量更复杂的表达式时，这涉及引入一个辅助变量以及一个等式约束来定义该变量等于其中一个表达式。) 通过使用 `solexpand` 替代 `expand`，可以查看 AMPL 发送给求解器的互补约束形式：

```ampl
ampl: solexpand Pri_Compl['AA1'];
subject to Pri_Compl['AA1'];
	- 70000 + 60*Level['P1'] + 8*Level['Pla'] + 8*Level['P2'] + 40*Level['P2a'] + 15*Level['P2b'] + 70*Level['P3'] + 25*Level['P3c'] + 60*Level['P4'] >= 0 complements 0 <= Price['AA1'];
```

一个预定义的整数数组 `_scvar` 给出了互补变量在通用变量数组 `_var` 和 `_varname` 中的索引。此术语可用于显示这些变量的名称列表：

```ampl
ampl: display {i in 1..3} (_sconname[i], _svarname[_scvar[i]]);
:	_sconname[i]	_svarname[_scvar[i]] :=
1	"Pri_Compl['AA1'].R"	"Price['AA1']"
2	"Pri_Compl['AC1'].R"	"Price['AC1']"
3	"Pri_Compl['BC1'].R"	"Price['BC1']"
;
```

当约束 `i` 是一个普通的等式或不等式时，`_scvar[i]` 为 0。`_sconname` 中互补约束的名称以 `.L` 或 `.R` 为后缀，具体取决于发送给求解器的约束中的 `expr` 是源自原始约束中 `complement` 的左参数还是右参数。

# 参考文献

[1] 理查德·W·科特尔, Jong- Shi Pang, 理查德·E·斯通. The Linear Complementarity Problem. 学术出版社 (圣地亚哥, CA, 1992). 关于线性互补问题的百科全书式账户，并对这些问题的产生方式进行优美的概述。

史蒂文·P·迪克斯 与 迈克尔·C·费里斯, "MCPLIB: 非线性混合互补问题集." 《优化方法与软件》 5, 4 (1995) 第 319- 345 页。一篇关于非线性互补问题的详尽综述，包括问题描述与数学公式。

迈克尔·C·费里斯 与 庞琼实, "互补问题的工程与经济应用." 《SIAM评论》 39, 4 (1997) 第 669- 713 页。多种互补问题测试案例，最初以 GAMS 建模语言编写，但目前许多已翻译为 AMPL。

# 练习

19- 1. 第 19.1 节中的经济学例子使用了一个关于价格的线性需求函数。构建一个具有以下各项特征的非线性需求函数。定义一个相应的互补问题，尽可能多地使用图 19- 2 中的数据。

使用 PATH 等求解器计算一个均衡解。将该解与第 19.1 节中展示的恒定需求和线性需求替代方案的解进行比较。

(a) 当价格 `i` 接近零时，需求接近 `demzero[i]` 并以接近 `demrate[i]` 的速率递减。然而，当价格 `i` 显著增加后，需求及其递减速率都趋近于零。

(b) 当价格 `i` 接近零时，需求近似恒定为 `demzero[i]`，但当价格 `i` 接近 `demlim[i]` 时，需求迅速降至零。

(c) 对于 `i` 的需求实际上随着价格上升而增加，直到在价格为 `demarg[i]` 时达到值 `demmax[i]`。然后需求随价格下降。

19- 2. 对于前一个问题中的每个情景，尝试不同的 Level 和 Price 值的起始点。判断是否存在唯一的均衡点。

19- 3. 玩家 A 与 B 之间的双矩阵博弈由两个 $m$ 行 $n$ 列的“收益”矩阵定义，其元素记为 $a_{ij}$ 和 $b_{ij}$。在博弈的一轮中，玩家 A 有 $m$ 个选择，玩家 B 有 $n$ 个选择。若 A 选择 $i$ 且 B 选择 $j$，则 A 与 B 分别赢得 $a_{ij}$ 与 $b_{ij}$；负的收益被解释为损失。

我们可以引入“混合”策略，其中 A 以概率 $p_i^A$ 选择 $i$，B 以概率 $p_j^B$ 选择 $j$。那么玩家 A 的期望收益为：

$$
\sum_{j = 1}^{n}a_{ij}\times p_{j}^{B},\mathrm{~若~A~选择~}i
$$

玩家 B 的期望收益为：

$$
\sum_{i = 1}^{m}b_{ij}\times p_{i}^{A},\mathrm{~若~B~选择~}j
$$

“纯”策略是每个玩家有一个概率为 1，其余为 0 的特殊情况。

若一对策略使得任何一方都无法仅通过改变自己的策略来提高期望收益，则称该对策略为纳什均衡 (Nash equilibrium)。

(a) 证明纳什均衡的要求等价于以下类似互补的条件：

对所有满足 $p_i^A > 0$ 的 $i$，当玩家 A 选择策略 $i$ 时的期望收益等于 A 在所有可能策略中的最大期望收益；  
对所有满足 $p_j^B > 0$ 的 $j$，当玩家 B 选择策略 $j$ 时的期望收益等于 B 在所有可能策略中的最大期望收益。

(b) 为了构建一个 AMPL 中的互补问题，其解为纳什均衡 (Nash equilibrium)，可以使用以下 param 声明来定义表示收益矩阵的参数：

```ampl
param nA > 0; # 玩家 A 可选的行动数  
param nB > 0; # 玩家 B 可选的行动数  
param payoffA {1..nA, 1..nB}; # 玩家 A 的收益  
param payoffB {1..nA, 1..nB}; # 玩家 B 的收益
```

定义混合策略的概率必须是变量。此外，为每个玩家的最大期望收益定义一个变量也很方便：

```ampl
var PlayA {i in 1..nA}; # 玩家 A 的混合策略  
var PlayB {j in 1..nB}; # 玩家 B 的混合策略  
var MaxExpA; # 玩家 A 的最大期望收益  
var MaxExpB; # 玩家 B 的最大期望收益
```

为以下约束编写 AMPL 声明：

- 混合策略中的概率必须非负。  
- 每个玩家的混合策略中的概率之和必须为 1。  
- 玩家 A 在选择任意特定策略 $i$ 时的期望收益不得超过 A 的最大期望收益。  
- 玩家 B 在选择任意特定策略 $j$ 时的期望收益不得超过 B 的最大期望收益。

(c) 编写一个 AMPL 模型，用于表示一个平方互补系统，该系统满足 (b) 中的约束和 (a) 中的条件。

(d) 通过将其应用于“石头剪刀布 (rock-scissors-paper)”游戏来测试你的模型，该游戏的两个玩家的收益矩阵为：

$$
\begin{array}{ccc}
0 & 1 & -1 \\
-1 & 0 & 1 \\
1 & -1 & 0
\end{array}
$$

确认找到的均衡是每个玩家以相等概率在三种策略中进行选择。

(e) 证明对于两个玩家都具有如下收益矩阵的游戏：

$$
\begin{array}{cccc}
-3 & 1 & 3 & -1 \\
2 & 3 & -1 & -5
\end{array}
$$

存在多个均衡点，其中至少有一个使用混合策略，另一个使用纯策略。

运行诸如 PATH 求解器之类的求解器只会返回一个均衡解。为了找到更多解，可以尝试更改初始解或将某些变量固定为 0 或 1。

19-4. 两家公司现在必须决定未来在其产品中采用标准 1 (standard 1) 还是标准 2 (standard 2)。如果他们选择了相同的标准，由于公司 A 的技术更优越，因此公司 A 的收益更高。如果他们选择了不同的标准，由于公司 B 的市场份额更大，因此公司 B 的收益更高。这些因素导致了一个双矩阵博弈，其收益矩阵为：

$$
\begin{array}{cc}
{\tt A} = \begin{array}{cc}
10 & 3 \\
2 & 9
\end{array} &
{\tt B} = \begin{array}{cc}
4 & 6 \\
7 & 5
\end{array}
\end{array}
$$

(a) 使用 PATH 等求解器寻找纳什均衡。验证它是一个混合策略，其中 A 选择各标准的概率均为 1/2，而 B 选择标准 1 和标准 2 的概率分别为 3/7 和 4/7。

为什么在此应用中混合策略并不合适？

(b) 当公司 A 决定选择标准 1 时，你可以通过以下命令查看结果：

```rl
ampl: fix PlayA[1] := 1;
ampl: solve;
presolve, constraint ComplA[1].L:    all variables eliminated, but upper bound = - 1 < 0
```

解释 AMPL 的预处理 (presolve) 阶段是如何推断出此时该互补问题无可行解的。

(c) 通过进一步实验，证明在该情形下不存在仅包含纯策略的纳什均衡 (Nash equilibrium)。