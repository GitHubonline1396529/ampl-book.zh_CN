# 整数线性规划

许多线性规划问题要求某些变量取整数值。当变量表示诸如包裹或人员等不可分割的实体时，这种要求会自然出现——至少对于所建模的情形而言，将其分割是没有意义的。整数变量也用于构建表示逻辑条件的方程组，我们将在本章稍后部分对此进行说明。

在一些情况下，前几章中描述的优化技术足以找到一个整数解。对于某些网络线性规划问题，可以保证存在整数最优解，如第 15.5 节所述。即使没有这样的保证，线性规划求解器也可能恰好为特定模型实例找到一个整数最优解。例如，在我们指定的数据下（图 4-2），多商品运输模型（图 4-1）的求解就出现了这种情况。

即使求解器未返回整数解，也很有可能得到一个大多数变量取整数值的解。具体来说，许多求解器能够返回一个“极点”解，其中非边界变量的数量不超过约束 (constraint) 的数量。如果变量的边界是整数，则所有处于边界的变量都将是整数；如果其余数据也是整数，则剩余变量中的许多也可能为整数。随后，你可以调整相对较少的非整数变量，从而获得一个在实际应用中足够接近可行和最优的整数解。

图 16-4 和图 16-5 中的调度线性规划提供了一个例子。由于变量数量远超约束数量，最优解中大多数变量取其下界 0，同时其他一些变量也恰好为整数。剩余变量可以向上舍入到一个稍高成本但仍满足约束条件的整数解。

尽管存在所有这些可能性，但仍有许多情况下，求解器必须显式地执行整数性限制。整数规划求解器面临的问题比线性规划求解器要困难得多，然而，它们通常需要更多的计算机时间和内存，并且往往需要用户在模型构建和选项选择方面提供更多帮助。

因此，与线性规划相比，整数规划能够求解的问题规模将更加受限。

本章首先描述 AMPL 中普通整数变量的声明，然后介绍如何使用 0-1（或二进制）变量来建模逻辑条件。最后一节提供了一些关于有效构建和求解整数规划的建议。

## 20.1 整数变量

通过在 var 声明的限定短语中添加关键字 integer，可以将声明的变量限制为整数值。

例如，在分析第 2.3 节中的饮食模型时，我们得到了以下最优解：

```matlab
ampl: model diet.mod;
ampl: data diet2a.dat;
ampl: solve;
MINOS 5.5: optimal solution found.
13 iterations, objective 118.0594032
ampl: display Buy;
Buy [*] :=
BEEF  5.36061
CHK   2
FISH  2
HAM   10
MCH   10
MTL   10
SPG   9.30605
TUR   2
;
```

如果我们希望购买的食物量为整数，我们可以在模型的 var 声明（图 2-1）中添加 integer，如下所示：

```matlab
var Buy {j in FOOD} integer >= f_min[j], <= f_max[j];
```

然后我们可以尝试重新求解：

```matlab
ampl: model dieti.mod;
data diet2a.dat;
ampl: solve;
MINOS 5.5: ignoring integrality of 8 variables
MINOS 5.5: optimal solution found.
13 iterations, objective 118.0594032
```

正如你所看到的，MINOS 求解器不处理整数性约束。它忽略了这些约束并返回了与之前相同的最优值。

为了得到整数最优解，我们切换到一个能够处理整数性的求解器：

```matlab
ampl: option solver cplex;
ampl: solve;
CPLEX 8.0.0: optimal integer solution; objective 119.3
11 MIP simplex iterations
1 branch-and-bound nodes
ampl: display Buy;
Buy [*] :=
BEEF  9
CHK   2
FISH  2
HAM   8
MCH   10
MTL   10
SPG   7
TUR   2
;
```

将此解与之前的解进行比较，我们可以看到整数规划的一些典型特征。最小成本从 $118.06 增加到了 $119.30；因为整数性是对变量值的额外约束，它只能使目标函数变得不那么有利。饮食中不同食物的数量也发生了变化，但变化方式是不可预测的。在原始最优解中具有分数数量的两种食物 BEEF 和 SPG，分别从 5.36061 增加到 9，从 9.30605 减少到 7；此外，HAM 从上限 10 下降到 8。显然，你不能总是通过将非整数最优解四舍五入到最接近的整数值来推导出整数最优解。

## 20.2 0-1 变量和逻辑条件

只能取 0 和 1 值的变量是整数变量的一个特例。通过巧妙地将这些 “0-1” 或 “二进制” 变量纳入目标函数和约束条件中，整数线性规划可以指定各种逻辑条件，而这些条件无法仅通过线性约束以任何实际方式描述。

为了介绍 0-1 变量在 AMPL 中的使用，我们回到图 4-1 的多产品运输模型（multicommodity transportation model）。该模型中的决策变量 Trans[i,j,p] 表示从起始地 i（属于集合 ORIG）运输到目的地 j（属于集合 DEST）的产品 p（属于集合 PROD）的吨数。在图 4-2 给出的小型数据示例中，产品包括带钢（bands）、线圈（coils）和钢板（plate）；起始地包括 GARY、CLEV 和 PITT，共有七个目的地。

该模型将从 i 到 j 运输产品 p 的成本表示为 cost[i,j,p] * Trans[i,j,p]，无论运输量多少都是如此计算。这种 “可变” 成本是纯线性规划的典型特征，在这种情况下允许许多起始地-目的地对之间进行小批量运输。在以下示例中，我们将描述如何使用 0-1 变量来抑制小批量运输；同样的技术也可以适用于许多其他逻辑条件。

为了便于比较，我们关注从每个起始地到每个目的地运输的总吨数，即对所有产品求和的结果。这些总运输量的最优值由线性规划求解器确定如下：

```
ampl: model multi.mod;
ampl: data multi.dat;
ampl: solve;
MINDS 5.5: optimal solution found.
41 iterations, objective 199500
ampl: option display_eps .000001;
ampl: option display_transpose -10;
ampl: display {i in ORIG, j in DEST}
     sum {p in PROD} Trans[i,j,p];
sum[p in PROD] Trans[i,j,p] [*,*]
:   DET  FRA  FRE  LAR  LAN  STL  WIN :=
CLEV  625  275  325  225  400  550  200
GARY    0    0  625  150    0  625    0
PITT  525  625  225  625  100  625  175
;
```

在该解中，数值 625 频繁出现，这是由于多产品约束（multicommodity constraints）造成的：

```
subject to Multi {i in ORIG, j in DEST}:
   sum {p in PROD} Trans[i,j,p] <= limit[i,j];
```

在我们的示例数据中，对于所有的 i 和 j，limit[i,j] 都等于 625；解中出现的六个 625 对应于六个使多产品限制约束紧绷的运输路线。其他路线的运输量最低为 100；四个 0 的实例表示未使用的路线。

尽管在该解中所有运输量恰好为整数，但我们愿意运输分数数量。因此我们不会将 Trans 变量声明为整数变量，而是通过使用 0-1 整数变量来扩展模型。

# 固定成本

抑制小批量运输的一种方法是为实际使用的每条起始地-目的地路线添加一个 “固定” 成本。为此，我们将成本参数重命名为 vcost，并声明另一个参数 fcost 来表示使用从 i 到 j 路线的固定费用评估：

param vcost {ORIG, DEST, PROD} >= 0; # 路线上的可变成本  
param fcost {ORIG, DEST} > 0; # 路线上的固定成本

我们希望当从 i 到 j 的产品总运输量 —— 即 sum {p in PROD} Trans[i,j,p] —— 为正时，fcost[i,j] 被加入目标函数；当总运输量为零时，不加入任何成本。使用 AMPL 表达式，我们可以最直接地将目标函数写成如下形式：

minimize Total_Cost:  
sum {i in ORIG, j in DEST, p in PROD} vcost[i,j,p] * Trans[i,j,p] +  
sum {i in ORIG, j in DEST} if sum {p in PROD} Trans[i,j,p] > 0 then fcost[i,j];

AMPL 接受这个目标函数，但将其视为“非线性”的（按第 18 章的定义），因此在尝试最小化它时，你可能无法得到可接受的结果。

作为更实用的替代方案，我们可以为每条从 i 到 j 的路线引入一个新的变量 Use[i,j]，如下所示：当 sum {p in PROD} Trans[i,j,p] 为正时，Use[i,j] 取值为 1，否则为 0。那么与从 i 到 j 的路线相关的固定成本就是 fcost[i,j] * Use[i,j]，这是一个线性项。要在 AMPL 中声明这些新变量，我们可以说它们是整数变量，取值范围为 >= 0 且 <= 1；等价地，我们可以使用关键字 binary：

var Use {ORIG, DEST} binary;

然后目标函数可以写成一个线性表达式：

minimize Total_Cost:  
sum {i in ORIG, j in DEST, p in PROD} vcost[i,j,p] * Trans[i,j,p] +  
sum {i in ORIG, j in DEST} fcost[i,j] * Use[i,j];

由于模型中同时包含连续（非整数）变量和整数变量，它构成了所谓的“混合整数”规划。

为了完成模型，我们需要添加约束条件，以确保 Trans 和 Use 按预期方式相关联。这是该建模方法中的“巧妙”部分；我们只需修改前面提到的 Multi 约束，使其包含 Use 变量：

subject to Multi {i in ORIG, j in DEST}:  
sum {p in PROD} Trans[i,j,p] <= limit[i,j] * Use[i,j];

如果 Use[i,j] 为 0，该约束表示 sum {p in PROD} Trans[i,j,p] <= 0。

由于从 i 到 j 的这批运输量是若干非负变量的和，它必须等于 0。另一方面，当 Use[i,j] 为 1 时，该约束变为 sum {p in PROD} Trans[i,j,p] <= limit[i,j]。

这与之前多产品运输限制一致。虽然该约束没有直接阻止当 Use[i,j] 为 1 时 sum {p in PROD} Trans[i,j,p] 为 0 的情况，但只要 fcost[i,j] 为正，这种组合在最优解中永远不会出现。因此，Use[i,j] 将为 1 当且仅当 sum {p in PROD} Trans[i,j,p] 为正，这正是我们想要的结果。完整模型如图 20-1a 所示。

为了展示该模型如何求解，我们在示例数据中添加了一个固定成本表（见图 20-1b）：

```ampl
param fcost:
FRA DET LAN WIN STL FRE LAF :=
GARY 3000 1200 1200 1200 2500 3500 2500
CLEV 2000 1000 1500 1200 2500 3000 2200
PITT 2000 1200 1500 1500 2500 3500 2200 ;
```

如果我们使用与之前相同的求解器，那么对变量 Use 的整数性限制将被忽略：

```ampl
ampl: model multmip1.mod;
ampl: data multmip1.dat;
ampl: solve;
MINOS 5.5: 忽略 21 个变量的整数性
MINOS 5.5: 找到最优解。
43 次迭代，目标函数值 223504
ampl: option display_eps .000001;
ampl: option display_transpose -10;
```

```ampl
set ORIG := GARY CLEV PITT ;
set DEST := FRA DET LAN WIN STL FRE LAF ;
set PROD := bands coils plate ;

param supply (tr):
GARY CLEV PITT :=
bands 400 700 800
coils 800 1600 1800
plate 200 300 300 ;

param demand (tr):
FRA DET LAN WIN STL FRE LAF :=
bands 300 300 100 75 650 225 250
coils 500 750 400 250 950 850 500
plate 100 100 0 50 200 100 250 ;

param limit default 625 ;

param vcost :=
[*,*,bands]:
FRA DET LAN WIN STL FRE LAF :=
GARY 30 10 8 10 11 71 6
CLEV 22 7 10 7 21 82 13
PITT 19 11 12 10 25 83 15
[*,*,coils]:
FRA DET LAN WIN STL FRE LAF :=
GARY 39 14 11 14 16 82 8
CLEV 27 9 12 9 26 95 17
PITT 24 14 17 13 28 99 20
[*,*,plate]:
FRA DET LAN WIN STL FRE LAF :=
GARY 41 15 12 16 17 86 8
CLEV 29 9 13 9 28 99 18
PITT 26 14 17 13 31 104 20 ;

param fcost:
FRA DET LAN WIN STL FRE LAF :=
GARY 3000 1200 1200 1200 2500 3500 2500
CLEV 2000 1000 1500 1200 2500 3000 2200
PITT 2000 1200 1500 1500 2500 3500 2200 ;
```

```ampl
ampl: display sum {i in ORIG, j in DEST, p in PROD}
         vcost[i,j,p] * Trans[i,j,p];
sum[i in ORIG, j in DEST, p in PROD]
vcost[i,j,p] * Trans[i,j,p] = 199500

ampl: display Use;
Use [*,*]
:
     DET FRA FRE LAF LAN STL WIN :=
CLEV   1 0.44 0.52 0.36 0.64 0.88 0.32
GARY   0    0    1 0.24    0    1    0
PITT 0.84    1 0.36    1 0.16    1 0.28 ;
```

如你所见，总可变成本与之前相同，而 Use 变量取了一系列不同的小数值。这个解并没有告诉我们任何新信息，而且也没有简单的方法将其转换为一个较好的整数解。在这种情况下，整数规划求解器对于获得任何实用结果都是必不可少的。

切换到适当的求解器后，我们发现考虑固定成本的真实最优解如下：

```ampl
ampl: option solver cplex; solve;  
CPLEX 8.0.0: 最优整数解；目标函数值 229850  
295 次 MIP 单纯形法迭代  
19 个分支定界节点。  

ampl: display {i in ORIG, j in DEST} sum {p in PROD} Trans[i,j,p];  
sum{p in PROD} Trans[i,j,p] [*,*]:  
     DET  FRA  FRE  LAF  LAN  STL  WIN :=  
GARY 625  275    0  425  350  550  375  
CLEV   0    0  625    0  150  625    0  
PITT 525  625  550  575    0  625    0 ;  

ampl: display Use;  
Use [*,*]:  
     DET  FRA  FRE  LAF  LAN  STL  WIN :=  
GARY   1    1    0    1    1    1    1  
CLEV   0    0    1    0    1    1    0  
PITT   1    1    1    1    0    1    0 ;
```

加入整数约束后，总成本从 $ 223,504 增加到了 $ 229,850，但未使用的路线数量也如我们所希望的那样增加到了七条。

# 零或最小值限制

尽管考虑固定成本的解使用了更少的路线，但仍有部分路线上运输量相对较低。在实际应用中，可能即使可变成本也不适用，除非运输的吨数达到某个最小值。因此，假设我们定义一个参数 `minload` 来表示在一条路线上可以运输的最小吨数。我们可以添加一个约束，要求每条路线上运输量的总和 (对所有产品求和) 至少为 `minload`：

```ampl
subject to Min_Ship {i in ORIG, j in DEST}    # 错误  
    sum {p in PROD} Trans[i,j,p] >= minload;
```

但这样做会强制每条路线上的运输量至少为 `minload`，这不是我们想要的。我们希望运输的吨数要么为零，要么至少为 `minload`。要直接表达这一点，我们可以写成：

```ampl
subject to Min_Ship {i in ORIG, j in DEST}    # 不被允许  
    sum {p in PROD} Trans[i,j,p] = 0 or sum {p in PROD} Trans[i,j,p] >= minload;
```

但目前版本的 AMPL 不支持在约束条件中使用逻辑运算符。

我们可以通过使用变量 `Use[i,j]` (与前一个例子类似) 来实现所需的“零或最小值”限制：

```ampl
subject to Min_Ship {i in ORIG, j in DEST}:  
    sum {p in PROD} Trans[i,j,p] >= minload * Use[i,j];
```

当从 `i` 到 `j` 的总运输量为正时，`Use[i,j]` 为 1，`Min_Ship[i,j]` 就变成了我们所期望的最小运输量约束。另一方面，当从 `i` 到 `j` 没有运输量时，`Use[i,j]` 为 0；此时约束变为 $ 0 \geq 0 $，没有实际影响。

在加入这些新限制并设定 `minload` 为 375 后，得到的解如下：

```markdown
AMPL: model multmip2.mod;  
AMPL: data multmip2.dat;  
AMPL: solve;  
CPLEX 8.0.0: optimal integer solution; objective 233150  
279 MIP simplex iterations  
17 branch-and-bound nodes.  

AMPL: display {i in ORIG, j in DEST}  
> sum {p in PROD} Trans[i,j,p];  

sum{p in PROD}Trans[i,j,p] [*,*]:  
     DET  FRA  FRE  LAR  LAN  STL  WIN :=  
CLLEY  625  425  425    0  500  625    0  
GARY     0    0  375  425    0  600    0  
PITT   525  475  375  575    0  575  375 ;  

与之前的解相比，我们发现虽然仍有 7 条未使用的路线，但这些路线并不相同；为满足最小运输量的要求，必须对解进行大幅调整。总成本因此上升了约 $1.4\%$。

# 基数限制

尽管我们已经添加了上述约束，起始地 PITT 仍然服务了 6 个目的地，而 CLLEY 服务了 5 个，GARY 服务了 3 个。我们希望显式添加进一步的限制：每个起始地最多只能向 maxserve 个目的地运输货物，其中 maxserve 是模型的一个参数。这可以看作是对某个集合的大小或基数 (cardinality) 的限制。实际上，原则上可以写成如下的 AMPL 约束形式：

subject to Max_Serve {i in ORIG}: # 不允许  
    card {j in DEST: sum {p in PROD} Trans[i,j,p] > 0} <= maxserve;

然而，这样的声明会被拒绝，因为 AMPL 目前不允许约束使用依赖于变量定义的集合。

set ORIG: # 起始地  
set DEST: # 目的地  
set PROD: # 产品  

param supply {ORIG,PROD} >= 0; # 起始地的可用量  
param demand {DEST,PROD} >= 0; # 目的地的需求量  

check {p in PROD}:  
    sum {i in ORIG} supply[i,p] = sum {j in DEST} demand[j,p];  

param limit {ORIG,DEST} >= 0; # 路线的最大运输量  
param minload >= 0; # 非零运输的最小量  
param maxserve integer > 0; # 最大服务目的地数  
param vcost {ORIG,DEST,PROD} >= 0; # 路线上的可变运输成本  
var Trans {ORIG,DEST,PROD} >= 0; # 待运输的单位数量  
param fcost {ORIG,DEST} >= 0; # 使用某条路线的固定成本  
var Use {ORIG,DEST} binary; # 仅当路线被使用时为 1  

minimize Total_Cost:  
    sum {i in ORIG, j in DEST, p in PROD} vcost[i,j,p] * Trans[i,j,p] +  
    sum {i in ORIG, j in DEST} fcost[i,j] * Use[i,j];  

subject to Supply {i in ORIG, p in PROD}:  
    sum {j in DEST} Trans[i,j,p] = supply[i,p];  

subject to Max_Serve {i in ORIG}:  
    sum {j in DEST} Use[i,j] <= maxserve;  

subject to Demand {j in DEST, p in PROD}:  
    sum {i in ORIG} Trans[i,j,p] = demand[j,p];  

subject to Multi {i in ORIG, j in DEST}:  
    sum {p in PROD} Trans[i,j,p] <= limit[i,j] * Use[i,j];  

subject to Min_Ship {i in ORIG, j in DEST}:  
    sum {p in PROD} Trans[i,j,p] >= minload * Use[i,j];  

再次使用 0-1 变量提供了一种方便的替代方法。由于变量 Use[i,j] 恰好在起始地 i 服务目的地 j 时为 1，否则为 0，因此我们可以用 sum {j in DEST} Use[i,j] 表示由 i 服务的目的地数量。所需的约束变为：
```

受制于

$$
\text{Max\_Serve} \{i \in \text{ORIG}\}: \sum_{j \in \text{DEST}} \text{Use}[i,j] \leq \text{maxserve};
$$

将此约束添加到先前的模型中，并将 maxserve 设置为 5，我们得到混合整数模型，如图 20-2a 所示，其数据如图 20-2b 所示。优化结果如下：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/e62c6eda2165ef4f24413d8a369f2d77b1fa50865fed9d0ab775755da6c1ac88.jpg)  
图 20-2b：图 20-2a 的数据 (multmip3.dat)。

```ampl
ampl: model multmip3.mod;
ampl: data multmip3.dat;

ampl: solve;
CPLEX 8.0.0: optimal integer solution; objective 235625
392 MIP simplex iterations
36 branch-and-bound nodes

ampl: display {i in ORIG, j in DEST}
      sum {p in PROD} Trans[i,j,p];
sum{p in PROD} Trans[i,j,p] [*,*]
:   DET  FRA  FRE  LAR  LAN  STL  WIN :=
CLEY  625  375  550    0  500  550    0
GARY    0    0    0  400    0  625  375
PITT  525  525  625  600    0  625    0
;
```

在目标函数增加 $1.1\%$ 的成本下，通过重新安排使得 GARY 可以服务 WIN，从而将 PITT 所服务的目的地数量减少到五个。

注意，本节的三个整数解分别从三个不同的起始地服务了 WIN —— 这是整数规划解在约束条件发生微小变化时可能出现剧烈跳变的一个很好的例子。

# 20.3 整数规划的实际考虑

通常而言，任何整数规划 (Integer Programming) 问题都比具有相同规模和结构的线性规划 (Linear Programming) 问题难解得多。在前面的例子中，求解器 (Solver) 报告的迭代 (Iterations) 次数粗略地反映了这种差异：求解原始线性多商品运输问题需要 41 次迭代，而混合整数版本则需要大约 280 到 400 次迭代。计算时间也呈现出类似的变化：线性规划几乎是瞬间求解的，而混合整数规划则明显更慢。

随着问题规模的增大，整数规划的难度增长速度也快于可比的线性规划问题。因此，可求解的整数规划问题的实际规模限制会更加严格。实际上，AMPL 可以轻松生成对你的计算机来说在合理的时间或内存内难以求解的整数规划问题。因此，在决定使用包含整数变量的模型之前，应考虑是否可以使用替代的连续线性或网络模型来获得足够好的结果。如果你必须使用整数规划模型，请尝试在逐渐增大规模的数据集上运行，以便了解所需的计算机资源。

如果你确实遇到了一个难以求解的整数规划问题，那么你应该考虑对其进行重新建模，使其更容易处理。整数规划求解器会尝试遍历整数变量所有可能的取值组合；尽管它采用了一种复杂策略，可以排除绝大多数不可行或次优的组合，但剩下仍需检查的组合数量可能仍然非常庞大。因此，你应该尽量少使用整数变量。对于那些不是 0-1 变量的变量，应尽可能收紧其上下界，以减少可能需要检查的组合数量。

求解器通过固定或限制某些整数变量，然后求解去除其余整数约束后得到的线性规划“松弛问题”（relaxation），来获得关于整数规划解的有价值信息。你可以通过重新建模来协助这种策略，使得松弛问题的解更接近于相关整数问题的解。例如，在多商品运输模型（multicommodity transportation model）中，如果 $\text{Use}[i,j]$ 为 0，则每个单独变量 $\text{Trans}[i,j,p]$ 必须为 0；而如果 $\text{Use}[i,j]$ 为 1，则 $\text{Trans}[i,j,p]$ 不能大于 $\text{supply}[i,p]$ 或 $\text{demand}[j,p]$ 中的较小值。这提示我们可以添加如下约束：

subject to Avail {i in ORIG, j in DEST, p in PROD}: Trans[i,j,p] $\leq$ min(supply[i,p], demand[j,p]) * Use[i,j];

尽管这些约束并未排除任何之前可行的整数解，但它们往往会在松弛问题的解中，对于使用从 $i$ 到 $j$ 路线的情况，迫使 $\text{Use}[i,j]$ 更接近于 1。因此，松弛问题变得更精确，这可能有助于求解器更快地找到最优整数解；这种优势可能会超过处理更多约束所带来的额外成本。然而，这类权衡通常出现在比本节示例规模大得多的问题中。

如本例所示，在整数规划中，模型的构建选择比在线性规划中更为关键。对于大规模问题，简单的重新建模可能导致求解时间发生巨大变化。然而，重新建模的效果往往难以预测，它取决于模型和数据的结构，以及求解器所使用策略的细节。通常你需要进行一些实验，才能确定哪种方法最有效。

你也可以尝试通过更改某些默认设置来帮助求解器，这些设置决定了求解器如何初始处理问题以及如何搜索整数解。许多这类设置可以在 AMPL 内部进行调整，具体方法详见使用特定求解器的相关说明。

最后，求解器可能会提供提前停止的选项，返回一个整数解，该解已被确定能使目标值接近最优值的一个小百分比内。在某些情况下，这样的解在求解过程的早期阶段就能找到；通过不坚持要求求解器继续寻找可证明的最优整数解，您可以节省大量的计算时间。

总之，通常需要通过实验来求解整数或混合整数线性规划。如果您谨慎地使用整数变量，并且愿意不断尝试替代的公式、设置和策略，直到获得可接受的结果，那么您成功的几率会更大。

# 参考文献

Ellis L. Johnson, Michael M. Kostreva and Uwe H. Suhl, "Solving 0- 1 Integer Programming Problems Arising from Large- Scale Planning Models." Operations Research 33 (1985) pp. 803- 819. 一项案例研究，其中预处理、重新表述和算法策略被应用于解决一类困难的整数线性规划问题。

George L. Nemhauser and Laurence A. Wolsey, *Integer and Combinatorial Optimization*. John Wiley & Sons (New York, NY, 1988). 对整数规划问题、理论和算法的综述。

亚历山大·施里弗，《线性与整数规划理论》。约翰·威利父子出版社 (纽约州纽约市，1986 年)。一本关于该学科基础知识的指南，包含特别全面的参考文献集合。

劳伦斯·A·沃尔西，《整数规划》。威利-国际科学出版社 (纽约州纽约市，1998 年)。一本实用的、中等水平的指南，适合熟悉线性规划的读者。

# 练习题

20-1. 练习 1-1 优化了一个允许各种媒体任意组合的广告活动。假设相反，您必须在几种媒体中分配一个 100 万美元的广告活动，但对于每种媒体，您的唯一选择是是否使用该媒体。下表显示了每种媒体的成本和受众（以千为单位），以及如果使用该媒体将需要投入的创意时间（分为三类）。最后一列显示了您在成本和创意时间上的限制。

<table><tr><td></td><td>电视</td><td>杂志</td><td>广播</td><td>报纸</td><td>邮件</td><td>电话</td><td>限制</td></tr><tr><td>受众</td><td>300</td><td>200</td><td>100</td><td>150</td><td>100</td><td>50</td><td></td></tr><tr><td>成本</td><td>600</td><td>250</td><td>100</td><td>120</td><td>200</td><td>250</td><td>1000</td></tr><tr><td>文案</td><td>120</td><td>50</td><td>20</td><td>40</td><td>30</td><td>5</td><td>200</td></tr><tr><td>美术</td><td>120</td><td>80</td><td>0</td><td>60</td><td>40</td><td>0</td><td>300</td></tr><tr><td>其他</td><td>20</td><td>20</td><td>20</td><td>20</td><td>20</td><td>200</td><td>200</td></tr></table>

您的目标是在限制条件下最大化受众。

(a) 针对这种情况建立一个 AMPL 模型，对每种媒体使用一个 0-1 整数变量。  
(b) 使用整数规划求解器寻找最优解。同时求解放松整数限制的问题，并比较所得结果。你能否通过观察非整数解来猜测整数最优解？

20-2. 回到图 4-1 和图 4-2 的多商品运输问题。使用适当的整数变量施加以下各项限制。（各部分分开处理；不要试图将所有限制放在一个模型中。）求解所得整数规划问题，并从总体上评论由于限制条件的加入，解发生了怎样的变化。同时求解放松整数限制的相应线性规划问题，并将 LP 解与整数解进行比较。

(a) 要求每个起始地-目的地链路上的运输量为 100 吨的整数倍。为了适应这一限制，需求只能以最接近的 100 吨为单位来满足 —— 例如，由于 FRE 的带钢需求为 225 吨，因此只允许向 FRE 运输 200 或 300 吨。  
(b) 要求除 STL 外的每个目的地最多由两个起始地提供服务。  
(c) 要求使用的起始地-目的地链路数量尽可能少，不考虑成本。  
(d) 要求对每个向目的地 j 提供产品 p 的起始地，要么不运输任何货物，要么至少运输需求量 [j, p] 与 150 吨中的较小者。

20-3. 员工排班问题是整数规划的常见来源，因为安排一个人的一部分工作时间可能没有意义。

(a) 求解练习 4-4(b) 的问题，要求表示在时段 t 的班次 s 中工作的班组数量的变量 $Y_{st}$ 全部为整数 (integer)。验证最优整数解仅仅是将线性规划解向上取整到下一个整数。

(b) 类似地，尝试为练习 4-4(c) 中加入了库存量的问题寻找整数解。将该问题的难度与 (a) 中的问题进行比较。在这种情况下，最优整数解是否与线性规划解向上取整的结果相同？

(c) 求解图 16-4 和图 16-5 中的排班问题，并要求分配给每个班次的员工数量为整数。证明该解优于向上取整所得的解。

(d) 假设所需主管人数为员工人数的 1/10，并四舍五入到最接近的整数。再次求解主管的排班问题。在这种情况下，整数规划问题是更难还是更容易求解？与向上取整解相比，改进程度更显著还是更不显著？

20-4. 所谓的背包问题 (knapsack problem) 在许多情境中都会出现。其最简单的形式是，你从一组物品开始，每个物品都有已知的重量和价值。问题在于决定将哪些物品放入你的背包中。你希望这些物品的总价值尽可能大，但它们的重量不能超过某个预设的限制。

(a) 简单背包问题的数据可以用 AMPL 编写如下：

```ampl
set OBJECTS;
param weight {OBJECTS} > 0;
param value {OBJECTS} > 0;
```

使用这些声明，为背包问题构建一个 AMPL 模型。

使用你的模型来解决以下物品的背包问题，重量限制为 100：

<table>
<tr><td>物品</td><td>a</td><td>b</td><td>c</td><td>d</td><td>e</td><td>f</td><td>g</td><td>h</td><td>i</td><td>j</td></tr>
<tr><td>重量</td><td>55</td><td>50</td><td>40</td><td>35</td><td>30</td><td>30</td><td>15</td><td>15</td><td>10</td><td>5</td></tr>
<tr><td>价值</td><td>1000</td><td>800</td><td>750</td><td>700</td><td>600</td><td>550</td><td>250</td><td>200</td><td>200</td><td>150</td></tr>
</table>

(b) 假设你要装满几个相同的背包，由以下额外参数指定：

```ampl
param knapsacks > 0 integer; # 背包数量
```

为这种情况建立一个 AMPL 模型。不要忘记添加一个约束，即每个物品只能放入一个背包！

使用 (a) 中的数据，求解 2 个重量限制为 50 的背包。这个解与之前的解有什么不同？

(c) 表面上看，前面的背包问题类似于指派问题 (assignment problem)；我们有一组物品和一组背包，希望从前者向后者做出最优指派。第 15.2 节中描述的指派问题类型与 (b) 中描述的背包问题之间的本质区别是什么？

(d) 修改 (a) 中的公式，使其同时适应背包的体积限制和重量限制。使用以下体积再次求解：

<table>
<tr><td>物品</td><td>a</td><td>b</td><td>c</td><td>d</td><td>e</td><td>f</td><td>g</td><td>h</td><td>i</td><td>j</td></tr>
<tr><td>体积</td><td>3</td><td>3</td><td>3</td><td>2</td><td>2</td><td>2</td><td>2</td><td>1</td><td>1</td><td>1</td></tr>
</table>

这个解的总重量、体积和价值与你在 (a) 中找到的解相比如何？

(e) 练习 20-1 中的媒体选择问题如何被视为像 (d) 中那样的背包问题？

(f) 假设你可以将每种物品最多放入 3 个，而不是仅 1 个。修改 (a) 的模型以适应这种可能性，并使用相同数据重新求解。最优解如何变化？

(g) 回顾在练习 2-6 中描述的轧辊切割问题。给定宽轧辊的供应量、对窄轧辊的订单，以及一组切割模式，我们寻求一种模式组合，以使用最少的材料满足所有订单。

当该问题被解决时，算法会为每个订购的轧辊宽度返回一个“对偶值”。可以将对偶值解释为每获得一个额外窄轧辊可能实现的宽轧辊节省量；例如，对于一个 50 英寸的轧辊，其值为 0.5 表示如果能获得 10 个额外的 50 英寸轧辊，你可能节省 5 个宽轧辊。

自然会提出这样一个问题：在所有适合于一个宽轧辊内的窄轧辊模式中，哪个模式具有最大的总（对偶）值？解释如何将其视为一个背包问题。

对于练习 2-6(a) 的问题，宽轧辊为 110 英寸；在使用给定的六种模式的解中，订购宽度的对偶值为：

20 英寸 0.0  40 英寸 0.5  50 英寸 1.0  55 英寸 0.0  75 英寸 1.0

最大值模式是什么？证明它不是给定的模式之一，并且添加它可以让你得到一个更好的解。

20-5. 回顾在第 4.2 节中介绍的多期生产模型。在图 4-4 的公式中添加 0-1 变量和适当的约束，以施加以下描述的每项限制。（分别处理每个部分。）使用图 4-5 的数据求解，并确认解正确遵守了限制。

(a) 要求对于每个产品 p 和周 t，该周要么不生产该产品，要么至少生产 2500 吨。
(b) 要求任何一周内只能生产一种产品。
(c) 要求每种产品在任何三周的时间段内最多生产两周。假设在“第 -1 周”仅生产了带钢，而在“第 0 周”同时生产了带钢和线圈。