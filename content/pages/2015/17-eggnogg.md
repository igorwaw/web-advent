Title: 2015 day 17: No Such Thing as Too Much
Date: 2023-05-13 17:32
Status: hidden
Category: Python
Series: 2015

The elves ordered too much eggnog. I need to store it in the containers. 
Container capacities are gives as the puzzle input.

So it's combinatorics again. Actually easier than the previous ones.

```python
#!/usr/bin/python3

from itertools import combinations


INPUTFILE="17-input.txt"
CAPACITY=150

with open(INPUTFILE) as inputfile:
        containers = [int(x) for x in inputfile.read().splitlines()]

print(containers)

container_combinations=[]
part2reached=False
for num_containers in range(len(containers)+1): # we're tryng combinations of 1, 2... containers until len(containers)
    container_combinations.extend(   i for i in combinations(containers, num_containers) if sum(i)==CAPACITY )
    if not part2reached:
        if container_combinations: # that was the smallest number of containers that worked
            part2reached=True
            part2answer=len(container_combinations)

print("Part 1, number of combinations: ", len(container_combinations))
print("Part 2, number of combinations: ", part2answer)
```