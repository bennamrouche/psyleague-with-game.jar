**Note: this is a very early version, most probably contains some bugs / invalid descriptions. If you encounter any problems, please msg me on twitter/discord**

Simple cmd-line league system for bot contests.
Install the latest version with `pip install psyleague --upgrade`
AFAIK, it requires python 3.8 or newer.

## Main Features
- League with simple automatic matchmaking system: Start the league server, add bots, look at the results table
- Add/remove/modify bots without restarting the server
- All data stored in human-readable formats
- [Soon!] Support for different ranking models
## Quick Setup Guide
- Install `psyleague`
- Create a script that given two bots, simulates the game and prints out space-separated list of ranks, i.e. if first player wins print "0 1", if the second player wins print "1 0", in case of a draw print "0 0".
- Run `psyleague config` in your contest directory to create a new config file
- In `psyleague.cfg` you have to modify `cmd_bot_setup` and `cmd_play_game`. `cmd_bot_setup` is executed immediately when you add a new game. `cmd_play_game` is executed when `psyleague` wants to play a single game. %DIR% -> `dir_bots` (from config), %NAME% -> `BOT_NAME`, %SRC% -> `SOURCE` (or BOT_NAME if SOURCE was not provided), %P1% & %P2% -> `BOT_NAME` of the player 1 & 2 bots.
- Run `psyleague run` in a terminal - this is the "server" part that automatically plays games. In order to kill it, use keyboard interrupt (ctrl+C).
- In a different terminal, start adding bots by running `psyleague bot add BOT_NAME -s SOURCE` to add a new bot to the league. As soon as you have 2 bots added, `psyleague run` will start playing games.
- Run `psyleague show` to see the current leaderboard
- Remember to update `n_workers` 

## Debugging
- Make sure that your script for playing games works correctly and the only thing it prints to the stdout is the list of integers on a single line. Anything printed to stderr will be ignored. For example, the following python code is going to work for most of the CodinGame contests, assuming that `referee.jar` contains the modified judge that accepts two bots:
```
import sys, subprocess, random
if __name__ == '__main__':
    cmd = f'java -jar referee.jar -p1 "{sys.argv[1]}" -p2 "{sys.argv[2]}" -d seed={random.randrange(0, 2**31)}'
    task = subprocess.run(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.DEVNULL)
    output = task.stdout.decode('UTF-8').split('\n')
    p1_score = int(output[-5])
    p2_score = int(output[-4])
    if (p1_score > p2_score):
        print("0 1")
    elif (p1_score < p2_score):
        print("1 0")
    else:
        print("0 0")
```
- Set `n_workers` to 1 and run server with verbose turned on: `psyleague run --verbose` to see if your `cmd_play_game` script is called correctly. 
        
## Ranking Models
- trueskill
	-  this is the same model that CodinGame uses, except here you have the ability to reduce `tau` so that the ranking can stabilize after a while. Unless you're running 10K+ games per bot, there's probably no reason to reduce `tau` even more. 
## MatchMaking
## Other Details
- **This library was not yet properly tested. There's a (very small?) chance of corrupting results and/or major slowdown with many bots/games.**
- **Most of the referees for CodinGame detect if your bot goes above allowed time for each turn. If you spawn too many workers your bots will start timing out randomly.**
- Config file is read only once at the startup. If you have updated config file, you have to restart `psyleague run` to reflect the changes
- You can modify `psyleague.db` to make direct changes to the bots/stats, but don't do that while `psyleague run` is running. In order to reset everything, it's enough to delete `psyleague.db` & `psyleague.games`.
- If you want to see the list of planned changes, see [the top of the source file](https://github.com/FakePsyho/psyleague/blob/main/psyleague/psyleague.py)
- Currently `psyleague.games` is used only for logging, but in the future you'll have the ability to recalculate ratings under a different ranking model

