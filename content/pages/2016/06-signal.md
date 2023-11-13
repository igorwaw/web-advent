Title: 2016 day 06: Signals and Noise
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We need to find the most common and the least common letter in each column
of the input. Counter class again.

```python
#!/usr/bin/python3

from collections import Counter

INPUTFILE="06-input.txt"

inputcols=[]

with open(INPUTFILE) as f:
    for line in f:
        for i,letter in enumerate( line.strip() ): # iterate over letters + get column number
            try:
                inputcols[i]+=letter
            except IndexError: # we don't have a string for this column yet (we're on line 1 of file)
                inputcols.append(letter)


message1=""
message2=""


for col in inputcols:
    lettercount=Counter(col).most_common()
    letter1=lettercount[0][0]
    letter2=lettercount[-1][0]
    message1+=letter1
    message2+=letter2

print("Part 1, message: ", message1)
print("Part 2, message: ", message2)

```