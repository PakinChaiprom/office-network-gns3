# IP Plan

Address space: `10.10.0.0/16` (internal), `203.0.113.0/29` (public, from ISP)

| Block | Range | Used for |
|---|---|---|
| HQ | `10.10.0.0/20` | HQ user and server subnets |
| Branch | `10.10.16.0/20` | Branch subnets |
| Infrastructure | `10.10.255.0/24` | Point-to-point links (`.0/25`), loopbacks (`.128/25`) |

Sizing rule: needed = hosts expected in 3 years + gateways (HQ: 3, branch: 1).
Subnets are allocated largest first so every network address falls on its block boundary.

## HQ subnets

Gateway convention: `.1` = HSRP virtual IP, `.2` = DSW1, `.3` = DSW2.
Phase 1 has no HSRP yet, so DSW1's SVIs use `.1` until Phase 2.

| VLAN | Name | Network | Gateway | Broadcast | Hosts in 3 yrs + gateways / Usable |
|---|---|---|---|---|---|
| 30 | OPS | `10.10.0.0/25` | `10.10.0.1` | `10.10.0.127` | 123 / 126 |
| 50 | GUEST | `10.10.0.128/26` | `10.10.0.129` | `10.10.0.191` | 53 / 62 |
| 20 | FIN | `10.10.0.192/26` | `10.10.0.193` | `10.10.0.255` | 38 / 62 |
| 40 | SRV | `10.10.1.0/27` | `10.10.1.1` | `10.10.1.31` | 15 / 30 |
| 10 | MGMT | `10.10.1.32/27` | `10.10.1.33` | `10.10.1.63` | 15 / 30 |
| 999 | NATIVE | – | – | – | no hosts, no SVI |
| 998 | PARKING | – | – | – | unused ports, no SVI |

## DHCP (server: R-HQ, relayed by the DSW SVIs)

| Pool | Excluded (gateways + static) | DHCP range | Addresses / Needed | DNS | Lease |
|---|---|---|---|---|---|
| OPS | `.1 – .3` | `.4 – .126` | 123 / 120 | `10.10.1.5`, `10.10.1.6` | default (1 day) |
| GUEST | `.129 – .138` | `.139 – .190` | 52 / 50 | `198.51.100.1` | 4 hours |
| FIN | `.193 – .202` | `.203 – .254` | 52 / 35 | `10.10.1.5`, `10.10.1.6` | default (1 day) |
| MGMT | `.33 – .42` | `.43 – .62` | 20 / IT PCs | `10.10.1.5`, `10.10.1.6` | default (1 day) |
| SRV | – | no pool | – | – | servers use static IPs |

## SRV static addresses (`10.10.1.0/27`)

| Address | Used by |
|---|---|
| `.1` / `.2` / `.3` | Gateway VIP / DSW1 / DSW2 |
| `.4` | SRV-WEB (static NAT to `203.0.113.3`) |
| `.5` | DNS1 (internal) |
| `.6` | DNS2 (internal) |
| `.7 – .30` | Other servers |

## Branch subnets

Gateway convention: `.1` = R-BR subinterface

| VLAN | Name | Network | Gateway | Host range | Broadcast | Needed / Usable |
|---|---|---|---|---|---|---|
| 60 | BR | `10.10.16.0/26` | `10.10.16.1` | `.2 – .62` | `10.10.16.63` | 31 / 62 |
| 10 | BR-MGMT | `10.10.16.64/28` | `10.10.16.65` | `.66 – .78` | `10.10.16.79` | 5 / 14 |

BR-MGMT needed = BR-SW + gateway + 3 spare.
The branch DHCP pool lives on R-BR (Phase 3) and uses the same internal DNS servers as HQ.

## Point-to-point links (`10.10.255.0/25`)

| Link | Network | Side A | Side B |
|---|---|---|---|
| R-HQ – DSW1 | `10.10.255.0/30` | R-HQ `.1` | DSW1 `.2` |
| R-HQ – DSW2 | `10.10.255.4/30` | R-HQ `.5` | DSW2 `.6` |
| R-HQ – R-BR (serial) | `10.10.255.8/30` | R-HQ `.9` | R-BR `.10` |

## Loopbacks (`10.10.255.128/25`)

Used as OSPF router-id and SSH target.

| Device | Loopback0 |
|---|---|
| R-HQ | `10.10.255.128/32` |
| DSW1 | `10.10.255.129/32` |
| DSW2 | `10.10.255.130/32` |
| R-BR | `10.10.255.131/32` |

## Layer 2 switch management IPs

| Device | VLAN | IP | Default gateway |
|---|---|---|---|
| ASW1 | 10 | `10.10.1.36` | `10.10.1.33` |
| BR-SW | 10 | `10.10.16.66` | `10.10.16.65` |

## Public addresses (`203.0.113.0/29`)

| Address | Used by |
|---|---|
| `203.0.113.1` | ISP router (default gateway) |
| `203.0.113.2` | R-HQ WAN interface, also PAT for all inside users |
| `203.0.113.3` | Static NAT for SRV-WEB (`10.10.1.4`) |
| `203.0.113.4 – .6` | Spare |

## Design notes

- Infrastructure sits outside both site blocks so links and loopbacks are easy to spot in `show ip route`. Trade-off: they are not covered by the site summaries and are advertised individually.
- MGMT is split per site because the HQ–branch WAN link is routed (Layer 3), so a VLAN cannot stretch across it.
- OPS excludes only the three gateway addresses. Excluding `.1 – .10` like the other pools would leave 116 addresses for 120 expected users. OPS has 3 addresses of headroom left after 3 years; growth beyond that means moving it to a /24.
- GUEST uses an external DNS server so the Phase 4 ACL can block guests from all of `10.10.0.0/16` with no exceptions.
- GUEST has a 4-hour lease because guest devices come and go; a 1-day lease would fill the pool with addresses nobody is using.
