
# 指定数据

正如我们在本书中反复强调的，优化问题的 AMPL 模型与定义特定问题实例的数据值之间是有区别的。第 5 章至第 8 章重点关注了描述模型所需的集合、参数、变量、目标函数和约束的声明。在本章和下一章中，我们将更详细地研究用于指定数据的语句。

几乎每一章都会出现 AMPL 数据语句的示例。这些语句提供了多种格式，用于列出集合和参数值的列表和表格。某些格式最适合在文本编辑或文字处理环境中创建和维护，而其他格式则易于从数据库系统和电子表格等程序中生成。display 命令（第 12 章）也会以这些格式输出数据。在可能的情况下，集合和参数使用了类似的语法和概念。

本章首先解释如何将 AMPL 的 data 命令与数据语句结合使用，从诸如在我们的示例中文件名以 .dat 结尾的文件中读取数据值。data 命令的选项还允许或强制重新读取选定的集合和参数。

后续章节将描述数据语句，首先针对集合与参数数据的列表形式，然后是表格形式，接着是关于变量初始值、索引集合的值以及默认值的简要说明。数据语句格式的摘要见 A.12 节。

最后一节将介绍 read 命令，该命令用于将未格式化的值列表读入集合与参数中。第 10 章将专门介绍 AMPL 处理存储在关系数据库表中的数据的功能。

# 9.1 格式化数据：data 命令

诸如 param 和 var 这样的声明，以及 solve 和 display 这样的命令，都是在模型模式 (model mode) 下执行的，这是大多数建模活动的标准模式。然而，模型模式在读取大量集合与参数值列表时并不方便。因此，AMPL 在数据模式 (data mode) 下读取其数据语句，该模式由 data 命令启动。在最常见的用法中，此命令由关键字 data 后跟一个文件名组成。例如，

# ampl: data diet.dat;

将从名为 diet.dat 的文件中读取数据。包含空格、分号或不可打印字符的文件名必须用引号括起来。

在数据模式下读取时，AMPL 将任何空白字符序列（包括空格、制表符和换行符）视为单个空格。分隔字符串或数字的逗号也会被忽略。合理使用这些分隔符有助于将数据排列成易于阅读的列表和表格；我们的示例结合使用了空格和换行符。但如果数据语句是由其他数据管理软件生成并直接发送给 AMPL 的，则可以忽略视觉外观，采用任何方便的格式。

数据文件通常包含许多字符字符串，表示集合成员或符号参数的值。因此，在数据模式下，AMPL 通常不要求字符串用引号括起来。然而，如果字符串中包含字母、数字、下划线、句点、加号 (+) 和减号 (-) 之外的任何字符，则必须用引号括起来，例如 A&P。可以使用一对单引号 ($^{\prime}\mathbb{A}\& \mathbb{P}^{\prime}$) 或双引号 ($^{\prime \prime}\mathbb{A}\& \mathbb{P}^{\prime \prime}$)，除非字符串本身包含引号，此时必须使用另一种引号将其包围（如 "DOMINICK'S"），或者将包围引号在字符串内重复（如 'DOMINICK''S'）。

看起来像数字的字符串（例如 $" + 1"$ 或 $"3\in 4"$）也必须加引号，以便将其与实际为数字的集合成员或参数值区分开来。具有相同内部表示的数字被视为相同，例如 2、2.00、2. e0 和 $0.02\mathrm{E} + 2$ 都表示相同的集合成员。

当 AMPL 完成读取数据模式下的文件时，它通常会恢复到执行 data 命令之前的模式。因此，一个数据文件本身可以包含从其他文件读取数据的 data 命令。但如果数据文件中的最后一个 data 语句缺少结尾的分号，则无论之前的模式是什么，都将保持在数据模式。

不带文件名的 data 命令会使 AMPL 进入数据模式，因此后续的输入将被视为数据语句：

ampl: model dietu.mod;  
ampl: data;  
ampl data: set MINREQ := A B1 B2 C CAL;  
ampl data: set MAXREQ := A MA CAL;  
ampl data: display NUTR;  
set NUTR := A B1 B2 C CAL NA;  
ampl:

当 AMPL 遇到任何不以数据语句开头关键字（如 set 或 param）开始的语句（如 display）时，将退出数据模式。model 命令（无论是否带文件名）也会导致返回模型模式。

模型组件可以通过多个 data 命令从任意数量的数据文件中赋值。无论文件数量多少，AMPL 都会检查是否任何组件被多次赋值，并将重复赋值标记为错误。然而，在某些情况下，通过发出新的 data 语句来更改数据是方便的；例如，在为模型的一个场景求解后，您可能希望通过读取对应于第二个场景的新数据文件来修改某些数据。新文件中的数据值通常会被视为错误的重复，但您可以通过首先给出 reset data 或 update data 命令告诉 AMPL 接受它们。这些替代方法在第 11.3 节中描述，同时还介绍了使用 reset data 对随机计算的参数进行重新采样，以及使用 let 直接赋值新的集合或参数值。

# 9.2 列表中的数据

对于一个无索引（标量）参数，数据语句会赋一个值：

param avail := 40;

然而，典型模型的大多数参数是基于集合索引的，其值在本节和下一节中介绍的各种列表和表格中指定。

我们从简单的一维对象集合和基于它们索引的一维参数集合开始。然后转向二维集合和参数，对于这些参数，我们有额外的选项将数据组织成“切片”。接着展示二维选项如何轻松推广到更高维度，并提供一些三维示例。最后，我们展示如何将集合及其索引参数的数据语句结合起来，以提供更简洁和方便的表示形式。

# 一维集合和参数列表

对于基于一维集合索引的参数，如

set PROD;  
param rate {PROD} > 0;

集合的指定可以简单地列出其成员：

set PROD := bands coils plate;

参数的指定几乎可以相同，只是在每个集合成员后添加一个值：

param rate := bands 200 coils 140 plate 160 ;

该参数的定义也可以写成

param rate := bands 200 coils 140 plate 160 ;

因为多余的空格和换行符会被忽略。

如果一个一维集合被声明为 ordered 或 circular 属性 (见第 5.6 节)，那么其成员的顺序将由定义它的数据语句决定。例如，我们在图 5-4 中定义有序集合 WEEKS 的成员时指定了

set WEEKS := 27sep 04oct 11oct 18oct ;

集合中的成员必须互不相同；AMPL 会在出现重复时发出警告：

duplicate member coils for set PROD context: set PROD := bands coils plate coils >>>; <<<

同样，对于一个参数来说，在其索引集合的每个成员上只能被赋值一次。违反此规则将引发类似的错误信息：

rate['bands'] already defined context: param rate := bands 200 bands 160 >>>; <<<

由 >>> 和 <<< 标记的上下文并非错误的确切位置，但该信息足以说明问题。

可以通过在 := 操作符后直接放置分号的方式来将一个集合指定为空集合。对于一个在空集合上索引的参数，不会有关联的数据。

# 二维集合与参数列表

将数据列表扩展到二维情况基本上是直接的，但每个集合成员需要用一对对象表示。例如，考虑图 6-2a 中的以下集合：

set ORIG; # 起始地
set DEST; # 目的地
set LINKS within {ORIG,DEST}; # 运输链接集合

ORIG 和 DEST 的成员可以像任何一维集合一样指定：

set ORIG := GARY CLEV PITT ;
set DEST := FRA DET LAN WIN STL FRE LAF ;

然后 LINKS 的成员可以指定为元组列表，就像你在模型的索引表达式中看到的那样，

set LINKS := (GARY,DET) (GARY,LAN) (GARY,STL) (GARY, LAF) (CLEV,FRA) (CLEV,DET) (CLEV,LAN) (CLEV,WIN) (CLEV,STL) (CLEV, LAF) (PITT,FRA) (PITT,WIN) (PITT,STL) (PITT,FRE);

或者作为不带括号和逗号的成对列表：

set LINKS := GARY DET GARY LAN GARY STL GARY LAF CLEV FRA CLEV DET CLEV LAN CLEV WIN CLEV STL CLEV LAF PITT FRA PITT WIN PITT STL PITT FRE ;

每对中的成员顺序是有意义的——第一个必须来自 ORIG，第二个必须来自 DEST——但这些对本身可以以任意顺序出现。

还有一种更简洁的方式来描述这一对集合，即列出与每个第一个分量对应的所有第二个分量：

set LINKS := (GARY,*) DET LAN STL LAF (CLEV,*) FRA DET LAN WIN STL LAF (PITT,*) FRA WIN STL FRE ;

同样也可以列出与每个第二个分量对应的所有第一个分量：

set LINKS := (*,FRA) CLEV PITT (*,DET) GARY CLEV (*,LAN) GARY CLEV (*,WIN) CLEV PITT (*,LAF) GARY CLEV (*,FRE) PITT (*,STL) GARY CLEV PITT ;

诸如此类的表达式 (GARY, *) 或 (*, FRA)，形如一对有序元素但其中某个分量被 * 替代，称为 数据模板 (data template)。每个模板后跟随一个列表，列表中的条目将替代 * 以生成有序对；这些有序对共同构成集合的一个 切片 (slice)，该切片贯穿集合中 * 所在的维度。一个不包含任何 * 的元组，例如 (GARY, DET)，实际上是一个仅指定其自身的模板，因此其后不跟随任何值。而在另一个极端情况下，在仅由有序对构成的表格中，集合 LINKS 的定义如下：

set LINKS := GARY DET GARY LAN GARY STL GARY LAF CLEV FRA CLEV DET CLEV LAN CLEV WIN CLEV STL CLEV LAF PITT FRA PITT WIN PITT STL PITT FRE ;

一个默认模板 (*, *) 适用于所有条目。

对于一个在二维集合上索引的参数，AMPL 的列表格式同样由集合的格式推导而来，只需将参数值置于集合成员之后即可。因此，如果我们有参数 cost 在集合 LINKS 上索引：

param cost {LINKS} >= 0;

那么集合 LINKS 的集合数据语句可扩展为如下 cost 的参数数据语句：

param cost := GARY DET 14 GARY LAN 11 GARY STL 16 GARY LAF 8 CLEV FRA 27 CLEV DET 9 CLEV LAN 12 CLEV WIN 9 CLEV STL 26 CLEV LAF 17 PITT FRA 24 PITT WIN 13 PITT STL 28 PITT FRE 99 ;

通过集合切片构成的列表同样可以扩展，方法是在每个隐含的集合成员后放置一个参数值。因此，对应于我们为 LINKS 编写的简洁数据语句：

set LINKS := (GARY,*) DET LAN STL LAF (CLEV,*) FRA DET LAN WIN STL LAF (PITT,*) FRA WIN STL FRE ;

存在如下 cost 值的语句：

param cost := [GARY,*) DET 14 LAN 11 STL 16 LAF 8 [CLEV,*) FRA 27 DET 9 LAN 12 WIN 9 STL 26 LAF 17 [PITT,*) FRA 24 WIN 13 STL 28 FRE 99 ;

模板用方括号表示，以区别于集合模板所用的圆括号，但其工作方式相同。例如，模板 [GARY,*) 表示接下来的条目将用于 cost 中第一个索引为 GARY 的值，而条目 DET 14 则为 cost["GARY", "DET"] 赋值为 14。

上述所有内容同样适用于在第一个维度上进行切片的模板。例如，你也可以通过以下方式指定参数 cost：

param cost := [* ,FRA] CLEV 27 PITT 24 [* ,DET] GARY 14 CLEV 9 [* ,LAN] GARY 11 CLEV 12 [* ,WIN] CLEV 9 PITT 13 [* ,STL] GARY 16 CLEV 26 PITT 28 [* ,FRE] PITT 99 [* ,LAF] GARY 8 CLEV 17

你甚至可以把如下有序对列表的例子：

param cost := GARY DET 14 GARY LAN 11 GARY STL 16 GARY LAF 8

也看作是这种形式的一个实例，其对应于默认模板 $[*,*]$。

# 高维集合和参数的列表

用于表示二维集合与参数的数据列表概念可以自然地扩展到更高维度的情况。唯一值得注意的区别是，对于高于二维的情况，可以在多个维度上进行非平凡的切片 (slice)。因此，我们在此仅通过一些三维的示例进行说明，随后简要介绍 AMPL 数据列表格式的一般规则，这些规则详见附录 A.12。

我们采用第 6.3 节中的例子，其中我们提出了一个多商品运输模型的一个版本，该模型定义了一个三元组集合 ROUTES 以及在其上索引的成本参数：

``` ampl
set ROUTES within {ORIG, DEST, PROD};
param cost {ROUTES} >= 0;
```

假设 ORIG 和 DEST 如前定义，PROD 仅包含成员 bands 和 coils，而 ROUTES 包含来自集合 {ORIG, DEST, PROD} 的某些三元组。那么 ROUTES 的成员可以通过一个三元组列表最简单地给出，如下所示：

``` ampl
set ROUTES :=
(GARY, LAN, coils) (GARY, STL, coils) (GARY, LAF, coils)
(CLEV, FRA, bands) (CLEV, FRA, coils) (CLEV, DET, bands)
(CLEV, DET, coils) (CLEV, LAN, bands) (CLEV, LAN, coils)
(CLEV, WIN, coils) (CLEV, STL, bands) (CLEV, STL, coils)
(CLEV, LAF, bands)
(PITT, FRA, bands) (PITT, WIN, bands) (PITT, STL, bands)
(PITT, FRE, bands) (PITT, FRE, coils);
```

或者

``` ampl
set ROUTES :=
GARY LAN coils     GARY STL coils     GARY LAF coils
CLEV FRA bands     CLEV FRA coils     CLEV DET bands
CLEV DET coils     CLEV LAN bands     CLEV LAN coils
CLEV WIN coils     CLEV STL bands     CLEV STL coils
CLEV LAF bands
PITT FRA bands     PITT WIN bands     PITT STL bands
PITT FRE bands     PITT FRE coils ;
```

像之前一样使用模板，但每个模板包含三个条目，我们可以通过在每个模板中放置一个 `*` 来将集合的定义分解为沿某一维度的切片。在以下示例中，我们沿第二个维度进行切片：

``` ampl
set ROUTES :=
(CLEV, *, bands)    FRA  DET  LAN  STL  LAF
(PITT, *, bands)    FRA  WIN  STL  FRE
(GARY, *, coils)    LAN  STL  LAF
(CLEV, *, coils)    FRA  DET  LAN  WIN  STL
(PITT, *, coils)    FRE ;
```

由于集合中不包含起始地为 GARY 且产品为 bands 的成员，因此省略了模板 (GARY, *, bands)。

当集合的维度大于二时，也可以同时在多个维度上进行切片。特别地，对两个维度进行切片时，自然地需要在每个模板中放置两个 `*`。下面我们沿第一个和第三个维度进行切片：

``` ampl
set ROUTES :=
(*, FRA, *)    CLEV bands    CLEV coils    PITT bands
(*, DET, *)    CLEV bands    CLEV coils
(*, LAN, *)    GARY coils    CLEV bands    CLEV coils
(*, WIN, *)    CLEV coils    PITT bands
(*, STL, *)    GARY coils    CLEV bands    CLEV coils    PITT bands
(*, FRE, *)    PITT bands    PITT coils
(*, LAF, *)    GARY coils    CLEV bands ;
```

由于这些模板包含两个 `*`，因此其后必须跟随成对的分量，这些分量按从左到右的顺序代入模板以生成集合成员。例如，模板 `(*, FRA, *)` 后跟 `CLEV bands` 表示 `(CLEV, FRA, bands)` 是该集合的一个成员。

以上任何形式也都可以用于给出参数 cost 的值。我们可以写成：

param cost := [CLEV,\*,bands] FRA 27 DET 9 LAN 12 STL 26 LAF 17 [PITT,\*,bands] FRA 24 WIN 13 STL 28 FRE 99 [GARY,\*,coils] LAN 11 STL 16 LAF 8 [CLEV,\*,coils] FRA 23 DET 8 LAN 10 WIN 9 STL 21 [PITT,\*,coils] FRE 81 ;

或者

param cost := [*,*,bands] CLEV FRA 27 CLEV DET 9 CLEV LAN 12 CLEV STL 26 CLEV LAF 17 PITT FRA 24 PITT WIN 13 PITT STL 28 PITT FRE 99 [*,*,coils] GARY LAN 11 GARY STL 16 GARY LAF 8 CLEV FRA 23 CLEV DET 8 CLEV LAN 10 CLEV WIN 9 CLEV STL 21 PITT FRE 81

或者

param cost := CLEV DET bands 9 CLEV DET coils 8 CLEV FRA bands 27 CLEV FRA coils 23 CLEV LAF bands 17 CLEV LAN bands 12 CLEV LAN coils 10 CLEV STL bands 26 CLEV STL coils 21 CLEV WIN coils 9 GARY LAF coils 8 GARY LAN coils 11 GARY STL coils 16 PITT FRA bands 24 PITT FRE bands 99 PITT FRE coils 81 PITT STL bands 28 PITT WIN bands 13 ;

通过在模板中将 * 放在不同位置，我们可以在三种不同方式中进行一维切片，或在三种不同方式中进行二维切片。（模板 [* ,* ,* ] 将指定一个三维列表，如 param cost := CLEV DET bands 9 CLEV DET coils 8 CLEV FRA bands 27

如上所示。）

更一般地，一个用于列表形式的 n 维集合或参数的模板必须有 n 个条目。每个条目要么是一个合法的集合成员，要么是 *。集合的模板用括号括起来（就像集合表达式中的元组），而参数的模板用方括号括起来（就像参数的下标）。模板后面是一系列条目，每个条目由一个对应于每个 * 的集合成员组成，如果是参数模板，则还包含一个参数值。每个条目通过将其集合成员代入模板中的 * 来定义一个 n 元组；该元组要么被添加到指定的集合中，要么将该元组索引的参数赋值为条目中的值。

一个模板适用于其与下一个模板（或数据语句结尾）之间的所有条目。具有不同数量 * 的模板甚至可以在同一个数据语句中一起使用，只要每个参数仅被赋值一次即可。如果没有出现模板，则假定使用一个全部为 *S 的模板。

# 集合和参数的组合列表

当我们为一个集合和一个在其上索引的参数给出数据语句时，例如：

set PROD := bands coils plate ; param rate := bands 200 coils 140 plate 160 ;

我们实际上指定了集合的成员两次。AMPL 允许我们在 param 数据语句中包含集合的名称来避免这种重复：

param: PROD: rate := bands 200 coils 140 plate 160 ;

AMPL 使用此语句同时确定 PROD 的成员资格和 rate 的值。

另一种常见的冗余情况是，当我们需要为多个参数提供数据，而这些参数都基于相同的索引集时，例如 图 1-4a 中的 rate、profit 和 market 都基于 PROD 索引。我们可以不为每个参数分别写数据语句，

```ampl
param rate := bands 200 coils 140 plate 160 ;
param profit := bands 25 coils 30 plate 29 ;
param market := bands 6000 coils 4000 plate 3500 ;
```

而是通过在关键字 param 后列出所有三个参数名称，将这些语句合并成一个：

```ampl
param: rate profit market :=
bands   200  25  6000
coils   140  30  4000
plate   160  29  3500 ;
```

由于 AMPL 会忽略多余的空格和换行符，我们可以选择将这些信息重新排列成更易于阅读的表格形式：

<table>
<tr><td>param:</td><td>rate</td><td>profit</td><td>market :=</td></tr>
<tr><td>bands</td><td>200</td><td>25</td><td>6000</td></tr>
<tr><td>coils</td><td>140</td><td>30</td><td>4000</td></tr>
<tr><td>plate</td><td>160</td><td>29</td><td>3500 ;</td></tr>
</table>

无论采用哪种方式，我们仍可以选择在语句中添加索引集的名称，从而将集合和所有三个参数的规格合并在一起。

<table>
<tr><td>param: PROD:</td><td>rate</td><td>profit</td><td>market :=</td></tr>
<tr><td>bands</td><td>200</td><td>25</td><td>6000</td></tr>
<tr><td>coils</td><td>140</td><td>30</td><td>4000</td></tr>
<tr><td>plate</td><td>160</td><td>29</td><td>3500 ;</td></tr>
</table>

同样的规则也适用于任何更高维度的集合列表以及基于它们索引的参数。因此，对于我们的二维示例 LINKS，我们可以写成

```ampl
param: LINKS: cost :=
GARY DET 14   GARY LAN 11   GARY STL 16   GARY LAF 8
CLEV FRA 27   CLEV DET 9   CLEV LAN 12   CLEV WIN 9   CLEV STL 26   CLEV LAF 17
PITT FRA 24   PITT WIN 13   PITT STL 28   PITT FRE 99;
```

来指定 LINKS 的成员以及在其上索引的参数 cost 的值，或者

```ampl
param: LINKS: cost limit :=
GARY DET 14 1000   GARY LAN 11 800    GARY STL 16 1200   GARY LAF 8 1100
CLEV FRA 27 1200   CLEV DET 9 600     CLEV LAN 12 900    CLEV WIN 9 950
CLEV STL 26 1000   CLEV LAF 17 800
PITT FRA 24 1500   PITT WIN 13 1400   PITT STL 28 1500   PITT FRE 99 1200;
```

来同时指定 cost 和 limit 的值。在使用模板时，同样适用这些选项，从而可以进一步选择如下形式：

```ampl
param: LINKS: cost :=
[GARY,*] DET 14   LAN 11   STL 16   LAF 8
[CLEV,*] FRA 27   DET 9    LAN 12   WIN 9   STL 26   LAF 17
[PITT,*] FRA 24   WIN 13   STL 28   FRE 99;
```

以及

```ampl
param: LINKS: cost limit :=
[GARY,*] DET 14   LAN 11 800   STL 16 1200   LAF 8 1100
[CLEV,*] FRA 27 1200   DET 9 600   LAN 12 900   WIN 9 950
STL 26 1000   LAF 17 800
[PITT,*] FRA 24 1500   WIN 13 1400   STL 28 1500   FRE 99 1200;
```

在这里，索引集的成员与两个参数一起指定；例如，模板 [GARY, *] 后跟集合成员 DET 以及值 14 和 1000，表示 (GARY, DET) 应被添加到集合 LINKS 中，cost[GARY, DET] 的值为 14，limit[GARY, DET] 的值为 1000。

正如我们的示例所示，为多个参数或一个集合和参数提供值的 param 语句的关键在于第一行，该行由 param 后跟一个冒号组成，然后是可选的索引集名称后跟一个冒号，最后是由 $\coloneqq$ 赋值运算符终止的参数名称列表。列表中的每个后续项都包含与最近模板中 $\star_{\mathrm{S}}$ 数量相等的集合成员数，以及与第一行列出的参数数量相等的参数值数。

通常，param 语句第一行列出的参数都基于相同的集合进行索引。然而，情况不一定如此，如图 5-1 所示的饮食模型变体。对于该模型，营养限制由以下语句给出：

set MINREQ; set MAXREQ; param n_min {MINREQ} >= 0; param n_max {MAXREQ} >= 0;

因此，n_min 和 n_max 分别基于可能重叠但不太可能是相同营养素集合 (nutrient set) 进行索引。

该模型的示例数据指定为：

set MINREQ := A B1 B2 C CAL; set MAXREQ := A NA CAL; param: n_min n_max := A 700 20000 C 700 B1 0 B2 0 NA 50000 CAL 16000 24000;

每个句点或点 (.) 表示 AMPL 对应参数和索引没有给定值。例如，由于 MINREQ 不包含成员 NA，因此参数 n_min[NA] 未定义；因此在数据语句中为 NA 和 n_min 提供了一个 . 作为条目。我们不能简单地为此条目留空，因为 AMPL 会将其视为 50000：数据模式处理会忽略所有多余的空格。也不应在该条目中填入零；否则我们将收到类似以下错误消息：

error processing param n_min: invalid subscript n_min['NA'] discarded.

当 AMPL 首次尝试访问 n_min 时，通常是在第一次求解时。

当我们在 param 语句的第一行命名一个集合时，该集合必须尚未具有值。如果图 5-1 中的参数数据规范被写为：

<table><tr><td>param:</td><td>NUTR:</td><td>n_min</td><td>n_max :=</td></tr><tr><td>A</td><td>700</td><td></td><td></td></tr><tr><td>C</td><td>700</td><td></td><td>.</td></tr><tr><td>B1</td><td>0</td><td></td><td>.</td></tr><tr><td>B2</td><td>0</td><td></td><td>.</td></tr><tr><td>NA</td><td>.</td><td>50000</td><td></td></tr><tr><td>CAL</td><td>16000</td><td>24000 ;</td><td></td></tr></table>

则 AMPL 将生成错误消息：

dietu.dat, line 16 (offset 366): NUTR was defined in the model context: param: NUTR >>> : <<< n_min n_max :=

因为模型中对 NUTR 的声明：

set NUTR = MINREQ union MAXREQ;

已经将其定义为 MINREQ 和 MAXREQ 的并集 (union)。

# 9.3 表格中的数据

数据的表格格式，其索引沿左侧和顶部边缘排列，值对应于索引对，可能比上一节中描述的列表格式更简洁或更易于阅读。这里我们首先描述二维参数的表格，然后描述来自更高维参数的切片。我们还展示了如何在具有 $^+$ 或 - 条目的表格中指定相应的多维集合 (multidimensional set)，而不是参数值条目。

AMPL 还支持表格格式的便捷扩展，其中沿左侧和顶部边缘可以出现两个以上的索引。指定此类表格的规则在本节末尾附近提供。

# 二维表格

对于在两个集合上索引的参数的数据值，例如来自图 3-1a 的运输模型的运输成本 (transport cost) 数据：

set ORIG; set DEST; param cost {ORIG,DEST} >= 0;

很自然地在表格中指定（图 3-1b）：

<table><tr><td>param cost:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>: =</td></tr><tr><td>GARY</td><td>39</td><td>14</td><td>11</td><td>14</td><td>16</td><td>82</td><td>8</td><td></td></tr><tr><td>CLEV</td><td>27</td><td>9</td><td>12</td><td>9</td><td>26</td><td>95</td><td>17</td><td></td></tr><tr><td>PITT</td><td>24</td><td>14</td><td>17</td><td>13</td><td>28</td><td>99</td><td>20</td><td>;</td></tr></table>

行标签给出第一个索引，列标签给出第二个索引，因此例如 cost["GARY","FRA"] 被设置为 39。为了使 AMPL 能够识别这是表格，参数名称后必须跟一个冒号，而 $\coloneqq$ 运算符跟在列标签列表之后。

对于较大的索引集合，表格的列变得无法在单个屏幕或页面的宽度内查看。为了处理这种情况，AMPL 提供了几种替代方案，我们在上面的小表格上进行说明。

当只有一个索引集合过大时，可以转置表格，使列标签对应于较小的集合：

<table><tr><td colspan="4">param cost (tr):</td></tr><tr><td></td><td>GARY</td><td>CLEV</td><td>PITT</td></tr><tr><td>FRA</td><td>39</td><td>27</td><td>24</td></tr><tr><td>DET</td><td>14</td><td>9</td><td>14</td></tr><tr><td>LAN</td><td>11</td><td>12</td><td>17</td></tr><tr><td>WIN</td><td>14</td><td>9</td><td>13</td></tr><tr><td>STL</td><td>16</td><td>26</td><td>28</td></tr><tr><td>FRE</td><td>82</td><td>95</td><td>99</td></tr><tr><td>LAF</td><td>8</td><td>17</td><td>20</td></tr></table>

参数名称后的 (tr) 表示转置表格，其中列标签给出第一个索引，行标签给出第二个索引。当两个索引集合都很大时，可以以某种方式分割表格或其转置。由于换行符被忽略，每行可以跨多行分割：

<table><tr><td rowspan="2">参数 cost:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td></tr><tr><td>STL</td><td>FRE</td><td>LAF</td><td>:=</td></tr><tr><td rowspan="2">GARY</td><td>39</td><td>14</td><td>11</td><td>14</td></tr><tr><td>16</td><td>82</td><td>8</td><td></td></tr><tr><td rowspan="2">CLEV</td><td>27</td><td>9</td><td>12</td><td>9</td></tr><tr><td>26</td><td>95</td><td>17</td><td></td></tr><tr><td rowspan="2">PITT</td><td>24</td><td>14</td><td>17</td><td>13</td></tr><tr><td>28</td><td>99</td><td>20</td><td>;</td></tr></table>

或者表格可以按列分割为几个较小的表格：

<table><tr><td>参数 cost:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>:=</td></tr><tr><td>GARY</td><td>39</td><td>14</td><td>11</td><td>14</td><td></td></tr><tr><td>CLEV</td><td>27</td><td>9</td><td>12</td><td>9</td><td></td></tr><tr><td>PITT</td><td>24</td><td>14</td><td>17</td><td>13</td><td></td></tr><tr><td></td><td>STL</td><td>FRE</td><td>LAF</td><td>:=</td><td></td></tr><tr><td>GARY</td><td>16</td><td>82</td><td>8</td><td></td><td></td></tr><tr><td>CLEV</td><td>26</td><td>95</td><td>17</td><td></td><td></td></tr><tr><td>PITT</td><td>28</td><td>99</td><td>20</td><td>;</td><td></td></tr></table>

冒号表示每个新子表格的开始；在本例中，每个子表格具有相同的行标签，但列标签的子集不同。

在图 6-2a 中所示的此模型的替代公式中，cost 并非对 ORIG 和 DEST 成员的所有组合进行索引，而是对这些集合中成对子集进行索引：

set LINKS within {ORIG,DEST}; param cost {LINKS} >= 0;

正如我们在第 9.2 节中所见，LINKS 的成员可以通过成对列表简洁地给出：

set LINKS := (GARY,*) DET LAN STL LAF (CLEV,*) FRA DET LAN WIN STL LAF (PITT,*) FRA WIN STL FRE;

cost 的值可以像这样以表格形式给出，而不是以类似列表的形式给出：

param cost: FRA DET LAN WIN STL FRE LAF := GARY 14 11 16 8 CLEV 27 9 12 9 26 17 PITT 24 13 28 99

对于 LINKS 中存在的所有成对组合，都会给出一个 cost 值，而对于不在 LINKS 中的成对组合，则使用点 (.) 作为占位符。点可以在任何 AMPL 表格中出现，以表示“此处未指定值”。

集合 LINKS 本身也可以通过类似于 cost 的表格给出：

set LINKS: FRA DET LAN WIN STL FRE LAF := GARY - + + - + + + CLEV + + + + + + + PITT + - - + + + - ;

符号 + 表示属于该集合的成对组合，而符号 - 表示不属于该集合的成对组合。任何 AMPL 的参数表格格式都可以用于集合。

# 高维数据的二维切片

为了提供超过二维的参数数据，我们可以在表格中指定二维切片中的值。使用切片的规则与使用列表的规则大致相同。例如，再次考虑由以下方式定义的三维参数 cost：

定义 ROUTES 为 {ORIG, DEST, PROD} 的子集；参数 cost {ROUTES} >= 0；

在上一节中，我们以列表形式指定的该参数值：

param cost :=

[*,*,bands] CLEV FRA 27 CLEV DET 9 CLEV LAN 12 CLEV STL 26 CLEV LAF 17 PITT FRA 24 PITT WIN 13 PITT STL 28 PITT FRE 99 [*,*,coils] GARY LAN 11 GARY STL 16 GARY LAF 8 CLEV FRA 23 CLEV DET 8 CLEV LAN 10 CLEV WIN 9 CLEV STL 21 PITT FRE 81

也可以用表格形式表示为：由于我们处理的是二维表格，模板中必须有两个 *。表格值的行标签替换第一个 *，列标签替换第二个 *，除非在模板后通过 $(\pm \tau)$ 指定了相反的顺序。你可以省略那些没有重要条目的行或列，例如上面 $[*,*,coils]$ 表格中的 GARY 行。

<table><tr><td colspan="9">param cost :=</td></tr><tr><td>[*,*,bands]:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>:=</td></tr><tr><td>CLEV</td><td>27</td><td>9</td><td>12</td><td>.</td><td>26</td><td>.</td><td>17</td><td></td></tr><tr><td>PITT</td><td>24</td><td>.</td><td>.</td><td>13</td><td>28</td><td>99</td><td>.</td><td></td></tr><tr><td>[*,*,coils]:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>:=</td></tr><tr><td>GARY</td><td>.</td><td>.</td><td>11</td><td>.</td><td>16</td><td>.</td><td>8</td><td></td></tr><tr><td>CLEV</td><td>23</td><td>8</td><td>10</td><td>9</td><td>21</td><td>.</td><td>.</td><td></td></tr><tr><td>PITT</td><td>.</td><td>.</td><td>.</td><td>.</td><td>.</td><td>81</td><td>.</td><td>;</td></tr></table>

与之前一样，表格中任何切片中的点表示该元组不属于表格。

可以通过在每个数字出现的位置放置一个 $^+$ 来构造指定集合 ROUTES 的类似表格：

set ROUTES :=

$(\star ,\star ,\mathtt{bands})$ : FRA DET LAN WIN STL FRE LAF := CLEV + + + - + + PITT + - - + + + - $(\star ,\star ,\mathtt{coils})$ : FRA DET LAN WIN STL FRE LAF := GARY - - + - + + + CLEV + + + + + + - PITT - - - - + - 

由于模板现在是集合模板而不是参数模板，它们用括号而不是方括号括起来。

# 高维表格

通过在每行左侧或每列顶部放置多个索引，你可以在单个表格中描述多维数据，而无需一系列切片。我们将继续使用三维的成本数据来说明各种可能性。

通过将前两个索引（来自集合 ORIG 和 DEST）放在左侧，第三个索引（来自集合 PROD）放在顶部，我们生成了以下成本的三维表格：

<table><tr><td colspan="3">参数 cost: bands coils :=</td></tr><tr><td>CLEV FRA</td><td>27</td><td>23</td></tr><tr><td>CLEV DET</td><td>8</td><td>8</td></tr><tr><td>CLEV LAN</td><td>12</td><td>10</td></tr><tr><td>CLEV WIN</td><td>.</td><td>9</td></tr><tr><td>CLEV STL</td><td>26</td><td>21</td></tr><tr><td>CLEV LAF</td><td>17</td><td>.</td></tr><tr><td>PITT FRA</td><td>24</td><td>.</td></tr><tr><td>PITT WIN</td><td>13</td><td>.</td></tr><tr><td>PITT STL</td><td>28</td><td>.</td></tr><tr><td>PITT FRE</td><td>99</td><td>81</td></tr><tr><td>GARY LAN</td><td>.</td><td>11</td></tr><tr><td>GARY STL</td><td>.</td><td>16</td></tr><tr><td>GARY LAF</td><td>.</td><td>8 ;</td></tr></table>

将第一个索引放在左侧，第二个和第三个索引放在顶部，我们得到以下表格，为了方便起见，将其分为两部分：

<table><tr><td>参数 cost:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td></tr><tr><td></td><td>: bands</td><td>bands</td><td>bands</td><td>bands</td><td>bands</td><td>bands</td><td>bands</td></tr><tr><td></td><td>CLEV</td><td>27</td><td>9</td><td>12</td><td>.</td><td>26</td><td>.</td></tr><tr><td></td><td>PITT</td><td>24</td><td>.</td><td>.</td><td>13</td><td>28</td><td>99</td></tr><tr><td></td><td>: FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td></tr><tr><td></td><td>: coils</td><td>coils</td><td>coils</td><td>coils</td><td>coils</td><td>coils</td><td>coils</td></tr><tr><td></td><td>GARY</td><td>.</td><td>.</td><td>11</td><td>.</td><td>16</td><td>.</td></tr><tr><td></td><td>CLEV</td><td>23</td><td>8</td><td>10</td><td>9</td><td>21</td><td>.</td></tr><tr><td></td><td>PITT</td><td>.</td><td>.</td><td>.</td><td>.</td><td>81</td><td>. ;</td></tr></table>

通常，每个表格标题行前必须有冒号，而 $\mathbf{\dot{\rho}} = \mathbf{\dot{\rho}}$ 只放在最后一个标题行之后。

如果未给出相反的指示，则按索引出现的顺序取索引，首先从左侧取，然后从顶部取。与其他表格一样，您可以添加指示符 (tr) 来转置表格，这样索引仍按顺序取，但首先从顶部取，然后从左侧取：

<table><tr><td>param cost (tr):</td><td>CLEV</td><td>CLEV</td><td>CLEV</td><td>CLEV</td><td>CLEV</td><td>CLEV</td><td></td><td></td></tr><tr><td></td><td>:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>LAF</td><td></td></tr><tr><td></td><td>bands</td><td>27</td><td>8</td><td>12</td><td>.</td><td>26</td><td>17</td><td></td></tr><tr><td></td><td>coils</td><td>23</td><td>8</td><td>10</td><td>9</td><td>21</td><td>.</td><td></td></tr><tr><td></td><td>:</td><td>PITT</td><td>PITT</td><td>PITT</td><td>GARY</td><td>GARY</td><td>GARY</td><td></td></tr><tr><td></td><td>:</td><td>FRA</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAN</td><td>STL</td><td>LAF</td></tr><tr><td></td><td>bands</td><td>24</td><td>13</td><td>28</td><td>99</td><td>.</td><td>.</td><td>.</td></tr><tr><td></td><td>coils</td><td>.</td><td>.</td><td>.</td><td>81</td><td>11</td><td>16</td><td>8 ;</td></tr></table>

模板也可用于更精确地指定内容的位置。对于多维表格，模板中有两个符号，\* 表示出现在左侧的索引，: 表示出现在顶部的索引。例如，模板 $[\ast ,\cdot ,\ast ]$ 给出的表示形式中，第一个和第三个索引在左侧，第二个索引在顶部：

<table><tr><td>param cost :=</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>[*,*,*]:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>: =</td></tr><tr><td>CLEV bands</td><td>27</td><td>9</td><td>12</td><td>.</td><td>26</td><td>.</td><td>17</td><td></td></tr><tr><td>CLEV coils</td><td>23</td><td>8</td><td>10</td><td>9</td><td>21</td><td>.</td><td>.</td><td></td></tr><tr><td>PITT bands</td><td>24</td><td>.</td><td>.</td><td>13</td><td>28</td><td>99</td><td>.</td><td></td></tr><tr><td>PITT coils</td><td>.</td><td>.</td><td>.</td><td>.</td><td>.</td><td>81</td><td>.</td><td></td></tr><tr><td>GARY coils</td><td>.</td><td>.</td><td>11</td><td>.</td><td>16</td><td>.</td><td>8</td><td>;</td></tr></table>

此类表格中索引的顺序总是保持不变。无论采用何种转置或模板，第三个索引永远不会正确地出现在第一个索引之前。

对于四维或更高维度的参数，切片和多维表格的概念可以结合使用，从而提供特别广泛的表格格式选择。例如，如果 cost 的索引为 ORIG、DEST、PROD 和 1..T，那么可以使用模板 $[\texttt{*},: ,\text{bands},\*]$ 和 $[\texttt{*},: ,\text{coils},\*]$ 来指定通过第三个索引的两个切片，每个切片由一个多维表格指定，其中两个索引在左侧，一个在顶部。

# 格式选择

切片的排列方式用于表示多维数据，但这不会影响数据值在模型中的使用方式，因此可以选择最方便的格式。对于上面提到的成本参数，可能沿着第三个维度进行切片会更直观，这样每个产品的运输成本数据可以组织成一个单独的表格。或者，将所有的起始地-产品对放在左侧可以得到一种特别简洁的表示方式。再举一个例子，考虑来自图 6-3 的收入参数：

```ampl
set PROD; # 产品 (products)
set AREA {PROD}; # 每个产品的市场区域 (market areas for each product)
param T > 0; # 周数 (number of weeks)
param revenue {p in PROD, AREA[p], 1..T} >= 0;
```

由于索引集 `AREA[p]` 对于每个产品 p 可能不同，因此沿第一个维度（即 `PROD`）进行切片是最具吸引力的。在图 6-4 所示的示例数据中，它们看起来如下所示：

```ampl
param T := 4;

set PROD := bands coils;

set AREA[bands] := east north;
set AREA[coils] := east west export;

param revenue :=
[bands,*,*]:
     1    2    3    4 :=
east 25.0 26.0 27.0 27.0
north 26.5 27.5 28.0 28.5

[coils,*,*]:
     1   2   3   4 :=
east 30  35  37  39
west 29  32  33  35
export 25 25 25 28;
```

我们为每个产品 p 提供了一个单独的收入表，其中行由 `AREA[p]` 中的市场区域标记，列由 `1..T` 中的周数标记。

# 9.4 数据语句的其他特性

AMPL 数据格式提供了额外的功能来处理特殊情况。这里我们描述那些用于为参数指定默认值、定义索引集合中各个集合成员关系以及为变量分配初始值的数据语句。

# 默认值

数据语句必须恰好为你模型中的所有参数提供值。如果你为一个不存在的参数提供了值，将会收到错误信息：

```
error processing param cost: invalid subscript cost['PITT','DET','coils'] discarded.
```

或者，如果你未能为一个确实存在的参数提供值，也会收到错误信息：

```
error processing objective Total Cost: no value for cost['CLEV','LAN','coils']
```

该错误信息通常在你输入 `solve` 后，当 AMPL 首次尝试使用该参数时出现。

如果某个值在数据语句中会多次出现，你可以通过包含一个默认短语来避免重复指定它，该短语会在未显式给出值时提供默认值。例如，假设上面的参数 `cost` 是针对所有可能的三元组进行索引的：

```ampl
set ORIG;
set DEST;
set PROD;
param cost {ORIG, DEST, PROD} >= 0;
```

但对一些不应被使用的路线分配了非常高的成本。这可以通过以下方式表达：

```ampl
param cost default 9999 :=
[*,*,bands]:
          FRA DET LAN WIN STL FRE LAF :=
CLEV      27   9  12   .  26   .  17
PITT      24   .  13  28   99   .   .

[*,*,coils]:
          FRA DET LAN WIN STL FRE LAF :=
GARY       .  11   .  16   .   8   .
CLEV      23   8  10   9  21   .   .
PITT       .   .   .   .  81   .   . ;
```

像 cost["GARY","FRA","bands"] 这样缺失的参数，以及那些通过使用点号明确标记为 "omitted" 的参数（如 cost["GARY","FRA","coils"]），都会被赋予值 9999。总共，有 24 个值为 9999 被分配。

当您希望索引集合的所有参数都被赋予相同的值时，默认特性尤其有用。例如，在图 3-2 中，我们通过将所有供应量和需求量设置为 1，将运输模型应用于指派问题。模型声明如下：

param supply {ORIG} >= 0; param demand {DEST} >= 0;

但在数据中，我们只给出了默认值：

param supply default 1; param demand default 1;

由于没有指定其他值，默认值 1 会自动分配给 supply 和 demand 的每个元素。

如第 7 章所述，模型中的参数声明可以包含默认表达式。这提供了一种指定单一默认值的替代方式：

param cost {ORIG,DEST,PROD} >= 0, default 9999;

然而，如果您只是想避免在数据文件中存储大量 9999，那么最好将默认短语放在数据语句中。只有当您希望默认值以某种方式依赖于其他数据时，才应将默认短语放在模型中。例如，可以通过指定以下内容为每个产品赋予不同的任意大的成本：

param huge_cost {PROD} > 0; param cost {ORIG, DEST, p in PROD} >= 0, default huge_cost[p];

关于默认值与参数语句中 = 短语的关系的讨论，请参见第 7.5 节。

# 索引集合

对于索引集合 (indexed collection)，单独的数据语句指定集合中每个集合的成员。例如，在图 6-3 的示例中，名为 AREA 的集合由集合 PROD 索引：

set PROD; # 产品  
set AREA {PROD}; # 每个产品的市场区域  

这些集合的成员关系在图 6-4 中通过以下方式给出：

set PROD := bands coils;  
set AREA[bands] := east north;  
set AREA[coils] := east west export;  

您可以对索引集合使用任何集合的数据语句格式。唯一的区别是关键字 set 后面的集合名称带有下标。

对于其他集合，您可以通过给出空元素列表来指定索引集合的一个或多个成员为空。如果您只想为索引集合中非空成员提供数据语句，请在模型中将空集定义为默认值：

set AREA {PROD} default {};  

否则，您将收到关于未提供数据语句的任何集合的警告。

# 变量的初始值

您可以选择使用为参数赋值的任何选项为模型的变量分配初始值。变量的名称代表其值，而约束的名称代表相关对偶变量 (dual variable) 的值。（有关对偶变量的简短解释，请参见第 12.5 节。）

任何参数数据语句都可以为变量指定初始值。变量或约束的名称可以直接代替参数名称，使用本章前几节所描述的任意格式。为了帮助明确意图，可以在数据语句开头使用关键字 var 来代替 param。例如，下表为图 3-1a 中的变量 Trans 提供了初始值：

<table><tr><td>var Trans:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>: =</td></tr><tr><td>GARY</td><td>100</td><td>100</td><td>800</td><td>100</td><td>100</td><td>500</td><td>200</td><td></td></tr><tr><td>CLEV</td><td>900</td><td>100</td><td>100</td><td>500</td><td>500</td><td>200</td><td>200</td><td></td></tr><tr><td>PITT</td><td>100</td><td>900</td><td>100</td><td>500</td><td>100</td><td>900</td><td>200</td><td>;</td></tr></table>

另一个例子，在图 1-4 的模型中，一个表格可以同时为参数 rate、profit 和 market 赋值，并为变量 Make 提供初始值：

<table><tr><td>param:</td><td>rate</td><td>profit</td><td>market</td><td>Make</td><td>: =</td></tr><tr><td>bands</td><td>200</td><td>25</td><td>6000</td><td>3000</td><td></td></tr><tr><td>coils</td><td>140</td><td>30</td><td>4000</td><td>2500</td><td></td></tr><tr><td>plate</td><td>160</td><td>29</td><td>3500</td><td>1500</td><td>;</td></tr></table>

所有前面描述的关于默认值的功能同样适用于变量。

在输入 solve 命令之前，可以使用第 12.1 至 12.4 节中描述的 display、print 或 printf 命令查看变量的初始值（以及包含这些初始值的表达式的值）。初始值也可以选择性地传递给求解器 (solver)，如第 14.1 节和 A.18.1 节所述。在返回解之后，变量将不再具有其初始值，但即使如此，您仍可以通过在变量名称后添加适当的后缀来引用初始值，如 A.11 节所示。

初始值最常见的用途是为非线性优化问题提供良好的初始猜测值，相关内容将在第 18 章中讨论。

# 9.5 读取未格式化数据：read 命令

read 命令提供了一种特别简单的方法将值导入 AMPL，前提是您需要的值以固定顺序列在一个文件中。该文件必须是未格式化的，也就是说它只包含要读取的值——不包含集合或参数名称，也不包含冒号或 $\coloneqq$ 运算符。

在最简单的形式中，read 指定一个参数列表和一个从中读取值的文件。文件中的值按照它们出现的顺序被赋给列表中的条目。例如，如果您想从一个名为 week_data.txt 的文件中读取我们简单生产模型（图 4-4）中的周数和每周可用的工时，

param  $\mathrm{T} > 0$  . param avail {1. .T} >= 0;

而该文件的内容为

4 40403240

然后你可以给出命令

read T, avail[1], avail[2], avail[3], avail[4] <week_data.txt;

或者你可以使用索引表达式以更简洁和通用的方式表达相同的内容：

read T, {t in 1..T} avail[t] <week_data.txt;

符号 $<$ 文件名 指定用于读取的文件名。（类似地，$>$ 表示写入文件；参见 A.15。）

通常，read 命令的形式为 read 项目列表 <文件名; 其中项目列表是以逗号分隔的项目列表，每个项目可以是以下任意一种：

参数 {索引 参数 {索引 (项目列表 )

前两种在上面的例子中已经使用，而第三种允许对多个项目应用相同的索引。使用相同的生产示例，要从按参数组织的文件中读取以下参数的值：

param procost {PROD} >= 0;  
param invcost {PROD} >= 0;  
param revenue {PROD,1..T} >= 0;

你可以分别读取每个参数：

read {p in PROD} procost[p] < cost_data;  
read {p in PROD} invcost[p] < cost_data;  
read {p in PROD, t in 1..T} revenue[p,t] < cost_data;

从文件 cost_data 中依次读取所有生产成本，然后是所有库存成本，最后是所有收入。

如果数据是按产品组织的，你可以这样写：

read {p in PROD} (procost[p], invcost[p], {t in 1..T} revenue[p,t]) <cost_data;

这样会依次读取第一个产品的生产成本、库存成本和各周收入，然后是第二个产品，依此类推。

括号中的项目列表本身也可以包含括号中的项目列表。因此，如果你想同时从同一文件中读取参数 param market {PROD,1..T} >= 0;

你可以这样写：

read {p in PROD} (procost[p], invcost[p], {t in 1..T} (revenue[p,t], market[p,t])) <cost_data;

在这种情况下，对于每种产品，你会像以前一样读取两种成本，然后对于每一周读取该产品的收入和市场需求。

正如我们所描述的，read 语句的项目列表形式取决于数据值在文件中的排列顺序。当你读取像 PROD 这样本身没有内在顺序的字符串集合索引数据时，读取值的顺序就是 AMPL 内部表示它们的顺序。如果集合成员直接来自集合数据语句，那么排序将与数据语句中的顺序相同。否则，最好在模型的集合声明中添加 ordered 或 ordered by 短语，以确保排序始终符合你的预期；有关有序集合的更多信息，请参见第 5.6 节。

另一种避免了解集合成员顺序的方法是在读取的文件中显式指定它们。例如，考虑如何使用 read 语句而不是数据语句来获取第 9.4 节中定义的成本参数值，该参数定义为 param cost {ORIG, DEST, PROD} >= 0, default 9999;

你可以如下设置 read 语句：

param ntriples integer; param ic symbolic in ORIG; param jc symbolic in DEST; param kc symbolic in PROD; read ntriples, {1..ntriples} (ic, jc, kc, cost[ic, jc, kc]) < cost_data;

对应的文件 cost_data 必须以类似以下内容开头：

18 CLEV FRA bands 27 PITT FRA bands 24 CLEV FRA coils 23

还需 15 个条目以提供第 9.4 节示例中所示的全部 18 个数据值。

用于 read 命令的文件中的字符串，如果包含字母、数字、下划线、句点、$^+$ 和 - 之外的任何字符，则必须加引号，就像在数据模式中一样。然而，read 语句本身是在模型模式下解释的，因此如果语句引用了任何特定字符串，例如 read {t in 1..T} revenue["bands",t]; 则该字符串必须加引号。跟在 $<$ 后面的文件名除非包含空格、分号或不可打印字符，否则不需要加引号。

如果 read 语句中没有 $<$ 文件名，则从当前输入流中读取值。因此，如果您在 AMPL 提示符下输入了 read 命令，则可以在后续提示符下输入值，直到所有列出的项都被赋值。例如：

ampl: read T, {t in 1..T} avail[t]; ampl? 4 ampl? 40 40 32 40 ampl: display avail; avail [*]:= 1 40 2 40 3 32 4 40 当所有需要的输入都被读取后，提示符从 ampl? 变回 ampl:。

文件名 "-"（字面意义上的减号）被视为 AMPL 进程的标准输入；这在交互式提供输入时非常有用。

在 AMPL 脚本中进一步使用 read 命令，直接从脚本文件中读取值或在命令行提示用户输入值的内容将在第 13 章中描述。

我们所有的示例都假设底层集合（如 ORIG 和 PROD）已经通过本章前面描述的数据语句或其他方式（如数据库访问或将在后续章节中描述的赋值）被赋予了值。因此，read 语句通常是对其他输入命令的补充而不是替代。它在处理由 AMPL 之外的程序为某些参数生成的长数据文件时特别有用。

# 练习

练习 9-1. 第 9.2 节给出了三维集合 ROUTES 的各种数据语句。按如下方式为该集合构建一些其他替代方案：

(a) 使用形如 (CLEV, FRA, *) 的模板。  
(b) 使用形如 $(\star,\star,\mathsf{bands})$ 的模板，并使用列表格式。  
(c) 使用形如 $(\mathrm{CLEV},\star,\star)$ 的模板，并使用表格格式。  
(d) 使用带有一个 * 的模板指定集合的一些成员，使用带有两个 * 的模板指定另一些成员。

9-2. 重写图 5-4 中的生产模型数据，使其仅由三个数据语句组成，安排如下：集合 PROD 和参数 rate、inv0、prodcost 和 invcost 在一个表格中给出。集合 WEEKS 和参数 avail 在一个表格中给出。参数 revenue 和 market 在一个表格中给出。

9-3. 对于图 3-2 所示数据的指派问题，假设你收到的关于人员对办公室偏好的唯一信息如下：

Coullard M239 M233 D241 D237 D239  
Daskin D237 M233 M239 D241 D239  
C246 C140 Hazen C246 D237 M233 M239 C250 C251 D239  
Hopp D237 M233 M239 D241 C251 C250  
Iravani D237 C138 C118 D241 D239  
Lineisky M233 M239 C250 C251 C246 D237  
Mehrotra D237 D239 M239 M233 D241 C118 C251  
Nelson D237 M233 M239  
Smilowitz M233 M239 D239 D241 C251 C250 D237  
Tamhane M239 M233 C251 C250 C118 C138 D237  
White M239 M233 D237 C246  

这意味着，例如，Coullard 的第一选择是 M239，第二选择是 M233，依此类推，直到她的第五选择 D239，但她没有对其他办公室表达任何偏好。

为了在第 3 章中解释的图 3-1a 的运输模型中使用这些信息，你必须将 cost["Couillard","M239"] 设为 1，cost["Couillard","M233"] 设为 2，以此类推。对于未排名的办公室（例如 C246），你可以将 cost["Couillard","C246"] 设为 99，以表示这是一个非常不受欢迎的分配。

(a) 使用列表格式和默认短语，将上述信息转换为参数 cost 的适当 AMPL 数据语句。  
(b) 同样地，但使用表格格式。

9-4. 第 9.2 节和第 9.3 节给出了一个三维参数 cost（在路径集合 ROUTES 上索引的三元组）的各种数据语句。请按以下方式为该参数构造其他替代方案：

(a) 使用形如 [CLEV, FRA, \*] 的模板。  
(b) 使用形如 $[\pi, \pi,$ bands] 的模板，并采用列表格式。  
(c) 使用形如 [CLEV, \*, \*] 的模板，并采用表格格式。  
(d) 使用带有一个 \* 的模板指定部分参数值，使用带有两个 \* 的模板指定其他参数值。

9-5. 对于图 6-4 中的三维参数 revenue，请按以下方式构造替代的数据语句：

(a) 使用形如 $[*, \text{east}, *]$ 的模板，并采用表格格式。  
(b) 使用形如 $[*, *, 1]$ 的模板，并采用表格格式。  
(c) 使用形如 $[\text{bands}, *, 1]$ 的模板。

9-6. 给定以下声明，

```
set ORIG;  
set DEST;  
var Trans {ORIG, DEST} >= 0;
```

你如何使用数据语句为所有 Trans 变量赋初值 300？

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

# 建模命令

AMPL 提供了多种命令，如 model、solve 和 display，用于告诉 AMPL 建模系统如何处理模型和数据。尽管这些命令可能使用 AMPL 表达式，但它们本身并不是建模语言的一部分。它们旨在用于一种环境中：你发出一个命令，等待系统显示响应，然后决定下一步要发出什么命令。命令可以直接键入，或通过图形用户界面（如 AMPL 网站上提供的界面）中的菜单隐式给出。命令也出现在脚本中，相关内容见第 13 章。

本章首先描述命令环境的一般原则。第 11.2 节接着介绍最常用于设置和求解优化问题的命令。

在求解问题并查看结果之后，下一步通常是进行修改并再次求解。本章的其余部分解释了在不重新启动 AMPL 会话的情况下可以进行的各种修改。第 11.3 节描述了重新读取数据和修改特定数据值的命令。第 11.4 节描述了完全删除或重新定义模型组件的功能，以及临时删除约束、固定变量或放松变量整数性的功能。（方便的模型信息检查命令可以在第 12 章中找到，特别是第 12.6 节。）

# 11.1 命令和选项的一般原则

要开始一个交互式的 AMPL 会话，必须启动 AMPL 程序，例如通过在提示符下键入命令 amp1，或从菜单中选择它，或点击图标。启动过程在不同的操作系统中必然有所不同；有关详细信息，应参考随 AMPL 软件提供的系统特定说明。

# 命令

命令 如果你使用的是基于文本的界面，在启动 AMPL 后，你应该首先看到的是 AMPL 的提示符：

ampl:

每当你看到这个提示时，表示 AMPL 已经准备好读取并解释你输入的内容。与大多数命令解释器一样，AMPL 会一直等待，直到你按下 “enter” 或 “return” 键，然后处理你在该行输入的所有内容。

一条 AMPL 命令以分号结尾。如果你在一行中输入了一条或多条完整的命令，AMPL 会处理它们，打印出相应的信息，然后再次显示 `ampl:` 提示符。如果你在一行的中间结束了一条命令，系统会提示你在下一行继续输入；你可以通过提示符末尾是问号而不是冒号来判断 AMPL 正在提示你继续输入命令：

```
ampl: display {i in ORIG, j in DEST}
amp1? sum {p in PROD} Trans[i,j,p];
```

你可以在一行中输入任意数量的字符（只要不超过操作系统所施加的限制），并且可以在任意多行中继续输入一条命令。

一些命令会使用文件名来读取或写入信息。文件名可以是任何非分号 `;` 和引号 `" 或 '` 之外的可打印字符序列，或者是由匹配的引号包围的任意字符序列。正确的文件名规则由操作系统决定，而不是由 AMPL 决定。本书中的示例使用了诸如 `diet.med` 这样的文件名，这些文件名几乎适用于任何操作系统。

要结束一次 AMPL 会话，请输入 `end` 或 `quit`。

如果你是通过图形界面运行 AMPL，交互的细节可能会有所不同，但命令界面通常仍然可用，并且实际上在后台也会使用该界面，因此了解如何有效使用它是非常值得的。

# 选项

AMPL 命令的行为不仅取决于你直接输入的内容，还取决于各种选项，例如选择不同的求解器 (solver)、控制结果显示方式等。

每个选项都有一个名称和一个值，该值可以是一个数字或一个字符串。例如，选项 `prompt1` 和 `prompt2` 是指定提示符的字符串。选项 `display_width` 的值是数字，表示 `display` 命令输出的字符宽度。

`option` 命令用于显示和设置选项的值。如果 `option` 后面跟着一个选项名称列表，AMPL 会回复当前的值：

```
ampl: option prompt1, display_width;
option prompt1 'ampl: ';
option display_width 79;
ampl:
```

选项名称中的 `*` 是一个通配符，可以匹配任意字符序列：

```
ampl: option prom*;
option prompt1 'ampl: ';
option prompt2 'ampl? ';
ampl:
```

命令 `option *`，或者仅输入 `option`，会列出所有当前的选项及其值。

当 `option` 后面跟着一个名称和一个值时，它会将指定选项重置为该值。在下面的例子中，我们更改了提示符和显示宽度，然后验证后者是否已更改：

```
ampl: option prompt1 "A> ", display_width 60;
A> option display_width;
option display_width 60;
A>
```

你可以通过用匹配的引号 `'...'` 或 `"..."` 将字符串括起来来指定任何字符串值，如上所示；如果字符串看起来像名称或数字，则可以省略引号。两个连续的引号 (`''` 或 `""`) 表示一个空字符串，这对于某些选项来说是一个有意义的值。另一方面，如果你想将一个长字符串分布在多行中，可以在每行的末尾放置反斜杠字符 `\`。

当 AMPL 启动时，它会将许多选项设置为初始值或默认值。例如，`prompt1` 选项被初始化为 `'ampl: '`，因此提示符以标准方式显示。`display_width` 选项的默认值为 79。其他选项，特别是与特定求解器 (solver) 相关的选项，最初是未设置的：

```
ampl: option cplex_options;
option cplex_options ''; # 未定义
```

要将所有选项恢复为默认值，请使用命令 `reset options`。

AMPL 不维护一个有效的选项主列表，而是接受你定义的任何新选项。因此，如果你拼错了一个选项名称，很可能会意外地定义一个新选项，如下例所示：

```
ampl: option display_width 60;
ampl: option display_w";
option display_width 60;
option display_width 79;
```

`option` 语句也不会检查你是否为选项分配了一个有意义的值。只有在后续命令使用某个选项时，才会被告知值错误。在这方面，AMPL 选项非常类似于操作系统或 shell 的“环境变量”。实际上，你可以使用环境变量的设置来覆盖 AMPL 的选项默认值；有关详细信息，请参阅你的系统特定文档。

# 11.2 模型和数据的设置与求解

要将求解器应用于模型实例，本书中的示例使用 `model`、`data` 和 `solve` 命令：

```
ampl: model diet.mod;
data diet.dat;
solve;
MINDS 5.5: optimal solution found.
6 iterations, objective 88.2
```

`model` 命令指定一个包含模型声明的文件（第 5 至第 8 章），而 `data` 命令指定一个包含模型组件数据值的文件（第 9 章）。`solve` 命令会将优化问题的描述发送给求解器，并检索结果以供检查。本节将更详细地介绍 AMPL 中用于设置和求解模型的主要功能。关于后续更改和重新求解模型的功能，请参见第 11.4 节。

# 输入模型和数据

AMPL 维护一个“当前”模型，即当你输入 `solve` 时将发送给求解器的模型。在交互式会话开始时，当前模型是空的。`model` 命令从文件中读取声明并将其添加到当前模型中；`data` 命令从文件中读取数据语句，为当前模型中已有的组件提供值。因此，你可以使用多个 `model` 或 `data` 命令来构建优化问题的描述，从不同的文件中读取模型和数据的不同部分。

你也可以直接在 AMPL 提示符下输入模型及其数据的部分内容。诸如 param、var 和 subject to 等模型声明充当命令，向当前模型添加组件。第 9 章中的数据语句也充当命令，为已定义的组件（如集合和参数）提供数据值。然而，由于模型和数据语句看起来非常相似，因此你需要告诉 AMPL 你将要输入的是哪一种。AMPL 始终以 "model mode" 启动；语句 data（不带文件名）将解释器切换到 "data mode"，而语句 model（不带文件名）将其切换回来。任何不以数据语句方式开头的命令（如 option、solve 或 subject to）也会将数据模式切换回模型模式。

如果一个模型声明了多个目标函数，AMPL 默认会将所有目标函数传递给求解器。大多数求解器只处理一个目标函数，并且通常默认选择第一个。objective 命令允许你选择单个目标函数传递给求解器；它由关键字 objective 后跟 minimize 或 maximize 声明中的一个名称组成：

objective Total_Number;

如果模型具有索引化的目标函数集合，则必须提供下标以指示选择哪一个：

objective Total_Cost["A&P"];

多个目标函数的使用通过 8.3 节中的两个示例进行了说明。

# 求解模型

solve 命令启动了一系列活动。首先，它使 AMPL 从你提供的模型和数据中生成一个特定的优化问题。如果你忽略了提供某些必要的数据，则会打印错误消息；如果你的数据值违反了 var 或 param 声明中的限定短语或 check 语句所施加的任何限制，你也会收到错误消息。AMPL 会等到你输入 solve 时才验证数据限制，因为限制可能以复杂的方式依赖于许多不同的数据值。诸如除以零之类的算术错误也在此阶段被捕获。

在生成优化问题后，AMPL 进入 "presolve" 阶段，试图使问题对求解器来说更容易。有时 presolve 会极大地减小问题的规模，从而使问题实质上更容易求解。通常，presolve 的工作在幕后进行，你无需关心它。在极少数情况下，当存在多个最优解时，presolve 可能会显著影响变量的最优值，或者可能干扰内置在求解器软件中的其他预处理例程。此外，presolve 有时会检测到不存在可行解，因此不会费心将你的程序发送给求解器。例如，如果你大幅减少 steel4.mod 中某一种资源的可用性，则 AMPL 会产生错误消息：

```text
ampl: model steel4.mod;
ampl: data steel4.dat;
ampl: let avail['reheat'] := 10;
ampl: solve;
presolve: 约束 Time['reheat'] 无法满足：body <= 10 不能 >= 11.25；差值 = -1.25

对于这些情况，你应当查阅第 14.1 节中关于 presolve 的详细描述。

由 AMPL 生成的优化问题（可能经过 presolve 修改后）最终会被发送到你所选择的求解器 (solver)。每个版本的 AMPL 都会附带一个默认的求解器，如果你没有指定其他求解器，它将被自动使用；输入 option solver 查看其名称：

ampl: option solver;
option solver minos;

如果你安装了多个求解器，可以通过更改 solver 选项在它们之间切换：

ampl: model steelT.mod;
data steelT.dat;
ampl: solve;
MINOS 5.5: 找到最优解。
15 次迭代，目标函数值 515033

ampl: reset;
ampl: model steelT.mod;
ampl: data steelT.dat;
ampl: option solver cplex;
ampl: solve;
CPLEX 8.0.0: 最优解；
目标函数值 515033
16 次对偶单纯形迭代（第 I 阶段 0 次）

ampl: reset;
ampl: model steelT.mod;
ampl: data steelT.dat;
ampl: option solver snopt;
ampl: solve;
SNOPT 6.1-1: 找到最优解。
15 次迭代，目标函数值 515033

在这个例子中，我们在每次求解之间重置 (reset) 了问题，这样各个求解器都是在相同的初始条件下被调用，从而可以比较它们的表现。如果不进行 reset，前一个求解器找到的解的信息会被传递给下一个求解器，可能使后者获得显著的优势。当一系列类似的线性规划 (LP) 问题要发送给同一个求解器时，将信息从前一次求解传递到下一次是最有用的；我们将在第 14.2 节中更详细地讨论这种情况。

几乎任何求解器都可以处理线性规划问题，尽管专门为线性规划设计的求解器通常表现最佳。其他类型的优化问题，例如非线性规划（第 18 章）和整数规划（第 20 章），只能由专为它们设计的求解器处理。出现诸如 "ignoring integrality" 或 "can't handle nonlinearities" 之类的消息，说明你选择的求解器与模型不匹配。

如果你的优化问题不算太复杂，你应该可以不参考特定求解器的说明而直接使用 AMPL：正确设置 solver 选项，输入 solve，然后等待结果即可。

如果你的求解器花费很长时间才返回解，或者返回 AMPL 时没有任何 "optimal solution" 消息，那么就该进一步阅读相关文档了。每个求解器都是一套复杂的算法和策略集合，可以从中做出多种组合选择。对于大多数问题，求解器会自动做出良好的选择，但你也可以通过 AMPL 选项传递自己的选择。具体细节可能因求解器而异，因此更多信息请查阅随你的 AMPL 软件一起提供的、针对特定求解器的说明文档。
```

如果您的问题需要很长时间来优化，您会希望看到一些求解器 (solver) 进度的证据显示在屏幕上。出于此目的的指令也在特定求解器的说明中进行了描述。

# 11.3 修改数据

许多建模项目涉及求解一系列问题实例，每个实例由略有不同的数据定义。我们在此描述 AMPL 重置参数值同时保持模型不变的功能。这些功能包括用于重置数据模式输入和重新采样随机参数的命令，以及用于直接赋新值的 let 命令。

# 重置

要删除几个模型组件的当前数据，而不更改当前模型本身，请使用 reset data 命令，例如：

reset data MINREQ, MAXREQ, amt, n_min, n_max;

然后您可以使用 data 命令为这些集合 (set) 和参数读取新值。要删除所有数据，请键入 reset data。

update data 命令的工作方式类似，但在分配新值之前实际上不会删除任何数据。因此，如果您键入 update data MINREQ, MAXREQ, amt, n_min, n_max;

但只读取了 MINREQ、amt 和 n_min 的新值，则 MAXREQ 和 n_max 的先前值将保留。如果您使用的是 reset data，则 MAXREQ 和 n_max 将没有值，并且在您下次尝试求解时会收到错误消息。

# 重新采样

reset data 命令还起到重新采样第 7.6 节中描述的随机计算参数的作用。继续使用第 7.6 节中介绍的 steel4.mod 变体，如果参数 avail 的定义被更改为使其值由随机函数给出：

param avail_mean {STAGE} >= 0; param avail_variance {STAGE} >= 0; param avail {s in STAGE} = Normal(avail_mean[s], avail_variance[s]);

相应的数据为：

param avail_mean := reheat 35 roll 40 ; param avail_variance := reheat 5 roll 2 ; 则在每次 reset data 后 AMPL 将从正态分布中获取新样本。不同的样本导致不同的解，从而导致不同的最优目标值 (objective)：

ampl: model steel4r.mod; ampl: data steel4r.dat; ampl: solve; MINOS 5.5: optimal solution found. 3 iterations, objective 187632.2489 ampl: display avail; reheat 32.3504 roll 43.038 ; ampl: reset data avail; ampl: solve; MINOS 5.5: optimal solution found. 4 iterations, objective 158882.901 ampl: display avail; reheat 32.0306 roll 32.6855 ;

只有 reset data 具有此效果；如果您发出 reset 命令，则 AMPL 的随机数生成器将被重置，并且 avail 的值将从头开始重复。（第 7.6 节解释了如何重置生成器的"种子"以获得不同的随机数序列。）

# let 命令

`let` 命令还允许你在保持模型不变的情况下更改特定的数据值，但它比 `reset data` 或 `update data` 更适合用于小规模或易于描述的更改。例如，你可以使用它来求解图 5-1 中的饮食模型 (diet model)，并尝试一系列对食物 CHK 的购买上限 `f_max["CHK"]`：

```ampl
ampl: model dietu.mod;
ampl: data dietu.dat;
ampl: solve;
MINOS 5.5: optimal solution found.
5 iterations, objective 74.27382022
ampl: let f_max["CHK"] := 11;
ampl: solve;
MINOS 5.5: optimal solution found.
1 iterations, objective 73.43818182
ampl: let f_max["CHK"] := 12;
ampl: solve;
MINOS 5.5: optimal solution found.
0 iterations, objective 73.43818182
```

将上限放宽到 11 会略微降低成本，但进一步放宽显然没有收益。

在 `let` 关键字之后可以给出索引表达式 (indexing expression)，在这种情况下，会对指定索引集的每个成员进行更改。你可以使用此功能将所有上界更改为 8：

```ampl
let {j in FOOD} f_max[j] := 8;
```

或者将所有上界提高 10%：

```ampl
let {j in FOOD} f_max[j] := 1.1 * f_max[j];
```

一般来说，该命令由关键字 `let`、（如有需要的）索引表达式和赋值语句组成。任何未通过 `=` 语句定义的集合或参数都可以出现在赋值运算符 `:=` 的左侧，而右侧可以是当前可以计算的任何适当表达式。

尽管 AMPL 对使用 `let` 可以更改的内容没有施加任何限制，但在更改任何影响其他数据索引的集合或参数时应小心。

例如，在求解图 4-4 和图 4-5 中的多周期生产问题后，可能会想将周数 T 从原始数据中的 4 改为 3：

```ampl
ampl: let T := 3;
ampl: solve;
Error executing "solve" command:
error processing param avail:
    invalid subscript avail[4] discarded.
error processing param market:
    2 invalid subscripts discarded:
    market['bands', 4]
    market['coils', 4]
error processing param revenue:
    2 invalid subscripts discarded:
    revenue['bands', 4]
    revenue['coils', 4]
error processing var Sell['coils', 1]:
    invalid subscript market['bands', 4]
```

这里的问题是，AMPL 仍然保留了第 4 周参数（如 `avail[4]`）的当前数据，而随着 T 被更改为 3，这些数据变得无效。如果你想在使用相同数据的同时正确减少线性规划中的周数，必须声明两个参数：

```ampl
param Tdata integer > 0;
param T integer <= Tdata;
```

在参数声明中使用 `1..Tdata` 进行索引，同时在变量、目标函数和约束中保留 `1..T`；然后你就可以随意使用 `let` 更改 T。

你也可以使用 `let` 命令更改变量的当前值。这有时是探索替代解的便捷功能。例如，在这里

ceil $(x)$  $\mathcal{X}$ 的向上取整 (下一个较大的整数)  
floor $(x)$  $\mathcal{X}$ 的向下取整 (下一个较小的整数)  
precision $(x,n)$  $\mathcal{X}$ 四舍五入到 $n$ 位有效数字  
round $(x,n)$  $\mathcal{X}$ 四舍五入到小数点后 $n$ 位  
round $(x)$  $\mathcal{X}$ 四舍五入到最近的整数  
trunc $(x,n)$  $\mathcal{X}$ 截断到小数点后 $n$ 位  
trunc $(x)$  $\mathcal{X}$ 截断为整数 (丢弃小数部分)

表 11-1：舍入函数。

下面展示的是当我们按上述方法求解最优饮食问题后，将最优解向下取整到下一个最小的整数包数时发生的情况：

```
ampl: model dietu.mod; data dietu.dat; solve;
MINOS 5.5: optimal solution found. 5 iterations, objective 74.27382022

ampl: let {j in FOOD} Buy[j] := floor(Buy[j]);
ampl: display Total_Cost, n_min, Diet_Min.slack;

Total_Cost = 70.8

n_min Diet_Min.slack :=
 A   700   231
 B1    0   580
 B2    0   475
 C   700   -40
 CAL 16000 -640
```

由于我们使用了 `let` 来更改变量的值，目标函数和松弛值会自动根据新的、已舍入的值重新计算。成本降低了约 \$3.50，但该解现在缺少 `C` 要求的近 6% 和 `CAL` 要求的 4%。

AMPL 提供了多种可用于此目的的舍入函数。它们汇总于表 11-1 中。

# 11.4 修改模型

提供了一些命令来帮助你在不修改模型文件或执行完整重置的情况下对当前模型进行有限的更改。本节将介绍那些可以完全删除或重新定义模型组件，并临时移除约束、固定变量或放松变量整数限制的命令。

# 删除或重新定义模型组件

`delete` 命令用于删除先前声明的模型组件，前提是其他组件未在其声明中使用它。该命令的形式非常简单，即 `delete` 后跟以逗号分隔的模型组件名称列表：

```
ampl: model dietobj.mod;
ampl: data dietobj.dat;
ampl: delete Total_Number, Diet_Min;
```

通常情况下你不能删除一个集合、参数或变量，因为它们在模型中后续会被使用；但你可以删除任何目标函数或约束。你也可以指定形式为 `check` $n$ 的“名称”来删除当前模型中的第 $n$ 个检查语句。

`purge` 命令具有相同的形式，但关键字 `purge` 替代了 `delete`。它不仅删除列出的组件，还删除所有直接（通过引用）或间接（通过引用其依赖项）依赖于它们的组件。例如，在 `diet.mod` 中我们有：

```
param f_min {FOOD} >= 0;
param f_max {j in FOOD} >= f_min[j];
var Buy {j in FOOD} >= f_min[j], <= f_max[j];
minimize Total_Cost: sum {j in FOOD} cost[j] * Buy[j];
```

命令 `purge f_min` 会删除参数 `f_min`，以及声明中引用了 `f_min` 的组件，包括参数 `f_max` 和变量 `Buy`。它还会删除目标函数 `Total Cost`，因为该目标函数通过引用 `Buy` 间接依赖于 `f_min`。

如果你不确定哪些组件依赖于某个给定组件，可以使用 `xref` 命令来查找：

```
ampl: xref f_min; # 4 个实体依赖于 f_min：f_max Buy Total Cost Diet
```

与 `delete` 和 `purge` 类似，`xref` 命令也可以应用于任何模型组件列表。

一旦某个组件被 `delete` 或 `purge` 删除，该组件名称之前被隐藏的含义将再次可见。例如，在删除名为 `prod` 的约束后，AMPL 会再次将 `prod` 识别为迭代乘法运算符（见表 7-1）。

如果没有之前被隐藏的含义，则通过 `delete` 或 `purge` 删除的组件名称将变为未使用状态，并且之后可以被声明为任何新组件的名称，无论其类型如何。然而，如果你只想对声明进行相对有限的修改，那么使用 `redeclare` 可能更方便。你可以通过编写关键字 `redeclare`，然后接上你希望替换的完整新声明来更改任何组件的声明。再次查看 `diet.mod` 文件，例如：

```
ampl: redeclare param f_min {FOOD} > 0 integer;
```

该命令仅更改了 `f_min` 的有效性条件。所有依赖于 `f_min` 的组件声明保持不变，之前为 `f_min` 读入的值也保持不变。

有关 `delete`、`purge`、`xref` 和 `redeclare` 可应用于哪些组件类型的完整列表，请参见 A.18.5 节。

# 更改模型：fix、unfix；drop、restore

更改模型最简单（但也最彻底）的方法是使用 `reset` 命令，它会清除当前的所有模型和数据。执行 `reset` 后，你可以发出新的 `model` 和 `data` 命令以设置不同的优化问题；其效果类似于输入 `quit` 然后重新启动 AMPL，但选项不会重置为其默认值。如果你的操作系统或 AMPL 的图形环境允许你在保持 AMPL 活动的同时编辑文件，那么 `reset` 对调试和实验非常有用；你可以对模型或数据文件进行修改，输入 `reset`，然后读取修改后的文件。（如果你需要退出 AMPL 来运行文本编辑器，可以使用 A.21.1 节中描述的 `shell` 命令。）

`drop` 命令指示 AMPL 忽略当前模型中的某些约束或目标函数。例如，图 5-1 中的约束最初包括：

```
subject to Diet_Max {i in MAXREQ}: sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i];
```

`drop` 命令可以指定忽略这些约束中的某一个：

```
drop Diet_Max["CAL"];
```

或者它可以指定忽略由某个集合索引的所有约束或目标函数：

```
drop {i in MAXNOT} Diet_Max[i];
```

其中 MAXNOT 之前被定义为 MAXREQ 的某个子集。可以通过 `drop {i in MAXREQ} Diet_Max[i];` 忽略整个约束集合；

或者更简单地：

`drop Diet_Max;`

一般来说，该命令由关键字 `drop`、可选的索引表达式以及可能带下标的约束名称组成。连续的 `drop` 命令具有累积效果。

`restore` 命令用于撤销 `drop` 的效果。其语法相同，只是关键字改为 `restore`。

`fix` 命令将指定变量固定在其当前值上，如同存在一个约束要求这些变量必须等于这些值；`unfix` 命令则用于撤销该效果。这些命令的语法与 `drop` 和 `restore` 相同，只是它们指定的是变量而不是约束。例如，这里我们将饮食问题的所有变量初始化为其下界，固定所有表示每包钠含量超过 $1200\,\mathrm{mg}$ 的食物变量，并对剩余变量进行优化：

```
ampl: let {j in FOOD} Buy[j] := f_min[j];
ampl: fix {j in FOOD: amt["NA",j] > 1200} Buy[j];
ampl: solve;
MINDS 5.5: optimal solution found.
7 iterations, objective 86.92
Objective = Total_Cost['A&P']
ampl: display {j in FOOD} (Buy[j].lb,Buy[j],amt["NA",j]);

: Buy[j].lb Buy[j] amt['NA',j] :=
BEEF 2 2 938
CHK 2 2 2180
FISH 2 10 945
HAM 2 2 278
MCH 2 9.42857 1182
MTL 2 10 896
SPG 2 2 1329
TUR 2 2 1397
```

除了通过单独的语句设置和固定变量外，还可以在 `fix` 命令中添加赋值短语：

```
ampl: fix {j in FOOD: amt["NA",j] > 1200} Buy[j] := f_min[j];
```

`unfix` 命令的工作方式相同，用于撤销 `fix` 的效果，并可选择性地重置变量的值。

# 放松整数性 (relax_integrality)

将选项 `relax_integrality` 从默认值 0 改为任意非零值：

```
option relax_integrality 1;
```

告诉 AMPL 忽略所有对变量整数值的限制。声明为整数 (integer) 的变量保留你为其指定的边界，而声明为二进制的变量下界为 0，上界为 1。要恢复整数性限制，请将 `relax_integrality` 选项重新设置为 0。

变量名称后跟后缀 `.relax` 表示其当前的整数性放松状态：0 表示强制整数性，非零表示放松。你可以利用该后缀仅对选定变量放松整数性。例如，

```
let Buy['CHK'].relax = 1
```

仅对变量 `Buy['CHK']` 放松整数性，而

```
let {j in FOOD: f_min[j] > allow_frac} Buy[j].relax := 1;
```

对所有最小购买量 (minimum purchase amount) 至少为某个截断参数 `allow_frac` 的食物变量放松整数性。

一些与 AMPL 配合使用的 求解器 (solver) 提供了它们自己的指令来放松整数性限制，但这些指令的效果不一定与 AMPL 的 `relax_integrality` 选项或 `.relax` 后缀相同。这种区别源于 AMPL 的问题简化阶段（即 presolve 阶段）所产生的影响（见第 14.1 节）。AMPL 在 presolve 阶段之前会移除整数性限制，因此 求解器 接收到的是原始整数问题的一个真正的连续松弛问题。然而，如果松弛操作由 求解器 执行，则在 AMPL 的 presolve 阶段期间整数性限制仍然有效，因此 AMPL 可能会因此进行一些额外的紧缩和简化。

举一个简单的例子，假设饮食模型中的变量声明允许通过设置一个额外的参数 `scale` 来调整食物上限 `f_max`：

```ampl
var Buy {j in FOOD} integer >= f_min[j], <= scale * f_max[j];
```

在图 2-3 的示例中，所有的 `f_max` 值均为 10；假设我们将 `scale` 设置为 0.95。首先，以下是求解未松弛问题的结果：

```ampl
ampl: option relax_integrality 0;
ampl: let scale := 0.95;
ampl: solve;
CPLEX 8.0.0: optimal integer solution; objective 122.89
6 MIP simplex iterations
0 branch-and-bound nodes
```

当在 AMPL 中未指定松弛时，presolve 阶段会发现所有变量的上界为 9.5，并且由于它知道这些变量必须取整数值，因此会将这些上界向下取整为 9。然后这些上界会被发送给求解器，即使我们指定了求解器的整数性松弛指令，这些上界也不会改变：

```ampl
ampl: option cplex_options 'relax';
ampl: solve;
CPLEX 8.0.0: relax
Ignoring integrality of 8 variables.
CPLEX 8.0.0: optimal solution; objective 120.2421057
2 dual simplex iterations (0 in phase I)
ampl: display Buy;
Buy [*] :=
BEEF 8.39898
CHK 2
FISH 2
HAM 9
MCH 9
MTL 9
SPG 8.93436
TUR 2
;
```

相反，如果将 `option relax_integrality` 设置为 1，则 presolve 会将上界保持为 9.5 并发送给求解器，从而导致一个约束更宽松的问题，因此目标值更低：

```matlab
ampl: option relax_integrality 1;
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 119.15075453
3 dual simplex iterations (0 in phase I)
ampl: display Buy;
Buy [*] :=
BEEF 6.8798
CHK 2
FISH 2
HAM 9.5
MCH 9.5
MTL 9.5
SPG 9.1202
TUR 2
;
```

在前一个解中处于上界 9 的变量，现在处于上界 9.5。

同样的情况可能在更不明显的情况下出现，并导致意外的结果。一般来说，在 AMPL 的 `relax_integrality` 选项下，整数规划的最优值（对于最小化问题）可能低于求解器的松弛指令所报告的最优值；对于最大化问题则可能更高。

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

# 命令脚本

你可能会发现，使用 AMPL 命令环境最频繁的阶段是在模型的初始开发过程中，此时结果尚不熟悉且修改频繁。当模型形式最终确定下来后，你可能会发现自己反复输入相同的命令序列，以求解不同的数据集合。为了加速这一过程，你可以安排 AMPL 从文件中读取常用的命令序列，或者自动重复命令序列，并根据中间结果决定如何继续以及何时停止。

脚本 (script) 是一系列命令，保存在一个文件中，供反复使用。脚本可以包含任何 AMPL 命令，也可以包括编程语言结构，如 `for`、`repeat` 和 `if`，用于重复执行语句或有条件地执行语句。实际上，这些及相关命令使你能够在 AMPL 命令语言中编写小程序。另一组命令则允许逐步执行脚本，以便观察或调试。本章将介绍 AMPL 命令脚本，并以格式化打印和敏感性分析为例进行说明。

AMPL 命令脚本能够直接处理模型定义中核心的字符串集合。例如，`for` 语句可以指定对某个集合的每个成员执行一次的命令。为了支持处理字符串的脚本，AMPL 提供了多种字符串函数和运算符，其使用方法将在本章最后一节中介绍。

# 13.1 运行脚本：`include` 和 `commands`

AMPL 提供了多个命令，可使输入从文件中读取。命令

`include filename` 会被指定文件的内容所替代。`include` 甚至可以出现在其他语句的中间，并且不需要以分号结尾。

我们示例中常见的 `model` 和 `data` 命令是 `include` 的特例，它们会在读取指定文件之前，将命令解释器设置为模型模式或数据模式。相比之下，`include` 不会改变当前模式。为了简化起见，本书示例总是假设 `model` 用于读取模型声明文件，而 `data` 用于读取数据值文件。然而，你可以使用 `model`、`data` 或 `include` 中的任意一个命令来读取任何文件；唯一的区别在于读取开始时所设置的模式。例如，在处理一个小型模型时，你可能会发现将所有模型声明、一个 `data` 命令以及所有数据语句存储在一个文件中更为方便；此时，无论是使用 `model` 还是 `include` 命令都可以一次性读取该文件，从而同时设置模型和数据。

举例来说，如果文件 `dietu.run` 包含以下内容：

```
model dietu.mod;
data dietu.dat;
solve;
option display_1col 5;
option display_round 1;
display Buy;
```

那么包含该文件将加载模型和数据，运行问题，并显示变量的最优值：

ampl: include dietu.run; MINDS 5.5: 最优解 (optimal solution) 已找到。迭代 6 次。目标值 74.27382022 Buy $[\star ]\coloneqq$ BEEF 2.0 FISH 2.0 MCH 2.0 SPG 5.3 CHR 10.0 HAM 2.0 MTLD 6.2 TUR 2.0

当一个被包含的文件本身包含 include、model 或 data 命令时，第一个文件的读取会被暂停，直到被包含文件的内容处理完毕。在这个例子中，命令 include dietu.run 导致后续包含文件 dietu.mod 和 dietu.dat。

一种特别有用的 include 文件是包含一组 option 命令的列表，这些命令你希望在运行其他任何命令之前执行，以修改默认选项。你可以安排在启动时自动包含这样的文件；甚至可以让 AMPL 在会话结束时自动写入这样的文件，以便下次恢复你的选项设置。此安排的详细信息取决于你的操作系统；参见第 A.14.1 节和第 A.23 节。

语句 commands filename; 与 include 非常相似，但它是一个真正的语句，需要以分号结尾，并且只能出现在允许语句的上下文中。

为了说明 commands 的用法，考虑我们如何对第 4.2 节中的多周期生产问题进行简单的敏感性分析。第 3 周只有 32 小时的生产时间，而其他周为 40 小时。假设我们想查看第 3 周每增加一小时能带来多少额外利润。我们可以通过反复求解、显示解值并增加 avail[3] 来实现：

mpl: model steelT.mod; ampl: data steelT.dat; ampl: solve; MINDS 5.5: 最优解 (optimal solution) 已找到。迭代 15 次，目标值 515033 ampl: display Total_Profit >steelT.sens; ampl: option display_1col 0; ampl: option omit_zero_rows 0; ampl: display Make >steelT.sens; ampl: display Sell >steelT.sens; ampl: option display_1col 20; ampl: option omit_zero_rows 1; ampl: display Inv >steelT.sens; ampl: let avail[3] := avail[3] + 5; ampl: solve; MINDS 5.5: 最优解 (optimal solution) 已找到。迭代 1 次，目标值 532033 ampl: display Total_Profit >steelT.sens; ampl: option display_1col 0; ampl: option omit_zero_rows 0; ampl: display Make >steelT.sens; ampl: display Sell >steelT.sens; ampl: option display_1col 20; ampl: option omit_zero_rows 1; ampl: display Inv >steelT.sens; ampl: let avail[3] := avail[3] + 5; ampl: solve; MINDS 5.5: 最优解 (optimal solution) 已找到。迭代 1 次，目标值 549033 ampl:

若要继续以步长 5 尝试 avail[3] 的值直到 62，我们必须以相同方式完成另外四次求解循环。我们可以通过创建一个包含要重复命令的新文件来避免重复输入相同命令：

solve; display Total_Profit >steelT.sens; option display_1col 0; option omit_zero_rows 0; display Make >steelT.sens; display Sell >steelT.sens; option display_1col 20; option omit_zero_rows 1; display Inv >steelT.sens; let avail[3] := avail[3] + 5;

如果我们把该文件命名为 `steelT.sal`，那么只需键入一行命令 `commands steelT.sal` 即可执行其中的所有命令：

```
ampl: model steelT.mod;
ampl: data steelT.dat;
ampl: commands steelT.sal;
MINOS 5.5: optimal solution found.
15 iterations, objective 515033
ampl: commands steelT.sal;
MINOS 5.5: optimal solution found.
1 iterations, objective 532033
ampl: commands steelT.sal;
MINOS 5.5: optimal solution found.
1 iterations, objective 549033
ampl: commands steelT.sal;
MINOS 5.5: optimal solution found.
2 iterations, objective 565193
ampl:
```

（所有来自 `display` 命令的输出都被重定向到了文件 `steelT.sens` 中，当然我们也可以让它显示在屏幕上。）

在这种情况以及其他许多情况下，你也可以使用 `include` 来代替 `commands`。但一般来说，在命令脚本中最好使用 `commands`，以避免与 `repeat`、`for` 和 `if` 语句发生意外的交互。

# 13.2 遍历集合：for 语句

上面的例子仍然需要重复键入某些命令。AMPL 提供了循环命令，可以自动完成这类工作，并提供多种选项来控制循环的持续时间。

我们首先介绍 `for` 语句，它对某个集合中的每个成员执行一次语句或语句集合。例如，为了执行我们多周期生产问题的敏感性分析脚本四次，可以使用一个简单的 `for` 语句，后跟我们想要重复的命令：

```
ampl: model steelT.mod;
ampl: data steelT.dat;
ampl: for {1..4} commands steelT.sal;
MINOS 5.5: optimal solution found.
15 iterations, objective 515033
MINOS 5.5: optimal solution found.
1 iterations, objective 532033
MINOS 5.5: optimal solution found.
1 iterations, objective 549033
MINOS 5.5: optimal solution found.
2 iterations, objective 565193
ampl:
```

位于 `for` 和命令之间的表达式可以是任意的 AMPL 索引表达式。

除了从单独的文件中读取命令外，我们还可以将它们写成 `for` 语句的主体，并用花括号括起来：

```
model steelT.mod;
data steelT.dat;
for {1..4} {
    solve;
    display Total_Profit > steelT.sens;
    option display_1col 0;
    option omit_zero_rows 0;
    display Make > steelT.sens;
    display Sell > steelT.sens;
    option display_1col 20;
    option omit_zero_rows 1;
    display Inv > steelT.sens;
    let avail[3] := avail[3] + 5;
}
```

如果将该脚本存储在 `steelT.sa2` 中，那么只需键入以下命令即可完成整个迭代敏感性分析：

这种方法通常更清晰、更易于操作，特别是当我们使循环变得更加复杂时。作为一个第一个例子，考虑如何为 `avail[3]` 的连续值编制一个包含目标函数值和约束 `Time[3]` 的对偶值的表格。为此目的编写的脚本如图 13-1 所示。在读取模型和数据之后，脚本提供了用于存储表格值的额外声明：

```
# ampl: commands steelT.sa2

set AVAIL3;
param avail3_obj {AVAIL3};
param avail3_dual {AVAIL3};
```

集合 AVAIL3 将包含所有我们想要尝试的 `avail[3]` 的不同值；对于每个这样的值 a，`avail3_obj[a]` 和 `avail3_dual[a]` 将是相关的目标函数值和对偶值。一旦这些设置完成，我们将集合值赋给 AVAIL3：

然后使用 for 循环遍历这个集合：

```ampl
for {a in AVAIL3} {
    let avail[3] := a;
    solve;
    let avail3_obj[a] := Total_Profit;
    let avail3_dual[a] := Time[3].dual;
}
```

我们在这里看到，for 循环可以遍历任意集合，并且在循环内可以使用遍历该集合的索引（本例中为 a）。循环结束后，通过显示 `avail3_obj` 和 `avail3_dual` 即可生成所需的表格，如图 13-1 脚本末尾所示。如果该脚本存储在文件 steelT.sa3 中，则可通过一条命令生成所需结果：

```ampl
ampl: commands steelT.sa3;
avail3_obj avail3_dual :=
32    515033    3400
37    532033    3400
42    549033    3400
47    565193    2980
```

在此示例中，我们通过在脚本中包含命令 `option solver_msg 0` 抑制了来自求解器的消息。

AMPL 的 for 循环也便于生成格式化表格。假设在求解多周期生产问题后，我们希望以吨数和占市场限额的百分比形式显示销售数据。我们可以使用 display 命令生成如下表格：

```ampl
ampl: display {t in 1..T, p in PROD}
      (Sell[p,t], 100*Sell[p,t]/market[p,t]);
:      Sell[p,t]    100*Sell[p,t]/market[p,t]    :=
1    bands    6000    100
1    coils    307     7.675
2    bands    6000    100
2    coils    2500    100
3    bands    1400    35
3    coils    3500    100
4    bands    2000    30.7692
4    coils    4200    100
```

通过编写使用 printf 命令（A.16）的脚本，我们可以创建更有效的表格：

```ampl
ampl: commands steelT.tabl;
SALES        bands            coils
week 1     6000   100.0%     307    7.7%
week 2     6000   100.0%    2500  100.0%
week 3     1399    35.0%    3500  100.0%
week 4     1999    30.8%    4200  100.0%
```

编写此表格的脚本可以简化为两条 printf 命令：

```ampl
printf "\n%s%14s%17s\n", "SALES", "bands", "coils";
printf {t in 1..T}: "week %d%9d%7.1f%%%9d%7.1f%%\n", t,
    Sell["bands",t], 100*Sell["bands",t]/market["bands",t],
    Sell["coils",t], 100*Sell["coils",t]/market["coils",t];
```

然而，这种方法存在不理想的限制，因为它假设总是只有两种产品，并且它们总是被命名为 coils 和 bands。实际上，printf 语句无法写出行数和列数都依赖于数据的表格，因为其格式字符串中的条目数量始终是固定的。

生成表格的更通用脚本如图 13-2 所示。每次遍历 {1..T} 的“外层”循环会生成表格的一行。在每次遍历中，“内层”循环遍历 PROD 以生成该行的产品条目。与之前的示例相比，printf 语句更多，但它们更短且更简单。我们使用多个语句来写入每一行的内容；printf 除了在格式字符串中出现换行符 (\n) 时，不会自动开始新的一行。

循环可以嵌套任意深度，并且可以遍历任何能通过 AMPL 集合表达式表示的集合。循环对集合的每个成员都会执行一次遍历，如果集合是有序的（如 1..T 这样的数字集合，或声明为 ordered 或 circular 的集合），则遍历的顺序由集合的排序决定。如果集合是无序的（如 PROD），则 AMPL 会选择遍历的顺序，

```ampl
printf "\nSALES"; 
printf {p in PROD}: "%14s ", p; 
printf "\n"; 
for {t in 1..T} { 
    printf "week %d", t; 
    for {p in PROD} { 
        printf "%9d", Sell[p,t]; 
        printf "%7.1f%%", 100 * Sell[p,t]/market[p,t]; 
    } 
    printf "\n"; 
}
```

图 13-2：使用嵌套循环生成格式化销售表格 (steelT.tab1)。

但每次的选择都是一致的；图 13-2 中的脚本依赖于这种一致性，以确保表格中某一列的所有条目都对应同一个产品。

## 13.3 带条件的迭代：repeat 语句

另一种循环结构是 repeat 语句，它会在满足某个逻辑条件时继续迭代。

回到灵敏度分析示例，我们希望利用约束 `Time[3]` 的对偶值 (dual value) 的一个性质：每增加一小时的 `avail[3]`，所能实现的额外利润最多为 `Time[3].dual`。当 `avail[3]` 足够大时，即第三周的产能超过了可使用的量，相应的对偶值会降至零，此时进一步增加 `avail[3]` 不会对最优解产生影响。

我们可以通过编写具有以下形式之一的 `repeat` 语句来指定当对偶值降至零时停止循环：

```ampl
repeat while Time[3].dual > 0 { ... };
repeat until Time[3].dual = 0 { ... };
repeat { ... } while Time[3].dual > 0;
repeat { ... } until Time[3].dual = 0;
```

此处的循环体 (由 $\{\ldots \}$ 表示) 必须用花括号括起来。只要 `while` 条件为真，或者 `until` 条件为假，循环就会继续执行。出现在循环体之前的条件会在每次循环开始前进行测试；如果在第一次循环之前 `while` 条件为假，或者 `until` 条件为真，则循环体永远不会执行。出现在循环体之后的条件会在每次循环结束后进行测试，因此在这种情况下循环体至少会执行一次。如果没有 `while` 或 `until` 条件，循环将无限重复，必须通过其他方式（如下文所述的 `break` 语句）终止。

图 13-3 展示了一个使用 `repeat` 的完整脚本。在这个具体应用中，我们选择将 `until` 短语放在循环体之后，因为直到第一次求解执行完毕之前，我们并不希望测试 `Time[3].dual`。该脚本还有另外两个值得注意的特点，因为它们与许多此类脚本相关。

在脚本开始时，我们不知道 `repeat` 语句会进行多少次循环。因此，我们无法像图 13-1 中那样提前确定 `AVAIL3`。取而代之的是，我们最初将其声明为空集：

```ampl
set AVAIL3 default {};
param avail3_obj {AVAIL3};
param avail3_dual {AVAIL3};
```

并在每次求解后将新的 `avail[3]` 值添加进去：

```ampl
let AVAIL3 := AVAIL3 union {avail[3]};
let avail3_obj[avail[3]] := Total_Profit;
let avail3_dual[avail[3]] := Time[3].dual;
let avail[3] := avail[3] + avail3_step;
} until Time[3].dual = 0;
display avail3_obj, avail3_dual;
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/6b2950f99e0f5d5f43b9aad93fb22af20e282fdd39de99684d8bc33eeafd3c5d.jpg)  
图 13-3：记录灵敏度的脚本 (steelT.sa4)。

通过向 `AVAIL3` 添加新成员，我们同时也为以 `AVAIL3` 为索引的参数 `avail3_obj` 和 `avail3_dual` 创建了新的分量，因此我们可以继续为这些分量赋值。对集合的任何更改都会以与参数更改相同的方式传播到所有使用该集合的声明中。

由于计算机中数字的表示精度有限，求解器返回的结果可能与使用精确算术计算出的结果略有差异。通常情况下你不会注意到这一点，因为 `display` 命令默认会将数值四舍五入到六位有效数字。例如：

ampl: model steelT.mod; data steelT.dat; solve; ampl: display Make; Make  $[\star ,\star ]$  (tr) :     bands      coils 1   5990       1407 2   6000       1400 3   1400       3500 4   2000       4200

通过将 display_precision 设置为 0 来取消四舍五入后显示的内容如下：

ampl: option display_precision 0;  ampl: display Make;  Make  $[*,*)$  (tr)  :     bands                 coils 1   5989.999999999999     1407.0000000000002 2   6000                  1399.9999999999998 3   1399.99999999999995   3500 4   1999.9999999999993    4200  ;

这些看似微小的差异在脚本使用 求解器 返回的值进行比较时可能会产生不良影响。例如，从四舍五入的表格中你会认为 `Make["coils", 2] >= 1400` 为真，而从第二个表格中可以看出实际上它是假的。

你可以通过更仔细地编写算术测试来避免这种意外；例如，不使用 `until Time[3].dual = 0`，而可以使用 `until Time[3].dual <= 0.0000001`。或者，你可以对 求解器 返回的所有解值进行四舍五入，使得本应相等的数字真正相等。在图 13-3 开头处的语句 `option solution_precision 10;` 就有这种效果；它规定解值应四舍五入到 10 位有效数字。此选项及相关四舍五入选项在第 12.3 节中有详细讨论和示例说明。

最后请注意，脚本中声明集合 AVAIL3 时使用的是 `default {}` 而不是 `= {}`。前者允许在脚本执行过程中通过 `let` 命令更改 AVAIL3，而后者则将 AVAIL3 永久定义为空集。

# 13.4 条件测试：if-then-else 语句

在第 7.3 节中，我们描述了条件（if-then-else）表达式，它生成一个可用于任何表达式上下文的算术值或集合值。if-then-else 语句使用相同的语法形式来有条件地控制语句或语句组的执行。

在最简单的情况下，if 语句会对一个条件进行求值，并在条件为真时执行指定操作：

```ampl
if Make["coils", 2] < 1500 then printf "under 1500\n";
```

该操作也可以是用花括号分组的一系列命令，就像 for 和 repeat 命令一样：

```ampl
if Make["coils", 2] < 1500 then {
    printf "Fewer than 1500 coils in week 2. \n";
    let market["coils",2] := market["coils",2] * 1.1;
}
```

可选的 else 指定一个替代操作，该操作也可以是单个命令：

```ampl
if Make["coils",2] < 1500 then {
    printf "Fewer than 1500 coils in week 2. \n";
    let market["coils",2] := market["coils",2] * 1.1;
} else printf "At least 1500 coils in week 2. \n";
```

或一组命令：

```ampl
if Make["coils",2] < 1500 then printf "under 1500\n";
else {
    printf "at least 1500\n";
    let market["coils",2] := market["coils",2] * 0.9;
}
```

AMPL 执行这些命令时，首先对 if 后面的逻辑表达式进行求值。如果表达式为真，则执行 then 后面的命令；如果表达式为假，则执行 else 后面的命令（如果有的话）。

`if` 命令主要用于控制脚本中的执行流程。在图 13-2 中，我们可以通过将打印 `Sell[p,t]/market[p,t]` 的语句放在 `if` 语句中来抑制任何 $100\,\%$ 的出现：

```ampl
if Sell[p,t] < market[p,t] then
    printf "%7.1f%%", 100 * Sell[p,t]/market[p,t];
else
    printf "---";
```

在图 13-3 的脚本中，我们可以在 `repeat` 循环内部使用 `if` 命令，以检测对偶值 (dual value) 自上次循环以来是否发生了变化，如图 13-4 的脚本所示。该循环会创建一个表，其中每个发现的不同对偶值恰好对应一个条目。

`then` 或 `else` 后面的语句本身也可以是一个 `if` 语句。在格式化示例中（图 13-2），我们可以通过编写以下代码来特别处理 $0\,\%$ 和 $100\,\%$：

```
if Sell[p,t] < market[p,t] then
  if Sell[p,t] = 0 then
    printf " ";
  else
    printf "%7.1f%%", 100 * Sell[p,t] / market[p,t];
else
  printf "---";
```

或者等价地，但可能更清晰地写成：

```
if Sell[p,t] = 0 then
  printf " ";
else
  if Sell[p,t] < market[p,t] then
    printf "%7.1f%%", 100 * Sell[p,t] / market[p,t];
  else
    printf "---";
```

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/7be36adbc812790d7caac9a7edf106f7fa9959256a616271ccf9f7e64f4a84ce.jpg)  
图 13-4：在循环中测试条件 (steelT.sa5)。

在所有情况下，`else` 都与最近的、可用的 `if` 配对。

# 13.5 终止循环：break 和 continue

还有另外两个语句可以与循环语句配合使用，使某些脚本更容易编写。`continue` 语句用于终止 `for` 或 `repeat` 循环的当前轮次；当前轮次中后续的所有语句都会被跳过，并继续执行控制下一轮循环开始的测试（如果有的话）。`break` 语句则完全终止 `for` 或 `repeat` 循环，并立即将控制权转移到循环结束后的下一条语句。

作为这两个命令的示例，图 13-5 展示了另一种编写图 13-4 中循环的方法，仅当与 `avail[3]` 关联的对偶值发生变化时才创建表项。求解后，我们测试新的对偶值是否等于前一个对偶值：

``` 
if Time[3].dual = previous_dual then continue;
```

如果是，则对于当前的 `avail[3]` 值无需执行任何操作，`continue` 语句将跳转到当前轮次的末尾；执行将在下一轮循环中恢复，从循环的开头开始。

在向表中添加条目后，我们测试对偶值是否降至零：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/d2d0a00f20985afa2fa899acd679b97aa256da573834a2d8da1f8b719bd41a39.jpg)  
图 13-5：在循环中使用 break 和 continue (steelT.sa7)。

```
if Time[3].dual = 0 then break;
```

如果是，则循环结束，`break` 语句跳出循环；执行将传递到脚本中循环后面的 `display` 命令。由于此示例中的 `repeat` 语句没有 `while` 或 `until` 条件，它依赖 `break` 语句来终止循环。

当 `break` 或 `continue` 位于嵌套循环内部时，它仅作用于最内层的循环。这一约定通常能达到预期效果。例如，考虑如何扩展图 13-5，以便对每个 `avail[t]` 进行单独的敏感性分析。`repeat` 循环将嵌套在一个 `for {t in 1..T}` 循环中，但 `continue` 和 `break` 语句仍将如前所述作用于内层的 `repeat` 循环。

确实存在一些情况，脚本的逻辑需要跳出多层循环。例如，在图 13-5 的脚本中，我们可以设想，当 `Time[3].dual` 等于零时并不停止，而是当任意 `t` 对应的 `Time[t].dual` 小于 2700 时停止。实现该条件的一种方法似乎是：

```ampl
for {t in 1..T} if Time[t].dual < 2700 then break;
```

然而，该语句并未达到预期效果，因为 `break` 仅适用于包含它的内层 `for` 循环，而不是我们所希望的外层 `repeat` 循环。在这种情况下，我们可以为循环命名，`break` 或 `continue` 可以通过名称指定其应作用的循环。利用这一特性，示例中的外层循环可命名为 `sens_loop`：

```ampl
repeat sens_loop {
```

其中的终止条件测试可以引用该名称：

```ampl
for {t in 1..T} if Time[t].dual < 2700 then break sens_loop;
```

循环名称紧跟在 `repeat` 或 `for` 之后，并在 `break` 或 `continue` 之后出现。

# 13.6 逐步执行脚本

如果你认为某个脚本未按预期运行，可以指示 AMPL 逐条命令地执行它。该功能可用于为脚本提供一种基本形式的“符号调试器”。

要逐步执行一个不调用其他脚本的脚本，请将选项 `single_step` 从默认值 0 重置为 1。例如，以下是开始逐步执行图 13-5 中脚本的方式：

```ampl
ampl: option single_step 1;
ampl: commands steelT.sa7;
steelT.sa7:2(18) data <2>ampl:
```

表达式 `steelT.sa7:2(18)` 给出了 AMPL 在处理脚本过程中暂停时的文件名、行号和字符号。其后跟随的是即将执行的下一条命令（data）的开头。下一行将返回到 `ampl:` 提示符。前面的 `<2>` 表示输入嵌套的层级；“<2”表示当前执行位于通过原始输入流发出的 `commands` 语句的作用域内。

此时，你可以使用 `step` 命令逐条执行脚本中的命令。单独输入 `step` 可执行一条命令，

```ampl
<2>ampl: step
steelT.sa7:4(36) option <2>ampl: step
steelT.sa7:5(66) option <2>ampl: step
steelT.sa7:11(167) let <2>ampl:
```

如果 `step` 后面跟一个数字，则会执行该数量的命令。除了与模型声明相关的命令（如此例中的 `model` 和 `param`）外，每条命令都会被计数。

每次执行 `step` 后，你都会返回到 AMPL 提示符。你可以继续逐步执行，直到脚本结束，但在某个时刻你可能希望显示一些值以检查脚本是否正常工作。以下序列捕捉到了对偶值发生变化的一个位置：

$< 2 >$ ampl: display avail[3], Time[3].dual, previous_dual;  
avail[3] $=$ 22  
Time[3].dual $=$ 3620  
previous_dual $=$ 3620  

$< 2 >$ ampl: step steelT.sa7:17(317) continue  
$< 2 >$ ampl: step steelT.sa7:15(237) let  
$< 2 >$ ampl: step steelT.sa7:16(270) solve  
$< 2 >$ ampl: step steelT.sa7:17(280) if  
$< 2 >$ ampl: step steelT.sa7:19(331) let  
$< 2 >$ ampl: display avail[3], Time[3].dual, previous_dual;  
avail[3] $=$ 23  
Time[3].dual $=$ 3500  
previous_dual $=$ 3620  

$< 2 >$ ampl:

在 单步执行 模式下，可以 输入 任意 一系列 AMPL 命令。每 执行 完 一个 命令 后，会 重新 出现 $< 2 > \text{ampl}$ 提示符，提醒 你 仍 处于 该 模式 中， 并 可 使用 step 命令 继续 执行 脚本。

为 了 帮助 你 逐步 执行 冗长 的 复合 命令 （如 for、repeat 或 if），AMPL 提供 了 几种 替代 step 的 命令。next 命令 会 跳过 整个 复合 命令，而 不是 进入 其中。如果 我们 从头 开始 执行，每次 执行 next 都 会 使 下 一条 语句 被 执行；对 于 repeat 命令 来说，整个 命令 会 被 执行 完毕，仅 在 第 28 行 的 display 命令 处 再次 停下：

ampl: option single_step 1;  
ampl: commands steelT.sa7;  
steelT.sa7:2(18) data ...  
$< 2 >$ ampl: next  
steelT.sa7:4(36) option ...  
$< 2 >$ ampl: next  
steelT.sa7:5(66) option ...  
$< 2 >$ ampl: next  
steelT.sa7:11(167) let ...  
$< 2 >$ ampl: next  
steelT.sa7:14(225) repeat ...  
$< 2 >$ ampl: next  
steelT.sa7:28(539) display ...  
$< 2 >$ ampl:

输入 next $n$ 可以 按 这种 方式 跳过 $n$ 条 命令。

skip 和 skip $n$ 命令 的 工作 方式 类似 于 step 和 step $n$，不同 之处 在于 它们 会 跳过 脚本 中 的 下 1 条 或 $n$ 条 命令，而 不会 执行 它们。

# 13.7 字符串 操作

能够 处理 任意 字符 集合 是 AMPL 脚本 的 一个 关键 优势。这里 我们 介绍 字符串 连接 运算符 以及 几种 用于 构建 字符串 表达式 的 函数，这些 表达式 可以 在 AMPL 语句 中 任何 集合 成员 可以 出现 的 地方 使用。更多 细节 请 参见 附录 A.4.2，表 A-4 总结 了 所有 字符串 函数。

我们 还 展示 了 如何 使用 字符串 表达式 来 指定 用于 其他 目的 的 字符 序列，而 不 仅仅 是 作为 集合 成员。这 一 特性 允许 AMPL 脚本 例如 在 循环 的 每次 迭代 中 根据 循环 索引 集合 的 内容 信息 写入 不同 的 文件 或 设置 不同 的 选项 值。

# 字符串 函数 与 运算符

连接 运算符 & 接受 两个 字符串 作为 操作数，并 返回 一个 由 左 操作数 和 右 操作数 依次 组成 的 字符串。例如，给定 由 diet.mod 和 diet2.dat（图 2-1 和 2-3）定义 的 集合 NUTR 和 FOOD，你 可以 使用 连接 操作 来 定义 一个 集合 NUTR_FOOD，其 成员 表示 营养素-食物 对：

ampl: model diet.mod;  
ampl: data diet2.dat;  
ampl: display NUTR, FOOD;  

set NUTR := A B1 B2 C NA CAL;  
set FOOD := BEEF CHK FISH HAM MCH MTL SPG TUR;  

ampl: set NUTR_FOOD := setof {i in NUTR, j in FOOD} i & "_" & j;  
ampl: display NUTR_FOOD;

set NUTR_FOOD := A_BEEF B1_BEEF B2_BEEF C_BEEF NA_BEEF CAL_BEEF  
                 A_CHK B1_CHK B2_CHK C_CHK NA_CHK CAL_CHK  
                 A_FISH B1_FISH B2_FISH C_FISH NA_FISH CAL_FISH  
                 A_HAM B1_HAM B2_HAM C_HAM NA_HAM CAL_HAM  
                 A_MCH B1_MCH B2_MCH C_MCH NA_MCH CAL_MCH  
                 A_MTL B1_MTL B2_MTL C_MTL NA_MTL CAL_MTL  
                 A_SPG B1_SPG B2_SPG C_SPG NA_SPG CAL_SPG  
                 A_TUR B1_TUR B2_TUR C_TUR NA_TUR CAL_TUR;  

这并不是一个通常需要定义的集合，但如果必须读取包含类似 "B2_BEEF" 字符串的数据时，它可能是有用的。

作为 & 运算符参数出现的数字会被自动转换为字符串。例如，在一个多周模型中，可以通过如下声明创建一组通用名称的周期 WEEK1、WEEK2 等：

param T integer > 1;  
set WEEKS ordered = setof {t in 1..T} "WEEK" & t;  

& 的数值操作数总是按照 A.16 节中定义的全精度（或等效地，$\frac{9}{10} = 09$ 格式）进行转换。因此，这种转换在连接数值常量或遍历整数集或常量索引时会产生预期结果，如我们的示例所示。然而，对计算出的分数值进行全精度转换有时可能会产生令人惊讶的结果。前述示例的一个变体似乎会创建成员 WEEK0.1、WEEK0.2 等组成的集合：

param T integer > 1;  
set WEEKS ordered = setof {t in 1..T} "WEEK" & 0.1 * t;  

但实际生成的集合却不同：

ampl: let T := 4;  
ampl: display WEEKS;  

set WEEKS := WEEK0.1 WEEK0.30000000000000004 WEEK0.2 WEEK0.4;  

因为 0.1 在二进制表示中无法精确存储，所以在“全”精度下，$0.1 \times 3$ 的值与 0.3 略有不同。这种行为难以预测，但可以通过使用 sprintf 指定显式转换来避免。sprintf 函数以与 printf（见第 A.16 节）相同的方式执行格式转换，区别在于结果格式化字符串不会发送到输出流，而是成为函数的返回值。对于我们的示例来说，"WEEK" & 0.1*t 可以替换为 sprintf("WEEK%3.1f", 0.1*t)。

length 字符串函数接受一个字符串作为参数，并返回其中字符的数量。match 函数接受两个字符串参数，返回第二个字符串在第一个字符串中作为子串首次出现的位置，如果第二个字符串从未出现在第一个字符串中，则返回零。例如：

ampl: display {j in FOOD} (length(j), match(j, "H"));  

: length(j) match(j, 'H') :=  
BEEF 4 0  
CHK 3 2  
FISH 4 4  
HAM 3 1  
MCH 3 3  
MTL 3 0  
SPG 3 0  
TUR 3 0

substr 函数接受一个字符串和一个或两个整数作为参数。它返回第一个参数的一个子字符串，该子字符串从第二个参数指定的位置开始；其长度由第三个参数指定，或者如果没有提供第三个参数，则延伸到字符串的末尾。如果第二个参数大于第一个参数的长度，或者第三个参数小于 1，则返回一个空字符串。

作为结合多个此类函数的示例，假设您想使用 diet.mod 中的模型，但要以如下表格形式提供营养成分 (nutrient) 数据：

param: NUTR_FOOD: amt_nutr :=  
A_BEEF 60  
B1_BEEF 10  
CAL_BEEF 295  
CAL_CHK 770  

那么除了模型中使用的参数 amt 的声明之外，

```ampl
set NUTR;  
set FOOD;  
param amt {NUTR, FOOD} >= 0;
```

您还需要声明一个集合和一个参数来保存来自“非标准”表格的数据：

```ampl
set NUTR_FOOD;  
param amt_nutr {NUTR_FOOD} >= 0;
```

为了使用该模型，您需要编写某种赋值语句，将集合 NUTR_FOOD 和参数 amt_nutr 中的数据导入到集合 NUTR 和 FOOD 以及参数 amt 中。一种解决方案是首先提取集合，然后转换参数：

```ampl
set NUTR = setof {ij in NUTR_FOOD} substr(ij, 1, match(ij, "_") - 1);  
set FOOD = setof {ij in NUTR_FOOD} substr(ij, match(ij, "_") + 1);  
param amt {i in NUTR, j in FOOD} = amt_nutr[i & "_" & j];
```

作为替代方案，您可以使用如下脚本同时提取集合和参数：

```ampl
param iNUTR symbolic;  
param jFOOD symbolic;  
param upos > 0;  

let NUTR := {};  
let FOOD := {};  

for {ij in NUTR_FOOD} {  
    let upos := match(ij, "_");  
    let iNUTR := substr(ij, 1, upos - 1);  
    let jFOOD := substr(ij, upos + 1);  
    let NUTR := NUTR union {iNUTR};  
    let FOOD := FOOD union {jFOOD};  
    let amt[iNUTR, jFOOD] := amt_nutr[ij];  
}
```

在任一替代方案中，诸如 NUTR_FOOD 的成员中缺少 "_" 之类的错误最终会通过错误消息提示。

AMPL 提供了另外两个函数 sub 和 gsub，它们像 match 一样在第一个参数中查找第二个参数，然后将找到的第一个匹配项（sub）或所有匹配项（gsub）替换为第三个参数。这三个函数的第二个参数实际上是一个正则表达式；如果它包含某些特殊字符，则会被解释为可能匹配多个子字符串的模式。例如，模式 $\mathrm{"B}[0-9]+\_ "$ 匹配任何由 B 开头、后跟一个或多个数字和下划线且出现在字符串开头的子字符串。这些特性的详细信息见 A.4.2 节。

# AMPL 命令中的字符串表达式

字符串表达式可以在多种情况下代替字面字符串出现：在作为命令一部分的文件名中，包括模型、数据和命令文件名，以及在 `$>$` 或 `$>>$` 后指定输出重定向的文件名中；在通过 option 命令赋给 AMPL 选项的值中；以及在 table 语句中指定的字符串列表、数据库行名和列名中。在所有这些情况下，都必须通过将字符串表达式用括号括起来来标识它。

以下是一个涉及文件名的示例。此脚本使用字符串表达式为 data 语句和 display 语句的输出重定向指定文件：

```ampl
model diet.mod;  
set CASES = 1..3;  

for {j in CASES} {  
    reset data;  
    data ("diet" & j & ".dat");  
    solve;  
    display Buy > ("diet" & j & ".out");  
}
```

结果是使用一系列不同的数据文件 `diet1.dat`、`diet2.dat` 和 `diet3.dat` 来求解 `diet.mod`，并将解保存到文件 `diet1.out`、`diet2.out` 和 `diet3.out` 中。索引 `j` 的值会如前所述自动从数字转换为字符串。

以下脚本使用字符串表达式来指定选项 `cplex_options` 的值，该选项包含传递给 CPLEX 求解器的指令：

```
model sched.mod;
data sched.dat;
option solver cplex;
set DIR1 = {"primal", "dual"};
set DIR2 = {"primalopt", "dualopt"};
for {i in DIR1, j in DIR2} {
    option cplex_options (i & " " & j);
    solve;
}
```

此脚本中的循环将同一个问题求解四次，每次使用 directives `primal` 和 `dual` 与 directives `primalopt` 和 `dualopt` 的不同配对。

关于在 `table` 语句中使用字符串表达式处理多个数据库文件、表或列的示例，请参见第 10.6 节。

# 14

# 与求解器的交互

本章更详细地描述了 AMPL 用于控制和调整发送给求解器的问题，以及提取和解释求解器返回信息的各种机制。其中最重要的一个是 presolve 阶段，它执行简化和变换，通常可以减少求解器实际看到的问题规模；这是 14.1 节的主题。模型组件上的后缀允许与高级求解器交换各种有用信息，如 14.2 和 14.3 节所述。命名问题使 AMPL 脚本能够在单个模型内管理多个问题实例，并执行在截然不同的模型之间交替的迭代过程，如 14.4 和 14.5 节所示。

# 14.1 Presolve 阶段

AMPL 的 presolve 阶段尝试在问题实例生成之后、发送给求解器之前对其进行简化。当执行 `solve` 命令或生成实例的其他命令时，它会自动运行，如 A.18.1 节所述。在返回解之后，presolve 所做的任何简化都会被逆转，因此你可以从原始问题的角度查看解。因此，presolve 通常在后台静默进行。只有当你将选项 `show_stats` 从默认值 0 更改为 1 时，才会报告其效果：

```
ampl: model steelT.mod; data steelT.dat;
ampl: option show_stats 1;
ampl: solve;
Presolve eliminates 2 constraints and 2 variables.
Adjusted problem:
24 variables, all linear
12 constraints, all linear; 38 nonzeros
1 linear objective; 24 nonzeros.
MINOS 5.5: optimal solution found.
15 iterations, objective 515033
```

你可以通过测试确定哪些变量和约束被 presolve 消除，如 14.2 节所述，查看哪些变量或约束的状态为 `pre`：

```
ampl: print {j in 1.._nvars:
       ampl? _var[j].status = "pre"}: _varname[j];
Inv['bands', 0]
Inv['coils', 0]
ampl: print {i in 1.._ncons:
       ampl? _con[i].status = "pre"}: _conname[i];
Init_Inv['bands']
Init_Inv['coils']
```

然后你可以使用 `show` 和 `display` 来检查被消除的组件。

在本节中，我们介绍 presolve 阶段的操作以及从 AMPL 控制它的选项。接着我们解释当 presolve 检测到不存在可行解时会发生什么。然而，我们不会试图解释整个 presolve 算法；本章末尾的参考文献之一包含完整的描述。

# presolve 阶段的活动

为了生成一个问题实例，AMPL 首先为每个变量分配其 var 声明中指定的边界，或者在未给出下界或上界时分配特殊边界 $-\infty$ 和 $\infty$。然后 presolve 阶段尝试使用这些边界以及线性约束来推导出仍然满足问题所有可行解的更紧边界。同时，presolve 尝试使用更紧的边界来检测可以固定的变量和可以删除的约束。

Presolve 最初寻找只有一个变量的约束。这类等式固定一个变量，然后可以从问题中删除该变量。不等式为变量指定一个边界，该边界可以合并到现有边界中。在上面所示的 steelT.mod（图 4-4）示例中，presolve 消除了从声明 `subject to Initial {p in PROD}: Inv[p,0] = inv0[p];` 生成的两个约束，以及由这些约束固定的两个变量。

Presolve 继续寻找可以通过当前边界证明冗余的约束。从 dietu.mod（图 5-1）中消除的约束提供了一个示例：

ampl: model dietu.mod; data dietu.dat;  
ampl: option show_state 1;  
ampl: solve;  
预处理消除了 3 个约束。  
调整后的问题：  
8 个变量，均为线性  
5 个约束，均为线性；39 个非零元素  
1 个线性目标函数；8 个非零元素。  
MINOS 5.5：找到最优解。  
5 次迭代，目标函数值 74.27382022  

ampl: print {i in 1.._ncons:  
ampl? _con[i].status = "pre"}: _conname[i];  
Diet_Min['B1']  
Diet_Min['B2']  
Diet_Max['A']  

进一步调查发现，约束 Diet_Min['B1'] 是冗余的，因为它是由以下语句生成的：  
subject to Diet_Min {i in MINREQ}:  
sum {j in FOOD} amt[i,j] * Buy[j] >= n_min[i];  

而数据中 n_min['B1'] 等于零。显然，任何变量组合都满足该条件，因为所有变量都有非负下界。一个不那么平凡的例子是 Diet_Max['A']，它由以下语句生成：  
subject to Diet_Max {i in MAXREQ}:  
sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i];  

通过将每个变量设置为其在该约束左侧的上界，我们可以得到任何解可能提供的该营养素总量的上界。特别地，对于营养素 A：  

ampl: display sum {j in FOOD} amt['A',j] * f_max[j];  
sum{j in FOOD} amt['A',j]*f_max[j] = 2860  

由于数据文件中 n_max['A'] 为 20000，因此这是另一个不可能被变量违反的约束。  

完成这些测试后，预处理的第一阶段就结束了。其余部分包括对问题进行一系列遍历，每次尝试从当前边界和线性约束中推导出更紧的变量边界。我们在此仅展示一个来自 multi.mod 和 multi.dat（图 4-1 和 4-2）生成的问题实例的结果示例：

ampl: model multi.mod;  
ampl: data multi.dat;  
ampl: option show_stats 1;  
ampl: solve;  
预处理消除了 7 个约束和 3 个变量。  
调整后的问题：  
60 个变量，均为线性  
44 个约束，均为线性；165 个非零元素  
1 个线性目标函数；60 个非零元素。  
MINOS 5.5：找到最优解。  
41 次迭代，目标函数值 199500  

ampl: print {j in 1.._nvars:  
ampl? _var.status[j] = "pre"}: _varname[j];  
Trans['GARY', 'LAN', 'plate']  
Trans['CLEV', 'LAN', 'plate']  
Trans['PITT', 'LAN', 'plate']  

ampl: print {i in 1.._ncons:  
ampl? _con[i].status = "pre"}: _conname[i];  
Demand['LAN','plate']  
Multi['GARY','LAN']  
Multi['GARY','WIN']  
Multi['CLEV','LAN']  
Multi['CLEV','WIN']  
Multi['PITT','LAN']  
Multi['PITT','WIN']  

我们可以通过展开被消除的需求约束来了解一些简化的原因：  

ampl: expand Demand['LAN','plate'];  
subject to Demand['LAN','plate']:  
Trans['GARY','LAN','plate'] + Trans['CLEV','LAN','plate'] +  
Trans['PITT','LAN','plate'] = 0;

因为在数据中 demand['LAN','plate'] 为零，该 约束 (constraint) 强制三个非负 变量 (variable) 的和为零，因此在任何解中这三个变量的上限都必须为零。由于它们已经有下限零，因此可以固定它们并删除该约束。其他被删除的约束都类似如下形式：

ampl: expand Multi['GARY','LAN'];
subject to Multi['GARY','LAN']:
Trans['GARY','LAN','bands'] + Trans['GARY','LAN','coils'] +
Trans['GARY','LAN','plate'] <= 625;

这些约束可以被删除，因为左侧变量的上限之和小于 625。然而，这些变量在问题中最初并未给定上限。相反，这些上限是由 presolve 的第二部分推导出来的。对于这个简单问题，不难看出推导出的界限是如何产生的：沿任何一条链接运输的任何产品的数量不能超过该产品在链接目的地的需求量。对于目的地 LAN 和 WIN，三种产品的总需求小于从任何起点出发的运输总量上限 625，因此总运输量约束是多余的。

# 控制 presolve 的效果

对于更复杂的问题，presolve 对变量和约束的删除可能不容易解释，但它们可能代表问题的相当大一部分。因此，解决问题实例所需的时间和内存可能会大幅减少。在极少数情况下，当存在多个最优解时，presolve 也可能显著影响变量的最优值，或干扰求解器软件中内置的其他预处理例程。要完全关闭 presolve，请将选项 presolve 设置为 0；要仅关闭第二部分，请将其设置为 1。该选项的更高值表示 presolve 第二部分中执行的最大遍数；默认值为 10。

在 presolve 之后，AMPL 会保存两组变量的下限和上限：一组反映由 presolve 删除的约束所隐含的界限收紧，另一组反映由 presolve 无法删除的约束进一步推导出的界限收紧。使用任一组界限，问题都有相同的解，但根据所使用的优化方法和问题的具体情况，整体求解 时间 (time) 可能会因使用其中一组界限而更低。

对于连续变量，通常 AMPL 会向求解器传递第一组界限，但您可以通过将选项 var_bounds 从默认值 1 更改为 2 来指示它传递第二组界限。当应用活动集方法（如 单纯形法 (simplex method)）时，第二组界限往往会产生更多退化变量，从而导致更多退化迭代，可能阻碍求解进程。

对于整数变量，AMPL 会将任何小数形式的下界 (lower bound) 向上取整为下一个更高的整数，而将任何小数形式的上界 (upper bound) 向下取整为下一个更低的整数。然而，由于有限精度计算的不准确性，某个边界值的计算结果可能与整数值略有差异。例如，一个本应为 7 的下界，可能会被计算为 7.00000000001，在这种情况下，你并不希望该边界被向上取整为 8！为了应对这种可能性，AMPL 在取整之前，会从每个下界中减去选项 presolve_inteps 的值，并将其加到每个上界中。如果将该设置增大到选项 presolve_intepsmax 的值会对任何变量的取整边界产生影响，AMPL 将发出警告。presolve_inteps 和 presolve_intepsmax 的默认值分别为 1.0e-6 和 1.0e-5。

你可以通过后缀 .lb1 和 .ub1 查看第一组 presolve 边界，通过 .lb2 和 .ub2 查看第二组边界。原始边界（只有在关闭 presolve 时才会发送给求解器）表示为 .lb0 和 .ub0。后缀 .lb 和 .ub 则根据当前的 presolve 和 var_bounds 选项值，给出当前要传递给求解器的边界值。

# 在 presolve 中检测不可行性

如果 presolve 确定某个变量的下界大于其上界，则不存在满足所有边界和其他约束的解，此时将打印一条错误信息。例如，如果我们本意是将 market["bands"] 改为 5000，却错误地改成了 500，steel3.mod（图 1-5a）将会发生如下情况：

```ampl
ampl: model steel3.mod;
ampl: data steel3.dat;
ampl: let market["bands"] := 500;
ampl: solve;
inconsistent bounds for var Make['bands']:
    lower bound = 1000
    upper bound = 500;
    difference = 500
```

这是一个简单的情况，因为变量 Make["bands"] 的上界明显被降低到了下界之下。presolve 更复杂的测试还可以发现并非由单个变量引起的不可行性。例如，考虑模型中的如下约束：

subject to Time: sum {p in PROD} 1/rate[p] * Make[p] <= avail;

如果我们把 avail 的值减少到 13 小时，presolve 会推断出该约束不可能被满足：

```ampl
ampl: let market["bands"] := 5000;
ampl: let avail := 13;
ampl: solve;
presolve: 约束 Time 无法满足：
    body <= 13 不能 >= 13.2589；
    差值 = -0.258929
```

约束 Time 的“主体 (body)”是 sum {p in PROD} 1/rate[p] * Make[p]，即包含变量的部分（见第 12.5 节）。因此，在我们设定的 avail 值下，该约束对主体表达式的值施加了 13 的上界。另一方面，如果我们将主体表达式中的每个变量都设为其下界，则可以得到在任何可行解中主体表达式的下界。

```ampl
ampl: display sum {p in PROD} 1/rate[p] * Make[p].lb2;
sum[p in PROD] 1/rate[p] * (Make[p].lb2) = 13.2589
```

因此，预处理 (presolve) 报告主体 $< = 13$ 不能 $> = 13.2589$，即表示主体的上界与下界冲突，意味着没有任何解能满足所有问题的边界和约束。

预处理报告了约束 Time 的两个边界之间的差值为 -0.258929（保留六位小数）。因此，在这种情况下我们可以推测，大约 13.258929 是使问题存在可行解的最小 avail 值，我们可以通过实验验证这一点：

ampl: let avail := 13.258929;  
ampl: solve;  
MINDS 5.5: 找到最优解。  
0 次迭代，目标函数值 61750.00214  

然而，如果我们把 avail 稍微调低一点，又会得到不可行的信息：

ampl: let avail := 13.258928;  
ampl: solve;  
presolve: 约束 Time 无法满足：  
body <= 13.2589 不能 >= 13.2589;  
差值 = -5.71429e-07  
设置 $presolve_eps >= 6.86e-07$ 可能会有帮助。

虽然这里的下界与上界在六位小数内相同，但在完整精度下它大于上界，正如差值为负所表明的那样。

在这种情况下第二次输入 solve 命令会让 AMPL 忽略预处理的结果，并将看似矛盾的推导边界发送给求解器：

ampl: solve;  
MINOS 5.5: 找到最优解。  
0 次迭代，目标函数值 61749.99714  
ampl: option display_precision 10;  
ampl: display commit, Make;

:     commit       Make  
bands   1000     999.9998857  
coils    500     500  
plate    750     750  

MINOS 声称找到了一个最优解，尽管 Make["bands"] 略小于其下界 commit["bands"]！这里 MINOS 应用了内部容差，允许忽略小的不可行性；AMPL/MINOS 文档解释了该容差的工作原理以及如何更改它。每个求解器都以自己的方式应用可行性容差，因此不同求解器给出不同结果并不令人意外：

ampl: option solver cplex;  
ampl: option send_statuses 0;  
ampl: solve;  
CPLEX 8.0.0: 列 'xl' 的界不可行。  
不可行问题。  
1 次对偶单纯形法迭代（第 I 阶段 0 次）

在这里，CPLEX 应用了自身的预处理程序，并检测到了 AMPL 所发现的相同不可行性。（你可能会看到一些关于名为 dumbdd 的“后缀”的附加行；这涉及一个无界方向，你可以通过 AMPL 的求解器定义的后缀功能在第 14.3 节中描述的方法获取。）

当某个 变量 (variable) 或 约束体 (constraint body) 的隐含 下界 (lower bound) 和 上界 (upper bound) 相等时（至少在实际应用中如此），就会出现这种情况。由于计算中的不精确性，下界可能略微大于上界，导致 AMPL 的 预处理 (presolve) 报告问题不可行。为避免这种困难，你可以将选项 presolve_eps 从默认值 0 重置为某个小的正值。当下界与上界之间的差异小于该值时，这些差异将被忽略。如果将当前 presolve_eps 值增加到不超过 presolve_epsmax 的值会改变 presolve 对问题的处理方式，则 presolve 会显示一条相关消息，例如：

Setting \$presolve_eps >= 6.86e-07 might help.

如上例所示。选项 presolve_eps 的默认值为零，presolve_epsmax 的默认值为 1.0e-5。

另一种相关的情况是，计算中的不精确性导致某个变量或约束体的隐含下界略低于其隐含上界。此时虽然未检测到不可行性，但近乎相等的边界可能使 求解器 (solver) 的工作变得异常困难。因此，当变量或约束体的上界减去下界为正值但小于选项 presolve_fixeps 的值时，该变量或约束体将被固定在两个边界的平均值上。如果将 presolve_fixeps 的值增加到最多不超过 presolve_fixepsmax 的值会改变 presolve 的结果，则会显示一条相关消息。

预处理 (presolve) 显示的独立消息数量被限制为 presolve_warnings 的值，默认为 5。将选项 show_stats 增加到 2 可能会显示一些关于 presolve 运行的额外信息，包括对结果产生影响的迭代次数，以及 presolve_eps 和 presolve_inteps 需要增加或减少到什么值才能产生影响。

# 14.2 从 求解器 (solver) 中获取结果

除了解和相关的数值外，获取 solve 命令结果的某些符号信息也很有用。例如，在 AMPL 命令脚本中，你可能希望测试最近一次 solve 是否遇到了无界或不可行的问题。或者，在使用 单纯形法 (simplex method) 解决一个 线性规划 (linear program) 后，你可能希望利用 最优基划分 (optimal basis partition) 为解决相关问题提供良好的起点。AMPL-求解器接口允许求解器返回这些及相关类型的状态信息，供你检查和使用。

# 求解结果

求解器完成其工作是因为它已识别出一个 最优解 (optimal solution) 或遇到了某些其他终止条件。除了变量的值之外，求解器还可以设置两个内置的 AMPL 参数和一个 AMPL 选项，以提供有关优化过程结果的信息：

```python
ampl: model diet.mod;
ampl: data diet2.dat;
ampl: display solve_result_num, solve_result;
solve_result_num = -1
solve_result = '?'
ampl: solve;
MINOS 5.5: 不可行问题。
9 次迭代
ampl: display solve_result_num, solve_result;
solve_result_num = 200
solve_result = infeasible
```

ampl: option solve_result_table;
option solve_result_table '\
0 solved\
100 solved?\
200 infeasible\
300 unbounded\
400 limit\
500 failure\
';

在 AMPL 会话开始时，`solve_result_num` 为 -1，`solve_result` 为 '?'。然而，每个 `solve` 命令都会重置这些参数，以便它们描述求解器运行结束时的状态：`solve_result_num` 用数字表示，`solve_result` 用字符串表示。`solve_result_table` 选项列出了可能的组合，其解释如下：

`solve_result` 值范围 数字 字符串解释  
0-99 solved 找到最优解  
100-199 solved? 指示最优解，但可能有错误  
200-299 infeasible 约束无法满足  
300-399 unbounded 目标函数可以无限制地改善  
400-499 limit 被设定的限制（如迭代次数）停止  
500-599 failure 被求解器中的错误条件停止  

通常，这种状态信息用于脚本中，可以测试它以区分必须以不同方式处理的情况。例如，图 14-1 描述了一个 AMPL 脚本，用于饮食模型，该脚本读取营养成分的名称（从标准输入读取，使用文件名 -，如第 9.5 节所述）、饮食中该营养成分的初始上限以及减少上限的步长。循环持续运行，直到上限减少到问题变得不可行的点，此时打印适当的消息和之前找到的解表。一次典型的运行如下所示：

```
ampl: commands diet.run; <1>
ampl? NA <1>
ampl? 60000 <1>
ampl? 3000
-------------------------------- infeasible at 48000 --------------------------------
:     N_obj      N_dual      :=
51000  115.625    -0.0021977
54000  109.42     -0.00178981
57000  104.05     -0.00178981
60000  101.013     7.03757e-19
;
```

这里钠（数据中的 NA）的上限从 60000 以 3000 为步长逐步减少，直到问题在上限为 48000 时变得不可行。

`diet.run` 中测试不可行性的关键语句是：

```ampl
model diet.mod;
data diet2.dat;
param N symbolic in NUTR;
param nstart > 0;
param nstep > 0;
read N, nstart, nstep <- ; # 交互式读取数据
set N_MAX default {};
param N_obj {N_MAX};
param N_dual {N_MAX};
option solver_msg 0;
for {i in nstart..0 by -nstep} {
    let n_max[N] := i;
    solve;
    if solve_result = "infeasible" then {
        printf "------------------------------------------------------------------------------------------------------------------------";
        break;
    }
    let N_MAX := N_MAX union {i};
    let N_obj[i] := Total_Cost;
    let N_dual[i] := Diet[N].dual;
}
display N_obj, N_dual;
```

图 14-1：带有不可行性测试的敏感性分析 (diet.run)。

if solve_result $=$ "infeasible" then { printf "------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------......" break; }

`if` 条件也可以等价地写成 $200 \leq$ `solve_result_num` $< 300$。通常你会希望避免使用这种替代写法，因为它会让脚本变得晦涩难懂。然而，在需要对不同的求解器 (solver) 终止条件进行精细区分时，这种方式偶尔会很有用。例如，以下是 CPLEX 求解器针对不同最优性条件设置的一些值：

`solve_result_num` 终止时的消息  
0 最优解  
1 原始问题具有无界最优面  
2 最优整数解  
3 在 mipgap 或 absmipgap 范围内的最优整数解  

在所有这些情况下，`solve_result` 的值都是 `"solved"`，但如果你需要在它们之间进行区分，可以测试 `solve_result_num`。`solve_result_num` 的解释完全取决于求解器；你必须查阅特定求解器的文档，以了解它返回哪些值以及它们的含义。

AMPL 的求解器接口会显示几行类似以下内容的信息：

```
MINOS 5.5: infeasible problem. 9 iterations
```

用于总结已完成的求解过程。如果你运行的脚本频繁执行 `solve` 命令，这些消息可能会产生大量输出；你可以通过将选项 `solver_msg` 设置为 0 来抑制这些消息的显示。内置的符号参数 `solve_message` 始终包含最近一次求解器返回的消息，即使消息显示已被关闭。你可以显示此参数以验证其值：

```pythonmpl
display solve_message;
solve_message = 'MINOS 5.5: infeasible problem. 9 iterations'
```

由于 `solve_message` 是一个符号参数，其值是一个字符串。它在脚本中最有用，你可以使用字符串函数（第 13.7 节）来测试消息中是否包含最优性或其他结果的指示信息。

例如，`diet.run` 中的测试也可以写成：

```pythonmpl
if match(solve_message, "infeasible") > 0 then {
```

然而，由于返回消息在不同求解器之间有所不同，因此在大多数情况下，对 `solve_result` 的测试会更简单且更少依赖于特定求解器。

只有在 AMPL 调用求解器成功的情况下，才能返回如上所述的求解结果。调用可能失败，因为操作系统无法找到或执行指定的求解器，或者由于某些低级错误导致求解器无法尝试或完成优化。典型原因包括拼写错误的求解器名称、求解器安装或授权不当、资源不足，以及求解器进程因执行错误（“核心转储”）或键盘中断（“break”）而终止。在这些情况下，`solve` 后面的错误消息是由操作系统而不是求解器生成的，您可能需要咨询系统专家来追踪问题。例如，类似 `can't open at 8871.nl` 的消息通常表示 AMPL 无法写入临时文件；它可能试图写入已满的磁盘，或写入您没有写入权限的目录（文件夹）。（临时文件的目录在选项 `TMPDIR` 中指定。）

内置参数 `solve_exitcode` 记录了最近一次求解器调用的成功或失败。初始值为 -1，当调用成功时重置为 0，否则重置为某个与系统相关的非零值：

```python
ampl: reset;
ampl: display solve_exitcode;
solve_exitcode = -1

ampl: model diet.mod;
ampl: data diet2.dat;
ampl: option solver xplex;
ampl: solve;
Cannot invoke xplex: No such file or directory
```

ampl: display solve_exitcode;  
solve_exitcode = 1024  

ampl: display solve_result, solve_result_num;  
solve_result = '?'  
solve_result_num = -1  

在这里，由于求解器名称拼写错误 xplex，调用失败，反映在正值的 solve_exitcode 值中。状态参数 solve_result 和 solve_result_num 也被重置为其初始值 '?' 和 -1。

如果 solve_exitcode 超过选项 solve_exitcode_max 中的值，则 AMPL 会中止当前正在执行的复合语句（include、commands、repeat、for、if）。solve_exitcode_max 的默认值为 0，因此当求解器调用失败时，AMPL 通常会中止复合语句。将 solve_exitcode_max 设置为较高值的脚本可以测试 solve_exitcode 的值，但通常其解释在不同操作系统或求解器之间并不一致。

# 目标函数和问题的求解器状态

有时为了方便起见，希望能够引用在特定目标函数最近一次优化时获得的求解结果。为此，AMPL 为每个内置求解结果参数关联了一个“状态”后缀：

内置参数 后缀  
solve_result .result  
solve_result_num .result_num  
solve_message .message  
solve_exitcode .exitcode  

附加在目标函数名称后，该后缀表示在该目标函数为当前目标的最近一次求解中，相应内置参数的值。

作为示例，我们再次考虑第 8.3 节中为指派模型定义的多个目标函数：

最小化 总成本：sum {i in ORIG, j in DEST} cost[i,j] * Trans[i,j]；  
最小化 Pref_of {i in ORIG}；sum {j in DEST} cost[i,j] * Trans[i,j]；

在最小化这三个目标中的三个之后，我们可以查看所有目标的求解状态值：

ampl: model transp4.mod; data assign.dat; solve;  
CPLEX 8.0.0：最优解；目标值 28 24 次对偶单纯形迭代（第 I 阶段 0 次）  
目标 = 总成本

mpl: objective Pref_of('Coullard');  
appl: solve;  
CPLEX 8.0.0：最优解；目标值 1 3 次单纯形迭代（第 I 阶段 0 次）  
appl: objective Pref_of('Hazen');  
appl: solve;  
CPLEX 8.0.0：最优解；目标值 1 5 次单纯形迭代（第 I 阶段 0 次）  
appl: display Total_Cost.result, Pref_of.result;  
Total_Cost.result = solved  
Pref_of.result [*] :=  
Coullard solved  
Daskin '?'  
Hazen solved  
Hopp '?'  
Iravani '?'  
Linetsky '?'  
Mehrotra '?'  
Nelson '?'  
Smilowitz '?'  
Tamhane '?'  
White '?';

对于尚未使用的那些目标，.result 后缀保持不变（在这种情况下为其初始值 '?'）。

这些相同的后缀也可以应用于我们将在本章后面描述其用途的 "problem" 名称。当附加到一个 problem 名称上时，它们指的是该 problem 当前时执行的最近一次优化。

# 变量的求解器状态

除了提供如上所述的优化过程整体状态的返回外，AMPL 还允许求解器为每个变量返回单独的状态。此功能主要用于在线性规划通过单纯形法 (simplex) 或由内点（障碍）方法后接 "crossover" 例程求解后，报告变量的基状态。基状态对于某些非线性求解器（尤其是 MINOS）所返回的解也相关，这些求解器采用了基本解概念的扩展。

除了 AMPL 模型中由 var 语句声明的变量外，求解器还定义与约束关联的 "松弛" 或 "人工" 变量。这些变量的求解器状态也以类似方式定义，如本节后面所述。变量和约束都有一个 "AMPL 状态"，用于区分当前问题中的那些变量和那些已通过预求解或诸如 drop 等命令从问题中移除的变量。AMPL 状态的解释及其与求解器状态的关系在本节末尾讨论。

最优基本解中求解器状态值的主要用途是为下一次优化运行提供良好的起点。当 send_statuses 选项保持其默认值 1 时，它会指示 AMPL 在每次求解时将状态信息与发送给求解器的变量信息一起包含进去。你可以在几乎任何在对问题进行小改动后重新求解的敏感性分析中看到此功能的效果。

例如，考虑当图 6-3 中的多周期生产 (multi-period production) 示例在劳动力可用量 (availability of labor) 增加百分之五后被重复求解时会发生什么。当 send_statuses 选项被设为 0 时，每次运行求解器 (solver) 都会报告大约 18 次对偶单纯形迭代 (dual simplex iterations)：

```ampl: model steelT3.mod;
ampl: data steelT3.dat;
ampl: option send_statuses 0;
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 514521.7143
18 dual simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 537104
19 dual simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 560800.4
19 dual simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 585116.22
17 dual simplex iterations (0 in phase I)
```

然而，如果 send_statuses 保持默认值 1，则只有第一次求解需要 18 次迭代。随后的运行最多只需几次迭代：

```ampl: model steelT3.mod;
ampl: data steelT3.dat;
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 514521.7143
18 dual simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 537104
2 dual simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 560800.4
0 simplex iterations (0 in phase I)
ampl: let {t in 1..T} avail[t] := 1.05 * avail[t];
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 585116.22
1 dual simplex iterations (0 in phase I)
```

第一次之后的每次求解都会自动使用前一次求解中 变量 (variables) 的 基状态 (basis statuses) 来构建一个起点，该起点距离 最优解 (optimal solution) 仅需几次 迭代 (iterations) 即可到达。在第三次求解的情况下，前一个基仍然保持最优；因此，求解器立即确认最优性并报告进行了 0 次迭代。

以下讨论解释了如何在 AMPL 环境中查看、解释和更改变量的状态值。如上所示，您无需了解这些内容即可使用最优基作为起点，但在某些高级情况下，这些功能可能会很有用。

AMPL 通过在变量名后附加 `.sstatus` 来引用变量的 求解器状态 (solver status)。因此，您可以使用 display 打印变量的状态。在会话开始时（或在 reset 之后），当尚未解决任何问题时，所有变量的状态均为 none：

```matlab
ampl: model diet.mod;
ampl: data diet2a.dat;
ampl: display Buy.sstatus;
Buy.sstatus [*] :=
BEEF none    CHK none    FISH none    HAM none
MCH none    MTL none    SPG none    TUR none;
```

在调用 单纯形法 (simplex) 求解器后，同样的 display 命令将列出在最优基本解中变量的状态：

```matlab
ampl: solve;
MINOS 5.5: 找到最优解。
13 次迭代，目标函数值为 118.0594032
ampl: display Buy.sstatus;
Buy.sstatus [*] :=
BEEF bas
CHK low
FISH low
HAM upp
MCH upp
MTL upp
SPG bas
TUR low;
```

其中两个变量 Buy['BEEF'] 和 Buy['SPG'] 的状态为 bas，表示它们在最优基中。有三个变量的状态为 low，另外三个为 upp，分别表示它们是非基变量且处于 下界 (lower bound) 或 上界 (upper bound)。求解器状态值的对照表存储在选项 sstatus_table 中：

```ampl
option sstatus_table;
option sstatus_table '\
0 none 未分配状态\
1 bas 基变量\
2 sup 超基变量\
3 low 非基变量，通常等于下界\
4 upp 非基变量，通常等于上界\
5 equ 非基变量，上下界相等\
6 btw 非基变量，处于上下界之间\
';
```

前两列给出了表示状态值的数字和短字符串。（数字主要用于 AMPL 与求解器之间的通信，尽管你可以通过使用后缀 `.sstatus_num` 代替 `.sstatus` 来访问它们。）第三列是注释。对于许多教科书中单纯形法定义的非基变量，只有 `low` 状态适用；其他非基状态是为大规模实现中更通用的有界变量单纯形法所必需的。像 MINOS 这样的求解器使用 `sup` 状态来处理非线性问题。这是 AMPL 的标准 `sstatus_table`；求解器可能会替换为自己的对照表，在这种情况下，其文档将说明表中各项的含义。

你可以使用 `let` 命令更改变量的状态。当你希望在问题发生小的、明确定义的改变后重新求解时，这个功能有时会很有用。例如在本章后面的章节中，我们使用了一个模式切割模型（图 14-2a），其中包含如下声明：

```ampl
param nPAT integer >= 0;     # 模式数量
set PATTRNS = 1..nPAT;       # 模式集合
var Cut {PATTRNS} integer >= 0;  # 使用每种模式切割的卷数
```

在相关的脚本中（图 14-3），每次主循环执行时 `nPAT` 增加一，导致一个新的变量 `Cut[nPAT]` 被创建。它像所有新变量一样初始求解器状态为 `"none"`，但由于模式生成过程的构造方式，可以保证在扩展后的切割问题重新求解时立即进入基变量。因此我们将其状态设为 `"bas"`：

```ampl
let Cut[nPAT].sstatus := "bas";
```

事实证明，这种改变往往能够减少每次重新优化切割问题时的迭代次数，至少对于某些单纯形法求解器是如此。然而，以这种方式设置少量状态并不能保证减少迭代次数。其成功与否取决于具体的问题和求解器，以及它们与许多复杂因素的相互作用：

- 在问题和状态被修改后，传递给求解器 (solver) 的状态在下一次求解时可能无法正确定义一个基本解 (basic solution)。
- 在问题被修改后，除非将选项 `preserve` 设置为零，否则 AMPL 的预处理阶段 (presolve phase) 可能会向求解器发送变量和约束的不同子集。因此，传递给求解器的状态可能不是下一次求解有用起点，也可能无法正确定义一个基本解。

某些求解器（特别是 MINOS）在构建下一次求解的起点时，会使用变量的当前值以及变量的状态（除非将选项 `reset_initial_guesses` 设置为 1）。

每个求解器都有自己的一套方法，在必要时调整从 AMPL 接收到的状态，以生成一个它能够使用的初始基本解。因此，通常需要通过一些实验来确定修改状态的特定策略是否有用。

对于具有多个 var 声明的模型，AMPL 提供的变量通用同义词（第 12.6 节）为获取状态的整体摘要提供了便利方式。例如，在 display 语句中使用类似 `_var`、`_varname` 和 `_var.sstatus` 的表达式，可以轻松指定 steelT3.mod 中所有基本变量及其最优值的表格：

ampl: display {j in 1.._nvars: _var[j].sstatus = "bas"} (_varname[j], _var[j]);  
: _varname[j] _var[j] :=  
1 "Make['bands',1]" 5990  
2 "Make['bands',2]" 6000  
3 "Make['bands',3]" 1400  
4 "Make['bands',4]" 2000  
5 "Make['coils',1]" 1407  
6 "Make['coils',2]" 1400  
7 "Make['coils',3]" 3500  
8 "Make['coils',4]" 4200  
15 "Inv['coils',1]" 1100  
21 "Sell['bands',3]" 1400  
22 "Sell['bands',4]" 2000  
23 "Sell['coils',1]" 307  
;

通过命令 display _varname, _var; 可以生成所有变量的类似列表。

# 约束的求解器状态

单纯形法 (simplex method) 的实现通常会为从 AMPL 接收到的每个约束添加一个变量。每个添加的变量在其相关约束中的系数为 1 或 $-1$，而在所有其他约束中的系数为 0。如果相关约束是一个不等式，则该添加变量用作“松弛变量 (slack)”或“剩余变量 (surplus)”；其边界被选择为能够将不等式转化为等价的方程。如果相关约束本身就是等式，则添加的变量是一个“人工变量 (artificial)”，其上下界均为零。

高效的大型单纯形求解器通过将这些“逻辑变量 (logical variables)”添加到从 AMPL 获得的“结构变量 (structural variables)”中，获得了两个优势：线性规划问题

被转换为一种更简单的形式，其中唯一的不等式是变量的边界，而求解器的初始化 (或称为 "crash") 例程可以被设计为快速找到一个初始基。给定任何初始基，单纯形法的第一阶段会找到一个可行解的基 (如有必要)，第二阶段则找到一个最优解的基；在某些求解器的消息中，这两个阶段是有所区分的：

```python
ampl: model steelP.mod;
ampl: data steelP.dat;
ampl: solve;
CPLEX 8.0.0: optimal solution; objective 139217527
dual simplex iterations (0 in phase I)
```

因此，求解器通常以与结构变量几乎相同的方式处理所有逻辑变量，仅需非常小的调整来处理上下限均为零的情况。一个基本解由所有变量 (结构变量和逻辑变量) 的基状态集合所定义。

为了适应逻辑变量的状态，AMPL 允许求解器返回与约束以及变量相对应的状态值。约束的求解器状态写作约束名称后缀加上 .sstatus，被解释为与该约束相关联的逻辑变量的状态。例如，在我们的饮食模型中，所有约束都是不等式：

```python
subject to Diet {i in NUTR}:
    n_min[i] <= sum {j in FOOD} amt[i,j] * Buy[j] <= n_max[i];
```

逻辑变量是松弛变量，其状态种类与结构变量相同：

```python
ampl: model diet.mod;
ampl: data diet2a.dat;
ampl: option show_stats 1;
ampl: solve;
8 variables, all linear
6 constraints, all linear; 47 nonzeros
1 linear objective; 8 nonzeros.
MINDS 5.5: optimal solution found.
13 iterations, objective 118.0594032
```

```python
ampl: display Buy.sstatus;
Buy.sstatus [+] :=
BEEF  bas
CHK   low
FISH  low
HAM   upp
MCH   upp
MTL   upp
SPG   bas
TUR   low
;

ampl: display Diet.sstatus;
Diet.sstatus [*] :=
A   bas
B1  bas
B2  low
C   bas
CAL bas
NA  upp
```

总共有 六个 基本 变量 (basic variable)，数量 与 六个 约束 (constraint) (集合 NUTR 的 每个 成员 对应 一个) 相等，这 在 基本 解 (basic solution) 中 总是 成立 的。在 我们 的 运输 模型 (transportation model) 中，约束 是 等式：

```python
subject to Supply {i in ORIG}:
    sum {j in DEST} Trans[i,j] = supply[i];

subject to Demand {j in DEST}:
    sum {i in ORIG} Trans[i,j] = demand[j];
```

逻辑 变量 (logical variable) 是 人工 变量 (artificial variable)，当 它们 是 非 基本 变量 (nonbasic variable) 时，状态 为 "equ"。以下是 使用 AMPL 的 通用 约束 同义词 (generic constraint synonym) (类似于 之前 展示 的 变量 同义词 (variable synonym)) 显示 所有 约束 状态 (constraint status) 的 方法：

ampl: model transp.mod;  
ampl: data transp.dat;  
ampl: solve;  
MINOS 5.5: optimal solution found.  
13 iterations, objective 196200  
ampl: display _conname, _con.slack, _con.sstatus;  

: _conname _con.slack _con.sstatus :=  
1 "Supply['GARY']" - 4.54747e- 13 equ  
2 "Supply['CLEV']" 0 equ  
3 "Supply['PITT']" - 4.54747e- 13 equ  
4 "Demand['FRA']" - 6.82121e- 13 bas  
5 "Demand['DET']" 0 equ  
6 "Demand['LAN']" 0 equ  
7 "Demand['WIN']" 0 equ  
8 "Demand['STL']" 0 equ  
9 "Demand['FRE']" 0 equ  
10 "Demand['LAF']" 0 equ  
;

有一个 人工 变量 (artificial variable) 出现 在 最优 基 (optimal basis) 中，即 约束 Demand['FRA'] 对应 的 变量，尽管 其 松弛 值 (slack value) 本质 上 为 零，就 像 任何 可行 解 (feasible solution) 中 所有 的 人工 变量 一样。(事实上，由于 模型 中 方程 存在 线性 依赖 关系 (linear dependence)，这个 问题 的 每个 基 (basis) 中 都 必须 包含 一些 人工 变量。)

# AMPL 状态

只有 那些 AMPL 实际 发送 给 求解器 (solver) 的 变量 (variable)、目标 函数 (objective) 和 约束 (constraint)，在 求解 后 才能 接收 到 求解器 的 状态 (solver status)。因此，为 了 区分 这些 与 在 求解 前 被 移除 的 组件 (component)，AMPL 还 维护 了 一个 单独 的 “AMPL 状态” (AMPL status)。你 可以 像 处理 求解器 状态 一样 处理 AMPL 状态，只需 使用 后缀 .astatus 替代 .sstatus，并 参考 选项 astatus_table 获取 被 识别 值 的 总结：

ampl: option astatus_table;  
option astatus_table '\  
0 in normal state (in problem)  
1 drop removed by drop command  
2 pre eliminated by presolve  
3 fix fixed by fix command  
4 sub defined variable, substituted out  
5 unused not used in current problem  
1  

下面 是 一个 常见 情况 的 示例，使用 了 我们 的 一个 饮食 模型 (diet model)：

ampl: model dietu.mod;  
ampl: data dietu.dat;  
ampl: drop Diet_Min['CAL'];  
ampl: fix Buy['SPG'] := 5;  
ampl: fix Buy['CHK'] := 3;  
ampl: solve;  
MINDS 5.5: optimal solution found.  
3 iterations, objective 54.76  
ampl: display Buy.astatus;  

Buy.astatus [*] :=  
BEEF in  
CHK fix  
FISH in  
HAM in  
MCH in  
MTL in  
SPG fix  
TUR in  
;  

ampl: display Diet_Min.astatus;  

Diet_Min.astatus [*] :=  
A in  
B1 pre  
B2 pre  
C in  
CAL drop  
;
```

一个 AMPL 状态为 `in` 表示该组件被包含在发送给 求解器 的问题中，例如变量 `Buy['BEEF']` 和约束 `Diet_Min['A']`。其他三种状态表示被排除在问题之外的组件：

- 变量 `Buy['CHK']` 和 `Buy['SPG']` 的 AMPL 状态为 `"fix"`，因为使用了 `fix` 命令来指定它们在解中的值。  
- 约束 `Diet_Min['CAL']` 的 AMPL 状态为 `"drop"`，因为它被 `drop` 命令移除了。  
- 约束 `Diet_Min['B1']` 和 `Diet_Min['B2']` 的 AMPL 状态为 `"pre"`，因为它们在 AMPL 的 presolve 阶段被简化操作移除了。

此处未显示 AMPL 状态 `"unused"`，该状态用于表示未出现在任何 目标函数 或约束中的变量；以及状态 `"sub"`，用于表示通过代入法 （如第 18.2 节所述） 被消除的变量和约束。本章后面将定义的 `objective` 命令及 `problem` 命令也会起到固定或丢弃未使用模型组件的作用。

对于一个变量或约束，通常你只关注其中一种状态：如果该变量或约束被包含在最近一次发送给 求解器 的问题中，则关注 求解器 状态；否则关注 AMPL 状态。因此，AMPL 提供了后缀 `.status` 来表示当前关注的状态：

```
ampl: display Buy.status, Buy.astatus, Buy.sstatus;
: Buy.status Buy.astatus Buy.sstatus :=
BEEF low in low
CHK fix fix none
FISH low in low
HAM low in low
MCH bas in bas
MTL low in low
SPG fix fix none
TUR low in low

ampl: display Diet_Min.status, Diet_Min.astatus,
ampl? Diet_Min.sstatus;
: Diet_Min.status Diet_Min.astatus Diet_Min.sstatus :=
A bas in bas
B1 pre pre none
B2 pre pre none
C low in low
CAL drop drop none
```

通常，如果 `name.astatus` 为 `"in"`，则 `name.status` 等于 `name.sstatus`，否则 `name.status` 等于 `name.astatus`。

# 14.3 通过后缀与 求解器 交换信息

我们已经看到，为了表示与模型组件相关的值，AMPL 使用附加在组件名称后的各种限定符或 后缀。后缀由一个句点或“点”(.) 加上一个（通常较短的）标识符组成。例如，与变量 `Buy[j]` 相关的 约简成本 写作 `Buy[j].rc`，而所有此类变量的约简成本可以通过命令 `display Buy.rc` 查看。有许多内置的此类后缀，详见 A.11 节中的表格。

然而，AMPL 无法预知 求解器 可能与模型组件关联的所有值。这些值作为输入被识别或作为输出被计算，取决于每个 求解器 及其算法的设计。为了提供一种开放式的表示方式，可以在 AMPL 会话期间定义新的 后缀，这些 后缀 可以由用户定义用于向 求解器 发送值，也可以由 求解器 定义用于返回值。

本节介绍了用户定义和求解器定义的后缀，并以 CPLEX 求解器的功能为例进行说明。我们展示了用户定义的后缀如何向整数规划求解器传递变量选择和分支方向的偏好。敏感性分析提供了一个具有数值的求解器定义后缀的例子，而不可行性诊断则展示了符号型（字符串值）后缀的工作方式。在 AMPL 脚本中报告无界方向的例子中，展示了求解器定义的后缀，必须在使用前对其进行声明。

# 用户定义后缀：整数规划指令

大多数 求解器 (solver) 都能识别 各种 算法选择 或 设置，每种 设置 都由 一个 单一 的 值 控制，该 值 适用于 整个 被求解 的 问题。因此，你可以 通过 设置 一串 指令 来 修改 选定 的 设置，如下 例 所示，将 CPLEX 求解器 应用于 一个 整数 规划 问题：

```matlab
ampl: model multmip3.mod;
ampl: data multmip3.dat;
ampl: option solver cplex;
ampl: option cplex_options 'nodesel 3 varsel 1 backtrack 0.1';
ampl: solve;
CPLEX 8.0.0: nodesel 3
varsel 1
backtrack 0.1
CPLEX 8.0.0: optimal integer solution; objective 2356251052
MIP simplex iterations 75
branch-and-bound nodes
```

然而，有 一些 求解器 设置 更为 复杂，因为 它们 需要 为 模型 中 的 每个 组件 单独 设置 值。这些 设置 数量 太多，无法 用 一串 指令 来 表示。因此，求解器 接口 可以 被 设置 为 能够 识别 用户 为 求解器 目的 而 特别 定义 的 新 后缀 (suffix)。

例如，对于 整数 规划 中 的 每个 变量，CPLEX 都 能 识别 一个 独立 的 分支 优先级 和 一个 独立 的 优选 分支 方向，分别 用 区间 $[0, 9999]$ 和 $[-1, 1]$ 中 的 整数 表示。AMPL 的 CPLEX 驱动 程序 将 后缀 `.priority` 和 `.direction` 识别 为 这些 设置。要 使用 这些 后缀，我们 首先 通过 一个 后缀 命令 来 为 当前 的 AMPL 会话 定义 每个 后缀：

```matlab
ampl: suffix priority IN, integer, >= 0, <= 9999;
ampl: suffix direction IN, integer, >= -1, <= 1;
```

这些 语句 的 效果 是 定义 形如 `name.priority` 和 `name.direction` 的 表达式，其中 `name` 表示 当前 模型 中 的 任意 变量、目标 或 约束。参数 `IN` 指定 这些 后缀 对应 的 值 将 被 求解器 读取，随后 的 短语 则 对 可接受 的 值 进行 限制（类似于 参数 声明 中 的 用法）。

这些 新定义 的 后缀 可以 通过 `let` 命令（第 11.3 节）或 后续 的 声明（如 A.8、A.9、A.10 和 A.18.8 节 所述）来 赋值。在 当前 示例 中，我们 希望 使用 这些 后缀 来 为 二元 变量 `Use[i, j]` 分配 对应 的 CPLEX 优先级 和 方向 值。通常，这些 值 是 基于 对 问题 的 了解 和 与 类似 问题 的 经验 来 选择 的。以下是 一种 可能 的 设置：

```matlab
ampl: let {i in ORIG, j in DEST} Use[i,j].priority := sum {p in PROD} demand[j,p];
ampl: let Use["GARY","FRE"].direction := -1;
```

未 被 赋予 `.priority` 或 `.direction` 值 的 变量 将 被 赋予 默认 值 零（在 本例 中，所有 约束 和 目标 也是如此），你可以 通过 如下 命令 查看：

```matlab
ampl: display Use.direction;
Use.direction [*,*] (tr) :
           CLEV  GARY  PITT :=
    DET      0     0     0
    FRA      0     0     0
    FRE      0    -1     0
    LAF      0     0     0
    LAN      0     0     0
    STL      0     0     0
    WIN      0     0     0
;
```

在 如上 所示 赋予 后缀 值 后，CPLEX 在 搜索 解 的 过程 中 所需 的 单纯形法 (simplex) 迭代 次数 和 分支定界 (branch- and- bound) 节点 数 都 减少 了：

```matlab
ampl: option reset_initial_guesses 1;
ampl: solve;
CPLEX 8.0.0: nodesel 3
varsel 1
backtrack 0.1
CPLEX 8.0.0: optimal integer solution; objective 235625799
69 MIP simplex iterations
1 branch- and- bound nodes
```

(我们将选项 reset_initial_guesses 设置为 1，以便第一次 CPLEX 运行的最优解不会被传递回第二次运行。)

有关 CPLEX 识别的后缀以及如何确定相应设置的更多信息，请参阅 CPLEX 驱动程序文档。其他求解器接口可能为不同目的识别不同的后缀；您需要分别检查每个想要使用的求解器。

# 求解器定义的后缀：敏感性分析

当关键字 sensitivity 包含在 CPLEX 的指令列表中时，将计算经典敏感性范围，并在三个新后缀 `.up`、`.down` 和 `.current` 中返回：

```matlab
ampl: model steelT.mod; data steelT.dat;
ampl: option solver cplex;
ampl: option cplex_options 'sensitivity';
ampl: solve;
CPLEX 8.0.0: sensitivity
CPLEX 8.0.0: optimal solution; objective 515033
16 dual simplex iterations (0 in phase I)
suffix up OUT;
suffix down OUT;
suffix current OUT;
```

求解命令输出末尾的三行显示了 AMPL 根据求解器结果自动执行的后缀命令。这些语句是自动执行的；您无需手动输入。每个命令中的参数 OUT 表示这些是求解器将写出值的后缀（与前面的例子形成对比，其中参数 IN 表示求解器应读入的后缀值）。

敏感性后缀的解释如下。对于变量，后缀 `.current` 表示当前问题中的目标函数 (objective) 系数，而 `.down` 和 `.up` 给出目标系数的最小和最大值，使得当前 LP 基保持最优：

<table><tr><td colspan="4">ampl: display Sell.down, Sell.current, Sell.up;</td><td></td></tr><tr><td></td><td>Sell.down</td><td>Sell.current</td><td>Sell.up</td><td>:=</td></tr><tr><td>bands 1</td><td>23.3</td><td>25</td><td>1e+20</td><td></td></tr><tr><td>bands 2</td><td>25.4</td><td>26</td><td>1e+20</td><td></td></tr><tr><td>bands 3</td><td>24.9</td><td>27</td><td>27.5</td><td></td></tr><tr><td>bands 4</td><td>10</td><td>27</td><td>29.1</td><td></td></tr><tr><td>coils 1</td><td>29.2857</td><td>30</td><td>30.8571</td><td></td></tr><tr><td>coils 2</td><td>33</td><td>35</td><td>1e+20</td><td></td></tr><tr><td>coils 3</td><td>35.2857</td><td>37</td><td>1e+20</td><td></td></tr><tr><td>coils 4</td><td>35.2857</td><td>39</td><td>1e+20</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td></tr></table>

对于约束 (constraint)，解释类似，但适用于约束的常数项（所谓的右侧值）：

<table><tr><td colspan="4">ampl: display Time.down, Time.current, Time.up;</td><td></td></tr><tr><td></td><td>Time.down</td><td>Time.current</td><td>Time.up</td><td>:=</td></tr><tr><td>1</td><td>37.8071</td><td>40</td><td>66.3786</td><td></td></tr><tr><td>2</td><td>37.8071</td><td>40</td><td>47.8571</td><td></td></tr><tr><td>3</td><td>25</td><td>32</td><td>45</td><td></td></tr><tr><td>4</td><td>30</td><td>40</td><td>62.5</td><td></td></tr><tr><td>;</td><td></td><td></td><td></td><td></td></tr></table>

你可以使用通用同义词 (第 12.6 节) 来显示所有变量或约束的范围表，类似于独立版本的 CPLEX 生成的表格。（在 .down 列中值为 $-1e+20$ 以及在 .up 列中值为 $1e+20$ 对应于 CPLEX 在其表格中称为 -无穷大和 +无穷大。）

# 求解器定义的后缀：不可行性诊断

对于没有可行解的线性规划，你可以要求 CPLEX 找到一个不可约的不可行子集 (Irreducible Infeasible Subset，或称为 IIS)：一组约束和变量边界，它们是不可行的，但当移除其中任意一个约束或边界时就变为可行。如果存在一个较小的 IIS 并且可以找到，它能够为不可行性的来源提供有价值的线索。你可以通过将 iisfind 指令从默认值 0 更改为 1（用于相对较快的版本）或 2（用于较慢的版本，但倾向于找到更小的 IIS）来开启 IIS 查找器。

以下示例展示了如何将 IIS 查找应用于第 2 章中的不可行饮食问题。在 solve 检测到没有可行解之后，使用指令 'iisfind 1' 重新求解：

ampl: model diet.mod; data diet2.dat; option solver cplex;
ampl: solve;
CPLEX 8.0.0: 不可行问题。
4 次对偶单纯形迭代 (第 I 阶段 0 次)
constraint.dunbdd 返回
后缀 dunbdd OUT;
ampl: option cplex_options 'iisfind 1';
ampl: solve;
CPLEX 8.0.0: iisfind 1
CPLEX 8.0.0: 不可行问题。
0 次单纯形迭代 (第 I 阶段 0 次)
返回包含 7 个变量和 2 个约束的 iis。
constraint.dunbdd 返回
后缀 iis symbolic OUT;
option iis_table '\
0 non 不在 iis 中\
1 low 在下界\
2 fix 固定\
3 upp 在上界\
';

同样，AMPL 会显示任何已自动执行的后缀语句。我们关注的是名为 `.iis` 的新后缀，它是符号型或字符串值的。相关的选项 `iis_table` 也由求解器驱动程序设置，并由 `solve` 自动显示，它展示了可能与 `.iis` 关联的字符串并简要描述了它们的含义。

你可以使用 `display` 查看返回的 `.iis` 值：

```
ampl: display _varname, _var.iis, _conname, _con.iis;
_varname _var.iis _conname _con.iis :=
"Buy['BEEF']" upp "Diet['A']" non
"Buy['CHK']" low "Diet['B1']" non
"Buy['FISH']" low "Diet['B2']" low
"Buy['HAM']" upp "Diet['C']" non
"Buy['MCH']" non "Diet['NA']" upp
"Buy['MTL']" upp "Diet['CAL']" non
"Buy['SPG']" low "Buy['TUR']" low
```

这些信息表明，不可行约束集 (IIS) 包含变量的四个下界和三个上界，以及提供饮食中维生素 B2 下界和钠上界的约束。这些限制共同作用时无可行解，但去掉其中任意一个限制后，其余限制将允许找到一个解。

如果对去掉边界不感兴趣，则可能只想列出 IIS 中的约束。一条打印语句可以生成一个简洁的列表：

```
ampl: print {i in 1.._ncons: _con[i].iis != "non"} _conname[i];
Diet['B2']
Diet['NA']
```

在这种情况下，你可以得出结论：为了避免违反购买数量的边界，可能需要在饮食中接受较少的维生素 B2 或较多的钠，或者两者兼而有之。然而，要进一步确定具体需要减少或增加多少，以及为了获得可行性可能需要接受哪些其他变化，则需要进一步的实验。（一个线性规划可能有多个不可约的不可行子集，但 CPLEX 的 IIS 查找算法一次只能检测到一个 IIS。）

# 求解器定义的后缀：无界性的方向

对于一个无界的线性规划——即实际上具有最小值 $-\infty$ 或最大值 $+\infty$ 的线性规划——求解器可以返回形式为 $X + \alpha d$ 的可行解射线，其中 $\alpha \geq 0$。从 CPLEX 返回时，可行解 $X$ 由变量的值给出，而无界性方向 $d$ 则通过求解器定义的后缀 `.unbdd` 与每个变量关联的附加值给出。

无界性方向的一个应用可以在模型 `trnlocld.mod` 和脚本 `trnlocld.run` 中找到，该模型和脚本将 Benders 分解应用于仓库选址和运输问题的组合；该模型、数据和脚本可从 AMPL 网站获得。我们不会在这里尝试描述整个分解方案，而是专注于通过将零一变量 `Build[i]`（表示要建造的仓库）固定为试验值 `build[i]` 而得到的子问题。在其对偶形式中，该子问题为：

```
var Supply_Price {ORIG} <= 0;
var Demand_Price {DEST};

maximize Dual_Ship_Cost:
    sum {i in ORIG} Supply_Price[i] * supply[i] * build[i] +
    sum {j in DEST} Demand_Price[j] * demand[j];

subject to Dual_Ship {i in ORIG, j in DEST}:
    Supply_Price[i] + Demand_Price[j] <= cost[i,j];
```

当所有 `build[i]` 值都设置为零时，不建造任何仓库，原始子问题是不可行的。因此，始终具有可行解的子问题的对偶形式必须是无界的。

正如本章其余部分将解释的那样，我们通过将子问题的组件收集到一个 AMPL "problem" 中，然后指示 AMPL 仅求解该问题来解决子问题。当从 AMPL 命令行应用此方法到对偶子问题时，CPLEX 以预期的方式返回无界性方向：

```
ampl: model trnloc1d.mod;  
ampl: data trnloc1.dat;  
ampl: problem Sub: Supply_Price, Demand_Price, Dual_Ship_Cost, Dual_Ship;  
ampl: let {i in ORIG} build[i] := 0;  
ampl: option solver cplex, cplex_options 'presolve 0';  
ampl: solve;  
CPLEX 8.0.0: presolve 0  
CPLEX 8.0.0: unbounded problem.  
25 dual simplex iterations (25 in phase I)  
variable.unbdd returned 6 extra simplex iterations for ray (1 in phase I)  
suffix unbdd OUT;
```

后缀信息表明，`.unbdd` 已被自动创建。你可以使用该后缀来显示无界方向，在本例中很简单：

```
ampl: display Supply_Price.unbdd;  
Supply_Price.unbdd [*] :=  
1 - 1  
4 - 1  
7 - 1  
10 - 1  
13 - 1  
16 - 1  
19 - 1  
22 - 1  
25 - 1  
2 - 1  
5 - 1  
8 - 1  
11 - 1  
14 - 1  
17 - 1  
20 - 1  
23 - 1  
3 - 1  
6 - 1  
9 - 1  
12 - 1  
15 - 1  
18 - 1  
21 - 1  
24 - 1  
;  

ampl: display Demand_Price.unbdd;  
Demand_Price.unbdd [*] :=  
A3 1  
A6 1  
A8 1  
A9 1  
B2 1  
B4 1  
;
```

我们用于 Benders 分解的脚本（trnloc1d.run）会反复求解子问题，并根据主问题生成不同的 `build[i]` 值。每次求解后都会测试结果是否无界，并据此构造主问题的一个扩展。主循环的核心如下：

```
repeat {  
solve Sub;  
if Dual_Ship_Cost $<=$ Max_Ship_Cost $+$ 0.00001 then break;  
if Sub.result $=$ "unbounded" then {  
let nCUT := nCUT + 1;  
let cut_type[nCUT] := "ray";  
let {i in ORIG} supply_price[i,nCUT] := Supply_Price[i].unbdd;  
let {j in DEST} demand_price[j,nCUT] := Demand_Price[j].unbdd;  
} else {  
let nCUT := nCUT + 1;  
let cut_type[nCUT] := "point";  
let {i in ORIG} supply_price[i,nCUT] := Supply_Price[i];  
let {j in DEST} demand_price[j,nCUT] := Demand_Price[j];  
}  
solve Master;  
let {i in ORIG} build[i] := Build[i];  
}
```

然而，在此上下文中尝试使用 `.unbdd` 会失败：

```
ampl: commands trnlocld.run;  
trnlocld.run, line 39 (offset 931): Bad suffix .unbdd for Supply_Price  
context: let {i in ORIG} supply_price[i,nCUT] := >>> Supply_Price[i].unbdd; <<<
```

问题在于 AMPL 在开始执行任何命令之前会扫描整个 repeat 循环中的所有命令。因此，它在任何不可行的子问题有机会定义该后缀之前就遇到了对 `.unbdd` 的使用。为使脚本按预期运行，需要在 repeat 循环之前添加语句 `suffix unbdd OUT;`，以便在扫描循环时 `.unbdd` 已经被定义。

# 定义和使用后缀

一个新的 AMPL 后缀 (suffix) 通过一个由关键字 suffix、后缀名以及一个或多个可选限定符 (qualifier) 组成的语句来定义，这些限定符用于指示可以与该后缀关联的值以及其使用方式。例如，我们已经看到定义 suffix priority IN, integer, $\geq 0$, $\leq 9999$，其中包含输入输出 (IN)、类型 (integer) 和边界 ($\geq 0$, $\leq 9999$) 限定符。

后缀语句使得 AMPL 能够识别形如 `component-name.suffix-name` 的后缀表达式，其中 `component-name` 指代任何当前已声明的 变量 (variable)、约束 (constraint) 或目标函数（或下一节中定义的问题）。后缀的定义将一直有效，直到下一次 reset 命令或当前 AMPL 会话结束。`suffix-name` 遵循与 AMPL 中其他名称相同的命名规则。但后缀拥有独立的命名空间，因此后缀可以与参数、变量或其他模型组件同名。后缀语句中的可选限定符可以以任意顺序出现；它们的形式和作用如下所述。

后缀语句中的可选类型限定符用于指示可以与后缀表达式相关联的值类型，默认情况下为所有数值：

```rsuffix type values allowed
none specified 任意数值
integer 整数数值
binary 0 或 1
symbolic 由 option suffix-name_table 指定的字符串列表
```

所有数值型后缀表达式的初始值为 0。它们允许的取值范围还可以通过如下形式的一个或两个边界限定符进一步限制：

$$ \text{arith-expr} \leq \text{arith-expr} $$

其中 `arith-expr` 是任何不包含变量的算术表达式。

对于每个符号型后缀，AMPL 会自动定义一个相关的数值型后缀 `suffix-name_num`。此时必须创建一个 AMPL 选项 `suffix-name_table`，用于定义 `.suffix-name` 与 `.suffix-name_num` 值之间的对应关系，如下例所示：

```rsuffix iis symbolic OUT;
option iis_table '\0 non not in the iis\1 low at lower bound\2 fix fixed\3 upp at upper bound\';
```

表中的每一行由一个整数值、一个字符串值和一个可选注释组成。每个字符串值都与其相邻的整数值相关联，并且也与小于下一行整数的更高整数值相关联。将字符串值赋给 `.suffix-name` 表达式等价于将对应的数值赋给 `.suffix-name_num` 表达式。后者的初始值为 0，并遵循上述的类型和边界限定符。（通常在 AMPL 命令和脚本中使用符号后缀的字符串值，而在与求解器通信时使用数值。）

可选的 in-out 限定符用于确定后缀值与求解器之间的交互方式：

in-out 后缀值的处理方式

IN 由 AMPL 在调用求解器前写入，然后由求解器读取  
OUT 由求解器写入，然后由 AMPL 在求解器完成后读取  
INOUT 既读又写，如上述 IN 和 OUT  
LOCAL 既不读也不写

如果未指定 in-out 关键字，则默认为 INOUT。

我们已经看到，可以通过 let 语句对后缀表达式进行赋值或重新赋值：

```let Use["GARY","FRE"].direction := -1;```

这里仅对一个变量赋值了 suffix 值，但通常索引集合中的所有变量都会被赋值 suffix：

```ampl
var Use {ORIG, DEST} binary;
let {i in ORIG, j in DEST} Use[i,j].priority := sum {p in PROD} demand[j,p];
```

在这种情况下，suffix 值的赋值可以与变量的声明合并：

```ampl
var Use {i in ORIG, j in DEST} binary,
    suffix priority sum {p in PROD} demand[j,p];
```

一般来说，`var` 声明中的一个或多个短语可以由关键字 `suffix` 组成，后跟一个先前定义的 suffix 名称以及用于计算相关 suffix 表达式的表达式。

# 14.4 模型之间的交替

第 13 章描述了如何设置 AMPL 命令的“脚本”，以运行执行重复操作的程序。在多个示例中，脚本通过在循环中包含 `solve` 语句来求解一系列相关的模型实例。结果是一个简单的灵敏度分析算法，使用 AMPL 的命令语言编程实现。

通过使用两个模型，可以构建更强大的算法过程。一个模型的最优解为另一个模型提供新数据，两个模型交替求解，直到最终满足某个终止条件。经典的列生成、割平面生成、分解和拉格朗日松弛方法都是基于这种方案，这些方法在本章末尾引用的文献中有详细描述。

要以这种方式使用两个模型，脚本必须有某种方式在它们之间切换。切换可以通过预先定义的 AMPL 特性完成，或者更清晰高效地通过定义分别命名的问题和环境来实现。

我们通过一个用于“卷材修整”或“下料问题”的基本形式的脚本来说明这些可能性，使用一种著名的、基础的列生成过程。为了简洁起见，我们在此仅简要描述该过程，而本章末尾的参考文献提供了详细描述的来源。AMPL 网站上有其他几个生成、分解和松弛方案的示例，我们稍后也会使用其中的一些片段，但不会展示完整模型。

在卷材修整问题中，我们希望将某种商品（如纸卷）的长原始宽度切割成较小宽度的组合，以满足给定订单，并尽可能减少浪费。这个问题可以看作是决定在每个原始宽度的卷材上应在哪里进行切割，以生产出一个或多个已订购的小宽度。然而，用决策变量来表达这样的问题很不方便，并且会导致一个整数规划问题，除非实例非常小，否则很难求解。

为了推导出一个更易于处理的模型，所谓的 Gilmore-Gomory 方法定义了一个切割模式 (cutting pattern) 为原材料卷的一种可行切割方式。因此，一个模式由各种所需宽度的卷材数量构成，且这些卷材的总宽度不得超过原材料的宽度。如果 (如在练习 2-6 中) 原材料宽度为 $110"$，而所需宽度有 $20"$、$45"$、$50"$、$55"$ 和 $75"$，那么两个 $45"$ 的卷材和一个 $20"$ 的卷材便构成一个可接受的模式，同样地，一个 $50"$ 和一个 $55"$ 的卷材 (剩余 $5"$ 的废料) 也构成一个可接受的模式。基于这种观点，图 14-2 中的两个简单线性 program (linear program) 可以协同工作，从而找到一个高效的切割方案。

切割优化模型 (图 14-2a) 用于在给定一系列已知可使用的切割模式的前提下，找到所需切割的原材料卷的最小数量 (min)。这实际上与饮食模型非常相似，其中 variables (variables) 表示被切割的模式而不是被购买的食物项，而 constraints (constraints) 则对切割宽度施加下限，而不是对提供的营养成分施加下限。

模式生成模型 (图 14-2b) 则试图识别出一个新的模式，该模式可用于切割优化中，以减少所需原材料卷的数量，或者确定不存在这样的新模式。该模型的 variable (variable) 是新模式中每种所需宽度的数量；唯一的 constraint (constraint) 确保该模式的总宽度不超过原材料宽度。我们在此不打算解释其目标函数，只是指出 variables (variable) 的系数由切割优化模型的线性松弛所对应的“对偶值 (dual value)”或“对偶价格 (dual price)”给出。

我们可以通过反复交替求解这两个问题来寻找一个良好的切割方案。首先，切割优化问题的连续 variables 松弛生成一些对偶价格，然后模式生成问题利用这些价格生成一个新模式，接着在模式集合扩展一个模式后重复该过程。当我们重复该过程直到模式生成问题表明不存在新模式可以带来改进时，我们就停止重复。此时，我们便得到了在 (可能) 切割分数个原材料卷的情况下的最优解。我们还可以在恢复整数 constraints 的条件下，最后一次运行切割优化模型，以得到最佳的整数解——

参数 roll_width $> 0$ # 原始卷的宽度  
集合 WIDTHS # 需要切割的宽度集合  
参数 orders {WIDTHS} $> 0$ # 每种宽度的需求量  
参数 nPAT 整数 $\geq 0$ # 模式数量  
集合 PATTERNS $= \{1, 2, \ldots, \mathrm{nPAT}\}$ # 模式集合  
参数 nbr {WIDTHS, PATTERNS} 整数 $\geq 0$  
检查 {j in PATTERNS}: sum {i in WIDTHS} i * nbr[i,j] $\leq$ roll_width # 模式的定义：nbr[i,j] 表示在模式 j 中宽度为 i 的卷的数量  
变量 Cut {PATTERNS} 整数 $\geq 0$ # 使用每种模式切割的卷数  
最小化 Number: # 最小化切割的原始卷总数  
sum {j in PATTERNS} Cut[j]  
约束 Fill {i in WIDTHS}:  
sum {j in PATTERNS} nbr[i,j] * Cut[j] $\geq$ orders[i] # 对于每种宽度，切割的总卷数满足总需求量  

图 14-2a：基于模式的切割优化问题模型 (cut.mod)

参数 price {WIDTHS} 默认值 0.0 # 切割优化的对偶价格  
变量 Use {WIDTHS} 整数 $\geq 0$ # 每种宽度在模式中的数量  
最小化 Reduced Cost: 1 - sum {i in WIDTHS} price[i] * Use[i]  
约束 Width_Limit: sum {i in WIDTHS} i * Use[i] $\leq$ roll_width  

图 14-2b：用于模式生成问题的背包模型 (cut.mod, 续)

使用生成的模式进行优化，或者我们可以简单地将分数卷数向上取整到下一个最大整数，如果结果可以接受的话。

这就是 Gilmore-Gomory 过程。就我们的两个 AMPL 模型而言，其步骤可以描述如下：

选择足以满足需求的初始模式  
重复  
求解 (分数) 切割优化问题  
令 price[i] 等于 Fill[i].dual  
对每个模式 i 求解模式生成问题  
如果最优值 $< 0$，则添加一个新模式，该模式切割 Use[i] 个宽度为 i 的卷  
否则找到最终整数解并停止  

一种简单的初始化方法是为每种宽度生成一个模式，该模式包含尽可能多的该宽度副本，以适应原始卷的宽度。这些模式显然可以覆盖任何需求，尽管不一定以经济的方式。

图 14-3 显示了作为 AMPL 脚本实现的 Gilmore-Gomory 过程。文件 cut.mod 包含图 14-2 中的切割优化和模式生成模型。由于这些模型没有共同的变量或约束，因此可以使用简单的 solve 语句和交替的目标函数编写脚本：

repeat { objective Number; solve; .. objective Reduced Cost; solve; }

然而，在这种方法下，每次 solve 都会将两个模型生成的所有变量和约束发送给求解器。这种安排效率低下且容易出错，特别是在处理更大更复杂的迭代过程时。

我们可以通过使用 fix 和 drop 命令来抑制其他变量和约束，从而确保只将当前相关的变量和约束发送给求解器。然后，我们的循环结构将如下所示：

repeat {unfix Cut; restore Fill; objective Number; fix Use; drop Width_Limit; solve; ... unfix Use; restore Width_Limit; objective Reduced Cost; fix Cut; drop Fill; solve; ...}

在每次 solve 之前，先前固定的 variable 和被丢弃的 constraint 也必须通过 unfix 和 restore 命令恢复。这种方法虽然高效，但极易出错，并且使脚本难以阅读。

因此，作为替代方案，AMPL 允许通过 problem 语句来区分模型，如图 14-3 所示：

problem Cutting_Opt: Cut, Number, Fill; option relax_integrality 1; problem Pattern_Gen: Use, Reduced_Cost, Width_Limit; option relax_integrality 0;

第一条语句定义了一个名为 Cutting_Opt 的 problem，它由 Cut 变量、Fill 约束以及目标函数 Number 组成。该语句同时将 Cutting_Opt 设为当前 problem；后续对 var、minimize、maximize、subject to 和 option 语句的使用将仅作用于该 problem。例如，通过将 option relax_integrality 设置为 1，我们确保当 Cutting_Opt 为当前 problem 时，Cut 变量的整数条件会被放松。类似地，我们定义另一个 problem Pattern_Gen，它由 Use 变量、Width_Limit 约束和目标函数 Reduced_Cost 组成；该 problem 成为新的当前 problem，此时我们将 relax_integrality 设置为 0，因为该 problem 仅接受整数解才有意义。

图 14-3 中的 for 循环用于创建初始切割模式，之后主 repeat 循环执行 Gilmore-Gomory 过程，如前所述。语句

solve Cutting_Opt;

将恢复 Cutting_Opt 作为当前 problem 及其环境，并求解相应的线性规划 (linear program)。然后赋值语句将 Cutting_Opt 的最优对偶价格传递给 Pattern_Gen 将使用的参数 price[i]。在 AMPL 中，所有集合和参数都是全局的，因此无论当前 problem 是什么，它们都可以被引用或修改。

主循环的后半部分切换到 problem Pattern_Gen 及其环境，并将相关的整数规划 (integer program) 发送给求解器 (solver)。如果得到的目标函数值足够负，则由 Use[i] 变量返回的模式将被添加到下一轮 Cutting_Opt 使用的数据中。否则无法继续优化，循环终止。

脚本最后通过以下语句求解使用所有生成模式的最佳整数解：

option Cutting_Opt.relax_integrality 0; solve Cutting_Opt;

表达式 Cutting_Opt.relax_integrality 表示在 Cutting_Opt 环境中 relax_integrality 选项的值。我们将在下一节更详细地讨论这类名称及其用途。

作为其工作方式的一个示例，图 14-4 显示了切割 110 英寸的原材料卷以满足宽度分别为 20、45、50、55 和 75 的成品卷的需求量 48、35、24、10 和 8 的数据。图 14-5 显示了当使用图 14-2 和图 14-4 所示的模型和数据运行图 14-3 的脚本时产生的输出。最优的分数解使用五种不同的模式切割 46.25 个原材料卷，如果将分数值向上取整到下一个整数，则需要 48 个卷。最终使用整数变量求解显示了如何应用六种模式的集合来仅使用 47 个原材料卷满足需求。

# 14.5 命名问题

正如我们的切割优化示例所示，编写在两个 (或多个) 模型之间交替的清晰高效脚本的关键在于使用命名问题，这些命名问题代表模型组件的不同子集。在本节中，我们将更详细地描述如何使用 AMPL 的问题语句来定义、使用和显示命名问题。最后我们还将介绍一个类似的概念，即命名环境，它有助于在 AMPL 选项集合之间切换。

本节中的图示取自切割优化脚本和 AMPL 网站上的一些其他示例脚本。这些脚本背后的逻辑解释超出了本书的范围；在本章末尾的参考文献中给出了一些进一步学习的建议。

# 定义命名问题

在 AMPL 会话的任何时候，都存在一个由变量、目标和约束列表组成的当前问题。默认情况下，当前问题被命名为 Initial，并且包含到目前为止定义的所有变量、目标和约束。然而，您可以定义其他由这些组件子集组成的 "命名" 问题，并使它们成为当前问题。当一个命名问题被设为当前问题时，问题子集中的所有模型组件都将被激活，而所有其他变量、目标和约束将被停用。更准确地说，问题子集中的变量将被解固定，其余变量将在其当前值处被固定。问题子集中的目标和约束将被恢复，其余的将被丢弃。 (固定和丢弃在第 11.4 节中讨论。)

您可以通过一个问题声明最直接地定义一个问题，该声明给出问题的名称及其组件列表。因此在图 14-3 中我们有：

```ampl
problem Cutting_Opt: Cut, Number, Fill;
```

定义了一个名为 Cutting_Opt 的新问题，该问题包含来自图 14-2 模型中的所有 Cut 变量、目标函数 Number 以及所有 Fill 约束。同时，Cutting_Opt 成为当前问题。任何固定的 Cut 变量将被解除固定，而所有其他已声明的变量则在其当前值处被固定。如果目标函数 Number 此前被删除，则会被恢复，而所有其他已声明的目标函数则被删除；同样地，任何被删除的 Fill 约束会被恢复，而所有其他已声明的约束则被删除。

对于更复杂的模型，一个问题的组成部分列表通常包括多个变量和约束集合，例如在 stochl.run (来自 AMPL 网站的一个示例) 中的这个例子：

```ampl
problem Sub: Make, Inv, Sell, Stage2_Profit, Time, Balance2, Balance;
```

通过在问题名称后指定索引表达式，你可以定义一个问题的索引集合，例如在 multi.2.run (另一个网站示例) 中的这些定义：

```ampl
problem SubII {p in PROD}: Reduced_Cost[p],
  {i in ORIG, j in DEST} Trans[i,j,p],
  {i in ORIG} Supply[i,p],
  {j in DEST} Demand[j,p];
```

对于集合 PROD 中的每一个 $\mathbb{P}$，都会定义一个 SubII[p] 问题。其组成部分包括目标函数 Reduced_Cost[p]、对于每个在 ORIG 中的 i 和在 DEST 中的 j 组合的变量 Trans[i,j,p]，以及对于每个在 ORIG 中的 i 的约束 Supply[i,p] 和对于每个在 DEST 中的 j 的约束 Demand[j,p]。

一个问题声明的形式和解释自然与其他指定模型组件列表的 AMPL 语句相似。声明以关键字 problem 开头，接着是一个之前未用于任何其他模型组件的问题名称、一个可选的索引表达式（用于定义一个问题的索引集合）和一个冒号。冒号后是用逗号分隔的变量、目标函数和约束列表，这些将被包含在问题中。该列表可以包含以下任意形式的条目，其中“component”指的是任何变量、目标函数或约束：

一个组件名称，例如 Cut 或 Fill，指的是所有具有该名称的组件。  
一个带下标的组件名称，例如 Reduced_Cost[p]，指的是该组件本身。  
一个索引表达式后跟一个带下标的组件名称，例如 {i in ORIG} Supply[i,p]，指的是索引集中每个成员的一个组件。

为了避免在多个组件以相同方式索引时重复索引表达式，问题语句也允许一个索引表达式后跟一个括号内的组件列表。例如，以下写法是等价的：

{i in ORIG} Supply1[i,p], {i in ORIG} Supply2[i,p], {i in ORIG, j in DEST} Trans[i,j,p], {i in ORIG, j in DEST} Use[i,j,p]  
{i in ORIG} (Supply1[i,p], Supply2[i,p], {j in DEST} (Trans[i,j,p], Use[i,j,p]))

正如这些示例所示，括号内的列表可以包含任何在组件列表中有效的项，甚至可以是一个索引表达式后跟另一个括号列表。这种递归形式也出现在 AMPL 的 print 命令中，但比 display 命令所允许的列表格式更通用。

每当声明一个变量、目标或约束时，它会自动被添加到当前问题（如果最近的问题语句指定了一个问题的索引集合，则添加到所有当前问题）中。因此，在我们的切割库存示例中，图 14-2 的所有模型组件默认首先被放入问题 Initial 中；然后，当运行图 14-3 的脚本时，通过使用 problem 语句将这些组件分配到问题 Cutting_Opt 和 Pattern_Gen 中。作为替代方案，我们可以声明空问题，然后通过 AMPL 声明填充其成员。图 14-6（cut2.mod）展示了如何为图 14-2 的模型完成这一操作。这种方法在较简单的应用中有时更为清晰或易于实现。

任何使用 drop/restore 或 fix/unfix 的操作也会修改当前问题。drop 命令的效果是从当前问题中移除约束或目标，而 restore 命令的效果是向当前问题中添加约束或目标。类似地，fix 命令从当前问题中移除变量，而 unfix 命令则添加变量。例如，multil.run 使用以下问题语句：

问题 Cutting_Opt;  
参数 nPAT integer $\geq 0$ default 0;  
参数 roll_width;  
集合 PATERNS $= 1$..nPAT;  
集合 WIDTHS;  
参数 orders {WIDTHS} $> 0$;  
参数 nbr {WIDTHS, PATERNS} integer $\geq 0$;  
check {j in PATERNS}: sum {i in WIDTHS} i \* nbr[i,j] $\leq$ roll_width;  
变量 Cut {PATERNS} $\geq 0$;  
最小化 Number: sum {j in PATERNS} Cut[j];  
约束 Fill {i in WIDTHS}: sum {j in PATERNS} nbr[i,j] \* Cut[j] $\geq$ orders[i];  

问题 Pattern_Gen;  
参数 price {WIDTHS};  
变量 Use {WIDTHS} integer $\geq 0$;  
最小化 Reduced_Cost: 1 - sum {i in WIDTHS} price[i] \* Use[i];  
约束 Width_Limit: sum {i in WIDTHS} i \* Use[i] $\leq$ roll_width;

用于定义其分解过程第一阶段和第二阶段的命名问题。相比之下，multila.run 最初指定问题如下：  
problem Master: Artificial, Weight, Excess, Multi, Convex;  
problem Sub: Artif_Reduced_Cost, Trans, Supply, Demand;  
以定义初始问题，然后在需要将问题转换为适用于第二阶段的形式时执行：  
problem Master;  
drop Artificial;  
restore Total_Cost;  
fix Excess;  
problem Sub;  
drop Artif_Reduced_Cost;  
restore Reduced_Cost;  

由于在整个过程中使用了名称 Master 和 Sub，因此脚本中的一个循环就足以实现两个阶段。

或者，一个重新声明的问题语句可以为一个问题提供新的定义。例如，上面的 drop、restore 和 fix 命令可以被替换为 redeclare problem Master: Total Cost, Weight, Multi, Convex; redeclare problem Sub: Reduced Cost, Trans, Supply, Demand;

然而，像其他声明一样，这不能在复合语句 (if、for 或 repeat) 内使用，因此不能在 multila.run 示例中使用。

reset 命令的一种形式允许您撤销对问题定义所做的任何更改。例如，reset problem Cutting_Opt; 将 Cutting_Opt 的定义重置为最近一次定义它的的问题语句中的组件列表。

# 使用命名问题

接下来我们描述更改当前问题的替代方法。任何更改通常会导致不同的目标和约束被丢弃，不同的变量被固定，结果是为求解器生成了不同的优化问题。然而，仅通过更改当前问题，不会影响与模型组件关联的值。所有先前声明的组件都是可访问的，无论当前问题是什么，它们都保持相同的值，除非它们被 let 或 data 语句显式更改，或者在变量和目标值及相关量 (如对偶值、栈和简化成本) 的情况下通过求解更改。

任何仅引用一个问题 (而不是问题的索引集合) 的问题语句都会使该问题成为当前问题。例如，在切割库存脚本的开始，我们希望先使一个命名问题成为当前问题，然后使另一个命名问题成为当前问题，以便我们可以在问题的环境中调整某些选项。cut1.run 中的问题语句 (图 14-3)：

问题 Cutting_Opt: Cut, Number, Fill; option relax_integrality 1; 问题 Pattern_Gen: Use, Reduced_Cost, Width_Limit; option relax_integrality 0; 既用于定义新问题，也用于使这些问题成为当前问题。cut2.run 中的类似语句更简单：

problem Cutting_Opt; option relax_integrality 1; problem Pattern_Gen; option relax_integrality 0;

这些语句仅用于使命名问题成为当前问题，因为问题已经由 cut2.mod 中的问题语句 (图 14-6) 定义。

问题语句也可以引用问题的索引集合，如之前引用的 multi2.run 示例：

problem SubII {p in PROD}: Reduced_Cost[p], ...

这种形式定义了潜在的许多问题，每个 PROD 集合的成员对应一个问题。后续的问题语句可以一次使集合的一个成员成为当前问题，如具有以下形式的循环 for {p in PROD} { problem SubII[p]; }

或在诸如 problem SubII["coils"] 的语句中引用特定成员。

正如之前的示例所示，solve 语句也可以包含一个问题名称，这种情况下命名的问题将变为当前问题，然后被发送到求解器。因此，诸如 solve Pattern_Gen 这样的语句的效果与先执行 problem Pattern_Gen 再执行 solve 的效果完全相同。

# 显示命名问题

仅由 problem 组成的命令用于显示当前问题：

mpl: model cut.mod;  
appl: data cut.dat;  
appl: problem;  
problem Initial;  
appl: problem Cutting_Opt: Cut, Number, Fill;  
appl: problem Pattern_Gen: Use, Reduced_Cost, Width_Limit;  
appl: problem;  
problem Pattern_Gen;

在定义其他命名问题之前，当前问题始终为 Initial。show 命令可以列出已定义的命名问题：

mpl: show problems;  
problems: Cutting_Opt Pattern_Gen

我们也可以使用 show 查看构成特定问题或索引问题集合的变量、目标和约束：

mpl: show Cutting_Opt, Pattern_Gen;  
problem Cutting_Opt: Fill, Number, Cut;  
problem Pattern_Gen: Width_Limit, Reduced_Cost, Use;

并使用 expand 查看当前问题在所有数据值被替换后的显式目标和约束：

mpl: expand Pattern_Gen;  
minimize Reduced Cost:  
 - 0.166667*Use[20] - 0.416667*Use[45] - 0.5*Use[50] - 0.5*Use[55] - 0.833333*Use[75] + 1;  
subject to Width_Limit:  
 20*Use[20] + 45*Use[45] + 50*Use[50] + 55*Use[55] + 75*Use[75] <= 110;

有关 show 和 expand 的进一步讨论，请参见第 12.6 节。

# 定义和使用命名环境

正如在 AMPL 会话的任何时刻都存在一个当前问题一样，也始终存在一个当前环境。问题是一组非固定变量以及未被丢弃的目标和约束，而环境则记录了所有 AMPL 选项的值。通过命名不同的环境，脚本可以轻松地在不同的选项设置集合之间切换。

在默认操作模式下 (该模式对于许多用途已经足够)，当前环境始终与当前问题同名。在 AMPL 会话开始时，当前环境被命名为 Initial，之后每当 problem 语句定义一个新的命名问题时，也会同时定义一个与该问题同名的新环境。环境在创建时会继承当时的所有选项设置，但在其作为当前环境时所做的新设置将被保留。任何更改当前问题的 problem 或 solve 语句也会切换到相应命名的环境，并相应地设置选项。

例如，我们用于切割库存问题的脚本 (图 14-3) 在设置模型和数据后，按如下方式继续执行：

option solver cplex, solution_round 6;  
option display_1col 0, display_transpose -10;  
problem Cutting_Opt: Cut, Number, Fill;  
option relax_integrality 1;  
problem Pattern_Gen: Use, Reduced_Cost, Width_Limit;  
option relax_integrality 0;

选项 `solver` 和其他三个选项 (由前两个选项语句修改) 在任何问题语句之前更改；因此，它们的新设置会被后续定义的环境继承，并且在脚本的其余部分中保持不变。接下来，一个问题语句定义了一个新问题和一个名为 `Cutting_Opt` 的新环境，并使其成为当前问题和环境。随后的选项语句将 `relax_integrality` 更改为 `1`。此后，当 `Cutting_Opt` 是脚本中的当前问题 (和环境) 时，`relax_integrality` 的值将是 `1`。最后，另一个问题和选项语句对问题 (和环境) `Pattern_Gen` 执行了类似操作，不同之处在于在该环境中 `relax_integrality` 被重新设置为 `0`。

这些初始语句的结果是确保在重复循环中的每个后续 `solve` 语句都能正确设置。`solve Cutting_Opt` 的结果是将当前环境设置为 `Cutting_Opt`，从而将 `relax_integrality` 设置为 `1`，并导致切割优化 (Cutting_Opt) 问题的线性松弛被求解。类似地，`solve Pattern_Gen` 的结果是导致图案生成 (Pattern_Gen) 问题作为整数规划问题被求解。我们也可以在循环内使用选项语句来切换 `relax_integrality` 的设置，但通过这种方法，我们使循环——脚本的关键部分——尽可能简单。

在更复杂的情况下，可以通过使用由关键字 `environ` 后跟名称组成的语句，独立于命名问题来声明命名环境：

```
environ Master;
```

环境拥有自己的命名空间。如果该名称之前未被用作环境名称，则将其定义为环境名称，并与当前所有选项值相关联。否则，该语句的效果是使该环境 (及其相关选项值) 成为当前环境。

通过在问题语句中的冒号前放置 `environ` 和环境名称，先前声明的环境也可以与新命名问题的声明相关联：

```
problem MasterII environ Master:
```

然后，每当关联的问题成为当前问题时，命名环境会自动成为当前环境。在这种情况下，会覆盖通常创建的与问题同名的环境。

通过在环境名称后放置 AMPL 索引表达式，可以在 `environ` 语句中声明环境的索引集合。然后，该名称以通常方式“下标”以引用各个环境。

命名环境以与命名问题相同的方式处理更改。如果在某个特定环境为当前环境时更改了某个选项的值，则会记录新值，并且当该环境再次成为当前环境时，将恢复该值。

# 参考文献

[1] Vasek Chvátal, 《Linear Programming》, Freeman, New York, NY, 1983.

[2] Marshall L. Fisher, “An Applications Oriented Guide to Lagrangian Relaxation”, 《Interfaces》, 15 卷第 2 期 (1985), 第 10–21 页。

[3] Robert Fourer 与 David M. Gay, “Experience with a Primal Presolve Algorithm”, 载于《Large Scale Optimization: State of the Art》, W. W. Hager、D. W. Hearn 和 P. M. Pardalos 编, Kluwer Academic Publishers, Dordrecht, The Netherlands, 1994, 第 135–154 页。

[4] Robert W. Haessler, “Selection and Design of Heuristic Procedures for Solving Roll Trim Problems”, 《Management Science》, 34 卷第 12 期 (1988), 第 1460–1471 页。

[5] Leon S. Lason, 《Optimization Theory for Large Systems》, Macmillan, New York, NY, 1970；后由 Dover 重印, Mineola, NY, 2002。

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

# 非线性规划

虽然任何违反第 8 章线性规则的模型都“不是线性的”，但传统上“非线性规划 (nonlinear program)”这一术语的使用范围更为狭窄。在本章中，非线性规划与线性规划类似，具有连续 (而非整数或离散) 变量；其目标函数和约束中的表达式不必是线性的，但必须表示“光滑 (smooth)”函数。直观上，一个单变量的光滑函数的图像如图 18-1a 所示，在每一点上都有明确定义的斜率；不像图 18-1b 中那样存在跳跃，或像图 18-1c 中那样存在折点。从数学上讲，任何数量变量的光滑函数必须是连续的，并且在每一点上都必须有明确定义的梯度 (一阶导数组成的向量)；图 18-1b 和 18-1c 分别展示了函数不连续和不可微的点。

这类函数的优化问题之所以被特别关注，有几个原因：它们容易与其他“非线性”问题区分开来；它们具有广泛而多样的应用；并且它们可以通过一些成熟的算法类型来求解。实际上，大多数非线性规划求解器都使用依赖于连续性和可微性假设的方法。即使有这些假设，非线性规划通常也比类似的线性规划更难建立和求解。

本章首先介绍数学规划中非线性的来源。我们并不试图系统地涵盖各种非线性模型，而是给出一些示例，说明非线性出现的原因和方式。接下来的部分将讨论非线性对 AMPL 变量和表达式的影响。最后，我们将指出在尝试求解非线性规划时可能遇到的一些困难。

尽管本章的重点是非线性优化，请注意 AMPL 也可以表示非线性方程组或不等式组，即使没有需要优化的目标函数。存在专门用于这种情况的求解器，许多用于非线性优化的求解器也能较好地找到方程组或不等式组的可行解。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/e0f04df39837466c8484b2ffe549de4583a50620e99b05e90b6d77124ebd60a7.jpg)  
图 18-1：非线性函数的类别。

(a) 光滑且连续的函数

(b) 不连续的函数

(c) 连续但不可微的函数

# 18.1 非线性的来源

我们在此讨论在优化模型中引入非线性的三种方式：通过放弃线性假设、通过构建非线性函数以达到预期效果，以及通过建模本身具有非线性的物理过程。

作为示例，我们描述一些线性网络流模型 net1.mod 的非线性变体，该模型在第 15 章中引入（见图 15-2a）。该线性规划的目标是最小化总运输成本，即最小化总成本：

$$
\text{minimize Total Cost: } \sum_{(i,j) \in \text{LINKS}} \text{cost}[i,j] \times \text{Ship}[i,j];
$$

其中，$\text{cost}[i,j]$ 和 $\text{Ship}[i,j]$ 分别表示城市 $i$ 与城市 $j$ 之间每单位的运输成本和总运输量，$\text{LINKS}$ 表示所有存在运输路线的城市对集合。约束条件是每个城市的流量平衡：

$$
\text{约束条件 平衡 } \{k \in \text{CITIES}\}: \text{supply}[k] + \sum_{(i,k) \in \text{LINKS}} \text{Ship}[i,k] = \text{demand}[k] + \sum_{(k,j) \in \text{LINKS}} \text{Ship}[k,j];
$$

其中非负参数 $\text{supply}[i]$ 和 $\text{demand}[i]$ 分别表示城市 $i$ 的可用或所需单位数量。

# 放弃线性假设

线性网络流模型假设从城市 $i$ 到城市 $j$ 运输的每一单位货物都产生相同的运输成本 $\text{cost}[i,j]$。图 18-2a 展示了这种情况下运输成本与运输量之间的典型关系图；该图为一条斜率为 $\text{cost}[i,j]$ 的直线（因此称为线性）。图 18-2 中的其他图则展示了运输成本可能依赖于运输量的多种非线性方式。

在图 18-2b 中，成本也倾向于随运输量线性增长，但在某些关键运输量处，单位成本（即直线的斜率）会发生突变。这种函数称为分段线性（piecewise linear）。严格来说，它不是线性的，但也不是平滑的非线性函数。分段线性目标函数的使用是第 17 章的主题。

在图 18-2c 中，函数本身会发生突变。当没有运输时，运输成本为零；但一旦有任何运输量，成本从一个大于零的值开始呈线性增长。在这种情况下，从 $i$ 到 $j$ 的链路存在一个固定成本，再加上每单位运输量的可变成本。同样，这不是线性规划技术可以处理的函数，但它也不是平滑的非线性函数。固定成本通常通过使用整数变量来处理，这将在第 20 章中讨论。

其余的图展示了本章中我们希望考虑的平滑非线性函数类型。图 18-2d 展示了一种凹成本函数。每增加一单位运输量的增量成本（即图形的斜率）最初较大，但随着运输量的增加而减小；在某一点之后，成本几乎呈线性增长。这是对图 18-2c 中固定成本函数的一种连续替代方式。它也可以用来近似图 18-2b 中的情形，即随着运输量的增加，可获得批量折扣的情况。

图 18-2e 显示了一种凸成本函数。对于较小的运输量，成本大致呈线性增长，但当运输量接近某个临界值时，成本会急剧上升。这种函数用于建模这样一种情况：即首先使用成本最低的运输方式，随着运输单位数量的增加，运输成本逐渐升高。这个临界值实际上代表了运输量的一个上限。

这些是最简单的函数形式。你所选用的函数将取决于你要建模的具体情况。图 18-2f 显示了一种既非凹也非凸的函数形式，它结合了前两个例子的特征。

线性函数除了系数 (或斜率) 不同之外，本质上都是一样的；而非线性函数可以由无穷多种不同的公式来定义。因此，在构建非线性规划模型时，你需要自行推导或指定能够准确表示当前情况的非线性函数。在运输问题示例的目标函数中，例如，一种可能的做法是将原来的乘积项 `cost[i,j] * Ship[i,j]` 替换为：

\[
\frac{cost1[i,j] + cost2[i,j] \cdot Ship[i,j]}{1 + Ship[i,j]} \cdot Ship[i,j]
\]

该函数在运输量较小时迅速增长，而在运输量较大时趋于平缓，基本上呈线性关系。因此，它体现了一种实现图 18-2d 所示曲线的方法。

另一种指定非线性目标函数的方法是关注图 18-2 中各图的斜率。在线性情形 (图 18-2a) 中，图线的斜率恒定；这就是为什么我们可以使用单一参数 `cost[i,j]` 来表示单位运输成本。在分段线性情形 (图 18-2b) 中，每个区间内的斜率是恒定的；我们可以通过第 17 章中介绍的方法来表达这类分段线性函数。

然而，在非线性情形下，斜率随着运输量的变化而连续变化。这提示我们可以回到最初网络流问题的线性形式，并将参数 `cost[i,j]` 转换为变量 `Cost[i,j]`：

```ampl
var Cost {ORIG, DEST}; # 每单位运输成本变量  
var Ship {ORIG, DEST} >= 0; # 运输量  
minimize Total_Cost: sum {i in ORIG, j in DEST} Cost[i,j] * Ship[i,j];
```

这不再是一个线性目标函数，因为它包含了变量与变量之间的乘积。我们再添加一些方程以说明成本与运输量之间的关系：

```ampl
subject to Cost_Relation {i in ORIG, j in DEST}:  
Cost[i,j] = (cost1[i,j] + cost2[i,j]*Ship[i,j]) / (1 + Ship[i,j]);
```

这些方程也是非线性的，因为它们涉及除以一个包含变量的表达式。很容易看出，当运输量接近零时，`Cost[i,j]` 接近 `cost1[i,j]`，但当运输水平足够高时，会趋于 `cost2[i,j]`。因此，只要第一个成本大于第二个成本，就能实现图 18-2d 中的凹成本 (concave cost)。

非线性的假设也可以出现在约束中。网络流模型的约束只体现了弱线性假设，即从一个城市运出的总量是运往其他各城市的运输量之和。但在图 1-6a 的生产模型中，约束体现了一个强假设：制造每种产品 `p` 在每个阶段 `s` 所使用的小时数随着生产水平线性增长。

# 实现非线性效果

有时非线性来源于需要建模一种线性函数根本无法表现出期望行为的情况。

例如，在交通流的网络模型中，可能有必要考虑拥堵因素。对于小的运输量，穿越运输链接的总时间应基本上为常数，但当接近链接容量时，应迅速增加至无穷大。没有线性函数具备这种性质，因此我们被迫使旅行时间成为运输负载的非线性函数，以获得期望的效果。

表示旅行时间的一种可能函数如下：

$$
\mathtt{time}[\mathtt{i},\mathtt{j}] + (\mathtt{sens}[\mathtt{i},\mathtt{j}]\star \mathtt{ship}[\mathtt{i},\mathtt{j}]) / (1 - \mathtt{ship}[\mathtt{i},\mathtt{j}] / \mathtt{cap}[\mathtt{i},\mathtt{j}])
$$

当 Ship[i,j] 的值较小时，该函数近似为 time[i,j]，但当 Ship[i,j] 接近 cap[i,j] 时趋向无穷大；第三个参数 sens[i,j] 控制着函数在两个极端之间的形状。该函数始终是凸的，因此其图形类似于图 18-2e。（练习 18-4 建议了如何将该旅行时间函数整合到交通流的网络模型中。）

另一个例子是，我们可能希望允许需求只能被近似满足。我们可以通过引入一个变量 Discrepancy[k] 来建模这种可能性，用以表示交付量与需求量之间的偏差。这个变量可以为正也可以为负，它被加到平衡约束的右侧：

subject to Balance {k in CITIES}: supply[k] $+$ sum {i,k) in LINKS} Ship[i,k] $=$ demand[k] $+$ Discrepancy[k] $+$ sum {k,j) in LINKS} Ship[k,j];

一种已确立的方法是通过在目标函数中添加惩罚成本来防止偏差变得过大。如果该惩罚与偏差量成正比，则我们得到一个凸分段线性惩罚项，

最小化 总成本 (Total Cost)：$\sum_{(i,j) \in LINKS} cost[i,j] \times Ship[i,j] + \sum_{k \in CITIES} pen \times Discrepancy[k]$；

其中 pen 是一个正参数。AMPL 可以轻松地将此目标函数转换为线性形式。

然而，这种形式的惩罚项可能无法达到我们想要的效果，因为它对差异 (discrepancy) 的每一单位都施加了相同的惩罚。为了抑制较大的差异，我们希望随着差异的加剧，每单位的惩罚逐渐增大，但这一特性无法通过线性惩罚函数（或具有有限段的分段线性函数）实现。取而代之的是，一个更合适的惩罚函数应为二次函数：

最小化 总成本 (Total_Cost)：$\sum_{(i,j) \in LINKS} cost[i,j] \times Ship[i,j] + \sum_{k \in CITIES} pen \times Discrepancy[k]^2$；

在涉及逼近或数据拟合的优化问题中，目标函数为某些量的平方和的非线性目标非常常见。

# 对本质上非线性过程的建模

在诸如石油精炼 (N)、电力传输和结构设计等物理活动的模型中，存在许多非线性来源。大多数情况下，这些模型中的非线性并非源于对线性假设的放松，而是由支配力、体积、电流等的固有非线性关系所导致。物理模型中非线性函数的形式可能更容易确定，因为它们来源于经验测量和物理基本定律（而非经济因素）。另一方面，物理模型中的非线性往往涉及更复杂的函数形式以及变量之间的相互作用。

举一个简单的例子，在天然气管道网络的模型中，不仅要考虑城市之间的运输量 (shipments)，还要考虑各个城市的压力，且压力受到某些界限的约束。因此，除了流量变量 Ship[i, j] 外，模型还必须定义一个变量 Press[k] 来表示每个城市 k 的压力。如果城市 i 的压力大于城市 j，则流量从 i 流向 j，并满足以下关系：

$$
\mathtt{Flow}[\mathtt{i},\mathtt{j}]^2 = \mathtt{c}[\mathtt{i},\mathtt{j}]^2 \times (\mathtt{Press}[\mathtt{i}]^2 - \mathtt{Press}[\mathtt{j}]^2)
$$

其中 $\mathtt{c}[\mathtt{i},\mathtt{j}]$ 是由管道的长度、直径和效率以及气体性质决定的常数。管道上的压缩机和阀门会产生不同的非线性流量关系。其他类型的网络，特别是电力传输网络，也有其特定的非线性流量关系，这些关系由物理规律决定。

如果你知道想要包含在模型中的非线性表达式的代数形式，那么你很可能已经能看出如何在 AMPL 中编写它。本章接下来的两节将分别考虑与为非线性规划声明变量以及编写非线性表达式相关的一些具体问题和特性。然而，为了避免你因编写非线性表达式过于简单而忽视潜在问题，最后一节将就求解非线性规划给出一些警示性建议。

# 18.2 非线性变量

尽管 AMPL 中为非线性规划声明变量的方式与线性规划相同，但变量的两个特性——初值 (initial values) 和自动替代 (automatic substitution) 在处理非线性模型时特别有用。

# 变量的初值

你可以为 AMPL 变量指定值。在优化之前，这些“初始”值可以通过 AMPL 命令进行显示和操作。当你输入 solve 命令时，这些初始值会被传递给求解器，求解器可能会将它们作为解的初始猜测值。在求解器完成工作后，这些初始值将被计算出的最优值所取代。

所有用于为参数赋值的 AMPL 特性同样也适用于变量。在 var 声明中还可以通过可选的 $\coloneqq$ 短语来指定初始值；对于运输问题的例子，你可以写成

$$
\mathtt{var}\ \mathtt{Ship}\{\mathtt{LINKS}\} >= 0,\ \mathtt{\coloneqq}\ 1;
$$

将每个 Ship[i,j] 的初始值设为 1，或者写成

$$
\mathtt{var}\ \mathtt{Ship}\{(\mathtt{i,j})\ \mathtt{in}\ \mathtt{LINKS}\} >= 0,\ \mathtt{\coloneqq}\ \mathtt{cap}[\mathtt{i,j}] - 1;
$$

将每个 Ship[i,j] 初始化为 cap[i,j] 减 1。或者，也可以在数据语句中与模型的其他数据一起给出初始值：

<table><tr><td>var Ship:</td><td>FRA</td><td>DET</td><td>LAN</td><td>WIN</td><td>STL</td><td>FRE</td><td>LAF</td><td>:=</td></tr><tr><td>GARY</td><td>800</td><td>400</td><td>400</td><td>200</td><td>400</td><td>200</td><td>200</td><td></td></tr><tr><td>CLEV</td><td>800</td><td>800</td><td>800</td><td>600</td><td>600</td><td>500</td><td>600</td><td></td></tr><tr><td>PITT</td><td>800</td><td>800</td><td>800</td><td>200</td><td>300</td><td>800</td><td>500</td><td>,</td></tr></table>

所有用于参数的数据语句同样也可以用于变量，如第 9.4 节所述。

所有这些为常规（“原始”）变量赋值的特性同样也适用于与约束相关联的对偶变量（第 12.5 节）。AMPL 将对约束名称的赋值解释为对相关联的对偶变量或（在非线性规划中更常见的术语）相关联的拉格朗日乘数 (Lagrange multiplier) 的赋值。少数求解器，例如 MINOS，可以利用这些乘数的初始值。

通过为变量提供良好的初始值，通常可以加快求解器的工作速度。即使对于线性规划问题也是如此，但在非线性情况下效果更为显著。初始猜测值的选择可能会决定求解器找到的“最优”目标函数值，甚至可能影响求解器是否能找到任何最优解。这些可能性在本章最后一节中将进一步讨论。

如果你没有为某个变量指定初始值，那么 AMPL 会暂时将其设为零。如果求解器内置了确定初始值的程序，那么它可能会重新设置未初始化变量的值，同时利用已初始化变量的值。否则，未初始化的变量将保持为零。虽然零是一个显而易见的起点，但它并没有特殊的意义；在第 18.4 节中我们将给出的一些例子中，除非将初始值从零重新设定，否则求解器无法成功进行优化。

# 变量的自动替换

在上一节的一个例子中已经出现了变量替换的问题，我们在其中声明了变量来表示运输成本，然后通过一个约束条件用其他变量来定义它们：

subject to Cost_Relation {(i,j) in LINKS}: Cost[i,j] = (cost1[i,j] + cost2[i,j]*Ship[i,j]) / (1 + Ship[i,j]);

如果将等号右侧的表达式替换所有出现的 Cost[i,j]，就可以从模型中消除 Cost 变量，并且这些约束无需传递给求解器。有两种方法可以告诉 AMPL 自动进行此类替换。

第一种方法是将选项 `substout` 从默认值 `0` 改为 `1`，这样可以告诉 AMPL 寻找所有具有上述形式的“定义”约束：等号左侧为单个变量。采用这种替代方式时，AMPL 会尝试使用尽可能多的这类约束，将变量从模型中替换出去。当你输入 `solve` 命令后，模型和数据生成了一个非线性规划 (nonlinear program) 问题时，约束将按照它们在模型中出现的顺序进行扫描。一个约束被识别为“定义”约束的条件是：它在等号左侧只有一个变量；该左侧变量的声明未指定任何限制（如整数性或边界）；并且该左侧变量尚未出现在已被识别为定义约束的其他约束中。

然后，等号右侧的表达式将替换其他约束中出现的左侧变量，定义约束本身将被删除。这些规则为 AMPL 提供了一种简单的方法来避免循环替换，但这也意味着替换的性质和数量可能依赖于约束的排序。

作为替代，如果您想明确指定要代换出某个特定的变量集合，请在变量声明中使用 `=` 短语。对于前面的例子，您可以写成：

```ampl
var Cost {(i,j) in LINKS} = (cost1[i,j] + cost2[i,j]*Ship[i,j]) / (1 + Ship[i,j]);
```

然后变量 `Cost[i,j]` 将会被 `=` 符号后面的表达式所替代。这种类型的声明可以以任意顺序出现，但需满足通常的要求，即在 `=` 短语中出现的每个变量都必须事先定义。

可以代换出去的变量在优化问题中并不是数学上必需的。尽管如此，它们往往具有重要的描述性作用；通过为非线性表达式关联名称，它们使得更复杂的表达式能够被清晰地书写。此外，即使这些变量已从传递给求解器 (solver) 的问题中移除，它们的名称仍可用于浏览优化结果。

当同一个非线性表达式在目标函数和约束中多次出现时，引入一个定义变量来表示它可以使模型更加简洁且更具可读性。AMPL 也能高效地处理这种代换。在为求解器生成非线性规划 (nonlinear program) 的表示形式时，AMPL 并不会将整个定义表达式的副本替换到定义变量的每个出现位置。相反，它将表达式分解为线性和非线性部分，并保存一份非线性部分的副本以及其值应被替换的位置列表；只有线性部分的项会在多个位置被显式替换。这种对线性项的单独处理对某些求解器（如 MINOS）是有利的，因为它们会特殊处理线性项，但您可以通过将选项 `linelim` 设置为零来关闭这一功能。

从求解器的角度来看，代换会减少约束和变量的数量，但往往会使约束和目标表达式更加复杂。因此，在某些情况下，如果不代换定义变量，求解器的表现可能会更好。在开发新模型时，您可能需要通过实验来确定哪些代换能够带来最佳结果。

# 18.3 非线性表达式

AMPL 的任何算术运算符 （表 7-1） 和算术函数 （表 7-2） 都可以应用于 变量 （variable） 以及 参数 （parameter） 。 如果由此产生的任何 目标 （objective） 或 约束 （constraint） 不满足 线性 （第 8 章） 或 分段线性 （第 17 章） 的规则， AMPL 会将其视为 “非线性” 。 当您键入 solve 时， AMPL 会传递足够的指令， 使您的 求解器 （solver） 能够计算每个 目标 和 约束 中的每个 表达式 （expression） ， 并在适当时提供导数。

如果你使用的是典型的非线性 求解器 ， 那么你需要自己将 目标函数 （objective function） 和 约束 定义为该 求解器 所要求的 “光滑” 函数。 在这方面， AMPL 的 表达式 （expression） 语法的通用性可能会产生误导。 例如， 如果你试图使用 变量 Flow[i,j] 来表示点 i 和 j 之间的流量， 那么你可能会写出像 cost[i,j] * abs(Flow[i,j]) 或 if Flow[i,j] $= 0$ then 0 else base[i,j] $^+$ cost[i,j]*Flow[i,j] 这样的 表达式 。

这些 表达式 显然不是线性的， 但第一个 表达式 不是光滑的 （其斜率在零点处突然改变） ， 第二个 表达式 甚至不是连续的 （其值在零点处突然跳跃） 。 如果你尝试使用这样的 表达式 ， AMPL 不会报错， 你的 求解器 甚至可能返回它声称是 最优解 的结果——但结果可能是错误的。

将非光滑函数 （如 abs、 min 和 max） 应用于 变量 的 表达式 通常会产生非光滑的结果； 同样， 如果 if-then-else 表达式 中 if 后面的 条件 （condition） 包含 变量 ， 那么该 表达式 通常也不是光滑的。 尽管如此， 在某些情况下， 通过精心编写的 表达式 可以保持光滑性。 例如， 再次考虑第 18.1 节中的流量-压力关系。 如果城市 i 的压力大于城市 j 的压力， 那么流量方向是从 i 到 j， 并且流量与压力的关系为：

$$
\mathtt{Flow}[\mathtt{i},\mathtt{j}]^{\wedge}2 = \mathtt{c}[\mathtt{i},\mathtt{j}]^{\wedge}2\ast (\mathtt{Press}[\mathtt{i}]^{\wedge}2 - \mathtt{Press}[\mathtt{j}]^{\wedge}2)
$$

如果城市 j 的压力大于城市 i 的压力， 则可以写出类似的方程：

$$
\mathtt{Flow}[\mathtt{j},\mathtt{i}]^{\wedge}2 = \mathtt{c}[\mathtt{j},\mathtt{i}]^{\wedge}2\ast (\mathtt{Press}[\mathtt{j}]^{\wedge}2 - \mathtt{Press}[\mathtt{i}]^{\wedge}2)
$$

但由于常数 $\mathbb{C}[\mathbf{i},\mathbf{j}]$ 和 $\mathbb{C}[\mathbf{j},\mathbf{i}]$ 指的是同一条管道， 因此它们是相等的。 因此， 我们不需要为每个方向的流量定义单独的 变量 ， 而是可以让 $\mathtt{Flow}[\mathtt{i},\mathtt{j}]$ 的符号不受限制， 正值表示从 i 到 j 的流量， 负值表示相反方向的流量。 使用这个 变量 ， 前面的一对流量-压力 约束 可以被替换为一个：

$$
\begin{array}{rl}
& (\texttt{if}\ \mathtt{Flow}[\mathtt{i},\mathtt{j}] >= 0\ \mathtt{then}\ \mathtt{Flow}[\mathtt{i},\mathtt{j}]^{\wedge}2\ \mathtt{else}\ -\mathtt{Flow}[\mathtt{i},\mathtt{j}]^{\wedge}2) \\
& \qquad = \mathtt{c}[\mathtt{i},\mathtt{j}]^{\wedge}2 \ast (\mathtt{Press}[\mathtt{i}]^{\wedge}2 - \mathtt{Press}[\mathtt{j}]^{\wedge}2)
\end{array}
$$

通常，使用 if 表达式会产生非光滑 (nonsmooth) 约束，但在这种情况下，它产生了一个函数，其两个二次部分在 $\mathtt{Flow}[\mathtt{i},\mathtt{j}]$ 为零时“光滑相接”，如图 18-3a 所示。

另一个例子是图 18-3b 中的凸函数，最简单的写法是 $\log (\mathtt{Flow}[\mathtt{i},\mathtt{j}]) / (\mathtt{Flow}[\mathtt{i},\mathtt{j}] - 1)$，但不幸的是，如果 $\mathtt{Flow}[\mathtt{i},\mathtt{j}]$ 为 1，则该表达式简化为 $0 / 0$，这会被报告为错误。实际上，即使 $\mathtt{Flow}[\mathtt{i},\mathtt{j}]$ 只是非常接近 1，该表达式也无法准确计算。如果我们改写为

if abs(Flow[i,j]- 1) > 0.00001 then log(Flow[i,j] / (Flow[i,j]- 1) else 1.5 - Flow[i,j]/2

则在 Flow[i,j] 的量级较小时，会代之以一个高度精确的线性近似。这种替代方式在严格的数学意义上并非光滑函数，但在数值上足够接近光滑，足以用于某些 求解器。

在发送给 求解器 的问题实例中，AMPL 会区分线性与非线性约束和目标函数，并将每个约束和目标表达式分解为线性项之和加上一个非线性部分。由于某些变量被固定而变成线性的附加项也会被识别为线性项。例如，在我们 18.1 节的例子中，

minimize Total Cost: sum {i in ORIG, j in DEST} Cost[i,j] * Ship[i,j];

每次固定一个 Cost[i,j] 变量都会产生一个线性项；如果所有 Cost[i,j] 变量都被固定，则目标函数会被表示为线性函数提供给 求解器。变量可以通过 fix 命令（第 11.4 节）或通过 presolve 阶段的操作（第 14.1 节）来固定；尽管预求解算法会忽略非线性约束，但它会处理所有可用的线性约束，以及随着变量固定而变为线性的约束。

AMPL 的内置函数只是模型构建中最常用的一些函数。在需要时可以引入其他有用的函数库。例如，要使用名为 statlib 的函数库中的累积正态分布函数和反累积正态分布函数，首先需要用如下语句加载该库：

load statlib.dll;

并通过 AMPL 的 function 语句声明这些函数：

function cumnormal; function invcumnormal;

然后模型就可以使用这些函数来构成表达式，例如 cumnormal(mean[i], sdev[i], Inv[i,t]) 和 invcumnormal(6)。如果这些函数作用于变量，AMPL 还会安排在求解过程中执行函数求值。

函数声明指定了库函数的名称及其 (可选的) 必需参数。参数的数量可以是任意多个，甚至可以是迭代的参数集合。每个函数的声明必须出现在其使用之前。为了方便起见，可以随库一起提供包含函数声明的脚本，因此像 `include statlib` 这样的语句就足以提供对库中所有函数的访问权限。库的文档将说明可用的函数及其参数的数量和含义。

确定正确的加载命令可能涉及许多细节，这些细节取决于你所使用的系统类型甚至其特定配置。有关可能性及相关 `AMPLFUNC` 选项的进一步讨论，请参见第 A.22 节。

如果你有雄心，可以编写并编译自己的函数库。相关说明和示例可从 AMPL 网站获取。

# 18.4 非线性规划的陷阱

虽然 AMPL 使你能够构建各种非线性优化模型，但当你输入 `solve` 时，并没有任何求解器能保证每次都得到可接受的解。求解器所使用的算法容易受到非线性函数复杂性所带来的各种困难的影响。这与线性情况形成了令人遗憾的对比，在线性情况下，一个设计良好的求解器几乎可以可靠地求解任何线性规划。

本节简要介绍非线性规划的陷阱。我们重点关注两类常见困难：函数值域违规和多个局部最优解，然后简要提及其他几种陷阱。

为了说明问题，我们从图 18-4 所示的非线性运输模型开始。它与我们之前的运输示例 (图 3-1a) 相同，只是目标函数中的项 `cost[i,j] * Trans[i,j]` 被替换为非线性项：

```
minimize Total Cost:
  sum {i in ORIG, j in DEST}
    rate[i,j] * Trans[i,j] / (1 - Trans[i,j]/limit[i,j]);
```

每项都是 `Trans[i,j]` 的凸递增函数，如图 18-2e 所示；在 `Trans[i,j]` 相对较小的值时，它近似于斜率为 `rate[i,j]` 的线性函数，但当 `Trans[i,j]` 接近 `limit[i,j]` 时趋于无穷大。相关数据值也类似于第 3 章中线性运输示例所使用的数据，见图 18-5。

# 函数值域违规

尝试使用给定的模型和数据进行求解时失败了：

```
ampl: model nltrans.mod;
ampl: data nltrans.dat;
ampl: solve;
MINOS 5.5 Error evaluating objective Total_Cost
can't compute 8000/0
MINOS 5.5: solution aborted.
8 iterations, objective 0
```

求解器的消息令人费解，但强烈暗示在计算目标函数时发生了除以零的情况。这只有在表达式

```
1 - Trans[i,j]/limit[i,j]
```

等于零时才会发生。

```
param supply {ORIG} >= 0;    # 起始地的可用量
param demand {DEST} >= 0;    # 目的地的需求量
check: sum {i in ORIG} supply[i] = sum {j in DEST} demand[j];

param rate {ORIG,DEST} >= 0; # 每单位的基础运输成本
param limit {ORIG,DEST} > 0; # 运输单位的限制

var Trans {i in ORIG, j in DEST} >= 0;  # 待运输的单位数

minimize Total Cost:
  sum {i in ORIG, j in DEST}
    rate[i,j] * Trans[i,j] / (1 - Trans[i,j]/limit[i,j]);
```

$$\sum_{i \in ORIG, j \in DEST} \frac{rate[i,j] \cdot Trans[i,j]}{1 - \frac{Trans[i,j]}{limit[i,j]}}$$

约束条件  
供应量  
$$\sum_{j \in DEST} Trans[i,j] = supply[i], \quad \forall i \in ORIG$$

需求量  
$$\sum_{i \in ORIG} Trans[i,j] = demand[j], \quad \forall j \in DEST$$

图 18-4: 非线性运输模型 (nltrans.mod)。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/95e49b4d1777325531b114cd5d87f9eab92e9cdf0b15f2dd9ee435f8bdef05db.jpg)  
图 18-5: 非线性运输模型的数据 (nltrans.dat)。

在某点为零。如果我们使用 display 来打印 Trans[i, j] 等于 limit[i, j] 的对：

impl: display {i in ORIG, j in DEST: Trans[i,j] = limit[i,j]};  
set {i in ORIG, j in DEST: Trans[i,j] == limit[i,j]} := (GARY, LAF) (PITT, LAN);

impl: display Trans['GARY','LAF'], limit['GARY','LAF'];  
Trans['GARY','LAF'] = 1000  
limit['GARY','LAF'] = 1000

我们可以看到问题所在。求解器允许 Trans[GARY, LAF] 的值为 1000，这等于 limit[GARY, LAF]。结果，目标函数项 rate[GARY, LAF] * Trans[GARY, LAF] / (1 - Trans[GARY, LAF]/limit[GARY, LAF]) 的计算结果为 $8000 / 0$。由于求解器无法评估目标函数，它放弃了寻找最优解。

因为非线性优化算法的行为可能对初始猜测的选择敏感，我们可能希望求解器从不同的起点取得更大的成功。为了确保比较有意义，我们首先设置

ampl: option send_statuses 0;

这样，上一次求解返回的变量状态信息将不会被发送回去，以帮助确定下一次求解的起点。然后可以使用 AMPL 的 let 命令建议，例如，每个 Trans[i, j] 的新初始值为 limit[i, j] 的一半：

ampl: let {i in ORIG, j in DEST} Trans[i, j] := limit[i, j]/2;  
ampl: solve;  
MINDS 5.5: 当前点无法改进。  
32 次迭代，目标值 - 7.385903389e+18

这次求解器运行完成，但仍然存在问题。目标值小于 $- 10^{18}$，或者在实际应用中为 $- \infty$，并且解被描述为“无法改进”而不是最优。

检查求解器返回的解中 Trans[i, j]/limit[i, j] 的值，可以为困难提供线索：

ampl: display {i in ORIG, j in DEST} Trans[i, j]/limit[i, j];  
Trans[i, j]/limit[i, j] [*,*] (tr)  
: CLEV GARY PITT :=  
DET - 6.125e- 14 0 2  
FRA 0 1.5 0.1875  
FRE 0.7 1 0.5  
LAF 0.4 0.15 0.5  
LAN 0.375 7.03288e- 15 0.5  
STL 2.9 0 0.5  
WIN 0.125 0 0.5  
;

这些比率表明，对于几对运输路径（如 Trans[CLEV, STL]），其运输量显著超过了限制。更严重的是，Trans[GARY, FRE] 的比率正好为 1，这表明它恰好达到了 limit[GARY, FRE] 的限制。如果我们以完整精度显示它们，可以看到：

ampl: option display_precision 0;  
ampl: display Trans['GARY', 'FRE'], limit['GARY', 'FRE'];  
Trans['GARY', 'FRE'] = 500.0000000000028  
limit['GARY', 'FRE'] = 500

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/791cd5f55e0e18743fdf7f8c7770a2a0d35a24c4b4fd510958fcdad7be66570e.jpg)  
图 18-6：成本函数 $y = \frac{x}{1 - \frac{x}{c}}$ 中的奇点。

变量值仅略微超过限制，因此成本项出现了巨大的负值。如果我们绘制整个成本函数的图像，如图 18-6 所示，可以看到成本函数在 limit[GARY, FRE] 处的奇点右侧趋向于 $-\infty$。

上述两次运行中的误差来源在于我们假设：由于当 Trans[i,j] 从下方趋近于 limit[i,j] 时，目标函数趋向于 $+\infty$，因此求解器会将 Trans[i,j] 保持在 0 和 limit[i,j] 之间。但至少对于这个求解器来说，我们必须通过为每个 Trans[i,j] 显式设置一个略小于 limit[i,j] 的上界来强制执行这一假设，但该上界必须足够接近，以不影响最终的最优解：

```
var Trans {i in ORIG, j in DEST} >= 0, <= .9999 * limit[i,j];
```

经过这一修改后，求解器能够轻松找到最优解：

```
mpl: option display_precision 6;  
ampl: model nltransb.mod; data nltrans.dat; solve;  
MINDS 5.5: optimal solution found.  
81 iterations, objective 1212117  
ampl: display Trans;  
Trans [*,*] (tr)  
: CLEV GARY PITT :=  
DET 586.372 187.385 426.242  
FRA 294.993 81.2205 523.787  
FRE 365.5 369.722 364.778  
LAF 490.537 0 509.463  
LAN 294.148 0 305.852  
STL 469.691 761.672 468.637  
WIN 98.7595 0 301.241  
;
```

这些变量值远离任何奇点，其中 Trans[i,j]/limit[i,j] 在所有情况下都小于 0.96。（如果你将初始猜测值改为 limit[i,j]/2，如前所述，你应该会发现解是相同的，但求解器只需要大约一半的迭代次数即可找到解。）

这里的直接教训是，非线性函数在变量预期范围之外可能会表现出相当糟糕的行为。图 18-2e 中的不完整图形使这个成本函数看起来误导性地“表现良好”，而图 18-6 则显示了需要设置边界以使变量远离奇点。

更一般的教训是，非线性函数所带来的困难可能导致求解器以各种方式失败。在开发非线性模型时，你需要警惕求解器给出的奇怪结果，并可能需要进行一些侦探工作，以将问题追溯到模型中的缺陷。

# 多个局部最优解

为了说明非线性优化中的另一种困难，我们考虑一个稍作修改的目标函数，其公式如下：

```
minimize Total Cost: sum {i in ORIG, j in DEST} rate[i,j] * Trans[i,j]^0.8 / (1 - Trans[i,j]/limit[i,j]);
```

通过将运输量提高到 0.8 次幂，我们使成本函数在较低运输量时呈凹形，在较高运输量时呈凸形，如图 18-2f 所示。尝试求解这个新模型时，我们再次遇到技术难题：

```
ampl: model nltransc.mod; data nltrans.dat; solve;
MINOS 5.5: Error evaluating objective Total Cost: can't evaluate pow'(0,0.8)
MINOS 5.5: solution aborted.
8 iterations, objective 0
```

这次我们自然怀疑到 `Trans[i,j]^0.8`，这是我们在模型中唯一修改的表达式。错误信息中提到的 `pow'(0,0.8)` 提供了进一步线索，它表示指数（幂）函数在零点的导数。当 `Trans[i,j]` 为零时，该函数有一个明确的值，但其关于变量的导数（即图 18-2f 中曲线的斜率）是无穷大。因此，总成本对任何变量在零点的偏导数无法返回给求解器；由于求解器的优化算法需要所有偏导数，它因此放弃求解。

这是另一种形式的范围违规问题，同样可以通过施加一些边界条件来避免解靠近问题点。在本例中，我们将下界从零调整为一个非常小的正数：

```
var Trans {i in ORIG, j in DEST} >= 1e-10, <= 0.9999 * limit[i,j], := 0;
```

我们也可以将初始猜测值从零移开，但在本例中求解器会自动处理，因为初始值只是建议起始点。

调整边界后，求解器正常运行并报告了一个解：

```
ampl: model nltransd.mod; data nltrans.dat; solve;
MINDS 5.5: optimal solution found.
65 iterations, objective 427568.1225
ampl: display Trans;
Trans [* ,* ] (tr)
CLEV    GARY    PITT
...
DET   689.091  1e-10   510.909
FRA    1e-10   199.005 700.995
FRE   385.326 326.135 388.54
LAF   885.965 114.035  1e-10
LAN   169.662  1e-10   430.338
STL   469.956 760.826 469.218
WIN    1e-10    1e-10   400 ;
```

我们可以将每个 `1e-10` 视为零，因为与解中的其他值相比，这样的小值可以忽略不计。

接下来，我们再次尝试使用 `limit[i,j]/2` 作为初始猜测值，希望加快求解速度。结果如下：

```
ampl: let {i in ORIG, j in DEST} Trans[i,j] := limit[i,j]/2;
ampl: solve;
MINDS 5.5: optimal solution found.
40 iterations, objective 355438.2006
ampl: display Trans;
Trans [* ,* ] (tr)
:     CLEV    GARY    PITT
...
DET   540.601 265.509 393.89
FRA   328.599  1e-10  571.401
FRE   364.639 371.628 363.732
LAF   491.262  1e-10  508.738
LAN   301.741  1e-10  298.259
STL   469.108 762.863 468.029
WIN   104.049  1e-10  295.951 ;
```

不仅解完全不同，而且最优值降低了 17%！第一个解并不能真正最小化在约束条件下所有可行解中的目标函数值。

实际上，这两种解都可以被认为是正确的，因为它们都是局部最优解 (local optimum)。也就是说，每个解都比其他附近的解成本更低。本章所讨论的所有经典非线性优化方法，都是为了寻找局部最优解而设计的。从一个指定的初始猜测开始，这些方法并不能保证找到全局最优解 (global optimum)，即在所有满足约束条件的解中目标函数值最好的解。通常来说，寻找全局最优解比寻找局部最优解要困难得多。

幸运的是，在许多情况下，局部最优解就已经完全令人满意了。当目标函数和约束满足某些性质时，任何局部最优解同时也是全局最优解；本节开头所考虑的模型就是一个例子，其中目标函数的凸性 (convexity) 与约束的线性 (linearity) 保证了求解器将找到一个全局最优解。（线性规划 (Linear programming) 是一个更特殊的情况，具有这种性质；这就是为什么在前面的章节中我们从未遇到过不是全局最优的局部最优解。）

即使存在多个局部最优解，对所建模问题的了解也可能帮助你识别出全局最优解。也许你可以选择一个接近全局最优解的初始解，或者可以添加一些约束来排除已知包含局部最优解的区域。

最后，即使你无法保证找到的解是全局最优的，你可能也满足于找到一个非常好的局部最优解。一种直接的方法是系统地尝试一系列不同的起始点，并从中选取最优解。作为一个简单的例子，假设我们在示例中按如下方式声明变量：

```javascript
param alpha >= 0, <= 1;
var Trans {i in ORIG, j in DEST}  >= 1e-10, <= .9999 * limit[i, j], 
    := alpha * limit[i, j];
```

对于每一个 `alpha` 值，我们都会得到一个不同的初始猜测，从而可能得到不同的解。下表列出了 `alpha` 从 0 到 1 变化时的一些目标函数值：

<table>
<tr><td>alpha</td><td>Total_Cost</td></tr>
<tr><td>0.0</td><td>427568.1</td></tr>
<tr><td>0.1</td><td>366791.2</td></tr>
<tr><td>0.2</td><td>366791.2</td></tr>
<tr><td>0.3</td><td>366791.2</td></tr>
<tr><td>0.4</td><td>366791.2</td></tr>
<tr><td>0.5</td><td>355438.2</td></tr>
<tr><td>0.6</td><td>356531.5</td></tr>
<tr><td>0.7</td><td>376043.3</td></tr>
<tr><td>0.8</td><td>367014.4</td></tr>
<tr><td>0.9</td><td>402795.3</td></tr>
<tr><td>1.0</td><td>365827.2</td></tr>
</table>

我们之前找到的 `alpha` 为 0.5 的解仍然是最佳的，但鉴于这些结果，我们现在更倾向于相信它是一个非常好的解。我们还可能观察到，尽管报告的目标函数值随着起始点的选择而变化得有些不规律（这是非线性规划问题普遍具有的特征），但第二佳的 `Total_Cost` 值是通过将 `alpha` 设为 0.6 得到的。这表明，在 0.5 附近的 `alpha` 值进行更细致的搜索可能是值得的。

一些更复杂的全局优化方法正是尝试以这种方式搜索起始点，但它们采用了一种更为复杂且系统化的程序来决定下一步尝试哪些起始点。其他方法则将全局优化视为一个组合问题，并应用受整数规划（第 20 章）启发的求解方法。全局优化方法目前仍处于相对早期的发展阶段，随着经验的积累、新想法的尝试以及可用计算能力的进一步提升，这些方法有望得到改进。

# 其他陷阱

许多其他因素也会影响非线性 求解器 的效率和成功率，包括模型的构建方式以及变量单位 （或缩放比例） 的选择。通常来说，当非线性出现在目标函数中而非约束条件中时，更容易处理。AMPL 提供的自动替换变量选项 （在本章前面已描述） 在这方面可能会有所帮助。另一个经验法则是，变量的取值不应相差超过几个数量级；当某些变量以百万计，而另一些变量以千分之一计时，求解器可能会被误导。一些 求解器 会自动对问题进行缩放以尽量避免这种情况，但通过明智地选择变量表达的单位，你可以大大帮助它们。

除了我们已经讨论过的失败模式外，非线性 求解器 还有许多其他失败模式。一些非线性优化方法可能会卡在某种“驻点”（stationary point） 上，而这些驻点在任何意义上都不是最优的；可能会在需要最小值时误将极大值识别为最优解 （反之亦然）；也可能错误地表明约束条件无可行解。在这些情况下，你唯一能做的可能是尝试不同的起始猜测值；有时指定一个满足许多非线性约束条件的可行起始点会有所帮助。你也可以通过为变量设置合理的 上界 和 下界 来提高 求解器 成功的可能性。例如，如果你知道某些变量的最优值为 80 是合理的，而 800 则不合理，那么你可以为它们设定 400 的 上界 。（一旦得到一个指示的最优解，你应该检查这些“安全”边界是否被任何变量达到；如果达到，则应放宽边界并重新求解问题。）

本节的目的是说明在处理非线性模型时需要格外谨慎。如果你遇到无法通过本文所述简单方法解决的困难，你可能需要查阅 非线性规划 方面的教科书、你所使用的特定 求解器 的文档，或咨询一位熟悉非线性优化技术的数值分析师。

# 参考文献

Roger Fletcher, Practical Methods of Optimization. John Wiley & Sons (New York, NY, 1987). 理论和方法的简明综述。

Philip E. Gill, Walter Murray and Margaret H. Wright, Practical Optimization. Academic Press (New York, NY, 1981). 理论、算法和实用建议。

Jorge Nocedal and Stephen J. Wright, Numerical Optimization. Springer Verlag (Heidelberg, 1999). 关于光滑函数优化方法的教材。

Richard P. O'Neill, Mark Williard, Bert Wilkins and Ralph Pike, "A Mathematical Programming Model for Allocation of Natural Gas." Operations Research 27, 5 (1979) pp. 857- 873. 第 18.1 节中描述的天然气管道网络非线性关系的来源。

# 练习

18- 1. 在第 18.4 节的最后一个例子中，尝试更多的起始点，看看是否能找到一个更好的局部最优解。你能找到的最佳解是什么？

18- 2. 下面的小模型 fence.mod 用于确定用给定长度的围栏所能围成的最大面积矩形场地的尺寸：

param fence $>0$ var Xfield $>= 0$ var Yfield $>= 0$ maximize Area: Xfield * Yfield; subject to Enclose: 2*Xfield $+$ 2*Yfield $<=$ fence;

众所周知, 最优场地是一个正方形。

(a) 当我们尝试用 40 米长的围栏解决这个问题时, 使用变量的默认初始猜测值零, 得到了以下结果:

```python
ampl: solve;
MINOS 5.5: optimal solution found.
0 iterations, objective 0
ampl: display Xfield, Yfield;
Xfield = 0
Yfield = 0
```

什么可以解释这个意外的结果？在任何可用的非线性求解器上尝试同样的问题, 并报告你观察到的行为。

(b) 如有必要, 使用不同的起始点运行你的求解器, 确认 40 米围栏的最优尺寸确实是 $10 \times 10$。

(c) 通过一个类似的模型进行实验, 用于确定用给定面积的纸张所能包装的最大体积盒子的尺寸。

(d) 解决与 (c) 中相同的问题, 但包装的是圆柱体而不是盒子。

18-3. 在一个无名行星上, 一个下落物体在 (主要是) 每秒间隔 $t_j$ 被观察到具有大约以下高度 $h_j$:

<table>
<tr><td>tj</td><td>0.0</td><td>0.5</td><td>1.5</td><td>2.5</td><td>3.5</td><td>4.5</td><td>5.5</td><td>6.5</td><td>7.5</td><td>8.5</td><td>9.5</td><td>10.0</td></tr>
<tr><td>hj</td><td>100</td><td>95</td><td>87</td><td>76</td><td>66</td><td>56</td><td>47</td><td>38</td><td>26</td><td>15</td><td>6</td><td>0</td></tr>
</table>

根据该行星上的物理定律, 物体在任意 时间 的高度应由以下公式给出:

$$
h_j = a_0 - a_1t_j - \frac{1}{2}a_2t_j^2,
$$

其中 $a_0$ 是初始高度, $a_1$ 是初始速度, $a_2$ 是重力加速度。但由于观测并非完全精确, 因此不存在 $a_0$、$a_1$ 和 $a_2$ 的取值能使所有数据完全符合该公式。相反, 我们希望估计这三个值, 通过选择它们来最小化"平方和":

$$
\sum_{j = 1}^{n}[h_{j} - (a_{0} - a_{1}t_{j} - \frac{1}{2}a_{2}t_{j}^{2})]^{2}.
$$

其中 $t_j$ 和 $h_j$ 是表格中第 $j$ 项的观测值, $n$ 是观测数量。该表达式衡量了理想公式与观测行为之间的误差。

(a) 创建一个 AMPL 模型, 用于最小化任意观测数量 $n$ 的 $t_j$ 和 $h_j$ 的平方和。该模型应包含三个 变量 和一个目标函数, 但无 约束。

(b) 使用 AMPL 数据语句表示上述样本观测值, 并求解由此产生的非线性规划问题, 以确定 $a_0$、$a_1$ 和 $a_2$ 的估计值。

18-4. 本题涉及一个非常简单的"交通流量"网络:

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/6a33bd49a4ba4f447a8b598afcf23c1fab82984fedc5614604af28dd31226010.jpg)

交通按箭头方向流动, 从交叉点 $a$ 进入, 从 $d$ 离开, 并经过 $b$ 或 $c$ 或两者。以下数据值给出了连接各交叉点的道路信息:

<table><tr><td>从</td><td>到</td><td>时间</td><td>容量</td><td>敏感性</td></tr><tr><td>a</td><td>b</td><td>5.0</td><td>10</td><td>0.1</td></tr><tr><td>a</td><td>c</td><td>1.0</td><td>30</td><td>0.9</td></tr><tr><td>c</td><td>b</td><td>2.0</td><td>10</td><td>0.9</td></tr><tr><td>b</td><td>d</td><td>1.0</td><td>30</td><td>0.9</td></tr><tr><td>c</td><td>d</td><td>5.0</td><td>10</td><td>0.1</td></tr></table>

具体而言，我们假设时间单位为分钟，容量单位为每分钟车辆数，敏感性单位为每（每分钟车辆数）的分钟数。

以下 AMPL 语句可用于表示此类网络：

set inters; # 道路交叉点  
param EN symbolic in inters; # 网络入口  
param EX symbolic in inters; # 网络出口  
set roads within {i in inters, j in inters: i <> EX and j <> EN};  
param time {roads} > 0;  
param cap {roads} > 0;  
param sens {roads} > 0;

(a) 从该网络入口到出口的最短路径是多少分钟？构建一个最短路径模型，参照 图 15-7 的方式，验证你的答案。

(b) 从入口到出口，网络能够维持的最大交通流量是多少 (单位: 辆/分钟)？构建一个类似于 图 15-6 的最大流模型来验证你的答案。

(c) 问题 (a) 仅关注交通速度，而问题 (b) 只考虑交通流量。实际上，这两个量是相互关联的。随着道路上交通流量从零开始增加，通过该道路所需的时间也会增加。

当车辆很少时，行驶时间只会适度增加，但当交通流量接近道路 容量 (cap) 时，行驶时间会迅速增加。因此，需要用一个非线性函数来建模这一现象。我们通过以下 约束 (constraints) 来定义道路 $(i, j)$ 上的行驶 时间 (T)：

```
var X {roads} >= 0; # 进入道路 (i,j) 的车辆数 (单位: 辆/分钟)
var T {roads}; # 道路 (i,j) 的行驶时间
subject to travel Time {(i,j) in roads}
    T[i,j] = time[i,j] + (sens[i,j] * x[i,j]) / (1 - x[i,j]/cap[i,j]);
```

你可以确认，当道路使用较少时，$(i,j)$ 上的行驶时间接近 `time[i,j]`，但当使用量接近 `cap[i,j]` 辆/分钟时，行驶时间趋于无穷大。参数 `sens[i,j]` 的大小控制着随着更多车辆试图进入道路时，行驶时间增加的速率。

假设我们想分析网络如何最优地处理每分钟一定数量的车辆。目标是最小化从入口到出口的平均行驶时间：

```
param through > 0; # 使用网络的车辆数 (单位: 辆/分钟)
minimize Avg_Time: 
    (sum {(i,j) in roads} T[i,j] * x[i,j]) / through;
```

非线性表达式 $\mathrm{T}[i, j] \cdot \mathrm{X}[i, j]$ 表示道路 $(i,j)$ 上的行驶分钟数乘以进入该道路的车辆数（单位：辆/分钟）——即道路 $(i,j)$ 上的车辆总数。因此，目标函数中的求和给出了整个系统中的车辆总数。将该总和除以通过网络的车辆数（单位：辆/分钟），即可得到每辆车的平均行驶分钟数。

通过添加以下内容完成模型：

- 一个 约束 (constraint)，在除入口 (EN) 和出口 (EX) 之外的每个交叉口，进入的车辆总数（单位：辆/分钟）等于离开的车辆总数。
- 一个 约束 (constraint)，离开入口 (EN) 的车辆总数（单位：辆/分钟）等于使用网络的总流量（由 `through` 表示）。
- 对 变量 (variables) 设置适当的边界。（第 18.4 节中的示例应能提示需要哪些边界。）

使用 AMPL 证明，对于上述示例，当吞吐量为 4.0 辆/分钟时，最优策略是将一半车辆沿路径 $a \rightarrow b \rightarrow d$ 发送，另一半沿路径 $a \rightarrow c \rightarrow d$ 发送，此时平均行驶时间约为 8.18 分钟。

(d) 通过尝试大于和小于 4.0 的 `through` 参数值，绘制最小平均行驶时间随吞吐量变化的函数图像。同时，记录不同吞吐量下的最优解中使用了哪些行驶路线，并在你的图像上总结这些信息。

你的图中的信息与 (a) 和 (b) 部分的解之间有什么关系？

(e) (c) 中的模型假设你可以让汽车司机走特定的路线。例如，在吞吐量为 4.0 的最优解中，不允许司机从 $c$ “抄近路”到 $b$。

如果所有司机都可以自由选择他们喜欢的路线，会发生什么？观察表明，在这种情况下，交通往往会达到一种稳定解，其中任何路线的行驶时间都不小于平均值。吞吐量为 4.0 的最优解并不稳定，因为——正如你可以验证的——路线 $a \rightarrow c \rightarrow b \rightarrow d$ 上的行驶时间仅为约 7.86 分钟；如果有许可，一些司机会尝试抄近路。

为了使用 AMPL 找到一个稳定解，我们必须添加一些数据来指定从入口 (EN) 到出口 (EX) 的可能路线：

```
param choices integer > 0; # 路线数量
set route {1..choices} within roads;
```

这里 route 是一个索引集合的集合；对于每个 r 在 1..choices 中，表达式 route[r] 表示一个不同的道路子集，这些道路共同组成一条从 EN 到 EX 的路线。对于我们的网络，choices 应该是 3，路线集合应该是 $\{(a,b),(b,d)\}$、$\{(a,c),(c,d)\}$ 和 $\{(a,c),(c,b),(b,d)\}$。使用这些数据值，可以通过一组额外的约束条件来确保稳定性条件，这些约束条件表明任何路线的行驶时间不少于所有路线的平均行驶时间：

```
subject to Stability {r in 1..choices}:
    sum {(i,j) in route[r]} T[i,j] >= (sum {(i,j) in roads} T[i,j] * x[i,j]) / through;
```

证明在吞吐量为 4.0 的稳定解中，略多于 $5\%$ 的司机会抄近路，平均行驶时间增加到约 8.27 分钟。因此，如果从未修建从 $c$ 到 $b$ 的道路，交通会更快！（这种现象被称为 Braess 悖论，以一位交通分析师命名，他注意到当在慕尼黑的道路系统中添加某个链接后，交通似乎变得更糟。）

(f) 通过尝试吞吐量值大于和小于 4.0 的情况，绘制稳定行驶时间作为吞吐量函数的图。在图上标出哪些吞吐量下的稳定时间大于最优时间。

(g) 现在假设你被雇用来分析从 $a$ 直接到 $d$ 开设一条额外的蜿蜒道路的可能性，行驶时间为 5 分钟，容量为 10，敏感度为 1.5。使用上面开发的模型，写一份关于进行此更改后果的分析报告，作为吞吐量值的函数。

18-5. 回到上一练习 (e) 中构建的模型。本练习要求通过替换一些变量来减少变量数量，如第 18.2 节所述。

(a) 将选项 show_stats 设置为 1，并求解问题。从你得到的额外输出中，验证有 10 个变量。

接下来，重复执行该会话，并将选项 `substout` 设置为 1。通过查看输出信息，验证某些变量是否通过代入被消去。这些变量必须是哪些变量？

(b) 除了设置 `substout` 外，你还可以通过在变量声明中添加合适的 = 表达式，来指定某个变量应被代入消去。修改问题 (a) 中的模型以使用此功能，并验证结果是否相同。

(c) 本模型中有一个表示平均旅行时间的长表达式出现了两次。定义一个新的变量 `Avg` 来表示该表达式。验证在求解所得模型时，AMPL 也会代入消去该变量，并且结果与之前相同。

18-6. 在《使用 GINO 进行建模与优化》一书中，Judith Liebman、Leon Lasdon、Linus Schrage 和 Allan Waren 描述了设计用于储存氨气的钢制储罐时遇到的如下问题。决策变量为：

$T$ 储罐内部温度  
$I$ 保温层厚度  

储罐内部压力是温度的函数：

$$
P = e^{-3950 / (T + 460) + 11.86}
$$

目标是最小化储罐的成本，成本由三部分组成：保温成本（取决于厚度）；钢制容器成本（取决于压力）；以及用于冷却氨气的再冷凝过程成本（取决于温度和保温层厚度）。结合工程和经济方面的考虑，得出以下公式：

$$
\begin{array}{l}
C_I = 400I^{0.9} \\
C_V = 1000 + 22(P - 14.7)^{1.2} \\
C_R = 144(80 - T) / I
\end{array}
$$

(a) 在 AMPL 中将该问题表述为一个两变量问题，以及一个六变量问题（其中四个变量可以被代入消去）。你更倾向于使用哪种表述方式？

(b) 使用你偏好的表述方式，确定最低成本储罐的参数。

(c) 增大 $C_R$ 中的因子 144 会导致再冷凝成本成比例增加。尝试几个更大的值，并大致描述随着该因子增大，总成本如何随之增加。

18-7. 社会核算矩阵（social accounting matrix）是一张表格，用于显示经济中每个部门流向其他各部门的资金流。下面是一个简单的五部门示例，空白项表示已知为零的资金流：

<table>
<tr><td></td><td>LAB</td><td>H1</td><td>H2</td><td>P1</td><td>P2</td><td>总计</td></tr>
<tr><td>LAB</td><td>15</td><td>3</td><td>130</td><td>80</td><td>220</td><td></td></tr>
<tr><td>H1</td><td>?</td><td></td><td></td><td></td><td>?</td><td></td></tr>
<tr><td>H2</td><td>?</td><td></td><td></td><td></td><td>?</td><td></td></tr>
<tr><td>P1</td><td>15</td><td>130</td><td></td><td>20</td><td>190</td><td></td></tr>
<tr><td>P2</td><td>25</td><td>40</td><td>55</td><td></td><td>105</td><td></td></tr>
</table>

如果矩阵被完美估计，它将是平衡的：每一行的和（从一个部门流出的流量）将等于相应列的和（流入的流量）。然而，实际上存在几个困难：

- 一些条目（如上所述标记为 ?）没有可靠的估计值。

- 在估计的表中，行和不一定等于列和。

- 我们对每个部门的总流入（或流出）流量有单独的估计值，如表中行右侧所示。这些估计值不一定等于估计行或列的和。

可以使用非线性规划来调整该矩阵，通过找到在某种意义上最接近给定矩阵的平衡矩阵。

对于一组部门 $S$，令 $E_{T} \subseteq S$ 表示我们有估计总流量的部门子集，令 $E_{A} \subseteq S \times S$ 包含所有有已知估计值的部门对 $(i, j)$。给定的数据值为：

$t_i$ 估计的行/列和，$i \in E_T$  
$a_{ij}$ 估计的社会核算矩阵条目，$(i, j) \in E_A$

令 $S_A \subseteq S \times S$ 包含所有应在矩阵中有条目的行-列对 $(i, j)$ —— 这包括包含 ? 而不是数字的条目。我们希望确定调整后的条目 $A_{ij}$，对于每个 $(i, j) \in S_A$，这些条目是真正平衡的：

$$
\begin{array}{r}
\sum_{j\in S:(i,j)\in S_A}A_{ij} = \sum_{j\in S:(j,i)\in S_A}A_{ji}
\end{array}
$$

对所有 $i \in S$ 成立。你可以将这些方程视为变量 $A_{ij}$ 的约束。

没有最佳函数来衡量“接近”，但一个流行的选择是估计值与调整值之间差异的平方和 —— 对于矩阵和行与列的和 —— 并以估计值为尺度。为了方便，我们将调整后的和写成定义变量：

$$
T_{i} = \sum_{j\in S:(i,j)\in S_{A}}A_{ij}
$$

然后目标是最小化

$$
\begin{array}{r}
\sum_{(i,j)\in E_A}(a_{ij} - A_{ij})^2 /a_{ij} + \sum_{i\in E_T}(t_i - T_i)^2 /t_i
\end{array}
$$

为此问题建立一个 AMPL 模型，并确定一个最优的调整矩阵。

18-8. 一个管道网络的布局如下：

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/2704b0ee1d720ab432723b799488a2aa686677b15c6944852f80dd1f0528be55.jpg)

圆圈代表接头，箭头代表管道。接头 1 和 2 是流量的来源，接头 9 是流量的汇点或目的地，但管道中的流量可以是任一方向。与每个接头 $i$ 相关联的是在该接头处提取的流量 $w_i$ 和海拔高度 $e_i$：

<table>
<tr><td></td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td><td>7</td><td>8</td><td>9</td><td>10</td></tr>
<tr><td>$w_i$</td><td>0</td><td>0</td><td>200</td><td>0</td><td>0</td><td>200</td><td>150</td><td>0</td><td>0</td><td>150</td></tr>
<tr><td>$e_i$</td><td>50</td><td>40</td><td>20</td><td>20</td><td>0</td><td>0</td><td>0</td><td>20</td><td>20</td><td>20</td></tr>
</table>

我们的 决策 变量 (decision variables) 是从 $i$ 到 $j$ 的管道中的流量 $F_{ij}$，正值表示沿箭头方向的流动，负值表示相反方向的流动。自然地，在每个节点处流入的流量必须等于流出的流量加上在该节点提取的量，除了源点和汇点。

管道的 "水头损失" (head loss) 是推动流体通过管道所需能量的一种度量。在我们的情况下，从 $i$ 到 $j$ 的管道水头损失与流量的平方成正比：

$$
H_{ij} = Kc_{ij}F_{ij}^{2}\mathrm{~if~}F_{ij} > 0,
$$

$$
H_{ij} = -Kc_{ij}F_{ij}^2\mathrm{~if~}F_{ij}< 0,
$$

其中 $K = 4.96407\times 10^{- 6}$ 是一个转换常数，$c_{ij}$ 是根据管道的直径、摩擦系数和长度计算出的一个因子：

<table><tr><td>from</td><td>to</td><td>cij</td></tr><tr><td>1</td><td>3</td><td>6.36685</td></tr><tr><td>2</td><td>4</td><td>28.8937</td></tr><tr><td>3</td><td>10</td><td>28.8937</td></tr><tr><td>3</td><td>5</td><td>6.36685</td></tr><tr><td>3</td><td>8</td><td>43.3406</td></tr><tr><td>4</td><td>10</td><td>28.8937</td></tr><tr><td>4</td><td>6</td><td>28.8937</td></tr><tr><td>5</td><td>6</td><td>57.7874</td></tr><tr><td>5</td><td>7</td><td>43.3406</td></tr><tr><td>6</td><td>7</td><td>28.8937</td></tr><tr><td>8</td><td>4</td><td>28.8937</td></tr><tr><td>8</td><td>9</td><td>705.251</td></tr></table>

对于处于相同高程的两个节点 $i$ 和 $j$，从 $i$ 到 $j$ 流动时的压力降等于水头损失。压力和水头损失都以英尺为单位测量，因此在对节点之间的高程差进行修正后，我们有如下关系：

$$
H_{ij} = (P_i + e_i) - (P_j + e_j)
$$

最后，我们希望保持源点和汇点的压力均为 200 英尺。

(a) 为这种情况建立一个通用的 AMPL 模型，并整理上述数据的 数据 语句 (data statements)。

(b) 这里没有目标函数，但你仍然可以使用一个非线性 求解器 (solver) 来寻找一个可行解。通过将选项 show_stats 设置为 1，确认 变量 数量等于方程数量，从而在解中没有 "自由度"。（但这并不能保证解是唯一的。）

检查你的 求解器 是否找到了方程的解，并 显示 (display) 结果。

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

# AMPL 参考手册

AMPL 是一种用于代数建模和数学规划的语言：一种用于以代数符号表达优化问题（如线性规划）的计算机可读语言。本附录总结了 AMPL 的特性，特别强调了在前几章中未完全涵盖的技术细节。然而，并非每个特性、构造和选项都被列出；AMPL 网站 www.ampl.com 包含最新和最完整的信息。

使用以下符号约定。字面文本以等宽字体打印，而句法类别以斜体字体打印。用斜方括号 $\llbracket$ 和 $\rrbracket$ 括起来的短语或子短语是可选的，带有下标 opt 的构造也是可选的。

# A.1 词法规则

AMPL 模型涉及 变量 (variable)、 约束 (constraint) 和目标，这些通过 集合 (set) 和 参数 (parameter) 来表达。这些被称为模型实体 (model entity)。每个模型实体都有一个字母数字名称：由一个或多个 Unicode UTF-8 字母、数字和下划线组成的字符串，其格式不能与数值常量混淆。大写字母与小写字母是不同的。

数值常量以标准科学记数法书写：可选的符号、可能包含小数点的一串数字，以及以字母 d、D、e 或 E 开头的可选指数字段，例如 1.23D-45。AMPL 中的所有算术运算都使用相同的精度（在大多数机器上为双精度），因此所有指数表示法是同义的。

字面量 (literal) 是由单引号 ' 或双引号 " 分隔的字符串；如果字面量中出现分隔字符，则必须将其加倍，例如 $\prime \times \prime \prime \gamma^{\prime}$，这是一个包含三个字符 $\mathbf{u}^{\prime}\gamma$ 的字面量。换行符只有在前面有 $\backslash$ 时才能出现在字面量中。分隔符的选择是任意的；'abc' 和 "abc" 表示相同的字面量。

字面量与数值常量不同：1 和 '1' 没有关系。

输入是自由格式的；空白符（空格、制表符或换行符的任意序列）可以出现在任何标记之间。每个语句以分号结尾。

注释以 # 开头，并延伸到当前行的末尾，或者由 $f^{\star}$ 和 $\star /$ 分隔，在这种情况下它们可以跨越多行且不嵌套。注释可以出现在声明、命令和数据中的任何位置。

以下词汇是保留的，不能在其他上下文中使用：

<table><tr><td>Current</td><td>complements</td><td>integer</td><td>solve_result_num</td></tr><tr><td>IN</td><td>contains</td><td>less</td><td>suffix</td></tr><tr><td>INOUT</td><td>default</td><td>logical</td><td>sum</td></tr><tr><td>Infinity</td><td>dimen</td><td>max</td><td>symbolic</td></tr><tr><td>Initial</td><td>div</td><td>min</td><td>table</td></tr><tr><td>LOCAL</td><td>else</td><td>option</td><td>then</td></tr><tr><td>OUT</td><td>environ</td><td>setof</td><td>union</td></tr><tr><td>all</td><td>exists</td><td>shellExitCode</td><td>until</td></tr><tr><td>binary</td><td>forall</td><td>solveExitCode</td><td>while</td></tr><tr><td>by</td><td>if</td><td>solve_message</td><td>within</td></tr><tr><td>check</td><td>in</td><td>solve_result</td><td></td></tr></table>

以下划线开头的词也是保留的。其他关键字、函数名等是预定义的，但其含义可以重新定义。例如，词 prod 预定义为类似于 sum 的乘积运算符，但它可以在声明中重新定义，例如

set prod; # products

一旦一个词被重新定义，原始含义就无法访问。

AMPL 为几个关键字和运算符提供了同义词；首选形式在左侧。

\*  $\begin{array}{rl}{=}&{= =}\\{< >}&{!=}\\{\mathrm{and}}&{\& \&}\\{\mathrm{not}}&{!}\\{\mathrm{or}}&{||}\\{\mathrm{prod}}&{\mathrm{product}}\end{array}$

# A.2 集合成员



一个集合 (set) 包含零个或多个元素或成员 (element or member)，每个元素都是一个由一个或多个组件 (component) 组成的有序列表。集合的每个成员必须是不同的。所有成员必须具有相同数量的组件；这个公共数量被称为集合的维度 (dimension)。

文字集合 (literal set) 写作花括号 `{` 和 `}` 之间以逗号分隔的成员列表。如果集合是一维的，则成员简单地是数值常量或文字字符串，或任何求值为数字或字符串的表达式：

$$
\begin{array}{rl}
& \{\text{"a", "b", "c"}\} \\
& \{1, 2, 3, 4, 5, 6, 7, 8, 9\} \\
& \{\text{t, t + 1, t + 2}\}
\end{array}
$$

对于多维集合，每个成员必须写成上述形式的带括号的逗号分隔列表：

$$
\begin{array}{rl}
& \{("a", "2"), ("a", "3"), ("b", "5")\} \\
& \{(1, 2, 3), (1, 2, 4), (1, 2, 5), (1, 3, 7), (1, 4, 6)\}
\end{array}
$$

数值成员的值是将其十进制表示四舍五入为浮点数的结果。看起来不同但四舍五入后得到相同浮点数的数值成员，例如 1 和 0.01E2，被认为是相同的。

# A.3 索引表达式和下标

AMPL 中的大多数实体 (entity) 都可以定义为在集合上索引的集合；通过在实体名称后附加带括号的下标来选择单个项目。可能下标的范围在实体声明中的索引表达式 (indexing expression) 中指示。求和 (sum) 等归约算子也使用索引表达式来指定迭代操作的集合。

下标 (subscript) 是一个由逗号分隔的符号或数值表达式列表，用方括号括起来，如 `supply[i]` 和 `cost[j, p[k] + 1, "O+"]`。每个下标表达式必须求值为数字或文字。结果值或值序列必须给出相关的一维或多维索引集合的一个成员。

索引表达式是一个由逗号分隔的集合表达式列表，后面可选地跟一个冒号和一个逻辑 "such that" 表达式，所有内容都用花括号括起来：

indexing: `{sexpr-list}`  
`{sexpr-list : lexpr}`  
sexpr-list:  
`sexpr`  
`dummy-member in sexpr`  
`sexpr-list , sexpr`

每个集合表达式前面可以有一个虚拟成员 (dummy member) 和关键字 `in`。一维集合的虚拟成员是一个未绑定的名称，即当前未定义的名称。多维集合的虚拟成员是一个用括号括起来的由表达式或未绑定名称组成的逗号分隔列表；该列表必须至少包含一个未绑定的名称。

虚拟成员会引入一个或多个虚拟索引 (dummy indices，即其组成部分中的未绑定名称)，这些索引的作用域 (scope) 或定义范围 (range of definition) 从其后紧接的 `sexpr` 开始；索引的作用域一直延续到使用该索引表达式的声明结束，或使用该索引表达式的操作数结束为止。当一个虚拟成员含有一个或多个表达式组成部分时，该虚拟成员中的虚拟索引会遍历集合的一个切片 (slice)，也就是说，它们会取所有使得该虚拟成员属于该集合的值。

{A} # 集合  
{A, B} # 所有对，分别来自 A 和 B  
{i in A, j in B} # 同上  
{i in A, B} # 同上  
{i in A, C[i]} # 所有对，分别来自 A 和 C[i]  
{i in A, (j,k) in D} # 来自 A 的一个元素和来自 D 的一个（本身为对的）元素  
{i in A: p[i] > 0} # A 中所有使得 p[i] 为正的 i  
{i in A, j in C[i]: i <= j} # i 和 j 必须为数值  
{i in A, (i,j) in D: i <= j} # 所有满足 i 在 A 中且 i, j 在 D 中的对  
#（i 值相同）且 i <= j  

索引表达式中的可选部分 `: lexpr` 只选择满足逻辑表达式的成员，并排除其他成员。lexpr 通常涉及索引表达式的一个或多个虚拟索引。

# A.4 表达式

在 AMPL 的算术表达式和逻辑表达式中可以组合各种项目。不能包含变量的表达式称为常量表达式 (constant expression, cexpr)。

<table><tr><td>优先级</td><td>名称</td><td>类型</td><td>备注</td></tr><tr><td>1</td><td>if-then-else</td><td>A, S</td><td>A: 若无 else，则假定为 "else 0"  
S: 必须有 "else sexpr"</td></tr><tr><td>2</td><td>or ||</td><td>L</td><td></td></tr><tr><td>3</td><td>exists forall</td><td>L</td><td>逻辑归约运算符</td></tr><tr><td>4</td><td>and &&</td><td>L</td><td></td></tr><tr><td>5</td><td>< > <= >=</td><td>L</td><td></td></tr><tr><td>6</td><td>in not in</td><td>L</td><td>集合成员关系</td></tr><tr><td>6</td><td>within not within</td><td>L</td><td>S within T 表示集合 S⊆ 集合 T</td></tr><tr><td>7</td><td>not !</td><td>L</td><td>逻辑否定</td></tr><tr><td>8</td><td>union diff symdiff</td><td>S</td><td>symdiff = 对称差集</td></tr><tr><td>9</td><td>inter</td><td>S</td><td>集合交集</td></tr><tr><td>10</td><td>cross</td><td>S</td><td>叉积或笛卡尔积</td></tr><tr><td>11</td><td>setof .. by</td><td>S</td><td>集合构造器</td></tr><tr><td>12</td><td>+ - less</td><td>A</td><td>a less b ≡ max(a - b, 0)</td></tr><tr><td>13</td><td>sum prod min max</td><td>A</td><td>算术归约运算符</td></tr><tr><td>14</td><td>* / div mod</td><td>A</td><td>div ≡ 整数的截断商</td></tr><tr><td>15</td><td>+ -</td><td>A</td><td>一元加号，一元减号</td></tr><tr><td>16</td><td>^ **</td><td>A</td><td>指数运算</td></tr></table>

运算符按优先级递增顺序列出。指数运算和 if-then-else 是右结合的；其他运算符是左结合的。“Type”列指示结果类型：A 表示算术类型，L 表示逻辑类型，S 表示集合类型。

表 A-1：算术、逻辑和集合运算符。

即使可能涉及虚拟索引。逻辑表达式（记作 lexpr）在作为 cexpr 的一部分时不得包含变量。集合表达式记作 sexpr。

表 A-1 总结了算术、逻辑和集合运算符；“type”列指示运算符产生的是算术值（A）、逻辑值（L）还是集合值（S）。

算术表达式由常用的算术运算符、内置函数以及算术归约运算符（如 sum）构成：

expr: number variable expr arith-op expr arith-op is + - less * / mod div ** unary-op expr unary-op is + - built-in(exprlist) if lexpr then expr [ else expr ] reduction-op indexing expr reduction-op is sum prod max min ( expr )

内置函数列于表 A-2 中。

算术归约运算符 (reduction-op) 用于如下表达式中：

sum {i in Prod} cost[i] * Make[i]

索引表达式 (indexing expr) 的范围延伸至 expr 的末尾。如果操作是针对一个空集，则结果为该操作的单位元：sum 为 0，prod 为 1，min 为 Infinity，max 为 -Infinity。

逻辑表达式 (lexpr) 出现在需要 “true” 或 “false” 值的地方：在 check 语句中，在索引表达式（冒号后）的 “such that” 部分，以及在 if lexpr then ... else ... 表达式中。在这些上下文中出现的数值会被隐式转换为逻辑值：0 被解释为 false，所有其他数值被解释为 true。

lexpr:

expr expr compare-op expr compare-op is  $\begin{array}{rlr}{< }&{< =}&{=}&{=}&{!=}&{< >}&{>}&{> =}\end{array}$  lexpr logic-op lexpr logic-op is or and && not lexpr member in sexpr member not in sexpr sexpr within sexpr sexpr not within sexpr opname indexing lexpr opname is exists or forall lexpr )

in 运算符测试集合成员关系。其左操作数是一个潜在的集合成员，即一个表达式或括号内以逗号分隔的表达式列表，表达式的数量必须等于右操作数（必须是一个集合表达式）的维度。within 运算符测试一个集合是否包含于另一个集合中。两个集合操作数必须具有相同的维度。

逻辑归约运算符 exists 和 forall 分别是 or 和 and 的迭代形式。当应用于空集时，exists 返回 false，forall 返回 true。

集合表达式 (sexpr) 生成集合。

sexpr: { [ member [ , member... ] ] } sexpr set-op sexpr opname indexing sexpr expr .. expr [ by expr ] setof indexing member if lexpr then sexpr else sexpr sexpr interval infinite-set indexing

成员的组成部分可以是任意常量表达式。A.6.3 节描述了区间和无限集。

当用作二元运算符时，union 和 inter 表示二元集合运算中的并集和交集。这些关键字也可以用作归约运算符。

... 运算符用于构造集合。默认的 by 子句是 by 1。一般来说，$e_1 \ldots e_2$ by $e_3$ 表示数字

$$
e_1,e_1 + e_3,e_1 + 2e_3,\ldots ,e_1 + \left\lfloor \frac{e_2 - e_1}{e_3}\right\rfloor e_3
$$

四舍五入到集合成员。（符号 $\lfloor x\rfloor$ 表示 $x$ 的向下取整，即小于或等于 $x$ 的最大整数。）

setof 运算符是集合构造运算符；member 是表达式或用括号括起来的以逗号分隔的表达式列表。结果集合由通过迭代索引表达式获得的所有成员组成；结果表达式的维度是 member 中组件的数量。

abs $(x)$ 绝对值 $|x|$  
acos $(x)$ 反余弦 $\cos^{- 1}(x)$  
acosh $(x)$ 反双曲余弦 $\cosh^{- 1}(x)$  
alias $(V)$ 模型实体 $\nu$ 的别名  
asin $(x)$ 反正弦 $\sin^{- 1}(x)$  
asinh $(x)$ 反双曲正弦 $\sinh^{- 1}(x)$  
atan $(x)$ 反正切 $\tan^{- 1}(x)$  
atan2 $(y,x)$ 反正切 $\tan^{- 1}(y / x)$  
atanh $(x)$ 反双曲正切 $\tanh^{- 1}(x)$  
ceil $(x)$ 向上取整 $\mathcal{X}$（下一个较大的整数）  
ctime() 当前时间作为字符串  
ctime(t) 时间 $t$ 作为字符串  
cos $(x)$ 余弦  
exp $(x)$ $e^x$  
floor $(x)$ 向下取整 $\mathcal{X}$（下一个较小的整数）  
$\log (x)$ $\log_e(x)$  
$\log_{10}(x)$  
$\max (x,y,\ldots)$ 最大值（两个或多个参数）  
min $(x,y,\ldots)$ 最小值（两个或多个参数）  
precision $(x,n)$ $\mathcal{X}$ 四舍五入到 $n$ 位有效十进制数字  
round $(x,n)$ $\mathcal{X}$ 四舍五入到小数点后 $n$ 位  
round $(x)$ $\mathcal{X}$ 四舍五入到整数  
sin $(x)$ 正弦  
$\sinh (x)$ 双曲正弦  
sqrt $(x)$ 平方根  
tan $(x)$ 正切  
$\tan (x)$ 双曲正切  
time() 当前时间（以秒为单位）  
trunc $(x,n)$ $\mathcal{X}$ 截断到小数点后 $n$ 位  
trunc $(x)$ $\mathcal{X}$ 截断到整数  

表 A-2：内置算术函数。

amp1: set $\textbf{y} =$ setof {i in 1. .5} (i,i^2);  
amp1: display $\mathbf{y}$;  
set $\texttt{y} \coloneqq \texttt{(1,1)} \texttt{(2,4)} \texttt{(3,9)} \texttt{(4,15)} \texttt{(5,25)}$;

# A.4.1 内置函数

内置算术函数列于表 A-2 中。函数 alias 接受一个模型实体名称作为参数，并返回其别名，该别名是一个在 A.5 节中描述的字面值。函数 $\operatorname{round}(x,n)$ 和 $\operatorname{trunc}(x,n)$ 将 $x$ 转换为十进制字符串，并将其四舍五入或截断到小数点后 $n$ 位（若 $n<0$ 则为小数点前 $-n$ 位）；类似地，函数 precision$(x,n)$ 将 $x$ 四舍五入为 $n$ 个有效十进制数字。对于 round 和 trunc 函数，若 $n$ 缺失则默认为 0；因此提供通常的四舍五入或截断为整数的功能。

几种内置随机数生成函数可用，如表 A-3 所列。所有这些函数均基于一个周期非常长的均匀随机数生成器。可通过命令行参数 $-sn$（A.23）或选项 randseed 指定初始种子 $n$，而 $-s$ 或

Beta(a,b) 密度 $(x) = x^{a - 1}(1 - x)^{b - 1} / (\Gamma (a)\Gamma (b) / \Gamma (a + b)),$ $x$ 在 [0, 1] 区间内  
Cauchy() 密度 $(x) = 1 / (\pi (1 + x^{2}))$  
Exponential() 密度 $(x) = e^{- x}$，$x > 0$  
Gamma(a) 密度 $(x) = x^{a - 1}e^{- x} / \Gamma (a)$，$x\geq 0$，$a > 0$  
Irand224() 在 $[0,2^{24})$ 上的整数均匀分布  
Normal$(\mu ,\sigma)$ 均值为 $\mu$，方差为 $\sigma$ 的正态分布  
Normal01() 均值为 0，方差为 1 的正态分布  
Poisson$(\mu)$ 概率 $(k) = e^{- \mu}\mu^{k} / k!,k = 0,1,\ldots$  
Uniform$(m,n)$ 在 $[m,n)$ 上的均匀分布  
Uniform01() 在 [0, 1) 上的均匀分布

表 A-3：内置随机数生成函数。

选项 randseed '' 指示 AMPL 选择并打印一个种子。不提供 -s 参数等同于指定 -s1。

Irand224() 返回区间 $[0, 2^{24})$ 中的一个整数。给定相同的种子，形如 floor$(m * \mathrm{Irand}224() / n)$ 的表达式在大多数计算机上将产生相同的值，当 $m$ 和 $n$ 是合理大小的整数表达式时，即 $|n| < 2^{k - 24}$ 且 $|m| < 2^{k}$，对于能正确计算 $k$ 位浮点整数乘积和商的机器；对于当前大多数感兴趣的机器而言 $k \geq 47$。对集合进行操作的函数在 A.6 节中描述。

对集合进行操作的函数在 A.6 节中描述。

# A.4.2 字符串与正则表达式

在模型或命令中几乎所有可以使用字面字符串的地方，也可以使用括号括起来的字符串表达式。字符串通过连接以及表 A-4 中列出的内置字符串和正则表达式函数创建。

字符串连接运算符 & 将其参数连接成一个字符串；它的优先级低于所有算术运算符。数值操作数会被转换为完整精度的十进制字符串，如同通过 printf 格式 %g（见 A.16 节）。

s & t 连接字符串 $s$ 和 $t$  
num(s) 将字符串 $s$ 转换为数字；如果去除前导和尾随空格后不能得到一个有效的十进制数，则报错  
num0(s) 去除前导空格，并尽可能多地将 $s$ 解释为数字，但不引发错误  
ichar(s) 字符串 $s$ 中第一个字符的 Unicode 值  
char(n) 字符 $n$ 的字符串表示；ichar 的逆操作  
length(s) 字符串 $s$ 的长度  
substr(s, m, n) 从位置 $m$ 开始的字符串 $s$ 的 $n$ 字符子串；如果省略 $n$，则为字符串的剩余部分  
sprintf(fmt, exprlist_opt) 根据格式字符串 fmt 格式化参数  
match(s, re) 正则表达式 $r$ 在字符串中的起始位置，若未找到则返回 0  
sub(s, re, repl) 将字符串 $s$ 中第一个匹配正则表达式 $r$ 的部分替换为 repl  
gsub(s, re, repl) 将字符串 $s$ 中所有匹配正则表达式 $r$ 的部分替换为 repl  

表 A-4：内置字符串和正则表达式函数。

字符串不会隐式转换为数字，但 $\mathrm{num}(s)$ 和 $\mathrm{num}0(s)$ 可执行显式转换。两者都会忽略前导和尾随空格；如果剩下的部分不是有效数字，num 会报错，而 num0 会尽可能多地转换，如果没有看到数字前缀则返回 0。

match、sub 和 gsub 函数接受表示正则表达式的字符串作为第二个参数。AMPL 正则表达式类似于标准 Unix 正则表达式。除了某些元字符外，任何字面字符 $c$ 都是一个匹配自身在目标字符串中出现的正则表达式。元字符 `*` 是一个匹配任意字符的正则表达式。用方括号括起来的字符列表是一个匹配列表中任意字符的正则表达式，并且列表可以缩写：`[a-z0-9]` 匹配任何小写字母或数字。以字符 `^` 开头并用方括号括起来的字符列表是一个匹配列表外任意字符的正则表达式：`[^0-9]` 匹配任何非数字字符。如果 $r$ 是一个正则表达式，则 $r^*$ 匹配 0 个或多个 $r$，$r+$ 匹配 1 个或多个 $r$，而 $r?$ 匹配 0 个或 1 个 $r$。`^r` 仅在 $r` 出现在字符串开头时匹配 $r$，而 $r$` 仅在字符串末尾匹配 $r$。括号用于分组，`|` 表示“或”，$r_1|r_2$ 匹配 $r_1$ 或 $r_2$。通过在元字符前加上反斜杠可以关闭其特殊含义。

在 sub 和 gsub 的替换模式（第三个参数）中，`&` 和 `\0` 都代表整个匹配的字符串，而 `\1, \2, ..., \9` 分别代表模式中第一个、第二个、……、第九个括号表达式匹配的字符串。

选项 (A.14.1) 是命名的字符串值，其中一些会影响各种 AMPL 命令 (A.14)。选项 opname 的当前值表示为 $\S$ opname。

# A.4.3 分段线性项

在变量、约束和目标声明中，分段线性项具有以下形式之一：

$$
\begin{array}{rl}
& << breakpoints~;~slopes >> var\\
& << breakpoints~;~slopes >> (cexpr)\\
& << breakpoints~;~slopes >> (var,~cexpr)\\
& << breakpoints~;~slopes >> (cexpr,~cexpr)
\end{array}
$$

其中 breakpoints 是断点列表，slopes 是斜率列表。每个这样的列表都是由逗号分隔的 cexpr 列表，每个 cexpr 前面可以有一个索引表达式（其作用域仅覆盖该 cexpr）。索引表达式必须指定一个明显有序的集合（见 A.6.2），或者可以是特殊形式 `{if lexpr}`。

如果 lexpr 为假，则导致 expr 被省略。在命令中，还允许使用更一般的形式

$$
\begin{array}{rl}
& << breakpoints~;~slopes >> (expr)\\
& << breakpoints~;~slopes >> (expr,~expr)
\end{array}
$$

并且变量可以出现在断点和斜率的表达式中。

在通过索引扩展任何索引表达式后，斜率列表的长度必须比断点列表的长度多一，并且断点必须按非递减数值顺序排列。（对斜率的顺序没有要求。）AMPL 将结果解释为如下定义的分段线性函数 $f(x)$。设 $s_j$，$1 \leq j \leq n$，和 $b_i$，$1 \leq i \leq n - 1$，分别表示斜率和断点，并令

$b_{0} = - \infty$ 和 $b_{n} = +\infty$。则 $f(0) = 0$，且对于 $b_{i - 1}\leq x\leq b_{i}$，$f$ 的斜率为 $s_i$，即 $f^{\prime}(x) = s_{i}$。对于只有一个参数的形式（变量 var 或常量表达式 expr），结果为 $f(var)$ 或 $f(expr)$。具有两个操作数的形式被解释为 $f(var) - f(expr)$。这会添加一个常数，使得当 var 等于 expr 时结果为零。

当分段线性项出现在其他线性约束或目标中时，AMPL 会将涉及同一变量的两个或多个分段线性项合并为一个单项。

# A.5 模型实体的声明

模型实体的声明具有以下通用形式：

entity name alias opt indexing opt body opt

其中 name 是一个字母数字名称，该名称之前未通过声明分配给任何实体，alias 是一个可选的字面量，indexing 是一个可选的索引表达式，entity 是以下关键字之一：

set param var arc minimize maximize subject to node

此外，还有几种其他结构在技术上属于实体声明，但将在后面描述；这些包括 environ、problem、suffix 和 table。

实体可以省略，在这种情况下，假定其服从 (subject to)。各种声明的主体由初始部分之后的其他短语组成，这些短语大多是可选的。每个声明以分号结尾。

声明可以以任意顺序出现，但每个名称必须在其被使用之前声明。与分段线性项类似，变量、约束和目标声明允许使用一种特殊的索引表达式形式：

$$
\{\texttt{if}\ l e x p r\}
$$

如果逻辑表达式 lexpr 为真，则产生一个简单的（无下标的）实体；否则该实体将被排除在模型之外，后续尝试引用它时将导致错误消息。例如，如果参数 testing 被赋予一个大于 100 的值，则以下声明将变量 Test 包含在模型中：

param testing; var Test {if testing $>$ 100} $>= 0$ .

# A.6 集合声明

集合声明的形式为

set declaration: set name alias opt indexing opt attributes opt，其中 attributes 是一个属性列表，属性之间可以用逗号分隔（可选）：

attribute: dimen n within sexpr $=$ sexpr defaults sexpr

集合的维度是常数正整数 $n$，或者是 sexpr 的维度，或者默认为 1。短语 within sexpr 要求所声明的集合是 sexpr 的子集。可以给出多个 within 短语。$=$ 短语指定集合的值；这意味着该集合不会在数据部分（A.12.1）或 let（A.18.9）等命令中被赋予值。default 短语为集合指定一个默认值，用于在数据部分未给出值时使用。$=$ 和 default 短语互斥。如果两者都未给出，且集合未通过数据语句定义，则在模型生成过程中引用该集合将导致错误消息。由于历史原因，$\coloneqq$ 目前在集合和参数声明中是 $=$ 的同义词，但这种 $\coloneqq$ 的用法已被弃用。

在 $=$ 或 default 短语中的 sexpr 可以是 $\{\}$，即空集，其维度由声明中的任何 dimen 或 within 短语推断得出，如果两者都不存在，则维度为 1。在其他上下文中，$\{\}$ 表示空集。

允许对索引集合进行递归定义，只要所赋的值能够按顺序计算，且仅引用先前计算的值。例如，

set nodes; set arcs within nodes cross nodes; param max_iter $=$ card(nodes) - 1; # card(s) $=$ s 中元素的数量 set step {s in 1..max_iter} dimen $\begin{array}{rl}{2}&{=}\end{array}$ if $\texttt{s} == \texttt{1}$ then arcs else step[s-1] union setof {k in nodes, (i,k) in step[s-1], (k,j) in step[s-1]} (i,j); set reach $=$ step[max_iter];

在集合 reach 中计算由 nodes 和 arcs 表示的图的传递闭包。

# A.6.1 基数和元数函数

函数 `card` 作用于任何有限集合 `card(sexpr)`，返回 `sexpr` 中成员的数量。如果 `sexpr` 是一个索引表达式，则括号可以省略。例如，

$$
\mathtt{card}(\{\mathtt{i}\mathtt{in}\mathtt{A}\colon \mathtt{x}[\mathtt{i}] >= 4\})
$$

也可以写成

$$
\mathtt{card}\{\mathtt{i}\mathtt{in}\mathtt{A}\colon \mathtt{x}[\mathtt{i}] >= 4\}
$$

函数 `arity` 返回其集合参数的元数（arity）；函数 `indexarity` 返回其参数索引集合的元数。

# A.6.2 有序集合

一个具名的一维集合可以具有与其关联的顺序。有序集合的声明包括以下短语之一：

ordered [ by [reversed ]sexpr ] circular [ by [reversed ]sexpr ]

关键字 `circular` 表示该集合是有序的，并且是循环的，即其第一个成员是最后一个成员的后继，而最后一个成员是第一个成员的前驱。

目前，二维或更高维度的集合不能被定义为有序或循环。

如果集合 S 是由 T 排序的，或由 T 循环排序的，那么集合 T 本身必须是一个包含 S 的有序集合，并且 S 继承 T 的顺序。如果排序短语是 `by reversed T`，那么 S 仍然从 T 继承其顺序，但顺序是相反的。

如果 S 是有序或循环的，且未指定按 `sexpr` 排序，则适用以下两种情况之一。如果 S 的成员是通过模型中的 $\mathrm{a} = \{\text{member - list}\}$ 表达式显式指定的，或通过数据段中的成员列表指定的，则 S 采用该列表的顺序。如果 S 是通过模型中的赋值或默认 `sexpr` 计算得到的，那么 AMPL 将保留明显有序集合（如下所述）的顺序，否则可以任意选择顺序。

有序集合的函数总结在表 A-5 中。

如果 $S$ 是一个具有 $n$ 个成员的有序集合 (ordered set) 的表达式，$e$ 是 $S$ 的第 $j$ 个成员，并且它是一个整数 (integer) 表达式，那么 $\mathtt{next}(e,S,k)$ 是 $S$ 的第 $j + k$ 个成员，如果 $1\leq j + k\leq n$，否则为错误。如果 $S$ 是循环的，那么 $\mathtt{next}(e,S,k)$ 是 $S$ 的第 $1 + ((j + k - 1) \bmod n)$ 个成员。函数 $\mathtt{nextw}$（带环绕的 next）与 $\mathtt{next}$ 相同，不同之处在于它将所有有序集合视为循环的；$\mathtt{prev}(e,S,k)\equiv \mathtt{next}(e,S, - k)$，以及 $\mathtt{prevw}(e,S,k)\equiv \mathtt{nextw}(e,S, - k)$。

允许几种缩写形式。如果未给出 $k$，则默认为 1。如果同时省略 $k$ 和 $S$，那么 $e$ 必须是在索引表达式作用于 $S$ 范围内的一个虚拟索引，例如在 $\{e$ in $S\}$ 中。

有五个其他函数适用于有序集合：$\mathtt{first}(S)$ 返回 $S$ 的第一个成员，$\mathtt{last}(S)$ 返回最后一个成员，$\mathtt{member}(j,S)$ 返回 $S$ 的第 $j$ 个成员，$\mathtt{ord}(e,S)$ 和 $\mathtt{ord0}(e,S)$ 返回成员 $e$ 在 $S$ 中的序数位置。如果 $e$ 不在 $S$ 中，则 $\mathtt{ord0}$ 返回 0，而 $\mathtt{ord}$ 会报错。同样，如果 $e$ 是某个针对 $S$ 的索引表达式作用域中的哑指标 (dummy index)，则 $\mathsf{ord}(e) = \mathsf{ord}(e,S)$ 且 $\mathsf{ord}0(e) = \mathsf{ord}0(e,S)$。

一些集合显然是有序的，例如算术级数 (arithmetic progressions)、区间 (intervals)、有序集合的子集 (subsets of ordered sets)、then 和 else 子句均为有序集合的 if 表达式，以及左操作数为有序集合的集合差集 (set differences)（但不包括对称差集 (symmetric differences)）。

# A.6.3 区间与其他无限集合

A.6.3 区间与其他无限集合 为了方便指定有序集合以及为导入函数 (imported functions)（A.22 节）提供原型，存在几种无限集合 (infinite sets)。禁止在无限集合上进行迭代，但可以检查元素是否属于这些集合，并利用它们来指定集合的排序。最自然的无限集合是区间 (interval)，它可以是闭区间 (closed)、开区间 (open) 或半开区间 (half-open)，并遵循标准数学记号。存在实数 (浮点数) 和整数 (integer) 的区间，分别由关键字 interval 和 integer 引入：

next $(e,S,k)$ 集合 $S$ 中成员 $e$ 之后第 $k$ 个位置的成员  
next $(e,S)$ 与上相同，但 $k = 1$  
next $(e)$ 成员 $e$ 所属集合的下一个成员（$e$ 为哑指标）  
nextw $(e,S,k)$ 集合 $S$ 中成员 $e$ 之后第 $k$ 个位置的成员，循环处理  
nextw $(e,S)$ next $(e,S)$ 的循环版本  
nextw $(e)$ next $(e)$ 的循环版本  
prev $(e,S,k)$ 集合 $S$ 中成员 $e$ 之前第 $k$ 个位置的成员  
prev $(e,S)$ 与上相同，但 $k = 1$  
prev $(e)$ 成员 $e$ 所属集合的前一个成员（$e$ 为哑指标）  
prevw $(e,S,k)$ 集合 $S$ 中成员 $e$ 之前第 $k$ 个位置的成员，循环处理  
prevw $(e,S)$ prev $(e,S)$ 的循环版本  
prevw $(e)$ prev $(e)$ 的循环版本  
first $(S)$ $S$ 的第一个成员  
last $(S)$ $S$ 的最后一个成员  
member $(j,S)$ $S$ 的第 $j$ 个成员；$1\leq j\leq \operatorname {card}(S)$，$j$ 为整数  
ord $(e,S)$ 成员 $e$ 在 $S$ 中的序数位置  
ord $(e)$ 成员 $e$ 在以其为哑指标的集合中的序数位置  
ord0 $(e,S)$ 成员 $e$ 在 $S$ 中的序数位置；若不存在则为 0  
ord0 $(e)$ 与 ord $(e)$ 相同  
card $(S)$ 集合 $S$ 中成员的数量  
arity $(S)$ 若 $S$ 是集合则为其元数 (arity)，否则为 0；用于 _SETS 的 indexarity  
indexarity $(E)$ 实体 $E$ 的索引集合的元数  
card、arity 和 indexarity 也适用于无序集合  

表 A-5：有序集合的函数。

# interval:

区间 $[a,b]\equiv \{x\colon a\leq x\leq b\}$  
区间 $(a,b]\equiv \{x\colon a< x\leq b\}$  
区间 $[a,b)\equiv \{x\colon a\leq x< b\}$  
区间 $(a,b)\equiv \{x\colon a< x< b\}$  
整数区间 $[a,b]\equiv \{x\colon a\leq x\leq b$ 且 $x\in I\}$  
整数区间 $(a,b]\equiv \{x\colon a< x\leq b$ 且 $x\in I\}$  
整数区间 $[a,b)\equiv \{x\colon a\leq x< b$ 且 $x\in I\}$  
整数区间 $(a,b)\equiv \{x\colon a< x< b$ 且 $x\in I\}$  

其中 $a$ 和 $b$ 表示任意算术表达式，$I$ 表示整数集。在函数原型 (A.22) 和声明短语中：

区间内按 [反向 ] 区间循环按 [反向 ] 区间排序的区间中，关键字 interval 可以省略。

预定义的无限集 Reals 和 Integers 分别是所有浮点数和整数的集合，按数值顺序排列。预定义的无限集 ASCII、EBCDIC 和 Display 都表示字符串和数字的全集，任何一维集合的成员都从中抽取；ASCII 按 ASCII 排序序列排序，EBCDIC 按 EBCDIC 排序序列排序，Display 按 display 命令中使用的排序方式排序 (见章节 A.16)。在 Display 中，数字排在字面量之前，并按数值排序；字面量按 ASCII 排序序列排序。

# A.7 参数声明

参数声明包含一个可选属性列表，属性之间可用逗号分隔：

参数声明：param 名称 别名可选 索引可选 属性可选

属性可以是以下任意一项：

属性：binary integer symbolic relop 表达式 insexpr $=$ 表达式 default 表达式 relop：

关键字 `integer` 将参数限制为整数；`binary` 将其限制为 0 或 1。如果指定了 `symbolic`，则参数可以取任何字面量或数值（按集合成员的方式进行舍入），并且不允许使用涉及 `<`、`<=`、`>=` 和 `>` 的属性；否则参数为数值型，只能取数值。

涉及比较运算符的属性指定参数必须满足给定关系。`in` 属性指定对参数是否属于给定集合的检查。

`=` 和 `default` 属性类似于集合声明中的对应属性，并且互斥。

允许对索引参数进行递归定义，只要赋值能够按照只引用先前计算值的顺序进行计算即可。例如，

```
param comb 'n choose $\mathrm{k}^{\prime}$ {n in 0..N, k in 0..n} 
    := if k = 0 or k = n then 1 
       else comb[n-1,k-1] + comb[n-1,k];
```

计算从 $n$ 个事物中每次选取 $k$ 个的方法数。

在符号参数的递归定义中，关键字 `symbolic` 必须出现在对该参数的所有引用之前。

# A.7.1 检查语句

核查语句提供供应量断言，以帮助验证已读取或生成的数据是否正确；其语法为

$$
\mathtt{check}[\mathtt{indexing}_{\mathrm{opt}}:] \ \mathtt{lexpr};
$$

每个核查语句会在执行 `solve`、`write`、`solution` 或 `check` 命令之一时进行求值。

# A.7.2 无穷大

`Infinity` 是一个预定义参数；它是上界被视为不存在（即无穷大）的阈值，而 `-Infinity` 是下界被视为不存在的阈值。因此，给定

```
set A;
param Ub{A} default Infinity;
param Lb{A} default -Infinity;
var V {i in A} >= Lb[i], <= Ub[i];
```

对于在数据部分未给出 `Lb` 值的 $v$ 的分量，其下界无限制，而未给出 `Ub` 值的分量其上界无限制。类似地，可以为可选的下界和上界约束进行安排。在具有 IEEE 算术（大多数现代系统）的计算机上，`Infinity` 是 IEEE 的 $\infty$ 值。

# A.8 变量声明

变量声明以关键字 `var` 开始：

```
variable declaration: var name alias_opt indexing_opt attribute_opt;
```

变量声明的可选属性可以用逗号分隔；这些属性包括：`binary`、`integer`、`symbolic`、`>= expr`、`<= expr`、`= expr`、`:= expr`、`default expr`、`= expr`、`coeff indexing_opt constraint expr`、`cover indexing_opt constraint`、`obj indexing_opt objective expr`、`suffix sufname expr`

# attribute:

与参数类似，`integer` 将变量限制为整数值，而 `binary` 将其限制为 0 或 1。`>=` 和 `<=` 短语指定边界，而 `:=` 短语指定初始值。`default` 短语为可在数据部分提供的初始值指定默认值（A.12.2）；`default` 和 `:=` 是互斥的。`= expr` 短语仅在前面未出现任何属性时允许；它使变量成为定义变量（A.8.1）。每个后缀 `sufname expr` 短语为先前声明的后缀 `sufname` 指定一个初始值。

如果指定了 `symbolic`，则 `in sexpr` 也必须出现，且需要数值的属性 (如 $\geq$ expr) 被排除。如果 `in sexpr` 在没有 `symbolic` 的情况下出现，则集合表达式 `sexpr` 必须是区间和数字离散集合的并集。无论哪种方式，`in sexpr` 都将变量限制在 `sexpr` 中。

`coeff` 和 `obj` 短语用于列式系数生成；它们指定要放入命名约束或目标中的系数，这些约束或目标必须先前使用占位符 `to_come` 声明 (见 A.9 和 A.10)。索引的范围限于该短语，并且可以具有特殊形式

$$
\{\texttt{if}\ lexpr\}
$$

仅当 `lexpr` 为真时才贡献系数。`cover` 短语等效于 `expr` 为 1 的 `coeff` 短语。

弧 (arc) 是一种特殊的网络变量，其声明方式与普通变量不同，使用关键字 `arc` 而不是 `var`。它们可以通过形如 `from indexing opt node expr` 和 `to indexing opt node expr` 的可选属性短语，为节点约束 (A.9) 提供系数。

这些短语在语法上与 `coeff` 短语类似，唯一的区别在于最后的表达式是可选的；如果省略该表达式，则等价于指定其值为 1。

# A.8.1 定义变量

在某些非线性模型中，为了方便地定义一些对约束或目标函数有贡献的具名数值，可以使用定义变量。例如：

```
set A;
var v {A};
var w {A};
subject to C {i in A}: w[i] = vexpr;
```

其中 `vexpr` 是一个包含变量 `v` 的表达式。

目前来看，集合 `C` 的成员是约束，我们把一个无约束问题转化成了一个有约束问题，这可能并不是一个好主意。将选项 `substout` 设置为 1，可以指示 AMPL 消除约束集合 `C`。AMPL 通过告知求解器这些约束定义了其左侧的变量，从而实现这一点。实际上，这些定义变量就变成了具名的公共子表达式。

当 `substout` 选项为 1 时，像 `C` 这样为定义变量提供定义的约束称为定义约束 (defining constraint)。AMPL 通过扫描模型中约束的出现顺序来决定哪些变量是定义变量。只有在变量声明中没有附加约束（如整数性或界）时，该变量才有资格成为定义变量。一旦某个变量出现在某个定义约束的右侧，它就不再具备成为定义变量的资格——如果没有这一限制，AMPL 可能需要求解隐式方程。一旦某个变量被认定为定义变量，如果它随后出现在另一个本应为定义约束的左侧，则该约束将被视为普通约束处理。

一些 求解器 会对 线性变量 进行 特殊处理，因为 它们 的 高阶导数 为 零。对于 这类 求解器，对 线性定义变量 进行 特殊处理 可能 是 有益 的。否则，即使 在 定义变量 的 方程 右侧 仅 以 线性 方式 使用，这些 变量 在 求解器 看来 也 会 被 当作 非线性变量。通过 执行 高斯消元 而非 显式传递 线性变量定义，AMPL 可以 使 求解器 将 这些 右侧变量 视为 线性变量。这种 做法 通常 会 导致 填充 (fill-in)，即 问题 的 稀疏性 降低，但 可能 使 求解器 对 问题 有 更 准确 的 认识。当 选项 `linelim` 取 默认值 1 时，AMPL 会 对 线性定义变量 进行 特殊处理；而 当 `linelim` 为 0 时，AMPL 会 等同 对待 所有 定义变量。

变量声明 中 可以 包含 形如 $= expr$ 的 短语，其中 $expr$ 是 一个 包含 变量 的 表达式。这种 短语 表示 该 变量 将 被 定义 为 值 $expr$。这种 定义性声明 使得 某些 模型 可以 更 紧凑 地 编写。

然而，识别 定义变量 并不 总是 一个 好 主意 —— 虽然 这样 可以 减少 变量 数量，但 可能 导致 问题 更加 非线性，并且 由于 稀疏性 丧失，求解 成本 可能 更 高。通过 使用 定义约束（而 不是 使用 定义变量声明），并 分别 在 $\text{substout} = 0$ 和 $\text{substout} = 1$ 的 情况 下 翻译 和 求解 问题，可以 判断 识别 定义变量 是否 值得。另一方面，如果 识别 某个 定义变量 明显 是 有益 的，那么 在 其 声明 中 直接 定义 它 通常 比 提供 一个 单独 的 定义约束 更 方便；特别是，如果 所有 定义变量 都 在 其 声明 中 被 定义，则 无需 担心 $\text{substout}$ 的 设置。

定义声明 的 一个 限制 是：下标变量 必须 在 其 被 使用 之前 定义。

# A.9 约束声明

约束声明 的 形式 如下：

约束声明: [ subject to ] 名称 别名可选 索引可选 [ := 初始对偶值 ] [ default: 初始对偶值 ] [ : 约束表达式 ] [ 后缀初始化 ] ;

约束声明 中 的 关键字 subject to 可以 省略，但 通常 为了 清晰 起见 最好 保留。可选 的 $\coloneqq$ 初始对偶值 指定 了 与 该 约束 关联 的 对偶变量（拉格朗日乘数）的 初始 猜测值。同样，default 和 $\coloneqq$ 子句 是 互斥 的，default 用于 数据 部分 未 给出 的 初始值。约束声明 必须 以 以下 形式 之一 指定 一个 约束：

约束表达式: expr $<=$ expr expr $=$ expr expr $>=$ expr cexpr $<=$ expr $<=$ cexpr cexpr $>=$ expr $>=$ cexpr

为了 启用 按列 生成 约束 系数，其中 一个 expr 可以 具有 以下 形式 之一：

to_come $^+$ expr expr $^+$ to_come to_come

在 变量声明 (A.8) 中 为 该 约束 指定 的 项 将 被 放置 在 to_come 的 位置。

节点 (node) 是一种特殊的约束 (constraint)，可以向弧发送流量或从弧接收流量。它们的声明以关键字 `node` 开头，而不是 `subject to`。纯转运节点没有约束体；它们必须满足“流入”等于“流出”。其他节点是源点或汇点；它们以以上形式之一指定约束，但不得提及 `to_come`，并且恰好有一个 `expr` 必须具有以下形式之一：

`net_in + expr`　`net_out + expr`　`expr + net_in`　`expr + net_out`　`net_in`　`net_out`

关键字 `net_in` 被替换为表示节点净流入 (net inflow) 的表达式：由弧声明中的 `to` 语句（A.8 节）对该节点约束所贡献的项，减去由 `from` 语句所贡献的项。`net_out` 的处理方式类似；它是 `net_in` 的相反数。

可选的后缀初始化语句 (suffix-initialization) 具有如下形式：  
`suffix-initialization: suffix sufname expr`  
前面可以加逗号，其中 `sufname` 是先前声明的后缀 (suffix)。

# A.9.1 互补约束 (Complementarity constraints)

除了上述形式外，约束声明还可用于表达互补约束，其形式为：  
`name alias opt indexing opt : constr1 complements constr2`  
其中 `constr1` 和 `constr2` 由 1、2 或 3 个通过运算符 `< =`、`> =` 或 `=` 分隔的表达式组成。在 `constr1` 和 `constr2` 总共必须包含两个显式的不等式运算符，其中 `=` 被视为两个不等式。若互补约束中的 `constr1` 或 `constr2` 包含两个不等式，则该约束必须具有如下形式之一：  
- `expr1 < = exp2 < = exp3` complements `expr4`  
- `expr3 > = exp2 > = exp1` complements `expr4`  
- `expr4` complements `expr1 < = exp2 < = exp3`  
- `expr4` complements `expr3 > = exp2 > = exp1`

在所有这些情况下，约束要求不等式成立，并且满足以下条件：  
- 若 `expr1 = exp2`，则 `expr4 ≥ 0`  
- 若 `expr2 = exp3`，则 `expr4 ≤ 0`  
- 若 `expr1 < exp2 < exp3`，则 `expr4 = 0`

在具有均衡约束 (equilibrium constraints) 的数学规划中，互补约束可以与其他约束和目标函数共存。

求解器 (solver) 看到的互补约束为标准形式：  
`expr complements lower bound < = variable < = upper bound`

一个同义词（A.19.4）`_scvar{i in 1.._sncons}` 指示了每个约束（若存在）所对应的互补变量。若 `_scvar[i] in 1.._snvars`，则变量 `_svar[_scvar[i]]` 与约束 `_scon[i]` 互补；否则 `_scvar[i] == 0`，此时 `_scon[i]` 为普通约束。同义词 `_cconname{1.._nccons}` 表示建模者所看到的互补约束的名称。

# A.10 目标声明 (Objective declarations)

目标声明的形式为以下之一：

```
objective declaration: maximize name alias opt indexing opt [ : expression ] [ suffix-initializations ] ;
minimize name alias opt indexing opt [ : expression ] [ suffix-initializations ] ;
```

并且可以指定以下形式之一的表达式：

```
expression: expr to_come + expr
           expr + to_come
           to_come
```

`to_come` 形式允许按列生成系数，类似于约束（A.9 节）。若未指定上述任何表达式，则等同于指定 `: to_come`。后缀初始化（suffix-initializations）可如约束声明中那样出现。

如果有多个目标函数，发送给求解器 (solver) 的目标函数可以通过 objective 命令进行设置；参见章节 A.18.6。默认情况下，所有目标函数都会被发送。

# A.11 辅助值的后缀表示法

变量 (variable)、约束 (constraint)、目标函数 (objective) 和问题 (problem) 都有多种相关的辅助值。例如，变量具有上下界和约简成本 (reduced cost)，约束具有对偶值 (dual value) 和松弛变量 (slack)。这些值通过 name.suffix 的形式进行访问，其中 name 是一个简单或带下标的变量、约束、目标函数或问题名称，.suffix 是表 A-6、A-7 和 A-8 中列出的可能选项之一。

对于一个约束，.body、.lb 和 .ub 值对应于约束的一种变形形式。如果约束涉及单个不等式，则从左侧减去右侧，并将任何常数项移至右侧；如果约束涉及双不等式，则类似地从所有三个表达式（左、中、右）中减去中间的任何常数项。然后该约束具有形式 $lb \leq body \leq ub$，其中 $lb$ 和 $ub$ 是（可能是无限的）常数。

以下规则用于确定约束 $c$ 的下对偶值和上对偶值（c.ldual 和 $c$.udual）。求解器返回一个单一的对偶值 $c$.dual，该值可能适用于 .body $\geq lb$ 或 body $\leq ub$。对于等式约束 $(lb = ub)$，AMPL 使用 $c$.dual 的符号来判断。对于最小化问题，若 $c$.dual $> 0$，则意味着如果约束为 body $\geq lb$ 时会得到相同的最优解，因此 AMPL 设置 $c$.ldual $= c$.dual 且 $c$.udual $= 0$；类似地，若 $c$.dual $< 0$，则 $c$.ldual $= 0$ 且 $c$.udual $= c$.dual。对于最大化问题，不等式方向相反。

对于不等式约束 $(lb < ub)$，AMPL 使用与边界接近程度来判断 $c$.ldual 或 $c$.udual 是否等于 $c$.dual。如果 body $- lb < ub -$ body，则 $c$.ldual $= c$.dual 且 $c$.udual $= 0$；否则，$c$.ldual $= 0$ 且 $c$.udual $= c$.dual。

模型声明可以引用表 A-6、A-7 和 A-8 中描述的任何带后缀的值。这在引入新的声明（这些声明在模型已被翻译和求解之后）时最为有用。特别是，提供后缀 .val 和 .dual 是为了使新约束可以引用当前原问题和对偶问题变量以及目标函数的最优值。

astatus AMPL 状态 (A.11.2) . init 当前初始猜测值 . init0 初始初始猜测值 (由 $\coloneqq$、数据或默认值设定) . lb 当前下界 . lb0 初始下界 . lb1 预求解得出的较弱下界 . lb2 预求解得出的较强下界 . lrc 下 reduced cost (对于变量 $> = 1b$) . lslack 下松弛 (val - lb) . rc reduced cost . relax 若为正，则忽略整数限制 . slack min(lslack, uslack) . sstatus 求解器状态 (A.11.2) . status 状态 (A.11.2) . ub 当前上界 . ub0 初始上界 . ub1 预求解得出的较弱上界 . ub2 预求解得出的较强上界 . urc 上 reduced cost (对于变量 $< =$ ub) . uslack 上松弛 (ub - val) . val 变量的当前值

表 A-6：变量的点后缀。

对于互补约束，类似 constraint.lb、constraint.body 等的后缀表示法进行了扩展，使得 constraint.Lsuffix 和 constraint.Rsuffix 分别对应 constr1.suffix 和 constr2.suffix，并且互补约束的 constraint.slack（或未修饰的名称）表示互补约束满足程度的一种度量：如果 constr1 和 constr2 各自包含一个不等式，则新的度量是 min(constr.slack, constr.slack)，如果两者都严格满足不等式则为正值，如果互补约束恰好满足则为 0，如果至少有一个 constr1 或 constr2 被违反则为负值。对于形式为 $expr_1 <= expr_2 <= expr_3$ 且与 $expr_4$ 互补的约束，.slack 值为：

$$
\begin{array}{rl}
& \min(expr_2 - expr_1, expr_4)\mathrm{~若~}expr_1 >= expr_2 \\
& \min(expr_3 - expr_2, -expr_4)\mathrm{~若~}expr_3 <= expr_2 \\
& -\mathsf{abs}(expr_4)\mathrm{~若~}expr_1 < expr_2 < expr_3
\end{array}
$$

因此在所有情况下，如果互补约束恰好成立，则 .slack 值为 0，如果其中一个必要的不等式被违反，则为负值。

# A.11.1 后缀声明

A.11.1 后缀声明 后缀声明引入新的后缀，这些后缀可以在后续声明、let 命令和函数调用（带有 OUT 参数，A.22）中赋值。后缀声明以关键字 suffix 开头：

. astatus AMPL 状态 (A.11.2) . body 约束主体的当前值 . dinit 对偶变量的当前初始猜测值 . dinit0 对偶变量的初始初始猜测值 (由 $\coloneqq$、数据或默认值设定) . dual 当前对偶变量 . lb 下界 . lbs 求解器的 1b (针对固定变量进行了调整) . ldual 下对偶值 (对于主体 $> =$ 1b) . lslack 下松弛 (主体 - 1b) . slack min(lslack, uslack) . sstatus 求解器状态 (A.11.2) . status 状态 (A.11.2) . ub 上界 . ubs 求解器的 ub (针对固定变量进行了调整) . udual 上对偶值 (对于主体 $< =$ ub) . uslack 上松弛 (ub - 主体)

表 A-7：约束的点后缀。

. val 目标函数的当前值

表 A-8：目标函数的点后缀。

后缀声明：后缀名称 别名 $opt$ 属性 $opt$ 后缀声明的可选属性可以用逗号分隔；这些属性包括 属性：binary integer symbolic $>=$ expr $<=$ expr 方向 方向：IN OUT INOUT LOCAL

最多只能指定一个方向；如果未指定方向，AMPL 默认为 INOUT。这些方向是从求解器的角度来看的：IN 后缀值是输入到求解器的；OUT 后缀值是由求解器赋值的；INOUT 值既是 IN 也是 OUT；LOCAL 值对求解器不可见。

符号后缀用 symbolic 属性声明；在符号后缀名称后添加 _num 得到相关联的数值后缀名称；求解器看到的是相关联的数值。如果 symsuf 是一个符号后缀，选项 symsuf_table 按如下方式将 symsuf 与 symsuf_num 关联。$\mathparagraph$ symsuf_table 的每一行应以一个数值限制值开头，

后跟一个字符串值和可选注释，所有内容用空格分隔。数值限制值必须逐行递增。与 .sufname_num 值相关联的字符串值是小于或等于该值的最大数值限制值对应的字符串值。如果 .sufname_num 值小于 $\mathbb{S}$ symsuf_table 第一行中的限制值，则使用 .symsuf_num 值作为 .symsuf 值。

# A.11.2 状态

一些求解器维护一个基，并区分基本变量和各种类型的非基本变量及约束。AMPL 内置的符号后缀 .sstatus 允许求解器向 AMPL 返回基信息，并在后续求解 (A.18.1) 中提供先前的最优基，这有时能更快地求解更新后的问题。

AMPL 对约束的 drop/restore 状态 (A.18.6) 和对变量的 fix/unfix 状态 (A.18.7) 反映在内置的符号后缀 .astatus 中。内置的符号后缀 .status 派生自 .astatus 和 .sstatus：如果变量或约束，比如 $x$，在当前问题中，则 $x$ .status = $x$ .sstatus；否则 $x$ .status = $x$ .astatus。如果 $x$ 在当前问题中，AMPL 赋值 $x$ .astatus_num = 0，因此确定 .status 的规则是 $x$ .status $\equiv$ if $x$ .astatus_num $== 0$ then $x$ .sstatus else $x$ .astatus。

当选项 astatus_table 为默认值时，$x$ .astatus = 'in' 当且仅当 $x$ .astatus_num = 0。

# A.12 标准数据格式

AMPL 支持一种标准格式来描述集合和参数值，这些集合和参数值与模型结合形成特定的优化问题。

数据段由一系列标记 (字面量和可打印字符组成的字符串) 组成，标记之间由空白符 (空格、制表符和换行符) 分隔。标记包括关键字、字面量、数字和分隔符 ( ) [ ] : , ; := *。语句是一系列以分号结尾的标记。注释可以像在声明中一样出现。在所有情况下，将数据排列成整齐的行和列只是为了便于人类阅读；AMPL 会忽略格式。

数据段以 data 命令开始，以输入结束或返回模型模式的命令结束 (A.14)。

在数据段中，模型实体可以按任何方便的顺序赋值，与它们声明的顺序无关。

# A.12.1 集合数据

定义 集合 (set) 的语句由关键字 set、集合名称、可选的 $\equiv$ 和成员组成。一维集合最简单地通过列出其成员来指定，成员之间可以用逗号分隔。如果字符串是字母数字组成的且不表示数字，则可以省略字面量字符串的单引号或双引号。

数据段中的对象可以是数字或字符字符串。与模型中一样，可以通过用引号 ( ' 或 " ) 将其包围来指定字符字符串。然而，由于数据中出现了许多字符串，AMPL 允许数据语句省略仅由可能出现在名称或数字中的字符组成的字符串周围的引号，除非需要用引号来区分字符串和数字。

集合数据语句的一般形式为：  
set- data- statement:  
set set- name : $=$ set- spec set- spec set- spec :  
set- template opt member- list  
set- template opt member- table  
member- table

set- name 必须是单独声明的集合的名称，或者是索引集合中的下标集合名称。可选的模板形式为：  
set- template:  
templ- item, templ- item, ... templ- item: object *  
其中 templ- items 的数量必须等于命名集合的维度。如果没有给出模板，则假定为全 $*$ 的模板。

set- spec 有两种形式：列表格式和表格格式。set- spec 的列表格式为：  
member- list:  
member- item  
member- item  
member- item: object object

member- item 中的对象数量必须与前面模板中的 $*$ 数量匹配，且模板中不能将 : 作为 templ- item；对象从左到右替换 $*$ ，生成一个成员并添加到正在指定的集合中。在模板不包含 $*$ 的特殊情况下，member- list 应该为空，而模板本身指定要添加的一个成员。

set- spec 的表格格式如下：

member- table:  
$(\mathtt{tr})_{opt}$ t- header t- header $\begin{array}{rl}{\ldots}&{:=}\\{\pm}&{\pm}\\{\pm}&{\pm}\end{array}$  
row- label row- label  
t- header: object object ...  
row- label: object object

必须至少有一个 $t$ - 标题，每行标签中至少有一个对象，并且 $t$ - 标题和行标签的数量应与前述模板中的 $*$ 和 : 的数量一致。如果前述模板中包含任何 : ，则 : 的数量必须与 $t$ - 标题数量一致；否则，如果可选的 $(\mathtt{tr})$ 出现，则初始的 $*$ 被视为 : ；如果 $(\mathtt{tr})$ 没有出现，则末尾的 $*$ 被视为 : 。每个显示为 $\pm$ 的表格项必须是 $^+$ 或 - 符号之一。每个 - 项将被忽略，而每个 $^+$ 项的行标签将依次替代模板中的 $*$ ，同时与 $^+$ 对应的 $t$ - 标题中的对象将替代 : ，从而生成一个集合成员。

要定义一个复合集合 (set)，可以列出所有成员。每个成员是一个用括号括起来、以逗号分隔的组件列表，连续成员之间可以有可选的逗号。或者，可以通过一个表格 (table) 或一系列表格来描述二维集合的成员。在这样的表格中，行标签对应第一个下标，列标签对应第二个下标；$\epsilon_{+}^{,,}$ 表示集合中存在的一对元素，而 $\epsilon_{- }^{,,}$ 表示集合中不存在的一对元素。冒号 (:) 用于引入表格，在此上下文中是强制性的。如果 (tr) 出现在冒号之前，则表格将被转置，行和列的角色互换。

一般来说，集合语句涉及一系列一维和二维集合表格。一维表格以一个新的模板开始（其后可选地跟 $\equiv$），或者单独以 $\equiv$ 开始，在这种情况下将保留之前的模板。默认 (default) 模板是 $(\star ,\ldots ,\star)$ ，即星号 $\star$ 的数量与集合的维度相同。二维表格以一个可选的新模板开始，后跟 : 或 (tr) 以及一个可选的冒号，再后跟一列列表标签和 $\equiv$ 。不包含 $\star \star$ 的模板表示其本身。(tr) 的效果将持续到出现新模板为止。

对于索引集合，每个组件集合必须在单独的数据语句中给出。不需要按照父集合中的顺序指定子集成员。

# A.12.2 参数数据

指定参数数据或变量初始值的语句有两种形式。第一种形式类似于集合数据语句：

param- data- statement: param param- name param- default  $\mathbf{\Phi}_{opt}\coloneqq$  param- spec param- spec param- spec: param- template opt value- list param- template opt value- table value- table

其中添加了一个可选的 param- default，将在下文描述。param- name 通常是模型中声明的参数名称，但也可以是变量或约束的名称；可以使用关键字 var 代替 param 以明确区分。

参数 (param) 语句的模板与集合 (set) 数据语句中的内容相同，但使用方括号 (类似下标) 而非圆括号表示：

param 模板：[ templ- item, templ- item, ... ] templ- item: object \* :

值列表 (value- list) 与之前定义的成员列表 (member- list) 类似，但还指定了参数或变量的值：

值列表：value- item value- item ... value- item: object object ... entry

对象 (object) 用于替换模板中的 $\star \star$ 以定义集合成员，而以该集合成员为索引的参数或变量则被赋予与该项 (entry) 关联的值 (见下文)。

值表 (value- table) 与之前定义的成员表 (member- table) 类似，但其项 (entry) 是值而非 $^+$ 或 -：

值表：$(\mathsf{tr})_{\mathrm{opt}}$ t- header t- header t- header : $=$ row- label entry entry entry row- label entry entry entry

t- header: object object ...

row- label: object object ...

entry: number string default- symbol

与集合语句中一样，符号 `(tr)` 表示转置；它意味着这是一个二维表，其后的冒号是可选的。该符号持续有效，直到出现新的模板为止。

表格可以分多个列块给出。

每个项的 `row-label` 和 `t-header` 条目用于替换模板中的 `*` 和 `:` 以定义集合成员，而以该集合成员为索引的参数或变量则被赋予由该项指定的值。项可以是数字（用于变量和取数值的参数），或字符串（用于声明为 `symbolic` 属性的变量和参数）。若项为默认符号 (`default symbol`)（见下文），则忽略该值。

参数数据语句的第二种形式用于定义多个参数，并可选择性地定义这些参数的索引集合：

`param-data-alternate`:  
`param` `param-default`<sub>opt</sub> `param-name` `param-name` ... : `value-item` `value-item` ...  
`param` `param-default`<sub>opt</sub>: `set-name`: `param-name` `param-name` ... : `value-item` `value-item` ...

所命名的参数必须具有相同的维度。如果指定了可选的 `set-name`，则其成员也由该语句定义。每个 `value-item` 由一个可选模板、一个对象列表和一个值列表组成：

`value-item`: `template`<sub>opt</sub> `object` ... `entry` `entry` ...

假定所有 `*` 的初始模板（数量与命名参数的公共维度相同），并且模板在出现新模板之前一直有效。对象的数量必须与当前模板中 `*` 的数量相等；当它们替换当前模板中的 `*` 时，定义了一个集合成员。如果正在定义一个集合，则将此成员添加到该集合中。由该成员索引的参数被赋予与后续条目关联的值，这些条目遵循与前面描述的表格条目相同的规则。值按照参数名称在语句开头出现的顺序进行赋值；条目的数量必须等于命名参数的数量。

参数数据语句的可选 `default` 短语形式如下：

`param default`: 默认数值

如果存在此短语，则语句中命名但未显式赋值的任何参数将被赋予该数值。

数据项可以指定为 `.` 而不是显式值。这通常被视为缺失值，在模型中引用它会导致错误消息。然而，如果存在默认值，则每个缺失值都由该默认值确定。默认值可以通过模型中参数声明中的 `default` 短语指定（A.7），或通过数据语句中参数名称后面的可选短语 `default r` 指定。在后一种情况下，$r$ 必须是数值常量。

默认值符号可以出现在一维和二维表格中。默认符号最初是一个点（`.`）。维护一个默认值符号堆栈，当前符号位于顶部。`defaultsym` 语句（仅在数据部分中识别）将新符号压入堆栈，而 `nodefaultsym` 将“无符号”指示符压入堆栈。语句 `defaultsym;`（不带符号）弹出堆栈。

具有三个或更多索引的参数可以通过一系列一维和二维表格赋值，每个非默认值切片对应一个或多个表格。

总之，定义一个索引参数的 param 语句以关键字 param 和参数名称开头，后跟可选的默认值和可选的 $\coloneqq$。然后是一系列一维和二维参数表格，它们类似于一维和二维集合表格，但模板使用方括号而不是圆括号，二维表格包含数字（或对于符号参数，是字面量）而不是 $+^{\circ}\mathrm{s}$ 和 $- \mathrm{^s}$，并且对应于 $k^{\ast}$ 模板的一维表格包含 $k + 1$ 列而不是 $k$ 列，最后一列是数字列或默认符号列（.）。一种特殊形式是关键字 param、可选默认值和单个（未转置的）二维表格，它定义了由公共集合索引的多个参数；另一种特殊形式是 param 后跟参数名称、可选的 $\coloneqq$ 和数值，它定义了一个标量参数。

变量 (variable) 与约束 (constraint) 的名称可以出现在数据段中参数名称可能出现的任何位置，用于指定变量的初始值以及与约束关联的对偶变量。默认值的规则与参数的规则相同。关键字 var 在数据语句中是 param 的同义词。

# A.13 数据库访问与表格

AMPL 的表格 (table) 功能允许从外部介质（如数据库、文件或电子表格）获取数据，并将数据返回至外部介质。表格声明用于建立外部关系表的列与 AMPL 中的集合、参数、变量和表达式之间的连接。read table 与 write table 命令利用这些连接从表格中读取数据到 AMPL，或将数据写回表格。AMPL 使用表格处理器 (table handler) 来实现这些连接。内置的表格处理器允许读写 ".tab" 与 ".bit" 文件，以保存和恢复数值，并试验 AMPL 的表格功能；要访问数据库与电子表格，至少需要安装或加载另一个处理器 (A.22)。内置集合 _HANDLERS 列出了当前可用的处理器，符号参数 _handler_lib{_HANDLERS} 则指明每个处理器来自哪个共享库。

表格声明的形式为：

```
table declaration: table table-name indexing opt in-out opt string-list opt : key-spec, data-spec, data-spec, ... ;
```

其中 in-out 为 IN、OUT 或 INOUT 之一；IN 表示数据流入 AMPL，OUT 表示数据流出 AMPL，INOUT 表示双向流动。若未指定 in-out，则默认为 INOUT。可选的 string-list 给出用于访问外部数据的驱动程序、文件及驱动程序特定选项的名称；其内容取决于所使用的表格处理器以及可能的操作系统。

表格声明中的 key-spec 指定用于唯一标识待访问数据的关键列：

```
set-io opt [key-col-spec, key-col-spec, ...]
```

可选的 `set-io` 短语形式为 `set-name arrow`，其中 `arrow` 为 `<-`、`->` 或 `<->` 之一；它指示信息移动的方向：从关键列到 AMPL 集合（通过 `read table`），从集合到列（通过 `write table`），或两者皆有，具体取决于 `in-out`。每个 `key-col-spec` 命名外部表中的一列，并将其与一个虚拟索引关联。形式为 `key-name` 的 `key-col-spec` 将 `key-name` 同时用于两种目的；形式为 `key-name ~ data-col-name` 的 `key-col-spec` 则引入 `key-name` 作为虚拟索引，`data-col-name` 作为外部介质中列的名称；`data-col-name` 可以为名称、带引号的字符串或括号括起的字符串表达式。

每个 `data-spec` 命名外部表中的一个数据列。在最简单的情况下，外部名称与 AMPL 名称相同。若不相同，则可使用如下语法将外部名称与内部名称关联：

```
data-spec: param-name ~ data-col-name
```

每个数据规格 (data-spec) 可选地以 `IN`、`OUT` 或 `INOUT` 之一结尾，用于覆盖默认的表格方向，并指示 `read table` 是否应将该列读入 AMPL（`IN` 或 `INOUT`），以及 `write table` 是否应将该列写入外部介质（`OUT` 或 `INOUT`）。

特殊语法允许使用索引表达式来描述一列或多列数据：

```
indexing expr-col-desc
indexing ( expr-col-desc, expr-col-desc, ... )
```

其中 `expr-col-desc` 的形式为：

```
expr [ ~ colname ] in-out-opt
```

另一种特殊语法允许迭代数据列：

```
indexing < data-spec, data-spec, ... >
```

后者不可嵌套，但可包含前者。

在表声明之后，通过以下命令进行数据访问：

```
read table table-name subscriptopt :
write table table-name subscriptopt
```

这些命令会引用表声明中提供的信息。

# A.14 命令语言概述

AMPL 能识别表 A-9 中列出的命令。命令不属于模型的一部分，但会使 AMPL 按如下方式执行操作。

命令环境识别两种模式。在模型模式下（即 AMPL 启动时所处的模式），它识别模型声明（A.5）和所有下述命令。当遇到 `data` 语句时，它会切换到数据模式，在该模式下仅识别数据模式语句（A.12）。当遇到不能作为数据模式语句开头的关键字或文件结束时，它会返回模型模式。除 `data`、`end`、`include`、`quit` 和 `shell` 外的命令也会使 AMPL 进入模型模式。

形式如下的短语：

```
include filename
```

会使指定文件被插入。在此处及后续出现文件名的上下文中，若文件名包含分号、引号、空格或不可打印字符，则必须以字面量形式给出，即 `'filename'` 或 `"filename"`。在 `include` 之外的上下文中，文件名也可以是带括号的字符串表达式（A.4.2）。`include` 命令可以嵌套；在模型模式和数据模式下都会被识别。以下序列：

```
model; include filename
data; include filename
```

可分别简写为：

```
model filename
data filename
```

`commands` 命令类似于 `include`，但它是一条语句，必须以分号结尾。当 `data` 或 `commands` 命令出现在复合命令中 (即循环体、`if` 命令 (A.20.1) 的 `then` 或 `else` 部分，或简单地出现在大括号内的一系列命令中) 时，它会在控制流到达该命令时执行，而不是在读取复合命令时执行。在这种情况下，如果 `data` 或 `commands` 命令未指定文件，则 AMPL 会从当前输入文件中读取命令或数据，直到遇到 `end` 命令或当前文件结束为止。

对于包含短语以及模型、数据和命令的文件，如果文件名简单 (例如不包含斜杠 `/`)，则会在由选项 `ampl_include` (A.14.1) 指定的目录 (文件夹) 中查找：`ampl_include` 的每个非空行指定一个目录；如果 `ampl_include` 为空或完全为空白，则在当前目录中查找文件。

选项 `insertprompt` (默认为 `'<%d>'`) 指定一个插入提示符，该提示符会紧接在标准输入的常规提示符之前出现。如果存在 `%d`，它表示当前插入的层级，即数据和命令文件嵌套的层级，这些命令指定文件并出现在复合命令内部。

# 表 A-9：命令列表

| 命令 | 功能 |
|------|------|
| call | 调用导入的函数 |
| cd | 更改当前目录 |
| check | 执行所有 check 命令 |
| close | 关闭文件 |
| commands | 从文件读取并解释命令 |
| data | 切换到数据模式；可选择包含文件内容 |
| delete | 删除模型实体 |
| display | 显示模型实体和表达式；还包括 `csvdisplay` 和 `_display` |
| drop | 删除一个约束或目标函数 (objective) |
| end | 结束当前输入文件的输入 |
| environ | 为问题实例设置环境 |
| exit | 以指定状态值退出 AMPL |
| expand | 展开 (expand) 并显示模型实体 |
| fix | 将变量固定在其当前值 |
| include | 包含文件内容 |
| let | 更改数据值 |
| load | 加载动态函数库 |
| model | 切换到模型模式；可选择包含文件内容 |
| objective | 选择要优化的目标函数 |
| option | 设置或显示选项值 |
| print | 以未格式化方式打印模型实体和表达式 |
| print | 以格式化方式打印模型实体和表达式 |
| problem | 定义或切换到指定名称的问题 |
| purge | 移除模型实体 |
| quit | 终止 AMPL |
| read | 从文件读取输入 |
| read table | 从数据表 (table) 读取输入 |
| declare | 更改实体的声明 |
| reload | 重新加载动态函数库 |
| remove | 删除文件 |
| reset | 将指定实体重置为其初始状态 |
| restore | 撤销一个 drop 命令 |
| shell | 临时退出到操作系统以运行命令 |
| show | 显示模型实体的名称 |
| soloexpand | 显示求解器看到的展开形式 |
| solution | 导入变量值，如同来自求解器一样 |
| solve | 将当前实例发送给求解器并获取解 |
| update | 允许更新数据 |
| unfix | 撤销一个 fix 命令 |
| unload | 卸载动态函数库 |
| write | 写出一个问题实例 |
| write table | 将数据写入数据表 (table) |
| xref | 显示实体之间的依赖关系 |

# A.14.1 选项和环境变量

AMPL 维护着一系列影响命令和求解器行为的选项值。这些选项类似于 Windows 和 Unix 操作系统中的“环境变量”；实际上，AMPL 会从这些系统的环境中继承其初始选项。然而，如果这些选项未通过这种方式继承，AMPL 会为许多选项提供自己的默认值。

`option` 命令提供了一种查看和更改选项的方法。它具有以下形式之一：

```
option redirectionopt option opname [evalue] redirectionopt
```

第一种形式会打印所有已被更改的选项，或其默认值可能由 AMPL 提供的选项。在第二种形式中，如果存在 evalue，则将其赋值给 opname；否则，将打印当前与 opname 关联的值（一个字符串）。opname 是一个选项名称，前面可以带有环境名称（A.18.8）和一个句点。选项名称也可以是一个名称模式 (name-pattern)，即包含一个或多个 $\star \mathrm{s}$ 的名称。在名称模式中，\* 代表任意可能为空的名称字符序列，因此可以匹配多个名称；例如：

option \*col\*;

列出所有名称中包含字符串 "col" 的选项。也可以通过括号内的字符串表达式指定特定的环境或选项名称。

evalue 是一个由一个或多个字面量、数字、括号内的字符串表达式以及形如 $\mathparagraph$ opname 或 $\mathbb{S}\mathbb{S}$ opname 的选项引用组成的、以空白字符分隔的序列，其中 opname 通常不包含 $\star \mathrm{s}$。一般来说，$\mathparagraph$ opname 表示选项 opname 的当前值，而 $\mathbb{S}\mathbb{S}$ opname 表示默认值，即从操作系统继承的值（如果有的话）或由 AMPL 提供的值。如果剩下的部分是一个名称或数字，则字面量周围的引号可以省略。显示的选项值采用一种可作为选项命令读取的格式。

# A.15 输入和输出的重定向

可选的重定向短语可以与各种 AMPL 命令一起使用，以将其输出捕获到文件中供后续处理。它适用于所有形式的 display 和 print 命令，也适用于大多数其他可以产生输出的命令，例如 solve、objective、fix、drop、restore 和 expand。

重定向具有以下形式之一：

> filename  
>> filename  
< filename （用于 read 命令）

其中 filename 可以是数据和命令中可能出现的任何形式（A.14）。当某个命令首次在重定向中指定 filename 时，文件将被打开；第一种重定向形式会在首次打开时覆盖文件内容，而第二种形式则会将输出追加到当前内容之后。形式 < filename 仅用于 read 命令的输入（A.17）。一旦打开，filename 将保持打开状态，直到 reset 命令执行，或通过 close 命令显式关闭：

close filenameopt;

只要 filename 保持打开状态，输出形式的重定向会将输出追加到文件的当前内容中。不带 filename 的 close 命令将关闭所有打开的文件和管道。close 命令可以指定一个以逗号分隔的文件名列表。变体 remove filename ; 会关闭并删除 filename。

# A.16 打印和显示命令

display、print 和 printf 命令用于打印任意表达式。它们的形式如下：

display [ indexing: ] disparglist redirectionopt ;  
print [ indexing: ] arglist redirectionopt ;  
printf [ indexing: ] fmt , arglist redirectionopt ;

如果存在索引 (indexing)，其作用域将延伸到命令的末尾，并且对于索引集中的每个成员，该命令都会执行一次。格式字符串 fmt 类似于 C 编程语言中的 printf 格式字符串，其详细说明见下文。

arglist 是一个（可能为空的、以逗号分隔的）表达式和迭代参数列表 (iterated-arglists) 的列表；迭代参数列表的形式如下之一：  
indexing expr  
indexing ( nonempty-arglist )  
其中 expr 是任意表达式。这些表达式也可以包含简单或带下标的变量 (variable)、约束 (constraint) 和目标名称；约束名称表示该约束当前的对偶值 (dual value)。disparglist 的描述见下文。

可选的重定向 (A.15) 会使输出被发送到文件，而不是显示 (display) 在标准输出上。

print 命令将其 arglist 中的项目打印 (print) 在一行上，项目之间以空格分隔，并以换行符结尾；分隔符可通过选项 print_separator 更改。文字仅在数据模式下必须加引号时才会被加引号。默认情况下，数值表达式将以完整精度打印 (print)，但可通过选项 print_precision 或 print_round 更改，如下所述。

printf 命令根据其格式字符串 fmt 打印 (printf) 其 arglist 中的项目。其行为类似于 C 中的 printf 函数。格式字符串中的大多数字符会按原样打印 (print)。转换说明 (conversion specifications) 是例外。它们以 $\%$ 开头，以格式字母结尾，如表 A-10 所总结。在 $\%$ 和格式字母之间可以有以下内容：-（表示左对齐）；+（强制显示符号）；0（用前导零填充）；最小字段宽度；句点；以及精度，用于指定从字符串中打印 (print) 的最大字符数，或对于 $\%f$ 和 $\%e$，表示小数点后要打印 (print) 的位数，或对于 $\%g$，表示有效数字的位数，或对于 $\%d$，表示最小数字位数。字段宽度和精度可以是十进制数字，或者为 \*，此时它将被 arglist 中下一个项目的值替换。每个转换说明会消耗 arglist 中的一个或（当涉及 $\%s$ 时）多个项目，并格式化其消耗的最后一个项目。对于 $\%g$，精度为 0（即 %.0g）表示能四舍五入到被格式化值的最短十进制字符串。允许使用标准 C 转义序列：$\backslash a$（警报或响铃）、$\backslash b$（退格）、$\backslash f$（换页）、$\backslash n$（换行）、$\backslash r$（回车）、$\backslash t$（水平制表符）、$\backslash v$（垂直制表符）、$\backslash x d$ 和 $\backslash x d d$。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/263e893d34912d20b02bd4f0fa8f3e9401fc30d47fb0edf195aa7be702505a15.jpg)  
表 A-10：printf 格式中的转换说明。

其中 $d$ 表示一个十六进制数字，而 $\backslash d$、$\backslash dd$ 和 $\backslash ddd$ 中的 $d$ 表示一个八进制数字。格式 $\frac{9}{6} Q$ 用于按照数据段引用规则打印 (print) 字符串值；格式 $\frac{9}{6} Q$ 总是对字符串加引号。

sprintf 函数 (A.4.2) 根据一个格式字符串对其参数列表进行格式化，该格式字符串使用相同的转换说明符。

display 命令将各种实体按照适当的方式格式化为表格或列表。其 display 列表类似于 print 或 printf 的参数列表，但要显示的项也可以是一个集合表达式或未加下标的索引参数、变量、约束 (con) 或集合的名称；此外，迭代参数列表不能嵌套，即它们仅限于 indexing expr 和 indexing ( exprlist ) 的形式，其中 exprlist 是一个非空的、由逗号分隔的表达式列表。display 命令单独打印 (print) 标量表达式和集合，并将索引实体按具有相同下标数量的方式分组，然后在各自的表格或表格序列中打印 (print) 每组内容。

默认情况下，display 命令将数值表达式四舍五入到六位有效数字，但可以通过选项 display_precision 或 display_round 进行更改，如下所述。

一些以 _precision 结尾的选项控制浮点数转换为可打印值时的精度；正值表示四舍五入到该位有效数字，0 或其他值表示四舍五入到最短的十进制字符串，该字符串在适当地四舍五入为机器数时会得到相应的数值。如果设置为整数值，$display_round$ 和 $print_round$ 会分别覆盖 $display_precision$ 和 $print_precision$，对于表 A-11 中的类似选项也是如此。例如，$display_round\ n$ 会使 display 命令将数值四舍五入到小数点后 n 位（如果 n < 0，则为小数点前 -n 位）。负精度的 $f$ 格式与 print_round 为负值时 print 命令的行为相同。影响打印的选项包括表 A-11 中所示的选项。

<table><tr><td>csvdisplay_precision</td><td>csvdisplay 和 _display 的精度 (0 表示完整精度)</td></tr><tr><td>csvdisplay_round</td><td>csvdisplay 和 _display 的舍入方式 (' ' 表示完整精度)</td></tr><tr><td>display_col</td><td>一维表按每行一个元素显示时的最大元素数</td></tr><tr><td>display_eps</td><td>将小于 $display_eps$ 的绝对数值显示为零</td></tr><tr><td>display_max_2d_cols</td><td>若大于 0，则为二维显示中的最大数据列数</td></tr><tr><td>display_precision</td><td>当 display_round 非数值时，display 命令的精度</td></tr><tr><td>display_round</td><td>display 命令的小数位舍入数</td></tr><tr><td>display_transpose</td><td>若行数 - 列数 < display_transpose，则转置表格</td></tr><tr><td>display_width</td><td>print 和 display 命令的最大行长度</td></tr><tr><td>expand_precision</td><td>当 expand_round 非数值时，expand 命令的精度</td></tr><tr><td>expand_round</td><td>expand 命令的小数位舍入数</td></tr><tr><td>gutter_width</td><td>display 命令的列间间距</td></tr><tr><td>objective_precision</td><td>求解器显示的目标函数值精度</td></tr><tr><td>omit_zero_cols</td><td>若非零，则在显示中省略全零列</td></tr><tr><td>omit_zero_rows</td><td>若非零，则在显示中省略全零行</td></tr><tr><td>output_precision</td><td>非线性表达式 (.nl) 文件中使用的精度</td></tr><tr><td>print_precision</td><td>当 print_round 非数值时，print 命令的精度</td></tr><tr><td>print_round</td><td>print 命令的小数位舍入数</td></tr><tr><td>print_separator</td><td>print 命令打印值的分隔符</td></tr><tr><td>solution_precision</td><td>当 solution_round 非数值时，solve 或 solution 命令的精度</td></tr><tr><td>solution_round</td><td>solve 或 solution 命令的小数位舍入数</td></tr></table>

表 A-11：控制打印的选项。

`_display` 和 `csvdisplay` 命令是 `display` 命令的变体，它们以比 `display` 更规则的格式输出表格：表格的每一行都以 $s$ 个下标开头，并以 $k$ 个项目结尾，所有项目都用逗号分隔。`_display` 和 `csvdisplay` 在输出的表头方面有所不同。`_display` 的表头由一行组成，该行以 `_display` 开头，后跟三个整数 $s$、$k$ 和 $n$（表示表头后跟随的表格行数），每个整数前面都有一个空格。如果 `csvdisplay_header` 为 1，`csvdisplay` 会在数据值前添加一个表头行，列出 $k$ 个索引和 $n$ 个表达式的名称。如果 `csvdisplay_header` 为 0，则省略此表头行。

# A.17 读取数据

`read` 命令提供了一种将非格式化数据读入 AMPL 参数和其他组件的方法，其语法类似于 `print` 命令：

```
read [索引 : ] 参数列表 重定向选项;
```

与 `print` 命令类似，可选的索引机制会使 `read` 命令针对指定索引集的每个成员分别执行。

`read` 命令将其输入视为一个由空白字符分隔的、未格式化的数据值序列。`arglist` 是一个以逗号分隔的参数列表，用于指定一系列组件，输入值将被赋给这些组件。与 `print` 命令类似，`arglist` 是一个以逗号分隔的参数列表，其中的参数可以是以下任意形式：

```
arg: component-ref indexing-expr component-ref indexing-expr (arglist)
```

`component-ref` 必须是一个可能带有后缀的 `parameter`（参数）、`variable`（变量）或 `constraint`（约束）的引用，或者是一个带后缀的问题名称；将值读入集合成员或更一般的表达式是没有意义的。所有索引必须显式给出，例如 `read {j in DEST} demand[j]` 而不是 `read demand`。值会按照读取顺序赋给参数，因此后面的参数可以使用同一 `read` 命令中先前读取的值。

如果没有指定重定向，值将从当前输入流中读取。因此，如果 `read` 命令是在 AMPL 提示符下发出的，则需要在后续提示符下逐个输入值，直到 `arglist` 中的所有条目都被赋值为止。当所有必要的输入读取完毕后，提示符将从 `ampl?` 恢复为 `ampl:`。如果 `read` 命令位于通过 `include` 或 `commands` 读取的脚本文件中，则输入将从同一文件中紧接在结束 `read` 命令的分号之后的位置开始读取。

大多数情况下，`read` 的输入位于一个单独的文件中，该文件通过可选的重定向指定；其形式为 `<filename`，其中 `filename` 是一个字符串或括号括起的字符串表达式，用于标识文件。多个 `read` 命令可以访问同一文件，在这种情况下，每个 `read` 命令都从上一个 `read` 命令结束的位置开始读取。若要强制从头开始读取，可以在重新读取之前 `close filename`。

如果脚本中包含一个 read 命令需要从交互式输入中读取值，则必须将输入源重定向到标准输入；通过将文件名指定为 -（减号）可以实现这一点。这通常用于从用户处读取交互式响应。

# A.18 建模命令

# A.18.1 solve 命令

solve 命令的形式为：

solve redirectionopt

该命令会使 AMPL 将当前转换后的问题写入目录 $\mathparagraph$ TMPDIR 的临时文件中（除非当前优化问题自上次 write 命令以来未发生更改），然后调用一个 solver（求解器），并尝试从求解器中读取最优的原始变量和对偶变量。如果成功，最优变量值将可用于打印和其他用途。可选的重定向用于指定求解器的标准输出。

当前 求解器 选项 的 值 决定 调用 哪个 求解器。在 $\mathparagraph$ solver 后 添加 '_oopt' 可 得到 一个 选项 的 名称，若 该 选项 被 定义 为 一个 非空 字符串，则 由 该 字符串 的 首 字母 决定 写入 的 临时 问题 文件 的 格式；否则，AMPL 使用 其 通用 的 二进制 输出 格式（格式 b）。例如，若 $\mathparagraph$ solver 为 supersol，则 $\mathparagraph$ supersol_oopt（若 非空）决定 输出 格式。命令行 选项 '-o?'（A.23）显示 当前 支持 的 输出 格式 的 摘要。

AMPL 向 求解器 传递 两个 命令行 参数：临时 文件 的 根名称，以及 字符串 -AMPL。AMPL 期望 求解器 将 对偶 和 原始 变量 的 值 写入 文件 stub.sol，前面 附有 注释，若 适当，该 注释 报告 目标 函数 值 到

\ $objective_precision 有效 数字。在 读取 解 时，若 \$solution_round 为 整数，则 AMPL 将 原始 变量 四舍五入 到 小数点 后 \$solution_round 位；若 \$solution_precision 为 正 整数，则 四舍五入 到 \$solution_precision 有效 数字；这些 选项 的 默认值 表明 不 进行 四舍五入。

变量 总是 具有 一个 当前 值。变量 声明 或 数据 部分 可以 指定 初始 值，否则 为 0。选项 reset_initial_guesses 控制 传递 给 求解器 的 初始 猜测。若 选项 reset_initial_guesses 的 值 为 默认值 0，则 当前 变量 值 被 作为 初始 猜测 传递。将 选项 reset_initial_guesses 设置 为 1 会 导致 原始 初始 值 被 发送。因此 \$reset_initial_guesses 影响 第二 次 求解 命令 的 初始 猜测，以及 在 解 命令（如下 所述）后 的 初始 求解 命令 的 初始 猜测。

约束 总是 具有 一个 关联 的 当前 对偶 变量 值 (拉格朗日乘数)。初始 对偶 值 为 0，除非 在 数据 部分 指定 或 在 约束 声明 中 通过 $\epsilon =$ initial_dual 或 default initial_dual 短语 指定。是否 将 对偶 初始 猜测 传递 给 求解器 由 选项 dual_initial_guesses 控制。其 默认值 1 使 AMPL 将 当前 对偶 变量 (若 \$reset_initial_guesses 为 0) 或 原始 初始 对偶 变量 传递 给 求解器；若 \$dual_initial_guesses 设置 为 0，则 AMPL 将 忽略 对偶 变量 的 初始 值。

AMPL 的 预求解 阶段 会 计算 变量 的 两组 边界。第一组 边界 反映 了 由 被 消除 的 约束 所 隐含 的 声明 边界 的 任何 收紧。第二组 边界 则 包含 了 预求解 从 无法 从 问题 中 消除 的 约束 中 推导 出 的 第一组 边界 的 进一步 收紧。无论 使用 哪 一组 边界，问题 的 解 都 是 相同 的，但 对于 使用 活动集 策略 (如 单纯形法) 的 求解器 来说，使用 收紧 后 的 边界 可能 会 在 退化 问题 上 遇到 更多 困难。求解器 通常 在 使用 第一组 边界 时 运行 得 更快，但 有时 使用 第二组 边界 会 更快。默认 情况下，AMPL 传递 第一组 边界，但 如果 选项 var_bounds 设置 为 2，则 AMPL 会 传递 第二组 边界。变量 的 .lb 和 .ub 后缀 始终 反映 var_bounds 的 当前 设置；.lb1 和 .ub1 对应 第一组 边界，.lb2 和 .ub2 对应 第二组 边界。

如果 输出 样式 为 m，AMPL 会 写入 一个 常规 的 MPS 文件，而 选项 integer_markers 的 值 决定 了 是否 在 MPS 文件 中 通过 'INTORG' 和 'INTEND' 'MARKER' 行 来 标识 整数 变量。默认 情况下，integer_markers 为 1，这 会 导致 写入 这些 行；指定 选项 integer_markers 0 会 使 AMPL 省略 'MARKER' 行。

选项 relax_integrality 会 使 变量 的 integer 和 binary 属性 在 求解 和 写入 命令 中 被 忽略。也 可以 通过 设置 变量 的 .relax 后缀 来 控制 这一点 (A.11)。

默认 情况下，类型 为 IN 或 INOUT 的 suffix 值 (A.11.1) 会 被 发送 给 求解器，而 类型 为 OUT 或 INOUT 的 suffix 的 更新 值 会 从 求解器 获取，但 suffix 值 的 发送 和 接收 可以 通过 适当 设置 选项 send_suffixes 来 控制：如果 send_suffixes 为 1 或 3，则 suffix 值 会 被 发送 给 求解器；如果 send_suffixes 为 2 或 3，则 会 从 求解器 请求 更新 的 suffix 值。

是否 将 .sstatus 值 (A.11.2) 发送 给 求解器 由 选项 send_suffixes 和 send_statuses 决定；将 send_statuses 设置 为 0 会 在 send_suffixes 允许 发送 其他 suffix 时 阻止 .sstatus 值 的 发送。

大多数 求解器 (solver) 都会为 AMPL 内置的 符号参数 (symbolic parameter) `solve_message` 提供一个值。默认情况下，AMPL 会打印更新后的 `solve_message`，但将选项 `solver_msg` 设置为 0 可以抑制这种打印。大多数 求解器 (solver) 还会提供一个 数值返回码 (numeric return code) `solve_result_num`，它有一个对应的 符号值 (symbolic value) `solve_result`，该值从 `solve_result_num` 和 $\text{solve_result_table}$ 推导而来，类似于 符号后缀值 (symbolic suffix values)（A.11.1）。

默认情况下，AMPL 会对 变量 (variables) 和 约束 (constraints) 进行排列，使 求解器 (solver) 首先看到 非线性项 (nonlinear terms)。一些 求解器 (solver) 要求这样做，但对于其他 求解器 (solver)，有时通过适当设置选项 `nl_permute` 来 抑制 (suppress) 这些排列是有用的。该选项是以下各项的总和：

1 表示 排列约束 (permute constraints)  
2 表示 排列变量 (permute variables)  
4 表示 排列目标函数 (permute objective function)  

其 默认值 (default value) 为 3。

当存在 互补性约束 (complementarity constraints) 时，如果 “不等式互补不等式” 约束 (complementarity inequalities) 的数量加上 方程 (equations) 的数量等于 变量 (variables) 的数量，则称该 约束系统 (constraint system) 是 方阵的 (square)。某些 互补性求解器 (complementarity solvers) 要求系统为方阵，因此 AMPL 默认会对 非方阵系统 (non-square systems) 发出警告。可以通过调整选项 `compl_warn` 来更改此行为，该选项的值为以下各项之和：

1  对 非方阵互补性系统 (non-square complementarity systems) 发出警告  
2  发出警告并将 非方阵互补性系统 (non-square complementarity systems) 视为不可行  
4  忽略 变量 (variables) 与 方程 (equations) 的 显式匹配 (explicit matching)

# A.18.2 solution 命令

`solution` 命令的形式为 `solution filename;`

该命令使 AMPL 从 `filename` 中读取 原始变量 (original variables) 和 对偶变量 (dual variables) 的值，就像在执行 `solve` 命令期间由 求解器 (solver) 写入的一样。

# A.18.3 write 命令

`write` 命令的形式为 `write outopt-value opt;`

其中可选的 `outopt-value` 必须遵循 文件名 (filename) 的 引号规则 (quoting rules)。如果 `outopt-value` 存在，则 `write` 将 `outopt` 设置为 `outopt-value`。无论 `outopt-value` 是否存在，`write` 都会按照 `outopt` 的指示写出 转换后的模型问题 (transformed problem)：`outopt` 的 第一个字母 (first letter) 指定 输出格式 (output format)（A.18.1），其余部分用作创建 文件名 (filename) 的“词干”(stub)。例如，若 `outopt` 为 `"b/tmp/diet"`，`write` 命令将创建文件 `/tmp/diet.nl`，如果 `auxfiles` 选项有相应要求（A.18.4），还会创建 辅助文件 (auxiliary files) `/tmp/diet.row`、`/tmp/diet.col` 等。`solve` 命令关于 初始猜测值 (initial guesses)、边界 (bounds)、后缀 (suffixes) 等的规则同样适用。

# A.18.4 辅助文件 (Auxiliary files)

`solve` 和 `write` 命令可能导致 AMPL 写出 辅助文件 (auxiliary files)。对于 `solve` 命令，在 求解器名称 (solver name) 后添加 `_auxfiles` 可得到 控制所写辅助文件 (control the auxiliary files written) 的 选项名 (option name)；对于 `write` 命令，则由 `auxfiles` 扮演此角色。表 A-12 中所示的 辅助文件 (auxiliary files) 仅在 控制选项 (control option) 的值 包含指定的 关键字母 (key letter) 时才会被写出。如果 关键字母 (key letter) 为 大写 (uppercase)，则对应的 辅助文件 (auxiliary file) 仅在 问题是非线性时 (problem is nonlinear) 才会被写出。

| 关键字母 | 文件名        | 说明                                                                 |
|----------|---------------|----------------------------------------------------------------------|
| a        | stub.adj      | 加到 目标函数 (objective) 值上的常数                                 |
| C        | stub.col      | 求解器看到的变量的 AMPL 名称                                         |
| e        | stub.env      | 环境（由 solve 命令写入）                                            |
| f        | stub.fix      | 因其值已知而从问题中消去的变量                                       |
| p        | stub.spc      | MINOS 的 “specs” 文件，用于输出格式 m                                |
| r        | stub.row      | 求解器看到的 约束 (con) 和 目标函数 (objective) 的 AMPL 名称         |
| s        | stub.slc      | 从问题中消除的 约束 (con)                                            |
| u        | stub.unv      | 未使用的变量                                                         |

表 A-12：辅助文件。

# A.18.5 更改模型：delete、purge、redeclare

命令

delete namelist;

将删除 namelist 中的每个名称，并恢复其之前的含义（如果没有任何其他实体依赖于该名称，即 交叉引用 (xref) name (A.19.2) 报告无依赖项）。

命令

purge namelist;

将删除每个名称及其所有直接和间接依赖项。

语句 redeclare entity-declaration;

redeclare entity-declaration; 用给定的声明替换指定实体的任何现有声明，前提是该实体没有依赖项，或者新声明不改变实体的性质（其种类，例如 集合 (set) 或 参数 (param)，及其下标的数量）。redeclare 可以应用于以以下任意项开头的语句：

弧 (arc) 函数 最小化 (minimize) 参数 (param) 集合 (set) 后缀 (suffix) 变量 (var) 检查 (check) 最大化 (maximize) 节点 (node) 问题 (problem) subject to 表 (table)

会导致循环依赖的 redeclare 会被拒绝。

命令 delete check n; 删除第 n 个 check，而 redeclare check n indexing opt : ... ; 重新声明第 n 个 check。

# A.18.6 drop、restore 和 objective 命令

这些命令的形式为 drop indexingopt constr-or-obj-name redirectionopt : restore indexingopt constr-or-obj-name redirectionopt : objective objective-name redirectionopt : 其中 constr-or-obj-name 是 约束 (con) 或 目标函数 (objective) 的可能带下标的名称。drop 命令指示 AMPL 在（write 和 solve 命令中）不传输指定的实体；restore 命令取消相应 drop 命令的效果。如果 constr-or-obj-name 没有下标但是一个 约束 (con) 或 目标函数 (objective) 的索引 集合 (set) 的名称，则 drop 和 restore 会影响该 集合 (set) 的所有成员。

objective 命令安排仅将命名的 目标函数 (objective) 传递给 求解器 (solver)。发出 objective 命令等同于 drop 所有 目标函数 (objective)，然后 restore 命名的 目标函数 (objective)。

# A.18.7 fix 和 unfix 命令

这些命令的形式为  
`fix` indexingopt varname $\begin{array}{rl} [ & := \end{array}$ expr ] redirectionopt :  
`unfix` indexingopt varname $\begin{array}{rl} [ & := \end{array}$ expr ] redirectionopt :

其中 varname 是变量的可能带下标的名称。`fix` 命令指示 AMPL 在（write 和 solve 命令中）将指定变量视为固定在其当前值上，即视为常量；`unfix` 命令取消相应 `fix` 命令的效果。如果 varname 没有下标但是一个变量的索引集合的名称，则 `fix` 和 `unfix` 会影响该集合的所有成员。

在结束分号前可以出现可选的 $\coloneqq$ expr，在这种情况下，表达式会被赋值给正在 `fix` 或 `unfix` 的变量，就像通过 `let`（A.18.9）赋值一样。

# A.18.8 命名问题和环境 (Named Problems and Environments)

`problem` 声明 / 命令有三个功能：声明一个新问题，使先前声明的问题成为当前问题，以及打印当前问题的名称（以建立当前问题的问题命令的形式）。

`problem` name indexingopt environopt suffix-initializationopt : item-list  
声明一个新问题并指定其中的变量、约束和目标。出现在指定约束和目标中的其他变量将被固定（但可以通过 `unfix` 命令取消固定）。新问题将成为当前问题。初始时，当前问题是 `Initial`。问题声明中的 item-list 是一个逗号分隔的变量、约束和目标名称列表，这些名称可以是带下标的，并且每个名称前面可以选择性地加上索引。suffix-initialization 类似于约束声明中的初始化，不同之处在于它们出现在冒号之前。

命令 `problem` name : 使 name（一个先前声明的问题）成为当前问题，并且

problem redirectionopt

打印当前问题名称。`Drop/restore` 和 `fix/unfix` 命令仅适用于当前问题。变量值与参数一样是全局的；变量的固定 / 未固定状态取决于问题。类似地，约束的 drop / restore 状态取决于问题（缩减成本也是如此）。当前问题不限制 `let` 命令（A.18.9）。

声明问题时，可以可选地指定与问题相关的环境：环境短语的形式为

environ envname

指定问题的初始环境为 envname，如果环境是索引的，则 envname 必须带有下标。否则将创建一个与问题同名的新非索引环境，并且它继承当时的当前环境（选项值集合）。

在选项命令中，无修饰（常规）选项名称引用当前环境中的选项，符号 envname.opname 引用环境 envname 中的 $\S$ opname。声明

environ envname indexingopt :

声明环境 envname（或索引环境集合，如果存在索引）。如果没有索引，envname 成为当前问题的当前环境。

对于先前声明的环境，命令

environ envname

使指定的环境成为当前环境，命令

environ indexingopt envname $\coloneqq$ envname1

将环境 envname 复制到 envname1，其中 envname 和 envname1 如果声明时带有索引，则必须是下标的。初始环境称为 `Initial`。

# A.18.9 修改数据：reset, update, let

reset 命令有几种形式。

reset

使 AMPL 忘记所有模型声明和数据，并关闭所有通过重定向打开的文件，同时保留当前选项设置。

reset options

使 AMPL 将所有选项恢复到初始状态。它忽略当前的 $\S$ OPTIONS_IN 和 $\S$ OPTIONS_INOUT；如果需要，可以手动包含它们命名的文件。

reset data

使 AMPL 忘记所有在数据模式下读取的赋值，并允许为不同的问题实例读取数据。

reset data 名称列表

使 AMPL 忘记为 name-list 中实体读取的任何数据；名称之间的逗号是可选的。

reset data 命令强制重新计算所有 $=$ 表达式，而 reset data p 即使在 $\mathbb{P}$ 用 $=$ 表达式声明时，也强制重新计算该表达式中的随机函数（以及表达式中的任何用户定义函数）。

当问题（包括当前问题）的索引表达式改变时会进行调整，但先前显式的 drop/restore 和 fix/unfix 命令仍保持有效。reset problem 命令取消对先前显式 drop、restore、fix 和 suffix 命令的这种处理，并将问题恢复到其声明的 drop/fix 状态。该命令的形式为：

reset problem ; 适用于当前问题
reset problem probname

如果后一种形式提到了当前问题，则其效果与第一种形式相同。reset problem 不会影响问题的环境。

update data ;

允许在后续数据部分中更新所有数据一次：当前值可能被覆盖，但不会丢弃任何值。

update data name-list ;

仅授予对 name-list 中实体的更新权限。

let 命令

let indexing-opt name := expr ;

将可能被索引的集合、参数或变量 name 的值更改为表达式的值。如果 name 是集合或参数，则其声明不得指定 $=$ 短语。

命令

let indexing-opt name.suffix := expr ;

分配相应的后缀值（如果允许）。某些后缀值是派生的，不能被赋值；尝试这样做会导致错误消息。

# A.19 检查模型

# A.19.1 show 命令

命令

show namelist opt 重定向 opt；若未提供 namelist，则列出所有模型实体。如果实体有声明，则显示其声明；否则，根据每个名称的首字母列出相应类型的模型实体：

ch.. $==> $ 检查 c. $==> $ 约束 e.. $==> $ 环境 f. $==> $ 函数 o.. $==> $ 目标 pr.. $==> $ 问题 p.. $==> $ 参数 su.. $==> $ 后缀 s.. $==> $ 集合 t.. $==> $ 表格 v.. $==> $ 变量

# A.19.2 xref 命令

命令 xref 显示直接或间接依赖于指定实体的实体：

xref itemlist redirectionopt

# A.19.3 expand 命令

expand 命令用于打印生成的约束和目标：

expand indexingopt itemlist redirectionopt ;

itemlist 可以采用与问题声明中允许的形式相同的形式。如果为空，则展开所有未被丢弃的约束和目标。变体 solexpand indexingopt itemlist redirectionopt ; 显示约束和目标在求解器 (solver) 看来是如何呈现的。除非在 itemlist 中明确指定，否则它会省略 presolve 阶段消除的约束和变量。

expand 和 solexpand 命令均允许变量出现在 itemlist 中；对于每个变量，命令会显示该变量在线性相关的（未丢弃的，对于 solexpand 还包括未被 presolve 消除的）约束和目标中的线性系数，并在变量在某个约束或目标中也以非线性方式参与时标注“$+$ 非线性”。

选项 expand_precision 和 expand_round 控制 expand 命令打印数值的方式。默认情况下，数值按 6 位有效数字打印。

# A.19.4 通用名称

AMPL 提供了一些通用名称，可用于访问模型实体而无需使用模型特定的名称。其中一些名称在表 A-13 中有描述；完整的当前列表可在 AMPL 网站上找到。

这些同义词和集合可用于 display 和其他命令中。它们呈现的是建模者视角（presolve 之前）。类似地，将 _ 更改为 _s 的自动更新实体（即 _snvars、_svarnames、_svar 等）则提供求解器视角，即 presolve 之后的视角。然而，由于互补约束的处理方式（A.9.1），存在例外情况：_cvar、_sconname 或 _snccons 均不存在。

# A.19.5 check 命令

命令

check;

导致所有 check 语句被求值。

# A.20 脚本和控制流语句

AMPL 提供了类似于传统编程语言中控制流语句的语句，使得可以编写一个程序，以自动执行一系列语句。

<table><tr><td>nvars</td><td>当前模型中的变量 (variable) 数量</td></tr><tr><td>ncons</td><td>当前模型中的约束 (constraint) 数量</td></tr><tr><td>nobjs</td><td>当前模型中的目标函数数量</td></tr><tr><td>varname{1...nvars}</td><td>当前模型中变量的名称</td></tr><tr><td>connname{1...ncons}</td><td>当前模型中约束的名称</td></tr><tr><td>objname{1...nobjs}</td><td>当前模型中目标函数的名称</td></tr><tr><td>var{1...nvars}</td><td>当前模型中变量的同义词</td></tr><tr><td>con{1...ncons}</td><td>当前模型中约束的同义词</td></tr><tr><td>obj{1...nobjs}</td><td>当前模型中目标函数的同义词</td></tr><tr><td>PARS</td><td>所有已声明参数名称的集合</td></tr><tr><td>SETS</td><td>所有已声明集合名称的集合</td></tr><tr><td>VARS</td><td>所有已声明变量名称的集合</td></tr><tr><td>CONS</td><td>所有已声明约束名称的集合</td></tr><tr><td>OBJS</td><td>所有已声明目标函数名称的集合</td></tr><tr><td>PROBS</td><td>所有已声明问题名称的集合</td></tr><tr><td>ENVS</td><td>所有已声明环境名称的集合</td></tr><tr><td>FUNCSC</td><td>所有已声明用户定义函数的集合</td></tr><tr><td>ncons</td><td>预处理 (presolve) 前的互补约束 (complementarity constraint) 数量</td></tr><tr><td>ccname{1...ncons}</td><td>互补约束的名称</td></tr><tr><td>scvar{1...sncons}</td><td>若 scvar[i]&gt;0，则 scvar[scvar[i]] 与 scon[i] 互补</td></tr><tr><td>snbvars</td><td>二元 (0,1) 变量的数量</td></tr><tr><td>snccons</td><td>预处理后的互补约束数量</td></tr><tr><td>snivars</td><td>整数变量 (不包括二元变量) 的数量</td></tr><tr><td>snlcc</td><td>线性互补约束的数量</td></tr><tr><td>snlnc</td><td>线性网络约束的数量</td></tr><tr><td>snnlcc</td><td>非线性互补约束的数量::_snccons=snlcc+_snnlcc</td></tr><tr><td>snnlcons</td><td>非线性约束的数量</td></tr><tr><td>snnlnc</td><td>非线性网络约束的数量</td></tr><tr><td>snnlobjs</td><td>非线性目标函数的数量</td></tr><tr><td>snnlv</td><td>非线性变量的数量</td></tr><tr><td>snzcons</td><td>约束雅可比矩阵 (Jacobian matrix) 非零元素的数量</td></tr><tr><td>snzobjs</td><td>目标函数梯度非零元素的数量</td></tr></table>

表 A-13：通用同义词和集合。

# A.20.1 for、repeat 和 if-then-else 语句

多个命令允许对 AMPL 命令列表进行条件执行和循环操作：

if lexpr then cmd  
if lexpr then cmd else cmd  
for loopname opt indexing cmd  
repeat loopname opt opt- while { cmds } opt- while ;  
break loopname opt ;  
continue loopname opt ;

在这些语句中，`cmd` 是一个单独的、可能为空的命令 (以分号结尾) ，或者是由零个或多个命令组成的序列 (用花括号括起来) 。`lexpr` 是一个逻辑表达式。`loopname` 是一个可选的循环名称 (该名称必须在语法上循环开始之前未被绑定) ，在循环语法结束之后该名称将超出作用域。如果存在，可选的 `while` 条件具有以下形式之一：

    while lexpr
    until lexpr

如果指定了 `loopname`，则 `break` 和 `continue` 适用于指定的外层循环；否则它们适用于直接包含的循环。`break` 会终止循环，而 `continue` 会导致循环进入下一次迭代 (如果被 `repeat` 循环的可选初始和最终 `while` 子句，或 `for` 循环的索引所允许) 。在 `for` 循环中，索引生成的虚拟索引可以在 `cmd` 中出现。`for` 循环的整个索引集在循环开始执行之前被实例化，因此循环体执行所使用的虚拟索引集合不会受到循环体内赋值的影响。

`break` 的变体 `break commands;`、`break all;` 和 `terminate` 分别终止当前的 `commands` 命令或所有的 `commands` 命令 (如果存在) ，否则它们的行为类似于 `quit` 命令。

循环和 `if-then-else` 结构会一直被保留，直到语法上完整为止。由于 `else` 子句是可选的，AMPL 必须向前查看一个标记以检查其是否存在。因此，在最外层，必须发出一个空命令 (仅一个分号) 或其他命令或声明，以执行一个没有 `else` 子句的最外层 `if` 语句。(在这方面，文件结束意味着隐式的空语句。)

在以缺少可选最终部分的复合命令结尾的命令文件末尾，会自动添加一个分号：

    repeat ... { ... } # 没有最终条件或分号
    if ... then { ... } # 没有 else 子句

AMPL 有三对提示符，其文本可以通过选项设置进行更改。默认设置为：

    option cmdprompt1 '%s ampl: ';
    option cmdprompt2 '%s ampl? ';
    option dataprompt1 'ampl data: ';
    option dataprompt2 'ampl data? ';
    option prompt1 'ampl: ';
    option prompt2 'ampl? ';

当期望一个新语句时显示 `prompt1`，而当上一行输入尚未构成完整命令时 (例如，缺少末尾的分号) 则显示 `prompt2`。

在数据模式下，将使用 `dataprompt1` 和 `dataprompt2` 的值。当在 `if`、`for` 或 `repeat` 语句中间开始一个新行时，将使用 `cmdprompt1` 和 `cmdprompt2` 的值，并将 `%s` 替换为适当的命令名称；例如：

    ampl: for {t in time} {
    for...}
    { ? ampl: if t <= 6 for...}
    { ? ampl? then let cmin[t] := 3;
    if ... then {...}
    ? ampl: else let cmin[t] := 4;
    for...}
    { ? ampl: };
    ampl:

# A.20.2 逐步执行命令

可以逐个命令地逐步执行 AMPL 脚本中的命令。通过以下方式启用单步模式：

    option single_step n :

其中 $n$ 是一个正整数；它指定如果插入层级最多为 $n$，则 AMPL 应该表现得像在每个命令之前插入了 `commands -` 一样：它应从标准输入读取命令，直到结束或遇到其他文件结束信号（在 Unix 上是 control-D，在 Windows 上是 control-Z）。在此模式下可能会出现一些特殊命令：

`step` $n_{\mathrm{opt}}$ 执行下一个命令，或 $n$ 个命令  
`skip` $n_{\mathrm{opt}}$ 跳过下一个命令，或 $n$ 个命令  
`next` $n_{\mathrm{opt}}$ 如果下一个命令是 `if-then-else` 或循环命令，则执行整个复合命令，或 $n$ 个命令，然后再次停止（除非该复合命令本身指定了 `commands -`）  
`cont` 执行直到当前插入层级的所有嵌套复合命令结束

# A.21 计算环境

A.21 计算环境  
AMPL 在操作系统环境中运行，通常作为一个独立程序运行，但有时也隐藏在图形用户界面或更大的系统后台。其行为受到外部环境中值的影响，并且它可以设置成为该环境一部分的值。参数 `_pid` 给出了 AMPL 进程的进程 ID（是系统上运行的进程中唯一的数字）。

# A.21.1 shell 命令

`shell` 命令提供了一个临时跳转到操作系统的机会（如果允许的话）来运行命令。

```
shell 'command-line' redirection_{\mathrm{opt}} :
shell redirection_{\mathrm{opt}} :
```

第一个版本运行包含在文字字符串中的 `command-line`。在第二个版本中，AMPL 调用一个操作系统 shell，并在该 shell 终止时控制权返回到 AMPL。在调用 shell 之前，AMPL 将当前选项及其值列表写入由选项 `shell_env_file` 指定的文件（如果有）。shell 程序的名称由选项 `SHELL` 确定。

# A.21.2 cd 命令

`cd` 命令报告或更改 AMPL 的工作目录。

```
cd
cd new-directory :
```

参数 `cd` 被设置为此值。

![](https://cdn-mineru.openxlab.org.cn/result/2025-09-04/c27db974-6ee8-418f-9d81-3e975c6552ef/d2dd717744044a0698d9d928d8896c01b7ca7d72eee253c344f992020ba9b225.jpg)  
表 A-14：内置计时参数。

# A.21.3 quit、exit 和 end 命令

`quit` 命令使 AMPL 停止运行，而不写入任何由 `$\) 0\(utopt` 暗示的文件，而 `end` 命令使 AMPL 表现得好像已到达当前输入文件的末尾，但不会恢复到模型模式。在命令解释的顶层，任一命令都会终止 AMPL 会话。命令 `exit` 是 `quit` 的同义词，但它可以向周围环境返回一个状态：

```
exit expression_{\mathrm{opt}};
exit expression_{\mathrm{opt}};
```

# A.21.4 内置计时参数

A.21.4 内置计时参数

AMPL 具有内置参数，用于记录各种 CPU 和经过的时间，如表 A-14 所示。大多数现代操作系统都会分别跟踪两种 CPU 时间：系统时间（操作系统代表进程所花费的时间，例如读写文件）和用户时间（进程本身在操作系统之外所花费的时间）。通常系统时间远小于用户时间；如果不是这样，查明原因有时可以提供改进性能的方法。由于在性能较差时查看系统时间和用户时间分别有助于分析，AMPL 提供了用于记录这两种时间以及它们总和的内置参数。AMPL 将求解器 (solver) 和 shell 命令都作为独立进程运行，因此它提供了单独的参数来记录每种进程所花费的时间，以及 AMPL 进程本身的时间。

# A.21.5 日志记录

如果选项 log_file 是一个非空字符串，则将其视为文件名，AMPL 会将从标准输入读取的所有内容复制到该文件中。如果选项 log_model 为 1，则从其他文件读取的命令和声明也会复制到日志文件中；如果 log_data 为 1，则从其他文件读取的数据部分也会复制到日志文件中。

# A.22 导入函数

有时，借助 AMPL 未内置的函数来表达模型会更加方便。AMPL 提供了导入函数的功能，并可选择性地检查其参数列表的一致性。注意：使用导入函数的实际细节高度依赖于系统。本节仅涉及语法；具体信息将在系统特定的文档中找到，例如在 AMPL 网站上。

在翻译问题时，可能需要对导入函数进行求值；例如，如果该函数在确定集合内容时起作用，则 AMPL 必须能够对该函数求值。在这种情况下，该函数必须与 AMPL 链接，可能是动态链接。另一方面，如果导入函数的唯一作用是计算约束或目标的值，则 AMPL 永远不需要对该函数求值，并可以简单地将其引用传递给（非线性）求解器。

在引用导入函数之前，必须在函数声明中声明它们。该语句的形式为：

```
function name aliasopt (domain-spec) opt typeopt [ pipe litseqopt [ format fmt ] ] :
```

其中 name 是函数名称，而 domain-spec 相当于函数原型：

```
domain-spec: domain-list nonempty-domain-list
```

domain-list 是一个（可能为空的、逗号分隔的）集合表达式列表、星号 $(\ast \ast)$、方向词（IN、OUT 或 INOUT）、方向词后跟集合表达式，以及迭代域列表：

```
iterated-domain-list: indexing ( nonempty-domain-list )
```

迭代域列表 (iterated-domain-list) 等价于对索引集中的每个成员，将其域列表 (domain-list) 重复一次，而出现在索引中的哑变量 (dummy variables) 的定义域则扩展至该域列表。方向词 (direction words) 用于指示信息的流向：流入函数 (IN)、从函数流出 (OUT)，或两者皆有，其中 IN 为默认方向。在函数调用中，OUT 参数在调用函数的命令结束时被赋予函数所指定的值。

在函数声明中省略可选的 (domain-spec) 与指定 $(\ldots)$ 是等价的。根据 domain-spec 是否以 ... 结尾，函数调用时必须至少或恰好提供与 domain-spec 中集合数量相同的参数（在迭代域列表展开之后）。AMPL 会检查与 domain-list 中集合对应的每个参数是否属于该集合。在 domain-list 中单独出现的 * 表示对该参数不进行域检查。

对于返回值不重要的函数，可以使用 call 命令进行调用：

call funcname(arglist);

类型 (Type) 可以为 symbolic 或 random 或两者皆有；symbolic 表示该函数返回一个字面量（字符串）值而非数值，random 表示该“函数”对于相同的参数可能返回不同的值，即 AMPL 应当假设每次调用该函数时返回不同的值。

命令 load libnamesopt; unloadabed libnamesopt; reload libnamesopt;

用于加载、卸载或重新加载共享库（函数和表处理器从中导入）；libnames 是一个由逗号分隔的库名称列表。当 load 和 unload 命令中至少提到了一个 libname 时，$\mathparagraph$ AMPLFUNC 会被修改以反映当前加载库的完整路径名。reload 命令首先卸载其参数，然后重新加载它们。这可以改变加载库的顺序，并影响导入函数的可见性：第一个名称优先。在没有参数时，load 会加载当前 $\mathparagraph$ AMPLFUNC 中的所有库；unload 会卸载所有当前加载的库，而 reload 会重新加载它们（如果某些库已被重新编译，这将非常有用）。

关键字 `pipe` 表示这是一个管道函数 (pipe function)，意味着 AMPL 应该启动一个独立的进程来计算该函数。每当需要函数值时，AMPL 会向该函数进程写入一行参数，然后从该进程中读取包含函数值的一行。（当然，这仅在允许创建多个进程的系统上可行。）`litseq` 是一个或多个相邻的字面量 (literal) 或带括号的字符串表达式组成的序列，AMPL 会将其连接起来，并作为要调用的进程描述传递给操作系统（即传递给 SHELL）。如果没有指定 `litseq`，则 AMPL 会传递一个值为函数名称的单个字面量。如果存在可选的格式 `fmt`，那么 `fmt` 必须是一个适用于 `printf` 的格式字符串，用于告诉 AMPL 如何格式化它发送给函数进程的每一行。如果没有指定 `fmt`，则 AMPL 使用空格分隔传递给管道函数的参数。

例如：

```python
amp1: function mean2 pipe "awk '{print ($1+$2)/2}'";
amp1: display mean2(1,2) + 1;
mean2(1,2) + 1 = 2.5
```

函数 `mean2` 默认期望返回数值；如果它返回一个不代表数值的字符串，AMPL 将报错。

以下函数是 symbolic 的，用于说明格式化和参数传递。

```python
amp1: function fl symbolic pipe "awk '' '{printf \"%s\\n\", $1}'; ";
amp1: function gl symbolic pipe "awk '' '{printf \"XX%s\\n\", $1}'; ";
amp1: function cat symbolic pipe format ">>%s<<\\n";
amp1: display fl(2/3);
fl(2/3) = x0.666666666666667
amp1: display gl('abc');
gl('abc') = XXabc
amp1: display cat('some words');
cat('some words') = ">>'some words'<<"
```

`fl` 的声明指定了一个包含 3 个字面量的 `litseq`，而 `gl` 指定了一个字面量；由于 `cat` 的 `litseq` 为空，因此它被当作其 `litseq` 为 `'cat'` 来处理。每个 `litseq` 中的字面量都会去除包围它们的引号，相邻的每对引号中只保留一个，并将（反斜杠，换行）对替换为单个换行字符；处理结果被连接起来，形成传递给操作系统作为要启动的进程描述的字符串。因此，对于上述四个管道函数，系统看到的命令分别为：

```
awk '{print ($1+$2)/2}'
awk '{printf "x%s\n", $1}'
awk '{printf "XX%s\n", $1}'
cat
```

函数 `cat` 展示了可选的格式 `fmt` 用法。如果 `fmt` 结果的字符串不以换行符结尾，AMPL 会追加一个换行字符。如果没有指定 `fmt`，则每个数值参数会被转换为最短的十进制字符串，该字符串能四舍五入到参数值。

注意：管道函数返回的行必须是完整的一行，即必须以换行符结尾，并且管道函数进程必须刷新其缓冲区以防止死锁。（管道函数无法与大多数标准 Unix 程序一起使用，因为它们不会在每行末尾刷新输出。）

导入的函数可以使用传统的函数调用方式来调用，如上所示。此外，还允许使用迭代参数。更准确地说，如果 f 是一个导入的函数，则对 f 的调用形式为 f (arglist)，其中 arglist 与 print 和 printf 命令中的参数列表类似，是一个可能为空的、由逗号分隔的表达式和迭代参数列表：

ampl: function mean pipe 'awk '{x = 0; for(i = 1; i <= NF; i++) x += $(i); printf "%.17g\n", x / NF}' f;  
ampl: display mean({i in 1..100} i);  
mean({i in 1..100} (i)) = 50.5  
ampl: display mean({i in 1..50} (i,i + 50));  
mean({i in 1..50} (i, i + 50)) = 50.5  
ampl: display mean({i in 0..90 by 10}({j in 1..10} i + j));  
mean({i in 0..90 by 10} ({j in 1..10} (i + j))) = 50.5

命令 reset function name 会关闭所有管道函数，使得它们在再次被调用时重新启动。如果显式指定了某个函数名，则仅关闭该函数。

# A.23 AMPL 调用

AMPL 通常作为独立命令在某种操作系统环境中调用。当 AMPL 开始执行时，可以交互式地输入上述（A.14 节）描述的声明、命令和数据部分。根据运行 AMPL 的操作系统不同，调用时可能伴随一个或多个命令行参数，用于设置各种属性和选项，并指定要读取的文件。这些可以通过输入以下命令查看：

ampl - - 2

某些选项的初始化可能由命令行参数决定。参数 `-?` 会列出这些选项及其命令行等价形式。

有时，为了在不同的 AMPL 会话之间记住选项设置，可以利用操作系统继承环境变量的功能，如上所述。选项 OPTIONS_IN、OPTIONS_INOUT 和 OPTIONS_OUT 提供了一种实现方式。如果在继承的环境中 $OPTIONS_IN 非空，则它指定了一个文件（应包含选项命令），AMPL 在处理命令行参数或进入命令环境之前会先读取该文件。OPTIONS_INOUT 类似于 OPTIONS_IN；AMPL 会在读取 $OPTIONS_IN 后读取文件 $OPTIONS_INOUT（如果非空）。在执行结束时，如果 $OPTIONS_INOUT 非空，AMPL 会将当前的选项设置写入文件 $OPTIONS_INOUT。如果 $OPTIONS_OUT 非空，则在执行结束时将其视为 $OPTIONS_INOUT 处理。

命令行参数 -v 会打印当前使用的 AMPL 命令版本；该信息也可通过选项 version 获取。

命令行选项 `-R`（仅在作为第一个命令行选项时被识别，且不会出现在 `-?` 选项列表中）会将 AMPL 置于受限的“服务器模式”下。在此模式中，AMPL 拒绝执行 `cd` 和 shell 命令，禁止更改选项 `TMPDIR`、`amp1_include` 和 `PATH`（或所使用操作系统的搜索路径），禁用管道函数，并限制选项 `solver` 和文件重定向中的名称只能为字母数字（因此它们只能写入当前目录，至少在 Unix 系统上，该目录无法更改）。通过在 shell 脚本中调用 AMPL，并在调用 `amp1 -R` 之前适当调整当前目录和环境变量，可以控制 AMPL 运行的目录及其初始环境。

在支持导入函数库的系统上，命令行选项 `-i libs` 指定 AMPL 初始应加载的导入函数（A.22）和表处理程序（A.13）库。如果未指定 `-i libs`，则 AMPL 默认使用 `-i $\mathparagraph$ AMPLFUNC`。此处 `libs` 是一个字符串，可能跨越多行，每行指定一个库或目录的名称。对于目录，AMPL 会在该目录中查找名为 `amplfunc.dll` 的库。如果 `libs` 为空且当前目录中存在 `amplfunc.dll`，则 AMPL 初始加载 `amplfunc.dll`。如果操作系统认为 `ampltabl.dll` 安装在标准位置，AMPL 也会尝试加载该库，该库可提供“标准”的数据库处理程序和函数。

.. 参见 等差数列；参见 分号语句终止符  
“..” 参见 引号  
“>>> <<” 参见 错误信息  
\# 注释 10, 453  
$\hat{\mathbf{\Omega}}_{\mathrm{~\scriptsize~\hat{~}{~\Omega~}~}}\star \star$ 指数 111, 456  
$^+$ 在集合数据表中 156-157, 475  
$>,\> >$ 输出重定向 220, 239, 251, 273, 481  
$\coloneqq$ 属性 131, 398, 466, 468  
$=$ 属性 另请参见 定义变量  
$=$ 属性 74, 118-120, 211, 462, 465  
$\S$ 列名缩写 230  
列名运算符 177, 184-185, 188, 190, 478  
-? 命令行参数 499-500  
$< \ldots >$ 数据规范 196-197, 479  
.default 符号 120, 153, 156-157, 228, 477  
$\{\}$ 空集 74, 146, 162, 262, 264, 341, 456-457, 462  
\* 在 option 命令中 205, 481  
$<$ 输入重定向 163, 165, 481, 485  
$<< \ldots >>$ 分段线性表达式 367, 369, 376, 380-382, 460  
$\frac{9}{6}$ printf 的 c 转换字符 239, 482  
$->$ 读/写状态 189, 193, 478  
$\epsilon_{--}$ 读/写状态 175, 177, 182, 193  
$\epsilon_{--}$ 读/写状态 193, 478  
标准输入文件名 165, 283, 485, 495  
& 字符串连接运算符 270, 459  
$\coloneqq$ 与 $=$ 的区别 初始值 211  

abs 函数 113, 140, 380, 458  
Access 数据库 170, 174-175, 178, 192, 194  
Access 文件扩展名 .mdb 199-200  
阿克曼函数 126  
acos 函数 113, 458

acosh 函数 113, 458 active-set 方法 279, 486 活动模型 43 广告模型 21, 450 聚合模型 55, 88 阿胡贾, 拉文德拉 K. 347 航空公司机组调度 362 阿拉托雷, 海梅 20 代数建模语言 xvii, xxi 别名函数 458 ampl 命令行 203, 499 数据提示 144 ampl: 提示 5, 165, 204, 485 ampl? 提示 165, 204, 485 AMPL 网页 参见 www.ampl.com ampl_include 选项 479 _ampl_time 参数 251, 496 AMPLFUNC 选项 498, 500 and 运算符 114, 456 弧声明 334-341, 353, 467 沿弧损失 326 参数 参见 命令行参数 算术表达式 61, 76, 111-114, 456 函数, 表 113, 458 逻辑, 集合运算符, 表 456 运算符 456 运算符, 表 112 进展 59, 75, 457 归约运算符 456 转换为逻辑 123, 457 元数函数 462, 464 阿罗诺夫斯基, 朱利叶斯 S. 20 数组 参见 一维集合, 参数 ASCII 文件扩展名, .tab 201, 477 ASCII 集合 87, 464

asin 函数 113, 458

asinh 函数 113, 458

可赋值表达式 185

分配模型 49-51, 53, 135, 330-333, 348

.astatus 后缀 294-295

astatus_table 选项 294

atan, atan2 函数 113, 458

atanh 函数 113, 458

属性

: $=$ 131, 398, 466, 468

$=$ 74, 118-120, 211, 462, 465

二进制 117, 122, 131, 441, 465-466

循环 83, 98, 146, 463

按循环 86, 463

系数 356, 362, 466

覆盖 361, 467

默认 74, 119-120, 160-161, 462, 465-466, 468, 476-477

维度 93, 96, 462

从 334, 336, 338, 341, 353, 361, 363, 467

整数 36, 116, 131, 438, 441, 465-466

目标 334, 339, 341, 356, 363, 466

有序 83, 98, 146, 463

按有序 86, 463

有序的继承 86, 463

管道 498

随机 497

反向 86, 463

后缀 466, 469-470, 489

符号 123, 465, 476, 497

到 334, 336, 338, 341, 353, 361, 363, 467

详细 200

辅助文件，表 488

bad_subscripts 选项 253

基本解 287, 289-290, 292, 298, 473

比尔, E.M.L. xxi

Benders 分解 301

本特利, 乔恩 L. xxi

伯特塞卡斯, 迪米特里 P. 347

Beta 函数 459

双矩阵博弈 426, 433

二进制文件扩展名，.bit 201, 477

二进制属性 117, 122, 131, 441, 465-466

二分图 330

比斯霍普, 约翰内斯 xxi

.bit 二进制文件扩展名 201, 477

比克斯比, 罗伯特 E. xxi

混合

模型 37-39

非线性 38

.body 参见 约束体

有界变量互补 423-424

变量的边界 10, 14, 17, 32, 35, 130, 139, 241-242, 276, 279, 466, 470-471, 486

Braess 悖论 414

break 语句 266, 493

断点 参见 分段线性表达式

内置函数 参见 函数

内置

表处理程序 174, 201

计时参数，表 496

by 子句 75, 457

card 函数 79, 98, 462, 464

基数限制 445

笛卡尔积 参见 交叉积

餐饮商问题 347, 350

Cauchy 函数 459

cd 命令 495

_cd 当前目录 479, 495, 500

ceil 函数 212, 237, 458

char 函数 459

check 语句 46, 56, 117, 465

切丽, 洛琳达 L. xxi

查瓦塔尔, 瓦塞克 20, 317

循环

属性 83, 98, 146, 463

按属性 86, 463

close 命令 251, 481, 485

cmdprompt1, cmdprompt2 选项 494

coeff 属性 356, 362, 466

数据列的集合，索引 196

集合，索引 100-104, 161-162

表格，索引 193

列

键 170, 172, 175, 178, 180, 183, 193

名称，有效 171-172, 177, 185

列名

缩写，\(\) 230$

运算符，\~ 177, 184-185, 188, 190, 478

数据中的逗号分隔符 144

命令 参见 语句

命令模式 参见 模型模式

命令

语言 479-496

空值 494

命令

\* 在选项中 205, 481

cd 495

关闭 251, 481, 485

继续 495 csvdisplay 484 数据 11, 143-144, 206, 214, 256, 473, 479 删除 213, 488 删除检查 488 显示 12, 58, 67, 81, 104, 124, 219-238, 482-484 丢弃 214, 489 结束 204, 496 环境 490 退出 496 展开 13, 247-248, 492 修复 215, 489 包含 255-256, 479 让 36, 210-212, 405, 491 加载 402, 477, 498 模型 11, 206, 214, 256, 479 下一个 269, 495 目标 135-136, 206, 489 选项 204, 481 打印 238-239, 482-484 格式化打印 124, 239-240, 261, 482-484 问题 301, 307, 311, 489 清除 213, 488 退出 5, 204, 214, 496 读取 163, 484 读取表格 175, 177, 180, 477 重新声明 213, 488 重新声明检查 488 重新加载 498 移除 482 重置 17, 214, 481, 490 重置数据 122, 209-210, 490 重置选项 205, 490 重置问题 491 恢复 214, 489 shell 214, 495 显示 246, 491 跳过 270, 495 solexpand 248, 492 解 487 求解 206-207, 485-486 求解问题 315 步进 268, 495 后缀 296, 299, 302 解除修复 215, 489 卸载 498 更新数据 209, 491 写入 487 写入表格 178, 186, 477 交叉引用 213, 247, 492 命令行 ampl 203, 499

参数，-? 499-500 参数，-i 500 参数，-o 485 参数，-R 500 参数，-s 458 参数，-v 500 命令逐步执行 268-270, 495 命令表 480 命令语句 256, 479 注释，#，/*...*/ 10, 453 数据中的注释 47, 453, 473 比较运算符 参见 关系运算符 compl_warn 选项 487 互补性 有界变量 423-424 约束 427-428, 469, 471, 487 约束，后缀 428, 431, 471 通用名称和 431 预处理和 429 问题 420-423 补集运算符 469 计算参数 118-121 连接 参见 字符串连接运算符 凹函数 383, 393 条件声明 参见 if 索引表达式 条件表达式 参见 逻辑表达式，if-then-else 常量表达式 455, 457, 461 约束体 35, 242, 280, 470 互补性 427-428, 469, 471, 487 声明 xv, 2, 5, 9, 16, 137-139, 340, 461, 468-469 定义 386, 399, 467 双不等式 32, 139, 470 要么/要么 59 等式 21, 30, 130 非线性 59, 140, 492 冗余 276-278 约束声明，即将到来 356, 362, 468 cont 命令 495 继续 参见 行继续 continue 语句 266, 493 连续性假设 38, 391 控制流语句 492 算术到逻辑的转换 123, 457 数字到字符串的转换 271, 459 转换字符，printf 表 483 凸函数 383, 393, 396

cos 函数 113, 458 成本函数, 非线性 394 理查德·W·科特尔 432 科莱特·库拉德 xxi, 136, 141, 166, 287 覆盖属性 361, 467 覆盖问题, 集合 362, 364 CPLEX 求解器 36, 50, 208, 281 加里·I·克拉默 xxi 信用评分模型 386 机组调度, 航空公司 362 叉积 97- 98, 456 叉积算子 98, 456 csvdisplay 命令 484 ctime 函数 458 cut.mod, 文件 306 下料问题 41, 305, 364, 452

乔治·B·丹齐格 xxi, 39, 52 .dat 文件 参见 文件 数据重置 参见 重置 数据命令 数据值, 省略 参见 默认符号 数据列, 索引集合 196 数据中的逗号分隔符 144 数据中的注释 47, 453, 473 索引集合的数据 101, 161 格式 473- 477 模式 143, 206, 256, 479 多维列表 148- 151 多维表格 156- 160 一维列表 145- 146 参数 477 集合 473- 475 集合和参数 151- 154 表格, 集合和参数 476 二维列表 146- 148 二维表格 154- 156 未格式化数据 163 验证 12, 46, 93, 118 值, 默认 160- 161 数据规范, <...> 196- 197, 479 数据命令 11, 143- 144, 206, 214, 256, 473, 479 数据库处理程序 参见 表处理程序 数据-模型分离 7, 11, 74, 143 dataprompt1, dataprompt2 选项 494 声明 参见 语句声明 弧 334- 341, 353, 467 约束 xv, 2, 5, 9, 16, 137- 139, 340, 461, 468- 469 节点 334- 340, 353, 468 目标 134- 137, 470

参数 8, 110- 111, 465- 466 问题 489 集合 8, 74, 461- 465 表 174, 178, 180, 186, 477 变量 5, 8, 466- 467 声明函数 402, 497 to_come 约束 356, 362, 468 声明, 后缀 471 默认读/写状态 参见 INOUT 默认数据值 160- 161 默认符号 120, 153, 156- 157, 228, 477 默认属性 74, 119- 120, 160- 161, 462, 465- 466, 468, 476- 477 与 $=$ 初始值 119 定义变量 466- 468 定义约束 386, 399, 467 删除检查命令 488 命令 213, 488 导数 407 饮食模型 27- 37, 39, 77- 78, 135 整数解 36, 439 diff 算子 76, 98, 456- 457 可微性假设 391 dimen 属性 93, 96, 462 有向网络图 320 .direction 后缀 296 史蒂文·P·迪克斯 433 显示命令 484 显示命令 12, 58, 67, 81, 104, 124, 219- 238, 482- 484 格式化选项 227- 230 格式化选项表 227 数值选项 232- 238 数值选项表 232 显示集 87, 464 display_1col 选项 63, 228, 231 display_eps 选项 235 display_precision 选项 233, 263, 406, 483 display_round 选项 234- 235, 483 display_transpose 选项 229 display_width 选项 204, 229 显示索引表达式 224- 227 参数 220- 224 集合 220 除法算子 111, 456 双重不等式约束 32, 139, 470 删除命令 214, 489

阿恩·德鲁德 xxi .dual 参见 对偶值 对偶值 17, 22, 40, 66- 67, 162, 243- 245, 259, 262, 266, 470, 482 变量 398 变量 162, 243, 468, 472, 485 变量, 初始值 185, 468, 472, 477, 486 dual_initial_guesses 选项 486 虚拟索引 31, 47, 79, 92, 94- 95, 99, 138, 184, 188, 190- 191, 197, 455, 463 作用域 47, 321 古塔姆·杜塔 20 约翰·M·达顿 20

EBCDIC 集合 87, 464 经济分析 42, 415 均衡模型 419, 422 退出期权 252 二选一约束 59 埃姆林, 格雷斯·R. xxi 空键说明 182 集合, 索引 82, 146 字符串 205, 234, 252, 272 表条目 173, 187 空集 $\{ \}$ 74, 146, 162, 262, 264, 341, 456- 457, 462 终端效应 67, 379 end 命令 204, 496 environ 命令 490 环境选项 490 变量 205, 481 变量, 表格 484 环境, 初始 316, 490 环境, 命名 316- 317 等式约束 21, 30, 130 均衡应力模型 388 埃里克森, 沃伦 39 错误消息 12, 83, 85, 94, 116, 119, 146, 160, 207, 211, 224, 252, 338 \ 转义序列, printf 482 求值顺序 100, 111, 132, 467, 473 Excel 电子表格 174, 176, 196 Excel 文件扩展名, .xls 199- 200 存在算符 115, 456- 457 exit 命令 496 exp 函数 113, 458 expand 命令 13, 247- 248, 492 expand_precision 选项 484 expand_round 484 指数函数 459 指数运算 \*, \*\* 111, 456 表达式 算术 61, 76, 111- 114, 456 可赋值 185 常量 455, 457, 461 索引 9, 79- 82, 99- 100, 110, 138, 455 线性 132- 134, 139, 383 逻辑 80, 99, 114- 116, 455, 457 元字符, 正则 460 非线性 114, 133, 400- 403 正则 273, 460 集合 455, 457 简化 46, 130, 134, 139 字符串 194, 197, 273, 459 表达式  $\epsilon < \ldots > >$  分段线性 367, 369, 376, 380- 382, 460

法比安, 蒂博尔 20 阶乘 126 可行性容差 281 可行解 4, 35 费里斯, 迈克尔·C. 433 斐波那契数 126 文件包含 参见 模型, 数据, 包含命令 文件输出 参见 输出重定向 文件, 临时 485 文件 blend.mod 38 cut2.mod 313 cut.dat 309 cut.mod 306 diet2.dat 34 diet.dat 33 diet.mod 32 dietu.dat 82 dietu.mod 78 econ2.mod 424 econ.dat 421 econmin.mod 420 econ.mod 423 econnl.mod 425 fence.mod 411 iocol1.mod 356 iocol2.mod 358 iorow.mod 355 minmax.mod 381 multic.mod 103 multi.dat 58 multi.mod 57 multmip1.dat 443 multmip1.mod 442 multmip3.dat 447 multmip3.mod 446 netl.mod, netl.dat 322 netlnode.mod 335 net2.dat 325 net2.mod 324 net3.dat 327 net3.mod 326 net3node.mod 336 netasgn.mod 333 netfeeds.mod 344 netmax3.mod 339 netmax.mod 329 netmcol.mod 363 netmulti.mod 342 netshort.mod 331 netthru.mod 346 nltrans.mod, nltrans.dat 404 prod.dat 10 prod.mod 8 sched.dat 361 sched.mod 361 steel2.dat 14 steel3.mod, steel3.dat 15 steel4.dat 17 steel4.mod 16 steel1.mod, steel.dat 11 steelP.dat 65 steelpl1.dat 374 steelpl1.mod 373 steelP.mod 64 steelT0.mod 59 steelT2.dat 85 steelT2.mod 84 steelT3.dat 102 steelT3.mod 101 steelT.mod, steelT.dat 62 transp2.mod 93 transp3.dat 95 transp3.mod 95 transp.dat 48 transpl1.mod 368 transpl2.dat 371 transpl2.mod 370 transp.mod 47 文件名语法 11, 144, 165, 204, 479 文件名 - 标准输入 165, 283, 485, 495 扩展名, .bit 二进制 201, 477 扩展名, .mdb Access 199- 200 扩展名, .tab ASCII 201, 477 扩展名, .xls Excel 199- 200 文件, 辅助文件表 488 第一个函数 84, 463 费舍尔，马歇尔·L. 317 fix 命令 215, 489 固定成本 440- 444 变量 131, 241, 402 弗莱彻，罗杰 410 浮点表示法 76, 87, 144, 233, 235, 263, 271, 279, 453- 454, 483 向下取整函数 212, 457- 458 流 参见 网络流 for 语句 258- 262, 493 forall 运算符 115, 456- 457 福特，莱斯特·R. 小 347 格式化选项 显示 227- 230 显示表格 227 弗尔，罗伯特 xxi, 20, 318, 384 from 属性 334, 336, 338, 341, 353, 361, 363, 467 福尔克森，D·R 347 函数 abs 113, 140, 380, 458 acos 113, 458 acosh 113, 458 别名 458 元数 462, 464 asin 113, 458 asinh 113, 458 atan, atan2 113, 458 atanh 113, 458 Beta 459 基数 79, 98, 462, 464 柯西分布 459 向上取整 212, 237, 458 char 459 cos 113, 458 ctime 458 exp 113, 458 指数分布 459 first 84, 463 floor 212, 457- 458 Gamma 459 gsub 273, 460 ichar 459 indexarity 462, 464 Irand224 459 last 84, 463 length 271, 459 log, log10 113, 458 match 271, 285, 460 max 113, 380, 458 member 463 min 113, 140, 458

next 84- 85,463 nextw 84- 85,463 Normal,Normal01 209,459 num, num0 460 ord,ord0 85,463 Poisson 459 precision 458 prev,prevw 83- 84,463 round 212,458 sin 113,458 sinh 113,458 sprintf 271,459,483 sqrt 113,458 sub 273,460 substr 271,459 tan 113,458 tanh 113,458 time 458 trunc 212,458 Uniform,Uniform01 459 函数声明 402,497 函数 凹函数 383,393 凸函数 383,393,396 导入函数 497- 499 集合函数 83- 85 集合函数表格 464 分段线性函数 365,379- 384,393,395 可分函数 381,383 光滑函数 391,400- 402 算术函数表格 113,458 随机数函数表格 459 正则表达式函数表格 459 舍入函数表格 212 字符串函数表格 459

Gamma 函数 459 加里 (Garry), 苏珊·加纳 (Susan Gainer) 39 加斯 (Gass), 索尔·I (Saul I.) 39 加斯曼 (Gassman), H·I xxi 盖 (Gai), 大卫·M (David M.) 318 泛化名称 249,291,429,492 与互补性 431 泛化名称表格 493 gentimes 选项 250 吉尔 (Gill), 菲利普·E (Philip E.) 410 Gilmore- Gomory 算法 305- 306 全局最优解 408 Glover,Fred 347 图 二分图 330 有向网络 320 传递闭包 462

格罗斯 (Gross), 埃里克·H (Eric H.) xxi gsub 函数 273,460 gutter_width 选项 230

黑斯勒 (Hessler), 罗伯特·W (Robert W.) 318 哈格 (Hogg), W·W 318 处理程序 请参见 表格处理程序_handler_lib 477 _HANDLERS 477 赫恩 (Hearn), D·W 318 希拉勒 (Hilal), 赛义德·S (Said S.) 39

- i 命令行参数 500 ichar 函数 459 IEEE 算术 (参见 浮点数表示法) if 索引表达式 138, 338, 341, 362, 460-461, 467 if-then-else 嵌套 266 运算符 115, 117, 133, 264, 457 语句 264-266, 493-494 i_is 后缀 299 iisfind, iis_table 选项 299 导入函数 497-499 IN, INOUT 后缀属性 472 in 运算符 78, 98, 114, 455-456, 465 IN 读/写状态 174, 177, 180, 182, 186, 478 include (参见 also model, data) include 命令 255-256, 479 搜索路径 479 索引 (参见 dummy index) indexarity 函数 462, 464 索引集合, 数据 101, 161 集合的数据列 196 集合的集合 100-104, 161-162 表的集合 193 目标 134 索引表达式 9, 79-82, 99-100, 110, 138, 455 表达式, 显示 224-227 表达式, 作用域 80, 94-95, 455-456, 482 在空集上 82, 146 索引表达式, if 138, 338, 341, 362, 460-461, 467 不可行解 (在预处理中 279) 不可行解 34, 299, 374 无限循环 262, 267 集合 86, 463 无穷大 242, 276, 338, 456, 466 有序属性的继承 86, 463 初始环境 316, 490 问题 311, 315, 489 对偶变量的初始值 185, 468, 472, 477, 486 变量的初始值 131, 162, 398, 405, 408-409, 466, 471, 477, 486 初始值 $\coloneqq$ 与 $=$ 211 默认值 与 $=$ 119 初始化, 后缀 466, 469-470, 489 INOUT 读/写状态 186, 193, 195 输入重定向, $<$ 163, 165, 481, 485 输入-输出模型 37, 42, 354-358 插入提示选项 268, 479 整数规划, 松弛 309, 448 编程 xvi, 36, 437-438, 448-449 解, 四舍五入 35, 309, 360, 439, 448 解 (在饮食模型中 36, 439; 在网络模型中 344; 在调度模型中 360; 在运输模型中 51, 439-448) 整数属性 36, 116, 131, 438, 441, 465-466 集合 86, 463-464 integer_markers 选项 486 整数集 87, 464 inter 运算符 76, 98, 103, 456-457 内点算法 287 交集, 集合 (参见 inter) 区间集 86, 463-464 Irand224 函数 459 不可约不可行子集 299 迭代运算符 (参见 运算符) Jacobs, Walter 347, 351 Johnson, Ellis L. 449 Kahan, Gerald 363 Kelly, Paul xxi Kendrick, David A. 20 Kernighan, Mark D. xxi 关键字 (参见 保留字) Klingman, Darwin 347 背包问题 23, 306, 451 Kontogiorgis, Spyros 384 Kostreva, Michael M. 449 Kuhn, Harold W. xxi Kuip, C. A. C. xxi

拉格朗日乘数 (Lagrange Multiplier) 参见 对偶变量 (Dual Variable) 大规模优化 (Large-Scale Optimization) xvi 莱昂·拉斯登 (Leon Lasdon) 318, 414 最后函数 (Final Function) 84, 463 .1b 下界 (Lower Bound) 35, 241-242, 279, 470, 486 .1b0, .1b1, .1b2 下界 (Lower Bound) 279 最小二乘 (Least Squares) 388, 412 长度函数 (Length Function) 271, 459 扬·卡雷尔·伦斯特拉 (Jan Karel Lenstra) xxi 小于运算符 (Less Than Operator) 111, 456 let 命令 (let Command) 36, 210-212, 405, 491 库 (Library) 参见 导入函数 (Import Function) 朱迪思·利布曼 (Judith Liebman) 414 行继续符 (Line Continuation Character) \ 205, 453 线性表达式 (Linear Expression) 132-134, 139, 383 程序 (Program), 网络 (Network) 319, 343, 353 规划 (Programming) xvi, 1-5, 129 成本 (Cost), 产出的线性性 (Linearity of Output) 38, 393 linelim 选项 (linelim Option) 467 链接线性规划 (Linked Linear Programs) 55, 60 列表数据 (List Data) 多维 (Multidimensional) 148-151 一维 (One-Dimensional) 145-146 二维 (Two-Dimensional) 146-148 字面数字 (Literal Number) 75, 111, 144, 453-454 集合 (Set) 74, 454 字符串 (String) 74, 144, 453-454, 473 加载命令 (Load Command) 402, 477, 498 局部最优 (Local Optimum) 407-408 LOCAL 后缀属性 (LOCAL Suffix Attribute) 472 对数 (Logarithm), log10 函数 (log10 Function) 113, 458 log_file 选项 (log_file Option) 251, 496 log_model, log_data 选项 (log_model, log_data Options) 252, 496 逻辑运算 (Logical Operations), 集合运算符 (Set Operators), 表格 (Table) 456 转换 (Transformation), 算术到逻辑 (Arithmetic to Logical) 123, 457 表达式 (Expression) 80, 99, 114-116, 455, 457 运算符 (Operator) 114, 457 参数 (Parameter) 122-123 循环 (Loop) 无限 (Infinite) 262, 267 名称 (Name) 268, 494 嵌套 (Nesting) 261, 267 网络流中的损失 (Loss in Network Flow) 326, 336 托德·洛 (Todd Lowe) xxi

下界 (Lower Bound) .lb 35, 241-242, 279, 470, 486 .lb0, .lb1, .lb2 279

下界 .lb 35, 241- 242, 279, 470, 486 .lb0, .lb1, .lb2 279 托马斯·L·马格南蒂 347 边际值 参见 对偶值 罗伊·E·马斯顿 384 匹配函数 271, 285, 460 数学规划 xv- xvi 矩阵 参见 二维集合、参数 矩阵生成器 xvii, xxi 最大函数 113, 380, 458 运算符 114, 456 最大化 参见 目标声明 最大流模型 328- 329, 337- 339, 412 最小-最大目标 382 . mdb Access 文件扩展名 199- 200 亚历山大·米劳斯 xxi, 20 成员、虚拟 参见 虚拟索引 成员函数 463 成员资格测试 参见 in, within 集合成员资格运算符 78- 79 内存使用 88, 97, 448 消息 参见 错误消息 元字符、正则表达式 460 最小函数 113, 140, 458 运算符 114, 456 最小化 参见 目标声明 最小-最大目标 382 MINOS 求解器 6, 208, 281, 398 混合整数规划 441 . mod 文件 参见 文件 模运算符 111, 456 模型文件 参见 文件 模型活动 43 广告 21, 450 聚合 55, 88 分配 49- 51, 53, 135, 330- 333, 348 混合 37- 39 信用评分 386 饮食 27- 37, 39, 77- 78, 135 经济均衡 419, 422 均衡应力 388 投入产出 37, 42, 354- 358 最大流 328- 329, 337- 339, 412 模式 143, 206, 256, 479, 496 多商品运输 56- 59, 450 多期规划 67- 69, 452 多期生产 59- 63 网络流 319, 333- 339, 416 非线性运输 403- 410 石油炼制 24- 25 分段线性生产 370- 377 分段线性运输 366- 369 生产 6- 12, 15, 420 生产和运输 63- 67 调度 37, 359- 362, 451 最短路径 329- 330 钢铁生产 2- 7, 10- 19 应力分析 388 运输 44- 49, 52, 330- 333 转运 319- 327, 334- 337, 347 模型命令 11, 206, 214, 256, 479 模型-数据分离 7, 11, 74, 143 建模语言 xvi- xviii MPEC 系统 426 MPS 文件 486 多商品运输模型 56- 59, 450 多维列表数据 148- 151 表格数据 156- 160 多期规划模型 67- 69, 452 生产模型 59- 63 多目标 134- 137, 206, 240 解 6, 49, 137, 407 沃尔特·默里 410 卡塔·G·穆尔蒂 347 名称循环 268, 494 语法 453 命名环境 316- 317 问题 309- 316 名称 通用 249, 291, 429, 492 通用名称表 493 纳什均衡 426, 433 乔治·L·内姆豪泽 450 NEOS xx 嵌套循环 261, 267 嵌套 if-then-else 266 净流入、净流出 334, 340, 469 网络流、损耗 326, 336 流模型 319, 333- 339, 416 图、有向 320 线性规划 319, 343, 353 模型、整数解 344

换行 144, 453, 473, 498 \n printf 中的换行 239, 261, 482 下一命令 269, 495 函数 84- 85, 463 nextw 函数 84- 85, 463 nl_permute 选项 487 豪尔赫·诺塞达尔 410 节点声明 334- 340, 353, 468 不存在的数据值 参见 default 符号 非线性约束 59, 140, 492 成本函数 394 表达式 114, 133, 400- 403 编程 xvi, 391- 397, 403- 410 运输模型 403- 410 变量 397- 400 混合中的非线性 38 物理模型 397 非负变量 3, 10, 45, 130, 377 Normal, Normal01 函数 209, 459 not in 运算符 79, 456 运算符 114, 456 within 运算符 79, 456 空命令 494 num, num0 函数 460 数字表示 参见浮点数表示 数字字面量 75, 111, 144, 453- 454 转换为字符串 271, 459 数字集合 75- 76 数值到逻辑转换 457 数值选项, 显示 232- 238

- o 命令行参数 485 obj 属性 334, 339, 341, 356, 363, 466 目标 3, 9 声明 134-137, 470 函数 xv, 31, 240, 341 max-min, min-max 382 目标函数, to_come 362, 470 objective 命令 135-136, 206, 489 objective_precision 选项 484-485 索引目标 134 多目标 134-137, 206, 240 ODBC 表处理器 169, 174, 194, 198 炼油厂模型 24-25 omit_zero_cols 选项 66, 232 omit_zero_rows 选项 50, 66, 231

省略的数据值 参见 default 符号 一维列表数据 145- 146 参数 110 集合 110, 454 集合数据 473 理查德·P·奥尼尔 411 运算符优先级 9, 77, 111- 114, 116, 133, 456, 459 同义词 454 运算符列名 177, 184- 185, 188, 190, 478 & 字符串连接 270, 459 and 114, 456 补集 469 交叉 98, 456 差集 76, 98, 456- 457 除法 111, 456 存在 115, 456- 457 全称 115, 456- 457 if- then- else 115, 117, 133, 264, 457 in 78, 98, 114, 455- 457, 465 交集 76, 98, 103, 456- 457 less 111, 456 最大值 114, 456 最小值 114, 456 模运算 111, 456 非 114, 456 not in 79, 456 not within 79, 456 或 114, 456 乘积 74, 114, 456 setof 98, 272, 457 求和 74, 113, 456 对称差 76, 98, 456- 457 并集 76, 98, 103, 456- 457 within 78, 93, 97- 98, 114, 456- 457, 462 运算符 算术 456 算术归约 456 逻辑 114, 457 关系 114, 465 集合 76- 78 集合成员 78- 79 算术运算符表 112 算术、逻辑、集合运算符表 456 元组 98- 100 最优基 参见基本解 全局最优 408 局部最优 407- 408 选项

ampl_include 479 AMPLFUNC 498, 500 astatus_table 294 bad_subscripts 253 cmdprompt1, cmdprompt2 494 compl warp 487 dataprompt1, dataprompt2 494 display_col 63, 228, 231 display_eps 235 display_precision 233, 263, 406, 483 display_round 234- 235, 483 display_transpose 229 display_width 204, 229 dual_initial_guesses 486 eexit 252 expand_precision 484 gentimes 250 gutter_width 230 iisfind_iis_table 299 insertprompt 268, 479 integer_markers 486 linelim 467 log_file 251, 496 log_model, log_data 252, 496 nl_permute 487 objective_precision 484- 485 omit_zero_cols 66, 232 omit_zero_rows 50, 66, 231 pl_linearize 383 presolve 139, 278, 290 presolve_eps, presolve_epsmax 281 presolve_fixeps, presolve_fixepsmax 282 presolve_inteps, presolve_intepsmax 279 presolve_warnings 252, 282 print_precision 239, 482- 483 print_round 239, 482- 483 print_separator 238, 482 prompt1, prompt2 204, 494 randseed 122, 458 relax_integrality 215, 217, 309, 486 reset_initial_guesses 291, 297, 486 send_statuses 288 send_suffixes 486 SHELL 495 shell_env_file 495 show_stats 250, 275, 282 single_step 268

solution_precision 236, 238, 264, 484, 486 solution_round 236, 238, 484, 486 solve_exitcode, solve_exitcode_max 285 solve_message 285 solve_result, solve_result_num 283- 284 solve_result_table 283 solver 37, 207 solver_msg 260 sstatus_table 289 substout 399, 414, 467- 468 times 250 TMPDIR 285, 485 var_bounds 279, 486 version 500 option command 204, 481 * in 205, 481 option- name pattern 481 options 204- 205, 481 environment 490 options display formatting 227- 230 display numeric 232- 238 table of display formatting 227 table of display numeric 232 options, reset 205, 490 OPTIONS_IN, OPTIONS_OUT, OPTIONS_INOUT 490, 499 $option value 460, 481 $\$option default value 481 or operator 114, 456 ord, ord0 function 85, 463 order of evaluation 100, 111, 132, 467, 473 ordered sets 82- 87, 98, 223, 261, 463- 464 orderedattribute 83, 98, 146, 463 attribute, inheritance of 86, 463 by attribute 86, 463 Orlin, James B. 347 OUT read/write status 177, 186, 190, 192, 478 suffix attribute 472 output redirection, >, >> 220, 239, 251, 273, 481 pairs sets of ordered 91- 92 slices of 93- 96 subsets of 93- 96 Pang, Jong- Shi 432- 433 parameter

并设置数据表 476 数据 477 数据声明 475 数据表 154 数据模板 147- 148, 150, 475 声明 8, 110- 111, 465- 466 定义, 递归 120, 122, 465 一维 110 二维 110 参数 计算 118- 121 显示 220- 224 逻辑 122- 123 随机生成 121- 122, 209 限制 116- 118 符号 123- 124, 323, 328 内置时间表 496 帕达洛斯, P. M. 318 PATH 求解器 422 模式, 选项名 481 惩罚函数 369- 377, 385, 396 分段线性 372, 375- 376 菲利普斯, 南希 347 物理模型, 非线性 397 pid 进程 ID 495 分段线性函数 在零点 367, 376 函数 365, 379- 384, 393, 395 惩罚函数 372, 375- 376 生产模型 370- 377 运输模型 366- 369 分段线性表达式 $\leq \leq \ldots > > 367$ 369, 376, 380- 382, 460 派克, 拉尔夫 411 管道属性 498 pl_ linearize 选项 383 规划模型, 多周期 67- 69, 452 泊松函数 459 预状态 276 优先级, 运算符 9, 77, 111- 114, 116, 133, 456, 459 精度函数 458 预定义词 74, 112, 454 重定义 213, 454 预求解 60, 65, 207, 241, 275- 282, 486 和互补性 429 不可行性 279 presolve 选项 139, 278, 290 presolve_eps, presolve_epsmax 选项 281 presolve_fixeps, presolve_fixepsmax 选项 282 presolve_inteps, presolve_intepsmax 选项 279

presolve_warnings 选项 252, 282 prev, prevw 函数 83- 84, 463 价格, 影子 见对偶值 原始变量 243, 398, 485 打印命令 238- 239, 482- 484 print_precision 选项 239, 482- 483 print_round 选项 239, 482- 483 print_separator 选项 238, 482 printf 命令 124, 239- 240, 261, 482- 484 转换字符, $\frac{9}{10}$ 239, 482 转换字符表 483 \ 转义序列 482 字符串引号, $\frac{9}{10}$ 482 483 .priority 后缀 296 问题 见模型 问题 caterer 347, 350 互补性 420- 423 切割 stock 41, 305, 364, 452 声明 489 背包 23, 306, 451 集合覆盖 362, 364 问题, 初始 311, 315, 489 问题命令 301, 307, 311, 489 问题, 命名 309- 316 prod 运算符 74, 114, 456 生产和运输模型 63- 67 模型 6- 12, 15, 420 模型代数形式 7 模型, 多周期 59- 63 模型, 分段线性 370- 377 编程 整数 xvi, 36, 437- 438, 448- 449 线性 xvi, 1- 5, 129 数学 xv- xvi 混合整数 441 非线性 xvi, 391- 397, 403- 410 随机 69- 71 提示 ampl: 5, 165, 204, 485 ampl? 165, 204, 485 ampl 数据 144 prompt1, prompt2 选项 204, 494 清除命令 213, 488

$\frac{9}{10}$ , $\frac{9}{10}$ printf 字符串引号 483 限定名 见后缀 查询, 结构化查询语言 184, 199 退出命令 5, 204, 214, 496 引号 74, 144, 165, 204- 205, 453, 473

- R 命令行参数 500 随机数函数，459 生成器种子表 122, 210, 458 随机属性 497 随机生成参数 121-122, 209 randseed 选项 122, 458 .rc 参见 reduced cost read 命令 163, 484 表命令 175, 177, 180, 477 读取文件，参见 model, data reading scalar from table 182 读/写状态 -> 189, 193, 478 <- 175, 177, 182, 193 <-> 193, 478 IN 174, 177, 180, 182, 186, 478 INOUT 186, 193, 195 OUT 177, 186, 190, 192, 478 实数集 87, 464 递归参数定义 120, 122, 465 集合定义 462 重新声明检查命令 488 命令 213, 488 预定义词的重新定义 213, 454 重定向 $\gimel$ ,>> 输出 220, 239, 251, 273, 481 $\epsilon$ 输入 163, 165, 481, 485 约简成本 17, 22, 40, 67, 243-245 约简运算符，算术 456 冗余约束 276-278 正则表达式 273, 460 函数，459 表 元字符 460 关系运算符 114, 465 表 170-171, 173, 178 .relax 后缀 486 relax_integrality 选项 215, 217, 309, 486 整数规划松弛 309, 448 重新加载命令 498 删除命令 482 重复语句 262-264, 493 浮点表示 76, 87, 144, 233, 235, 263, 271, 279, 453-454, 483 保留词同义词 454 词 74, 112 词表 454 重置 命令 17, 214, 481, 490 数据命令 122, 209- 210, 490 选项命令 205, 490 问题命令 491 reset_initial_guesses 选项 291, 297, 486 恢复命令 214, 489 参数限制 116- 118 .result 后缀 287 反向属性 86, 463 可逆活动 377- 379 Richton, Robert E. xxi Rinnooy Kan, Alexander H. G. xxi 卷材修边问题 参见 cutting- stock problem 四舍五入函数 212, 458 四舍五入 76, 87, 233, 237, 263, 271, 279, 454, 457, 483 函数表 212 至整数解 35, 309, 360, 439, 448

- s 命令行参数 458 Saunders, Michael A. xxi 表中读取标量 182 沿弧缩放 参见网络流中的损失 变量缩放 410 调度 航空公司机组 362 模型 37, 359-362, 451 模型，其中的整数解 360 Schrage, Linus 414 Schrijver, Alexander xxi, 450 范围 参见 also dummy index 虚拟索引的范围 47, 321 索引表达式 80, 94-95, 455-456, 482 脚本 18, 179, 195, 255, 304, 485, 495 脚本 cut.run 308 diet.run 284 dietu.run 256 steelT.sa1 258 steelT.sa2 259 steelT.sa3 260 steelT.sa4 263 steelT.sa5 266 steelT.sa7 267 steelT.tab1 261 搜索路径，包含 479 种子，随机数生成器 122, 210, 458 Seip, Robert H. xxi 自引用 117

分号语句终止符 9, 144, 204, 256, 453, 473, 494  
send_statuses 选项 288  
send_suffixes 选项 486  
敏感性分析 194, 256, 262, 304  
可分离函数 381, 383  
数据中的分隔符，逗号 144  
集合差集 参见 diff, symdiff  
集合交集 参见 interset  
集合 参见索引集合  
集合与参数数据 151-154  
集合与参数数据表 476  
算术、逻辑运算符，表格 456  
集合覆盖问题 362, 364  
数据 473-475  
一维数据 473  
数据语句 474  
数据表 474  
数据模板 147, 150, 474  
二维数据 475  
声明 8, 74, 461-465  
递归定义 462  
表达式 455, 457  
字面量 74, 454  
成员规则 454  
成员运算符 78-79  
数字集合 75-76  
一维集合 110, 454  
运算符 76-78  
二维集合 110, 455  
ASCII 集合 87, 464  
数据表中的 +, - 156-157, 475  
显示集 87, 464  
EBCDIC 集合 87, 464  
整数集合 86, 463-464  
整数集 87, 464  
区间集合 86, 463-464  
实数集 87, 464  
setof 运算符 98, 272, 457  
集合显示 220  
集合函数 83-85  
索引集合 100-104, 161-162  
无限集合 86, 463  
有序对集合 91-92  
元组集合 96-98  
有序集合 82-87, 98, 223, 261, 463-464  
无序集合 73-74  
影子价格 参见对偶值  
shell 命令 214, 495  
SHELL 选项 495  
shell 脚本 500  
shell_env_file 选项 495  
谢泼德, J. R. xxi  
最短路径模型 329-330  
show 命令 246, 491  
show_stats 选项 250, 275, 282  
侧约束 342, 345, 363  
变量 343  
有效数字 233  
单纯形算法 xvi, xxi, 279, 287, 344, 486  
表达式简化 参见 also presolve  
表达式简化 46, 130, 134, 139  
sin 函数 113, 458  
single_step 选项 268  
单步模式 495  
奇异点 406-407  
sinh 函数 113, 458  
辛哈, 拉克什曼 P. xxi  
skip 命令 270, 495  
松弛后缀 81, 241-242, 244, 472  
切片 58, 91, 95, 97, 147-149, 455  
有序对的切片 93-96  
斜率 参见分段线性表达式  
光滑函数 391, 400-402  
SNOPT 求解器 48, 208  
社会核算矩阵 415  
软约束 参见惩罚函数  
solexpand 命令 248, 492  
解，基本解 287, 289-290, 292, 298, 473  
solution 命令 487  
solution_precision 选项 236, 238, 264, 484, 486  
solution_round 选项 236, 238, 484, 486  
解  
可行解 4, 35  
不可行解 34, 299, 374  
多重解 6, 49, 137, 407  
solve 命令 206-207, 485-486  
problem 命令 315  
solve_exitcode, solve_exitcode_max 选项 285  
solve_message 选项 285  
solve_result, solve_result_num 选项 283-284  
solve_result_table 选项 283  
_solve_time 参数 251, 496  
求解器  
CPLEX 36, 50, 208, 281  
MINOS 6, 208, 281, 398  
PATH 422

SNOPT 48, 208 状态后缀 286, 288 求解器状态 .status 473 求解器选项 37, 207 solver_msg 选项 260 求解器定义的后缀 298-304 空间需求 .参见内存使用 稀疏集 111 子集 94 电子表格 238 Excel 174, 176, 196 sprintf 函数 271, 459, 483 SQL 查询 184, 199 SQL SELECT 语句 199 sqrt 函数 113, 458 平方系统 422, 425, 427, 487 .sstatus 后缀 290, 292, 295, 473 sstatus_table 选项 289 s.t. .参见约束声明 标准输入文件名 - 165, 283, 485, 495 初始猜测值 .参见变量的初始值 语句 .另请参见命令、声明 语句 参数数据 475 集合数据 474 终止符，分号 9, 144, 204, 256, 453, 473, 494 语句 break 266, 493 check 46, 56, 117, 465 命令 256, 479 continue 266, 493 for 258-262, 493 if-then-else 264-266, 493-494 repeat 262-264, 493 语句 控制流 492 .sstatus 求解器状态 473 后缀 295 钢铁生产模型 2-7, 10-19 step 命令 268, 495 步进执行命令 268-270, 495 随机规划 69-71 理查德·E·斯通 432 应力分析模型 388 字符串 从数字转换为字符串 271, 459 表达式 194, 197, 273, 459 函数表 459 字面量 74, 144, 453-454, 473 字符串连接运算符 & 270, 459 引号 %q, %Q 格式化打印 483

sub 函数 273, 460 subject to .参见约束声明 下标 8, 16, 110, 455 子集测试 .参见 in, within 成对子集 93-96 变量替换 399-400, 414 substout 选项 399, 414, 467-468 substr 函数 271, 459 such-that 条件 .参见逻辑条件 后缀声明 471 后缀 .astatus 294-295 属性 IN, INOUT, OUT, LOCAL 304, 472 .direction 296 .iis 299 .priority 296 .relax 486 .result 287 .slack 81, 241-242, 244, 472 .sstatus 290, 292, 295, 473 .status 295 后缀属性 466, 469-470, 489 命令 296, 299, 302 初始化 466, 469-470, 489 后缀 互补约束 428, 431, 471 求解器状态 286, 288 求解器定义 298-304 表 472 用户定义 296-297, 302 乌韦·H·苏尔 449 平方和 388, 412 求和运算符 74, 113, 456 符号参数 123-124, 323, 328 符号属性 123, 465, 476, 497 对称差 .参见 syndiff 同义词表 454 语法错误 .参见错误信息

.tabASCH 文件扩展名 201, 477 表 声明 174, 178, 180, 186, 477 条目 空 173, 187 处理程序 169, 197-202, 477 内置处理程序 174, 201 ODBC 处理程序 169, 174, 194, 198 算术函数表 113, 458 算术、逻辑、集合运算符表 456 算术运算符表 112

辅助文件 488  
内置时序参数 496  
命令 480  
环境变量 484  
函数集合 464  
通用名称 493  
随机数函数 459  
正则表达式函数 459  
保留字 454  
舍入函数 212  
字符串函数 459  
后缀 472  
同义词 454  
参数数据 154  
从标量读取 182  
关系运算 170-171, 173, 178  
集合数据 474  
显示格式选项表 227  
显示数值选项表 232  
printf 转换字符表 483  
转置 (tr) 32, 155, 157, 228-229, 475-476  
表格，索引集合 193  
表格数据 多维 156-160  
二维 154-156  
tan 函数 113, 458  
tanh 函数 113, 458  
迈克尔·T·塔亚布汗 20  
模板 参数数据 147-148, 150, 475  
集合数据 147, 150, 474  
临时文件 485  
time 函数 458  
times 选项 250  
时序参数，内置表 496  
TMPDIR 选项 285, 485  
to 属性 334, 336, 338, 341, 353, 361, 363, 467  
to_come 约束声明 356, 362, 468  
目标函数 362, 470  
容差，可行性 281  
(tr)，表格转置 32, 155, 157, 228-229, 475-476  
交通流 参见最大流模型  
图的传递闭包 462  
运输模型 44-49, 52, 330-333  
整数解 51, 439-448  
多商品 56-59, 450  
非线性 403-410  
分段线性 366-369  
生产与运输 63-67  
转置 参见表格转置  
转运模型 319-327, 334-337, 347  
trunc 函数 212, 458  
元组运算符 98-100  
元组 73, 91, 96, 146  
元组集合 96-98  
二维列表数据 146-148  
参数 110  
集合 110, 455  
集合数据 475  
表格数据 154-156  

.ub 上界 35, 240-242, 279, 470, 486  
.ub0, .ub1, .ub2 上界 279  
unfix 命令 215, 489  
未格式化数据 163  
Unicode 453, 459  
Uniform, Uniform01 函数 459  
并集运算符 76, 98, 103, 456-457  
unload 命令 498  
无序集合 73-74  
未指定数据值 参见默认符号  
until 条件 262, 267, 494  
update data 命令 209, 491  
用户定义后缀 296-297, 302  

- v 命令行参数 500  
有效列名 171-172, 177, 185  
数据验证 12, 46, 93, 118  
克里斯·J·范·怀克 xxi  
罗伯特·J·范德贝 20  
var_bounds 选项 279, 486  
变量数据 162, 398, 477  
声明 5, 8, 466-467  
变量 xv, 2, 129-132  
变量界限 10, 14, 17, 32, 35, 130, 139, 241-242, 276, 279, 466, 470-471, 486  
定义 466-468  
对偶变量 162, 243, 468, 472, 485  
环境变量 205, 481  
固定变量 131, 241, 402  
初始值 131, 162, 398, 405, 408-409, 466, 471, 477, 486  
对偶变量初始值 185, 468, 472, 477, 486  
非线性变量 397-400  
非负变量 3, 10, 45, 130, 377  
原始变量 243, 398, 485  
变量缩放 410  
变量替换 399-400, 414  
0-1 变量 439-448  

向量 参见一维集合、参数  
verbose 属性 200  
version 选项 500  
查看数据 参见 display 命令  
朱莉安娜·维尼亚利 xxi  
通·武哈克 xxi

沃伦，阿兰 414 while 条件 262, 267, 494 空格 144, 453, 473 通配符 \* 205, 481 威尔金斯，伯特 411 威廉姆斯，H·保罗 67 威利尔德，马克 411 within 运算符 78, 93, 97- 98, 114, 456- 457, 462 沃尔西，劳伦斯·A. 450 词语 预定义 74, 112, 454

保留 74, 112 赖特，玛格丽特·H. xxi, 410 斯蒂芬·J. 410 write 命令 487 table 命令 178, 186, 477 www.ampl.com xviii, xx, 19, 453

.xls Excel 文件扩展名 199- 200 xref 命令 213, 247, 492

零 分段线性函数 367, 376 抑制 231- 232 零一规划 参见 整数规划 零一变量 439- 448 零或最小限制 444