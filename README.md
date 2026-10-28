# esphome-hlk-ld2402-with-web-gui-tuning
ESPHome-Externer Baustein – Hailink LD2402 Radarsensor mit integrierter Web-GUI zur Parametrierung

A ESPhome External Component Working With HLK-LD2402 Radar Sensor. Tuning with build-in web gui

## Gebrauchsanweisung:
* Die Firmware-Version des Radars muss mindestens v3.3.5 betragen. Verwenden Sie die offizielle Host-Software, um eine Verbindung zum UART des Radars herzustellen und die Firmware sowie die OTA-Firmware zu überprüfen.
* Es werden ausschließlich HA-Entitäten vom Typ „Sensor“ bereitgestellt (Entfernung, Anwesenheit, Firmware-Version, Betriebsmodus usw.). Alle Entitäten zur Parametrierung sind in der Web-GUI zusammengefasst.
* Es wird ein HTTP-API-Endpunkt bereitgestellt, der mit externen HTTP-Tools aufgerufen werden kann.
* Es gibt keine Funktion zur automatischen Schwellenwertgenerierung.
* Getestet und für funktionstüchtig befunden unter ESPHome 2026.9.0; bitte mit der ESP-IDF-Toolchain kompilieren.
* Unterstützung mehrerer Instanzen, d. h. Konfiguration mehrerer LD2402-Komponenten. Die Unterscheidung erfolgt über die ID.
* Es gelten bestimmte Anforderungen an den Arbeitsspeicher und die Komponentenbibliothek: Es wird nur die ESP32-Serie unter Verwendung des ESP-IDF-Frameworks unterstützt; ESP8266 wird nicht unterstützt.
* Spezielle Konfiguration für Arbeitsspeicher und Watchdog:
  ```
  esp32:
    variant: esp32c3
    framework:
      type: esp-idf
      sdkconfig_options:
        CONFIG_ESP_TASK_WDT_TIMEOUT_S: "30"
        CONFIG_ESP_INT_WDT_TIMEOUT_MS: "800"
      advanced:
        loop_task_stack_size: 10240
  ```
* 100% AI Coding

## Gebrauchsanweisung:
*
## Screenshots der Benutzeroberfläche
* HA-Entität
  <img width="344" height="171" alt="image" src="https://github.com/user-attachments/assets/a772bc08-965b-43f1-9026-e1af5b18d4b0" />
  <img width="315" height="107" alt="image" src="https://github.com/user-attachments/assets/103855b2-38f1-4175-9c6e-4e58c3946b29" />
* web-gui
* 
   <img src="https://github.com/gasment/esphome_hlk_ld2402_with_web_gui_tuning/blob/main/web-gui.webp" />

## Konfigurationsbeispiel
```
external_components:
  - source:
      type: git
      url: https://github.com/gasment/esphome_hlk_ld2402_with_web_gui_tuning
      ref: main
    components: [ld2402]
    refresh: always

uart:
  id: uart_ld2402
  tx_pin: GPIO4   #your uart tx pin
  rx_pin: GPIO3  #your uart rx pin
  baud_rate: 115200
  parity: NONE
  stop_bits: 1
  data_bits: 8  


ld2402:
  - id: ld2402_radar
    uart_id: uart_ld2402
    web_port: 8080  #set a web-gui port
    web_username: admin # set a web-gui basic-auth username
    web_password: admin  # set a web-gui basic-auth password

binary_sensor:
  - platform: ld2402
    ld2402_id: ld2402_radar
    name: "Occupancy"
    device_class: occupancy

sensor:
  - platform: ld2402
    ld2402_id: ld2402_radar
    distance:
      name: "Target Distance"
      filters:
          - throttle: 2s  
text_sensor:
  - platform: ld2402
    ld2402_id: ld2402_radar
    firmware_version:
      name: "LD2402 Firmware"
      icon: "mdi:chip"
      entity_category: "diagnostic"
    work_mode:
      name: "LD2402 Work Mode"
      icon: "mdi:cog-play"
      entity_category: "diagnostic"
```
## web调参
* 访问设备IP:web_port,进入调参界面
* 实时能量需要先开启工程模式，调试完记得关闭工程模式
* 任何修改都记得保存到flash来持久化
  
## http api
| 端点 | 方法 | 用途 |
|---|---|---|
| `/` | GET | Web 管理页面（HTML） |
| `/api/info` | GET | 获取雷达状态/配置信息 |
| `/api/cmd` | POST | 发送控制命令 |
| `/sse` | GET | 实时能量数据流|

* cmd端点payload格式，可通过web调试或分析源码ld2402.cpp内的handle_api_cmd方法取得


## 多实例
s使用多实例后，如打开工程模式不显示实时能量值，可以先关闭工程模式，刷新下页面后再打开
```
ld2402:
  - id: ld2402_radar_1
    uart_id: uart_ld2402_1
    web_port: ${web_port_1}
    web_username: ${web_user}
    web_password: ${web_passwd}

  - id: ld2402_radar_2
    uart_id: uart_ld2402_2
    web_port: ${web_port_2}   #需要使用不同的端口，多实例间的端口最好不要相邻
    web_username: ${web_user}
    web_password: ${web_passwd}
```
