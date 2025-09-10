# 运输与指派模型

第 1 章和第 2 章中的线性规划都是经典的“活动”模型 (classical "activity" models) 的示例。在这些模型中，变量和约束处理的是明显不同种类的活动——例如生产的钢材吨数与使用的轧机时间小时数，或者购买的食物包数与提供的营养百分比。要使用这些模型，你必须提供诸如吨/小时或百分比/包这样的系数，将变量中的活动单位转换为约束中的相应活动量。

本章讨论一种显著不同但同样常见的模型，其中某种物品被运输或指派，但不发生转化。由此产生的约束反映了可用性的限制和交付需求，具有特别简单的形式。

我们首先描述所谓的运输问题 (transportation problem)，其中单一商品需从多个起始地 (origins) 运送到多个目的地 (destinations)，以实现总体成本最小化 (minimum overall cost)。该问题引出了最简单的最小成本流 (minimum-cost flows) 线性规划类型。接着我们将其推广为运输模型 (transportation model)，这是有效管理所有数据、变量和约束的关键步骤。

与饮食模型 (diet model) 一样，运输模型的力量在于其适应性 (adaptability)。我们继续考虑从起始地到目的地的“流” (flow) 的其他一些解释，并详细探讨其中一种特定解释，其中变量表示指派 (assignments) 而非运输量 (shipments)。

运输模型只是最基础的最小成本流模型 (minimum-cost flow model)。更一般的模型通常最好表示为网络 (networks)，其中节点 (nodes)（其中一些可能是起始地或目的地）通过传输某种流的弧 (arcs) 相连接。AMPL 提供了便于描述网络流模型的特性，包括可直接指定网络结构的节点和弧声明 (node and arc declarations)。网络模型及相关 AMPL 特性将在第 15 章中讨论。

# 3.1 运输问题的线性规划

假设我们 (可能通过第 1 章所述的方法) 决定在以下三个轧机地点生产钢卷 (steel coils)：

加里 (GARY) 印第安纳州 1400  
克利夫兰 (CLEV) 俄亥俄州 2600  
匹兹堡 (PITT) 宾夕法尼亚州 2900  

总计 6900 吨 必须以不同数量运输，以满足七个汽车工厂地点的订单需求：

弗雷明汉 (FRA) 马萨诸塞州 900 底特律 (DET) 密歇根州 1200 兰辛 (LAN) 密歇根州 600 温莎 (WIN) 安大略省 400 圣路易斯 (STL) 密苏里州 1700 弗里蒙特 (FRE) 加利福尼亚州 1100 拉斐特 (LAF) 印第安纳州 1000

我们现在有一个优化问题：从工厂到工厂运输线圈的最便宜计划是什么？

要回答这个问题，我们需要编制一张每吨运输成本的表格：

<table><tr><td></td><td>GARY</td><td>CLEV</td><td>PITT</td></tr><tr><td>FRA</td><td>39</td><td>27</td><td>24</td></tr><tr><td>DET</td><td>14</td><td>9</td><td>14</td></tr><tr><td>LAN</td><td>11</td><td>12</td><td>17</td></tr><tr><td>WIN</td><td>14</td><td>9</td><td>13</td></tr><tr><td>STL</td><td>16</td><td>26</td><td>28</td></tr><tr><td>FRE</td><td>82</td><td>95</td><td>99</td></tr><tr><td>LAF</td><td>8</td><td>17</td><td>20</td></tr></table>

令 GARY:FRA 表示从 GARY 运输到 FRA 的吨数，其他城市对也类似。那么目标函数可以写成如下形式：

最小化

39 GARY:FRA + 27 CLEV:FRA + 24 PITT:FRA + 14 GARY:DET + 9 CLEV:DET + 14 PITT:DET + 11 GARY:LAN + 12 CLEV:LAN + 17 PITT:LAN + 14 GARY:WIN + 9 CLEV:WIN + 13 PITT:WIN + 16 GARY:STL + 26 CLEV:STL + 28 PITT:STL + 82 GARY:FRE + 95 CLEV:FRE + 99 PITT:FRE + 8 GARY:LAF + 17 CLEV:LAF + 20 PITT:LAF

总共有 21 个决策变量。即使像这样的小型运输问题也有很多变量，因为每个工厂和工厂的组合都有一个变量。

通过从可以最便宜地运输到工厂的工厂供应每个工厂，我们可以实现最低的运输成本。但这样我们将从 PITT 运输 900 吨，从 CLEV 运输 1600 吨，其余的从 GARY 运输——这些数量与之前决定的生产水平完全不一致。我们需要添加一个约束，即从 GARY 到七个工厂的运输总量等于 1400 的生产水平：

GARY:FRA \(^+\) GARY:DET \(^+\) GARY:LAN \(^+\) GARY:WIN \(^+\) GARY:STL \(^+\) GARY:FRE \(^+\) GARY:LAF \(= 1400\)

其他两个工厂也有类似的约束：

CLEV:FRA \(^+\) CLEV:DET \(^+\) CLEV:LAN \(^+\) CLEV:WIN \(^+\) CLEV:STL \(^+\) CLEV:FRE \(^+\) CLEV:LAF \(= 2600\) PITT:FRA \(^+\) PITT:DET \(^+\) PITT:LAN \(^+\) PITT:WIN \(^+\) PITT:STL \(^+\) PITT:FRE \(^+\) PITT:LAF \(= 2900\)

在工厂也需要类似的约束，以确保运输量等于订购量。在 FRA，从三个工厂收到的运输总量必须等于订购的 900 吨：

$$
\mathrm{GARY:FRA + CLEV:FRA + PITT:FRA = 900}
$$

其他六个工厂也类似：

加里:底特律 \(^+\) 克利夫兰:底特律 \(^+\) 匹兹堡:底特律 \(= 1200\)  
加里:兰辛 \(^+\) 克利夫兰:兰辛 \(^+\) 匹兹堡:兰辛 \(= 600\)  
加里:温莎 \(^+\) 克利夫兰:温莎 \(^+\) 匹兹堡:温莎 \(= 400\)  
加里:STL \(^+\) 克利夫兰:STL \(^+\) 匹兹堡:STL \(= 1700\)  
加里:FRE \(^+\) 克利夫兰:FRE \(^+\) 匹兹堡:FRE \(= 1100\)  
加里:拉斐特 \(^+\) 克利夫兰:拉斐特 \(^+\) 匹兹堡:拉斐特 \(= 1000\)

我们总共有十个约束，每个钢厂和每个工厂各一个。如果再加上所有变量都必须为非负的要求，我们就得到了一个完整的运输问题线性规划 (linear program) 模型。

我们甚至不会尝试展示如果将所有这些约束输入到 AMPL 模型文件中会是什么样子。显然，我们希望建立一个通用模型来处理这个问题。

# 3.2 运输问题的 AMPL 模型

运输问题背后存在两组基本的对象集合：供应源或起始地 (在我们的例子中是钢厂) 和目的地 (工厂)。因此，我们在 AMPL 模型中以这两个集合的声明开始：

```ampl
set ORIG; set DEST;
```

在每个起始地 (在我们的情况下是生产的钢卷吨数) 都有某种物资的供应量，在每个目的地都有对相同物资的需求量 (订购的钢卷吨数)。AMPL 使用带集合索引的 param 语句来定义像这样的非负量；在本例中，我们增加了一个额外的改进，即使用 check 语句来检验数据的有效性：

```ampl
param supply {ORIG} >= 0; param demand {DEST} >= 0;

check: sum {i in ORIG} supply[i] = sum {j in DEST} demand[j];
```

check 语句表示供应量总和必须等于需求量总和。按照我们模型的设置方式，除非满足这个条件，否则根本不可能有任何解。通过将其放在 check 语句中，我们告诉 AMPL 在读取数据后测试这个条件，如果违反了该条件就发出错误消息。

对于每个起始地和目的地的组合，都有一个运输成本和一个表示运输量的变量。同样，前几章的思想很容易被改编以产生适当的 AMPL 语句：

```ampl
param cost {ORIG,DEST} >= 0; var Trans {ORIG,DEST} >= 0;
```

对于特定的起始地 i 和目的地 j，我们从 i 向 j 运输 Trans[i,j] 单位，每单位成本为 cost[i,j]；这对组合的总成本是 cost[i,j] * Trans[i,j]。

对所有组合求和，我们得到目标函数：

```ampl
minimize Total_Cost: sum {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];
```

也可以写成

```ampl
sum {j in DEST, i in ORIG} cost[i,j] * Trans[i,j];
```

或者写成

只要您以某种数学上正确的方式表达目标函数，AMPL 就会整理这些项。

剩下的就是指定两组约束：起始地的约束和目的地的约束。如果我们把这两组约束命名为 Supply 和 Demand，它们的声明将如下开始：

```ampl
subject to Supply {i in ORIG}: subject to Demand {j in DEST}
```

为了完成起始地 i 的 Supply 约束，我们需要说明从 i 运出的所有货物量之和等于可用的供应量。由于从 i 运往特定目的地 j 的运输量是 Trans[i,j]，运往所有目的地的运输量必须是

```python
set ORIG; # 起始地
set DEST; # 目的地
param supply {ORIG} >= 0; # 起始地的供应量
param demand {DEST} >= 0; # 目的地的需求量
check: sum {i in ORIG} supply[i] = sum {j in DEST} demand[j];
param cost {ORIG,DEST} >= 0; # 单位运输成本
var Trans {ORIG,DEST} >= 0; # 运输量

minimize Total_Cost: sum {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];

subject to Supply {i in ORIG}: sum {j in DEST} Trans[i,j] = supply[i];
subject to Demand {j in DEST}: sum {i in ORIG} Trans[i,j] = demand[j];

sum {j in DEST} Trans[i,j]

由于我们已经定义了一个在起始地上索引的参数 supply，因此 i 处的可用量为 supply[i]。于是约束为
subject to Supply {i in ORIG}: sum {j in DEST} Trans[i,j] = supply[i];

（请注意，名称 supply 和 Supply 并无关联；AMPL 区分大小写。）另一组约束与此类似，只是 i 属于 ORIG 和 j 属于 DEST 的角色互换，且求和等于 demand[j]。

我们现在可以展示完整的运输模型，如图 3-1a 所示。你可能已经注意到，我们始终使用索引 i 遍历集合 ORIG，索引 j 遍历集合 DEST。这并非 AMPL 的要求，但这种约定使模型更易于阅读。你可以随意命名自己的索引，但请记住索引的作用域（即模型中具有相同含义的部分）是从定义它的表达式开始到结束。因此，在需求约束
subject to Demand {j in DEST}: sum {i in ORIG} Trans[i,j] = demand[j];
中，j 的作用域延伸到声明末尾的分号，而 i 的作用域仅限于求和项 Trans[i,j]。由于 i 的作用域在 j 的作用域内部，这两个索引必须具有不同的名称。此外，索引不得与集合或其他模型组件同名。索引作用域将在第 5.5 节中通过更多示例进行详细讨论。

运输模型的数据值如图 3-1b 所示。为了定义 DEST 和 demand，我们使用了一种输入格式，允许将集合及其索引的一个或多个参数一起指定。集合名称用冒号包围。（我们还展示了一些注释，它们可以像在模型中一样出现在数据语句之间。）

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/19796322ceb0f49317073bc1f68feee18b4a5058e7824e24a8191b6369eeec17.jpg)
图 3-1b：运输模型的数据 (transp.dat)。

如果模型存储在文件 transp.mod 中，数据存储在 transp.dat 中，我们可以求解该线性规划并查看输出：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/ae3ad6ab3a6d8f2c3cbff5087d356f2a37542e5fbeaa1ff07c85e86673144849.jpg)

通过显示变量 Trans，我们看到大多数目的地仅由一个工厂供应，但 CLEV、GARY 和 PITT 都向 LAF 运输。

将此解与另一个求解器 SNOPT 给出的解进行比较是很有启发性的：

```python
ampl: option solver snopt;
ampl: solve;
SNOPT 6.1-1: Optimal solution found.
15 iterations, objective 196200
```

<table><tr><td colspan="5">appl: display Trans;</td></tr><tr><td>Trans</td><td>[*,*]</td><td>(tr)</td><td></td><td></td></tr><tr><td>:</td><td>CLEV</td><td>GARY</td><td>PITT</td><td>: =</td></tr><tr><td>DET</td><td>1200</td><td>0</td><td>0</td><td></td></tr><tr><td>FRA</td><td>0</td><td>0</td><td>900</td><td></td></tr><tr><td>FRE</td><td>0</td><td>1100</td><td>0</td><td></td></tr><tr><td>LAF</td><td>400</td><td>0</td><td>600</td><td></td></tr><tr><td>LAN</td><td>600</td><td>0</td><td>0</td><td></td></tr><tr><td>STL</td><td>0</td><td>300</td><td>1400</td><td></td></tr><tr><td>WIN</td><td>400</td><td>0</td><td>0</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td></tr></table>

最小成本仍然是 196200，但实现方式不同。运输问题通常具有这样的替代最优解 (alternative optimal solution)，特别是当目标函数中的系数为整数时。

不幸的是，没有简单的方法可以描述所有的最优解。不过，通过处理多个目标，你也许能够得到更好的最优解选择，我们将在第 8.3 节举例说明。

# 3.3 运输模型的其他解释

顾名思义，当某些材料从一组起始地 (origins) 运送到一组目的地 (destinations) 时，运输模型 (transportation model) 是适用的。在给定起始地的可用量和目的地的需求量的情况下，问题是在最小运输成本下满足这些需求。

更广义地来看，运输模型不必涉及“材料”的运输。只要在各起始地的可用量和目的地的需求量能以某种单位衡量，并且单位运输成本可以确定，运输模型就可以应用。例如，它们可以用于模拟向经销商运送汽车，或者将军事人员调动到新的岗位。

再进一步广义来看，运输模型甚至不需要涉及“运输”。起始地的量可能只是与各个目的地相关联，而目标函数衡量的是这种关联的某种价值，与实际运输无关。这种情况下，结果通常被称为“指派”模型 (assignment model)。

一个特别著名的例子是，某个部门需要将若干人员指派到数量相等的办公室。此时，起始地代表各个个人，目的地代表各个办公室。由于每个人被指派到一个办公室，且每个办公室由一个人占用，因此所有参数 supply[i] 和 demand[j] 的值均为 1。我们将 Trans[i,j] 解释为指派给办公室 j 的人员 i 的“数量”；也就是说，如果 Trans[i,j] 为 1，则人员 i 将占用办公室 j；如果 Trans[i,j] 为 0，则人员 i 不占用办公室 j。

目标函数又如何呢？一种可能是要求人们给办公室排序，给出他们的第一选择、第二选择，等等。然后我们令 `cost[i,j]` 表示人 `i` 给办公室 `j` 的排名。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/0cc481459161eb919a0c4bafbe96d5ea1ba57d154e17c64ea308a34edb0dafce.jpg)  
图 3-2：分配问题的数据 (assign.dat)。

这样约定后，目标函数中的每一项 `cost[i,j] * Trans[i,j]` 就表示：如果人 `i` 被分配到办公室 `j`（即 `Trans[i,j]` 等于 1），那么该项表示人 `i` 对办公室 `j` 的偏好；如果人 `i` 没有被分配到办公室 `j`（即 `Trans[i,j]` 等于 0），则该项为零。由于目标函数是所有这些项的总和，它必然等于所有非零项的和，也就是每个人对他们所分配到的办公室的排名之和。通过最小化这个总和，我们可以希望找到一个能让许多人满意的分配方案。

为了使用运输模型达到这个目的，我们只需提供适当的数据即可。图 3-2 是一个例子，将 11 个人分配到 11 个办公室。我们使用默认选项将所有的供应量和需求量设为 1，而无需手动输入所有的 1。如果我们将这个数据集存储在 `assign.dat` 中，就可以将其与我们已有的运输模型一起使用：

```python
mpl::model transp.mod;
mpl::data assign.dat;
mpl::solve;

CPLEX 8.0.0: optimal solution; objective 28
24 dual simplex iterations (0 in phase I)
```

通过将选项 `omit_zero_rows` 设置为 1，我们可以只打印目标函数中的非零项。（显示结果的选项将在第 12 章中介绍。）这个列表告诉我们每个人被分配的房间以及他们对该房间的偏好：

ampl: option omit_zero_rows 1;  
ampl: display {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];  
cost[i,j]*Trans[i,j] :=  
Councillard C118 6  
Daskin D241 4  
Hazen C246 1  
Hopp D237 1  
Irvani C138 2  
Linetsky C250 3  
Mehrotra D239 2  
Nelson C140 4  
Smilowitz M233 1  
Tamhane C251 3  
White M239 1 ;

这个解相当成功，尽管它分配了两个第四选择和一个第六选择。

不难看出，当所有的 `supply[i]` 和 `demand[j]` 值都为 1 时，任何满足所有约束条件的 `Trans[i,j]` 必然介于 0 和 1 之间。但我们怎么知道在最优解中，每个 `Trans[i,j]` 必然等于 0 或 1，而不是其他值呢？我们能够依赖运输模型的一个特殊性质：只要所有的供应量和需求量都是整数，并且所有变量的上下界也都是整数，就必然存在一个完全由整数组成的最优解。此外，我们使用的求解器（solver）总是能找到这样的整数解之一。但不要因为这个有利的结果而误以为在所有其他情况下都能保证整数解；即使在看起来与运输模型非常相似的例子中，寻找整数解也可能需要特殊的求解器，并且需要更多的工作。第 20 章将详细讨论整数性问题。

将 100 个人分配到 100 个房间的问题包含一万个变量；将 1000 个人分配到 1000 个房间则会产生一百万个变量。然而，在这种规模的应用中，大多数分配方案可以预先排除，因此实际的决策变量数量不会太大。在查看初始解之后，你可能希望进一步排除一些分配方案 —— 在我们的例子中，也许不应允许低于第五选择的分配 —— 或者你可能希望强制某些分配以特定方式完成，以便查看其余部分如何最优地完成。这些情况需要能够直接处理配对子集（如人员与办公室，或起始地与目的地）的模型。AMPL 中用于描述配对及其他“复合”对象的功能将在第 6 章中介绍。

# 练习

3-1. 这个运输模型用于寻找最低成本的运输计划，来源于 Dantzig 的《Linear Programming and Extensions》。一家公司在西雅图和圣地亚哥设有工厂，其每周产能分别为 350 和 600 箱。它在纽约、芝加哥和托皮卡有客户，每周订单量分别为 325、300 和 275 箱。涉及的距离如下：

<table><tr><td></td><td>纽约</td><td>芝加哥</td><td>托皮卡</td></tr><tr><td>西雅图</td><td>2500</td><td>1700</td><td>1800</td></tr><tr><td>圣地亚哥</td><td>2500</td><td>1800</td><td>1400</td></tr></table>

运输成本为每箱每千英里 90 美元。请在 AMPL 中建立该模型并求解，以确定最低成本及各运输量。

3-2. 一个小型制造操作使用三台机器生产六种零件。下个月需要一定数量的每种零件，且每台机器能容纳的零件数量有限；此外，在不同机器上生产相同零件的成本也不同。具体成本及相关数值如下：

<table><tr><td colspan="8">零件</td></tr><tr><td>机器</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td><td>产能</td></tr><tr><td>1</td><td>3</td><td>3</td><td>2</td><td>5</td><td>2</td><td>1</td><td>80</td></tr><tr><td>2</td><td>4</td><td>1</td><td>1</td><td>2</td><td>2</td><td>1</td><td>30</td></tr><tr><td>3</td><td>2</td><td>2</td><td>5</td><td>1</td><td>1</td><td>2</td><td>160</td></tr><tr><td>需求量</td><td>10</td><td>40</td><td>60</td><td>20</td><td>20</td><td>30</td><td></td></tr></table>

(a) 使用图 3-1a 中的模型，为此问题创建一个数据语句文件；将机器视为起始地，零件视为目的地。每台机器应生产多少零件，才能使总成本最小？

(b) 如果机器 2 的容量增加到 50，制造商可能能够略微降低生产总成本。分析这种情况需要对模型进行什么小的修改？总成本降低了多少，生产计划在哪些方面发生了变化？

(c) 现在假设容量是以小时为单位，而不是以零件数量为单位，并且在不同机器上制造相同零件所需的时间略有不同：

<table><tr><td colspan="8">零件</td></tr><tr><td>机器</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td><td>容量</td></tr><tr><td>1</td><td>1.3</td><td>1.3</td><td>1.2</td><td>1.5</td><td>1.2</td><td>1.1</td><td>50</td></tr><tr><td>2</td><td>1.4</td><td>1.1</td><td>1.1</td><td>1.2</td><td>1.2</td><td>1.1</td><td>90</td></tr><tr><td>3</td><td>1.2</td><td>1.2</td><td>1.5</td><td>1.1</td><td>1.1</td><td>1.2</td><td>175</td></tr></table>

修改供应约束，使其限制每个产地的总生产时间，而不是总生产数量。新的最优解有什么不同？在哪台机器上使用了所有可用时间？

(d) 重新求解前一个问题，但将目标函数改为最小化总机器小时数，而不是总成本。

3-3. 本练习涉及图 3-1 中运输模型和数据的推广。

(a) 添加两个参数 supply pct 和 demand pct，分别表示可发送到任何一家工厂的轧机 (mill) 供应量 (supply) 的最大比例，以及可由任何一家轧机 (mill) 满足的工厂需求量 (demand) 的最大比例。将这些参数纳入图 3-1a 的模型中。

求解以下情况：发送到任何一家工厂的轧机 (mill) 供应量 (supply) 不得超过 50%，由任何一家轧机 (mill) 满足的工厂需求量 (demand) 不得超过 85%。这种变化如何影响最小成本和最优运输量？

(b) 假设轧机 (mill) 不自己生产板坯，而是从另外两家工厂获得板坯，其中可提供的吨位如下：

MIDTWN 2700 HAMGTN 4200

从工厂到轧机 (mill) 运输一吨板坯的成本如下：

<table><tr><td></td><td>GARY</td><td>CLEV</td><td>PITT</td></tr><tr><td>MIDTWN</td><td>12</td><td>8</td><td>17</td></tr><tr><td>HAMGTN</td><td>10</td><td>5</td><td>13</td></tr></table>

所有其他数据值与之前相同，但 supply pct 被重新解释为可发送到任何一家轧机 (mill) 的工厂供应量 (supply) 的最大比例。

将这种情况表述为一个 AMPL 模型。你需要两个索引变量集合，一个用于从工厂到轧机 (mill) 的运输量，另一个用于从轧机 (mill) 到工厂的运输量。每个轧机 (mill) 的运输量必须等于供应量 (supply)，每个工厂的运输量必须等于需求量 (demand)，如前所述；此外，每个轧机 (mill) 的运出量必须等于运入量。

求解所得的线性规划。最低成本解中的运输量是多少？

(c) 除了运输成本的差异之外，工厂和 mills 的生产成本可能也不同。解释如何将生产成本纳入模型。

(d) 在钢坯轧制过程中，一部分钢材会作为废料损失掉。假设每个 mill 的损失比例可能不同，请修改模型以考虑废料损失。

(e) 实际上，废料并非真正损失，而是可以出售用于回收。进一步修改模型，以考虑每个 mill 产生的废料价值。

3-4. 本练习考虑第 3.3 节中介绍的指派问题的变体。

(a) 尝试重新排列数据中 DEST 成员的列表（图 3-2），然后重新求解。找到一种重新排列方式，使得你的求解器 (solver) 报告出不同的最优指派。

(b) 即使某一个人被分配到排名非常靠后的办公室，这种指派也可能无法接受，即使总排名之和是最优的。具体来说，我们的解使得某一个人得到了她的第六选择；为了排除这种情况，将成本数据中所有偏好排名为六或更大的数值改为 99，这样它们就会变得非常不具吸引力。( 在后续章节中你会学到更方便的功能来实现相同的目的，但这种粗略的方法目前是可行的。) 重新求解指派问题，并验证结果是一个同样好的指派，其中没有人得到比第五选择更差的办公室。

现在应用同样的方法尝试让每个人都得到不差于第四选择的办公室。你发现了什么？

(c) 现在假设办公室 C118、C250 和 C251 不再可用，你必须将两个人分别安排到 C138、C140 和 C246。在这些办公室的每个排名上加 20，以反映任何人更喜欢私人办公室而不是共享办公室的事实。处理这种情况还需要对模型和数据进行哪些其他修改？你得到了什么最优指派？

(d) 有些人可能具有资历，使他们在选择办公室时应得到更多的考虑。解释如何增强模型以使用每个人的资历等级数据。