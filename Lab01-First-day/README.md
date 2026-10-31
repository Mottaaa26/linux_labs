## Scenario

You just joined a small company. The previous sysadmin left without writing any documentation, and you've been handed a server (your VM). Your manager has left you three tasks for the day.

## Lab Rules

- Don't look up the full solution online. Search for **concepts** ("how to check disk usage per folder in Linux") and use `man`, `tldr`, or `--help`.
- Write down everything you try, including what doesn't work.
- Take a snapshot of the VM before you start.

---

## Mission 1 · "Can this server handle it?"

> _"We want to install an application that requires Ubuntu 22.04 or later, at least 2 GB of RAM, 2 CPUs, and 10 GB of free disk space. Does this server meet the requirements? If not, tell me what's missing."_

### Hints

- Many commands that show hardware information start with `ls…` (short for _list_).
- Memory is either "free" or not; there's a command that shows free space per filesystem.
- The OS version is stored in a file inside `/etc`.

### Deliverable

A short answer to your manager ("it meets / doesn't meet the requirements because…"), with evidence for each requirement. This is also where your server spec sheet comes from.

---

## Mission 2 · "Alert: the disk is filling up"

### Scenario Setup

Run this without analyzing it too much; it simulates the mess the previous sysadmin left behind:

```bash
sudo mkdir -p /opt/app_antigua/cache
sudo fallocate -l 1G /opt/app_antigua/cache/.datos_temp
sudo fallocate -l 500M /var/tmp/respaldo_2023.tar.gz
```

> _"We got an alert that free disk space dropped. Find out what's taking up space, decide what can be safely deleted, and free it up."_

### Hints

- There's a command that shows space usage **per folder**, not just per disk. Combine it with `sort` to order the results.
- `find` can search for files by size.
- Watch out: some files don't show up with a plain `ls`.
- Ubuntu caches the packages it downloads; find out how to clean that cache.

### Questions to Answer

- Why is it a bad idea to delete files from `/usr` or `/var/log` without checking first?
- What criteria did you use to decide that a file was safe to delete?

### Deliverable

Free space before and after, what you found, and what you decided to delete and why.

---

## Mission 3 · "Did someone try to break in?"

### Scenario Setup

From PowerShell, try to connect 3 times with a wrong password, and 2 times with a user that doesn't exist (for example, `admin`). Then log in normally.

> _"Security flagged some suspicious login attempts. Find out how many failed attempts there were, which usernames were used, from which IP address, and at what time."_

### Hints

- System logs live in a folder inside `/var`. Look for one related to authentication.
- You can also check the logs of a specific service with `journalctl`.
- `grep` filters lines that contain a given word. Which word appears in the failed attempts?

### Questions to Answer

- Is the IP address shown in the logs your computer's IP? Why not? (Hint: look into how VirtualBox's NAT mode works.)
- If this were a server exposed to the internet, what measures would you take?

### Deliverable

A timeline of the attempts (time, username, IP address, result).

---

## Bonus Mission · "How is access configured?"

> _"Which port is SSH listening on, and is direct root login allowed?"_

Find the answer in the service's configuration, and confirm it with a command that shows open ports.