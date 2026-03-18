# Basic Linux Commands
https://www.terminaltemple.com/

## 1. Navigation Commands

### pwd (Print Working Directory)
Shows the current directory you're in.
```bash
pwd
# Output: /home/user/documents
```

### ls (List)
Lists files and directories.
```bash
ls              # Basic listing
ls -l           # Long format (detailed)
ls -a           # Show hidden files (starting with .)
ls -lh          # Human-readable file sizes
ls -lt          # Sort by modification time
ls -ltr         # Sort by time, reverse order (oldest first)
root@radius-server-play:~# ls shruthi/shruthi1
file1.txt  radius_server.py
```

### cd (Change Directory)
Navigate between directories.
```bash
cd /path/to/directory    # Go to specific path
cd ~                     # Go to home directory
cd ..                    # Go up one level
cd -                     # Go to previous directory
cd                       # Go to home directory
```

## 2. File Operations

### cat (Concatenate)
Display file contents.
```bash
cat file.txt                    # Display file
cat file1.txt file2.txt         # Display multiple files


### touch
Create empty file or update timestamp.
```bash
touch newfile.txt               # Create empty file
touch file1 file2 file3         # Create multiple files
```

### cp (Copy)
Copy files or directories.
```bash
cp source.txt destination.txt       # Copy file
cp file.txt /path/to/directory/     # Copy to directory
cp -r sourcedir/ destdir/           # Copy directory recursively
```

### mv (Move/Rename)
Move or rename files/directories.
```bash
mv oldname.txt newname.txt          # Rename file
mv file.txt /path/to/directory/     # Move file
mv *.txt /backup/                   # Move all .txt files
```

### rm (Remove)
Delete files or directories.
```bash
rm file.txt                 # Delete file
rm -r directory/            # Delete directory recursively
rm -f file.txt              # Force delete (no confirmation)
rm -rf directory/           # Force delete directory (DANGEROUS!)
rm *.log                    # Delete all .log files
```

### mkdir (Make Directory)
Create directories.
```bash
mkdir newdir                    # Create directory
mkdir -p dir1/dir2/dir3         # Create nested directories
mkdir dir1 dir2 dir3            # Create multiple directories
```

### rmdir
Remove empty directories.
```bash
rmdir emptydir              # Remove empty directory
rmdir -p dir1/dir2/dir3     # Remove nested empty directories
```

## 3. Viewing File Contents



### tail
View last lines of a file.
```bash
tail file.txt           # Last 10 lines
tail -n 20 file.txt     # Last 20 lines
tail -f logfile.log     # Follow file (real-time updates)
```

## 4. Search Commands

### find ( find <directory> -name <file name or "*extension">)
Search for files and directories.
```bash
find /path -name "*.txt"            # Find all .txt files
find . -name "file.txt"             # Find in current directory
find . -name ".*txt" --> 

```

history --> list of all the commands that are executed.

  601  find . -name switch.py
  602  find . -name switch123.py
  604  find . -name "*.py"
  606  find . -name "*.txt"
  607  find /root/ -name "*.py"
  611  find --help
  612  history

### grep (grep <functionality> <pattern> <file/directort>)
Search text within files.
```bash
grep "pattern" file.txt             # Search for pattern
grep -i "pattern" file.txt          # Case-insensitive search
grep -r "pattern" /path/      (mostly usecd)      # Recursive search in directory
```
root@radius-server-play:~# grep HandleAuthPacket radius_server.py 
    def HandleAuthPacket(self, pkt):
root@radius-server-play:~# grep state radius_server.py 
        state = pkt.get("State", [None])[0]
        if state and username in self.pending_challenges:
            state = os.urandom(16).hex()
            reply["State"] = [state.encode()]
            self.pending_challenges[username] = stateroot@radius-server-play:~# grep State radius_server.py 
        state = pkt.get("State", [None])[0]
            reply["State"] = [state.encode()]
root@radius-server-play:~# grep -i state radius_server.py 
        state = pkt.get("State", [None])[0]
        if state and username in self.pending_challenges:
            state = os.urandom(16).hex()
            reply["State"] = [state.encode()]
            self.pending_challenges[username] = state

grep -r radius /root/.  ( list all the files which has word "radius" in folder root)


### which
Find location of a command.
```bash
which python            # Shows path: /usr/bin/python
which java
```
 which python 
/opt/homebrew/bin/python

 whoami
s.kandakatla


whoami
root


## 5. File Permissions

### chmod (Change Mode)
Change file permissions.
```bash
chmod 755 script.sh             # rwxr-xr-x
chmod 644 file.txt              # rw-r--r--
chmod +x script.sh              # Add execute permission
chmod -w file.txt               # Remove write permission
chmod u+x script.sh             # Add execute for user
chmod g-w file.txt              # Remove write for group
chmod o+r file.txt              # Add read for others
```

**Permission Numbers:**
- 4 = read (r)
- 2 = write (w)
- 1 = execute (x)
- 755 = rwxr-xr-x (owner: all, group: read+execute, others: read+execute)
- 644 = rw-r--r-- (owner: read+write, group: read, others: read)




## 6. System Information

### uname
System information.
```bash
uname -a            # All system info
uname -r            # Kernel version
uname -m            # Machine hardware
```

### whoami
Display current user.
```bash
whoami              # Shows: username
```

### hostname
Display or set system hostname.
```bash
hostname            # Show hostname
hostname -I         # Show IP addresses
```

root@radius-server-play:~# hostname
radius-server-play
root@radius-server-play:~# hostname -I
10.192.238.126 10.192.237.162 10.10.0.5 2620:128:e00c:4148:f816:3eff:fe7f:7456 fdb0:341a:455b:2144:f816:3eff:fe0a:29a0 
root@radius-server-play:~# 


## 7. Disk Usage

### df (Disk Free)
Show disk space usage.
```bash
df -h               # Human-readable format
root@radius-server-play:~# df
Filesystem     1K-blocks    Used Available Use% Mounted on
tmpfs            1637576    1316   1636260   1% /run
/dev/sda3      162881404 7388848 148671300   5% /
tmpfs            8187876       0   8187876   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
tmpfs            1637572      16   1637556   1% /run/user/0
root@radius-server-play:~# 
root@radius-server-play:~# 
root@radius-server-play:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           1.6G  1.3M  1.6G   1% /run
/dev/sda3       156G  7.1G  142G   5% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.6G   16K  1.6G   1% /run/user/0
```

### du (Disk Usage)
Show directory/file space usage.
```bash 
du -h               # Human-readable
du -sh *            # Summary of each item in current directory
```

root@radius-server-play:~/shruthi# df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           1.6G  1.3M  1.6G   1% /run
/dev/sda3       156G  7.1G  142G   5% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.6G   16K  1.6G   1% /run/user/0

root@radius-server-play:~/shruthi# du -h
12K	./shruthi1
20K	.
root@radi

root@radius-server-play:~/shruthi/shruthi1# du -sh *
4.0K	file1.txt
4.0K	radius_server.py

## 8. Process Management

### ps (Process Status)
Show running processes.
```bash
ps                  # Processes for current shell
ps -ef              # All processes (full format)
ps aux | grep java  # Find Java processes
```

### top
Real-time process monitoring.
```bash
top                 # Interactive view
# Keys: q (quit), k (kill), M (sort by memory), P (sort by CPU)
```

### kill
Terminate processes.
```bash
kill PID                # Graceful termination (SIGTERM)
kill -9 PID             # Force kill (SIGKILL)
kill -15 PID            # SIGTERM (same as default)
killall processname     # Kill by name
```



## 9. Network Commands

### ping
Test network connectivity.
```bash
ping google.com         # Continuous ping (Ctrl+C to stop)
ping -c 4 google.com    # Ping 4 times
```

### ifconfig / ip
Network interface configuration.
```bash
ifconfig                # Show network interfaces (older)
ip addr show            # Show IP addresses (modern)
ip link show            # Show network interfaces
```

### netstat
Network statistics.
```bash
netstat -tuln           # Show listening ports
netstat -an             # All connections
netstat -r              # Routing table
```

### wget
Download files from web.
```bash
wget http://example.com/file.txt        # Download file
wget -O newname.txt http://url          # Save with different name
```

### curl
Transfer data from/to server.
```bash
curl http://example.com                 # Get webpage
curl -O http://example.com/file.txt     # Download file
curl -I http://example.com              # Show headers only
```






## 12. Compression & Archiving

### tar (Tape Archive)
Create and extract archives.
```
tar -xzvf archive.tar.gz            # Extract compressed archive
```



## 13. Help & Manual

### man (Manual)
Display command manual.
```bash
man ls              # Show manual for ls command
```