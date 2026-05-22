# Essential Linux Commands

## 1. Basic Navigation & File Operations

```bash
pwd                      # Print working directory
ls -la                   # List directory contents (detailed, including hidden files)
cd /path/to/dir          # Change directory to target path
cd ..                    # Go up one directory level
cd ~                     # Go to user's home directory
mkdir mydir              # Create a new directory
touch file.txt           # Create an empty file or update file timestamp
cp file.txt copy.txt     # Copy a file
cp -r folder/ backup/    # Copy a directory recursively
mv old.txt new.txt       # Rename or move a file/directory
rm file.txt              # Delete a file
rm -rf folder/           # Force delete a directory and its contents recursively
cat file.txt             # Display the entire content of a file
less file.txt            # Interactive viewer (scroll through a file page-by-page)
head -n 10 file.txt      # Display the first 10 lines of a file
tail -n 10 file.txt      # Display the last 10 lines of a file
```

---

## 2. File Permissions & Ownership

```bash
chmod 755 script.sh      # Set read/write/execute for owner, read/execute for others
chmod +x script.sh       # Make a script executable
chown user:group file    # Change owner and group ownership of a file
```

---

## 3. System Information

```bash
whoami                   # Show the current logged-in user
uname -a                 # Print complete system details (kernel version, architecture)
uptime                   # Show how long the system has been running
df -h                  # Display available disk space in human-readable format
du -sh *               # Summarize disk usage of files/folders in the current directory
history                  # Show previously executed command history
```

---

## 4. Advanced File Search & Disk Usage

```bash
find / -name "*.log"     # Search for files ending in .log starting from root
tree -L 2                # Display directory structure up to 2 levels deep
```

---

## 5. Text Processing

```bash
grep -r "pattern" .      # Search recursively for a pattern in files
awk '{print $1, $3}' f   # Print the 1st and 3rd column of a file/stream
sed 's/old/new/g' file   # Find & replace occurrences of 'old' with 'new' in a file
tail -f /var/log/syslog  # Monitor log output in real-time as it appends
wc -l file               # Count total lines in a file
```

---

## 6. Process Management

```bash
ps aux                   # List all running processes on the system
kill -9 <PID>            # Force terminate a process by its Process ID (PID)
systemctl status service # Check the running status of a systemd service
journalctl -u service -f # Follow logging output for a specific systemd service
```

---

## 7. Networking

```bash
ip addr show             # List all network interfaces and assigned IP addresses
ss -tulnp                # Show listening sockets (ports) and associated processes
curl -I https://url.com  # Fetch and display HTTP response headers
dig example.com          # Perform a DNS lookup
```
