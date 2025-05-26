---
title: pyOptInterface - what can it do?
description: pyOptInterface just turned 1 year old, and I finally had an opportunity to take it for a spin. Let's have a look!
slug: pyoptinterface-review
date: 2025-05-25
comments: true
categories:
    - Optimization
---

![cover](pyoptinterface-review/cover.jpg)

## TL;DR

🧮 `pyOptInterface` is a new Python modeling framework first released in April 2024 and is currently in version 0.4.1.

👏 It is well-written, and really quite fast. It has decent documentation, and also supports conic and nonlinear programming.

😕 It does however not use all the nice things Python has to offer, which makes it more painful to work with than it has to be.

⌛ With some effort, this could become my new daily driver for optimization modeling. However in its current form, the usage pains are higher than the awesome tech behind it.

<!-- more -->

## What is `pyOptInterface`?

It ([code](https://github.com/metab0t/PyOptInterface/tree/master), [docs](https://metab0t.github.io/PyOptInterface/index.html)) is a modeling framework in Python, i.e. a way that allows you to write optimization models and then interface with different solvers. As of May 2025, it supports Gurobi, COPT, HiGHS, Mosek and Ipopt. So far, so straightforward. But is it any good? And how does it compare to [`pyomo`](https://github.com/Pyomo/) or [`python-mip`](https://github.com/coin-or/python-mip)? Let's find out!

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

> I am going at this from a LP and MIP perspective, which is the vast majority of optimization use cases.

| Criteria    | `pyOptInterface`    | `pyomo`             | `python-mip`
| ----------- | --------------------| --------------------| --------------------
| FOSS (license)       | ✅ ([MPL 2.0](https://github.com/metab0t/PyOptInterface/tree/master?tab=License-1-ov-file))                | ✅ ([Custom](https://github.com/Pyomo/pyomo?tab=License-1-ov-file))                 | ✅ ([EPL 2.0](https://github.com/coin-or/python-mip?tab=EPL-2.0-1-ov-file))
| Ease-of-use | ❌                 | ❌                  | ✅
| Fast        | ✅                | ❌                  | ✅
| Maintained (last release) | ✅ ([19/3/2025](https://github.com/metab0t/PyOptInterface/releases/tag/v0.4.1))                | ✅ ([16/4/2025](https://github.com/Pyomo/pyomo/releases/tag/6.9.2))                 | ❌ ([4/1/2023](https://github.com/coin-or/python-mip/releases/tag/1.15.0))

!!! note
    I have also heard of [`feloopy`](https://github.com/feloopy/feloopy), but have not had a chance to take it for a spin so I can't give my opinion yet.

Needless to say that all of this is my own opinion, but let us dive into why I have that opinion of `pyOptInterface`.

## The good

It's free and open-source, which is critical. However, there is a caveat:

!!! warning
    The [MPL 2.0](https://www.mozilla.org/en-US/MPL/2.0/) license is a **copyleft** license. Specifically, this means that any works derived from this license also **have to be open-sourced under the same license**:

    > "All distribution of Covered Software in Source Code Form, including any Modifications that You create or to which You contribute, must be under the terms of this License." (Section 3.1)

    So if you use this code to create code that you distribute, e.g. sell, then you need to open-source that code. For fun, [here](https://github.com/edent/BMW-OpenSource?tab=readme-ov-file) is how BMW fell into that one.

However, I am strong proponent of FOSS since [the world runs on it](https://truelist.co/blog/linux-statistics/). So thumbs up from me.

Also, it seems to be maintained, although still not on v1. But I am sure that will come.

Finally, it is fast. Since it hooks into the C++ interfaces of the various solvers, it saves itself the model construction hell of `pyomo` in Python, and hence is a breeze to work with in my experience.

## The not so good

Really the only thing it falls short is its ease of use. And since I rate that quite highly, it is unfortunately sufficiently annoying for me to put the project aside for now. What do I mean? Let's a have look at some code derived from a [project of mine](https://github.com/RichardOberdieck/opti_test/blob/main/opti_test/model_builder.py) using `pyOptInterface`.

The variable building was fine:

```python
import pyoptinterface as poi
from pyoptinterface import highs, VariableDomain


model = highs.Model()

x = {c: model.add_variable(domain=VariableDomain.Binary, 
            name=f"x_{c}") for c in connections}
```

There is also an [`add_variables`](https://metab0t.github.io/PyOptInterface/common_model_interface.html#model.add_variables) and an [`add_m_variables`](https://metab0t.github.io/PyOptInterface/common_model_interface.html#model.add_m_variables), probably inspired by [Gurobi's matrix-friendly API](https://docs.gurobi.com/projects/optimizer/en/current/reference/python/func_mfunctions.html).

With this variable, you can build expressions, such as objective functions:

```python
model.set_objective(sum(c.get_cost() * x[c] for c in connections))
```

This can be done [faster](https://metab0t.github.io/PyOptInterface/expression.html), but at what cost? Let's have a look:

```python
def fast_expr():
    expr = poi.ExprBuilder()
    for i in range(N):
        expr += 0.5 * x[i] * x[i]
        expr -= x[i]
```

This may be faster, but I certainly don't want to write or maintain this.

And this brings us to the main issue: creating constraints. Here is how that goes:

```python
model.add_linear_constraint(
    sum(x[c] for c in get_connections_for_link(link)) - y[link], 
    poi.Eq, 
    0
)
```

Mathematically, this would be $\sum_{c\in C_l} x_c = y_l$ for a given $l$. So a very, very easy constraint. Yet, it is a pain to write, because:

- Everything with a variable needs to be on the left-hand side
- There is no [operator overload](https://www.geeksforgeeks.org/operator-overloading-in-python/), so we need to define the left-hand side, operator, and right-hand side separately.

How could it be done differently? Let's look how this code would look in `python-mip`:

```python
from mip import Model, BINARY, xsum


model = Model(solver_name="CBC")

x = {c: model.add_var(f"x_{c}", var_type=BINARY) for c in connections}

model.set_objective(xsum(c.get_cost() * x[c] for c in connections))

model += xsum(x[c] for c in get_connections_for_link(link)) == y[link]
```

This is way, way easier to write. And when you have to write a lot of constraints, this stuff matters.

## Conclusion

As [I've said before](https://oberdieck.dk/2024/11/12/goodbye-python-mip/), there is a big hole in the optimization landscape for a well-performing, easy to handle modeling framework. However, in its current form `pyOptInterface` is not it. However, they did a lot of things right. So with some ChatGPT, some inspiration from `python-mip` and other places and hopefully some contributors, this should be doable. I'm looking forward to it!