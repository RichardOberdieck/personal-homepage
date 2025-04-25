---
title: pydantic validates your inputs, or does it?
description: Sometimes pydantic does not behave the way you expect. To me, this is unexpected behavior, but hey, what do I know...
slug: pydantic-fun
date: 2025-04-25
comments: true
categories:
    - Python
---

![cover](pydantic-fun/cover.jpg)

## TL;DR

🎊 `pydantic` calls itself [the most widely used data validation library for Python](https://docs.pydantic.dev/)

😧 Yet, it does *not* validate default values, nor assignments by default. That's ... unexpected

<!-- more -->

## Background

`pydantic` is a Python library that tries to bring type-safety to Python. Rather than type *hints* (it may be true, but doesn't have to be), `pydantic` will validate an input. An example:

```
from pydantic import BaseModel
 
class Test(BaseModel):
    foo: str
 
t = Test(foo=[1,2,3])  # Raises a ValidationError
```

## Let's have some fun with `pydantic` validation

However, this works without problem:
 
```
from pydantic import BaseModel
 
class Test(BaseModel):
    foo: str = [1,2,3]
```
 
Equally, this also works:
 
```
from pydantic import BaseModel
 
class Test(BaseModel):
    foo: str
 
t = Test(foo="1")
t.foo = [1,2,3]
```
 
Why? Because `pydantic` argues that this is performance plus backwards compatibility, at least according to [this Github issue](https://github.com/pydantic/pydantic/issues/9389). I disagree. The reason to use `pydantic` is type-safety.
 And being able to violate the type-safety in the *default settings*  is not expected behavior.
 
Alas, there is a setting so solve it:
 
```
class Test(BaseModel, validate_default=True, validate_assignment=True):
    foo: str
```
 
As you would expect, the `validate_default` validates the default values, while `validate_assignment` validates the assignment.