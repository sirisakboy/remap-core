# ECU Tuner — รายการคัดเฉพาะแกนกลางที่นำไปต่อยอดได้

**ชุดไฟล์ที่ดาวน์โหลดและจัดระเบียบแล้ว:** [ecu_tuner_core_pack.zip](ecu_tuner_core_pack.zip)  
**รายงานวิเคราะห์รายไฟล์:** [ecu_tuner_core_pack/FILE_ANALYSIS_TH.md](ecu_tuner_core_pack/FILE_ANALYSIS_TH.md)

**หลักการคัด:** ไม่ยก repository ทั้งก้อน ไม่เก็บ demo/UI/firmware ที่ผูกกับบอร์ดหรือรถรุ่นเดียว และไม่ใส่ data dump จำนวนมากลง core. เก็บเฉพาะ library/API/format parser ที่ reuse ได้; ส่วนที่เป็นข้อมูลรถให้เป็น plug-in/profile แยกต่างหาก.

## ข้อสรุปก่อนเลือก

การรับรองว่าใช้ได้กับ **ทุก OS + ทุกภาษา + อะแดปเตอร์ทุกแบบ** ทำไม่ได้ด้วยการคัด source files จาก repository อย่างเดียว เพราะบางอะแดปเตอร์ใช้ vendor driver เฉพาะ OS และ ECU แต่ละรุ่นมีรายละเอียด protocol ไม่เหมือนกัน. แนวทางที่ขยายได้ที่สุดคือมี **สัญญา API กลางและ schema ที่ไม่ผูกกับภาษา** แล้วให้ driver/protocol เป็น plug-in.

รูปแบบการเชื่อมต่อจากซอฟต์แวร์หลายภาษา:

- **JSON-RPC 2.0 ผ่าน local service** เป็นช่องทางหลักสำหรับ Python, JavaScript/TypeScript, C#, Java และภาษาอื่นที่อ่าน JSON/สื่อสารกับ process ได้
- **C ABI** เป็นช่องทางเสริมสำหรับแอป native/embedded ที่ต้องเรียก library โดยตรง
- สร้าง client bindings จาก schema ได้ภายหลัง; อย่าฝัง logic ECU ไว้ใน UI แต่ละภาษา

## คัดไว้เป็นแกนจริง ๆ

| อันดับ | สิ่งที่เก็บ/นำไปต่อยอด | Repository/ไฟล์อ้างอิงที่เลือก | ข้อจำกัด/ข้อกำหนด |
|---|---|---|---|
| 1 | **Serial transport abstraction** สำหรับ FTDI/CH340/USB-UART ที่ OS มองเห็นเป็น COM/TTY | [libserialport](https://github.com/sigrokproject/libserialport) — C library ข้ามแพลตฟอร์ม | เป็น serial I/O เท่านั้น ไม่ใช่ K-Line transceiver และไม่ทำ wake-up/KWP. ใช้ผ่าน C ABI หรือ wrapper ได้หลายภาษา. License LGPL-3.0; ตรวจเงื่อนไขก่อนแจกจ่าย |
| 2 | **Protocol layer สำหรับ KWP2000** — เก็บเฉพาะ framing/state/การแยก protocol ออกจากอุปกรณ์ | [ISO.h](https://github.com/aster94/Keyword-Protocol-2000/blob/master/src/ISO.h), [KWP2000.h](https://github.com/aster94/Keyword-Protocol-2000/blob/master/src/KWP2000.h), [KWP2000.cpp](https://github.com/aster94/Keyword-Protocol-2000/blob/master/src/KWP2000.cpp) | ดาวน์โหลดเฉพาะ source 3 ไฟล์ไว้ในโฟลเดอร์ reference; ไม่เอา `.ino`, pin configuration หรือรายการรุ่นรถไปเป็นค่า universal. โค้ดผูก Arduino และมี Honda/Yamaha placeholders. Repository `LICENSE` เป็น GPL-3.0 แต่ไฟล์ source มี MIT headers จึงต้องเคลียร์ license ก่อน copy/ดัดแปลง. |
| 3 | **BIN/XDF offline parser + diff/preflight** | ใช้ [xdf_merge.py](https://github.com/whalenlg/xdf_parser/blob/main/xdf_merge.py) เป็น parser reference; MIT | โปรเจกต์ตั้งต้นจาก Porsche 944 ไม่ใช่ parser ที่ยืนยันกับ XDF ทุกแบบ. README แสดง MIT badge แต่ snapshot ไม่มี LICENSE/COPYING; ดาวน์โหลดไว้เป็น reference only และห้ามถือว่าอนุมัติให้นำ source ไปแจกจ่ายจนยืนยันสิทธิ์. ไม่เอาไฟล์ map/BIN ตัวอย่างมาปะปนกับ core |
| 4 | **API contract, ECU-profile schema, test vectors และ log/replay** | **สร้างในโปรเจกต์ของเรา** — ไม่มี repository ที่พบซึ่งครอบคลุมทุกยี่ห้อ/อะแดปเตอร์แทนได้ | นี่คือส่วนที่ทำให้ภาษา/OS อื่นต่อยอดได้จริง: กำหนดข้อมูล request/response, capabilities, ECU ID, units/scaling, errors, checksum profile และ sample logs แบบมี version/provenance |
| 5 | **Adapter interface สำหรับ J2534/CAN/อุปกรณ์เฉพาะ** | ออกแบบ interface กลางเอง; ใช้ [PyJ2534](https://github.com/Shamtam/PyJ2534) หรือ [openport-j2534](https://github.com/bisak/openport-j2534) เป็น backend/reference เฉพาะเมื่อจำเป็น | ไม่รวม driver ทั้งหมดเข้า core. PyJ2534 เป็น Python/Windows DLL wrapper (AGPL-3.0); openport-j2534 เป็น OpenPort 2.0 backend บน Linux/macOS (GPL-3.0-or-later) และ K-Line ยังไม่ยืนยันกับ ECU จริงตามรายงานของ repo. เลือก backend ตาม OS/adapter ที่จะรองรับจริง |

## เก็บเป็น ECU plug-in เท่านั้น — ไม่ใช่ universal core

- **Honda:** จาก [HondaECU-1](https://github.com/aeaphichart/HondaECU-1) เลือกเฉพาะ `src/ecmids.py` และ XDF/profile ที่ตรงกับ ECU เป้าหมายจริง. ไม่คัดทั้ง `bins/`, GUI, driver และ project tree. Repo เก่าและ GPL-3.0.
- **DTC:** ใช้ตาราง [Honda](https://github.com/AutotronicCommunity/dtc_honda/blob/main/index.html) / [Yamaha](https://github.com/AutotronicCommunity/dtc-yamaha/blob/main/index.html) เป็นแหล่งข้อมูลตั้งต้นเท่านั้น; ย้ายเฉพาะ record ที่ตรวจสอบแล้วเข้า JSON schema กลาง พร้อมรุ่นรถ/แหล่งอ้างอิง/ภาษา.
- **Checksum:** ไม่มี checksum profile สากลให้เก็บ. [Form1.vb](https://github.com/aeaphichart/MX-KING-CHECKSUM-CORRECTOR/blob/main/Form1.vb) ควรพิจารณาเฉพาะ Yamaha MX King V1 (2PV0) ที่ตรงเป๊ะเท่านั้น; รุ่นอื่นสร้าง profile และ test vector แยก.
- **Delphi MT05 / Suzuki / Kawasaki / Yamaha:** ใส่เป็น protocol/ECU plug-in เฉพาะเมื่อเลือกกล่องและรุ่นรถเป้าหมาย ไม่ใส่ service IDs หรือ calibration data เหล่านี้ในแกนกลาง.

## ไม่คัดเข้าชุดหลัก

- GUI, dashboard และ web UI ที่ผูกกับ ESP32/Qt/รถรุ่นเดียว
- ตัวอย่าง `.ino`, firmware/bootloader ของ project board และ wiring ที่ไม่ใช่มาตรฐานอุปกรณ์ทั่วไป
- BIN/ROM ที่ไม่ทราบ provenance/สิทธิ์ใช้ หรือไม่ตรง ECU เป้าหมาย
- repository map collections ทั้งก้อน, DTC HTML ทั้งก้อน และโปรเจกต์ flasher ของรถรุ่นอื่น
- checksum/flash code ที่ไม่มี test vector สำหรับ ECU เป้าหมาย

## โครงสร้างเป้าหมายแบบสั้น

```text
portable-core/
  api-schema/       # JSON Schema หรือ Protocol Buffers; versioned
  service/          # JSON-RPC local service; C ABI ถ้าจำเป็น
  transport/
    serial/         # libserialport backend
    j2534/          # adapter plugin ตาม OS/vendor
    can/            # plugin ตาม CAN backend
  protocols/
    kwp2000/        # protocol กลาง ไม่ hardcode รุ่นรถ
    uds/            # เพิ่มเมื่อมีเป้าหมาย/หลักฐานรองรับ
  ecu-profiles/     # รุ่นรถ/ECU ID/HW-SW version แยกไฟล์
  formats/
    bin/
    xdf/
  checksum-profiles/ # algorithm + range/parameters + known-good vectors
  data/
    dtc/            # normalized, attributed, versioned
  tests/
    logs/           # replay fixtures และ regression tests
```

**เกณฑ์ตัดสินก่อนนำไฟล์ใดเข้าจริง:** (1) license ใช้ต่อได้, (2) แยกจากฮาร์ดแวร์/รถรุ่นเฉพาะได้, (3) มี test หรือทำ test vector เพิ่มได้, (4) ไม่มี binary/data ที่ไม่ทราบสิทธิ์, (5) ไม่ทำให้การรองรับรุ่นที่ยังไม่ได้ทดสอบถูกแสดงเป็น “รองรับแล้ว”.
