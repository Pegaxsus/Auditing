
### Filter by Host / IP

| Filter | Description |
|--------|-------------|
| `host 192.168.1.1` | Traffic to/from IP |
| `src host 192.168.1.1` | Traffic from IP only |
| `dst host 192.168.1.1` | Traffic to IP only |
| `not host 192.168.1.1` | Exclude IP |
| `net 192.168.1.0/24` | Entire subnet |
| `src net 10.0.0.0/8` | From RFC1918 range |
| `host 192.168.1.1 and host 192.168.1.2` | Between two hosts |

### Filter by Port

| Filter | Description |
|--------|-------------|
| `port 80` | HTTP |
| `port 443` | HTTPS |
| `port 53` | DNS |
| `dst port 443` | Outbound HTTPS |
| `src port 53` | DNS responses |
| `port 80 or port 443` | HTTP + HTTPS |
| `tcp port 80` | TCP HTTP only |
| `udp port 53` | UDP DNS only |
| `portrange 1-1023` | Well-known ports |
| `not port 53` | Exclude DNS |

### Filter by MAC Address

| Filter | Description |
|--------|-------------|
| `ether host aa:bb:cc:dd:ee:ff` | Traffic to/from MAC |
| `ether src aa:bb:cc:dd:ee:ff` | From MAC only |
| `ether dst aa:bb:cc:dd:ee:ff` | To MAC only |
| `ether broadcast` | All broadcast frames |
| `ether multicast` | All multicast frames |

### Filter by Protocols

| Filter | Description |
|--------|-------------|
| `tcp` | TCP only |
| `udp` | UDP only |
| `icmp` | ICMP only |
| `arp` | ARP only |
| `not arp` | Exclude ARP |
| `not arp and not port 53` | Exclude ARP + DNS noise |

### Pentesting Quick Filters

| Filter | Use case |
|--------|---------|
| `port 21 or port 23 or port 110 or port 143` | Plaintext credential protocols |
| `src net 192.168.1.0/24 and not dst net 192.168.1.0/24` | Outbound LAN traffic |
| `tcp and dst portrange 1-1023` | Inbound service connections |
| `ether broadcast` | ARP / DHCP monitoring |
| `not port 53 and not arp` | Clean capture — no DNS/ARP noise |
| `tcp port 53` | DNS over TCP — zone transfers / large responses |

---