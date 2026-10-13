# Enterprise Network Lab — Single Point of Failure (SPOF) Analysis

## Purpose

The requirements (§11) state that known single points of failure must be
identified and documented, and the Success Criteria require both that major SPOFs
are documented and that key failure scenarios are tested.

This document now satisfies both parts. The **identification** was produced during
Phase 1, before the network existed. The **Test Result** column, deliberately left
open at that point, was completed after implementation by disabling each component
in turn and recording the observed impact. All seven SPOFs have been tested.

A **Single Point of Failure** is any component whose failure, on its own, disrupts
part or all of the network because no redundant component can take over. The
initial HQ design deliberately uses one of each core device (1 Edge Router,
1 Firewall, 1 Core, plus a single ISP link), so most core components are SPOFs by
design. This is an accepted, documented trade-off: high availability is a Phase 2
extension, not a Phase 1 requirement.

Testing did not only confirm the register. It **corrected** it in four places, and
the corrections changed the severity ranking. Those are set out in sections 4 and
5.

---

## 1. SPOF Register

| ID | Component | Predicted Impact (Phase 1) | Scope of Impact | Current Mitigation | Future Mitigation (Phase 2) | Test Result |
|----|-----------|----------------------------|-----------------|--------------------|------------------------------|-------------|
| SPOF-1 | **Edge Router** | All traffic to/from the Internet is lost | Entire site loses Internet; internal traffic unaffected | None — accepted risk for initial build | Second edge router (Edge-R2) + redundant routing paths | ✅ Confirmed, **wider**: also VPN and estate-wide NTP |
| SPOF-2 | **Firewall** | Internet access lost and all policy-controlled traffic through the firewall stops | Entire site — Internet and any inter-zone traffic that transits the firewall | None — accepted risk | Second firewall (FW-2), HA firewall pair with failover | ✅ **Corrected, narrower**: internal network fully operational |
| SPOF-3 | **Core layer** | Internal routing between VLANs/segments collapses | Whole internal network — most severe SPOF | None — accepted risk | Redundant core device + redundant links | ✅ Confirmed, **wider**: total collapse including wireless and DHCP |
| SPOF-4 | **ISP connection** | Internet connectivity lost at the WAN edge | Internet only; all internal services keep working | None — single ISP by design | Second ISP link (dual-homing) | ✅ Confirmed, **wider**: also VPN |
| SPOF-5 | **Access Switch** | Endpoints on that switch lose connectivity | Local — only the user group on that switch | Two access switches limit blast radius to one group | Redundant uplinks / stacked or additional access switches | ✅ Confirmed; also one access point |
| SPOF-6 | **DNS Server (DS1)** | Name resolution fails; users cannot reach services by name | All corporate users — high functional impact | None — single DNS server | Redundant/secondary DNS server | ✅ Confirmed exactly as predicted |
| SPOF-7 | **Monitoring / Syslog Server (DS3)** | Loss of monitoring and log collection | IT visibility only — network keeps operating | None — accepted; non-service-affecting | Redundant collector / log forwarding | ✅ Confirmed; also loses already-collected logs |

**Legend**

| Symbol | Meaning |
|--------|---------|
| ✅ | Tested, with the observed impact recorded in section 3 |
| **wider** | The failure affects more than Phase 1 predicted |
| **narrower** | The failure affects less than Phase 1 predicted |

---

## 2. Test Method

Each component was disabled in turn — powered off, or its link shut down — and the
same set of probes was run before the component was restored and the next test
begun. Describing every failure in terms of one fixed probe set is what makes the
results comparable: each row of section 3 answers the same questions.

| Probe | Capability tested |
|-------|-------------------|
| Workstation → its own gateway | Local routing |
| Workstation → DNS and Web servers, by IP and by name | Inter-VLAN routing, server segment, name resolution |
| Workstation → Internet | Perimeter path |
| Guest client → Internet | Guest path, which is separate from the corporate path |
| Wireless client association | Controller and access-point availability |
| IT workstation → VLAN 99 | Infrastructure management |
| Remote VPN client → internal resource | Remote access |
| `show ntp status` on an access switch | Time distribution |
| Syslog arrival on the monitoring server | Log collection |

**Two methodological points**, both learned during the campaign:

- **Every probe was repeated before its result was recorded.** In one test a first
  attempt showed a failure that had nothing to do with the component under test —
  a transient address-resolution loss — and would have been recorded as an
  inexplicable result had it not been repeated. A failure test produces a lot of
  simultaneous breakage, which is exactly the condition in which a single
  unrepeated observation is least trustworthy.
- **Both directions of the prediction were checked.** It is natural to test whether
  the predicted breakage occurred. It is just as important to test whether anything
  predicted to break kept working, which is how the correction to SPOF-2 was found.

---

## 3. Test Results

### SPOF-1 — Edge Router (EDGE-R1)

**Observed.** All Internet traffic lost. Remote-access VPN lost. The internal
network remained fully operational: inter-VLAN routing, server access, name
resolution and infrastructure management all continued.

**Additional finding — time synchronisation collapses estate-wide.** The core
switch moved to `unsynchronized, stratum 16` and the devices began to drift apart
from one another.

This confirms the reservation already recorded in 2.6.5: concentrating the time
source on a single device creates a dependency beyond that device's primary role.
The consequence is not loss of connectivity but loss of **log correlation**. The
logs generated during the outage — precisely the logs an investigation would want
— cannot be reliably ordered against each other, because the common reference they
depend on is the device that failed. Sub-stage 2.10.7 relies on exactly that
shared reference.

**Why the analysis missed it.** The register assessed the edge router as the
Internet path. By the end of the design phase it had also become the network's
time reference (D19) and, after implementation, the VPN termination point (D11).
Neither role existed when the register was written.

---

### SPOF-2 — Firewall (FW-1)

**Observed.** Internet access lost. The Guest network lost in its entirety.
Remote-access VPN lost.

**The internal network remained fully operational.** Users reached the servers,
name resolution worked, and IT retained access to every VLAN and to the management
segment.

**Correction of the Phase 1 estimate.** The register described the impact as
"Internet plus all policy-controlled transit traffic", which implies that internal
communication also collapses. It does not. Sub-stage 2.5.3 places inter-VLAN
routing on the core switch, so traffic between departments is routed *and filtered*
there and never transits the firewall at all. The firewall's real scope is the
perimeter: Internet, Guest and VPN.

This is not a weakness in the design — the design is the reason the impact is
narrow. It is an error in the analysis, which was written before the routing design
existed and was never revisited against it.

**Why Guest fails completely.** Not because of filtering, but because the Guest
gateway `10.0.60.1` lives on the firewall itself, a direct consequence of DD-2.
When the firewall goes, Guest clients lose their default gateway and the segment
has no router at all. This is the one case where the firewall's failure removes a
whole user population rather than a capability.

---

### SPOF-3 — Core Layer (CORE-SW1)

**Observed.** Total collapse. Workstations could not reach even their own default
gateway, because the SVIs live on the core switch.

Lost simultaneously: inter-VLAN routing, DHCP, server access, name resolution,
Internet access, the Guest network, infrastructure management, and the **entire
wireless estate** — the controller connects to the core, so the access points lost
their controller and dropped their clients.

**The only communication that survived** was between workstations in the same VLAN
on the same access switch, where no routing is required. That is the floor of what
a collapsed core leaves behind: Layer 2 within a single closet.

**Verdict.** Rank 1 and the characterisation "widest blast radius" are confirmed,
and the blast radius is in fact total rather than merely widest. The core is not
one role but five — inter-VLAN routing, every SVI, the DHCP server, the Guest
handoff, and the controller's uplink — which is why no capability survives it.

That accumulation of roles matters for Phase 2: making the core redundant is
structurally harder than adding a second firewall, because five functions have to
move or be duplicated, not one.

---

### SPOF-4 — ISP Connection (link ISP-R ↔ EDGE-R1)

**Observed.** All traffic to and from the Internet lost. The internal network
remained fully operational: inter-VLAN routing, server access and infrastructure
management continued normally.

**Additional finding — remote access is lost too.** The VPN tunnel terminates on
the same interface that carries the provider link, so remote Finance and IT users
cannot connect for the duration of the outage.

The Phase 1 description — "Internet only; all internal services keep working" —
was written before the VPN design (2.9) existed. The first half remains accurate;
the second is incomplete, because remote access is an internal service reached
from outside.

---

### SPOF-5 — Access Switch (ACC-SW1)

**Observed.** Only the users connected to that switch lost connectivity. The
second switch's users retained full functionality: server access, name resolution
and Internet.

**This confirms decision A9** — distributing users across both access switches
rather than grouping them by department. Because no department sits entirely on
one switch, no department is left entirely without service. The failure degrades
every department a little instead of removing one department completely, which for
a business is the better failure mode and was the reason for the decision.

**Additional observation.** LAP-1 went down with the switch, so wireless coverage
in its area was lost while LAP-2 continued. Wireless degrades rather than fails,
for the same structural reason: the access points are distributed, one per switch.

---

### SPOF-6 — DNS Server (DS1)

**Observed.** Name resolution stopped completely: queries failed with
`could not find host`, and the browser reported `Host Name Unresolved`.

Connectivity itself was untouched. The web server loaded normally when reached by
IP address, and Internet access continued to work.

**This confirms the assessment in 2.10.4** that losing DNS "feels like a total
outage to the user" while the network continues to operate correctly. The gap
between perceived and actual impact is the whole point of the entry, and it is why
rank 4 is justified: no path is broken, but almost nothing a user does works.

---

### SPOF-7 — Monitoring / Syslog Server (DS3)

**Observed.** No impact on users. All network services, Internet access and
infrastructure management remained operational. The only loss was Syslog
collection.

**Additional finding — the logs already collected are lost with the collector**,
because they are held in memory rather than written to storage. The failure
therefore removes not only future visibility but the existing record.

This sharpens rather than contradicts the Phase 1 assessment. "Non-service-
affecting" remains correct for users, and rank 7 stands. But for an investigation
the entry is worse than it appears: a collector that loses its history on failure
cannot be relied on as the record of what happened before it failed. A real
deployment would write logs to disk and forward them onward; in this build the
loss is total.

It is also worth stating what this test could *not* cover. The firewall sends no
Syslog at all (D16) and no device sends SNMP (D15), so the monitoring that this
SPOF removes is already substantially smaller than the monitoring design of 2.10
specifies. The low rank reflects the build, not the design.

---

## 4. Revised Severity Ranking

Ranked by how much of the network each failure actually takes down, as measured
rather than predicted.

| Rank | Was | Component | Measured impact |
|-----:|----:|-----------|-----------------|
| 1 | 1 | Core layer (SPOF-3) | Total collapse: routing, DHCP, servers, name resolution, Internet, Guest, management, wireless |
| 2 | 3 | Edge Router (SPOF-1) | Internet, VPN, and estate-wide loss of time synchronisation |
| 3 | 2 | Firewall (SPOF-2) | Internet, Guest and VPN. Internal network fully operational |
| 4 | 4 | DNS Server (SPOF-6) | Name resolution. Connectivity untouched |
| 5 | 5 | ISP connection (SPOF-4) | Internet and VPN. Internal network survives |
| 6 | 6 | Access Switch (SPOF-5) | That switch's users and one access point. Every department survives via the other switch |
| 7 | 7 | Monitoring Server (SPOF-7) | Syslog collection only, including the records already held |

**One change: the edge router and the firewall swap places.** The firewall moves
down because inter-VLAN traffic never transits it; the edge router moves up because
it carries three roles rather than one.

Ranks 1, and 4 through 7, are unchanged from the Phase 1 assessment.

---

## 5. What the Testing Changed

Four entries differ from the Phase 1 analysis. Three impacts are wider than
predicted and one is narrower.

| SPOF | Change | Cause |
|------|--------|-------|
| SPOF-1 | Wider — also NTP estate-wide | The edge router became the time source in 2.6.5, after the register was written |
| SPOF-2 | **Narrower** — internal network unaffected | Inter-VLAN routing was placed on the core in 2.5.3, after the register was written |
| SPOF-3 | Wider — also wireless and DHCP | The controller and the DHCP server were placed on the core in 2.8 and 2.4.7 |
| SPOF-4 | Wider — also VPN | The VPN design (2.9) did not exist when the register was written |

**All four come from one omission.** This document was produced during Phase 1,
before the routing design (2.5), the wireless design (2.8), the VPN design (2.9)
and the management design (2.10) existed. It assessed each device by the role it
had at that moment and was never revisited to ask what *else* each device had
acquired by the end of the design phase.

Three devices had picked up a second or third role — the edge router became the
time source and the VPN endpoint, the ISP link became the VPN path, the core became
the wireless aggregation point and the DHCP server — and one device turned out to
carry less than had been assumed.

The lesson is methodological rather than technical: **a SPOF analysis is only
valid against the design that existed when it was written.** It should be rerun at
the close of the design phase, when every device's full set of roles is finally
known, and not only completed with test results afterwards. This document
anticipated being *completed* later; what it did not anticipate was being
*corrected* later.

A practical form of that check: for each device, list every function it performs,
not the function it is named for. Every correction above would have been caught by
that one question.

---

## 6. Implications for Phase 2

The requirements (§12) sketch a high-availability architecture with a second edge
router, a second firewall, a second ISP link and a redundant core. The test results
say something about the order in which those are worth doing.

| Priority | Investment | Removes |
|---------:|------------|---------|
| 1 | Redundant core | The only failure that takes the whole network down |
| 2 | Second edge router | Internet, VPN and time synchronisation |
| 3 | Second firewall | Internet, Guest and VPN — with the internal network already safe |
| 4 | Secondary DNS server | The failure users feel most, for the least engineering effort |
| 5 | Second ISP link | Internet and VPN, and only if the edge router is already redundant |

Two observations follow.

**The firewall is a lower priority than the Phase 1 ranking implied.** A second
edge router removes more risk than a second firewall, which reverses the original
order. This is worth stating plainly because the opposite is the intuitive
assumption — the firewall is the security device, so it feels like the critical
one — and the measurement says otherwise.

**A secondary DNS server is the best value available.** It is the cheapest item on
the list, it requires no change to the topology, and it removes the failure that
users experience as a total outage. Ranked fourth by blast radius, it would
arguably be first by return on effort.

**An external time source** is not redundancy, but it belongs in the same
conversation. Taking time from outside the network, rather than from the edge
router, removes the NTP dependency found in SPOF-1 without adding a device
(D19 records why it was not possible in this build).

---

## 7. Notes

- Items ranked 1–3 map directly to the Phase 2 high-availability architecture
  sketched in the requirements (dual ISP, Edge-R1/R2, FW-1/FW-2, redundant core),
  confirming that the future design removes the top SPOFs first — though in a
  different order than Phase 1 assumed.
- SPOF-5 is partially mitigated **now**, and the testing confirmed it. Two access
  switches with users distributed across both (decision A9) means a single switch
  failure degrades every department rather than removing one.
- Guest is the only user population whose gateway sits on a perimeter device
  (DD-2). It is therefore the only population that loses service entirely when a
  perimeter device fails, which is visible in SPOF-2 and in no other test.
- The detailed policy-verification results and the full test narrative are in the
  companion document, **Enterprise Network Lab — Test Results**. Deviations between
  the design and the implemented network are recorded in 3.6; references of the
  form **D16**, **D19** point there.
