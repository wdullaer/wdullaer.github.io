---
author: Wouter Dullaert
comments: true
date: 2010-06-23 15:55:57+00:00
layout: post
slug: useful-linux-scripts
title: "Useful Linux Scripts"
excerpt: "We didn't come up with this one, but it's a very handy reference guide. In good medieval monastery style, we're copying it here in order to safeguard this information for future generations."
wordpress_id: 92
categories:
- Tools
tags:
- administration
- cli
- linux
- script
- server
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

We didn't come up with this one, but it's a very handy reference guide. In good medieval monastery style, we're copying it here in order to safeguard this information for future generations.

## PS command:

The PS command is useful to check for performance problems:

1. Displaying top CPU-consuming processes:

    ```bash
    ps aux | head -1; ps aux | sort -rn +2 | head -10
    ```

2. Displaying top 10 memory-consuming processes:

    ```bash
    ps aux | head -1; ps aux | sort -rn +3 | head
    ```

3. Displaying process in order of being penalized:

    ```bash
    ps -eakl | head -1; ps -eakl | sort -rn +5
    ```

4. Displaying process in order of priority:

    ```bash
    ps -eakl | sort -n +6 | head
    ```

5. Displaying process in order of nice value

    ```bash
    ps -eakl | sort -n +7
    ```

6. Displaying the process in order of time

    ```bash
    ps vx | head -1;ps vx | grep -v PID | sort -rn +3 | head -10
    ```

7. Displaying the process in order of real memory use

    ```bash
    ps vx | head -1; ps vx | grep -v PID | sort -rn +6 | head -10
    ```

8. Displaying the process in order of I/O

    ```bash
    ps vx | head -1; ps vx | grep -v PID | sort -rn +4 | head -10
    ```

9. Displaying WLM classes

    ```bash
    ps -a -o pid, user, class, pcpu, pmem, args
    ```

10. Determinimg process ID of wait processes:

    ```bash
    ps vg | head -1; ps vg | grep -w wait
    ```

11. Wait process bound to CPU (replace PID with the actual process number)

    ```bash
    ps -mo THREAD -p PID
    ```


## lsof Command
1. List all open files:

    ```bash
    lsof
    ```

2. List all open Internet, x.25 (HP-UX), and UNIX domain files:

    ```bash
    lsof -i -U
    ```

3. List all open IPv4 network files in use by the process whose PID is 1234:

    ```bash
    lsof -i 4 -a -p 1234
    ```

4. List all files using any protocol on ports 513, 514, or 515 of host wonderland.cc.purdue.edu:

    ```bash
    lsof -i @wonderland.cc.purdue.edu:513-515
    ```

5. List all files using any protocol on any port of mace.cc.purdue.edu (cc.purdue.edu is the default domain)::

    ```bash
    lsof -i @mace
    ```  

6. List all open files for login name "abe", or user ID 1234, or process 456, or process 123, or process 789:

    ```bash
    lsof -p 456,123,789 -u 1234,abe
    ```

7. List all open files on device /dev/hd4:

    ```bash
    lsof /dev/hd4
    ```

8. Find the process that has /u/abe/foo open:

    ```bash
    lsof /u/abe/foo
    ```

9. Send a SIGHUP to the processes that have /u/abe/bar open:

    ```bash
    kill -HUP `lsof -t /u/abe/bar`
    ```

10. Find any open file, including an open UNIX domain socket file, with the name /dev/log:

    ```bash
    lsof /dev/log
    ```

11. Find processes with open files on the NFS file system named /nfs/mount/point whose server is  inaccessible, and presuming your mount table supplies the device number for /nfs/mount/point:

    ```bash
    lsof -b /nfs/mount/point
    ```

12. Do the preceding search with warning messages suppressed:

    ```bash
    lsof -bw /nfs/mount/point
    ```

13. Ignore the device cache file:

    ```bash
    lsof -Di
    ```

14. Obtain PID and command name field output for each process, file descriptor, file device number, and file inode number for each file of each process:

    ```bash
    lsof -FpcfDi
    ```

15. List the files at descriptors 1 and 3 of every process running the lsof command for login ID "abe" every 10 seconds:

    ```bash
    lsof -c lsof -a -d 1 -d 3 -u abe -r10
    ```

16. List the current working directory of processes running a command that is exactly four characters long and has an  `o` or `O` in character three with this regular expression form of the -c c option:

    ```bash
    lsof -c /^..o.$/i -a -d cwd
    ```

17. Find an IP version 4 socket file by its associated numeric dot-form address:

    ```bash
    lsof -i@128.210.15.17
    ```

18. Display list of open ports:

    ```bash
    lsof -i
    ```

19. List information about TCP sessions on your server (specifically SSH in this example):

    ```bash
    lsof -i tcp@`hostname`:22
    ```

20. List information about all TCP session:

    ```bash
    lsof -i tcp@`hostname`
    ```

21. List information about all sockets using port 53 (will display named information on UDP/TCP)

    ```bash
    lsof -i @`hostname`:53
    ```

22. List information about all UDP sessions

    ```bash
    lsof -i udp@`hostname`
    ```

23. List all open files with "ssh" in them:

    ```bash
    lsof -c ssh
    ```

24. List everything but with UIDs insted of the UID name from /etc/passwd:

    ```bash
    lsof -l
    ```

25. List all open files with "ssh" and only the UIDs:

    ```bash
    lsof -l -c ssh
    ```

26. List all open files for the /tmp dir. Very slow, but good for finding that nasty process that's holding a file open (although:  fuser -m /tmp, will do the same thing):

    ```bash
    lsof+D /tmp
    ```


## fuser and netstat Commands
1. Kill all processes accessing the file system /home in any way:

    ```bash
    fuser -km /home
    ```

2. Invoke something if no other process is using /dev/ttyS1:

    ```bash
    if fuser -s /dev/ttyS1; then :; else something; fi
    ```

3. Some Important Command to find DDOS Attack:

    ```bash
    fuser telnet/tcp shows all processes at the (local) TELNET port.
    ```
    ```bash
    netstat -anp |grep 'tcp\|udp' | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
    ```
    ```bash
    netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
    ```
    ```bash
    netstat -ntu | grep -v TIME_WAIT | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
    ```
    ```bash
    netstat -an | grep :80 | awk '{print $5}' | cut -f1 -d":" | sort | uniq -c | sort -n
    ````

4. netstat command example:

    ```bash
    netstat –listen
    ```

5. Display open ports and established TCP connections:

    ```bash
    netstat -vatn
    ```

6. For UDP port try following command:

    ```bash
    netstat -vaun
    ```

7. If you want to see FQDN then remove -n flag:

    ```bash
    netstat -vat
    ```
