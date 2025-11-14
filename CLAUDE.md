# CLAUDE.md - AI Assistant Guide for architecture-networking

## Project Overview

This repository contains a network architecture assignment for **FIT9137** (Introduction to Computer Networks and Security). The project implements a comprehensive enterprise network topology using **IMUNES/CORE network emulator**, featuring:

- Multi-router topology with static routing
- DHCP server configuration
- DNS and web services
- Mail server configuration
- Firewall implementation with iptables
- DMZ (Demilitarized Zone) setup
- Internal and external network segmentation

## Repository Structure

```
architecture-networking/
├── FIT9137.imn           # IMUNES network topology configuration (2705 lines)
├── RFP Report.pdf        # Project documentation and requirements
├── README.md             # Basic project description
└── CLAUDE.md            # This file - AI assistant guide
```

### File Descriptions

| File | Purpose | Format |
|------|---------|--------|
| `FIT9137.imn` | Complete network topology definition with all node configurations, routing tables, firewall rules, and service configurations | IMUNES/CORE `.imn` format (TCL-based) |
| `RFP Report.pdf` | Project requirements, network design documentation, and implementation report | PDF |
| `README.md` | High-level project description | Markdown |

## Network Topology Overview

### Node Types and Count

The network consists of **27+ nodes** organized into:

- **20 Routers** (including specialized service routers)
- **7 LAN Switches** (sw1-sw7)
- Multiple client and server machines

### Key Network Nodes

#### Core Routers
- **R1** (94.78.71.1, 94.78.37.1, 94.78.231.1, 94.78.122.1) - Main gateway router with DHCP and firewall
- **R2** (94.78.122.2) - Secondary router
- **R3** (94.78.37.2) - Tertiary router
- **R4** - Fourth core router

#### Service Nodes
- **dns** (94.78.215.10) - DNS server in DMZ
- **web** (94.78.215.11) - Web server in DMZ
- **mail** (94.78.215.12) - Mail server in DMZ (SMTP on port 25)
- **ssh** - SSH access server
- **intranet** - Internal web services
- **localweb** - Local web server
- **Internet** - External internet gateway

#### Client Nodes
- **client1**, **client2** - Internal clients
- **extClient1**, **extClient2** - External clients

#### Mythological Named Servers
- **minerva** - Application server
- **clio** - Application server
- **apollo** - Application server
- **artemis** - Application server
- **demeter** - Application server
- **leto** - Application server

#### LAN Switches
- **sw1** through **sw7** - Network segmentation switches

## IP Address Scheme

### Primary Subnets

| Subnet | Netmask | Purpose | DHCP Range |
|--------|---------|---------|------------|
| 94.78.71.0/24 | 255.255.255.0 | Internal network | 94.78.71.127 - 94.78.71.254 |
| 94.78.215.0/24 | 255.255.255.0 | DMZ (DNS, Web, Mail servers) | - |
| 94.78.37.0/24 | 255.255.255.0 | Routing segment | - |
| 94.78.122.0/24 | 255.255.255.0 | Routing segment | - |
| 94.78.231.0/24 | 255.255.255.0 | Routing segment | - |
| 94.78.182.0/24 | 255.255.255.0 | Internal network segment | - |
| 94.78.5.0/24 | 255.255.255.0 | Internal network segment | - |
| 60.98.165.0/24 | 255.255.255.0 | External network | - |
| 43.131.67.0/24 | 255.255.255.0 | External network | - |
| 43.131.110.0/24 | 255.255.255.0 | External network with DHCP | 43.131.110.127 - 43.131.110.254 |

## Network Services Configuration

### 1. DHCP Service (on R1)

**Location**: R1 router
**Configuration**: `/etc/dhcp/dhcpd.conf`

**Key Parameters**:
- Default lease time: 36000 seconds (10 hours)
- Max lease time: 72000 seconds (20 hours)
- DNS server: 94.78.215.10
- Domain name: "talos.edu"

**DHCP Pools**:
1. Subnet 94.78.71.0/24: Range 94.78.71.127-254, Gateway 94.78.71.1
2. Subnet 43.131.110.0/24: Range 43.131.110.127-254

### 2. Static Routing

**On R1**, routes are configured to:
- Route DMZ traffic (94.78.215.0/24) via 94.78.122.2
- Route internal networks (94.78.182.0/24, 94.78.5.0/24) via 94.78.122.2
- Route external networks (60.98.165.0/24, 43.131.67.0/24, 43.131.110.0/24) via 94.78.122.2
- Default route via 94.78.37.1

### 3. Firewall Configuration (on R3)

**Default Policy**: DROP all traffic (INPUT, OUTPUT, FORWARD)

**Allowed Traffic**:

#### DMZ Services (Inbound)
- DNS queries (UDP port 53) to 94.78.215.10
- HTTP (TCP port 80) to 94.78.215.11
- SMTP (TCP port 25) to 94.78.215.12

#### DMZ Services (Outbound)
- NEW connections from DNS server (94.78.215.10)
- NEW connections from web server (94.78.215.11)
- NEW connections from mail server (94.78.215.12)

#### Internal to DMZ Communication
- Bidirectional traffic between internal networks (94.78.71.0/24, 94.78.182.0/24, 94.78.5.0/24) and DMZ (94.78.215.0/24)

#### Inter-router Communication
- Full access between eth1 and eth2 interfaces
- Stateful firewall between eth1/eth2 and eth3 (NEW outbound, ESTABLISHED/RELATED inbound)

#### Management Access
- SSH (TCP port 22) from internal network 94.78.71.0/24
- ICMP (ping) within the 94.78.0.0/16 supernet

### 4. Mail Server Configuration

**Postfix Configuration**:
- Hostname: "mail"
- Uses standard Postfix service with custom hostname setting
- Accessible on TCP port 25 through firewall

## File Format: IMUNES .imn

The `.imn` file format is a TCL-based configuration format used by IMUNES/CORE network emulator.

### Structure

Each node is defined with:
```tcl
node nX {
    type [router|lanswitch|pc]
    model [static|quagga|...]
    network-config {
        hostname NAME
        !
        interface ethY
         ip address X.X.X.X/YY
        !
    }
    canvas c1
    iconcoords {X Y}
    labelcoords {X Y}
    interface-peer {ethX nY}
    custom-config {
        # Service configurations (DHCP, Firewall, Routing, etc.)
    }
    services {DefaultRoute IPForward DHCP ...}
}
```

### Key Sections

1. **Node Definition**: Basic node properties (type, model, hostname)
2. **Network Config**: Interface IP addresses and network settings
3. **Interface Peers**: Connections between nodes
4. **Custom Configs**: Service-specific configurations (DHCP, firewall rules, static routes)
5. **Services**: List of enabled services on the node

## Development Workflows

### Working with the Network Topology

1. **Viewing/Editing Topology**:
   - Use IMUNES/CORE GUI to visualize: `imunes FIT9137.imn`
   - Text editing requires understanding of `.imn` TCL format
   - Always backup before making changes: `cp FIT9137.imn FIT9137.imn.backup`

2. **Starting the Network Simulation**:
   ```bash
   imunes -b FIT9137.imn          # Start in background
   imunes -e experiment_id        # Execute with experiment ID
   ```

3. **Accessing Node Consoles**:
   ```bash
   # Once simulation is running
   himage node_name command       # Execute command on node
   ```

4. **Testing Connectivity**:
   - Test DHCP: Check if clients receive IP addresses
   - Test DNS: Query the DNS server (94.78.215.10)
   - Test HTTP: Access web server (94.78.215.11:80)
   - Test SMTP: Connect to mail server (94.78.215.12:25)
   - Test Firewall: Verify blocked/allowed traffic patterns

5. **Stopping the Simulation**:
   ```bash
   imunes -c experiment_id        # Clean up experiment
   ```

### Modifying Firewall Rules

Firewall rules are located in R3's custom-config section (around line 242):

```bash
# Pattern: iptables -A CHAIN -options -j ACTION
iptables -A FORWARD -p tcp --dport 80 -d 94.78.215.11 -j ACCEPT
```

**When adding new rules**:
1. Identify the correct chain (INPUT, OUTPUT, FORWARD)
2. Specify source/destination as needed
3. Define protocol and ports
4. Set action (ACCEPT, DROP, REJECT)
5. Remember: Default policy is DROP, so explicit ACCEPT rules are required

### Modifying DHCP Configuration

DHCP config is in R1's custom-config section (around line 82):

```bash
subnet X.X.X.0 netmask 255.255.255.0 {
  pool {
    range X.X.X.START X.X.X.END;
    default-lease-time 36000;
    option routers X.X.X.GATEWAY;
    option domain-name-servers X.X.X.DNS;
    option domain-name "domain.name";
  }
}
```

### Adding Static Routes

Static routes are in the StaticRoute service section:

```bash
/sbin/ip route add DESTINATION/MASK via GATEWAY
```

## Key Conventions for AI Assistants

### When Working with This Repository

1. **File Handling**:
   - The `.imn` file is the primary configuration - treat it with care
   - Always read the relevant section before making changes
   - The file is 2705 lines - use grep/search to find specific sections
   - Validate TCL syntax after modifications

2. **Network Changes**:
   - Understand the impact on routing before modifying IP addresses
   - Firewall changes require testing both allowed and blocked traffic
   - DHCP changes affect client connectivity - update range carefully
   - Document all network changes in commit messages

3. **IP Address Management**:
   - Primary internal network: 94.78.71.0/24 (with DHCP)
   - DMZ network: 94.78.215.0/24 (static IPs)
   - External networks: 60.98.165.0/24, 43.131.67.0/24, 43.131.110.0/24
   - Never duplicate IP addresses across nodes
   - Update both interface configs and routing tables when changing IPs

4. **Service Configuration**:
   - Services are defined in custom-config sections
   - Each service has a declaration and a file/script section
   - Common services: DHCP, DNS, Firewall, StaticRoute, DefaultRoute, IPForward
   - Service order matters for dependencies

5. **Security Considerations**:
   - The firewall implements a default-deny policy
   - DMZ servers (dns, web, mail) have restricted outbound access
   - SSH access is limited to internal network (94.78.71.0/24)
   - Stateful inspection is used for internet-facing interfaces

6. **Testing Requirements**:
   - Any topology changes should be tested in IMUNES/CORE
   - Verify routing tables after route modifications
   - Test firewall rules for both allow and deny cases
   - Confirm DHCP leases are issued correctly
   - Check DNS resolution and web/mail service accessibility

7. **Documentation**:
   - Update this CLAUDE.md when making structural changes
   - Document complex firewall rules with inline comments
   - Keep the RFP Report.pdf in sync with implementation
   - Use clear commit messages describing network changes

8. **Common Tasks**:

   **Adding a new server to DMZ**:
   1. Add node definition with type router
   2. Configure interface IP in 94.78.215.0/24 range
   3. Add static route on R1 to DMZ via 94.78.122.2
   4. Update firewall rules on R3 to allow required ports
   5. Connect node to appropriate switch

   **Adding a new internal subnet**:
   1. Define subnet addressing scheme
   2. Configure router interface for the subnet
   3. Add static routes on other routers
   4. Optionally configure DHCP on R1
   5. Update firewall rules if DMZ access needed

   **Modifying firewall policy**:
   1. Locate R3 firewall custom-config section (line ~242)
   2. Add new iptables rules in appropriate chain
   3. Consider both directions of traffic
   4. Remember stateful rules (NEW, ESTABLISHED, RELATED)
   5. Test with simulation

9. **Troubleshooting**:
   - If routing fails: Check static routes and default routes
   - If DHCP fails: Verify DHCP server config and network connectivity
   - If firewall blocks: Check iptables rules and chain policies
   - If service unreachable: Verify service is running and firewall allows traffic
   - Use `himage` to execute diagnostic commands on nodes during simulation

10. **Version Control**:
    - Commit .imn changes with descriptive messages
    - Include affected nodes/services in commit message
    - Example: "Add HTTPS firewall rule for web server (94.78.215.11)"
    - Keep binary PDF file changes separate from .imn changes

## Academic Context

This is a **FIT9137** university assignment focused on:
- Network architecture design
- Routing protocol implementation (static routing)
- DHCP server configuration
- Firewall security policies
- DMZ design for public-facing services
- Network segmentation and access control

**Learning Objectives**:
- Understand enterprise network topology design
- Configure routing between multiple subnets
- Implement security through firewall rules
- Deploy network services (DNS, web, mail)
- Practice with network emulation tools

## Quick Reference

### Critical IP Addresses
- R1 Gateway: 94.78.71.1 (internal), 94.78.37.1, 94.78.231.1, 94.78.122.1
- DNS Server: 94.78.215.10
- Web Server: 94.78.215.11
- Mail Server: 94.78.215.12

### Critical Ports
- DNS: UDP 53
- HTTP: TCP 80
- SMTP: TCP 25
- SSH: TCP 22

### DHCP Ranges
- Internal: 94.78.71.127-254
- External: 43.131.110.127-254

### Domain
- talos.edu

## Tools and Dependencies

**Required**:
- IMUNES or CORE network emulator
- Linux environment (for simulation)
- iptables (firewall)
- ISC DHCP server
- Postfix (mail server)

**Optional**:
- Wireshark (packet analysis)
- tcpdump (traffic monitoring)
- Network testing tools (ping, traceroute, nmap)

---

**Last Updated**: 2025-11-14
**Topology Version**: FIT9137.imn (2705 lines, 27 nodes)
**Assignment**: FIT9137 - Computer Networks and Security
