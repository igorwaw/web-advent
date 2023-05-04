Title: 2015 day 2: I Was Told There Would Be No Math
Date: 2023-04-15 17:32
Status: hidden
Category: Python
Series: 2015

We need to calculate how much wrapping paper (part 1) and ribbon (part 2) we need for the presents. We are given the list of dimensions
to calculate the area/perimeter, we also need to add some extra paper/ribbon, we even get the equations.

Simple thing again. This time I went with Python because I wanted to solve it fast and move to more interesting puzzles.


```python
#!/usr/bin/python3

INPUTFILE="02-input.txt"

totalpaper=0
totalribbon=0

with open(INPUTFILE) as inputfile:
    for line in inputfile:
        x,y,z=line.rstrip().split('x')
        dimensions=[int(x), int(y), int(z)]
        dimensions.sort()
        #print(dimensions[0], dimensions[1], dimensions[2])
        totalpaper += 3*dimensions[0]*dimensions[1] + 2*dimensions[1]*dimensions[2] + 2*dimensions[2]*dimensions[0]
        totalribbon+= 2*dimensions[0]+2*dimensions[1] + dimensions[0]*dimensions[1]*dimensions[2]

print("Total paper needed:  ",totalpaper)
print("Total ribbon needed: ",totalribbon)
```

