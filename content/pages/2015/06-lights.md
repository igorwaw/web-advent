Title: 2015 day 06: Probably a Fire Hazard
Date: 2023-05-05 17:32
Status: hidden
Category: Python
Series: 2015

## Part 1

We've got 1 million lights in a 1000x1000 grid and instructions to turn on/turn off/toggle some lights. The puzzle only requires
to count how many lights are on, but I couldn't resist the temptation to add visualization. I used Pygame library, or rather a
very small subset of it: drawing rectangles, drawing single pixels, waiting for event (just one - quit) and delay.

![screenshot]({static}/images/2015day6.png)

Light grid is stored in a 2D array, initialized with zeroes using nested comprehensions: `lights = [[0 for _ in range(WIDTH)] for _ in range(HEIGHT)]`.
Another notable thing is how I counted the lights: `lit=sum(map(sum,lights))`. The inner function `map(sum,lights)` returns a list containing
sums of each row, and the outer sum just sums it.

Input processing is very simple this time, splitting on space or comma was all that was needed: `elems=re.split(r'\s|,', cmd)`. Then I call a command
rect_on or rect_off which sets some elements to 0 or 1 using nested loops and also calls a pygame function to draw rectangle; rect_toggle is similar
but needs to work on single pixels. Some might say that using loops here is non-pythonic and I should have used list comprehension. In my opinion
comprehension for 2D arrays is not very readable. It's faster, but I needed to slow down the code for visualization anyway.

## New Python feature: walrus operator

**Walrus operator :=** was introduced in Python 3.7. A common source of frustration for C programmers is mistaking comparison
with assignment: `if (i=5)` assigns the value and is probably not what the developer intended. However, `if ( i=some_function() )`
is likely valid, it both assigns the value and checks if it's true. Python, like many modern languages, initialy solved the problem
by separating those operations. Which means I would have to do:

```python
line = inputfile.readline().rstrip()
if (line):
```

instead of `if (line := inputfile.readline().rstrip()):`. Walrus operator allows to combine those two again, and since it's
neither = nor ==, it can't be easily mistaken. In this case there's not much difference. But walrus operator makes the code
not only shorter but more readable when placed in a loop statement or list comprehension. Example without and with walrus operator:

```python
x=get_value()
while(x):
    do_something(x)
    x=get_value()

...

while(x:=get_value)
    do_something(x)
```



## Code

```python
#!/usr/bin/python3

import re
import pygame


# constants
INPUTFILE="06-input.txt"
WIDTH=1000
HEIGHT=1000
FPS=25 # how many animation frames / calculation steps per second

cwhite=(255,255,255)
cblack=(0,0,0)

# global variables
lights=[]



############ PARSE FILE

def run_cmd(cmd):
    elems=re.split(r'\s|,', cmd)
    if elems[0]=='toggle': rect_toggle( int(elems[1]), int(elems[2]), int(elems[4]), int(elems[5]) )
    elif elems[1]=='on'  : rect_on    ( int(elems[2]), int(elems[3]), int(elems[5]), int(elems[6]) )
    elif elems[1]=='off' : rect_off   ( int(elems[2]), int(elems[3]), int(elems[5]), int(elems[6]) )
    else: raise(ValueError)



########## FUNCTIONS ################

def rect_on(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            lights[i][j]=1
    pygame.draw.rect(screen, cwhite, pygame.Rect(x1,y1,x2-x1,y2-y1))
    

def rect_off(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            lights[i][j]=0
    pygame.draw.rect(screen, cblack, pygame.Rect(x1,y1,x2-x1,y2-y1))

def pixel_toggle(x,y):
    if lights[x][y]:
        lights[x][y]=0
        screen.set_at((x,y), cblack)
    else:
        lights[x][y]=1
        screen.set_at((x,y), cwhite)

def rect_toggle(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            pixel_toggle(i,j)


# initialize pygame
pygame.init()
screen=pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Day 6: Probably a Fire Hazard')
clock = pygame.time.Clock()
screen.fill((0, 0, 0)) # Fill the background with black
running=True



# initialize the rest
lights = [[0 for _ in range(WIDTH)] for _ in range(HEIGHT)]
with open(INPUTFILE) as inputfile:
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT: running=False

        if (line := inputfile.readline().rstrip()):
            run_cmd(line)
        pygame.display.flip()
        clock.tick(FPS) # wait here


    #end event loop, cleanup here
    pygame.quit()
    lit=sum(map(sum,lights))
    print(f"Lights on: {lit}, off: {WIDTH*HEIGHT-lit}")
```

## Part 2

It turns out the lights have many levels of brightness, "on" really means
"inrease by 1", off - "decrease by one" and toggle - "increase by 2". Of course
it does.

Unusually, I decided to put part 2 into a separate file. I also changed the 
visualization, instead of updating screen after each instruction, I do it only
once, at the end of the program. Updating at every step would really slow down
the calcultion as there are many cases of controlling individual pixels and this is slow
in Pygame.


```python
#!/usr/bin/python3

import re
import pygame


# constants
INPUTFILE="06-input.txt"
WIDTH=1000
HEIGHT=1000
FPS=500 # how many animation frames / calculation steps per second

cwhite=(255,255,255)
cgrey1=(180,180,180)
cgrey2=(220,220,220)
cblack=(0,0,0)

# global variables
lights=[]



############ PARSE FILE

def run_cmd(cmd):
    elems=re.split(r'\s|,', cmd)
    if elems[0]=='toggle': rect_toggle( int(elems[1]), int(elems[2]), int(elems[4]), int(elems[5]) )
    elif elems[1]=='on'  : rect_on    ( int(elems[2]), int(elems[3]), int(elems[5]), int(elems[6]) )
    elif elems[1]=='off' : rect_off   ( int(elems[2]), int(elems[3]), int(elems[5]), int(elems[6]) )
    else: raise(ValueError)



########## FUNCTIONS ################

def rect_on(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            lights[i][j]+=1
    

def rect_off(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            lights[i][j]-=1
            lights[i][j] = max(lights[i][j], 0)


def rect_toggle(x1, y1, x2, y2):
    for i in range(x1, x2+1):
        for j in range(y1, y2+1):
            lights[i][j]+=2

# initialize pygame
pygame.init()
screen=pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Day 6: Probably a Fire Hazard')
clock = pygame.time.Clock()
screen.fill((0, 0, 0)) # Fill the background with black
running=True



# initialize the rest
lights = [[0 for _ in range(WIDTH)] for _ in range(HEIGHT)]
with open(INPUTFILE) as inputfile:
    for line in inputfile:
        run_cmd(line)

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: running=False
    for i in range(WIDTH):
        for j in range(HEIGHT):
            if lights[i][j]==0:
                screen.set_at((i,j), cblack)
            elif lights[i][j]<=10:
                screen.set_at((i,j), cgrey1)
            elif lights[i][j]<=20:
                screen.set_at((i,j), cgrey2)
            else:
                screen.set_at((i,j), cwhite)
    pygame.display.flip()
    clock.tick(FPS) # wait here


#end event loop, cleanup here
pygame.quit()
lit=sum(map(sum,lights))

print(f"Total brightness: {lit}")
```