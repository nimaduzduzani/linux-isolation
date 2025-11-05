# 🧠 Linux cgroup v2 - Memory Limiting Example

## 📘 Description

**Control Groups (cgroups)** are a Linux kernel feature that allows system administrators and developers to allocate, limit, and monitor system resources — such as **CPU**, **memory**, **disk I/O**, and **network bandwidth** — for processes.  

With **cgroup v2**, Linux introduced a unified hierarchy that simplifies configuration and improves control efficiency.  
Cgroups are essential for technologies like **Docker**, **Podman**, and **Kubernetes**, where resource isolation is critical.

This example demonstrates how to manually create a cgroup, set memory limits, attach a process to it, and test its behavior under stress.

---

## 🧩 Steps to Create a Memory-Limited cgroup

### 1. Check the cgroup version
```bash
cat /proc/mounts | grep cgroup2
```
This verifies that cgroup version 2 is active on your system.

---

### 2. Navigate to the cgroup directory
```bash
cd /sys/fs/cgroup/
```

---

### 3. Create a new cgroup
```bash
sudo mkdir /sys/fs/cgroup/bash_limit
cd /sys/fs/cgroup/bash_limit
```
This creates a new memory control group named `bash_limit`.

---

### 4. Define memory limits
```bash
echo 100M | sudo tee /sys/fs/cgroup/bash_limit/memory.max
echo 80M | sudo tee /sys/fs/cgroup/bash_limit/memory.high
```
- **memory.max:** Hard memory limit for processes in the cgroup.  
- **memory.high:** Soft threshold where the kernel starts reclaiming memory.

---

### 5. Get your bash process ID
```bash
echo $$
ps
```
`$$` shows the PID of your current bash process.

---

### 6. Assign the process to the cgroup
```bash
echo $$ | sudo tee /sys/fs/cgroup/bash_limit/cgroup.procs
```
This adds your bash process to the newly created memory-limited cgroup.

---

### 7. Install the stress testing tool
```bash
sudo apt install stress-ng -y
```
`stress-ng` is a tool used to generate artificial load to test system performance under resource limits.

---

### 8. Run a memory stress test
```bash
stress-ng --vm 1 --vm-bytes 200M --timeout 30s
```
This command allocates 200MB of memory for 30 seconds, exceeding the 100MB limit defined earlier.  
You should observe the process being restricted or terminated as the kernel enforces cgroup limits.

---

## ⚙️ Summary

With **cgroup v2**, you can:
- Isolate and control resource usage per process or service
- Prevent runaway processes from consuming all system memory
- Reproduce container-like environments manually
- Understand low-level resource management used by container runtimes

---

## 🧑‍💻 Author

Created by **Nima Fakhr**  
Demonstration of **Linux cgroup v2** for process resource management and memory control.
