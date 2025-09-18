

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
