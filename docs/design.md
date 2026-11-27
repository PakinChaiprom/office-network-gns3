English | [ภาษาไทย](design.th.md)

# Design Decisions

Each decision links to the evidence that backs it. Numbers come from [ip-plan.md](ip-plan.md) and [tests/phase1](../tests/phase1/README.md).

## Phase 1: HQ LAN

### 1. Subnetting

Address space from the brief: `10.10.0.0/16` (private) and `203.0.113.0/29` (public).

**1.1 Three blocks first.** The private range is split into three blocks before any subnet is made: HQ (`10.10.0.0/20`), branch (`10.10.16.0/20`) and infrastructure (`10.10.255.0/24`). This makes the network easier to manage: an address tells you at once which part of the network it belongs to, and each block can be summarized as one route. HQ-R already reaches every HQ subnet with a single static route, `10.10.0.0/20` ([HQ-R config](../configs/phase1/HQ-R.txt)).

**1.2 VLSM sized for 3 years.** Each block is divided per department with VLSM, largest subnet first, sized by the number of hosts expected in 3 years rather than today. Every HQ subnet reserves 3 addresses for gateways (HSRP virtual IP, DSW1, DSW2) and the branch reserves 1; most subnets also reserve 7 more for static devices added later. After allocation every department has more than 10 addresses of headroom, except OPS, which fits its 3-year estimate exactly (3 spare). If OPS grows beyond that, it can move to a /24: HQ has only used `10.10.0.0 – 10.10.1.63` (320 addresses) of its /20 (2^12 = 4,096 addresses).

### 2. VLANs

**2.1 Numbering.** A VLAN number means the same function at every site, at HQ or the branch (for example, 10 = MGMT at both), so there is no confusion during changes and maintenance. Numbers go up in steps of 10, leaving room for related VLANs later (for example, 21 for FIN's IP phones).

**2.2 One VLAN = one subnet = one broadcast domain.** A broadcast in one VLAN never reaches another VLAN without passing through the gateway, so traffic between departments can be controlled with ACLs at one point. In the Phase 1 test, FIN's ARP never reached the OPS port ([04_arp_ops.pcapng](../tests/phase1/04_arp_ops.pcapng)).

**2.3 Inter-VLAN routing on the Layer 3 switch.** On real hardware a Layer 3 switch routes between VLANs in hardware, without relying on the CPU of an external router. In this emulator the benefit shows up as less traffic through the router when there are many users. In the earlier [VLAN lab](https://github.com/PakinChaiprom/cisco-routing-labs/tree/main/vlan-intervlan), one ping through router-on-a-stick put 4 frames on the router link, while with an SVI the router link carried none.

**2.4 Native VLAN 999.** The trunk's native VLAN is one that no access port belongs to (VLAN 999). This blocks double-tagging VLAN hopping, which only works when the attacker sits in the native VLAN.

**2.5 Parking VLAN 998.** Unused ports are shut down and placed in VLAN 998. If one is enabled by mistake, it lands in a VLAN with no hosts and no gateway instead of VLAN 1.

### 3. STP

**3.1 DSW1 is the root.** In Phase 1, DSW1 is the root bridge for every VLAN, with priority set to 4096 by hand. DSW1 holds the gateways, so traffic converges there anyway. Without a set priority, the root is the switch with the lowest MAC address, which could be any switch, such as an access switch, and traffic would then pass through a point we did not choose ([05_stp_dsw1.txt](../tests/phase1/05_stp_dsw1.txt)). In Phase 2 the root role will be split with DSW2.

### 4. DHCP

**4.1 One DHCP server on HQ-R.** All HQ pools live on one router, which is easier to manage and avoids a problem coming in Phase 2. When DSW2 is added, pools on DSW1 would fail together with DSW1; on HQ-R, DHCP keeps working through DSW2 if DSW1 fails.

**4.2 Relay to HQ-R's loopback.** Every SVI that needs DHCP has `ip helper-address`, because a router does not forward broadcasts. The helper points to HQ-R's loopback rather than a physical interface: in Phase 2, DSW2 adds a second path to HQ-R, and an interface address would stop working if that one interface had a problem, even though the other path was still up. The relay was confirmed in the capture: the Discover goes to the loopback, and HQ-R picks the pool from the relay address, giaddr ([02_dora.pcapng](../tests/phase1/02_dora.pcapng)).

**4.3 Exclusions before pools.** In each subnet, the addresses that must not be handed out are excluded before the pool is created, to prevent duplicate addresses. Changing `excluded-address` later does not remove existing bindings: any binding inside a newly excluded range must be cleared with `clear ip dhcp binding`. Both lessons came from a real conflict in Phase 1 ([Problems found](../tests/phase1/README.md#problems-found-and-fixed)).

**4.4 GUEST settings.** GUEST uses an external DNS server, so a later ACL can block guests from all internal addresses with no exceptions. GUEST has a 4-hour lease because guest devices come and go often; a long lease would fill the pool with addresses held by devices that have already left.

## Later phases (to be written)

- Why the branch gets its own DHCP pool (Phase 3)
- Why the STP root and HSRP active must be the same switch (Phase 2)
- Why OSPF instead of static routes or RIP (Phase 3)
- Why the ACLs sit where they do, and why on both DSWs (Phase 4)
- What would change on real hardware (Rapid PVST+, LACP, separate firewall, ...)
