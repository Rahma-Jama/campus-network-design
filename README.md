# Enterprise Campus LAN — Cisco Packet Tracer

A simulated three-tier enterprise campus network built to practice translating CCNA theory into a working, documented design — VLAN segmentation, redundant gateways, dynamic routing, centralised DHCP, and a simulated internet edge.


---

## Topology Overview

**Core** — 2 routers, fully meshed to each other and to both distribution switches, providing redundant paths with no single point of failure.

**Distribution** — 2 multilayer switches, meshed to the core and to each other, hosting all VLAN SVIs. Gateway redundancy is handled with HSRP, with active roles split across both switches per VLAN so traffic load isn't concentrated on one device.

**Access** — 3 access switches, each with dual uplinks (one to each distribution switch) so a single distribution switch failure doesn't isolate an access switch.

**WAN edge** — A separate router simulating the ISP, connected to the core routers, running OSPF and hosting a loopback (`8.8.8.8`) to act as a pingable "internet" target for testing NAT and reachability, since Packet Tracer's Cloud device doesn't provide real internet access.

**End devices** — PCs across three VLANs (10, 20, 30), plus 2 servers.

*(Full topology diagrams are in `/topology`.)*

---

## VLAN / Addressing Summary

| VLAN | Purpose | Subnet | Gateway (HSRP virtual) |
|---|---|---|---|
| 10 | Access — PCs | 192.168.1.0/28 | 192.168.1.14 |
| 20 | Access — PCs | 192.168.1.16/28 | 192.168.1.30 |
| 30 | Servers | 192.168.1.32/29 | 192.168.1.38 |

Point-to-point router/switch links and WAN links use separate address space, kept deliberately out of the VLAN ranges to avoid overlap.

---

## Concepts Demonstrated

- **VLAN segmentation and 802.1Q trunking** across access and distribution layers
- **Redundant physical topology** — meshed core/distribution, no single point of failure
- **Spanning Tree Protocol** — manual root bridge / bridge ID configuration to control topology deterministically rather than relying on default election
- **HSRP (First Hop Redundancy Protocol)** — virtual gateway IPs per VLAN, with active roles deliberately split across both distribution switches for basic load distribution
- **OSPF (single area)** — dynamic routing between core, distribution, and the simulated ISP router
- **Centralized DHCP** — pool configuration, exclusions, and `ip helper-address` relay across VLAN boundaries
- **NAT** — translating internal VLAN traffic for the simulated WAN edge
- **IP subnetting design** — right-sized `/28` and `/29` blocks per VLAN based on actual host counts, with room to grow
- **Structured documentation** — a timestamped build/troubleshooting changelog (`/changelog.md`) capturing real failures and root-cause fixes, not just a list of finished features

---

## Troubleshooting Highlights

These are the moments I think say more about my understanding than the finished diagram does — full detail is in the changelog.

- **Missing trunk between access and distribution** — PC1 couldn't reach its gateway; traced to a trunk that existed on one side of the link but not the other.
- **STP broadcast storm on VLAN 20** — all ports on a VLAN were forwarding simultaneously due to an unmanaged Layer 2 loop across the dual-homed access uplinks; fixed by explicitly setting root bridge priority instead of leaving election to chance.
- **Duplicate SVI IP conflict** — both distribution switches were initially configured with the *same* IP on the same VLAN interface, instead of unique real IPs plus a shared HSRP virtual IP. Caught and corrected once I understood how HSRP actually works, rather than assuming a shared IP was the mechanism.
- **DHCP configured on the wrong device** — an access switch was running a DHCP pool with no actual route to the requesting subnet, producing addresses from the wrong range. Root-caused and moved DHCP service to a router with proper reachability across VLANs.
- **DHCP renewal silently failing after adding OSPF** — new routes existed, but `service dhcp` hadn't been enabled on the router, so it wasn't actually acting as a DHCP server despite having a correctly configured pool.


---

## Lessons Learned / What I'd Do Differently

- **Document while building, not after.** Early parts of this project were reconstructed from memory and `show running-config` output rather than logged in the moment — the later sections of the changelog, written live as I worked, are noticeably more detailed and useful than the earlier ones.


---

## Repository Structure

```
/campus-network-design/
  README.md          <- this file
  change log.pdf         <- full timestamped build/troubleshooting log
  /topology/            <- topology diagrams
  /configs/              <- full device running-configs (plain text)
  campus-lan-project.pkt           <- Packet Tracer file
```
