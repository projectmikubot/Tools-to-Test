# fingerprintx

**Source:** https://github.com/praetorian-inc/fingerprintx  
**Author:** Praetorian  
**Language:** Go

## Description

fingerprintx is a standalone utility for service discovery on open ports. Similar to httpx but for any protocol — it identifies what service is running on an open port by probing with application-layer fingerprints. Supports 51 protocols over TCP and UDP, including databases, remote access, industrial protocols, messaging, and more.

Designed to pair with port scanners like naabu: scan first, fingerprint second.

### Key Features

- Application-layer service fingerprinting (not just port/banner)
- 51 service detection plugins (TCP + UDP)
- Fast mode: only test the default service for a port (80/20 rule)
- JSON, CSV, and plain-text output
- Reads from stdin — pipeline-friendly
- Automatic metadata collection (version, headers, etc.)

### Supported Protocols (51 total)

**Databases:** PostgreSQL, MySQL, MSSQL, OracleDB, DB2, Sybase, Firebird, MongoDB, CouchDB, Cassandra, Redis, Elasticsearch, InfluxDB, Neo4j  
**Vector DBs:** ChromaDB, Milvus, Pinecone  
**Remote Access:** SSH, RDP, Telnet, VNC  
**File Transfer:** FTP, SMB, Rsync  
**Web:** HTTP/HTTPS  
**Mail:** SMTP, IMAP, POP3  
**Industrial:** Modbus, IPMI  
**Messaging:** Kafka, MQTT, SMPP  
**Networking:** DNS, DHCP, NTP, SNMP, NetBIOS-NS, IPSEC, STUN, OpenVPN  
**Dev Tools:** JDWP, Java RMI  
**Telecom:** Diameter, RTSP, SNPP, Echo, Linux RPC  

## Installation

```bash
# Via Go
go install github.com/praetorian-inc/fingerprintx/cmd/fingerprintx@latest

# From source
git clone git@github.com:praetorian-inc/fingerprintx.git
cd fingerprintx
go build ./cmd/fingerprintx

# Docker
git clone git@github.com:praetorian-inc/fingerprintx.git
cd fingerprintx
docker build -t fingerprintx .
```

## Usage

```
fingerprintx [flags]

TARGET SPECIFICATION:
  Requires HOST:PORT or IP:PORT (port assumed open)

  -t, --targets strings   target or comma-separated target list
  -l, --list string        input file containing targets

OUTPUT:
  --json            JSON output
  --csv             CSV output
  -o, --output string  output file

OPTIONS:
  -f, --fast           fast mode (only test default service for port)
  -U, --udp            run UDP plugins
  -w, --timeout int    timeout in milliseconds (default 500)
  -v, --verbose        verbose mode
  -h, --help           help
```

## Common Usage Examples

```bash
# Single target
fingerprintx -t praetorian.com:80

# Multiple targets (comma-separated)
fingerprintx -t praetorian.com:80,127.0.0.1:8000

# JSON output (includes metadata like version, headers)
fingerprintx -t 127.0.0.1:8000 --json

# Read targets from file
fingerprintx -l open_ports.txt

# Cat file into stdin
cat input.txt | fingerprintx

# Fast mode (only probe default service per port — much faster for large lists)
fingerprintx -f -l ports.txt

# Include UDP service detection
fingerprintx -t 192.168.1.1:161 -U

# Custom timeout (milliseconds)
fingerprintx -t 10.0.0.1:22 -w 2000

# Output to file
fingerprintx -l targets.txt --json -o fingerprints.json

# Docker
docker run --rm fingerprintx -t praetorian.com:80 --json
```

## Pipeline Integration

```bash
# naabu → fingerprintx (primary workflow)
naabu 127.0.0.1 -silent 2>/dev/null | fingerprintx

# Scan subnet, fingerprint all discovered ports
naabu -host 192.168.1.0/24 -p 1-65535 -silent | fingerprintx --json

# Pipe into brutus for credential testing
naabu -host 10.0.0.0/24 -silent | fingerprintx --json | brutus --json

# Full discovery chain
subfinder -d example.com -silent | \
  naabu -silent | \
  fingerprintx --json -o services.json
```

## Output Examples

```
# Default output (SERVICE://HOST:PORT)
http://127.0.0.1:8000
ftp://127.0.0.1:21
ssh://10.0.0.1:22

# JSON output
{
  "ip": "127.0.0.1",
  "port": 8000,
  "service": "http",
  "transport": "tcp",
  "metadata": {
    "responseHeaders": { "Server": ["Apache/2.4.41"] },
    "status": "200 OK",
    "statusCode": 200,
    "version": "Apache/2.4.41"
  }
}
```
