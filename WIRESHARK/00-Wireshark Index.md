A practical, hands-on reference for pentesters, CTF players, and network forensics analysts. > Part of the [Auditing](https://github.com/pegaxsus/Auditing) toolkit collection. 🦈

This guide is a structured, progressive reference for using Wireshark in real-world security contexts. It starts from the ground up — installation and UI — and goes deep into protocol analysis, threat hunting, CTF strategies, and CLI usage.

As you may suspect and noticed, it is partially made with AI, mixed with my knowledge on the field and focusing on the parts I usually make use on my work.

I made this as a method to have a handy online documentation so I dont have to rely on my memory or a physical pendrive.

---

### Audience

- Pentesters who want a public-online reference.
- Blue teamers and forensics analysts hunting threats.
- Anyone who wants to understand what's really on the wire.

---

### Contents

| # | Page | Description |
|---|------|-------------|
| 01 | [Introduction](./01-Introduction.md) | What is Wireshark, installation, UI tour |
| 02 | [Capture Basics](./02-Capture-Basics.md) | Interfaces, starting captures, file formats |
| 03 | [Capture Filters](./03-Capture-Filters.md) | BPF syntax and common capture filters |
| 04 | [Display Filters](./04-Display-Filters.md) | Filter language, operators, cheatsheet |
| 05 | [Protocol Analysis](./05-Protocol-Analysis.md) | HTTP, DNS, TLS, FTP, SMB deep dives |
| 06 | [Following Streams](./06-Following-Streams.md) | TCP/UDP/HTTP stream reassembly |
| 07 | [Exporting Objects](./07-Exporting-Objects.md) | Extracting files from captures |
| 08 | [Statistics & Graphs](./08-Statistics-and-Graphs.md) | Protocol hierarchy, I/O graphs, conversations |
| 09 | [tshark CLI](./09-tshark-CLI.md) | Terminal usage, scripting, remote capture |
| 10 | [CTF Usage](./10-CTF-Usage.md) | Strategies, common patterns, .pcap challenges |
| 11 | [Network Forensics](./11-Network-Forensics.md) | Threat hunting, IOC extraction, anomaly detection |
| 12 | [Coloring Rules](./12-Coloring-Rules.md) | Custom rules for quick visual triage |
| 13 | [Tips & Tricks](./13-Tips-and-Tricks.md) | Shortcuts, profiles, Lua scripts, plugins |

---

### 🔧 Part of the Auditing Toolkit

This Wireshark guide is one part of a broader collection of security tool references.  
More tools will be added over time.

| Tool | Status |
|------|--------|
| Wireshark | ✅ In progress |
| Nmap | 🔜 Coming soon |
| *More...* | 📋 Planned |

---

*Last updated: 2026 · Maintained by [pegaxsus](https://github.com/pegaxsus)*





























## Installation
1. Put the marker/probe in places SPAs commonly use:
    
    - Query string (`?q=...`)
        
    - Hash (`#q=...`)
        
2. In DevTools:
    
    - Network: confirm the server response didn’t already contain it (to distinguish reflected vs DOM).
        
    - Elements: confirm it appears only after JS runs (DOM-based).
        
3. Optional but strong: in **Sources → Event Listener Breakpoints**:
    
    - Enable **DOM Mutations** (subtree modifications / attribute modifications)
        
    - Reload and see what script modifies the DOM near the injection point
        
4. Look at the code: if you see patterns like assigning untrusted data into HTML-producing APIs, that’s typically your root cause.