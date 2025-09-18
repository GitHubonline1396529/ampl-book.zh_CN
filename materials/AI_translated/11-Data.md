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