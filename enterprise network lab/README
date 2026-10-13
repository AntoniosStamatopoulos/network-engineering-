# Enterprise Network Lab

A segmented enterprise network for a 42-user single site — designed top-down,
built in Cisco Packet Tracer, and tested until it either worked or the reason it
could not was written down.

Four departments isolated from one another, a guest network isolated by topology
rather than by rule, wireless that places each employee in their own department's
network, remote access with different reach for Finance and IT, and a management
plane only IT can see.

---

## The brief

A small company needs a network for its headquarters. Four departments, 42 users.
Visitors need Wi-Fi but must never reach anything internal. Finance and IT need
remote access, with IT getting more of it. Management of the equipment belongs to
IT alone. One edge router, one firewall, one core, two access switches — no
redundancy yet, but every single point of failure identified and tested.

The full requirements are in
[`docs/phase-1-analysis/`](docs/phase-1-analysis/).

---

## How it was built

Four phases, in order, each validated before the next began. This is the method —
analyse, then design logically, then design physically, then build and test — and
the project follows it rather than describing it.

### Phase 1 — Analysis

Before any design: who uses the network, what they reach, and how much of it
moves. Seven user communities, five data stores, and a traffic characterization
that records **denied flows as well as permitted ones** — because a flow that must
not exist is a requirement too.

This phase also produced the single-point-of-failure register, and closed with a
validation matrix checking that every requirement from the brief had been
addressed by an analysis artifact. Twenty-six requirements, all traceable.

→ [`docs/phase-1-analysis/`](docs/phase-1-analysis/)

### Phase 2 — Logical design

Segments became VLANs, VLANs got addresses, addresses got routes, and traffic got
a policy. Each sub-stage records the alternatives considered and what the chosen
option gives up.

The decisions that shaped everything else:

| Decision | Chosen | Rejected | Why |
|---|---|---|---|
| **Inter-VLAN routing** | Layer 3 core switch | Router-on-a-stick · routing on the firewall | Line-rate switching; keeps internal traffic off a device that has no reason to inspect it |
| **Guest path** | No gateway on the core | An ACL blocking guest traffic | A rule can be deleted. An absent gateway cannot |
| **Routing protocol** | Static | OSPF | One path exists, so there is nothing to choose between — with a written trigger for when that changes |
| **Addressing** | `10.0.<vlan>.0/24` | VLSM | `10.0.30.45` is a Sales host without a lookup table. Worth more than the addresses it wastes |
| **Wireless VLANs** | One SSID per department, WPA2-PSK | 802.1X with RADIUS | The better mechanism needs infrastructure the brief does not fund — recorded with what it costs and when to revisit |
| **VPN addressing** | One pool per group | A single shared pool | The address is the identity: it is what lets the firewall apply different rules to Finance and IT |
| **Management** | In-band, dedicated VLAN | Out-of-band network | One site, five devices. Recorded: a core failure takes management down with it |
| **Unused ports** | Parked in a VLAN with no gateway | Left in VLAN 1 | A free socket in a meeting room should reach nothing |

**A pattern runs through these.** Where isolation must be absolute, it is enforced
by something not existing — no gateway for guest, no gateway for unused ports, no
route inward on the edge router that avoids the firewall. Where access must be
selective, it is enforced by rules, because only a rule can distinguish one source
from another. The allocation is deliberate, and it is why the design survives a
deleted ACL.

→ [`docs/phase-2-logical-design/`](docs/phase-2-logical-design/)

### Phase 3 — Physical design and implementation

Equipment selected against the logical requirements, a connection map, and a port
assignment for every interface on every device — including the unused ones, which
are shut down and parked rather than left active.

Then the build. **Twenty deviations between design and implementation are
recorded**, each with its cause and its consequence. Three were errors in the
design that the build exposed; two were choices the design had deliberately left
open; fifteen are limitations of the simulator.

The principle applied throughout: **the design is not rewritten to match the
tool.** Where a capability was unavailable, the requirement stands and the
shortfall is recorded with what it costs.

→ [`docs/phase-3-physical-design/`](docs/phase-3-physical-design/) ·
[the deviation register](docs/phase-3-physical-design/3.6-implementation-deviations.md)

### Testing

Two campaigns. **Eighteen policy tests** covering every permit and deny the design
specifies, and **seven failure tests**, one per single point of failure, each run
against the same probe set so the results compare.

Policy was verified by sending traffic between real endpoints, not by reading the
configuration — necessary, because the platform does not display `access-group`
statements. A configuration review would have shown nothing at all.

→ [`docs/testing/`](docs/testing/)

---

## What the testing found

All eighteen policy tests pass. More usefully, the campaign found three faults
that were invisible in the configuration:

- **No access list was applied anywhere**, despite all five being correctly
  written. The network held a complete access policy and enforced none of it.
- **The inbound lists broke DHCP.** A DHCP request carries source `0.0.0.0` and
  destination `255.255.255.255`, so it matches no rule written in terms of VLAN
  subnets.
- **The management VLAN stayed reachable from user VLANs.** The rule was correct
  and the direction it was applied in was not: an outbound list does not filter
  traffic addressed to the interface it sits on.

The failure testing then **corrected the project's own SPOF analysis in four
places** — three impacts wider than predicted, one narrower. That analysis had
been written in Phase 1, before the routing, wireless and VPN designs existed, and
had assessed each device by the role it held at that moment. By the end, three
devices had quietly acquired a second role.

The lesson is recorded rather than quietly fixed: **an analysis whose conclusions
depend on the design must be rerun when the design is finished**, not merely
completed with results afterwards.

---

## What is not implemented

**SNMP does not exist on any device** in this environment, and **no device can log
a rule match** — the firewall implements no logging at all, and the switches
reject the `log` keyword on access list entries.

Together these mean the network enforces a policy it cannot observe. Every control
works and is demonstrable by test; producing a record of it after the fact would
not be possible. The captive portal and certificate-based VPN authentication are
likewise unavailable, and the guest DHCP service stores its pool but refuses to
start.

All of it is in the deviation register with the consequence spelled out. A gap
documented with its consequence is a known limitation; a gap left unrecorded is a
defect nobody finds until it matters.

---

## Evidence

Eight images, each with what it shows and what it proves, are in
[`screenshots/`](screenshots/) — the logical diagram, the built topology, and six
captures taken from the running network: guest isolation, wireless VLAN
placement, differentiated remote access, and central log collection.

---

## Repository

```
├── enterprise-network-lab.pkt      the network, openable in Packet Tracer
├── configs/                        running configuration of every device
├── docs/
│   ├── phase-1-analysis/           requirements · traffic · validation
│   ├── phase-2-logical-design/     VLANs · addressing · routing · zones · policy
│   ├── phase-3-physical-design/    devices · topology · ports · deviations
│   └── testing/                    test results · SPOF analysis
└── screenshots/
```

Start with `docs/testing/` to see whether it works, or
`docs/phase-2-logical-design/` to see why it is built this way.

**Device configurations** carry the role and the design reference in each header.
Credentials are removed. Comments mark the places where a setting would otherwise
read as arbitrary — why one trunk uses a different native VLAN, why a `deny` in a
NAT access list does not drop packets, why some objects are present but unused.

---

## Opening it

Requires **Cisco Packet Tracer 9.0 or later**. A file saved in 9.0 will not open
in 8.x.

No setup is needed — every device holds its configuration. Addresses follow one
rule, `10.0.<vlan>.x`, and public address space uses the RFC 5737 documentation
ranges throughout, so nothing here references a real routable address.
