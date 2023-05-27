Title: 2016 day 01: No Time for a Taxicab
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

Title is a reference to a Taxi Metric, but you don't even need to know what's that.
As usual, first days of Advent of Code are simple.

We're moving on a 2D grid. We need to keep track of visited point, set is an obvious structure
for it. I used an immutable dataclass for point, though a simple tuple of 2 ints would do.

```python
#!/usr/bin/python3

from dataclasses import dataclass

INPUTFILE="01-input.txt"


#immutable data class
@dataclass(frozen=True)
class Point:
    x: int
    y: int

# read input file
with open(INPUTFILE) as f:
    line=f.readline().strip()
    directions=line.split(", ")


current=Point(0,0)
visited=set()
visited.add(current)
direction=0 # 0=N, 1=E, 2=S, 3=W
part2done=False
for i in directions:
    move=int(i[1:])
    if i[0]=="R":
        direction=(direction+1)%4
    else:
        direction=(direction-1)%4
    #print(direction)
    newx=current.x
    newy=current.y
    for j in range(move):
        if direction==0:
            newy+=1
        elif direction==1:
            newx+=1
        elif direction==2:
            newy-=1
        elif direction==3:
            newx-=1
        current=Point(newx, newy)
        #print(current)
        if (not part2done) and (current in visited):
            print("Visited twice: ", current)
            #calculate distance
            print("Distance (part 2): ", abs(current.x)+abs(current.y))
            part2done=True
        # add current to visited
        visited.add(current)



# calculate distance
print("Final position: ", current)
print("Distance (part 1): ", abs(current.x)+abs(current.y))
```