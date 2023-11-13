Title: 2016 day 08: Two-Factor Authentication
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We have a series of instructions for the screen and need to interpret
them. Emulating strange hardware is my favourite part of Advent of Code.
But this one was too simple.

I used Numpy. One of the instructions is rotate, Numpy provides function
"roll" that does just that. I also packed most of the code in a class.
Both aren't really needed for a simple task like this.

```python
#!/usr/bin/python3

import numpy as np

WIDTH=50
HEIGHT=6
INPUTFILE="08-input.txt"

class Display:
    height: int
    width: int
    screen: np.array
    def __init__(self, _width:int, _height: int):
        self.height=_height
        self.width=_width
        self.screen=np.zeros((self.height, self.width))  # use screen[y][x] when indexing

    def count_pixels(self):
        return np.count_nonzero(self.screen)

    def display(self):
        for i in range(self.height):
            print('|',end='')
            for j in range(self.width):
                if self.screen[i][j]==0:
                    print(' ', end='')
                else:
                    print('█', end='')
            print('|', i)

    def rotate_col(self, x: int, by: int):
        newcolumn=np.roll(self.screen[:,x], by)
        self.screen[:, x]=newcolumn

    def rotate_row(self, y: int, by: int):
        newrow=np.roll(self.screen[y], by)
        self.screen[y]=newrow

    def draw_rect(self, w: int, h: int):
        self.screen[0:h, 0:w]=1


def do_rect(line: str, display: Display):
    _,coords=line.strip().split(' ')
    w,h=coords.split('x')
    w=int(w)
    h=int(h)
    display.draw_rect(w, h)
    
    

def do_rotate(line: str, display: Display):
    _,axis,coord,_,by=line.strip().split(' ')
    by=int(by)
    _,addr=coord.split('=')
    addr=int(addr)
    #print(f"Rotating {axis} {addr} by {by}")
    if axis=="row":
        display.rotate_row(addr, by)
    else:
        display.rotate_col(addr, by)


d=Display(WIDTH, HEIGHT)

with open(INPUTFILE) as f:
    for line in f:
        if line[0:4]=="rect":
            do_rect(line, d)
        elif line[0:6]=="rotate":
            do_rotate(line, d)
        else:
            print("wrong command")


d.display()

print("Part 1, pixels on: ", d.count_pixels() )
```