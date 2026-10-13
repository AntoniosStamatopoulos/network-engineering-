# Enterprise Network Lab — Requirements Validation Matrix

## Purpose

This document closes the analysis phase (Phase 1). It does not introduce new
design; it **validates** that every requirement from the Project Requirements
Sheet has been addressed by the analysis deliverables (User Communities, Data
Stores, Device List, and Traffic Characterization), and it flags any item that
belongs to a later deliverable rather than to this phase.

> **Note on scope.** This matrix asks whether each requirement was **addressed by
> the analysis**, not whether it was ultimately **implemented**. The two are
> different questions and the second belongs to 3.6 and the test results. A row
> marked ✅ here means the analysis covered the requirement; it does not claim the
> finished network delivers it. Section 4 records where the difference matters.

**Legend**

| Symbol | Meaning |
|--------|---------|
| ✅ | Covered in Phase 1 analysis |
| ⏳ | Deliberately deferred to Phase 2 (out of scope by design) |
| ⚠️ | Not a gap — points to an upcoming deliverable (firewall policy / SPOF doc) |

---

## 1. Validation Table

| # | Requirement (source) | Covered in | Status |
|---|----------------------|------------|--------|
| §2 | Four departments (HR 8, Finance 8, Sales 20, IT 6) | UC1–UC4 (Traffic §1) + Device List | ✅ |
| §2 | Addressing supports future growth | Device List (scalability) | ✅ |
| §3 | DNS Server | DS1 (Traffic §2) + flows §3.1 | ✅ |
| §3 | Web Server | DS2 (Traffic §2) + flows §3.1 | ✅ |
| §3 | Servers in a separate segment | DS1–DS3 in server segment | ✅ |
| §3 | File Server (future) | DS5 (flagged future) | ⏳ |
| §4 | Guest Wi-Fi → Internet ALLOW | Flow §3.2 | ✅ |
| §4 | Guest → all internal DENY | Denied flows §4.2 | ✅ |
| §5 | Branch Office | — | ⏳ |
| §6 | HR/Finance/Sales mutual isolation | Denied flows §4.1 | ✅ |
| §6 | IT → internal networks ALLOW | Flows §3.3 | ✅ |
| §6 | HR/Finance/Sales → Servers ALLOW | Flows §3.1 | ✅ |
| §7 | Each department → Internet ALLOW | Flows §3.2 | ✅ |
| §7 | Internet → internal DENY (default) | See Note #1 | ⚠️ |
| §7 | Firewall / Edge Router / ISP | Device List | ✅ |
| §8 | Corporate Wi-Fi per department | UC1–UC4 (access method) | ✅ |
| §8 | Guest Wi-Fi isolated | Flows §3.2 + Denied §4.2 | ✅ |
| §9 | VPN only for Finance & IT | UC6, UC7 + flows §3.6 | ✅ |
| §9 | Finance VPN restricted scope | Flows §3.6 (Part A) | ✅ |
| §9 | IT VPN broad scope | Flows §3.6 (Part B) | ✅ |
| §10 | Management restricted to IT | Flows §3.4 | ✅ |
| §10 | HR/Finance/Sales/Guest → mgmt DENY | See Note #2 | ⚠️ |
| §10 | Syslog + SNMP monitoring | Flows §3.5 + DS3 | ✅ |
| §11 | 1 Edge Router, 1 Firewall, 1 Core, 2 Access | Device List | ✅ |
| §11 | Documentation of SPOFs | See Note #3 | ⚠️ |
| §12 | Scalability (VLANs, users, sites) | Device List (scalability) | ✅ |

**26 requirements.** 21 covered, 2 deferred, 3 pointing forward.

---

## 2. Notes on ⚠️ Items

These items are **not gaps in the analysis**. They are requirements that, by their
nature, are satisfied by a deliverable produced *after* traffic characterization.
They are recorded here so nothing is lost in the handover to the design phase.

**Note #1 — Internet → internal DENY by default (§7).**
This is a firewall rule, not a traffic flow. A denied flow carries no traffic to
characterize, so it cannot appear in the traffic tables. It belongs to the
**Firewall Policy / ACL design** deliverable.

*Delivered* as firewall rules 9 and 10 in 2.7.3.

**Note #2 — Non-IT networks → management interfaces DENY (§10).**
Same reasoning: this is a security policy statement, partially captured already in
the denied-flows section, but its explicit enforcement (blocking HR/Finance/Sales/
Guest from device management interfaces) belongs to the **Firewall / ACL design**.

*Delivered* as core ACL rules 1 and 2 in 2.7.2, with an additional inbound deny
required at implementation (2.7.2a). Verified by test — T13.

**Note #3 — Single Point of Failure documentation (§11).**
The requirements explicitly ask for SPOFs to be identified and documented. This is
a small standalone deliverable (SPOF analysis), not a traffic flow or a device
entry. ~~**To verify:** confirm whether this already exists in the Device List; if
not, it is the one remaining Phase 1 item to produce before design begins.~~

*Resolved.* The SPOF analysis exists as a standalone document. It registers seven
single points of failure with their impact, scope and mitigation. Its Test Result
column, left open at Phase 1 by design, was completed after implementation.

**All three ⚠️ items have been delivered.** None was a gap, and the handover this
section exists to protect worked as intended.

---

## 3. Phase 1 Closure Summary

| Category | Count | Notes |
|----------|------:|-------|
| Core requirements covered (✅) | 21 | Fully addressed by Phase 1 analysis |
| Deferred to Phase 2 (⏳) | 2 | File Server, Branch Office — out of scope by design |
| Pointing to next deliverable (⚠️) | 3 | 2 × firewall policy, 1 × SPOF doc |
| **Total** | **26** | |

**Conclusion.** The Phase 1 analysis is **complete and internally consistent**.
All core HQ requirements are traceable to a concrete analysis artifact. The three
⚠️ items are correctly positioned as inputs to the firewall policy design and the
SPOF documentation, not as omissions. Phase 1 is formally closed and the project
proceeded to the design phase (logical topology → VLAN plan → IP addressing →
security zones → firewall policy).

---

## 4. Phase 1 in Retrospect

Added after implementation. This section does not change the matrix; it records
what the closure check could and could not establish.

### What this matrix verified correctly

Every requirement was traceable to an analysis artifact, and every ⚠️ item was
delivered by the phase it pointed to. No requirement was lost in the handover from
analysis to design, which is what this document exists to prevent. On its own
terms it worked.

### What it could not verify

**This matrix checks coverage, not correctness.** It asks *was this requirement
addressed?* and cannot ask *was the assessment right?* The distinction turned out
to matter in one place.

The SPOF analysis is ticked here as delivered, and it was. But testing later showed
four of its seven entries to be wrong — three impacts wider than predicted, one
narrower. The cause was not carelessness: the analysis was written during Phase 1,
before the routing, wireless, VPN and management designs existed, so it assessed
each device by the role it held at that moment. By the end of Phase 2, three
devices had acquired roles the analysis never saw.

A coverage check cannot detect that. A requirement can be fully addressed by an
artifact that is itself out of date.

### The lesson for the method

**An analysis deliverable whose conclusions depend on the design should be
re-validated at the close of the design phase, not only at the close of the
analysis phase.** The SPOF analysis is the clear case; the traffic characterization
would be worth the same second look, since traffic volumes depend on decisions
taken after it was written.

This is a gap in the top-down method as applied here rather than in its execution.
The method validates each phase against the requirements as it closes. It does not
prescribe revisiting an earlier phase when a later one changes the facts the
earlier one assumed.

### Where the implementation diverged

For the record, three requirements ticked ✅ above were not fully delivered by the
finished network. The analysis addressed each correctly; the environment could not
implement them.

| Requirement | Status in the build | Recorded in |
|-------------|---------------------|-------------|
| §10 Syslog + SNMP monitoring | SNMP not implemented on any device; device Syslog only, no policy logging | 3.6 — D15, D16 |
| §4 Guest Wi-Fi isolated *(the captive portal element of §8's Guest controls)* | Isolation delivered and verified; the portal is not implemented | 3.6 — D9 |
| §9 VPN authentication | Delivered with a shared group key rather than per-user certificates | 3.6 — D20 |

These are implementation outcomes, not validation failures. The matrix above
remains correct as a record of Phase 1.

---

## Amendment Record

| Section | Change | Reason |
|---------|--------|--------|
| Purpose | Scope note added distinguishing *addressed by the analysis* from *implemented* | The matrix is now read alongside 3.6, and the two answer different questions |
| 1 | Row count stated under the table | — |
| 2 | Delivery recorded for Notes #1 and #2 | Both deliverables now exist |
| 2 | Note #3 resolved; the "to verify" instruction struck through rather than deleted | The SPOF analysis exists and has been tested |
| 3 | **Counts corrected**: 21 ✅ and 2 ⏳, not 20 and 3. The ⏳ note already listed two items, so the count contradicted its own description | Arithmetic error |
| 3 | Total row added | Makes the arithmetic checkable at a glance |
| 3 | Conclusion moved to past tense; the conditional on Note #3 removed | The condition is satisfied |
| **4** | **New section.** What the closure check verified, what it structurally could not, the methodological lesson, and the three requirements the implementation did not fully deliver | Post-implementation review |
