# RTM: จองคิวตรวจสุขภาพ (Booking)
อ้างอิง: spec.md Draft v2 | tasks.md | test-cases.md
สร้างด้วย /verify เมื่อ 2569-10-07 09:05 | test: backend 4 passed / frontend 1 passed

## 1. ตามรอยไปข้างหน้า (requirement ไป โค้ด ไป test)
| ID | AC | task | โค้ด (ไฟล์: ฟังก์ชัน) | test (ผล) | สถานะ |
|---|---|---|---|---|---|
| FR-BKG-01 | AC-BKG-05 | T-02 | backend/app/slots/service.py:list_available_slots; backend/app/slots/router.py:get_slots | backend/tests/test_AC_BKG_05.py::test_AC_BKG_05 (ผ่าน) | ช่องโหว่ |
| FR-BKG-02 | AC-BKG-02 | T-04 | ไม่มีรหัสที่ตรวจคิวที่ยังไม่ได้ใช้ในวันเดียวกัน | ไม่มี | ยังไม่ถึง |
| FR-BKG-03 | AC-BKG-03 | T-05, T-11, T-12 | ไม่มีรหัสที่เสนอ 3 ช่วงว่างและตรวจว่ามีการจองซ้อน | ไม่มี | ยังไม่ถึง |
| FR-BKG-04 | AC-BKG-01 | T-03, T-06 | backend/app/booking/service.py:create_booking; backend/app/booking/router.py:create_booking | backend/tests/test_AC_BKG_01.py::test_AC_BKG_01 (ผ่าน) | รอ Q-xx |
| FR-BKG-05 | AC-BKG-04 | T-07 | ไม่มีระบบคิวส่งข้อความซ้ำและไม่มีฟังก์ชันบันทึกคิวค้างส่ง | ไม่มี | ยังไม่ถึง |
| FR-BKG-06 | ไม่มี AC | T-02, T-10 | backend/app/slots/service.py:list_available_slots; backend/app/slots/router.py:get_slots | backend/tests/test_AC_BKG_05.py::test_AC_BKG_05 (ผ่าน) | ช่องโหว่ |
| NFR-PERF-01 | AC-BKG-05 | T-02 | backend/app/slots/service.py:list_available_slots | backend/tests/test_AC_BKG_05.py::test_AC_BKG_05 (ผ่าน) | ครบ |
| NFR-SEC-01 | ไม่มี AC | ไม่มี task | ไม่มีโค้ดเปิด HTTPS/TLS หรือการเข้ารหัสข้อมูลรับส่ง | ไม่มี | ยังไม่ถึง |
| NFR-REL-02 | AC-BKG-04 | T-07 | ไม่มีการส่งซ้ำภายใน 5 นาที และไม่มี queue retry | ไม่มี | ยังไม่ถึง |
| NFR-USE-01 | ไม่มี AC | ไม่มี task | ไม่มี flow ทดสอบผู้ใช้ใหม่ 8/10 คนภายใน 3 นาที | ไม่มี | ยังไม่ถึง |
| CON-TECH-01 | ไม่มี AC | T-01 | backend/app/config.py:DATABASE_URL (ค่าเริ่มต้น sqlite:///./dev.db) | backend/tests/test_T01_schema.py (ผ่าน) | ช่องโหว่ |
| DOM-PDPA-01 | AC-BKG-06 | T-08 | backend/app/db/models.py:AuditLog และ backend/app/main.py:lifespan ไม่มี middleware บันทึก audit log | ไม่มี | ยังไม่ถึง |
| IF-IDP-01 | ไม่มี AC | T-03 | backend/app/auth/idp.py:get_verified_hn | backend/tests/test_AC_BKG_01.py::test_AC_BKG_01 (ผ่าน) | ครบ |
| IF-HIS-01 | ไม่มี AC | T-09 | ไม่มี backend/app/his/client.py และไม่มีการค้น HN จาก HIS | ไม่มี | ยังไม่ถึง |
| IF-NOT-01 | AC-BKG-04 | T-07 | ไม่มี async queue หรือ worker ส่งข้อความซ้ำ | ไม่มี | ยังไม่ถึง |

## 2. ตามรอยย้อนกลับ (โค้ด ไป requirement)
| โค้ด (ไฟล์: ฟังก์ชัน หรือ endpoint) | อ้าง ID | ตรงกับข้อความใน spec ไหม | หมายเหตุ |
|---|---|---|---|
| backend/app/slots/router.py:get_slots | FR-BKG-01, FR-BKG-06 | ไม่ครบ | แสดงช่วงว่างและกรอง package_code ถูก แต่ใช้ช่วงเวลาเพียง 14 วัน ไม่ตรง spec ที่ต้องการภายใน 30 วันข้างหน้า |
| backend/app/booking/router.py:create_booking | FR-BKG-04, IF-IDP-01 | ส่วนหนึ่งตรง | ตรวจ Authorization แล้วบันทึก booking และตัดที่นั่งได้ แต่ยังไม่มีการปฏิเสธคิวที่ยังไม่ได้ใช้ในวันเดียวกัน และยังไม่มีการส่งข้อความยืนยันตาม IF-NOT-01 |
| backend/app/booking/service.py:create_booking | FR-BKG-02, FR-BKG-04 | ไม่ครบ | ไม่มีการตรวจว่าผู้รับบริการมีคิวที่ยังไม่ได้ใช้ในวันเดียวกัน และ queue_no ถูกสุ่ม/เดาเป็น A001 โดยไม่รอคำตอบ Q-02 |
| backend/app/config.py:DATABASE_URL | CON-TECH-01 | ไม่ตรง | ค่าเริ่มต้นเป็น SQLite จึงไม่บังคับใช้ PostgreSQL ในทุก environment |
| backend/app/booking/router.py:cancel_booking | Out of scope | ไม่ตรง | DELETE /bookings/{booking_id} เป็นฟีเจอร์ยกเลิกคิวซึ่งอยู่ใน Out of scope และไม่เกี่ยวข้องกับ UC-01 |
| backend/app/db/models.py:Booking | IF-HIS-01 | ส่วนหนึ่งตรง | มีคอลัมน์ hn เท่านั้นและไม่มี national_id แต่ยังไม่มี logic ค้น HN จาก HIS ก่อนจอง |

## 3. ข้อค้นพบ
ชนิด: AC ไม่มี test / test อ่อน / โค้ดไม่มี FR / FR ไม่มี AC / เดา Q-xx / ละเมิด Constraint / ตัวเลขไม่ตรง spec / อ้าง ID ผิดเรื่อง
ทีมตัดสิน: แก้โค้ด / แก้ spec / เพิ่ม Q-xx / ไม่ใช่ปัญหา (พร้อมเหตุผล 1 บรรทัด)

| F-ID | ชนิด | อยู่ที่ | ขัดกับ | รายละเอียด | ทีมตัดสิน |
|---|---|---|---|---|---|
| F-01 | ตัวเลขไม่ตรง spec | backend/app/slots/service.py:list_available_slots | FR-BKG-01 | โค้ดตั้ง DAYS_AHEAD = 14 แต่ spec ระบุภายใน 30 วันข้างหน้า จึงไม่ตรงกับ requirement ที่ผู้ใช้ได้รับ | แก้โค้ด |
| F-02 | ละเมิด Constraint | backend/app/config.py:DATABASE_URL | CON-TECH-01 | ค่าเริ่มต้นสภาวะเป็น sqlite:///./dev.db ซึ่งไม่เป็น PostgreSQL และทำให้ระบบอาจรันบนฐานข้อมูลผิดกฎของโรงพยาบาล | แก้โค้ด |
| F-03 | เดา Q-xx | backend/app/booking/service.py:next_queue_no | FR-BKG-04 และ Q-02 | รหัสกำหนด queue_no เป็น A001, A002 ... โดยอิงความคาดหวังที่ยังไม่ได้รับคำตอบจากเจ้าหน้าที่เวชระเบียน จึงเป็นการเดาแทนการปฏิบัติตาม spec | เพิ่ม Q-xx |
| F-04 | โค้ดไม่มี FR | backend/app/booking/router.py:cancel_booking | Out of scope | มี endpoint DELETE /bookings/{booking_id} ซึ่งเป็นฟังก์ชันยกเลิกคิว ไม่อยู่ใน UC-01 และขัดกับหัวข้อ Out of scope | แก้โค้ด |
| F-05 | FR ไม่มี AC | backend/app/slots/service.py:list_available_slots | FR-BKG-06 | requirement ระบุการเปลี่ยนแพ็กเกจแล้วคำนวณช่วงว่างใหม่ แต่ spec ไม่มี AC ที่ตรวจตรงนี้ จึงไม่สามารถยืนยันว่ามีการทดสอบตาม Then ครบ | แก้ spec |

## 4. แก้แล้ว
| F-ID | แก้อย่างไร | รู้ได้อย่างไร |
|---|---|---|
