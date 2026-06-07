# 🛡️ NetWatch — Network Packet Analyzer & Traffic Monitor

> A Python-based CLI tool for real-time network packet capture, protocol dissection, traffic analysis, and basic intrusion detection.

---

## 📸 What It Does

NetWatch captures raw network packets directly from the wire, dissects them at the Ethernet → IP → Transport layers, and gives you a clean, color-coded live feed in your terminal. It also silently watches for threat patterns (SYN floods, ICMP floods, suspicious ports) and alerts you inline.

```
 ███╗   ██╗███████╗████████╗██╗    ██╗  █████╗ ████████╗ ██████╗██╗  ██╗
 ...
────────────────────────────────────────────────────────────────────────
  TIME          PROTO  SRC                →   DST                  SIZE  INFO
────────────────────────────────────────────────────────────────────────
12:34:01.221  TCP    192.168.1.5        →  142.250.1.1         1420 B  54321→443(HTTPS) [SYN]
12:34:01.310  UDP    192.168.1.5        →  8.8.8.8              72 B   43210→53(DNS)
12:34:01.560  ICMP   10.0.0.2           →  192.168.1.5          84 B   Echo Request  [!ALERT]
```

---

## ⚙️ Features

| Feature | Description |
|---|---|
| **Live Packet Capture** | Raw socket sniffer at Layer 2 |
| **Protocol Dissection** | Ethernet → IPv4 → TCP / UDP / ICMP |
| **Service Recognition** | 20+ known ports (DNS, HTTP, SSH, RDP…) |
| **Threat Detection** | SYN flood, ICMP flood, suspicious ports, Telnet |
| **Flexible Filters** | Protocol, host IP, port number |
| **Session Statistics** | Packets, bytes, rates, top talkers, top ports |
| **CSV Logging** | Export captures for later analysis |
| **Color-coded Output** | Protocol-specific colors, red alerts |
| **Quiet Mode** | Silent capture + periodic stats only |

---

## 🚀 Quick Start

### Requirements

- **Python 3.7+** (no third-party dependencies — stdlib only)
- **Linux** (uses `AF_PACKET` raw sockets)
- **Root / sudo** privileges (required for raw socket access)

### Installation

```bash
git clone https://github.com/YOUR_USERNAME/netwatch.git
cd netwatch
chmod +x netwatch.py
```

No `pip install` needed — NetWatch uses only Python's standard library.

---

## 📖 Usage

```bash
sudo python3 netwatch.py [OPTIONS]
```

### Options

| Flag | Description |
|---|---|
| `-i, --iface IFACE` | Network interface to listen on (e.g. `eth0`, `wlan0`) |
| `-p, --proto PROTO` | Filter by protocol: `tcp`, `udp`, `icmp` |
| `--host IP` | Filter: only show packets to/from this IP |
| `--port PORT` | Filter: only show packets on this port |
| `-c, --count N` | Stop capturing after N packets |
| `-l, --log FILE` | Save capture to a CSV file |
| `--stats N` | Print session stats every N seconds |
| `-q, --quiet` | Suppress per-packet output (stats only) |
| `-v, --version` | Show version |

---

## 💡 Examples

```bash
# Capture everything on all interfaces
sudo python3 netwatch.py

# Monitor a specific interface
sudo python3 netwatch.py -i eth0

# Watch only TCP traffic
sudo python3 netwatch.py -p tcp

# Monitor a specific host
sudo python3 netwatch.py --host 192.168.1.100

# Watch HTTPS traffic only
sudo python3 netwatch.py --port 443

# Capture 200 packets and save to file
sudo python3 netwatch.py -c 200 -l capture.csv

# Silent mode with stats every 10 seconds (good for long runs)
sudo python3 netwatch.py -q --stats 10

# Combined: TCP from a specific host, log it, stop after 500 packets
sudo python3 netwatch.py -p tcp --host 10.0.0.1 -c 500 -l suspicious.csv
```

---

## 🔍 How It Works

### Packet Capture Flow

```
NIC (kernel)
    ↓
AF_PACKET raw socket (Layer 2)
    ↓
Ethernet Frame Parser
    ↓  (filters non-IPv4)
IPv4 Header Parser
    ↓
Protocol Dispatcher
   ├── TCP  → Ports, Flags (SYN/ACK/RST…)
   ├── UDP  → Ports, Length
   └── ICMP → Type/Code (Echo, Unreachable…)
    ↓
Filter Engine (proto / host / port)
    ↓
Threat Detector (sliding window)
    ↓
Display + Logger + Stats
```

### Threat Detection Rules

| Threat | Detection Logic |
|---|---|
| **SYN Flood** | >50 SYN (no ACK) packets from same IP in 5s window |
| **ICMP Flood** | >30 ICMP packets from same IP in 5s window |
| **Suspicious Port** | Destination port in known exploit/backdoor set |
| **Telnet** | Any connection to port 23 (cleartext protocol) |

Detected threats are flagged inline with `[!ALERT]` and summarized in the session stats.

---

## 📊 Session Statistics

Press `Ctrl+C` at any time (or use `--stats N`) to see:

- Total packets captured and bytes transferred
- Packets-per-second and bits-per-second throughput
- Per-protocol breakdown with visual bar chart
- Top 5 source IPs (talkers)
- Top 5 ports by traffic volume
- All alerts triggered during the session

---

## 📁 CSV Log Format

When logging with `-l file.csv`, each row contains:

```
timestamp, proto, src_ip, src_port, dst_ip, dst_port, size, flags
```

Example:
```
2025-09-01T12:34:01.221,TCP,192.168.1.5,54321,142.250.1.1,443,1420,SYN
2025-09-01T12:34:01.310,UDP,192.168.1.5,43210,8.8.8.8,53,72,
```

CSV output can be analyzed with Python, Excel, Wireshark (import), or any SIEM.

---

## 🧠 Architecture

```
netwatch.py
├── Color          — ANSI color constants
├── PacketParser   — Stateless header parsers (Ethernet, IP, TCP, UDP, ICMP)
├── Stats          — Thread-safe counters and alert log
├── ThreatDetector — Sliding-window anomaly detection
├── NetWatch       — Main sniffer loop, filter engine, display, logging
└── CLI (argparse) — Argument parsing and entry point
```

---

## 🔐 Security Notes

- NetWatch **requires root** because raw socket access (`AF_PACKET`) is a privileged operation. This is standard for any packet sniffer.
- Only use on networks you own or have explicit authorization to monitor. Unauthorized packet capture may violate laws in your jurisdiction.
- Captured CSV files may contain sensitive data — store and handle them appropriately.

---

## 🗺️ Roadmap / Possible Extensions

- [ ] IPv6 support
- [ ] ARP spoofing detection
- [ ] Port scan detection (horizontal scanning)
- [ ] DNS query/response extraction
- [ ] HTTP request extraction from TCP payload
- [ ] GeoIP lookup for source IPs
- [ ] JSON output mode
- [ ] Curses-based live dashboard (ncurses TUI)
- [ ] pcap file export (Wireshark-compatible)

---

## 👤 Author

**Aman** — B.Tech CSE (Cybersecurity)  
Built as a portfolio project demonstrating: raw socket programming, protocol dissection, real-time anomaly detection, and CLI tool design.

---

## 📜 License

MIT License — free to use, modify, and distribute with attribution.
