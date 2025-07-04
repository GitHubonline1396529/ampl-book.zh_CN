# Introduction

As our title suggests, there are two aspects to the subject of this book. The first is mathematical programming, the optimization of a function of many variables subject to constraints. The second is the AMPL modeling language, which we designed and implemented to help people use computers to develop and apply mathematical programming models. 

We intend this book as an introduction both to mathematical programming and to AMPL. For readers already familiar with mathematical programming, it can serve as a user’s guide and reference manual for the AMPL software. We assume no previous knowledge of the subject, however, and hope that this book will also encourage the use of mathematical programming models by those who are new to the field. 

## Mathematical programming

The term “programming” was in use by 1940 to describe the planning or scheduling of activities within a large organization. “Programmers” found that they could represent the amount or level of each activity as a variable whose value was to be determined. Then they could mathematically describe the restrictions inherent in the planning or scheduling problem as a set of equations or inequalities involving the variables. A solution to all of these constraints would be considered an acceptable plan or schedule. 

Experience soon showed that it was hard to model a complex operation simply by specifying constraints. If there were too few constraints, many inferior solutions could satisfy them; if there were too many constraints, desirable solutions were ruled out, or in the worst case no solutions were possible. The success of programming ultimately depended on a key insight that provided a way around this difficulty. One could specify, in addition to the constraints, an objective: a function of the variables, such as cost or profit, that could be used to decide whether one solution was better than another. Then it didn’t matter that many different solutions satisfied the constraints — it was sufficient to find one such solution that minimized or maximized the objective. The term mathematical programming came to be used to describe the minimization or maximization of an objective function of many variables, subject to constraints on the variables. 

In the development and application of mathematical programming, one special case stands out: that in which all the costs, requirements and other quantities of interest are terms strictly proportional to the levels of the activities, or sums of such terms. In mathematical terminology, the objective is a linear function, and the constraints are linear equations and inequalities. Such a problem is called a linear program, and the process of setting up such a problem and solving it is called linear programming. Linear programming is particularly important because a wide variety of problems can be modeled as linear programs, and because there are fast and reliable methods for solving linear programs even with thousands of variables and constraints. The ideas of linear programming are also important for analyzing and solving mathematical programming problems that are not linear. 

All useful methods for solving linear programs require a computer. Thus most of the study of linear programming has taken place since the late 1940’s, when it became clear that computers would be available for scientific computing. The first successful computational method for linear programming, the simplex algorithm, was proposed at this time, and was the subject of increasingly effective implementations over the next decade. Coincidentally, the development of computers gave rise to a now much more familiar meaning for the term “programming”. 

In spite of the broad applicability of linear programming, the linearity assumption is sometimes too unrealistic. If instead some smooth nonlinear functions of the variables are used in the objective or constraints, the problem is called a nonlinear program. Solving such a problem is harder, though in practice not impossibly so. Although the optimal values of nonlinear functions have been a subject of study for over two centuries, computational methods for solving nonlinear programs in many variables were developed only in recent decades, after the success of methods for linear programming. The field of mathematical programming is thus also known as large scale optimization, to distinguish it from the classical topics of optimization in mathematical analysis. 

The assumptions of linear programming also break down if some variables must take on whole number, or integral, values. Then the problem is called integer programming, and in general becomes much harder. Nevertheless, a combination of faster computers and more sophisticated methods have made large integer programs increasingly tractable in recent years. 

## The AMPL modeling language

Practical mathematical programming is seldom as simple as running some algorithmic method on a computer and printing the optimal solution. The full sequence of events is more like this:

- Formulate a model, the abstract system of variables, objectives, and constraints that represent the general form of the problem to be solved.
- Collect data that define a specific problem instance.
- Generate a specific objective function and constraint equations from the model and data.
- Solve the problem instance by running a program, or solver, to apply an algorithm that finds optimal values of the variables.
- Analyze the results.
- Refine the model and data as necessary, and repeat.
