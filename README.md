# Enterprise Network Architecture - FIT9137

A comprehensive enterprise network topology implementation for **FIT9137** (Introduction to Computer Networks and Security) using IMUNES/CORE network emulator.

## Overview

This project implements a multi-tier enterprise network featuring:

- **27+ Network Nodes** - Routers, switches, servers, and clients
- **Static Routing** - Multi-hop routing across 9 subnets
- **DHCP Services** - Automated IP address assignment for internal and external clients
- **DMZ Architecture** - Isolated zone for public-facing services
- **Firewall Security** - iptables-based default-deny policy with granular access control
- **Network Services** - DNS, Web (HTTP), and Mail (SMTP) servers

## Network Architecture

### Core Components

| Component | Count | Description |
|-----------|-------|-------------|
| Routers | 20 | Core routing infrastructure (R1-R4) plus service routers |
| Switches | 7 | LAN segmentation (sw1-sw7) |
| Servers | 10+ | DNS, Web, Mail, SSH, Intranet, and application servers |
| Clients | 4 | Internal (client1, client2) and external (extClient1, extClient2) |

### IP Address Scheme

The network uses the 94.78.0.0/16 address space with the following key subnets:

- **94.78.71.0/24** - Internal network with DHCP (pool: .127-.254)
- **94.78.215.0/24** - DMZ for public services (DNS, Web, Mail)
- **94.78.182.0/24** - Internal application servers
- **94.78.5.0/24** - Internal services
- **43.131.110.0/24** - External network with DHCP

### Key Services

| Service | IP Address | Port | Location |
|---------|------------|------|----------|
| DNS | 94.78.215.10 | UDP 53 | DMZ |
| Web Server | 94.78.215.11 | TCP 80 | DMZ |
| Mail Server | 94.78.215.12 | TCP 25 | DMZ |
| DHCP Server | 94.78.71.1 | - | R1 Router |

## Features

### 1. DHCP Configuration
- **2 DHCP pools** for internal and external networks
- **10-hour default lease** time (36000 seconds)
- **20-hour maximum lease** time (72000 seconds)
- Domain: `talos.edu`
- DNS server: `94.78.215.10`

### 2. Firewall Security (R3)
- **Default-deny policy** on all chains (INPUT, OUTPUT, FORWARD)
- **DMZ protection** - Only essential ports (53, 80, 25) accessible from outside
- **Stateful inspection** - ESTABLISHED and RELATED connections tracked
- **Internal network isolation** - Controlled access between internal networks and DMZ
- **SSH access** - Limited to internal network (94.78.71.0/24)

### 3. Static Routing
- Multi-hop routing across all subnets
- Default gateway configuration
- Routing tables optimized for DMZ and internal network separation

## Files

| File | Size | Description |
|------|------|-------------|
| `FIT9137.imn` | 2705 lines | Complete network topology in IMUNES format |
| `RFP Report.pdf` | 482 KB | Project documentation and requirements |
| `CLAUDE.md` | - | AI assistant guide for working with this codebase |

## Requirements

- **IMUNES** or **CORE** network emulator
- **Linux** environment (Ubuntu/Debian recommended)
- **iptables** for firewall functionality
- **ISC DHCP Server** for DHCP services
- **Postfix** for mail server functionality

## Getting Started

### Running the Simulation

```bash
# Start the network simulation
imunes -b FIT9137.imn

# Or with explicit experiment ID
imunes -e <experiment_id>
```

### Accessing Node Consoles

```bash
# Execute commands on specific nodes
himage R1 ifconfig
himage client1 ping 94.78.215.10
himage web netstat -tlnp
```

### Testing Services

```bash
# Test DHCP (from a client node)
himage client1 dhclient eth0

# Test DNS resolution
himage client1 nslookup www.example.com 94.78.215.10

# Test web server
himage client1 curl http://94.78.215.11

# Test mail server
himage client1 telnet 94.78.215.12 25
```

### Stopping the Simulation

```bash
# Clean up the experiment
imunes -c <experiment_id>
```

## Network Topology Highlights

- **R1** acts as the main gateway with DHCP server and default routing
- **R2** provides secondary routing functionality
- **R3** implements the firewall protecting the DMZ and internal networks
- **R4** handles additional routing segments
- **DMZ subnet (94.78.215.0/24)** hosts all public-facing services
- **Internal subnets** are protected behind the firewall with controlled access

## Security Design

The network implements a **layered security approach**:

1. **Perimeter Defense** - Firewall (R3) with default-deny policy
2. **Network Segmentation** - DMZ separated from internal networks
3. **Service Isolation** - Each service (DNS, Web, Mail) on dedicated nodes
4. **Access Control** - Granular firewall rules for each service
5. **Stateful Inspection** - Connection tracking for return traffic

## Academic Context

This project is part of **FIT9137** coursework, demonstrating:

- Enterprise network design principles
- Static routing configuration
- DHCP service deployment
- Firewall security implementation
- DMZ architecture for public services
- Network service integration (DNS, Web, Mail)

## Documentation

For detailed technical documentation including:
- Complete node inventory and IP addressing
- Service configuration details
- Firewall rules breakdown
- Development workflows
- AI assistant conventions

See **[CLAUDE.md](CLAUDE.md)** for comprehensive technical documentation.

## License

This is an academic project for FIT9137 - Introduction to Computer Networks and Security.

---

**Assignment**: FIT9137 Network Architecture
**Nodes**: 27+
**Subnets**: 9
**Services**: DHCP, DNS, Web, Mail, SSH
**Security**: Firewall with DMZ
