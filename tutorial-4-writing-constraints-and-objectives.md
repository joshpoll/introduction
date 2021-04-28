---
description: Writing Constraints & Objectives
---

# Tutorial 4: Writing Constraints & Objectives

## Introduction

We've been using constraints and objectives in our previous tutorials, and now we will begin writing our own constraints and objectives! Being able to write your own constraints and objectives is an important step in becoming an **advanced Penrose developer**. 

You are already equipped to create beautiful diagrams with what you have already learned from the previous tutorials as _an user of the Penrose system_, whereas with the knowledge from this tutorial, you will be _extending the existing Penrose system_, contributing to the platform for many other users. 

We will start with understanding what, and how are constraints and objectives done in Penrose, and then we will go through several examples line by line to apply our conceptual understanding concretely. 

## TO-DO: What is optimization / energy function? What is a constraint? 

Things that need to be introduced here \(used later\)

* penalty, energy function 
* multiplying by weight \(for `repel`\)

## Conceptual: How To Come Up With Constraints?

In the normal world, when we want a circle A contained in another circle B, this is normally what comes into our mind. **\[ add more on what this section is about \]**

But let's pause for a second, and _really_ think about what does it meant for a circle to be contained in another circle mathematically? 

There are generally 3 scenarios for the containment relationship between 2 circles. 

![](.gitbook/assets/circles.png)

We have completely contained, overlapping but not contained, and completely disjoint. It is visually obvious to any of us. We get showed 2 circles, and in a split second, we can identify their containment relationship. Unfortunately, Penrose does not have eyes,  but good news is, it speaks math! Therefore, let's take a new look at these circles.

![](.gitbook/assets/w_cent_rad.png)

**\[ change diagram to have labels with vector notation, i.e. c\_a and c\_b for center of a and b\]** 

Recall the general equation for a circle where p is some point, c is the center and r is the radius. 

$$
||p-c|| < r
$$

The center coordinate and radius are the information we have about **any** circle, and we will use these information to determine two circle's containment relationship. We begin with getting the distance between the circles' centers. 

**\[ insert diagram with distance labeled \]**

Notice how the distance is progressively smaller as A is more and more contained in B as expected. So a potentially working but not very good energy function would be returning the distance between the centers. We will take a step further and subtract the difference between A and B's radii,  `B-A` from the distance `d`.

## Concrete: How We Write Constraints

With our Penrose triple, the syntax is most likely familiar to you from your prior programming experiences, but with constraints, we will be writing code in a particular way that allows Penrose to use a particular technique called **automatic differentiation,** _**autodiff**_ ****for short. The Penrose system uses autodiff to find the optimized diagram. Essentially, we need to write constraints and objectives in a specific way in order for Penrose to do its magic. For more on autodiff, read [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#introduction). 

So now, how exactly do we write it? 

### 1. Autodiff functions

We enter this new world without our normal operations such as `+`  and `-`. Instead of what we normally can do across different languages such as javascript, python or c, we now have to use special number types and operations. 

But no worries, it's straightforward. We have unary, binary, trinary, n-ary, and composite operations. The list of autodiff functions can be found [here](https://github.com/penrose/penrose/wiki/Autodiff-guide#to-use-the-autodiff). For example, instead of `a + b`, we now do `add(a, b)`.  The composite operations are accessed through `ops.functionName`.

### 2. Special Number Types

All numbers need to actually be special types called `VarAD`  to be valid inputs for the autodiff functions. We can convert to and from normal numbers and `VarAD` using the functions `varOf: number -> VarAD` and `constOf: number -> VarAD` , and `numOf: VarAD -> number`, where the type `number` is our common const number values like `1, 2, 3, ...`.

For example, if we need to do `5 + 3` , the equivalent autodiff expression is `add(constOf(5), constOf(3))` or `add(varOf(5), varOf(5))`. 

### 3. Write functional code

We must write these functions in _straight-line functional style_ \(i.e. no imperative style, no mutating state, no for-loops or if statements\). These restrictions are so the Penrose system can work its magic. Therefore we need to avoid writing things like `x = x + 1`  which translates to `let x = add(x, constOf(1))`  in autodiff code, and instead use constant intermediate variables like this: `const x0 = ... const x1 = add(x0, constOf(1))`. 

### 4. Zero-based inequality to penalty \(will be edited down according to how the definition section goes\)

For every constraint function we write, we take in shapes and output a number or a tensor as penalty. In short, Penrose will try to minimize the outputs of all the constraint functions used for the diagram. 

When we write a constraint, for example, we want to constrain one circle `s1` to be smaller than another circle `s2` by some amount `offset`. In math, we would require that `r1 - r2 > offset`. An inequality constraint needs to be written in the form `p(x) > 0`.  

Since we penalize the amount the constraint is greater than `0`. So, this constraint is written as `r1 - r2 - offset > 0`, or `p(r1, r2) = r1 - r2 - offset`. 

#### Some general rules on writing penalty: 

* Let's say I want the constraint `f(x) <= c` to be true.
* I translate it to the zero-based inequality `f(x) - c <= 0`.
* I translate the inequality constraint into an energy \(penalty\) `E(x) = f(x) - c` — it is greater than `0` iff the constraint is violated, and the more the constraint is violated, the higher the energy is \(e.g. if `f(x)` is way bigger than `c` then the energy is a lot bigger than `0`\). 

### 5. Accessing Shape Field Value

One common operation is to access the parameter of the shape, which is done via `shapeName.propertyName.contents`, which will return a `VarAD`.  For example, if you had a circle `c` as input, and you want its radius, doing `c.r.contents` will give you something like `5.0` \(wrapped in a `VarAD`\).

## Constraints Example: minSize & maxSize

We will go through examples of `minSize` and `maxSize` constraints that are specifically for _**circles only**_. 

```text
minSize: ([shapeType, props]: [string, any]) => {
    const limit = 20;
    return sub(constOf(limit), props.r.contents);
}
```

Here we see several things in play. 

* **Input:** The function takes in the shape, which is represented by a string of its shape name, which would be `"Circle"` in this case, and a `prop` which contains the properties of the circle. 
* **Numbers:** Instead of directly using constant numbers `20`, we have to use `constOf(limit)` in order to pass it as a valid input for the autodiff function.
* **Operations:** Instead of using the subtraction operator `-` like we normally do, we have to use the autodiff function `sub` .
* **Accessing Shape Property:** We access the shape's property value by `shapeName.propertyName.contents` , where we have `propertyName = r` for radius. 
* **Logic:** We want the input circle to have a minimum size as the function name suggests, and remember with these functions, and we want to minimize whatever value we return. For example, with a bad small circle with a radius of 1, we will return 19, whereas with a good big circle with a radius of 30, we will return -10, which is much smaller, thus meaning it's good. 

```text
import { canvasSize } from "renderer/ShapeDef";

maxSize: ([shapeType, props]: [string, any]) => {
    const limit = Math.max(...canvasSize);
    return sub(props.r.contents, constOf(limit / 6.0));
}
```

The function `maxSize` is very similar to `minSize` plus using a global variable `canvasSize` that is the size of the canvas, which we use as a limit of our circle's radius. It is divided by `6.0` as you can imagine, if we simply constraint the radius by `canvasSize`, we can have a circle with a radius of `canvasSize`, which can cover the entire canvas and we don't want that. You can feel free to play around with the amount you divide by, but `6.0` has proven to work pretty well. 

## Objectives Example: circle repel

With objectives, we want them to output the "badness" of the inputs \(as a number or Tensor\), and have local minima where we want the solution to be.

Now we look at an objective that makes two circles repel, i.e. encouraging the two circles to be far apart from each other. 

```text
repel: ([t1, s1]: [string, any], [t2, s2]: [string, any],
                                       weight = 10.0) => {
    const repelWeight = 10e6;
    let res = inverse(ops.vdistsq(s1.center.contents, s2.center.contents));
    return mul(res, constOf(repelWeight));
}
```

We will look at this code together step by step,

* **Input:** The function takes in similar inputs as the constraint functions we've just looked at, where for convenience, instead of `shapeType` we are simply using `t`. For the last input `weight`, `repel` typically needs to have a weight multiplied since its magnitude is small. 
* **Operations:** Here we use 3 autodiff functions, `inverse`, `mul`, and `ops.vdistsq.`The `mul` function does multiplication,  `inverse(v)` function returns `1 / v` and `ops.vdistsq(v, w)` returns the Euclidean distance squared between vectors `v` and `w`. Remember `ops` is for composite functions that work on vectors. 
* **Logic:** 

## TO-DO: Exercises

* Writing the circle functions for rectangles? 

Reference: [https://github.com/penrose/penrose/wiki/Getting-started\#writing-new-objectivesconstraintscomputations](https://github.com/penrose/penrose/wiki/Getting-started#writing-new-objectivesconstraintscomputations)

