# Essential Linux Commands & Fundamentals

## Introduction
This guide is designed to take you from a complete beginner to an intermediate level in Linux. You'll learn how the system is structured, how to navigate the command line, manage users, control file permissions, process text, monitor system health, and automate tasks.

---

## 1. Linux Fundamentals

### What is Linux?
Linux is a family of open-source, Unix-like operating systems based on the Linux kernel, first released by Linus Torvalds in 1991. Because it is open source, it is packaged into different distributions (or "distros") that include the kernel, system utilities, and package managers.
- **Ubuntu/Debian**: User-friendly, popular for desktop and servers.
- **CentOS/RHEL/Fedora**: Enterprise-focused, stable.
- **Arch Linux**: Lightweight and customizable, aimed at advanced users.

### What is a Shell?
A **shell** is a command-line interpreter that acts as an interface between the user and the operating system kernel. When you type a command in the terminal, the shell interprets it and passes it to the OS.
Common types of shells:
- **Bash (Bourne Again Shell)**: The default and most common shell in Linux systems. It supports scripting, history, and aliases.
- **Zsh (Z Shell)**: Has advanced features like better autocomplete, spelling correction, and customization (e.g., Oh My Zsh).
- **Fish (Friendly Interactive Shell)**: Focused on user-friendliness with syntax highlighting and auto-suggestions out of the box.
- **Dash**: A lightweight, fast shell used mainly for executing system scripts during boot.

### Linux File System Architecture
Unlike Windows, which uses drive letters (`C:`, `D:`), Linux uses a unified directory tree starting from the **Root directory (`/`)**. Everything in Linux (files, directories, hardware devices, processes) is treated as a file.

Here is the hierarchy:
- `/` - The Root directory (parent of all files and folders).
- `/home` - Contains personal folders for users (e.g., `/home/alice`).
- `/root` - The home directory for the superuser (Administrator).
- `/etc` - Contains system configuration files (e.g., network settings, user databases).
- `/var` - Contains variable data like system logs (`/var/log`) and mail spools.
- `/usr` - User binaries, libraries, and documentation.
- `/bin` & `/sbin` - Essential command binaries (e.g., `ls`, `cd`, `systemctl`).
- `/tmp` - Temporary files deleted upon reboot.

### Understanding Absolute vs. Relative Paths
- **Absolute Path**: Starts from the root (`/`) and specifies the complete directory structure.
  *Example*: `/home/alice/documents/report.txt`
- **Relative Path**: Starts from your current working directory.
  *Example*: If you are in `/home/alice`, the relative path to the same file is `documents/report.txt`.
  - `.` represents the current directory.
  - `..` represents the parent directory.

### Boot Process, Run-levels, and Targets
When a Linux system boots up, it goes through a specific sequence:
1. **BIOS/UEFI**: Performs hardware checks (POST) and locates the boot loader.
2. **Bootloader (GRUB)**: Loads the Linux kernel into memory.
3. **Kernel**: Initializes hardware drivers and mounts the root filesystem `/`.
4. **Init/Systemd**: The first process (`PID 1`) starts system services and transitions the system to its operational state.

#### Run-Levels and systemd Targets
Traditionally, Linux systems used **Run-levels** to define the system state. Modern distributions use **systemd Targets** instead:

| Run-level | systemd Target | Description |
|---|---|---|
| **0** | `poweroff.target` | Shut down/Halt the system. |
| **1** | `rescue.target` | Single-user rescue mode (no networking, root shell). |
| **3** | `multi-user.target` | Multi-user text mode (no GUI, network enabled). |
| **5** | `graphical.target` | Multi-user graphical mode (standard desktop). |
| **6** | `reboot.target` | Reboot the system. |

#### Shutdown and Reboot Commands
Always shut down safely to prevent file system corruption:
- `shutdown`: Shuts down the system in 1 minute (default).
- `shutdown now` or `shutdown -h now` or `poweroff`: Safely shut down and power off the system immediately.
- `shutdown -r`: Reboots the system after 1 minute.
- `shutdown -r now` or `reboot`: Safely reboot the system immediately.
- `shutdown 13:45`: Scheduled shutdown at a specific time (24-hour HH:MM format).
- `shutdown -h +10 "System updating"`: Shut down in 10 minutes and broadcast the warning message to all logged-in users.

---

## 2. Basic Command Line & File Operations

To run commands, you type them into the terminal and press Enter.

### Print Working Directory (`pwd`)
Find out exactly where you are in the filesystem hierarchy.
- **Usage**: `pwd`
- **Example Output**: `/home/alice`

### List Directory Contents (`ls`)
Lists the files and folders in a directory.
- **Common Options**:
  - `ls` - Simple listing.
  - `ls -l` - Long format (shows permissions, owner, size, date).
  - `ls -a` - Show all files, including hidden files (starting with `.`).
  - `ls -lh` - Long format with human-readable file sizes (e.g., KB, MB).
- **Example**: `ls -la`

### Change Directory (`cd`)
Navigate between folders.
- **Usage**:
  - `cd /var/log` (Change to absolute path)
  - `cd ..` (Move up one level)
  - `cd ~` or just `cd` (Go to your home directory)
  - `cd -` (Switch back to the previous directory you were in)

### Creating Files & Directories (`mkdir`, `touch`)
- **`mkdir`**: Create directories.
  - *Example*: `mkdir project`
  - *Example (Nested directories)*: `mkdir -p project/src/tests` (creates parent directories if they don't exist).
- **`touch`**: Creates an empty file or updates the timestamp of an existing file.
  - *Example*: `touch readme.md`

### Copying Files & Directories (`cp`)
- **Copy file**: `cp source.txt backup.txt`
- **Copy directory recursively** (must use `-r`): `cp -r project/ project_backup/`

### Moving and Renaming (`mv`)
The `mv` command is used for both moving files to another location and renaming them.
- **Rename a file**: `mv oldname.txt newname.txt`
- **Move a file**: `mv newname.txt /tmp/`

### Deleting Files & Directories (`rm`, `rmdir`)
> [!WARNING]
> Linux does not have a "Recycle Bin". Once a file is deleted with `rm`, it is gone.
- **Delete a file**: `rm file.txt`
- **Delete an empty directory**: `rmdir empty_folder`
- **Force delete a directory and all of its contents recursively**: `rm -rf folder/` (Use with extreme caution!).

### Inspecting File Content (`cat`, `less`, `head`, `tail`)
- **`cat`**: Displays the entire file contents in the terminal. Good for small files.
  - *Example*: `cat /etc/hostname`
- **`less`**: Opens files interactively, letting you scroll up/down using arrow keys. Press `q` to quit.
  - *Example*: `less /var/log/syslog`
- **`head`**: Shows the top/beginning lines of a file.
  - *Example*: `head -n 10 file.txt` (Shows first 10 lines).
- **`tail`**: Shows the bottom/ending lines of a file.
  - *Example*: `tail -n 10 file.txt` (Shows last 10 lines).
  - *Real-time monitoring*: `tail -f /var/log/syslog` (Follows/displays logs in real-time as they are appended).

### Command Line History & Reuse
The shell stores a history of previously executed commands. You can list, search, and reuse them to save time.

#### Listing and Saving History
- `history`: Lists all previously executed commands with their history numbers.
- `history > term1_history.txt`: Saves the current session's command history to a text file.

#### Event Designators (Quick Execution)
- `!!`: Repeats the very last command executed.
- `!n`: Executes command number `n` from the history list (e.g., `!37`).
- `!string`: Executes the most recent command starting with the specified string (e.g., `!cat`).
- `!?keyword?`: Executes the most recent command containing `keyword`.

#### Searching & Modifying Previous Commands
- **Reverse Search**: Press `Ctrl+r` and start typing a keyword. The shell will search backwards through your history. Press `Ctrl+r` repeatedly to cycle through older matches. Press `Enter` to run the found command.
- **Substitution**: `^old^new^` replaces the text `old` with `new` in the previous command and executes it immediately.
  - *Example*: If you ran `md5sum archive.tar.gz` and want to run `sha1sum`, type: `^md5^sha1^` (replaces `md5` with `sha1` and executes the command).

#### Configuration Variables
- `echo $HISTSIZE`: Shows the maximum number of commands stored in memory for the current session.
- `export HISTTIMEFORMAT="%F %T "`: Appends date and time stamps to the output of the `history` command.

---

## 3. User & Group Administration

Linux is a multi-user system. Users belong to **groups** to share permissions.

### Understanding Groups
- **Primary Group**: The default group assigned when the user is created. Usually named after the user.
- **Secondary Group**: Additional groups the user is added to, allowing access to resources like Docker, database administration, or sudo access.

#### Managing Groups
- **Create a group**: `sudo groupadd developers`
- **View all system groups**: `cat /etc/group` (contains group names, password placeholders, GIDs, and user list).

### Managing Users
- **Create a user**:
  - `sudo useradd -m -s /bin/bash aaron`: Creates a new user `aaron` with a home directory (`-m`) and sets their login shell to `/bin/bash` (`-s`).
- **Create a user with a primary group**: `sudo useradd -g developers alice`
- **Add a user to a secondary group**: `sudo usermod -aG testers alice` (Note: `-a` means append, `-G` specifies groups. Without `-a`, you overwrite all other secondary groups!).
  - *Example (multiple groups)*: `sudo usermod -aG daemon,adm aaron` (adds `aaron` to both `daemon` and `adm` groups).
- **Delete a user**: `sudo userdel -r alice` (The `-r` flag deletes the user's home directory and mail spool).

### Identity & Authentication
- **`id`**: Displays the User ID (UID), Group ID (GID), and all group memberships for a user.
  - *Usage*: `id` (current user) or `id username`
- **`groups`**: Provides a quick list of groups a specific user belongs to.
  - *Usage*: `groups` or `groups username`
- **`passwd`**: Sets or changes user passwords.
  - *Usage*: `passwd` (change your own password) or `sudo passwd username` (set/change another user's password).
- **`whoami`**: Displays the active username of the current shell session. Useful after switching user contexts.

### Switching Users (`su`)
- `su [username]`: Switches to the target user account in a non-login shell. It preserves the environment of the original user.
- `su - [username]`: Switches to the target user account in a full login shell. This resets the environment variables, updates the `PATH`, and switches to the target user's home directory, inheriting the target user's custom shell configs.

### User & Group Databases
- `/etc/passwd`: Stores basic account details (Username, Encrypted password placeholder `x`, UID, GID, User Info, Home Directory, Default Shell).
- `/etc/group`: Defines system groups, GIDs, and lists members of each group.
- `/etc/shadow`: Stores encrypted password hashes and password aging/expiry policies. Accessible only by root.

---

## 4. File Permissions & Ownership

### The Permission Layout
When you run `ls -l` (long listing), you see an output line like:
```bash
-rwxr-xr-- 1 alice developers 1024 Jun 10 12:00 script.sh
```
This line breaks down into the following columns/fields:
1. `-rwxr-xr--`: File type and permission bits.
   - First character: File type: `-` (regular file), `d` (directory), or `l` (symbolic link).
   - Next 3 (`rwx`): **Owner (User)** permissions.
   - Next 3 (`r-x`): **Group** permissions.
   - Last 3 (`r--`): **Others** permissions (everyone else).
2. `1`: Hard link count (number of directory entries pointing to this file's inode).
3. `alice`: The file Owner (User).
4. `developers`: The Group ownership assigned to the file.
5. `1024`: File size in bytes.
6. `Jun 10 12:00`: Timestamp of last modification.
7. `script.sh`: Name of the file or directory.

| Symbol | Permission | Meaning on Files | Meaning on Directories |
|---|---|---|---|
| `r` | Read | View file contents. | List files inside directory (`ls`). |
| `w` | Write | Modify file contents. | Add, delete, or rename files in the directory. |
| `x` | Execute | Run file as program/script. | Access/enter the directory (`cd` into it). |

### Modifying Permissions (`chmod`)

#### 1. Symbolic Mode
Using symbols to add (`+`), remove (`-`), or set (`=`) permissions.
- `chmod u+x script.sh` (Give owner execute permission)
- `chmod g-w script.sh` (Remove write permission from the group)
- `chmod o+r script.sh` (Give read permission to others)
- `chmod a+rx script.sh` (Give all users read/execute permissions)

#### 2. Numeric Mode (Octal)
Permissions are represented by binary bits summed up:
- `r` = 4
- `w` = 2
- `x` = 1
- `-` = 0

To calculate a permission set, add up the numbers for each group:
- `rwx` = 4 + 2 + 1 = 7 (Full permissions)
- `rw-` = 4 + 2 + 0 = 6 (Read and write)
- `r-x` = 4 + 0 + 1 = 5 (Read and execute)
- `r--` = 4 + 0 + 0 = 4 (Read only)

**Examples**:
- `chmod 755 script.sh` -> Owner: `rwx` (7), Group: `r-x` (5), Others: `r-x` (5).
- `chmod 644 document.txt` -> Owner: `rw-` (6), Group: `r--` (4), Others: `r--` (4).
- `chmod 777 public_file.txt` -> Dangerous! Full access for everyone.

### Changing Ownership (`chown`)
Only the superuser (root) can change who owns a file.
- **Change owner**: `sudo chown alice script.sh`
- **Change owner and group**: `sudo chown alice:developers script.sh`
- **Change only group**: `sudo chown :developers script.sh`
- **Change recursively (directories)**: `sudo chown -R alice:developers /var/www/`

### Advanced Permissions (Special Bits)
Beyond standard read, write, and execute permissions, Linux offers special permissions and conditional behavior for advanced access control.

#### Directory Execution Rules
For a directory, the execute bit (`x`) acts as a search bit allowing access.
- You must have `x` permissions to enter a directory using `cd` or to query attributes of files within it.
- If you have read (`r`) but lack execute (`x`) on a directory, you can list the names of the files in the directory, but you cannot `cd` into it, read file contents, or view metadata (like file size or permissions) for files inside.

#### Conditional Execute Bit (`X`)
The capital `X` is a conditional execute permission that is extremely useful for recursive changes (`chmod -R`).
- It applies the execute permission **only** to directories, or to files that already have execute permission for at least one user class.
- *Example*: `chmod -R a+rX project/` ensures that all directories are accessible and searchable, but leaves standard non-executable files (like `.txt` or `.md`) untouched.

#### SetUID & SetGID (`s`)
Set User ID (SetUID) and Set Group ID (SetGID) allow files/programs to execute with the permissions of the file's owner or group, rather than the user running them.
- **SetUID (`chmod u+s file`)**: When a program with SetUID is run, it executes with the privileges of the file's owner (often `root`).
  - *Example*: The `/usr/bin/passwd` command has the SetUID bit set because it needs to write changes to `/etc/shadow` (which only `root` can write to), even when run by standard users.
- **SetGID (`chmod g+s directory`)**: When set on a directory, any new files created inside will inherit the group ownership of the parent directory, rather than the primary group of the creating user.
- **Capital `S` Warning**: If you see a capital `S` in the permission listing (e.g., `-rwSr-xr-x`), it indicates a configuration error where the SetUID/SetGID bit is enabled but the underlying execution bit (`x`) is **not** set. The special functionality will not work until you add the execution bit.

#### The Sticky Bit (`t`)
The Sticky Bit creates a "restricted deletion" environment on directories.
- When set on a directory (e.g., `/tmp`), users can only rename or delete files that they themselves own.
- Even if a directory has full write permissions (`777`), the sticky bit prevents users from deleting files created by other users in that shared space.
- **Representation**: Indicated by a `t` (or capital `T` if the underlying execute bit is not set) at the end of the permission block (e.g., `drwxrwxrwt`).
- **Command**: `chmod +t directory/` or octal `1777` (where `1` sets the sticky bit).

---

## 5. Text Processing & Searching

Searching and editing text is critical on headless systems.

### Finding Files (`find`, `tree`)
- **`find`**: Searches for files in a directory tree.
  - *Example*: `find . -name "*.log"` (Search for files ending in `.log` in the current folder and subfolders).
  - *Example*: `find /home -size +100M` (Find files in `/home` larger than 100MB).
- **`tree`**: Visualizes files in a graphical folder structure.
  - *Example*: `tree -L 2` (Limit to 2 levels deep).

### Searching & Filtering Text (`grep`)
`grep` searches files for matching lines containing a pattern.
- **Basic search**: `grep "error" syslog.log`
- **Case-insensitive search**: `grep -i "error" syslog.log`
- **Recursive search in directories**: `grep -r "TODO" ./src`
- **Count occurrences**: `grep -c "warning" syslog.log`

### Stream Editor (`sed`)
Used to find and replace text within streams or files.
- **Format**: `sed 's/old_text/new_text/g' file.txt`
- **In-place file edit** (modifies file directly): `sed -i 's/localhost/127.0.0.1/g' config.conf`

### Pattern Scanning and Processing (`awk`)
Extracts columns and runs operations on structured text (space/tab delimited).
- **Print first column**: `awk '{print $1}' data.txt`
- **Print first and third column**: `awk '{print $1, $3}' data.txt`
- **Extract IP addresses from logs**: `last | awk '{print $3}'`

### Line & Word Counting (`wc`)
- **Count lines in file**: `wc -l file.txt`
- **Count characters**: `wc -c file.txt`

---

## 6. Package Management

Linux distributions download software from secure repositories using package managers.

### Debian/Ubuntu (`apt` & `apt-get`)
Uses `.deb` packages.
- **Update repository definitions** (do this before installing): `sudo apt update`
- **Upgrade all installed packages**: `sudo apt upgrade`
- **Install a package**: `sudo apt install curl`
- **Uninstall a package**: `sudo apt remove curl`
- **Search for a package**: `apt search nginx`

### RedHat/CentOS/Fedora (`yum` & `dnf`)
Uses `.rpm` packages.
- **Install package**: `sudo yum install curl`
- **Update packages**: `sudo yum update`
- **Remove package**: `sudo yum remove curl`
- **Search package**: `yum search nginx`

---

## 7. Process Management & System Monitoring

### Viewing Processes (`ps`, `top`, `htop`)
- **`ps`**: Displays static snapshots of running processes.
  - `ps aux`: Displays processes for all users (`a`), in a user-friendly format with CPU/memory stats (`u`), including processes not attached to a terminal (`x`).
  - `ps -ef`: Displays a full list (`-f`) of every process running on the system (`-e`) for all users, showing parent PID relationships (PPID).
- **`top`**: Interactive real-time process monitor showing CPU and RAM usage.
  - *Interactive Controls*:
    - `M`: Sort processes by memory usage.
    - `P`: Sort processes by CPU usage.
    - `q`: Quit the monitor.
- **`htop`**: A modernized, colorized, user-friendly CLI task manager. You can scroll, search, and kill processes directly.

### Terminating Processes (`kill`)
Every process has a Process ID (PID).
- **Graceful termination**: `kill <PID>` (Sends SIGTERM, letting the app clean up).
- **Force termination**: `kill -9 <PID>` (Sends SIGKILL, immediately terminating the app).
  - *Example*: `kill -9 1234`
- **Kill by name**: `killall firefox`

### Job Control
Job control allows you to control multiple tasks within a single shell session.

- **Background Execution (`&`)**: Runs a job in the background immediately, leaving the shell free for other commands.
  - *Example*: `dd if=/dev/urandom of=/dev/null &`
- **Suspending a Job (`Ctrl+z`)**: Stops (suspends) the current foreground job and returns control back to the shell.
- **`jobs`**: Lists all active jobs and their status in the current shell session.
  - *Example*: `jobs -l` (includes PIDs for each job).
- **`bg`**: Resumes a suspended job and runs it in the background.
  - *Example*: `bg %2` (resumes job number 2 in the background).
- **`fg`**: Brings a background or suspended job to the foreground.
  - *Example*: `fg %1` (brings job number 1 to the foreground).

### Disk and Memory Usage (`df`, `du`, `free`)
- **Disk Space**: `df -h` (Show mounted filesystems, total, used, and free space in human-readable GB/MB format).
- **Folder Size**: `du -sh *` (Summarizes the disk space occupied by each file/folder in the current directory).
- **Memory/RAM**: `free -m` (Displays total, used, free, and swap memory in Megabytes).

### Service Management (`systemctl`, `journalctl`)
On modern Linux systems running **systemd**, services are managed using `systemctl`.
- **Start a service**: `sudo systemctl start nginx`
- **Stop a service**: `sudo systemctl stop nginx`
- **Restart a service**: `sudo systemctl restart nginx`
- **Check status**: `systemctl status nginx`
- **Enable startup on boot**: `sudo systemctl enable nginx`
- **Disable startup on boot**: `sudo systemctl disable nginx`

#### Viewing Logs (`journalctl`)
`journalctl` queries the systemd journal logs.
- **Follow logs in real-time**: `journalctl -f`
- **View logs for a specific service**: `journalctl -u nginx -f`
- **View boot logs**: `journalctl -b`

---

## 8. Networking Basics

### Network Interface and IP Address (`ip`)
- **Show interfaces and IP addresses**: `ip addr show` (or simply `ip a`).
- **Show routing tables**: `ip route`

### Listing Ports and Sockets (`ss`)
`ss` replaces the older `netstat` tool.
- **Show listening ports with process names**: `sudo ss -tulnp`
  - `-t`: TCP ports.
  - `-u`: UDP ports.
  - `-l`: Listening sockets.
  - `-n`: Numeric ports (instead of service names).
  - `-p`: Process using the port.

### Network Troubleshooting (`ping`, `traceroute`, `dig`)
- **Test network connection**: `ping -c 4 google.com` (sends 4 packets).
- **Trace routing path**: `traceroute google.com` (shows intermediate hops).
- **DNS Lookup**: `dig google.com` (returns DNS records for the domain).

### Fetching Data (`curl`)
A command-line tool to transfer data from or to a server.
- **Fetch content**: `curl https://example.com`
- **Show headers only**: `curl -I https://example.com`
- **Download file**: `curl -o filename.html https://example.com`

---

## 9. Automation of Jobs (Cron)

`cron` allows you to schedule scripts or commands to run automatically in the background at set times.

### The Crontab Format
A cron job is defined by 5 time fields followed by the command to execute:

```text
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of the week (0 - 6) (0 is Sunday)
│ │ │ │ │
* * * * *  /path/to/command
```

### Special Symbols
- `*` : Every interval (e.g., `*` in the hour column means "every hour").
- `,` : Value list (e.g., `1,15` in the minute column means "at minute 1 and minute 15").
- `-` : Range (e.g., `1-5` in the day of the week column means "Monday through Friday").
- `*/n` : Step values (e.g., `*/15` in the minute column means "every 15 minutes").

### Managing Cron Jobs
Every user has their own crontab file.
- **Edit your crontab**: `crontab -e` (Opens the crontab file in the default text editor).
- **List your crontab jobs**: `crontab -l`
- **Remove all cron jobs**: `crontab -r`

### Common Examples
- **Run a script daily at 2:30 AM**:
  ```bash
  30 2 * * * /home/alice/backup.sh
  ```
- **Run a cleanup script every Sunday at midnight**:
  ```bash
  0 0 * * 0 /home/alice/cleanup.sh
  ```
- **Run an API ping task every 10 minutes**:
  ```bash
  */10 * * * * curl -s https://example.com/api/ping > /dev/null
  ```

---

## 10. Links & Inodes (Hard vs. Soft Links)

In Linux, files are referenced by directory entries pointing to an **inode** (index node). An inode is a data structure on the filesystem that stores metadata about a file (such as owner, group, permissions, size, and physical disk block locations) but does not store the file's actual name or file contents.

### Hard Links
A **hard link** is an additional directory entry (another filename) pointing to the same underlying inode as the original file.
- **Key Characteristics**:
  - They share the exact same inode number, file size, permissions, ownership, and modification timestamps.
  - Changes made to the content of one link are instantly reflected in all other hard links.
  - Editing metadata (like changing permissions) on one hard link changes it for all of them.
- **Limitations**:
  - Cannot span across different filesystems (since inode numbers are filesystem-specific).
  - Cannot be created for directories (to prevent directory loops/cycles).
- **Deletion Behavior**:
  - The actual data on the disk remains accessible as long as at least one hard link points to the inode.
  - The file size and space are only freed when the last hard link is deleted and the link count drops to `0`.

### Soft Links (Symbolic Links)
A **soft link** (or symlink) is a separate, special file that functions as a shortcut, pointing to the pathname of the target file or directory rather than its inode.
- **Key Characteristics**:
  - Has its own unique inode number and permissions.
  - The file size of a symlink is simply the length of the target path string it points to (not the size of the target content).
- **Advantages**:
  - Can cross filesystem boundaries.
  - Can point to directories.
- **Deletion Behavior**:
  - If the original file is deleted or renamed, the symlink remains but becomes a **broken link** (pointing to a non-existent path).
  - Deleting the symlink does not affect the original file.

### Summary Comparison

| Feature | Hard Link | Soft Link (Symbolic) |
|---|---|---|
| **Points to** | Target's Inode directly | Target's Pathname |
| **Inode Number** | Same as the original file | Different from the original file |
| **Cross Filesystem** | No | Yes |
| **Supports Directories** | No | Yes |
| **If Target is Deleted** | Data remains accessible via link | Link becomes broken (dangling) |
| **Command to Create** | `ln target linkname` | `ln -s target linkname` |