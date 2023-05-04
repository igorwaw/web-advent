Title: 2015 day 04: The Ideal Stocking Stuffer
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

Santa is mining AdventCoins for gifts. We need to calculate some MD5 hashes - Python
conveniently has a function for this in the standard library - until we find the right one.
Just one loop and some string comparisons.

```python
#!/usr/bin/python3

import hashlib


puzzleinput="iwrupvqb"

i=0
foundfirst=False
foundsecond=False
while not (foundfirst and foundsecond):
    nextval=puzzleinput+str(i)
    nexthash=hashlib.md5(nextval.encode()).hexdigest()
    if not foundfirst and nexthash.startswith("00000"):
        print("Found first hash for value ",i)
        foundfirst=True
    if not foundsecond and nexthash.startswith("000000"):
        print("Found second hash for value ",i)
        foundsecond=True
    i+=1



```