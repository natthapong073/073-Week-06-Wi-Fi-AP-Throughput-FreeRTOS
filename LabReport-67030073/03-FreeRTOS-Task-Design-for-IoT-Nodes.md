# สถาปัตยกรรม Multi-Tasking ด้วย FreeRTOS สำหรับโหนด IoT

ในระบบปฏิบัติการเรียลไทม์ (RTOS) บน ESP-IDF การทำงานของสแตกระบบเครือข่าย Wi-Fi และ TCP/IP ทำงานแยกอิสระในรูปแบบ Asynchronous หากโปรแกรมเมอร์นำโค้ดอ่านเซนเซอร์ที่ต้องหน่วงเวลา (เช่น `vTaskDelay` หรือ I2C Polling) ไปวางไว้ภายใน Event Handler ของ Wi-Fi จะส่งผลให้เครือข่ายหยุดชะงัก (Block Wi-Fi Event Loop)

การแก้ปัญหามาตรฐานทางวิศวกรรมคือการใช้ **FreeRTOS Multi-Task Architecture** โดยแยกหน้าที่ระหว่าง **Sensor Collection Task** กับ **Network Task** และเชื่อมต่อกันด้วย **FreeRTOS Queue**

---

## 1. ผังการออกแบบระบบแบบ Multi-Task (Task Architecture Diagram)

```mermaid
graph TD
    subgraph Core 1 [Application Tasks Core]
        SensTask["Sensor Collector Task<br/>(Reads Temp, Humidity, Motion)"]
        SensTask -->|xQueueSend()| SensQueue[("FreeRTOS Queue<br/>(sensor_data_t)")]
    end

    subgraph Core 0 [System & Network Core]
        SensQueue -->|xQueueReceive()| NetTask["Wi-Fi / Web Server Task<br/>(Responds HTTP / TCP Benchmark)"]
        WifiDrv["ESP-IDF Wi-Fi Driver & SoftAP"]
        WifiDrv <--> NetTask
    end
```

---

## 2. โครงสร้างข้อมูลสตรีมใน Queue (`sensor_data_t`)

เพื่อความปลอดภัยในการเข้าถึงข้อมูลจากหลาย Task (Thread-Safe Data Exchange) จะส่งโครงสร้างข้อมูลผ่าน Queue แทนการใช้ตัวแปร Global:

```c
typedef struct {
  float temperature;
  float humidity;
  uint32_t lux;
  int8_t client_rssi;
  uint32_t timestamp_ms;
} sensor_data_t;
```

---

## 3. รูปแบบการทำงานของแต่ละ Task (Task Code Flow)

### 3.1 Sensor Collector Task (ผู้ส่งข้อมูลลง Queue)
ทำหน้าที่อ่านค่าฮาร์ดแวร์ตามเวลาที่กำหนด (เช่น ทุกๆ 1 วินาที) แล้วยัดลง Queue:

```c
void vSensorTask(void *pvParameters) {
  sensor_data_t data;
  while (1) {
    // 1. Read hardware sensors
    data.temperature = read_temperature();
    data.humidity = read_humidity();
    data.timestamp_ms = xTaskGetTickCount() * portTICK_PERIOD_MS;

    // 2. Send to Queue (Non-blocking or wait max 100ms)
    xQueueSend(xSensorQueue, &data, pdMS_TO_TICKS(100));

    // 3. Delay Task
    vTaskDelay(pdMS_TO_TICKS(1000));
  }
}
```

### 3.2 Network / Web Server Task (ผู้รับข้อมูลจาก Queue)
ทำหน้าที่รอดึงข้อมูลจาก Queue เมื่อมี Request เข้ามาจาก Wi-Fi:

```c
void vNetworkTask(void *pvParameters) {
  sensor_data_t received_data;
  while (1) {
    // Wait for data from Queue
    if (xQueueReceive(xSensorQueue, &received_data, portMAX_DELAY) == pdTRUE) {
      // Process network packet or serve HTTP JSON response
      ESP_LOGI("NET_TASK", "Served Temp: %.2f C, Hum: %.2f %%",
               received_data.temperature, received_data.humidity);
    }
  }
}
```

---

## 4. การตรวจสอบหน่วยความจำสแตกในสไตล์ Forensic (`uxTaskGetStackHighWaterMark`)

ปัญหายอดฮิตในระบบ IoT ที่ใช้ RTOS คือ **Stack Overflow** (หน่วยความจำสแตกของ Task ไม่เพียงพอจนบอร์ดคราช) ใน ESP-IDF นักศึกษาสามารถตรวจสอบหน่วยความจำสแตกที่เหลืออยู่ต่ำสุดในสไตล์ Forensic ได้ด้วย:

```c
UBaseType_t uxHighWaterMark = uxTaskGetStackHighWaterMark(NULL);
ESP_LOGI("FORENSIC", "Task Stack High Water Mark: %d bytes remaining",
         uxHighWaterMark * sizeof(StackType_t));
```

> **กฎสำคัญ**: หากค่า `High Water Mark` เข้าใกล้ 0 แสดงว่าสแตกกำลังจะเต็ม ต้องขยายขนาด `configMINIMAL_STACK_SIZE` ตอนสร้าง `xTaskCreate()`!
