Title: 2016 day 19: An Elephant Named Joseph
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016



Elves passing presents in a (large) circle. For part 1 - to the
next elf, part 2 - to the elf sitting opposite. Elf without
presents is eliminated.

I tried brute force first - worked for part 1, but was too slow
for part 2. So we need to work smarter.

## Part 1

I didn't know the problem described here, but I thought the name must
mean something and googled for "Joseph algorithm". Google was smart
enough to find "Josephus algorithm" for me. A standard solution for
Josephus algorithm (I'm not pretending I invented it) also worked for
part 1.

## Part 2

There are generalized Josephus solutions for finding survivor with a group
of size n with a step of k. So, if we modify k at each step to be
number_of_remaining _elves//2, that would work, right? Wrong. Well, it *should*
work, probably I made some off-by-1 errors or something like that, but I couldn't
make it right.

Instead, I checked the answer on paper for some low numbers to try and find a pattern.

```python
1 : 1
2 : 1
3 : 3
4 : 1
5 : 2
6 : 3
7 : 5
8 : 7
9 : 9
10 : 1
11 : 2
12 : 3
13 : 4
14 : 5
15 : 6
16 : 7
17 : 8
18 : 9
19 : 11
20 : 13
```

One elf keeps its present. If there are 2, fist one wins. That's just a special case that
needs to be dealt with `if n<3`. What do we have next? For 3 and 9 we're getting 3 and 9.
These numbers are powers of 3. Concidence?

So for our number elves, we need to find the nearest power of 3 that's smaller then that.
Logarithm will give us the exponent (hey, I still remember something from high school maths!)
and luckily Python has a function for calculating logarithm with an arbitrary base, not
only natural logarithm like many languages. That saved me a terrible burden of replacing a base-3
logarithm with a natural logarithm (just kidding, that would be one extra operation).

```python
base=int(log(n-1,3))
closestpower=3**base
```

Then, for the next few steps, we're just getting one element away from the nearest power of 3.
After that, something happens. We're not moving by 1, but by 2? Some more trial end error
coding got me at `2*n-3*closestpower` and I have to admit I have no idea why, must be something
to do with maths, but it works.


## Code

```python
#!/usr/bin/python3

from math import log

NUM_ELVES=3001330


def josephus(n: int) -> int:
    l = n - (1 << (n.bit_length() - 1))
    return 2*l + 1

def josephus2(n: int) -> int:
    if n<3:
        return 1
    exponent=int(log(n-1,3))
    closestpower=3**exponent
    if n-closestpower <= closestpower:
        return n-closestpower
    return 2*n-3*closestpower


print(f"Part 1, {josephus(NUM_ELVES)}")
print(f"Part 2, {josephus2(NUM_ELVES)}")
```