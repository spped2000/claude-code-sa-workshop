# System Analysis — Rental Contract Module

> **Reference output สำหรับ Demo A** · generated โดย `mango-sa-analyzer` skill จาก `mock-data/requirement_rental_TH.txt` + `mock-data/rental_contract_MOCK.csv` (instructor fallback ถ้า live demo พัง)

---

## 1. ภาพรวมระบบ

ระบบบริหารสัญญาเช่าอสังหาริมทรัพย์ (คอนโด/ร้านค้า/สำนักงาน) สำหรับบริษัท อสังหาฯ สมมุติ จำกัด แทนที่ Excel เดิมที่ใช้ติดตามสัญญา ครอบคลุมตั้งแต่:

- ร่างสัญญา → ผู้เช่าเซ็น → เก็บค่าเช่ารายเดือน
- แจ้งเตือนก่อนสัญญาหมดอายุ 60 วัน + รองรับการต่ออายุ
- เก็บเงินประกันแบบรวมศูนย์ (เดิมกระจายหลายบัญชี)
- Dashboard ผู้บริหารแบบ self-service (เดิมทำมือทุกเดือน)

**ขอบเขตเทคโนโลยี:** Backend Python, PostgreSQL, on-premise Windows Server 2019 (ลูกค้ากำหนด); Frontend ยังเปิดให้แนะนำ

**ข้อจำกัดโครงการ:** Go-live ภายใน 4 เดือน, งบประมาณ 1.5 ล้านบาท, ทีม 5 คน (Mango 3 + ลูกค้า 2)

---

## 2. Actors และ Use Cases

### Actors

| Actor | Role | สิทธิ์หลัก |
|---|---|---|
| **เจ้าหน้าที่การตลาด** | สร้างสัญญาใหม่ บันทึกข้อมูลผู้เช่า | CRUD: tenant, contract (draft) |
| **เจ้าหน้าที่การเงิน** | ออกใบแจ้งหนี้ รับชำระ จัดการเงินประกัน | CRUD: invoice, payment, deposit |
| **ผู้บริหาร** | ดู dashboard อัตราการเช่า รายได้ค้างรับ | Read-only: aggregate reports |
| **ผู้เช่า (Tenant)** | ผู้รับใบแจ้งหนี้/notification (passive actor) | Receive notifications |
| **ระบบบัญชี (External)** | รับ/ส่ง transaction (integration) | API consumer/provider |

### Use Cases หลัก

1. **UC-01:** สร้างสัญญาใหม่ (Marketing → กรอกข้อมูลผู้เช่า + เลือกหน่วย + คำนวณเงินประกัน)
2. **UC-02:** ออกใบแจ้งหนี้รายเดือน (Finance → trigger อัตโนมัติทุกวันที่กำหนด)
3. **UC-03:** บันทึกการชำระเงิน + อัปเดตสถานะ (Finance)
4. **UC-04:** แจ้งเตือนก่อนสัญญาหมดอายุ 60 วัน (System → Marketing + Tenant)
5. **UC-05:** ต่ออายุสัญญา (Marketing → reuse existing tenant + unit, สร้าง contract ใหม่)
6. **UC-06:** ล็อค Key Card อัตโนมัติเมื่อค้างชำระ >15 วัน (System → external door access API)
7. **UC-07:** ดู Dashboard อัตราการเช่า / รายได้ค้างรับ (Executive)
8. **UC-08:** คืนเงินประกันเมื่อสัญญาสิ้นสุด (Finance)

---

## 3. Business Rules

| BR | Rule | ที่มา |
|---|---|---|
| **BR-01** | `monthly_rent` ต้องอยู่ระหว่าง 5,000 ≤ x ≤ 500,000 บาท/เดือน | requirement line 25 |
| **BR-02** | `deposit` = `monthly_rent × 2` สำหรับที่อยู่อาศัย (คอนโด), `monthly_rent × 3` สำหรับพาณิชย์ (ร้านค้า/สำนักงาน) | line 26 |
| **BR-03** | ระยะเวลาสัญญา (`end_date - start_date`) ≥ 180 วัน (6 เดือน) | line 27 |
| **BR-04** | ผู้เช่าคนเดียว (national_id เดียวกัน) ห้ามมี contract status = `ใช้งาน` ซ้อนกันในหน่วยเดียวกัน (`unit_id`) | line 24 |
| **BR-05** | หน่วยที่ `unit_status = "กำลังซ่อมบำรุง"` ต้อง reject การสร้าง contract ใหม่ (HTTP 409) | line 28 |
| **BR-06** | ถ้า payment ค้าง > 15 วันนับจาก due_date → trigger ล็อค Key Card อัตโนมัติ (call external API) | line 29 |
| **BR-07** | ระบบต้องส่ง notification ก่อน end_date 60 วัน (UC-04) | line 15 |
| **BR-08** | Contract `status` enum จำกัดที่ {ใช้งาน, หมดอายุ, ยกเลิก} (สอดคล้องกับ CSV mock) | derived from `rental_contract_MOCK.csv` |

> **Cross-check:** BR-01 ถึง BR-06 ทุกข้อ testable ด้วยตัวเลขหรือ enum — ไม่ใช้คำว่า "ระบบควรเร็ว/ดี/มีประสิทธิภาพ"

---

## 4. Non-functional Requirements

### Performance
- **NFR-01:** GET /contracts (list, paginated 50/page) p95 ≤ 500ms
- **NFR-02:** Dashboard KPI aggregate query p95 ≤ 2s (ถ้าเกิน → cache ทุก 5 นาที)
- **NFR-03:** Notification job (BR-07) ทำงาน batch ทุก 06:00 น. สำเร็จภายใน 10 นาที (สูงสุด 10,000 contracts)

### Security
- **NFR-04:** ทุก endpoint ต้องผ่าน JWT auth + RBAC (Marketing/Finance/Executive)
- **NFR-05:** เลขบัตรประชาชน (national_id) ต้อง encrypt at rest (PostgreSQL pgcrypto) + mask ใน UI (เห็นเฉพาะ 4 หลักท้าย)
- **NFR-06:** ไฟล์สัญญา PDF เก็บใน folder ที่ ACL จำกัด, file path ไม่อยู่ใน DB plain text

### Availability
- **NFR-07:** Uptime ≥ 99% (~7 ชม. downtime/เดือน) — เนื่องจาก on-prem ไม่ใช่ HA
- **NFR-08:** Backup PostgreSQL daily, restore RTO ≤ 4 ชม.

### Scalability
- **NFR-09:** รองรับ 5,000 active contracts, 50 concurrent users (ตามขนาดบริษัท)
- **NFR-10:** ตาราง `payment` คาด growth ~60,000 row/ปี (5,000 × 12) → partitioning by year หลังปี 3

---

## 5. Assumptions (ข้อสมมุติฐาน — ต้องให้ stakeholder ยืนยัน)

> **สำคัญ:** ทุก assumption เกิดจากจุดที่ requirement ไม่ระบุชัด — SA propose default ไว้เพื่อไม่บล็อค dev แต่ต้อง confirm ก่อน production

| A | จุดคลุมเครือ | Proposed Default | Action ต้องทำ |
|---|---|---|---|
| **A-01** | Requirement บอก "ส่งใบแจ้งหนี้อัตโนมัติ" แต่ไม่ระบุช่องทาง (email/SMS/LINE) | **Default: Email** เพราะมี email ในข้อมูลผู้เช่าอยู่แล้ว (ดู requirement line 32). LINE ต้องการ LINE Notify token + onboarding ผู้เช่า | ขอ stakeholder ยืนยันใน meeting kickoff |
| **A-02** | "ผู้บริหารต้องการ real-time" — ไม่ระบุ granularity (วินาที/นาที/ชั่วโมง) | **Default: ทุก 5 นาที** (cache refresh) — เพียงพอสำหรับ KPI dashboard, ลด DB load | ดูตัวอย่าง dashboard ใกล้เคียงกับลูกค้าก่อน sign-off |
| **A-03** | "คืนเงินประกัน" — ไม่ระบุเงื่อนไขการหัก (ค่าซ่อม, ทำความสะอาด, ค่าน้ำ-ไฟค้าง) | **Default: ระบบบันทึก deduction items แบบ free-text + amount, ผู้ใช้ Finance กรอกเอง** ไม่ทำ auto-calc deduction | นัด workshop กับฝั่งบริหารอาคารหา rule จริง |
| **A-04** | "เชื่อมต่อระบบบัญชี" — ไม่ระบุระบบ (ERP เดิม? QuickBooks? Xero?) | **Default: Export CSV รายเดือนสำหรับ import มือ** (ทำ export endpoint ก่อน) — Phase 2 ค่อยทำ direct integration | ขอชื่อระบบ + API doc ก่อน sprint 3 |
| **A-05** | "ล็อค Key Card อัตโนมัติ" (BR-06) — ไม่มีข้อมูลว่าระบบ Key Card คือยี่ห้อไหน + มี API หรือไม่ | **Default: เขียน adapter pattern + interface `KeyCardLocker`** สำหรับ inject implementation ภายหลัง; mock ไว้ก่อนสำหรับ test | ขอเอกสาร integration spec ของ Key Card vendor |
| **A-06** | "ทีม dev ฝั่งลูกค้า 2 คน" — ไม่ระบุ skill (Python? PostgreSQL?) | **Default: Mango ทำ backend หลัก, ฝั่งลูกค้ารับผิดชอบ frontend หลังได้ API doc** | คุยกับ PM ฝั่งลูกค้า confirm role |

> **ถ้าไม่มี assumption จุดไหนเลย** = req อาจสมบูรณ์ผิดปกติ → ตรวจสอบใหม่อีกรอบ

---

## Cross-check Summary (สำหรับ instructor)

| Output artifact | สิ่งที่ตรวจ | ผ่าน |
|---|---|---|
| system_analysis.md | 5 sections ครบ | ✅ |
| BR count | 8 BRs (ขั้นต่ำ 5) | ✅ |
| Assumption count | 6 assumptions พร้อม proposed default (ขั้นต่ำ 3) | ✅ |
| CSV field cross-ref | ทุก field ใน BR/Use Case match `rental_contract_MOCK.csv` header | ✅ |

---

*Generated: workshop reference output · Source: `mock-data/requirement_rental_TH.txt` + `mock-data/rental_contract_MOCK.csv`*
