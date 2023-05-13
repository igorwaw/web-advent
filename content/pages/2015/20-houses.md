Title: 2015 day 20: Infinite Elves and Infinite Houses
Date: 2023-05-13 17:32
Status: hidden
Category: Python
Series: 2015

Infinite number of elves deliver presents to infinite number of houses - that looks a lot like
[Hilbert's Hotel](https://en.wikipedia.org/wiki/Hilbert's_paradox_of_the_Grand_Hotel). Luckily,
we don't really have to deal with infinities, we need to find a first house that gets more than
a specified number of presents. And there are several ways.

### Use brute force

* Iterate through all house numbers starting from 1. For each house:
* Iterate through elves - numbers from 1 to house number, let's call that value i
* Calculate: house number modulo i, if it's equal to 0, that house get 10*i presents

```python
        housenumber=0
        while True:
            numpresents=0
            housenumber+=1
            for i in range(1,housenumber+1):
                if housenumber%i==0:
                    numpresents+=10*i
            if numpresents>=TARGETNUMBER:
                break
```

Easy to think of, easy to code. The only problem: it would take a long time (my input is an 8-digit number).

There are some possible improvements. We don't have to start from house number 1. Obviously
we need to be somewhere closer to the target number of present, we can start eg. from that
number divided by 100. Much faster than starting from 1, but still slow. And, if we start
too high we might miss the smallest number.

I actually don't know how much time it would take. After several minutes I hit Ctrl-C
and tried another approach.

### Use RAM and smarter force

The brute force solution has 2 loops: external iterates through houses, internal through
elves. That means the internal loop runs many times doing the same calculations all over
again. And, with each iteration of the external loop, internal loop needs one more iteration,
running slower and slower.

Let's reverse the logic:

* Allocate a data structure (eg. in Python list of ints) holding numbers of presents for each
house, initial value 0 (that's a few dozen MB, no problem for a PC)
* Iterate through elves, starting from number 1
* Iterate through houses, notice that elf number n puts presents in every nth house, starting with house
number n (eg. elf number ten starts with house 10, than goes to 20, 30 etc.) - which means we don't need
to start with house 1, increment by 1 and do a modulo operation, instead we can use a proper initial value
and step in the loop, massively speeding up the calculation
* For every visited house, increase the number of presents by 10*elf number
* Finally, iterate through the list of houses and find index of the first one that meets our criteria.

### Use maths

Not the solution I would find by myself, since I'm more into loops and data structures than divisors and such
stuff. Just for completeness: each house gets 10 presents if it's number is divisible by the elf's number.
So, total number of presents equals to 10 times sum of all house number's divisors (including trivial divisors).

## Code

```python
#!/usr/bin/python3

from tqdm import tqdm

TARGETNUMBER=33100000
LIMIT=TARGETNUMBER//10

houses = [0 for _ in range(LIMIT)]
houses2 = [0 for _ in range(LIMIT)]


for elfnumber in tqdm(range(1, LIMIT)):
    numvisit=0
    for housenumber in range(elfnumber, LIMIT, elfnumber):
        houses[housenumber]+=10*elfnumber
        if numvisit<50:
            numvisit+=1
            houses2[housenumber]+=11*elfnumber



# find the smallest house number with the specified number of presents:
for i in range(2, LIMIT):
    if houses[i]>=TARGETNUMBER:
        part1result=i
        break
for i in range(2, LIMIT):
    if houses2[i]>=TARGETNUMBER:
        part2result=i
        break


print(f"Part 1: house number {part1result} got {houses[part1result]} presents")
print(f"Part 2: house number {part2result} got {houses[part2result]} presents")
```