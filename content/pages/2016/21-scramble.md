Title: 2016 day 21: Scrambled Letters and Hash
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

## Part 1

We need to scramble a password according to a list of instructions. Easy.
First I parse the list of instructions with my favourite match/case. For
every instruction there's a simple function. There are operations such as
swapping letters or rotating the string. All of them can be done with
some string slicing and concatenation. Other then some annoying of-by-one
errors in indexing, very simple.


## Part 2

We need to unscramble a password. That is a bit more complicated. First, we need
to iterate over the instruction in the reverse order, that's easy:
`for line in reversed(instructions):`. Then, some instructions work the same
both ways, eg. swapping letters. Others are easily reversed, eg. for rotation
I just change direction. Except one instruction, for rotating string based on a
position of a letter. I wasn't able to figure out how to reverse the
formula, so I ended up trying all possibilities, getting the numbers and putting
them in a lookup table, eg. if we're rotating on a letter that's on the 1st position,
we need to rotate by 7.

```python
#!/usr/bin/python3

INPUTFILE="21-input.txt"
STARTPASS="abcdefgh"
ENDPASS="fbgdceah"

def swap_positions(text, pos1, pos2):
    char1, char2 = text[pos1], text[pos2]
    tmppass=text[:pos1]+char2+text[pos1+1:]
    text=tmppass[:pos2]+char1+tmppass[pos2+1:]
    return text

def move_positions(text :str, pos1, pos2, inv):
    if inv:
        pos1,pos2=pos2,pos1
    char1=text[pos1]
    tmppass=text[:pos1]+text[pos1+1:]
    return tmppass[:pos2]+char1+tmppass[pos2:]

def swap_letters(text, char1, char2):
    pos1=text.index(char1)
    pos2=text.index(char2)
    tmppass=text[:pos1]+char2+text[pos1+1:]
    text=tmppass[:pos2]+char1+tmppass[pos2+1:]
    return text

def rotate(text, steps, direction, inv):
    if inv:
        direction = "right" if direction=="left" else "left"
    if direction=="right":
        steps=-steps
    return text[steps:]+text[:steps]

def rotate_on_letter(text, char1, inv):
    invrot = [7, 7, 2, 6, 1, 5, 0, 4]
    pos1=text.index(char1)
    if inv:
        steps= -invrot[pos1]
    else:
        steps=pos1 + (2 if pos1>=4 else 1)
    if steps>len(text):
        steps-=len(text)
    return rotate(text, steps, "right", inv)


def reverse(text, pos1, pos2):
    tmppass=text[pos1:pos2+1]
    return text[:pos1]+tmppass[::-1]+text[pos2+1:]

def run_instruction(password, line, inv=False, debug=False):
    if debug: print(password)
    match line.split():
        case ("swap", "position", pos1, "with", "position", pos2):
            return swap_positions(password, int(pos1), int(pos2))
        case ("swap", "letter", l1, "with", "letter", l2):
            return swap_letters(password, l1, l2)
        case ("rotate", direction, numsteps, _):
            return rotate(password, int(numsteps), direction, inv)
        case ("rotate", "based", "on", "position", "of", "letter", letter):
            return rotate_on_letter(password, letter, inv)
        case ("reverse", "positions", pos1, "through", pos2):
            return reverse(password, int(pos1), int(pos2))
        case ("move", "position", pos1, "to", "position", pos2):
            return move_positions(password, int(pos1), int(pos2), inv)
        case _:
            raise ValueError("unknown instruction")
    


with open(INPUTFILE) as f:
    instructions=f.read().splitlines()

password=STARTPASS
for line in instructions:
    password=run_instruction(password,line,False)
print(f"Part 1: {password}")

password=ENDPASS
for line in reversed(instructions):
    password=run_instruction(password,line,inv=True)
print(f"Part 2: {password}")
```