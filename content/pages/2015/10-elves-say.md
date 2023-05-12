Title: 2015 day 10: Elves Look, Elves Say
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

We're checking how fast look-and-say sequence grows. I've never heard of it and it doesn't
look particularly useful, but appareantly it has some uses in number theory and was analyzed
by the famous John Conway. Wikipedia explains:

To generate a member of the sequence from the previous member, read off the digits of the previous member, counting the number of digits in groups of the same digit. For example:

* 1 is read off as "one 1" or 11.
* 11 is read off as "two 1s" or 21.
* 21 is read off as "one 2, one 1" or 1211.
* 1211 is read off as "one 1, one 2, two 1s" or 111221.
* 111221 is read off as "three 1s, two 2s, one 1" or 312211

Easy thing in any language with strings that:

* are dynamic length,
* can be treated as an array of characters,
* are reasonably fast, because they're going to get *really* long.

I think the code is self explanatory. We're iterating through the string and
comparing current character to the previous one. If it's the same - the streak continues.
If it's different - construct the part of the result as "streak length" and "previous character".
When the loop ends, do the same construction again. Special case: string of length 1.
Remember to convert integers to strings when needed. And that's it.

Not the prettiest code as there's some repetition, but it's readable and resonably fast.


```python
#!/usr/bin/python3

SEED=1113222113
PART1_ROUNDS=40
PART2_ROUNDS=50

def look_and_say(number: str) -> str:
    prev_char=number[0]
    streaklen=1
    if len(number)==1: # special case for 1-digit number
        return f'1{prev_char}'
    newnumber=""
    for i in range(1, len(number)):
        next_char=number[i]
        if next_char==prev_char: # streak continues
            streaklen+=1
        else: # streak ended, change character and create new string
            newnumber+=str(streaklen)+prev_char
            prev_char=next_char
            streaklen=1
    newnumber+=str(streaklen)+prev_char
    return newnumber



number=str(SEED)
for i in range(PART2_ROUNDS):
    #print(f"Round {i+1}  {number} becomes ", end="")
    number=look_and_say(number)
    #print(number)
    if i==PART1_ROUNDS-1:
        print("Part 1 result: ", len(number))

print("Part 2 result: ", len(number))
```