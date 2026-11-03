# Mission 2 · Alert: The Disk Is Filling Up

## Exercise

> _"We got an alert that free disk space dropped. Find out what's taking up space, decide what can be safely deleted, and free it up."_

Check the free space per folder and not by disc unit, so i can detele the unnecesary files in every folder in order to make space in the disk. Then order the results.

## Solution

I searched for a command that shows the space usage of a folder, the result was 'du' keyword, then i look up in the terminal the function with command: man du and du --help.

To see the space usage for a folder: du -sh /path/to/folder. We write du, then -sh that means --sumarize (display only a total for each argument) and --human-readable (print sizes in human readable format). The result in the / directory was 7,0 G. To sort the information i have to apply the next command:

`sudo du -sh /* | sort -h`

it list the files in the given path, the sort command is to make it readable for me.

Now that i have the list, i have to find a way to see the files that consumes the most space. The lab recommends the find command, so i check it with ``man find`` and ``find --help``.

The instructions are not quite clear, so i look up the command in internet. 

I found something. The web show use examples for the command and guided me trough the man find command.

The files with the most occupied space are:
* /usr -> 2,5G
* /swap.img -> 1,7G
* /var -> 1,2G
* /opt -> 1,1G

| Item | Location | Size | Decision | Reason |
| ---- | -------- | ---- | -------- | ------ |
|      |          |      |          |        |

_Data collected on: YYYY-MM-DD, HH:MM (UTC−5)_

Commands used:

- `sudo du -sh /* | sort -h` → [m2-du-sh.md](commands/mission02/m2-du-sh.md)

## Answer to My Manager

> The short answer to the request: the result, what I did, and what I recommend.

## Problems Encountered

| Problem | Cause | Solution |
| ------- | ----- | -------- |
|         |       |          |
