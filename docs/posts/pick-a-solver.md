---
title: How should I benchmark optimization solvers?
description: What's the best way to compare solvers? What should I measure and how? Should I care about the hardware? Let's find out!
slug: pick-a-solver
date: 2023-07-06
comments: true
categories:
    - Optimization
---

![cover](pick-a-solver/thinking.png)

## TL;DR

😈 Speed is the most important factor when it comes to mathematical optimization solvers

🫘 Benchmarking requires **three things**: reliable hardware, representative test data, robust testing setup

🗄 Organize your models as .mps or .lp files in a folder, and solve each model using a CLI with multiple random seeds

🚀 When done right, a benchmarking exercise will **improve the overall testing and deployment** of your optimization project

<!-- more -->

## Let's talk about performance

![speed-gif](https://tenor.com/view/speed-i-am-speed-meme-kid-gif-14031709.gif)

The performance of your model depends on three things:

- Hardware
- Test data
- Code/Test setup

## Hardware

![hardware-meme](https://i.imgflip.com/38ewu1.jpg)

From 1991 to 2015, [machines have improved an estimated 1.3M x](https://hepg.hks.harvard.edu/files/hepg/files/panel_1_r_bixby.pdf). On top of that, we have multi-threading, bus speed improvements, system-on-a-chip (SOCs), all of which make your code run faster.

> Just to give you an idea how much improvement this actually is: \
**10 seconds now = 5 months in 1991**

This means: it matters where your code runs! Different machines can and will produce vastly different runtimes, especially in the cloud:

- A general rule of thumb: **Be Close To Prod (BCTP)**, i.e. benchmark on hardware that you want to run on later in production.
- Remember: in the cloud, you don't know the actual hardware your models run on, so you need to be extra careful. This means doing multiple runs at different times of the day, ideally in different availability zones and regions.
- Aim to be as close to metal as possible: the more abstraction layers (VM, Docker, Kubernetes etc.) you have, the more noise you introduce.

## How to cheat in benchmarks: the test data

![data-meme](https://memecreator.org/static/images/memes/5132283.jpg)

The first rule of benchmarking is: **your test set defines your test results**

The second rule of benchmarking is: **your test set defines your test results**

The good news is that you *should* have good test data anyways, because we all test our applications very thoroughly, right? 🫣

This is the most "helpful" part of doing benchmarking. If you do it right, you will have a really good starting point to track the performance of your application. Whoop!

What does that mean in practice?

- As many *distinct* instances as possible (at least > 5, ideally > 15)
- As many *diverse* instances as will be seen in real life: from the smallest to the largest, covering all the different combinations of settings you may have in your code

> But what if I only have a single, or maybe two problems? Then you can either (a) work very hard to get more, or (b) be acutely aware of the fact that this introduces huge statistical bias. If a solver has a heuristic that just happens to work great for this one instance, it might fool you. Look out.

## Code/Testing setup

What do you actually test? Broadly speaking, there are two options:

1. [Preferred] Pure solve: you provide an [.mps](https://en.wikipedia.org/wiki/MPS_(format)) or [.lp](https://www.gurobi.com/documentation/current/refman/lp_format.html) file to the solver, it solves it and you measure the output.
2. Solve in code: you run some code, which builds and runs the model, and measure the output.

The main difference is that in option 1, the output (be it time or optimality gap) is exclusively produced by the solving process, while in option 2 we have an additional layer on top of the solving process itself.

![docker-birth-meme](https://external-preview.redd.it/aR6WdUcsrEgld5xUlglgKX_0sC_NlryCPTXIHk5qdu8.jpg?auto=webp&s=5fe64dd318eec71711d87805d43def2765dd83cd)

This means you need to make sure you have the same versions, database connections, latencies etc.

## Getting Things Done: the actual benchmarking

![gtd-meme](https://memecreator.org/static/images/memes/5182233.jpg)

If your benchmark only involves solving a single model, then best workflow for benchmarking is as follows:

1. Put all your model files in a folder
2. Run each file in that folder with all the solvers you want to test using a command line interface with 3-5 different random seeds
3. Save the log files and extract the info about the runtimes.

An example on how this is done is the famous [Mittelmann benchmarks](https://plato.asu.edu/bench.html) where you can actually see [all the logs](https://plato.asu.edu/ftp/milp_log8/) etc.

> It is really important to run all benchmarks with multiple [random seeds](https://en.wikipedia.org/wiki/Random_seed) (minimum 3, ideally 5). This way, you can average out the noise of [pseudo-random number generators](https://en.wikipedia.org/wiki/Pseudorandom_number_generator). Leave a comment if you want to have a post about this topic in detail.

### But what if my setup is not as simple?

I've seen quite a few cases where the benchmarking could not be done this easily. A common example is if you have to solve multiple optimization problems in a row, and the output of one is the input of another.

In those cases, you need to write code to get the benchmark done. Also, this type of benchmarking does explicitly not include model building times. If those are large (> 20%) in your model, then this should be part of your benchmark as well.

### Should I tune the solvers?

![tune-meme](https://1.bp.blogspot.com/_jdlVKzYJWXk/TQdbVVZ_tfI/AAAAAAAAAy0/ib6DXlaGG9E/s1600/tehoo_loytyy.jpg)

All solvers have parameters that can be tuned: tolerances, algorithm choices or how much time is spent in different sections, etc. This affects the solve time, sometimes on a little, to a lot (no solution to finding a solution even). So this begs the questions:

1. How do you tune? I'll make a separate blog post about this, so stay tuned :)
2. Should I compare solvers tuned or un-tuned?

The practical answer is that you should do what you have time for. When you have a lot of experience, you will be able to tell quite quickly whether there are obvious things to be done, but even then, tuning can be very time consuming.

Some solvers offer that they will tune for you, and I would always accept this offer, even if it is just to see what is possible. Suppose that solver A tunes your model for you, and tells you that primal simplex is the way to go for the root relaxation. You can then use that information with solver B and see whether it makes a difference as well.

Some parameters do not translate between solvers. However, even that is valuable as this may then be a competitive advantage for that solver.

## Conclusion

Benchmarking is hard. But there is a science and a craft behind it. So for as long as you have good test data, keep your hardware setup as clean as possible, and test the right parts of your code with sufficient levels of random seeds, you will get a reliable answer out.

And I think it's fun! It's like going to a race track, but you get to play with the cars. And then sit back, and find out who's the best for you.

<script data-goatcounter="https://oberdieck.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>