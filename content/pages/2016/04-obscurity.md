Title: 2016 day 03: Security Through Obscurity
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

Two tasks. Part 1 was about calculating a checksum using an unusual algorithm. Two slightly
interesting things. Counter object, which greatly simplified the code. And a way to sort dict
descending by value, then ascending by key: `sorted(lettercounter, key=lambda x: (-x[1],x[0]))`.
Get it? Let's start from the inside. We define a lamba function for key, it returns a tuple.
x[1] is the value, - for reversing the sort order. Then x[0] is the key.

Part 2 was a simple cypher - addition, subtraction and modulo is all that's needed.

```python
#!/usr/bin/python3

import re
from collections import Counter


rx1=re.compile(r"([a-z-]+)([0-9]+)\[([a-z]+)\]")

INPUTFILE="04-input.txt"

def calculate_sum(name):
    name=name.replace("-","")
    lettercounter=Counter(name).most_common()
    sortedcounter=sorted(lettercounter, key=lambda x: (-x[1],x[0]))  # sort descending by value, then ascending by key
    checksum=""
    for l,_ in sortedcounter[:5]:
        checksum+=l
    return checksum

def decrypt_name(name, sector):
    newname=""
    for letter in name:
        if letter=="-":
            newletter=" "
        else:
            lettercode=ord(letter)
            lettercode+=sector%26
            if lettercode>ord("z"):
                lettercode-=26
            newletter=chr(lettercode)
        newname+=newletter
        #print(f"{letter} becomes {newletter}")
    return newname


sum_sector=0
with open(INPUTFILE) as f:
    for line in f:
        if roomsearch := rx1.search(line):
            roomname, roomsector, roomsum=roomsearch.groups()
            newsum=calculate_sum(roomname)
            if roomsum == newsum:
                newname=decrypt_name(roomname,int(roomsector))
                sum_sector+=int(roomsector)
                #print(f"Correct room: {roomname}, sector {roomsector}, checksum {roomsum}, new name {newname}")
                if "northpole" in newname:
                    print("Part 2, ID of northpole storage: ", roomsector)
        else:
            print("Wrong room format (or wrong regexp): ", line)
print("Part 1, sum of sectors: ", sum_sector)


```