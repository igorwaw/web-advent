Title: 2016 day 11: Radioisotope Thermoelectric Generators
Date: 2023-05-17 17:32
Status: hidden
Category: Python
Series: 2016

Puzzle description is a long and quite convoluted. We need to move some Radioisotope Thermoelectric Generators
and microchips to the fourth floor. Generators emit radiation which will damage the microchip on the same floor,
unless the chip is connected to a corresponding generator. Elevator can take one or two devices (won't go empty).

It's one of the tough ones. I tried to solved it myself, got stuck and searched for solutions.
Then I combined some parts of <https://eddmann.com/posts/advent-of-code-2016-day-11-radioisotope-thermoelectric-generators/>
with what I wrote before (or what I decided to do in a different way).

## First thoughts

* we have to check many possible ways to solve the problem,
* instruction suggests we probably shouldn't check *all* ways because that would make the problem too large,
* the tricky part is the data structure, how do we model the problem?
* enough thinking, let's do some exploratory coding

## Input

Parsing input file is easy. Each line is one floor, so `for i,line in enumerate(f):` gives a floor number (starting from 0, as
it should - I can just add 1 while printing) and a line. Then we need to capture a word before " generator" and "-compatible",
easy with regexps:

```python
rx1=re.compile(r"(\w+) generator")
rx2=re.compile(r"(\w+)-compatible")
```

Now I can store the data in many ways. Should I have a data structure that describes a floor with all possible
devices? Or a dict of generators and a dict of microchips with element as the key, floor as the value? Or a list of
generators and a list of microchips on each floor? At first I went with the last approach. It makes it easy to check
for valid state (see below).  But it makes it harder to find the next state. We can move:

* 1 or 2 microchips,
* 1 or 2 generators,
* 1 of each type of object.

It's easier if they are in the same data structure. I can combine the lists together
eg. with itertools.chain, but then I would have two objects called "promethium" with no way
to tell which one is a generator and which one is a microchip. So, let's keep them all
in one list. But that means I need to also store the type, not just the name. Dataclasses to the rescue:

```python
@dataclass
class Device:
    name: str
    type: str
```

Also, while we're at it, let's change the list to set. Lists are faster for iterating, sets are
faster for most other operations. Sets can only store hashable objects, but that only requires
changing `@dataclass` to `@dataclass(frozen=True)`. And the input parsing code looks like this:

```python
floors=[]
with open(INPUTFILE) as f:
    for i,line in enumerate(f):
        floors.append(set())
        for rtg in rx1.findall(line):
            floors[i].add( Device(rtg, "generator") )
        for chip in rx2.findall(line):
            floors[i].add( Device(chip, "microchip") )

current_floor=0
max_floor=i+1
```

For a fully OOP solution, I should also create a Floor class containing a set of devices plus some
functions operating on it. But I decided that a list of sets is good enough.

## What's a valid state?

* If there's no generator on the floor, there's no radiation, so all chips are safe,
* If there are no microchips on the floor, the radiation won't harm anything,
* If there are both chips and generators on the floor, every chip needs to have a coresponding generator

First condition is easy to check. Second condition will be checked implicitly by the third condtion
(we're iterating over microchips, if there are none, there's nothing to iterate on). But the third
one is tricky. Now, it was easy when I had a separate list of chips and generators:

```python
    for e in microchips:
        if e not in generators:
            return False
```

Or even shorter, though less readable: `return all(e in generators for e in microchips)`
Function "all" returns True if all elements in the iterable are True, in this case, if
we can find a generator for every chip. I don't usually use it because for me this syntax
is less readable (maybe I'm just not used to it). There's also another function "any" which
returns True if at least one element is True.

But now we need to filter the list to find microchips-only and generators-only iterables.
We also need to extract first element of the tuple. So the for loop becomes this monstrosity:

```python
    for device in filter(lambda x: x.type=="microchip", floor):
        if device.name not in [ dev.name for dev in filter(lambda x: x.type=="generator", floor)]:
            return False
    return True
```

That is horrible! How it would look with all?

```python
    return all(
        device.name
        in [dev.name for dev in filter(lambda x: x.type == "generator", floor)]
        for device in filter(lambda x: x.type == "microchip", floor)
    )
```

For me, it looks even worse and it's not faster, so I left the loop.


## Wrapping the gamestate together

Since we have to create many possible states, store them, give them as an argument to the function
or return them from the function, it makes sense to store them in an object. I also added a print
function for debugging.

```python
@dataclass
class Gamestate:
    elevator: int
    floors: list
    moves: int = 0

    def print(self) -> None:
        global maxfloor
        for i in range(maxfloor):
            print(f"Floor {i+1}: {self.floors[i]}  is valid? {is_valid_floor(self.floors[i])}")
```

## Finding the next steps

We can move the devices up or down. The elevator can't go below the first floor or above the last
floor, so we can immediately skip this direction and only check the other one.

We can move 2 devices from the current floor: `combinations(state.devices(), 2)`. We can also move 1 device
at a time: `combinations(state.devices(), 1)`. How do we iterate over both at a time? There's a handy function
in itertool called chain that combines iterators.

Next, we need to copy the gamestate. In the new object, we have to modify two floors: remove object from one
and add them to the other one. We have sets, which have functions just for this. Finally, we need to check
if those two floors are stil valid.

## Checking the end condition

We need to move all devices to the highest floor. How to check that? Easiest way: check if all other floors are empty.
Let's add one function to the Gamestate class:

```python
    def is_done(self) -> bool:
        global maxfloor
        for i in range(maxfloor-1):
            if len(self.floors[i]):
                return False
        return True
```

## Do not go back

What if we take a device up one level, then in the next step down one level, then up again etc.?
That's an infinite loop, and since we're checking all possible moves, we would unavoidably hit it.
We need to store the states that were already checked and skip them.

But there's more. It's not really important which generator or microchip is moved. A unique state
can be described by number of generators and microchips on each floor plus the elevator position.
I decided to encode a state in a string and make it readable to help debugging. I added this
function to the Gamestate class:

```python
    def get_seen_state(self) -> str:
        retval=f"elevator:{self.elevator} "
        for i,floor in enumerate(self.floors):
            count_chips = sum(1 for _ in filter(lambda x: x.type=="microchip", floor))
            count_rtgs = sum(1 for _ in filter(lambda x: x.type=="generator", floor))
            retval+=f"floor{i}:m{count_chips}g{count_rtgs} "
        return retval
```

## New Python features: filter and lambda

**Filter** is a function that selects elements from an iterable object (list, set, tuple etc.). It takes two
arguments, first is a function that returns True or False, second is an iterable. We can use a fully fledged
function here, but lambda is a more popular choice for simple functions.

**Lambda** is a small, anonymous function. The syntax might look a bit awkward in the beginning (believe me,
it's MUCH better than lambda in C++), it looks like this: `lambda arguments: expression`. Only one expression
and it's automatically returned. The above code: `filter(lambda x: x[1]=="c", floor)` without lambda could
be written like this:

```python
def check_if_chip(x):
    return x.type=="microchip"

filter(check_if_chip, floor)
```

or even more explicit:

```python
def check_if_chip(x):
    if x.type=="microchip"
        return True
    else:
        return False

filter(check_if_chip, floor)
```

## Breadth First Search

We have an initial state, we generate possible next moves, where to go from here? We can follow one path until it finds a solution
or a dead end (eg. using a recursive function), then follow another path etc., but that's not an optimal solution. We're
looking for a minimal number of moves, if it's (for example) 50, there's no point in exploring paths that took 51 moves
or more. Which means we should use a BFS, or Breadth First Search algorithm:

* get all possible second steps from the initial state,
* for each possible second step, get a list of possible third steps,
* and so on,

Until the solution is found. It is then guaranteed to be the shortest solution. For the BFS, we need a **queue** - a data
structure that operates on a First In - First Out basis. Python has a **deque**, double-ended queue which can also function
as a stack. We start by adding the initial simulation state to the queue: `queue = deque([state])`. Then, in a loop, we
get the oldest element from the queue: `state = queue.popleft()`, generate all possible next steps for it and
add them to the queue: `queue.append(next_state)`. And of course, at every iteration we check if we reached the end.

## Part 2

Second part is the same algorithm, we're just adding 4 more devices to the data structure. All it did was increasing calculation
time 5s to 37s.

## Code

```python
#!/usr/bin/python3

from collections import deque
import re
from dataclasses import dataclass
from itertools import chain, combinations
from copy import deepcopy


INPUTFILE="11-input.txt"

rx1=re.compile(r"(\w+) generator")
rx2=re.compile(r"(\w+)-compatible")




def is_valid_floor(floor: set) -> bool:
    # floor is valid if there's no generator...
    if not [e for e in floor if e.type=="generator" ]:
        return True
    # ...or for every chip there's a relevant generator
    for device in filter(lambda x: x.type=="microchip", floor):
        if device.name not in [ dev.name for dev in filter(lambda x: x.type=="generator", floor)]:
            return False
    return True


@dataclass(frozen=True)
class Device:
    name: str
    type: str

@dataclass
class Gamestate:
    elevator: int
    floors: list
    moves: int = 0

 
    def is_done(self) -> bool:
        global max_floor
        for i in range(max_floor-1):
            if len(self.floors[i]):
                return False
        return True

    def get_seen_state(self) -> str:
        retval=f"elevator:{self.elevator} "
        for i,floor in enumerate(self.floors):
            count_chips = sum(1 for _ in filter(lambda x: x.type=="microchip", floor))
            count_rtgs = sum(1 for _ in filter(lambda x: x.type=="generator", floor))
            retval+=f"floor{i}:m{count_chips}g{count_rtgs} "
        return retval

    def print(self) -> None:
        global max_floor
        print(f"--- elevator: {self.elevator+1}   moves: {self.moves}  -----")
        for i in range(max_floor):
            print(f"Floor {i+1}: {self.floors[i]}  is valid? {is_valid_floor(self.floors[i])}")


def get_next_move(state: Gamestate) -> Gamestate:
    global max_floor
    current_floor=state.elevator
    devices_to_move=chain( combinations(state.floors[current_floor], 2), combinations(state.floors[current_floor], 1)  )
    for devs in devices_to_move:
        for i in [-1, 1]:
            next_floor=current_floor+i
            if next_floor<0 or next_floor>=max_floor:
                continue

            newstate=deepcopy(state)
            newstate.elevator=next_floor
            newstate.moves+=1
            newstate.floors[current_floor]=newstate.floors[current_floor].difference(devs)
            newstate.floors[next_floor]=newstate.floors[next_floor].union(devs)

            if (is_valid_floor(newstate.floors[current_floor]) and is_valid_floor(newstate.floors[next_floor])):
                yield newstate


def count_moves(state: Gamestate) -> int:
    seen = set()
    queue = deque([state])

    while queue:
        state = queue.popleft()
        if state.is_done():
            return state.moves
        
        for next_state in get_next_move(state):
            if (key := next_state.get_seen_state()) not in seen:
                seen.add(key)
                # next_state.print()
                queue.append(next_state)




# parse input
floors=[]
with open(INPUTFILE) as f:
    for i,line in enumerate(f):
        floors.append(set())
        for rtg in rx1.findall(line):
            floors[i].add( Device(rtg, "generator") )
        for chip in rx2.findall(line):
            floors[i].add( Device(chip, "microchip") )

current_floor=0
max_floor=i+1

gamestate=Gamestate(current_floor, floors)
#gamestate.print()
print("Part 1: ") #47
print(count_moves(gamestate))
print("Part 2: ") #71
# reset and add devices
current_floor=0
floors[0]=floors[0].union([ Device("elerium", "generator"), Device("elerium", "microchip"), 
                            Device("dilithium", "generator"), Device("dilithium", "microchip") ])
gamestate=Gamestate(current_floor, floors)
print(count_moves(gamestate))
```
