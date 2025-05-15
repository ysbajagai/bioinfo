---
layout: default
title: 🖥️ Unix Bash for High Performance Computing (HPC)
---
# Unix Bash for High Performance Computing (HPC)

## 🖥️ What is Unix/Linux?

Unix is a powerful multiuser operating system. Linux is a Unix-like system commonly used in HPC. You interact with it using a command-line interface (CLI) or shell (bash).
Welcome to the beginner’s guide to Unix and the command line interface for HPC users.

---

## 🧭 Navigating the Unix Filesystem

### Show your current directory
```bash
pwd
```

### List files in the current directory
```bash
ls -lh
```

### Change directory
```bash
cd /path/to/folder
```

### Go up one directory
```bash
cd ..
```

---

## 📁 Working with Files and Directories

### Create a directory
```bash
mkdir myfolder
```

### Create a file
```bash
touch file.txt
```

### Copy a file
```bash
cp file1.txt file2.txt
```

### Move (or rename) a file
```bash
mv file1.txt folder/
```

### Delete a file or folder
```bash
rm file.txt
rm -r folder/
```

---

## 📄 Viewing and Editing Files

### Show file contents
```bash
cat file.txt
```

### Scroll through a file
```bash
less file.txt
```

### Show first/last lines
```bash
head file.txt
tail file.txt
```

---

## 🔍 Searching and Finding Files

### Search inside files
```bash
grep "search_term" file.txt
```

### Find files by name
```bash
find . -name "*.txt"
```

---

## 🔄 Permissions and Ownership

### Check permissions
```bash
ls -l
```

### Change permissions
```bash
chmod 755 script.sh
```

### Change ownership (admin only)
```bash
sudo chown user:group file.txt
```

---

## 🛠️ Useful Commands

| Command | Description |
|---------|-------------|
| `man <command>` | Show manual/help |
| `history` | Show previously used commands |
| `clear` | Clear terminal screen |
| `whoami` | Show current user |

---

## 🧪 Practice Exercises

1. Create a folder named `unix_practice`
2. Inside it, create three `.txt` files
3. Copy one file to a subfolder
4. Use `grep` to search a keyword in a file
5. Change file permissions using `chmod`
6. Use `find` to locate all `.txt` files in your home directory

---
## 🛠️ Bash Scripting Basics
A script is a file of commands executed line by line.
Example: my_script.sh

```bash
#!/bin/bash
echo "Starting job..."
cd /home/user/myproject
python analysis.py
```
Run it with:
```bash
bash my_script.sh
```
---
## 📝 Writing & Submitting Jobs to a Scheduler (SLURM)
---
### Basic SLURM Script: job.slurm

```bash
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --output=out.txt
#SBATCH --time=00:10:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2

module load python
python3 analysis.py
```
---
### Submit and manage jobs

```bash
sbatch job.slurm      # Submit job
squeue -u $USER       # Check job status
scancel <jobid>       # Cancel job
```
---
## 🔍 Monitoring Jobs

```bah
top                  # Real-time CPU usage
htop                 # Enhanced top (if available)
squeue -u $USER      # Job queue
sacct -j <jobid>     # Job accounting info
```
---
## ✅ Best Practices for Bash in HPC

| Practice              | Example                        |
| --------------------- | ------------------------------ |
| Start with a shebang  | `#!/bin/bash`                  |
| Enable safe scripting | `set -euo pipefail`            |
| Use logging           | `command >> log.txt 2>&1`      |
| Use comments          | `# This part runs the model`   |
| Use variables         | `FILENAME=data.txt`            |
| Automate with loops   | `for f in *.txt; do ...; done` |

---

## 🚀 Quick Reference: Bash Cheat Sheet

| Command            | Description            |
| ------------------ | ---------------------- |
| `pwd`              | Show current directory |
| `cd dir/`          | Change directory       |
| `ls -l`            | List files             |
| `mkdir dir`        | Create folder          |
| `rm file`          | Delete file            |
| `cp a b`           | Copy a to b            |
| `mv a b`           | Rename/move a to b     |
| `bash script.sh`   | Run script             |
| `sbatch job.slurm` | Submit job             |
| `squeue -u $USER`  | View jobs              |



