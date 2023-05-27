Title: 2016 day 20: Firewall Rules
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We have some blocklists and need to find IPs outside of them. IPs are 32-bit
numbers here, not the usual 4-octet notation. Good, that's easier to compare.
Blocklists re given in no particular order, sorting them is an obvious first
step. They can also overlap.

Once again it's an exercise in optimization. There are some nice, easy to code
ways to do it that would work for smaller spaces and look pythonic. Eg. I could
create ranges for blocklists, combine them with itertools.chain, unpack them into
one list and just check `if ip not combined_blocklist`. But such ways would take
too much time or memory to check 2^32 addresses. Solution? Code like it's
1980 and only operations you have are +, - and <. I only got pythonic for
input parsing because it only happens once.

```python
#!/usr/bin/python3


INPUTFILE="20-input.txt"
MAXIP=2**32

with open(INPUTFILE) as f:
    data=sorted(f.read().splitlines())
    ipranges=[ ( int(x),int(y) ) for x,y in [line.split("-")  for line in data ] ]

ipranges.sort()
num_valid=0
ip=0
part1done=False

for limit_l,limit_h in ipranges:
    if ip<limit_l: # outside the lower limit, valid IP
        num_valid+=limit_l-ip
        if not part1done:
            print(f"Part 1: {ip}")
            part1done=True
    ip=max(ip, limit_h+1) # skip past the end of the current blocklist
# add IPs from the end of the last blocklist till the max value
num_valid+=MAXIP-ip

print(f"Part 2: {num_valid}")
```