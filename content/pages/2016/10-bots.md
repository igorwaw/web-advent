Title: 2016 day 10: Balance Bots
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

The instruction wasn't very clear, but here's the situation. We have 2 types of bot commands:

- take microchip: this can always be executed
- give microchip to another bot or place it in the output: this can only be executed if the bot has 2 microchips

So here's my take: I process the input file, execute the first type of instructions immediately and put other in the queue.
Then I iterate over the instruction queue until it's empty: if the bot is ready, command is executed and removed from the queue.


```python
#!/usr/bin/python3

from collections import defaultdict
from dataclasses import dataclass

INPUTFILE="10-input.txt"

@dataclass
class BotInstruction:
    source: int
    type1: str
    target1: int
    type2: str
    target2: int

    def execute(self) -> bool:
        global bots
        if bots[self.source].check_and_swap():
            print(f"Part 1: {self.source}")
        bots[self.source].give(self.type1, self.target1, self.type2, self.target2)

@dataclass
class OutputBin:
    chip: int = 0

    def add(self, value: int) -> None:
        self.chip=value

@dataclass
class Bot:
    chip1: int = 0
    chip2: int = 0

    def add(self, value: int) -> None:
        if not self.chip1:
            self.chip1=value
        else:
            self.chip2=value

    def is_ready(self) -> bool:
        return self.chip1 > 0 and self.chip2 > 0

    def check_and_swap(self) -> bool:
        if self.chip1>self.chip2:
            self.chip1, self.chip2 = self.chip2, self.chip1
        return self.chip1==17 and self.chip2==61
            

    def give(self, type1: str, target1: int, type2: str, target2: int) -> None:
        global bots
        # print(f"{self.chip1} to {type1} {target1}, {self.chip2} to {type2} {target2} ")
        if type1=="bot":
            bots[target1].add(self.chip1)
        else:
            bins[target1].add(self.chip1)
        if type2=="bot":
            bots[target2].add(self.chip2)
        else:
            bins[target2].add(self.chip2)
        self.chip1=0
        self.chip2=0


# MAIN

bots=defaultdict(Bot)
bins=defaultdict(OutputBin)
instructions=[]

with open(INPUTFILE) as f:
    for line in f:
        if line.startswith("value"):
            _, value, _, _, _, botnum = line.split()
            bots[int(botnum)].add(int(value))
        else:
            _, firstbot, _, _, _, type1, target1, _, _, _, type2, target2 = line.split()        
            instructions.append(BotInstruction(int(firstbot), type1, int(target1), type2, int(target2)) )

while instructions:
    for cmd in instructions.copy():
        if bots[cmd.source].is_ready():
            instructions.remove(cmd)
            cmd.execute()


part2 = bins[0].chip * bins[1].chip * bins[2].chip
print(f"Part 2: {part2}")
```