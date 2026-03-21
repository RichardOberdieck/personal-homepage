# Tech Talks

While I was at [Gurobi](https://www.gurobi.com/) from 2020 to 2023, [Ed Klotz](https://youtu.be/zoUWrwd4EGc?si=ieVEl-i6R8bdL5wV) and I created the Tech Talks, with [Kostja Siefen](https://www.linkedin.com/in/kostjasiefen/) providing moderation for most of the time. The premise was simple:

> Do a deep dive into a technical topic we find interesting that is appealing to beginners and experts.

Below you can find the list of all the tech talks we did with a brief description. But before I do that, I would like to give a big shoutout to Ed.

## Ed Klotz

Ed, or EdK as he is also known, in order to distinguish him from [Ed Rothberg](https://www.linkedin.com/in/ed-rothberg-2182871/), aka EdR, aka the "RO" in GUROBI, is the most impressive person I have worked with at Gurobi. And that is saying something.

By impressive I mean that he impressed me. Constantly. In some of these tech talks it may look as if we are somewhat of an equal, but we are definitely not. His ability to speak clearly and with ease at a high-level, introductory level, as well as at the deepest expert level, is extraordinary. He can give you a great, "simple" definition of the simplex algorithm while then diving into the depth of why a certain LP relaxation will be not as good due a detail in an LP file that only he would spot. If you ever have the chance to attend a talk by Ed, do it!

> Fun fact: While EdK did his PhD (with, of course, [George Dantzig](https://en.wikipedia.org/wiki/George_Dantzig)), he drove up to Berkeley once to listen to the (maybe) first talk of [
Narendra Karmarkar](https://en.wikipedia.org/wiki/Narendra_Karmarkar) on his new algorithm to solve linear programming algorithms in polynomial time. Today, we know those methods as interior point methods.

But enough about this, let's get to the talks.

!!! note
    The first two or three tech talks were not recorded (on purpose), and so they are not listed here.

##  Visualization Tools: grblogtools

<iframe width="560" height="315" src="https://www.youtube.com/embed/wbg4Zr_A1s8?si=ZtBlNDxpq71Jw3Xb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This talk was a bit unusual as it focussed more on something Gurobi-specific, which we generally tried to avoid. It was also special because [Matthias](https://www.linkedin.com/in/matthias-miltenberger/) led most of the tech talk. The point was though to show that:

- There is a lot of information in the log output of a solver
- There is "logic to the madness"
- With some clever regex you can parse this log into dataframes, which then allow you to use the full power of Python on it.

The project has since been renamed to [gurobi-logtools](https://github.com/Gurobi/gurobi-logtools) and is still free and open source!

## A Practical Tour through Non-Convex Optimization

<iframe width="560" height="315" src="https://www.youtube.com/embed/j1i3ebGCrP4?si=Ha-Bsk85iGkneXWS" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This is the tech talk that took the most amount of work, and is maybe our best. I recently heard that it is used in at least one class as the recommended introduction to non-convex optimization! What makes it so special? Well, we built a fully functioning spatial branch-and-bound solver. And then we used the fact that we had the code to build [**accurate** visual representations of the spatial branching process](https://youtu.be/j1i3ebGCrP4?si=Dch-p2hSU-HqxTGU&t=2881).

## Holiday Edition: Santa's Bag of Interesting & Unusual Optimization Applications

<iframe width="560" height="315" src="https://www.youtube.com/embed/sgcPVGsB8VE?si=ihGR8suawkqz8HI5&amp;start=2881" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

What a fun one! We looked to "gamify" optimization a bit which was hilarious. First, Ed showed off his own creation of the matrix game, where the aim was to match a target matrix with some "template ones". Next, I showed a problem that I still think is surprisingly hard, which is to find the presentation of a Sudoku that requires the least amount of numbers but still is unique. Finally, we looked at a few games and books that we both enjoyed, including the wonderful [cutting plane game](https://bigkwii.github.io/cutting-plane-game-2-web-test/).

## Converting Weak to Strong MIP Formulations

<iframe width="560" height="315" src="https://www.youtube.com/embed/e4h4ZUlUpR8?si=gxYqZK5tnERo_ovD&amp;start=2881" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Probably our most watched tech talk. The aim was to demystify the concept of a "strong" MIP formulation versus a "weak" one:

- Weakness refers to the fact that the solution of the LP relaxation provides a (loose) dual bound that is not easily closed.
- This means the tree needs a to spend a lot effort trying to repair the solution.
- Typically, this is because relaxing integrality gives access to unrealistically favorable objective function values.

So what to do? Typically, you have three options: add cuts, change the formulation completely, or change the algorithm (e.g. use Benders decomposition).

## Converting Weak to Strong MIP Formulations Part 2

<iframe width="560" height="315" src="https://www.youtube.com/embed/YxqykMu083k?si=ZfWAmPYiF6QPenf8&amp;start=2881" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The only "part 2" we ever made. It was just too big of a topic to cover in a single hour. We decided to do it because:

- There was a lot of interest in Part 1.
- We felt that we could add to Part 1 by providing some real-world examples on where such weakness made dramatic difference in the time (or even ability) to solve a model to proven optimality.

This time the wonderful [Steve Edwards](https://www.linkedin.com/in/steven-edwards-81a2406b/) joined us to also share one of his "war stories" on this topic. 

> Ed and I put these two Tech Talks together into a [book chapter](https://link.springer.com/chapter/10.1007/978-981-99-5491-9_4), which I at least think came out quite nice.

## Holiday Edition: Prevent the Grinch from Stealing your Optimization Projects

<iframe width="560" height="315" src="https://www.youtube.com/embed/8e_FnR_oP1k?si=bZicFg8iFk9cTkXj&amp;start=2881" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

We had been wanting to do a tech talk on "common problems in optimization", and Ed has a soft spot for the Grinch, so we went all-in. It was relatively "easy" to prepare, but only because the three of us all earned our scars with each one of these "Grinches". It's really a good introduction to "what can possibly go wrong", optimization edition.
