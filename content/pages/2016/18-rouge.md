Title: 2016 day 18: Like A Rouge
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

A simple cellular automaton - like Conway's Life, but in one dimension.
Nothing special here. Input data has "^" and "." characters, I changed
them to 1s and 0s for easy calculation. 

Instruction says that tile is a trap if one of these conditions is true;

- Its left and center tiles are traps, but its right tile is not.
- Its center and right tiles are traps, but its left tile is not.
- Only its left tile is a trap.
- Only its right tile is a trap.

After coding all those, turns out it can be simplified to "left is
different than right". Also, when we're on the edge, we should consider
places outside of the corridor as safe. Which means getting the new row
based on the old row is as simple as that:

```python
def get_next_row(oldrow: list) -> list:
    newrow=[]
    for i in range(len(oldrow)):
        left=oldrow[i-1] if i>=1 else 1
        right=oldrow[i+1] if i<len(oldrow)-1 else 1
        newrow.append(1 if left==right else 0)
    return newrow
```

## Part 2

For part 2, the corridor has 400000 rows instead of 40. Which means if I wrote the code
in a very inefficient way, it would take ages to run. But it takes 8 seconds and it's quite
elegant, so I don't feel compelled to look for a better solution.

## Code

```python
#!/usr/bin/python3


STARTROW="^^.^..^.....^..^..^^...^^.^....^^^.^.^^....^.^^^...^^^^.^^^^.^..^^^^.^^.^.^.^.^.^^...^^..^^^..^.^^^^"
NUMROWS1=40
NUMROWS2=400000


def get_next_row(oldrow: list) -> list:
    newrow=[]
    for i in range(len(oldrow)):
        left=oldrow[i-1] if i>=1 else 1
        right=oldrow[i+1] if i<len(oldrow)-1 else 1
        newrow.append(1 if left==right else 0)
    return newrow


row=[ 1 if x=="." else 0 for x in STARTROW ]
sumsafe=sum(row)
for i in range(NUMROWS2-1):
    row=get_next_row(row)
    sumsafe+=sum(row)
    if i==NUMROWS1-2:
        print(f"Part 1: {sumsafe}")        

print(f"Part 2: {sumsafe}")
```