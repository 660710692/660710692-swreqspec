# แผนฟีเจอร์จองคิวตรวจสุขภาพ (Booking)

## 1. สรุปแนวทาง

ฟีเจอร์นี้ให้ผู้รับบริการที่ยืนยันตัวตนแล้วเลือกแพ็กเกจ วัน และช่วงเวลาตรวจภายใน 30 วัน เพื่อสร้างการจองและรับหมายเลขคิว (FR-BKG-01, FR-BKG-04)  
ระบบจะแสดงจำนวนที่นั่งคงเหลือ ตรวจสอบการมีคิวเดิมในวันเดียวกัน และป้องกันการจองซ้ำเมื่อมีผู้ใช้รายอื่นยืนยันก่อน (FR-BKG-01, FR-BKG-02, FR-BKG-03)  
การยืนยันการจองจะทำรายการตัดที่นั่งและออกหมายเลขคิวเป็นธุรกรรมเดียวกัน เพื่อให้ผลลัพธ์ไม่เกิดรายการจองซ้อน (FR-BKG-03, FR-BKG-04)  
การส่งข้อความยืนยันจะส่งผ่านคิวแบบ asynchronous และเก็บงานที่ล้มเหลวเพื่อส่งซ้ำภายใน 5 นาที โดยไม่ทำให้การจองล้มเหลว (IF-NOT-01, FR-BKG-05, NFR-REL-02)  
ระบบจะตรวจสิทธิ์จาก IDP, ใช้ HN เป็นข้อมูลอ้างอิงจาก HIS โดยไม่เก็บเลขบัตรประชาชนในตารางการจอง และบันทึก audit log ตาม DOM-PDPA-01 (IF-IDP-01, IF-HIS-01, DOM-PDPA-01)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| MySQL | CON-TECH-01 | ใช้เป็นฐานข้อมูลการจองและข้อมูลที่เกี่ยวข้องตามมาตรฐานฝ่าย IT |
| React (Vite) | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นของรายวิชาสำหรับหน้าบ้าน |
| Python FastAPI | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นของรายวิชาสำหรับหลังบ้าน |
| กลไกคิวงาน asynchronous | IF-NOT-01 | ต้องรองรับการส่งคำขอแจ้งเตือนโดยไม่รอผลลัพธ์ |
| TLS 1.2 ขึ้นไป | NFR-SEC-01 | ใช้กับการรับส่งข้อมูลการจอง |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับ |
|---|---|---|
| Booking | booking_id, HN, package_id, slot_id, booking_date, queue_number, status, created_at | FR-BKG-02, FR-BKG-04, FR-BKG-05 |
| Package | package_id, package_name, ข้อมูลที่ใช้กำหนดช่วงเวลาตามแพ็กเกจ (รอคำตอบ Q ที่เกี่ยวข้อง) | FR-BKG-06 |
| TimeSlot | slot_id, service_date, start_time, end_time, quota_reference, remaining_seats | FR-BKG-01, FR-BKG-03, FR-BKG-04, FR-BKG-06 |
| NotificationOutbox | notification_id, booking_id, channel, status, retry_at, attempt_count | FR-BKG-04, FR-BKG-05, NFR-REL-02, IF-NOT-01 |
| AuditLog | audit_id, accessor, accessed_at, HN/รหัสผู้รับบริการตามคำตอบของทีม, action | DOM-PDPA-01, AC-BKG-06 |
| PatientReference | HN และข้อมูลอ้างอิงที่ได้รับจาก HIS ตามสิทธิ์ | IF-IDP-01, IF-HIS-01 |

ไม่มีฟิลด์เลขบัตรประชาชนใน `Booking` ตาม IF-HIS-01 และข้อมูลโควตาหลักของ `TimeSlot` เป็นข้อมูลที่ UC-09 กำหนดและฟีเจอร์นี้อ่านมาใช้ตาม ASM-01 ไม่สร้างความสามารถจัดการโควตา (Out of scope: UC-09)

## 4. API / หน้าจอ

| รายการ | Input / Output หลัก | รองรับ |
|---|---|---|
| หน้าจอเลือกแพ็กเกจและช่วงเวลา | Input: package_id, ช่วงวันที่; Output: วัน/ช่วงเวลาที่ว่างและจำนวนที่นั่งคงเหลือ | FR-BKG-01, FR-BKG-06 |
| `GET /booking/availability` | Input: package_id, date range ไม่เกิน 30 วัน; Output: slot และ remaining seats | FR-BKG-01, FR-BKG-06 |
| `POST /booking/confirm` | Input: HN จากบริบทการยืนยันตัวตน, package_id, slot_id; Output: booking_id, queue_number หรือข้อผิดพลาดพร้อม booking เดิม/ตัวเลือกใกล้เคียง | FR-BKG-02, FR-BKG-03, FR-BKG-04 |
| หน้าจอผลการจอง | Input: ผลจากการยืนยัน; Output: หมายเลขคิวและสถานะการส่งข้อความ | FR-BKG-04, FR-BKG-05 |
| `POST /notifications/outbox` หรือ worker ภายในระบบ | Input: booking notification; Output: สถานะส่งสำเร็จ/รอส่งซ้ำ | FR-BKG-05, NFR-REL-02, IF-NOT-01 |
| `GET /patient/context` | Input: ผลยืนยันจาก IDP; Output: ข้อมูลผู้รับบริการที่อ้างอิงด้วย HN จาก HIS | IF-IDP-01, IF-HIS-01 |
| audit middleware/service | Input: การเข้าถึงข้อมูลการจอง; Output: audit log ที่เก็บอย่างน้อย 1 ปี | DOM-PDPA-01 |

รายละเอียดการเลือก “ช่วงเวลาใกล้เคียง” และรูปแบบหมายเลขคิวจะยังไม่ลงรายละเอียดจนกว่าจะตอบ Q-01 และ Q-02

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| CON-TECH-01 | โมเดลข้อมูลและ persistence ใช้ MySQL | ใช้แล้ว |
| DOM-PDPA-01 | Entity `AuditLog`, audit middleware/service และ test `AC-BKG-06` | ใช้แล้ว |
| IF-IDP-01 | ตรวจผลยืนยันตัวตนก่อน `GET /patient/context` และ `POST /booking/confirm` | ใช้แล้ว |
| IF-HIS-01 | `PatientReference`, การใช้ HN ใน `Booking` และการไม่เก็บเลขบัตรประชาชน | ใช้แล้ว |
| IF-NOT-01 | `NotificationOutbox` และ worker แบบ asynchronous ที่ไม่บล็อกการยืนยัน | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| AC-BKG-01 | `test_AC_BKG_01_confirm_booking_updates_seat` | เตรียม slot 09.00 น. เหลือ 1 ที่ ยืนยันการจอง ตรวจว่าบันทึก booking มี queue number และ remaining seats เป็น 0 |
| AC-BKG-02 | `test_AC_BKG_02_reject_existing_unused_booking` | เตรียม booking ที่ยังไม่ได้ใช้ในวันเดียวกัน ยืนยันรายการใหม่ ตรวจว่าถูกปฏิเสธและคืน queue number เดิม |
| AC-BKG-03 | `test_AC_BKG_03_handle_concurrent_slot_full` | จำลองผู้ใช้สองรายแย่งที่นั่งสุดท้าย ตรวจว่ารายการหลังได้รับข้อความ “ช่วงเวลาเต็ม” ได้ 3 ตัวเลือก และมี booking สำเร็จเพียงรายการเดียว |
| AC-BKG-04 | `test_AC_BKG_04_enqueue_failed_notification_within_five_minutes` | ทำให้ระบบแจ้งเตือนไม่ตอบสนอง ตรวจว่า booking ยังสำเร็จ แสดง queue number และ outbox มี retry_at ไม่เกิน 5 นาที |
| AC-BKG-05 | `test_AC_BKG_05_availability_p95_under_two_seconds_for_200_users` | ยิงคำขอค้นหาช่วงเวลาพร้อมกัน 200 คน วัด p95 และตรวจว่าค่าไม่เกิน 2 วินาที |
| AC-BKG-06 | `test_AC_BKG_06_write_audit_log_on_booking_access` | เปิดดูข้อมูลการจอง ตรวจว่ามี audit log ระบุผู้เข้าถึง เวลา และรหัสผู้รับบริการตามนิยามที่ทีมยืนยัน |

NFR-USE-01 ต้องทดสอบกับผู้ใช้ใหม่ 10 คนตามนิยามที่ทีมกำหนด โดยวัดเวลาตั้งแต่จุดเริ่มต้นที่กำหนดจนจองสำเร็จ และนับว่าต้องมีอย่างน้อย 8 คนที่ทำได้ภายใน 3 นาที

## 7. ลำดับงาน

1. กำหนด schema และ transaction สำหรับ `Booking`, `TimeSlot`, `Package` และ `NotificationOutbox` โดยไม่เก็บเลขบัตรประชาชน (FR-BKG-01, FR-BKG-04, IF-HIS-01)
2. เชื่อมบริบทการยืนยันตัวตนกับการค้นข้อมูล HIS และสร้าง patient context ด้วย HN (IF-IDP-01, IF-HIS-01)
3. สร้างการค้นหา availability ภายใน 30 วันและคำนวณตามแพ็กเกจ (FR-BKG-01, FR-BKG-06)
4. สร้างหน้าจอเลือกแพ็กเกจ/วัน/ช่วงเวลาและหน้าจอผลการจอง (FR-BKG-01, FR-BKG-04, FR-BKG-05)
5. สร้าง confirm transaction ที่ตรวจคิวเดิม ตัดที่นั่ง และออกหมายเลขคิวโดยป้องกัน race condition (FR-BKG-02, FR-BKG-03, FR-BKG-04)
6. สร้าง outbox/worker สำหรับการแจ้งเตือนแบบ asynchronous และ retry ภายใน 5 นาที (FR-BKG-05, NFR-REL-02, IF-NOT-01)
7. เพิ่ม audit log และ retention อย่างน้อย 1 ปี พร้อมการป้องกันการเข้าถึงที่ไม่ผ่าน IDP (DOM-PDPA-01, IF-IDP-01)
8. เพิ่ม integration, concurrency, performance และ usability tests ตาม AC-BKG-01 ถึง AC-BKG-06 และ NFR-USE-01
9. ตรวจการรับส่งข้อมูลผ่าน TLS 1.2 ขึ้นไปและทบทวน traceability กับทุก requirement (NFR-SEC-01)

## 8. สิ่งที่ยังไม่ทำ

- Q-01 “ช่วงเวลาใกล้เคียง” นับเฉพาะวันเดียวกัน หรือรวมวันถัดไปด้วย? -> ถามพยาบาลคัดกรอง  
  ส่วนการคัดเลือกและเรียงลำดับ 3 ตัวเลือกใน FR-BKG-03 จะยังไม่สร้างจนกว่าจะได้คำตอบ
- Q-02 หมายเลขคิวรีเซ็ตรายวัน หรือนับต่อเนื่อง? -> ถามเจ้าหน้าที่เวชระเบียน  
  ส่วนกติกาการออกหมายเลขคิวใน FR-BKG-04 จะยังไม่สรุปจนกว่าจะได้คำตอบ
- นิยามและจุดเริ่มต้นของ “ผู้ใช้ใหม่” และการเริ่มจับเวลาใน NFR-USE-01 ยังไม่ระบุ  
  แผนการวัด usability จะยังไม่ลงรายละเอียดจนกว่าทีมจะยืนยันนิยาม
- ผลของแพ็กเกจต่อระยะเวลาหรือโควตาของช่วงเวลาใน FR-BKG-06 ยังไม่ระบุ  
  จะยังไม่กำหนดสูตรคำนวณ availability จนกว่าจะได้ข้อมูลจากทีมคลินิก/ทีมเอง
- “รหัสผู้รับบริการ” ใน DOM-PDPA-01 และ AC-BKG-06 ยังไม่ระบุว่าเป็น HN หรือรหัสอื่น  
  จะยังไม่ล็อกฟิลด์ audit log จนกว่าจะได้คำตอบจากทีมความปลอดภัย/เจ้าหน้าที่เวชระเบียน
- การจัดการกรณี IDP, HIS หรือระบบแจ้งเตือนไม่ตอบสนองนอกกรณีที่ระบุใน FR-BKG-05 ยังไม่มีข้อกำหนด  
  จะไม่เพิ่มพฤติกรรมหรือ error flow นอก spec
