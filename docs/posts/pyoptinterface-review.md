---
title: pyOptInterface - what can it do?
description: pyOptInterface just turned 1 year old, and I finally had an opportunity to take it for a spin. Let's have a look!
slug: pyoptinterface-review
date: 2025-04-09
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

It ([code](https://github.com/metab0t/PyOptInterface/tree/master), [docs](https://metab0t.github.io/PyOptInterface/index.html)) is a modeling framework in Python, i.e. a way that allows you to write optimization models and then interface with different solvers. As of April 2025, it supports Gurobi, COPT, HiGHS, Mosek and Ipopt.