+++
title = "If You're not writing a Program, don't use a programming language"
author = ["Joel Lee"]
tags = ["Distributed Systems"]
date = "2020-01-16"
draft = false
+++

## Inside very large program, there is an algorithm trying to get out {#inside-very-large-program-there-is-an-algorithm-trying-to-get-out}

-   Euclid's Algorithm -> For computing GCD of integers M, N such that both M and N are multiples of it.
-   If we were writing a program, it would have to have types.
-   Algorithm is only useful inside of this program.
-   Usually, fails to get out of the program because the programmer doesn't know the algorithm is there
-   Result: Programs are hard to debug because we're debugging the algorithm at the code level. Because we are trying to optimize an algorithm at the code level.


## Can we do better? {#can-we-do-better}

-   Describe algorithms with math.
-   One algo is most generally useful in practice We describe the execution of an algorithm as a sequnece of states
-   Sequence of states are behaviors. We should convert the algorithm to behaviors.


## How to describe an algorithm: {#how-to-describe-an-algorithm}

-   Have an initial predicate on states: \\((x=M)^(y=N)\\)
-   Next state predicate on pairs of states
-   We don't need a programming language. We just need simple math.
-   Behavior represents an execution of Euclid's algorithm iff
-   InitE is true on \s\_1


## Properties you should know about {#properties-you-should-know-about}

-   Safety property: What is allowed to ahppen
-   Liveliness property: What must eventually happen (i.e. The algorithm must eventually produce an output)
-   Any property can be written as safety ^ liveliness
-   Invariance: Partial correctness of Euclid's algorithm: if it has terminated then x = GCD(M, N)
-   Box in front means that it must be true for all states of the behavior.
-   Satisfying a property is just logical implicaiton.


## A Closer Look at Formulas {#a-closer-look-at-formulas}

-   State is an assignment of values to all possible variables.
-   Allows behaviors in which the values of x and y never change.
-   Math works great on tiny examples but everyone thinks that it doesn't work in the real world. Instead of 6 lines we might have 100s of lines.
-   Large formulas are handled by hierachical decomposition.
-   Math has the best hierachical decomposition known to man.
