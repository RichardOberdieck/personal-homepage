---
title: How to test optimization models
description: Finally! My favorite optimization topic gets the light of day. Why is testing optimization models hard? Why is it important? And how to do it? Let's find out!
authors: [oberdieck]
slug: testing-optimization
date: 2025-09-28
image: experiment.png
comments: true
categories:
    - Modeling
---

![cover](testing-optimization/experiment.png)

## TL; DR

Testing optimization models is difficult because the model only makes sense as a whole. Testing individual constraints or variables adds little to no value.

Separate out logic such as set generation for constraints so that they can easily be tested.

Use property-based testing (e.g. with `hypothesis` in Python) for the model code.

The properties for the property-based testing can be provided directly by the business, e.g. "the solution should always have 4 routes to CPH".

LLMs do a great job of writing boilerplate hypothesis code, use them!

<!-- more -->

## Background

I have been thinking about how to test optimization code for almost 10 years at this point. To say that I have a fascination for this topic is a severe understatement. Why? Glad you asked!

### OR people write code, but never learn it

All of us **have** to write code. To quote Paul Rubin:

> At some point write #TODO

Having said this, we are never *taught* the craft of software engineering. How to structure code, versioning, SOLID principles and, yes, testing.

The result? Code written by OR people often is completely or mostly untested. Even if it is, there often is no systematic approach as to *what* should be tested how.

This needs to change.

### It's important

Testing is [crucial](). So this is not a nice-to-have, but a fundamental part to ensure correctness. Without it, we are "flying blind". And software is never done. So if the optimization model code is not tested, we can never make any change and be confident it still works as expected. This means slow to no development, which is bad.

### It's not trivial

In principle, testing is easy: for a given function/class, **arrange** the system[^1] to specific starting condition, trigger the **action** to be tested, and **assert** that the result is as expected:

[^1]: This is often called the "system under test".

This works really well for a lot of code. Even the most complex algorithms can (and should) be broken down into small components, and then each can be tested. However, this is tricky for **optimization model** code, i.e. code that encodes a mathematical model:

$$
a+b = 2
$$

In code, this may look something like this:

```python
x = generate_variables(model)

# Constraints
add_maxscout_constraint(model, x)
add_max_1_session_constraint(model, x)
add_max_1_of_overlapping_sessions_constraint(model, x)
add_unavailable_time_constraint(model, x)
add_age_constraint(model, x)
add_outofcamp_activities_constraint(model, x)

add_objective(model, x)

status = model.optimize(max_seconds=300)
```

Why is it tricky? Because the model only makes sense as a whole. Take the `add_age_constraint(model, x)` function: as you expect, it adds a constraint related to age. So how would you test it? If you give the function 3 variables $x_0$, $x_1$ and $x_2$, it adds $x_0 + x_1 + x_2 geq 18$ to the `model` object. But:

- The `model` object is often not easily inspected
- You essentially test the modeling library you are using, which is bad practice
- Not so applicable in this case, but often several constraints (or even objectives) have to all be added to achieve a given behavior (e.g. if you are describing an epigraph)

This is my first statement:

!!! note
    Optimization model code needs to be tested as a whole

## How to structure code involving optimization models

Optimization models consist of four components:

- The variables
- The equations
- The (sub)sets of variables to use for each of the components in the equations
- The parameters/coefficients for each equation 

All four of these components can involve "logic", that is, some form of calculation. And this calculation should be tested.

But we have to test model code as a whole, which is tricky. This leads to my second statement:

!!! note
    Logic should be moved out of the optimization model code as much as possible.

What does this mean in practice? Say you need to apply a constraint only to a subset of the variables. You could write:

```python
for s in selections:
    for s1 in selections:
        if s.scout_group == s1.scout_group and s.activity != s1.activity:
            model += x[s] + x[s1] <= 1
```

Looks simple enough, no? Yes, but there is logic in the `if` statement. And because of the way this code is written, you cannot test that the `if` statement "does its job" without setting up a model, and running all of it. This makes testing a lot harder. Instead, you should do the following:

```python
for s in selections:
    for s1 in get_selections_with_same_group_but_different_activities(selections, s):
        model += x[s] + x[s1] <= 1
```

Now, this is a bit extreme in this particular case. Even I may not do this all the time. However, it illustrates the point: when you factor out the `if` statement, we have the following function:

```python
def get_selections_with_same_group_but_different_activities(selections: list[Selection], s: Selection) -> list[Selection]:
    return [s1 for s1 in selections if s.scout_group == s1.scout_group and s.activity != s1.activity]
```

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
    What about callbacks? In most modeling languages, callbacks are actually just a function anyways, so they can be tested without any issue. The only thing is that one has to create a proper mock object to inject. Check out [this repo]() where I show how to do mocking in Python.

## How to test?

Now everything is in place: we have isolated the model code to its lowest logic complexity, and are ready to start testing. But we need to test everything together. So how should that look like?

In its purest form, a mathematical model is a mathematical representation of a business problem. Therefore, the best test on whether the model does what it should is whether it represents the original problem as expected. Consider the example from earlier: the business problem may state that an age limit needs to be respected. Programmatically, this would look something like this:

```python
for s in optimal_selections:
    assert s.age <= age_limit
```

Note that also this test **knows nothing about optimization!** It takes a solution and checks whether the solution behaves as expected. The beauty of this is that such a constraint can (almost) be written by a business person. They may say "we need to ensure that any solution fulfills the age limit", and then this is the code representation of that sentence. So by having this test, we implicitly ensured that the mathematical representation we chose solves the original business problem.

It goes even further than that: should we chose to change variables, add new constraints, strengthen the formulation etc. we can still rely on the same tests. This allows us to make changes **confidently**, the hallmark of a good test.

But how can we "ensure that any solution" has a given property? Let me introduce you to **property-based testing**.

### Property-based testing

Arjun vid here

Classically, a unit test is based on a single instance. Take the cost calculation from before: a normal unit test would look something like this:

```python
def test_get_cost(c):
    # Arrange
    c.origin = Coordinate(x=0, y=0)
    c.destination = Coordinate(x=1000, y=0)

    # Act
    cost = get_cost(c)

    # Assert
    assert cost == 12  # Assumes a cost_per_km of 12
```

Given the complexity of almost all mathematical models, one would need to create hundreds of test cases to cover all possible combinations and create the confidence we are looking for. But what if we could *automatically generate* those test cases?

This is the premise behind property-based testing. Instead of hard-coding the instance, they are generated automatically. In Python, [`hypothesis`]() is the most commonly used tool for this purpose. To stay with the cost example, a valid `hypothesis` test would look something like this:

```python

```

So now we test that any solution we get is divisible by 12, which is the `cost_per_km`. The advantage is clear: by having a tool doing the work for you, instance generation is extremely easy. The downside? You need to write a good instance generation setup, which needs to fulfill two key criteria:

1. The test instances need to be sensible, i.e. they cannot all be infeasible or trivial solutions.
2. The instances need to be small enough so that they can be solved quickly

How do we get there?

[Insert AI GIF]

Thank the AI gods for ChatGPT: by parsing your code as context to your LLM of choice, you can easily prompt it to provide really good `hypothesis` code. And once that it is in place, all you need to do is implement the properties you would like to test.

Sounds abstract? Here is an example from [this repo]() where I used this approach to optimize the array cable layout of a wind farm:

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

I encourage you to look at that repo (as well as [this one]()) to get a concrete sense of how this looks in practice. So my last statement is this:

!!! note
    Use property-based testing for testing optimization model code. The properties should be expected solution properties, and can very well be defined by the business.

### Other testing approaches

What are the alternatives? Well, one could try to test the constraints individually. I did this for fun once, where I checked that the constraints were added as expected to the model. However, this is a lot of work and essentially only tests that whatever modeling interface you use is doing its job. 

Another [approach]() is to write out past instances as LP files, as well as the expected objective function values etc. Then by loading those files and solving them, one can test that those values are still found. My opinion is that this can be a really good test for models where (a) the solution times are not very high and (b) the rate of change in the code is low. For instance, it is an excellent test to check whether a new version of a solver, or switching to a different solver, will yield the expected results.

However, it does not actually test correctness. Suppose you have a bug in the code that calculates the coefficients of the objective function. Then the past results will be as wrong as the current ones. Similarly, suppose you need to change the objective function because of a business requirement. Then you need to either overwrite the expected objective function values of the LP files (which defeats the point of the test) or delete the files.

Having said all of this, storing past instances is crucial and should always be done. Be it for version upgrades, performance testing, detecting drift. However, they do not replace the need to test for correctness.

## Final thoughts

Testing is like exercise: we all know we should do it, and the more you do it, the easier it gets. Testing optimization model code is not easy. However by being disciplined about it, it is relatively straightforward and highly value adding.

As I said (and you can probably tell by now if you made it this far), I am very passionate and opinionated about this topic. And as [Tim Minchin said]():

> Opinions are like arseholes, in that everyone has one. But I would say that they differ from arseholes significantly in that opinions should often and frequently be examined.

So please, challenge my statements in this blog post. Discuss. The more our community talks about this topic, the better. I don't claim to have found the magic bullet. This post is simlpy a distillation of countless hours of thought and struggle with this topic. 