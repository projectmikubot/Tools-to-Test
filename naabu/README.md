# naabu

**Source:** https://github.com/projectdiscovery/naabu  
**Author:** ProjectDiscovery  
**Language:** Go

## Description

naabu is a fast, reliable port scanner written in Go, designed for attack surface discovery. It performs SYN/CONNECT/UDP scans and integrates naturally into recon pipelines with tools like httpx, nuclei, and fingerprintx. Emphasizes speed, simplicity, and low resource usage.

### Key Features

- Fast SYN/CONNECT/UDP probe-based scanning
- Passive port enumeration via Shodan InternetDB
- IPv4/IPv6 support (IPv6 experimental)
- Host discovery mode
- NMAP integration for service discovery
- Multiple input formats: STDIN, HOST, IP, CIDR, ASN
- Multiple output formats: JSON, TXT, STDOUT
- CDN/WAF exclusion support
- Automatic IP deduplication for DNS scans

## Installation

```bash
# Prerequisites
sudo apt install -y libpcap-dev   # Linux
brew install libpcap               # macOS

# Install via Go
go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest

# Docker
docker pull projectdiscovery/naabu
```

## Usage

```
naabu [flags]

INPUT:
  -host string[]     hosts to scan (comma-separated)
  -l, -list string   file containing list of hosts
  -eh string         hosts to exclude (comma-separated)
  -ef string         file of hosts to exclude

PORT:
  -p, -port string       ports to scan (e.g. 80,443, 100-200)
  -tp, -top-ports string top ports to scan [full,100,1000] (default 100)
  -ep string             ports to exclude (comma-separated)
  -ec, -exclude-cdn      skip full port scans for CDN/WAF (only 80,443)

RATE-LIMIT:
  -c int    worker threads (default 25)
  -rate int packets per second (default 1000)

OUTPUT:
  -o, -output string  write output to file
  -j, -json           JSON lines output
  -csv                CSV output
  -silent             results only

SCAN TYPE:
  -s, -scan-type string  SYN or CONNECT (default "c")

HOST DISCOVERY:
  -sn   host discovery only
  -wn   enable host discovery with port scan
  -pe   ICMP echo ping
  -arp  ARP ping

OPTIMIZATION:
  -retries int    retries per port (default 3)
  -timeout int    timeout in milliseconds (default 1000)
  -verify         re-verify open ports with TCP
```

## Common Usage Examples

```bash
# Scan a single host (top 100 ports)
naabu -host hackerone.com

# Scan specific ports
naabu -host 10.10.10.1 -p 22,80,443,3306,5432

# Scan top 1000 ports
naabu -host 10.10.10.1 -tp 1000

# Scan a CIDR range
naabu -host 192.168.1.0/24 -p 80,443,22

# Read targets from file
naabu -l targets.txt -p 80,443 -o results.txt

# JSON output
naabu -host example.com -json -o ports.json

# SYN scan (requires root)
sudo naabu -host 10.0.0.1 -s SYN -p full

# Exclude CDN targets from full scan
naabu -host example.com -exclude-cdn

# Passive scan via Shodan InternetDB (no packets sent)
naabu -host example.com -passive

# Pipe into fingerprintx for service fingerprinting
naabu -host 192.168.1.0/24 -silent | fingerprintx

# Full recon pipeline with nuclei
naabu -l hosts.txt -silent | httpx -silent | nuclei -t technologies/

# Rate-limited scan
naabu -host 10.0.0.0/16 -rate 500 -c 10 -p 80,443,8080,8443

# Scan with NMAP service detection on found ports
naabu -host example.com -nmap-cli 'nmap -sV -sC'

# Docker
docker run --rm projectdiscovery/naabu -host example.com
```

## Pipeline Integration

naabu pairs naturally with the ProjectDiscovery toolkit:

```bash
# subfinder → naabu → httpx → nuclei
subfinder -d example.com -silent | \
  naabu -silent | \
  httpx -silent | \
  nuclei -t cves/

# naabu → fingerprintx (Praetorian)
naabu 192.168.1.1 -silent 2>/dev/null | fingerprintx

# naabu → brutus (credential testing)
naabu -host 10.0.0.0/24 -p 22,3306,5432,6379 -silent | \
  nerva --json | \
  brutus --json
```
