Title: Advent of Code
Date: 2023-04-15 17:30
Status: published
Category: info
URL:
save_as: index.html

[Advent of Code](https://adventofcode.com/) is a serious of programming puzzles published every December since 2015. Every day, from 1st to 25th, you'll get a 2-part programming challenge. It's a great way to improve your programming skills, maybe learn a new language or new tricks in the one you already know. Each year the challenges start simple, but after a few days they become, well, challenging. What's important it's they are quite fun - unlike other programming puzzles I tried in the past, they don't focus too much on maths and theoretical CS, and they are linked by a story.

## Editions

- [2015]({filename}2015.md)
- [2016]({filename}2016.md)

## Choosing a programming language

I solved most tasks with **Python**. It's the easiest choice, as the language provides useful data structures (eg. sets and 2D arrays are often needed), text processing tools (regexps, strings as arrays of characters, splitting) and mathematical functions.

Python has been evolving fast in the recent years and Linux distributions weren't always able to keep up. 
But I think some of the new features, such as dataclasses and match/case are awesome. They make the code
both shorter and more readable. I decided to use them when they seemed a good choice, but document it,
in case you're stuck with an old version. Several tasks require Python 3.7 (dataclasses), 3.8 (walrus
operator) or even 3.10 (match/case). I also drew the line at Python 3.6, which was released in 2016 and
has numerous additions, such as f-strings and variable annotations. Some programs *might* work with older
versions of Python 3, but I made absolutely no effort to ensure it.

For the extra challenge, I used **C** and **C++** for some tasks. Modern C++ actually provides much of
the same tools as Python, but in a less convenient package. C only works for the simplest tasks for me,
more complex ones would take me weeks instead of hours.

![Python](https://img.shields.io/badge/python-%3E%3D3.10-blue)
![C](https://img.shields.io/badge/C-C99-green)
![C++](https://img.shields.io/badge/C++-C++11-green)

Other modern languages like JavaScript, Ruby, Go or Rust would work too, but I don't know them well enough. And some awesome people intentionally chose wrong tool for the job and solved the challenges with Bash, Excel, SQL, m4 or even jq.

## Getting help

Is it OK to look at someone elses code? In my opinion, AoC is about two things: learning and having fun. If you're stuck, you get none.
Looking and someone's code and implementing similar solution at least allows you to learn, and then you're free to go on to other tasks

Personally, I always tried to solve the tasks by myself first and in most cases, I managed. Even then I later googled for
other solutions to see possible improvements. Every year, there are some really hard puzzles that you can't solve without
knowing the right algorithm or math trick.

One good source are Reddit threads, they generally show up as a first Google result for "advent of code year 20xx day yy". An even better source, and an inspiration for my site, is [Dazbo’s Advent of Code Walkthroughs](https://aoc.just2good.co.uk/).