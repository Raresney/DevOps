# Session 02 — Linux, Bash and Networking Intro

Summary of FiiPractic 2026 — Linux essential commands, users & permissions, Bash scripting, and networking basics.

## Infrastructure

| VM | Hostname | IP |
|---|---|---|
| app | app.fiipractic.lan | 192.168.100.10 |
| gitlab | gitlab.fiipractic.lan | 192.168.100.20 |

## 1. Command Reference

| Command | Example | Description |
|---|---|---|
| `mkdir` | `mkdir dir1` | Create a new directory |
| `cd` | `cd dir1` | Change directory |
| `ls` | `ls dir1` | List directory contents |
| `touch` | `touch file1` | Create an empty file |
| `cat` | `cat file` | Display file contents |
| `echo` | `echo "FiiPractic"` | Print a message |
| `vim` | `vim file1` | Edit/create a file with vim |
| `cp` | `cp file1 file2` | Copy a file |
| `mv` | `mv file1 ../file1` | Move or rename a file |
| `rm` | `rm file1` | Delete a file |
| `rmdir` | `rmdir dir1` | Delete an empty directory |
| `stat` | `stat file1` | Show file info |
| `grep` | `grep "text" file1` | Search for a string in a file |
| `ip a` | `ip a` | Show network interfaces |
| `ip route` | `ip route` | Show routing table |
| `tracepath` | `tracepath <host>` | Trace packet route |
| `pwd` | `pwd` | Print current directory |
| `ping` | `ping <host>` | Check host reachability |
| `ss -lt` | `ss -lt` | Show listening TCP sockets |
| `man` | `man <cmd>` | Show manual page |

## 2. Essential Commands Exercises

1. Make sure `FiiPractic-App` is running (`vagrant up app`)
2. Open an SSH session to `FiiPractic-App`
3. Create a folder `linux` and enter it
4. Create a file `message` using `touch`
5. Write `Fii Practic 2024` into `message` using `echo`
6. Verify with `cat`
7. Add a new line `Hello World !` using `vim`
8. Copy `message` to `message_copy`
9. Add `FiiPractic` to `message_copy` using `echo`
10. Search for "Hello" in `message` using `grep`
11. Check last modification time with `stat`
12. Check if `hosts` exists in `/etc` using `ls` and `grep`
13. Write all files/folders from `/etc` into a file called `results`
14. Verify if `passwd` exists in `/etc` using `grep` on the results file

## 3. Users and Permissions

```bash
# Create user
useradd fiipractic

# Verify
cat /etc/passwd | grep fiipractic

# Switch user
su - fiipractic

# Check current directory (should be /home/fiipractic)
pwd

# Create a file
touch fiipractic_file

# Check permissions
ls -l fiipractic_file

# Remove write permission for everyone
chmod 440 fiipractic_file

# Try to write — should fail
echo "test" > fiipractic_file
```

## 4. Bash Scripting

Task: modify `script.sh` to create files/directories. The script generates a list of N elements and then creates files or directories with those names in the current folder.

## 5. Networking Intro

1. Display network interface info on `FiiPractic-App`
2. List listening TCP sockets and save to a file `sockets`
3. Verify connectivity to `FiiPractic-GitLab`
4. Install netcat:
```bash
yum install nc
```
5. Write a Bash script that runs a simple HTTP server:
```bash
echo -e "HTTP/1.1 200 OK\r\n\r\n<body>FiiPractic <b>2024</b></body>\r\n\r\n" | nc -l 8080
```
6. Run the script and access `http://192.168.100.10:8080/`

## 6. Access Control Lists (ACLs)

### Reference

| Command | Description |
|---|---|
| `getfacl file1` | Show ACL permissions |
| `getfacl -R dir` | Show ACL recursively |
| `setfacl -m u:user:rw file1` | Set user permissions |
| `setfacl -m g:group:rw file1` | Set group permissions |
| `setfacl -b file1` | Remove all ACL permissions |

### Exercises

1. Create user `yonder`
2. Create group `aclgroup`, add `yonder` and `fiipractic` to it
3. Create directory `acl_test` with file `yonderfile` inside
4. View permissions with `getfacl` and compare with `ls -l`
5. Give `yonder` read+write access with `setfacl`
6. View permissions again with `getfacl`
7. Give `aclgroup` read access, keep `root` as owning group
8. Verify `fiipractic` can read the file
9. Remove `yonder`'s permissions
10. Verify `yonder` can still access via group

## Extra

### Fix Me Scripts
Debug intentionally broken scripts in the `fix_me_scripts` directory.

### Banking App
Build a Bash banking application with:
- Create a new client (initial balance: 0)
- Modify a client's balance
- Delete a client
- List all clients with balances

Data stored in `bank.csv`:
```
Client, Sold curent
```

### File Transfer with Netcat

**server.sh** — listen on a port, save received file:
```bash
nc -l <PORT> > "received_files/<FILE_NAME>"
```

**client.sh** — send a file to the server:
```bash
nc <SERVER_IP> <PORT> < "<FILE_TO_SEND>"
```

### Wireshark
1. Install [Wireshark](https://www.wireshark.org/download.html)
2. Select the host-only adapter
3. Filter: `ip.src==192.168.100.10/32 && tcp.port eq 8080`
4. Start the netcat server and access `http://192.168.100.10:8080/`
5. Observe the traffic

### Bandit
[OverTheWire Bandit](https://overthewire.org/wargames/bandit/bandit0.html)
