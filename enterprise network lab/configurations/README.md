# Device configurations

The running configuration of every device in the lab, captured from the device
itself with `show running-config` after the network was built and tested. These
are not hand-written examples: they are what the working network actually ran.

## The files

| File | Device | Role in the design |
|------|--------|--------------------|
| `config-core-sw1.txt` | CORE-SW1 | Layer 3 switch, collapsed core. Holds the SVIs and therefore every internal gateway, routes between VLANs, applies the inter-VLAN filtering, and serves DHCP. Carries the Guest VLAN as Layer 2 transit with no SVI of its own. |
| `config-acc-sw1.txt` | ACC-SW1 | Access switch. Assigns endpoints to VLANs, trunks to the core, hardens the native VLAN. Also serves LAP-1 and the WLC attachment. |
| `config-acc-sw2.txt` | ACC-SW2 | Second access switch, same role. Users are distributed across both switches rather than grouped by department, so no department is fully lost if one switch fails. |
| `config-edge-r1.txt` | EDGE-R1 | Edge router. Terminates the ISP link, performs PAT, and terminates the remote-access VPN. Deliberately holds no route into the internal network except through the firewall. |
| `config-fw-1.txt` | FW-1 | Firewall. The single policy enforcement point: three zones at security levels 0, 50 and 100, the Guest gateway, and all Internet-bound policy. |
| `config-isp-r.txt` | ISP-R | Simulated provider router. Exists only to give the lab something on the far side of the WAN link and a reachable external host; it is not part of the design and is not hardened. |

**There is no configuration file for the WLC or the access points.** The wireless
controller in this platform is configured entirely through its web interface and
has no CLI export, and the lightweight access points hold no configuration of
their own — they take it from the controller. The wireless design is documented in
sub-stage 2.8 and the resulting state is shown in the screenshots folder.

## How to read them

Each file opens with a short header naming the device, its role, and the design
sub-stages it implements, so a configuration can be checked against the document
that specified it.

Comments were added only where a setting would otherwise read as arbitrary — a
native VLAN that is not 1, an ACL that denies in order to exempt rather than to
block, a deny-based NAT exemption list. Settings that explain themselves were left
alone. **The comments exist in these text files only**, not in the devices.

Suggested order if you are reading the configurations rather than looking something
up: CORE-SW1 first, because it holds the addressing and the internal policy; then
an access switch; then the firewall and the edge router together, since the
boundary only makes sense as a pair.

## Credentials

Passwords, enable secrets and VPN group keys have been replaced with `<REMOVED>`.
Nothing else was edited — no line was reordered, removed or tidied, so the
structure of each file is the device's own output.

The Packet Tracer file in the repository root contains the live configuration,
which means every credential in it is visible to anyone who opens it. That is
normal for a lab and is the reason a lab should only ever use credentials that
exist nowhere else.
