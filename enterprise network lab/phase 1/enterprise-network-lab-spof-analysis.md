# Enterprise Network Lab — Single Point of Failure (SPOF) Analysis

## Purpose

The requirements (§11) state that known single points of failure must be
identified and documented, and the Success Criteria require that major SPOFs are
documented and that key failure scenarios are tested. This document satisfies the
**identification and documentation** part now (Phase 1 analysis). The **Test
Result** column is intentionally left open and is completed later, during the
failure-testing phase, once the network is built in Packet Tracer.

A **Single Point of Failure** is any component whose failure, on its own, disrupts
part or all of the network because no redundant component can take over. The
initial HQ design deliberately uses one of each core device (1 Edge Router,
1 Firewall, 1 Core, plus a single ISP link), so most core components are SPOFs by
design. This is an accepted, documented trade-off: high availability is a Phase 2
extension, not a Phase 1 requirement.

**Legend**

| Symbol | Meaning |
|--------|---------|
| ✅ | Analysed and documented (Phase 1) |
| ⏳ | To be completed during the testing phase |

---

## 1. SPOF Register

| ID | Component | Failure Impact | Scope of Impact | Current Mitigation (Phase 1) | Future Mitigation (Phase 2) | Test Result |
|----|-----------|----------------|-----------------|------------------------------|------------------------------|-------------|
| SPOF-1 | **Edge Router** | All traffic to/from the Internet is lost | Entire site loses Internet; internal traffic unaffected | None — accepted risk for initial build | Second edge router (Edge-R2) + redundant routing paths | ⏳ |
| SPOF-2 | **Firewall** | Internet access lost and all policy-controlled traffic through the firewall stops | Entire site — Internet and any inter-zone traffic that transits the firewall | None — accepted risk | Second firewall (FW-2), HA firewall pair with failover | ⏳ |
| SPOF-3 | **Core layer** | Internal routing between VLANs/segments collapses | Whole internal network — most severe SPOF | None — accepted risk | Redundant core device + redundant links | ⏳ |
| SPOF-4 | **ISP connection** | Internet connectivity lost at the WAN edge | Internet only; all internal services keep working | None — single ISP by design | Second ISP link (dual-homing) | ⏳ |
| SPOF-5 | **Access Switch** | Endpoints on that switch lose connectivity | Local — only the user group on that switch (not site-wide) | Two access switches limit blast radius to one group | Redundant uplinks / stacked or additional access switches | ⏳ |
| SPOF-6 | **DNS Server (DS1)** | Name resolution fails; users cannot reach services by name | All corporate users — high functional impact | None — single DNS server | Redundant/secondary DNS server | ⏳ |
| SPOF-7 | **Monitoring / Syslog Server (DS3)** | Loss of monitoring and log collection | IT visibility only — network keeps operating for users | None — accepted; non-service-affecting | Redundant collector / log forwarding | ⏳ |

---

## 2. Severity Ranking

Ranked by how much of the network each failure takes down, to prioritise future
high-availability work:

| Rank | Component | Why |
|-----:|-----------|-----|
| 1 | Core layer (SPOF-3) | Brings down all internal routing — the widest blast radius |
| 2 | Firewall (SPOF-2) | Internet plus all policy-controlled transit traffic |
| 3 | Edge Router (SPOF-1) | All Internet connectivity |
| 4 | DNS Server (SPOF-6) | Users cannot resolve names; feels like a total outage to them |
| 5 | ISP connection (SPOF-4) | Internet lost, internal network survives |
| 6 | Access Switch (SPOF-5) | Local only — one user group |
| 7 | Monitoring Server (SPOF-7) | No user-facing impact; affects IT visibility only |

---

## 3. Notes

- Items ranked 1–3 map directly to the Phase 2 high-availability architecture
  already sketched in the requirements (dual ISP, Edge-R1/R2, FW-1/FW-2, redundant
  core), confirming the future design removes the top SPOFs first.
- SPOF-5 (Access Switch) is partially mitigated **now**: using two access switches
  instead of one means a single switch failure affects only its own user group,
  not the whole site.
- The **Test Result** column is completed during the failure-testing phase. For
  each SPOF, a test will disable the component (or its link) in Packet Tracer and
  record the observed impact, satisfying the Success Criteria requirement that key
  failure scenarios are tested.
