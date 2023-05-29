Title: 2016 day 16: Dragon Checksum
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We need to generate data that would fill a disk of a specific size,
than calculate the checksum. Algorithm for generating and for checksum
is described, unusual but nothing complicated.

For part 1, disk has a size 272 and for part 2: 35651584. This year, many
problems were easy to solve with a simple algorithm for part 1, but part 2
made a problem space much bigger, requiring a clever solution. If that was
supposed to be one of them, it didn't work. Sure, I guess there is a way
to calculate the checksum without actually generating and then iterating
over 35 million characters. But my program runs in 7 seconds and uses less
RAM then the editor I used to write it.

```python
#!/usr/bin/python3

INPUT="00111101111101000"

def dragon(a: str) ->str:
    b="".join("0" if c=="1" else "1" for c in reversed(a))
    return a+"0"+b

def checksum(data: str) -> str:
    csum=""
    for i in range(0,len(data)-1,2):
        csum += "1" if data[i]==data[i+1] else "0"
    return csum if len(csum)%2 else checksum(csum)

def get_checksum_for_disk(initval: str, size: int):
    data=initval
    while len(data)<=size:
        data=dragon(data)
    data=data[:size]
    return checksum(data)

print(f"Part 1: {get_checksum_for_disk(INPUT,272)}")
print(f"Part 2: {get_checksum_for_disk(INPUT,35651584)}")
```