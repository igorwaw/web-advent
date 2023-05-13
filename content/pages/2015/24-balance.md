Title: 2015 day 24: It Hangs in the Balance
Date: 2023-05-13 17:32
Status: hidden
Category: Python
Series: 2015

## Part 1

Up to this point I wrote these documents months after writing the code. This one is different - I'm writing and coding simultanously. I'll use this opportunity to show my approach to AoC puzzles. I usually go step by step, starting with the input data. Then I think of the data structures. Then how to get a step closer towards solution. The code is often quite verbose in the beginning (eg. standard loops instead of list comprehension), later I simplify it.

First, I need to read the input data. I prepared 2 files, 24-input.txt is my input for the puzzle, 24-small.txt is a sample data from the problem description.

```python
INPUTFILE="24-small.txt"

packages=[]
with open(INPUTFILE) as f:
    for line in f:
        packages.append(int(line))
```

Now, the instruction says we need to divide packages using the following conditions:

- each container weighs the same,
- first container has the smallest possible number of packages,
- if there are several ways to satisfy the second condition, choose one with the smallest QE.

So, target weight is 1/3 of the total weight. Let's hope it divides evenly. Let's also check number of packages and total weight.

```python
weight=sum(packages)//3
print(f"Number of packages:  {len(packages)} total weight {sum(packages)} container weight {weight}")
```

An obvious solutions: brute force! Generate all (permutations? combinations? with or without replacement? I'll have to check, AS USUAL) and find the best one. But wait, we only really care about the first container, that greatly reduces the problem space.

OK, so let's check if we can satisfy the weight requirement using only one element. Then two element (permutations? combinations? I checked, order doesn't matter and there's no replacement, so it's combinations). Then 3-element combinations, and so on.

```python
has_right_weight=[]
for i in range(1,len(packages)):
    has_right_weight.extend(  x for x in combinations(packages,i) if sum(x)==target_weight )
```

But we need to have the smallest possible number of packages. So if we find that n-element combination satisfies the requirement, no need to check further. The moment that something appears in our list, break:

```python
    if has_right_weight:
        break
```

Let's check with the small dataset - only one 2-element combination, like the instruction says. With the full data - many 6-element combinations. So now we need to find one with the lowest "quantum entaglement", defined as product of the elements. Python 3.10 has a math.prod function, which saves me from an effort of writing one myself.

```python
qe= [ math.prod(x) for x in has_right_weight ]
print(f"Part 1: {min(qe)}")
```

## Part 2

So now we have to divice packages between 4 containers instead of 3. The solution is obvious: I need to put the code into a function, taking number of containers as a parameter, then call it twice.

While I'm at it, let's shorten the code. Unusually for me, I used list comprehension instead of for loops in most places because it was obvious for me. The only exception is input processing.

The result code is this:

```python
#!/usr/bin/python3

import math
from itertools import combinations

INPUTFILE="24-input.txt"


def calculate_min_qe(num_containers: int):
    target_weight=sum(packages)//num_containers
    has_right_weight=[]

    for i in range(1,len(packages)):
        has_right_weight.extend(  x for x in combinations(packages,i) if sum(x)==target_weight )
        if has_right_weight:
            break

    qe= [ math.prod(x) for x in has_right_weight ]
    return min(qe)


packages=[]
with open(INPUTFILE) as f:
    packages.extend(int(line) for line in f)

print(f"Part 1: {calculate_min_qe(3)}")
print(f"Part 2: {calculate_min_qe(4)}")
```