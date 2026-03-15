# brutus

**Source:** https://github.com/praetorian-inc/brutus  
**Author:** Praetorian  
**Language:** Go

## Description

Brutus is a fast, zero-dependency multi-protocol credential testing tool written in pure Go. It addresses a critical gap in offensive tooling: efficient credential validation across diverse network services beyond HTTP. A modern Hydra alternative designed for penetration testers and red team operators.

Single binary, no external dependencies, cross-platform (Linux, Windows, macOS). Integrates natively with Nerva and naabu for full reconnaissance-to-credential-test pipelines.

### Key Features

- **Zero dependencies:** Single binary, no libssh-dev, no compilation headaches
- **24 protocols:** SSH, RDP, MySQL, PostgreSQL, MSSQL, Redis, MongoDB, SMB, LDAP, WinRM, SNMP, HTTP Basic Auth, and more
- **Embedded bad keys:** Built-in known SSH private keys (Vagrant, F5 BIG-IP, ExaGrid, etc.) tested automatically
- **Pipeline integration:** Native JSON support for Nerva and naabu workflows
- **Go library:** Importable directly into custom security tooling
- **Production features:** Rate limiting, connection pooling, comprehensive error handling

### Supported Protocols (24+)

SSH, RDP, MySQL, PostgreSQL, MSSQL, Redis, MongoDB, SMB, LDAP, WinRM, SNMP, HTTP Basic Auth, FTP, Telnet, VNC, IMAP, POP3, SMTP, Memcached, CouchDB, Cassandra, Elasticsearch, OracleDB, and more.

## Installation

```bash
# Linux (amd64)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-linux-amd64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/

# macOS (Apple Silicon)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-darwin-arm64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/

# macOS (Intel)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-darwin-amd64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/

# Via Go
go install github.com/praetorian-inc/brutus/cmd/brutus@latest
```

## Usage

```
brutus [flags]

TARGET:
  -t, --target string    single target (host:port or service://host:port)
  -l, --list string      file of targets

CREDENTIALS:
  -u, --usernames string  comma-separated usernames or file path
  -p, --passwords string  comma-separated passwords or file path
  -k, --key string        path to SSH private key

OUTPUT:
  --json    JSON output format

PIPELINE:
  Reads JSON from stdin (Nerva/naabu output)
```

## Common Usage Examples

```bash
# Test SSH credentials against a single host
brutus -t ssh://10.0.0.1:22 -u root,admin,ubuntu -p password,admin123

# Test MySQL with credential list files
brutus -t mysql://10.0.0.1:3306 -u users.txt -p passwords.txt

# Test Redis (often no auth required — brutus will detect)
brutus -t redis://10.0.0.1:6379

# Test SMB with password spray
brutus -t smb://10.0.0.1 -u administrator -p 'Password1,Welcome1,Summer2024!'

# Test SSH with a found private key across common usernames
brutus -t ssh://10.0.0.1:22 -u root,admin,ubuntu,deploy -k /path/to/id_rsa

# JSON output
brutus -t ssh://10.0.0.1:22 -u admin -p admin --json

# Test HTTP Basic Auth
brutus -t http://10.0.0.1:8080 -u admin -p admin,password,letmein
```

## Pipeline Integration

```bash
# Full credential audit pipeline: naabu → nerva → brutus
naabu -host 10.0.0.0/24 -p 22,3306,5432,6379 -silent | \
  nerva --json | \
  brutus --json

# SSH lateral movement — spray a found key across a subnet
naabu -host 10.0.0.0/24 -p 22 -silent | \
  nerva --json | \
  brutus -u root,admin,ubuntu,deploy -k /path/to/found_key --json

# Discover HTTP Basic Auth panels and test default creds
naabu -host 10.0.0.0/24 -p 80,443,3000,8080,9090 -silent | \
  nerva --json | \
  brutus --json

# naabu → fingerprintx → brutus
naabu -host 192.168.1.0/24 -silent | \
  fingerprintx --json | \
  brutus --json
```

## Use Cases

- **Internal assessments:** Validate discovered credentials across multiple services
- **Password reuse testing:** Test credentials across databases and file shares
- **Default credential checks:** Identify defaults on newly deployed infrastructure
- **Post-phish/dump validation:** Rapid credential validation after password dumps
- **Lateral movement discovery:** Spray compromised keys/creds across network ranges
- **Compliance audits:** Generate audit trails for password policy validation

## Why Brutus Over Hydra?

| Feature | Brutus | Hydra |
|---|---|---|
| Dependencies | Zero (single binary) | libssh-dev, libmysqlclient-dev, etc. |
| Deployment | Download & run | Compile from source or package manager |
| Pipeline JSON | Native | Manual scripting required |
| SSH bad keys | Embedded | Manual |
| Platform | Linux/macOS/Windows | Primarily Linux |
| Maintenance | Active (Praetorian) | Aging |
