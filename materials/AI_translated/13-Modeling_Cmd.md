
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
