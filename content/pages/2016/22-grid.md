Title: 2016 day 22: Grid Computing
Date: 2023-11-13 17:32
Status: hidden
Category: Python
Series: 2016


We have a cluster of storage nodes organized in a grid and we need to move
data from one specific node to an access node. First thing, 
data structures. I decided to have a dataclass for each node, storing position
in a grid and used/available/total disk space. Then a list of nodes (we don't
need a 2D structure as every node knows its position), input parsing with
regexps.

## Part 1

Calculate how many pairs of nodes can be used for moving data, regardless of
position in a grid, we're only interested in used/available space. Easy, just
some elementary maths.

## Part 2

On the other hand, part 2 was very hard. I couldn't even code a universal solution.
A usual BFS brute force wouldn't work as the problem space is too big. Instead I
combined the strong points of the computer: looking up data and doing simple
maths - and strong points of human: pattern recognition. First, I printed the grid.

There is only one empty node. Which means we need to move the "hole" next to the
node we're interested in. In my case, it happens to be last but one node on
the last row. So I would simply move it up (1 step for each row), except there
are also a few nodes much larger then the rest, that can't be copied. They form
a "wall" which we need to pass to the left and then go back to the right.

Finally, we need to move the data using the empty node, then move the empty node
(by going down, to the side and up again). So, moving the data by one column
takes 5 steps. Complete formula is `(emptyx+1-wallbegin)+emptyy+wallsize+(width-1)*5`
which at least makes the solution viable for other setups where the wall is in one
row, extends to the right edge and the hole is down and to the right relative to the
first node of the wall.

```python
#!/usr/bin/python3

import re
from dataclasses import dataclass

INPUTFILE="22-input.txt"

@dataclass(frozen=True)
class Node:
    x: int
    y: int
    size: int
    used: int
    available: int

rx1=re.compile(r"x(\d+)-y(\d+)\s+(\d+)T\s+(\d+)T\s+(\d+)T")

grid=[]

with open(INPUTFILE) as f:
    for i, line in enumerate(f):
        if i<=1:
            continue
        m=rx1.search(line)
        grid.append(Node( *map(int,m.groups() )))

viable=0

print(grid[2])

for nodeA in grid:
    if nodeA.used==0:
        continue
    for nodeB in grid:
        if nodeA!=nodeB and nodeA.used<=nodeB.available:
            viable+=1

print(f"Part 1: {viable}")

# print grid
width=max( node.x for node in grid )
height=max( node.y for node in grid )

gridmap=[[0 for _ in range(width+1)] for _ in range(height+1)]
for node in grid:
    gridmap[node.y][node.x]="█" if node.used>90 else "."
    if node.used==0:
        gridmap[node.y][node.x]="#"
        emptyx,emptyy=node.x, node.y
# show start and end node
gridmap[0][width]="S"
gridmap[0][0]="E"

wallbegin=min(node.x for node in grid if node.used>90)
wallsize=width+1-wallbegin

for row in gridmap:
    for d in row:
        print(d,end="")
    print("")

print(f"Width {width} height {height} empty node {emptyx, emptyy} wall size {wallsize}")

part2=(emptyx+1-wallbegin)+emptyy+wallsize+(width-1)*5
print(f"Part 2: {part2}")
```