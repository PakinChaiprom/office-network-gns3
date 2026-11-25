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

Gateway convention: `.1` = HSRP virtual IP, `.2` = DSW1, `.3` = DSW2

| VLAN | Name | Network | Gateway (VIP) | Host range | Broadcast | Needed / Usable |
|---|---|---|---|---|---|---|
| 30 | OPS | `10.10.0.0/25` | `10.10.0.1` | `.4 – .126` | `10.10.0.127` | 123 / 126 |
| 50 | GUEST | `10.10.0.128/26` | `10.10.0.129` | `.132 – .190` | `10.10.0.191` | 53 / 62 |
| 20 | FIN | `10.10.0.192/26` | `10.10.0.193` | `.196 – .254` | `10.10.0.255` | 38 / 62 |
| 40 | SRV | `10.10.1.0/27` | `10.10.1.1` | `.4 – .30` | `10.10.1.31` | 15 / 30 |
| 10 | MGMT | `10.10.1.32/27` | `10.10.1.33` | `.36 – .62` | `10.10.1.63` | 15 / 30 |
| 999 | NATIVE | – | – | – | – | no hosts |
| 998 | PARKING | – | – | – | – | unused ports |

## Branch subnets

Gateway convention: `.1` = R-BR subinterface

| VLAN | Name | Network | Gateway | Host range | Broadcast | Needed / Usable |
|---|---|---|---|---|---|---|
| ?? | BR | `10.10.16.0/26` | `10.10.16.1` | `.2 – .62` | `10.10.16.63` | 31 / 62 |
| 10 | BR-MGMT | `??/28` | `??` | `??` | `??` | ?? / 14 |

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
| ASW1 | 10 | `??` | `10.10.1.33` |
| BR-SW | 10 | `??` | `??` |

## Public addresses (`203.0.113.0/29`)

| Address | Used by |
|---|---|
| `203.0.113.1` | ISP router (default gateway) |
| `203.0.113.2` | R-HQ WAN interface, also PAT for all inside users |
| `203.0.113.3` | Static NAT for SRV-WEB |
| `203.0.113.4 – .6` | Spare |

## Design notes

- Infrastructure sits outside both site blocks so links and loopbacks are easy to spot in `show ip route`. Trade-off: they are not covered by the site summaries and are advertised individually.
- MGMT is split per site because the HQ–branch WAN link is routed (Layer 3), so a VLAN cannot stretch across it.
