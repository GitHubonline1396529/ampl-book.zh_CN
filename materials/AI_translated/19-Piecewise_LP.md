
# 分段线性规划 (Piecewise-Linear Programs)

有几类线性规划问题会使用并非真正线性的函数，而是由相连的线性段拼接而成的函数：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/1c2d9ded8b4c651c2beb237cd5288e21d8047b21929ebad1d72a56d8ec60066d.jpg)

这些“分段线性 (piecewise-linear)”项很容易想象，但用传统的代数记号来描述却可能很困难。因此，AMPL 提供了一种特殊的、简洁的方式来书写它们。

本章通过分段线性目标函数的例子介绍 AMPL 的分段线性记号。在 17.1 节中，使用可能包含多个段的项来更准确地描述成本，比单一的线性关系更加精确。17.2 节展示了如何使用两段或三段的项来实现诸如对约束偏离的惩罚、处理不可行性以及建模“可逆”活动等目的。最后，17.3 节描述了可以用其他 AMPL 运算符和函数书写的分段线性函数；其中一些最有效地处理方式是转换为分段线性记号，而另一些则只能通过更复杂的变换来处理。

虽然本章中的分段线性例子都很容易求解，但表面上相似的例子可能要困难得多。因此，本章最后一节提供了构造和使用分段线性项的指导原则。我们解释了如何通过分段线性项的凸性或凹性来刻画那些容易处理的情况。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/db69b1ec3211891c61bb324490ce51b8553618dcbd8e309a1cd3a447bae14fda.jpg)  
图17-1：分段线性函数，包含三个斜率。

# 17.1 成本项 (Cost terms)

分段线性常用于比单纯线性项更真实地描述成本。在这种应用中，分段线性项与非线性项的目的大致相同，但没有第 18 章将要描述的一些困难。

为了明确比较，我们将使用与第 18 章相同的运输问题例子。我们通过一个简单的例子介绍 AMPL 的分段线性项记号，该例子中每个运输路径的成本层级（和线性段）数量是固定的。然后我们展示如何通过对记号的扩展，使用索引表达式通过数据来指定不同数量的段。

# 固定段数 (Fixed numbers of pieces)

在线性运输模型中，如图 3-1a 所示，从某个起始地到某个目的地，任意数量的单位都可以按相同的单位成本进行运输。然而，更现实的情况是，最优惠的费率可能仅适用于有限数量的单位；超过此限制的运输量需支付更高的费率。举例来说，假设为每对起始地-目的地指定了三个成本费率等级。那么沿某条路径运输的总成本将随着运输量的增加而呈分段线性增长，分为三段，如图 17-1 所示。

为了建模这种三段成本，我们将图 3-1a 中的参数 cost 替换为三个费率和两个限制：

```ampl
param rate1 {i in ORIG, j in DEST} >= 0;
param rate2 {i in ORIG, j in DEST} >= rate1[i,j];
param rate3 {i in ORIG, j in DEST} >= rate2[i,j];
param limit1 {i in ORIG, j in DEST} > 0;
param limit2 {i in ORIG, j in DEST} > limit1[i,j];
```

从 i 到 j 的运输量按每单位 rate1[i,j] 的费率计费，最多不超过 limit1[i,j] 单位；然后按每单位 rate2[i,j] 的费率计费，最多不超过 limit2[i,j] 单位；之后按每单位 rate3[i,j] 的费率计费。通常 rate2[i,j] 会大于 rate1[i,j]，rate3[i,j] 会大于 rate2[i,j]，但如果从 i 到 j 的路径没有三个不同的费率，则它们可能相等。

在线性运输模型中，目标函数通过变量和参数 cost 表示如下：

```ampl
var Trans {ORIG, DEST} >= 0;
minimize Total_Cost: sum {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];
```

我们可以类似地表达一个分段线性目标函数，通过引入三组变量，每组变量表示在每个费率下运输的数量：

```ampl
var Trans1 {i in ORIG, j in DEST} >= 0 <= limit1[i,j];
var Trans2 {i in ORIG, j in DEST} >= 0 <= limit2[i,j] - limit1[i,j];
var Trans3 {i in ORIG, j in DEST} >= 0;
minimize Total_Cost: sum {i in ORIG, j in DEST} (
    rate1[i,j] * Trans1[i,j] +
    rate2[i,j] * Trans2[i,j] +
    rate3[i,j] * Trans3[i,j]
);
```

但这样一来，新变量就必须引入到所有约束中，而且在需要显示最优结果时，我们也必须处理这些变量。为了避免这些麻烦，我们更希望直接用原始变量来显式描述分段线性成本函数。由于在代数记号中没有标准方式来描述分段线性函数，AMPL 提供了自己的语法来实现这一目的。

图 17-1 中所示的分段线性函数在 AMPL 中可写成如下形式：

```ampl
<<limit1[i,j], limit2[i,j]; rate1[i,j], rate2[i,j], rate3[i,j]>> Trans[i,j]
```

位于 $< <$ 与 $>>$ 之间的表达式描述了一个分段线性函数 (piecewise-linear function)，其后跟随的是所应用变量的名称。（你可以将其视为对 Trans[i,j] 的“乘法”，但其系数为一系列数值而非单一数值。）该表达式的两个部分分别为：函数斜率发生变化的断点列表，以及斜率列表——在本例中即为费率 (cost rates)。两个列表之间由分号分隔，每个列表内部的成员由逗号分隔。由于第一个斜率应用于第一个断点之前的值，最后一个斜率应用于最后一个断点之后的值，因此斜率的数量必须比断点数量多一。

虽然断点和斜率列表足以描述优化中的分段线性成本函数，但它们并不能唯一确定该函数。例如，如果我们在每一点的成本上都加上 10，尽管所有断点和斜率保持不变，但得到的成本函数却是不同的。为了解决这种歧义性，

```ampl
set ORIG; # 起始地
set DEST; # 目的地
param supply {ORIG} >= 0; # 起始地的供应量
param demand {DEST} >= 0; # 目的地的需求量
check: sum {i in ORIG} supply[i] = sum {j in DEST} demand[j];
param ratel {i in ORIG, j in DEST} >= 0;
param ratel2 {i in ORIG, j in DEST} >= ratel[i,j];
param ratel3 {i in ORIG, j in DEST} >= rate2[i,j];
param limitl {i in ORIG, j in DEST} > 0;
param limit2 {i in ORIG, j in DEST} > limitl[i,j];
var Trans {ORIG,DEST} >= 0; # 运输量
minimize Total_Cost:
  sum {i in ORIG, j in DEST}
    <<limitl[i,j], limit2[i,j]; ratel[i,j], rate2[i,j], rate3[i,j]>> Trans[i,j];
subject to Supply {i in ORIG}: sum {j in DEST} Trans[i,j] = supply[i];
subject to Demand {j in DEST}: sum {i in ORIG} Trans[i,j] = demand[j];
```

AMPL 假设分段线性函数在零点处的值为零，如图 17-1 所示。其他可能情况的选项将在本章后面讨论。

将所有链路的成本相加，现在分段线性的目标函数可写为：

```ampl
minimize Total_Cost:
  sum {i in ORIG, j in DEST}
    <<limit1[i,j], limit2[i,j]; ratel[i,j], rate2[i,j], rate3[i,j]>> Trans[i,j];
```

变量和约束的声明与之前保持一致；完整模型如图 17-2 所示。

# 分段数量可变

在前一个示例中所采用的方法，最适合于每项仅有少量线性段的情况。例如，如果每项有 12 段而非 3 段，则定义从 ratel[i,j] 到 rate12[i,j] 以及从 limit1[i,j] 到 limit11[i,j] 的模型将变得难以处理。此外，在实际中，每项更可能是最多有 12 段，而非恰好 12 段；对于段数少于 12 的项，可以通过使某些费率相等来处理，但对于大量段数的情况，这将变得

（注：原文未完整）

这将是一个繁琐的设备，需要许多不必要的数据值，并且会掩盖每项中实际的段数。

一个更好的方法是让段数（即运输费率的数量）本身成为模型的一个参数，并按链路进行索引：

```ampl
param npiece {ORIG,DEST} integer >= 1;
```

然后，我们可以对所有链路和段的组合进行费率和限制的索引：

```ampl
param rate {i in ORIG, j in DEST, p in 1..npiece[i,j]} >= if p = 1 then 0 else rate[i,j,p-1];
param limit {i in ORIG, j in DEST, p in 1..npiece[i,j]-1} > if p = 1 then 0 else limit[i,j,p-1];
```

对于任何特定的起始地 i 和目的地 j，成本项中的线性段数由 `npiece[i,j]` 给出。斜率为 `rate[i,j,p]`，其中 p 的范围从 1 到 `npiece[i,j]`，而中间的断点为 `limit[i,j,p]`，其中 p 的范围从 1 到 `npiece[i,j]-1`。与之前一样，斜率的数量比断点多一个。

为了在这些数据值中使用 AMPL 的分段线性函数表示法，我们必须提供断点和斜率的索引列表，而不是前一个示例中的显式列表。这是通过在斜率和断点值前面放置索引表达式来实现的：

```AMPL
minimize 总成本 (Total Cost):
    sum {i in 起始地 (ORIG), j in 目的地 (DEST)}
        <<{p in 1..npiece[i,j]-1} limit[i,j,p]; {p in 1..npiece[i,j]} rate[i,j,p]>> Trans[i,j];
```

同样，模型的其余部分是相同的。图 17-3a 显示了整个模型，图 17-3b 说明了如何指定数据。请注意，由于 `npiece["PITT","STL"]` 是 1，因此 `Trans["PITT","STL"]` 只有一个斜率且没有断点；这意味着在目标函数中，`Trans["PITT","STL"]` 是一个单段线性项。

# 17.2 常见的两段和三段项

简单的分段线性项在其他线性模型中有各种用途。在本节中，我们介绍三种情况：允许有限违反约束、分析不可行性以及表示在负值和正值水平下均有意义的变量的成本。

# “软”约束的惩罚项

线性规划最容易表达“硬”约束：例如，生产必须至少达到某个水平，或者使用的资源不得超过可用资源。实际情况往往没有这么明确。生产和资源使用可能

set 起始地 (ORIG); # 起始地 (origins)  
set 目的地 (DEST); # 目的地 (destinations)  
param 供应量 (supply) {起始地 (ORIG)} >= 0; # 起始地的供应量 (amounts available at origins)  
param 需求量 (demand) {目的地 (DEST)} >= 0; # 目的地的需求量 (amounts required at destinations)  
check: sum {i in 起始地 (ORIG)} 供应量 (supply)[i] = sum {j in 目的地 (DEST)} 需求量 (demand)[j];  
param npiece {起始地 (ORIG), 目的地 (DEST)} integer >= 1;  
param rate {i in 起始地 (ORIG), j in 目的地 (DEST), p in 1..npiece[i,j]} >= if p = 1 then 0 else rate[i,j,p-1];  
param limit {i in 起始地 (ORIG), j in 目的地 (DEST), p in 1..npiece[i,j]-1} > if p = 1 then 0 else limit[i,j,p-1];  
var Trans {起始地 (ORIG), 目的地 (DEST)} >= 0; # 运输量 (units to be shipped)  
minimize 总成本 (Total Cost):  
sum {i in 起始地 (ORIG), j in 目的地 (DEST)}  
  sum {p in 1..npiece[i,j]} rate[i,j,p] * (Trans[i,j] - if p = 1 then 0 else limit[i,j,p-1]);  
subject to Supply {i in 起始地 (ORIG)}: sum {j in 目的地 (DEST)} Trans[i,j] = 供应量 (supply)[i];  
subject to Demand {j in 目的地 (DEST)}: sum {i in 起始地 (ORIG)} Trans[i,j] = 需求量 (demand)[j];  

在某些情况下，我们对某些变量的取值有偏好水平，但允许通过接受额外成本或减少利润来违反这些水平。这种“软”约束可以通过在目标函数中添加分段线性“惩罚”项来建模。

例如，我们回到第 4 章中开发的多周期生产模型。如图 4-4 所示，约束条件表明，在第 1 周到第 T 周的每一周中，用于生产所有产品的总工时不得超过可用工时：

假设在实际中，每周可以使用更多的工时，但超出部分将对总利润造成每小时的惩罚。具体而言，我们将参数 avail[t] 替换为两个可用性水平和一个每小时惩罚率：

param avail_min {1..T} >= 0;  
param avail_max {t in 1..T} >= avail_min[t];  
param time_penalty {1..T} >= 0;  

在第 t 周，最多可以无惩罚地使用 avail_min[t] 小时，而最多可以使用 avail_max[t] 小时，超过 avail_min[t] 的每小时将减少 time_penalty[t] 的利润。

为建模这种情况，我们引入一个新变量 Use[t] 来表示生产所使用的工时。显然，Use[t] 不得小于零，也不得大于 avail_max[t]。取代之前的约束条件，我们现在规定用于生产所有产品的总工时必须等于 Use[t]：

```ampl
var Use {t in 1..T} >= 0, <= avail_max[t];
subject to Time {t in 1..T}: sum {p in PROD} (1/rate[p]) * Make[p,t] = Use[t];
```

现在我们可以用这个新变量来描述每小时的惩罚。如果 Use[t] 在 0 到 avail_min[t] 之间，则无惩罚；如果 Use[t] 在 avail_min[t] 和 avail_max[t] 之间，则每小时的惩罚为 time_penalty[t]。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a53df94a249a2c047e5d65fb152bc46b477349a681eeee6b27ec6659b2b4782f.jpg)  
图 17-3b：分段线性模型的数据 (transpl2.dat)。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/035e45b527dcbed97c0760a3f801af85889fe494aaa9880dbea9800b243ffc4e.jpg)  
图 17-4：用于工时的分段线性惩罚函数。

即超过 avail_min[t]。也就是说，惩罚是 Use[t] 的分段线性函数，如图 17-4 所示，在 avail_min[t] 处有一个断点，其两侧的斜率分别为 0 和 time_penalty[t]。使用前面介绍的语法，我们可以将目标函数的表达式重写为：

最大化 Net_Profit：  
```ampl
sum {p in PROD, t in 1..T} (revenue[p,t]*Sell[p,t] - prodcost[p]*Make[p,t]) - sum {t in 1..T} <<avail_min[t]; 0, time_penalty[t];> Use[t];
```

第一个求和项与之前相同的总利润表达式，而第二个求和项则是所有周次上的分段线性惩罚函数之和。在 `< <` 和 `> >` 之间是断点 avail_min[t] 以及周围的斜率列表，即 0 和 time_penalty[t]；随后是变量 Use[t]。

完整的修订模型如图 17-5a 所示，第 4 章中的小数据集增加了新的可用资源和惩罚项后，如图 17-5b 所示。在最优解中，我们发现使用的工时如下所示：

<table><tr><td>appl: model steelpl1.mod; data steelpl1.dat; solve;
MINOS 5.5: optimal solution found.
21 iterations, objective 457572.8571</td></tr></table>

<table><tr><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;nl&gt;</td></tr><tr><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;nl&gt;</td></tr><tr><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;nl&gt;</td></tr><tr><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;fcel&gt;</td><td>&lt;nl&gt;</td></tr><tr><td>&lt;ecel&gt;</td><td></td><td></td><td></td></tr></table>

在第 1 周和第 3 周，我们仅使用无惩罚的可用工时；而在第 2 周和第 4 周，我们也使用了受惩罚的工时。分段线性规划的解通常表现为这种形式，其中许多（尽管不一定全部）变量会“卡”在某个断点上。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/3474a91dd963d24bdfba4d6f751c75a2397623bfa4429b406d372af30bb5ce46.jpg)  
图 17-5a：带惩罚函数的分段线性目标函数 (steelpl1.mod)。

# 处理不可行性

图 17-5b 中的参数 commit[p,t] 表示每种产品每周的最低生产量。如果我们修改数据以提高这些承诺值：

```ampl
param commit: 1 2 3 4 :=
bands 3500 5900 3900 6400
coils 2500 2400 3400 4100 ;
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/2a5b5eacaea99d5b5b87aaf45f297f09c26e5fc68f11eb18f61ae6241bda323e.jpg)  
图 17-5b：图 17-5a 的数据 (steelpl1.dat)。

那么将没有足够的工时来生产这些最低数量的产品，求解器会报告问题不可行：

```python
ampl: model steelpl1.mod;
ampl: data steelpl2.dat;
ampl: solve;
MINOS 5.5: infeasible problem.
13 iterations
```

在返回的解中，最后一期的线圈 (coils) 库存为负数：

```python
ampl: option display_1col 0;
ampl: display Inv;
Inv [*,*] (tr) : bands coils :=
                0    10      0
                1     0    937
                2     0    287
                3     0      0
                4     0  -2700
;
```

并且在多个时期内，线圈的生产量低于最低要求：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/515add97e9e3b7f45cef643747aa3ba0404193de1ab8a2c0d3d6861d8fa3029a.jpg)  
图 17-6：工时使用量的惩罚函数，包含两个断点。

<table>
<tr><td>ampl:</td><td>display</td><td colspan="3">commit, Make, market;</td></tr>
<tr><td></td><td>commit</td><td>Make market</td><td>:=</td><td></td></tr>
<tr><td>bands 1</td><td>3500</td><td>3490</td><td>6000</td><td></td></tr>
<tr><td>bands 2</td><td>5900</td><td>5900</td><td>6000</td><td></td></tr>
<tr><td>bands 3</td><td>3900</td><td>3900</td><td>4000</td><td></td></tr>
<tr><td>bands 4</td><td>6400</td><td>6400</td><td>6500</td><td></td></tr>
<tr><td>coils 1</td><td>2500</td><td>3437</td><td>4000</td><td></td></tr>
<tr><td>coils 2</td><td>2400</td><td>1750</td><td>2500</td><td></td></tr>
<tr><td>coils 3</td><td>3400</td><td>2870</td><td>3500</td><td></td></tr>
<tr><td>coils 4</td><td>4100</td><td>1400</td><td>4200</td><td></td></tr>
<tr><td>;</td><td></td><td></td><td></td><td></td></tr>
</table>

这些都是求解器返回的不可行结果的典型表现。这些不可行性分散在解的各处，因此很难判断需要做出哪些更改才能实现可行性。通过扩展惩罚的概念，我们可以更好地将不可行性集中在易于理解的地方。

假设我们希望从工时短缺的角度来看待不可行性。设想我们将图 17-4 的分段线性惩罚函数扩展为图 17-6 所示的形式。现在 Use[t] 可以超过 avail_max[t]，但每超过一小时将面临极其陡峭的惩罚，因此解只会在绝对必要的情况下才会使用超过 avail_max[t] 的工时。

在 AMPL 中，新的惩罚函数通过以下更改引入：

var Use {t in 1..T} >= 0;

maximize Total_Profit:
    sum {p in PROD, t in 1..T} (
        revenue[p,t]*Sell[p,t] - 
        prodcost[p]*Make[p,t] - 
        invcost[p]*Inv[p,t]
    ) - 
    sum {t in 1..T} <<avail_min[t], avail_max[t]; 0, time_penalty[t], 100000>> Use[t];

原来的上界 avail_max[t] 现在变成了一个断点，并在其右侧引入了一个非常大的斜率 100,000。现在我们得到了一个可行解，其工时使用情况如下：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/328fbf4ea7acf9fb708eb0e810556f49ec0d667ccfc2fc9620cc7c9565cadac4.jpg)  
图 17-7：销售的惩罚函数。

ampl: model steelpl2.mod;
data steelpl2.dat;
solve;
MINDS 5.5: optimal solution found.
19 iterations, objective -1576814.857

ampl: display avail_max, Use;
: avail_max Use :=
1    42     42
2    42     42
3    40     41.7357
4    42     61.2857
;
```

该表表明，只有在最后一周增加约 21 个工时，才能满足所有承诺。

或者，我们可以从承诺过多的角度来看待不可行性。为此，当 $\mathsf{S}\in \mathsf{I}\mathsf{I}[\mathsf{p},\mathsf{t}]$ 低于 commit[p,t] 时，我们从目标函数中为每一单位减去一个非常大的惩罚值；该惩罚值作为 $\mathsf{S}\in \mathsf{I}\mathsf{I}[\mathsf{p},\mathsf{t}]$ 的函数如图 17-7 所示。

由于该函数在 commit[p,t] 处有一个断点，其右侧斜率为 0，左侧为一个非常负的值，因此 AMPL 的表示形式似乎可以为 <<commit[p,t]; -100000, 0>> Sell[p,t]。

但请记住，AMPL 的约定是此类函数在零处取值为零。图 17-7 明确显示我们希望惩罚函数在零处取正值，以便在 commit[p,t] 及其之后降为零。实际上，我们希望函数在零处取值为 100000 * commit[p,t]，我们可以通过将此常数添加到惩罚表达式中来正确表示该函数：

$$
\ll \mathsf{commit}[\mathsf{p},\mathsf{t}]; -100000, 0 >> \mathsf{Sell}[\mathsf{p},\mathsf{t}] + 100000 \times \mathsf{commit}[\mathsf{p},\mathsf{t}]
$$

通过使用第二个参数明确说明分段线性函数应在何处取零值，可以更简洁地表达相同内容：

$$
\ll \mathsf{commit}[\mathsf{p},\mathsf{t}]; -100000, 0 >> (\mathsf{Sell}[\mathsf{p},\mathsf{t}], \mathsf{commit}[\mathsf{p},\mathsf{t}])
$$

这表示函数应在 commit[p,t] 处为零，如图 17-7 所示。在完整模型中，我们有：

var Sell {p in PROD, t in 1..T} >= 0, <= market[p,t];  
maximize Total_Profit:  
sum {p in PROD, t in 1..T} (revenue[p,t] * Sell[p,t] - prodcost[p] * Make[p,t] - invcost[p] * Inv[p,t])  
- sum {t in 1..T} <<avail_min[t]; 0, time_penalty[t] >> Use[t]  
- sum {p in PROD, t in 1..T} <<commit[p,t]; -100000, 0>> (Sell[p,t], commit[p,t]);

模型的其余部分与图 17-5a 中的相同。请注意，Sell[p,t] 出现在目标函数的线性项和分段线性项中，AMPL 会自动识别这些项的总和也是分段线性的。

使用相同数据的此版本产生如下销售量的解：

mpl: model steelpl3.mod; data steelpl2.dat; solve;  
MINOS 5.5: optimal solution found. 24 iterations, objective -293856347  
mpl: display Sell, commit;  
:    Sell   commit    :=  
bands  1   3500    3500  
bands  2   5900    5900  
bands  3   3900    3900  
bands  4   6400    6400  
coils  1      0    2500  
coils  2   2400    2400  
coils  3   3400    3400  
coils  4   3657    4100  

为了在给定工时内完成任务，第一周线圈的交付承诺减少了 2500 吨，第四周减少了 443 吨。

# 可逆活动

本书中几乎所有 线性规划 (linear program) 都是用非负 变量 (variable) 来建模的。但有时一个 变量 (variable) 在取负值和正值时都有意义，而在许多这样的情况下，相关 成本 (cost) 是分段线性的，且在零点处有一个断点。

图 17-5a 中的 库存 (Inv) 变量 (variable) 提供了一个例子。我们定义 `Inv[p,t]` 表示在第 `t` 周 (week) 结束时产品 `p` 的库存吨数。也就是说，在第 `t` 周之后，有 `Inv[p,t]` 吨产品 `p` 已生产但未销售。因此，`Inv[p,t]` 的负值可以合理地解释为表示已销售但未生产的吨数 —— 实际上就是缺货吨数。在这种解释下，物料平衡 约束 (constraint) 仍然有效：

subject to Balance {p in PROD, t in 1..T}:  
Make[p,t] + Inv[p,t-1] = Sell[p,t] + Inv[p,t];

这种分析表明，我们应该从模型中 `Inv` 的声明中移除 `$>= 0$`。如果预期未来几周销售价格会下降，则缺货可能特别有吸引力，如下所示：

<table><tr><td>param revenue:</td><td>1</td><td>2</td><td>3</td><td>4</td><td>:=</td></tr><tr><td>bands</td><td>25</td><td>26</td><td>23</td><td>20</td><td></td></tr><tr><td>coils</td><td>30</td><td>35</td><td>31</td><td>25</td><td>/</td></tr></table>

当我们使用适当修改后的模型和数据文件重新求解时，结果却不是我们所预期的：

ampl: model steelpl4.mod; data steelpl4.dat; solve;  
MINDS 5.5: optimal solution found.  
15 iterations, objective 1194250.  
ampl: display Make,Inv,Sell;  
Make Inv Sell  
: bands 0 10  
bands 1 0 - 5990 6000  
bands 2 0 - 11990 6000  
bands 3 0 - 15990 4000  
bands 4 0 - 22490 6500  
coils 0 0  
coils 1 0 - 4000 4000  
coils 2 0 - 6500 2500  
coils 3 0 - 10000 3500  
coils 4 0 - 14200 4200  

问题的根源在于目标函数中从销售收入中减去了 `invcost[p] * Inv[p,t]`。当 `Inv[p,t]` 为负值时，减去的是一个负数，从而增加了表面总利润。缺货量越大，总利润增加得越多 —— 因此出现了奇怪的解：最大可能的销售量被缺货，而没有生产任何产品！

对于该模型，一个适当的库存成本函数如图 17-8 所示。当 `Inv[p,t]` 变得更正（库存增加）或变得更负（缺货增加）时，它都会增加。我们在 AMPL 中通过声明缺货成本来表示这个分段线性函数，与库存成本一起：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/d2817adfa03f7733753938b8e525d729e562e976b8659bd1d9a7ecad4294b8ed.jpg)  
图 17-8: 库存成本函数。

param invcost {PROD} >= 0;  
param backcost {PROD} >= 0;

然后，目标函数中 `Inv[p,t]` 项的斜率分别为 `-backcost[p]` 和 `invcost[p]`，断点在零处，正确的目标函数为：

最大化 总利润 (Total_Profit)：  
\[
\sum_{p \in PROD, t \in 1..T} \left( revenue[p,t] \cdot Sell[p,t] - prodcost[p] \cdot Make[p,t] - \langle\langle 0; -backcost[p], invcost[p] \rangle\rangle Inv[p,t] \right) - \sum_{t \in 1..T} \langle\langle avail\_min[t]; 0, time\_penalty[t] \rangle\rangle Use[t];
\]

与我们的第一个例子不同的是，这里的分段线性函数是被减去而不是加上。但结果仍然是分段线性的；这等价于我们加上了表达式 \(\langle\langle 0; backcost[p], -invcost[p] \rangle\rangle Inv[p,t]\)。

当我们做出这一修改，并在数据中加入一些缺货成本 (backcost) 后，得到了一个看起来更合理的解。然而，仍存在一种倾向，即在后期不生产任何产品并将所有订单推迟交付；这是一种“末端效应”，其发生的原因是模型没有考虑到在最后一期之后延期生产的实际成本。作为快速修正，我们可以通过添加一个约束条件来排除期末的任何未交付订单，即规定最后一周的库存必须为非负数：

```ampl
subject to Final {p in PROD}: Inv[p,T] >= 0;
```

在加入该约束并设定带钢 (bands) 的 backcost 值为 1.5、线圈 (coils) 的 backcost 值为 2 后进行求解：

- impl: model steelpl5.mod; data steelpl5.dat; solve;
- MINOS 5.5: optimal solution found.
- 20 iterations, objective 370752.8571
- impl: display Make, Inv, Sell;
- : Make Inv Sell :=
  - bands 0 . 10 .
  - bands 1 4142.86 0 4152.86
  - bands 2 6000 0 6000
  - bands 3 3000 0 3000
  - bands 4 3000 0 3000
  - coils 0 . 0 .
  - coils 1 2000 0 2000
  - coils 2 1680 -820 2500
  - coils 3 2100 -800 2080
  - coils 4 2800 0 2000

根据这个计划，大约有 800 吨线圈将在第 2 周和第 3 周延迟一周交付。

# 17.3 其他分段线性函数

许多简单的分段线性函数在 AMPL 中可以用几种等效方式建模。例如图 17-4 所示的函数可以写成：

```ampl
if Use[t] > avail_min[t] then time_penalty[t] * (Use[t] - avail_min[t]) else 0
```

或者更简洁地写成：

```ampl
max(0, time_penalty[t] * (Use[t] - avail_min[t]))
```

当前版本的 AMPL 无法识别这些表达式为分段线性函数，因此如果你尝试在一个目标函数中包含类似表达式的模型进行求解，很可能得不到满意的结果。为了利用可应用于分段线性项的线性规划技术，你需要使用分段线性语法：

\[
\langle\langle avail\_min[t]; 0, time\_penalty[t] \rangle\rangle Use[t]
\]

以便识别这种结构并将其传递给求解器。

同样的建议也适用于函数 abs。假设我们希望鼓励使用的工时尽可能接近 avail_min[t]，那么我们希望惩罚项等于 time_penalty[t] 乘以 Use[t] 相对于 avail_min[t] 的偏差量，无论是高于还是低于。这样的项可以写作：

\[
time\_penalty[t] \cdot abs(Use[t] - avail\_min[t])
\]

然而，若要以显式分段线性 (piecewise-linear) 的方式表达，你应该将其写为：

\[
<<avail\_min[t]; -time\_penalty[t], time\_penalty[t]>> Use[t]
\]

或等价地：

```ampl
<<avail_min[t]; - time_penalty[t], time_penalty[t] >> Use[t]
```

如此例所示，将一个分段线性函数乘以一个常数，等同于将它的每一段斜率分别乘以该常数。

作为目标函数中常见分段线性形式的最后一个例子，我们回到第 15 章中讨论过的那种指派模型 (assignment model)。回忆一下，对于集合 PEOPLE 中的 i 和集合 PROJECTS 中的 j，cost[i,j] 表示人员 i 在项目 j 上工作一小时的成本，而决策变量 Assign[i,j] 则表示人员 i 被指派到项目 j 上工作的小时数：

set PEOPLE;  
set PROJECTS;  
param cost {PEOPLE,PROJECTS} >= 0;  
var Assign {PEOPLE,PROJECTS} >= 0;

我们最初将目标函数设定为所有指派任务的总成本：

$$\sum_{i \in PEOPLE, j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$$

如果我们希望得到的是最公平的指派方案，而非成本最低的呢？那么我们可以最小化任何一个人所承担任务的最大成本：

set PEOPLE;  
set PROJECTS;  
param supply {PEOPLE} >= 0; # 每个人可工作的时间（小时）  
param demand {PROJECTS} >= 0; # 每个项目所需的时间（小时）  
check: $\sum_{i \in PEOPLE} supply[i] = \sum_{j \in PROJECTS} demand[j]$;  
param cost {PEOPLE,PROJECTS} >= 0; # 每小时工作的成本  
param limit {PEOPLE,PROJECTS} >= 0; # 对项目的最大贡献  
var M;  
var Assign {i in PEOPLE, j in PROJECTS} >= 0, <= limit[i,j];  
minimize Max_Cost: M;  
subject to M_def {i in PEOPLE}  
 M >= $\sum_{j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$;  
subject to Supply {i in PEOPLE}:  
 $\sum_{j \in PROJECTS} Assign[i,j] = supply[i]$;  
subject to Demand {j in PROJECTS}:  
 $\sum_{i \in PEOPLE} Assign[i,j] = demand[j]$;

图 17-9：最小化最大值的指派模型 (minmax.mod)。

minimize Max_Cost: max {i in PEOPLE} $\sum_{j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$;

这个函数在某种意义上也是分段线性的；它由不同人员 i 的线性函数 $\sum_{j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$ 组合而成。然而，它并非在各个变量上分别具有分段线性性质——用数学术语来说，它不可分离 (not separable)——因此不能使用 $<<\ldots>>$ 符号来表示。

这是只能通过将模型重写为线性规划 (linear program) 来处理分段线性的情况之一。我们引入一个新变量 M 来表示最大值。然后，我们添加约束条件，以确保 M 大于或等于它所代表的最大值中的每一项：

var M;  
minimize Max_Cost: M;  
subject to M_def {i in PEOPLE}  
 M >= $\sum_{j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$;

由于 M 被最小化，在最优解中它实际上等于所有 i 属于 PEOPLE 的 $\sum_{j \in PROJECTS} cost[i,j] \cdot Assign[i,j]$ 的最大值。其他约束条件与任何指派问题中的约束相同，如图 17-9 所示。

这种重新表述可以应用于任何具有 “min-max” 目标的問題。同样的想法也适用于类似的 “max-min” 目标，只需将 minimize 替换为 maximize，并在约束中使用 $M \leq \ldots$。

# 17.4 分段线性优化的指导原则

AMPL 的分段线性记号能够指定各种有用的函数。我们在下面总结了其各种形式，其中大多数已在本章前面的部分中举例说明。

由于这种记号非常通用，它可以用来指定许多无法通过任何高效且可靠的算法轻松优化的函数。我们最后描述了最有可能出现在可处理模型中的那种分段线性函数，特别强调凸性或凹性这一性质。

# 分段线性表达式的形式

一个 AMPL 分段线性项具有一般形式：

```
<<breakpoint-list; slope-list>> pl-argument
```

其中 `breakpoint-list` 和 `slope-list` 各自由一个或多个由逗号分隔的项组成。一个项可以是一个单独的算术表达式，或者是一个索引表达式后跟一个算术表达式。在后一种情况下，索引表达式必须是一个有序集合；该项通过为每个集合成员计算一次算术表达式来展开为一个列表（如图 17-3a 的示例所示）。

在任何索引项展开之后，斜率的数量必须比断点数量多一个，并且断点必须是非递减的。生成的分段线性函数通过按照给定顺序交错斜率和断点来构造，第一个斜率位于第一个断点的左侧，最后一个斜率位于最后一个断点的右侧。通过在空集上索引断点，可以指定没有断点且只有一个斜率的情况，此时函数是线性的。

`pl-argument` 可以具有以下形式之一：

```
var-ref (arg-expr) (arg-expr, zero-expr)
```

`var-ref`（对先前声明变量的引用）或 `arg-expr`（一个算术表达式）指定对分段线性函数进行求值的点。`zero-expr` 是一个指定函数为零位置的算术表达式；当省略 `zero-expr` 时，假定函数在零处为零。

# 分段线性模型的建议

正如我们在所有示例中看到的那样，AMPL 对变量的分段线性函数的术语仅限于描述单个变量的函数。在模型声明中，`breakpoint-list`、`slope-list` 和 `zero-expr`（如果有）中不得出现变量，而 `arg-expr` 只能是对单个变量的引用。（然而，在 `display` 等命令中的分段线性表达式可以无限制地使用变量。）

一个单变量的分段线性函数在乘以或除以一个不含变量的算术表达式后，仍然是这样的函数。当同一个变量的分段线性函数与线性函数相加或相减时，AMPL 也将其视为该变量的一个分段线性函数。模型变量的一个可分 (separable) 分段线性函数是各个变量的分段线性函数或线性函数通过 $^+$、- 或 sum 运算得到的和或差。优化器可以有效地处理这些可分函数，而这些函数也正是我们示例中出现的那些。

如果一个分段线性函数的相邻斜率是非递减的（包括断点），则该函数是凸的；如果斜率是非递增的，则该函数是凹的。最容易被求解器处理的两类分段线性优化问题是：在满足线性约束条件下，最小化一个可分的凸分段线性函数，或最大化一个可分的凹分段线性函数。你可以很容易地验证本章所有示例都属于这两类问题。在这些情况下，AMPL 可以通过将其转换为一个等价的线性规划 (LP) 问题，应用任何 LP 求解器求解，然后再将解转换回来的方式获得解；当你输入 `solve` 命令时，整个过程会自动进行。

在上述两种情况之外，优化一个可分的分段线性函数必须视为整数规划（第 20 章的主题）的一个应用，而 AMPL 必须将分段线性项转换为等价的整数规划形式。同样，这也是自动完成的，以便由合适的求解器进行求解。然而，由于整数规划通常比类似规模的线性规划要难得多，因此你不应假设任何可分的分段线性函数都能轻易地被优化；可能需要通过实验来确定你的求解器能够处理的实例规模。最好的结果可能来自于接受一种被称为“二阶特殊有序集 (special ordered sets of type 2)”选项的求解器；详情请查阅与求解器相关的文档。

约束的情况可以用类似的方式描述。然而，只有在非常有限的情况下，约束中的可分分段线性函数才能通过线性规划进行处理：

如果它是凸函数，并且位于 $\leq$ 约束的左侧（或等价地，位于 $\geq$ 约束的右侧）；  
如果它是凹函数，并且位于 $\geq$ 约束的左侧（或等价地，位于 $\leq$ 约束的右侧）。

约束中的其他分段线性项必须通过整数规划技术来处理，前述针对目标函数的评论同样适用。

如果你使用的求解器能够直接处理分段线性项，你可以通过设置以下选项来关闭 AMPL 向线性规划或整数规划形式的转换：

将选项 pl_linearize 设为 0。在最小化凸函数或最大化凹的可分离分段线性函数的情况下，可以通过分段线性的 LP (Linear Program，线性规划) 技术的推广非常高效地处理。一个用于非线性规划的求解器 (solver) 也可能接受分段线性函数，但除非它专门设计用于“不可微”优化，否则不太可能可靠地处理它们。

困难和容易的分段线性情况之间的差异可能很细微。本章的运输示例是容易的，特别是因为运输费率随着运输量的增加而增加。如果规模经济导致运输费率随着运输量的增加而降低，那么同样的例子就会变得困难，因为这时我们将最小化一个凹函数而不是凸函数。我们不能明确地说运输费率应该是哪种方式；它们的行为取决于所建模情况的具体细节。

在所有情况下，分段线性优化的难度随着片段总数的增加而逐渐增加。因此，当成本可以用相对较少的片段来描述或近似时，分段线性成本函数最为有效。如果你需要超过大约十几个片段来准确描述成本，那么使用第 18 章中描述的非线性函数可能更好。

# 参考文献

[1] Robert Fouer. “The simplex algorithm for piecewise linear programming III: computational analysis and applications.” Mathematical Programming 53 (1992) 213–235. A survey of the conversion of piecewise linear programs to linear programs, and of applications.

罗伯特·福尔 和 罗伊·E. 马斯顿，“求解分段线性程序：单纯形方法的实验。”《ORSA 计算杂志》4 (1992) 第 16-31 页。对各种应用的描述以及求解经验。

斯皮罗斯·孔托吉奥吉斯，“单调优化的实用分段线性近似。”《INFORMS 计算杂志》12 (2000) 第 324-340 页。关于通过分段线性函数近似非线性函数时选择断点的指南。

# 练习

17-1. 分段线性模型有时是第 18 章中描述的非线性模型的替代方案，用一系列直线段代替平滑曲线。本练习处理图 18-4 中所示的模型。

(a) 重新制定图 18-4 的模型，使其近似每个非线性项

$$
\mathtt{Trans}[\mathtt{i},\mathtt{j}] / (\mathtt{l} - \mathtt{Trans}[\mathtt{i},\mathtt{j}] / \mathtt{limit}[\mathtt{i},\mathtt{j}])
$$

通过一个具有三段的分段线性项来近似。将断点设置在 $(1 / 3) * \mathtt{limit}[\mathtt{i}, \mathtt{j}]$ 和 $(2 / 3) * \mathtt{limit}[\mathtt{i}, \mathtt{j}]$ 处。选择斜率，使得当 $\mathtt{Trans}[\mathtt{i}, \mathtt{j}]$ 为 $0$、$1 / 3 * \mathtt{limit}[\mathtt{i}, \mathtt{j}]$、$2 / 3 * \mathtt{limit}[\mathtt{i}, \mathtt{j}]$ 或 $11 / 12 * \mathtt{limit}[\mathtt{i}, \mathtt{j}]$ 时，该近似等于原始非线性项；你应该会发现，无论 $\mathtt{limit[i,j]}$ 的大小如何，这三项的斜率都是 $3 / 2$、$9 / 2$ 和 $36$。最后，对 $\mathtt{Trans[i,j]}$ 施加一个显式的上限 $0.99 * \mathtt{limit[i,j]}$。

(b) 使用图 18-5 中给出的数据求解该近似模型，并将最优运输量与非线性模型推荐的数量进行比较。

(c) 制定一个更复杂的版本，其中每个项的线性段数由参数 nsl 给出。将断点设置在 $(\mathtt{k} / \mathtt{ns}1)$ * limit[i,j]，其中 k 从 1 到 nsl-1。选择斜率，使得当 Trans[i,j] 为 $(\mathtt{k} / \mathtt{ns}1)$ * limit[i,j]（k 从 0 到 nsl-1）或当 Trans[i,j] 为 (nsl-1/4)/nsl * limit[i,j] 时，分段线性函数等于原始非线性函数。

通过验证当 nsl 为 3 时是否得到与 (b) 中相同的结果来检查你的模型。然后，通过尝试更高的 nsl 值，确定需要多少线性段才能使所有运输量的近似值与原始非线性模型推荐的数量相差在约 $10\%$ 以内。

17-2. 本练习探讨如何将图 3-1a 中运输模型的需求约束转换为第 17.2 节中描述的“软”约束。

假设在每个目的地 j 处，不是给定一个称为 demand[j] 的单一参数，而是给定以下四个参数来描述一个更复杂的情况：

dem_min_abs[j] 必须运送到 j 的绝对最小值  
dem_min_ask[j] 偏好的最小运输量至 j  
dem_max_ask[j] 偏好的最大运输量至 j  
dem_max_abs[j] 可运送到 j 的绝对最大值  

此外，对于超出偏好限制的运输量，还有两种惩罚成本：

dem_min_pen 每单位运输量低于 dem_min_ask[j] 的惩罚  
dem_max_pen 每单位运输量超过 dem_max_ask[j] 的惩罚

由于运送到 j 的总量不再固定，引入了一个新变量 Receive[j] 来表示在 j 处接收到的数量。

(a) 修改图 3-1a 中的模型以使用这些新信息。修改将包括声明 Receive[j] 并设定适当的下界和上界，在目标函数中添加一个三段分段线性惩罚项，并在约束中用 Receive[j] 替换 demand[j]。

(b) 将以下需求信息添加到图 3-1b 的数据中：

<table><tr><td></td><td>dem_min_abs</td><td>dem_min_ask</td><td>dem_max_ask</td><td>dem_max_abs</td></tr><tr><td>法兰克福 (FRA)</td><td>800</td><td>850</td><td>950</td><td>1100</td></tr><tr><td>底特律 (DET)</td><td>975</td><td>1100</td><td>1225</td><td>1250</td></tr><tr><td>兰辛 (LAN)</td><td>600</td><td>600</td><td>625</td><td>625</td></tr><tr><td>温莎 (WIN)</td><td>350</td><td>375</td><td>450</td><td>500</td></tr><tr><td>圣路易斯 (STL)</td><td>1200</td><td>1500</td><td>1800</td><td>2000</td></tr><tr><td>弗雷斯诺 (FRE)</td><td>1100</td><td>1100</td><td>1100</td><td>1125</td></tr><tr><td>拉斐特 (LAF)</td><td>800</td><td>900</td><td>1050</td><td>1175</td></tr></table>

令 dem_min_pen 和 dem_max_pen 分别为 2 和 4。找出新的最优解 (optimal solution)。在该解中，哪些目的地 (destinations) 的运输量 (shipments) 超出了偏好的水平？

17-3. 当使用图 2-3 的数据运行图 2-1 的饮食模型时，不存在可行解。本题要求你使用 17.2 节的思想来寻找一些良好的近似可行解。

(a) 修改模型，使其在付出极高惩罚代价的情况下，可以购买超过指定最大值的食物。在得到的解中，哪些最大值被超过了？
(b) 修改模型，使其在付出极高惩罚代价的情况下，可以供应超过指定最大值的营养成分。在得到的解中，哪些最大值被超过了？
(c) 使用像 $10^{20}$ 这样极大的惩罚系数可能会给求解器 (solver) 带来数值上的困难。通过实验观察，当你使用像 $10^{20}$ 和 $10^{30}$ 这样的惩罚项时，现有的求解器表现如何。

17-4. 在练习 4-4(b) 的模型中，从一个时期到下一个时期的人员变化被限制在某个数量 $M$ 以内。作为施加此限制的替代方法，假设我们引入一个新的变量 (variable) $D_{t}$，它表示在时期 $t$ 内人员（所有班次）数量的变化。这个变量可以为正值，表示人员数量比上一时期有所增加，也可以为负值，表示人员减少。

为了使用这个变量，我们引入一个定义约束 (constraint)：

$$
\begin{array}{r}
D_{t} = \sum_{s\in S}(Y_{st} - Y_{s,t - 1}),
\end{array}
$$

对于每个 $t = 1,\ldots ,T$。然后我们估计每增加一名人员从一个时期到下一个时期的成本 (cost) 为 $c^{+}$，每减少一名人员的成本为 $c^{-}$；因此，对于每个月 $t$，必须在目标函数中包含以下成本：

$$
\begin{array}{rl}
{c^{-}D_{t},} & {\mathrm{if}\ D_{t}< 0;}\\ 
{c^{+}D_{t},} & {\mathrm{if}\ D_{t} > 0.}
\end{array}
$$

在 AMPL 中相应地重新构建模型，使用分段线性函数来表示这种额外成本。

使用 $c^{-} = -20000$ 和 $c^{+} = 100000$，结合之前给定的数据进行求解。该解与练习 4-4(b) 的解相比如何？

17-5. 下面这个 “信用评分” 问题出现在许多场景中，包括信用卡申请的处理。一组申请人 APPL 填写申请表，每人需回答一组问题 QUES。申请人 $i$ 对问题 $j$ 的回答会被转换为一个数字 ans[i,j]；典型的数字包括在当前地址居住的年数、月收入，以及住房拥有情况指标 （例如，拥有住房为 1，否则为 0）。

为了总结这些回答，发卡机构选择一组权重 Wt[j]，然后通过线性公式为 APPL 中的每个申请人 $i$ 计算得分：

发卡机构还会选择一个阈值 Cut；当申请人的得分大于或等于该阈值时授予信用，得分小于该阈值时则拒绝。通过这种方式，决策可以客观地进行 （尽管未必总是最明智的）。

为了选择权重和阈值，发卡机构收集一组先前已接受的申请样本，其中一些来自最终成为优质客户的人，另一些来自从未付款的人。如果我们用集合 GOOD 和 BAD 表示这两组人，则理想权重和阈值 （针对这些数据） 应满足：

然而，由于申请回答与信用状况之间的关系充其量是模糊的，因此无法找到能满足所有这些不等式的 Wt[j] 和 Cut 的值。取而代之的是，

发卡机构必须选择某种意义上尽可能最佳的值。有无数种方法可以做出这种选择；这里，我们自然考虑一种优化方法。

(a) 假设我们定义一个新变量 Diff[i]，它等于申请人 $i$ 的得分与阈值之间的差：

Diff[i] = sum {j in QUES} ans[i,j] * Wt[j] - Cut

显然，不良情况是：当 $i$ 属于 GOOD 时 Diff[i] 为负，以及当 $i$ 属于 BAD 时 Diff[i] 为非负。为了抑制这些情况，我们可以要求发卡机构最小化以下函数：

sum {i in GOOD} max(0,-Diff[i]) + sum {i in BAD} max(0,Diff[i])

解释为什么最小化该函数有助于产生理想的权重和阈值选择。

(b) 上述表达式是变量 Diff[i] 的分段线性函数。请使用 AMPL 的分段线性函数表示法重新编写它。

(c) 将 (b) 中的表达式整合到一个 AMPL 模型中以求解权重和阈值。

(d) 根据这种方法，Cut 的任意正值与其他正值一样好。我们可以将其固定在一个方便的整数 —— 例如 100。解释为什么可以这样做。

(e) 使用 Cut 值为 100，将该模型应用于以下虚构的信用数据：

<table><tr><td>集合 GOOD := _17 _18 _19 _22 _24 _26 _28 _29 ;</td></tr><tr><td>集合 BAD := _15 _16 _20 _21 _23 _25 _27 _30 ;</td></tr><tr><td>集合 QUES := Q1 Q2 R1 R2 R3 S2 T4 ;</td></tr><tr><td>参数 ans: Q1 Q2 R1 R2 R3 S2 T4 :=</td></tr><tr><td>_15 1.0 10 15 20 10 8 10</td></tr><tr><td>_16 0.0 5 15 40 8 10 8</td></tr><tr><td>_17 0.5 10 25 35 8 10 10</td></tr><tr><td>_18 1.5 10 25 30 8 6 10</td></tr><tr><td>_19 1.5 5 20 25 8 8 8</td></tr><tr><td>_20 1.0 5 5 30 8 8 6</td></tr><tr><td>_21 1.0 10 20 30 8 10 10</td></tr><tr><td>_22 0.5 10 25 40 8 8 10</td></tr><tr><td>_23 0.5 10 25 25 8 8 14</td></tr><tr><td>_24 1.0 10 15 40 8 10 10</td></tr><tr><td>_25 0.0 5 15 15 10 12 10</td></tr><tr><td>_26 0.5 10 15 20 8 10 10</td></tr><tr><td>_27 1.0 5 10 25 10 8 6</td></tr><tr><td>_28 0.0 5 15 40 8 10 8</td></tr><tr><td>_29 1.0 5 15 40 8 8 10</td></tr><tr><td>_30 1.5 5 20 25 10 10 14 ;</td></tr></table>

所选择的权重是什么？使用这些权重，有多少优质客户会被拒绝发卡，又有多少高风险客户会被批准发卡？

你应该会发现许多高风险客户的评分正好处于临界值。为什么在解中会出现这种情况？你应该如何调整临界值来应对这一问题？

(f) 为了强制评分在期望方向上远离临界值，可能更倾向于使用以下目标函数：

```
sum {i in GOOD} max(0,- Diff[i]+offset) + sum {i in BAD} max(0,Diff[i]+offset)
```

其中 offset 是一个正参数，其值由用户提供。解释为什么这一改变会产生期望的效果。尝试 offset 值为 2 和 10，并将结果与 (e) 中的结果进行比较。

(g) 假设向信用风险较高的客户发放信用卡被认为比拒绝向信用良好的客户发放信用卡更不可取。你将如何修改模型以考虑这一点？

(h) 假设当某人的申请被接受时，其评分还将用于建议初始信用额度。因此，确保高信用风险客户不会获得很高的评分尤为重要。你将如何在分段线性目标函数项中添加部分以体现这一关切？

17-6. 在练习 18-3 中，我们建议通过最小化一个非线性的“平方和”函数来从不精确的数据中估计位置、速度 和加速度值：

$$
\sum_{j = 1}^{n}\left[h_{j} - (a_{0} - a_{1}t_{j}^{-1}{}_{2}a_{2}t_{j}^{2})\right]^{2}.
$$

另一种方法是最小化绝对值之和：

$$
\sum_{j = 1}^{n}\left|h_{j} - (a_{0} - a_{1}t_{j}^{-1}{}_{2}a_{2}t_{j}^{2})\right|.
$$

(a) 在练习 18-3 的模型中，直接用绝对值之和替代平方和，首先使用 abs 函数，然后使用 AMPL 的显式分段线性表示法。

解释为什么这两种公式都不太可能被任何求解器有效处理。

(b) 为了有效地对这种情况进行建模，我们引入变量 $e_j$ 来表示各个公式 $h_j - (a_0 - a_1t_j^{-1}{}_2a_2t_j^2)$，这些公式的绝对值正在被取用。然后，我们可以将绝对值之和的最小化表示为以下约束优化问题：

最小化 $\sum_{j = 1}^{n}\left|e_{j}\right|$

约束条件为 $e_j = h_j - (a_0 - a_1t_j^{- 1}{}_2a_2t_j^2), j = 1,\ldots ,n$

使用 AMPL 为该公式编写模型，对项 $\left|e_j\right|$ 使用分段线性记号。

(c) 使用练习 18-3 中的数据求解 $a_0, a_1$ 和 $a_2$。该估计值与最小二乘估计值之间有多大差异？

使用 display 打印最小二乘和最小绝对值解的 $e_j$ 值。最明显的定性差异是什么？

(d) 另一种可能性是关注最大绝对偏差，而不是总和：

$$
\max_{j = 1,\ldots ,n}\left|h_j - (a_0 - a_1t_j^{-1}{}_2a_2t_j^2)\right|.
$$

构建一个 AMPL 线性规划来最小化该量，并在相同数据上进行测试。比较所得的估计值和 $e_j$ 值。在这种情况下，你会选择三个估计值中的哪一个？

17-7. 平面结构由一组通过杆件连接的节点组成。例如，在下图中，节点由圆圈表示，杆件由两个圆圈之间的连线表示：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/3342fa9861ac7dbf20fe6c1107cb06b9decfb92e9d972ef9bd06677b9b0c8e83.jpg)

考虑寻找满足某些外部力的最小重量结构的问题。我们令 $J$ 为节点集合，$B\subseteq J\times J$ 为允许的杆件集合；对于上面的图，我们可以取 $J = \{1,2,3,4,5\}$，以及

$$
B = \{(1,2),(1,3),(1,4),(2,3),(2,5),(3,4),(3,5),(4,5)\} .
$$

杆件的“起点”和“终点”是任意的。例如，连接节点 1 和 2 的杆件可以由 (1,2) 或 (2,1) 表示，但不需要两者都表示。

我们可以使用二维欧几里得坐标来指定每个节点在平面上的位置，以某个任意点作为原点：

$a_{i}^{x}$ 节点 $i$ 相对于原点的水平位置  
$a_{i}^{y}$ 节点 $i$ 相对于原点的垂直位置  

对于该例子，如果原点恰好位于节点 2 处，我们可能有

$$
\begin{array}{rl} & (a_1^x,a_1^y) = (0,2),(a_2^x,a_2^y) = (0,0),(a_3^x,a_3^y) = (2,1),\\ & (a_4^x,a_4^y) = (4,2),(a_5^x,a_5^y) = (4,0). \end{array}
$$

其余数据由作用在节点上的外力组成：

$f_{i}^{x}$ 作用在节点 $i$ 上的外力的水平分量  
$f_{i}^{y}$ 作用在节点 $i$ 上的外力的垂直分量  

为了抵抗该力，节点的一个子集 $S\subseteq J$ 被固定在位置上。（可以证明，固定两个节点足以保证解的存在。）

外力在杆件上引起应力，我们可以将其表示为

$$
\begin{array}{rl}
{F_{ij}} & {\mathrm{若} > 0\mathrm{，表示杆件}(i,j)\mathrm{的拉力}}\\ 
& {\mathrm{若} < 0\mathrm{，表示杆件}(i,j)\mathrm{的压力}} 
\end{array}
$$

一组应力处于平衡状态，当且仅当在所有节点（除固定节点外）处，外部力、拉力与压力在水平方向和垂直方向上均平衡。即对于每个节点 $k\notin S$，有

$$
\begin{array}{rl}
& {\sum_{i\in J:(i,k)\in B}c_{ik}^{x}F_{ik} - \sum_{j\in J:(k,j)\in B}c_{kj}^{x}F_{kj} = f_{k}^{x}}\\ 
& {\sum_{i\in J:(i,k)\in B}c_{ik}^{y}F_{ik} - \sum_{j\in J:(k,j)\in B}c_{kj}^{y}F_{kj} = f_{k}^{y},} 
\end{array}
$$

其中 $c_{st}^{x}$ 和 $c_{st}^{y}$ 分别表示从节点 $s$ 到节点 $t$ 的方向与水平轴和垂直轴夹角的余弦值，

$$
\begin{array}{r}
c_{st}^{x} = (a_{t}^{x} - a_{s}^{x}) / l_{st},\\ 
c_{st}^{y} = (a_{t}^{y} - a_{s}^{y}) / l_{st}, 
\end{array}
$$

而 $l_{st}$ 是杆件 $(s,t)$ 的长度：

$$
l_{st} = \sqrt{(a_t^x - a_s^x)^2 + (a_t^y - a_s^y)^2}.
$$

通常，存在无穷多种不同的平衡应力集合。然而，可以证明：一个给定的应力系统将在结构总重量最小的情况下实现，当且仅当各杆件的横截面积与其应力的绝对值成正比。由于杆件的重量与横截面积和长度的乘积成正比，因此我们可以将杆件 $(i,j)$ 的（适当缩放后的）重量表示为

$$
w_{ij} = l_{ij}\cdot |F_{ij}|.
$$

问题的目标是寻找一组满足平衡条件的应力 $F_{ij}$，并最小化所有杆件 $(i,j)\in B$ 的重量 $w_{ij}$ 之和。

(a) 此线性规划 (Linear Program) 的索引集合可在 AMPL 中声明为：

```ampl
set joints;
set fixed within joints;
set bars within {i in joints, j in joints: i <> j};
```

使用这些集合声明，构建一个用于最小重量结构设计问题的 AMPL 模型。使用本章中的分段线性记号表示目标函数中的绝对值项。

(b) 现在特别考虑一个具有如下节点的结构：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a06f98a48f16294c009405e94fe68de50550672bc1483a9a38579aefa37ed5f3.jpg)

假设节点之间水平和垂直方向上的单位长度均为 1，且原点位于左下角；因此 $(a_{1}^{x},a_{1}^{y}) = (0,2)$，$(a_{15}^{x},a_{15}^{y}) = (5,0)$。

假设在节点 1 和节点 7 上分别施加 3.25 和 1.75 单位的竖直向下的外力，即 $f_{1}^{y} = - 3.25$，$f_{7}^{y} = - 1.75$，其余所有 $f_{i}^{x} = 0$ 且 $f_{7}^{y} = 0$。令 $S = \{6,15\}$。最后，允许的杆件包括所有不直接穿过其他节点的杆件；例如 (1, 2)、(1, 9) 或 (1, 13) 是允许的，但 (1, 3)、(1, 12) 或 (1, 14) 不允许。

确定线性规划所需的所有数据，并将其表示为 AMPL 数据语句。

(c) 使用 AMPL 求解该线性规划，并检查所确定的最小重量结构。

绘制最优结构的示意图，标明杆件的横截面以及应力的性质。如果某根杆件上的力为零，则其横截面为零，可以在图中省略。

(d) 在所有可能的杆件都允许使用的情况下，重复 (b) 和 (c) 部分的内容。所得结构是否不同？是否更轻？