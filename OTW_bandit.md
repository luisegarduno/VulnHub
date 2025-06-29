
# Solutions

- Level 0: (Start Here)
- Level 0 → Level 1: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
- Level 1 → Level 2: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx
- Level 2 → Level 3: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
- Level 3 → Level 4: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
- Level 4 → Level 5: 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
- Level 5 → Level 6: HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
- Level 6 → Level 7: morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
- Level 7 → Level 8: dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
- Level 8 → Level 9: 4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
- Level 9 → Level 10: FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
- Level 10 → Level 11: dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
- Level 11 → Level 12: 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
- Level 12 → Level 13: FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
- Level 13 → Level 14: MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
- Level 14 → Level 15: 8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
- Level 15 → Level 16: kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
- Level 16 → Level 17: EReVavePLFHtFlFsjn3hyzMlvSuSAcRD
- Level 17 → Level 18: x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO
- Level 18 → Level 19: cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8
- Level 19 → Level 20: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
- Level 20 → Level 21: EeoULMCra2q0dSkYj561DX7s1CpBuOBt
- Level 21 → Level 22: tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q
- Level 22 → Level 23: 0Zf11ioIjMVN551jX3CmStKLYqjk54Ga
- Level 23 → Level 24: gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 

---------------------------

# Walkthrough

Level 0: Start Here
- Options: enable ssh on windows if not done already | wsl | 

ls -halt

- `h`: lists file sizes in human readable format (KB, MB) when combined with -hl
- `a`: lists all files + directories including the hidden files in the current folder or directory
- `t`: sorts files & directories by modification time (newest at the top) 

*cd* Examples:
| Command | Description |
| ------- | ----------- |
| `cd` | (take us home) <- show example using gui |
| `cd foldername` | go to folder |
| `cd .`  | go to current folder |
| `cd ..` | go "back" up one directory level |
| `cd /`  | go to highest path |
| `cd ./` | go to current folder |

*ls* Examples:
| Command | Description |
| ------- | ----------- |
| `ls /`  | list files & directories in highest path mention (important example) |
| `ls ..` | list files & directories in directory one level up |
| `ls ./` | list files & directories in current directory (important example) |
| `ls .`  | list files & directories in current directory |
| `ls foldername/` | list files & directories in a folder located within current directory |
           
**Level 0 → Level 1**: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
- bring up tab to autocomplete
- mention pasting passwords (linux ctrl+shift+v, windows ctrl+v or right click)

## Level 1
**Level 1 → Level 2**: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx
- pwd: helps with different between ./ (current folder) and "/" (full path) 

## Level 2
**Level 2 → Level 3**: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

## Level 3
**Level 3 → Level 4**: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

## Level 4
**Level 4 → Level 5**: 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
* SOLUTION:
```bash
file ./*
```

## Level 5
**Level 5 → Level 6**: HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
- human-readable makes it tricky, leave out for now
- when using the find command, the second argument is always the directory location where you'll be searching
- size n[cwbkMG]
  - File uses n units of space, rounding up.  The following suffixes can be used:
    - `b': for 512-byte blocks (this is the default if no suffix is used)
    - `c'    for bytes
    - `w'    for two-byte words
    - `k'    for Kilobytes (units of 1024 bytes)
    - `M'    for Megabytes (units of 1048576 bytes)
    - `G'    for Gigabytes (units of 1073741824 bytes)
* SOLUTION:
```bash
find inhere/ -size 1033c -not -executable
```

## Level 6
**Level 6 → Level 7**: morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
USE LS to check if data.txt exists

- find -help (looks for -user and -group
* SOLUTION:
```bash
find / -size 33c -user bandit7 -group bandit6
```

## Level 7
**Level 7 → Level 8**: dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
- grep --help (SEE EXAMPLE LINE 3)
* SOLUTION:
```bash
grep "millionth" data.txt
```

## Level 8
**Level 8 → Level 9**: 4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
USE LS to check if data.txt exists

- unsure where to start
- assuming "strings" prints out a list of all the strings in a file
    - could save the output to a new file
    - or use pipe "|" - connect the output of one command directly to the input of another command
- "sort" can sort things (such as strings alphabetically)
- "uniq" helps find uniq things
    - uniq --help (find -c first, then also find -u)
* SOLUTION:
```bash
strings data.txt | sort | uniq -u
```
- mention the important of ordering
- we can verify by using "grep xxxxxx data.txt"

## Level 9
**Level 9 → Level 10**: FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

USE LS to check if data.txt exists
- Take note that the instructions mentions "STRINGS"
    - use the "strings" command
- Then sort them
    - Mention that we can scroll to find the answer but better to know to do efficiently
- add grep to the pipe
* SOLUTION:
```bash
strings data.txt | sort | grep "===="
```

## Level 10
**Level 10 → Level 11**: dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr

USE LS to check if data.txt exists

- Instructions mention data.txt is ENCODED in base64
- We verify this by running 'cat data.txt'
- see if base64 --help mentions anything about decoding

*SOLUTION*:
```bash
cat data.txt | base64 -d
```
