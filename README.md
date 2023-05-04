Download Link: https://assignmentchef.com/product/solved-cs502-project-6-optimization
<br>
Your task in this assignment is to implement a series of optimizations for the CPS compiler. In the skeleton code you will nd <sub>src/miniscala/CPSOptimizer.scala</sub>. This le contains the optimizer mechanism, followed by two speci c instantiations: <sub>CPSOptimizerHigh</sub> and <sub>CPSOptimizerLow</sub>. Your task is to ll in the optimizer mechanism, which consists of shrinking and non-shrinking optimizations (as described in the lectures) and the speci c implementation of <sub>CPSOptimizerHigh</sub>. The speci c implementation of <sub>CPSOptimizerLow</sub> is given.

More speci cally, you are expected to implement:

Dead code elimination

Common-subexpression elimination

Constant folding

Neutral/absorbing elements optimization

Shrinking inlining

General inlining, according to a predetermined heuristics (see below: Notes on Implementation)

For bonus grade, you may implement:

Block primitive optimizations

For fun and fame (see optimization challenge), you may implement:

Any optimizations you can think about

Do not implement:

η-reduction

Conti cation (already implemented. Can be turned on by uncommenting line 49 in Main.scala. Careful the tests do not use it.)

Once the optimizer is fully functional, it should optimize the following example correctly:

<strong>Input:</strong>

def printChar(c: Int) = putchar(c);

def functionCompose(f: Int =&gt; Int, g: Int =&gt; Int) = (x: Int) =&gt; f(g(x));

def plus(x: Int, y: Int) = x + y;   def succ(x: Int) = x + 1;   def twice(x: Int) = x + x;

printChar(functionCompose(succ,twice)(39));   printChar(functionCompose(succ,succ)(73));   printChar(functionCompose(twice,succ)(4));   0 <strong>Output:</strong>

vall t_2 = 79;

valp t_3 = byte-write(t_2);   vall t_5 = 75;

valp t_6 = byte-write(t_5);   vall t_8 = 10;

valp t_9 = byte-write(t_8);   vall t_11 = 0;   halt(t_11)

Closures? Blocks? The optimizer eliminated all abstractions and gave us the simplest inline code possible. &#x1f600; Now, to start hacking on the optimizer:

cd proj6/compiler sbt

&gt; run ../library/miniscala.lib ../examples/pascal.scala

…

[info] Running miniscala.Main ../library/miniscala.lib ../examples/pascal.scala enter size (0 to exit)&gt; 12

(1 11 55 165 330 462 462 330 165 55 11 1 )

(1 10 45 120 210 252 210 120 45 10 1 )

(1 9 36 84 126 126 84 36 9 1 )

(1 8 28 56 70 56 28 8 1 )

(1 7 21 35 35 21 7 1 )

(1 6 15 20 15 6 1 )

(1 5 10 10 5 1 )

(1 4 6 4 1 )

(1 3 3 1 )

(1 2 1 )

(1 1 ) (1 )

enter size (0 to exit)&gt; 0

Instruction Stats

=================

6205  LetP

2998  LetL     …

The “Instruction stats” indicate <strong>how many instructions were executed</strong> while computing the pascal triangle. This is in contrast to the total instructions in the output program, which is constant regardless of the input. NOTE: the number is when you use conti cation.

<h2>Notes on Implementation</h2>

The implementation sketch that is given to you is not a trivial mapping of the theory into code, so here are some notes to guide you through it.

The optimizations are split in shrinking (in method <sub>shrink</sub>) and non-shrinking (or general inlining, in method inline) optimizations. Remember that shrinking optimizations are safe to apply as often as desired, while inlining may lead to arbitrarily large code, or even diverge the optimizer.

The general idea of the optimizer (see method <sub>apply</sub>) is the following: After an initial step of iteratively applying <sub>shrink</sub> until a xed point, we iteratively apply a step of <sub>inline</sub> and a step of <sub>shrink</sub> until one of the following happens:

The tree does not change

We reach a speci ed number of iterations, or

The resulting tree’s size exceeds <sub>maxSize</sub>, given as a function of the initial size.

The two rst conditions are checked by the <sub>fixedPoint</sub> function itself, while the third is checked within inline.

The <sub>inline</sub> function is actually a small series of inlining steps. During each inlining step, we choose to inline functions/continuations of increasing size. For continuations, this size is linear to the number of steps, but for functions it is exponential (according to the Fibonacci sequence). We stop inlining after a speci ed number of steps, or if the tree grows to more than <sub>maxSize.</sub>

The internal traversals of both the <sub>shrink</sub> and <sub>inline</sub> functions accept an implicit argument of the <sub>State </sub>type, which, as you may have guessed, tracks the local state of the optimization. You are supposed to update it as you traverse the subtrees using the designated methods. The descriptions in the source le will hopefully make each eld’s role to the optimization clear.

<h2>Testing</h2>

Testing will be slightly different this time. We have 3 sets of tests:

<strong>Blackbox</strong> tests check the program you output runs correctly, therefore the optimizations keep the semantics of the language (see test/miniscala/test/CPSOptimizer_Blackbox.scala)

<strong>Greybox</strong> tests execute the program while recording the execution trace and then require certain properties of the trace, such as, for example that at most 2 functions were de ned (see

test/miniscala/test/CPSOptimizer_Greybox.scala and the additions to

src/miniscala/CPSInterpreter.scala for more details)

<strong>Optimization challenge</strong> allows showing off how good your optimizer is on larger applications: we will run an example application and output the execution statistics (see src/miniscala/CPSInterpreter.scala) — you can compete with your colleagues and with our reference implementation to get the best results possible. And yes, you should brag about your results on Piazza!

This assignment gives you a lot of liberty into how and what you optimize, so go ahead and write the best optimizer in the entire class!

Challenge Results

For the reference compiler we have developed, the challenge results are given below. Different results form these statistics are encouraged, especially if they reduce the numbers in one category or the other.

NOTE: Turn on conti cation in order to improve your numbers.

Challenge: maze.scala with 12 for both inputs.

Instruction Stats

=================

1320209  LetP

1076643  LetL

1003773  LetC   446036  If

293889  AppF

113226  AppC

1  LetF

1  Halt




Value Primitives Stats

======================

510160  CPSBlockGet$

247412  CPSOr$

241868  CPSArithShiftL$

239466  CPSBlockTag$

30504  CPSBlockSet$

14660  CPSBlockAlloc

10367  CPSSub$

10105  CPSAdd$

8621  CPSArithShiftR$

3433  CPSAnd$

1611  CPSMul$

1056  CPSXOr$

662  CPSByteWrite$

264  CPSMod$

14  CPSBlockLength$

6  CPSByteRead$




Logic Primitives Stats

======================

380718  CPSEq$

63198  CPSNe$

2058  CPSLt$

52  CPSGt$

10  CPSLe$




Functions defined: 27

Continuations defined: 1003773