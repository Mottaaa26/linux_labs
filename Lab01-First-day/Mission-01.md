# Mission 1 · Can This Server Handle It?

> _"We want to install an application that requires Ubuntu 22.04 or later, at least 2 GB of RAM, 2 CPUs, and 10 GB of free disk space. Does this server meet the requirements? If not, tell me what's missing."_

## Problem

I need to check the server's operating system, CPUs, memory, and free disk space, compare them to the requirements established by the application, and report the result to my manager.

## Investigation

First, I wanted to know which command I could use to see the operating system information. I searched for it and found `hostnamectl`. I thought the command for this was `uname -a`, but that command shows kernel information, not the OS version.

Then I wanted to know about memory; for that I can use `free -h`.

For the CPU information, I can use the `lscpu` command.

Finally, for the disk information I have `df -h`. The `-h` flag means "human-readable": it shows sizes in K, M, and G instead of raw bytes.

I didn't know the difference between GiB and GB. Linux tools like `free -h` and `df -h` use binary units (1 GiB = 1024³ bytes = 1,073,741,824 bytes), while requirements usually use decimal units (1 GB = 1000³ bytes = 1,000,000,000 bytes). That's why I converted the values before comparing them.

## Requirements Check

| Requirement      | Needed                | This server                | Meets? | Command used  |
| ---------------- | --------------------- | -------------------------- | ------ | ------------- |
| Operating system | Ubuntu 22.04 or later | Ubuntu 26.04.1 LTS         | Yes    | `hostnamectl` |
| RAM              | 2 GB                  | 1.6 GiB (≈ 1.72 GB)        | No     | `free -h`     |
| CPUs             | 2                     | 1                          | No     | `lscpu`       |
| Free disk space  | 10 GB                 | 4.5 GiB (≈ 4.83 GB) on `/` | No     | `df -h`       |

_Data collected on: 2026-10-27, 01:19 (UTC−5)_

## Commands Explained

### `hostnamectl`

Part of systemd, which Ubuntu uses to manage system settings. It lets you inspect and modify host information without manually editing system files. The field I used is `Operating System`.

```text
Operating System: Ubuntu 26.04.1 LTS
```

Full output: [m1-hostnamectl.md](commands/mission01/m1-hostnamectl.md)

### `free -h`

`free` reads memory statistics from `/proc/meminfo` and prints them in a compact table. The `-h` flag ("human-readable") converts raw byte counts into units like MiB and GiB, so the output is easier to interpret. The field I used is `total` in the `Mem` row.

```text
               total        used        free      shared  buff/cache   available
Mem:           1,6Gi       350Mi       1,1Gi       1,0Mi       319Mi       1,3Gi
```

Full output: [m1-free-h.md](commands/mission01/m1-free-h.md)

### `lscpu`

Part of the **util-linux** package. It provides a structured summary of CPU and architecture details, reading data from `/proc/cpuinfo`, sysfs, and other kernel interfaces. The field I used is `CPU(s)`.

```text
CPU(s):                      1
  Model name:                AMD Ryzen 5 5500U with Radeon Graphics
```

`Model name` shows the host's physical CPU, but the VM can only use the number of CPUs assigned to it in VirtualBox.

Full output: [m1-lscpu.md](commands/mission01/m1-lscpu.md)

### `df -h`

Stands for **disk free**. The `-h` flag means **human-readable**, converting raw byte counts into K, M, G, or T. It reports usage for **mounted filesystems**, not individual directories. The field I used is `Avail` on the filesystem mounted on `/`.

```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  4,8G  4,5G  52% /
```

Full output: [m1-df-h.md](commands/mission01/m1-df-h.md)

## Answer to My Manager

> The server **does not meet** the requirements. The OS version is sufficient, but:
>
> - It has **1 CPU** (2 required).
> - It has **1.6 GiB of usable RAM** (≈ 1.72 GB; 2 GB required).
> - It has **4.5 GiB of free disk space** on `/` (≈ 4.83 GB; 10 GB required).
>
> Since this is a virtual machine, all three can be fixed by increasing its resources in VirtualBox: 2 CPUs, at least 3 GB of RAM (part of the assigned memory is reserved by the system), and a larger root volume. Before enlarging the virtual disk, I'll check whether the LVM volume group still has unallocated space that can be added to `/`.

## Problems Encountered

| Problem                                                                                               | Cause                                                                                   | Solution                                                                            |
| ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| The VM has 2 GB of RAM assigned, but `free -h` shows only 1.6 GiB total                               | Part of the memory is reserved by the firmware and the kernel                           | Used the usable value for the comparison and recommended assigning more RAM         |
| At first I used the disk **Size** (9.8 GiB) as free space, so the disk seemed to meet the requirement | `Size` is the total capacity of the filesystem; the free space is in the `Avail` column | Corrected the value to 4.5 GiB available, so the disk does not meet the requirement |

## References

- `hostnamectl(1)` manual page — `man hostnamectl`
- `free(1)` manual page — `man free`
- `lscpu(1)` manual page — `man lscpu`
- `df(1)` manual page — `man df`