---
description: Writing Constraints & Objectives
---

# Tutorial 4: Writing Constraints & Objectives

## Introduction

We've been using constraints and objectives in our previous tutorials, and now we will begin writing our own constraints and objectives! Being able to write your own constraints and objectives is an important step in becoming an **advanced Penrose developer**. 

You can for sure create beautiful diagrams with what you have already learned from the previous tutorials as _an user of the Penrose system_, whereas with the knowledge from this tutorial, you will be _extending the existing Penrose system_, producing tools other people can use as well. Therefore, read on if you want to understand the system side inner-workings of Penrose. 

We will start with understanding what, and how are constraints and objectives done in Penrose, and then we will go through several examples line by line to apply our conceptual understanding concretely. 

## What is a constraint?

* what is optimization, energy 
* shifting from using the system versus writing a new part of the system 
* constraint examples of math only 
* how do you make one circle contained in another? \(math only\) 

## Overview: How We Write Constraints

With our Penrose triple, the syntax is most likely familiar to you from your prior programming experiences, but with constraints, we will be writing code in a particular way that allows Penrose to use a particular technique called **automatic differentiation,** _**autodiff**_ ****for short. The Penrose system uses autodiff to find the optimized diagram. Essentially, we need to write constraints and objectives in a specific way in order for Penrose to do its magic. For more on autodiff, read [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#introduction). 

So now, how exactly do we write it? 

### 1. Autodiff functions

We enter this new world without our normal operations such as `+`  and `-`. Instead of what we normally can do across different languages such as javascript, python or c, we now have to use special number types and operations. 

But no worries, it's straightforward. We have unary, binary, trinary, n-ary, and composite operations. The list of autodiff functions can be found [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#to-use-the-autodiff). For example, instead of `a + b`, we now do `add(a, b)`. 

### 2. Special Number Types

All numbers need to actually be special types called `VarAD`  to be valid inputs for the autodiff functions. We can convert to and from normal numbers and `VarAD` using the functions `varOf: number -> VarAD` and `constOf: number -> VarAD` , and `numOf: VarAD -> number`, where the type `number` is our common const number values like `1, 2, 3, ...`.

For example, if we need to do `5 + 3` , the equivalent autodiff expression is `add(constOf(5), constOf(3))` or `add(varOf(5), varOf(5))`. 

### 3. Write functional code

We must write these functions in _straight-line functional style_ \(i.e. no imperative style, no mutating state, no for-loops or if statements\). These restrictions are so the Penrose system can work its magic. Therefore we need to avoid writing things like `x = x + 1`  which translates to `let x = add(x, constOf(1))`  in autodiff code, and instead use constant intermediate variables like this: `const x0 = ... const x1 = add(x0, constOf(1))`. 

### 4. Zero-based inequality to penalty 

For every constraint function we write, we take in shapes and output a number or a tensor as penalty. In short, Penrose will try to minimize the outputs of all the constraint functions used for the diagram. 

When we write a constraint, for example, we want to constrain one circle `s1` to be smaller than another circle `s2` by some amount `offset`. In math, we would require that `r1 - r2 > offset`. An inequality constraint needs to be written in the form `p(x) > 0`.  

Since we penalize the amount the constraint is greater than `0`. So, this constraint is written as `r1 - r2 - offset > 0`, or `p(r1, r2) = r1 - r2 - offset`. 

#### Some general rules on writing penalty: 

* Let's say I want the constraint `f(x) <= c` to be true.
* I translate it to the zero-based inequality `f(x) - c <= 0`.
* I translate the inequality constraint into an energy \(penalty\) `E(x) = f(x) - c` — it is greater than `0` iff the constraint is violated, and the more the constraint is violated, the higher the energy is \(e.g. if `f(x)` is way bigger than `c` then the energy is a lot bigger than `0`\). **That is the form of all the constraints in the system \(and the form you'll be writing them in\).** 

## Constraints Example: minSize & maxSize

We will go through examples of `minSize` and `maxSize` constraints that are specifically for _**circles only**_. 

```text
minSize: ([shapeType, props]: [string, any]) => {
    const limit = 20;
    return sub(constOf(limit), props.r.contents);
}
```

insert description

```text
maxSize: ([shapeType, props]: [string, any]) => {
    const limit = Math.max(...canvasSize);
    return sub(props.r.contents, constOf(limit / 6.0));
}
```

insert description

## Constraints Example: canvas containment

## Objectives Example: circle repel

## Exercises

