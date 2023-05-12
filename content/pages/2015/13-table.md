Title: 2015 day 13: Knights of the Dinner Table
Date: 2023-05-09 17:32
Status: hidden
Category: Python
Series: 2015

We need to find optimal seating arrangement for the family. Everyone has their happines
increased or decreased based on who they sit next to. We need to maximize total happines.

Although it doesn't look like that, it's a similar puzzle to day 9, Travelling Santa
Problem. So similar that most of my code was actually a copy-paste. Again, it would require
some sophisticated algorithms for larger dataset, but we only have 8 people so we might as
well do brute-force.

Spoilers below:
There are 2 key differences between day 9 (distance finding) and this puzzle:

* on day 9, you had to sum up distance from A to B, then B to C etc., here you have
to sum up happiness from A to B and from B to A, B to C and C to B etc.
* the table is round, so you have to also add happiness between last person and first
person - the easiest way is to add the first person again at the end.

Possible optimizations: similar to day 9, you can skip sitting arrangements that are
the reverse of another sitting arrangement, but I didn't bother as the program was fast
enough. Though I can clearly see how it grows with the size of the dataset: on an old
laptop it took 0.2s for the first part and 1.4s for the second part with one extra
person.


```python
#!/usr/bin/python3

from itertools import permutations



FILENAME="13-input.txt"


def calculate_happiness():
    global edges, people
    happiness=[]
    for people_permut in permutations(people):
        sitting_arrangement=list(people_permut)
        sitting_arrangement.append(sitting_arrangement[0])
        #print(sitting_arrangement)
        current_happiness=0
        for i in range(len(sitting_arrangement)-1):
            current_happiness+=edges[( sitting_arrangement[i], sitting_arrangement[i+1] )]
            current_happiness+=edges[( sitting_arrangement[i+1], sitting_arrangement[i] )]
        happiness.append(current_happiness)
    return happiness

# parse input
edges={}
people=set()
with open(FILENAME) as inputfile:
    for line in inputfile:
        person, _, direction, howmuch, _, _, _, _, _, _, otherperson = line.strip().split()
        otherperson=otherperson.strip('.')
        if direction=='gain':
            value=int(howmuch)
        elif direction=='lose':
            value=-int(howmuch)
        else:
            raise ValueError
        people.add(person)
        edges[(person, otherperson)]=value

happiness=calculate_happiness()
print("Part 1: ", max(happiness))

# add another person
for person in people:
    edges[person, "me"]=0
    edges["me", person]=0
people.add("me")

happiness=calculate_happiness()
print("Part 2: ", max(happiness))
```