Title: 2015 day 15: Science for Hungry People
Date: 2023-05-13 17:32
Status: hidden
Category: Python
Series: 2015

We're baking cookies. We have 4 ingredients which have 5 numeric properties
(capacity, durability, flavor, texture and calories) . Our cookie takes 100 spoons
of ingredients, we need to maximize the score (ignoring calories for part 1, getting
exactly 500 calories for part 2).

There are many ways to mix the ingredients, from (100, 0, 0, 0) to
(0, 0, 0, 100). Specificaly, there are 176847 ways. Problem space is mall enough
for brute force to be useful, large enough that it makes sense to optimize.

The puzzle calls for some combined combinatorics (pun intended).
First, generate all **combinations with replacement** of numbers from 0 to 100
that together add up to 100. Then, generate **permutations** of those.
Why "with replacement"? Because the numbers can repeat, we might have 20 spoons of
first ingredient and 20 spoons of the second ingredient. Why permutations?
Because the first operation disregards order, eg. if it tries 
(1, 2, 3, 94) it won't try (94, 1, 2, 3) because these are the same elements
in a different order. We need permutations for that. Both are available in the
itertools module.

First version of the code was slow, so I used tqdm module (from PyPI) to add
a progress bar. After some fixes, the program finishes in a second, so it's not
really needed, but I left it anyway. Maybe someone wants to run it on a 386?


```python
#!/usr/bin/python3

import re
from itertools import permutations, combinations_with_replacement
from tqdm import tqdm
from dataclasses import dataclass


FILENAME="15-input.txt"
MAXSIZE=100
TARGETCALORIES=500

@dataclass
class Ingredient:
    name: str
    capacity: int
    durability: int
    flavor: int
    texture: int
    calories: int


class Cookie:
    calories: int
    totalscore: int

    def __init__(self, ingr: tuple) -> None:
        global ingredients
        capacity=0
        durability=0
        flavor=0
        texture=0
        self.calories=0
        for num, amount in enumerate(ingr):
            capacity += amount*ingredients[num].capacity
            durability += amount*ingredients[num].durability
            flavor += amount*ingredients[num].flavor
            texture += amount*ingredients[num].texture
            self.calories += amount*ingredients[num].calories
        # any negative becomes 0
        capacity = max(capacity, 0)
        durability = max(durability, 0)
        flavor = max(flavor, 0)
        texture = max(texture, 0)
        self.totalscore=capacity*durability*flavor*texture


# parse input file
ingredients=[]
rx=re.compile( r"(\w+): capacity (\-?\d+), durability (\-?\d+), flavor (\-?\d+), texture (\-?\d+), calories (\-?\d+)" )
with open(FILENAME) as inputfile:
    for line in inputfile:
        name, capacity, durability, flavor, texture, calories = rx.match(line).groups()
        ingredients.append(Ingredient(name, int(capacity), int(durability), int(flavor), int(texture), int(calories)))

# calculate permutations
num_ingredients=len(ingredients)
ingr_combinations=[ i for i in combinations_with_replacement(range(MAXSIZE), num_ingredients) if sum(i)==MAXSIZE ]
print (f"Got {len(ingr_combinations)} combinations, calculating permutations of combinations")
ingr_permutations=set()
for combination in ingr_combinations:
    for i in permutations(combination):
        ingr_permutations.add(i)
print(f"Got {len(ingr_permutations)} permutations of combinations")

# bake cookies
maxscore=0
maxwithtarget=0
for recipe in tqdm(ingr_permutations):
    cookie=Cookie(recipe)
    maxscore=max(maxscore, cookie.totalscore)
    if cookie.calories == TARGETCALORIES and cookie.totalscore > maxwithtarget:
        maxwithtarget=cookie.totalscore

print("Part 1, max score: ", maxscore)
print("Part 2, max score: ", maxwithtarget)
```