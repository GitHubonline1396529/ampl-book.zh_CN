# 显示命令 (Display Commands)

AMPL 提供了丰富多样的命令和选项，以帮助您检查和报告优化结果。第 12.1 节介绍 display 命令，这是将集合成员和数值整理成列表和表格的最便捷命令；第 12.2 和 12.3 节详细说明了 display 的选项，这些选项可以让您更精确地控制列表和表格的排列方式以及数字的显示格式。第 12.4 节描述了 print 和 printf 命令，这两个相关命令在准备发送给其他程序的数据以及格式化简单报告时非常有用。

虽然我们的示例基于集合、参数和变量（以及涉及它们的表达式）的显示，但您可以使用相同的命令来检查对偶值（dual values）、松弛变量（slacks）、约简成本（reduced costs）以及其他与最优解（optimal solution）相关的量；相关规则在第 12.5 节中进行了说明。

AMPL 还提供了访问建模和求解信息的方法。第 12.6 节描述了一些功能，当您希望在命令行查看参数声明、显示问题实例中的特定约束、列出所有变量的值和边界（无论其名称如何），或者记录 AMPL 和求解器（solver）活动的耗时时，这些功能非常有用。

最后，第 12.7 节讨论了操作 AMPL 命令输出的一般功能。这些功能包括命令输出的重定向、输出的日志记录以及错误消息的抑制。

# 12.1 浏览结果：display 命令

检查数据和结果值的最简单方法是输入 `display` 以及您想要查看的内容描述。`display` 命令会自动将值格式化为直观且熟悉的排列方式；尽可能地，它使用与第 9 章中描述的数据语句相同的列表和表格格式。我们的示例使用了在其他章节中定义的模型中的参数和变量。

正如我们将在第 12.7 节中更详细描述的那样，可以通过在 `display` 命令末尾添加 `>filename` 来将 `display` 命令的输出捕获到文件中；这种重定向机制也适用于大多数其他产生输出的命令。

# 显示集合

通过输入 `display` 和集合名称列表来显示集合的内容。以下示例来自图 6-2a 的模型：

```rl
ampl: display ORIG, DEST, LINKS;
set ORIG := GARY CLEV PITT;
set DEST := FRA DET LAN WIN STL FRE LAF;
set LINKS := (GARY, DET) (GARY, LAF) (CLEV, LAN) (CLEV, LAF) (PITT, STL) (GARY, LAN) (CLEV, FRA) (CLEV, WIN) (PITT, FRA) (PITT, FRE) (GARY, STL) (CLEV, DET) (CLEV, STL) (PITT, WIN);
```

如果您指定了一个索引集合族（indexed collection of sets）的名称，则会显示该族中的每个集合（来自图 6-3）：

```rl
ampl: display PROD, AREA;
set PROD := bands coils;
set AREA[bands] := east north;
set AREA[coils] := east west export;
```

可以通过下标查看索引集合族中的特定成员，例如 `display AREA["bands"]`。

`display` 的参数不必是一个已声明的集合；它可以是第 5 章或第 6 章中描述的任何求值为集合的表达式。例如，你可以显示所有集合 `AREA[p]` 的并集：

```rl
ampl: display union {p in PROD} AREA[p];
set union {p in PROD} AREA[p] := east north west export;
```

或者显示所有运输成本大于 500 的运输链接集合：

```rl
ampl: display {(i,j) in LINKS: cost[i,j] * Trans[i,j] > 500};
set {(i,j) in LINKS: cost[i,j]*Trans[i,j] > 500} := 
(GARY,STL) (CLEV,DET) (CLEV,WIN) (PITT,FRA) (PITT,FRE) 
(GARY,LAF) (CLEV,LAN) (CLEV,LAF) (PITT,STL);
```

由于该集合的成员依赖于变量 `Trans[i,j]` 的当前值，因此你不能在模型中引用它，但在 `display` 命令中是合法的，因为在 `display` 命令中变量被视为参数。

# 显示参数和变量

`display` 命令可以显示一个标量模型组件的值：

```rl
ampl: display T;
T = 4
```

或者显示索引集合中各个组件的值（见图 1-6b）：

```rl
ampl: display avail["reheat"], avail["roll"];
avail['reheat'] = 35
avail['roll'] = 40
```

或者显示任意表达式的值：

```rl
ampl: display sin(1)^2 + cos(1)^2;
sin(1)^2 + cos(1)^2 = 1
```

然而，`display` 的主要用途是显示整个索引数据集合。对于“一维”数据——即在简单集合上索引的参数或变量——AMPL 使用列格式（见图 4-6b）：

```rl
ampl: display avail;
avail [*]:= 
reheat  35
roll    40
```

对于“二维”参数或变量——即在成对集合或两个简单集合上索引的参数或变量——对于少量数据，AMPL 会形成一个列表（见图 4-1）：

```rl
ampl: display supply;
supply := 
CLEV bands  700
CLEV coils  1600
CLEV plate  300
GARY bands  400
GARY coils  800
GARY plate  200
PITT bands  800
PITT coils  1800
PITT plate  300
```

而对于大量数据，则使用表格形式：

```rl
ampl: display demand;
demand [*,*] :    bands  coils  plate :=
DET              300    750    100
FRA              300    500    100
FRE              225    850    100
LAF              250    500    250
LAN              100    400      0
STL              650    950    200
WIN               75    250     50
;
```

你可以通过设置选项 `display_1col` 来控制格式之间的选择，该选项将在下一节中描述。

在有序对集合上索引的参数或变量（或其他任何模型实体）也被视为二维对象，并以类似方式显示。以下是针对本节早前显示的集合 `LINKS` 上索引的参数的显示（来自图 6-2a）：

```cpp
ampl: display cost;
cost :=
CLEV DET  9
CLEV FRA  27
CLEV LAF  17
CLEV LAN  12
CLEV STL  26
CLEV WIN  9
GARY DET  14
GARY LAF  8
GARY LAN  11
GARY STL  16
PITT FRA  24
PITT FRE  99
PITT STL  28
PITT WIN  13
;
```

同样，这也可以以表格形式显示，下一节将展示如何实现。

为了显示以三维或更高维度索引的值，AMPL 会再次为少量数据形成列表。然而，多维实体通常涉及大量数据，在这种情况下，AMPL 会将值“切片”成二维表格，如图 4-6 中的这个变量所示：

```rl
ampl: display Trans;
Trans [CLEV,*,*]:
: bands coils plate :=
DET     0   750     0
FRA     0     0     0
FRE     0     0     0
LAF     0   500     0
LAN     0   400     0
STL     0    50     0
WIN     0   250     0

[GARY,*,*]:
: bands coils plate :=
DET     0     0     0
FRA     0     0     0
FRE   225   850   100
LAF   250     0     0
LAN     0     0     0
STL   650   900   200
WIN     0     0     0

[PITT,*,*]:
: bands coils plate :=
DET   300     0   100
FRA   300   500   100
FRE     0     0     0
LAF     0     0   250
LAN   100     0     0
STL     0     0     0
WIN    75     0    50
```

在第一个表格的头部，模板 $[\mathrm{CLEV}, *, *]$ 表示该切片是通过第一个分量中的 CLEV 进行的，因此在行 LAF 和列 coils 中的条目表示 Trans["CLEV","LAF","coils"] 的值为 500。由于 Trans 的第一个索引在此情况下始终为 CLEV、GARY 或 PITT，因此总共有三个切片表格。但 AMPL 并不总是通过第一个分量进行切片；它会选择切片方式，以使显示中包含尽可能少的表格。

相同维度的两个或多个分量的显示始终以列表格式呈现，无论这些分量是一维的（如图 4-4）：

```rl
ampl: display inv0, producost, invcost;
:   inv0 producost invcost :=
bands   10       10     2.5
coils    0       11       3
```

还是二维的（如图 4-6）：

ampl: display rate, make_cost, Make;
:   rate make_cost   Make :=
CLEV bands  190      190      0
CLEV coils  130      170   1950
CLEV plate  160      185      0
GARY bands  200      180   1125
GARY coils  140      170   1750
GARY plate  160      180    300
PITT bands  230      190    775
PITT coils  160      180    500
PITT plate  170      185    500

或任何更高维度。索引以列表形式出现在左侧，最后一个索引变化最快。

从这些示例中可以看出，display 通常会按字母或数字顺序排列行和列标签，而不管它们在数据文件中可能以何种顺序给出。然而，当标签来自有序集合时，原始顺序会被保留（如图 5-3）：

ampl: display avail;
avail [*]:=
27sep   40
04oct   40
11oct   32
18oct   40

因此，即使模型中某些集合的顺序在其公式中没有显式作用，也值得将它们声明为有序集合。

# 显示索引表达式

`display` 命令可以显示在 AMPL 模型中有效的任何算术表达式的值。对于单值表达式没有困难，例如图 4-4 中的这三个利润组成部分：

```ampl
ampl: display sum {p in PROD, t in 1..T} revenue[p,t] * Sell[p,t],
       sum {p in PROD, t in 1..T} prodcost[p] * Make[p,t],
       sum {p in PROD, t in 1..T} invcost[p] * Inv[p,t];

sum{p in PROD, t in 1 .. T} revenue[p,t] * Sell[p,t] = 787810
sum{p in PROD, t in 1 .. T} prodcost[p] * Make[p,t] = 269477
sum{p in PROD, t in 1 .. T} invcost[p] * Inv[p,t] = 3300
```

然而，假设你想要查看所有 `revenue[p,t] * Sell[p,t]` 的个别值。由于你可以输入 `display revenue, Sell` 来显示 `revenue[p,t]` 和 `Sell[p,t]` 的个别值，你可能希望通过输入以下内容来请求这些值的乘积：

```ampl
ampl: display revenue * Sell;
syntax error
context:  display revenue >>> * << Sell;
```

AMPL 不识别这种数组算术。要显示表达式的索引集合，你必须显式指定索引：

```ampl
ampl: display {p in PROD, t in 1..T} revenue[p,t] * Sell[p,t];

revenue[p,t] * Sell[p,t] [* , *] (tr)
:     bands   coils    :=
1   150000    9210
2   156000   87500
3    37800  129500
4    54000  163800 ;
```

要对两个或多个表达式应用相同的索引，请在索引表达式后用括号括起它们的列表：

```ampl
ampl: display {p in PROD, t in 1..T}
       (revenue[p,t] * Sell[p,t], producost[p] * Make[p,t]);

: revenue[p,t]*Sell[p,t] producost[p]*Make[p,t] :=
bands 1   150000   59900
bands 2   156000   60000
bands 3    37800   14000
bands 4    54000   20000
coils 1     9210   15477
coils 2    87500   15400
coils 3   129500   38500
coils 4   163800   46200 ;
```

索引表达式后跟一个表达式或括号括起的表达式列表被视为单个显示项，它指定了某些索引值的集合。`display` 命令可以包含如上所示的这些项之一，或它们的逗号分隔列表。

索引表达式的值的呈现遵循与单个参数 (parameter) 和变量 (variable) 相同的规则。实际上，你可以将像 `display revenue, Sell` 这样的命令视为以下命令的简写：

```ampl
ampl: display {p in PROD, t in 1..T} (revenue[p,t], Sell[p,t]);

: revenue[p,t] Sell[p,t] :=
bands 1    25  6000
bands 2    26  6000
bands 3    27  1400
bands 4    27  2000
coils 1    30   307
coils 2    35  2500
coils 3    37  3500
coils 4    39  4200 ;
```

但是，如果你重新排列索引表达式，使 `t in 1..T` 在前，则列表的行首先按 `1..T` 的成员排序：

```ampl
ampl: display {t in 1..T, p in PROD} (revenue[p,t], Sell[p,t]);

: revenue[p,t] Sell[p,t] :=
1 bands    25  6000
1 coils    30   307
2 bands    26  6000
2 coils    35  2500
3 bands    27  1400
3 coils    37  3500
4 bands    27  2000
4 coils    39  4200 ;
```

这种默认呈现方式的改变只能通过在 `display` 后放置显式索引表达式来实现。

除了对单个显示项建立索引外，还可以指定一个集合，使整个显示命令基于该集合建立索引 —— 也就是说，可以要求对索引集合的每个成员执行一次该命令。这一功能在重新排列多维表格的切片时尤其有用。在本节前面显示三维变量 Trans（基于集合 $\{\text{ORIG}, \text{DEST}, \text{PROD}\}$ 建立索引）时，AMPL 选择通过 ORIG 的成员对值进行切片，从而生成一系列二维表格。

如果想要显示通过 PROD 的切片该怎么办？如前面示例中那样重新排列索引表达式，并不能可靠地达到预期效果；display 命令总是选择最小的索引集，而当存在多个最小索引集时，它并不一定会选择第一个。相反，可以明确指定对 PROD 中的每个 $\mathbb{P}$ 进行单独显示：

ampl: display {p in PROD}:  
ampl? {i in ORIG, j in DEST} Trans[i,j,p];  
Trans[i,j,'bands']  $[^{\star},^{\star}]$  (tr) :      CLEV    GARY    PITT  
$\begin{array}{rl}{\mathbf{\Psi}}&{:=}\end{array}$  
DET        0       0     300  
FRA        0       0     300  
FRE        0     225       0  
LAF        0     250       0  
LAN        0       0     100  
STL        0     650       0  
WIN        0       0      75  
:  
Trans[i,j,'coils']  $[^{\star},^{\star}]$  (tr) :      CLEV    GARY    PITT  
$\begin{array}{rl}{\mathbf{\Psi}}&{:=}\end{array}$  
DET      750       0       0  
FRA        0       0     500  
FRE        0     850       0  
LAF      500       0       0  
LAN      400       0       0  
STL       50     900       0  
WIN      250       0       0  
:  
Trans[i,j,'plate']  $[^{\star},^{\star}]$  (tr) :      CLEV    GARY    PITT  
$\begin{array}{rl}{\mathbf{\Psi}}&{:=}\end{array}$  
DET        0       0     100  
FRA        0       0     100  
FRE        0     100       0  
LAF        0       0     250  
LAN        0       0       0  
STL        0     200       0  
WIN        0       0      50  
```

如该示例所示，如果 display 命令在开头指定了索引表达式并紧跟一个冒号，则该索引集将应用于整个命令。对于该集合的每个成员，冒号后的表达式将被分别计算并显示。

## 12.2 显示的格式选项

display 命令采用一些简单的规则来为数据选择良好的排列方式。通过更改多个选项，可以控制整体排列、零值的处理方式以及行宽。这些选项总结在表 12-1 中，并在括号中给出了默认值。

## 列表与表格的排列方式

一维参数或变量的显示可能会产生非常长的列表，例如图 16-5 中调度模型的以下示例：

```
display_1col    表格以列表格式显示时的最大元素数量 (20)
display_transpose   若行数 - 列数 < display_transpose，则转置表格 (0)
display_width   最大行宽 (79)
gutter_width    表格列之间的间隔 (3)
omit_zero_cols  若非 0，则在显示中省略全零列 (0)
omit_zero_rows  若非 0，则在显示中省略全零行 (0)
```

表 12-1：显示的格式选项（含默认值）

```
ampl: display required;
required [*]:=
Fri1  100    Fri2   78    Fri3   52
Mon1  100    Mon2   78    Mon3   52
Sat1  100    Sat2   78
Thu1  100    Thu2   78    Thu3   52
Tue1  100    Tue2   78    Tue3   52
Wed1  100    Wed2   78    Wed3   52
```

可以使用选项 display_1col 来请求更紧凑的格式：

```
ampl: option display_1col 0;
ampl: display required;
required [*] :=
Fri1  100   Mon1  100   Sat1  100   Thu2   78   Tue2   78   Wed2   78
Fri2   78   Mon2   78   Sat2   78   Thu3   52   Tue3   52   Wed3   52
Fri3   52   Mon3   52   Thu1  100   Tue1  100   Wed1  100
```

当要显示的值的数量小于或等于 display_1col 时，将使用单列列表格式；否则将使用紧凑格式。display_1col 的默认值为 20；将其设置为 0 可强制使用紧凑格式，或将其设置为一个非常大的数字以强制使用列表格式。

多维显示会以类似的方式受到选项 display_1col 的影响。当值的数量小于或等于 display_1col 时，将使用单列列表格式；否则将使用适当的紧凑格式——在这种情况下是表格。我们在前一节中展示了这种差异的例子，其中 supply 的显示为列表形式，因为它只有 9 个值；而 demand 的显示为表格形式，因为其 21 个值超过了选项 display_1col 的默认设置 20。

由于索引为有序对集合的参数或变量也被视为二维的，因此 display_1col 的值也会影响其显示。以下是前一节中显示的参数 cost（索引为集合 LINKS，来自图 6-2a）的表格格式：

```
ampl: option display_1col 0;
ampl: display cost;
cost [*,*] (tr)
         :  CLEV  GARY  PITT :=
    DET  :   9     14    24
    FRA  :  27      .    24
    FRE  :  99
    LAF  :  17     8      .
    LAN  :  12    11      .
    STL  :  26    16    28
    WIN  :   9    13      .
```

点 (.) 表示索引集中不存在的组合。因此，在表格的 GARY 列中，FRA 行显示为点，因为对 (GARY, FRA) 不属于 LINKS；此问题中未定义 cost["GARY", "FRA"]。另一方面，LINKS 确实包含对 (GARY, LAF)，并且 cost["GARY", "LAF"] 在表中显示为 8。

在选择表格方向时，display 命令默认优先考虑行而非列；也就是说，如果列数将超过行数，则表格会被转置。因此，前一节中 demand 的表格按第一坐标标记行，按第二坐标标记列，因为它按 DEST（有 7 个成员）和 PROD（有 3 个成员）进行索引。相比之下，cost 的表格按第一坐标标记列，按第二坐标标记行，因为它按 ORIG（有 3 个成员）和 DEST（有 7 个成员）进行索引。转置的表格在其第一行中用 (tr) 表示。

可以通过更改 display_transpose 选项来反转表格的转置状态。正值倾向于强制转置：

ampl: option display_transpose 5; ampl: display demand;
demand $[*,*]$ (tr) :
DET FRA FRE LAF LAN STL WIN :=
bands 300 300 225 250 100 650 75
coils 750 500 850 500 400 950 250
plate 100 100 100 250 0 200 50 ;

而负值则倾向于抑制转置：

ampl: option display_transpose -5; ampl: display cost;
cost $[*,*]$ :
DET FRA FRE LAF LAN STL WIN :=
CLEV 9 27 17 12 26 9
GARY 14 8 11 16 . 
PITT 24 99 28 13 ;

规则如下：仅当行数减去列数小于 display_transpose 时，表格才会被转置。在默认值为零的情况下，display_transpose 提供了上述默认行为。

# 控制行宽

display_width 选项给出了由 display 生成的行上最大字符数（如图 16-4 模型所示）：

ampl: option display_width 50, display_1col 0; ampl: display required;
required $[*,*]$ =
Fri1 100 Mon3 52 Thu3 52 Wed2 78
Fri2 78 Sat1 100 Tue1 100 Wed3 52
Fri3 52 Sat2 78 Tue2 78 Mon1 100
Thu1 100 Tue3 52 Mon2 78 Thu2 78
Wed1 100 ;

当表格宽度超过 display_width 时，它会被垂直切割成两个或更多的表格。每个表格中的行名称相同，但列不同：

<table><tr><td colspan="9">mpl: option display_width 50; display cost;</td><td></td></tr><tr><td colspan="9">cost [*,*]</td><td></td></tr><tr><td>:</td><td>C118</td><td>C138</td><td>C140</td><td>C246</td><td>C250</td><td>C251</td><td>D237</td><td>D239</td><td>: =</td></tr><tr><td>Coullard</td><td>6</td><td>9</td><td>8</td><td>7</td><td>11</td><td>10</td><td>4</td><td>5</td><td></td></tr><tr><td>Daskin</td><td>11</td><td>8</td><td>7</td><td>6</td><td>9</td><td>10</td><td>1</td><td>5</td><td></td></tr><tr><td>Hazen</td><td>9</td><td>10</td><td>11</td><td>1</td><td>5</td><td>6</td><td>2</td><td>7</td><td></td></tr><tr><td>Hopp</td><td>11</td><td>9</td><td>8</td><td>10</td><td>6</td><td>5</td><td>1</td><td>7</td><td></td></tr><tr><td>Iravani</td><td>3</td><td>2</td><td>8</td><td>9</td><td>10</td><td>11</td><td>1</td><td>5</td><td></td></tr><tr><td>Linetsky</td><td>11</td><td>9</td><td>10</td><td>5</td><td>3</td><td>4</td><td>6</td><td>7</td><td></td></tr><tr><td>Mehrotra</td><td>6</td><td>11</td><td>10</td><td>9</td><td>8</td><td>7</td><td>1</td><td>2</td><td></td></tr><tr><td>Nelson</td><td>11</td><td>5</td><td>4</td><td>6</td><td>7</td><td>8</td><td>1</td><td>9</td><td></td></tr><tr><td>Smilowitz</td><td>11</td><td>9</td><td>10</td><td>8</td><td>6</td><td>5</td><td>7</td><td>3</td><td></td></tr><tr><td>Tamhane</td><td>5</td><td>6</td><td>9</td><td>8</td><td>4</td><td>3</td><td>7</td><td>10</td><td></td></tr><tr><td>White</td><td>11</td><td>9</td><td>8</td><td>4</td><td>6</td><td>5</td><td>3</td><td>10</td><td></td></tr><tr><td>:</td><td>D241</td><td>M233</td><td>M239</td><td>: =</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Coullard</td><td>3</td><td>2</td><td>1</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Daskin</td><td>4</td><td>2</td><td>3</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Hazen</td><td>8</td><td>3</td><td>4</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Hopp</td><td>4</td><td>2</td><td>3</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Iravani</td><td>4</td><td>6</td><td>7</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Linetsky</td><td>8</td><td>1</td><td>2</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Mehrotra</td><td>5</td><td>4</td><td>3</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Nelson</td><td>10</td><td>2</td><td>3</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Smilowitz</td><td>4</td><td>1</td><td>2</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>Tamhane</td><td>11</td><td>2</td><td>1</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>White</td><td>7</td><td>2</td><td>1</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>:</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></table>

如果表格的列标题比值宽得多，display 会引入缩写以将所有列保持在一起 (图 4-4)：

mpl: option display_width 40;  ampl: display {p in PROD, t in 1..T} (revenue[p,t]*Sell[p,t],  ampl? prodcost[p]*Make[p,t], invcost[p]*Inv[p,t]);  # \(1 = revenue[p,t]*Sell[p,t]\)  # \(2 = prodcost[p]*Make[p,t]\)  # \(3 = invcost[p]*Inv[p,t]\)  : \(\) 1 \(\) 2 \(\) 3 \(\)\equiv\(  bands 1 150000 59900 0  bands 2 156000 60000 0  bands 3 37800 14000 0  bands 4 54000 20000 0  coils 1 9210 15477 3300  coils 2 87500 15400 0  coils 3 129500 38500 0  coils 4 163800 46200 0  ;

另一方面，如果标题比值窄，你可以通过减少 option gutter_width — 列之间的空格数 — 从默认值 3 减少到 2 或 1，从而在一行中挤入更多内容。

# 零值的省略

在某些变量数量远超约束数量的线性规划中，大多数变量的最优值为零。例如在图 3-2 的指派问题中，所有变量的最优值形成如下表格，其中每行和每列都只有一个 1：

# ampl: display Trans;

<table><tr><td colspan="12">运输 [ * * ]</td><td></td></tr><tr><td></td><td>C118</td><td>C138</td><td>C140</td><td>C246</td><td>C250</td><td>C251</td><td>D237</td><td>D239</td><td>D241</td><td>M233</td><td>M239</td><td>: =</td></tr><tr><td>库拉德 (Coullard)</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>达斯金 (Daskin)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td></td></tr><tr><td>哈曾 (Hazen)</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>霍普 (Hopp)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>伊拉瓦尼 (Iravani)</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>利涅茨基 (Linetsky)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>梅赫罗特拉 (Mehrotra)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>纳尔逊 (Nelson)</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>斯米尔诺维茨 (Smilowitz)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td></td></tr><tr><td>坦哈内 (Tamhane)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td></tr><tr><td>怀特 (White)</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></table>

通过将 omit_zero_rows 设置为 1，所有零值被抑制，列表缩减为感兴趣的条目：

ampl: option omit_zero_rows 1;

ampl: display Trans;

Trans := Coullard C118 1 Daskin D241 1 Hazen C246 1 Hopp D237 1 Iravani C138 1 Linetsky C250 1 Mehrotra D239 1 Nelson C140 1 Smilowitz M233 1 Tamhane C251 1 White M239 1 ;

如果非零条目的数量小于 display_1col 的值，则数据以列表形式打印，如这里所示。如果非零数量大于 display_1col，则使用表格格式，且 omit_zero_rows 选项仅抑制包含全零条目的表格行。

例如，本章前面提到的三维变量 Trans 的显示将被压缩为如下形式：

ampl: display Trans; Trans [CLEV,\*,\*]: bands coils plate :  $=$  DET 0 750 0 LAF 0 500 0 LAN 0 400 0 STL 0 50 0 WIN 0 250 0

[GARY,\*,\*]: bands coils plate :  $=$  FRE 225 850 100 LAF 250 0 0 STL 650 900 200

[PITT, *, *]： bands coils plate ： $=$  DET 300 0 100 FRA 300 500 100 LAF 0 0 250 LAN 100 0 0 WIN 75 0 50

对应的选项 `omit_zero_cols` 在设置为 1 时会抑制所有全为零的列，例如会从 `Trans [CLEV, *, *]` 中消除两列。

## 12.3 用于显示的数值选项

由 `display` 命令生成的表格或列表中的数字，是计算机内部数值表示形式到数字和符号字符串的转换结果。AMPL 用于调整此转换的选项如表 12-2 所示。在本节中，我们首先考虑仅影响数字外观的选项，然后考虑同时影响基础解值的选项。

| 选项名 | 含义 | 默认值 |
|--------|------|--------|
| `display_eps` | 与零不同显示的最小量级 | 0 |
| `display_precision` | 显示数字四舍五入到的精度位数；若为 0 则为完整精度 | 6 |
| `display_round` | 显示数字四舍五入到小数点左侧或（如果为负）右侧的位数，优先于 `display_precision` | " " |
| `solution_precision` | 解值四舍五入到的精度位数；若为 0 则为完整精度 | 0 |
| `solution_round` | 解值四舍五入到小数点左侧或（如果为负）右侧的位数，优先于 `solution_precision` | " " |

表 12-2：用于显示的数值选项（默认值）。

## 数值的显示外观

在我们迄今为止的所有示例中，`display` 命令对每个数值都显示相同的有效数字位数：

```ampl
ampl: display {p in PROD, t in 1..T} Make[p,t]/rate[p];
Make[p,t]/rate[p] [*,*] :=
bands  coils
1   29.95   10.05
2   30      10
3   20      12
4   32.1429 7.85714
;

ampl: display {p in PROD, t in 1..T} prodcost[p]*Make[p,t];
prodcost[p]*Make[p,t] [*,*] :=
bands     coils
1   59900     15477
2   60000     15400
3   40000     18480
4   64285.7   12100
;
```

（参见图 6-3 和图 6-4）。默认情况下，无论结果是 7.85714 还是 64285.7，都使用六位有效数字。某些数字看起来位数较少，但只是因为末尾的零被省略了；例如 29.95 表示的是精确到六位数字的 29.9500，而 59900 表示 59900.0。

通过将 `display_precision` 选项更改为六以外的值，可以改变报告的有效数字位数：

```ampl
ampl: option display_precision 3;
ampl: display Make['bands',4] / rate['bands'], prodcost['bands'] * Make['bands',4];
Make['bands',4]/rate['bands'] = 32.1
prodcost['bands']*Make['bands',4] = 64300

ampl: option display_precision 9;
ampl: display Make['bands',4] / rate['bands'], prodcost['bands'] * Make['bands',4];
Make['bands',4]/rate['bands'] = 32.1428571
prodcost['bands']*Make['bands',4] = 64285.7143

ampl: option display_precision 0;
ampl: display Make['bands',4] / rate['bands'], prodcost['bands'] * Make['bands',4];
Make['bands',4]/rate['bands'] = 32.14285714285713
prodcost['bands']*Make['bands',4] = 64285.71428571427
```

在最后一个例子中，`display_precision` 的值为 0 时会被特殊处理；它告诉 `display` 尽可能精确地表示数字，使用所需的任意多位数字。（准确地说，所显示的数字是这样的最短十进制表示：当正确舍入到计算机的表示时，它会精确地给出计算机中存储的值。）

以给定精度显示数字会为每个数字提供相同程度的有用信息，但由于小数点后位数的不同，它们看起来可能参差不齐。要指定舍入到固定的小数位数（无论结果精度如何），你可以设置选项 `display_round`。非负值指定小数点后要显示的位数：

```ampl
ampl: option display_round 2;
ampl: display {p in PROD, t in 1..T} Make[p,t]/rate[p];
```

```
Make[p,t]/rate[p] [*.*] (tr)
: bands coils :=
1 29.95 10.05
2 30.00 10.00
3 20.00 12.00
4 32.14 7.86
;
```

负值表示在小数点前进行舍入。例如，当 `display_round` 为 `-2` 时，所有数字都会舍入到百位：

```ampl
ampl: option display_round -2;
ampl: display {p in PROD, t in 1..T} producost[p]*Make[p,t];
```

```
producost[p]*Make[p,t] [*.*] (tr)
: bands coils :=
1 59900 15500
2 60000 15400
3 40000 18500
4 64300 12100
;
```

`display_round` 的任何整数值都会覆盖 `display_precision` 的效果。要关闭 `display_round`，请将其设置为某个非整数值，例如空字符串 `''`。

根据你使用的求解器，你可能会发现一些本应为零的解值并不总是完全为零。例如，以下是某个求解器对第 3.3 节指派问题中目标函数项 `cost[i,j] * Trans[i,j]` 的报告：

```ampl
ampl: option omit_zero_rows 1;
ampl: display {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];
```

```
cost[i,j]*Trans[i,j] :=
Coullard C118 6
Coullard D241 2.05994e-17
Daskin D237 1
Hazen C246 1
Hopp D237 6.86647e-18
Hopp D241 4
... 9 行省略
White C246 2.74659e-17
White C251 -3.43323e-17
White M239 1
;
```

像 `6.86647e-18` 和 `-3.43323e-17` 这样的微小数值在此问题的上下文中没有意义；在精确解中它们应为零，但由于求解器算法与计算机数字表示方式的交互，导致它们略微偏离零。

为避免以无意义的精度查看这些数字，你可以为 `display_round` 选择一个合理的设置——在这种情况下为 `0`，因为小数点后没有感兴趣的数字：

```ampl
ampl: option display_round 0;
ampl: display {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];
```

```
cost[i,j]*Trans[i,j] :=
Coullard C118 6
Coullard D241 0
Daskin D237 1
Hazen C246 1
Hopp D237 0
Hopp D241 4
Iravani C118 0
Iravani C138 2
Linetsky C250 3
Mehrotra D239 2
Nelson C138 0
Nelson C140 4
Smilowitz M233 1
Tamhane C118 0
Tamhane C251 3
White C246 0
White C251 0
White M239 1
;
```

现在小数字仅表示为 `0` (如果为正) 或 `$-0$` (如果为负)。但如果您希望完全抑制它们的显示，则必须设置一个单独的选项 `display_eps`：

```ampl
option display_eps 1e-10;
display {i in ORIG, j in DEST} cost[i,j] * Trans[i,j];
```

```
cost[i,j]*Trans[i,j] :=
Coullard C118   6
Daskin  D237   1
Hazen   C246   1
Hopp    D241   4
Iravani C138   2
Linetsky        C250   3
Mehrotra        D239   2
Nelson  C140   4
Smilowitz       M233   1
Tamhane C251   3
White   M239   1
;
```

在 `display` 的所有输出中，任何绝对值小于 `display_eps` 值的数值都会被视为精确的零。

# 解值的舍入

选项 `display_precision`、`display_round` 和 `display_eps` 仅影响数字的显示方式，而不影响其实际值。您可以通过尝试显示在上表中具有正值的所有 `i` 属于 `ORIG` 和 `j` 属于 `DEST` 的对，比较 `cost[i,j]*Trans[i,j]` 与 0，来观察这一点：

```ampl
display {i in ORIG, j in DEST: cost[i,j]*Trans[i,j] > 0};
```

```ampl
set {i in ORIG, j in DEST: cost[i,j]*Trans[i,j] > 0} :=
(Coullard, C118) (Iravani, C118) (Smilowitz, M233)
(Coullard, D241) (Iravani, C138) (Tamhane, C251)
(Daskin, D237) (Linetsky, C250) (White, C246)
(Hazen, C246) (Mehrotra, D239) (White, M239)
(Hopp, D237) (Nelson, C138) (Hopp, D241) (Nelson, C140);
```

尽管像 `2.05994e-17` 这样的值在显示时被视为零，但在测试中它大于零。您可以通过将上面的 `>0` 改为例如 `>0.1` 来解决此问题。或者，您可以设置选项 `solution_round`，使 AMPL 在从求解器接收解值时将它们舍入到合理的精度：

```ampl
option solution_round 10;
solve;
```

MINDS 5.5: optimal solution found.  
40 iterations, objective 28

```ampl
display {i in ORIG, j in DEST: cost[i,j]*Trans[i,j] > 0};
```

```ampl
set {i in ORIG, j in DEST: cost[i,j]*Trans[i,j] > 0} :=
(Coulard, C118) (Iravani, C138) (Smilowitz, M233)
(Daskin, D237) (Linetsky, C250) (Tamhane, C251)
(Hazen, C246) (Mehrotra, D239) (White, M239)
(Hopp, D241) (Nelson, C140);
```

选项 `solution_precision` 和 `solution_round` 的工作方式与 `display_precision` 和 `display_round` 相同，不同之处在于它们仅在从求解器返回时应用于解值，并且它们会永久更改返回的值，而不仅仅是它们的显示方式。

即使舍入值不接近零，也可能产生影响。例如，我们首先使用几个显示选项来获取图 16-4 调度模型分数解的紧凑列表：

```ampl
model sched.mod;
data sched.dat;
solve;
```

MINDS 5.5: optimal solution found.  
19 iterations, objective 265.6

```ampl
option display_width 60;
option display_1col 5;
option display_eps 1e-10;
option omit_zero_rows 1;
display Work;
```

Work[*] :=  
10 28.8 30 14.4 71 35.6 106 23.2 123 35.6  
18 7.6 35 6.8 73 28 109 14.4  
24 6.8 66 35.6 87 14.4 113 14.4  
;

每个值 `Work[j]` 表示分配给计划 `j` 的工人数量。我们可以通过将小数值向上舍入到下一个最高整数来快速得到一个实际的计划；使用 `ceil` 函数进行舍入，我们看到所需的工人总数应该是：

```ampl
display sum {j in SCHEDS} ceil(Work[j]);
```

sum{j in SCHEDS} ceil(Work[j]) = 273

然而，如果我们从前面的表格中复制这些数字并手动向上舍入，我们会发现它们的总和只有 271。问题的根源可以通过以完整精度显示这些数字来看到：

ampl: option display_eps 0;
ampl: option display_precision 0;
ampl: display Work;
Work[*] :=
 10 28.799999999999997 73 28.000000000000018
 18 7.599999999999998 87 14.399999999999995
 24 6.79999999999999 95 -5.876671973951407e-15
 30 14.40000000000001 106 23.200000000000006
 35 6.799999999999995 108 4.685288280240683e-16
 55 -4.939614313857677e-15 109 14.4
 66 35.6 113 14.4
 71 35.599999999999994 123 35.5999999999999999
;

问题的一半是由于 Work[108] 的微小正值，它被向上舍入到 1。另一半是由于 Work[73]；虽然在精确解中它是 28，但它从求解器返回时略大一些，值为 28.000000000000018，所以它被向上舍入到 29。

在这种情况下确保我们的算术正确工作的最简单方法是在求解前再次设置 solution_round：

ampl: option solution_round 10;
ampl: solve;
MINDS 5.5: optimal solution found.
19 iterations, objective 265.6
ampl: display sum {j in SCHEDS} ceil(Work[j]);
sum{j in SCHEDS} ceil(Work[j]) = 271

我们选择 10 作为 solution_round 的值，因为我们观察到求解器值中的微小不准确性出现在小数点后第 10 位之后。

solution_round 或 solution_precision 的效果适用于求解器返回的所有值。要仅修改某些值，请使用第 11.3 节中描述的赋值 (let) 命令以及表 11-1 中的舍入函数。

# 12.4 其他输出命令：print 和 printf

两个额外的 AMPL 命令具有与 display 命令几乎相同的语法，但不会自动格式化其输出。print 命令完全不进行格式化，而 printf 命令需要明确描述所需的格式。

# print 命令

print 命令产生一行输出：

ampl: print sum {p in PROD, t in 1..T} revenue[p,t]*Sell[p,t], " ", sum {p in PROD, t in 1..T} producost[p]*Make[p,t], " ", sum {p in PROD, t in 1..T} invcost[p]*Imr[p,t];
787810 269477 3300
ampl: print {t in 1..T, p in PROD} Make[p,t];
5990 1407 6000 1400 1400 3500 2000 4200

或者，如果后面跟有索引表达式和冒号，则为索引集的每个成员产生一行输出：

ampl: print {t in 1..T}: {p in PROD} Make[p,t];
5990 1407 6000 1400 1400 3500 2000 4200

通常，打印 (print) 的条目之间会以空格分隔，但可以通过选项 print_separator 来更改这一设置。例如，你可能希望将 print_separator 设置为制表符 (tab)，以便将数据导入电子表格；为此，请输入 option print_separator $\mathrm{''}\rightarrow \mathrm{''}$，其中 $\rightarrow$ 表示按下制表键后产生的结果。

关键字 print（可选的索引表达式和冒号）后面跟着一个打印项，或由逗号分隔的打印项列表。一个打印项可以是一个值，或一个索引表达式后跟一个值或括号括起来的值列表。因此，打印项与 display 项非常相似，只是只允许出现单个值；虽然你可以写 display rate，但你必须显式指定 print {p in PROD} rate[p]。此外，集合不能作为 print 的参数，但其成员可以：

ampl: print PROD;  
语法错误  
上下文: print >>> PROD; >>>  
ampl: print {p in PROD} (p, rate[p]);  
bands 200  
coils 140

与 display 不同的是，print 允许在索引项中嵌套索引：

ampl: print {p in PROD} (p, rate[p], {t in 1..T} Make[p,t]);  
bands 200 5990 6000 1400 2000  
coils 140 1407 1400 3500 4200

print 输出中数字的表示方式由 print_precision 和 print_round 选项控制，它们的工作方式与 display 命令中的 display_precision 和 display_round 选项完全相同。初始时 print_precision 为 0，print_round 为空字符串，因此默认情况下 print 会使用足够多的数字来尽可能精确地表示每个值。在上述示例中，print_round 被设置为 0，以便将数字四舍五入为整数。

在交互式操作时，你可能会发现 print 在屏幕上以比 display 更紧凑的格式查看少量值时非常有用。当输出重定向到文件时，print 对于以方便电子表格和其他数据分析工具使用的格式写入未格式化的结果非常有用。与 display 一样，只需在 print 命令末尾添加 >filename 即可。

# print 命令

printf 的语法与 print 完全相同，只是第一个打印项是一个字符字符串，用于为其余项提供格式化指令：

ampl: printf "Total revenue is $6.2f.\backslash n", $  
ampl? sum {p in PROD, t in 1..T} revenue[p,t] * Sell[p,t];  
Total revenue is $ 787810.00

格式字符串包含两种类型的对象：普通字符（直接复制到输出中）和转换说明符（控制后续打印项的显示方式）。每个转换说明符以字符 \% 开头，以转换字符结尾。例如，\%6.2f 表示转换为十进制表示，至少占 6 个字符宽度，小数点后保留两位数字。完整的规则与 C 语言中的 printf 函数基本相同；详见附录中的 A.16 节摘要。

printf 的输出不会自动换行。必须在格式字符串中显式使用 \backslash \backslash \mathfrak{n}（表示“换行”字符）来指示换行。要生成一系列行，请使用 printf 的索引版本：

ampl: printf {t in 1. .T} "3i%12.2f%12.2f\n", t,  
      ampl? sum {p in PROD} revenue[p,t]*Sell[p,t],  
      ampl? sum {p in PROD} prdcost[p]*Make[p,t];  
1 159210.00 75377.00  
2 243500.00 75400.00  
3 167300.00 52500.00  
4 217800.00 66200.00  

此 printf 对于冒号前索引集合中的每个成员都会执行一次；对于每个 t in 1. .T，再次应用该格式，并且 \n 字符会生成另一个换行。

printf 命令主要用于将输出重定向到文件时，以可读格式打印简短的摘要报告。由于格式字符串中的转换说明符数量必须与要打印的值的数量匹配，因此 printf 无法方便地生成每行项目数可能在每次运行中变化的表格，例如所有 Make[p,t] 值的表格。

# 12.5 相关解值

在解释线性规划 (linear program) 的解时，集合、参数和变量是最明显的查看对象，但 AMPL 还提供了检查与最优解相关的目标函数、边界、松弛变量、对偶价格和约简成本的方法。

正如我们已经在许多示例中展示的那样，AMPL 通过使用“限定”名称来区分与模型组件相关的各种值，这些名称由变量或约束标识符、点 (.) 和预定义的“后缀”字符串组成。例如，变量 Make 的上界称为 Make.ub，而 Make["coils", 2] 的上界写为 Make["coils", 2].ub。（注意后缀位于下标之后。）限定名称可以像非限定名称一样使用，因此 display Make.ub 会打印 Make 变量上界表，而 display Make, Make.ub 则会打印最优值和上界的列表。

# 目标函数

目标函数的名称（来自 minimize 或 maximize 声明）指的是从变量的当前值计算出的目标值。该名称可用于在 display、print 或 printf 中表示最优目标值：

ampl: print 100 * Total_Profit /  
      ampl? sum {p in PROD, t in 1. .T} revenue[p,t] * Sell[p,t];  
65.37528084182735  

如果当前模型声明了多个目标函数，则可以引用其中任何一个，即使只优化了一个目标函数。

# 边界和松弛变量

变量上的后缀 .lb 和 .ub 分别表示其下界 (lower bound) 和上界 (upper bound)，而 .slack 表示变量值与其较近边界之间的差值。以下是来自图 5-1 的示例：

<table><tr><td colspan="5">amp1: display Buy.lb, Buy, Buy.ub, Buy.slack;</td><td></td></tr><tr><td>:</td><td>Buy.lb</td><td>Buy</td><td>Buy.ub</td><td>Buy.slack</td><td>: =</td></tr><tr><td>BEEF</td><td>2</td><td>2</td><td>10</td><td>0</td><td></td></tr><tr><td>CHK</td><td>2</td><td>10</td><td>10</td><td>0</td><td></td></tr><tr><td>FISH</td><td>2</td><td>2</td><td>10</td><td>0</td><td></td></tr><tr><td>HAM</td><td>2</td><td>2</td><td>10</td><td>0</td><td></td></tr><tr><td>MCH</td><td>2</td><td>2</td><td>10</td><td>0</td><td></td></tr><tr><td>MTL</td><td>2</td><td>6.23596</td><td>10</td><td>3.76404</td><td></td></tr><tr><td>SPG</td><td>2</td><td>5.25843</td><td>10</td><td>3.25843</td><td></td></tr><tr><td>TUR</td><td>2</td><td>2</td><td>10</td><td>0</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td><td></td></tr></table>

报告的边界是发送给 求解器 (solver) 的边界。因此，它们不仅包括在 var 声明的 $\geq$ 和 $\leq$ 短语中指定的边界，还包括由 AMPL 的 presolve 阶段从约束中推导出的某些边界。其他后缀允许您查看原始边界以及由 presolve 推导出的附加边界；有关详细信息，请参见第 14.1 节中关于 presolve 的讨论。

两个相等的边界表示一个固定的变量，该变量通常由 presolve 消除。因此，在图 4-4 的规划模型中，约束 $\mathtt{Inv}[\mathtt{p},0] = \mathtt{inv0}[\mathtt{p}]$ 固定了初始库存：

ampl: display {p in PROD} (Inv[p,0].lb, inv0[p], Inv[p,0].ub);  
: Inv[p,0].lb inv0[p] Inv[p,0].ub :=  
bands 10 10 10  
coils 0 0 0  

在图 4-6 的生产与运输模型中，约束 sum {i in ORIG} Trans[i,j,p] = demand[j,p] 导致 presolve 将三个变量固定为零，因为 demand["LAN","plate"] 为零：

ampl: display {i in ORIG} ampl? (Trans[i,"LAN","plate"].lb, Trans[i,"LAN","plate"].ub);  
: Trans[i,'LAN','plate'].lb Trans[i,'LAN','plate'].ub :=  
CLEV 0 0  
GARY 0 0  
PITT 0 0  

如此例所示，presolve 对边界的调整可能取决于数据以及约束的结构。

边界和松弛的概念对模型的约束也有类似的解释。任何 AMPL 约束都可以写成标准形式：

下界 ≤ 主体 ≤ 上界

其中主体是包含变量的所有项的总和，而下界和上界仅依赖于数据。后缀 .lb、.body 和 .ub 给出了约束的这三个部分的当前值。例如，在图 5-1 的饮食模型中，我们有声明：  
subject to Diet_Min {i in MINREQ}: sum {j in FOOD} amt[i,j] * Buy[j] ≥ n_Min[i];  
subject to Diet_Max {i in MAXREQ}: sum {j in FOOD} amt[i,j] * Buy[j] ≤ n_Max[i];

以及以下约束边界：

ampl: display Diet_Min.lb, Diet_Min.body, Diet_Min.ub;  
:Diet_Min.lb Diet_Min.body Diet_Min.ub :=  
A 700 1013.98 Infinity  
B1 0 605 Infinity  
B2 0 492.416 Infinity  
C 700 700 Infinity  
CAL 16000 16000 Infinity  

ampl: display Diet_Max.lb, Diet_Max.body, Diet_Max.ub;  
:Diet_Max.lb Diet_Max.body Diet_Max.ub :=  
A -Infinity 1013.98 20000  
CAL -Infinity 16000 24000  
NA -Infinity 43855.9 50000  

自然地，≤ 约束没有下界，≥ 约束没有上界；AMPL 使用 -Infinity 和 Infinity 来表示这些情况。如果一个约束由两个 ≤ 或 ≥ 运算符指定，则其上下界都可以是有限的；参见第 8.4 节。对于等式约束，这两个边界是相同的。

后缀 .slack 表示的是约束体（body）与最近边界之间的差值：

ampl: display Diet_Min.slack;  
:Diet_Min.slack [*]:=  
A 313.978  
B1 605  
B2 492.416  
C 0  
CAL 0  

对于仅包含一个 ≤ 或 ≥ 运算符的约束，即使不等号两边都含有变量，松弛值（slack）始终等于该运算符左右两侧表达式的差值。松弛值为零的约束是在最优解中真正起作用的约束。

# 对偶变量值与简化成本

在线性规划中，每一个约束都对应一个量，它有不同的名称：对偶变量（dual variable）、边际值（marginal value）或影子价格（shadow price）。在 AMPL 命令环境中，这些对偶变量值以约束名称表示，无需添加任何限定性后缀。例如，在图 4-6 中有一组名为 Demand 的约束：

subject to Demand {j in DEST, p in PROD}: sum {i in ORIG} Trans[i,j,p] = demand[j,p];

可以通过以下命令查看与这些约束相关联的对偶变量值表：

<table><tr><td colspan="4">ampl: display Demand;</td><td></td></tr><tr><td colspan="4">Demand [*,*]</td><td></td></tr><tr><td>:</td><td>bands</td><td>coils</td><td>plate</td><td>: =</td></tr><tr><td>DET</td><td>201</td><td>190.714</td><td>199</td><td></td></tr><tr><td>FRA</td><td>209</td><td>204</td><td>211</td><td></td></tr><tr><td>FRE</td><td>266.2</td><td>273.714</td><td>285</td><td></td></tr><tr><td>LAF</td><td>201.2</td><td>198.714</td><td>205</td><td></td></tr><tr><td>LAN</td><td>202</td><td>193.714</td><td>0</td><td></td></tr><tr><td>STL</td><td>206.2</td><td>207.714</td><td>216</td><td></td></tr><tr><td>WIN</td><td>200</td><td>190.714</td><td>198</td><td></td></tr></table>

求解器会将最优对偶变量 (dual variables) 值连同“原始”变量 (primal variables) 的最优值一起返回给 AMPL。在此我们只能简要总结最常见的对偶变量解释方式；有关对偶理论及其应用的详细内容可以在任何一本线性规划 (linear program) 教科书中找到。

我们先通过一个例子来说明，考虑上面的需求约束 Demand["DET","bands"]。如果我们改变该约束中参数 demand["DET","bands"] 的值，则目标函数 Total_Cost 的最优值也会相应地发生变化。如果我们绘制 Total_Cost 的最优值与 demand["DET","bands"] 所有可能取值之间的关系图，结果将是一条成本“曲线”，它显示了底特律 (DET) 对带钢 (bands) 的总需求如何影响整体成本。

要确定整条成本曲线需要额外的计算，但你可以通过最优对偶值 (dual values) 来了解一些相关信息。当你使用某个特定的 demand["DET","bands"] 值求解线性规划 (linear program) 后，该约束的对偶价格 (dual price) 会告诉你在当前需求值下成本曲线的斜率。在我们的例子中，从上表中可以看出，曲线在当前需求处的斜率为 201。这意味着，对于 DET 每增加一吨带钢 (bands) 的需求，总生产与运输成本将以每吨 201 美元的速率增加；反之，每减少一吨需求，总成本将减少 201 美元。

再举一个不等式约束的例子，考虑来自同一模型的以下约束：

subject to Time {i in ORIG}: sum {p in PROD} (1/rate[i,p]) * Make[i,p] <= avail[i];

<ph-9092a>  
图 12-1：目标函数的分段线性图。

在这种情况下，观察对偶值 (dual values) 和松弛变量 (slacks) 会很有启发性：

mpl: display Time, Time.slack;  
: Time Time.slack  
= CLEY -1522.86 0  
GARY -3040 0  
PITT 0 10.5643

当松弛变量 (slacks) 为正值时，对偶值 (dual values) 为零。实际上，正的松弛意味着最优解并未用尽 PITT 的所有可用时间 (avail)；因此，稍微改变 avail["PITT"] 的值不会影响最优解。另一方面，当松弛变量为零时，对偶值可能具有显著意义。以 GARY 为例，其对偶值为 -3040，这意味着在 GARY 每增加一小时可用时间 (avail)，总成本将以每小时 3040 美元的速率减少；反之，每减少一小时可用时间，总成本将增加 3040 美元。

通常，如果我们绘制最优目标函数值相对于某个约束常数项的关系曲线，对于最小化问题，该曲线将是下凸的分段线性曲线（见图 12-1）；对于最大化问题，则是上凸的分段线性曲线（即上下颠倒的形状）。

从之前介绍的标准形式下界 $\leq$ 体部 $\leq$ 上界来看，最优对偶值可以这样理解：如果某个约束的松弛变量为正，则对偶值为零；如果松弛变量为零，则约束的体部必须等于其中一个（或两个）边界值，此时对偶值与相等的边界相关。具体来说，对偶值是目标函数值相对于该边界值的曲线斜率，是在当前边界值处的斜率；等价地，它也是最优目标函数值相对于该边界值的变化率。

对变量的约束界限 (bounds) 也可以进行几乎相同的分析。此时对偶值 (dual value) 的作用由变量所谓的“约简成本” (reduced cost) 扮演，该值可以通过 AMPL 命令环境中的后缀 `.rc` 来查看。例如，以下是图 5-1 中变量的界限和约简成本：

<table><tr><td>ampl:</td><td>display</td><td>Buy.lb,</td><td>Buy,</td><td>Buy.ub,</td><td>Buy.rc;</td></tr><tr><td>:</td><td>Buy.lb</td><td>Buy</td><td>Buy.ub</td><td>Buy.rc</td><td>: =</td></tr><tr><td>BEEF</td><td>2</td><td>2</td><td>10</td><td>1.73663</td><td></td></tr><tr><td>CHK</td><td>2</td><td>10</td><td>10</td><td>-0.853371</td><td></td></tr><tr><td>FSH</td><td>2</td><td>2</td><td>10</td><td>0.255281</td><td></td></tr><tr><td>HAM</td><td>2</td><td>2</td><td>10</td><td>0.698764</td><td></td></tr><tr><td>MCH</td><td>2</td><td>2</td><td>10</td><td>0.246573</td><td></td></tr><tr><td>MTL</td><td>2</td><td>6.23596</td><td>10</td><td>0</td><td></td></tr><tr><td>SPG</td><td>2</td><td>5.25843</td><td>10</td><td>0</td><td></td></tr><tr><td>TUR</td><td>2</td><td>2</td><td>10</td><td>0.343483</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td><td></td></tr></table>

由于 `Buy["MTL"]` 在其上下界之间存在松弛，因此其约简成本为零。`Buy["HAM"]` 处于其下界，因此其约简成本表明，当该下界每增加一个单位时，总成本大约增加 70 美分；或者当该下界每减少一个单位时，总成本大约减少 70 美分。另一方面，`Buy["CHK"]` 处于其上界，其负的约简成本表明，当该上界每增加一个单位时，总成本大约减少 85 美分；或者当该上界每减少一个单位时，总成本大约增加 85 美分。

如果当前某个界限值恰好位于相关曲线的一个转折点上 (即图 12-1 中斜率突然变化的位置之一)，那么当该界限增加时目标函数值将以一种速率变化，而当该界限减少时将以另一种速率变化。在极端情况下，这两种变化率中的任意一种都可能是无限的，这表明如果该界限进一步增加或减少，线性规划问题将变得不可行。然而，求解器只向 AMPL 报告一个最优对偶价格或约简成本，这个值可能是任一方向的变化率，或者是介于两者之间的某个值。

无论如何，对偶价格或约简成本只能给出目标值分段线性曲线上的一个斜率。因此，这些量只能作为目标对某些变量或约束边界敏感性的初步指导。如果这种敏感性对你的应用非常重要，你可以进行一系列运行，设置不同的边界值；参见第 11.3 节了解快速更改数据小部分的方法。（确实存在一些算法，可以在一个或多个边界发生线性变化时，找到分段线性曲线的部分或全部，但当前版本的 AMPL 并不直接支持这些算法。）

## 12.6 模型和实例的其他显示功能

在本节中，我们收集了各种用于显示模型信息或从模型生成的问题实例信息的实用命令和其他功能。

有两个命令允许你从命令行查看 AMPL 模型：`show` 列出模型组件的名称并显示各个组件的定义，而 `xref` 列出依赖于给定组件的所有组件。`expand` 命令显示 AMPL 从模型和数据生成的选定目标和约束，或变量的类似信息。AMPL 的变量、约束或目标的“通用”名称允许列出或测试适用于所有变量、约束或目标的内容。

### 显示模型组件：show 命令

单独使用时，`show` 命令会列出当前模型所有组件的名称：

```
mpl: model multmin3.mod;
appl: show;
parameters: demand fcost limit maxserve minload supply vcost
sets: DEST ORIG PROD
variables: Trans Use
constraints: Demand Max_Serve Min_Ship Multi Supply
objective: Total_Cost
checks: one, called check 1.
```

此显示可以限制为一种或多种类型的组件：

```
mpl: show vars;
variables: Trans Use
appl: show obj, constr;
objective: Total_Cost
constraints: Demand Max_Serve Min_Ship Multi Supply
```

`show` 命令还可以显示各个组件的声明，从而免去你在模型文件中查找的麻烦：

```
mpl: show Total_Cost;
minimize Total_Cost:
  sum{i in ORIG, j in DEST, p in PROD} vcost[i,j,p]*Trans[i,j,p] +
  sum{i in ORIG, j in DEST} fcost[i,j]*Use[i,j];

appl: show vcost, fcost, Trans;
param vcost{ORIG, DEST, PROD} >= 0;
param fcost{ORIG, DEST} >= 0;
var Trans{ORIG, DEST, PROD} >= 0;
```

如果 `show` 后面的项目是当前模型中组件的名称，则显示该组件的声明。否则，根据其首字母或前两个字母将该项目解释为组件类型；参见 A.19.1 节。（由于 AMPL 在解析和翻译模型时执行的转换，显示的声明可能与模型文件中的形式在非本质方面有所不同。）

由于模型中的 `check` 语句没有名称，AMPL 会按照它们出现的顺序进行编号。因此，若要查看第三个 `check` 语句，你可以输入：

```ampl
ampl: show check 3;
check{p in PROD} :
  sum{i in ORIG} supply[i,p] == sum{j in DEST} demand[j,p];
```

单独使用 `show checks` 会显示模型中 `check` 语句的数量。

# 显示模型依赖关系：`xref` 命令

`xref` 命令列出所有直接（通过引用）或间接（通过引用其依赖项）依赖于指定组件的模型组件。如果指定了多个组件，则每个组件的依赖项会分别列出。以下是一个来自 `multmip3.mod` 的示例：

```ampl
ampl: xref demand, Trans;
# 2 entities depend on demand: Check 1 Demand
# 5 entities depend on Trans: Total_Cost Supply Demand Multi Min_Ship
```

一般来说，该命令就是关键字 `xref` 后跟一组用逗号分隔的集合、参数、变量、目标函数和约束名称的任意组合。

# 显示模型实例：`expand` 命令

在检查模型及其数据的正确性时，你可能希望查看 AMPL 生成的一些具体约束。`expand` 命令可以显示给定索引集合中的所有约束，或者你指定的特定约束：

```ampl
ampl: model nltrans.mod;
ampl: data nltrans.dat;
ampl: expand Supply;
subject to Supply['GARY']:
  Trans['GARY','FRA'] $^+$ Trans['GARY','DET'] $^+$ Trans['GARY','LAN'] $^+$
  Trans['GARY','WIN'] $^+$ Trans['GARY','STL'] $^+$ Trans['GARY','FRE'] $^+$
  Trans['GARY','LAF'] $=$ 1400;
subject to Supply['CLEV']:
  Trans['CLEV','FRA'] $^+$ Trans['CLEV','DET'] $^+$ Trans['CLEV','LAN'] $^+$
  Trans['CLEV','WIN'] $^+$ Trans['CLEV','STL'] $^+$ Trans['CLEV','FRE'] $^+$
  Trans['CLEV','LAF'] $=$ 2600;
subject to Supply['PITT']:
  Trans['PITT','FRA'] $^+$ Trans['PITT','DET'] $^+$ Trans['PITT','LAN'] $^+$
  Trans['PITT','WIN'] $^+$ Trans['PITT','STL'] $^+$ Trans['PITT','FRE'] $^+$
  Trans['PITT','LAF'] $=$ 2900;
```

（参见图 18-4 和 18-5。）展开约束中的项的顺序不一定与约束声明中符号项的顺序一致。目标函数也可以用同样的方式展开：

```ampl
ampl: expand Total Cost;
minimize Total Cost:
  39*Trans['GARY','FRA']*(1 - Trans['GARY','FRA']/500) +
  14*Trans['GARY','DET']*(1 - Trans['GARY','DET']/1000) +
  11*Trans['GARY','LAN']*(1 - Trans['GARY','LAN']/1000) +
  14*Trans['GARY','WIN']*(1 - Trans['GARY','WIN']/1000) +
  16*...
  15 lines omitted
  Trans['PITT','FRE']*(1 - Trans['PITT','FRE']/500) +
  20*Trans['PITT','LAF']*(1 - Trans['PITT','LAF']/900)
```

当对变量应用 `expand` 命令时，它会列出该变量在线性目标函数和约束中的所有非零系数。

mpl: expand Trans; 系数 Trans[GARY,FRA]: Supply[GARY] 1 Demand[FRA] 1 TotalCost 0+nonlinear 系数 Trans[GARY,DET]: Supply[GARY] 1 Demand[DET] 1 TotalCost 0+nonlinear 系数 Trans[GARY,LAN]: Supply[GARY] 1 Demand[LAN] 1 TotalCost 0+nonlinear 系数 Trans[GARY,WIN]: Supply[GARY] 1 Demand[WIN] 1 TotalCost 0+nonlinear 17 项省略

当一个变量也出现在目标函数或约束中的非线性表达式中时，会附加项 \(^+\) nonlinear 来表示这些表达式。

单独的 expand 命令会产生模型中所有变量、目标函数和约束的展开。由于单个 expand 命令可能产生非常长的列表，因此您可能希望将其输出重定向到文件中，方法是在末尾加上 >filename，如下面第 12.7 节所述。

展开输出中的数字格式由选项 expand_precision 和 expand_round 控制，这两个选项的作用类似于第 12.3 节中描述的 display 命令的 display_precision 和 display_round。

expand 的输出反映了问题的“建模者视角”；它基于最初读取和翻译的模型和数据。但 AMPL 的 presolve 阶段（第 14.1 节）在将问题发送给求解器之前可能会对问题进行显著简化。要查看 presolve 后问题的“求解器视角”的展开，请使用关键字 solexpand 替代 expand。

# 变量、约束和目标函数的通用同义词

有时需要制作适用于所有变量、约束或目标函数的列表或测试。为此，AMPL 提供了自动更新的参数，这些参数保存当前生成问题实例中的变量、约束和目标函数的数量：

_nvars 当前问题中的变量数量 _ncons 当前问题中的约束数量 _nobjs 当前问题中的目标函数数量

相应索引的参数包含所有组件的 AMPL 名称：

_varname{1.._nvars} 当前问题中的变量名称 _conname{1.._ncons} 当前问题中的约束名称 _objname{1.._nobjs} 当前问题中的目标函数名称

最后，以下组件的同义词也可用：

_var{1.._nvars} 当前问题中的变量同义词 _con{1.._ncons} 当前问题中的约束同义词 _obj{1.._nobjs} 当前问题中的目标函数同义词

这些同义词允许您通过编号而不是通常的索引名称来引用组件。以变量为例，_var[5] 指问题中的第五个变量，_var[5].ub 指其上界，_var[5].rc 指其约简成本，等等，而 _varname[5] 是一个字符串，给出变量的真实 AMPL 名称。表 A-13 列出了所有用于集合、变量等的通用同义词。

通用名称 (generic name) 在需要对所有变量进行属性制表时非常有用，其中这些变量是在多个不同的 var 声明中定义的：

```ampl
ampl: model net3.mod
ampl: data net3.dat
ampl: solve;
MINDS 5.5: optimal solution found.
3 iterations, objective 1819
ampl: display {j in 1.._nvars} ampl?(_varname[j], _var[j], _var[j].ub, _var[j].rc);

_varname[j]                _var[j] _var[j].ub _var[j].rc :=
"PD_Ship['NE']"              250       250       -0.5
"PD_Ship['SE']"              200       250 -1.11022e-16
"DW_Ship['NE','BOS']"         90        90          0
"DW_Ship['NE','EWR']"        100       100       -1.1
"DW_Ship['NE','BWI']"         60       100          0
"DW_Ship['SE','EWR']"         20       100  2.22045e-16
"DW_Ship['SE','BWI']"         60       100  2.22045e-16
"DW_Ship['SE','ATL']"         70        70          0
"DW_Ship['SE','MCO']"         50        50          0
```

另一个用途是列出具有某种属性的所有变量，例如在最优解中远离上界的变量：

```ampl
ampl: display {j in 1.._nvars: ampl? _var[j] < _var[j].ub - 0.00001} _varname[j];

_varname[j]
[*] :=
 2   "FD_Ship['SE']"
 5   "DW_Ship['NE','BWI']"
 6   "DW_Ship['SE','EWR']"
 7   "DW_Ship['SE','BWI']"
 1
```

同样的注释也适用于约束和目标函数。使用 printf (12.4, A.16) 而不是 display 可以获得更精确的信息格式化。

与 expand 命令的情况一样，这些参数和通用同义词反映了建模者对问题的看法；它们的值是从最初读取和转换的模型和数据中确定的。AMPL 的 presolve 阶段可能会在问题发送到求解器之前对其进行显著简化。要使用反映 presolve 后求解器对问题看法的参数和通用同义词，请在上述名称中将 _ 替换为 _s；例如对于变量，使用 _snvars、_svarname 和 _svar。

其他预定义集合和参数表示模型组件的名称和维度 (元数)。它们在 A.19.4 中进行了总结。

# 资源列表

将选项 show_stats 从默认值 0 更改为非零值，可以请求 AMPL 生成的优化问题规模的摘要统计信息：

```ampl
ampl: model steelT.mod;
ampl: data steelT.dat;
ampl: option show_stats 1;
ampl: solve;
Presolve eliminates 2 constraints and 2 variables.
Adjusted problem:
24 variables, all linear
12 constraints, all linear; 38 nonzeros
1 linear objective; 24 nonzeros.
MINDD 5.5: optimal solution found.
15 iterations, objective 515033
```

附加行报告了整数变量和非线性组件的数量（如适用）。

将选项 times 从默认值 0 更改为非零值，可以请求 AMPL 转换器的时间和内存需求摘要。同样，通过将选项 gentimes 更改为非零值，可以获得 AMPL 的 genmod 阶段在生成模型实例时消耗资源的详细摘要。

当 AMPL 似乎挂起或耗时远超预期时，`gentimes` 所生成的显示信息可帮助将问题与特定模型组件关联起来。通常，某个参数、变量或约束被索引到了一个远大于预期的集合上，从而导致需要过多的时间和内存。

这些命令所给出的计时仅适用于 AMPL 翻译器，而不包括 求解器 (solver)。一系列预定义参数 (见表 A-14) 可用于处理 AMPL 和 求解器 的时间。例如，`_solve_time` 始终等于最近一次 `solve` 命令所需的总 CPU 秒数，而 `_ampl_time` 则等于 AMPL 所用的总 CPU 秒数，不包括 求解器 和其他外部程序所花费的时间。

许多 求解器 也提供用于请求求解时间分解的指令。然而，具体细节差异较大，因此关于如何请求和解读这些计时的信息将在 AMPL 与各 求解器 接口的文档中提供，而非本书中。

# 12.7 操作输出的通用功能

此处描述如何将 AMPL 的部分或全部输出定向到文件，以及如何控制警告和错误信息的数量。

# 输出重定向

本书中的示例均展示了交互式会话中命令的输出，其中输入命令和打印响应交替出现。但你也可以通过添加 `$>$` 和文件名将所有此类输出重定向到一个文件中：

```
ampl: display ORIG, DEST, PROD >multi.out;
ampl: display supply >multi.out;
```

第一个指定 `>multi.out` 的命令将创建一个同名的新文件 (或覆盖任何已存在的同名文件)。随后的命令会将内容追加到该文件末尾，直到会话结束或执行匹配的 `close` 命令：

```
ampl: close multi.out;
```

若要打开一个文件并将输出追加到已有内容之后 (而非覆盖)，请使用 `$>>$` 代替 `$>$`。一旦文件被打开，后续使用 `$>$` 和 `$>>$` 的效果相同。

# 输出日志

`log_file` 选项指示 AMPL 将后续命令和响应保存到一个文件中。该选项的值是一个被解释为文件名的字符串：

```
ampl: option log_file 'multi.tmp';
```

日志文件会收集所有 AMPL 语句及其产生的输出，但以下所述的少数情况除外。将 `log_file` 设置为空字符串：

```
ampl: option log_file '';
```

将停止向文件写入；空字符串是该选项的默认值。

当 AMPL 通过 `model` 或 `data` 命令 (或第 13 章定义的 `include` 命令) 从输入文件读取时，默认情况下不会将这些文件中的语句复制到日志文件中。若希望 AMPL 回显输入文件的内容，请将选项 `log_model` (用于模型模式下的输入) 或 `log_data` (用于数据模式下的输入) 从默认值 0 更改为非零值。

当你调用一个 求解器 (solver) 时，AMPL 至少会记录几行摘要信息，包括目标函数值、解的状态以及所需的工作量。通过 求解器 特定的指令，通常可以请求更多的 求解器 输出，例如迭代过程或分支定界节点的日志。许多 求解器 会自动将其所有输出发送到 AMPL 的日志文件中，但这种兼容性并非普遍适用。如果某个 求解器 的输出未出现在你的日志文件中，则应查阅该 求解器 的 AMPL 接口的补充文档；可能该 求解器 接受非标准指令以将输出重定向到文件。

# 消息限制

通过指定选项 `eexit` $n$，其中 $n$ 是某个整数，你可以确定 AMPL 如何处理错误消息。如果 $n$ 不为零，则在产生 $\mathrm{abs}(n)$ 条错误消息后，任何 AMPL 语句都会被终止；负值仅导致当前语句被终止，而正值则会导致整个 AMPL 会话终止。此选项的效果最常出现在使用模型和数据语句时出现问题的情况下，例如使用了错误的文件：

```
ampl: option eexit -3;
ampl: model diet.mod;
ampl: data diet.mod;
diet.mod, line 4 (offset 32):
     expected ; ( [ : or symbol
     context:  param cost >>> { <<< FOOD} > 0;
diet.mod, line 5 (offset 56):
     expected ; ( [ : or symbol
     context:  param f_min >>> { <<< FOOD} >= 0;
diet.mod, line 6 (offset 81):
     expected ; ( [ : or symbol
     context:  param f_max >>> { <<< j in FOOD} >= f_min[j];
Bailing out after 3 warnings.
```

`eexit` 的默认值是 `-10`。将其设置为 `0` 会导致显示所有错误消息。

`eexit` 设置也适用于在你输入 `solve` 后由 AMPL 预处理 (presolve) 阶段产生的不可行性警告。这些警告的数量同时受到选项 `presolve_warnings` 值的限制，该值通常被设置为较小的值；默认值为 `5`。

一个 AMPL 数据语句可能指定了与非法索引组合相对应的值，这可能是由于多种错误引起的，例如模型中的索引集不正确、索引顺序错误、(tr) 的误用以及打字错误等。类似的错误也可能由改变索引集成员关系的 `let` 语句引起。AMPL 在输入 `solve` 后会捕获这些错误。显示的无效组合数量受选项 `bad_subscripts` 值的限制，默认值为 `3`。
