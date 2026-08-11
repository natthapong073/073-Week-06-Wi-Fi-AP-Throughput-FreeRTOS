# อภิธานศัพท์ คำย่อ และนิยามทางเทคนิคประจำสัปดาห์ที่ 6 (Glossary of Terms)

ตารางนี้รวบรวมคำศัพท์ นิยาม คำย่อ และแนวคิดสำคัญในสัปดาห์ที่ 6 สำหรับอ้างอิงและทบทวนความเข้าใจ:

---

## 1. คำศัพท์เกี่ยวกับ Wi-Fi SoftAP และระบบเครือข่าย

| คำศัพท์ / คำย่อ | คำเต็ม | นิยามและความหมายทางเทคนิค |
| :--- | :--- | :--- |
| **SoftAP** | Software-enabled Access Point | โหมดการทำงานที่ ESP32 ทำหน้าที่เป็น Access Point กระจายสัญญาณ Wi-Fi ของตนเอง |
| **Beacon Frame** | Beacon Frame | แพ็กเกจที่ Access Point กระจายออกไปในอากาศเป็นระยะ เพื่อประกาศชื่อ SSID และคุณสมบัติของเครือข่าย |
| **Beacon Interval** | Beacon Interval | ช่วงเวลาห่างในการกระจาย Beacon Frame (ปกติกำหนดเป็น 100 TUs ≈ 102.4 ms) |
| **DTIM** | Delivery Traffic Indication Message | จำนวนรอบของ Beacon Frame ที่ AP จะส่งแพ็กเกจ Broadcast/Multicast ข้อมูลให้ Client ที่หลับ (Power Save Mode) ตื่นขึ้นมารับ |
| **DHCPS** | Dynamic Host Configuration Protocol Server | บริการแจกจ่ายหมายเลข IP Address อัตโนมัติแก่ลูกข่ายที่เข้ามาต่อกับ ESP32 AP |
| **IP Lease Pool** | IP Lease Pool | ช่วงของหมายเลข IP Address ที่ DHCP Server อนุญาตให้แจกจ่ายได้ (เช่น 192.168.4.2 ถึง 192.168.4.11) |

---

## 2. คำศัพท์เกี่ยวกับฟิสิกส์คลื่นวิทยุและประสิทธิภาพ (RF Physics & Performance)

| คำศัพท์ / คำย่อ | คำเต็ม | นิยามและความหมายทางเทคนิค |
| :--- | :--- | :--- |
| **RSSI** | Received Signal Strength Indicator | ดัชนีวัดความแรงของสัญญาณวิทยุที่ภาครับ มีหน่วยเป็น dBm (ค่าลบ ยิ่งเข้าใกล้ 0 ยิ่งแรง) |
| **dBm** | Decibel-milliwatts | หน่วยวัดกำลังของสัญญาณวิทยุเทียบกับ 1 มิลลิวัตต์ในสเกลลอการิทึม |
| **Tx Power** | Transmit Power | กำลังส่งออกของภาคส่งวิทยุ ESP32 ปรับได้ตั้งแต่ 2 dBm (8) ถึง 20 dBm (80) |
| **FSPL** | Free Space Path Loss | ค่าการสูญเสียกำลังของคลื่นวิทยุเมื่อเดินทางผ่านอวกาศหรืออากาศแปรผันตามระยะทางและระดับความถี่ |
| **Throughput** | Network Throughput | อัตราการรับส่งข้อมูลจริงระดับแอปพลิเคชันต่อหนึ่งหน่วยเวลา (หน่วย: Kbps หรือ KB/s) |
| **RTT / Latency** | Round Trip Time / Latency | ระยะเวลาที่แพ็กเกจเดินทางจากผู้ส่งไปยังผู้รับและได้รับการตอบรับกลับมา (หน่วย: ms) |
| **Retransmissions** | Retransmission Rate | อัตราการส่งแพ็กเกจซ้ำเนื่องจากแพ็กเกจหล่นหายในอากาศจากสัญญาณอ่อนหรือรบกวน |

---

## 3. คำศัพท์เกี่ยวกับระบบปฏิบัติการ FreeRTOS และ Sensor Fusion

| คำศัพท์ / คำย่อ | คำเต็ม | นิยามและความหมายทางเทคนิค |
| :--- | :--- | :--- |
| **FreeRTOS** | Real-Time Operating System | ระบบปฏิบัติการเรียลไทม์ที่ ESP-IDF ใช้บริหารจัดการ Multi-Tasking |
| **Task** | FreeRTOS Task | เธรดการทำงานอิสระที่มีหน่วยความจำสแตก (Stack) และลำดับความสำคัญ (Priority) เป็นของตนเอง |
| **Queue** | FreeRTOS Queue | โครงสร้างข้อมูลแบบ FIFO (First-In, First-Out) ที่ปลอดภัยสำหรับการแลกเปลี่ยนข้อมูลระหว่าง Task (Thread-Safe) |
| **High Water Mark** | Task Stack High Water Mark | ค่าการวัดสแตกคงเหลือต่ำสุดของ Task หากเข้าใกล้ 0 แสดงว่าเสี่ยงต่อการเกิด Stack Overflow |
| **Proximity** | Proximity Distance | ระยะทางประเมินความใกล้-ไกลระหว่างอุปกรณ์โดยคำนวณจากระดับความแรงสัญญาณ RSSI |
| **Sensor Fusion** | Sensor & Signal Fusion | การนำข้อมูลจากหลายเซนเซอร์และสัญญาณวิทยุมารวมกันเพื่อตัดสินใจหรือยืนยันตัวตนได้อย่างแม่นยำ |
