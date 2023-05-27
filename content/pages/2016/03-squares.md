Title: 2016 day 03: Squares With Three Sides
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

```python
#!/usr/bin/python3

INPUTFILE="03-input.txt"

#part 1
num_triangles=0
with open(INPUTFILE) as f:
    for line in f:
        sides=[int(i) for i in line.split()]
        sides.sort()
        if sides[0]+sides[1]>sides[2]:
            num_triangles+=1
print("Part 1: ", num_triangles)

#part 2
triangles=[]
num_triangles=0
with open(INPUTFILE) as f:
    lines=[]
    for line in f:
        lines.append([ int(i) for i in line.split() ])
        if len(lines)>=3:
            # process a triangle
            triangles.extend([ lines[0][i], lines[1][i], lines[2][i] ] for i in range(3))
            lines=[]
#print(triangles)
for sides in triangles:
    #print(sides)
    sides.sort()
    if sides[0]+sides[1]>sides[2]:
        num_triangles+=1
print("Part 2: ", num_triangles)

```