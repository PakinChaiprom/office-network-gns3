[English](design.md) | ภาษาไทย

# เหตุผลการออกแบบ

ทุกข้อมีลิงก์ไปที่หลักฐาน ตัวเลขมาจาก [ip-plan.md](ip-plan.md) และ [tests/phase1](../tests/phase1/README.md)

## Phase 1: LAN สำนักงานใหญ่

### 1. การแบ่ง Subnet

จาก private IP ที่ได้รับมาคือ `10.10.0.0/16` และ public IP คือ `203.0.113.0/29`

**1.1** เริ่มต้นได้ทำการแบ่ง private IP เป็น 3 block คือ HQ (`10.10.0.0/20`), BR (`10.10.16.0/20`) และ Infrastructure (`10.10.255.0/24`) เหตุผลที่แบ่งเป็น 3 block ใหญ่ก่อนเพื่อให้จัดการได้ง่าย เช่น ดู IP แล้วรู้ทันทีว่าอยู่ส่วนไหน และสามารถสรุป route ของแต่ละ block ได้ (HQ-R ใช้ static route เดียวคือ `10.10.0.0/20` ไปหาทุก subnet ของ HQ: [config HQ-R](../configs/phase1/HQ-R.txt))

**1.2** จากนั้นทำการแบ่ง IP ของ block หลักเป็น subnet ย่อยตามแต่ละแผนก โดยทำแบบ VLSM เริ่มจากแผนกที่ต้องการ IP เยอะที่สุดก่อน และที่สำคัญต้องดูที่การคาดการณ์จำนวน IP ที่ต้องใช้ในอนาคต ไม่ใช่ความต้องการปัจจุบัน และเผื่อ IP ว่างแต่ละตำแหน่งไว้สำหรับ gateway HQ 3 IP, BR 1 IP ตามจำนวนอุปกรณ์ และอีก 7 IP สำหรับอุปกรณ์ต่าง ๆ ในอนาคต หลังจากแบ่งเสร็จแต่ละแผนกจะเหลือที่เผื่อขยาย > 10 ยกเว้น OPS ที่พอดีกับจำนวนที่คาดไว้ (เหลือ 3) เมื่อจำนวนเกินที่คาดไว้ในอนาคตจะทำการขยายเป็น /24 ได้ เพราะ IP ของ HQ ใช้ไปแค่ `10.10.0.0 – 10.10.1.63` (320 IP) จาก /20 ที่มีทั้งหมด 2^12 = 4,096 IP

### 2. VLAN

**2.1** การตั้ง VLAN จะตั้งโดยเลขเดียวกันหมายถึงหน้าที่เดียวกันทุกส่วน ไม่ว่าจะอยู่ที่ HQ หรือ BR (เช่น 10 = MGMT ทั้งสองที่) ทำให้ไม่เกิดความสับสนเวลาแก้ไข/บำรุงรักษา และจะตั้งเลข VLAN โดยเว้นทีละ 10 เพื่อเหลือที่ให้ VLAN ที่จะเกี่ยวข้องกับ VLAN นั้น ๆ ในอนาคต

**2.2** ยึดไว้เสมอว่า 1 VLAN = 1 Subnet = 1 broadcast domain เพื่อความสะอาดและปลอดภัยของข้อมูล เช่น broadcast ของ VLAN หนึ่งจะไม่ไปที่ VLAN อื่นโดยไม่ผ่าน gateway ทำให้สามารถตั้ง ACL ไว้ที่จุดเดียวได้ (ผลทดสอบ: ARP ของ FIN ไม่ไปถึง port ของ OPS: [04_arp_ops.pcapng](../tests/phase1/04_arp_ops.pcapng))

**2.3** ทำ inter-VLAN routing ที่ L3 switch เพื่อให้สามารถส่งข้อมูลระหว่าง VLAN ด้วย hardware ไม่จำเป็นต้องพึ่งพา CPU ของ router ภายนอก สำหรับ emulator การใช้ L3 switch ในการทำ inter-VLAN routing ช่วยลดปริมาณ traffic ที่ผ่าน router เมื่อมีผู้ใช้เยอะ (ใน [lab VLAN](https://github.com/PakinChaiprom/cisco-routing-labs/tree/main/vlan-intervlan) แบบ router-on-a-stick ping 1 ครั้งมี 4 frame บนสายไป router ส่วนแบบ SVI ไม่มีเลย)

**2.4** ตั้ง native VLAN ให้เป็น VLAN ที่ไม่มี access port ไหนเข้าได้บนสาย trunk ในที่นี้ใช้ VLAN 999 เพื่อกันการทำ VLAN hopping แบบ double tagging

**2.5** ตั้ง VLAN ไว้สำหรับ interface ที่ไม่มีการใช้งาน เพื่อถึงแม้ port ถูกเปิดใช้งานโดยไม่ตั้งใจ ก็จะอยู่ใน VLAN ที่ไม่มีการใช้งาน และไม่ตกไปที่ VLAN 1 ในที่นี้ใช้ VLAN 998

### 3. STP

**3.1** ใน Phase 1 ตั้งให้ DSW1 เป็น root สำหรับทุก VLAN โดยการ set priority เป็น 4096 เพราะ DSW1 เป็นที่อยู่ของ gateway ถ้าไม่ set priority root จะถูกเลือกจาก MAC address ที่ต่ำที่สุด ซึ่งอาจเป็น switch ตัวไหนก็ได้ เช่น access switch ทำให้ traffic ไปผ่านจุดที่เราไม่ต้องการ ([05_stp_dsw1.txt](../tests/phase1/05_stp_dsw1.txt)) ใน Phase 2 จะแบ่งหน้าที่ root กับ DSW2

### 4. DHCP

**4.1** เลือกวาง DHCP pool ไว้ที่ router เดียวทำให้จัดการได้ง่าย และกันปัญหาที่อาจเกิดขึ้น เช่น ใน Phase 2 เมื่อมีการต่อ DSW2 เข้ามาเพิ่ม ถ้าวาง DHCP ไว้ที่ DSW1 ถ้า DSW1 ล่ม DHCP จะล่มไปด้วย แต่ถ้าวางที่ router หลัก ถึง DSW1 ล่มก็ยังมี DSW2 ในการส่งต่อข้อมูล

**4.2** มี helper-address ชี้ไปที่ loopback ของ DHCP server บนทุก SVI ที่ต้องการ DHCP เนื่องจาก router ไม่ forward broadcast packet เหตุผลที่ helper-address ชี้ไปที่ loopback เพราะถ้าชี้ไปที่ interface ของ server ใน Phase 2 จะมีการต่อ DSW2 เข้ามาเพิ่ม และถ้าใช้ interface เดียวในการส่งข้อมูล ถ้ามีปัญหาที่ interface นั้นจะไม่สามารถไปใช้ทางสำรองที่เหลืออยู่ได้ ([02_dora.pcapng](../tests/phase1/02_dora.pcapng))

**4.3** ก่อนการสร้าง pool IP ของแต่ละแผนกต้อง exclude IP ที่ไม่ต้องการให้แจกจ่ายของ subnet นั้น ๆ ก่อน เพื่อกันการใช้ IP ซ้ำกัน และการแก้ excluded-address ภายหลังจะไม่ลบ binding ที่มีอยู่แล้ว binding ที่อยู่ในช่วงที่เพิ่ง exclude ต้องลบออกด้วย `clear ip dhcp binding` ([ปัญหาที่เจอใน Phase 1](../tests/phase1/README.md#problems-found-and-fixed))

**4.4** Guest ใช้ DNS ภายนอก เพื่อให้สามารถเขียน ACL block ได้ทั้งหมดในอนาคต และใช้ lease 4 ชม. เพราะ Guest มีการเข้าออกบ่อย เพื่อกันปัญหา IP หมด pool เนื่องจากมีการเข้าใช้งานค้างในระบบ

## Phase ต่อไป (ยังไม่ได้เขียน)

- ทำไมสาขามี DHCP pool ของตัวเอง (Phase 3)
- ทำไม STP root กับ HSRP active ต้องเป็น switch ตัวเดียวกัน (Phase 2)
- ทำไมใช้ OSPF ไม่ใช้ static route หรือ RIP (Phase 3)
- ทำไมวาง ACL ตรงนั้น และทำไมต้องมีบน DSW ทั้งสองตัว (Phase 4)
- ถ้าใช้อุปกรณ์จริงจะเปลี่ยนอะไร (Rapid PVST+, LACP, firewall แยก ฯลฯ)
