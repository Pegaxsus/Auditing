# Introduction

> **Navigation:** [Index](00-index) | [Next: Capture Basics →](./02-Capture-Basics)
---
## Table of Contents
- [What is Wireshark](#-what-is-wireshark)
- [How it Works](#how-it-works)
- [Installation](#installation)
  - [Windows](#windows)
  - [Linux](#Linux)
  - [Docker](#Docker)
- [Terminology](#terminology)
- [Deep Dive](#Deep-Dive)

---

## What is Wireshark

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

## How it Works

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

### The capture library
Wireshark doesn't capture packets by itself — it relies on a system-level library:

| OS | Library | Notes |
|----|---------|-------|
| Linux / macOS | `libpcap` | Built into most distros |
| Windows | `Npcap` | Installed alongside Wireshark |
| Headless / Docker | `libpcap` | CLI via `tshark` |

---

### Installation
[Go to the top](#introduction)   

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

#### Linux
##### Debian / Ubuntu
```
sudo apt update 
sudo apt install wireshark -y 
# If prompted, say yes
# For CLI use
sudo apt install tshark -y
```

##### RHEL / Fedora
```
# Fedora
sudo dnf install wireshark wireshark-cli -y 
# RHEL
sudo yum install epel-release -y 
sudo yum install wireshark wireshark-cli -y
```
##### Arch
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

> Notes
> - **Disposable, isolated** capture environment 
> - You are on a machine where you cannot install software natively 
> - You need **headless captures** on a remote server via SSH  
> - Docker containers share the host network stack when using `--net=host`.

##### Headless — tshark CLI

This is the most practical Docker use case — a lightweight container running tshark to capture or analyze traffic.

**1. Pull a base image and install tshark**
```dockerfile
# Dockerfile.tshark
FROM ubuntu:22.04

RUN apt update && apt install -y tshark && \
    apt clean && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["tshark"]
```

Build it:
```bash
docker build -f Dockerfile.tshark -t tshark-box .
```

**2. Run a live capture**
```bash
# Capture on host network interface eth0, 60 seconds
docker run --rm --net=host \
  --cap-add NET_RAW \
  --cap-add NET_ADMIN \
  tshark-box -i eth0 -a duration:60 -w /tmp/capture.pcap
```

- `--net=host` — container uses the host's network interfaces
- `--cap-add NET_RAW` — allows raw packet capture
- `--cap-add NET_ADMIN` — allows interface configuration (promiscuous mode)

**3. Analyze an existing `.pcap` file**
```bash
# Mount a local file into the container and analyze it
docker run --rm \
  -v /path/to/your/file.pcap:/data/capture.pcap \
  tshark-box -r /data/capture.pcap
```

**4. Save capture to host machine**
```bash
docker run --rm --net=host \
  --cap-add NET_RAW --cap-add NET_ADMIN \
  -v $(pwd)/captures:/output \
  tshark-box -i eth0 -a duration:30 -w /output/capture.pcap
```

The `-v` flag mounts your local `captures/` folder into the container — the `.pcap` persists after the container exits.

---

##### GUI — Wireshark via X11 Forwarding

Running the full Wireshark GUI inside Docker requires forwarding the display to your host machine via X11.

**Requirements on the host:**
- Linux with an X server running (standard desktop)
- `xhost` utility installed

**1. Allow Docker to access your display**
```bash
xhost +local:docker
```

This temporarily grants local Docker containers access to your X display.

**2. Dockerfile for GUI Wireshark**
```dockerfile
# Dockerfile.wireshark-gui
FROM ubuntu:22.04

RUN apt update && apt install -y wireshark-qt libgl1 && \
    apt clean && rm -rf /var/lib/apt/lists/*

# Allow non-root capture inside container
RUN usermod -aG wireshark root

ENTRYPOINT ["wireshark"]
```

Build it:
```bash
docker build -f Dockerfile.wireshark-gui -t wireshark-gui .
```

**3. Run with display forwarding**
```bash
docker run --rm \
  --net=host \
  --cap-add NET_RAW \
  --cap-add NET_ADMIN \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  wireshark-gui
```

- `-e DISPLAY=$DISPLAY` — passes your display variable into the container
- `-v /tmp/.X11-unix:/tmp/.X11-unix` — mounts the X11 socket

The full Wireshark GUI should open on your host desktop.

> 💡 On macOS, X11 forwarding requires **XQuartz** installed and running.  
> On Windows, use **VcXsrv** or **WSL2 with WSLg** for X11 support.

---

##### Saving Captures Outside the Container

Always mount a host volume when you want to keep capture files:
```bash
-v $(pwd)/captures:/output
```

Then write captures to `/output` inside the container. Files will appear in `./captures/` on your host.

---


### Terminology
[Go to the top](#introduction)   

#### Network Fundamentals

These are the building blocks. If you already know networking, skim this — but the definitions here are intentionally framed for packet analysis context, not just general networking.

| Term | Definition |
|------|-----------|
| **Packet** | A unit of data transmitted over a network. Contains a header (metadata) and a payload (actual data). |
| **Frame** | A packet at the data link layer (Layer 2). Includes MAC addresses. A frame carries a packet inside it. |
| **Segment** | A unit of data at the transport layer (Layer 4 — TCP). TCP breaks streams into segments. |
| **Datagram** | A self-contained unit at the network layer (Layer 3 — IP/UDP). No guaranteed delivery. |
| **Header** | Metadata prepended to data at each layer. Contains source/destination addresses, protocol info, flags. |
| **Payload** | The actual data being carried. What the header is wrapping. |
| **MTU** | Maximum Transmission Unit. The largest packet size an interface can handle (typically 1500 bytes on Ethernet). |
| **Latency** | Time for a packet to travel from source to destination. Visible in Wireshark via delta timestamps. |
| **Bandwidth** | The capacity of a network link, measured in bits per second. |
| **Socket** | A combination of IP address + port. Identifies a specific communication endpoint. |

#### Capture Concepts

| Term | Definition |
|------|-----------|
| **Promiscuous Mode** | NIC mode where the interface captures ALL packets on the segment, not just those addressed to it. Required for sniffing. |
| **Monitor Mode** | Wi-Fi specific. Captures raw 802.11 frames including management/control frames, without associating to an AP. |
| **Live Capture** | Capturing packets in real time from a network interface. |
| **Offline Analysis** | Opening and analyzing a previously saved `.pcap` file. No live traffic involved. |
| **Ring Buffer** | A capture strategy that writes to multiple files in rotation, overwriting oldest files. Prevents disk exhaustion on long captures. |
| **Snaplen** | Snapshot length. The maximum number of bytes captured per packet. Default is 262144 bytes (full packet). Reducing it speeds up capture but truncates payloads. |
| **BPF** | Berkeley Packet Filter. The low-level filter language used at capture time to limit what gets captured. Runs in the kernel for efficiency. More in [03 — Capture Filters](./03-Capture-Filters.md). |
| **pcap** | The standard file format for packet captures. Extension `.pcap`. |
| **pcapng** | Next-generation pcap format. Supports multiple interfaces, comments, and metadata in one file. Wireshark's default save format. |

#### Wireshark-Specific Terms
| Term | Definition |
|------|-----------|
| **Dissector** | A protocol parser built into Wireshark. Each protocol (HTTP, DNS, TLS...) has its own dissector that decodes raw bytes into human-readable fields. |
| **Display Filter** | A filter applied *after* capture to show only matching packets. Does not affect what was captured. More in [04 — Display Filters](./04-Display-Filters.md). |
| **Capture Filter** | A BPF filter applied *during* capture to limit what gets saved. Cannot be changed mid-capture. More in [03 — Capture Filters](./03-Capture-Filters.md). |
| **Follow Stream** | A Wireshark feature that reassembles a full TCP/UDP/HTTP conversation from individual packets. More in [06 — Following Streams](./06-Following-Streams.md). |
| **Coloring Rule** | A rule that assigns colors to packets matching a display filter. Used for fast visual triage. More in [12 — Coloring Rules](./12-Coloring-Rules.md). |
| **Profile** | A saved Wireshark configuration (filters, columns, coloring rules). Allows switching between setups per engagement type. |
| **Packet List** | The top panel in the Wireshark UI. Shows one row per packet with summary info. |
| **Packet Details** | The middle panel. Shows the decoded protocol tree for the selected packet. |
| **Packet Bytes** | The bottom panel. Shows the raw hex and ASCII representation of the selected packet. |
| **Expert Info** | Wireshark's built-in analysis system that flags anomalies — retransmissions, resets, malformed packets. |
| **IO Graph** | A graph of packet/byte rate over time. Useful for spotting traffic spikes and patterns. More in [08 — Statistics & Graphs](./08-Statistics-and-Graphs.md). |
| **tshark** | The terminal-based version of Wireshark. Same engine, no GUI. More in [09 — tshark CLI](./09-tshark-CLI.md). |
| **dumpcap** | The underlying capture binary Wireshark and tshark use. Can run standalone for lightweight captures. |


#### Protocol Stack Terms
Wireshark always shows packets decoded by layer. Knowing these terms helps you navigate the packet details panel.

| Layer | Name | Common Protocols | What Wireshark Shows |
|-------|------|-----------------|---------------------|
| Layer 2 | Data Link | Ethernet, 802.11 | MAC src/dst, frame type |
| Layer 3 | Network | IP, IPv6, ICMP | IP src/dst, TTL, fragmentation |
| Layer 4 | Transport | TCP, UDP | Ports, flags, sequence numbers |
| Layer 5-7 | Application | HTTP, DNS, TLS, FTP, SMB | Protocol-specific fields | Key terms you'll see constantly in Wireshark: | Term | Definition | |------|-----------| | **TTL** | Time To Live. Decremented at each router hop. Useful for OS fingerprinting and detecting spoofed packets. | | **Seq / Ack** | TCP sequence and acknowledgment numbers. Track the order and delivery of segments in a stream. | | **SYN / FIN / RST** | TCP flags. SYN initiates a connection, FIN closes it gracefully, RST closes it abruptly. | | **Checksum** | An error-detection value. Wireshark may flag checksum errors — often benign due to offloading. | | **TLS Handshake** | The negotiation phase before encrypted data flows. Visible in Wireshark even without decryption. More in [05 — Protocol Analysis](./05-Protocol-Analysis.md). | | **DNS Query / Response** | A request to resolve a domain to an IP, and its answer. Extremely useful in forensics and threat hunting. | --- ## Pentesting Context Terms Terms that come up specifically in offensive / forensic use of Wireshark: | Term | Definition | |------|-----------| | **Credential Harvesting** | Extracting usernames and passwords from unencrypted protocols (HTTP, FTP, Telnet, SMTP). | | **Stream Reassembly** | Reconstructing a full application-layer conversation from raw packets. Essential for reading transferred files or commands. | | **IOC** | Indicator of Compromise. An observable artifact in traffic — a suspicious IP, domain, user-agent, or pattern. | | **C2 Traffic** | Command and Control. Traffic between malware and its operator. Often disguised as legitimate protocols (HTTP, DNS). | | **Lateral Movement** | Attacker activity moving between hosts inside a network. Visible as unusual SMB, RDP, or WMI traffic. | | **Exfiltration** | Data being stolen out of a network. Look for large outbound transfers, DNS tunneling, or unusual destinations. | | **PCAP Carving** | Extracting files, images, or documents from a packet capture. More in [07 — Exporting Objects](./07-Exporting-Objects.md). | | **Traffic Baseline** | Normal traffic patterns for a network. Knowing the baseline makes anomalies obvious. | | **Beaconing** | Regular, periodic connections from a compromised host to a C2 server. Often visible as evenly-spaced packets to the same IP. | ---

Key terms you'll see constantly in Wireshark:

| Term | Definition |
|------|-----------|
| **TTL** | Time To Live. Decremented at each router hop. Useful for OS fingerprinting and detecting spoofed packets. |
| **Seq / Ack** | TCP sequence and acknowledgment numbers. Track the order and delivery of segments in a stream. |
| **SYN / FIN / RST** | TCP flags. SYN initiates a connection, FIN closes it gracefully, RST closes it abruptly. |
| **Checksum** | An error-detection value. Wireshark may flag checksum errors — often benign due to offloading. |
| **TLS Handshake** | The negotiation phase before encrypted data flows. Visible in Wireshark even without decryption. More in [05 — Protocol Analysis](./05-Protocol-Analysis.md). |
| **DNS Query / Response** | A request to resolve a domain to an IP, and its answer. Extremely useful in forensics and threat hunting. |

#### Pentesting Context Terms
Terms that come up specifically in offensive / forensic use of Wireshark:

| Term | Definition |
|------|-----------|
| **Credential Harvesting** | Extracting usernames and passwords from unencrypted protocols (HTTP, FTP, Telnet, SMTP). |
| **Stream Reassembly** | Reconstructing a full application-layer conversation from raw packets. Essential for reading transferred files or commands. |
| **IOC** | Indicator of Compromise. An observable artifact in traffic — a suspicious IP, domain, user-agent, or pattern. |
| **C2 Traffic** | Command and Control. Traffic between malware and its operator. Often disguised as legitimate protocols (HTTP, DNS). |
| **Lateral Movement** | Attacker activity moving between hosts inside a network. Visible as unusual SMB, RDP, or WMI traffic. |
| **Exfiltration** | Data being stolen out of a network. Look for large outbound transfers, DNS tunneling, or unusual destinations. |
| **PCAP Carving** | Extracting files, images, or documents from a packet capture. More in [07 — Exporting Objects](./07-Exporting-Objects.md). |
| **Traffic Baseline** | Normal traffic patterns for a network. Knowing the baseline makes anomalies obvious. |
| **Beaconing** | Regular, periodic connections from a compromised host to a C2 server. Often visible as evenly-spaced packets to the same IP. |



---

### DEEP-DIVE
[Go to the top](#introduction)   

#### The Capture Stack
Most people think of Wireshark as one program. In reality it is a **pipeline of components**, each with a specific responsibility:
```
┌─────────────────────────────────────────────────────┐
│                   NETWORK INTERFACE                  │
│              (physical or virtual NIC)               │
└───────────────────────┬─────────────────────────────┘
                        │ raw frames
                        ▼
┌─────────────────────────────────────────────────────┐
│                   KERNEL SPACE                       │
│                                                      │
│   ┌─────────────────────────────────────────────┐   │
│   │         Socket / Packet Filter              │   │
│   │   AF_PACKET (Linux) / NDIS (Windows)        │   │
│   │                                             │   │
│   │   BPF program runs HERE — in kernel space   │   │
│   │   Matching packets copied to ring buffer    │   │
│   └─────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────┘
                        │ filtered packets
                        ▼
┌─────────────────────────────────────────────────────┐
│                   USER SPACE                         │
│                                                      │
│   ┌──────────────┐     ┌────────────────────────┐   │
│   │   libpcap    │     │        Npcap           │   │
│   │ (Linux/macOS)│     │       (Windows)        │   │
│   └──────┬───────┘     └───────────┬────────────┘   │
│          └──────────────┬──────────┘                 │
│                         ▼                            │
│                    ┌─────────┐                       │
│                    │ dumpcap │  ← capture engine     │
│                    └────┬────┘                       │
│                         │                            │
│              ┌──────────┴──────────┐                 │
│              ▼                     ▼                 │
│          ┌────────┐          ┌──────────┐            │
│          │ tshark │          │Wireshark │            │
│          │  (CLI) │          │  (GUI)   │            │
│          └────────┘          └──────────┘            │
└─────────────────────────────────────────────────────┘
```

Each layer has a specific job:

| Layer | Responsibility |
|-------|---------------|
| NIC | Receives raw electrical/optical signals, decodes into frames |
| Kernel socket | Exposes raw frames to userspace via `AF_PACKET` (Linux) or NDIS (Windows) |
| BPF | Filters packets in kernel space before they reach userspace — efficient |
| libpcap / Npcap | Provides a portable API over the kernel socket layer |
| dumpcap | Handles actual packet capture and writes to `.pcapng` files |
| tshark / Wireshark | Reads from dumpcap, dissects protocols, presents to the user |

---

#### How the Kernel Sees Traffic

On Linux, when a packet arrives at a NIC:

1. The NIC driver copies the frame into a **kernel ring buffer** via DMA (Direct Memory Access)
2. The kernel raises a **soft interrupt** (`NET_RX_SOFTIRQ`) to process the frame
3. The frame travels up the **network stack**: NIC driver → `netif_receive_skb` → protocol handlers
4. If a raw socket (`AF_PACKET`) is open, a **copy** of the frame is delivered there before any routing decisions

This is why Wireshark sees traffic that isn't destined for your machine — it taps in at the raw socket level, before the kernel decides to drop or forward it.
```
Packet arrives
      │
      ▼
  NIC Driver
      │
      ▼
 netif_receive_skb
      │
      ├──────────────────────► AF_PACKET socket ──► libpcap ──► dumpcap
      │                        (Wireshark sees it HERE)
      ▼
 IP routing / TCP stack
 (normal processing)
```

---

#### libpcap and Npcap Internals

**libpcap** (Linux/macOS) abstracts the platform-specific socket APIs:

- Opens an `AF_PACKET` socket in `SOCK_RAW` mode
- Sets the interface to **promiscuous mode** via `SIOCSIFFLAGS`
- Attaches a **BPF program** to the socket for kernel-level filtering
- Reads packets from the kernel ring buffer (`PACKET_MMAP` for efficiency)

**Npcap** (Windows) works differently because Windows has no `AF_PACKET`:

- Installs a **kernel-mode driver** (NDIS lightweight filter driver)
- The driver sits between the NIC driver and the Windows network stack
- Intercepts frames at the NDIS layer and copies them to userspace
- Also supports **loopback capture** (which WinPcap could not do)

This is why Npcap requires a driver installation — it's not just a library, it modifies the Windows network stack.

---

#### BPF — Berkeley Packet Filter

BPF is one of the most elegant pieces of the capture stack.  
When you write a capture filter like `tcp port 80`, Wireshark compiles it into a **BPF bytecode program** and loads it into the kernel.

The kernel then runs this tiny program against every incoming packet **before copying it to userspace**. Packets that don't match are discarded at the kernel level — they never reach Wireshark.
```
Capture filter: "tcp port 80"
        │
        ▼
  Compiled to BPF bytecode:
  ┌─────────────────────────────────────┐
  │  ldh  [12]          ; load ethertype│
  │  jeq  #0x800, L1    ; is it IPv4?   │
  │  ret  #0            ; no → drop     │
  │  L1: ldb [23]       ; load protocol │
  │  jeq  #6, L2        ; is it TCP?    │
  │  ret  #0            ; no → drop     │
  │  L2: ldh [34]       ; load dst port │
  │  jeq  #80, pass     ; port 80?      │
  │  ldh  [36]          ; load src port │
  │  jeq  #80, pass     ; port 80?      │
  │  ret  #0            ; no → drop     │
  │  pass: ret #65535   ; accept packet │
  └─────────────────────────────────────┘
        │
        ▼
  Runs in kernel — zero context switching per packet
```

**Why this matters for pentesters:**  
Capture filters are far more efficient than display filters.  
On a busy network, using a capture filter instead of capturing everything can be the difference between a clean capture and thousands of dropped packets.

> 🔗 More on writing capture filters: [03 — Capture Filters](./03-Capture-Filters.md)

---

#### Dumpcap, tshark and Wireshark — Who Does What

Many people don't realize Wireshark splits responsibilities across three binaries:

| Binary | Role | Can run standalone? |
|--------|------|-------------------|
| `dumpcap` | Raw packet capture only. Writes `.pcapng`. Minimal privileges needed. | ✅ Yes |
| `tshark` | Capture + dissection + output. CLI. Calls dumpcap internally. | ✅ Yes |
| `wireshark` | GUI frontend. Calls dumpcap for capture, dissects and renders. | ✅ Yes |

This separation exists for **security reasons**:  
Only `dumpcap` needs elevated privileges to open raw sockets.  
`tshark` and `wireshark` run as your normal user and communicate with `dumpcap` via a pipe.
```
wireshark (your user)
    │
    │ spawns
    ▼
dumpcap (wireshark group / cap_net_raw)
    │
    │ writes packets via pipe
    ▼
wireshark dissects and renders
```

You can use `dumpcap` directly for minimal-overhead headless captures:
```bash
# Capture on eth0, write to file, no dissection overhead
dumpcap -i eth0 -w /tmp/capture.pcapng

# Capture with a BPF filter
dumpcap -i eth0 -f "tcp port 443" -w /tmp/tls.pcapng

# Ring buffer — 10 files of 100MB each
dumpcap -i eth0 -b filesize:102400 -b files:10 -w /tmp/ring.pcapng
```

---

#### Dissector Engine

When Wireshark reads a packet, it runs it through a **dissector chain**:
```
Raw bytes
    │
    ▼
Frame dissector       → Ethernet header
    │
    ▼
IP dissector          → IPv4/IPv6 header
    │
    ▼
TCP/UDP dissector     → Transport header, ports
    │
    ▼
Application dissector → HTTP / DNS / TLS / SMB ...
    │
    ▼
Human-readable fields in Packet Details panel
```

Wireshark selects dissectors by:
1. **Port heuristics** — TCP 80 → try HTTP dissector
2. **Heuristic dissectors** — try to detect protocol from payload patterns
3. **User override** — right-click → Decode As → force a specific dissector

**Why this matters:**  
On non-standard ports (e.g. HTTP on port 8080, or C2 over port 443), Wireshark may not automatically pick the right dissector.  
Knowing how to force a dissector is essential for protocol analysis in pentesting.

> 🔗 Decode As and custom dissectors: [13 — Tips & Tricks](./13-Tips-and-Tricks.md)

---

#### Promiscuous vs Monitor Mode — Technical Difference

These are often confused. They operate at different layers and serve different purposes:

| | Promiscuous Mode | Monitor Mode |
|--|-----------------|-------------|
| **Layer** | Layer 2 — Ethernet | Layer 1/2 — 802.11 RF |
| **What it captures** | All Ethernet frames on the segment | All 802.11 frames in the air (including management, control) |
| **Requires association** | No | No — completely passive |
| **Works on wired** | ✅ Yes | ❌ No |
| **Works on Wi-Fi** | ✅ Yes (but limited) | ✅ Yes (full RF capture) |
| **Sees other networks** | ❌ No | ✅ Yes (all SSIDs on channel) |
| **Enable in Wireshark** | Capture Options → ✅ Promiscuous | Capture Options → ✅ Monitor |

**In promiscuous mode on Wi-Fi:**  
The adapter is still associated to an AP and only sees frames on that BSS.  
You get more frames than normal mode but not raw 802.11.

**In monitor mode on Wi-Fi:**  
The adapter decouples from any AP entirely.  
You capture raw 802.11 frames — beacon frames, probe requests, authentication, deauth, and data — across all networks on the current channel.  
This is what you need for WPA handshake capture.

> ⚠️ Monitor mode requires a compatible wireless adapter and drivers.  
> Not all adapters support it. On Linux, `iw list` shows if `monitor` is in the supported interface modes.

---

#### Capture Privileges — Why Root Isn't the Answer

Running Wireshark as root gives it access to everything on the system.  
A vulnerability in the dissector engine (which parses untrusted network data) would be exploited with root privileges.

The correct privilege model uses Linux **capabilities**:
```bash
# View dumpcap's capabilities
getcap $(which dumpcap)
# Output: /usr/bin/dumpcap cap_net_admin,cap_net_raw=eip

# cap_net_raw  → open raw sockets (needed for capture)
# cap_net_admin → set interface flags (needed for promiscuous mode)
```

These two capabilities are the **minimum required** for packet capture.  
By assigning them only to `dumpcap` (not to Wireshark or tshark), the attack surface is minimized.

If dumpcap is missing these capabilities (e.g. after a manual install):
```bash
sudo setcap cap_net_raw,cap_net_admin=eip $(which dumpcap)
```

---

#### Performance — What Happens When You Drop Packets

On high-traffic networks, Wireshark can drop packets. Here's why and how to mitigate it:

**Drop causes:**
- Kernel ring buffer fills faster than userspace reads it
- Dissection overhead in the GUI causes processing lag
- Disk I/O too slow to write pcapng in real time

**How to detect drops:**
```bash
# tshark shows drop stats at end of capture
tshark -i eth0 -a duration:10
# Output includes: X packets captured, Y packets dropped
```

**How to reduce drops:**

| Technique | How |
|-----------|-----|
| Use a capture filter | Reduce volume before it hits userspace |
| Use `dumpcap` directly | No dissection overhead during capture |
| Increase ring buffer size | `dumpcap -B 64` (64MB kernel buffer) |
| Write to fast storage | Use SSD or tmpfs (`/dev/shm`) for capture files |
| Disable GUI rendering | Capture to file, analyze later |

---

#### Wireshark on High-Speed Networks

Standard Wireshark struggles above ~1 Gbps sustained traffic.  
For high-speed environments (red team infrastructure, data center taps), alternatives exist:

| Tool | Approach | Use Case |
|------|---------|---------|
| `dumpcap -B` | Larger kernel buffer | Moderate traffic spikes |
| `tcpdump` | Lightweight, less overhead | Quick CLI captures |
| `PF_RING` | Kernel bypass, zero-copy | 10Gbps+ capture |
| `DPDK` | Full kernel bypass | Line-rate capture on servers |
| `Zeek` | Network analysis framework | High-speed logging without full pcap |
| Hardware TAPs | Physical layer capture | No software overhead at all |

> 💡 For most pentesting engagements you will never hit these limits.  
> This becomes relevant when capturing on trunk ports, core switches, or monitoring egress on a busy server.

---
[Go to the top](#introduction)   
> **Navigation:** [Home](00-wireshark-index.md) | [Next: Capture Basics →](./02-Capture-Basics)
---

