Title: 2015 day 05: Doesn't He Have Intern-Elves For This?
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

Santa needs help figuring out which strings in his text file are naughty or nice. There are different
rules for part 1 and 2. So the main program just calls the function *is_nice1* and *is_nice2* for each line of the input:

```python
nice_1=0
nice_2=0
with open(INPUTFILE) as inputfile:
    for line in inputfile:
        if is_nice_1(line.rstrip()):  nice_1+=1
        if is_nice_2(line.rstrip()):  nice_2+=1
    
print("Number of nice lines (version 1): ", nice_1)
print("Number of nice lines (version 2): ", nice_2)
```

## Nice rules

The easiest thing to check is rule 3 of part 1: It does not contain the strings ab, cd, pq, or xy, even if they are part of one of the other requirements.
All we need is a list of naughty strings. We can then iterate over it and check the input string:

```python
    for pair in NAUGHTYPAIRS:
        if pair in chstring:
            return False
```

Then, there are regular expressions. Personally, I don't like to construct complicated regexps, they are prone to bugs. These are not so bad yet. All are precompiled and checked the same way:

```python
        if not rx1.search(chstring):
            return False
```

and the regexps are:

```python
rx1=re.compile(r"(\w)\1")
rx2=re.compile(r'(\w)\w\1')
rx3=re.compile(r'(\w\w)\w*\1')
```

Here's the meanining:

- **rx1** -  \w  means any letter, in parentheses means remember it, \1 mean repeat the first parentheses, so combined it means a pair of letters
- **rx2** -  (\w) and \1 has the same meaning as in rx1 and there's another letter in between
- **rx3** -  (\w\w) is a 2-letter group, \1 is the same group, repeated and there's \w* in between, meaning any number of letters (including 0)

A slightly more interesting case is rule 1 of part 1: It contains at least three vowels (aeiou only). Interesting, because it can be solved with a useful Python class called Counter. It can count occurences of all characters in the string. We then need to check vowelcounter[letter] for letter in VOWELS (wow, Python code is exactly the same as English explanation!) and sum it:

```python
VOWELS=("a", "e", "i", "o", "u")
vowelcounter=Counter(chstring)
vowelcount = sum(vowelcounter[letter] for letter in VOWELS)
if vowelcount<3:
    return False
```

## Full code below

```python
#!/usr/bin/python3

from collections import Counter
import re

INPUTFILE="05-input.txt"


NAUGHTYPAIRS=("ab", "cd", "pq", "xy")
VOWELS=("a", "e", "i", "o", "u")
rx1=re.compile(r"(\w)\1")
rx2=re.compile(r'(\w)\w\1')
rx3=re.compile(r'(\w\w)\w*\1')

def is_nice_1(chstring: str) -> bool:
    for pair in NAUGHTYPAIRS:
        # condition: doesn't contain naughty pair
        if pair in chstring:
            #print ("naughty pair: ",chstring)
            return False
        # condition: at least 3 vowels
        vowelcounter=Counter(chstring)
        vowelcount = sum(vowelcounter[letter] for letter in VOWELS)
        if vowelcount<3:
            #print (f"less than 3 vowels ({vowelcount}): {chstring}")
            return False
        # condition: letter appears 2 times in a row
        if not rx1.search(chstring):
            #print ("no repeated character: ", chstring)
            return False
    #print("nice: ", chstring)
    return True
            
def is_nice_2(chstring: str) ->bool:
    # condition: contains a letter which repeats with another letter between (xyx)
    if not rx2.search(chstring):
        print ("no (xyx): ", chstring)
        return False
    if not rx3.search(chstring):
        print ("no repeated 2-char group: ", chstring)
        return False
    print("nice: ", chstring)
    return True
 
    
nice_1=0
nice_2=0
with open(INPUTFILE) as inputfile:
    for line in inputfile:
        if is_nice_1(line.rstrip()):  nice_1+=1
        if is_nice_2(line.rstrip()):  nice_2+=1
    
print("Number of nice lines (version 1): ", nice_1)
print("Number of nice lines (version 2): ", nice_2)


```