English | [ภาษาไทย](README.th.md)

# Office Network in GNS3: HQ + Branch

> Work in progress. Phase 1 (HQ LAN) is done; Phase 2 is next.

Two-site office network designed and built in GNS3 from a written client brief.
Full brief: [docs/requirements.pdf](docs/requirements.pdf) · IP plan: [docs/ip-plan.md](docs/ip-plan.md)

## Progress

| Phase | Scope | Status |
|---|---|---|
| 1 | HQ LAN: VLSM, VLANs, trunks, STP, inter-VLAN routing, DHCP | [Done](tests/phase1/README.md) |
| 2 | Resilience: DSW2, EtherChannel, HSRP, failover tests | Not started |
| 3 | WAN and internet: OSPF, branch, NAT/PAT | Not started |
| 4 | Security: ACLs, SSH, NTP, syslog | Not started |
| 5 | Troubleshooting tickets T1–T8 | Not started |

## Repository layout

```
docs/             requirements, design decisions, IP plan, platform limitations
configs/          VLAN setup and running-configs per phase (no passwords)
tests/            show outputs, pcaps and results summary per phase
troubleshooting/  one file per ticket
```
