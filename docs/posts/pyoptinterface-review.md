---
title: pyOptInterface - what can it do?
description: pyOptInterface just turned 1 year old, and I finally had an opportunity to take it for a spin. Let's have a look!
slug: pyoptinterface-review
date: 2025-05-25
comments: true
categories:
    - Optimization
---

## TL;DR

- `pyOptInterface` is a new Python modeling framework first released in April 2024 and is currently in version 0.4.1.
- It is well-written, and really quite fast. It has decent documentation, and also supports conic and nonlinear programming.
- It does however not use all the nice things Python has to offer, which makes it more painful to work with than it has to be.
- With some effort, this could become my new daily driver for optimization modeling. However in its current form, the usage pains are higher than the awesome tech behind it.

<!-- more -->

## What is `pyOptInterface`?

It ([code](https://github.com/metab0t/PyOptInterface/tree/master), [docs](https://metab0t.github.io/PyOptInterface/index.html)) is a modeling framework in Python, i.e. a way that allows you to write optimization models and then interface with different solvers. As of April 2025, it supports Gurobi, COPT, HiGHS, Mosek and Ipopt. So far, so straightforward. But is it any good? And how does it compare to [`pyomo`](https://github.com/Pyomo/), [`python-mip`](https://github.com/coin-or/python-mip) and maybe even [`feloopy`](https://github.com/feloopy/feloopy)? Let's find out!

!!! note
    Almost all modeling frameworks are written in Python. The only non-Python one I am aware of is the C# package [`Optano.Modeling`](https://www.nuget.org/packages/OPTANO.Modeling), not counting [`JuMP`](https://jump.dev/), which is baked directly into Julia.

## Why and how to choose a modeling framework?

- It makes **switching solvers simple**. This is relevant e.g. when you:
    - have a limited number of licenses for a commercial solver
    - are evaluating different solvers
- It may be a nicer interface than the underlying solver you want to use
- It makes your code more accessible for use, e.g. if you want to give your end user the ability to toggle between solvers

So given these use cases, my criteria for a good modeling framework is:
 
1. It has to be **free and open-source**. This is important because I need to check how it works, fork it if necessary and not have an additional expense if I am already using to avoid buying more licenses.
2. It needs to be **easy to use**. If the main "product" is the interface, it should be a nice interface.
3. It should be **fast**. This is less important than ease of use for me, because I could just use the interface of the solver directly, if it is easier to use. However, if two frameworks are roughly of the same ease-of-use, speed becomes a critical factor.
4. It should be **maintained**. I don't like using libraries which have not had an update in years.

## So, how does `pyOptInterface` do?

| Criteria    | How does it do?       |
| ----------- | ----------------------|
| FOSS        | :material-check:      |
| Ease-of-use | :material-close:      |
| Fast        | :material-check-all:  |
| Maintained  | :material-check-all:  |