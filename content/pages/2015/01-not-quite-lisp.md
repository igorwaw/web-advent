Title: 2015 day 1: Not Quite Lisp
Date: 2023-04-15 17:31
Status: hidden
Category: C
Series: 2015

Santa is trying to deliver presents in a large apartment building. He starts on the ground floor - floor 0 - and needs to travel up and down. 
The instructions are a list of brackets, "(" means go up and ")" means go down. We need to find:

- For part 1: what is the last floor?
- For part 2: at which instruction Santa will enter the basement for the first time?

That's clearly just a warm-up. All the operations are: reading one character/byte at time, comparison, incrementing/decrementing. I could solve it in any programming languge I know and possibly in some languages I don't know. So I used C while I still could.


```C
#include <stdio.h>
#include <stdbool.h>

const char* FILENAME="01-input.txt";

int main() {
    FILE* fileptr;
    char direction;
    int current_floor=0;
    int instr_counter=0;
    fileptr = fopen(FILENAME, "r");
    if (!fileptr) {
        printf("file: %s can't be opened\n", FILENAME);
        return 1;
    }

    bool above_ground=true;
    while ( (direction=getc(fileptr)) != EOF ) {
        ++instr_counter;
        if (direction=='(')
            ++current_floor;
        else if (direction==')')
            --current_floor;
        else {
            printf("Wrong direction: %c\n",direction);
            return 2;
        }
        if (above_ground && current_floor<0) { // entering basement first time
            above_ground=false;
            printf("Answer for part 2 (step when basement entered): %d\n", instr_counter);
        }
    }
    printf("Answer for part 1 (last floor): %d\n", current_floor);
}
```