Title: 2015 day 23: Opening the Turing Lock
Date: 2023-05-13 17:32
Status: hidden
Category: Python
Series: 2015

We're emulating a simple computer, with 2 registers and 6 instructions. Hardware-emulating puzzles are
always the most fun. And so was this one, except for one trick in the instruction. There's an
instruction jie - "jump if even", and next one is jio, which I automatically read as "jump if odd", although
the instruction explained it's "jump if one". Either I'm dumb or that trick is dumb, you decide.

Other than that, it's simple. Match/case to run the right command and some string parsing. Initial version had
6 more methods, one for each command, but since they were 2-6 lines and even that could be shortened, I moved all
of that directly into match/case. To be fair, I could even remove the whole class without affecting the readability
too much.

```python
#!/usr/bin/python3

class Computer:
    register: dict = {}
    pointer: int
    instructions: list

    def __init__(self, filename: str) -> None:
        with open(filename) as f:
            self.instructions = f.read().splitlines()
        self.reset()

    def reset(self) -> None:
        self.register["a"]=0
        self.register["b"]=0
        self.pointer=0


    def __repr__(self) -> str:
        return f"Registers: {self.register} pointer {self.pointer} current instruction {self.instructions[self.pointer]}"


    def run(self, debug=False) -> None:
        while self.pointer<len(self.instructions):
            if (debug): print(self)
            cmd = self.instructions[self.pointer][:3]
            arg = self.instructions[self.pointer][4:]
            self.pointer+=1  # will have to decrease by 1 for jumps
            match cmd:
                case "hlf": 
                    self.register[arg]=self.register[arg]//2
                case "tpl": 
                    self.register[arg]*=3
                case "inc": 
                    self.register[arg]+=1
                case "jmp": 
                    self.pointer+=int(arg)-1
                case "jie": 
                    register,offset=arg.split(",")
                    self.pointer += int(offset) -1 if self.register[register]%2==0 else 0
                case "jio": 
                    register,offset=arg.split(",")
                    self.pointer += int(offset) -1 if self.register[register]==1 else 0
                case _: raise ValueError
        


computer=Computer("23-input.txt")
computer.run()
print("Part 1, register B: ",computer.register["b"])
computer.reset()
computer.register["a"]=1
computer.run()
print("Part 1, register B: ",computer.register["b"])
```