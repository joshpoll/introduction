# Tutorial 4: Writing Constraints & Objectives

## Introduction

We've been using constraints and objectives in our previous tutorials, and now we will begin writing our own constraints and objectives! Being able to write your own constraints and objectives is an important step in becoming an advanced Penrose developer. 

We will start with understanding how are constraints and objectives done in Penrose, and then we will go through several examples line by line to apply our conceptual understanding concretely. 

## Overview: How We Write Constraints

For a more in-depth description, refer to [here](https://github.com/penrose/penrose/wiki/Getting-started#writing-new-objectivesconstraintscomputations). 

With our Penrose triple, the syntax is most likely familiar to you from your prior programming experiences, but with constraints, we will be writing code in a particular way that allows Penrose to use a particular technique called **automatic differentiation, autodiff** for short. The Penrose system uses autodiff to find the optimized diagram. Essentially, we need to write constraints and objectives in a specific way in order for Penrose to do its magic. For more on autodiff, read [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#introduction). 

So now, how exactly do we write it? 

### 1. Autodiff functions

We enter this new world without our normal numbers and operations such as `+`  and `-`. Instead of what we normally can do across different languages such as javascript, python or c, we now have to use special number types and operations. 

But no worries, it's straightforward. We have unary, binary, trinary, n-ary, and composite operations. The list of autodiff functions can be found [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#to-use-the-autodiff). For example, instead of `a + b`, we now do `add(a, b)`. 

### 2. Special Number Types

All numbers need to actually be special types called `VarAD`s that have special ops applied to them, so the system can track what operations are being applied to what. We convert to and from `number` and `VarAD` using the convenience functions `varOf` \(or `constOf`\) and `numOf`. You can also apply atomic and composite operations to `VarAD`s as described in the autodiff functions. Beyond that, you should not directly modify the `VarAD`s.

In the system, all arguments to objectives/constraints/functions are \(should be\) automatically converted to `VarAD`s, though if not, you can use the `constOfIf` function.

### 3. Write functional code

We must write these functions in _straight-line functional style_ \(i.e. no imperative style, no mutating state, no for-loops or if statements\). These restrictions are so the Penrose system can work its magic. Therefore we need to avoid writing things like `x = x + 1`  which translates to `let x = add(x, constOf(1))`  in autodiff code, and instead use constant intermediate variables like this: `const x0 = ... const x1 = add(x0, constOf(1))`. 

## Constraints Example: minSize & maxSize

## Constraints Example: canvas containment

## Objectives Example: circle repel

## Exercises

