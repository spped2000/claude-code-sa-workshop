# Exercise 1 — SA Handoff: OT Requirement → Full Spec

> **ช่วง:** หลัง Demo A (session 5.1) · **เวลา:** 30 นาที · **คะแนนผ่าน:** 70/100

## Scenario
ลูกค้า (HR ของ Mango) ส่ง requirement ระบบคำนวณโอทีมาทาง LINE แบบสั้นๆ
คุณเป็น SA ต้องแปลงเป็น spec ที่ Dev เริ่ม implement ได้เลย

## Inputs
1. `exercises/ex1_ot_requirement_TH.txt` — ข้อความ requirement จากลูกค้า (คลุมเครือจงใจ)
2. `mock-data/hr_employee_MOCK.csv` — โครงสร้างข้อมูลพนักงานปัจจุบัน

## Tasks

### Task 1 — System Analysis (25 คะแนน)
สร้างไฟล์ `answers/ex1/system_analysis.md` (Thai) ต้องมี:
1. ภาพรวมระบบ
2. Actors & Use Cases (≥3 actors, ≥6 use cases)
3. **Business Rules** — label BR-01, BR-02, ... ต้องมีอย่างน้อย 5 ข้อ
4. Non-functional requirements (performance, security, availability)
5. **Assumptions** — flag จุดคลุมเครือใน requirement **อย่างน้อย 3 ข้อ**

### Task 2 — ER Diagram (25 คะแนน)
สร้าง `answers/ex1/er_diagram.mmd` (Mermaid) ต้องมี:
- Entities อย่างน้อย 4: `Employee`, `OTRequest`, `OTRate`, `Approval`
- Cardinality ทุก relationship
- PK/FK markers
- Thai comments อธิบายแต่ละ entity

### Task 3 — Technical Spec (25 คะแนน)
สร้าง `answers/ex1/tech_spec.md` ต้องมี:
- API endpoints table (method | path | request | response | auth)
- **PostgreSQL DDL** (`CREATE TABLE`) — ต้อง run ได้จริงใน `psql`
- Mermaid sequence diagram สำหรับ flow "ขอ OT → หัวหน้าอนุมัติ"
- Validation rules table ที่ cross-reference BR-XX

### Task 4 — Token Discipline (25 คะแนน)
- ใช้ `/cost` อย่างน้อย 1 ครั้งระหว่างทำ exercise → capture screenshot
- ใช้ `/compact` เมื่อ context เริ่มยาว
- ปิดงานด้วยการให้ Claude สรุป 3 ไฟล์ที่สร้างใน 5 บรรทัด

## Self-check Sidebar — ทำทุก task ก่อนข้ามต่อไป

> **สำคัญ:** ก่อนส่ง Task ถัดไป ตรวจ checklist ของ Task ปัจจุบัน — ถ้าไม่ครบ ให้ re-prompt Claude แก้

### ✓ Self-check Task 1 (system_analysis.md)
- [ ] **5 sections** ครบ (ภาพรวม / Actors+UC / BR / NFR / Assumptions)
- [ ] **BR ≥5 ข้อ** ทุกข้อมี threshold ตัวเลขหรือ enum (ไม่ใช้คำ "เร็ว/ดี")
- [ ] **Assumption ≥3 ข้อ** แต่ละข้อมี: จุดคลุมเครือ + Proposed Default + Action
- [ ] Actors ≥3 + Use Cases ≥6

### ✓ Self-check Task 2 (er_diagram.mmd)
- [ ] เปิดใน VSCode + Mermaid Preview → **render ได้** ไม่ error
- [ ] **Entities ≥4** (Employee, OTRequest, OTRate, Approval ขั้นต่ำ)
- [ ] ทุก entity มี **PK marker**
- [ ] ทุก relationship มี **cardinality** (`||--o{` หรือคล้าย)
- [ ] Employee มี **self-reference** สำหรับ manager_id

### ✓ Self-check Task 3 (tech_spec.md)
- [ ] API endpoints **≥5**
- [ ] DDL: copy ไปทดสอบใน online Postgres validator → **ไม่ error**
- [ ] DDL มี **CHECK constraint** อย่างน้อย 2 ตัว link BR-XX
- [ ] Sequence diagram **render ได้** + แสดง validation step
- [ ] Validation rules table — **ทุก row link BR-XX**

### ✓ Self-check Task 4 (token discipline)
- [ ] เห็น `/cost` output ครั้งเดียว (screenshot ได้)
- [ ] เห็น `/compact` ทำงาน (Claude สรุป context)
- [ ] Claude สรุป 3 ไฟล์ใน 5 บรรทัด

---

## Suggested Claude Prompts (ใช้เป็น starter)

**Prompt 1 — warm up:**
```
Read exercises/ex1_ot_requirement_TH.txt and mock-data/hr_employee_MOCK.csv.
Confirm you've read both. Don't do anything yet.
```

**Prompt 2 — analysis:**
```
จากไฟล์ requirement ที่อ่านแล้ว สร้าง answers/ex1/system_analysis.md ในภาษาไทย
มีหัวข้อ: ภาพรวม, Actors & Use Cases, Business Rules (BR-XX), NFR, Assumptions.
Flag จุดคลุมเครือใน requirement อย่างน้อย 3 จุดใน section Assumptions.
```

**Prompt 3 — ER:**
```
จาก system_analysis.md ที่เพิ่งสร้าง + schema ของ hr_employee_MOCK.csv
generate answers/ex1/er_diagram.mmd (Mermaid erDiagram) มี 4 entities ขั้นต่ำ
และ cardinality ครบทุก relationship.
```

**Prompt 4 — tech spec:**
```
สร้าง answers/ex1/tech_spec.md — API endpoints table + PostgreSQL DDL
+ sequence diagram (Mermaid) สำหรับ flow ขอ OT + validation rules ที่ link BR-XX.
Keep under 500 lines.
```

---

## Expected Output — Answer Key Summary

(Instructor ใช้ตรวจ, ห้ามแจกก่อน)

### system_analysis.md
- **ควรมี Actors:** พนักงาน, หัวหน้า, HR, หัวหน้าของหัวหน้า (approval override)
- **ควรมี Use Cases:** ขอ OT, อนุมัติ OT, ปฏิเสธ OT, คำนวณเงิน OT, ดูรายงาน (รายคน/แผนก/เดือน/ปี)
- **BR ที่ควรเจอ (≥5):**
  - BR-01: โอทีธรรมดา rate 1.5 เท่า
  - BR-02: วันหยุด/นักขัตฤกษ์ rate 3 เท่า
  - BR-03: ขอเกิน 4 ชม./วัน ไม่ได้
  - BR-04: พนักงานใหม่ <3 เดือน ขอไม่ได้
  - BR-05: หัวหน้าอนุมัติของตัวเองไม่ได้
- **Assumptions ≥3 (เลือกจาก list ใน requirement file):** notification channel, ขอล่วงหน้า vs ย้อนหลัง, edit/cancel, OT rate PM, cap รายเดือน, วันหยุดใน system, approval delegation, export format

### er_diagram.mmd
- Employee (PK: employee_id) — FK: manager_id (self-reference)
- OTRequest (PK: request_id, FK: employee_id) — 1 Employee → many OTRequests
- OTRate (PK: rate_type) — lookup: weekday/weekend/holiday
- Approval (PK: approval_id, FK: request_id, approver_id) — 1 OTRequest → 1 Approval

### tech_spec.md
- Endpoints ขั้นต่ำ: POST /ot, GET /ot/:id, POST /ot/:id/approve, POST /ot/:id/reject, GET /ot?employee_id=…
- DDL มี CHECK constraints (hours BETWEEN 0.5 AND 4, rate_type IN (...))
- Sequence diagram ต้องแสดง validation steps
- Validation rules ≥5 rows, link BR-XX

---

## ⚠️ Common Pitfalls — ระวัง 5 จุด

| # | ปัญหา | ตัวอย่างที่พบ | Fix |
|---|---|---|---|
| 1 | **Fabricated field** — Claude สมมุติ field ที่ไม่มีใน CSV | DDL มี `phone_number`, `email` ทั้งที่ CSV ไม่มี | re-prompt: "ตรวจทุก field ใน DDL ต้องมีใน hr_employee_MOCK.csv header" |
| 2 | **Weak BR** — เขียน vague ไม่ testable | "BR-01: ระบบต้องคำนวณ OT ถูกต้อง" | re-prompt: "เขียน BR ใหม่ให้มี numeric threshold หรือ enum specific เช่น `hours ≤ 4`" |
| 3 | **Missing cardinality** — ER มี relationship แต่ไม่บอก 1:N | `Employee -- OTRequest` (ไม่มี cardinality) | re-prompt: "ระบุ cardinality ทุก relationship: `||--o{`, `}o--o{`, ฯลฯ" |
| 4 | **Vague assumption** — flag จุดคลุมเครือแต่ไม่ propose default | "A-01: อาจมีรายละเอียดเพิ่มเติม" | re-prompt: "เขียน Assumption ใหม่ ทุกข้อต้องมี 3 ส่วน: จุดคลุมเครือ + Proposed Default + Action" |
| 5 | **No BR-XX cross-ref** — validation rules ไม่ link BR | Validation table มีแค่ field + rule ไม่มี column "BR" | re-prompt: "เพิ่มคอลัมน์ BR-XX ใน validation rules table — ทุก rule ต้อง link" |

> **Tip:** ถ้าทำ Task 3 แล้ว Pitfall #1 หรือ #5 เกิด — ไม่ต้องตื่นตระหนก กลับมา re-prompt ในไฟล์เดิมพอ ไม่ต้องเริ่มใหม่

---

## Grading Rubric

| เกณฑ์ | คะแนน | เงื่อนไขผ่าน |
|---|---|---|
| system_analysis.md ครบ 5 sections + BR ≥5 + Assumptions ≥3 | 25 | ได้เต็มถ้าครบ, -5 ต่อส่วนที่ขาด |
| er_diagram.mmd render ได้ + entities ≥4 + cardinality ครบ | 25 | render ใน Mermaid preview ได้ = เต็ม |
| tech_spec.md + DDL ที่ `psql -f` ไม่มี error | 25 | instructor test ด้วย `psql --dry-run` |
| ใช้ `/cost` + `/compact` + summary 5 บรรทัด | 25 | เห็นการใช้จริง = เต็ม |

**ผ่าน:** ≥70/100 คะแนนรวม

---

## 🔓 Answer Key — เปิดได้หลังส่งงานเท่านั้น

หลัง instructor ปิดรับส่ง — เทียบงานตัวเองกับ:
- [`answer-keys/ex1/system_analysis.md`](../answer-keys/ex1/system_analysis.md)
- [`answer-keys/ex1/er_diagram.mmd`](../answer-keys/ex1/er_diagram.mmd)
- [`answer-keys/ex1/tech_spec.md`](../answer-keys/ex1/tech_spec.md)

> **วิธี diff ตัวเอง:** เปิด 2 ไฟล์ side-by-side ใน VSCode → ดูว่า BR ใดที่ตัวเองหายไป, Assumption ใดที่ answer key flag แต่ตัวเองไม่ได้ flag — นี่คือ **learning gap** ที่จะทำให้รอบหน้าดีขึ้น
