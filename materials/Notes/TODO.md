# 翻译项目有待处理的事项清单和笔记

## 关于这份文件

在实际的翻译过程中由于遇到的问题比较冗杂，以及后面可能还涉及到多人协作翻译，我认为有必要维护这份 Markdown 文件来记录一些翻译过程中遇到的问题，以便我自己，还有后续继而参与翻译的人能知道我们要做什么。其中的一些可能后续会在 Issue 中讨论。

除此之外，我也创建了一个 `materials/Note` 文件夹，记录所有翻译过程中需要记录的笔记。

## 当前翻译工作重点

### 整改 AI 机翻译文的格式

我认为比起提供一份完善的译本，现在更重要的事情还是首选提供一个“可读的”译本，同时这也有利于以降低后续的翻译难度，因此我们需要对现有的 AI 翻译文本进行一个简单的整改。

一方面，我们需要调整译文文本的 Markdown 格式。AI 翻译的译文文本里面存在很多的 Markdown 格式问题，例如将行间公式对象误解为公式对象，还有些会造成 Markdown 渲染器的解析错误，导致文本不可读。

另一方面，还有一些翻译存在明显的内容错误。例如下面的这段原文：

> The construct `{j in P}` is called an *indexing expression*. As you can see from ourexample, indexing expressions are used not only in  declaring parameters and variables, but in any context where the algebraic model does something “for each $j$ in $P$”. Thus  the `Limit` constraints are declared
> 
> ```ampl
> subject to Limit {j in P}
> ```
>
> because we want to impose a different restriction `0 <= X[j] <= u[j]` for each different product $j$ in the set $P$. In the same way, the summation in the objective is written
>
> ```ampl
> sum{j in P} c[j] * X[j]
> ```
>
> to indicate that the different terms `c[j] * X[j]`, for each `j` in the set `P`, are to be added together in computing the profit.

对此，AI 给出的直译文本的内容如下所示。可以看到，这里 AI 的翻译拧成了一个单一的段落，存在明显的格式理解上的问题：

> 构造式 `{j in P}` 被称为 *索引表达式 (indexing expression)*。正如示例所示，索引表达式不仅用于声明参数和变量，还用于任何代数模型中需要对“每个 $j \in P$”执行操作的地方。因此，声明限制约束是因为我们希望对集合 $P$ 中每个不同的产品 $j$ 施加不同的限制 $0 \leq \mathbb{X}[j] \leq \mathbb{U}[j]$。同样地，目标函数中的求和表达式是为了表明在计算利润时，需将集合 $P$ 中每个 $j$ 对应的不同项 $\mathsf{c}[\mathsf{j}] \star \mathsf{X}[\mathsf{j}]$ 相加。

显然，AI 无法理解 `subject to Limit {j in P}` 语句独立成段实际上是一个代码块元素，它把这行代码当作了前后文段的一部分，一同翻译进去了。还有 `0 <= X[j] <= u[j]`，AI 将其当作数学公式元素翻译成了 $0 \leq \mathbb{X}[j] \leq \mathbb{U}[j]$ 字样。

对此，我们简单整改后的文段大致应该长这样：

> 构造式 `{j in P}` 被称为 *索引表达式 (indexing expression)*。正如示例所示，索引表达式不仅用于声明参数和变量，还用于任何代数模型中需要对“每个 $j \in P$”执行操作的地方。因此，声明 `Limit` 约束
>
> ```ampl
> subject to Limit {j in P}
> ```
>
> 是因为我们希望对集合 $P$ 中每个不同的产品 $j$ 施加不同的限制 `0 <= X[j] <= u[j]`。同样地，目标函数中的求和表达式
>
> ```ampl
> sum {j in P} c[j] * X[j]
> ```
>
> 是为了表明在计算利润时，需将集合 $P$ 中每个 $j$ 对应的不同项 `c[j] * X[j]` 相加。