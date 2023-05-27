Title: 2016 day 05: How About a Nice Game of Chess?
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We're generating password by making a lot of md5 hashes and extracting specific characters. Nothing special, there's a handy md5 function
in hashlib module from the standard library. For part2, I store the password as an array of characters, not string (one thing that would
be easier in C...), convert on printing. Looks a bit messy, but makes it easier to insert characters at a specific position.

```python
#!/usr/bin/python3

import hashlib

doorid="uqwqemis"

i=0
p1password=""
p2password=['-','-','-','-','-','-','-','-']
part1done=False
part2done=False

while True:
    newid=doorid+str(i)
    newhash=hashlib.md5(newid.encode()).hexdigest()
    if i%1000000==0: # print stats so we can see some progress
        print(f"Iteration {i} newid {newid} md5 {newhash}")
    if newhash.startswith("00000"):
        #part1
        if not part1done:
            p1password+=newhash[5]
            print ("Found character for part 1, current password: ", p1password)
            if len(p1password)==8:
                print("Part 1 done")
                part1done=True
        #part2
        if not part2done:
            if newhash[5] in ("0","1","2","3","4","5","6","7"):
                newpos=int(newhash[5])
                if p2password[newpos]=="-":
                    p2password[newpos]=newhash[6]
                    print ("Found character for part 2, current password: ", "".join(p2password))
                    if not "-" in p2password:
                        print("Part 2 done")
                        part2done=True
    if part1done and part2done:
        break
    i+=1 

print("Part 1, password: ", p1password)
print("Part 2, password: ", "".join(p2password))
```