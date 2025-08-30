# Production Models: Maximizing Profits

As we stated in the Introduction, mathematical programming is a technique for solving certain kinds of problems — notably maximizing profits and minimizing costs — subject to constraints on resources, capacities, supplies, demands, and the like. AMPL is a language for specifying such optimization problems. It provides an algebraic notation that is very close to the way that you would describe a problem mathematically, so that it is easy to convert from a familiar mathematical description to AMPL. 

We will concentrate initially on linear programming, which is the best known and easiest case; other kinds of mathematical programming are taken up later in the book. This chapter addresses one of the most common applications of linear programming: maximizing the profit of some operation, subject to constraints that limit what can be produced. Chapters 2 and 3 are devoted to two other equally common kinds of linear programs, and Chapter 4 shows how linear programming models can be replicated and combined to produce truly large-scale problems. These chapters are written with the beginner in mind, but experienced practitioners of mathematical programming should find them useful as a quick introduction to AMPL.

We begin with a linear program (or LP for short) in only two decision variables, motivated by a mythical steelmaking operation. This will provide a quick review of linear programming to refresh your memory if you already have some experience, or to help you get started if you’re just learning. We’ll show how the same LP can be represented as a general algebraic model of production, together with specific data. Then we’ll show how to express several linear programming problems in AMPL and how to run AMPL and a solver to produce a solution. 

The separation of model and data is the key to describing more complex linear programs in a concise and understandable fashion. The final example of the chapter illustrates this by presenting several enhancements to the model. 

## A two-variable linear program

An (extremely simplified) steel company must decide how to allocate next week’s time on a rolling mill. The mill takes unfinished slabs of steel as input, and can produce either of two semi-finished products, which we will call bands and coils. (The terminology is not entirely standard; see the bibliography at the end of the chapter for some accounts of realistic LP applications in steelmaking.) The mill’s two products come off the rolling line at different rates:

```
Tons per hour: Bands 200
               Coils 140
```

and they also have different profitabilities:

```
Profit per ton: Bands $25
                Coils $30
```

To further complicate matters, the following weekly production amounts are the most that can be justified in light of the currently booked orders:

```
Maximum tons: Bands 6,000
              Coils 4,000
```

The question facing the company is as follows: If 40 hours of production time are available this week, how many tons of bands and how many tons of coils should be produced to bring in the greatest total profit?

 While we are given numeric values for production rates and per-unit profits, the tons of bands and of coils to be produced are as yet unknown. These quantities are the decisionvariableswhose values we must determine so as to maximize profits. The purpose of the linear program is to specify the profits and production limitations as explicit formulas involving the variables, so that the desired values of the variables can be determined systematically.

In an algebraic statement of a linear program, it is customary to use a mathematical shorthand for the variables. Thus we will write $X_B$ for the number of tons of bands to be produced, and $X_C$ for tons of coils. The total hours to produce all these tons is then given by

$$
(\text{hours to make a ton of bands}) \times X_B + (\text{hours to make a ton of coils}) \times X_C
$$

This number cannot exceed the 40 hours available. Since hours per ton is the reciprocal of the tons per hour given above, we have a constraint on the variables:

$$
(1 / 200) X_B + (1 / 140) X_C \leq 40.
$$

There are also production limits:

$$
0 \leq X_B \leq 6000
0 \leq X_C \leq 4000
$$

In the statement of the problem above, the upper limits were specified, but the lower limits were assumed — it was obvious that a negative production of bands or coils would be meaningless. Dealing with a computer, however, it is necessary to be quite explicit.

By analogy with the formula for total hours, the total profit must be

$$
(\text{profit per ton of bands}) \times X_B + (\text{profit per ton of coils}) \times X_C
$$

That is, our objective is to maximize 25 X B + 30 X C. Putting this all together, we have the following linear program:

$$
\begin{align*}
  \text{Maximize} \quad & 25 X_B + 30 X_C \\
  \text{Subject to} 
    \quad & (1 / 200) X_B + (1 / 140) X_C \leq 40 \\
    & X_B \leq 6000 \\
    & X_C \leq 4000
\end{align*}
$$

This is a very simple linear program, so we’ll solve it by hand in a couple of ways, and then check the answer with AMPL.

First, by multiplying profit per ton times tons per hour, we can determine the profit per hour of mill time for each product:

```
Profit per hour: Bands $5,000
                 Coils $4,200
```

Bands are clearly a more profitable use of mill time, so to maximize profit we should produce as many bands as the production limit will allow — 6,000 tons, which takes 30 hours. Then we should use the remaining 10 hours to make coils — 1,400 tons in all. The profit is $25 times 6,000 tons plus $30 times 1,400 tons, for a total of $192,000.

Alternatively, since there are only two variables, we can show the possibilities graphically. If $X_B$ values are plotted along the horizontal axis, and $X_C$ values along the vertical axis, each point represents a choice of values, or solution, for the decision variables:
