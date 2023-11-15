Title: 2016 day 17: Two Steps Forward
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

## Part 1

Finding a path in a vault. At this point I wrote so many programs to find
a shortest path in a 2D grid, I could probably do it in my sleep. There is
a slight twist: a set of doors open/closed depends on the hash of path taken.
That means there's no need to store "already visited states" as I usually do
for BFS: even if I enter the same place again, different doors are open so it
is a different state. And there's really no smart way to solve it, bruteforcing
the solution is the only way.

So, at each step we get a hash of current path, we only need first 4 characters:
`hashlib.md5(i.encode()).hexdigest()[:4]`. Each character describes one door,
they also have coordinates relative to the current place (eg. 0,-1 for up).
We iterate on those: `for (door, direction) in zip(doorhash,DIRECTIONS):`
If that direction is not out of range and door is open - yield it, add to the
queue.

When we reach the end point, break the loop and check the length of the path.

## Part 2

So now we need to find shortest and longest path. That's just a simple modification.
When we reach the end point, don't break the loop, instead append the path to the list
of possible paths.

In the end, just find the shortest and longest string in the list of paths. Python's
usual min and max functions can do that with just one extra argument `key=len` - meaning
it will run `len()` function on each element and use its result as a key for comparison.

## Code

```python
#!/usr/bin/python3

import hashlib
from dataclasses import dataclass
from typing import Tuple
from collections import deque

PASSCODE="pslxynzg"
WIDTH=4
HEIGHT=4
DIRECTIONS=( (0,-1), (0,1), (-1,0), (1,0) ) 
DIRNAME={ (0,-1):"U", (0,1):"D", (-1,0):"L", (1,0):"R" }


def gethash(i: str) -> str:
    return hashlib.md5(i.encode()).hexdigest()[:4]

def door_open(char: str) -> bool:
    return char in "bcdef"

@dataclass
class Gamestate:
    x: int = 1
    y: int = 1
    path: str=""

    def get_next_dir(self) -> Tuple[int, int]:
        doorhash=gethash(PASSCODE+self.path)
        for (door, direction) in zip(doorhash,DIRECTIONS):
            (deltax,deltay)=direction           
            if self.x+deltax<1 or self.x+deltax>WIDTH:
                continue
            if self.y+deltay<1 or self.y+deltay>HEIGHT:
                continue
            if door_open(door):
                yield direction


def get_paths(start_point: Gamestate, end_point: Tuple[int, int], pathlist: list) -> None:
    to_check=deque()
    to_check.append(start_point)
    while to_check:
        current_point=to_check.popleft()
        for dir in current_point.get_next_dir():
            newpath=current_point.path+DIRNAME[dir]
            newx=current_point.x+dir[0]
            newy=current_point.y+dir[1]
            if newx==end_point[0] and newy==end_point[1]:
                pathlist.append(newpath)
            else:
                newpoint=Gamestate(newx, newy, newpath)
                to_check.append(newpoint)


start=Gamestate()
end=(4,4)

paths=[]
get_paths(start, end, paths)
print(f"Part 1: { min(paths, key=len)  }")
print(f"Part 2: { len(max(paths, key=len))  }")
```