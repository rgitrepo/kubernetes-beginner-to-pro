Sure, here's a detailed tutorial for Linux commands with a table of contents:

# Table of Contents

1. [pwd](#1-pwd)
2. [cd](#2-cd)
3. [ls](#3-ls)
4. [mkdir](#4-mkdir)
5. [touch](#5-touch)
6. [cp](#6-cp)
7. [mv](#7-mv)
8. [rm](#8-rm)
9. [rmdir](#9-rmdir)
10. [cat](#10-cat)
11. [echo](#11-echo)
12. [vi and nano](#12-vi-and-nano)
13. [grep](#13-grep)
14. [ping](#14-ping)
15. [curl](#15-curl)
16. [wget](#16-wget)
17. [watch](#17-watch)
18. [base64](#18-base64)
19. [ps](#19-ps)
20. [grep for pipelines](#20-grep-for-pipelines)
21. [> and >>](#21-and)
22. [&> and 2>](#22-and-2)
23. [netstat](#23-netstat)
24. [tcpdump](#24-tcpdump)
25. [wc -l](#25-wc--l)
26. [chmod](#26-chmod)
27. [ssh](#27-ssh)
28. [tar](#28-tar)
29. [gzip](#29-gzip)
30. [scp](#30-scp)
31. [alias](#31-alias)
32. [clear](#32-clear)
33. [history](#33-history)
34. [head](#34-head)
35. [tail](#35-tail)
36. [crictl](#37-crictl)
37. [journalctl](#37-journalctl)

---

## 1. `pwd`
**Description**: Print Working Directory. It shows the current directory path.
**Usage**: To know the current directory you are in.
**Example**:
```sh
$ pwd
/home/user
```

## 2. `cd`
**Description**: Change Directory. It changes the current directory.
**Usage**: Navigate to a different directory.
**Examples**:
```sh
$ cd /home/user/Documents  # Changes to the Documents directory
$ cd ..                    # Goes up one directory level
$ cd ~                     # Changes to the home directory
$ cd -                     # Changes to the previous directory
```

## 3. `ls`
**Description**: List Directory Contents. It shows files and directories within the current directory.
**Usage**: View the contents of a directory.
**Examples**:
```sh
$ ls
Documents  Downloads  Music  Pictures  Videos

$ ls -l
total 8
drwxr-xr-x 2 user user 4096 Jul  8 12:00 Documents
drwxr-xr-x 2 user user 4096 Jul  8 12:00 Downloads

$ ls -a
.  ..  .bashrc  .profile  Documents  Downloads

$ ls -lh
total 8.0K
drwxr-xr-x 2 user user 4.0K Jul  8 12:00 Documents
drwxr-xr-x 2 user user 4.0K Jul  8 12:00 Downloads
```
**Important Flags**:
- `-l`: Use a long listing format.
- `-a`: Show all files including hidden files.
- `-h`: With `-l`, print sizes in human-readable format.
- `-R`: List directories recursively.

## 4. `mkdir`
**Description**: Make Directory. It creates a new directory.
**Usage**: Create a new directory.
**Examples**:
```sh
$ mkdir new_folder

$ mkdir -p /home/user/new_folder/sub_folder
```
**Important Flags**:
- `-p`: Create parent directories as needed.

## 5. `touch`
**Description**: Change file timestamps or create an empty file.
**Usage**: Create an empty file or update the timestamp of an existing file.
**Examples**:
```sh
$ touch newfile.txt  # Creates a new empty file
$ touch -c existingfile.txt  # Updates the timestamp if the file exists
```
**Important Flags**:
- `-c`: Do not create any files.

## 6. `cp`
**Description**: Copy files and directories.
**Usage**: Copy files or directories from one location to another.
**Examples**:
```sh
$ cp source.txt destination.txt  # Copy file to another file
$ cp -r source_dir/ destination_dir/  # Copy directory and its contents
$ cp -i source.txt destination.txt  # Prompt before overwrite
$ cp -u source.txt destination.txt  # Copy only when the source file is newer than the destination file or when the destination file is missing
```
**Important Flags**:
- `-r`: Copy directories recursively.
- `-i`: Prompt before overwrite.
- `-u`: Copy only when the source file is newer or the destination file is missing.

## 7. `mv`
**Description**: Move or rename files and directories.
**Usage**: Move files or directories to a different location or rename them.
**Examples**:
```sh
$ mv oldname.txt newname.txt  # Rename file
$ mv file.txt /home/user/Documents/  # Move file to another directory
$ mv -i file.txt /home/user/Documents/  # Prompt before overwrite
$ mv -u file.txt /home/user/Documents/  # Move only when the source file is newer or the destination file is missing
```
**Important Flags**:
- `-i`: Prompt before overwrite.
- `-u`: Move only when the source file is newer or the destination file is missing.

## 8. `rm`
**Description**: Remove files or directories.
**Usage**: Delete files or directories.
**Examples**:
```sh
$ rm file.txt  # Remove file
$ rm -r directory/  # Remove directory and its contents
$ rm -i file.txt  # Prompt before removal
$ rm -f file.txt  # Force removal without prompt
```
**Important Flags**:
- `-r`: Remove directories and their contents recursively.
- `-i`: Prompt before every removal.
- `-f`: Force removal without prompt.

## 9. `rmdir`
**Description**: Remove empty directories.
**Usage**: Delete empty directories.
**Examples**:
```sh
$ rmdir empty_directory/  # Remove empty directory
$ rmdir -p parent/child/grandchild/  # Remove directory and its parent directories if they are empty
```
**Important Flags**:
- `-p`: Remove parent directories if they are empty.

## 10. `cat`
**Description**: Concatenate and display file contents.
**Usage**: View the contents of a file.
**Examples**:
```sh
$ cat file.txt
Hello, World!

$ cat file1.txt file2.txt
Hello, World!
Welcome to Linux.

$ cat -n file.txt  # Number all output lines
1  Hello, World!
2  Welcome to Linux.
```
**Important Flags**:
- `-n`: Number all output lines.

## 11. `echo`
**Description**: Display a line of text.
**Usage**: Print text to the terminal or redirect it to a file.
**Examples**:
```sh
$ echo "Hello, World!"
Hello, World!

$ echo "Hello, World!" > file.txt  # Redirect output to a file
$ echo "Hello, World!" >> file.txt  # Append output to a file

$ echo -n "Hello, World!"  # Do not output the trailing newline
Hello, World!$
```
**Important Flags**:
- `-n`: Do not output the trailing newline.

## 12. `vi` and `nano`
**Description**: Text editors. `vi` is more powerful but has a steeper learning curve, while `nano` is simpler and more user-friendly.
**Usage**: Edit text files.
**Examples**:
```sh
$ vi file.txt  # Open file in vi editor
$ nano file.txt  # Open file in nano editor
```
**Basic Commands for `vi`**:
- `i`: Insert mode.
- `:w`: Save changes.
- `:q`: Quit.
- `:wq`: Save and quit.
- `:q!`: Quit without saving.

**Basic Commands for `nano`**:
- `Ctrl + O`: Save changes.
- `Ctrl + X`: Quit.
- `Ctrl + K`: Cut line.
- `Ctrl + U`: Paste line.

## 13. `grep`
**Description**: Print lines matching a pattern.
**Usage**: Search for text within files.
**Examples**:
```sh
$ grep "Hello" file.txt
Hello, World!

$ grep -r "Hello" /path/to/directory/
path/to/directory/file.txt:Hello, World!

$ grep -i "hello" file.txt  # Ignore case distinctions
Hello, World!

$ grep -n "Hello" file.txt  # Show line numbers
1:Hello, World!

$ grep -v "Hello" file.txt  # Invert match
Welcome to Linux.
```
**Important Flags**:
- `-r`: Recursively search directories.
- `-i`: Ignore case distinctions.
- `-n`: Show line numbers.
- `-v`:

 Invert match.

## 14. `ping`
**Description**: Send ICMP ECHO_REQUEST to network hosts.
**Usage**: Check the network connectivity to a host.
**Examples**:
```sh
$ ping www.google.com
PING www.google.com (172.217.15.100) 56(84) bytes of data.
64 bytes from ord30s21-in-f4.1e100.net (172.217.15.100): icmp_seq=1 ttl=57 time=10.3 ms

$ ping -c 4 www.google.com  # Send 4 packets
PING www.google.com (172.217.15.100) 56(84) bytes of data.
64 bytes from ord30s21-in-f4.1e100.net (172.217.15.100): icmp_seq=1 ttl=57 time=10.3 ms
64 bytes from ord30s21-in-f4.1e100.net (172.217.15.100): icmp_seq=2 ttl=57 time=10.2 ms
64 bytes from ord30s21-in-f4.1e100.net (172.217.15.100): icmp_seq=3 ttl=57 time=10.1 ms
64 bytes from ord30s21-in-f4.1e100.net (172.217.15.100): icmp_seq=4 ttl=57 time=10.0 ms
```
**Important Flags**:
- `-c`: Number of packets to send.

## 15. `curl`
**Description**: Transfer data from or to a server.
**Usage**: Fetch content from URLs.
**Examples**:
```sh
$ curl http://example.com  # Fetch content from URL
<!doctype html>
<html>
<head>
    <title>Example Domain</title>
    ...

$ curl -O http://example.com/file.zip  # Download file
```
**Important Flags**:
- `-O`: Write output to a file named as the remote file.

## 16. `wget`
**Description**: Non-interactive network downloader.
**Usage**: Download files from the web.
**Examples**:
```sh
$ wget http://example.com/file.zip  # Download file
--2024-07-08 12:34:56--  http://example.com/file.zip
Resolving example.com... 93.184.216.34
Connecting to example.com|93.184.216.34|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1024 (1.0K) [application/zip]
Saving to: 'file.zip'

file.zip           100%[===================>]   1.00K  --.-KB/s    in 0s

2024-07-08 12:34:56 (12.0 MB/s) - 'file.zip' saved [1024/1024]

$ wget -c http://example.com/file.zip  # Continue downloading a partially downloaded file
```
**Important Flags**:
- `-c`: Continue getting a partially downloaded file.

## 17. `watch`
**Description**: Execute a program periodically, showing output fullscreen.
**Usage**: Continuously monitor the output of a command.
**Examples**:
```sh
$ watch -n 5 df -h  # Update every 5 seconds
Every 5.0s: df -h                                        Mon Jul  8 12:35:00 2024

Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  1.5M  794M   1% /run
/dev/sda1        20G  8.4G  9.8G  47% /
```
**Important Flags**:
- `-n`: Interval in seconds.

## 18. `base64`
**Description**: Encode or decode file or standard input to/from Base64.
**Usage**: Encode data to Base64.
**Examples**:
```sh
$ echo "Hello" | base64
SGVsbG8K

$ echo "SGVsbG8K" | base64 --decode
Hello
```
**Important Flags**:
- `--decode`: Decode Base64 encoded data.

## 19. `ps`
**Description**: Report a snapshot of current processes.
**Usage**: View running processes.
**Examples**:
```sh
$ ps aux  # Detailed view of all processes
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  22560  3908 ?        Ss   12:00   0:01 /sbin/init
user      1000  0.0  0.2  63640  8920 ?        Ss   12:00   0:02 /lib/systemd/systemd --user

$ ps -ef  # Standard view of all processes
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:00 ?        00:01:00 /sbin/init
user      1000     1  0 12:00 ?        00:02:00 /lib/systemd/systemd --user

$ ps -e  # View all processes
  PID TTY          TIME CMD
    1 ?        00:01:00 init
 1000 ?        00:02:00 systemd

$ ps -f  # Full-format listing
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:00 ?        00:01:00 /sbin/init
user      1000     1  0 12:00 ?        00:02:00 /lib/systemd/systemd --user
```
**Important Flags**:
- `-a`: Show processes for all users.
- `-u`: Show processes with user and memory usage.
- `-x`: Show processes without a controlling terminal.
- `-e`: Show all processes.
- `-f`: Full-format listing.

## 20. `grep` (revisited for pipelines)
**Description**: Filter output of commands.
**Example**:
```sh
$ ps aux | grep ssh
root       999  0.0  0.1  65968  5984 ?        Ss   12:00   0:00 /usr/sbin/sshd -D
```

## 21. `>` and `>>`
**Description**: Redirect output.
**Usage**: Redirect command output to a file. `>` overwrites; `>>` appends.
**Examples**:
```sh
$ echo "Hello" > file.txt  # Overwrite file
$ echo "World" >> file.txt  # Append to file
```

## 22. `&>` and `2>`
**Description**: Redirect stdout and stderr.
**Examples**:
```sh
$ command &> output.txt  # Redirect both stdout and stderr
$ command 2> error.txt  # Redirect stderr
```

## 23. `netstat`
**Description**: Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.
**Usage**: Network troubleshooting and monitoring.
**Examples**:
```sh
$ netstat -tuln  # Show listening ports
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
udp        0      0 0.0.0.0:123             0.0.0.0:*

$ netstat -a  # Show all connections
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN
udp        0      0 0.0.0.0:123             0.0.0.0:*
```
**Important Flags**:
- `-t`: Show TCP connections.
- `-u`: Show UDP connections.
- `-l`: Show listening ports.
- `-n`: Show numerical addresses.

## 24. `tcpdump`
**Description**: Dump traffic on a network.
**Usage**: Network packet analysis.

`tcpdump` is a powerful command-line packet analyzer tool. It allows users to capture and display the packets being transmitted or received over a network to which the computer is attached.

### Important Flags

- `-c`: Specifies the number of packets to capture.
- `-w`: Write the raw packets to a file rather than parsing and printing them.

### Examples

#### Using `-c` Flag

The `-c` flag is used to specify the number of packets to capture. This can be useful when you want to capture a specific number of packets for analysis.

**Example**:
```sh
$ sudo tcpdump -c 5
```

**Output**:
```plaintext
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
12:34:56.789123 IP 192.168.1.10.12345 > 192.168.1.1.http: Flags [S], seq 123456789, win 65535, options [mss 1460,sackOK,TS val 123456789 ecr 0,nop,wscale 7], length 0
12:34:56.789456 IP 192.168.1.1.http > 192.168.1.10.12345: Flags [S.], seq 987654321, ack 123456790, win 65535, options [mss 1460,sackOK,TS val 987654321 ecr 123456789,nop,wscale 7], length 0
12:34:56.789789 IP 192.168.1.10.12345 > 192.168.1.1.http: Flags [.], ack 987654322, win 65535, options [TS val 123456790 ecr 987654321], length 0
12:34:56.790123 IP 192.168.1.10.12345 > 192.168.1.1.http: Flags [P.], seq 123456790:123456811, ack 987654322, win 65535, options [TS val 123456790 ecr 987654321], length 21: HTTP GET / HTTP/1.1
12:34:56.790456 IP 192.168.1.1.http > 192.168.1.10.12345: Flags [.], ack 123456811, win 65535, options [TS val 987654322 ecr 123456790], length 0
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

In this example, `tcpdump` captures 5 packets and then exits.

#### Using `-w` Flag

The `-w` flag is used to write the raw packet data to a file for later analysis. This can be useful when you want to capture packets for a longer period and analyze them later using tools like Wireshark.

**Example**:
```sh
$ sudo tcpdump -w capture.pcap
```

**Output**:
```plaintext
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

This command will not display the packet information on the screen. Instead, it will save the captured packets in a file named `capture.pcap`. You can stop the capture by pressing `Ctrl+C`.

#### Using Both `-c` and `-w` Flags

You can use both flags together to capture a specific number of packets and save them to a file.

**Example**:
```sh
$ sudo tcpdump -c 10 -w capture.pcap
```

**Output**:
```plaintext
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

This command captures 10 packets and saves them to `capture.pcap`.

### Analyzing the Capture File

To analyze the captured packets, you can use tools like `tcpdump` itself or Wireshark.

**Example**:
```sh
$ tcpdump -r capture.pcap
```

**Output**:
```plaintext
reading from file capture.pcap, link-type EN10MB (Ethernet)
12:34:56.789123 IP 192.168.1.10.12345 > 192.168.1.1.http: Flags [S], seq 123456789, win 65535, options [mss 1460,sackOK,TS val 123456789 ecr 0,nop,wscale 7], length 0
12:34:56.789456 IP 192.168.1.1.http > 192.168.1.10.12345: Flags [S.], seq 987654321, ack 123456790, win 65535, options [mss 1460,sackOK,TS val 987654321 ecr 123456789,nop,wscale 7], length 0
12:34:56.789789 IP 192.168.1.10.12345 > 192.168.1.1.http: Flags [.], ack 987654322, win 65535, options [TS val 123456790 ecr 987654321], length 0
...
```

In this example, `tcpdump` reads the packets from the `capture.pcap` file and displays them.

By using the `-c` and `-w` flags, `tcpdump` allows you to control the number of packets captured and save the raw packet data for future analysis.

## 25. `wc -l`
**Description**: Print newline counts.
**Usage**: Count lines in a file.
**Examples**:
```sh
$ wc -l file.txt


5 file.txt

$ cat file.txt | wc -l  # Count lines in the output of a command
5
```

## 26. `chmod`
**Description**: Change file modes or Access Control Lists.
**Usage**: Change file permissions.
**Examples**:
```sh
$ chmod 755 file.sh  # rwxr-xr-x
$ chmod u+x file.sh  # Add execute permission for the user

$ chmod -R 755 /path/to/directory  # Change permissions recursively
```
**Important Flags**:
- `-R`: Change permissions recursively.

## 27. `ssh`
**Description**: OpenSSH SSH client (remote login program).
**Usage**: Connect to a remote machine.
**Examples**:
```sh
$ ssh user@remote_host  # Connect to remote host
user@remote_host's password:

$ ssh -i /path/to/private_key user@remote_host  # Connect with private key
```
**Important Flags**:
- `-i`: Identity file (private key) for authentication.

## 28. `tar`
**Description**: Archive files.
**Usage**: Create or extract tar archives.
**Examples**:
```sh
$ tar -cvf archive.tar file1 file2  # Create tar archive
$ tar -xvf archive.tar  # Extract tar archive

$ tar -czvf archive.tar.gz file1 file2  # Create compressed tar archive
$ tar -xzvf archive.tar.gz  # Extract compressed tar archive
```
**Important Flags**:
- `-c`: Create a new archive.
- `-x`: Extract files from an archive.
- `-v`: Verbosely list files processed.
- `-f`: Use archive file.
- `-z`: Filter the archive through gzip.

## 29. `gzip`
**Description**: Compress or expand files.
**Usage**: Compress files.
**Examples**:
```sh
$ gzip file.txt  # Compress file
$ gunzip file.txt.gz  # Decompress file

$ gzip -r /path/to/directory  # Compress directory recursively
```
**Important Flags**:
- `-r`: Compress directories recursively.

## 30. `scp`
**Description**: Secure copy (remote file copy program).
**Usage**: Copy files between hosts.
**Examples**:
```sh
$ scp file.txt user@remote_host:/path/to/destination  # Copy file to remote host
user@remote_host's password:

$ scp -i /path/to/private_key file.txt user@remote_host:/path/to/destination  # Copy with private key
```
**Important Flags**:
- `-i`: Identity file (private key) for authentication.
- `-r`: Copy directories recursively.

## 31. `alias`
**Description**: Define or display aliases.
**Usage**: Create shortcuts for commands.
**Examples**:
```sh
$ alias ll='ls -la'  # Create alias
$ unalias ll  # Remove alias

$ alias grep='grep --color=auto'  # Add color to grep output
```

## 32. `clear`
**Description**: Clear the terminal screen.
**Usage**: Clean up the terminal window.
**Example**:
```sh
$ clear
```

## 33. `history`
**Description**: Show command history.
**Usage**: View previously used commands.
**Examples**:
```sh
$ history
1  ls
2  cd /home/user
3  pwd

$ !2  # Re-run command number 2
cd /home/user
```
**Important Flags**:
- `-c`: Clear the history list.

## 34. `head`
**Description**: Output the first part of files.
**Usage**: View the beginning of a file.
**Examples**:
```sh
$ head -n 10 file.txt  # View first 10 lines
Hello, World!
Welcome to Linux.
...
```
**Important Flags**:
- `-n`: Number of lines to show.

## 35. `tail`
**Description**: Output the last part of files.
**Usage**: View the end of a file.
**Examples**:
```sh
$ tail -n 10 file.txt  # View last 10 lines
Goodbye, World!
End of file.
...
```
**Important Flags**:
- `-n`: Number of lines to show.
- `-f`: Follow the log (like `tail -f`).


## 36. `crictl`
**Description**: Command-line interface for CRI-compatible container runtimes.
**Usage**: Manage containers in Kubernetes.
**Examples**:

sh
Copy code
$ crictl images  # List images
IMAGE               TAG                 IMAGE ID            SIZE
busybox             latest              6d5fcfe5ff17        1.2MB

$ crictl stats  # Show container stats
CONTAINER           CPU %               MEM USAGE / LIMIT   MEM %               NET I/O
d3b5a6a16b7f        0.00                512KiB / 1.0GiB     0.05                1.2kB / 600B


## 37. `journalctl`
**Description**: Query the systemd journal.
**Usage**: View logs from systemd services.
**Examples**:
```sh
$ journalctl -u ssh  # View logs for SSH service
-- Logs begin at Mon 2024-07-08 12:00:00 UTC, end at Mon 2024-07-08 12:34:56 UTC. --
Jul 08 12:00:01 hostname systemd[1]: Starting OpenSSH Daemon...
Jul 08 12:00:01 hostname sshd[999]: Server listening on 0.0.0.0 port 22.

$ journalctl -r  # Reverse output (newest entries first)
```
**Important Flags**:
- `-u`: Show logs for a specific unit.
- `-r`: Reverse output (newest entries first).
- `-f`: Follow the log (like `tail -f`).

This expanded guide with a table of contents provides detailed explanations, examples, and important flags for each command, ensuring a thorough understanding of their usage.
