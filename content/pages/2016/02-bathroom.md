Title: 2016 day 02: Bathroom Security
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

2D grid with a twist. I wrote a simple solution using way too many elifs.
Could be made a bit shorter by using hexadecimal numbers. Or much shorter
by using a loopup table. But I didn't bother, let's go to the more interesting
tasks.

```python
#!/usr/bin/python3

INPUTFILE="02-input.txt"

#part 1
current="5"
code1=[]
with open(INPUTFILE) as f:
    for line in f:
        line=line.strip()
        for i in line:
            if i == "D":
                if current in "123456":
                    current=str(int(current)+3)
            elif i == "L":
                if current not in "147":
                    current=str(int(current)-1)
            elif i == "R":
                if current not in "369":
                    current=str(int(current)+1)
            elif i == "U":
                if current in "456789":
                    current=str(int(current)-3)
        code1.append(current)
print("Part 1 code: ", ''.join(code1))

#part 2
#TODO: make the code shorter using hexadecimals

current="5"
code2=[]
with open(INPUTFILE) as f:
    for line in f:
        line=line.strip()
        for i in line:
            if i == "D":
                if current in "234":
                    current=str(int(current)+4)
                elif current=="1":
                    current="3"
                elif current=="6":
                    current="A"
                elif current=="7":
                    current="B"
                elif current=="8":
                    current="C"
                elif current == "B":
                    current="D"
            elif i == "L":
                if current in "346789":
                    current=str(int(current)-1)
                elif current=="B":
                    current="A"
                elif current=="C":
                    current="B"
            elif i == "R":
                if current in "235678":
                    current=str(int(current)+1)
                elif current=="A":
                    current="B"
                elif current=="B":
                    current="C"
            elif i == "U":
                if current in "678":
                    current=str(int(current)-4)
                elif current=="3":
                    current="1"
                elif current=="A":
                    current="6"
                elif current=="B":
                    current="7"
                elif current=="C":
                    current="8"
                elif current=="D":
                    current="B"

        code2.append(current)
print("Part 2 code: ", ''.join(code2))
```