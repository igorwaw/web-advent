Title: 2015 day 22: Wizard Simulator 20XX
Date: 2023-04-19 17:32
Status: hidden
Category: Python
Series: 2015

That was quite annoying. Coding the initial version didn't take much time. But then I spent hours debugging it.
The result I was getting was ALMOST right - that is, of the multiple won games, I was getting the right one, but also another one with slightly
lower cost. Turned out I was checking some conditions in the wrong order: in that game, boss died of poison, but only after it killed the player.

Puzzle description seems quite similar to the previous one, but the code is very different. Previously the number of combinations was small and it was easy to generate all of them in advance. This is different, there are millions of possible games and the only way to check is to play them all. I used a lot of
dataclasses:

* Spell holds both active effects and spells that can be cast, to simplify the code, each spell has all effects (eg. decrease boss HP, increase player HP, increase mana), but most of them have value 0.
* Game holds all of the current game's data: player's and boss's HP and other stats, list of active effects. In the beginning I had a separate class for player and boss - which is a more obvious solution - but since they are heavily coupled and each game has exactly one player and one boss, I decided to combine them.
* Gamestats records statistics, for debugging and extra fun. It keeps number of games won, lost and pruned, plus for the won games (but only if they are better than other won games) a more detailed report: list of spells and player's/boss's HP and the beginning of each player's round. The code is slow, to see progress and help with debugging, I added a Curses user interface that displays gamestats.

The most important game function, **game_round**, uses recursion run a new copy with every possible new spell. The same function covers 2 cases: player round and boss round, so it needs to pass one variable during recursion and make sure it alternates between the two.

Some optimizations to speed up the code:

* I check the game cost (mana spent) and if it's already higher than the best game, that branch is pruned.
* I also initially set the best game cost to some impossibly high value that I guessed instead of infinity, that way some branches are pruned even before the first won game
* Stats are ony displayed once every 10 000 games

It still needs about a minute to go through all possible combinations.


```python
#!/usr/bin/python3

from dataclasses import dataclass
from copy import deepcopy, copy
import curses



@dataclass
class Spell:
    name: str
    cost: int
    damage: int
    hpplus: int
    armor: int
    manaplus: int
    duration: int
    number: int



@dataclass
class Gamestats:
    gamerecorder: list
    games_played: int=0
    games_won: int=0
    games_lost: int=0
    games_pruned: int=0
    minimal_cost_game: int=9999 # initialize with impossibly high value

    def clear(self):
        self.gamerecorder=[]
        self.games_played=0
        self.games_won=0
        self.games_lost=0
        self.games_pruned=0
        self.minimal_cost_game=9999

@dataclass
class Game:
    player_hp: int
    mana: int
    armor: int
    mana_spent: int
    player_attack: int
    active_effects: list
    boss_hp: int
    boss_attack: int
    game_record: str


    def check_player_end(self) -> bool:
        global stats
        if self.player_hp<=0: 
            stats.games_played+=1
            stats.games_lost+=1
            return True
        return False

    def check_boss_end(self) -> bool:
        global stats
        if self.boss_hp<=0:
            stats.games_played+=1
            stats.games_won+=1
            if self.mana_spent<stats.minimal_cost_game: # only record better games
                gamedata=f"mana {self.mana} spent: {self.mana_spent} spells: {self.game_record} "
                stats.gamerecorder.append(gamedata)
            stats.minimal_cost_game = min(stats.minimal_cost_game, self.mana_spent)
            return True
        return False

    def check_pruned(self) -> bool:
        global stats
        # if we already spent more than the best game, no need to continue
        if self.mana_spent>stats.minimal_cost_game:
            stats.games_played+=1
            stats.games_pruned+=1
            return True
        return False



    def do_effects(self):
        for spell in copy(self.active_effects):
            self.armor+=spell.armor
            self.player_attack+=spell.damage
            self.mana+=spell.manaplus
            spell.duration-=1
            if spell.duration<=0:
                self.active_effects.remove(spell)
        


    def cast_spell(self, spell: Spell) -> bool:
        self.mana-=spell.cost
        self.mana_spent+=spell.cost
        if spell.number in [1, 2]: # missile or drain - instant effect
            self.boss_hp-=spell.damage
            self.player_hp+=spell.hpplus
            if self.check_boss_end():
                return True
        else: # delayed effect
            self.active_effects.append(copy(spell))
        self.game_record+=f"{spell.name} {self.player_hp}/{self.boss_hp}|"
        return self.check_pruned()



    def reset_effects(self):
        self.armor=0
        self.player_attack=0








spells = [
    Spell("Missile", 53, 4, 0, 0, 0, 0, 1),
    Spell("Drain", 73, 2, 2, 0, 0, 0, 2),
    Spell("Shield", 113, 0, 0, 7, 0, 6, 3),
    Spell("Poison", 173, 3, 0, 0, 0, 6, 4),
    Spell("Recharge", 229, 0, 0, 0, 101, 5, 5),
]



# each round:
# player: (spell effects, test hp, test mana), player casts new spell, optionally (missile and drain) spell has effect, test hp
# boss: (spell effects, test hp, boss attacks, test hp)


def display_stats(stdscr: curses.window, stats: Gamestats):
    try:
        outx=0
        outy=2
        outformat=curses.A_NORMAL
        outstring=f"Games played: {stats.games_played:,}"
        stdscr.addstr(outy, outx, outstring, outformat)
        outy+=1
        outstring=f"Won:          {stats.games_won:,}"
        stdscr.addstr(outy, outx, outstring, outformat)
        outy+=1
        outstring=f"Lost:         {stats.games_lost:,}"
        stdscr.addstr(outy, outx, outstring, outformat)
        outy+=1
        outstring=f"Pruned:       {stats.games_pruned:,}"
        stdscr.addstr(outy, outx, outstring, outformat)
        outy+=1
        outformat=curses.A_BOLD
        outstring = f"Lowest cost {stats.minimal_cost_game}"
        stdscr.addstr(outy, outx, outstring, outformat)
        outformat=curses.A_NORMAL
        outy+=1
        for i in stats.gamerecorder:
            outy+=1
            stdscr.addstr(outy, outx, i, outformat)

        stdscr.refresh()
    except Exception:
        pass

def top_message(stdscr: curses.window, message: str):
    outformat=curses.A_REVERSE
    stdscr.addstr(0, 0, message, outformat)
    stdscr.refresh()

def game_round(game: Game, stdscr: curses.window, part2, player_round=True):
    global spells, stats
    if stats.games_played%10000==0: # display stats every 10000 games
        display_stats(stdscr, stats)

    if player_round and part2:
        game.player_hp-=1
        if game.check_player_end():
            return

    game.do_effects()
    game.boss_hp-=game.player_attack # only does something if poison is active
    if game.check_boss_end(): # check if poison killed the bos
        return

    if player_round:
        game.reset_effects()
        # try to cast spell
        could_not_cast=True
        for spell in spells:
            already_active = any(effect.number==spell.number for effect in game.active_effects)
            if not already_active and game.mana>=spell.cost:
                could_not_cast=False
                newgame=deepcopy(game)
                if (newgame.cast_spell(spell)): # boss died
                    return
                game_round(newgame, stdscr, player_round=False, part2=part2)
        if could_not_cast: 
            game.player_hp=0
            game.check_player_end() # record lost game
            return
    else: # boss round
        # boss attacks
        attack = max(game.boss_attack-game.armor, 1)
        game.player_hp-=attack 
        game.reset_effects()
        if game.check_player_end():
            return
        game_round(game, stdscr, player_round=True, part2=part2)


def main(stdscr: curses.window):
    global stats
    game=Game(boss_attack=10, boss_hp=71, player_hp=50, mana=500, mana_spent=0, armor=0, player_attack=0, active_effects=[], game_record="")
    game_round(game, stdscr, part2=False)
    part1result=stats.minimal_cost_game
    top_message(stdscr, "Part 1 done, press any key to continue")
    stdscr.getch() # wait for keypress
    stdscr.clear()
    game=Game(boss_attack=10, boss_hp=71, player_hp=50, mana=500, mana_spent=0, armor=0, player_attack=0, active_effects=[], game_record="")
    stats.clear()
    game_round(game, stdscr, part2=True)
    top_message(stdscr, "Part 2 done, press any key to continue")
    stdscr.getch() # wait for keypress
    part2result=stats.minimal_cost_game
    
    return part1result,part2result


if (__name__=="__main__"):
    stats=Gamestats(gamerecorder=[])
    # wrapper to simplify init/cleanup of curses
    part1result,part2result=curses.wrapper(main)
    # print results again to the normal output
    print("Part 1:  ", part1result)
    print("Part 2:  ", part2result)
```

