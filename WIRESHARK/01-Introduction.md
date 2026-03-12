# Introduction

> **Navigation:** [Home](../wireshark-index) | [Next: Capture Basics →](./02-Capture-Basics)
---
## Table of Contents
- [What is Wireshark](#-what-is-wireshark)
- [How it Works](#how-it-works)
- [Installation](#installation)
  - [Windows](#Windows)
  - [Linux](#Linux.md)
  - [Docker](#Docker.md)
- [Terminology](#terminology.md)
- [Deep Dive](#Deep-Dive.md)

---

### What is Wireshark

Wireshark is an open-source **network protocol analyzer** — commonly called a **packet sniffer**.  It captures network traffic in real time and lets you inspect every packet traveling through an interface, down to the raw bytes.

It a tool oriented to perform network analysis, some uses include:

- Intercepting credentials.
- Analyzing protocols, understanding attack surfaces.
- Reconstructing incidents from captured traffic.
- Debugging network applications.
- Troubleshooting connectivity and performance issues.

> ⚠️ **Legal Notice**  
> - Capturing network traffic without authorization is **illegal** in most jurisdictions, check local law.  
> - Only use Wireshark on networks and systems you own or have **explicit written permission** to test.  

---

### How it Works

When data travels across a network, it moves in small chunks called **packets**.  
Normally, a network interface only processes packets addressed to it — but Wireshark puts the interface into **promiscuous mode**, which tells it to capture *all* packets it sees on the network segment, regardless of destination.

```
[ Network Traffic ]
        ▼
[ Network Interface (NIC*) ]
        │  ← Promiscuous Mode enabled
        ▼
[ Capture Library ]
  • libpcap  (Linux/macOS)
  • Npcap    (Windows)
        ▼
[ Wireshark Engine ]
  • Dissectors parse each protocol layer
  • Packets are decoded: Ethernet → IP → TCP → HTTP ...
        │
        ▼
[ You — GUI or tshark CLI ]
```

> `* NIC is any antenna in your hardware, including Wi-Fi, Ethernet, Bluetooth... 

---

### The capture library
Wireshark doesn't capture packets by itself — it relies on a system-level library:

| OS | Library | Notes |
|----|---------|-------|
| Linux / macOS | `libpcap` | Built into most distros |
| Windows | `Npcap` | Installed alongside Wireshark |
| Headless / Docker | `libpcap` | CLI via `tshark` |

### Dissectors
Once packets are captured, Wireshark uses **dissectors** — protocol-specific parsers — to decode each layer.  
It automatically identifies protocols (even on non-standard ports in many cases) and presents them in a human-readable format.

> 🔗 Want the full technical picture of the capture stack?  
> See [1.5 — Deep Dive](./01f-Deep-Dive.md)

---

### Installation
#### Windows
##### Download and Install
Go to the official download page — never use third-party sources:
[https://www.wireshark.org/download.html](https://www.wireshark.org/download.html)
Download the **Windows x64 Installer** (`.exe`).
Launch the `.exe` and follow the setup wizard. During installation you will be prompted to install **Npcap** — keep this checked. Npcap is the packet capture driver that Wireshark depends on to see network traffic.
##### Verifying the Installation
Verify installation with this commands (you should expect the installed verison)
```
wireshark --version
tshark --version
```
##### Run
Double click on the executable.
By default, Wireshark on Windows requires Administrator rights to capture.

---
#### Debian / Ubuntu
```
sudo apt update 
sudo apt install wireshark -y 
# If prompted, say yes
# For CLI use
sudo apt install tshark -y
```

#### RHEL / Fedora
```
# Fedora
sudo dnf install wireshark wireshark-cli -y 
# RHEL
sudo yum install epel-release -y 
sudo yum install wireshark wireshark-cli -y
```
#### Arch
```
sudo pacman -S wireshark-qt 
sudo pacman -S wireshark-cli
```
##### Verifying the Installation
```
wireshark --version 
tshark --version
```
##### Run
In Linux, capturing raw packets requires elevated privileges, execute always with sudo.

---


#### Docker



---
[Go to the top](#introduction)   
> **Navigation:** [Home](../Home.md) | [Next: Capture Basics →](./02-Capture-Basics.md)
---

