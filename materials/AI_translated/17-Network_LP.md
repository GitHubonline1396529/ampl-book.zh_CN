
# 15

# 网络线性规划

网络模型已在前几章中出现过，尤其是在第 3 章中的运输问题 (transportation problem) 中。我们现在重新回到这些模型的建模，并介绍 AMPL 处理这些问题的特性。

图 15-1 展示了通常用于描述网络问题的图示。图中的圆圈表示网络中的节点 (node)，箭头表示从一个节点到另一个节点的弧 (arc)。某种流量沿着弧的方向从节点流向节点。

各种各样的模型都涉及在 网络 (network) 上的优化问题。其中许多问题无法以直接的代数方式表达，或者求解非常困难。我们的讨论从一类特定的 网络优化 (network optimization) 模型开始，其中 决策变量 (decision variables) 表示 弧 (arc) 上的流量， 约束 (constraints) 仅限于两类：流量的简单上下界以及 节点 (node) 上的 流量守恒 (flow conservation)。以这种方式受限的模型所产生的问题被称为 网络线性规划 (network linear programs)。这类问题特别易于描述和求解，同时具有广泛的应用性。我们也会简要介绍一些对 网络流 (network flow) 形式的推广，这些推广也具有一些类似的优点。

我们从 最小成本转运模型 (minimum-cost transshipment models) 开始，这是 网络线性规划 (network linear programs) 最大且最直观的来源，然后继续讨论其他众所周知的情形： 最大流 (maximum flow)、 最短路径 (shortest path)、 运输 (transportation) 和 指派模型 (assignment models)。示例最初以标准 AMPL 变量 (variables) 和 约束 (constraints) 的形式给出，这些 变量 和 约束 在 var 和 subject to 声明中定义。在后续章节中，我们将引入 节点 (node) 和 弧 (arc) 声明，使得模型能够更直接地以 网络 结构来描述。最后一节讨论如何构建 网络 模型，以便生成的 线性规划 (linear program) 可以最高效地求解。

# 15.1 最小成本转运模型

作为一个具体示例，假设图 15-1 中的 节点 (node) 和 弧 (arc) 表示 城市 (cities) 和 城际运输链接 (intercity transportation links)。标记为 PITT 的 城市 中的一家制造工厂将在下周生产 450 000 包某种 产品 (product)，如图左侧的 450 所示。标记为 NE 和 SE 的 城市 是 东北 (Northeast) 和 东南 (Southeast) 分销中心 (distribution centers)，它们从工厂接收包裹并转运至编码为 BOS、EWR、BWI、ATL 和 MCO 的 仓库 (warehouses)。(常旅客会认出 波士顿 (Boston)、 纽瓦克 (Newark)、 巴尔的摩华盛顿 (BWI)、 亚特兰大 (Atlanta) 和 奥兰多 (Orlando)。) 这些 仓库 分别需要 90、120、120、70 和 50 千包，如图右侧的数字所示。对于每条 城际链接 (intercity link)，都有每千包的 运输成本 (shipping cost) 和可运输包裹的 上限 (upper limit)，图中对应箭头旁边的两个数字分别表示这两个数值。

在此 网络 (network) 上的优化问题是要找到一个最低 成本 (cost) 的 运输计划 (shipments)，该计划仅使用 可用链接 (available links)，遵守指定的 容量 (capacities)，并满足 仓库 的 需求量 (demand)。我们首先将其建模为一个 一般网络流问题 (general network flow problem)，然后考虑针对手头特定情况的替代方案。最后，我们介绍一些最常见的 网络流 约束 (constraints) 变化形式。

# 一般转运模型

要为任何 城市 到 城市 的 运输问题 (transportation problem) 编写 模型 (model)，我们可以从定义一组 城市 和一组 链接 (links) 开始。每条 链接 又由一个 起始城市 (origin city) 和一个 终点城市 (destination city) 定义，因此我们希望 链接 集合是 城市 有序对集合的一个 子集 (subset)：

```ampl
set CITIES;
set LINKS within (CITIES cross CITIES);
```

每个 城市 对应潜在的 供应量 (supply) 和 需求量 (demand)：

```ampl
param supply {CITIES} >= 0;
param demand {CITIES} >= 0;
```

对于图 15-1 所描述的问题，唯一的非零供应量应该是 PITT 的供应量，因为包裹是在那里制造并供应给分销网络的。唯一非零的需求量应该是对应于五个仓库的需求量。

成本 (cost) 和容量 (capacity) 是以 LINKS 为索引的：

```ampl
param cost {LINKS} >= 0;
param capacity {LINKS} >= 0;
```

决策变量亦是如此，其表示通过各 LINKS 运输的包裹数量。这些变量为非负值，并受容量限制：

var Ship {(i,j) in LINKS} >= 0, <= capacity[i,j];

目标函数为最小化总成本 (Total Cost)：  
sum {(i,j) in LINKS} cost[i,j] * Ship[i,j];

该式表示所有 LINKS 运输成本之和。

接下来描述约束条件。在每个城市，供应的包裹数量与运入的包裹数量之和必须等于需求的包裹数量与运出的包裹数量之和：

subject to Balance {k in CITIES}:  
supply[k] + sum {(i,k) in LINKS} Ship[i,k] = demand[k] + sum {(k,j) in LINKS} Ship[k,j];

由于表达式 sum {(i,k) in LINKS} Ship[i,k] 在虚拟索引 k 的定义范围内出现，因此求和将遍历所有满足 (i,k) 属于 LINKS 的城市 i。也就是说，该求和是对所有进入城市 k 的 LINKS 进行的；类似地，第二个求和是对所有从城市 k 出发的 LINKS 进行的。这种索引约定在第 6.2 节中已有说明，在代数形式描述网络平衡约束时经常很有用。图 15-2a 和图 15-2b 显示了针对图 15-1 所示具体问题的完整模型和数据。

如果将所有变量移到等号左边，常数项移到等号右边，则平衡约束变为：

subject to Balance {k in CITIES}:  
sum {(i,k) in LINKS} Ship[i,k] - sum {(k,j) in LINKS} Ship[k,j] = demand[k] - supply[k];

这种形式可以解释为：在每个城市 k，运入量减去运出量必须等于“净需求” (net demand)。如果没有任何城市同时拥有工厂和仓库（如我们的示例中所示），则正的净需求始终表示仓库城市，负的净需求表示工厂城市，净需求为零则表示转运城市。因此，我们可以仅使用一个参数 net_demand 来代替 demand 和 supply，通过 net_demand[k] 的符号来表示城市 k 的情况。这类替代建模方法在网络流模型的描述中经常出现。

# 专门的转运模型

set CITIES;  
set LINKS within (CITIES cross CITIES);  
param supply {CITIES} >= 0; # 城市的供应量  
param demand {CITIES} >= 0; # 城市的需求量  
check: sum {i in CITIES} supply[i] = sum {j in CITIES} demand[j];  
param cost {LINKS} >= 0; # 运输成本/1000 包裹  
param capacity {LINKS} >= 0; # 可运输的最大包裹数  
var Ship {(i,j) in LINKS} >= 0, <= capacity[i,j]; # 待运输的包裹数  
minimize Total Cost: sum {(i,j) in LINKS} cost[i,j] * Ship[i,j];  
subject to Balance {k in CITIES}:  
supply[k] + sum {(i,k) in LINKS} Ship[i,k] = demand[k] + sum {(k,j) in LINKS} Ship[k,j];

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/ebb43ef3e39938d87f9a770019af392315b83dc0ed732b1976c8768e12dab567.jpg)  
图 15-2a：通用转运模型 (net1.mod)。  
图 15-2b：通用转运模型的数据 (net1.dat)。

前述通用方法的优势在于能够适应任何供应量、需求量以及城市间连接的模式。例如，只需对数据进行简单更改，即可对某个配送中心建立工厂模型，或允许某些仓库之间存在运输连接。

通用模型的缺点在于无法清楚地展示预期的供应量、需求量和连接安排，实际上还会允许不恰当的安排。如果我们知道实际情况如图 15-1 所示，即一个工厂提供供应量，运输至配送中心，再由配送中心运输至满足需求的仓库，则可以对模型进行特化，以展示并强制实施此类结构。

为了明确表示在特化模型中存在三种不同类型的城市，我们可以分别声明它们。我们使用符号参数而非集合来保存工厂名称，以指定仅预期一个工厂：

```ampl
param p_city symbolic; 
set D_CITY; 
set W_CITY;
```

工厂与每个配送中心之间必须存在连接，因此我们需要一个配对子集，仅用于指定哪些连接将配送中心与仓库相连：

```ampl
set DW_LINKS within (D_CITY cross W_CITY);
```

通过这种方式组织声明，不可能指定不恰当的连接类型，例如两个仓库之间的连接或从仓库返回工厂的连接。

一个参数表示工厂的供应量，一组需求参数以仓库为索引：

```ampl
param p_supply >= 0; 
param w_demand {W_CITY} >= 0;
```

这些声明允许供应量和需求量仅在其所属位置定义。

在此阶段，我们可以按照之前模型所需的方式定义集合 CITIES 和 LINKS 以及参数 supply 和 demand：

```ampl
set CITIES = {p_city} union D_CITY union W_CITY; 
set LINKS = {p_city} cross D_CITY union DW_LINKS; 
param supply {k in CITIES} = if k = p_city then p_supply else 0; 
param demand {k in CITIES} = if k in W_CITY then w_demand[k] else 0;
```

模型的其余部分可以与通用情况完全相同，如图 15-3a 和 15-3b 所示。

param p_city symbolic; set D_CITY; set W_CITY; set Dw_LINKS within (D_CITY cross W_CITY); param p_supply $ \geq 0 $; # 工厂供应量 param w_demand {W_CITY} $ \geq 0 $; # 仓库需求量 check: p_supply $ = $ sum {k in W_CITY} w_demand[k]; set CITIES $ = $ {p_city} union D_CITY union W_CITY; set LINKS $ = $ ({p_city} cross D_CITY) union Dw_LINKS; param supply {k in CITIES} $ = $ if k $ = $ p_city then p_supply else 0; param demand {k in CITIES} $ = $ if k in W_CITY then w_demand[k] else 0; ### 其余部分与一般转运模型相同 ### param cost {LINKS} $ \geq 0 $; # 运输成本/1000 包 param capacity {LINKS} $ \geq 0 $; # 最大可运输包数 var Ship {(i,j) in LINKS} $ \geq 0 $, $ \leq $ capacity[i,j]; # 待运输包数 minimize Total Cost: sum {(i,j) in LINKS} cost[i,j] * Ship[i,j]; subject to Balance {k in CITIES}: supply[k] $ + $ sum {(i,k) in LINKS} Ship[i,k] $ = $ demand[k] $ + $ sum {(k,j) in LINKS} Ship[k,j];

或者，我们可以在整个模型中保留对不同类型城市和链接的引用。这意味着我们必须声明两种类型的成本、容量和运输量：

param pd_cost {D_CITY} $\geq 0$; param dw_cost {DW_LINKS} $\geq 0$; param pd_cap {D_CITY} $\geq 0$; param dw_cap {DW_LINKS} $\geq 0$; var PD_Ship {i in D_CITY} $\geq 0$, $\leq$ pd_cap[i]; var DW_Ship {(i,j) in DW_LINKS} $\geq 0$, $\leq$ dw_cap[i,j];

“pd”数量与从工厂到配送中心的运输量相关；因为它们都与从同一工厂出发的运输量有关，所以只需在 D_CITY 上建立索引。“dw”数量与从配送中心到仓库的运输量相关，因此自然在 DW_LINKS 上建立索引。

总运输成本现在可以表示为两个求和式的总和：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/36ae95db21573dd5fdac26f14f2e0035e410289e02994b3ef3588516ed4b5fe6.jpg)  
图 15-3b：专门化转运模型的数据 (net2.dat)。

minimize Total Cost: sum {i in D_CITY} pd_cost[i] * PD_Ship[i] + sum {(i,j) in DW_LINKS} dw_cost[i,j] * DW_Ship[i,j];

最后，必须有三种类型的平衡约束，每种城市对应一种。从工厂到配送中心的运输量必须等于工厂的供应量：

subject to P_Bal: sum {i in D_CITY} PD_Ship[i] = p_supply;

在每个配送中心，从工厂运入的货物必须等于运往所有仓库的货物：

subject to D_Bal {i in D_CITY}: PD_Ship[i] = sum {(i,j) in DW_LINKS} DW_Ship[i,j];

而在每个仓库，从所有配送中心运入的货物必须等于需求量：

subject to W_Bal {j in W_CITY}: sum {(i,j) in DW_LINKS} DW_Ship[i,j] = w_demand[j];

整个模型及其相应的数据如图 15-4a 和 15-4b 所示。

图 15-3 和图 15-4 所示的方法是等价的，因为它们会导致求解相同的线性规划问题。前者在尝试不同的网络结构时更为方便，因为任何更改只会影响模型中初始声明的数据。然而，如果网络结构不太可能发生变化，

```ampl
set D_CITY;
set W_CITY;
set Dw_LINKS within (D_CITY cross W_CITY);

param p_supply >= 0;  # 工厂可用量
param w_demand {W_CITY} >= 0;  # 仓库需求量
check: p_supply = sum {j in W_CITY} w_demand[j];

param pd_cost {D_CITY} >= 0;  # 运输成本/1000 包裹
param dw_cost {Dw_LINKS} >= 0;
param pd_cap {D_CITY} >= 0;  # 可运输的最大包裹数
param dw_cap {Dw_LINKS} >= 0;

var PD_Ship {i in D_CITY} >= 0, <= pd_cap[i];
var Dw_Ship {(i,j) in Dw_LINKS} >= 0, <= dw_cap[i,j];  # 待运输的包裹数

minimize Total_Cost:
    sum {i in D_CITY} pd_cost[i] * PD_Ship[i] +
    sum {(i,j) in Dw_LINKS} dw_cost[i,j] * Dw_Ship[i,j];

subject to P_Bal:
    sum {i in D_CITY} PD_Ship[i] = p_supply;

subject to D_Bal {i in D_CITY}:
    PD_Ship[i] = sum {(i,j) in Dw_LINKS} Dw_Ship[i,j];

subject to W_Bal {j in W_CITY}:
    sum {(i,j) in Dw_LINKS} Dw_Ship[i,j] = w_demand[j];
```

则后一种形式便于仅影响特定类型城市的修改，例如我们接下来描述的推广形式。

# 转运模型的变体

在网络流模型中，某些平衡约束可能是不等式而不是等式。在图 15-4 的示例中，如果工厂的产量有时会超过仓库的总需求量，则应在 P_Bal 约束中将 $=$ 替换为 $\leq$。

当从一条弧流出的流量不一定等于流入的流量时，会发生更实质性的修改。例如，在包裹从工厂运送到配送中心的过程中，可能会有少量包裹因损坏或失窃而丢失。假设引入参数 pd_loss 来表示损耗率：

$$
\mathtt{param\ pd\_loss\ \{D\_CITY\} >= 0,\ < 1;}
$$

则配送中心的平衡约束必须相应调整：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/87a47c17f2d0a5438295e76ac504d7991077065900290d5fd7dbeff377e8d618.jpg)  
图 15-4b：专用转运模型版本 2 的数据 (net3.dat)。

```ampl
subject to D_Bal {i in D_CITY}:
    (1 - pd_loss[i]) * PD_Ship[i] = sum {(i,j) in Dw_LINKS} Dw_Ship[i,j];
```

等号左侧的表达式已被修改，以反映当从工厂运输 PD_Ship[i] 包裹时，只有 (1 - pd_loss[i]) * PD_Ship[i] 包裹实际到达城市 i。

当网络中的流量未使用相同单位进行测量时，也会出现类似的变体。例如，如果需求以纸箱而不是千包裹为单位报告，则模型需要一个参数来表示每纸箱的包裹数：

```ampl
param ppc integer > 0;
```

然后，仓库的需求约束将调整如下：

受限于 W_Bal {j 属于 W_CITY}:  sum {(i,j) 属于 DW_LINKS} (1000/ppc) * DW_Ship[i,j]  = w_demand[j];

当从配送中心 i 运输 DW_Ship[i,j] 千个包裹时，项 (1000/ppc) * DW_Ship[i,j] 表示仓库 j 收到的纸箱数量。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/f7857c822eb933eea0b4587cca0e47d131201fac4b28dc75afd002f1a79b3ff2.jpg)  
图 15-5: 交通流量网络。

# 15.2 其他网络模型

并非所有网络线性规划都涉及物品运输或成本最小化。我们在此描述三个著名的模型类别 —— 最大流、最短路径以及运输/指派问题 —— 它们使用相同类型的变量和约束，但用途不同。

# 最大流模型

在某些网络设计应用中，关注的是尽可能多地将流量通过网络发送，而不是以最低成本发送流量。这种替代方案可以通过在流量的起始地和目的地删除平衡约束，同时替换一个代表某种意义上总流量的目标函数来轻松处理。

作为一个具体示例，图 15-5 展示了一个简单交通网络的图示。节点和弧线表示交叉口和道路；容量显示为道路旁边的数字，单位为每小时车辆数。我们希望找到可以从 $a$ 进入网络并在 $g$ 离开的最大交通流量。

针对这种情况的模型以一组交叉口开始，并使用符号参数来表示作为道路网络入口和出口的交叉口：

set INTER:  ```python  param entr symbolic in INTER;  param exit symbolic in INTER, <> entr;  ```

道路集合被定义为交叉口对的一个子集：

set ROADS within (INTER diff {exit}) cross (INTER diff {entr});

此定义确保没有道路从出口开始或在入口结束。

接下来，为每条道路定义容量和交通负载：

集合 INTER：# 交叉口参数 entr symbolic in INTER；# 道路网络入口参数 exit symbolic in INTER，<> entr；# 道路网络出口集合 ROADS within (INTER diff {exit}) cross (INTER diff {entr})；参数 cap {ROADS} $\geq 0$。# 容量变量 Traff {i,j) in ROADS} $\geq 0$，$\leq$ cap[i,j]；# 交通负载最大化 Entering_Traff：sum {entr,j) in ROADS} Traff[entr,j]；subject to Balance {k in INTER diff {entr,exit}}：sum {i,k) in ROADS} Traff[i,k] $=$ sum {k,j) in ROADS} Traff[k,j]；数据：set INTER：$=$ a b c d e f g；param entr：$=$ a；param exit：$=$ g；param：ROADS：cap：$=$ a b 50, a c 100 b d 40, b e 20 c d 60, c f 20 d e 50, d f 60 e g 70, f g 70；

图 15-6：最大交通流量模型和数据 (netmax.mod)。

param cap {ROADS} >= 0; var Traff {i,j) in ROADS} >= 0, <= cap[i,j];

约束条件表示，除了入口和出口外，流入每个交叉口的流量等于流出的流量：

subject to Balance {k in INTER diff {entr,exit}}：sum {i,k) in ROADS} Traff[i,k] $=$ sum {k,j) in ROADS} Traff[k,j];

给定这些约束条件，从入口流出的交通流量必须等于网络中的总流量，而这个总流量需要被最大化：

最大化 进入交通流量：sum { (entr,j) in ROADS } Traff[entr,j];

我们同样可以最大化进入出口的总流量。整个模型以及图 15-5 所示示例的数据如图 15-6 所示。任何线性规划求解器都会找到每小时 130 辆车的最大流量。

# 最短路径模型

如果你使用到目前为止我们任何一个模型的最优解，你将不得不沿着从供应（或入口）节点到需求（或出口）节点的某条路径发送每个包裹、车辆或其他对象。决策变量的值并不能直接说明最优路径是什么，或者每条路径上必须有多少流量。然而，通常推导出这些路径并不太困难，特别是当网络具有规律性或特殊结构时。

如果一个网络中只有一个单位的供应量和一个单位的需求量，那么最优解呈现出完全不同的性质。与每条弧相关的变量要么是 0 要么是 1，并且变量值为 1 的弧组成从供应节点到需求节点的最小成本路径。通常，“成本”实际上是时间或距离，因此最优解给出了最短路径。

只需对图 15-6 中的最大流模型进行少量修改即可将其转换为最短路径模型。每条从 i 到 j 的道路仍然有一个参数和一个变量，但我们称它们为 time[i,j] 和 Use[i,j]，它们乘积的总和构成了目标函数：

```python
param time{ROADS} >= 0;  # 道路通行时间
var Use{(i,j) in ROADS} >= 0;  # 若 (i,j) 在最短路径中则为 1
minimize Total_Time: sum{(i,j) in ROADS} time[i,j] * Use[i,j];
```

由于只有最优路径上的变量 Use[i,j] 等于 1，其余为 0，因此这个总和确实正确表示了穿越最优路径所需的总时间。唯一的其他变化是增加了一个约束条件，以确保在网络入口处恰好有一单位的流量可用：

```python
subject to Start: sum{(entr,j) in ROADS} Use[entr,j] = 1;
```

完整的模型如 图 15-7 所示。如果我们想象图 15-5 中弧上的数字是通行时间 (以分钟为单位) 而不是容量，则数据是相同的；AMPL 找到的解如下：

```python
ampl: model netshort.mod;
ampl: solve;
MINOS 5.5: optimal solution found.
1 iterations, objective 140
ampl: option omit_zero_rows 1;
ampl: display Use;
Use :=
a b   1
b e   1
e g   1
;
```

最短路径是 a → b → e → g，耗时 140 分钟。

# 运输与指派模型

最著名且应用最广泛的特殊网络结构是图 15-8 中所示的“二分”结构。节点分为两组，一组作为流量的起始地，另一组作为目的地。每条弧连接一个起始地到一个目的地。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a0766b7d288dc1ed7b2f6615a1b92f2904abc2010636ab7f1ee6fef52d17a974.jpg)  
图 15-7：最短路径模型和数据 (netshort. mod)。

在网络上的最小成本转运模型被称为运输模型。在第 3 章中介绍了每 个起始点都与每个目的地相连的特殊情况；图 3-1a 和 3-1b 显示了 AMPL 模型和示例数据。在本章早些时候开发的模型的一个更一般的例子中，集合 LINKS 指定了网络的弧，如图 6-2a 和 6-2b 所示。

在二分网络中，从起始点到目的地的每条路径由一条弧组成。或者换句话说，运输模型中沿弧的最优流量给出了从某个起始点运送到某个目的地的实际数量。这一特性使得运输模型也可以被看作所谓的指派模型，在这种模型中，沿弧的最优流量是从起始点分配给目的地的某种数量。在这种情况下，指派的含义可以广义地理解，并且特别地，不一定涉及任何形式的运输。

指派模型最常见的应用之一是将人员与适当的目标进行匹配，例如工作、办公室甚至其他人。每个起始节点与一个人相关联，每个目的地节点与一个目标相关联——例如，与一个项目相关联。集合可以定义如下：

set PEOPLE; set PROJECTS; set ABILITIES within (PEOPLE cross PROJECTS);

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/706153781a91c7046eeca1cf810c93d2afd1d8c62f94ba8c417f76d3748bbe9f.jpg)  
图 15-8：二分网络。

集合 ABILITIES 在我们之前的模型中起 LINKS 的作用；当且仅当人员 i 可以从事项目 j 时，一对 $(i,j)$ 才会被放入该集合中。

作为继续建模的一种可能性，节点 i 的供应量可以是人员 i 可用于工作的小时数，节点 j 的需求量可以是项目 j 所需的小时数。变量 Assign[i,j] 将表示分配给项目 j 的人员 i 的时间 (以小时计)。此外，每对 $(i,j)$ 还会关联每小时的成本，以及人员 i 可以为工作 j 贡献的最大小时数。最终模型如图 15-9 所示。

另一种可能性是根据人员而不是小时来进行指派。每个节点 $i$ 的供应量为 $1$（人），节点 $j$ 的需求量为项目 $j$ 所需的人员数量。供应约束确保 $Assign[i,j]$ 不大于 $1$；在最优解中，当且仅当人员 $i$ 被分配到项目 $j$ 时，它才等于 $1$。系数 $cost[i,j]$ 可能是将人员 $i$ 分配到项目 $j$ 的某种成本，在这种情况下，目标仍然是最小化总成本。或者该系数可以是人员 $i$ 对于项目 $j$ 的排名，可能是在从 $1$（最高）到 $10$（最低）的量表上。那么该模型将产生一个使排名总和达到最佳可能值的指派方案。

最后，我们可以设想一种指派模型，其中每个节点 $j$ 的需求量也为 $1$；此时问题就变成了将人员与项目进行匹配。在目标函数中，$cost[i,j]$ 可以表示人员 $i$ 完成项目 $j$ 所需的小时数，这种情况下模型将找出能使完成所有项目所需的总工时最少的指派方案。你可以通过将图 $15-9$ 中所有 $supply[i]$ 和 $demand[j]$ 的引用替换为 $1$ 来创建这种模型。

```ampl
set PEOPLE;
set PROJECTS;
set ABILITIES within (PEOPLE cross PROJECTS);

param supply {PEOPLE} >= 0;    # 每个人可提供的小时数
param demand {PROJECTS} >= 0;  # 每个项目需要的小时数
check: sum {i in PEOPLE} supply[i] = sum {j in PROJECTS} demand[j];

param cost {ABILITIES} >= 0;   # 每小时工作的成本
param limit {ABILITIES} >= 0;  # 对项目的最大贡献值

var Assign {(i,j) in ABILITIES} >= 0, <= limit[i,j];  # 分配变量

minimize Total_Cost:
    sum {(i,j) in ABILITIES} cost[i,j] * Assign[i,j];

subject to Supply {i in PEOPLE}:
    sum {(i,j) in ABILITIES} Assign[i,j] = supply[i];

subject to Demand {j in PROJECTS}:
    sum {(i,j) in ABILITIES} Assign[i,j] = demand[j];
```

表示排序的目标函数系数也可以用于该模型，从而产生我们在 $3.3$ 节中作为示例使用的那种指派模型。

# 15.3 通过节点和弧声明网络模型

AMPL 的代数表示法具有强大的表达能力，可以表达各种网络线性规划 (network linear program)，但由此产生的约束表达式往往不如我们所期望的那样自然。虽然在每个节点上“流出减流入”的约束思想很容易描述和理解，但相应的代数约束通常包含诸如 $sum {(i,k) in LINKS} Ship[i,k]$ 这样的项，这些项不容易快速理解。网络越复杂、越接近现实，这个问题就越严重。实际上，很难判断一个模型的代数约束是否代表了网络上有效的流量平衡集合，因此也难以判断是否可以使用专门的网络优化软件（本章后面将介绍）。

网络流的代数形式化方法往往存在问题，因为它们是显式地基于变量和约束构建的，而节点 (node) 和弧 (arc) 只是隐含在约束结构中。人们更倾向于以相反的方式处理网络流问题：他们设想给出节点和弧的显式定义，然后从中隐式地产生流变量和平衡 (Balance) 约束。为应对这种情况，AMPL 提供了一种替代方法，允许在网络模型中直接声明网络概念。

对 AMPL 的网络扩展包括两种新的声明类型：node（节点）和 arc（弧），它们取代了代数约束形式中的 subject to 和 var 声明。node 声明用于命名网络中的节点，并描述节点处的流量平衡约束。arc 声明用于命名和定义弧，通过指定弧所连接的节点，并提供与弧相关的可选信息（如上下界和成本 (cost)）来实现。

本节通过展示如何使用 node 和 arc 便捷地重新表述本章前面的几个示例，介绍这两种声明方式。下一节将更系统地介绍这些声明的规则。

# 一个通用的转运模型

在使用 node 和 arc 重写图 15-2a 的模型时，我们可以保留所有的集合和参数声明及其相关数据。变化只影响定义线性规划的三个声明——minimize、var 和 subject to。

网络中每个 CITIES 集合的成员对应一个节点。使用 node 声明，我们可以直接表达这一点：

```ampl
node Balance {k in CITIES}: net_in = demand[k] - supply[k];
```

关键字 net_in 表示“净流入”，即流入量减去流出量，因此该声明表示在每个节点 Balance[k] 处，净流入必须等于净需求。这与代数版本中名为 Balance[k] 的约束表达的是相同的内容，只是用简洁的术语 net_in 代替了冗长的表达式 `sum {(i,k) in LINKS} Ship[i,k] - sum {(k,j) in LINKS} Ship[k,j]`。

实际上，subject to 和 node 的语法几乎相同，区别仅在于流量守恒约束的表达方式。（关键字 net_out 也可用于表示流出量减去流入量，因此我们可以写成 `net_out = supply[k] - demand[k]`。）

网络中每个 LINKS 集合中的配对对应一条弧。这同样可以用 arc 声明直接表达：

```ampl
arc Ship {(i,j) in LINKS} >= 0, <= capacity[i,j], from Balance[i], to Balance[j], obj Total_Cost cost[i,j];
```

为 LINKS 中的每个配对定义一条弧 Ship[i,j]，其流量的下界 (>=) 和上界 (<=) 分别为 0 和 capacity[i,j]；从这个角度看，arc 声明与 var 声明是相同的。然而，arc 声明还包含额外的短语，用于说明该弧从名为 Balance[i] 的节点流向名为 Balance[j] 的节点，并在名为 Total_Cost 的目标函数中具有线性系数 cost[i,j]。这些短语使用了关键字 from、to 和 obj。

由于目标函数的信息已包含在 arc 声明中，因此在 minimize 声明中不再需要这些信息，minimize 声明简化为：

```ampl
minimize Total_Cost;
```

集合 CITIES; 集合 LINKS within (CITIES cross CITIES); 参数 supply {CITIES} >= 0; # 城市可提供的数量 参数 demand {CITIES} >= 0; # 城市所需数量 核查: sum {i in CITIES} supply[i] = sum {j in CITIES} demand[j]; 参数 cost {LINKS} >= 0; # 运输成本/1000 包 参数 capacity {LINKS} >= 0; # 可运输的最大包数 最小化 Total_Cost; 节点 Balance {k in CITIES}: net_in = demand[k] - supply[k]; 弧 Ship {(i,j) in LINKS} >= 0, <= capacity[i,j], 从 Balance[i], 到 Balance[j], 目标函数 Total_Cost cost[i,j];

图 15-10: 带有节点和弧的通用转运模型 (net1node.mod)。

整个模型如图 15-10 所示。

正如该描述所指出的，arc 和 node 分别取代了 var 和 subject to。实际上，AMPL 将弧声明视为变量定义，因此你仍然可以使用 display Ship 来查看图 15-10 网络模型中的最优流；它将节点声明视为约束定义。区别在于，node 和 arc 以更直接对应网络图的形式呈现模型。节点的描述总是在前面，然后是弧如何连接节点的描述。

# 一个专门的转运模型

node 和 arc 声明使得定义具有多种不同类型节点和弧的网络线性规划变得容易。例如，我们回到图 15-4a 的专门模型。

网络有一个工厂节点，每个 D_CITY 成员对应一个配送中心节点，每个 W_CITY 成员对应一个仓库节点。因此模型需要三个节点声明：

node Plant: net_out = p_supply; node Dist {i in D_CITY}; node Whse {j in W_CITY}: net_in = w_demand[j];

平衡条件表明，从节点 Plant 流出的流量必须为 p_supply，流入节点 Whse[j] 的流量为 w_demand[j]。（网络中没有流入工厂或从仓库流出的弧，因此 net_out 和 net_in 分别只是流出和流入的流量。）节点 Dist[i] 的条件可以写成 net_in = 0 或 net_out = 0，但由于这些是默认假设的，因此我们无需指定任何条件。

set D_CITY; set W_CITY; set Dw_LINKS within (D_CITY cross W_CITY); param p_supply >= 0; # 工厂可提供的数量 param w_demand {W_CITY} >= 0; # 仓库所需数量 check: p_supply = sum {j in W_CITY} w_demand[j]; param pd_cost {D_CITY} >= 0; # 运输成本/1000 包 param dw_cost {Dw_LINKS} >= 0; param pd_cap {D_CITY} >= 0; # 可运输的最大包数 param dw_cap {Dw_LINKS} >= 0; minimize Total_Cost; node Plant: net_out = p_supply; node Dist {i in D_CITY}; node Whse {j in W_CITY}: net_in = w_demand[j]; arc PD_Ship {i in D_CITY} >= 0, <= pd_cap[i], from Plant, to Dist[i], obj Total_Cost pd_cost[i]; arc Dw_Ship {(i,j) in Dw_LINKS} >= 0, <= dw_cap[i,j], from Dist[i], to Whse[j], obj Total_Cost dw_cost[i,j];

图 15-11：具有节点和弧的专门转运模型 (net3node.mod)。

该网络有两种类型的弧。从工厂到 D_CITY 中每个成员都有一条弧，可以声明为：

```ampl
arc PD_Ship {i in D_CITY} >= 0, <= pd_cap[i], from Plant, to Dist[i], obj Total_Cost pd_cost[i];
```

对于 DW_LINKS 中的每对 (i,j)，从配送中心 i 到仓库 j 也有一条弧：

```ampl
arc Traff {(i,j) in ROADS} >= 0, <= cap[i,j],
    from (if i == entr then Entr_Int else Intersection[i]),
    to (if j == exit then Exit_Int else Intersection[j]),
    obj Entering_Traff (if i == entr then 1 else 0);
```

该声明使用了条件表达式来动态确定弧的起点和终点节点。当 `i` 等于 `entr` 时，弧从 `Entr_Int` 节点出发；否则从 `Intersection[i]` 出发。类似地，当 `j` 等于 `exit` 时，弧指向 `Exit_Int` 节点；否则指向 `Intersection[j]`。目标函数系数也通过条件表达式设定，仅当 `i` 等于 `entr` 时为 1，其余情况为 0。

arc_Traff $\{\mathrm{(i,j)}$ 在 ROADS 中 $\} >= 0$ $<=$ cap[i,j], # 语法错误 从 (如果 $\mathrm{\bf i}=$ entr 则为 Entr_Int 否则为 Intersection[i] 到 (如果 $\mathrm{\bf j}=$ exit 则为 Exit_Int 否则为 Intersection[j] obj Entering_Traff (如果 $\mathrm{\bf i}=$ entr 则为 1);

然而，AMPL 中的 if-then-else 结构不适用于诸如 Entr_Int 和 Intersection[i] 之类的模型组件；此版本将被拒绝为语法错误。相反，您需要使用由索引表达式限定的 from 和 to 短语：

arc_Traff $\{\mathrm{(i,j)}$ 在 ROADS 中 $\} >= 0$ $<=$ cap[i,j], 从 {如果 $\mathrm{\bf i}=$ entr} Entr_Int, 从 {如果 $\mathrm{\bf i}<>$ entr} Intersection[i], 到 {如果 $\mathrm{\bf j}=$ exit} Exit_Int, 到 {如果 $\mathrm{\bf j}<>$ exit} Intersection[j], obj Entering_Traff (如果 $\mathrm{\bf i}=$ entr 则为 1);

此处以 if 开头的特殊索引表达式的工作方式与约束条件中的方式大致相同（第 8.4 节）；如果 if 后面的条件为真，则处理 from 或 to 短语。因此，Traff[i,j] 被声明为如果 i 等于 entr 则从 Entr_Int 出发，如果 i 不等于 entr 则从 Intersection[i] 出发，这正是我们所期望的。

作为替代方案，我们可以将三种不同类型节点的声明合并为一个。观察到 net_out 对于 Entr_Int 为正或零，对于 Exit_Int 为负或零，对于所有其他节点 Intersection[i] 为零，我们可以声明：

node Intersection {k 在 INTEGER 中}: (如果 $\mathrm{\bf k}=$ exit 则为 - Infinity) $<=$ net_out $<=$ (如果 $\mathrm{\bf k}=$ entr 则为 Infinity);

以前声明为 Entr_Int 和 Exit_Int 的节点现在只是 Intersection[entr] 和 Intersection[exit]，因此我们之前标记为“不正确”的弧线声明现在可以正常工作。此版本与前一版本之间的选择完全取决于方便性和个人喜好。（Infinity 是一个预定义的 AMPL 参数，可用于指定任何“无限大”的界限；其技术定义见 A.7.2 节。）

可以说，最方便且最具吸引力的 AMPL 表述既不是上述两种形式中的任何一种，而是通过稍微不同的方式解读图 15-5 中的网络图得出的。假设我们将进入入口节点和离开出口节点的箭头视为表示额外的弧，这些弧恰好只与一个节点相邻而不是两个节点。那么在每个交叉点处流入等于流出，节点声明简化为：

node Intersection {k in INTEGER};

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/695d4501a09ad2c5b35444e549aae7ed7e3e0f7bbe0523134b2cc9f489fb1950.jpg)  
图 15-12：具有节点和弧的最大流模型 (netmax3.mod)。

悬挂在入口和出口处的两条弧以显而易见的方式定义，但仅包含 to 或 from 短语：

arc Traff_In  $> = 0$  to Intersection[entr]; arc Traff_Out  $> = 0$  from Intersection[exit];

表示网络内部道路的弧按之前的方式声明：

arc Traff  $\{\mathrm{(i,j)}$  in ROADS  $\} > = 0$ $< =$  cap[i,j], from Intersection[i], to Intersection[j];

当模型以这种方式表示时，目标是最大化 `Traff_In`（或等价地最大化 `Traff_Out`）。我们可以通过在 `Traff_In` 的弧声明中添加 `obj` 短语来实现这一点，但在这种情况下，代数地定义目标可能更清晰：

```
maximize Entering_Traff: Traff_In;
```

此版本完整显示在图 15-12 中。

## 15.4 节点和弧声明的规则

通过示例定义了节点和弧之后，我们现在更全面地描述这些声明的必需和可选元素，并评论当两种声明都出现在同一模型中时，它们与常规声明 `minimize` 或 `maximize`、`subject to` 和 `var` 的交互。

### 节点声明

节点声明以关键字 `node`、名称、可选的索引表达式和冒号开始。冒号后面的表达式描述了节点处的平衡条件，可以具有以下任何形式：

```
net_expr  =  arith_expr
net_expr  <=  arith_expr
net_expr  >=  arith_expr
arith_expr  =  net_expr
arith_expr  <=  net_expr
arith_expr  >=  net_expr
arith_expr  <=  net_expr  <=  arith_expr
arith_expr  >=  net_expr  >=  arith_expr
```

其中 `arith_expr` 可以是使用先前声明的模型组件和当前定义的虚拟索引的任何算术表达式。`net_expr` 限制为以下之一：

```
± net_in
± net_out
± net_in  +  arith_expr
± net_out  +  arith_expr
arith_expr  ±  net_in
arith_expr  ±  net_out
```

（一元 `+` 可以省略）。以这种方式定义的每个节点都会在生成的线性规划中引入一个约束。在 AMPL 命令环境中，节点名称被视为约束名称，例如在 `display` 语句中。

对于使用 `net_in` 的声明，AMPL 通过将线性表达式（表示流入节点的流量减去流出节点的流量）代入到 `net_in` 出现在平衡条件中的位置来生成约束。对于使用 `net_out` 的声明也采用同样的处理方式，只不过此时 AMPL 代入的是流出减去流入的表达式。流入和流出的表达式是从弧声明中推断出来的。

### 弧声明

弧声明由关键字 `arc`、一个名称、一个可选的索引表达式以及一系列可选的限定短语组成。每条弧在生成的线性规划中创建一个变量，其值为沿该弧的流量；弧名称可用于在其他地方引用该变量。所有可能出现在 `var` 定义中的短语在弧定义中具有相同的意义；最常见的是使用 `>=` 和 `<=` 短语来指定弧上流量的 下界 (lower bound) 和 上界 (upper bound) 值。

`from` 和 `to` 短语用于指定由弧连接的节点。通常这些短语由关键字 `from` 或 `to` 后跟一个节点名称组成。一条弧被解释为对 `from` 节点的 净流出 (net outflow) 有贡献，同时对 `to` 节点的 净流入 (net inflow) 有贡献；这些解释使得与节点相关的约束可以被推断出来。

通常在 `arc` 声明中会指定一个 `from` 短语和一个 `to` 短语。然而，任何一个都可以省略，如图 15-12 所示。此外，每个短语后面也可以跟一个可选的索引表达式，该索引表达式应为以下两种类型之一：

一种索引表达式根据数据指定一个空集（此时 `from` 或 `to` 短语被忽略）或一个包含一个成员的集合（此时使用 `from` 或 `to` 短语）。一种特殊形式 `{if logical-expr}` 的索引表达式，只有当 `logical-expr` 计算结果为真时，才会使用 `from` 或 `to` 短语。

通过给出多个 `from` 或 `to` 短语，或者使用指定包含多个成员的集合的索引表达式，可以指定一条弧从两个或多个节点流出或流入流量。然而，这样得到的结果不是一个网络线性规划 (network linear program)，此时 AMPL 会显示一个适当的警告信息。

在 `from` 或 `to` 短语末尾，你可以添加一个表示乘以流量的因子的算术表达式，如我们在 15.3 节中关于运输损耗和单位变化的例子所示。如果因子在 `to` 短语中，则在确定流入指定节点的流量时，该因子会乘以弧变量；也就是说，对于沿弧的给定流量，等于 `to` 因子乘以流量的量被视为进入 `to` 节点。`from` 短语中的因子按类似方式解释。默认因子为 1。

可选的 `obj` 短语指定了一个系数，该系数将乘以弧变量，以在指定的目标函数中创建线性项。这样的短语由关键字 `obj`、先前在 `minimize` 或 `maximize` 声明中定义的目标函数名称，以及系数值的算术表达式组成。关键字后面可以跟一个索引表达式，其解释方式与 `from` 和 `to` 短语相同。

# 与目标声明的交互

如果目标函数中的所有项都通过弧声明中的 `obj` 短语指定，则目标的声明仅仅是 `minimize` 或 `maximize`，后跟可选的索引表达式和名称。此声明必须出现在引用该目标的弧声明之前。

或者，弧名称可以作为变量以通常的代数方式指定目标函数。在这种情况下，目标必须在弧之后声明，如图 15-12 所示。

```
set CITIES;
set LINKS within (CITIES cross CITIES);
set PRODS;

param supply {CITIES,PRODS} >= 0;    # 城市可用数量
param demand {CITIES,PRODS} >= 0;    # 城市需求数量

check {p in PRODS}:
   sum {i in CITIES} supply[i,p] = sum {j in CITIES} demand[j,p];

param cost {LINKS,PRODS} >= 0;       # 运输成本 / 1000 包
param capacity {LINKS,PRODS} >= 0;   # 最大运输包数
param cap_joint {LINKS} >= 0;        # 每链接最大总运输包数

minimize Total_Cost;

node Balance {k in CITIES, p in PRODS}:
   net_in = demand[k,p] - supply[k,p];

arc Ship {(i,j) in LINKS, p in PRODS} >= 0,
   <= capacity[i,j,p],
   from Balance[i,p], to Balance[j,p],
   obj Total_Cost cost[i,j,p];

subject to Multi {(i,j) in LINKS}:
   sum {p in PRODS} Ship[i,j,p] <= cap_joint[i,j];
```

图 15-13：带侧约束的多产品流 (netmulti.mod)。

# 与约束声明的交互

弧声明中定义的组件可以用作附加 subject to 声明中的变量。后者表示除节点流量平衡外施加的“侧约束” (side constraints)。

例如，考虑如何从图 15-10 中的节点和弧网络公式构建多产品 (Multi-product) 流问题。按照第 4.1 节中的方法，我们引入一组不同的产品 PRODS，并将其添加到所有参数、节点和弧的索引中。结果是每个产品都有一个独立的网络线性规划 (network linear program)，目标函数是所有产品的成本总和。为了将这些网络联系在一起，我们提供了沿任何链接 (LINKS) 的总运输量的联合限制：

param cap_joint {LINKS} >= 0; subject to Multi {(i,j) in LINKS}: sum {p in PRODS} Ship[p,i,j] <= cap_joint[i,j];

最终模型如图 15-13 所示，不是网络线性规划，但其网络和非网络部分被清晰地分离。

# 与变量声明的交互

正如弧变量可以在 subject to 声明中使用一样，普通的 var 变量也可以在节点声明中使用。也就是说，节点声明中的平衡条件可以包含对先前 var 声明定义的变量的引用。这些引用定义了网络线性规划 (network linear program) 的“侧变量” (side variables)。

作为一个例子，我们再次在集合 PRODS 上复制图 15-10 中的公式。这一次，我们通过引入一组原料 (feedstocks) 及相关数据将网络连接在一起：

set FEEDS;  
param yield {PRODS,FEEDS} >= 0;  
param limit {FEEDS,CITIES} >= 0;

我们可以想象，在城市 $\mathbf{k}$，除了可用于运输的产品供应量 supply[p, k] 外，最多还可以将 limit $[\mathbf{f},\mathbf{k}]$ 的原料 f 转化为产品；一单位原料 f 可以产生 yield[p, f] 单位的每种产品 p。变量 Feed[f, k] 表示在城市 k 使用的原料 f 的数量：

var Feed {f in FEEDS, k in CITIES} >= 0, <= limit[f,k];

此时，产品 $\mathbb{P}$ 在城市 $\mathbf{k}$ 的平衡条件可以表示为：净流出 (net_out) 等于净供应量加上从各种原料中衍生出的数量之和：

node Balance {p in PRODS, k in CITIES}:  
net_out $=$ supply[p,k] - demand[p,k] $+$ sum {f in FEEDS} yield[p,f] \* Feed[f,k];

弧变量保持不变，最终模型如图 15-14 所示。在给定城市 $\mathbf{k}$，变量 Feed[f, k] 出现在所有不同产品的节点平衡条件中，从而将产品网络整合成一个单一的线性规划 (linear program) 问题。

# 15.5 求解网络线性规划

本章中我们描述的所有模型都会产生某种具有“网络”特性的线性规划 (linear programming) 问题。AMPL 可以将这些线性规划问题发送给 LP 求解器 (solver)，并取回最优值，就像处理其他类型的 LP 问题一样。如果你以这种方式使用 AMPL，网络结构主要有助于模型的构建和结果的解释。

我们描述的许多模型也属于更狭义的一类问题，这类问题（令人困惑地）也被称为“网络线性规划” (network linear programs)。从建模角度来看，网络 LP 的变量必须表示网络弧上的流量，而约束只能是两种类型：对弧上流量的限制，以及对节点处流出量减去流入量的限制。更技术性地说，网络线性规划中的每个变量最多只能出现在两个约束中（除了变量的上下界外），且该变量在最多一个约束中的系数为 $+1$，在最多一个约束中的系数为 $-1$。

这种狭义的“纯”网络线性规划具有非常强的特性，使得它们的使用特别理想。只要供应量、需求量和

城市集合 CITIES；  
链接集合 LINKS，属于 (CITIES × CITIES) 的子集；  
产品集合 PRODS；  
参数 supply {PRODS, CITIES} ≥ 0；# 城市可提供的供应量  
参数 demand {PRODS, CITIES} ≥ 0；# 城市所需的需求量  
校验条件 {p in PRODS}：sum {i in CITIES} supply[p,i] = sum {j in CITIES} demand[p,j]；  
参数 cost {PRODS, LINKS} ≥ 0；# 每 1000 包产品的运输成本  
参数 capacity {PRODS, LINKS} ≥ 0；# 产品在链接上的最大运输包数  
原料集合 FEEDS；  
参数 yield {PRODS, FEEDS} ≥ 0；# 从原料中提取的产品量  
参数 limit {FEEDS, CITIES} ≥ 0；# 城市可用的原料量  
最小化 总成本 Total Cost；  
变量 Feed {f in FEEDS, k in CITIES} ≥ 0，≤ limit[f,k]；  
节点 Balance {p in PRODS, k in CITIES}：net_out = supply[p,k] - demand[p,k] + sum {f in FEEDS} yield[p,f] * Feed[f,k]；  
弧 Ship {p in PRODS, (i,j) in LINKS} ≥ 0，≤ capacity[p,i,j]，从 Balance[p,i] 到 Balance[p,j]，目标函数 TotalCost 的系数为 cost[p,i,j]；

如果所有变量的上下界均为整数，则网络线性规划 (network linear program) 必然存在一个所有流均为整数的最优解 (optimal solution)。此外，如果所用的求解器 (solver) 是一种寻找“极值”解的类型（例如基于单纯形法 (simplex method) 的求解器），它将始终能找到一个所有变量均为整数的最优解。在最短路径问题和某些指派问题中，我们已经在未明确说明的情况下利用了这一性质，假设变量的取值只能是零或一，而不会是介于两者之间的分数值。

网络线性规划可以通过专门利用网络结构的求解器，比其他同规模的线性规划 (linear program) 更快地求解。如果您使用节点 (node) 和弧 (arc) 声明来编写模型，AMPL 会自动将网络结构传递给求解器，从而自动应用求解器中任何可用的特殊网络算法。另一方面，如果使用 var 和 subject to 以代数模型 (algebraic model) 的方式表达网络，求解器可能无法识别该结构，某些情况下可能需要设置特定选项以确保其被识别。例如，在使用图 15-4a 中的代数模型时，您可能会看到通用 LP 算法的常规响应：

```
ampl: model net3.mod;
data net3.dat;
solve;
CPLEX 8.0.0: 最优解; 目标函数 1819
1 次对偶单纯形法迭代 (第 I 阶段 0 次)
```

但当使用图 15-11 中等价的 node 和 arc 形式时，您可能会得到一个稍有不同的响应，以反映特殊网络线性规划 (LP) 算法的应用：

```
ampl: model net3node.mod;
data net3.dat;
solve;
CPLEX 8.0.0: 最优解; 目标函数 1819
网络提取器发现 7 个节点和 7 条弧。
7 次网络单纯形法迭代。
0 次单纯形法迭代 (第 1 阶段 0 次)
```

要了解您常用的求解器在这种情况下如何表现，请查阅随您的 AMPL 安装包提供的求解器特定文档。

由于网络线性规划 (linear program) 更容易求解，特别是当数据为整数时，大规模应用的成功可能取决于是否可以建立一个纯网络模型。例如，在图 15-13 的多产品 (Multi) 流模型中，联合容量约束 (constraints) 破坏了网络结构 —— 它们代表了每个变量 (variable) 都参与的第三个约束 —— 但在正确表示问题时无法避免这些约束的存在。因此，多产品流问题不一定有整数解，并且通常比同等规模的单产品流问题更难求解。

在某些情况下，通过巧妙的重新建模，可以将看似更一般的模型转化为纯网络模型。例如，考虑图 15-10 的一个推广版本，其中节点和弧上都定义了容量：

```python
param city_cap {CITIES} >= 0;
param link_cap {LINKS} >= 0;
```

弧容量如前所述，表示城市之间运输量 (shipments) 的上限。节点容量则限制了吞吐量，即在某个城市处理的总流量，可以表示为该城市的供应量 (supply) 加上流入流量之和，或者等价地表示为该城市的 需求量 (demand) 加上流出流量之和。采用前者，我们得到如下约束：

```python
subject to through_limit {k in CITIES}:
  supply[k] + sum{(i,k) in LINKS} Ship[i,k] <= node_cap[k];
```

从这个角度看，吞吐量限制是另一个“附加约束” (side constraint) 的例子，它通过为每个变量增加第三个系数来破坏网络结构。但我们可以通过使用两个节点来表示每个城市，在不引入附加约束的情况下达到相同的效果：一个节点接收流入城市的流量以及任何供应量，另一个节点发送流出城市的流量以及任何需求量：

```python
node Supply {k in CITIES}: net_out = supply[k];
node Demand {k in CITIES}: net_in = demand[k];
```

城市 i 和 j 之间的运输链接由一条连接节点 Demand[i] 到节点 Supply[j] 的弧表示：

集合 CITIES; 集合 LINKS 属于 (CITIES 叉积 CITIES); 参数 supply {CITIES} >= 0; # 城市的可用量 参数 demand {CITIES} >= 0; # 城市的需求量 check: sum {i in CITIES} supply[i] = sum {j in CITIES} demand[j]; 参数 cost {LINKS} >= 0; # 每吨运输 成本 参数 city_cap {CITIES} >= 0; # 城市的最大吞吐量 参数 link_cap {LINKS} >= 0; # 链接上的最大运输量 最小化 Total_Cost; 节点 Supply {k in CITIES}: net_out = supply[k]; 节点 Demand {k in CITIES}: net_in = demand[k]; 弧 Ship {(i,j) in LINKS} >= 0, <= link_cap[i,j], from Demand[i], to Supply[j], obj Total_Cost cost[i,j]; 弧 Through {k in CITIES} >= 0, <= city_cap[k], from Supply[k], to Demand[k];

图 15-15：带节点容量的转运模型 (netthru.mod)。

弧 Ship {(i,j) in LINKS} >= 0, <= link_cap[i,j], from Demand[i], to Supply[j], obj Total_Cost cost[i,j]; from Supply[k], to Demand[k];

城市 $\mathbf{k}$ 的吞吐量由一种新的弧表示，从 Supply[k] 到 Demand[k]：

弧 Through {k in CITIES} >= 0, <= city_cap[k], from Supply[k], to Demand[k];

吞吐量限制现在由该弧流量的上界表示，而不是由侧约束表示，模型的网络结构得以保持。完整列表见图 15-15。

前面的例子展示了在开发网络模型时使用节点和弧声明的另一个优势。如果你仅以简单形式使用节点和弧——节点条件中没有变量，from 和 to 短语中没有可选因子——你的模型保证只产生纯网络线性规划。相比之下，如果你使用 var 和 subject to，则有责任确保生成的线性规划具有必要的网络结构。

前面的一些评论可以扩展到"广义网络"线性规划，其中每个变量仍然最多出现在两个约束中，但系数不一定为 $+1$ 和 $-1$。我们已经在弧上存在流量损失或单位变化的情况下看到了广义网络的例子。广义网络线性规划不一定有整数最优解，但确实存在快速算法。承诺提供"网络"算法的求解器可能或可能没有扩展到广义网络；在做出任何假设之前，请检查求解器特定的文档。

# 参考文献

[1] Ravindra K. Ahuja, Thomas L. Magnanti, and James B. Orlin. Network Flows: Theory, Algorithms, and Applications. Prentice-Hall (Englewood Cliffs, NJ, 1993).

[2] Dimitri P. Bertsekas. Network Optimization: Continuous and Discrete Models. Athena Scientific (Princeton, NJ, 1998).

[3] L. R. Ford, Jr. and D. R. Fulkerson. Flows in Networks. Princeton University Press (Princeton, NJ, 1962). 一篇关于网络线性规划和相关主题的高度有影响力的综述，刺激了大量后续研究。

[4] Fred Glover, Darwin Klingman, and Nancy V. Phillips. Network Models in Optimization and their Applications in Practice. John Wiley & Sons (纽约, 1992).

[5] Walter Jacobs. "The Caterer Problem." Naval Research Logistics Quarterly 1 (1954) 第 154-165 页。练习 15-8 中描述的网络问题的起源。

Katta G. Murty，《Network Programming》。Prentice-Hall（Englewood Cliffs, NJ, 1992）。

# 练习

15-1. 下图可以解释为表示一个网络转运问题：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/c1afd2d32c4732147956b869ba89fdf96fb23aff6b54ca22d5e852622e3a8b09.jpg)

指向节点 A 和 B 的箭头表示指定数量的供应量，分别为 100 和 50；从节点 E 和 F 指出的箭头类似地表示数量为 30 和 120 的需求量。其余箭头表示运输可能性，上面的数字是单位运输成本。每条弧的容量为 80。

(a) 通过开发适当的数据语句来解决此问题，数据语句应与图 15-2a 的模型配合使用。  
(b) 使用图 15-10 的节点和弧线公式检查是否得到相同的解。求解器是否似乎使用了与 (a) 中相同的算法？请使用你可用的每个 LP 求解器进行此比较。

15-2. 重新解释上一练习图中节点之间弧线上的数字，以解决以下问题。（忽略指向 A 和 B 的箭头以及从 E 和 F 指出的箭头上的数字。）

(a) 将节点之间弧线上的数字视为长度，使用如图 15-7 所示的模型找出从 A 到 F 的最短路径。  
(b) 将节点之间弧线上的数字视为容量，使用如图 15-6 所示的模型找出从 A 到 F 的最大流。  
(c) 推广 (b) 中的模型，使其能够在某种有意义的情况下找出从任意节点子集到另一任意节点子集的最大流。使用你的推广模型找出从 A 和 B 到 E 和 F 的最大流。

15-3. 第 4.2 节展示了如何通过在时间周期上复制静态模型，并使用库存将周期连接起来，从而构建一个多周期模型。考虑将相同的方法应用于图 15-4a 的专门转运模型。

(a) 构建图 15-4a 的多周版本，其中库存保存在配送中心（D_CITY 的成员）中。  
(b) 现在考虑为此模型扩展图 15-4b 的数据。假设初始库存为 NE 处 200，SE 处 75，且每 1000 包的库存持有成本为 NE 处每周 0.15，SE 处每周 0.12。设 5 周内的供应量和需求量如下：

<table><tr><td>周</td><td>供应量</td><td>BOS</td><td>EWR</td><td>BWI</td><td>ATL</td><td>MCO</td></tr><tr><td>1</td><td>450</td><td>50</td><td>90</td><td>95</td><td>50</td><td>20</td></tr><tr><td>2</td><td>450</td><td>65</td><td>100</td><td>105</td><td>50</td><td>20</td></tr><tr><td>3</td><td>400</td><td>70</td><td>100</td><td>110</td><td>50</td><td>25</td></tr><tr><td>4</td><td>250</td><td>70</td><td>110</td><td>120</td><td>50</td><td>40</td></tr><tr><td>5</td><td>325</td><td>80</td><td>115</td><td>120</td><td>55</td><td>45</td></tr></table>

保持成本和容量数据不变，在所有周期中保持一致。为此情况开发适当的数据文件，并解决多周问题。  
(c) 在这种情况下，多周模型可以被视为一个纯网络模型，其中弧线代表库存以及运输量。为了证明这一点，请仅使用节点和弧线声明重新表述 (a) 中的模型以表示约束和变量。

15-4. 对于以下每个网络问题，构建一个数据文件，使其能够使用图 15-2a 的通用转运模型进行求解。

(a) 图 3-1 的运输问题 (transportation problem)。  
(b) 图 3-2 的指派问题 (assignment problem)。  
(c) 图 15-6 的最大流问题 (maximum flow problem)。  
(d) 图 15-7 的最短路问题 (shortest path problem)。

15-5. 尽可能使用节点 (node) 和弧 (arc) 声明来重新表述以下网络模型：

(a) 图 6-2a 的运输模型 (transportation model)。  
(b) 图 15-7 的最短路模型 (shortest path model)。  
(c) 图 4-6 的生产/运输模型 (production/transportation model)。  
(d) 图 6-5 的多商品运输模型 (multicommodity transportation model)。

15-6. 某工业工程设计课程的教授面临将 28 名学生分配到八个项目的任务。每名学生必须被分配到一个项目，每个项目组必须包含 3 或 4 名学生。学生们已被要求对项目进行排序，其中 1 为最佳排名，数字越大表示排名越低。

(a) 使用 var 和 subject to 声明为此问题建立一个代数指派模型 (assignment model)。

(b) 根据以下排名表求解该指派问题：

<table><tr><td></td><td>A</td><td>ED</td><td>EZ</td><td>G</td><td>H1</td><td>H2</td><td>RB</td><td>SC</td><td></td><td>A</td><td>ED</td><td>EZ</td><td>G</td><td>H1</td><td>H2</td><td>RB</td><td>SC</td></tr><tr><td>艾伦</td><td>1</td><td>3</td><td>4</td><td>7</td><td>7</td><td>5</td><td>2</td><td>6</td><td>诺尔</td><td>7</td><td>4</td><td>1</td><td>2</td><td>2</td><td>5</td><td>6</td><td>3</td></tr><tr><td>布莱克</td><td>6</td><td>4</td><td>2</td><td>5</td><td>5</td><td>7</td><td>1</td><td>3</td><td>曼海姆</td><td>4</td><td>7</td><td>2</td><td>1</td><td>1</td><td>3</td><td>6</td><td>5</td></tr><tr><td>钟</td><td>6</td><td>2</td><td>3</td><td>1</td><td>1</td><td>7</td><td>5</td><td>4</td><td>莫里斯</td><td>7</td><td>5</td><td>4</td><td>6</td><td>6</td><td>3</td><td>1</td><td>2</td></tr><tr><td>克拉克</td><td>7</td><td>6</td><td>1</td><td>2</td><td>2</td><td>3</td><td>5</td><td>4</td><td>内森</td><td>4</td><td>7</td><td>5</td><td>6</td><td>6</td><td>3</td><td>1</td><td>2</td></tr><tr><td>康纳斯</td><td>7</td><td>6</td><td>1</td><td>3</td><td>3</td><td>4</td><td>5</td><td>2</td><td>诺曼</td><td>7</td><td>5</td><td>4</td><td>6</td><td>6</td><td>3</td><td>1</td><td>2</td></tr><tr><td>卡明</td><td>6</td><td>7</td><td>4</td><td>2</td><td>2</td><td>3</td><td>5</td><td>1</td><td>帕特里克</td><td>1</td><td>7</td><td>5</td><td>4</td><td>4</td><td>2</td><td>3</td><td>6</td></tr><tr><td>德明</td><td>2</td><td>5</td><td>4</td><td>6</td><td>6</td><td>1</td><td>3</td><td>7</td><td>罗林斯</td><td>6</td><td>2</td><td>3</td><td>1</td><td>1</td><td>7</td><td>5</td><td>4</td></tr><tr><td>英</td><td>4</td><td>7</td><td>2</td><td>1</td><td>1</td><td>6</td><td>3</td><td>5</td><td>舒曼</td><td>4</td><td>7</td><td>3</td><td>5</td><td>5</td><td>1</td><td>2</td><td>6</td></tr><tr><td>法默</td><td>7</td><td>6</td><td>5</td><td>2</td><td>2</td><td>1</td><td>3</td><td>4</td><td>西尔弗</td><td>4</td><td>7</td><td>3</td><td>1</td><td>1</td><td>2</td><td>5</td><td>6</td></tr><tr><td>福雷斯特</td><td>6</td><td>7</td><td>2</td><td>5</td><td>5</td><td>1</td><td>3</td><td>4</td><td>施泰因</td><td>6</td><td>4</td><td>2</td><td>5</td><td>5</td><td>7</td><td>1</td><td>3</td></tr><tr><td>古德曼</td><td>7</td><td>6</td><td>2</td><td>4</td><td>4</td><td>5</td><td>1</td><td>3</td><td>斯托克</td><td>5</td><td>2</td><td>1</td><td>6</td><td>6</td><td>7</td><td>4</td><td>3</td></tr><tr><td>哈里斯</td><td>4</td><td>7</td><td>5</td><td>3</td><td>3</td><td>1</td><td>2</td><td>6</td><td>杜鲁门</td><td>6</td><td>3</td><td>2</td><td>7</td><td>7</td><td>5</td><td>1</td><td>4</td></tr><tr><td>霍姆斯</td><td>6</td><td>7</td><td>4</td><td>2</td><td>2</td><td>3</td><td>5</td><td>1</td><td>沃尔曼</td><td>6</td><td>7</td><td>4</td><td>2</td><td>2</td><td>3</td><td>5</td><td>1</td></tr><tr><td>约翰逊</td><td>7</td><td>2</td><td>4</td><td>6</td><td>6</td><td>5</td><td>3</td><td>1</td><td>杨</td><td>1</td><td>3</td><td>4</td><td>7</td><td>7</td><td>6</td><td>2</td><td>5</td></tr></table>

有多少学生被分配到第二或第三选择？

(c) 有些项目在没有车的情况下比其他项目更难到达。因此，每个项目至少有一定数量被分配的学生必须有车；各项目的数量如下：

<table><tr><td>A</td><td>1</td><td>ED</td><td>0</td><td>EZ</td><td>0</td><td>G</td><td>2</td><td>H1</td><td>2</td><td>H2</td><td>2</td><td>RB</td><td>1</td><td>SC</td><td>1</td></tr></table>

有车的学生是：

<table><tr><td>Chung</td><td>Eng</td><td>Manheim</td><td>Nathan</td><td>Rollins</td></tr><tr><td>Demming</td><td>Holmes</td><td>Morris</td><td>Patrick</td><td>Young</td></tr></table>

修改模型以添加这个车的约束，并重新解决问题。比以前多多少学生必须被分配到第二或第三选择？

(d) 你在 (c) 中的公式化可以看作是带有附加约束的运输模型。通过定义适当的网络节点和弧，将其重新公式化为"纯"网络流模型，如第 15.5 节所述。使用 AMPL 编写公式，仅使用节点和弧声明来表示约束和变量。使用与 (c) 中相同的数据求解，以证明最优值是相同的。

15-7. 为了管理未来 12 个月的 excess cash，公司可以从几家不同银行购买 1 个月、2 个月或 3 个月的存单。当前的现金和已投资金额是已知的，而公司必须估计每个月的现金收入和支出，以及不同存单的回报。

公司的问题是确定最佳投资策略，受现金需求约束。（实际上，公司会将最优解的第一个月作为当前购买的指导，然后在下个月初用更新的估计重新求解。）

(a) 为此情况绘制网络图。将每个月显示为一个节点，投资、收入和支出显示为弧。

(b) 将相关的优化问题公式化为 AMPL 模型，使用节点和弧声明。假设任何先前购买的存单在早期月份到期的现金都包含在收入数据中。

对于此模型的目标函数有多种描述方式。解释你的选择。

(c) 假设公司对未来 12 个月的估计收入和支出（以千美元计）如下：

构建适当的数据文件，并求解由此产生的线性规划 (linear program)。使用 display 生成所指示购买的摘要。

(d) 公司政策禁止在任何一个月内将超过 $70\%$ 的现金投资于任何一家银行的新存单。设计一个从 (b) 中模型得出的附加约束 (side constraint) 来施加这一限制。

再次求解由此产生的线性规划，并总结所指示的购买。由于这一限制性政策，损失了多少收入？

15-8. 一位餐饮服务商已经预订了接下来 T 天的晚宴，因此每天对餐巾数量都有一定需求。他目前有一定数量的餐巾库存，每天都可以按一定价格购买新的餐巾。此外，用过的餐巾可以选择两种清洗服务：一种是较慢的服务，需要 4 天时间；另一种是更快但更昂贵的服务，需要 2 天时间。餐饮服务商的问题是，找到购买和清洗餐巾的最经济组合，以满足未来的餐巾需求。

(a) 不难看出，这个问题的决策变量应如下所示：

Buy[t] 为第 t 天购买的干净餐巾数量  
Carry[t] 第 t 天结束时仍持有的干净餐巾数量  
Wash2[t] 第 t 天结束后送往快速洗衣服务的用过餐巾数量  
Wash4[t] 第 t 天结束后送往慢速洗衣服务的用过餐巾数量  
Trash[t] 第 t 天结束后丢弃的用过餐巾数量  

这些变量受到两类约束，描述如下：

- 第 t 天通过购买、结余和清洗获得的干净餐巾数量必须等于第 t 天结束后送往清洗、丢弃或结余的餐巾数量。
- 第 t 天结束后清洗或丢弃的用过餐巾数量必须等于当天餐饮所需的餐巾数量。

请为这个问题建立一个 AMPL 线性规划模型。

(b) 请为这个问题建立一个替代的网络线性规划模型。使用节点和弧的声明方式在 AMPL 中编写该模型。

(c) “餐饮服务商问题”最初由美国空军的沃尔特·雅各布斯（Walter Jacobs）在 1954 年的一篇论文中提出。尽管这个问题在无数本关于线性规划和网络规划的书籍中都有介绍，但似乎从未被任何实际的餐饮服务商使用过。你认为这个问题最初来源于哪种实际应用？

(d) 由于这是一个虚构的问题，你可以自行设定相关数据。使用你设定的数据验证 (a) 和 (b) 中的模型是否得到相同的最优值。
