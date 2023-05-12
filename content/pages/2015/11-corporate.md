Title: 2015 day 11: Corporate Policy
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

Santa needs a new password that meets the "security" requirements. The puzzle
is very similar to day 5, finding naughty strings.

## New Python features

**zip** is a function that allows to iterate on two or more sequences at once. Usually it's used over
two or more lists, dicts, tuples etc. Example:

```python
>>> countries=["Austria", "United Kingdom", "Denmark", "Sweden", "Norway", "Poland", "Germany"]
>>> areacodes=[43, 44, 45, 46, 47, 48, 49]
>>> for country, code in zip(countries,areacodes):
...     print(f"Area code for {country} is {code}")
... 
Area code for Austria is 43
Area code for United Kingdom is 44
Area code for Denmark is 45
Area code for Sweden is 46
Area code for Norway is 47
Area code for Poland is 48
Area code for Germany is 49
```

Here I used it like this: `for ch1, ch2, ch3 in zip (text, text[1:], text[2:])` which means it combines 3 sequences:
some string, the same string starting from the second characted and the same string starting from the 3rd character.
On the first iteration, it returns characters 0, 1, 2; on the second iteration 1, 2, 3 etc. Note that the sequences
have different length, second and third one are 1 and 2 characters shorter from the first one. It doesn't matter for
the zip function, it just ends when the shortest sequence ends.

**ord** returns the Unicode (or ASCII, it's the same in this case) code of a character, **chr** is an opposite
function that returns character for the given code. In C and C++ I could just do arithmetic operations directly
on the characters, suprisingly Python typing is sometimes more strict, you can't just treat character as a number
without explicitly converting it first.

## Code


```python
#!/usr/bin/python3

import re

FORBIDDEN=['i', 'o', 'l']
rx1=re.compile(r"(\w)\1.*(\w)\2")


password="cqjxjnds"

def increment(text: str) ->str:
    char_array=list(text)
    for i in range(7,-1,-1):
        char_array[i]=chr(ord( char_array[i] ) + 1)
        if char_array[i]>'z':
            char_array[i]='a'
        else:
            break
    return "".join(char_array)

def rule1(text: str) -> bool: # 3 characters in a sequence
    for ch1, ch2, ch3 in zip (text, text[1:], text[2:]):
        if (ord(ch3)-ord(ch2)==1) and (ord(ch2)-ord(ch1)==1):
            return True
    return False

def rule2(text: str) -> bool:  # does not contain forbidden characters
    for char in text:
        if char in FORBIDDEN:
            return False
    return True

def rule3(text: str) -> bool:  # contains at least 2 different pairs
    if pairsearch := rx1.search(text):
        pair1, pair2= pairsearch.groups()
        if pair1 != pair2: # and they are different
                return True
    return False


print ("Old password: ", password)

while(True):
    password=increment(password)
    if rule1(password) and rule2(password) and rule3(password):
        break
print("Part 1: ", password)
while(True):
    password=increment(password)
    if rule1(password) and rule2(password) and rule3(password):
        break
print("Part 2: ", password)
```