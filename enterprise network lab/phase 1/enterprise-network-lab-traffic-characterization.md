# Enterprise Network Lab — Traffic Characterization

## 0. Purpose of This Document

This document characterizes the traffic of the Enterprise Network Lab (HQ).
It follows a top-down methodology: we first describe **who** generates traffic,
**where** the data lives, **what** applications run, and only then **how** traffic
flows, in **what volume**, and with **what quality requirements**.

Build order:

```text
1. User Communities        ← who generates traffic
2. Data Stores             ← where traffic goes
3. Applications            ← what kind of traffic
4. Traffic Flows           ← who talks to whom
5. Traffic Flow Type       ← client-server / peer / etc.
6. Volume & Direction      ← how much, which way
7. QoS Requirements        ← latency / jitter / loss / criticality
8. Traffic Matrix          ← the consolidated deliverable
```

---

# 1. User Communities

A user community is a group of users who share a location, a function, and a
common set of applications. Grouping users this way lets us reason about traffic
per group instead of per individual.

| ID   | User Community | Users | Location | Access Method | Primary Role |
|------|----------------|------:|----------|---------------|--------------|
| UC1  | HR             | 8     | HQ       | Wired + Corporate Wi-Fi | Human resources staff |
| UC2  | Finance        | 8     | HQ       | Wired + Corporate Wi-Fi | Finance / accounting staff |
| UC3  | Sales          | 20    | HQ       | Wired + Corporate Wi-Fi | Sales staff |
| UC4  | IT             | 6     | HQ       | Wired + Corporate Wi-Fi | Administration, management, troubleshooting |
| UC5  | Guest          | Variable | HQ    | Guest Wi-Fi only | Visitors (Internet-only) |
| UC6  | Finance (Remote) | Subset of UC2 | External | VPN | Remote finance access |
| UC7  | IT (Remote)    | Subset of UC4 | External | VPN | Remote administration |
|      | **Total (internal)** | **42** | | | |

Notes:

- UC6 and UC7 are not new people — they are UC2 and UC4 connecting from outside
  via VPN. They are listed separately because their **traffic flow**, **entry
  point**, and **access scope** differ from the on-site case.
- Guest (UC5) has no fixed count; it is sized generously in the addressing plan
  and is fully isolated from all internal resources.

---

# 2. Data Stores

A data store is any location where data or services live and are accessed by user
communities. Data stores are the destinations of most traffic flows.

| ID  | Data Store | Location | Hosted Services | Accessed By |
|-----|-----------|----------|-----------------|-------------|
| DS1 | DNS Server | Server segment | DNS resolution | All corporate users (UC1–UC4), remote users |
| DS2 | Web Server | Server segment | Internal HTTP/HTTPS | All corporate users (UC1–UC4), remote users |
| DS3 | Monitoring / Syslog Server | Server segment | SNMP manager, Syslog collector | IT (UC4), infrastructure devices |
| DS4 | Internet (external) | Outside firewall | External web/DNS | All users incl. Guest (via firewall) |
| DS5 | File Server *(future)* | Server segment | File sharing | Corporate users (Phase 2) |

**Assumption:** DS3 (a dedicated Monitoring/Syslog server) is introduced here to
resolve the open design question from the requirements. SNMP and Syslog need a
collector; placing it in the server segment keeps management data alongside other
protected resources.

---

# 3. Characterized Traffic Flows (ALLOW)

Each allowed flow is characterized by **Type**, **Volume**, **Direction**, and
**QoS / Criticality**. Denied flows are listed separately in Section 4, because a
denied flow carries no traffic to characterize.

## 3.1 User → Infrastructure Services

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| HR | DNS Server | DNS | Client-server | Very low (small, frequent queries) | Asymmetric | Low latency · **high criticality** |
| Finance | DNS Server | DNS | Client-server | Very low | Asymmetric | Low latency · **high criticality** |
| Sales | DNS Server | DNS | Client-server | Low (20 users, more queries) | Asymmetric | Low latency · **high criticality** |
| IT | DNS Server | DNS | Client-server | Very low | Asymmetric | Low latency · **high criticality** |
| HR | Web Server | HTTP/HTTPS | Client-server | Low–moderate | Asymmetric (small request, larger response) | Best-effort · moderate criticality |
| Finance | Web Server | HTTP/HTTPS | Client-server | Low–moderate | Asymmetric | Best-effort · moderate criticality |
| Sales | Web Server | HTTP/HTTPS | Client-server | Moderate (20 users) | Asymmetric | Best-effort · moderate criticality |
| IT | Web Server | HTTP/HTTPS | Client-server | Low | Asymmetric | Best-effort · moderate criticality |

## 3.2 User → Internet

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| HR | Internet | HTTP/HTTPS/DNS | Client-server | Moderate | Asymmetric (download-heavy) | Best-effort |
| Finance | Internet | HTTP/HTTPS/DNS | Client-server | Moderate | Asymmetric | Best-effort |
| Sales | Internet | HTTP/HTTPS/DNS | Client-server | **High** (20 users, browsing) | Asymmetric | Best-effort |
| IT | Internet | HTTP/HTTPS/DNS | Client-server | Low–moderate | Asymmetric | Best-effort |
| Guest | Internet | HTTP/HTTPS/DNS | Client-server | Variable | Asymmetric | Best-effort · **isolated** |

## 3.3 IT → Departments (Management / Troubleshooting)

The only allowed inter-department flows. IT is the administrative exception; all
other department-to-department traffic is denied (see Section 4).

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| IT | HR | Management / troubleshooting | Terminal-host | Very low, sporadic | Asymmetric | Security-critical · moderate criticality |
| IT | Finance | Management / troubleshooting | Terminal-host | Very low, sporadic | Asymmetric | Security-critical · moderate criticality |
| IT | Sales | Management / troubleshooting | Terminal-host | Very low, sporadic | Asymmetric | Security-critical · moderate criticality |

## 3.4 IT → Network Devices (Management)

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| IT | Switches | SSH | Terminal-host | Very low | Asymmetric (commands ↓, output ↑) | Security > bandwidth · **high criticality** |
| IT | Router | SSH | Terminal-host | Very low | Asymmetric | Security > bandwidth · **high criticality** |
| IT | Firewall | SSH / HTTPS (GUI) | Terminal-host | Low | Asymmetric | Security > bandwidth · **high criticality** |
| IT | Servers | SSH / admin | Terminal-host | Low | Asymmetric | Security > bandwidth · **high criticality** |
| IT | Network Devices | SNMP (polling) | Client-server | Low (periodic) | Asymmetric | Best-effort · moderate criticality |

## 3.5 Network Devices → Monitoring / Logging

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| Router | Monitoring Server | SNMP | Client-server (agent→mgr) | Low, steady | Asymmetric (↑ to server) | Best-effort · moderate criticality |
| Switches | Monitoring Server | SNMP | Client-server | Low, steady | Asymmetric | Best-effort · moderate criticality |
| Firewall | Monitoring Server | SNMP | Client-server | Low, steady | Asymmetric | Best-effort · moderate criticality |
| Network Devices | Syslog Server | Syslog (UDP 514) | Client-server (event-driven) | Low, bursty on events | Asymmetric (↑ to server) | Best-effort · moderate criticality |

## 3.6 Remote VPN Traffic

**Remote Finance** — restricted scope (per requirements §9):

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| Remote Finance | Finance resources | Application traffic | Client-server (over VPN) | Low (few users) | Asymmetric | **Confidentiality** · moderate criticality |
| Remote Finance | Required Servers | DNS/HTTPS | Client-server (over VPN) | Low | Asymmetric | Confidentiality · moderate criticality |

**Remote IT** — broad scope (per requirements §9):

| Source | Destination | Service | Type | Volume | Direction | QoS / Criticality |
|--------|-------------|---------|------|--------|-----------|-------------------|
| Remote IT | IT Network | Management | Terminal-host (over VPN) | Very low | Asymmetric | **Security-critical** · high criticality |
| Remote IT | Servers | Management | Terminal-host (over VPN) | Low | Asymmetric | Security-critical · high criticality |
| Remote IT | Network Devices | SSH/SNMP | Terminal-host (over VPN) | Very low | Asymmetric | Security-critical · high criticality |
| Remote IT | Other departments | Troubleshooting (where required) | Terminal-host (over VPN) | Very low, sporadic | Asymmetric | Security-critical · moderate criticality |

---

# 4. Denied Flows (Security Policy Reference)

These flows carry **no traffic** and are therefore not characterized. They are
documented here because denying them is a deliberate design decision (network
segmentation), and they feed directly into the later Firewall Policy / ACL design.

## 4.1 Department → Department (Denied)

| Source | Destination | Policy |
|--------|-------------|--------|
| HR | Finance | DENY |
| HR | Sales | DENY |
| Finance | HR | DENY |
| Finance | Sales | DENY |
| Sales | HR | DENY |
| Sales | Finance | DENY |

## 4.2 Guest → Internal (Denied)

| Source | Destination | Policy |
|--------|-------------|--------|
| Guest | HR | DENY |
| Guest | Finance | DENY |
| Guest | Sales | DENY |
| Guest | IT | DENY |
| Guest | Servers | DENY |

Guest traffic is permitted to the Internet only (see 3.2) and is fully isolated
from every internal resource.

---

# 5. Methodology & Assumptions

**Volume estimation.** Traffic volume is expressed qualitatively (very low / low /
moderate / high) rather than in absolute bandwidth figures. This follows the
Oppenheimer method, where the total load of an application is estimated as the load
per flow multiplied by the number of devices using it (Oppenheimer, *Top-Down
Network Design*, Cisco Press). Because this is a greenfield lab with no existing
network to baseline, per-flow load cannot be measured directly; qualitative
estimates are used, driven mainly by user count, transaction size, and frequency.
Oppenheimer explicitly notes that traffic-load estimates are unlikely to be
precise, and that the goal is to avoid critical bottlenecks rather than to achieve
exact numbers.

**QoS classification.** QoS levels follow the categories in the Cisco *Enterprise
QoS Solution Reference Network Design Guide*. The guide's numeric thresholds for
real-time traffic — one-way latency ≤ 150 ms, jitter ≤ 30 ms, loss ≤ 1 % — apply
to interactive voice and video. The current HQ design contains no real-time media;
its flows are elastic data (DNS, HTTP/HTTPS, management), which TCP recovers and
which tolerate delay. These are therefore correctly classified as **best-effort**,
with DNS and management flows flagged as **high criticality** on a functional basis
(other services depend on them, or they secure the infrastructure) rather than by a
latency budget. In a campus network, the primary QoS concern is managing packet
loss under congestion rather than latency or jitter.

**Direction.** Nearly all flows are client-server and therefore asymmetric: a small
upstream request produces a larger downstream response. This matters for capacity
planning, since the constrained direction is generally toward the users.

**Scope note.** Numeric QoS budgets and per-flow bandwidth figures become relevant
if real-time services (e.g. VoIP) are added in Phase 2, at which point the flows
above should be re-characterized with measured or codec-derived values.

## References

- Oppenheimer, P. *Top-Down Network Design*. Cisco Press.
- Cisco. *Enterprise QoS Solution Reference Network Design Guide.*
- Cisco. *Bandwidth, Latency, and QoS Considerations* (one-way latency ≤ 150 ms).
