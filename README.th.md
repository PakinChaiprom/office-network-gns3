[English](README.md) | ภาษาไทย

# ออกแบบเครือข่ายสำนักงานใน GNS3: HQ + สาขา

> กำลังทำ: Phase 1 (LAN สำนักงานใหญ่) เสร็จแล้ว ต่อไปคือ Phase 2

ออกแบบและสร้างเครือข่ายสองสาขาใน GNS3 ตามโจทย์ที่เขียนเป็นเอกสาร
โจทย์ฉบับเต็ม: [docs/requirements.pdf](docs/requirements.pdf) · IP plan: [docs/ip-plan.md](docs/ip-plan.md)

## ความคืบหน้า

| Phase | ขอบเขต | สถานะ |
|---|---|---|
| 1 | LAN สำนักงานใหญ่: VLSM, VLAN, trunk, STP, inter-VLAN routing, DHCP | [เสร็จ](tests/phase1/README.md) |
| 2 | ความทนทาน: DSW2, EtherChannel, HSRP, ทดสอบ failover | ยังไม่เริ่ม |
| 3 | WAN และอินเทอร์เน็ต: OSPF, สาขา, NAT/PAT | ยังไม่เริ่ม |
| 4 | ความปลอดภัย: ACL, SSH, NTP, syslog | ยังไม่เริ่ม |
| 5 | Ticket ฝึกแก้ปัญหา T1–T8 | ยังไม่เริ่ม |

## โครงสร้าง repository

```
docs/             โจทย์, เหตุผลการออกแบบ, IP plan, ข้อจำกัดของ emulator
configs/          คำสั่งสร้าง VLAN และ running-config ของแต่ละ phase (ไม่มีรหัสผ่าน)
tests/            output, pcap และสรุปผลตรวจรับของแต่ละ phase
troubleshooting/  หนึ่งไฟล์ต่อหนึ่ง ticket
```
