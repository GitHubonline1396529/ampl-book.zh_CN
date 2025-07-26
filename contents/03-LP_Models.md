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
