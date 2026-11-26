# Platform Limitations (NM-16ESW on Dynamips)

The switches are c3745 routers with an NM-16ESW EtherSwitch module (IOS 12.4). They behave differently from a Catalyst switch in the ways below.

| Limitation | Effect on the design | Workaround |
|---|---|---|
| VLANs are lost when the project is closed: they live in `vlan.dat` in flash, which is not kept | Trunks come up but no VLAN traffic passes after reopening | `vtp file nvram:vlan.dat` in global config, then in GNS3 uncheck "Automatically delete NVRAM and disk files" (node → Configure → Memories and disks). Tested on HQ-ASW1: VLANs and port assignments survived a close and reopen. Fallback: paste [`configs/vlan_database.txt`](../configs/vlan_database.txt) |
| VLANs are created only in `vlan database` mode | VLAN setup is not part of `show running-config` | Keep the VLAN commands in `configs/vlan_database.txt` |
| `show vlan brief` returns `% Ambiguous command` | – | Use `show vlan-switch brief` (and `show mac-address-table` instead of `show mac address-table`) |
| The trunk allowed list must include VLAN 1 and 1002–1005 | Cannot prune VLAN 1 from the trunk | `switchport trunk allowed vlan 1,10,20,30,40,50,999,1002-1005` |
| No DTP: trunks show mode `on` with negotiation disabled | Switch-spoofing attacks cannot be tested in this lab | – |
| STP is per-VLAN 802.1D (not Rapid PVST+), and each VLAN uses its own bridge MAC | Slower convergence than real Catalyst switches; failover times in Phase 2 will be longer | Note this when comparing failover results with real hardware |
| The c3745 is a router, so `ip routing` is on by default | `ip default-gateway` only takes effect after `no ip routing` | `no ip routing` on HQ-ASW1 (Layer 2 only) |
| Onboard Fa0/0 on the c3745 came up half duplex | `%CDP-4-DUPLEX_MISMATCH` against HQ-R | `speed 100` and full duplex on both ends |
| The GNS3 default config includes `no cdp log mismatch duplex` | Duplex mismatches are not logged on the c3745 side | Check duplex with `show interfaces`, not only the logs |
