# 介绍

正如书题所言，我们所探讨的主题包含两个方面。其一是数学规划 (Mathematical Programming)，即在满足约束条件的前提下，对多变量函数进行优化；其二是 AMPL 建模语言，这是我们为了帮助人们利用计算机开发与应用数学规划模型而设计并实现的语言工具。

本书旨在作为数学规划与 AMPL 的入门介绍。对于已经熟悉数学规划的读者而言，本书也可作为 AMPL 软件的用户指南与参考手册。不过，我们并不假设读者具备相关的预备知识，同时也希望本书能够鼓励初学者学习并运用数学规划模型。

## 数学规划

“规划 (Programming)” 这一术语在 1940 年前后已被用于描述大型组织内部活动的计划或调度。“计划员” (Programmers) 们发现，他们可以将每项活动的数量或水平表示为一个变量 (Variable)，其数值有待确定。随后，他们能够用数学的方式将计划或调度问题中固有的约束条件 (Constraints) 描述为一组涉及这些变量的方程或不等式。满足所有这些约束条件的解，将被视为可接受的计划或调度方案。

实践很快表明：仅仅通过规定约束条件来刻画一个复杂的操作是十分困难的。若约束条件过少，则许多劣质的解也能满足它们；若约束条件过多，则会将理想的解排除在外，或者在最糟糕的情况下，根本不存在可行解。规划的成功最终依赖于一个得以为上述困境提供出路的关键的洞见。除了约束条件之外，还可以额外指定一个目标——即变量的某个函数 (如成本或利润)，以此判断一个解是否优于另一个解。这样一来，即使有许多不同的解能够满足约束条件，也无关紧要——只需找到一个能使目标函数最小化或最大化的解即可。“数学规划”一词由此被用来描述：在变量受到约束条件限制的情况下，最小化或最大化多变量的目标函数的过程。

经验很快表明，仅仅通过指定约束条件来建模复杂操作是很困难的。如果约束条件太少，许多劣质解可能满足它们；如果约束条件太多，则排除了理想的解，或者在最坏的情况下，可能没有可行解。编程的成功最终取决于一个关键的洞见，以提供一种绕过这一困难的方法。除了约束条件之外，还可以指定一个目标 (Objective)：一个变量的函数（如成本或利润），用于判断一个解是否优于另一个解。这样一来，许多不同的解满足约束条件就无关紧要了——只需找到一个使目标函数最小化或最大化的解即可。术语“数学规划”开始用来描述在变量约束条件下，对多变量目标函数的最小化或最大化。

在数学规划的逐步发展与应用过程中，有一类特殊情形备受瞩目：所有成本、需求及其他相关量均严格与各项活动的水平成正比，或是此类各项的和。用数学的术语来讲，这意味着目标函数是一个线性函数 (Linear Function)，而约束条件均是线性的方程与不等式。此类问题被称为线性问题 (Linear Program)，而建立此类问题并求解的过程称为线性规划 (Linear Programming)。线性规划具有特别重要的意义，因为多种多样的问题都可以建模为线性规划，而且即使变量与约束的数量多达数千个，仍能有快速且可靠的方法求解线性规划。同时，线性规划的思想对于分析和求解那些非线性的数学规划问题也很重要。

所有能够有效求解线性规划的方法都需要借助计算机。因此，线性规划的研究大多兴起于 20 世纪 40 年代末——当时已明确计算机可以被用于科学计算领域。首个成功的线性规划计算方法——单纯形法 (Simplex Method)——便是在这一时期提出的，并在随后十年间通过日益高效的实现方式不断发展完善。巧合的是，计算机的发展使“编程 (Programming)”一词如今拥有了更为人熟知的含义[^1]。

尽管线性规划具有广泛的应用性，但其线性假设有时仍过于理想化。如果在目标函数或约束条件中使用了变量的某些光滑的非线性函数 (Nonlinear Functions)，则该问题被称为非线性规划。此类问题的求解更为困难，但在实践中并非不可能。尽管对非线性函数最优值的研究已持续了两个多世纪，但多变量非线性规划的计算方法直到近几十年，在线性规划方法取得成功之后，才得以发展。因此，数学规划领域也被称为大规模优化 (Large Scale Optimization)，以区别于数学分析中的经典最优化课题。

若某些变量必须取整数值，线性规划的假设条件便会失效。此类问题被称为整数规划 (integer programming)，其求解难度通常显著增加。然而，随着计算机运算速度的提升和计算方法的日益精进，近年来大规模整数规划问题已变得越来越易于处理。

## AMPL 建模语言

数学规划的实践往往不像在计算机上运行某种算法并打印最优解那样简单。完整的工作流程更接近于：

- 用数学公式表述一个模型，即构建变量、目标函数和约束条件的抽象系统，用于表示待解问题的一般形式。
- 收集用以定义一个特定问题实例的数据。
- 根据模型和数据，生成特定的目标函数和约束。
- 通过运行一个程序——或者说，求解器 (Solver)——来求解该问题实例，应用算法以找到变量的最优值。
- 分析结果。
- 根据需要改进模型和数据，并重复上述过程。

如果人们能像求解器那样处理数学规划问题，那么建模中的表述和生成阶段或许会相对来讲更直截了当 (straightforward) 一些。然而现实是，人类建模者理解问题的形式与求解器算法处理问题的形式之间存在诸多差异。因此，将模型的表述从“建模者形式” (Modeler’s Form) 转换到“算法形式” (Algorithm’s form) 是一个耗时、成本高昂且往往容易出错的过程。

在线性规划的特殊情形下，算法形式中最为重大的部分是约束系数矩阵——即所有约束中与所有变量相乘的数值的表格。这通常是一个非常稀疏 (大部分元素为零) 的矩阵，其行数和列数范围从数百到数十万不等，且非零元素呈现错综复杂的分布模式。生成系数矩阵紧凑表示的计算机程序称为矩阵生成器 (Matrix Generator)。已有若干编程语言被专门设计用于编写矩阵生成器，标准的计算机编程语言也常被应用于此。

尽管矩阵生成器能成功实现从建模者形式到算法形式的部分转换的自动化，它们却仍难以维护和调试。应对此问题的一种有效方案是采用数学规划建模语言 (Modeling Language for Mathematical Programming)。建模语言的设计初衷是用可直接作为计算机系统输入的形式来表达建模者形式。随后向算法形式的转换便能全然由计算机自己完成，无需中间编程的阶段。建模语言有助于使数学规划更加经济可靠；它们在开发新模型和记录需要频繁修改的模型时尤其具有优势。

由于建模者用于表达数学规划的形式不止一种，因此建模语言也存在多种类型。代数建模语言 (Algebraic Modeling Language) 是一种基于传统数学符号来描述目标函数与约束函数的流行变体。代数语言提供了计算机可读的数学符号的等价表示，例如 $x_j + y_j$、$\sum_{j = 1}^{n}a_{ij}x_j$、$x_j\geq 0$ 以及 $j\in S$——这些符号对于任何学习过代数或微积分的人都十分熟悉。熟悉度是代数建模语言的主要优势之一；另一个优势在于它们特别适用于各种线性、非线性和整数规划模型。

尽管数学规划的成功算法早在20世纪50年代就已投入使用，但代数建模语言的开发与推广直到20世纪70年代才正式开始。此后，计算技术与计算机科学的进步使这类语言变得日益通用和高效。

本书是关于 AMPL 的，一种用于数学规划的代数建模语言 (An Algebraic Modeling Language for Mathematical Programming, 缩写即“AMPL”)；它由作者在 1985 年左右设计和实现，自彼时起一路在发展至今。AMPL 以其算术表达式与常规代数符号的相似性及其集合和下标表达式的通用性与强大功能而著称。AMPL 还将代数符号扩展以表达常见的数学规划结构，例如网络流约束 (Network Flow Constraint) 和分段线性规划 (Piecewise Linear)。

本书阐述的是 AMPL，一种数学规划代数建模语言[^2]；该语言由其作者于 1985 年左右设计实现，并持续发展至今。AMPL 以其算术表达式与常规代数符号的高度相似性，以及其集合与下标表达式的通用性和强大功能而著称。同时，AMPL 还扩展了代数符号体系，使其能够表达常见的数学规划结构，例如网络流约束 (Network Flow Constraints) 与分段线性关系 (Piecewise Linearities)。

AMPL offers an interactive command environment for setting up and solving mathematical programming problems. A flexible interface enables several solvers to be available at once so a user can switch among solvers and select options that may improve solver performance. Once optimal solutions have been found, they are automatically translated back to the modeler’s form so that people can view and analyze them. All of the general set and arithmetic expressions of the AMPL modeling language can also be used for displaying data and results; a variety of options are available to format data for browsing, printing reports, or preparing input to other programs. 

Through its emphasis on AMPL, this book differs considerably from the presentation of modeling in standard mathematical programming texts. The approach taken by a typical textbook is still strongly influenced by the circumstances of 30 years ago, when a student might be lucky to have the opportunity to solve a few small linear programs on any actual computer. As encountered in such textbooks, mathematical programming often appears to require only the conversion of a “word problem” into a small system of inequalities and an objective function, which are then presented to a simple optimization package that prints a short listing of answers. While this can be a good approach for introductory purposes, it is not workable for dealing with the hundreds or thousands of variables and constraints that are found in most real-world mathematical programs. 

The availability of an algebraic modeling language makes it possible to emphasize the kinds of general models that can be used to describe large-scale optimization problems. Each AMPL model in this book describes a whole class of mathematical programming problems, whose members correspond to different choices of indexing sets and numerical data. Even though we use relatively small data sets for illustration, the resulting problems tend to be larger than those of the typical textbook. More important, the same approach, using still larger data sets, works just as well for mathematical programs of realistic size and practical value.

We have not attempted to cover the optimization theory and algorithmic details that comprise the greatest part of most mathematical programming texts. Thus, for readers who want to study the whole field in some depth, this book is a complement to existing textbooks, not a replacement. On the other hand, for those whose immediate concern is to apply mathematical programming to a particular problem, the book can provide a useful introduction on its own.

In addition, AMPL software is readily available for experiment: the AMPL web site, `www.ampl.com`, provides free downloadable “student” versions of AMPL and representative solvers that run on Windows, Unix/Linux, and Mac OS X. These can easily handle problems of a few hundred variables and constraints, including all of the examples in the book. Versions that support much larger problems and additional solvers are also available from a variety of vendors; again, details may be found on the web site. 

## Outline of the book

The second edition, like the first, is organized conceptually into four parts. Chapters 1 through 4 are a tutorial introduction to models for linear programming:

1. Production Models: Maximizing Profits
2. Diet and Other Input Models: Minimizing Costs
3. Transportation and Assignment Models
4. Building Larger Models

These chapters are intended to get you started using AMPL as quickly as possible. They include a brief review of linear programming and a discussion of a handful of simple modeling ideas that underlie most large-scale optimization problems. They also illustrate how to provide the data that convert a model into a specific problem instance, how to solve a problem, and how to display the answers.

The next four chapters describe the fundamental components of an AMPL linear programming model in detail, using more complex examples to examine major aspects of the language systematically:

5. Simple Sets and Indexing
6. Compound Sets and Indexing
7. Parameters and Expressions
8. Linear Programs: Variables, Objectives and Constraints

We have tried to cover the most important features, so that these chapters can serve as a general user’s guide. Each feature is introduced by one or more examples, building on previous examples wherever possible.

The following six chapters describe how to use AMPL in more sophisticated ways:

9. Specifying Data
10. Database Access
11. Modeling Commands
12. Display Commands
13. Command Scripts
14. Interactions with Solvers

The first two of these chapters explain how to provide the data values that define a specific instance of a model; Chapter 9 describes AMPL ’s text file data format, while Chapter 10 presents features for access to information in relational database systems. Chapter 11 explains the commands that read models and data, and invoke solvers; Chapter 12 shows how to display and save results. AMPL provides facilities for creating scripts of commands, and for writing loops and conditional statements; these are covered in Chapter 13. Chapter 14 goes into more detail on how to interact with solvers so as to make the best use of their capabilities and the information they provide.

Finally, we turn to the rich variety of problems and applications beyond purely linear models. The remaining chapters deal with six important special cases and generalizations:

15. Network Linear Programs
16. Columnwise Formulations
17. Piecewise-Linear Programs
18. Nonlinear Programs
19. Complementarity Problems
20. Integer Linear Programs

Chapters 15 and 16 describe additional language features that help AMPL represent particular kinds of linear programs more naturally, and that may help to speed translation and solution. The last four chapters cover generalizations that can help models to be more realistic than linear programs, although they can also make the resulting optimization problems harder to solve.

Appendix A is the AMPL reference manual; it describes all language features, including some not mentioned elsewhere in the text. Bibliography and exercises may be found in most of the chapters.

## About the second edition

AMPL has evolved a lot in ten years, but its core remains essentially unchanged, and almost all of the models from the first edition work with the current program. Although we have made substantial revisions throughout the text, much of the brand new material is concentrated in the third part, where the original single chapter on the command environment has been expanded into five chapters. In particular, database access, scripts and programming constructs represent completely new material, and many additional AMPL commands for examining models and accessing solver information have been added.

The first edition was written in 1992, just before the explosion in Internet and web use, and while personal computers were still rather limited in their capabilities; the first student versions of AMPL ran on DOS on tiny, slow machines, and were distributed on floppy disks.

Today, the web site at `www.ampl.com` is the central source for all AMPL information and software. Pages at this site cover all that you need to learn about and experiment with optimization and the use of AMPL:

- Free versions of AMPL for a variety of operating systems.
- Free versions of several solvers for a variety of problem types.
- All of the model and data files used as examples in this book.

The free software is fully functional, save that it can only handle problems of a few hundred variables and constraints. Unrestricted commercial versions of AMPL and solvers are available as well; see the web site for a list of vendors.

You can also try AMPL without downloading any software, through browser interfaces at `www.ampl.com/TRYAMPL` and the NEOS Server (`neos.mcs.anl.gov`). The AMPL web site also provides information on graphical user interfaces and new AMPL language features, which are under continuing development.

## Acknowledgements to the first edition

We are deeply grateful to Jon Bentley and Margaret Wright, who made extensive comments on several drafts of the manuscript. We also received many helpful suggestions on AMPL and the book from Collette Coullard, Gary Cramer, Arne Drud, Grace Emlin, Gus Gassmann, Eric Grosse, Paul Kelly, Mark Kernighan, Todd Lowe, Bob Richton, Michael Saunders, Robert Seip, Lakshman Sinha, Chris Van Wyk, Juliana Vignali, Thong Vukhac, and students in the mathematical programming classes at Northwestern University. Lorinda Cherry helped with indexing, and Jerome Shepheard with typesetting. Our sincere thanks to all of them.

## Bibliography

E. M. L. Beale, “Matrix Generators and Output Analyzers.”In Harold W. Kuhn (ed.), *Proceedings of the Princeton Symposium on Mathematical Programming*, Princeton University Press (Princeton, NJ, 1970) pp. 25–36. A history and explanation of matrix generator software for linear programming.

Johannes Bisschop and Alexander Meeraus,“On the Development of a General Algebraic Modeling System in a Strategic Planning Environment.” Mathematical Programming Study **20** (1982) pp. 1–29. An introduction to GAMS, one of the first and most widely used algebraic modeling languages.

Robert E. Bixby,“Solving Real-World Linear Programs: A Decade and More of Progress.”Operations Reearch **50** (2002) pp. 3)–15. A history of recent advances in solvers for linear programming. Also in this issue are accounts of the early days of mathematical programming by pioneers of the field.

George B. Dantzig,“Linear Programming: The Story About How It Began.” In Jan Karel Lenstra, Alexander H. G. Rinnooy Kan and Alexander Schrijver, eds., *History of Mathematical Programming: A Collection of Personal Reminiscences*. North-Holland (Amsterdam, 1991) pp. 19–31. A source for our brief account of the history of linear programming. Dantzig was a pioneer of such key ideas as objective functions and the simplex algorithm.

Robert Fourer,“Modeling Languages versus Matrix Generators for Linear Programming.” ACM Transactions on Mathematical Software **9** (1983) pp. 143–183. The case for modeling languages.

C. A. C. Kuip, “Algebraic Languages for Mathematical Programming.” European Journal of Operational Research **67** (1993) 25–51. A survey.

[^1]: **译者注**：在英语中，“Programming”具有“规划”与“编程”的双重含义。
[^2]: **译者注**：其名称“AMPL”源自英文“An Algebraic Modeling Language for Mathematical Programming”的首字母缩写，即“一种数学规划代数建模语言”。