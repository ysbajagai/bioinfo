---
layout: default
title: Unix Bash for HPC Users Workshop
---

# 🖥️ Unix Bash for High Performance Computing (HPC) Users

**Workshop Duration:** 4 hours  
**Target Audience:** Beginners to Intermediate Linux/HPC users

---

## 📌 Workshop Goals

- Learn essential bash commands  
- Understand Linux filesystem structure  
- Write and submit SLURM job scripts  
- Run parallel jobs with MPI  

---

## 🧠 Key Concepts

| Concept | Description |
|--------|-------------|
| **Shell (bash)** | Text-based interface to interact with the OS |
| **Cluster** | Group of computers working together |
| **Node** | One machine in a cluster |
| **Scheduler** | SLURM or PBS that manages job submissions |
| **Parallel Computing** | MPI (multi-node), OpenMP (multi-core) |

---

## 🛠️ Bash Commands

```bash
pwd
cd mydir
ls -lh
mkdir dir
cp a.txt b.txt
rm *.tmp
```

---

## ✍️ Bash Script Example

```bash
#!/bin/bash
echo "Job starting"
cd /home/user/myproject
python3 script.py
```

---

## 📤 SLURM Job Script

```bash
#!/bin/bash
#SBATCH --job-name=test
#SBATCH --output=output.txt
#SBATCH --time=00:30:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2

module load python
python3 script.py
```

---

## 🧩 Parallel Computing with MPI

### mpi_hello.c
```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
  MPI_Init(NULL, NULL);
  int rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  printf("Hello from %d\n", rank);
  MPI_Finalize();
  return 0;
}
```

Run it with:
```bash
mpicc mpi_hello.c -o hello
mpirun -np 4 ./hello
```

---

## 🧪 Exercises

- Create a bash script that lists all `.txt` files  
- Write a SLURM job to run a Python script  
- Compile and run a simple MPI job with 4 processes  

---

## 📚 Resources

- [SLURM Docs](https://slurm.schedmd.com/documentation.html)  
- [Bash Beginner’s Guide](https://tldp.org/LDP/Bash-Beginners-Guide/html/)  
- [MPI Tutorial](https://mpitutorial.com/)  
- [HPC University](https://hpcuniversity.org)  
