# 构建更大的模型

到目前为止我们所展示的线性规划 (LP) 都相当小，因此它们的数据和解都可以放在一页纸上。然而，在实际应用中遇到的大多数线性规划都包含数百或数千个变量和约束，有些甚至更大。

线性规划是如何变得如此庞大的呢？它们可能类似于我们之前展示的模型，但使用了更大的索引集和更多的数据。例如，一个钢铁厂可能被认为生产数百种不同的产品，如果将宽度、厚度和表面处理的每种变化都单独考虑的话。或者，一个大型组织可能在一个人力分配问题中涉及数千人。然而，这类应用并不像人们预期的那样常见。随着模型被细化到更高的详细程度，其数据值变得更加难以维护，其解决方案也更难以理解；超过某个临界点后，额外的细节将不再带来收益。因此，若只为规划几条生产线，相当详细的模型或许是合理的；但若要为整个公司做规划，使用一个可以在不同场景下多次运行的小型聚合工厂级模型可能更为合适。

大型线性规划更常见的来源是将多个小型线性规划连接在一起。在实际应用中，经常会出现许多我们之前讨论过的简单 LP 问题；以下是三种可能的情形：

- 需要运输多种产品，每种产品对应一个运输问题（如第 3 章所述）。
- 需要在多个星期内进行生产规划，每周对应一个生产问题（如第 1 章所述）。
- 多种产品在多个钢铁厂生产，并运输到多个工厂；每个钢铁厂对应一个生产问题，每种产品对应一个运输问题。

当通过变量或约束将这些 LP 问题连接在一起时，结果可能是一个非常大的 LP 问题。其中的任何一部分都不需要特别详细；问题的规模更多地来源于起始地、目的地、产品和周次等元素的大量组合。

本章将展示如何为上述三种情形构建 AMPL 模型。这些模型不可避免地比我们之前的模型更加复杂，并需要使用 AMPL 语言的更多功能。然而，由于它们建立在已经介绍过的较小模型的术语和逻辑基础上，因此这些大型模型仍然是可以管理的。

## 4.1 多产品运输模型

上一章的运输模型关注的是将单一产品从起始地运输到目的地。现在假设我们需要运输几种不同的产品。我们可以定义一个新的集合 PROD，其成员代表不同的产品，并将 PROD 添加到模型中每个组件的索引中；结果如图 4-1 所示。由于在这个版本中，供应量 (Supply)、需求量 (Demand)、成本 (cost) 和运输量 (Trans) 的索引多了一个集合，因此它们多了一个下标：supply[i,p] 表示从起始地 i 运输的产品 p 的数量，Trans[i,j,p] 表示从 i 到 j 运输的产品 p 的数量，以此类推。甚至检查语句现在也按 PROD 索引，以验证每种单独产品的供应量等于需求量。

如果我们观察供应约束 (Supply)、需求约束 (Demand) 和运输量 (Trans)，会发现其变量数量为 (起始地) $\times$ (目的地) $\times$ (产品)，而约束数量为 (起始地 $+$ 目的地) $\times$ (产品)。即使每个集合的成员数量不多，最终形成的线性规划 (linear program) 规模也可能相当大。例如，5 个起始地、20 个目的地和 10 种产品，会产生 1000 个变量和 250 个约束。然而，这个线性规划的规模具有一定的误导性，因为不同产品的运输量是相互独立的。也就是说，一种产品的运输量不会影响其他产品的运输量或运输成本。在这种情况下，更好的做法是为每种产品单独求解一个规模较小的运输问题 (transportation problem)。在 AMPL 中，我们可以使用上一章中的简单运输模型，并为每种产品准备一个不同的数据文件。

如果某些额外的情况导致不同产品之间产生关联，情形就会有所不同。例如，假设对从某个起始地到某个目的地的所有产品的总运输量存在限制，这可能是因为运输能力有限。为了在模型中体现这种限制，我们引入一个新的参数 limit，该参数以起始地和目的地的组合作为索引：

$$
\mathtt{param\ limit}\{\mathtt{ORIG},\mathtt{DEST}\} \geq 0;
$$

接着，我们新增 (起始地) $\times$ (目的地) 个约束，每个约束对应一个起始地 i 和目的地 j，表示从 i 到 j 的所有产品 p 的运输量总和不得超过 limit[i,j]：

subject to Multi {i in ORIG, j in DEST}:  
&nbsp;&nbsp;&nbsp;&nbsp;sum {p in PROD} Trans[i,j,p] <= limit[i,j];

在这些约束 (如图 4-1 所示) 的限制下，我们不能再独立地设定从 i 到 j 的某种产品的运输量，而必须同时考虑其他产品的运输量，因为受到限制的是所有产品的运输量总和。因此，我们只能求解这个大型线性规划问题。

在第 1 章中的 钢铁厂 (steel mill) 例子中，产品包括 带钢 (bands)、线圈 (coils) 和 钢板 (plate)。因此，多产品模型 (multi-commodity model) 的数据可能如图 4-2 所示。我们按常规方式调用 AMPL，得到如下求解结果：

```
ampl: model multi.mod; data multi.dat; solve;
CPLEX 8.0.0: optimal solution; objective 199500
41 dual simplex iterations (0 in phase I)
```

```ampl
set ORIG;  # 起始地
set DEST;  # 目的地
set PROD;  # 产品

param supply {ORIG,PROD} >= 0;  # 在起始地可用的数量
param demand {DEST,PROD} >= 0;  # 在目的地所需的数量

check {p in PROD}: sum {i in ORIG} supply[i,p] = sum {j in DEST} demand[j,p];

param limit {ORIG,DEST} >= 0;
param cost {ORIG,DEST,PROD} >= 0;  # 每单位的运输成本

var Trans {ORIG,DEST,PROD} >= 0;  # 待运输的单位数量

minimize Total_Cost:
  sum {i in ORIG, j in DEST, p in PROD} cost[i,j,p] * Trans[i,j,p];

subject to Supply {i in ORIG, p in PROD}:
  sum {j in DEST} Trans[i,j,p] = supply[i,p];

subject to Demand {j in DEST, p in PROD}:
  sum {i in ORIG} Trans[i,j,p] = demand[j,p];

subject to Multi {i in ORIG, j in DEST}:
  sum {p in PROD} Trans[i,j,p] <= limit[i,j];
```

图 4-1: 多产品运输模型 (multi.mod)。

```ampl
display {p in PROD}: {i in ORIG, j in DEST} Trans[i,j,p];
```

```
Trans[i,j,'bands'] [*,*) (tr) :
         CLEV   GARY   PITT :=
    DET     0      0    300
    FRA   225      0     75
    FRE     0      0    225
    LAF   225      0     25
    LAN     0      0    100
    STL   250    400      0
    WIN     0      0     75

Trans[i,j,'coils'] [*,*) (tr) :
         CLEV   GARY   PITT :=
    DET   525      0    225
    FRA     0      0    500
    FRE   225    625      0
    LAF     0    150    350
    LAN   400      0      0
    STL   300     25    625
    WIN   150      0    100 ;
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a256f56b37ce528fed156c183eb967b98002e4d0674412092166ef2083b76f7c.jpg)  
图 4-2: 多产品运输问题数据 (multi.dat)。  

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/f8c42ebf71ac6ddb386e8bf6e6a399231ca453852b917029bccf1281cac269f9.jpg)  

在我们对运输成本的描述以及 AMPL 对解的显示中，一个三维数据集合（即，以三个集合为索引）必须在二维屏幕上或页面上表示。我们通过沿一个索引“切片”数据来实现这一点，使其显示为一组二维表格。display 命令将对最佳切片索引进行猜测，但通过使用如上所示的显式索引表达式，我们可以告诉它为每个产品显示一个表格。

```ampl
set PROD;  # 产品
param T > 0;  # 周数
param rate {PROD} > 0;  # 每小时生产的吨数
param avail {1..T} >= 0;  # 每周可用的小时数
param profit {PROD, 1..T};  # 每吨利润
param market {PROD, 1..T} >= 0;  # 每周销售吨数的限制

var Make {p in PROD, t in 1..T} >= 0, <= market[p,t];  # 生产的吨数

maximize Total_Profit:
  sum {p in PROD, t in 1..T} profit[p,t] * Make[p,t];  # 所有产品在所有周的总利润
```

受限于 时间 {t 属于 1..T}：  
sum {p 属于 PROD} (1/rate[p]) * Make[p,t] <= avail[t]; # 所有产品使用的总小时数  
# 在每周内不得超过可用小时数

上述最优解仅从 GARY 向 STL 运输了 25 吨线圈，以及从 PITT 向 LAF 运输了 25 吨带钢。我们可能希望要求，如果运输任何数量的产品，则至少运输（例如）50 吨。用我们的模型来表达，即 Trans[i,j,p] $= 0$ 或 Trans[i,j,p] $\geq 50$。不幸的是，虽然可以在 AMPL 中写出这样的“二选一”约束，但它不是线性约束，因此线性规划 (LP) 求解器无法处理它。第 20 章将介绍如何使用更强大（但成本更高）的整数规划技术来处理此类以及相关的离散限制。

# 4.2 多周期生产模型

另一种常见的模型扩展方式是将其在时间维度上进行复制。为说明这一点，我们考虑如何将图 1-4a 中的模型用于未来 $T$ 周的生产计划，而不仅仅是一周。

我们首先为大多数感兴趣的量添加另一个索引集合。新增的集合表示从 1 到 $T$ 编号的周数，如图 4-3 所示。表达式 1..T 是 AMPL 对从 1 到 T 的整数集合的简写。我们已将所有参数和变量在此集合上进行了复制，除了 rate（生产速率），它被视为在时间上是固定的。因此，每周都有一个约束，利润项也按周和产品进行求和。

到目前为止，这只不过是对每一周分别建立了一个独立的 LP 模型，除非添加某些内容将这些周联系起来。正如我们能找到涉及所有产品的约束一样，我们也可以寻找涉及所有周生产的约束。然而，大多数多周期模型采用另一种方法，其中约束仅将每周的生产与下一周的生产联系起来。

假设我们允许将一周的部分生产放入库存，以便在之后的任何一周销售。因此，我们添加新的决策变量，以表示每周库存和销售的数量。变量 Make $[j,t]$ 仍被保留，但现在仅表示生产的数量，这些数量不再必然等于销售的数量。我们的新变量声明如下：

$$
\begin{array}{rl}
& {\mathsf{var~Make~\{PROD,1..T\}~>=~0;}} \\
& {\mathsf{var~Inv~\{PROD,0..T\}~>=~0;}} \\
& {\mathsf{var~Sell~\{p~in~PROD,~t~in~1..T\}~>=~0,~<=~\mathsf{market[p,t]};}}
\end{array}
$$

边界 market $[p,t]$ 表示一周内可销售的最大数量，自然地被赋予变量 Sell $[p,t]$。

变量 Inv $[p,t]$ 将表示产品 $p$ 在周期 $t$ 结束时的库存量。因此，Inv $[p,0]$ 的值表示第 0 周末（即第一周初，换句话说，现在）的库存量。我们的模型假设这些初始库存作为数据的一部分提供：

$$
\mathsf{param~inv0\{PROD\}~>=~0;}
$$

一个简单的约束保证变量 Inv $[p,0]$ 取这些值：

$$
\mathsf{subject~to~Init\_Inv~\{p~in~PROD\}:\quad Inv[p,0] = inv0[p]};
$$

将一个约束专门用于表示变量等于常数可能看起来 “低效”，但在将线性规划发送给求解器时，AMPL 会自动将 inv0[p] 的值代入任何出现 Inv[p,0] 的地方。在大多数情况下，我们可以专注于以最清晰或最简单的方式编写模型，并将效率问题留给计算机处理。

现在我们区分了销售、生产和库存，可以通过定义三个参数，显式地建模它们各自对利润的贡献：

param revenue {PROD, 1..T} >= 0;  
param producost {PROD} >= 0;  
param invcost {PROD} >= 0;

这些参数以下列方式纳入目标函数：

maximize Total_Profit: sum {p in PROD, t in 1..T} (  
 revenue[p,t] * Sell[p,t] - producost[p] * Make[p,t] - invcost[p] * Inv[p,t]  
);

如你所见，revenue[p,t] 是产品 p 在第 t 周每吨的销售收入；producost[p] 和 invcost[p] 是产品 p 每吨的生产成本和库存持有成本，适用于任何一周。

最后，在我们的模型中完全整合了销售和库存之后，我们可以添加关键的约束条件，将各周联系起来：通过生产或从库存中获得的产品在一周内的可用量，必须等于该周通过销售或转入库存而处理的数量：

subject to Balance {p in PROD, t in 1..T}:  
 Make[p,t] + Inv[p,t-1] = Sell[p,t] + Inv[p,t];

由于索引 t 来自一组数字，前一时期可以写成 t-1。实际上，t 可以用于任何算术表达式中；反过来，像 t-1 这样的 AMPL 表达式也可以用于任何它有意义的上下文中。还要注意，对于第一期约束（t 等于 1），左侧的库存项是 Inv[p,0]，即初始库存。

我们现在有了一个完整的模型，如图 4-4 所示。为了说明一个解，我们使用图 4-5 中所示的小样本数据文件；它代表了图 1-4b 数据的四周扩展。

如果我们将模型和数据分别放入文件 steelT.mod 和 steelT.dat 中，则可以调用 AMPL 来求解：

ampl: model steelT.mod;  
ampl: data steelT.dat;  
ampl: solve;  
MINDS 5.5: optimal solution found. 20 iterations, objective 515033  

ampl: option display_1col 0;  
ampl: display Make;  
Make [p, t] (tr) :  
 bands  coils  
1  5990   1407  
2  6000   1400  
3  1400   3500  
4  2000   4200  

ampl: display Inv;  
Inv [p, t] (tr) :  
 bands  coils  
0  10     0  
1   0    1100  
2   0       0  
3   0       0  
4   0       0  

ampl: display Sell;  
Sell [p, t] (tr) :  
 bands  coils  
1  6000   307  
2  6000  2500  
3  1400  3500  
4  2000  4200

集合 PROD; # 产品  
参数 $T > 0$; # 周数  
参数 rate $\{PROD\} >0$; # 每小时生产的吨数  
参数 inv0 $\{PROD\} \geq 0$; # 初始库存  
参数 avail $\{1..T\} \geq 0$; # 每周可用的小时数  
参数 market $\{PROD,1..T\} \geq 0$; # 每周销售吨数的上限  
参数 producost $\{PROD\} \geq 0$; # 每吨生产的成本  
参数 invcost $\{PROD\} \geq 0$; # 每吨库存的成本  
参数 revenue $\{PROD,1..T\} \geq 0$; # 每吨销售的收入  

变量 Make $\{PROD,1..T\} \geq 0$; # 生产的吨数  
变量 Inv $\{PROD,0..T\} \geq 0$; # 库存的吨数  
变量 Sell {p in PROD, t in 1..T} >= 0, <= market[p,t]; # 销售的吨数  

最大化 Total_Profit:  
$$\sum_{p \in PROD, t \in 1..T} \left( revenue[p,t] \cdot Sell[p,t] - producost[p] \cdot Make[p,t] - invcost[p] \cdot Inv[p,t] \right);$$  
# 所有周的总收入减去成本  

约束 Time {t in 1..T}:  
$$\sum_{p \in PROD} \frac{1}{rate[p]} \cdot Make[p,t] \leq avail[t];$$  
# 所有产品使用的总小时数不得超过每周可用的小时数  

约束 Init_Inv {p in PROD}:  
Inv[p,0] = inv0[p];  
# 初始库存必须等于给定值  

约束 Balance {p in PROD, t in 1..T}:  
Make[p,t] + Inv[p,t-1] = Sell[p,t] + Inv[p,t];  
# 生产的吨数和从库存中取出的数量必须等于销售和存入库存的数量  

# 图 4-4：多周期生产模型 (steelT.mod)。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/df9723d0443ebcf35030203916daf55cb5ce0c1a0509720f543e8563c0e1d8aa.jpg)  
图 4-5：多周期生产模型的数据 (steelT.dat)。

第一周生产的线圈被保留到第二周以更高的价格出售。在第二到第四周，线圈比带钢更有利可图，因此线圈被销售至上限，而带钢则填补剩余的产能。（将选项 display_1col 设置为零可使输出以更美观的格式呈现，如第 12.2 节所述。）

# 4.3 生产与运输模型

大型线性规划不仅可以通过连接同一类型的小模型来构建（如以上两个示例），还可以通过链接不同类型模型来创建。我们以一个结合了生产模型和运输模型特征的示例作为本章的结尾。

假设钢铁产品在多个工厂（mills）生产，并从这些工厂运送到各个客户工厂（factories）。对于每个工厂，我们可以定义一个独立的生产模型来优化每种产品的生产量。对于每种产品，我们可以定义一个独立的运输模型，将工厂作为起始地（origins），工厂作为目的地（destinations），以优化该产品的运输量。我们希望将所有这些独立模型整合为一个统一的生产与运输模型。

首先，我们将图 1-4a 的生产模型在工厂（即起始地）上进行复制，而不是像前一个示例那样在周数上进行复制：

设定 PROD； # 产品 (product) 集合  
ORIG； # 起始地 (origin)（钢铁厂）  
param rate {ORIG, PROD} > 0； # 起始地产量速率（吨/小时）  
param avail {ORIG} >= 0； # 起始地可用工时  
var Make {ORIG, PROD} >= 0； # 起始地产量（吨）  
subject to Time {i in ORIG}：sum {p in PROD} (1/rate[i,p]) \* Make[i,p] <= avail[i]；

我们暂时省略了与目标函数相关的部分，稍后再回来讨论。我们也省略了市场需求参数，因为现在需求已经正确地与运输模型中的目的地关联起来了。

下一步是像本章开头的多商品示例那样，将运输模型（见图 3-1a）在产品维度上进行复制：

set ORIG； # 起始地 (origin)（钢铁厂）  
set DEST； # 目的地 (destination)（工厂）  
set PROD； # 产品 (product)  
param supply {ORIG, PROD} >= 0； # 起始地可用量（吨）  
param demand {DEST, PROD} >= 0； # 目的地需求量（吨）  
var Trans {ORIG, DEST, PROD} >= 0； # 运输量（吨）

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/ee777f680939f756a6c52c8e20cd5f3693e896f03424225df7d1dadb9e733de9.jpg)  
图 4-6：生产/运输模型，第 3 版（steelP.mod）

subject to Supply {i in ORIG, p in PROD}：sum {j in DEST} Trans[i,j,p] = supply[i,p]；  
subject to Demand {j in DEST, p in PROD}：sum {i in ORIG} Trans[i,j,p] = demand[j,p]；

比较所得到的生产和运输模型，我们可以看到，起始地集合（ORIG）和产品集合（PROD）在这两个模型中是相同的。此外，运输模型中的“起始地可用量”（supply）实际上与生产模型中的“起始地产量”（Make）是同一事物，因为可供运输的钢材就是钢铁厂所生产的钢材。

因此，我们可以将这两个模型合并，去掉 supply 的定义，并将 supply[i,p] 替换为 Make[i,p]：

subject to Supply {i in ORIG, p in PROD}：sum {j in DEST} Trans[i,j,p] = Make[i,p]；

有几种方式可以添加目标函数以完善模型。最简单的方式可能是为每个变量定义每吨的成本。我们定义参数 make_cost，使得对于每个起始地 i 和产品 p，在目标函数中都有一个项 make_cost[i,p] \* Make[i,p]；同时定义 trans_cost，使得对于每个起始地 i、目的地 j 和产品 p，在目标函数中都有一个项 trans_cost[i,j,p] \* Trans[i,j,p]。完整模型如图 4-6 所示。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/23a7aa703b227dcc1bcaa368d49d4a2b9103b604a14c9fb4a7efae711fbc7bc6.jpg)  
图 4-7：生产/运输模型数据（steelP.dat）

回顾这个模型公式，我们可能会注意到，根据 Supply 的定义，非负表达式 sum {j in DEST} Trans[i,j,p] 可以被替换为 Make[i,p]。如果我们在目标函数和时间约束中所有出现 Make[i,p] 的地方都进行这种替换，那么就不再需要在模型中包含 Make 变量或供应约束，因此我们的线性规划将会变得更小。然而，在大多数情况下，我们最好还是保持模型如上所示。通过“消去”Make 变量，我们使模型更难以阅读，并且求解并没有变得容易多少。

作为基于此模型求解线性规划的一个实例，我们可以改编图 4-2 中的数据，如图 4-7 所示。以下是一些代表性的结果值：

```ampl
model steelP.mod;
data steelP.dat;
solve;
```

CPLEX 8.0.0：最优解；目标值 1392175  
27 次对偶单纯形迭代（第 I 阶段 0 次）

```ampl
option display_1col 5;
option omit_zero_rows 1, omit_zero_cols 1;
display Make;
```

Make  
$[*,\star ]$ ： bands coils plate  
$\begin{array}{rl}{\vdots =} & {} \end{array}$  
CLEV 0 1950 0  
GARY 1125 1750 300  
PITT 775 500 500 ;

```ampl
display Trans;
```

Trans [CLEV,\*,\*] ： coils :=  
DET 750  
LAF 500  
LAN 400  
STL 50  
WIN 250  

[GARY,\*,\*] ： bands coils plate  
$\begin{array}{rl}{\vdots =} & {} \end{array}$  
FREL 225 850 100  
LAF 250 0 0  
STL 650 900 200  

[PITT,\*,\*] ： bands coils plate  
$\begin{array}{rl}{\vdots =} & {} \end{array}$  
DET 300 0 100  
FRA 300 500 100  
LAF 0 0 250  
LAN 100 0 0  
WIN 75 0 50 ;

```ampl
display Time;
```

Time  
$[*,\star ] =$  
CLEV - 1300  
GARY - 2800 ;

正如人们可能预期的那样，最优解并不会将所有产品从所有工厂运输到所有目的地。我们使用了 `omit_zero_rows` 和 `omit_zero_cols` 选项来抑制打印全为零的表格行和列。时间约束的对偶值显示，如果增加产能放在 GARY，对总成本的影响最大；而如果放在 PITT，则没有影响。

我们还可以研究生产和运输的相对成本，这两者是目标函数的组成部分：

```ampl
display sum {i in ORIG, p in PROD} make_cost[i,p] * Make[i,p];
```

sum{i in ORIG, p in PROD} make_cost[i,p]\*Make[i,p] = 1215250  

```ampl
display sum {i in ORIG, j in DEST, p in PROD} trans_cost[i,j,p] * Trans[i,j,p];
```

sum{i in ORIG, j in DEST, p in PROD} trans_cost[i,j,p]\*Trans[i,j,p] = 176925  

显然，在这种情况下生产成本占主导地位。这些例子突出了 AMPL 评估和显示任何有效表达式的能力。

# 参考文献

H. P. Williams, 《Model Building in Mathematical Programming》（第 4 版）。约翰·威利父子出版社（纽约，1999 年）。这是一本涵盖多种模型及其组合的扩展汇编。

# 练习

4-1. 制定一个多周期版本的运输模型，其中在起始地保留库存。

4-2. 制定一个针对多种食品的运输模型与每个目的地饮食模型的组合。

4-3. 下列问题涉及第 4.2 节中的多周期生产模型和数据。

(a) 显示与约束 Time[t] 相关的边际值。在哪些时期增加生产能力最有价值？

(b) 通过争取更多销售，你可能能够提高上限 market[p,t]。显示简化成本 Sell[p,t].rc，并利用它们来建议你更愿意在每周争取更多带钢订单还是线圈订单。

(c) 如果库存成本均为正值，则任何最优解在最后一周之后的库存都将为零。为什么会这样？

这种现象是“末端效应 (end effect)”的一个例子。由于模型在周期 T 后结束，解决方案往往会表现得好像在此之后生产将被关闭。处理末端效应的一种方法是增加建模的周数；这样末端效应对于早期周的解决方案影响应该很小。另一种方法是修改模型以更好地反映库存的现实情况。描述你可能对约束和目标函数做出的一些修改。

4-4. 一家包装饼干和曲奇的生产商每月在其大型面包店运行多个班次。本练习关注的是一个用于决定每月雇佣多少班组的多周期规划模型。在模型的代数描述中，有班次集合 $S$ 和产品集合 $P$，规划周期为 T 个四周周期。相关的运营数据如下：

$l$ 生产线数量：任何班次中可以工作的最大班组数  
$r_p$ 产品 $p$ 的生产率，以每 1000 盒所需的班组小时数表示  
$h_t$ 在规划周期 $t$ 中一个班组工作的小时数  

以下数据由市场或管理考虑决定：

$w_{s}$ 班次 $s$ 中一个班组在一个周期内的总工资  
$d_{pt}$ 产品 $p$ 在周期 $t$ 内必须满足的需求量  
$M$ 从一个周期到下一个周期雇佣班组数的最大变化量  

模型的决策变量为：

$X_{pt}\geq d_{pt}$ 周期 $t$ 内烘焙的产品 $p$ 总箱数（以千箱为单位）  
$0\leq Y_{st}\leq l$ 周期 $t$ 内班次 $s$ 中雇佣的班组数  

目标函数是最小化所有雇佣班组的总成本，

$$
\begin{array}{r}
\sum_{s\in S}\sum_{t = 1}^{T}w_sY_{st}.
\end{array}
$$

每个周期中生产所需的总小时数不得超过所有班次提供的总小时数，

$$
\begin{array}{r}
\sum_{p\in P}r_pX_{pt}\leq h_t\sum_{s\in S}Y_{st},\quad \text{对于每个 } t = 1,\dots,T.
\end{array}
$$

班组数量的变化受以下约束限制：

$$
\begin{array}{r}
- M\leq \sum_{s\in S}(Y_{s,t + 1} - Y_{st})\leq M,\quad \text{对于每个 } t = 1,\dots,T - 1.
\end{array}
$$

根据 $M$ 的定义，此约束限制任何变化在减少 $M$ 个班组和增加 $M$ 个班组之间。

(a) 用 AMPL 建立该模型，并求解以下实例。共有 $T = 13$ 个时期，$l = 8$ 条生产线，每期最多可调整 $M = 3$ 个班组。产品包括 18REG、24REG 和 24PRO，其生产速率 $r_p$ 分别为 1.194、1.509 和 1.509。班组工作分为白班（工资 $w_{s}$ 为 \$44,900）和夜班（工资 \$123,100）。各时期的需求量和工作时间如下：

<table><tr><td>周期 t</td><td>d18REG,t</td><td>d24REG,t</td><td>d24PRO,t</td><td>h1</td></tr><tr><td>1</td><td>63.8</td><td>1212.0</td><td>0.0</td><td>156</td></tr><tr><td>2</td><td>76.0</td><td>306.2</td><td>0.0</td><td>152</td></tr><tr><td>3</td><td>88.4</td><td>319.0</td><td>0.0</td><td>160</td></tr><tr><td>4</td><td>913.8</td><td>208.4</td><td>0.0</td><td>152</td></tr><tr><td>5</td><td>115.0</td><td>298.0</td><td>0.0</td><td>156</td></tr><tr><td>6</td><td>133.8</td><td>328.2</td><td>0.0</td><td>152</td></tr><tr><td>7</td><td>79.6</td><td>959.6</td><td>0.0</td><td>152</td></tr><tr><td>8</td><td>111.0</td><td>257.6</td><td>0.0</td><td>160</td></tr><tr><td>9</td><td>121.6</td><td>335.6</td><td>0.0</td><td>152</td></tr><tr><td>10</td><td>470.0</td><td>118.0</td><td>1102.0</td><td>160</td></tr><tr><td>11</td><td>78.4</td><td>284.8</td><td>0.0</td><td>160</td></tr><tr><td>12</td><td>99.4</td><td>970.0</td><td>0.0</td><td>144</td></tr><tr><td>13</td><td>140.4</td><td>343.8</td><td>0.0</td><td>144</td></tr></table>

显示每班所需班组数。你会发现许多班组数为分数；你将如何将该解转换为整数最优解？

(b) 为保证一致性，你还应要求初始已知班组数（在第一个计划期之前已雇佣的班组数）与第一个计划期雇佣的班组数之间的变化不超过 $M$。在模型中加入这一限制。

使用初始班组数为 11 重新求解。你应该得到相同的解。

(c) 由于班组数在各时期之间的变化受到限制，某些时期雇佣的班组数超过了实际需求。一种解决方法是允许从一个时期向下一个时期结转库存，从而平滑各时期的生产需求。像图 4-4 的模型那样，为每种产品在每个时期之后的库存量添加一个变量，并添加将库存、生产和需求联系起来的约束。（由于可以结转库存，生产量 $X_{pt}$ 不必在每个时期都满足 $\geq$ 需求量 $d_{pt}$，这与之前的版本不同。）此外，还应规定初始库存为零。最后，在目标函数中加入每期每 1000 箱的库存成本。

假设库存成本为产品 18REG 的 $34.56$，以及 24REG 和 24PRO 的 $43.80$。求解得到的线性规划；显示人员规模和库存水平。这个解决方案有何不同？在劳动力成本上节省了多少，而库存成本增加了多少？

(d) 给定数据中的需求在某些时期达到峰值，此时特殊折扣促销生效。在这些峰值之前，特别是第 4 期之前，会提前建立大量库存。然而烘焙食品是易腐的，因此库存超过一定时期是不现实的。

修改模型，使库存变量由产品、时期和年龄索引，其中年龄从 1 到指定的限制 $A$。添加约束条件，任何时期后年龄为 1 的库存不得超过刚刚生产的产品数量，且时期 $t$ 后年龄 $a > 1$ 的库存不得超过时期 $t - 1$ 后年龄 $a - 1$ 的库存。

验证当最大库存年龄为 2 个时期时，基本上可以使用与 (c) 中相同的解决方案，但当最大库存年龄为 1 时，有些时期需要更多的工作人员。

(e) 现在假设不添加第三个库存变量索引如 (d) 中所示，而是施加以下库存约束：时期 $t$ 后产品 $p$ 的库存数量不得超过时期 $t - A + 1$ 到 $t$ 中产品 $p$ 的总产量。

解释为什么在先进先出的基础上管理库存时，这一约束足以防止任何库存超过 $A$ 个时期。通过展示在最大库存年龄为 2 或 1 时求解得到与 (d) 相同的结果来支持你的结论。

(f) 解释如何修改 (c)、(d) 和 (e) 中的模型来考虑非零的初始库存。

4-5. 多时期线性规划可能特别难以开发，因为它们需要与未来相关的数据。为了对冲未来的不确定性，这些 LP 的使用者通常会开发各种情景，包含对某些关键参数的不同预测。本练习要求你开发所谓的随机规划，找到一个在所有情景下都可视为稳健的解决方案。

(a) 每吨收入可能特别难以预测，因为它们依赖于波动的市场条件。设图 4-5 中的收入数据为情景 1，并考虑情景 2：

<table><tr><td>param</td><td>revenue:</td><td>1</td><td>2</td><td>3</td><td>4</td><td>: =</td></tr><tr><td></td><td>bands</td><td>23</td><td>24</td><td>25</td><td>25</td><td></td></tr><tr><td></td><td>coils</td><td>30</td><td>33</td><td>35</td><td>36</td><td>;</td></tr></table>

和情景 3：

<table><tr><td>param</td><td>revenue:</td><td>1</td><td>2</td><td>3</td><td>4</td><td>: =</td></tr><tr><td></td><td>bands</td><td>21</td><td>27</td><td>33</td><td>35</td><td></td></tr><tr><td></td><td>coils</td><td>30</td><td>32</td><td>33</td><td>33</td><td>;</td></tr></table>

通过求解这三个相关的线性规划来验证，即使在第一周，每种情景也会导致不同的最优生产和销售策略。然而，你只需要一种策略，而不是三种。随机规划方法的目的是确定一个单一解，该解在某种意义上能够产生“平均”良好的利润。

(b) 作为构建随机规划模型的第一步，考虑如何将这三种情景整合到一个线性规划中。定义参数 S 为情景的数量，并将收入数据复制到集合 1..S 上：

$$
\begin{array}{rl} 
& \mathtt{param~S~>~0;}\\ 
& \mathtt{param~revenue~\{PROD,1..T,1..S\}~>=~0;} 
\end{array}
$$

以类似的方式复制所有的变量和约束。（这个想法与本章前面复制模型组件到产品或周的做法相同。）

定义一组新的参数 prob[s]，用于表示你对情景 s 发生概率的估计：

param prob $\{1..S\} \Rightarrow 0,$ $\leq = 1$ check: 0.99999 $<$ sum {s in 1..S} prob[s] < 1.00001;

目标函数是期望利润，它等于所有情景下各情景发生概率乘以该情景下最优利润的总和：

maximize Expected_Profit: sum {s in 1..S} prob[s] * sum {p in PROD, t in 1..T} (revenue[p,t,s]*Sell[p,t,s] - producost[p]*Make[p,t,s] - invcost[p]*Inv[p,t,s]);

完成这个多情景 线性规划 (linear program) 的建模，并整理其数据。假设情景 1、2 和 3 的概率分别为 0.45、0.35 和 0.20。证明该解包含的每种情景的生产策略都与 (a) 中的策略相同。

(c) (b) 中的建模方式没有改进，因为它没有在情景之间建立联系。一种使模型可用的方法是添加“非预期性”（nonanticipativity）约束，要求所有情景下第一周的变量取值相同。这样结果将为你提供在所有周中最大化期望利润意义下的第一周最佳单一策略。策略在第一周之后仍会发散——但一周后你可以更新数据并再次运行随机规划模型，以生成第二周的策略：

针对 Make 变量 的非预期性约束可以写为 添加类似的 Inv 和 Sell 变量 的约束。求解该随机规划模型，并验证该解包含所有三种情景的单一第一周策略。

(d) 在得到 (c) 中的解之后，使用以下命令查看推荐策略在三种情景下将实现的利润：

display {s in 1..S} sum {p in PROD, t in 1..T} (revenue[p,t,s]*Sell[p,t,s] - prodcost[p]*Make[p,t,s] - invcost[p]*Inv[p,t,s]);

哪种情景的利润最高，哪种情景的利润最低？

使用情景 1、2 和 3 的概率分别为 0.0001、0.0001 和 0.9998 重复上述分析。你应该会发现策略 3 的利润上升了，而其他两种策略的利润下降了。解释这些利润代表什么，以及为什么结果符合你的预期。
