# 🖧 Network Namespaces and Virtual Ethernet (veth) Setup

## Description

This project demonstrates how to create isolated network environments on a Linux system using **Network Namespaces** and **Virtual Ethernet (veth) pairs**.  
Network Namespaces allow multiple isolated network stacks to exist on a single host, and veth pairs connect these namespaces so they can communicate with each other.

This setup is useful for testing networking configurations, container networking, or simulating multi-host environments without additional hardware.

---
## Steps to Create Network Namespaces and veth Pairs


### 1. Create Network Namespaces
```bash
sudo ip netns add ns1
sudo ip netns add ns2
```

### 2. Create a Virtual Ethernet Pair
```bash
sudo ip link add veth1 type veth peer name veth2
```

### 3. Assign Interfaces to Namespaces
```bash
sudo ip link set veth1 netns ns1
sudo ip link set veth2 netns ns2
```

### 4. Configure Interfaces and Assign IPs
```bash
# Namespace ns1
sudo ip netns exec ns1 ip link set veth1 up
sudo ip netns exec ns1 ip addr add 192.168.100.1/24 dev veth1

# Namespace ns2
sudo ip netns exec ns2 ip link set veth2 up
sudo ip netns exec ns2 ip addr add 192.168.100.2/24 dev veth2
```

### 5. Test Connectivity Between Namespaces
```bash
sudo ip netns exec ns1 ping -c 3 192.168.100.2
sudo ip netns exec ns2 ping -c 3 192.168.100.1
```

### 6. Create a Shell with `unshare` and Connect to ns1
```bash
# Create a new shell in a separate network namespace
sudo unshare --net bash &
UNSHARE_PID=$!

# Connect this shell to ns1
sudo nsenter --net=/var/run/netns/ns1 -t $UNSHARE_PID bash

# Test connectivity from the shell to ns2
ping -c 3 192.168.100.2
```

> In this step, you create a separate shell with its own network namespace and then attach it to `ns1` to interact with its network stack.

---

## Benefits

- **Isolation:** Each namespace has its own network stack (interfaces, IP routing, firewall rules).  
- **Experimentation:** Safe environment to test network configurations without affecting the host.  
- **Learning:** Understand Linux networking, containers, and network simulation.

---

## Author

Created by **Nima Fakhr**
