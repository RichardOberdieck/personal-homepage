---
title: Trust but verify - how to test optimization models
description: Finally! My favorite optimization topic gets the light of day. Why is testing optimization models hard? Why is it important? And how to do it? Let's find out!
authors: [oberdieck]
slug: testing-optimization
date: 2026-03-15
image: trust-but-verify.png
comments: true
categories:
    - Modeling
---

![cover](testing-optimization/trust-but-verify.jpg)

## TL; DR

😓 Testing optimization models is difficult because the model only makes sense as a whole. Testing individual constraints or variables adds little to no value.

✂️ Separate out logic such as set generation for constraints so that they can easily be tested.

🧪 Use property-based testing (e.g. with `hypothesis` in Python) for the model code.

👨‍💼 The properties for the property-based testing should be provided directly by the business, e.g. "the solution should always have 4 routes to CPH".

🤖 LLMs do a great job of writing boilerplate hypothesis code, use them! 

<!-- more -->

## Background

I have been thinking about how to test optimization code for almost 10 years at this point. To say that I have a fascination for this topic is a severe understatement. Why? Glad you asked!

### OR people write code, but never learn it

All of us **have** to write code. To quote [Paul Rubin](https://orinanobworld.blogspot.com/):

> Unless you magically find yourself endowed with limitless programming support, sooner or later (more likely sooner) you are likely to need to do some coding.

Having said this, we are never *taught* the craft of software engineering. How to structure code, versioning, SOLID principles and, yes, testing.

The result? Code written by OR people often is completely or mostly untested. Even if it is, there often is no systematic approach as to *what* should be tested how.

This needs to change.

### It's important

Testing is [crucial](https://blog.stackademic.com/the-spring-boot-horror-story-ill-never-forget-44a7db15d94e). So this is not a nice-to-have, but a fundamental part to ensure correctness. Without it, we are "flying blind". And software is never done. So if the optimization model code is not tested, we can never make any change and be confident it still works as expected. This means slow to no development, which is bad.

### It's not trivial

In principle, testing is easy: for a given function/class, **arrange** the system[^1] to specific starting condition, trigger the **action** to be tested, and **assert** that the result is as expected:

[^1]: This is often called the "system under test".

This works really well for a lot of code. Even the most complex algorithms can (and should) be broken down into small components, and then each can be tested. However, this is tricky for **optimization model** code, i.e. code that encodes a mathematical model. Here is the example we're going to be using:

> Given an offshore wind farm, connect the turbines such that the power from all turbines is collected but the cost of the cable layout is minimized.

$$
\begin{array}{lll}
\text{minimize} & \sum_{c} p_cx_c \quad & \text{(Minimize cost of cables)} \\
\text{subject to} & x_{\text{out}} = 1 & \text{(All turbines connected)} \\
& f_{in} + P = f_{out} & \text{(Flow balance on each turbine)} \\
& f_c \leq M_cx_c & \text{(Flow limit for each cable type)} \\
& x_i + x_j \leq 1 & \text{(Cables cannot cross)} \\
& \sum_{c} z_c \leq L & \text{(Max number of cable types)}
\end{array}
$$

!!! note
    I omitted the indices, just because I can. If you want the full details on this and much more, I suggest you read Martina Fischetti's [thesis](https://orbit.dtu.dk/files/158908591/PhDThesis.pdf).

In pseudo-code, this may look something like this:

```python
define_variables()

map_connections_to_links()
limit_flow_on_a_link()
ensure_flow_balance_for_unit()
enforce_link_being_built_for_unit()
ensure_cable_selection()
limit_number_of_cables()
add_non_crossing_constraints()

optimize()
```

Why is it tricky? Because the model only makes sense as a whole. Take the `ensure_flow_balance_for_unit()` function: as you expect, it adds a constraint related to a flow balance. So how would you test it? If you give the function 2 possible incoming connections $x_0$, $x_1$ and one possible outgoing connection $x_2$, it adds $x_0 + x_1 + P = x_2$ to a `model` object. But:

- The `model` object is often not easily inspected
- You essentially test the modeling library you are using, which is bad practice
- Not so applicable in this case, but often several constraints (or even objectives) have to all be added to achieve a given behavior (e.g. if you are describing an epigraph)

This is my first principle:

!!! tip "Principle"
    Optimization model code needs to be tested as a whole.

## How to structure code involving optimization models

Optimization models consist of four components:

- The variables
- The equations
- The (sub)sets of variables to use for each of the components in the equations
- The parameters/coefficients for each equation 

All four of these components can involve "logic", that is, some form of calculation. And this calculation should be tested.

But we have to test model code as a whole, which is tricky. This leads to my second principle:

!!! tip "Principle" 
    Logic should be moved out of the optimization model code as much as possible.

What does this mean in practice? Say you need to apply a constraint only to a subset of the variables. You could write:

```python
for cable_type in cable_types:
    for c in connections:
        if c.cable_type == cable_type:
            model.add_linear_constraint(x[c] <= z[cable_type])
```

Looks simple enough, no? Yes, but there is logic in the `if` statement. And because of the way this code is written, you cannot test that the `if` statement "does its job" without setting up a model, and running all of it. This makes testing a lot harder. Instead, you should do the following:

```python
def get_connections_with_same_cable_type(connections, cable_type):
        return {c for c in self.connections if c.cable_type == cable_type}

for cable_type in cable_types:
    for c in get_connections_with_same_cable_type(connections, cable_type):
        model.add_linear_constraint(x[c] <= z[cable_type])
```

Now, this is a bit extreme in this particular case. Even I may not do this all the time. However, it illustrates the point: when you factor out the `if` statement, we have the following function `get_connections_with_same_cable_type()`:

**This function does not know anything about optimization!** This means we can write good old unit tests for this, no need to worry about modeling languages, variables or anything else. And we reduced the logic tied up in the model code.

The same principle applies for logic involving parameters or coefficients: suppose you need to calculate the length of a cable for the objective function of your model. You could write:

```python
objective = 0
for c in connections:
    distance = np.sqrt((c.origin.x - c.destination.x) ** 2 + (c.origin.y - c.destination.y) ** 2) / 1000
    objective += distance * cost_per_km * x[c]
```

However, now the distance calculation cannot be tested independently. This means that any mistakes made will only be found by building and solving the entire model. Instead, following the same principle, we can refactor this:

```python
objective = xsum(get_cost(c)*x[c] for c in connections)
```

and then:

```python
def get_cost(c) -> float:
    return cost_per_km * np.sqrt((c.origin.x - c.destination.x) ** 2 + (c.origin.y - c.destination.y) ** 2) / 1000
```

Now, the function `get_cost(c)` can be tested without knowing anything about optimization. And by moving out the code we reduced amount of logic in the optimization model code.

Essentially, in its cleanest form, the model code should only consist of equations with pre-calculated sets and coefficients.

!!! note
    What about callbacks? In most modeling languages, callbacks are actually just a function anyways, so they can be tested without any issue. The only thing is that one has to create a proper mock object to inject.

## How to test?

Now everything is in place: we have isolated the model code to its lowest logic complexity, and are ready to start testing. But we need to test everything together. So how should that look like?

In its purest form, a mathematical model is a mathematical representation of a business problem. Therefore, the best test on whether the model does what it should is whether it represents the original problem as expected. Consider the example from earlier: the business problem may state that a maximum number of cables `max_number_of_cable_types` should be used. Programmatically, this would look something like this:

```python
max_number_of_cable_types == len(set([c.cable_type for c in layout]))
```

Note that also this test **knows nothing about optimization!** It takes a solution and checks whether the solution behaves as expected. The beauty of this is that such a constraint can (almost) be written by a business person. They may say "we need to ensure that any solution fulfills the cable limit requirement", and then this is the code representation of that sentence. So by having this test, we implicitly ensured that the mathematical representation we chose solves the original business problem.

!!! tip "Principle"
    A model encodes a business problem. Test solutions against business rules which do not know anything about optimization. Whenever possible, these rules should be defined by "the business".

It goes even further than that: should we chose to change variables, add new constraints, strengthen the formulation etc. we can still rely on the same tests. This allows us to make changes **confidently**, the hallmark of a good test.

But how can we "ensure that any solution" has a given property? Let me introduce you to **property-based testing**.

### Property-based testing

<iframe width="560" height="315" src="https://www.youtube.com/embed/mkgd9iOiICc?si=lkoduXf-uq5527ul" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Classically, a unit test is based on a single instance. Take the distance calculation from before: a normal unit test would look something like this:

```python
def test_get_distance(c):
    # Arrange
    origin = {'x': 0, 'y': 0}
    destination = {'x': 1000, 'y': 0}

    # Act
    distance = get_distance(origin, destination)

    # Assert
    assert distance == 1
```

Given the complexity of almost all mathematical models, one would need to create hundreds of test cases to cover all possible combinations and create the confidence we are looking for. But what if we could *automatically generate* those test cases?

This is the premise behind property-based testing. Instead of hard-coding the instance, they are generated automatically. In Python, [`hypothesis`](https://github.com/HypothesisWorks/hypothesis) is the most commonly used tool for this purpose. To stay with the cost example, a valid `hypothesis` test would look something like this:

```python
from hypothesis import given, strategies as st


def value():
    return st.floats(min_value=-1e6, max_value=1e6, 
                                    allow_nan=False)

@given(
        o=st.fixed_dictionaries({'x': value(), 'y': value()}),
        d=st.fixed_dictionaries({'x': value(), 'y': value()})
)
def test_get_distance(o, d):
            distance = get_distance(o, d)
            assert distance >= 0
```

So now we test that any distance we get is non-negative. The advantage is clear: by having a tool doing the work for you, instance generation is extremely easy. The downside? You need to write a good instance generation setup, which needs to fulfill two key criteria:

1. The test instances need to be sensible, i.e. they cannot all be infeasible or trivial solutions.
2. The instances need to be small enough so that they can be solved quickly

How do we get there?

![ai_gif](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExZGdla3p5bzBxaWFnOHNoY2VnZjhqa3c5YmF0cnRwanNrbjJocDF0ZSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/zN5xfuJBtbpnRDZncm/giphy.gif)

Thank the AI gods for LLMs: by parsing your code as context to the LLM of your choice, you can easily prompt it to provide really good `hypothesis` code. And once that it is in place, all you need to do is implement the properties you would like to test.

Sounds abstract? Here is an example from [this repo](https://github.com/RichardOberdieck/opti_test/tree/main) where I used this approach to optimize the array cable layout of a wind farm:

```python
@given(model_data=model_data_st())
@settings(deadline=None, max_examples=500)
def test_max_number_of_cable_types(model_data):
    # Arrange
    parameters = Parameters(mw_produced_per_turbine=8, max_number_of_cable_types=1)
    model_builder = ModelBuilder(model_data, parameters)

    # Act
    layout = model_builder.solve()

    # Assert
    if layout is None:  # Account for possible infeasibilities
        return
    assert 1 == len(set([c.cable_type for c in layout]))
```

In this code, I set the `max_number_of_cable_types=1`, and afterwards test that this specific setting is indeed respected. The optimization happens in the `model_builder.solve()` step.

I encourage you to look at that repo to get a concrete sense of how this looks in practice. So my last principle is this:

!!! tip "Principle"
    Use property-based testing for testing optimization model code. The properties should be expected solution properties, and can very well be defined by the business.

### Other testing approaches

What are the alternatives? Well, one could try to test the constraints individually. I did this for fun once, where I checked that the constraints were added as expected to the model. However, this is a lot of work and essentially only tests that whatever modeling interface you use is doing its job. 

Another approach is to write out past instances as LP files (sometimes called "golden LP files"), as well as the expected objective function values etc. Then by loading those files and solving them, one can test that those values are still found. My opinion is that this is a really good basis for performance testing, which I will cover in a (shorter) blog post soon. For instance, it is an excellent test to check whether a new version of a solver, or switching to a different solver, will yield the expected results.

However, it does not actually test correctness. Suppose you have a bug in the code that calculates the coefficients of the objective function. Then the past results will be as wrong as the current ones. Similarly, suppose you need to change the objective function because of a business requirement. Then you need to either overwrite the expected objective function values of the LP files (which defeats the point of the test) or delete the files.

Having said all of this, storing past instances is crucial and should always be done. Be it for version upgrades, performance testing, detecting drift. However, they do not replace the need to test for correctness.

## Final thoughts

Testing is like exercise: we all know we should do it, and the more you do it, the easier it gets. Testing optimization model code is not easy. However by being disciplined about it, it is relatively straightforward and highly value adding.

As I said (and you can probably tell by now if you made it this far), I am very passionate and opinionated about this topic. And as [Tim Minchin said](https://youtu.be/yoEezZD71sc?si=MAnmXN0Ubg6cGXWb):

> A famous Bond motto states that opinions are like arseholes, in that everyone has one. There is great wisdom in this but I would add that opinions differ significantly from arseholes in that yours should be constantly and thoroughly examined.

So please, challenge my statements. Discuss. The more our community talks about this topic, the better. I don't claim to have found the magic bullet. This post is simlpy a distillation of countless hours of thought and struggle with this topic. 