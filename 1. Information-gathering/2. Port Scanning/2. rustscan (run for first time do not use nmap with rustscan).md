### THIS TOOL finds initial ports, manually add them to an nmap scan. because in my testing it broke nmap results

```
# -n flag skips nmap
rustscan -a ip/range -n 

# try increase batch to 1000 -b 1000
```
2. take ports and put to nmap
```
nmap ip -p <rustscan found>
```

#  RustScan Cheat Sheet

> **RustScan**: A lightning-fast port scanner (written in Rust) that can integrate with Nmap.

---

##  Summary Table

| Option | Description          |
| ------ | -------------------- |
| `-a`   | Target IP/hostname   |
| `-p`   | Ports to scan        |
| `-r`   | Port range           |
| `-t`   | Timeout (ms)         |
| `-b`   | Batch size           |
| `-u`   | Threads              |
| `--`   | Pass options to Nmap |
| `-n`   | Skip Nmap            |
| `-g`   | Greppable output     |

---

## Basic Usage

```bash
rustscan -a <target>
```
- Scans top 5000 TCP ports on the target.

```bash
rustscan -a 192.168.1.1
```

---

##  Scan Specific Port(s)

```bash
rustscan -a <target> -p <port(s)>
```

Examples:
```bash
rustscan -a 10.10.10.10 -p 22
rustscan -a 10.10.10.10 -p 21,22,80,443
rustscan -a 10.10.10.10 -p 1-1000
```

---
## supply nmap script arguments
`nmap --script <scriptname> --script-args key1=value1,key2=value2`
## Use with Nmap (Most Common Usage)

```bash
rustscan -a <target> -- -A -sC -sV
```

- The `--` passes options directly to `nmap`.
- `-A` enables OS detection + version + script scanning.
- `-sC -sV` enables default scripts and version detection.

Example:
```bash
rustscan -a 192.168.1.100 -- -sC -sV -oN scan.txt
```

---

##  Speed & Concurrency Options

```bash
rustscan -a <target> -b 1500 -t 2000
```

- `-b`: Batch size (number of ports to scan at a time)
- `-t`: Timeout (in milliseconds)

Example:
```bash
rustscan -a 10.0.0.5 -b 3000 -t 1500
```

---

##  Scan All Ports

```bash
rustscan -a <target> -p 1-65535
```

> RustScan is optimized for full port sweeps.

---

##  Output to File

```bash
rustscan -a <target> -- -sC -sV -oN results.txt
```

Nmap-style output saved to `results.txt`.

---

##  Extra Tricks

### Skip nmap output, just fast port scanning:
```bash
rustscan -a 192.168.1.1 -r 1-65535 -n
```

- `-n`: No Nmap, just list open ports.

### Specify number of threads:
```bash
rustscan -a <target> -u 5000
```
- `-u`: Number of threads (default: 5000)

---

##  Bypass IDS/Rate Limit (Slow Scan)

```bash
rustscan -a <target> -t 5000 -b 100 -- -sC -sV
```

- Increases timeout & lowers batch size to avoid detection.

---

##  Scan Multiple Hosts

```bash
rustscan -a 10.10.10.10,10.10.10.11,10.10.10.12
```

Or use a file:
```bash
rustscan -f targets.txt -- -sC -sV
```

---

##  Common Scan Recipes

### 🔹 Fast + Nmap default scripts:
```bash
rustscan -a 10.10.10.10 -- -sC -sV
```

### 🔹 Full port scan with output:
```bash
rustscan -a 10.10.10.10 -p 1-65535 -- -sC -sV -oA full-scan
```

### 🔹 Stealthier scan:
```bash
rustscan -a target -t 3000 -b 100 -- -sS -Pn
```

---
```bash
rustscan -a 10.10.10.10 -r 1-1000 -g | xargs -I {} echo "Port {} open!"
```

- `-g`: greppable output (just the ports)

---
