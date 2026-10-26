# รายงานตรวจสอบ GitHub `AutotronicCommunity`
## ข้อมูลที่เกี่ยวข้องกับการพัฒนาซอฟต์แวร์แฟลช ECU รถจักรยานยนต์

**ตรวจเมื่อ:** 14 ตุลาคม 2026 (เวลาไทย)  
**ขอบเขต:** ตรวจเฉพาะข้อมูลสาธารณะบน GitHub ได้แก่ README, รายการไฟล์ และ source code ที่เปิดอ่านได้บางส่วน ไม่ได้ดาวน์โหลดหรือเรียกใช้ไฟล์ `.exe`/`.rar` และไม่ได้ทดสอบกับ ECU จริง

## สรุปผล

บัญชีนี้มีข้อมูลที่เป็นประโยชน์สำหรับ **เครื่องมือวินิจฉัย/อ่านข้อมูล ECU** และมีไฟล์เดโมของเครื่องมือแฟลชบางรุ่น แต่จากหลักฐานสาธารณะที่ตรวจพบ **ยังไม่พบ source code แบบเปิดที่ครบวงจรสำหรับเขียนแฟลช ECU อย่างปลอดภัย**

สิ่งที่หาได้: ตัวอย่าง K-Line ของ Honda บางตระกูล, โค้ดอ่าน Live Data, ข้อมูล pinout, ไฟล์ BIN/XDF ของ Yamaha Aerox และโปรแกรม Windows ที่แจกเป็นไบนารี/เดโม

สิ่งที่ยังขาดสำหรับการทำตัวเขียนแฟลชจริง: โปรโตคอลเขียนและกู้คืนที่ยืนยันแล้ว, ขอบเขต memory/การแบ่งบล็อก, วิธีตรวจสอบไฟล์และ checksum ของ ECU แต่ละรุ่น, เงื่อนไขความปลอดภัยขณะเขียน และชุดทดสอบ ECU บนโต๊ะทดลอง ดังนั้นไม่ควรถือว่า repo เหล่านี้เป็นคู่มือเพียงพอสำหรับทำ flasher ที่นำไปใช้กับรถจริง

## Repo ที่เกี่ยวข้องที่สุด

| Repo | สิ่งที่พบ | ใช้ประโยชน์ได้แค่ไหน |
|---|---|---|
| [Honda_Keihin_KLine_Protocol](https://github.com/AutotronicCommunity/Honda_Keihin_KLine_Protocol) | เอกสารและ log ของ Honda Keihin K-Line ระบุ 10,400 bps, ขั้นตอน wake-up/initialize, รูปแบบ request/response, checksum และตัวอย่างอ่านตารางข้อมูล | จุดเริ่มต้นที่ดีสำหรับ **อ่าน/วินิจฉัย** ECU รุ่นที่ตรงกับข้อมูลใน repo เท่านั้น ไม่ใช่คู่มือเขียนแฟลช |
| [scantoolHondaEsp32](https://github.com/AutotronicCommunity/scantoolHondaEsp32) และ [ScantoolHonda](https://github.com/AutotronicCommunity/ScantoolHonda) | Sketch ESP32 ทำ K-Line scan และหน้าเว็บ local แสดง RPM, TPS, อุณหภูมิ, MAP, แรงดันแบตเตอรี่, เวลา injector, ignition และความเร็ว โดย query ตาราง live-data | ใช้เป็นแนวทางทำ scanner/UI ได้ ไม่พบส่วนเขียน flash; โค้ดหนึ่งอ่าน buffer ด้วย offset คงที่โดยไม่ตรวจความยาว/หัวเฟรม/checksum ก่อน และเปิด Wi‑Fi AP ไม่มีรหัสผ่าน จึงไม่ควรนำไปใช้เป็น production code โดยตรง |
| [ADX-TUNER-PRO-ECU-CONNECTOR](https://github.com/AutotronicCommunity/ADX-TUNER-PRO-ECU-CONNECTOR) | มี VB.NET source สำหรับเปิด serial port, ตั้งจังหวะสื่อสาร และส่ง wake/init frame | เป็นตัวอย่างเริ่มต้นการเชื่อมต่อ ไม่พบการรับส่งข้อมูลครบวงจรหรือคำสั่งเขียน ECU |
| [SoftwareHondaFlasher](https://github.com/AutotronicCommunity/SoftwareHondaFlasher) | มี `.exe`/DLL และ README ระบุชัดว่าเดโมใช้ **Read ECU** ได้ ส่วน Write ECU ต้องใช้ license | ยืนยันว่ามีซอฟต์แวร์เชิงพาณิชย์ที่เกี่ยวข้อง แต่ไม่มี source ให้ตรวจสอบ และไม่ควรพยายามข้ามระบบ license |
| [SoftwareHondaECU](https://github.com/AutotronicCommunity/SoftwareHondaECU) | รวม installer/demo ของ MCU Innovations เช่น Honda K-Line/CAN, Yamaha ECU และ Renesas Recovery | เป็นไบนารีจากผู้พัฒนาภายนอก ไม่ใช่ source code สำหรับนำไปต่อยอดโดยอัตโนมัติ ต้องตรวจเงื่อนไข license/สิทธิ์ก่อนใช้ |
| [YAMAHA-FLASH-TOOL-V6-DEMO-VERSION](https://github.com/AutotronicCommunity/YAMAHA-FLASH-TOOL-V6-DEMO-VERSION) | มีโปรแกรม `.exe` เดโม ระบุเป้าหมาย MX King V1 `2PV0` / Exciter `2ND0` ปี 2015–2018 | บอกขอบเขตรุ่นของเดโมได้ แต่ไม่มี source/protocol ที่พอสร้าง implementation ใหม่หรือรับรองความปลอดภัย |
| [YamahaFlasher](https://github.com/AutotronicCommunity/YamahaFlasher) | พบไฟล์ archive `.rar` ไฟล์เดียว ไม่มีคำอธิบาย/เอกสารใน repo | ตรวจสอบการทำงาน ความเข้ากันได้ และ license จากหน้า repo ไม่ได้ |
| [ShindengenFlasher](https://github.com/AutotronicCommunity/ShindengenFlasher) | มี `.exe`; README ระบุว่าต้องใช้ license | ไม่มี source และไม่ใช่ฐานสำหรับทำ flasher แบบเปิด |
| [YAHAMA-FLASH-VIA-KLINE---BACKDOOR](https://github.com/AutotronicCommunity/YAHAMA-FLASH-VIA-KLINE---BACKDOOR) | README มีเพียง `COMINGSOON` พร้อมรูปภาพ | ยังไม่มี implementation ที่ตรวจสอบได้; ชื่อ repo สื่อถึงวิธีข้ามช่องทางปกติ จึงไม่ควรใช้เป็นฐานหรือพยายามดึงรายละเอียดเพื่อเลี่ยงการป้องกัน |
| [YAMAHA-AEROX-XDF--BIN](https://github.com/AutotronicCommunity/YAMAHA-AEROX-XDF--BIN) | มี BIN และ XDF สำหรับ Aerox/รหัส calibration ที่ระบุในชื่อไฟล์ | ช่วยดูโครงสร้าง/คำอธิบายตาราง calibration ได้ แต่ XDF/BIN **ไม่ใช่** โปรโตคอลสื่อสารหรือขั้นตอนเขียน ECU; ควรตรวจสิทธิ์และความตรงรุ่นของ firmware |
| [pinouECMHonda](https://github.com/AutotronicCommunity/pinouECMHonda) และ [pinoutECMYamaha](https://github.com/AutotronicCommunity/pinoutECMYamaha) | หน้า HTML รวม pinout/กลุ่มรุ่น Honda และ Yamaha | เป็นข้อมูลช่วยออกแบบ harness เบื้องต้น ต้องยืนยันจากเอกสาร service/manual และวัดกับ ECU รุ่นจริงก่อนจ่ายไฟหรือเชื่อมต่อ |
| [ARDUINO_READ_ECU_MX_KING_V1](https://github.com/AutotronicCommunity/ARDUINO_READ_ECU_MX_KING_V1) | ตัวอย่าง Arduino สำหรับอ่านข้อมูล ECU MX King V1 แบบเฉพาะรุ่นและแสดงข้อมูลดิบ | เป็นงานทดลองอ่านข้อมูลที่ hard-code หลายส่วน ไม่ใช่ writer ทั่วไปหรือหลักฐานว่าเขียนแฟลชได้อย่างปลอดภัย |
| [Honda-Motorcycle-ECU-Tools](https://github.com/AutotronicCommunity/Honda-Motorcycle-ECU-Tools) | เป็น fork ของเครื่องมืออ่านข้อมูล Honda CRF250L; README อธิบาย K-Line 10,400 bps และข้อจำกัดของ ELM327 บางรุ่น | มีประโยชน์กับการทำ reader/physical layer; ข้อมูลใน README ระบุว่าโปรโตคอล Honda ไม่ใช่ OBD-II มาตรฐาน จึงไม่ควรสรุปว่าใช้ได้กับ Honda ทุกรุ่น |
| [ChecksumCalculator](https://github.com/AutotronicCommunity/ChecksumCalculator) | มีเฉพาะ `.exe`; README แจ้งว่าต้องมี license | ไม่มี source และไม่ระบุอัลกอริทึมให้ตรวจสอบ จึงยังใช้ยืนยัน checksum ของ firmware ไม่ได้ |

## ข้อมูลเชิงเทคนิคที่ดึงไปใช้กับงานอ่านข้อมูลได้

ใน repo Honda Keihin มีเอกสาร K-Line ระบุการสื่อสารที่ **10,400 bit/s**, จังหวะ wake-up, เฟรม initialize, รูปแบบคำขออ่านตาราง และ checksum ของเฟรม ตัวอย่างหนึ่ง query ตาราง `0x17`; sketch ESP32 ใช้ตารางนี้เพื่ออ่านค่า live data และพยายามอัปเดตประมาณทุก 300 ms เอกสารยังเตือนว่า ECU อาจกลับเข้าสู่ sleep หากหยุดสื่อสารราว 2 วินาที

ข้อจำกัดสำคัญ: รายละเอียดนี้เป็นตัวอย่าง **diagnostic/read-only ของ ECU ที่ตรงรุ่น** ไม่ได้อธิบายการลบ/เขียน memory, การยืนยันตัวตนสำหรับ reflash, การแบ่ง block, การยืนยันผลหลังเขียน หรือ recovery หากไฟตก/การสื่อสารขาด จึงไม่ควรนำคำสั่งอ่านไปอนุมานเป็นคำสั่งเขียน

## สิ่งที่ต้องหาเพิ่มก่อนเริ่มทำตัวเขียนแฟลช

1. **กำหนดเป้าหมายแคบ ๆ:** ยี่ห้อ/รุ่น/ปี, รหัส ECU, หมายเลข part, hardware revision และ software/calibration ID ให้ชัดเจน รุ่นรถใกล้เคียงกันอาจใช้ ECU หรือ bootloader คนละแบบ
2. **ใช้วิธี reflash ที่ได้รับอนุญาต:** เอกสารจากผู้ผลิต/ผู้จำหน่าย หรือโปรโตคอลที่ได้รับอนุญาตสำหรับ ECU เป้าหมาย หลีกเลี่ยงการข้ามระบบป้องกันหรือระบบ license
3. **จัดเตรียม hardware ที่ปลอดภัย:** interface K-Line/CAN ที่มี transceiver และการป้องกันแรงดัน/กระแสตาม automotive environment, harness ที่ยืนยัน pinout แล้ว และแหล่งจ่ายไฟเสถียรพร้อมจำกัดกระแส
4. **ยืนยัน firmware ก่อนเขียน:** ระบุขนาด/ช่วง address, format, ความถูกต้องของไฟล์และ checksum/signature, ความเข้ากันได้กับ ECU และเก็บสำเนาเดิมที่ตรวจ hash ได้
5. **ออกแบบ state machine และ recovery:** ตรวจ ECU identity ก่อนเริ่ม, เงื่อนไขแรงดัน/การเชื่อมต่อ, timeout, บันทึกเหตุการณ์, verify หลังทำ และมีวิธีกู้คืนที่ผ่านการทดสอบกับ ECU สำรอง
6. **ทดสอบบน bench ก่อนรถจริง:** ใช้ ECU สำรอง/rig, ทดสอบกรณีไฟตก สายหลุด checksum ผิด และไฟล์ไม่ตรงรุ่น โดยไม่ทดลองกับรถที่ต้องใช้งานจริง
7. **จัดการสิทธิ์และข้อกำกับ:** ตรวจ license ของโปรแกรม/firmware, สิทธิ์การแจกจ่าย และข้อกำหนดด้าน emissions/ความปลอดภัยในตลาดที่จะจำหน่าย

## แนวทางพัฒนาที่แนะนำ

- **ระยะ 1 — Diagnostic MVP:** ทำเครื่องมือเชื่อมต่อ/อ่านข้อมูลแบบ read-only สำหรับ ECU รุ่นเดียว ใช้ log replay และตรวจเฟรม/ความยาว/checksum อย่างเข้มงวด
- **ระยะ 2 — ECU/file identification:** ทำฐานข้อมูลรุ่น ECU, อ่านและบันทึกข้อมูลที่ได้รับอนุญาต, ตรวจ version/hash และทำหน้าจอเตือนเมื่อไฟล์ไม่ตรงรุ่น โดยยังไม่เขียนกลับ
- **ระยะ 3 — Reflash ในห้องทดลองเท่านั้น:** เพิ่มการเขียนก็ต่อเมื่อมีโปรโตคอลที่ได้รับอนุญาตและแผน recovery ที่พิสูจน์แล้ว ใช้ bench ECU และทดสอบความล้มเหลวให้ครบก่อน
- **ระยะ 4 — Productization:** เพิ่มการอัปเดตที่ลงนาม, logging, UX ป้องกันผู้ใช้เลือก ECU/firmware ผิด และกระบวนการ QA/กฎหมาย

## ข้อสรุปสำหรับการตัดสินใจ

จาก repo นี้ **เริ่มทำ scanner/reader ได้ แต่ยังยืนยันไม่ได้ว่าจะสร้าง flasher ที่ปลอดภัยและใช้งานได้จริงจากข้อมูลสาธารณะเพียงอย่างเดียว** ทางที่เหมาะที่สุดคือเลือก ECU หนึ่งรุ่นก่อน แล้วแยกงานเป็น “วินิจฉัย/อ่าน” กับ “เขียน reflash” อย่างชัดเจน การเขียนควรรอจนได้เอกสารและวิธี recovery ที่ได้รับอนุญาต

> หมายเหตุความปลอดภัย: การเขียน ECU ผิดรุ่นหรือขาดช่วงอาจทำให้ ECU ใช้งานไม่ได้หรือทำให้การควบคุมเครื่องยนต์ผิดปกติ รายงานนี้ไม่ได้รับรองความถูกต้องของไฟล์/โปรโตคอล และไม่ได้ตรวจสอบความปลอดภัยของไบนารีที่เผยแพร่

## แหล่งอ้างอิงหลัก

- [โปรไฟล์ AutotronicCommunity](https://github.com/AutotronicCommunity)
- [Honda Keihin K-Line Protocol](https://github.com/AutotronicCommunity/Honda_Keihin_KLine_Protocol)
- [ESP32 Honda Scantool source](https://github.com/AutotronicCommunity/scantoolHondaEsp32)
- [Honda ECU Connector source](https://github.com/AutotronicCommunity/ADX-TUNER-PRO-ECU-CONNECTOR)
- [SoftwareHondaFlasher](https://github.com/AutotronicCommunity/SoftwareHondaFlasher)
- [SoftwareHondaECU](https://github.com/AutotronicCommunity/SoftwareHondaECU)
- [Yamaha Flash Tool V6 Demo](https://github.com/AutotronicCommunity/YAMAHA-FLASH-TOOL-V6-DEMO-VERSION)
- [Yamaha Aerox BIN/XDF](https://github.com/AutotronicCommunity/YAMAHA-AEROX-XDF--BIN)
- [Honda pinout](https://github.com/AutotronicCommunity/pinouECMHonda) · [Yamaha pinout](https://github.com/AutotronicCommunity/pinoutECMYamaha)
- [MX King V1 Arduino read example](https://github.com/AutotronicCommunity/ARDUINO_READ_ECU_MX_KING_V1)
- [Honda Motorcycle ECU Tools (fork)](https://github.com/AutotronicCommunity/Honda-Motorcycle-ECU-Tools)
