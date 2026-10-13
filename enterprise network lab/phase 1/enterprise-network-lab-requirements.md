# Enterprise Network Lab — Project Requirements Sheet

## 1. Project Overview

This project simulates the network infrastructure of a small-to-medium-sized company.

The goal is to design and implement a realistic entry-level enterprise network that demonstrates core networking, segmentation, security, connectivity, monitoring, and troubleshooting concepts.

The initial implementation will focus on the company's main headquarters (HQ). Additional features such as a branch office, file server, high availability, and advanced monitoring will be added later as extensions.

---

## 2. Company Departments

The company will contain four main departments:

| Department | Users |
|---|---:|
| HR | 8 |
| Finance | 8 |
| Sales | 20 |
| IT | 6 |
| **Total** | **42** |

The lab does not need to contain one virtual PC for every employee. A smaller number of representative endpoints may be used while the network addressing plan is designed to support the full number of users and future growth.

---

## 3. Server Infrastructure

The initial network will include a dedicated server network.

### Initial Servers

- DNS Server
- Web Server

### Future Server Expansion

- File Server

Servers should be placed in a separate network segment from employee devices.

---

## 4. Guest Network

A dedicated Guest Wi-Fi network will be provided for visitors.

### Guest Access Policy

- Guest Wi-Fi → Internet: **ALLOW**
- Guest Wi-Fi → Corporate Network: **DENY**
- Guest Wi-Fi → HR: **DENY**
- Guest Wi-Fi → Finance: **DENY**
- Guest Wi-Fi → Sales: **DENY**
- Guest Wi-Fi → IT: **DENY**
- Guest Wi-Fi → Servers: **DENY**

The Guest network must remain isolated from all internal corporate resources.

---

## 5. Branch Office

The initial project will contain only the main headquarters.

A Branch Office will be implemented later as a Phase 2 extension.

The future design should support connectivity between:

- Headquarters
- Branch Office

The exact WAN/VPN and routing design will be decided during the expansion phase.

---

## 6. Internal Network Access Policy

Departments should not have unrestricted access to each other.

The initial access policy will follow a least-access approach.

| Source | Destination | Policy |
|---|---|---|
| HR | Servers | ALLOW |
| Finance | Servers | ALLOW |
| Sales | Servers | ALLOW |
| IT | Internal Networks | ALLOW |
| HR | Finance | DENY |
| HR | Sales | DENY |
| Finance | HR | DENY |
| Finance | Sales | DENY |
| Sales | HR | DENY |
| Sales | Finance | DENY |
| Guest | Internal Networks | DENY |

The IT department will have broader access for administration, maintenance, and troubleshooting.

More specific application-level server access rules may be introduced later.

---

## 7. Internet Access and Firewall

The company will use a dedicated firewall between the internal network and the Internet.

### Internet Access Requirements

- HR → Internet: **ALLOW**
- Finance → Internet: **ALLOW**
- Sales → Internet: **ALLOW**
- IT → Internet: **ALLOW**
- Guest → Internet: **ALLOW**
- Internet → Internal Networks: **DENY by default**
- Servers → Internet: **Only where required**

The firewall will be used to implement security policies and control traffic between internal networks, guest networks, servers, and the Internet.

The project will include:

- Dedicated Firewall
- Edge Router
- ISP / Internet simulation

---

## 8. Corporate Wi-Fi

The company will provide two separate wireless environments:

### Corporate Wi-Fi

Corporate Wi-Fi will be available to employees.

The final design should support placing employees into the appropriate corporate network based on their department.

Example:

- HR employee → HR network
- Finance employee → Finance network
- Sales employee → Sales network
- IT employee → IT network

The exact Wi-Fi authentication and VLAN assignment method will be decided during implementation.

### Guest Wi-Fi

Guest Wi-Fi will remain completely isolated from the corporate network and will provide Internet access only.

---

## 9. Remote Access / VPN

Remote access will be available only to selected departments.

### Departments Allowed Remote Access

- Finance
- IT

### Finance VPN Policy

Remote Finance users should be able to access:

- Finance resources
- Required server resources

They should not receive unrestricted access to other departments.

### IT VPN Policy

Remote IT users should have broader access for administration and troubleshooting, including access to:

- IT network
- Servers
- HR network where required
- Finance network where required
- Sales network where required
- Network infrastructure devices

Remote VPN users should not automatically receive unrestricted access to the entire corporate network.

---

## 10. Network Management and Monitoring

Management of network infrastructure will be restricted to the IT department.

### Management Access

IT should be able to manage:

- Routers
- Switches
- Firewall
- Servers

The following networks must not have direct access to infrastructure management interfaces:

- HR
- Finance
- Sales
- Guest

### Initial Monitoring Features

The project should include:

- Ping / basic availability checks
- Syslog
- SNMP

### Future Monitoring Expansion

An advanced monitoring dashboard may be added later to display information such as:

- Device availability
- Interface status
- Network traffic
- CPU utilization
- Packet loss
- Server availability

---

## 11. Availability and Reliability

The initial project will implement basic reliability without full enterprise high availability.

### Initial Design

The initial network should contain:

- 1 Edge Router
- 1 Firewall
- 1 Core network layer
- 2 Access Switches

The project will include failure and troubleshooting tests for selected interfaces and links.

Known single points of failure should be identified and documented.

Examples may include:

- Edge Router
- Firewall
- Core network device
- ISP connection

### High Availability

Full redundancy will not be required for the initial version.

High availability will be implemented later as an advanced extension.

---

## 12. Scalability and Future Expansion

The network must be designed so it can grow without requiring a complete redesign.

The addressing, VLAN, switching, and routing structure should allow for:

- More users
- Additional departments
- Additional VLANs
- Additional switches
- Additional wireless access points
- Additional servers
- Additional sites

### Planned Future Extensions

- Branch Office
- File Server
- Advanced Monitoring Dashboard
- IPv6
- More advanced VPN configuration
- Additional VLANs
- Additional network devices

### High-Availability Expansion

The future architecture should support removal of major single points of failure through:

- Second Firewall
- Second Edge Router
- Second ISP connection
- Redundant network links
- Failover
- High-availability firewall architecture
- Redundant routing paths

A possible future architecture may include:

```text
                 Internet
                /        \
             ISP 1      ISP 2
               |          |
          Edge-R1      Edge-R2
               \        /
             FW-1    FW-2
                \    /
              Core Network
               /      \
            Access1  Access2
```

---

# Initial Project Scope

The first complete version of the Enterprise Network Lab will focus on the Headquarters environment and should include:

- HR department
- Finance department
- Sales department
- IT department
- DNS Server
- Web Server
- Corporate Wi-Fi
- Guest Wi-Fi
- Network segmentation
- Controlled inter-network access
- Firewall security policies
- Internet connectivity
- Remote VPN access for Finance and IT
- Network management restricted to IT
- Syslog
- SNMP
- Basic failure testing
- Documentation of single points of failure

---

# Phase 2 / Advanced Extensions

After the initial HQ network has been completed and tested, the project may be extended with:

1. Branch Office
2. File Server
3. Advanced monitoring dashboard
4. High-availability firewall
5. Redundant edge routers
6. Dual ISP connectivity
7. Redundant links
8. Failover testing
9. IPv6
10. More advanced remote-access policies

---

# Project Success Criteria

The initial project will be considered complete when:

- All departments have working network connectivity according to the defined policies.
- Departments are properly segmented.
- HR, Finance, and Sales cannot communicate directly with each other unless explicitly permitted.
- Required departments can access internal servers.
- IT can access and manage internal infrastructure.
- Guest Wi-Fi can access the Internet but cannot access internal corporate networks.
- Finance and IT can use remote VPN access according to their permissions.
- Internet access is routed through the firewall.
- Basic monitoring using Syslog and SNMP is operational.
- Key failure scenarios have been tested.
- Major single points of failure are documented.
- The network topology, addressing plan, configurations, security rules, and testing results are documented for the portfolio.

---

# Next Design Steps

After approval of this requirements sheet, the project will move into the design phase:

1. Device List
2. Logical Network Topology
3. VLAN Plan
4. IP Addressing Plan
5. Security Zone Design
6. Firewall Policy Design
7. Physical / Packet Tracer Topology
8. Implementation
9. Testing
10. Documentation
