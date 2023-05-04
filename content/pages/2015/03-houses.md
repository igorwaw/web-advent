Title: 2015 day 3: Perfectly Spherical Houses in a Vacuum
Date: 2023-04-15 17:33
Status: hidden
Category: Python
Series: 2015

Santa is delivering presents to an infinite two-dimensional grid of houses. He's got a list of instructions
telling him to go south, north, east or west. The instructions can take him to some houses more than once,
we need to count how many unique houses were visited.

Python has just the data structure for this: a **set**. All I need is to put coordinates of every visited house,
set will only add it if it's not already present and silently ignore it otherwise. At the end, I just need to check
the size of the set. I wrote some quick and dirty code for this.

For part 2, Santa got a mechanical helper. Santa and Robo-Santa start at the same position, every odd instruction (1, 2, 3...)
is for Santa, every even one (2, 4, 6...) for the robot. How many houses would be visited?

I decided to rewrite the code and solve a more general problem: not only for 1 and 2 Santas, but for an arbitrary number of Santas.
For that I created a class Santa with 3 attributes: current coordinates and a set of visited houses. Then, a function called santatour
creates a list of Santa objects: `santas = [ Santa() for i in range(numsantas)]`. Then I used an instruction counter and modulo arithmetics:
`s=counter%numsantas`, if numsantas equals 2, gives 0 for instruction 0, 2, 4... and 1 for 1, 3, 5. If just one Santa, it always gives 0.
For numsantas equals 3, it goes 0, 1, 2, 0, 1, 2... etc.

I could just record all visits in one set of visited houses, but again I wanted to solve a more generic problem. Each santa held its own
record, so I needed to get a **union**. A union of 2 sets contains all elements that belong to any of the sets, in Python you can use a union function
or | operator:  `totalvisited=totalvisited|santas[i].visited`


```python
#!/usr/bin/python3

INPUTFILE="03-input.txt"



class Santa:
    x: int
    y: int
    visited: 'set[int,int]'
    
    def __init__(self, x=0, y=0):
        self.x=0
        self.y=0
        self.visited=set()
        self.record_visit()

    def record_visit(self):
        #print("Recording visit in ", self.x, self.y)
        self.visited.add( (self.x, self.y) )


def santatour(numsantas: int=1):
    counter=0 # instruction counter
    santas = [ Santa() for i in range(numsantas)]
    with open(INPUTFILE) as inputfile:
        while True:
            s=counter%numsantas
            c=inputfile.read(1)
            #print(f"Counter {counter} instruction {c} santa {s}")
            if not c:
                break
            if   c=='<': santas[s].x-=1
            elif c=='>': santas[s].x+=1
            elif c=='^': santas[s].y+=1
            elif c=='v': santas[s].y-=1
            else:
                print("Error: unkown direction ",c)
            santas[s].record_visit()
            counter+=1
    print ("Houses visited by each Santa")
    totalvisitsnum=0
    totalvisited=set()
    for i in range(numsantas):
        visits=len(santas[i].visited)
        totalvisited=totalvisited|santas[i].visited
        print(f"  Santa {i}  houses {visits}")
        #print(santas[i])
    print("Total: ", len(totalvisited))

print("Part 1:")
santatour(1)

print("\n\nPart 2:")
santatour(2)
```
