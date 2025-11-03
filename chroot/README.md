# 🧱 SSH Chroot Jail Guide

## What Is Chroot Jail?

**Chroot Jail** refers to creating a virtual “prison” for users where they can only access a specific directory defined by the administrator.  
This technique uses the **Chroot** capability of the operating system. Users cannot move outside of the defined directory and can only access files and tools located within it.

### Common Use Cases
- Increase security for SSH users  
- Isolate specific users  
- Prevent possible misuse on multi-user servers  

---

## ⚙️ Steps to Configure a Chroot Jail

### 1. Create the Jail Directory
First, create a separate environment for users who will be restricted:

```bash
sudo mkdir -p /home/test
```

This directory will serve as the isolated environment.

Create necessary device files (`null`, `zero`, `stdin`, `stdout`, `stderr`, `random`, `tty`) for shell operations:

```bash
mkdir -p /home/test/dev/
cd /home/test/dev/
mknod -m 666 null c 1 3
mknod -m 666 tty c 5 0
mknod -m 666 zero c 1 5
mknod -m 666 random c 1 8
```

Set correct permissions:

```bash
chown root:root /home/test
chmod 0755 /home/test
ls -ld /home/test
```

---

### 2. Create a New User

```bash
sudo useradd -m -d /home/test/username username
sudo passwd username
```

- `-m`: Creates the user’s home directory  
- `-d`: Specifies the home path  

---

### 3. Copy Essential Files and Tools

```bash
sudo mkdir -p /home/test/bin
sudo cp /bin/bash /home/test/bin/
```

> Note: Programs used inside the jail (like `bash`) must include their libraries.  
> Use `ldd` to find dependencies:

```bash
ldd /bin/bash
```

Example (for `x86_64` or `aarch64`):
```bash
mkdir -p /home/test/lib64 /home/test/lib/aarch64-linux-gnu
cp -v /lib64/{libtinfo.so.6,libdl.so.2,libc.so.6,ld-linux-aarch64.so.1} /home/test/lib64/
```

---

### 4. Configure SSH for Chroot

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add:

```bash
Match User username
ChrootDirectory /home/test
AllowTCPForwarding no
X11Forwarding no
```

Create necessary system files inside the jail:

```bash
mkdir -p /home/test/etc
cp -vf /etc/{passwd,group} /home/test/etc/
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

---

### 5. Test the Jail

```bash
ssh username@localhost
```

You should now be inside `/home/test` and unable to access files outside it.

To add more commands for the user:
1. Check the `$PATH` after login  
2. Copy the executable into a directory in `$PATH`  
3. Use `ldd` to copy required libraries into `/lib` or `/lib64`

---

## ✅ Benefits of Using Chroot Jail

- **High Security:** Users only access necessary files  
- **Reduced Risk:** Minimizes potential internal misuse  
- **Better Management:** Isolate multiple users easily  

---

### 🧩 Common Error

If you see:
```
/bin/sh: No such file or directory
Connection closed.
```

It means your jail is missing `/bin/bash` or its required libraries.  
Fix it by running:

```bash
sudo cp /bin/bash /home/test/bin/
sudo ln -s bash /home/test/bin/sh
ldd /bin/bash
# Copy all listed libraries into /home/test/lib*/ accordingly
```

---

### 🧠 Author
Created by **Nima Fakhr**  
Based on Debian GNU/Linux configuration and practical SSH hardening steps.
