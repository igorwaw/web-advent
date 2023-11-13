Title: 2015 day 12: JSAbacusFramework.io
Date: 2023-05-09 17:32
Status: hidden
Category: Python
Series: 2015

For part 1, we need to sum all numbers in the JSON file. We don't really need to parse JSON,
just extract all the numbers and sum them. A simple regular expression will do for
extracting and classic map/reduce for the sum. I usually prefer a longer code for
redability's sake, but this just asks for putting all the contents of the main loop in
one line: `result+=sum( map(int, rx.findall(line) ) )`. 

How does it work? `rx.findall(line)` gives a list of all numbers in the line, using the following
regexp: `\-?\d+` meaning 0 or 1 minus sign followed by 1 or more digits. Then `map(int, rx.findall(line) )`
runs int() on every element of the list to create the list of integers. And then, just sum the list.

For part 2, we need to ignore JSON objects (but not lists) that contain a value "red".
So now we actually have to parse the JSON. I used a recursive function that iterates
through the data, checks types, adds integers, skips dictionaries if any value is "red",
recurse into lists and dictionaries.

Note that explicit type checking is usually frowned upon by Python programmers, but I
believe it's the right tool for this job.

```python
#!/usr/bin/python3

import re
import json

INPUTFILE="12-input.txt"

def parse_data(dict_or_list):
    retval=0
    if isinstance(dict_or_list, dict):
        for v in dict_or_list.values():
            if v=="red":
                return 0
            if isinstance(v, int):
                retval+=v
            elif isinstance(v, (dict, list)):
                retval+=parse_data(v)
    if isinstance(dict_or_list, list):
        for v in dict_or_list:
            if isinstance(v, int):
                retval+=v
            elif isinstance(v, (dict, list)):
                retval+=parse_data(v)
    return retval



# part 1

result=0
rx=re.compile( r'\-?\d+' )
with open(INPUTFILE) as inputfile:
    for line in inputfile:
        result+=sum( map(int, rx.findall(line) ) )
print("Part 1, sum: ", result)


# part 2

with open(INPUTFILE) as inputfile:
    data=json.load(inputfile)

result2=parse_data(data)
print("Part 2, sum: ", result2)
```