Title: 2015 day 19: Medicine for Rudolph
Date: 2023-05-12 17:32
Status: hidden
Category: Python
Series: 2015

## Part 1

We need a medicine for Rudolph. We've got a starting molecule and a list of possible replacements. First part was too easy: calculate how many
molecules can be made with a single replacements. Tricky bits are:

* when creating new molecule, I added them to the set and checked the set's length, that way I filtered out duplicate results,
* I had to use a tricky way to replace first, second..., nth occurence.


## Part 2

Then the second part: how many steps we need to reach the target molecule from a single electron? I admit I
had to check how other people did it. It was obvious for me I should check the other way around: from the
molecule to the electron. And a proper parser should help. I could write them when I was at the uni, but I
happily forgot them and hope to never need them again. I'm OK with regexps and such, but I run away screaming
when I hear about CNF, context-free grammar and so on.

Some people had luck with a randomized brute-force search: try to apply all the replacement rules, if a dead-end
was reached, shuffle the rules and try again. They said they reached a single electron in only a few shuffles.
I allowed for 200 thousands and only got some short molecules that couldn't be reduced any further.

I also tried some order in my replacements. I noticed there are different types of rules: some duplicate the
element, some generate a group containing Rn, Ar and something in between. Some generate an element from
an electron. I tried running them in different orders, trying to eliminate longer pieces first. Reached a dead
end every time.

In desperation, I based my solution on the analisys by CdiTheKing: <https://www.reddit.com/r/adventofcode/comments/3xflz8/comment/cy4h7ji/?utm_source=share&utm_medium=web2x&context=3>

It feels like cheating because I barely understand the reasoning and couldn't reach it myself. But I don't want
to spend more time on the puzzle that's no fun.

## Code

```python
#!/usr/bin/python3

import re

INPUTFILE="19-input.txt"


replacements=[]
with open(INPUTFILE) as inputfile:
    done_replacements=False
    for line in inputfile:
        line=line.strip()
        if line=='':
            done_replacements=True
            continue
        elif done_replacements:
            molecule=line
            break
        else:
            replacements.append( line.split(" => ") )

print("molecule: ", molecule)
#print("replacements: ", replacements)

# part 1
newmolecules=set()
for to_find,to_replace in replacements:
    for m in re.finditer(to_find, molecule): # gives list of positions where to_find string can be found in molecule
        index=m.start()
        #print(f"Replacing {to_find} with {to_replace} on position {index}")
        newmolecule=molecule[:index] + to_replace + molecule[index+len(to_find):]
        newmolecules.add(newmolecule) # need to add to a set, not just count, to skip duplicates
    
print("Part 1, number of molecules: ", len(newmolecules))

# part 2

num_capitals=sum(1 for char in molecule if char.isupper())
rx1=re.compile( r"Y" )
rx2=re.compile( r"Rn" )
rx3=re.compile( r"Ar" )
num_y =len(rx1.findall(molecule))
num_rn=len(rx2.findall(molecule))
num_ar=len(rx3.findall(molecule))
#print(f" atoms {num_capitals} y {num_y}  rn {num_rn} ar {num_ar}"  )
print("Part 2, min number of transforms: ", num_capitals-num_rn-num_ar-2*num_y-1)
```