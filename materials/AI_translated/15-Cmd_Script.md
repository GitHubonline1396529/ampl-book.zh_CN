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
