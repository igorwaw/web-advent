Title: 2016 day 15: Timing is everything
Date: 2023-05-22 17:32
Status: hidden
Category: Python
Series: 2016

We have a machine with rotating discs. Each disk has many possible positions, only
position 0 will let the ball fall through. Ball takes 1 second to fall between the disks.
For part 1, sizes and initial positions are puzzle input. Part 2 just adds one disk.

## Brute force

The naive solution is to simulate all the steps until we find the proper alignment.
But hold on, is it really naive? Although it had to iterate through 3 million steps,
it only took about 2 seconds. I'd say that solution is good enough.

Anyway, here it goes:

```python
def brute_force(sizes: list, positions: list) -> int:
    t=0
    while True:
        if all( (t+startpos+num)%size==0 for num,(startpos,size) in enumerate(zip(positions,sizes)) ):
            return t
        t+=1
```

Position of each disk is a sum of starting position and time, modulo number of positions. But we don't want
the disks to align on position 0. Remember that the ball takes a second to reach the next disk, so each
disk needs to be offset by 1. For the first one we need to solve: t+startpos==0, for the second one,
t+startpos==-1 etc. Let's just move the number to the left side of the equation (relax, that's only
primary school maths, it will get worse in the next solution) so the right side is always 0 and
it's easy to iterate.

I don't usually like the all function, but here it actually makes the code more readable - we are really
checking if **all** disks are in the right position.

Note that the function calculates the time when the machine aligns. The ball takes 1 second to reach
the first disk, so to get the time to release the ball, we need to subtract 1.

## Chinese Remainder Theorem

Even though the solution was fast, I wasn't satisfied with it. Sure, for all practical purposes it was OK,
but AoC is about learning and having fun. I wanted something better.

It was obvious there was no need to check all steps one by one. I made a following thought experiment:
let's choose the largest disk, say it has 17 positions. Find when it's properly aligned. Then
from this point, increment time by 17 instead of one and check if other disks aligned. At some point
one of them is aligned, say it has 3 positions. Next time they are both aligned the same way, is
3*17=51 seconds later, so that's the new value of step. Then we find the third disk and so on.
All our disk sizes are prime numbers.

I didn't code this solution, because at this point something rang a bell. I graduated in Computer Science,
meaning at some point I learned a bit about advanced mathematics: number theory, matrices, differential
equations, you name it. I then forgot much of it as it was 20 years ago and most of this stuff is not needed
for the daily work (except once in a blue moon it unexpectedly helps to solve a programming problem). But modulo
arithmetics appears in a lot of places, prime numbers are a known feature in cryptography. I remember I learned
something about solving a system of equations in modulo arithmetics with prime numbers. After a few minutes
of searching I found Chinese Remainder Theorem. Yep, that was it.

I'm not going to repeat what others wrote, Wikipedia has a theoretical background: <https://en.wikipedia.org/wiki/Chinese_remainder_theorem>
and Rosetta Code has example code: <https://rosettacode.org/wiki/Chinese_remainder_theorem>.
The hard part was calculacting the right residues, it took some trial and error to get rid of inevitable
off-by-one and "add or subtract" errors.

## Code

```python
#!/usr/bin/python3

from functools import reduce

INPUTFILE="15-input.txt"

def chinese_remainder(divisors: list, residues: list) -> int:
    csum = 0
    prod = reduce(lambda acc, b: acc*b, divisors)
    for divisor_i, a_i in zip(divisors, residues):
        p = prod // divisor_i
        csum += a_i * modular_inverse(p, divisor_i) * p
    return csum % prod
  

def modular_inverse(a: int, b: int) -> int:
    b0 = b
    x0, x1 = 0, 1
    if b == 1: return 1
    while a > 1:
        q = a // b
        a, b = b, a%b
        x0, x1 = x1 - q * x0, x0
    if x1 < 0: x1 += b0
    return x1

def get_residues(positions: list, sizes: list) -> list:
    residues=[]
    for i, (position, size) in enumerate(zip(positions, sizes)):
        newposition=(size-position-i)%size
        residues.append(newposition)
    return residues

def brute_force(sizes: list, positions: list) -> int:
    t=0
    while True:
        if all( (t+startpos+num)%size==0 for num,(startpos,size) in enumerate(zip(positions,sizes)) ):
            return t
        t+=1

# parse input
sizes=[]
positions=[]
with open(INPUTFILE) as f:
    for line in f:
        splitline=line.rstrip().split()
        size=int(splitline[3])
        position=int(splitline[11][:-1]) # skip last character - dot
        sizes.append(size)
        positions.append(position)


residues=get_residues(positions, sizes)
print(f"Disc sizes: {sizes}")
print(f"Disc positions: {positions}")
print(f"Residues: {residues}")

part1crt=chinese_remainder(sizes, residues)-1
part1force=brute_force(sizes,positions)-1

print(f"Part 1, Chinese Remainder Theorem: {part1crt}")
print(f"Part 1, brute force: {part1force}")

#part 2
sizes.append(11)
positions.append(0)
residues=get_residues(positions, sizes)
part2crt=chinese_remainder(sizes, residues)-1
part2force=brute_force(sizes,positions)-1
print(f"Part 2, Chinese Remainder Theorem: {part2crt}")
print(f"Part 2, brute force: {part2force}")
```