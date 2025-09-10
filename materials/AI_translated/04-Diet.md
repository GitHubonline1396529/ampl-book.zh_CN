# 2

# 饮食及其他输入模型：最小化成本

为了补充第 1 章中以利润最大化为目标的模型，我们现在考虑以成本最小化 (minimize costs) 为目标的线性规划 (linear program) 模型。最大化模型的约束 (constraints) 通常是资源可用性的上限，而最小化模型的约束则更可能是解中某些“品质”的下限。

作为成本最小化模型的一个直观示例，本章使用了著名的“饮食问题 (diet problem)”，该问题寻找一种食物组合，以满足对各种维生素含量的要求。我们将再次构建一个小型、明确的线性规划，并随后展示如何为该类所有线性规划问题制定一个通用模型。然而，由于您现在已经更加熟悉 AMPL，我们将花更多时间在 AMPL 上，而减少使用代数符号。

在制定饮食模型后，我们将讨论一些可能使其更加现实的修改。然而，该模型的全部功能来源于其广泛的应用性，许多与饮食无关的情形也可以应用。因此，我们以一种更通用的方式重写该模型作为本章的结尾，并讨论其在混合 (blending)、经济学和调度 (scheduling) 中的应用。

# 2.1 饮食问题的线性规划

考虑选择预制食品以满足特定营养需求的问题。假设以下类型的预煮晚餐可以按以下每包价格购买：

BEEF 牛肉 \$3.19 CHK 鸡肉 \$2.59 FISH 鱼 \$2.29 HAM 火腿 \$2.89 MCH 通心粉和奶酪 \$1.89 MTL 肉饼 \$1.99 SPG 意大利面 \$1.99 TUR 火鸡 \$2.49

这些晚餐每包提供以下维生素 A、C、B1 和 B2 的最低每日需求百分比：

<table><tr><td></td><td>A</td><td>C</td><td>B1</td><td>B2</td></tr><tr><td>BEEF</td><td>60%</td><td>20%</td><td>10%</td><td>15%</td></tr><tr><td>CHK</td><td>8</td><td>0</td><td>20</td><td>20</td></tr><tr><td>FISH</td><td>8</td><td>10</td><td>15</td><td>10</td></tr><tr><td>HAM</td><td>40</td><td>40</td><td>35</td><td>10</td></tr><tr><td>MCH</td><td>15</td><td>35</td><td>15</td><td>15</td></tr><tr><td>MTL</td><td>70</td><td>30</td><td>15</td><td>15</td></tr><tr><td>SPG</td><td>25</td><td>50</td><td>25</td><td>15</td></tr><tr><td>TUR</td><td>60</td><td>20</td><td>15</td><td>10</td></tr></table>

问题是找到满足一周需求（即每种营养素至少达到每日需求的 $700\%$）的最便宜的包装组合。

我们用 $X_{BEEF}$ 表示要购买的牛肉晚餐包数，用 $X_{CHK}$ 表示鸡肉晚餐包数，依此类推。那么饮食的总成本将是：

总成本 $=$ $\begin{array}{rl} & 3.19X_{BEEF} + 2.59X_{CHK} + 2.29X_{FISH} + 2.89X_{HAM} + \\ & 1.89X_{MCH} + 1.99X_{MTL} + 1.99X_{SPG} + 2.49X_{TUR} \end{array}$

维生素 A 需求总量的百分比由一个类似的公式给出，不同之处在于 $X_{BEEF}、X_{CHK}$ 等变量是乘以每包的百分比，而不是每包的成本：

满足维生素 A 日需求总量的百分比 $=$ $\begin{array}{rl} & 60X_{BEEF} + 8X_{CHK} + 8X_{FISH} + 40X_{HAM} + \\ & 15X_{MCH} + 70X_{MTL} + 25X_{SPG} + 60X_{TUR} \end{array}$

这个数值需要大于或等于 700 百分比。对于其他每种维生素，都有类似的公式，并且每种维生素的数值也都需要 $\geq 700$

将这些全部组合在一起，我们得到以下线性规划 (linear program)：

最小化 (Minimize)

$$
\begin{array}{rl} & 3.19X_{BEEF} + 2.59X_{CHK} + 2.29X_{FISH} + 2.89X_{HAM} + \\ & 1.89X_{MCH} + 1.99X_{MTL} + 1.99X_{SPG} + 2.49X_{TUR} \end{array}
$$

约束条件 (Subject to)

$$
\begin{array}{rl} & 60X_{BEEF} + 8X_{CHK} + 8X_{FISH} + 40X_{HAM} + \\ & 15X_{MCH} + 70X_{MTL} + 25X_{SPG} + 60X_{TUR}\geq 700 \end{array}
$$

$$
\begin{array}{rl} & 20X_{BEEF} + 0X_{CHK} + 10X_{FISH} + 40X_{HAM} + \\ & 35X_{MCH} + 30X_{MTL} + 50X_{SPG} + 20X_{TUR}\geq 700 \end{array}
$$

$$
\begin{array}{rl} & 10X_{BEEF} + 20X_{CHK} + 15X_{FISH} + 35X_{HAM} + \\ & 15X_{MCH} + 15X_{MTL} + 25X_{SPG} + 15X_{TUR}\geq 700 \end{array}
$$

$$
\begin{array}{rl} & 15X_{BEEF} + 20X_{CHK} + 10X_{FISH} + 10X_{HAM} + \\ & 15X_{MCH} + 15X_{MTL} + 15X_{SPG} + 10X_{TUR}\geq 700 \end{array}
$$

$$
\begin{array}{rl} & X_{BEEF}\geq 0,X_{CHK}\geq 0,X_{FISH}\geq 0,X_{HAM}\geq 0,\\ & X_{MCH}\geq 0,X_{MTL}\geq 0,X_{SPG}\geq 0,X_{TUR}\geq 0 \end{array}
$$

最后我们添加了一个符合常识的要求，即购买的任何食物包数不得少于零。

正如我们在第 1 章中首次处理生产 LP 时所做的那样，我们可以将显式的饮食 LP 的 AMPL 语句转录到一个文件中，例如 diet0.mod：

```ampl
var Xbeef >= 0;
var Xchk >= 0;
var Xfish >= 0;
var Xham >= 0;
var Xmch >= 0;
var Xmtl >= 0;
var Xspg >= 0;
var Xtur >= 0;

minimize cost:
    3.19*Xbeef + 2.59*Xchk + 2.29*Xfish + 2.89*Xham +
    1.89*Xmch + 1.99*Xmtl + 1.99*Xspg + 2.49*Xtur;

subject to A:
    60*Xbeef + 8*Xchk + 8*Xfish + 40*Xham + 15*Xmch +
    70*Xmtl + 25*Xspg + 60*Xtur >= 700;

subject to C:
    20*Xbeef + 0*Xchk + 10*Xfish + 40*Xham + 35*Xmch +
    30*Xmtl + 50*Xspg + 20*Xtur >= 700;

subject to B1:
    10*Xbeef + 20*Xchk + 15*Xfish + 35*Xham + 15*Xmch +
    15*Xmtl + 25*Xspg + 15*Xtur >= 700;

subject to B2:
    15*Xbeef + 20*Xchk + 10*Xfish + 10*Xham + 15*Xmch +
    15*Xmtl + 15*Xspg + 10*Xtur >= 700;
```

同样，只需几个 AMPL 命令即可读取该文件，将 LP 发送给求解器 (solver)，并检索结果：

```ampl
ampl: model diet0.mod;
ampl: solve;
MINOS 5.5: optimal solution found.
6 iterations, objective 88.2
ampl: display Xbeef,Xchk,Xfish,Xham,Xmch,Xmtl,Xspg,Xtur;
Xbeef = 0
Xchk = 0
Xfish = 0
Xham = 0
Xmch = 46.6667
Xmtl = -3.69159e-18
Xspg = -4.05347e-16
Xtur = 0
```

很快便找到了最优解，但这 hardly 是我们所希望的结果。通过单调地食用 $46\frac{2}{3}$ 包通心粉和奶酪（macaroni and cheese）将成本降到最低！你可以验证，这恰好提供了维生素 A、B1 和 B2 所需量的 $15\% \times 46\frac{2}{3} = 700\%$，以及远超所需的维生素 C；成本仅为 $\$1.89 \times 46\frac{2}{3} = \$88.20$。（肉饼和意大利面的微小负值可以视为零，就像我们在第 1.6 节中看到的微小正值一样。）

你可能会猜测，通过要求每种维生素的量恰好等于 $700\%$ 可以生成一个更好的解。这种要求可以通过在 AMPL 的约束中将每个 $\geq$ 改为 $=$ 来轻松实现。如果你继续求解这个修改后的线性规划（LP），你会发现饮食确实变得更加多样化：大约 19.5 包鸡肉、16.3 包通心粉和奶酪，以及 4.3 包肉饼。但由于等式约束比不等式约束更严格，成本上升到了 $\$89.99$。

# 2.2 饮食问题的 AMPL 模型

显然，为了产生一个哪怕 remotely 可接受的饮食方案，我们必须对我们的线性规划进行更广泛的修改。我们可能需要更改食物和营养素的集合，以及约束和界限的性质。就像上一章的生产示例一样，如果我们依赖一个可以与各种具体数据文件结合的通用模型，这将容易得多。

该模型处理两类事物：营养素（nutrients）和食物（foods）。因此，我们通过声明两个集合来开始 AMPL 模型：

```ampl
set NUTR;
set FOOD;
```

接下来，我们需要指定模型所需的数值。当然，每种食物都应有一个正的运输成本 (cost)：

$$
\mathtt{param}\ \mathtt{cost}\{j \in \mathtt{FOOD}\} > 0;
$$

我们还指定，对于每种食物，在饮食中包含的包数都有下限 (f_min) 和上限 (f_max)：

```ampl
param f_min {FOOD} >= 0;
param f_max {j in FOOD} >= f_min[j];
```

注意，在声明 `f_max` 时，我们需要一个虚拟索引 $j$ 来遍历 `FOOD`，以便说明每种食物的最大值必须大于或等于相应的最小值。

为了使该模型比我们目前的示例更具一般性，我们还为饮食中每种营养成分 (NUTR) 的数量指定了类似的下限 (n_min) 和上限 (n_max)：

```ampl
param n_min {NUTR} >= 0;
param n_max {i in NUTR} >= n_min[i];
```

最后，对于每一种营养成分和食物的组合，我们需要一个数字来表示一包食物中所含的营养成分量。你可能还记得第 1 章中提到，两个集合的这种“乘积”是通过将它们都列出来表示的：

```ampl
param amt {NUTR,FOOD} >= 0;
```

引用这个参数需要两个索引。例如，`amt[i,j]` 表示一包食物 $j$ 中所含的营养成分 $i$ 的量。

该模型的决策变量是购买不同食物的包数 (Buy)：

$$
\mathtt{var~Buy~\{j~in~FOOD\}~>=~f\_min[j],~<=~f\_max[j];}
$$

某种食物 $j$ 的购买包数将被称为 `Buy[j]`；在任何可接受的解中，它必须介于 `f_min[j]` 和 `f_max[j]` 之间。

购买食物 $j$ 的总成本 (Total_Cost) 等于每包的成本 `cost[j]` 乘以包数 `Buy[j]`。需要最小化的目标函数是所有食物 $j$ 的这一乘积之和：

```ampl
minimize Total_Cost: sum {j in FOOD} cost[j] * Buy[j];
```

这个 minimize 声明的作用与第 1 章中的 maximize 相同。

类似地，食物 $j$ 提供的营养成分 $i$ 的量等于每包中的营养成分量 `amt[i,j]` 乘以包数 `Buy[j]`。营养成分 $i$ 的总供应量是所有食物 $j$ 的这一乘积之和：

$$
\mathtt{sum}\{\mathtt{j}\mathtt{in}\mathtt{FOOD}\} \mathtt{amt}[i,j] \star \mathtt{Buy}[j]
$$

为了完成模型，我们只需指定每个这样的和必须介于适当的上下限之间。我们的约束声明以

```ampl
subject to Diet {i in NUTR}:
```

开头，表示对于 NUTR 中的每个成员 $i$，必须施加一个名为 `Diet[i]` 的约束。声明的其余部分给出了营养成分 $i$ 的约束的代数表达式：变量必须满足

```ampl
n_min[i] <= sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i]
```

```ampl
set NUTR;
set FOOD;
param cost {FOOD} > 0;
param f_min {FOOD} >= 0;
param f_max {j in FOOD} >= f_min[j];
param n_min {NUTR} >= 0;
param n_max {i in NUTR} >= n_min[i];
param amt {NUTR,FOOD} >= 0;
var Buy {j in FOOD} >= f_min[j], <= f_max[j];
minimize Total_Cost: sum {j in FOOD} cost[j] * Buy[j];
subject to Diet {i in NUTR}: n_min[i] <= sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i];
```

像这样的“双重不等式”按显而易见的方式解释：中间的和的值必须介于 `n_min[i]` 和 `n_max[i]` 之间。完整模型如图 2-1 所示。

# 2.3 使用 AMPL 饮食模型

通过指定适当的数据，我们可以求解与上述模型对应的任何线性规划。让我们首先使用本章开头的数据，这些数据以 AMPL 格式显示在图 2-2 中。

f_min 和 n_min 的值与原来给定的相同，而 f_max 和 n_max 暂时被设置为较大的值，以避免影响 最优解。在 amt 的表格中，符号 (tr) 表示我们已经“转置”了表格，使列对应第一个索引 (营养素)，行对应第二个索引 (食物)。或者，我们也可以修改模型，写成 param amt {FOOD,NUTR}，在这种情况下，我们必须在 约束中写成 amt[j,i]。

假设模型和数据分别存储在文件 diet.mod 和 diet.dat 中。然后使用 AMPL 读取这些文件并求解得到的 线性规划如下：

```
appl: model diet.mod;
appl: data diet.dat;
appl: solve;
MINDS 5.5: optimal solution found.
6 iterations, objective 88.2
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/0d2a102f9782db27c605b5b7fae74f9957950c58dd3446a6a56c2b2a8784e1fc.jpg)  
图 2-2：饮食模型的数据 (diet.dat)。

```
ampl: display Buy;
Buy [*]:= 
BEEF   0
CHK    0
FISH   0
HAM    0
MCH    46.6667
MTL   -1.07823e-16
SPG   -1.32893e-16
TUR    0
```

自然地，结果与之前相同。

现在假设我们想要进行以下改进。为了促进多样性，每周饮食中每种食物的包数必须在 2 到 10 包之间。每包中的钠和卡路里含量也已给出；总钠摄入量不得超过 40,000 毫克，总卡路里必须在 16,000 到 24,000 之间。所有这些更改都可以通过对数据进行少量修改来实现，如图 2-3 所示。将这些新数据放入文件 diet2.dat 中，我们可以再次运行 AMPL：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/410087a815243c71b1f1f5f9ddf36daa26ff97ebdaf20adbcacbe82f8a4e86f6.jpg)  
图 2-3：增强饮食模型的数据 (diet2.dat)。

```
ampl: model diet.mod;
ampl: data diet2.dat;
ampl: solve;
MINDS 5.5: infeasible problem.
9 iterations
```

消息 infeasible problem 告诉我们，我们对饮食的约束过于严格；无法满足所有限制条件。

AMPL 允许我们查看求解器在尝试找到解时产生的各种值。在第 1 章中，我们使用边际 (或对偶) 值来研究 最优解对 约束变化的敏感性。这里没有最优解，但求解器确实返回了它在尝试满足 约束时找到的最后一个解。我们可以通过显示与此解相关的一些值来寻找不可行性的来源：

<table><tr><td colspan="4">ampl: display Diet.lb, Diet.body, Diet.ub;</td><td></td></tr><tr><td>:</td><td>Diet.lb</td><td>Diet.body</td><td>Diet.ub</td><td>: =</td></tr><tr><td>A</td><td>700</td><td>1993.09</td><td>20000</td><td></td></tr><tr><td>B1</td><td>700</td><td>841.091</td><td>20000</td><td></td></tr><tr><td>B2</td><td>700</td><td>601.091</td><td>20000</td><td></td></tr><tr><td>C</td><td>700</td><td>1272.55</td><td>20000</td><td></td></tr><tr><td>CAL</td><td>16000</td><td>17222.9</td><td>24000</td><td></td></tr><tr><td>NA</td><td>0</td><td>40000</td><td>40000</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td></tr></table>

对于每种营养成分，`Diet.body` 是约束 `Diet[i]` 中各项 `amt[i,j] * Buy[j]` 的总和。`Diet.lb` 和 `Diet.ub` 的值是 `Diet[i]` 中总和的下界 (lower bounds) 和上界 (upper bounds) —— 在本例中，就是 `n_min[i]` 和 `n_max[i]` 的值。我们可以看到，求解器返回的饮食方案中维生素 B2 的供应量不足，而钠 (NA) 的含量已达到其上界。

此时，有两个明显的选择：我们可以降低对 B2 的需求，或者允许更多的钠摄入。如果我们尝试后者，并将钠的限制放宽至 $50,000\mathrm{mg}$，则可以得到一个可行解：

```
ampl: let n_max["NA"] := 50000;
ampl: solve;
MINOS 5.5: optimal solution found.
5 iterations, objective 118.0594032
ampl: display Buy;
Buy[*] :=
BEEF 5.36061
CHK 2
FISH 2
HAM 10
MCH 10
MTL 10
SPG 9.30605
TUR 2;
```

这至少是迈向可口饮食的一个开端，尽管我们需要花费 $118.06，相比原始的、限制较少的情况下的 $88.20 要高一些。显然，现在模型已经建立，我们可以轻松尝试许多其他可能性。（第 11.3 节描述了快速更改数据并重新求解的方法。）

解决方案中仍令人失望的一点是需要购买 5.36061 包牛肉和 9.30605 包意大利面。我们如何找到在整包装条件下尽可能最优的解？你可能会认为我们可以简单地将最优值四舍五入到整数（在优化领域中通常称为整数），但要以可行的方式做到这一点并不容易。使用 AMPL 修改报告的解，我们可以观察到，例如将牛肉向上取整到 6 包，意大利面向上取整到 10 包，将违反钠的限制：

```
ampl: let Buy["BEEF"] := 6;
ampl: let Buy["SPG"] := 10;
ampl: display Diet.lb, Diet.body, Diet.ub;
:   Diet.lb   Diet.body   Diet.ub    :=
A     700      2012       20000
B1    700      1060       20000
B2    700       720       20000
C     700      1730       20000
CAL  16000     20240      24000
NA      0      51522      50000;
```

（`let` 语句允许修改数据，详见第 11.3 节。）同样可以验证，将解向下取整到 5 包牛肉和 9 包意大利面会导致维生素 B2 不足。向上取整一个、向下取整另一个也不可行。通过足够多的尝试，你可以找到一个满足约束条件的、接近的全整数解，但仍然无法保证这是成本最低的全整数解。

AMPL 确实支持在变量声明中直接加入整数限制：

```
var Buy {j in FOOD} integer >= f_min[j], <= f_max[j];
```

然而，这只有在你使用能够处理变量必须为整数的问题的求解器时才有帮助。为此，我们转向 CPLEX，这是一个能够处理所谓整数规划 (integer programs) 的求解器。如果我们如上所述在变量 `Buy` 的声明中添加 `integer`，将生成的模型保存在文件 `diet1.mod` 中，并将更高的钠限制添加到 `diet2a.dat` 中，则可以按如下方式重新求解：

ampl: reset; model diet1.mod; data diet2a.dat; option solver cplex; solve; CPLEX 8.0.0: 最优整数解；目标函数值 119.3，11 次 MIP 单纯形法迭代，1 个分支定界节点。display Buy; Buy [*] := BEEF 9 CHK 2 FISH 2 HAM 8 MCH 10 MTL 10 SPG 7 TUR 2 ;

由于整数性是一个附加的约束条件，因此最佳整数解的成本比最佳“连续”解高出约 $1.24 并不令人意外。但饮食方案之间的差异却出人意料；有 3 种食物的数量发生了变化，每种变化都达到两个或更多包装单位。一般来说，整数性及其他“离散”限制会使模型的求解变得更加困难。我们将在第 20 章中详细讨论这一问题。

# 2.4 混合、经济与调度问题的推广

根据你的个人经验，饮食模型似乎并未被人们广泛用于选择晚餐。这些模型更适合应用于包装和个人偏好不占主导地位的情形，例如动物饲料的混合，或者大学食堂食物的配制。

饮食模型是线性规划建模的一个方便且直观的例子，它出现在许多不同的场景中。假设我们将该模型以更一般的形式重写，如图 2-4 所示。在饮食模型中被称为食物和营养素的对象，现在被更泛化地称为“输入”和“输出”。对于每个输入 j，我们必须决定使用数量为 X[j] 的输入，其取值范围在 in_min[j] 和 in_max[j] 之间，由此产生的成本等于 cost[j] * X[j]，同时我们会生成 io[i,j] * X[j] 单位的每种输出 i。我们的目标是找到成本最低的输入组合，使得对每种输出 i，其总量介于 out_min[i] 和 out_max[i] 之间。

在这类模型的一种常见应用中，输入是待混合的原材料。输出则是混合物的各种属性。原材料可以是动物饲料的成分，但也同样可以是调制成 PG (优质汽油) 的原油衍生物，或是投入焦炉的不同种类的煤炭。这些属性可以是某种物质的含量（如动物饲料中的钠或卡路里），也可能是更复杂的指标（如汽油的 vap (蒸汽压) 或 oct (研究辛烷值)），甚至是重量和体积等物理性质。

在另一个著名应用中，输入是经济某一部门的生产活动，而输出则是各种产品。参数 in_min 和 in_max 表示活动水平的上下限，而 out_min 和 out_max 则由需求量 (demand) 决定。因此，目标是以最低成本确定各项活动的水平，以满足需求。这种解释与经济均衡的概念相关，我们将在第 19 章中进一步阐述。

在另一个截然不同的应用中，输入是工作安排表，输出则对应于一个月中某些天的工作时长。对于特定的工作安排表 j，io[i,j] 表示按照安排表 j 工作的人在第 i 天工作的小时数（若无工作则为零），cost[j] 表示按照安排表 j 工作的人的月薪，x[j] 表示被分配到该安排表的工人数量。在此解释下，目标函数变为月工资总额 (Total_Cost)，而约束条件则表示对于每一天 i，被安排在该天工作的工人总数必须介于下限 out_min[i] 和上限 out_max[i] 之间。同样的方法可以用于各种其他调度情境中，其中的小时、天或月可以替换为其他时间段。

尽管线性规划 (Linear Program) 在此类应用中非常有用，但我们需要牢记支撑 LP 模型的基本假设。我们已经提到了“连续性”假设，即 X[j] 可以在 in_min[j] 和 in_max[j] 之间的任意值。对于混合 (Blending) 问题来说，这可能比调度 (Scheduling) 问题更合理得多。

另一个例子是，将目标函数写为

$$
\mathtt{sum}\{\mathtt{j}\mathtt{in}\mathtt{INPUT}\} \mathtt{cost}[\mathtt{j}]\star \mathtt{X}[\mathtt{j}]
$$

我们假设了“成本线性性 (Linearity of Costs)”，即输入的成本与所使用的输入量成正比，且总成本为各输入成本之和。

将约束条件写为

$$
\mathtt{out\_min[i]}\leq \mathtt{sum}\{\mathtt{j}\mathtt{in}\mathtt{INPUT}\}\mathtt{i o[i,j]}\star \mathtt{X[j]}\leq \mathtt{out\_max[i]}
$$

我们也假设了从特定输入中产生的输出 i 的产量与所使用的输入量成正比，且输出 i 的总产量为各输入产量之和。当输入为工作安排表、输出为工作时长时，“产量线性性 (Linearity of Yield)”假设不会造成问题。但在混合示例中，线性性是对原材料和品质性质的一种物理假设，这种假设可能成立，也可能不成立。例如，在早期炼油厂的应用中，人们就认识到，将铅作为输入添加会对混合物的辛烷值 (oct) 这一品质产生非线性影响。

AMPL 使得表达离散或非线性模型变得容易，但任何对连续性或线性性的偏离都可能导致最优解 (Optimal Solution) 更难获得。至少，它需要一个更强大的求解器来优化由此产生的数学规划。第 17 至 20 章将更详细地讨论这些问题。

# 参考文献

George B. Dantzig, "The Diet Problem." Interfaces 20, 4 (1990) pp. 43- 47. 对饮食问题起源的一个有趣叙述。

Susan Garner Garille 和 Saul I. Gass, "Stigler's Diet Problem Revisited." Operations Research 49, 1 (2001) pp. 1- 13. 对饮食问题起源及其多年来对线性规划和营养学家影响的回顾。

Said S. Hilal 和 Warren Erikson, "Matching Supplies to Save Lives: Linear Programming the Production of Heart Valves." Interfaces 11, 6 (1981) pp. 48- 56. 一个与饮食问题类似的不太令人愉悦的问题，涉及选择猪心脏供应商。

# 练习

2-1. 假设下面列出的食物每磅含有热量、蛋白质、钙、维生素 A 和成本如下所示。应该以什么数量购买这些食物，以至少满足列出的每日需求，同时使总成本最小？（这个问题来自 George B. Dantzig 的经典著作《Linear Programming and Extensions》第 118 页。我们相信他对营养值的判断，并出于怀旧原因保留了 1963 年该书出版时的价格。）

<table><tr><td></td><td>面包</td><td>肉类</td><td>土豆</td><td>卷心菜</td><td>牛奶</td><td>明胶</td><td>需求量</td></tr><tr><td>热量</td><td>1254</td><td>1457</td><td>318</td><td>46</td><td>309</td><td>1725</td><td>3000</td></tr><tr><td>蛋白质</td><td>39</td><td>73</td><td>8</td><td>4</td><td>16</td><td>43</td><td>70 g.</td></tr><tr><td>钙</td><td>418</td><td>41</td><td>42</td><td>141</td><td>536</td><td>0</td><td>800 mg.</td></tr><tr><td>维生素 A</td><td>0</td><td>0</td><td>70</td><td>860</td><td>720</td><td>0</td><td>500 I.U.</td></tr><tr><td>成本/磅</td><td>$0.30</td><td>$1.00</td><td>$0.05</td><td>$0.08</td><td>$0.23</td><td>$0.48</td><td></td></tr></table>

2-2. (a) 你的医生建议你多锻炼，具体来说，通过步行、慢跑、游泳、健身器械、室内协作娱乐和用餐时推离餐桌等组合方式每周至少消耗 2000 卡路里的额外热量。你对每项活动的耐受时间有限，以小时/周为单位；每项活动每小时消耗一定数量的卡路里，如下所示：

<table><tr><td></td><td>步行</td><td>慢跑</td><td>游泳</td><td>器械</td><td>室内</td><td>推离</td></tr><tr><td>卡路里</td><td>100</td><td>200</td><td>300</td><td>150</td><td>300</td><td>500</td></tr><tr><td>耐受时间</td><td>5</td><td>2</td><td>3</td><td>3.5</td><td>3</td><td>0.5</td></tr></table>

你应该如何在这些活动中分配锻炼时间，以最小化总耗时？

(b) 假设你还应使锻炼多样化 —— 前四项锻炼每项至少要做一小时，但步行、慢跑和器械锻炼的总时间不得超过四小时。以这种形式求解该问题。

2-3. (a) 一家软饮料制造商希望将三种糖大致以相等的数量混合，以确保产品口味均匀。供应商仅提供不同比例组合的糖，其成本/吨如下：

<table><tr><td>糖类</td><td>A</td><td>B</td><td>C</td><td>D</td><td>E</td><td>F</td><td>G</td></tr><tr><td>甘蔗糖</td><td>10%</td><td>10</td><td>20</td><td>30</td><td>40</td><td>20</td><td>60</td></tr><tr><td>玉米糖</td><td>30%</td><td>40</td><td>40</td><td>20</td><td>60</td><td>70</td><td>10</td></tr><tr><td>甜菜糖</td><td>60%</td><td>50</td><td>40</td><td>50</td><td>0</td><td>10</td><td>30</td></tr><tr><td>成本/吨</td><td>$10</td><td>11</td><td>12</td><td>13</td><td>14</td><td>12</td><td>15</td></tr></table>

建立一个 AMPL 模型，使得在生产包含 52 吨甘蔗糖、56 吨玉米糖和 59 吨甜菜糖的混合物时，供应成本最小。

(b) 制造商认为，为确保与供应商的良好关系，有必要从每个供应商处至少购买 10 吨。这对模型和最小成本解有何影响？

(c) 在 (a) 的模型基础上，建立一个替代模型，用于寻找混合一吨供应品的最低成本方式，且每种糖的含量占总量的 30% 至 37% 之间。

2-4. 在第 1 章末尾，我们说明了如何解释生产模型中约束的边际（或对偶）值以及变量的约简成本（reduced cost）。同样的思路也可应用于本章的饮食模型。

(a) 回到第 2.3 节中成功求解的饮食问题，我们可以按如下方式显示边际值：

<table><tr><td colspan="5">ampl: display Diet.lb,Diet.body,Diet.ub,Diet;</td></tr><tr><td></td><td>Diet.lb</td><td>Diet.body</td><td>Diet.ub</td><td>Diet</td></tr><tr><td>A</td><td>700</td><td>1956.29</td><td>20000</td><td>0</td></tr><tr><td>B1</td><td>700</td><td>1036.26</td><td>20000</td><td>0</td></tr><tr><td>B2</td><td>700</td><td>700</td><td>20000</td><td>0.404585</td></tr><tr><td>C</td><td>700</td><td>1682.51</td><td>20000</td><td>0</td></tr><tr><td>CAL</td><td>16000</td><td>19794.6</td><td>24000</td><td>0</td></tr><tr><td>NA</td><td>0</td><td>50000</td><td>50000</td><td>-0.00306905</td></tr></table>

如何解释这两个非零值？

(b) 对于同一问题，以下列表给出了约简成本（reduced costs）：

<table><tr><td colspan="5">ampl: display Buy.lb,Buy,Buy.ub,Buy.rc</td></tr><tr><td></td><td>Buy.lb</td><td>Buy</td><td>Buy.ub</td><td>Buy.rc</td></tr><tr><td>BEEF</td><td>2</td><td>5.36061</td><td>10</td><td>8.88178e-16</td></tr><tr><td>CHK</td><td>2</td><td>2</td><td>10</td><td>1.18884</td></tr><tr><td>FISE</td><td>2</td><td>2</td><td>10</td><td>1.14441</td></tr><tr><td>HAM</td><td>2</td><td>10</td><td>10</td><td>-0.302651</td></tr><tr><td>MCH</td><td>2</td><td>10</td><td>10</td><td>-0.551151</td></tr><tr><td>MTL</td><td>2</td><td>10</td><td>10</td><td>-1.3289</td></tr><tr><td>SPG</td><td>2</td><td>9.30605</td><td>10</td><td>0</td></tr><tr><td>TUR</td><td>2</td><td>2</td><td>10</td><td>2.73162</td></tr></table>

根据这些信息，如果你想通过吃超过 10 包某种食物来省钱，哪一种可能是你的最佳选择？

2-5. 一家连锁快餐店每周运营 7 天，从周一到周日需要的最少厨房员工数如下：45, 45, 40, 50, 65, 35, 35。每位员工被安排在周末（周六或周日）工作一天，并在一周内另外工作四天。管理层想知道满足每天需求所需的最少员工总数。

(a) 将此问题设置并求解为一个 线性规划 (linear program)。
(b) 根据第 2.4 节的讨论，解释如何将此问题视为图 2-4 中混合模型的一个特例。

2-6. 造纸厂的产出是标准的 110 英寸 (110") 宽的纸卷，这些纸卷被切割成更小的纸卷以满足订单需求。本周有以下宽度的订单：

<table><tr><td>宽度</td><td>订单数量</td></tr><tr><td>20"</td><td>48</td></tr><tr><td>45"</td><td>35</td></tr><tr><td>50"</td><td>24</td></tr><tr><td>55"</td><td>10</td></tr><tr><td>75"</td><td>8</td></tr></table>

造纸厂的老板想知道应该应用哪些切割模式，以便使用最少数量的 $110"$ 纸卷来完成订单。

(a) 一个切割模式由每种宽度的纸卷数量组成，例如两个 $45"$ 和一个 $20"$，或一个 $50"$ 和一个 $55"$（并产生 $5"$ 的废料）。假设一开始，我们只考虑以下六种模式：

<table><tr><td>宽度</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td></tr><tr><td>20"</td><td>3</td><td>1</td><td>0</td><td>2</td><td>1</td><td>3</td></tr><tr><td>45"</td><td>0</td><td>2</td><td>0</td><td>0</td><td>0</td><td>1</td></tr><tr><td>50"</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr><tr><td>55"</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td></tr><tr><td>75"</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr></table>

每种模式应切割多少卷，才能使所用的 110 英寸卷数最少？将该问题表示为一个 线性规划 (linear program) 并求解，假设生产的小卷数量只需大于或等于订单数量。

(b) 重新求解该问题，但增加限制条件：每种尺寸生产的卷数必须在订单数量的 10% 以下至 40% 以上之间。

(c) 找出另一种模式，将其加入上述模式后能改进最优解。

(d) 上述所有解都使用了分数卷数。你能找到满足约束条件且每种模式切割整数卷数的解吗？在每种情况下，你的整数解会使目标函数值增加多少？（参见第 20 章关于如何寻找最优整数解的讨论。）

2-7. 在练习 1-6 的炼油厂模型中，优质汽油的产量是一个 决策变量 (decision variable)。现在假设订单要求生产 42,000 桶。产品的辛烷值允许范围为 89 至 91，蒸汽压 (vap) 允许范围为 11.7 至 12.7。用于调和优质汽油的五种原料具有以下生产及/或采购成本：

SRG 9.57 N 8.87 RF 11.69 CG 10.88 B 6.75

其他数据与练习 1-6 相同。构建一个调和模型和数据文件来表示该问题。通过 AMPL 运行它们，以确定最优的调和配方。

2-8. 回顾图 2-4 将饮食模型推广为一个以最小 成本 (cost) 选择投入的模型，并对产出施加约束。

(a) 同样地，将图 1-6a 的生产模型推广为一个以最大 收入 (revenue) 选择产出的模型，并对投入施加约束。

(b) “投入-产出”模型 (input-output model) 是线性规划在经济分析中的最早应用之一。这种模型可以用一组活动 $A$ 和一组材料 $M$ 来描述。决策变量是活动运行的水平 $X_{j} \geq 0$；它们有下限 $u_{j}^{-}$ 和上限 $u_{j}^{+}$。

每个活动 $j$ 每单位有收入 $c_{j} > 0$，或每单位有成本 (用 $c_{j} < 0$ 表示)。因此，所有活动的总利润为 $\sum_{j \in A} c_{j} X_{j}$，该值需被最大化。

每个活动 $j$ 的每单位生产材料 $i$ 的数量为 $a_{ij} \geq 0$，或消耗材料 $i$ 的数量为 $a_{ij} < 0$。因此，若 $\sum_{j \in A} a_{ij} X_{j}$ 大于 0，则表示所有活动对材料 $i$ 的总产量；若小于 0，则表示所有活动对材料 $i$ 的总消耗量。

对于每种物料 $i$，其总产量存在一个上界 $b_{i}^{+} > 0$，或者其总消耗量存在一个下界 $b_{i}^{+}< 0$。类似地，其总产量存在一个下界 $b_{i} > 0$，或者其总消耗量存在一个上界 $b_{i}< 0$。

请写出该模型在 AMPL 中的表达式。

(c) 解释如何将最小成本投入选择模型 (minimum-cost input selection model) 和最大收益产出选择模型 (maximum-revenue output-selection model) 视为投入产出模型 (input-output model) 的特例。