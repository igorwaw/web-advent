Title: 2015 day 25: Let It Snow
Date: 2023-05-04 17:32
Status: hidden
Category: Python
Series: 2015

## Part 1

Like with the previous day, I'm writing and coding simultanously. 

I skipped reading the input data. That's just two numbers, so I put them in the code. At this point I don't need to practice parsing simple text files, right? I also added the other constants from the instruction:

```python
TARGETROW=2981
TARGETCOL=3075

FIRSTCODE=20151125
MULTIPLIER=252533
DIVIDER=33554393
```

We have a row and column, which number is that? I suppose there is a mathematical formula that can give it, but I couldn't easily find one.
But computers are really fast at simple arithmetics, so let's just iterate until we find one. We're starting at the first number which is at row 0, column 0. I'll also keep track of maximum row and column reached so far and I'll use the full loop. Perhaps later I'll be able to simplify. Let's try first 10 codes to begin.

```python
row=col=maxrow=maxcol=n=1
while n<10
    if (row==maxrow and col==maxcol):
        row+=1
        maxrow+=1
        col=1
    elif (row>1):
        row-=1
        col+=1
        maxcol+=1
    elif (row==1):
        maxrow+=1
        row=maxrow
        col=1
```

OK, looks like that already covers all cases - and some of them twice! There's no need to check for maxcol, whenever we get to the 1st row, we are always at the last column. Let's simplify already. Also, we need to calculate the code, now we're only moving on the diagonals. Let's generate first 10 codes and compare with the sample:

```python
row=col=maxrow=n=1
newcode=FIRSTCODE
while n<10:
    if (row>1):
        row-=1
        col+=1
    else:
        maxrow+=1
        row=maxrow
        col=1
    n+=1
    newcode=(newcode*MULTIPLIER)%DIVIDER
    print(f"{n} row {row} col {col} code {newcode}")
```

It worked! So let's get the answer already. But there's one more thing to simplify. We don't need the maxrow variable - we're moving on diagonals, so when we're at row 1, we're at column equal to max row reached. Also, we don't need to know n.

## Part 2

There's no part2. And the puzzle is simple, even easier than day 23 and 24. Nice of the author to remember that people need to do other things on Christmas than coding.

But I'm solving it in May. So let's rewrite the code to C. It's only a simple loop and arithmetic operations, shouldn't be hard. One gotcha - when we multiply the code, we get number too big for a standard **int**, but **unsigned long long** works just fine.

Let's compare the performance:

```sh
$ time ./25-snow.py
Part 1: code 9132360

real	0m5,044s
user	0m4,932s
sys	0m0,016s

$ time ./25-snow
Part 1: code 9132360

real	0m0,097s
user	0m0,087s
sys	0m0,000s
```

That's over 50 000 time faster! 

## Source code - Python version

```python
#!/usr/bin/python3

TARGETROW=2981
TARGETCOL=3075

FIRSTCODE=20151125
MULTIPLIER=252533
DIVIDER=33554393

row=col=1
newcode=FIRSTCODE
while True:
    if (row>1):
        row-=1
        col+=1
    else:
        row=col+1
        col=1

    newcode=(newcode*MULTIPLIER)%DIVIDER
    if row==TARGETROW and col==TARGETCOL:
        break

print(f"Part 1: code {newcode}")
```

## Source code - C version

```c
#include <stdio.h>

int main() {
    const int TARGETROW=2981;
    const int TARGETCOL=3075;

    const unsigned long long FIRSTCODE=20151125;
    const unsigned long long MULTIPLIER=252533;
    const unsigned long long DIVIDER=33554393;

    int row=1;
    int col=1;
    unsigned long long newcode=FIRSTCODE;
    while (1) {
        if (row>1) {
            row-=1;
            col+=1;
        }
        else {
            row=col+1;
            col=1;
        }
        newcode=(newcode*MULTIPLIER)%DIVIDER;
        if (row==TARGETROW && col==TARGETCOL)
            break;
     }
    printf("Part 1: code %llu\n", newcode);

}
```
