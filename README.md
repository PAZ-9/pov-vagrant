# Vagrant NAT Networking Lab

> Understanding how Vagrant virtual machines communicate with external networks through VirtualBox NAT networking.

---

## Overview

This lab demonstrates the default networking behavior of Vagrant virtual machines running on VirtualBox.

The objective is to understand:

* How Vagrant provisions virtual machines
* VirtualBox NAT networking
* Default routing inside a VM
* Internet connectivity from guest machines
* Traceroute path analysis
* NAT gateway functionality

---

## Lab Architecture

```text
                     Internet
                         │
                         │
                  Home Router
                  10.130.68.1
                         │
                    Host NIC
                  10.130.68.4
                         │
                   (NAT Applied)
                         │
    ┌─────────────────────────────────────┐
    │          VirtualBox Host            │
    │                                     │
    │ VirtualBox NAT Gateway              │
    │        10.0.2.2                     │
    │                                     │
    │  ubuntu_node1   10.0.2.15           │
    │  ubuntu_node2   10.0.2.15           │
    │  ubuntu_node3   10.0.2.15           │
    │                                     │
    └─────────────────────────────────────┘
```
<img width="912" height="869" alt="image" src="https://github.com/user-attachments/assets/e07ccc35-8cf4-4f06-b637-3f6c1ca79857" />


---

## Environment

### Host Machine

```text
Ubuntu 22.04 LTS
VirtualBox
Vagrant
```

### Vagrant Version

Verify installation:

```bash
vagrant --version
```

### VirtualBox Version

```bash
VBoxManage --version
```

---

## Install Vagrant

Download and install Vagrant from HashiCorp:

```text
https://portal.cloud.hashicorp.com/vagrant
```

Verify installation:

```bash
vagrant --version
```

---

## Create Project Directory

```bash
mkdir vagrant-nat-lab
cd vagrant-nat-lab
```

Initialize Vagrant:

```bash
vagrant init ubuntu/jammy64
```

---

## Vagrant Configuration

Example `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
end
```

---

## Start Virtual Machine

```bash
vagrant up
```

Expected output:

```text
Bringing machine 'default' up with 'virtualbox' provider...
```

Connect to the VM:

```bash
vagrant ssh
```

---

## Verify IP Configuration

Inside the VM:

```bash
ip addr
```

Expected:

```text
eth0
10.0.2.15/24
```

VirtualBox assigns a private IP address from its NAT network.

---

## Verify Routing Table

Inside the VM:

```bash
ip route
```

Example output:

```text
default via 10.0.2.2 dev eth0

10.0.2.0/24 dev eth0 proto kernel
10.0.2.2 dev eth0 proto dhcp
10.0.2.3 dev eth0 proto dhcp
```

### Route Analysis

| Address     | Purpose                  |
| ----------- | ------------------------ |
| 10.0.2.15   | VM IP Address            |
| 10.0.2.2    | VirtualBox NAT Gateway   |
| 10.0.2.3    | VirtualBox DNS Proxy     |
| 10.130.68.1 | Physical Network Gateway |

---

## Verify Internet Connectivity

Ping Google DNS:

```bash
ping 8.8.8.8
```

Verify DNS:

```bash
ping google.com
```

Expected:

```text
64 bytes from 8.8.8.8
```

---

## Traceroute Analysis

Install traceroute:

```bash
sudo apt update
sudo apt install traceroute -y
```

Run:

```bash
traceroute 8.8.8.8
```

Example:

```text
1  10.0.2.2
2  10.130.68.1
3  ISP Network
4  ISP Core Router
...
```

### Path Explanation

```text
VM
 │
 ▼
10.0.2.2
(VirtualBox NAT Gateway)
 │
 ▼
10.130.68.1
(Home Router)
 │
 ▼
ISP Network
 │
 ▼
Internet
 │
 ▼
8.8.8.8
```

This confirms that VirtualBox performs Network Address Translation (NAT) before forwarding traffic to the physical network.

---

## Understanding VirtualBox NAT

By default, Vagrant creates:

```text
Adapter 1 = NAT
```

Benefits:

* Internet access works automatically
* No additional network configuration required
* Safe isolation from the local network

Limitations:

* VM cannot be directly reached from other devices
* Inbound access requires port forwarding
* VM-to-VM communication requires additional adapters

---

## Inspect VirtualBox Network Settings

From the host:

```bash
VBoxManage showvminfo <vm-name>
```

Look for:

```text
NIC 1: NAT
```

---

## Learning Outcomes

After completing this lab, you will understand:

* Vagrant fundamentals
* VirtualBox NAT networking
* Default VM routing
* NAT gateway behavior
* DNS proxy operation
* Internet connectivity troubleshooting
* Traceroute analysis

---

## Cleanup

Stop VM:

```bash
vagrant halt
```

Destroy VM:

```bash
vagrant destroy -f
```

Remove project:

```bash
cd ..
rm -rf vagrant-nat-lab
```

---

## Key Takeaways

* Vagrant uses VirtualBox NAT networking by default.
* Virtual machines receive the IP address `10.0.2.15`.
* The NAT gateway is `10.0.2.2`.
* DNS requests are handled by `10.0.2.3`.
* Internet traffic is translated and forwarded through the host machine and physical router.
* NAT provides outbound connectivity while keeping guest machines isolated.

---

## References

* Vagrant Documentation
* VirtualBox Networking Documentation
* Ubuntu Networking Guide

---

