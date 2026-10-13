# Phase 2 — Logical Network Design

## What This Phase Is

Phase 2 designs **how the network works**, without selecting hardware. It works
with concepts and relationships — segments, VLANs, subnets, protocols, security
zones — not device models. Hardware selection belongs to Phase 3 (Physical
Design).

The reason for this order: the logical design defines *what each device must be
able to do*. Choosing hardware first risks selecting equipment that cannot meet
requirements that have not yet been defined.

---

## Methodology Note

This project follows the **Top-Down Network Design** methodology (Oppenheimer,
Cisco Press), adapted to the needs of this lab. The phase structure is preserved
(Analyze Requirements → Logical Design → Physical Design → Test/Optimize/Document),
with additional sub-stages for wireless, VPN, management, and security zones,
which arise from explicit requirements in the project requirements sheet.

Sub-stages 2.8, 2.9 and 2.10 are extensions rather than formal Oppenheimer stages;
they are included because requirements §8, §9 and §10 demand them.

---

## Sub-Stages and File Map

| Sub-stage | Content | File | Status |
|-----------|---------|------|--------|
| 2.1 | Logical Topology | `2.1-2.2-topology-and-hierarchy.md` | ✅ |
| 2.2 | Hierarchical Structure | `2.1-2.2-topology-and-hierarchy.md` | ✅ |
| 2.3 | VLAN Design | `2.3-2.4-vlan-and-addressing.md` | ✅ |
| 2.4 | IP Addressing Design | `2.3-2.4-vlan-and-addressing.md` | ✅ |
| 2.5 | Routing Design | `2.5-routing-design.md` | ✅ |
| 2.6 | Security Zones (Zero Trust principles) | `2.6-2.7-security-zones-and-firewall-policy.md` | ✅ |
| 2.7 | Firewall Policy Design | `2.6-2.7-security-zones-and-firewall-policy.md` | ✅ |
| 2.8 | Wireless Logical Design | `2.8-2.9-2.10-wireless-vpn-management.md` | ✅ |
| 2.9 | VPN Logical Design | `2.8-2.9-2.10-wireless-vpn-management.md` | ✅ |
| 2.10 | Management & Monitoring Design | `2.8-2.9-2.10-wireless-vpn-management.md` | ✅ |

**Reading order:** the files are numbered and should be read in sequence. Each
sub-stage depends on the ones before it — VLANs need the topology, addressing
needs the VLANs, routing needs the subnets, security zones need all of the above.

---

## Inputs from Phase 1

Every Phase 2 decision traces back to a Phase 1 artifact:

| Phase 1 artifact | Feeds into |
|------------------|------------|
| User Communities (UC1–UC7) | Number and purpose of segments / VLANs |
| Data Stores (DS1–DS5) | Server segment definition |
| Traffic flows (§3) | Topology shape and routing design |
| Denied flows (§4) | Security zones and firewall policy |
| Scalability requirement (§12) | Headroom in the addressing plan |
| SPOF analysis | Where high availability is added later |

---

## Key Design Decisions

Decisions taken at the start of this phase, recorded here so they are traceable:

**Management as a separate segment.** The management plane (device management
interfaces) is a distinct segment, not part of the IT user segment. IT users and
device management interfaces are different things at different trust levels; only
IT is permitted to reach management (§10).

**Guest terminates on the firewall.** The Guest network is treated as an
**untrusted security zone**, not as another internal corporate subnet. It connects
to a dedicated firewall interface rather than passing through the core. This gives
**structural isolation** — there is no path to internal resources to block, rather
than a path that is blocked by rules. This matches §4, which requires Guest to
remain isolated from all internal corporate resources.

**In-band management with a dedicated VLAN.** Management traffic is isolated in
its own VLAN (99) but carried on the production infrastructure, with access
restricted by core access control lists. Out-of-band management — a physically
separate network — offers stronger isolation and survives a core failure, but its
overhead is disproportionate to five infrastructure devices at a single site, and
§10 requires restricted access rather than physical separation. The costs this
accepts, and the triggers for revisiting it, are in 2.10.2.

**Zero Trust as principles, not full architecture.** Sub-stage 2.6 applies Zero
Trust *principles* — deny by default, least privilege, no implicit trust between
zones. It does not claim a full Zero Trust architecture: identity-based access,
micro-segmentation, and continuous verification are out of scope for this lab and
are documented as such.

---

## Phase Status

✅ **Complete.** All ten sub-stages are finished.

### Deferred by design

Three capabilities are specified as design requirements but not implemented, each
with a documented migration trigger:

| Capability | Sub-stage | Blocked by | Trigger |
|------------|-----------|------------|---------|
| Dynamic wireless VLAN assignment (802.1X) | 2.8.3 | No RADIUS or directory infrastructure | Authentication infrastructure added |
| Multi-factor authentication for VPN | 2.9.5 | Same, plus no MFA support in the implementation environment | Authentication infrastructure added |
| Dynamic routing (OSPF) | 2.5.2 | Single-path topology — nothing to select or fail over to | Branch office or high availability added |
| Out-of-band management | 2.10.2 | Overhead disproportionate to five devices at one site | Branch office, larger device count, or high availability |

The wireless and VPN entries share a single trigger: one infrastructure addition
resolves both.

### Known weaknesses accepted

Recorded here so they are visible without reading each sub-stage:

| Weakness | Where | Why accepted |
|----------|-------|--------------|
| Management isolation is rule-based, not structural | 2.10.2, 2.6.6 | In-band management chosen for proportionality; core ACL rules 1–2 carry the boundary |
| A core failure removes remote management as well as routing | 2.10.2, SPOF-3 | Single site; console access is the documented recovery path |
| Wireless passphrases are shared per department | 2.8.3 | No identity infrastructure; rotation is an administrative control |
| VPN uses certificates rather than true multi-factor | 2.9.5 | A certificate is a file and can be copied; a substitute, not an equivalent |

### Next phase

Phase 3 — Physical Design: technology and device model selection based on the
requirements this phase established.
