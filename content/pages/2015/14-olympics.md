Title: 2015 day 14: Reindeer Olympics
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

It's time for reindeer race. Reindeers spend each round either
flying or resting, have different flight speed and flight/rest cycles.
We need to find max distance travelled (part 1) and max score,
where each reindeer gets one point when he's at the first position (part 2).

Simple thing, split() for parsing input, addition and comparisons for
the main algorithm.

## New Python feature: dataclass

Python got many additions in recent years and dataclass is my favourite. It was introduced
in Python 3.7 and if you're stuck with an older version, it's worth upgrading just for this.

Simply import it: `from dataclasses import dataclass` and add decorator `@dataclass` before
the class definition. In the class, add type hints for all variables. And then, magically,
Python will create the class constructor, __repr method (used when you print your object),
__eq method (comparison operator) and several other things. It saves you from typing a lot
of boilerplate code. You can then add methods just like to a standard class, even overwrite
the automatically generated methods if they're not suitable.

But wait, there's more! There are parameters for the decorator, though not needed here I'll
decribe them briefly:

* `@dataclass(order=True)` generates method for comparison operators (<,>, <=, >=), allows to sort the objects,
* `@dataclass(frozen=True)` makes the object immutable, which among other things causes to generate a __hash function, meaning the object can be added to a dictionary or set.


## Code

```python
#!/usr/bin/python3

from collections import defaultdict
from dataclasses import dataclass

FILENAME="14-input.txt"
TIMELIMIT=2503

@dataclass
class Reindeer:
    # initial data
    name: str
    speed: int
    maxflighttime: int
    resttime: int
    # data that will change
    distance: int = 0
    cycletime: int = 0
    flying: bool = True
    score: int = 0
    
    
    def do_round(self):
        self.cycletime+=1
        if self.flying:
            self.distance+=self.speed
            if self.cycletime>=self.maxflighttime:
                self.flying=False
                self.cycletime=0
        elif self.cycletime>=self.resttime:
            self.flying=True
            self.cycletime=0


# parse input
reindeers=[]
maxdistances=defaultdict()
with open(FILENAME) as inputfile:
    for line in inputfile:
        name, _, _, speed, _, _, maxflighttime, _, _, _, _, _, _, resttime, _= line.split()
        reindeers.append(Reindeer(name, int(speed), int(maxflighttime), int(resttime)))

# do rounds
for _ in range(TIMELIMIT):
    for r in reindeers:
        r.do_round()
        maxdistances[r.name]=r.distance
    for r in reindeers:
        if r.distance==max(maxdistances.values()):
            r.score+=1

# find score
maxscore=0
for r in reindeers:
    print(r.name, r.score)
    maxscore=max(maxscore, r.score)


print("Part 1, max distance: ", max(maxdistances.values()))
print("Part 2, max score: ", maxscore)
```