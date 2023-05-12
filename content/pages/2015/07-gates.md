Title: 2015 day 07: Some Assembly Required
Date: 2023-05-06 17:32
Status: hidden
Category: Python
Series: 2015

We've got a list of wires that feed signals to other wires through logic gates. We need to know the state of the wire a,
but for that we need to know the state of the wire lx first. And lx references 2 other wires and so on, until we get
to the wires that have their state directly in the input file. I see two ways to do it:

* try to get wire a and do recursion, until you get to the wires with known values
* or iterate over the list of instructions, at each iteration do all calculations that are possible  (the data is already available) and remove those instructions, until the instruction queue is empty

I thought it would be cool to see how we get to the result, so I used the second approach and displayed the state of the wires in columns,
with a delay at every step.

![screenshot]({static}/images/2015day7.png)

## Parsing input

This puzzle gave me some trouble with input parsing. First I borrowed the solution from <https://aoc.just2good.co.uk/2015/7> and
used the Parsimonious library because I wanted to get to the visualization quickly. It worked, but I wanted to have my own parser,
no cheating. So I rewrote it using only string splitting and comparisons. It also worked, but it was long and there was some code
repetition. Next, I used match/case. Much better! There's still some repetition, but "good enough" beats "perfect".

## Python feature: exceptions

I used one Python feature that I haven't used in 2016 puzzles yet: exceptions. Exceptions are used in many
programming languages, usually to handle error conditions. But here I'm using them for flow control, like this:

```python
try value=int(arg)
    # handle the case when arg can be converted to int
except ValueError:
    # handle the case when it can't
```

It's not an error that the value can't be converted, it's a completely normal case (in fact, the "exceptional" case happens more
often then the "standard" case). C++ or Java developers would shudder at such design, but in Python culture it's considered OK.
Maybe the reason is exceptions make programs slower in many languages, in Python there's not much difference (it's slow anyway...)

## Python feature: match/case

**Structural pattern matching** was a new feature of Python 3.10. At a first glance, it's similar to switch/case instructions you
might aready know from other languages such as C:

```python
match argument:
    case "something":
        do_something()
    case "whatever":
        some_other_stuff()
    case _:
        raise ValueError
```

Nothing wrong in using it this way, it's slightly shorter and more readable then a long chain of if/else instructions.
But that's only about 1% of what it can do, I've yet to learn all the posibilities. So far I know how to:

* match strings (obviously),
* assign variables while checking,
* check type of the matched argument,
* if the argument is a tuple, check how many elements it contains.

All of that while being more readable then the code written without this feature. In fact, probably
more readable than my attempt to explain it. Here's the relevant part of the code:

```python
    match splitinstr:=instr.split():
        case (arg1, "AND"|"OR", arg2): return process_andor(splitinstr[1], arg1, arg2, wire_values, targetwire)
        case (arg1, "LSHIFT"|"RSHIFT", arg2): return process_shift(splitinstr[1], arg1, int(arg2), wire_values, targetwire)
        case ("NOT", arg1): return process_not(arg1, wire_values, targetwire)
        case _:  return process_assingment(splitinstr[0], wire_values, targetwire)
```

Variable instr contains one instruction from the input file, the part before the -> delimiter, eg. "e OR f -> g",
"NOT dm -> dn". I then split it, assign the tuple to splitinstr and check the cases.

First two cases are 3-element tupples. If the middle element contains LSHIFT or RSHIFT, it call process_shift function,
"AND" or "OR" call process_andor. In both cases, first and third element are assigned to arg1 and arg2 respectively.
Next, a 2-element tupple, first a string "NOT" and then any value of any type, that gets assigned to arg1. Finally,
the default case. If it wasn't any of the 5 logical operations, it must be assignment. It's a puzzle with known
input, for the real production code I would check the input more carefuly.



## Visualization

Curses library forces a specific program structure. Everything that uses curses needs to be placed in one function (by convention, called main)
which takes *stdscr* as an argument, and stdscr is the main curses window that covers the entire screen. This function is then called with
`curses.wrapper(main)`. Or specifically, that's not the only way to run curses program, but it's the most convenient.

Curses allow to build quite sophisticated text-mode user interfaces, but I only use it for one thing: print text in a specific place on the
screen. I start at the top left corner: `outx=0` `outy=0` then I move right by the colum width: `outx+=COLWIDTH` which I found experimentally,
large enough to fit the longest instruction and keep some space from the next column. If I'm too close to the screen's edge, I move to the
next row and back to the left, the library provides screen width in curses.COLS:

```python
        if (outx>=curses.COLS-COLWIDTH):
            outx=0
            outy+=1
```

I also highlighted some values using a different output format. Curses allow changing foreground and background color and also setting
attributes such as bold or blink. I simpy used `curses.color_pair(1)` - color pairs can be user defined but have a default value,
and I didn't care what colors are used as long as they are different then the default.

## Code

```python
#!/usr/bin/python3


from collections import defaultdict
import curses
import re




INPUTFILE="07-input.txt"
COLWIDTH=20  
DELAY=300 # 300ms delay to see how signals are calculated
#DELAY=0 # or no delay to get the answer sooner




def display_wires(stdscr, wires, iteration, instructions):
    """ prints the state of all logic gates in multiple columns using curses """
    stdscr.clear()
    outx=0
    outy=0
    for wire,value in instructions.items():
        # try to get the signal value for the wire
        # if we don't have it, print instruction for the wire
        outformat=curses.A_NORMAL
        try:
            outstring = f"{wire}: {str(wires[wire])}"
            outformat=curses.color_pair(1) # higlight the wires that have the signal
        except KeyError:
            outstring = f"{wire}: {value}"


        stdscr.addstr(outy, outx, outstring, outformat)
        #stdscr.addstr(outstring, outformat)
        outx+=COLWIDTH
        if (outx>=curses.COLS-COLWIDTH):
            outx=0
            outy+=1
    # print summary below
    outstring = f"Iteration: {str(iteration)}"
    try:
        a_val=str(wires['a'])
        ending="     PRESS ANY KEY TO CONTINUE"
    except KeyError:
        a_val=instructions['a']
        ending=""
    outstring += f"  current state of a: {a_val}{ending}"
    stdscr.addstr(outy+3, 5, outstring, curses.color_pair(1) )
    stdscr.refresh()



def execute(line : str, wire_values: dict) -> bool:
    # returns true if instruction executed, false otherwise
    instr,targetwire=line.split(" -> ")
    #print(f"{targetwire}: instruction {instr}")

    match splitinstr:=instr.split():
        case (arg1, "AND"|"OR", arg2): return process_andor(splitinstr[1], arg1, arg2, wire_values, targetwire)
        case (arg1, "LSHIFT"|"RSHIFT", arg2): return process_shift(splitinstr[1], arg1, int(arg2), wire_values, targetwire)
        case ("NOT", arg1): return process_not(arg1, wire_values, targetwire)
        case _:  return process_assingment(splitinstr[0], wire_values, targetwire)


def process_not(sourcewire: str, wire_values: dict, targetwire: str) -> bool:
    if sourcewire in wire_values:
        # The ~ operator in Python may return a signed -ve value.
        # We don't want this, so we & with a 16 bit mask of all 1s to convert to +ve representation
        wire_values[targetwire] = ~wire_values[sourcewire] & 0xFFFF
        return True
    else:
        return False


def process_assingment(instr: str, wire_values: dict, targetwire: str) -> bool:
    try: # simplest case first: 123 -> x
        val=int(instr)
        wire_values[targetwire]=val
        return True
    except ValueError: # not a number - another gate then
        if instr in wire_values:
            wire_values[targetwire]=wire_values[instr]
            return True
        else:
            return False

def process_shift(operation: str, sourcewire: str, arg2: int, wire_values: dict, targetwire: str) -> bool:
    if sourcewire not in wire_values:
        return False
    if operation=="LSHIFT":
        wire_values[targetwire] = wire_values[sourcewire] << arg2
    else:
        wire_values[targetwire] = wire_values[sourcewire] >> arg2
    return True


def process_andor(operation: str, arg1: str, arg2: str, wire_values: dict, targetwire: str) -> bool:
    if arg2 not in wire_values:
        return False
    try: # special case - 1st argument is a value not wire
        value=int(arg1)
        wire_values[targetwire] = value & wire_values[arg2] if operation=="AND" else value | wire_values[arg2]
        return True
    except ValueError:
        # normal case - it's a wire
        if arg1 not in wire_values:
            return False
        wire_values[targetwire] = wire_values[arg1] & wire_values[arg2] if operation=="AND" else wire_values[arg1] | wire_values[arg2]
        return True


def process_instructions(data, stdscr, wireinstr):
    wire_values = {}
    iteration=0
    while data:
        iteration+=1
        for line in data[:]:
            if execute(line, wire_values):
                # if the instruction worked, remove it from the queue
                #print(f"removing: {line}")
                data.remove(line)
        #print(f"Iteration {iteration} instructions left {len(instructions)}")
        display_wires(stdscr, wire_values, iteration, wireinstr)
        curses.napms(DELAY) # delay so we can see how the values are changing
    return wire_values


def main(stdscr):
    # read input file
    with open(INPUTFILE) as inputfile:
        data=inputfile.read().splitlines()

    # dictionary of instructions, only needed for visualization
    wireinstr={}
    for line in data:
        instr,wire=line.split(" -> ")
        instr=instr.strip()
        wire=wire.strip()
        wireinstr[wire]=instr

    # some more initialization
    curses.init_pair(1, curses.COLOR_RED, curses.COLOR_WHITE)
    


    # part 1
    results = process_instructions(data.copy(), stdscr, wireinstr)  # main loop goes here
    part1result=results['a']
    stdscr.getch()
    
    # part 2
    wire_b_instr = list(filter(re.compile(r"-> b$").search, data)) # return only rows that match
    wire_b_instr_index = data.index(wire_b_instr[0])  # the position of this instruction in the list
    data[wire_b_instr_index] = f"{part1result} -> b"  # replace the instruction with this new one
    results = process_instructions(data.copy(), stdscr, wireinstr)  # main loop goes here
    part2result=results['a']
    stdscr.getch()

    return part1result,part2result


# wrapper to simplify init/cleanup of curses
part1result,part2result=curses.wrapper(main)

# print results again to the normal output
print("Part 1: Value of input a is ", part1result)
print("Part 2: Value of input a is ", part2result)

```