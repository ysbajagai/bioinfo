---
layout: default
title: 🖥️ Unix Bash for High Performance Computing (HPC)
---
# Unix Bash for High Performance Computing (HPC)

## What is Unix/Linux?

Unix is a powerful multiuser operating system. Linux is a Unix-like system commonly used in HPC. You interact with it using a command-line interface (CLI) or shell (bash).
Welcome to the beginner’s guide to Unix and the command line interface for HPC users.

## Topics Covered

- Navigating the Linux file system
- Working with Files and Directories
- Viewing and Editing Files
- Searching and Finding Files
- Permissions and Ownership
- Useful Commands
- Basic shell scripting
- Submitting jobs to SLURM
- Running parallel jobs with MPI

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
