# สำรวจ repository ตามโครงสร้าง `ecu-tuner-pro`

**วันที่ประเมิน:** 14 ตุลาคม 2026  
**ขอบเขต:** สำรวจแหล่งโค้ดและข้อมูลที่ตรงกับแต่ละโมดูลใน tree; ประเมินจากหน้า repository/README/source ที่เข้าถึงได้ ไม่ได้ build หรือทดสอบกับ ECU/อะแดปเตอร์จริง

## สรุปผู้บริหาร

ยังไม่พบ repository เดียวที่ทำได้ครบทั้ง UI, diagnostic, live data, map editing, checksum, BIN write, hardware abstraction และ DTC สำหรับ Honda/Yamaha/Kawasaki/Suzuki/GPX Delphi MT05 พร้อมกัน สิ่งที่พบเป็นชิ้นส่วนที่มีขอบเขตแคบและระดับความพร้อมต่างกัน:

- **มี source ที่ใช้เป็น reference ได้:** K-Line/KWP2000 diagnostics บางรุ่น, Yamaha logger, Honda live-data projects, XDF/BIN parsers, J2534 adapter layers และ DTC seed data
- **ยังไม่ยืนยันหรือยังไม่มี source ครบ:** Yamaha in-vehicle flash writer, motorcycle-wide ECU/map database, checksum profiles ข้ามรุ่น และ open-source Delphi MT05 flash workflow
- **ข้อสรุปสำคัญ:** ISO 14230/KWP2000 เป็นมาตรฐานสื่อสาร ไม่ได้ทำให้บริการ ECU, wake-up sequence, data IDs, checksum หรือ flash procedure ใช้แทนกันข้ามค่าย/รุ่นได้

ดังนั้น tree ใช้เป็นภาพรวมได้ แต่ควรแยก **transport**, **มาตรฐาน protocol**, **ECU/model profile**, **map definitions** และ **checksum profile** ออกจากกัน แทนการถือว่าไฟล์แยกตามยี่ห้อเพียงอย่างเดียวจะให้การรองรับทุกโมเดล

## สัญลักษณ์

- **ตรง/มี reference** — พบ source หรือเครื่องมือที่ตรงกับงานบางส่วน
- **บางส่วน** — จำกัดรุ่น, อ่านอย่างเดียว, เป็น low-level, หรือเป็นเพียง format/UI reference
- **ไม่ยืนยัน/ไม่พบ** — ยังไม่มีหลักฐานพอว่าทำงานตามที่โฟลเดอร์ต้องการ

## ตาราง map requirement ↔ repository

| ส่วนใน tree | แหล่งที่พบ | ความตรงและข้อจำกัด |
|---|---|---|
| `app/components`, `views/Flasher`, `MapEditor`, `Diagnostics` | [MotoLink](https://github.com/fpoussin/MotoLink), [Honda K-Line KWP2000](https://github.com/andreibaw/Honda_K-Line_KWP2000), [Honda K-Line Smart Gauge](https://github.com/aeaphichart/Honda-K-Line-smart-guage-esp32-project-v1) | **บางส่วน.** MotoLink มี GUI/เครื่องมือ fuel mapping และ firmware ของอุปกรณ์ของโครงการ; เหมาะเป็นแนวทาง UI/telemetry แต่ไม่ใช่แอป multi-brand สำหรับ flash BIN. Honda K-Line KWP2000 มี responsive web UI/live data และ OLED สำหรับ CBR600RR ปี 2005; เป็น diagnostic มากกว่าตัวแก้ map/flasher. ยังไม่พบ UI เปิด-source ที่รวม table editor, 2D/3D map, live gauges, DTC และ flashing ครบ |
| `core/protocols/honda_kline.py` | [HondaECU-1](https://github.com/aeaphichart/HondaECU-1), [Honda K-Line KWP2000](https://github.com/andreibaw/Honda_K-Line_KWP2000), [Honda K-Line Smart Gauge](https://github.com/aeaphichart/Honda-K-Line-smart-guage-esp32-project-v1) | **บางส่วนถึงค่อนข้างตรงในรุ่นที่ระบุ.** HondaECU-1 เป็น source/reference สำหรับ diagnostic และ flashing บาง ECU พร้อมไฟล์รุ่นเฉพาะ แต่เป็น fork เก่า. K-Line KWP2000 repo ระบุทดสอบกับ CBR600RR ปี 2005 และเตือนว่า implementation ต่างกันตามรุ่น/ปี; ตัวอย่างนี้เน้น live data ไม่ใช่ universal writer. Smart Gauge เน้น telemetry/DTC |
| `core/protocols/yamaha_kline.py` | [Yamaha K-line Tool](https://github.com/aeaphichart/Yamaha-K-line-Tool), [Yamaha Aerox XDF/BIN](https://github.com/AutotronicCommunity/YAMAHA-AEROX-XDF--BIN) | **บางส่วน.** Yamaha-K-line-Tool เหมาะกับ K-Line logging/sniffing บนรุ่นก่อนปี 2015; ไม่ใช่หลักฐานของ flash writer. มี XDF/BIN เฉพาะ Aerox เป็น map-data seed; ยังไม่พบ source เปิดที่ยืนยันการ reflash Yamaha หลายรุ่น |
| `core/protocols/kawasaki_kline.py` | [Keyword-Protocol-2000](https://github.com/aster94/Keyword-Protocol-2000), [KDS](https://github.com/Eztys/KDS) | **บางส่วน.** Keyword-Protocol-2000 เป็น KWP2000 library; README ระบุการทดสอบสำเร็จกับ Kawasaki Versys 650 ปี 2012 (และ Suzuki GSX-R600 ปี 2011), ไม่ใช่การรับรองทุก Kawasaki/ทุก service. Eztys/KDS ให้ low-level K-Line initiate/send/receive เป็นหลัก ไม่ใช่ flash stack สำเร็จรูป |
| `core/protocols/suzuki_kline.py` | [Keyword-Protocol-2000](https://github.com/aster94/Keyword-Protocol-2000), [sv650sds](https://github.com/synfinatic/sv650sds) | **บางส่วน.** มีตัวอย่าง Suzuki GSX-R600 ปี 2011 ใน KWP2000 library. sv650sds เป็น reverse-engineering/tools สำหรับ SV650 ปี 2003–2005; decoder/protocol ยังไม่ครบและ repository ถูก archive ในปี 2026. ไม่ใช่ฐานข้อมูล Suzuki SDS ทั่วไป |
| `core/protocols/gpx_delphi.py` | [ECUDelphiMT05](https://github.com/ajfemia/ECUDelphiMT05), [HUD ECU Hacker](https://netcult.ch/elmue/HUD%20ECU%20Hacker/) | **บางส่วน; ต้องระบุ revision ของกล่อง.** ECUDelphiMT05 เป็น Arduino/CAN project ที่เน้นอ่านข้อมูล ไม่ยืนยันการเขียน flash. HUD ECU Hacker เป็นซอฟต์แวร์สำเร็จรูปที่หน้าเว็บระบุ MT05/MT05.2 ผ่าน K-Line ISO 14230 และ MT05.3 ผ่าน CAN พร้อมความสามารถ flash/tune; ไม่ใช่ source repository ให้ยกโค้ดมาใช้. แยก profile ตาม MT05 revision และตรวจ ECU จริงของรุ่น GPX เป้าหมายก่อน |
| `core/checksums/{crc16,crc32,additive}.py` | [MX-KING-CHECKSUM-CORRECTOR](https://github.com/aeaphichart/MX-KING-CHECKSUM-CORRECTOR), checksum routines ที่กระจายอยู่ในโปรเจกต์เฉพาะ ECU | **มีเฉพาะจุด.** การมี CRC16/CRC32/additive function ไม่ได้แปลว่าทราบ variant, ช่วง address, seed/init/final XOR, endianness หรือ checksum block ของ ECU นั้น. MX King เป็น utility เจาะจงรุ่น และการทำงานเขียนทับไฟล์เดิมตามที่ตรวจในรายงานก่อนหน้า. ยังไม่พบชุด checksum profile ที่ครอบคลุมทุกแบรนด์/กล่อง |
| `core/parser/bin_parser.py`, `map_resolver.py` | [whalenlg/xdf_parser](https://github.com/whalenlg/xdf_parser), [TunerPro XDF/BIN CLI editor](https://github.com/KingAiCodeForge/tunerpro-xdf-bin-cli-map-editor), [HondaECU-1](https://github.com/aeaphichart/HondaECU-1), [Yamaha Aerox XDF/BIN](https://github.com/AutotronicCommunity/YAMAHA-AEROX-XDF--BIN) | **บางส่วน.** xdf_parser เป็น Python XDF/BIN map tool ที่ยกตัวอย่าง Porsche 944; CLI editor มี workflow แบบ diff/preflight แต่ตัวอย่างที่ยืนยันเป็น BMW/Holden และ README ระบุว่าไม่ทำ checksum/flash. ใช้เป็นแนวทาง parser/editor ได้ ไม่ใช่แหล่ง map definitions มอเตอร์ไซค์. Resolver ควรผูกกับ ECU ID/HW/SW/version และ hash ของไฟล์ ไม่ควรเลือก map จากยี่ห้ออย่างเดียว |
| `hardware/serial_conn.py` | Honda K-Line projects; [MotoLink](https://github.com/fpoussin/MotoLink) | **บางส่วน.** มีตัวอย่าง serial/K-Line hardware. Honda K-Line KWP2000 ใช้ L9637D transceiver; หลักฐานนี้ไม่ควรนำไปต่อ K-Line เข้าขา USB-UART TTL โดยตรง. FTDI/CH340 เป็น USB-UART bridge ไม่ใช่ K-Line voltage transceiver; wake-up/timing/การต่อวงจรต้องผ่าน interface ที่เหมาะกับรถและทดสอบแยกรุ่น |
| `hardware/pass_thru.py` | [bisak/openport-j2534](https://github.com/bisak/openport-j2534), [PyJ2534](https://github.com/Shamtam/PyJ2534) | **บางส่วน.** openport-j2534 เป็น J2534 PassThru driver สำหรับ OpenPort 2.0 บน Linux/macOS; README ระบุ CAN ผ่านการทดสอบ แต่ K-Line ยังไม่มี ECU ตอบในการทดสอบที่รายงาน. PyJ2534 เป็น Python wrapper สำหรับ J2534 DLL ซึ่งพัฒนา/ทดสอบกับ Python 3.8 แบบ 32-bit บน Windows. J2534 เป็น adapter API/transport layer ไม่ได้ให้ OEM-specific ECU services มาโดยอัตโนมัติ |
| `database/ecu_maps.json` | [ecu-definitions](https://github.com/sostisoft/ecu-definitions), HondaECU-1, Yamaha Aerox XDF/BIN | **ไม่พอสำหรับทุกค่าย.** ecu-definitions ระบุ 352 XDF และ 671 XML โดยรวม แต่ในโฟลเดอร์ Motos มี 4 รายการ; หนึ่งรายการ Suzuki Denso เป็น XDF `32920-41G00`. เป็น collection ที่เอนเอียงไปทางรถยนต์ ไม่ใช่ motorcycle-wide database. Honda/Yamaha มีไฟล์เฉพาะรุ่นแยกกัน แต่ยังไม่เป็น coverage matrix ครบทุก ECU |
| `database/dtc_codes.json` | [DTC Honda](https://github.com/AutotronicCommunity/dtc_honda), [DTC Yamaha](https://github.com/AutotronicCommunity/dtc-yamaha), [Automotive-9/dtc-codes](https://github.com/Automotive-9/dtc-codes) | **มี seed data แต่ต้องทำความสะอาด/ตรวจสอบ.** DTC Honda/Yamaha จาก AutotronicCommunity เป็นตาราง static HTML ไม่ใช่ JSON/API และต้องเก็บแหล่งที่มา/รุ่นรถก่อนแปลง. Automotive-9 เป็น JSON DTC collection สำหรับค่ายรถยนต์ที่ระบุ; ไม่ใช่ฐานข้อมูล DTC มอเตอร์ไซค์. ควรแยก manufacturer code จาก generic P/U/C code และไม่ถือว่าการ map รหัสตรงกันทุกปี/ECU |

## Repository ที่น่าตรวจต่อ (แยกตามบทบาท)

| Repository / tool | ใช้เป็น reference สำหรับ | ไม่ควรตีความว่าเป็น |
|---|---|---|
| `AutotronicCommunity/HondaECU-1` | Honda ECU diagnostics/flashing บางรุ่น, XDF/BIN/model-specific data | universal Honda protocol หรือ support ทุกรุ่นปัจจุบัน |
| `andreibaw/Honda_K-Line_KWP2000` | Honda K-Line diagnostics/live data และตัวอย่าง UI/hardware | Honda ECU flash writer; README ระบุความต่างของ implementation ระหว่างรุ่น |
| `aeaphichart/Yamaha-K-line-Tool` | Yamaha logger/sniffer และ reference การสื่อสาร | Yamaha in-vehicle flash tool |
| `aster94/Keyword-Protocol-2000` | shared KWP2000 diagnostics สำหรับสองรุ่นที่ผู้เขียนระบุว่าทดสอบ | KWP2000 profile ที่ยืนยันแล้วสำหรับ Honda/Yamaha/ทุก Suzuki/Kawasaki |
| `Eztys/KDS` | low-level Kawasaki K-Line communication | KDS diagnostics/map flashing ครบชั้น |
| `synfinatic/sv650sds` | reverse-engineering/reference สำหรับ SV650 ปี 2003–2005 | Suzuki SDS implementation ทั่วไป |
| `ajfemia/ECUDelphiMT05` | Delphi MT05 CAN data reading | MT05 flash writer |
| HUD ECU Hacker | feature/reference สำหรับ MT05 และเครื่องมือสำเร็จรูป | source code ที่นำไป integrate ได้ |
| `MotoLink` | GUI/telemetry/fuel-mapper architecture | desktop ECU BIN flasher ทั่วไป |
| `whalenlg/xdf_parser`, `KingAiCodeForge/...` | XDF/BIN parsing/editing workflow | motorcycle definitions/checksum/flash support |
| `bisak/openport-j2534`, `PyJ2534` | J2534 adapter abstraction/driver API | แทน KWP2000/UDS หรือ OEM-specific service layer |

## ข้อเสนอปรับโครงสร้างก่อนเริ่มพัฒนา

โครงสร้างเดิมเข้าใจง่าย แต่ควรทำให้ protocol transport ไม่ผูกกับยี่ห้อ และทำให้ข้อมูลเฉพาะ ECU เป็น profile ที่ตรวจสอบ/เวอร์ชันได้ เช่น:

```text
core/
  transport/
    kline_uart.py       # electrical adapter/serial I/O; ไม่ใส่ OEM service logic
    can.py
    j2534.py            # PassThru API wrapper
  protocols/
    iso14230_kwp2000.py
    uds.py              # เพิ่มเมื่อมี ECU เป้าหมายและหลักฐานรองรับ
  ecu_profiles/
    honda/<model>/<ecu-id>.yaml
    yamaha/<model>/<ecu-id>.yaml
    suzuki/<model>/<ecu-id>.yaml
    kawasaki/<model>/<ecu-id>.yaml
    delphi/<revision>/<ecu-id>.yaml
  checksums/
    algorithms/         # CRC/additive primitives
    profiles/           # per-ECU ranges, parameters, validation vectors
  parser/
    bin_image.py
    xdf.py
    map_resolver.py
  database/
    maps/               # definitions มีแหล่งที่มาและรุ่น
    dtc/                # รหัส, ภาษา, applicability และ provenance
```

แนวทางนี้ช่วยไม่ให้ K-Line ถูกเข้าใจผิดว่าเป็น protocol ของยี่ห้อ และช่วยแยก `ECU profile` (ID, wake-up, service/data IDs, memory layout, checksum) ออกจาก parser และ physical adapter ได้ชัดเจน. ฝั่ง UI สามารถคง `Flasher`, `MapEditor`, `Diagnostics` ไว้ได้ แต่แต่ละหน้าควรแสดงสถานะ support ที่แยกกัน เช่น `read-only`, `offline BIN editing`, `experimental` และ `write verified`.

## ช่องว่าง/ความเสี่ยงที่ยังต้องปิด

1. **ไม่มี coverage matrix ครบทุกแบรนด์และรุ่น.** ต้องกำหนดรุ่น/ปี/ECU part number และ HW/SW version เป้าหมายก่อน จึงจะตัดสินได้ว่า protocol/map/checksum ใดตรงกัน
2. **การมี KWP2000 ไม่ได้ยืนยันการ reflash.** KWP2000 เป็นมาตรฐานข้อความ; manufacturer-specific services, timing และ memory operations ต้องมีหลักฐานแยก
3. **ฐานข้อมูล map บาง.** XDF เป็น definition format ไม่ใช่ firmware image และ definition จากรถยนต์ใช้แทนข้อมูลมอเตอร์ไซค์ไม่ได้
4. **DTC จากเว็บยังไม่ใช่ข้อมูล service-manual ที่ผ่านการยืนยัน.** เก็บ URL/license/รุ่นรถ/วันที่ดึงข้อมูล และแยกค่า manufacturer-specific กับ generic OBD ให้ชัด
5. **ข้อกำหนด license ต่างกัน.** ตัวอย่างที่พบมี GPL-3.0, AGPL-3.0 และ MIT; ก่อนนำ source มารวม/แจกจ่ายต้องตรวจ license และ dependency ของแต่ละ repo
6. **ไม่มี hardware/ECU test ในการสำรวจนี้.** การมีโค้ดใน repository ไม่เท่ากับผ่านการทดสอบกับอะแดปเตอร์และ ECU ที่ต้องการ

## ลำดับงานแนะนำ

1. เลือก **รถนำร่องหนึ่งรุ่น** และบันทึก make/model/year, ECU label/part number และ HW/SW ID; ระบุว่ารอบแรกต้องการ diagnostics, live data, offline map editing หรือ flash
2. เริ่มจาก **read-only**: ตรวจ adapter, connection, ECU identification, live data/DTC และทำ replay/unit tests จาก log โดยไม่ส่ง write service
3. ทำ parser และ XDF/BIN workflow แบบ offline พร้อม diff, backup/original preservation และ test vectors
4. เพิ่ม checksum เฉพาะ profile เมื่อมีไฟล์ตัวอย่างที่รู้ค่าและทดสอบผลได้; ห้ามให้ fallback เลือก checksum แบบเดา
5. พิจารณา flash เฉพาะรุ่นที่มีหลักฐานโปรโตคอล, image/profile และการทดสอบที่เหมาะสม พร้อมสิทธิ์ใช้งานและขั้นตอนป้องกันความเสียหาย

## รายงานเดิมใน workspace

- [`github_autotroniccommunity_ecu_flash_analysis_th.md`](github_autotroniccommunity_ecu_flash_analysis_th.md) — ตรวจ repository ของ AutotronicCommunity
- [`github_aeaphichart_ecu_flash_analysis_th.md`](github_aeaphichart_ecu_flash_analysis_th.md) — ตรวจ repository ของ aeaphichart
