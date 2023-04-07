
# Introduction

This is meant to be a repository of common high-throughput screening (HTS)
tasks using the python programming language. It's probably somewhere in-between
an introductory tutorial and a cookbook.

## Prerequisites 
This is going to assume some basic familiarity with python, such as what a
function is, the basic data types such as integers, lists and dictionaries. The
aim isn't to teach you python from scratch, but to teach python beginners how
they can apply their new python knowledge to help them analyse HTS data.

It's also going to assume you're somewhat familiar with typical HTS workflows
and terminology, such as multi-well plates, IC50 values, z-scores etc.


## Python

The code in these tutorials uses python 3.10 and python's [type-hints](https://docs.python.org/3/library/typing.html).
This is optional, and the code will run the same without it. I prefer it for clarity
and better IDE experience.

The code snippets are written with an aim of clarity rather than numerical stability
or handling edge-cases. Dealing with missing data or dividing by zero issues is
a separate topic.


### Libraries

The code snippets will use a few external libraries, mainly from the scipy ecosystem:

- pandas
- numpy
- matplotlib
- scipy
