Title: 2016 day 09: Explosives in Cyberspace
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2016

## Part 1

We have a compression algorithm: our input data contains markers such as (10x2) meaning take the next 10 characters and repeat them 2 times. "Compressed data" can also contain markers, but they should be skipped. We need to get the length of uncompressed file, skipping any whitespace.

Two things to consider: do we really decompress the data or just calculate the length? It's quite possible to calculate without reconstructing the string,
eg. marker (15x3) adds 14 characters (3*15 minus the original 15 minus the length of the marker 6). It is however possible that part 2 will require actually decompressing the data. I decided to take my chances.

Second consideration: how to parse the data? Two solutions come to mind immediately. First way, use regular expressions to find possible markers (easy) and - based on their position in the input - decide whether they are real markers or they are inside the data block of another marker. Second way, a state machine approach: parse the input character by character and do a different thing depending on the current state: starting with the state "no special block", after opening bracket switch to state "length", after "x" - "repeat", after closing bracket - "data", and after specified number of characters back to "no special block". This will automatically deal with markers inside data blocks, but is generally more complicated. Let's try the regexp solution.

First, let's prepare two input files: 09-input.txt contains my puzzle input and 09-small.txt contains test strings from the instruction. Now let's read the file, call function to get uncompressed length of each line (stub for now) and sum up everything.

```python
INPUTFILE="09-small.txt"

def get_uncompressed_length(string: str, debug: bool=False) -> int:
    return 1

with open(INPUTFILE) as f:
    totallength=0
    for line in f:
        line="".join(line.split())
        print(line)
        length=get_uncompressed_length(line)
        print (f"Uncompressed line length: {length}")
        totallength+=length

    print(f"Part 1: {totallength}")
```

That works. Just one explanation, `line="".join(line.split())` removes all whitespace from the line. Now to find the markers. Here's the regexp: `rx1=re.compile( r"\((\d+)x(\d+)\)" )`. Completely clear, as usual with regular expressions, right? Well, maybe not.

* it starts with `\(` which is an opening bracket and ends with a closing bracket `\)` - brackets need to be escaped by backslash because they have a special meaning
* then there's `(\d+)` - \d is a digit, + means it can be repeated (but there's at least one), brackets mean to remember it - it will be available as group(1)
* then x is just x
* and another group of digits, group(2)

We also need one special case: if the string checked doesn't contain any markers, just return its length, it won't change. If it does, first get the position of the beginning of the marker. Everything before that would be copied as-is, so again the length doesn't change. And then it's length*repeat. Then, we'll have to find the next marker, skipping the data block. But for now let's check if the regular expression is correct.

```python
def get_uncompressed_length(string: str, debug: bool=False) -> int:
    marker=rx1.search(string)
    if not marker:
        return len(string)
    # we got first marker
    retval=marker.start()
    length=int(marker.group(1))
    repeat=int(marker.group(2))
    if (debug):
        print(f"found marker: repeat {length} characters {repeat} times")
    retval+=length*repeat
    return retval
```

Great, how do we skip past the data block? With string slicing and iteration. 

```python
def get_uncompressed_length(string: str, debug: bool=False) -> int:
    retval=0
    while string:
        marker=rx1.search(string)
        if not marker:
            retval+=len(string)
            break
        else:
            # we got the marker
            retval+=marker.start()
            length=int(marker.group(1))
            repeat=int(marker.group(2))
            if (debug):
                print(f"found marker: repeat {length} characters {repeat} times")
            retval+=length*repeat
            newposition=marker.end()+length
            string=string[newposition:]
            if (debug):
                print(f"    searching new string {string}")
    return retval
```

It worked! Let's try with the full data. Also worked!

## Part 2

So now we also have to consider the markers inside the data blocks. And the instruction hints that it's better to calculate the size than actually uncompress, as the computer probably doesn't have enough memory. Looks like I made the right decision. 

So, all we have to do is a slight modification to the get_uncompressed_length function. We need to actually decompress the data block and call the get_uncompressed_length on it recursively. I'm not uncompressing the whole file at once, so hopefully I have enough RAM. As it turned out, my laptop
took several minutes to calculate, but RAM usage was minimal.

Why is it so slow? Let's check: `python3 -m cProfile ./09-explosives.py` shows that a lot of time is spent on various regexp functions.
The usual tradeoff: "state machine" approach would run much faster, if written in C - probably less than a second, but also take longer to code.

## Source code

```python
#!/usr/bin/python3

import re

INPUTFILE="09-input.txt"

rx1=re.compile( r"\((\d+)x(\d+)\)" )
 

def get_uncompressed_length(string: str, debug: bool=False, part2: bool=False) -> int:
    retval=0
    while string:
        marker=rx1.search(string)
        if not marker:
            retval+=len(string)
            break
        else:
            # we got the marker
            retval+=marker.start()
            length=int(marker.group(1))
            repeat=int(marker.group(2))
            if (debug):
                print(f"Checking {string}")
                print(f"    found marker: repeat {length} characters {repeat} times")
            if (part2):
                data_begin=marker.end()
                data_end=data_begin+length
                newdata=string[data_begin:data_end]*repeat
                retval+=get_uncompressed_length(newdata, debug=debug, part2=True)
            else:
                retval+=length*repeat
            newposition=marker.end()+length
            string=string[newposition:]
            if (debug):
                print(f"    searching new string {string}")
    return retval

with open(INPUTFILE) as f:
    part1length=0
    part2length=0
    for line in f:
        line="".join(line.split())
        part1length+=get_uncompressed_length(line, part2=False)
        part2length+=get_uncompressed_length(line, part2=True)

    print(f"Part 1: {part1length}")
    print(f"Part 2: {part2length}")
```