# Command · `sudo du -sh /* | sort -h`

```text
mottap@victoryjack:~$ sudo du -sh /* | sort -h
[sudo: authenticate] Password:
du: no se puede acceder '/proc/2164/task/2164/fd/8': No such file or directory
du: no se puede acceder '/proc/2164/task/2164/fdinfo/8': No such file or directory
du: no se puede acceder '/proc/2164/task/2165/fd/8': No such file or directory
du: no se puede acceder '/proc/2164/task/2165/fdinfo/8': No such file or directory
du: no se puede acceder '/proc/2164/fd/6': No such file or directory
du: no se puede acceder '/proc/2164/fdinfo/6': No such file or directory
0       /bin
0       /dev
0       /lib
0       /lib64
0       /proc
0       /sbin
0       /sys
0       /tmp
4,0K    /cdrom
4,0K    /media
4,0K    /mnt
4,0K    /srv
16K     /lost+found
36K     /root
44K     /home
1,1M    /run
6,3M    /etc
95M     /boot
598M    /snap
1,1G    /opt
1,2G    /var
1,7G    /swap.img
2,5G    /usr
mottap@victoryjack:~$
```