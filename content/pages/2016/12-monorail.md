Title: 2016 day 12: Leonardo's Monorail
Date: 2023-05-27 17:32
Status: hidden
Category: Python
Series: 2016

We need to simulate a computer with a very simple CPU architecture: 4 registers, 4 instructions. Easy. Very similar
to day 23 from 2015, in fact I copied some code. But at this point I learned how to use match/case more effectively,
not only to match strings, but also check arguments and extract them. Another trick I should have discovered sooner:
sometime the argument could be either a number or a register. How do we check for that in the parser? Simple, we don't,
we just run the command that reads the string, tries to convert it to int and if that fails - reads the register.

```python
#!/usr/bin/python3


INPUTFILE="12-input.txt"

class Computer:
    register: dict = {}
    pointer: int = 0
    instructions: list[str]

    def __init__(self, filename: str) -> None:
        with open(filename) as f:
            self.instructions = f.read().splitlines()
        self.reset()

    def reset(self) -> None:
        self.register["a"]=0
        self.register["b"]=0
        self.register["c"]=0
        self.register["d"]=0
        self.pointer=0


    def __repr__(self) -> str:
        return f"Registers: {self.register} pointer {self.pointer} current instruction {self.instructions[self.pointer]}"

    def get(self, arg: str) -> int:
        try:
            return int(arg)
        except ValueError:
            return self.register[arg]

    def run(self, debug=False) -> None:
        while self.pointer<len(self.instructions):
            cmd=self.instructions[self.pointer].split()
            if (debug): print(self)
            self.pointer+=1  # will have to decrease by 1 for jumps
            match cmd:
                case ("cpy", arg, target):
                    self.register[target]=self.get(arg)
                case ("inc", arg):
                    self.register[arg]+=1
                case ("dec", arg):
                    self.register[arg]-=1
                case ("jnz", testval, offset):
                    self.pointer += int(offset) -1 if self.get(testval)!=0 else 0
                case _: raise ValueError
        


computer=Computer("12-input.txt")
computer.run(debug=False)
print("Part 1, register A: ",computer.register["a"])
computer.reset()
computer.register["c"]=1
computer.run(debug=False)
print("Part 2, register A: ",computer.register["a"])
```