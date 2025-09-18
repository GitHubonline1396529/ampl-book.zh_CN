# 数据库访问

AMPL 中索引数据的结构与数据库应用中广泛使用的关联表结构有许多共同之处。AMPL 的 table 声明允许你利用这种相似性，显式定义 AMPL 中的集合、参数、变量和表达式与由其他软件维护的关系数据库表之间的连接。随后，read table 和 write table 命令可以使用这些连接将数据值导入 AMPL 或将数据和解值从 AMPL 导出。

由 AMPL 读取和写入的关系表存放在文件中，这些文件的名称和位置由你在表声明中指定。为了处理这些文件，AMPL 依赖于表处理器 (table handler)，这些是按需加载的附加组件。求解器或数据库软件的供应商可能会提供处理器。AMPL 内置了两种简单的关系表格式的处理器，适用于实验用途，同时 AMPL 网站提供了一个可与广泛使用的 ODBC 接口配合工作的处理器。

本章首先展示如何将 AMPL 实体与关系表的列对应起来，以及如何通过 AMPL 的表声明来描述和实现这些对应关系。接下来的章节将介绍读取和写入外部关系表的基本功能、处理读写同一张表时可能出现的复杂情况的附加规则、以及写入一系列表或列和读取电子表格数据的机制。最后一节简要介绍一些标准和内置的处理器。

## 10.1 数据对应的一般原则

考虑第 2 章中 diet.mod 的以下声明，定义了集合 FOOD 以及在其上索引的三个参数：

```ampl
set FOOD;
param cost {FOOD} > 0;
param f_min {FOOD} >= 0;
param f_max {j in FOOD} >= f_min[j];
```

给出这些组件值的关系表具有四列：

<table>
<tr><td>FOOD</td><td>cost</td><td>f_min</td><td>f_max</td></tr>
<tr><td>BEEF</td><td>3.19</td><td>2</td><td>10</td></tr>
<tr><td>CHK</td><td>2.59</td><td>2</td><td>10</td></tr>
<tr><td>FISH</td><td>2.29</td><td>2</td><td>10</td></tr>
<tr><td>HAM</td><td>2.89</td><td>2</td><td>10</td></tr>
<tr><td>MCH</td><td>1.89</td><td>2</td><td>10</td></tr>
<tr><td>MTL</td><td>1.99</td><td>2</td><td>10</td></tr>
<tr><td>SPG</td><td>1.99</td><td>2</td><td>10</td></tr>
<tr><td>TUR</td><td>2.49</td><td>2</td><td>10</td></tr>
</table>

以 FOOD 为标题的列列出了同样名为 FOOD 的 AMPL 集合的成员。这是表的关键列 (key column)；关键列中的条目必须是唯一的，就像集合的成员一样，以便每个关键值准确标识一行。以 cost 为标题的列给出了在集合 FOOD 上索引的同名参数的值；这里 cost["BEEF"] 的值指定为 3.19，cost["CHK"] 指定为 2.59，以此类推。其余两列给出了在 FOOD 上索引的另外两个参数的值。

该表有八行数据，每一行对应一个集合成员。因此，每一行都包含与一个成员——在此例中为一种食物——相关的所有表数据。

在数据库软件的语境中，表的行通常被视为数据记录，列则被视为每个记录中的字段。因此，数据录入表单的每个列都有一个对应的录入字段。饮食问题示例 (来自 Microsoft Access) 的一个表单可能如图 10-1 所示。通过表单底部的控件，可以逐条录入或查看数据记录，每条记录对应表中的一行。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/c964deef6dfbf353eeeebcee74d72dc25bb987597bf2fd5ad7f5c06a781988be.jpg)  
图 10-1：Access 数据录入表单。

在此示例中，参数并不是唯一在 FOOD 集合上建立索引的实体。还有如下变量：

```ampl
var Buy {j in FOOD} >= f_min[j], <= f_max[j];
```

以及一些可以显示的结果表达式：

```ampl
ampl: model diet.mod;
ampl: data diet2a.dat;
ampl: solve;
MINDS 5.5: optimal solution found.
13 iterations, objective 118.0594032

ampl: display Buy, Buy.rc, {j in FOOD} Buy[j]/f_max[j];
:                  Buy    Buy.rc    Buy[j]/f_max[j]    :=
BEEF    5.36061    8.88178e-16    0.536061
CHK     2          1.18884        0.2
FISH    2          1.14441        0.2
HAM     10        -0.302651       1
MCH     10        -0.551151       1
MTL     10        -1.3289         1
SPG     9.30605    0              0.930605
TUR     2          2.73162        0.2
;
```

所有这些都可以包含在以 FOOD 为索引的值的关系表中：

<table>
<tr><td>FOOD</td><td>cost</td><td>f_min</td><td>f_max</td><td>Buy</td><td>BuyRC</td><td>BuyFrac</td></tr>
<tr><td>BEEF</td><td>3.19</td><td>2</td><td>10</td><td>5.36061</td><td>8.88178e-16</td><td>0.536061</td></tr>
<tr><td>CHK</td><td>2.59</td><td>2</td><td>10</td><td>2</td><td>1.18884</td><td>0.2</td></tr>
<tr><td>FISH</td><td>2.29</td><td>2</td><td>10</td><td>2</td><td>1.14441</td><td>0.2</td></tr>
<tr><td>HAM</td><td>2.89</td><td>2</td><td>10</td><td>10</td><td>-0.302651</td><td>1</td></tr>
<tr><td>MCH</td><td>1.89</td><td>2</td><td>10</td><td>10</td><td>-0.551151</td><td>1</td></tr>
<tr><td>MTL</td><td>1.99</td><td>2</td><td>10</td><td>10</td><td>-1.3289</td><td>1</td></tr>
<tr><td>SPG</td><td>1.99</td><td>2</td><td>10</td><td>9.30605</td><td>0</td><td>0.930605</td></tr>
<tr><td>TUR</td><td>2.49</td><td>2</td><td>10</td><td>2</td><td>2.73162</td><td>0.2</td></tr>
</table>

其中前四列通常从数据库中读入 AMPL，后三列则是从 AMPL 写回数据库的结果。我们自定义了列标题 BuyRC 和 BuyFrac，因为这些列中的 AMPL 表达式通常在数据库管理系统中不是有效的列标题。表声明提供了输入/输出和命名方面的区分，后续章节将会展示这些内容。

diet.mod 的其他实体是在营养素集合 NUTR 上建立索引的：参数 n_min 和 n_max，与 Diet 约束相关的对偶价格和其他值，以及涉及这些值的表达式。然而，由于营养素与食物完全不同，因此以营养素为索引的值会存放在与食物不同的关系表中。该表可能如下所示：

<table><tr><td>营养</td><td>n_min</td><td>n_max</td><td>NutrDual</td></tr><tr><td>A</td><td>700</td><td>20000</td><td>0</td></tr><tr><td>B1</td><td>700</td><td>20000</td><td>0</td></tr><tr><td>B2</td><td>700</td><td>20000</td><td>0.404585</td></tr><tr><td>C</td><td>700</td><td>20000</td><td>0</td></tr><tr><td>钙</td><td>16000</td><td>24000</td><td>0</td></tr><tr><td>钠</td><td>0</td><td>50000</td><td>-0.00306905</td></tr></table>

如本例所示，任何具有多个索引集 (indexing set) 的模型都需要多个关系表 (relational table) 来保存其数据和结果。由多个表组成的数据库是关系型数据管理的标准特征，存在于除最简单的“平面文件 (flat file)”数据库包之外的所有数据库中。

在相同高维集合上建立索引的实体与关系表具有类似的对应关系，但每个维度都有一个关键列 (key column)。以第 4 章的 steelT.mod 模型为例，以下参数和变量均在相同二维集合 (product-time pairs) 上建立索引：

```ampl
set PROD; # 产品
param T > 0; # 周数
param market {PROD, 1..T} >= 0;
param revenue {PROD, 1..T} >= 0;
var Make {PROD, 1..T} >= 0;
var Sell {p in PROD, t in 1..T} >= 0, <= market[p,t];
```

因此，一个相应的关系表具有两个关键列，一列包含 PROD 的成员，另一列包含 1..T 的成员，然后每一参数和变量都有一个值列。如下例所示，对应于 steelT.dat 中的数据：

<table><tr><td>产品</td><td>时间</td><td>市场</td><td>收入</td><td>生产</td><td>销售</td></tr><tr><td>带钢</td><td>1</td><td>6000</td><td>25</td><td>5990</td><td>6000</td></tr><tr><td>带钢</td><td>2</td><td>6000</td><td>26</td><td>6000</td><td>6000</td></tr><tr><td>带钢</td><td>3</td><td>4000</td><td>27</td><td>1400</td><td>1400</td></tr><tr><td>带钢</td><td>4</td><td>6500</td><td>27</td><td>2000</td><td>2000</td></tr><tr><td>线圈</td><td>1</td><td>4000</td><td>30</td><td>1407</td><td>307</td></tr><tr><td>线圈</td><td>2</td><td>2500</td><td>35</td><td>1400</td><td>2500</td></tr><tr><td>线圈</td><td>3</td><td>3500</td><td>37</td><td>3500</td><td>3500</td></tr><tr><td>线圈</td><td>4</td><td>4200</td><td>39</td><td>4200</td><td>4200</td></tr></table>

在这个表中，两个关键列中的每个有序项目对都是唯一的，就像这些对在集合 $\{\mathrm{PROD},1\ldots \mathrm{T}\}$ 中是唯一的一样。表中的 market 列意味着，例如，market["bands",1] 是 6000，market["coils",3] 是 3500。从第一行我们还可以看到，revenue["bands",1] 是 25，Make["bands",1] 是 5990，Sell["bands",1] 是 6000。同样地，AMPL 模型中的各种名称被用作列标题，除了 TIME 必须被创造出来以代表表达式 1..T。与前面的示例一样，列标题可以是数据库软件接受的任何标识符，而表声明将负责与 AMPL 名称的对应关系（如下所述）。

具有足够相似索引的 AMPL 实体通常可以放入同一个关系表中。例如，我们可以通过添加一列 varInv 值来扩展 steelT.mod 表：

$$
\mathtt{varInv}\{\mathtt{PROD},0..T\} \geq 0;
$$

然后表将具有以下布局：

<table><tr><td>PROD</td><td>TIME</td><td>market</td><td>revenue</td><td>Make</td><td>Sell</td><td>Inv</td></tr><tr><td>bands</td><td>0</td><td>.</td><td>.</td><td>.</td><td>.</td><td>10</td></tr><tr><td>bands</td><td>1</td><td>6000</td><td>25</td><td>5990</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>2</td><td>6000</td><td>26</td><td>6000</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>3</td><td>4000</td><td>27</td><td>1400</td><td>1400</td><td>0</td></tr><tr><td>bands</td><td>4</td><td>6500</td><td>27</td><td>2000</td><td>2000</td><td>0</td></tr><tr><td>coils</td><td>0</td><td>.</td><td>.</td><td>.</td><td>.</td><td>0</td></tr><tr><td>coils</td><td>1</td><td>4000</td><td>30</td><td>1407</td><td>307</td><td>1100</td></tr><tr><td>coils</td><td>2</td><td>2500</td><td>35</td><td>1400</td><td>2500</td><td>0</td></tr><tr><td>coils</td><td>3</td><td>3500</td><td>37</td><td>3500</td><td>3500</td><td>0</td></tr><tr><td>coils</td><td>4</td><td>4200</td><td>39</td><td>4200</td><td>4200</td><td>0</td></tr></table>

我们在这里使用 "." 来标记那些对应于模型和数据中未定义值的表条目。例如，在该模型的数据中不存在 market["bands", 0]，尽管在结果中确实存在 Env["bands", 0] 的值。数据库包在处理这种 "缺失" 条目时各不相同。

参数和变量也可以在一个作为数据读取而非从一维集合构造的配对集合上进行索引。例如，在第 3 章的 transp3.mod 示例中，我们有：

set LINKS within {ORIG, DEST}; param cost {LINKS} >= 0; # 每单位运输成本 var Trans {LINKS} >= 0; # 实际运输单位数

相应的关系表具有两个关键列，对应于索引集 LINKS 的两个组成部分，再加上 LINKS 上索引的参数和变量各一列：

<table><tr><td>起始地</td><td>目的地</td><td>成本</td><td>运输</td></tr><tr><td>加里</td><td>底特律</td><td>14</td><td>0</td></tr><tr><td>加里</td><td>拉斐特</td><td>8</td><td>600</td></tr><tr><td>加里</td><td>兰辛</td><td>11</td><td>0</td></tr><tr><td>加里</td><td>圣路易斯</td><td>16</td><td>800</td></tr><tr><td>克利夫兰</td><td>底特律</td><td>9</td><td>1200</td></tr><tr><td>克利夫兰</td><td>弗兰克福</td><td>27</td><td>0</td></tr><tr><td>克利夫兰</td><td>拉斐特</td><td>17</td><td>400</td></tr><tr><td>克利夫兰</td><td>兰辛</td><td>12</td><td>600</td></tr><tr><td>克利夫兰</td><td>圣路易斯</td><td>26</td><td>0</td></tr><tr><td>克利夫兰</td><td>温莎</td><td>9</td><td>400</td></tr><tr><td>匹兹堡</td><td>弗兰克福</td><td>24</td><td>900</td></tr><tr><td>匹兹堡</td><td>弗雷斯诺</td><td>99</td><td>1100</td></tr><tr><td>匹兹堡</td><td>圣路易斯</td><td>28</td><td>900</td></tr><tr><td>匹兹堡</td><td>温莎</td><td>13</td><td>0</td></tr></table>

这里的结构与前面的例子相同。表中仅对实际存在于链接集合 (LINKS) 中的每个起始地-目的地对有一行，而不是对所有可能的起始地-目的地对都有一行。

# 10.2 表处理语句示例

为了在 AMPL 模型与关系表之间传输信息，我们首先需要一个表声明 (table declaration) 来建立它们之间的对应关系。该声明的某些细节取决于用于创建和维护表的软件。对于上面定义的四列表饮食数据 (diet data)，一些可能的声明如下：

对于数据库文件 diet.mdb 中的 Microsoft Access 表：  
`table Foods IN "ODBC" "diet.mdb": FOOD <- [FOOD], cost, f_min, f_max;`  

对于工作簿文件 diet.xls 中的 Microsoft Excel 范围：  
`table Foods IN "ODBC" "diet.xls": FOOD <- [FOOD], cost, f_min, f_max;`  

对于文件 Foods.tab 中的 ASCII 文本表：  
`table Foods IN: FOOD <- [FOOD], cost, f_min, f_max;`  

每个表声明由两部分组成。冒号之前，声明提供一般信息。首先是表名——在上面的例子中为 Foods——这是在 AMPL 内部引用该表时所使用的名字。关键字 IN 表示所有非键表列的默认状态是只读；AMPL 将从这些列中读取值，但不会向它们写入数据。

用于在外部数据库文件中定位表的详细信息由诸如 "ODBC" 和 "diet.mdb" 之类的字符串提供，并在需要时使用 AMPL 表名 (Foods) 作为默认值：

对于 Microsoft Access，将使用 AMPL 的 ODBC 处理程序从数据库文件 diet.mdb 中读取表。数据库文件中的表名默认为 Foods。  
对于 Microsoft Excel，将使用 AMPL 的 ODBC 处理程序从电子表格文件 diet.xls 中读取表。包含该表的电子表格范围默认为 Foods。  
如果没有提供详细信息，则默认使用 AMPL 内置的文本表处理程序从 ASCII 文本文件 Foods.tab 中读取表。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/470bcbcf94b2fb9254905f7149cef069d244a6d0fb12e5965148a2688bf205d4.jpg)  
图 10-2：Access 关系表。

通常，表声明中字符串的格式取决于所使用的表处理程序。在我们的示例中使用的处理程序所需的字符串在第 10.7 节中进行了简要描述，并在特定表处理程序的在线文档中进行了详细说明。

在冒号之后，表声明给出了 AMPL 实体与关系表列之间对应关系的详细信息。四个用逗号分隔的条目对应于表中的四列，其中由方括号 [...] 包围的键列是第一列。在此示例中，表列的名称 (FOOD, cost, f_min, f_max) 与相应的 AMPL 组件名称相同。表达式 FOOD <- [FOOD] 表示键列 FOOD 中的条目将被复制到 AMPL 中，以定义集合 FOOD 的成员。

表声明仅定义了一种对应关系。要将关系表列中的值读入 AMPL 集合和参数，必须给出显式的 read table 命令。

因此，如果数据值位于类似图 10-2 的 Access 关系表中，则可以将 Access 的表声明与 read table 命令一起使用，以将 FOOD 的成员以及 cost、f_min 和 f_max 的值读入相应的 AMPL 集合和参数中：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/a0303d29f348a562acb8c49d6d4502759e07f3ac8656660bc5e39514aa2dbdfb.jpg)  
图 10-3：Excel 工作表范围。

<table><tr><td>ampl:</td><td>model</td><td>diet.mod;</td><td></td></tr><tr><td>ampl:</td><td>table</td><td>Foods IN &quot;ODBC&quot; &quot;diet.mdb&quot;:</td><td></td></tr><tr><td>ampl?</td><td>FOOD &lt;- [FOOD], cost, f_min, f_max;</td><td></td><td></td></tr><tr><td>ampl:</td><td>read table Foods;</td><td></td><td></td></tr><tr><td>ampl:</td><td>display cost, f_min, f_max;</td><td></td><td></td></tr><tr><td>:</td><td>cost f_min f_max</td><td>:=</td><td></td></tr><tr><td>BEEF</td><td>3.19</td><td>2</td><td>10</td></tr><tr><td>CHK</td><td>2.59</td><td>2</td><td>10</td></tr><tr><td>FISH</td><td>2.29</td><td>2</td><td>10</td></tr><tr><td>HAM</td><td>2.89</td><td>2</td><td>10</td></tr><tr><td>MCH</td><td>1.89</td><td>2</td><td>10</td></tr><tr><td>MTL</td><td>1.99</td><td>2</td><td>10</td></tr><tr><td>SPG</td><td>1.99</td><td>2</td><td>10</td></tr><tr><td>TUR</td><td>2.49</td><td>2</td><td>10</td></tr></table>

（display 命令确认数据库值已按预期读取。）如果数据值位于类似图 10-3 的 Excel 工作表范围内，则将以相同方式读取这些值，但使用 Excel 的表声明：

ampl: model diet.mod;  ampl: table Foods IN "ODBC" "diet.xls":  ampl? FOOD <- [FOOD], cost, f_min, f_max;  ampl: read table Foods;

如果这些值位于包含如下文本表的文件 Foods.tab 中：

mpl.tab 1 3 FOOD cost f_min f_max BEEF 3.19 2 10 CHK 2.59 2 10 FISH 2.29 2 10 HAM 2.89 2 10 MCH 1.89 2 10 MTL 1.99 2 10 SPG 1.99 2 10 TUR 2.49 2 10

则将使用文本表的声明：

mpl: model diet.mod;  appl: table Foods IN: FOOD <- [FOOD], cost, f_min, f_max;  appl: read table Foods;

由于在以上三个例子中 AMPL 表名 Foods 是相同的，因此 read table 命令也相同：read table Foods。一般来说，read table 命令仅指定要读取的表的 AMPL 名称。所有关于要读取内容以及如何处理的信息都来自于表声明中命名表的定义。

为了创建上一节中的第二个（7 列）关系表示例，我们可以使用一对表声明：

table ImportFoods IN "ODBC" "diet.mdb" "Foods": FOOD <- [FOOD], cost, f_min, f_max; table ExportFoods OUT "ODBC" "diet.mdb" "Foods": FOOD <- [FOOD], Buy, Buy.rc ~ BuyRC, {j in FOOD} Buy[j]/f_max[j] ~ BuyFrac;

或使用一个结合了输入和输出信息的单一表声明：

table Foods "ODBC" "diet.mdb": [FOOD] IN, cost IN, f_min IN, f_max IN, Buy OUT, Buy.rc ~ BuyRC OUT, {j in FOOD} Buy[j]/f_max[j] ~ BuyFrac OUT;

这些示例展示了 AMPL 表名称（如 ExportFoods）可以与外部文件中对应表的名称（由后续字符串 "Foods" 指示）不同。这里还展示了其他一些有用的选项：IN 和 OUT 与表的各个列相关联，而不是与整个表相关联；[FOOD] IN 用作 FOOD <- [FOOD] 的缩写；表的列与变量 Buy 的值以及表达式 Buy.rc 和 Buy[j]/f_max[j] 相关联；Buy.rc ~ BuyRC 和 {j in FOOD} Buy[j]/f_max[j] ~ BuyFrac 将 AMPL 表达式（~ 运算符左侧）与数据库列标题（~ 运算符右侧）相关联。

为了将有意义的结果写回到 Access 数据库，我们需要读取所有饮食模型的数据，然后求解，最后给出 write table 命令。以下是使用单独的表声明来读取和写入 Access 表 Foods 的方式：

ampl: model diet.mod; ampl: table ImportFoods IN "ODBC" "diet.mdb" "Foods": FOOD <- [FOOD], cost, f_min, f_max; ampl: table Nutrs IN "ODBC" "diet.mdb": NUTR <- [NUTR], n_min, n_max; ampl: table Amts IN "ODBC" "diet.mdb": [NUTR, FOOD], amt; ampl: read table ImportFoods; ampl: read table Nutrs; ampl: read table Amts; ampl: solve; ampl: table ExportFoods OUT "ODBC" "diet.mdb" "Foods": FOOD <- [FOOD], Buy, Buy.rc ~ BuyRC, {j in FOOD} Buy[j]/f_max[j] ~ BuyFrac; ampl: write table ExportFoods;

以下是使用单一声明来读取和写入 Foods 的替代方法：

ampl: model diet.mod; ampl: table Foods "ODBC" "diet.mdb": [FOOD] IN, cost IN, f_min IN, f_max IN, Buy OUT, Buy.rc ~ BuyRC OUT, {j in FOOD} Buy[j]/f_max[j] ~ BuyFrac OUT; ampl: table Nutrs IN "ODBC" "diet.mdb": NUTR <- [NUTR], n_min, n_max; ampl: table Amts IN "ODBC" "diet.mdb": [NUTR, FOOD], amt; ampl: read table Foods; ampl: read table Nutrs; ampl: read table Amts; ampl: solve; ampl: write table Foods;

无论哪种方式，Access 表 Foods 最终都会增加三列，如图 10-4 所示。

对于其他类型的数据库文件，相同的操作处理方式类似。通常，`write table` 命令的行为由命令中先前声明的 AMPL 表以及通过表声明与 AMPL 表关联的外部文件状态决定。根据具体情况，`write table` 命令可能会创建一个新的外部文件或表，覆盖现有表，覆盖现有表中的某些列，或向现有表追加列。

对于多维 AMPL 实体，表声明是相同的，只是在方括号 `\[` 和 `\]` 之间必须指定多个键列。对于前面讨论的钢铁生产示例，与关系表的对应关系可以这样设置：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/4acd9f0340c42383425c5084945a7b2f8d211b55ef6fe72274829b8d617f7120.jpg)  
图 10-4：带有输出列的 Access 关系表。

```ampl
table SteelProd "ODBC" "steel.mdb": [PROD, TIME], market IN, revenue IN, Make OUT, Sell OUT, Inv OUT;
```

这里键列 `PROD` 和 `TIME` 没有被指定为 `IN`。这是因为要读入的参数 `market` 和 `revenue` 在 AMPL 模型中是基于集合 `{PROD, 1..T}` 索引的，而该集合的成员资格将通过使用其他更简单的表来指定。`read table SteelProd` 命令仅使用数据库每行中的 `PROD` 和 `TIME` 条目来确定与该行中 `market` 和 `revenue` 条目关联的索引对（下标）。

我们的运输示例也涉及二维实体的关系表，相关的表声明类似：

```ampl
table TransLinks "ODBC" "trans.xls" "Links": LINKS <- [ORIG, DEST], cost IN, Trans OUT;
```

这里的区别在于，`LINKS`（`cost` 和 `Trans` 所基于的 AMPL 索引对集合）是数据的一部分，而不是从更简单的集合或参数中确定的。因此我们写成 `LINKS <- [ORIG, DEST]`，以请求在将相应值读入 `cost` 的同时，将键列中的对读入 `LINKS`。这一区别将在下一节进一步讨论。

从我们目前的简单示例中可以看出，表语句在交互式输入时往往比较繁琐。因此它们通常被放置在 AMPL 程序或脚本中，并按第 13 章所述方式执行。`read table` 和 `write table` 语句也可以包含在脚本中。您可以定义一个表并

然后立即读取或写入它，正如我们在一些示例中所看到的那样，但如果将复杂的 `table` 语句与读取和写入表的语句分开，则脚本通常更具可读性。

本章的其余部分将重点介绍 `table` 语句。完整的示例脚本以及饮食、生产与运输示例的 Access 或 Excel 文件可以从 AMPL 网站获取。

# 10.3 从关系表中读取数据

若要仅从外部关系表中读取数据，应使用一个指定读/写状态为 `IN` 的表声明。因此，它应具有如下的一般形式：

```ampl
table 表名 IN 字符串列表opt : 键规范, 数据规范, 数据规范, ... ;
```

其中可选的字符串列表特定于所使用的数据库类型和访问方法。（为了简洁起见，大多数后续示例中未显示字符串列表。）键规范命名键列，数据规范给出数据列。随后通过命令 `read table 表名 ;` 将数据值从表中读入 AMPL 实体，该命令通过参考定义表名的表声明来确定要读取的值。

# 仅读取参数

要将数据列中的值赋给同名的 AMPL 参数，只需提供一个括号括起来的键列列表，然后是数据列的列表。最简单的情况是只有一个键列，例如：

table Foods IN: [FOOD], cost, f_min, f_max;

这表明关系表有四列，包括一个键列 FOOD 和三个数据列 cost、f_min 和 f_max。这些数据列与当前 AMPL 模型中的参数 cost、f_min 和 f_max 相关联。由于只有一个键列，所有这些参数必须在一维集合上进行索引。

当执行命令 read table Foods 时，关系表将逐行读取。某行在键列中的条目被解释为每个参数的下标，这些带下标的参数将被赋予该行中相关数据列的值。例如，如果关系表如下所示：

<table><tr><td>FOOD</td><td>cost</td><td>f_min</td><td>f_max</td></tr><tr><td>BEEF</td><td>3.19</td><td>2</td><td>10</td></tr><tr><td>CHK</td><td>2.59</td><td>2</td><td>10</td></tr><tr><td>FISH</td><td>2.29</td><td>2</td><td>10</td></tr><tr><td>HAM</td><td>2.89</td><td>2</td><td>10</td></tr><tr><td>MCH</td><td>1.89</td><td>2</td><td>10</td></tr><tr><td>MTL</td><td>1.99</td><td>2</td><td>10</td></tr><tr><td>SPG</td><td>1.99</td><td>2</td><td>10</td></tr><tr><td>TUR</td><td>2.49</td><td>2</td><td>10</td></tr></table>

处理第一行时，将值 3.19 赋给参数 cost['BEEF']，2 赋给 f_min['BEEF']，10 赋给 f_max['BEEF']；处理第二行时，将 2.59 赋给 cost['CHK']，2 赋给 f_min['CHK']，10 赋给 f_max['CHK']；依此类推，直到处理完剩余的六行。

在执行读取表格命令时，AMPL 不会对参数的声明方式做任何假设；它们不需要在名为 FOOD 的集合上建立索引，实际上，其索引集合的成员甚至可能尚未确定。只有在 AMPL 首次在某些计算中使用每个参数时，它才会检查从关键列 FOOD 中读取的条目，以确保每个条目都是该参数的有效下标。

对于多维参数，情况是类似的。每个数据列的名称也必须是 AMPL 参数的名称，并且该参数的索引集合的维度必须等于关键列的数量。例如，当在括号内列出两个关键列时：

table SteelProd IN: [PROD, TIME], market, revenue;

所列出的数据列 market 和 revenue 必须对应于在二维集合上索引的 AMPL 参数 market 和 revenue。

当执行 read table SteelProd 时，每行关键列中的条目将被解释为每个参数的一对下标。因此，如果关系表的内容如下：

<table><tr><td>PROD</td><td>TIME</td><td>market</td><td>revenue</td></tr><tr><td>bands</td><td>1</td><td>6000</td><td>25</td></tr><tr><td>bands</td><td>2</td><td>6000</td><td>26</td></tr><tr><td>bands</td><td>3</td><td>4000</td><td>27</td></tr><tr><td>bands</td><td>4</td><td>6500</td><td>27</td></tr><tr><td>coils</td><td>1</td><td>4000</td><td>30</td></tr><tr><td>coils</td><td>2</td><td>2500</td><td>35</td></tr><tr><td>coils</td><td>3</td><td>3500</td><td>37</td></tr><tr><td>coils</td><td>4</td><td>4200</td><td>39</td></tr></table>

处理第一行时，会将 6000 赋值给 market['bands', 1]，将 25 赋值给 revenue['bands', 1]；处理第二行时，会将 6000 赋值给 market['bands', 2]，将 26 赋值给 revenue['bands', 2]；以此类推，处理完全部八行。当 AMPL 首次需要这些参数的值时，关键列条目给出的下标对必须对 market 和 revenue 有效，但这些参数不需要在名为 PROD 和 TIME 的集合上声明。（实际上，在这个示例所来源的模型中，参数是在集合 $\{\mathrm{PROD}, 1 \ldots \mathrm{T}\}$ 上索引的，其中 $\mathrm{T}$ 是一个先前定义的参数。）

由于关系表只有一组关键列，因此 AMPL 会对每个由数据列命名的参数应用相同的下标。因此，这些参数通常在同一个 AMPL 集合上索引。然而，通过在对应于无效下标的行中留空条目，也可以在一个数据库表中容纳在相似集合上索引的参数。表示空条目的方式取决于所使用的数据库软件。

无索引 (标量) 参数的值可以通过一个具有一行且无键列的关系表来提供，这样每个数据列恰好包含一个值。相应的表声明具有一个空的键规范，即 [ ]。例如，为了读取参数 $\mathrm{T}$ 的值，该参数给出了 steelT.mod 中的周期数，表声明为 table SteelPeriods IN: [ ], T;

相应的关联表有一列，也命名为 $\mathrm{T}$，其一个条目为正整数。

# 读取集合和参数

通常方便的是从表的键列中读取集合的成员，同时从数据列中读取在该集合上索引的参数。为了指示应从表中读取一个集合，表声明中的键规范写成如下形式：

set-name <- [key-col-spec, key-col-spec, ...]

符号 <- 被设计为一个箭头，指向信息移动的方向，即从键列到 AMPL 集合。

最简单的情况涉及读取一维集合以及在其上索引的参数，例如 diet.mod 中的示例：

table Foods IN: FOOD <- [FoodName], cost, f_min, f_max;

当执行命令 read table Foods 时，关系表中键列 FoodName 的所有条目都被读入 AMPL 作为集合 FOOD 的成员，而数据列 cost、f_min 和 f_max 中的条目则按照之前描述的方式读入同名的 AMPL 参数中。如果键列命名为 FOOD，与 AMPL 集合相同，则适当的表声明变为 table Foods IN: FOOD <- [FOOD], cost, f_min, f_max;

在这种特殊情况下，键规范也可以写成缩写形式 [FOOD] IN。

读取多维集合及在其上索引的参数时采用类似的语法。例如在 transp3.mod 中，表声明可以为：

table TransLinks IN: LINKS <- [ORIG, DEST], cost;

当执行 read table TransLinks 时，表中的每一行提供来自键列 ORIG 和 DEST 的一对条目。所有这样的对被读入 AMPL 作为二维集合 LINKS 的成员。最后，列 cost 中的条目按照常规方式读入参数 cost。

与我们之前的多维示例一样，括号中的名称不必对应于 AMPL 模型中的集合。括号中的名称仅用于标识键列。箭头左侧的名称是唯一必须命名先前声明的 AMPL 集合的名称；此外，该集合必须被声明为具有与键列数量相同的维度或元数。

从关系表中读取集合 LINKS 是合理的，因为 LINKS 在模型中被明确声明，使得相应的数据需要单独读取：

```ampl
set ORIG; set DEST; set LINKS within {ORIG, DEST}; param cost {LINKS} >= 0;
```

相比之下，在类似的模型 `transp2.mod` 中，`LINKS` 是通过两个一维集合定义的：

```ampl
set ORIG; 
set DEST; 
set LINKS = {ORIG, DEST}; 
param cost {LINKS} >= 0;
```

而在 `transp.mod` 中，根本没有定义命名的二维集合：

```ampl
set ORIG; 
set DEST; 
param cost {ORIG, DEST} >= 0;
```

在这些情况下，读取参数 `cost` 仍然需要一个表格声明，但它不会指定任何关联集合的读取：

```ampl
table TransLinks IN: [ORIG, DEST], cost;
```

而是使用单独的关系表来为一维集合 `ORIG` 和 `DEST` 提供成员，以及为在其上建立索引的参数提供值。

当表格声明指定一个 AMPL 集合被赋予成员时，其数据规范（data-specs）列表可以为空。在这种情况下，只会读取关键列，并且读表的唯一操作是将关键列的值作为指定 AMPL 集合的成员进行赋值。例如，使用如下语句：

```ampl
table TransLinks IN: LINKS <- [ORIG, DEST];
```

随后的 `read table` 语句将仅从相应数据库表中的两个关键列中读取集合 `LINKS` 的值。

# 建立对应关系

AMPL 模型中的集合和参数声明并不一定在所有方面都与相关数据库中表的组织方式相对应。当差异较大时，可能需要使用数据库的查询语言（通常是 SQL）来派生具有模型所需结构的临时表；本章后面关于 ODBC 处理程序的讨论中给出了一个示例。然而，许多常见且简单的差异可以通过表格声明的功能直接处理。

命名上的差异可能是最常见的。表格声明可以通过形如 `param-name ~ data-col-name` 的数据规范（data-spec），将一个数据列与名称不同的 AMPL 参数关联起来。例如，如果表 `Foods` 被定义为：

```ampl
table Foods IN: [FOOD], cost, f_min ~ lowerlim, f_max ~ upperlim;
```

那么 AMPL 参数 `f_min` 和 `f_max` 将从关系表中的数据列 `lowerlim` 和 `upperlim` 中读取。（参数 `cost` 仍像以前一样从列 `cost` 中读取。）

一种类似的广义形式 `index ~ key-col-name` 可用于将一种虚拟索引与关键列关联起来。然后可以在一个或多个数据规范（data-spec）中的可选参数名（param-name）的下标中使用该索引。这种安排在许多情况下非常有用，其中关键列条目并不完全对应于接收表格值的参数的下标。以下是三种常见的情形。

当关系表中的某种编号系统与 AMPL 模型中的对应编号系统存在系统性差异时，可以通过涉及键列索引的简单表达式将一种编号方案转换为另一种编号方案。例如，如果关系表数据中的时间段是从 0 开始计数，而不是像模型中那样从 1 开始，则可以在表声明中进行如下调整：

```ampl
table SteelProd IN: [p \~ PROD, t \~ TIME], market[p,t+1] \~ market, revenue[p,t+1] \~ revenue;
```

在第二种情况下，如果 AMPL 参数具有来自相同集合但顺序不同的下标，则必须使用键列索引以提供正确的索引顺序。例如，如果 `market` 的索引为 `{PROD, 1..T}`，而 `revenue` 的索引为 `{1..T, PROD}`，则用于读取这两个参数值的表声明应写为如下形式：

```ampl
table SteelProd IN: [p \~ PROD, t \~ TIME], market, revenue[t,p] \~ revenue;
```

最后，如果某个 AMPL 参数的值被分配到多个数据库列中，则可以使用键列索引来描述每个列中包含的值。例如，如果收入 (revenue) 值在一列中为“带钢 (bands)”，在另一列中为“线圈 (coils)”，则相应的表声明可以写成如下形式：

```ampl
table SteelProd IN: [t \~ TIME], revenue["bands",t] \~ revbands, revenue["coils",t] \~ revcoils;
```

人们可能会尝试通过省略 `\~ data-colname` 来缩短此类声明，例如：

```ampl
table SteelProd IN: [p \~ PROD, t \~ TIME], market, revenue[t,p]; # ERROR
```

然而，这通常会被视为错误，因为在大多数数据库软件中，`revenue[t,p]` 不是关系表列的有效名称。相反，必须写成如下形式：

```ampl
table SteelProd IN: [p \~ PROD, t \~ TIME], market, revenue[t,p] \~ revenue;
```

以表明 AMPL 参数 `revenue[t,p]` 从表的 `revenue` 列中获取值。

更一般地，在任何情况下，只要列数据接收方的 AMPL 表达式本身不是数据库列的有效名称，就必须使用 `\~ synonym`。有效列名的规则通常与 AMPL 模型中有效组件名称的规则相同，但根据用于创建和维护表的数据库软件的不同，其细节可能会有所变化。

# 读取其他值

在用于输入的表声明中，任何允许参数名出现的位置都可以出现可赋值的 AMPL 表达式。如果一个表达式可以被赋值（例如，通过将其放在 `let` 命令中的 `$\equiv$` 左侧），则该表达式就是可赋值的。

变量名是可赋值的表达式。因此，表声明可以指定要读入变量的数据列，以便用于评估先前存储的解，或为求解器 (solver) 提供一个良好的初始解。

约束名称也是可赋值表达式。某些求解器 (如 MINOS) 将“读入约束”的值解释为初始对偶值。

任何通过可赋值后缀限定的变量或约束名称也是可赋值表达式。可赋值后缀包括预定义的  `.sstatus`  和  `.relax` ，以及任何用户定义的后缀。例如，如果饮食问题被修改为包含整数变量，则以下表格声明可以帮助为 CPLEX 求解器提供有用信息（参见第 14.3 节）：

```
table Foods IN: FOOD IN, cost, f_min, f_max, Buy, Buy.priority ~ prior;
```

执行  `read table Foods`  将按常规方式为集合  `FOOD`  提供成员，并为参数  `cost` 、 `f_min`  和  `f_max`  提供值，同时还将为  `Buy`  变量分配初始值和分支优先级。

# 10.4 将数据写入关系表

若要将外部关系表仅用于写入，应使用指定其读/写状态为  `OUT`  的表格声明。此类声明的一般形式为：

```
table table-name OUT string-list opt : key-spec, data-spec, data-spec, ...
```

其中可选的  `string-list`  特定于所使用的数据库类型和访问方法。（同样，大多数后续示例不包括  `string-list` 。）随后通过命令：

```
write table table-name
```

将 AMPL 表达式的值写入表中，该命令使用定义  `table-name`  的表格声明来确定要写入的信息。

用于写入的表格声明指定了一个外部文件，以及可能在该文件中显式（在  `string-list`  中）或隐式（通过默认规则）指定的关系表。通常，如果命名的外部文件或表不存在，则会创建它；否则将被覆盖。若要指定某些列应被替换或添加到表中，则表格声明必须包含一个或多个读/写状态为  `IN`  或  `INOUT`  的  `data-spec` ，如第 10.5 节所述。特定的表处理程序也可能有其自身的更详细规则，用于确定何时修改或覆盖文件和表，详见其文档说明。

用于写入外部表的表格声明中的  `key-spec`  和  `data-spec`  在表面上类似于用于读取的那些。然而，写入时允许的 AMPL 表达式范围要广泛得多，基本上包括所有集合值和数值表达式。此外，读取时所读取的表行是某些现有表的行，而要写入的行必须通过表格声明中的某些 AMPL 表达式来确定。具体而言，要写入的行可以从  `data-spec`  使用与  `display`  命令相同的约定推断出来，也可以从  `key-spec`  推断出来。下面将描述这两种替代方式所采用的特征表语法。

# 从数据规范推断要写入的行

如果 键规范 (key-spec) 只是用方括号括起来的键列名称列表， `[键列名, 键列名, ..]` ，那么表声明的工作方式就与 display 命令非常相似。它通过获取数据规范 (data-spec) 中显式或隐式声明的索引集的并集，来确定要写入的外部表行。数据规范列表的格式与 display 中的相同，只是列出的所有项目必须具有相同的维度。

在最简单的情况下，数据规范是索引在同一集合上的模型组件的名称：

table Foods OUT: [FoodName], f_min, Buy, f_max;

当执行 `write table Foods` 时，它会创建一个键列 `FoodName` 和数据列 `f_min`、`Buy` 和 `f_max`。由于与数据列对应的 AMPL 组件都索引在 AMPL 集合 `FOOD` 上，因此会为 `FOOD` 的每个成员创建一行。在代表性行中，`FOOD` 的一个成员会被写入键列 `FoodName`，而 `f_min`、`Buy` 和 `f_max` 下标为该成员的值会被写入同名的数据列中。对于饮食示例中使用的数据，生成的关系表如下：

<table><tr><td>FoodName</td><td>f_min</td><td>Buy</td><td>f_max</td></tr><tr><td>BEEF</td><td>2</td><td>5.36061</td><td>10</td></tr><tr><td>CHK</td><td>2</td><td>2</td><td>10</td></tr><tr><td>FISH</td><td>2</td><td>2</td><td>10</td></tr><tr><td>HAM</td><td>2</td><td>10</td><td>10</td></tr><tr><td>MCH</td><td>2</td><td>10</td><td>10</td></tr><tr><td>MTL</td><td>2</td><td>10</td><td>10</td></tr><tr><td>SPG</td><td>2</td><td>9.30605</td><td>10</td></tr><tr><td>TUR</td><td>2</td><td>2</td><td>10</td></tr></table>

与高维集合对应的表以类似方式处理，键规范中列出的用方括号括起来的键列名称的数量等于数据规范中项目的维度。因此，包含 `steelT.mod` 结果的表可以定义为 `table SteelProd OUT: [PROD, TIME], Make, Sell, Inv;`

由于 `Make` 和 `Sell` 索引在 \(\{\mathrm{PROD},1. .T\}\) 上，而 `Inv` 索引在 \(\{\mathrm{PROD},0. .T\}\) 上，随后的 `write table SteelProd` 命令将生成一个表，其中每一行对应这些集合并集的一个成员：

<table><tr><td>PROD</td><td>TIME</td><td>Make</td><td>Sell</td><td>Inv</td></tr><tr><td>bands</td><td>0</td><td>.</td><td>.</td><td>10</td></tr><tr><td>bands</td><td>1</td><td>5990</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>2</td><td>6000</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>3</td><td>1400</td><td>1400</td><td>0</td></tr><tr><td>bands</td><td>4</td><td>2000</td><td>2000</td><td>0</td></tr><tr><td>coils</td><td>0</td><td>.</td><td>.</td><td>0</td></tr><tr><td>coils</td><td>1</td><td>1407</td><td>307</td><td>1100</td></tr><tr><td>coils</td><td>2</td><td>1400</td><td>2500</td><td>0</td></tr><tr><td>coils</td><td>3</td><td>3500</td><td>3500</td><td>0</td></tr><tr><td>coils</td><td>4</td><td>4200</td><td>4200</td><td>0</td></tr></table>

在 `Make` 和 `Sell` 列中，有两行是空的，因为 `("banda", 0)` 和 `("coils", 0)` 并不是 `Make` 和 `Sell` 索引集的成员。我们在这里使用 `.` 来表示空的表格条目，但实际的空条目外观和处理方式将根据所使用的数据库软件的不同而有所不同。

如果这种形式被应用于书写带后缀的 变量 (variable) 或 约束 (constraint) 名称，例如与约束 `Diet` 相关的对偶变量 (dual) 和松弛变量 (slack) 值：

```ampl
table Nutrs OUT: [Nutrient], Diet.lslack, Diet.ldual, Diet.uslack, Diet.udual; # 错误
```

随后的 `write table Nutrs` 命令很可能会被拒绝执行，因为大多数数据库软件不允许中间带有 `.` 的名称作为列名。

```ampl
ampl: write table Nutrs;
执行 "write table" 命令时出错:
使用表处理程序 ampl.odbc 写入表 Nutrs 时出错:
列 2 的名称 "Diet.lslack" 包含非字母数字字符 '.'。
```

在这种情况下，每个 AMPL 表达式后面必须跟运算符 `\~` 和一个在关系表中使用的有效列名：

```ampl
table Nutrs OUT: [Nutrient],
Diet.lslack \~ lb_slack, Diet.ldual \~ lb_dual,
Diet.uslack \~ ub_slack, Diet.udual \~ ub_dual;
```

这表示由 Diet.lslack 表示的值应放在名为 lb_slack 的列中，由 Diet.ldual 表示的值应放在名为 lb_dual 的列中，以此类推。以这种方式定义表后，`write table Nutrs` 命令将生成预期的关系表：

<table><tr><td>Nutrient</td><td>lb_slack</td><td>lb_dual</td><td>ub_slack</td><td>ub_dual</td></tr><tr><td>A</td><td>1256.29</td><td>0</td><td>18043.7</td><td>0</td></tr><tr><td>B1</td><td>336.257</td><td>0</td><td>18963.7</td><td>0</td></tr><tr><td>B2</td><td>0</td><td>0.404585</td><td>19300</td><td>0</td></tr><tr><td>C</td><td>982.515</td><td>0</td><td>18317.5</td><td>0</td></tr><tr><td>CAL</td><td>3794.62</td><td>0</td><td>4205.38</td><td>0</td></tr><tr><td>NA</td><td>50000</td><td>0</td><td>0</td><td>-0.00306905</td></tr></table>

`\~` 也可用于不带后缀的名称，如果希望为数据库列分配一个不同于相应 AMPL 实体的名称。

数据列中的值的更一般表达式需要使用虚拟索引，其使用方式与 display 命令的数据列表中使用的方式相同。由于带索引的 AMPL 表达式很少是数据库的有效列名，因此它们通常应后跟 `\~` 数据列名，以为要写入的相应关系表列提供有效名称。例如，要写入一列 servings，包含每种食物的购买份数，以及一列 percent，给出购买量占允许最大量的百分比，表声明可以写成以下两种形式之一：

```ampl
table Purchases OUT: [FoodName],
Buy \~ servings, {j in FOOD} 100*Buy[j]/_max[j] \~ percent;
```

或者

```ampl
table Purchases OUT: [FoodName],
{j in FOOD} (Buy[j] \~ servings, 100*Buy[j]/_max[j] \~ percent);
```

无论哪种方式，由于两个 data-spec 都给出了在 AMPL 集合 FOOD 上索引的表达式，因此生成的表格中每行对应集合中的一个成员：

<table><tr><td>食物名称</td><td>份数</td><td>百分比</td></tr><tr><td>牛肉</td><td>5.36061</td><td>53.6061</td></tr><tr><td>鸡肉</td><td>2</td><td>20</td></tr><tr><td>鱼</td><td>2</td><td>20</td></tr><tr><td>火腿</td><td>10</td><td>100</td></tr><tr><td>MCH</td><td>10</td><td>100</td></tr><tr><td>牛奶</td><td>10</td><td>100</td></tr><tr><td>SPG</td><td>9.30605</td><td>93.0605</td></tr><tr><td>土耳其</td><td>2</td><td>20</td></tr></table>

data-spec 中的表达式也可以使用像 sum 这样的运算符，这些运算符会定义自己的虚拟索引。因此，对于 steelT.mod 模型，可以使用如下语句指定一个按周期划分的总产量和销售量表：

table SteelTotal OUT: [TIME], {t in 1..T} (sum {p in PROD} Make[p,t] ~ 已制造, sum {p in PROD} Sell[p,t] ~ 已销售);

作为一个二维示例，可以使用如下语句指定一个销售数量表以及满足需求比例的表：

table SteelSales OUT: [PROD, TIME], Sell, {p in PROD, t in 1..T} Sell[p,t]/market[p,t] ~ 需求比例;

生成的外部表将包含键列 PROD 和 TIME，以及数据列 Sell 和 需求比例。

# 根据键规范推断写入行

另一种形式的表声明指定了对于显式指定的 AMPL 集合的每个成员，要写入一行到表中。为了使声明以这种方式工作，键规范（key-spec）必须写成如下形式：

$$
集合规范 \rightarrow [键列规范, 键列规范, \ldots]
$$

与从键列列表指向 AMPL 集合并指示将值读入该集合的箭头 $<-$ 不同，这种形式使用箭头 $->$，它从 AMPL 集合指向键列列表，表示信息从集合写入到键列中。

行索引集的显式表达式由集合规范（set-spec）给出，它可以是 AMPL 集合的名称，或用花括号 $\{\}$ 括起来的任何 AMPL 集合表达式。键列规范（key-col-spec）给出了数据库中相应键列的名称。如有需要，虚拟索引可以出现在集合规范或键列规范中，我们将在后文展示。

这种形式最简单的情况是为模型组件在同一个一维集合上索引的情况写入数据库列，如 diet.mod 的示例所示：

table Foods OUT: FOOD -> [食物名称], f_min, Buy, f_max;

当执行 write table Foods 时，会为 AMPL 集合 FOOD 的每个成员创建一行。在该行中，集合成员会被写入键列“食物名称”，而以该集合成员为下标的 f_min、Buy 和 f_max 的值将被写入到相应的列中。

这些同名的数据列。（对于我们在饮食示例中使用的数据，生成的表格将与本节前面给出的 `FoodName` 表相同。）如果键列与 AMPL 集合具有相同的名称 `FOOD`，则相应的表格声明变为：

```
table Foods OUT: FOOD -> [FOOD], f_min, Buy, f_max;
```

在这种特殊情况下，键规范也可以简写为 `[FOOD] OUT`。

在 AMPL 名称和带后缀的名称中使用 `\~` 时，需遵循之前描述的规则，因此饮食松弛变量和对偶变量的示例应写为：

```
table Nutrs OUT: NUTR -> [Nutrient], Diet.1slack ~ lb_slack, Diet.ldual ~ lb_dual, Diet.uslack ~ ub_slack, Diet.udual ~ ub_dual;
```

使用 `write table Nutrs` 将生成与之前相同的表格。

对于数据列中的值，若需使用更一般的表达式，则需要使用虚拟索引。由于要写入的行由键规范确定，因此虚拟索引也在该处定义（而不是像上述替代形式中那样在数据规范中定义）。例如，若要指定一列包含所购买食物占允许最大值的百分比，则需写为：

$$
100 \star \mathrm{Buy}[\texttt{j}] / \texttt{f\_max}[\texttt{j}]
$$

这需要定义虚拟索引 `j`。该定义可以出现在如下形式的集合规范中：

```
table Purchases OUT: {j in FOOD} -> [FoodName], Buy[j] ~ servings, 100*Buy[j]/f_max[j] ~ percent;
```

或者出现在如下形式的键列规范中：

```
table Purchases OUT: FOOD -> [j ~ FoodName], Buy[j] ~ servings, 100*Buy[j]/f_max[j] ~ percent;
```

这两种形式是等价的。无论采用哪种方式，在写入每一行时，索引 `j` 都会取键列的值，并用于解释给出数据列值的表达式。在我们的示例中，生成的表格包含键列 `FoodName` 和数据列 `servings` 与 `percent`，其结果与之前相同。

类似地，之前 `SteelTotal` 表的示例可以写为：

```
table SteelTotal OUT: {t in 1..T} -> [TIME], sum {p in PROD} Make[p,t] ~ Made, sum {p in PROD} Sell[p,t] ~ Sold;
```

或者：

```
table SteelTotal OUT: {1..T} -> [t ~ TIME], sum {p in PROD} Make[p,t] ~ Made, sum {p in PROD} Sell[p,t] ~ Sold;
```

结果将包含键列 `TIME`（包含从 1 到 T 的整数）以及数据列 `Made` 和 `Sold`（包含两个求和表达式的值）。（注意，由于 `1..T` 是一个集合表达式而不是集合名称，因此必须用花括号括起来才能作为集合规范使用。）

对于更高维度集合的表格，处理方式类似，键列规范中括号内列出的键列数量应等于集合规范的维度。因此，包含来自 `steelT.mod` 结果的表格可以定义为：

表 SteelProd 输出：[PROD, 1. .T] -> [PROD, TIME]，Make, Sell, Inv；随后写入表 SteelProd 将生成如下形式的表格：

<table><tr><td>PROD</td><td>TIME</td><td>Make</td><td>Sell</td><td>Inv</td></tr><tr><td>bands</td><td>1</td><td>5990</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>2</td><td>6000</td><td>6000</td><td>0</td></tr><tr><td>bands</td><td>3</td><td>1400</td><td>1400</td><td>0</td></tr><tr><td>bands</td><td>4</td><td>2000</td><td>2000</td><td>0</td></tr><tr><td>coils</td><td>1</td><td>1407</td><td>307</td><td>1100</td></tr><tr><td>coils</td><td>2</td><td>1400</td><td>2500</td><td>0</td></tr><tr><td>coils</td><td>3</td><td>3500</td><td>3500</td><td>0</td></tr><tr><td>coils</td><td>4</td><td>4200</td><td>4200</td><td>0</td></tr></table>

此结果与之前 SteelProd 示例生成的表格不完全相同，因为此处要写入的行明确对应于集合 {PROD, 1. .T} 的成员，而不是从 Make、Sell 和 Inv 的索引集合推断出来的。特别是，Inv["bands", 0] 和 Inv["coils", 0] 的值不会出现在此表格中。

高维中虚拟索引 (dummy indices) 的选项与一维情况相同。因此，我们的 SteelSales 示例可以使用在集合规范中定义的虚拟索引编写：

```
table SteelSales OUT: [p in PROD, t in 1..T] -> [PROD, TIME], Sell[p,t] ~ sold, Sell[p,t]/market[p,t] ~ met;
```

或者使用添加到键列规范中的虚拟索引：

```
table SteelSales OUT: [PROD,1..T] -> [p ~ PROD, t ~ TIME], Sell[p,t] ~ sold, Sell[p,t]/market[p,t] ~ met;
```

如果虚拟索引恰好同时出现在集合规范和键列规范中，则以键列规范中的为准。

# 10.5 读写同一张表

要从关系表中读取数据，然后将结果写入同一张表，可以使用一对引用相同文件和表名的 table 声明。也可以将这些声明合并为一个，指定某些列用于读取，其他列用于写入。本节将给出这两种可能性的示例和说明。

# 使用两个 table 声明进行读写

单个外部表可以由一个 table 声明读取，之后再由另一个 table 声明写入。这两个 table 声明遵循上述读写规则。

然而，在这种情况下，通常希望写入表时添加或重写选定的列，而不是覆盖整个表。这种偏好可以通过在用于写入的 table 声明中包含输入列和输出列来传达给 AMPL 表处理器。可以通过逐列指定读/写状态（而不是为整个表指定），将用于 AMPL 输入的列与用于外部表输出的列区分开来。

例如，一个用于 `diet.mod` 的外部表可能包含 `cost`、`f_min` 和 `f_max` 列，用于模型输入，以及一个 `Buy` 列用于存放结果。如果该表作为 Microsoft Access 表保存在文件 `diet.mdb` 中，并命名为 `Diet`，那么用于将数据读入 AMPL 的表声明可以是：

```
table FoodInput IN "ODBC" "diet1.mdb" "Diet":
    FOOD <- [FoodName], cost, f_min, f_max;
```

相应的用于写入结果的声明将使用不同的 AMPL 表名，但引用相同的 Access 表和文件：

```
table FoodOutput "ODBC" "diet1.mdb" "Diet":
    [FoodName], cost IN, f_min IN, Buy OUT, f_max IN;
```

当执行 `read table FoodInput` 时，只有在 `FoodInput` 表声明中列出的三列会被读取；如果存在一个名为 `Buy` 的列，它将被忽略。之后，当问题被解决并执行 `write table FoodOutput` 时，只有在 `FoodOutput` 表声明中具有读写状态 `OUT` 的那一列会被写入 Access 表，而其他列则保持不变。

尽管具体细节可能因所使用的数据库软件不同而有所差异，但通常约定是：只有当表声明中所有数据列的读写状态均为 `OUT` 时，才意味着覆盖整个现有表或文件。否则，意味着对列进行选择性重写或新增。因此，如果我们的 AMPL 输出表声明为：

```
table FoodOutput "ODBC" "diet1.mdb" "Diet":
    [FoodName], Buy OUT;
```

那么执行 `write table FoodOutput` 会删除 Access 表 `Diet` 中的所有数据列。但如果我们使用如下声明：

```
table FoodOutput "ODBC" "diet1.mdb" "Diet":
    [FoodName], Buy;
```

则只会覆盖 `Buy` 列，就像我们最初给出的例子一样，因为存在一个不具有读写状态 `OUT` 的数据列（即 `Buy` 本身）。（未指定状态时，默认为 `INOUT`。）

# 使用相同表声明进行读写

在许多情况下，读取和写入外部表的所有信息都可以在同一个表声明中指定。`key-spec` 可以使用箭头 `<-` 将键列的内容读入 AMPL 集合，使用 `->` 将 AMPL 集合的成员写入键列，或使用 `<->` 同时完成这两项操作。`data-spec` 可以为列指定读写状态：`IN` 表示该列仅被读入 AMPL，`OUT` 表示该列仅从 AMPL 写出，`INOUT` 表示该列既被读取又被写入。

执行 `read table table-name` 命令时，只会读取在 `table-name` 声明中标记为 `IN` 或 `INOUT` 的键列或数据列。类似地，执行 `write table table-name` 命令时，只会向标记为 `OUT` 或 `INOUT` 的列写入数据。

例如，上面定义 `FoodInput` 和 `FoodOutput` 的声明可以替换为如下形式的表：

```ampl
table Foods "ODBC" "diet1.mdb" "Diet": FOOD <- [FoodName], cost IN, f_min IN, Buy OUT, f_max IN;
```

此时，读取表 `Foods` 将仅从关键列 `FoodName` 以及数据列 `cost`、`f_min` 和 `f_max` 中读取数据；而后续的写入操作将仅写入列 `Buy`。

# 10.6 表和列的索引集合

在某些情况下，声明一个索引的表集合，或在表中定义一个索引的数据列集合，会非常方便。本节将说明如何在表声明中指定这类索引。

为了说明表的索引集合，我们展示了一个脚本（第 13 章），该脚本能自动求解一系列分别存储的场景。为了说明列的索引集合，我们展示了如何读取一个二维电子表格表。

我们所有这些特性的示例都使用了 AMPL 的字符串表达式来生成一系列文件、表或列的名称。有关字符串表达式的更多信息，请参见第 13.7 节和 A.4.2 节。

# 表的索引集合

AMPL 的表声明可以像集合、参数和其他模型组件一样进行索引。在表名后可以添加一个可选的 `{indexing-expr}`：

```ampl
table table-name {indexing-expr}opt string-listopt : ...
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/9b462a15392509b52558d6e8a4c7acb160135322750822727886eb7885e9f0b2.jpg)  
图 10-5：包含敏感性分析表的 Access 数据库。

对于索引表达式所指定集合的每一个成员，都会定义一个表。该集合中的单个表通过在表名后附加带括号的下标来表示，方式与常规相同。

例如，以下声明定义了一个索引于 `diet.mod` 中食物集合的 AMPL 表集合，每个表对应于 Access 文件 `DietSens.mdb` 中的不同数据库表：

```ampl
table DietSens {j in FOOD} OUT "ODBC" "DietSens.mdb" ("Sens" & j): [Food], f_min, Buy, f_max;
```

根据标准 ODBC 表处理器的规则，Access 表名由字符串列表中的第三项给出，即字符串表达式 `("Sens" & j)`。因此，AMPL 表 `DietSens["BEEF"]` 对应于 Access 表 `SensBEEF`，AMPL 表 `DietSens["CHK"]` 对应于 Access 表 `SensCHK`，依此类推。以下 AMPL 脚本利用这些表记录了当每种食物“买一送一”时的最优饮食方案：

```ampl
for {j in FOOD} {
    let cost[j] := cost[j] / 2;
    solve;
    write table DietSens[j];
    let cost[j] := cost[j] * 2;
}
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/ddac64d5c969637bbfa778a717a69cebe8fca5d54f817b00ef8168d8198e1414.jpg)  
图 10-6：用于敏感性分析的替代 Access 表。

对于 `diet2a.dat` 中的数据，集合 `FOOD` 包含八个成员，因此在 Access 数据库中会写入八个表，如图 10-5 所示。如果表声明中字符串列表的第二个字符串（指定 Access 文件名）改为一个字符串表达式：

table DietSens {j in FOOD} OUT "ODBC" ("DietSens" & j & ".mdb"): [Food], f_min, Buy, f_max;

那么 AMPL 将写入八个不同的 Access 数据库文件，分别命名为 DietSensBEEF.mdb、DietSensCHK.mdb 等，每个文件包含一个默认名称为 DietSens 的表。（这些文件必须在执行 write table 命令之前创建完成。）

类似地，可以使用字符串表达式使索引集合中的每个 AMPL 表对应同一个 Access 表，但最优数量的数据列名不同：

table DietSens {j in FOOD} "ODBC" "DietSens.mdb": [Food], Buy ~ ("Buy" & j);

然后运行上述脚本将生成图 10-6 所示的 Access 表。在这种情况下，AMPL 表故意保留了默认的读写状态 INOUT。如果将读写状态指定为 OUT，则每个 write table 命令都会覆盖前一个命令创建的列。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/47aa2af964d71ff89567ae994bb79a09b7f8870cd826e03d3b9edbee64e6aafb.jpg)  
图 10-7：Excel 中的二维 AMPL 表。

# 数据列的索引集合

由于关系表的数据列与 AMPL 模型中实体的索引集合之间存在自然对应关系，因此表声明中的每个 data-spec 通常指向不同的 AMPL 参数、变量或表达式。然而，偶尔一个 AMPL 实体的值会被分散到多个数据列中。这种情况可以通过定义一个数据列集合来处理，其中每个列对应指定索引集的一个成员。

此功能最常见的用途是读取或写入二维表。例如，diet.mod 中参数的数据可能在 Excel 电子表格中表示为一个表，其中营养素标记行，食物标记列（见图 10-7）。要使用 AMPL 的外部数据库功能读取此表，我们必须将其视为具有一列关键列（标题为 NUTR）和多个以各个食物名称为标题的数据列。因此，我们需要一个键规范为一维且数据规范在 AMPL 集合 FOOD 上索引的表声明：

table dietAmts IN "ODBC" "Diet2D.xls": [i ~ NUTR], {j in FOOD} <amt[i,j] ~ (j)>;

键规范 [i ~ NUTR] 将第一个表列与集合 NUTR 关联。数据规范 $\{\mathrm{j~in~FOOD}\} < \ldots >$ 使 AMPL 为集合 FOOD 的每个成员生成一个单独的数据规范。具体来说，对于 FOOD 中的每个 j，AMPL 生成数据规范 amt[i,j] ~ (j)，其中 (j) 是食物 j 对应的外部表列标题的 AMPL 字符串表达式，而 amt[i,j] 表示该列中的值要写入的参数。（根据此处及其他 AMPL 声明和命令中使用的约定，(j) 周围的括号使其被解释为字符串表达式；没有括号时，它将表示名为 j 的单字符列名。）

类似的方法也适用于将二维表格写入电子表格。例如，在求解 `steelT.mod` 后，可以使用以下表格声明将结果写入电子表格：

```
table Results1 OUT "ODBC" "steel1out.xls": 
    {p in PROD} -> [Product], 
    Inv[p,0] ~ Inv0, 
    {t in 1..T} < 
        Make[p,t] ~ ('Make' & t), 
        Sell[p,t] ~ ('Sell' & t), 
        Inv[p,t] ~ ('Inv' & t) 
    >;
```

或者，使用显示风格的索引方式等价地写为：

```
table Results2 OUT "ODBC" "steel2out.xls": 
    [Product], 
    {p in PROD} Inv[p,0] ~ Inv0, 
    {t in 1..T} < 
        {p in PROD} (
            Make[p,t] ~ ('Make' & t), 
            Sell[p,t] ~ ('Sell' & t), 
            Inv[p,t] ~ ('Inv' & t)
        ) 
    >;
```

关键列用产品名称标记行。数据列包括一列表示初始库存，然后是三列分别表示每个周期的生产量、销售量和库存量，如图 10-8 所示。从概念上讲，二维表格的行索引和列索引之间具有对称性。但由于这些示例中的表格被当作关系表处理，因此表格声明必须以不同的方式处理行索引和列索引。结果是，描述行索引的表达式与描述列索引的表达式有显著不同。

正如这些示例所表明的，指定表格列的索引集合的一般形式为：

$$
\{\text{indexing-expr}\} < \text{data-spec},\text{data-spec},\text{data-spec},\ldots >
$$

其中每个 `data-spec` 具有之前给出的任意一种形式。对于由 `indexing-expr` 指定集合的每个成员，AMPL 会在尖括号 `< ... >` 内生成每个 `data-spec` 的一个副本。`indexing-expr` 还定义了一个或多个在索引集上运行的虚拟索引；这些索引用于 `data-spec` 中的表达式，并且还出现在用于指定外部数据库中列名的字符串表达式中。

# 10.7 标准和内置表格处理器

为了与外部数据库文件交互，AMPL 依赖于表格处理器（table handlers）。这些通常是共享库或动态链接库形式的附加组件，可以根据需要加载。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/9e7205715370e208f2fea4792fdee4aee1781acf7dbc3a00da113a032e196984.jpg)  
图 10-8：另一个二维 Excel 表格。

AMPL 自带一个可在 Microsoft Windows 下运行并通过开放式数据库连接 (Open Database Connectivity，ODBC) 应用程序接口通信的“标准”表格处理器；它能识别 Access、Excel 以及计算机上安装了 ODBC 驱动程序的其他应用程序所使用的格式的关系表。AMPL 或数据库软件的供应商可能会提供额外的处理器。

除了提供的处理器之外，AMPL 还内置了用于测试的最小 ASCII 和二进制关系表文件处理器。供应商可能还会包含其他内置处理器。如果你不确定当前 AMPL 副本识别哪些处理器，A.13 节中描述的功能可以帮助你获取当前活动处理器的列表以及使用它们的简要说明。

正如本章的引言示例所示，AMPL 通过表声明中的字符串列表 (string-list) 与处理器 (handler) 进行通信。字符串列表的形式和解释方式因处理器而异。本节的其余部分将描述 AMPL 的标准 ODBC 处理器所识别的字符串列表。在进行一般性介绍之后，将针对在前几节示例中频繁使用的两个应用程序——Access 和 Excel——提供具体说明。最后一小节将介绍内置的二进制 (binary) 和 ASCII 表处理器所识别的字符串列表。

# 使用标准 ODBC 表处理器

在以 table 表名 ... 开头的声明中，标准 ODBC 表处理器的字符串列表的一般形式为：

"ODBC" "connection-spec" "external-table-spec" opt "verbose" opt

第一个字符串告诉 AMPL，使用此表声明进行的数据传输应采用标准 ODBC 处理器。后续字符串则为该处理器提供指令。

第二个字符串标识了在执行 read table 表名 或 write table 表名 命令时要读取或写入的外部数据库文件。根据 connection-spec 的形式以及计算机上 ODBC 的配置，存在几种不同的可能性。

如果 connection-spec 是形如 name.ext 的文件名，其中 ext 是与已安装的 ODBC 驱动程序关联的 3 个字母的扩展名，则该命名文件即为数据库文件。在我们的多个示例中可以看到这种形式，其中形如 name.mdb 和 name.xls 的文件名分别指代 Access 和 Excel 文件。

其他形式的 connection-spec 更具体地与 ODBC 相关，其说明可参见在线文档。关于计算机上 ODBC 驱动程序、数据源名称、文件数据源及相关实体的配置信息，可以通过 Windows ODBC 控制面板进行查看和更改。

第三个字符串通常指定在执行 `read table` 或 `write table` 命令时要读取或写入的指定文件中的关系表 (relational table) 名称。如果省略第三个字符串，则关系表的名称将被视为与包含该表声明的 `table-name` 相同。在写入时，如果指定的表不存在，则会创建该表；如果表存在但表声明中的所有数据规范 (data-spec) 的读写状态 (read/write status) 均为 `OUT`，则该表将被覆盖。否则，写入操作将修改现有表；每个写入的列要么覆盖同名的现有列，要么成为追加到表中的新列。

或者，如果第三个字符串具有特殊形式 `"SQL=sql-query"`，

表格声明适用于通过结构化查询语言 (Structured Query Language, SQL) 语句临时创建的关系表。具体来说，首先通过执行 `sql-query` 所给定的 SQL 语句，在表格声明字符串列表中第二个字符串所指定的数据库文件中构建一个关系表。然后，将表格声明的常规解释应用于所构造的表格。由于向临时表写入数据没有意义，因此声明中指定的所有列的读写状态都应为 `IN`。通常 `sql-query` 是一条 `SELECT` 语句，这是 SQL 用于操作表格并创建新表的主要机制。

例如，如果您想在 `diet.mod` 中仅将成本 (cost) 小于等于 2.49 美元的食物作为数据读入，则可以使用 SQL 查询从数据库的食物表 (Foods) 中提取相关记录：

```ampl
table cheapFoods IN "ODBC" "diet.mdb" "SQL=SELECT * FROM Foods WHERE cost <= 2.49":
  FOOD <- [FOOD], cost, f_min, f_max;
```

然后，为了读取参数 `amt` 的相关数据（该参数以营养成分 (NUTR) 和食物 (FOOD) 为索引），您需要读取那些与成本小于等于 2.49 美元的食物相关的记录。以下是一种使用 SQL 查询提取所需记录的方法：

```ampl
option selectAmts "SELECT NUTR, Amts.FOOD, amt FROM Amts, Foods WHERE Amts.FOOD = Foods.FOOD and cost <= 2.49";

table cheapAmts IN "ODBC" "diet.mdb" ("SQL = " & $selectAmts):
  [NUTR, FOOD], amt;
```

在此我们使用了一个 AMPL 选项来存储包含 SQL 查询的字符串。然后表格声明的第三个字符串可以由相对较短的字符串表达式 `"SQL = " & $selectAmts` 给出。

在前三个字符串之后添加字符串 `verbose`，可以在 `read table` 或 `write table` 命令使用该表格声明时，请求诊断信息（例如 ODBC 报告的 `DSN=` 字符串）。

# 使用标准 ODBC 表处理器处理 Access 和 Excel

要设置用于读取或写入 Microsoft Access 文件的关系表对应关系，请在字符串列表的第二个字符串中将扩展名指定为 `mdb`：

```ampl
"ODBC" "filename.mdb" "external-table-spec" opt
```

第二个字符串所命名的文件必须存在，尽管在写入时它可能是一个尚未包含任何表的数据库。

要设置用于读取或写入 Microsoft Excel 电子表格的关系表对应关系，请在字符串列表的第二个字符串中将扩展名指定为 `xls`：

```ampl
"ODBC" "filename.xls" "external-table-spec" opt
```

在 这种 情况 下，第 二 个 字符串 标识 要 读取 或 写入 的 外部 Excel 工作簿 文件。对于 写入 操作，如果 第 二 个 字符串 所 指定 的 文件 尚 不存在，则 会 创建 该 文件。

第 三 个 字符串 所 指定 的 外部 表格 规范 (external-table-spec) 用于 识别 在 指定 文件 内 的 电子表格 范围 (spreadsheet range)，该 范围 将 被 读取 或 写入；如果 该 字符串 缺失，则 默认 采用 表格 声明 开头 给出 的 表名 (table-name)。在 读取 时，指定 的 范围 必须 存在 于 Excel 文件 中。在 写入 时，如果 该 范围 不存在，则 会在 一个 新建 的 工作表 (worksheet) 的 左上角 创建 该 范围，且 该 工作表 名 与 表名 相同。如果 该 范围 存在，但 表格 声明 中 所有 数据 规范 (data-spec) 的 读写 状态 (read/write status) 均 为 OUT，则 该 范围 会被 覆盖。否则，写入 操作 会 修改 现有 的 范围。所 写入 的 每一 列 要么 覆盖 具有 相同 名称 的 现有 列，要么 作为 新 列 追加 到 表格 中；所 写入 的 每一 行 要么 覆盖 具有 相同 关键 列 (key column) 条目 的 现有 行，要么 作为 新 行 追加 到 表格 中。

当 写入 操作 导致 现有 范围 扩展 时，会 分别 在 范围 的 底部 或 右侧 添加 行 或 列。所 添加 的 行 或 列 的 单元格 必须 为空；否则，写入 表格 的 尝试 将 失败，并且 write table 命令 会 引发 一条 错误 消息。在 表格 成功 写入 后，对应 的 范围 会被 创建 或 调整，以 精确 包含 该 表格 的 所有 单元格。

# 用于 文本 和 二进制 文件 的 内置 表格 处理器

出于 调试 和 演示 目的，AMPL 提供 了 两个 非常 简单 的 关系 表格 格式 的 内置 处理器。这些 格式 每个 文件 存储 一个 表格，并 传达 等效 的 信息。其中 一种 生成 可在 任何 文本 编辑器 中 查看 的 ASCII 文件，而 另 一种 则 创建 读写 速度 更快 的 二进制 文件。

对于 这些 处理器，表格 声明 的 字符串 列表 (string-list) 最多 包含 一个 字符串，用于 指定 包含 关系 表格 的 外部 文件。如果 该 字符串 形如 "filename.tab"，则 该 文件 被 视为 ASCII 文本 文件；如果 形如 "filename.bit"，则 被 视为 二进制 文件。如果 未 提供 字符串 列表，则 默认 为 文本 文件 table-name.tab。

在 读取 时，指定 的 文件 必须 存在。在 写入 时，如果 文件 不存在，则 会 创建 该 文件。如果 文件 存在，但 表格 声明 中 所有 数据 规范 的 读写 状态 均 为 OUT，则 该 文件 会被 覆盖。否则，写入 操作 会 修改 现有 文件；所 写入 的 每一 列 要么 替换 具有 相同 名称 的 现有 列，要么 作为 新 列 添加 到 表格 中。

文本 文件 的 格式 可以 通过 写入 一个 文件 并 在 文本 编辑器 中 查看 结果 来 检查。例如，以下 AMPL 会话，

ampl: model diet.mod;  
ampl: data diet2a.dat;  
ampl: solve;  
MINOS 5.5: optimal solution found.  
13 iterations, objective 118.0594032  
ampl: table ResultList OUT "DietOpt.tab":  
ampl? [FOOD], Buy, Buy.rc, {j in FOOD} Buy[j]/f_max[j];  
ampl: write table ResultList;

会 生成 一个 名为 DietOpt.tab 的 文件，其 内容 如下：

ampl.tab 1 3  FOOD Buy Buy.rc 'Buy[j]/f_max[j]'  BEEF 5.360613810741701 8.881784197001252e- 16 0.5360613810741701  CHK 2 1.1888405797101402 0.2  FISH 2 1.1444075021312856 0.2  HAM 10 - 0.30265132139812223 1  MCH 10 - 0.5511508951406658 1  MTL 10 - 1.3289002557544745 1  SPG 9.306052855924973 - 8.881784197001252e- 16 0.9306052855924973  TUR 1.9999999999999998 2.7316197783461176 0.19999999999999998

在第一行中，`ampl.tab` 标识这是一个 AMPL 关系表文本文件，后面跟着的是键列和非键列的数量。第二行给出了表的列名，可以是任意字符串。（在这种情况下，无需使用 ~ 运算符来指定有效的列名。）随后的每一行给出表中一行的值；数字以完整精度书写，不进行特殊格式化或对齐。
