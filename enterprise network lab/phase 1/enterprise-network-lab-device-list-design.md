# Enterprise Network Lab — Device List Design

## 1. Purpose of This Document

This document explains how the device list for the Enterprise Network Lab was derived from the project requirements.

The goal is not simply to list devices such as routers, switches, firewalls, and servers. The goal is to show the reasoning process used to move from:

```text
Business Requirement
        ↓
Technical Requirement
        ↓
Required Network Function
        ↓
Device Category
```

This makes the design easier to justify, document, troubleshoot, and expand later.

---

# 2. Design Principle

The device list should never start from:

> "Which devices do I want to use?"

Instead, it should start from:

> "What does the company need the network to do?"

Each device must exist because it satisfies one or more project requirements.

For example:

```text
Requirement:
Employees need Internet access
        ↓
Technical Need:
Traffic must be routed toward an ISP
        ↓
Network Function:
Edge routing
        ↓
Device:
Edge Router
```

Another example:

```text
Requirement:
Guest users must not access internal resources
        ↓
Technical Need:
Traffic must be inspected and filtered
        ↓
Network Function:
Security policy enforcement
        ↓
Device:
Firewall
```

---

# 3. Requirements Used to Build the Device List

The main requirements that affect device selection are:

- 42 employees
- Four departments:
  - HR
  - Finance
  - Sales
  - IT
- Separate internal network segments
- DNS Server
- Web Server
- Future File Server
- Corporate Wi-Fi
- Guest Wi-Fi
- Guest isolation
- Internet connectivity
- Dedicated firewall
- Remote Access VPN for Finance and IT
- Management access restricted to IT
- SNMP monitoring
- Syslog
- Two access switches
- Failure testing
- Future branch office
- Future high availability
- Future dual ISP connectivity

---

# 4. Initial Device List

The initial logical device list is:

| Device Category | Quantity | Main Purpose |
|---|---:|---|
| Edge Router | 1 | Connect the enterprise network to the ISP |
| Firewall | 1 | Security policy enforcement, Internet filtering, VPN |
| Core Switch | 1 | Central LAN aggregation |
| Access Switch | 2 | Connect employee endpoints and access devices |
| Wireless Access Point | 2 | Corporate and Guest Wi-Fi |
| DNS Server | 1 | Internal DNS service |
| Web Server | 1 | Internal web service |
| User PCs / Laptops | Representative devices | Simulate company users |
| ISP Device / Network | 1 | Simulate the Internet provider |
| External Internet Server | 1 | Test Internet connectivity and firewall policies |

The File Server, second router, second firewall, additional switches, and additional access points are planned as future extensions.

---

# 5. Edge Router

## Requirement That Creates the Need

The company requires Internet access for:

- HR
- Finance
- Sales
- IT
- Guest Wi-Fi

The corporate network therefore needs a connection to an Internet Service Provider.

## Technical Requirement

The network needs a device capable of routing traffic between:

```text
Enterprise Network
        ↕
       ISP
```

## Device Selected

**1 × Edge Router**

The device is called an edge router because it sits at the edge of the enterprise network.

Example position:

```text
Corporate Network
       |
    Firewall
       |
   Edge Router
       |
      ISP
       |
   Internet
```

## Why Only One Initially?

The initial project does not require full high availability.

Therefore:

```text
Initial Design:
1 × Edge Router
```

Future expansion:

```text
Edge Router 1
Edge Router 2
```

The second router will later allow us to study redundancy, failover, and dual ISP connectivity.

## Capabilities to Evaluate Later

Before selecting the exact router model, we must check whether it supports:

- IPv4 routing
- Static routes
- Default routes
- Dynamic routing for future expansion
- Sufficient interfaces
- Connection to the firewall
- Connection to the simulated ISP
- Future OSPF support
- Future branch-office connectivity
- SNMP
- Syslog

---

# 6. Firewall

## Requirements That Create the Need

The project requires:

- Guest isolation
- Internet security
- Traffic filtering
- Remote VPN access
- Restrictions between internal networks
- Protection of internal resources from Internet-originated traffic

Examples:

```text
Guest → Internet        ALLOW
Guest → Internal        DENY

Internet → Internal     DENY

Finance VPN             ALLOW
IT VPN                  ALLOW
```

## Technical Requirement

Traffic must be inspected before it is allowed to move between trusted and untrusted networks.

## Device Selected

**1 × Firewall**

Example:

```text
Internal Network
       |
    Firewall
       |
   Edge Router
       |
     Internet
```

## Main Roles

The firewall will eventually be responsible for:

- Security rules
- Internet access policies
- Guest network isolation
- Remote Access VPN
- Filtering between security zones
- Traffic logging
- Possibly NAT, depending on the final design

## Why a Dedicated Firewall?

The project specifically requires a separate firewall instead of using only router ACLs.

This makes the architecture more realistic and allows the lab to demonstrate both networking and basic network-security concepts.

## Capabilities to Evaluate Later

- Stateful firewalling
- Access-control rules
- VPN support
- NAT/PAT
- Multiple interfaces or zones
- Logging
- Management access
- SNMP / monitoring
- Future high-availability support

---

# 7. Core Switch

## Requirements That Create the Need

The company contains:

- HR
- Finance
- Sales
- IT
- Servers
- Corporate Wi-Fi
- Guest Wi-Fi
- Two access switches

All of these network segments must connect through a central LAN infrastructure.

## Technical Requirement

The design needs a central aggregation point for the internal network.

## Device Selected

**1 × Core Switch**

Example:

```text
                 Core Switch
                 /         \
          Access SW1     Access SW2
```

The core switch becomes the central point connecting:

- Access switches
- Server network
- Firewall uplink
- Possibly wireless infrastructure
- Future network devices

## Why Not Connect Everything Directly to the Firewall?

The firewall is primarily a security device.

The internal LAN should have its own switching architecture.

A cleaner design is:

```text
User
 ↓
Access Switch
 ↓
Core Switch
 ↓
Firewall
 ↓
Internet
```

rather than:

```text
Every device
    ↓
Firewall
```

## Capabilities to Evaluate Later

- VLAN support
- 802.1Q trunking
- Sufficient uplink interfaces
- Management interface
- SNMP
- Syslog
- Possibly Layer 3 switching
- Possibly inter-VLAN routing
- Future redundancy support

An important design decision later will be whether inter-VLAN routing happens on:

- the Core Switch,
- the Firewall,
- or another Layer 3 device.

---

# 8. Access Switches

## Requirements That Create the Need

The company has 42 users:

| Department | Users |
|---|---:|
| HR | 8 |
| Finance | 8 |
| Sales | 20 |
| IT | 6 |

Employee devices need wired connectivity.

The project also requires two access switches as part of the initial reliability and topology design.

## Technical Requirement

We need Layer 2 access connectivity for endpoint devices.

## Devices Selected

**2 × Access Switches**

Example:

```text
              Core Switch
              /         \
        Access SW1    Access SW2
         /     \       /     \
       HR    Finance  Sales    IT
```

## Why Two?

Using two access switches:

- creates a more realistic enterprise topology
- allows multiple VLANs across multiple switches
- allows trunking exercises
- allows uplink configuration
- allows failure testing
- avoids placing every endpoint on a single access device

## Why Not 42 Virtual PCs?

The company has 42 real users in the design, but the lab only needs representative endpoints.

Example lab representation:

```text
HR       → 2 PCs
Finance  → 2 PCs
Sales    → 2 PCs
IT       → 2 PCs
```

The addressing and switching design will still be sized for all 42 users and future growth.

## Capabilities to Evaluate Later

- Number of access ports
- VLAN support
- 802.1Q trunking
- STP support
- Management IP
- SNMP
- Syslog
- Port security
- PoE support if needed for APs
- Gigabit uplinks

---

# 9. Wireless Access Points

## Requirements That Create the Need

The company requires:

- Corporate Wi-Fi
- Guest Wi-Fi

## Technical Requirement

Wireless clients must be connected to the enterprise switching infrastructure.

## Devices Selected

**2 × Wireless Access Points**

Example:

```text
Laptop
  )))
   AP
   |
Access Switch
   |
Core Network
```

## Important Clarification

Two access points do not necessarily mean:

```text
AP1 = Corporate only
AP2 = Guest only
```

An enterprise AP can usually support multiple SSIDs.

For example:

```text
AP1
├── Corporate SSID
└── Guest SSID
```

The exact design will be decided later.

## Why Two APs?

Two APs give the lab:

- a more realistic wireless design
- room for testing multiple wireless clients
- room for future redundancy / coverage discussion
- flexibility for Corporate and Guest wireless segmentation

## Capabilities to Evaluate Later

- Multiple SSID support
- VLAN tagging
- WPA2/WPA3 support
- Corporate authentication support
- Guest network support
- Management access
- PoE support
- Integration with access switches

---

# 10. DNS Server

## Requirement That Creates the Need

The project explicitly requires an internal DNS Server.

## Technical Requirement

Internal users need name resolution.

Instead of accessing a service only through:

```text
192.168.x.x
```

users may access:

```text
intranet.company.local
```

DNS performs the mapping:

```text
intranet.company.local
        ↓
    IP Address
```

## Device Selected

**1 × DNS Server**

## Why It Helps the Lab

The DNS Server allows us to test:

- client/server connectivity
- internal services
- DNS resolution
- firewall rules
- VLAN access
- troubleshooting

## Capabilities to Evaluate Later

- Static DNS records
- Internal domain
- Client DNS configuration
- Reachability from approved VLANs
- Logging where possible

---

# 11. Web Server

## Requirement That Creates the Need

The project requires an internal Web Server.

## Technical Requirement

The company needs an internal service that employees can access.

Example:

```text
HR
Finance
Sales
   \ | /
 Web Server
```

## Device Selected

**1 × Web Server**

## Why It Is Useful

Without an internal application server, much of the testing would be limited to tools such as ping.

A Web Server lets us test real application traffic such as:

```text
HTTP
HTTPS
```

It also gives us a useful target for future firewall policies.

Example:

```text
HR → Web Server        ALLOW
Finance → Web Server   ALLOW
Guest → Web Server     DENY
```

## Capabilities to Evaluate Later

- HTTP
- HTTPS if supported
- Static internal webpage
- Server VLAN placement
- DNS integration

---

# 12. File Server — Future Expansion

The File Server is part of the planned project expansion, but not the initial implementation.

## Why It Is Not in the Initial Build

The first version should focus on:

- network connectivity
- segmentation
- routing
- security
- DNS
- web services
- monitoring

Once the base network is stable, the File Server can be added.

## Future Use Cases

It may later allow us to test:

- department-specific file access
- Finance-only resources
- HR-only resources
- server security rules
- backup concepts
- access-control policies

---

# 13. User PCs and Laptops

## Requirement That Creates the Need

The company has 42 employees and several network segments.

We need client devices to test the behavior of each network.

## Devices Selected

A representative number of PCs and laptops.

Possible initial lab:

| Department | Lab Endpoints |
|---|---:|
| HR | 2 |
| Finance | 2 |
| Sales | 2 |
| IT | 2 |
| Corporate Wi-Fi | 1–2 laptops |
| Guest Wi-Fi | 1–2 laptops |

This is enough to test the design without filling the topology with unnecessary endpoints.

## What They Will Be Used For

- DHCP testing
- DNS testing
- Internet testing
- VLAN testing
- Firewall testing
- Server access
- VPN simulation where possible
- Troubleshooting

---

# 14. ISP Simulation

## Requirement That Creates the Need

The company requires Internet connectivity.

A lab environment therefore needs something that represents the external ISP network.

## Device / Network Selected

**1 × Simulated ISP environment**

The exact implementation may use:

- a router
- a cloud object
- another simulated network

depending on the capabilities of the lab platform.

Example:

```text
Company
   |
Edge Router
   |
  ISP
   |
Internet
```

## Why It Is Needed

It allows us to test:

- Default routes
- NAT
- Firewall policies
- Internet reachability
- External services

---

# 15. External Internet Server

## Requirement That Creates the Need

We need a destination outside the company network to prove that Internet access works.

## Device Selected

**1 × External Web / Internet Server**

Example:

```text
HR PC
  |
Firewall
  |
Edge Router
  |
ISP
  |
External Server
```

## Tests It Enables

```text
HR → Internet Server       ALLOW
Finance → Internet Server  ALLOW
Sales → Internet Server    ALLOW
IT → Internet Server       ALLOW
Guest → Internet Server    ALLOW
```

while inbound unsolicited traffic can remain blocked:

```text
Internet Server → HR       DENY
Internet Server → Finance  DENY
Internet Server → Sales    DENY
```

This will help prove that firewall and Internet policies are working correctly.

---

# 16. Requirement-to-Device Traceability Matrix

| Requirement | Technical Need | Device / Component |
|---|---|---|
| Internet access | External routing | Edge Router |
| ISP connectivity | WAN connection | Edge Router + ISP simulation |
| Internet security | Traffic filtering | Firewall |
| Guest isolation | Security policy enforcement | Firewall |
| Finance/IT remote VPN | VPN termination | Firewall |
| Four departments | Network segmentation | Switch infrastructure |
| 42 users | Endpoint connectivity | Access Switches |
| Central LAN | Aggregation | Core Switch |
| Corporate Wi-Fi | Wireless access | Access Points |
| Guest Wi-Fi | Wireless + isolation | Access Points + Firewall |
| DNS | Name resolution | DNS Server |
| Internal website | Application service | Web Server |
| IT management | Management connectivity | Router/Switch/Firewall management |
| SNMP | Monitoring | Managed network devices |
| Syslog | Central logging capability | Managed network devices |
| Failure testing | Multiple infrastructure components | Core + Access topology |
| Future File Server | Expandable server segment | Server network |
| Future Branch Office | Expandable routing design | Future router/site |
| Future high availability | Redundancy | Future Router 2 + Firewall 2 + ISP 2 |

---

# 17. What Has Not Been Decided Yet

The device categories have been identified, but exact models have not yet been selected.

We still need to decide:

## Edge Router

- Exact model
- Number of interfaces
- Routing protocol support
- Future WAN requirements

## Firewall

- Exact firewall platform
- Number of interfaces
- VPN support
- NAT support
- Security-zone support

## Core Switch

- Layer 2 or Layer 3
- Number of ports
- Trunk support
- Inter-VLAN routing role

## Access Switches

- 24-port or 48-port
- PoE requirement
- Uplink speed
- Management capability

## Wireless Access Points

- Number of SSIDs
- VLAN support
- Security options
- PoE

## Servers

- IP addressing
- VLAN placement
- Services
- Access rules

---

# 18. Next Step

The next phase is **Device Specifications**.

For each device category, we will answer:

```text
What must this device be capable of doing
to satisfy our project requirements?
```

Example for the firewall:

```text
Firewall Requirements
├── VPN support
├── NAT/PAT
├── Security rules
├── Guest isolation
├── Multiple network zones
├── Logging
└── Management access
```

Only after defining these specifications should we select exact device models for the lab.

The design process therefore continues as:

```text
Requirements                 ✅
        ↓
Logical Device List          ✅
        ↓
Device Specifications        ← NEXT
        ↓
Device Selection
        ↓
Logical Topology
        ↓
VLAN Plan
        ↓
IP Addressing Plan
        ↓
Security Design
        ↓
Implementation
```
