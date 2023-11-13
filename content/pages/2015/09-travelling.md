Title: 2015 day 09: All in a Single Night
Date: 2023-05-09 17:32
Status: hidden
Category: Python
Series: 2015

We are given a list of distances between several cities. Santa needs to visit all of them exactly once
using shortest (part 1) and longest (part 2) route possible.

What a gentle introduction to graph theory! Every Computer Science graduate will instantly recognize
it as a Travelling Salesman Problem. It's a well known problem with many real-world applications - not
only the obvious one of finding a shortest route between cities, but also about placing data in the compute
cluster, and optimal use of pretty much everything, from storage shelves to industrial robots. It's
also deceptively hard. Brute-force approach of trying all possible routes is O(n!) meaning it scales with
the factorial of the number of nodes so it's out of the question for bigger datasets.
Luckily, we only have 7 cities, 7! is only 5040. There's no need to implement any clever algorithm.

So, it's Python again with no special libraries (C++ with the usual STL containers would also work):

* read input file, parse lines (just two split(), nothing fancy)
* we need 3 data structures: set of cities, dict of distances between cities (both initialized from the input data)
and list of route lengths, initially empty (note that we only need to store the distance, not the route)
* iterate through all permutations of cities
* for each permutations, get the distances from the dict, sum them up, store in the list of route lengths
* and just get the smallest/largest number from the list

## New language feature: itertools

Many puzzles in AoC involve combinatorics. In this case we need to generate all **permutations** of cities - or
arrange them in every possible order. Module itertools contains mostly functions for looping and iterating
over data, but also combinatoric functions: permutations, combinations, combinations_with_replacement and
product.

They are very simple to use. We have a set of cities (any iterable data structure would do):

```python
{'AlphaCentauri', 'Norrath', 'Straylight', 'Arbre', 'Faerun', 'Tambi', 'Snowdin', 'Tristram'}
```

and `permutations(cities)` will generate an iterable object containing:

```python
('AlphaCentauri', 'Norrath', 'Straylight', 'Arbre', 'Faerun', 'Tambi', 'Snowdin', 'Tristram')
('AlphaCentauri', 'Norrath', 'Straylight', 'Arbre', 'Faerun', 'Tambi', 'Tristram', 'Snowdin')
('AlphaCentauri', 'Norrath', 'Straylight', 'Arbre', 'Faerun', 'Snowdin', 'Tambi', 'Tristram')
...
('Tristram', 'Snowdin', 'Tambi', 'Faerun', 'Arbre', 'Norrath', 'Straylight', 'AlphaCentauri')
('Tristram', 'Snowdin', 'Tambi', 'Faerun', 'Arbre', 'Straylight', 'AlphaCentauri', 'Norrath')
('Tristram', 'Snowdin', 'Tambi', 'Faerun', 'Arbre', 'Straylight', 'Norrath', 'AlphaCentauri')
```

## Possible optimizations

* No need to store the length of every route, we only need to store it if it's shortest/longest then the previous ones.
* Every route has the same length in both directions, while the code generates all permutations. So, for
the example data we check Dublin -> London -> Belfast and Belfast -> London -> Dublin. We could cut
the problem size in half by filtering out those.

Those improvements would only save a few hundreds bytes of RAM and have negligible effect on time (and I'm not even sure which
way - there's less computation but some extra comparisons) so I skipped the premature optimizations.

## Code

```python
#!/usr/bin/python3

from itertools import permutations


FILENAME="09-input.txt"

def read_input(filename):
    distances={}
    cityset=set()
    with open(filename) as inputfile:
        for line in inputfile:
            cities, distance=line.split('=')
            distance=int(distance)
            city1, city2 = cities.split("to")
            city1=city1.strip()
            city2=city2.strip()
            cityset.add(city1)
            cityset.add(city2)
            distances[(city1,city2)] = distance
            distances[(city2,city1)] = distance
    return distances, cityset


distances, cities=read_input(FILENAME)

routelengths=[]
for route in permutations(cities):
    current_dist = sum(
        distances[(route[i], route[i + 1])] for i in range(len(route) - 1)
    )
    routelengths.append(current_dist)

print("Shortest route: ", min(routelengths))
print("Longest route: ", max(routelengths))
```