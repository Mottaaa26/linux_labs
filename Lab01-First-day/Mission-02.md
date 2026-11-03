# Mission 2 · Alert: The Disk Is Filling Up

## Exercise

> _"We got a disk usage alert. Find out what's taking up space and free whatever you can, without breaking anything. Keep in mind: the old application Carlos (the previous sysadmin) used was decommissioned a year ago, and our policy is to keep backups for 12 months. Before deleting anything, I want to know why it's safe to do so."_

I need to check the space used per folder, not only per disk, find the files that take up the most space, and delete only the ones that are unnecessary and safe to remove, in order to free space on the disk.

## Solution

I searched for a command that shows the space usage of a folder and found `du`. Then I looked it up in the terminal with `man du` and `du --help`.

To see the space usage of a folder I can use `du -sh /path/to/folder`: `-s` (`--summarize`) displays only a total for each argument, and `-h` (`--human-readable`) prints sizes in units like K, M, and G. For the `/` directory the result was 7.0G, but that is a single line. To see the usage of each folder inside `/` and sort it from smallest to largest, I used:

`sudo du -sh /* | sort -h`

The folders and files that take up the most space are:

- `/usr` → 2.5G
- `/swap.img` → 1.7G
- `/var` → 1.2G
- `/opt` → 1.1G

Now that I had the list, I needed a way to find the specific files that take up the most space. The lab recommends the `find` command, so I checked it with `man find` and `find --help`. The manual wasn't very clear to me, so I looked up usage examples on the internet, which guided me through the manual.

From `/`, I used `find` to look for regular files (`-type f`) bigger than 400 MB (`-size +400M`):

`sudo find . -type f -size +400M`

```text
./proc/kcore
find: ‘./proc/2753/task/2753/fdinfo/6’: No such file or directory
find: ‘./proc/2753/fdinfo/5’: No such file or directory
./opt/app_antigua/cache/.datos_temp
./swap.img
./var/tmp/respaldo_2023.tar.gz
```

Then I checked each result to decide whether it could be deleted without affecting the system:

- **`/proc/kcore`** is not a real file. It's the virtual representation of the system's physical memory, exposed by the kernel. Its size reflects the kernel's address space, not disk usage, so it doesn't take up any space on the disk.
- **`/opt/app_antigua/cache/.datos_temp`** is a hidden file (its name starts with `.`), so I used `ls -a` to see it. It belongs to the cache of the old application that was decommissioned a year ago, so I deleted it:

```text
mottap@victoryjack:/opt/app_antigua/cache$ ls -a
.  ..  .datos_temp
mottap@victoryjack:/opt/app_antigua/cache$ sudo rm .datos_temp
mottap@victoryjack:/opt/app_antigua/cache$ ls -a
.  ..
mottap@victoryjack:/opt/app_antigua/cache$
```

- **`/swap.img`** is a real 1.7G file that the kernel uses as swap space: when the RAM is full, it moves inactive memory pages there so processes can keep running. On this server it's about the same size as the RAM. The system needs it, so I kept it.
- **`/var/tmp/respaldo_2023.tar.gz`** is a backup from 2023. Unlike `/tmp`, which on this server is a `tmpfs` that is cleared on every reboot, `/var/tmp` keeps its data after a reboot, so this file would never be removed automatically. I checked the folder first:

```text
mottap@victoryjack:/var/tmp$ ls
respaldo_2023.tar.gz
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-ModemManager.service-YUeUSq
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-chrony.service-mnMVSe
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-fwupd.service-1wVh05
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-polkit.service-1a7j6i
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-systemd-logind.service-HWYweY
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-upower.service-zVIPhL
mottap@victoryjack:/var/tmp$
```

Since the backup is older than the 12-month retention policy, I deleted it:

```text
mottap@victoryjack:/var/tmp$ sudo rm -f respaldo_2023.tar.gz
[sudo: authenticate] Password:
mottap@victoryjack:/var/tmp$ ls -a
.
..
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-ModemManager.service-YUeUSq
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-chrony.service-mnMVSe
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-fwupd.service-1wVh05
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-polkit.service-1a7j6i
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-systemd-logind.service-HWYweY
systemd-private-b4fe1d97d2394bbaa3aec969e1ccb2d5-upower.service-zVIPhL
mottap@victoryjack:/var/tmp$
```

Finally, I checked Ubuntu's package cache and cleaned it with `apt clean`:

```text
mottap@victoryjack:/var/cache/apt/archives$ sudo du -sh /var/cache/apt/archives
17M     /var/cache/apt/archives
mottap@victoryjack:/var/cache/apt/archives$ sudo apt clean
mottap@victoryjack:/var/cache/apt/archives$ sudo du -sh /var/cache/apt/archives
16K     /var/cache/apt/archives
mottap@victoryjack:/var/cache/apt/archives$
```

Running `sudo du -sh /* | sort -h` again shows that `/opt` went from 1.1G to 12K and `/var` from 1.2G to 668M.

| Item                 | Location                   | Size         | Decision | Reason                                                                                  |
| -------------------- | -------------------------- | ------------ | -------- | --------------------------------------------------------------------------------------- |
| kcore                | `/proc/`                   | No real size | Keep     | Pseudo-file that represents the system's physical memory; it doesn't use disk space     |
| .datos_temp          | `/opt/app_antigua/cache/`  | 1G           | Delete   | Cache of an application decommissioned a year ago; nothing uses it                      |
| swap.img             | `/`                        | 1.7G         | Keep     | Swap space the kernel uses when the RAM is full; the system needs it                    |
| respaldo_2023.tar.gz | `/var/tmp/`                | 500M         | Delete   | Backup from 2023, older than the 12-month retention policy                              |
| APT package cache    | `/var/cache/apt/archives/` | 17M          | Clean    | Packages that are already installed; `apt` can download them again if they're needed   |

| Folder | Before | After |
| ------ | ------ | ----- |
| `/opt` | 1.1G   | 12K   |
| `/var` | 1.2G   | 668M  |

_Data collected on: 2026-11-03, 01:40 (UTC−5)_

**Why is it a bad idea to delete files from `/usr` or `/var/log` without checking first?** `/usr` contains the system's programs and libraries, installed and managed by `apt`; deleting files there can break commands or the whole system. In fact, `find` showed that `/usr` has no files bigger than 400 MB: its 2.5G is made of thousands of small library files. `/var/log` contains the logs needed for troubleshooting and security audits. Logs should be reduced with tools like `journalctl --vacuum-size` or log rotation, not deleted by hand.

**What criteria did I use to decide that a file was safe to delete?** Which application does it belong to, and is that application still in use? Does the system need it (like `swap.img` or the files in `/proc`)? Is it older than what the company policy requires to keep? Can it be regenerated if it's needed again (like the APT cache)?

Commands used:

- `sudo du -sh /* | sort -h` → [m2-du-sh-before.md](commands/mission02/m2-du-sh-before.md) · [m2-du-sh-after.md](commands/mission02/m2-du-sh-after.md)
- `sudo find . -type f -size +400M` (run from `/`) → [m2-find.md](commands/mission02/m2-find.md)
- `ls -a`, `sudo rm`, `sudo du -sh`, `sudo apt clean` → output shown above

## Answer to My Manager

> I freed about **1.5 GiB** of disk space:
>
> - Deleted `/opt/app_antigua/cache/.datos_temp` (1G), the cache of the application that was decommissioned a year ago.
> - Deleted `/var/tmp/respaldo_2023.tar.gz` (500M), a 2023 backup older than our 12-month retention policy.
> - Cleaned the APT package cache (17M).
>
> I kept `/swap.img` (1.7G) because the system uses it as swap space. `/usr` is large, but it only contains system programs and libraries, so nothing should be deleted there.
>
> Recommendations: the empty `/opt/app_antigua` folder is still on the server and can be removed too, since the application is no longer used. I also recommend storing backups outside the server, so they don't fill up the system disk.

## Problems Encountered

| Problem                                                     | Cause                                                                                                   | Solution                                                                                      |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `sudo find ./usr ...` returned *No such file or directory*  | I was already in `/usr`, so the relative path `./usr` pointed to `/usr/usr`                              | Used the absolute path `/usr` (or `.` for the current folder)                                 |
| `sudo find /usr -type f -size +400M` showed nothing         | It wasn't an error: there are no files bigger than 400M in `/usr`, only thousands of small library files | Confirmed with `sudo du -sh /usr/* \| sort -h` that most of the space is in `/usr/lib` (1.9G) |
| `du` and `find` showed *No such file or directory* under `/proc` | `/proc` is a virtual filesystem; processes appear and disappear while the command is reading it   | The errors don't affect the result; they can be hidden by redirecting errors with `2>/dev/null` |
