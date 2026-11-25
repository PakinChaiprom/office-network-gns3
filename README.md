English | [ภาษาไทย](README.th.md)

# Office Network in GNS3: HQ + Branch

> Work in progress. Phase 1 (HQ LAN) in progress.

Two-site office network designed and built in GNS3 from a written client brief.
Full brief: [docs/requirements.pdf](docs/requirements.pdf)

## Progress

| Phase | Scope | Status |
|---|---|---|
| 1 | HQ LAN: VLSM, VLANs, trunks, STP, inter-VLAN routing, DHCP | In progress |
| 2 | Resilience: DSW2, EtherChannel, HSRP, failover tests | Not started |
| 3 | WAN and internet: OSPF, branch, NAT/PAT | Not started |
| 4 | Security: ACLs, SSH, NTP, syslog | Not started |
| 5 | Troubleshooting tickets T1–T8 | Not started |

## Repository layout

```
docs/             requirements, design decisions, IP plan, platform limitations
configs/          VLAN setup and final running-configs (passwords removed)
tests/            show outputs and pcaps per phase, failover results
troubleshooting/  one file per ticket
```
