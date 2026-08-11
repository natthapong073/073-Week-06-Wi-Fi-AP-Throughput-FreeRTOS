# ฟิสิกส์คลื่นวิทยุ การปรับกำลังส่ง (Tx Power) และการประเมินความเร็ว (Speed Profiling)

ความแรงของสัญญาณคลื่นวิทยุ Wi-Fi (RF Signal) มีผลโดยตรงต่อความเร็วในการรับส่งข้อมูล (Data Rate / Throughput) และค่าความหน่วงเวลารอคอย (Latency) ในบทเรียนนี้จะอธิบายหลักการทางฟิสิกส์ วิธีการปรับกำลังส่งสัญญาณฮาร์ดแวร์บน ESP32 และการประเมินความเร็วเพื่อทำวิจัยเชิงสถิติ (Regression Analysis)

---

## 1. ค่าความแรงสัญญาณ RSSI และระดับความหมาย (RSSI Benchmark)

**RSSI (Received Signal Strength Indicator)** มีหน่วยวัดเป็น **dBm (decibel-milliwatts)** ซึ่งเป็นค่าลบสเกลแบบ Logarithmic:

$$P_{\text{dBm}} = 10 \cdot \log_{10} \left( \frac{P_{\text{mW}}}{1\text{ mW}} \right)$$

| ระดับ RSSI (dBm) | คุณภาพสัญญาณ | คำอธิบายพฤติกรรมในทางปฏิบัติ |
| :--- | :--- | :--- |
| **-30 ถึง -50 dBm** | **ดีเยี่ยม (Excellent)** | อุปกรณ์อยู่ใกล้กันมาก สัญญาณแรงเต็มเปี่ยม ไร้สิ่งกีดขวาง |
| **-50 ถึง -65 dBm** | **ดี (Good)** | ระยะทางปกติในห้องเรียน รับส่งข้อมูลด้วยความเร็วสูงสุด |
| **-67 ถึง -75 dBm** | **พอใช้ (Fair)** | เริ่มมีระยะห่าง หรือมีสิ่งกีดขวางบางส่วน ความเร็วเริ่มลดลง |
| **-75 ถึง -85 dBm** | **อ่อน (Weak)** | สัญญาณอ่อนมาก เกิด Packet Loss สูง TCP Retransmission ถ่ายโอนข้อมูลช้าลง |
| **ต่ำกว่า -90 dBm** | **วิกฤต (Unusable)** | หลุดการเชื่อมต่อ เกิดเหตุการณ์ `WIFI_EVENT_STA_DISCONNECTED` |

---

## 2. การควบคุมกำลังส่งฮาร์ดแวร์ด้วย ESP-IDF (`esp_wifi_set_max_tx_power`)

ESP-IDF มีฟังก์ชันสำหรับควบคุมกำลังส่งสูงสุดของวิทยุ Wi-Fi ใน ESP32:

```c
esp_err_t esp_wifi_set_max_tx_power(int8_t power);
esp_err_t esp_wifi_get_max_tx_power(int8_t *power);
```

### พารามิเตอร์ `power` (หน่วย: 0.25 dBm)
ค่า `power` ที่ป้อนเข้าฟังก์ชันจะคิดเป็น **4 เท่าของค่า dBm** (เช่น 1 dBm = 4):
* **`80`** = 20 dBm (กำลังส่งสูงสุด 100 mW - ค่า Default)
* **`60`** = 15 dBm
* **`40`** = 10 dBm
* **`20`** = 5 dBm
* **`8`**  = 2 dBm (กำลังส่งต่ำสุด)

> **ประโยชน์ในห้องเรียน**: นักศึกษาสามารถจำลอง "การถอยห่างของอุปกรณ์ไปไกลนับสิบเมตร" ได้ง่ายๆ เพียงสั่งลดค่า Tx Power ลงเหลือ `8` (2 dBm) โดยไม่ต้องเดินออกจากโต๊ะทดลอง!

---

## 3. การคำนวณความเร็ว (Throughput Speed Calculation)

การวัด Throughput หรือความเร็วในการรับส่งข้อมูลจริงระดับ Network Layer คำนวณจากปริมาณข้อมูลไบต์ทั้งหมดที่รับส่งได้ต่อหนึ่งหน่วยเวลา:

$$\text{Throughput (Kbps)} = \frac{\text{Total Bytes} \times 8}{1000 \times \Delta t \text{ (seconds)}}$$

$$\text{Throughput (KB/s)} = \frac{\text{Total Bytes}}{1024 \times \Delta t \text{ (seconds)}}$$

---

## 4. การทำแบบโมเดลถดถอย (Regression Analysis: RSSI vs Speed)

เมื่อบันทึกข้อมูลค่า RSSI คู่กับ Throughput ในระดับความแรงสัญญาณต่างๆ นักศึกษาสามารถพล็อต **Scatter Plot** และสร้างสมการถดถอยเพื่อทำนายความเร็ว:

```mermaid
gantt
    title RSSI vs Throughput Relationship Curve
    dateFormat X
    axisFormat %s
    section Max Speed Zone (-30 to -65 dBm) : 30, 65
    section Speed Degradation Zone (-65 to -80 dBm) : 65, 80
    section Disconnect Risk Zone (< -85 dBm) : 80, 95
```

### รูปแบบสมการ Logarithmic Regression ที่คาดหวัง:

$$\text{Speed (Kbps)} = a \cdot \ln(|\text{RSSI}|) + b$$

หรือการสร้างแบบจำลองประเภท **Piecewise Linear Model** เพื่อหาจุดตัดวิกฤต (Threshold Point) ที่ทำให้สตรีมข้อมูลเกิด Packet Drop ในระบบ IoT Real-Time
