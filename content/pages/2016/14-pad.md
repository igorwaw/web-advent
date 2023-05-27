Title: 2016 day 14: One Time Pad
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We need to generate 64 MD5 hashes. The hash is correct if it has 3 of the same characters in a row AND in the next 1000 hashes
there's one that has the same character repeated 5 times. Easy. 

It's clear to see that naive solution would calculate same hash many times. There are many clever ways to overcome it, eg. we can
start by computing all the hashes and store them in a lookup table, or we can have some sort of a moving window and keep 1000 recent
hashes. But Python has a much simpler solution: we can cache results of the function by using just one import statement:
`from functools import cache` and prepending the function with a decorator `@cache`.

For part 2, we need to use key stretching, each hash is looped 2016 times. Even with caching, it takes about 30 seconds.
How long without cache? I tried disabling it, ran the code for 20 minutes with debug enabled and it only got to about
10%.

```python
#!/usr/bin/python3

import hashlib
import re
from functools import cache

SALT="ahsbgdzn"
has3=re.compile(r"(\w)\1\1")

@cache
def gethash(i: int) -> str:
    newid=SALT+str(i)
    return hashlib.md5(newid.encode()).hexdigest()
# execution time: 2.6 vs 0.5

@cache
def getstretchedhash(i: int) -> str:
    newid=SALT+str(i)
    stretchedhash= hashlib.md5(newid.encode()).hexdigest()
    for _ in range(2016):
        stretchedhash=hashlib.md5(stretchedhash.encode()).hexdigest()
    return stretchedhash

def check5(startindex: int, text: str, stretched=False) -> int:
    for i in range(startindex+1,startindex+1000):
        nexthash = getstretchedhash(i) if stretched else gethash(i)
        if text in nexthash:
            return i
    return 0

# part1

def get64hashes(stretched=False, debug=False) -> int:
    i=0
    num_hashes=0
    while True:
        nexthash=getstretchedhash(i) if stretched else gethash(i)
        if m:=has3.search((nexthash)):
            matched=m.group(1)
            if index_of_5:=check5(i,matched*5, stretched):
                num_hashes+=1
                if (debug):
                    print(f"Got new hash number {num_hashes} at index {i}")
                if num_hashes==64:
                    return i
        i+=1

print(f"Part 1: {get64hashes(stretched=False)}")
print(f"Part 2: {get64hashes(stretched=True)}")
```