# Enterprise Network Lab — Test Results

## Purpose

This document records whether the finished network enforces the access policy the
design specifies. It sits outside the phase numbering, alongside the SPOF
analysis, because it is not a design deliverable: the phases specify and build the
network, this document reports on it.

Eighteen tests, derived directly from the Success Criteria and from the policy
tables in §4, §6, §7, §9 and §10 of the requirements.

**Scope.** Policy verification only. The seven failure tests — one per single point
of failure — are recorded in the companion document, **Enterprise Network Lab —
SPOF Analysis**, together with the register they test against. The two documents
divide the testing between them: this one asks *does the network allow and deny
the right things*, the other asks *what breaks when a device fails*.

Deviations between the design and what was built are recorded separately in 3.6;
references of the form **D5**, **D16** point there.

---

## Method

**The tests are functional, not declarative.** Every result below is a probe sent
between two real endpoints, not an inspection of a configuration file. This was
not a stylistic choice. The implementation environment does not return any output
for `show running-config | include access-group`, so there is no reliable way to
confirm from the device that a list is applied. The only trustworthy evidence is
traffic that behaves as the policy requires.

That turned out to matter — see *Defects found by this campaign*.

**Deny results are distinguished by failure mode.** A packet dropped by an access
list times out. A packet that reaches a host which declines the service is
refused. The two are not the same result and are recorded separately: a
`Connection refused` proves the access list permitted the traffic, which is a pass
for an access-control test even when the service itself is unavailable (D17).

**Each test was run from a real endpoint in the source VLAN**, not from the core
switch. A probe sourced from the router that holds the access list does not
traverse the list in the direction the policy describes, and would pass regardless.

---

## 1. Policy Verification

| # | Test | Expected | Observed | Result |
|---|------|----------|----------|--------|
| T1 | HR → DNS (53) and Web (80/443) in VLAN 50 | Permit | Reached both | ✅ |
| T2 | Finance → DNS and Web in VLAN 50 | Permit | Reached both | ✅ |
| T3 | Sales → DNS and Web in VLAN 50 | Permit | Reached both | ✅ |
| T4 | HR → Finance workstation | Deny | Timed out | ✅ |
| T5 | HR → Sales workstation | Deny | Timed out | ✅ |
| T6 | Finance → HR workstation | Deny | Timed out | ✅ |
| T7 | Finance → Sales workstation | Deny | Timed out | ✅ |
| T8 | Sales → HR workstation | Deny | Timed out | ✅ |
| T9 | Sales → Finance workstation | Deny | Timed out | ✅ |
| T10 | IT → HR, Finance and Sales workstations | Permit | Reached all three | ✅ |
| T11 | IT → VLAN 50, including SSH (22) to a server | Permit | Reached; SSH `Connection refused` by the host | ✅ (see D17) |
| T12 | Server → HR, Finance, Sales workstation (connection opened from the server) | Deny | Timed out, while replies to client-initiated sessions continued to arrive | ✅ |
| T13 | HR, Finance and Sales → VLAN 99, including the core's own SVI `10.0.99.1` | Deny | Timed out from all three | ✅ (fixed during testing) |
| T14 | IT → VLAN 99: SSH to core switch, both access switches, firewall management | Permit | All four reachable and authenticated | ✅ |
| T15 | Guest client → every internal subnet (`10.0.0.0/16`) | Deny | Timed out; `show access-list` on the firewall confirmed the hits on the deny rule | ✅ |
| T16 | Guest client → Internet | Permit | Reached, translated under the Guest access list only | ✅ |
| T17 | Each corporate VLAN → Internet | Permit | Reached, with translation visible on the edge router and none on the firewall | ✅ |
| T18 | Same remote workstation, Finance credentials then IT credentials | Differentiated | Finance pool reached Finance and the servers, denied VLAN 99; IT pool reached VLAN 99 | ✅ |

**Eighteen of eighteen.** Every Success Criterion concerning who may reach what is
satisfied by test.

### Notes on individual tests

**T12 is the test that proves the access lists are stateless, and why that
matters.** The core switch does not track connections, so a rule permitting users
to reach the servers does not implicitly permit the servers to reach users. The
test opens a connection *from* a server toward a workstation and confirms it is
denied, while replies to client-initiated sessions continue to arrive normally.
Those replies depend on explicit allowances — the `established` keyword for TCP
and a rule for DNS responses — which are recorded in section 2 of 3.6 as additions
the design tables omitted.

**T15 is evidenced twice.** The probe times out, and `show access-list` on the
firewall shows the hit counter on the Guest deny rule incrementing. A timeout
alone proves only that the traffic did not arrive; the counter proves *which rule*
stopped it. Where a device offers this, it is the stronger evidence.

**T18 is the test the design cared most about.** Requirement F9 — identified in
3.1.3 as the requirement whose failure carries the greatest security consequence —
is that remote access is differentiated by group rather than granted wholesale.
The test isolates it properly: one workstation, one tunnel configuration, two sets
of credentials, two different outcomes. The differentiation comes from the address
pool and the firewall rules that reference it, which is exactly the mechanism
2.9.3 specifies. It is also the requirement whose implementation deviated most
from the design (D11), which makes verifying it by test rather than by inspection
the point rather than a formality.

### Additional verifications

These are not access-control tests, but they were confirmed at the same time
because the policy tests depend on them:

| Check | Result |
|-------|--------|
| DHCP in VLANs 10, 20, 30, 40 | Correct address, mask, gateway and DNS server in each |
| Wireless client on each corporate SSID | Placed in its department's VLAN, with that VLAN's gateway and DNS server |
| Guest wireless | Associates and reaches the Internet; addressed statically (D5) |
| NTP | Switches report stratum 4 with the edge router as reference |
| Syslog — device events | Interface state changes, logins and configuration changes arrive at `10.0.50.30` with consistent timestamps, from the switches and routers. The firewall sends nothing (D16) |
| Syslog — policy events | **None recorded on any device.** The `log` keyword is rejected on extended access list entries by the core switch as well as the firewall, so no rule match is logged anywhere (D16) |
| Internal DNS resolution | Works for configured records; external names do not resolve (D7) |

---

## 2. Defects Found by This Campaign

Three faults were present in the network at the moment testing began. All three
had been introduced during configuration and none had been noticed. This is the
reason the campaign exists, and it is worth recording rather than presenting an
eighteen-for-eighteen result as if it had been achieved first time.

### 1. No access list was applied anywhere

The first segmentation test — HR to Finance — succeeded when it should have been
denied. All five access-group statements were absent: configured at some point,
lost at some later point, and invisible because the device reports nothing for the
command that would have shown them. Re-applied and retested.

*Consequence for the method:* this is precisely why the campaign tests behaviour
rather than configuration. A configuration review would have found nothing to look
at. The network had a complete and correct set of access lists and enforced none
of them.

### 2. The inbound user-VLAN access lists broke DHCP

Once the lists were applied, no workstation could obtain an address. DHCP requests
carry source `0.0.0.0` and destination `255.255.255.255`, so they match no rule
written in terms of VLAN subnets and fell to the implicit deny. Fixed with an
explicit `permit udp any any eq 67` at the head of each list.

*Why it was missed:* the policy tables in 2.7.2 describe traffic between known
subnets, which is what a policy is about. DHCP is the one flow that exists before
the host has an address to be a subject of policy at all.

### 3. The management VLAN was reachable from user VLANs

Rule 2 of 2.7.2 is applied outbound on VLAN 99, which filters traffic forwarded
*to* the management segment but not traffic addressed to the core switch's own SVI
at `10.0.99.1` — such traffic terminates on the interface and is never forwarded
out of it. HR could therefore ping the core switch's management address. Fixed by
adding a `deny ip <user VLAN> 10.0.99.0 0.0.0.255` inbound on each user VLAN,
before the permit.

This is the one worth dwelling on. The policy was correct, the access list
implementing it was correct, and the direction it was applied in was wrong in a
way that is invisible unless tested — the interface being protected was reachable
by way of the very interface the traffic entered.

**What the three have in common.** None would have been found by reading the
configuration. The first was an omission the device would not report, the second
was traffic the policy never contemplated, and the third was a correct rule in the
wrong place. All three were found by sending a packet and watching what happened
to it, which is the argument for running this campaign at all rather than treating
a completed configuration as a completed network.

Faults 2 and 3 are recorded in section 2 of 3.6 as additions the design tables
require.

---

## 3. Success Criteria Status

| Success Criterion | Evidence | Status |
|-------------------|----------|--------|
| All departments have connectivity per the defined policies | T1–T3, T10–T11, T17 | Met |
| Departments are properly segmented | T4–T9 | Met |
| HR, Finance and Sales cannot communicate directly | T4–T9 | Met |
| Required departments can access internal servers | T1–T3 | Met |
| IT can access and manage internal infrastructure | T10, T11, T14 | Met |
| Guest Wi-Fi reaches the Internet but not internal networks | T15, T16 | Met |
| Finance and IT can use VPN per their permissions | T18 | Met |
| Internet access is routed through the firewall | T17 | Met |
| Basic monitoring using Syslog and SNMP is operational | Device Syslog from the switches and routers. No policy logging on any device; SNMP not implemented | Partially met — D15, D16 |
| Key failure scenarios have been tested | SPOF analysis, seven tests | Met |
| Major single points of failure are documented | SPOF analysis | Met |
| Topology, addressing, configurations, rules and test results documented | Phases 1–3, 3.6, this document, SPOF analysis | Met |

One criterion is not fully met, and for one reason: the monitoring design of 2.10
cannot be exercised in this environment. SNMP does not exist on any device in the
build, and no device can record a rule match — the firewall implements no logging
at all, and the core switch rejects the `log` keyword on access list entries. Both
are recorded in 3.6 as D15 and D16, with the reasoning for not configuring a
clear-text SNMP agent that no collector could read.

Every other criterion is met and evidenced by a specific test.

**The shape of the result is worth stating.** Eighteen of eighteen access-control
tests pass, and the one criterion that fails is the one about observing the
network rather than controlling it. The policy is enforced correctly and
demonstrably; what the build cannot do is produce a record of having enforced it.
A reader should take both halves of that: the controls work, and proving it after
the fact would be difficult.

---

## Companion Documents

| Document | Covers |
|----------|--------|
| **SPOF Analysis** | The register of single points of failure, the seven failure tests, and the severity ranking as measured rather than predicted |
| **3.6 Implementation Decisions and Deviations** | Every point where the design could not be implemented as written, with its consequence |
