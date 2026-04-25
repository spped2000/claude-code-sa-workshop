---
name: mango-sa-analyzer
description: วิเคราะห์ requirement ภาษาไทยสำหรับ Mango ERP แปลงเป็น System Analysis + ER Diagram + Technical Spec ตาม format ที่ Mango SA ใช้จริง flag assumptions ทุกครั้งที่ requirement กำกวม
---

# Mango SA Analyzer Skill

## เมื่อไหร่ใช้
- SA รับ requirement จากลูกค้า Mango (Thai narrative) ต้องการแปลงเป็น technical spec
- มี `requirement_*.txt` หรือ requirement ที่ paste เข้ามา
- ต้องการ output ที่ dev เริ่ม implement ได้

## Output Contract (บังคับ)

ทุกครั้งที่เรียก skill นี้ ต้องสร้าง 3 ไฟล์:

### 1. `analysis/system_analysis.md` (Thai)
โครงสร้างคงที่:
```
# System Analysis — <Module Name>

## 1. ภาพรวมระบบ
## 2. Actors และ Use Cases
## 3. Business Rules
  - BR-01: <rule>
  - BR-02: <rule>
  - ... (ขั้นต่ำ 5 ข้อ ถ้าน้อยกว่า → warn)
## 4. Non-functional Requirements
  - Performance / Security / Availability / Scalability
## 5. Assumptions (สำคัญ!)
  - A-01: <จุดที่ requirement กำกวม>
  - A-02: ...
  - ... (ขั้นต่ำ 3 ข้อ — ถ้า req สมบูรณ์จริง ให้ write "ไม่พบจุดคลุมเครือ")
```

### 2. `analysis/er_diagram.mmd` (Mermaid)
```
erDiagram
  %% ใส่ Thai comment ต่อ entity
  ENTITY_NAME {
    type field_name PK "Thai description"
    type field_name FK
  }
  ENTITY_A ||--o{ ENTITY_B : "Thai relationship label"
```
- Cardinality ทุก relationship (`||--o{`, `}o--o{`, ฯลฯ)
- PK/FK markers
- Thai comments
- จบด้วย **ASCII summary ใน response** (text block) เพื่อให้ user sanity-check ก่อนเปิด preview

### 3. `analysis/tech_spec.md` (Thai + English tech terms)
- **API endpoints table**: method | path | request | response | auth
- **PostgreSQL DDL** — `CREATE TABLE` with CHECK constraints ที่ link กับ BR-XX
- **Sequence diagram** (Mermaid) — 1 flow หลัก
- **Validation rules table** — ทุก rule ต้อง reference BR-XX
- **Story points** — ต่อ endpoint (Fibonacci 1/2/3/5/8)

## Hard Rules

### 1. ภาษา
- **Content ทั้งหมดเป็นภาษาไทย** (ยกเว้น code/SQL/class/tech term)
- Technical term ที่ไม่แปล: entity, API, endpoint, PK, FK, BR-XX, DDL

### 2. Data constraint จาก CSV
ถ้า module มี mock CSV ให้ **อิง schema จาก CSV จริง** — ไม่ invent field ขึ้นมาเอง:
- Rental → `rental_contract_MOCK.csv`
- Construction → `construction_project_MOCK.csv` + `boq_MOCK.csv`
- HR → `hr_employee_MOCK.csv`

### 3. Business rules (BR-XX) ต้อง testable
❌ Bad: `BR-01: ระบบต้องเร็ว`
✅ Good: `BR-01: Response time ของ GET /contracts ≤ 500ms ที่ p95`

### 4. Assumptions ต้องเจาะจง (flag actionable)
❌ Bad: `A-01: อาจมีข้อมูลเพิ่มเติม`
✅ Good: `A-01: Requirement ไม่ระบุช่องทางส่ง invoice (email? SMS? LINE?) — สมมุติใช้ email เป็น default, ให้ stakeholder ยืนยัน`

### 5. Token management
หลังสร้าง 3 ไฟล์เสร็จ → แนะนำให้ user `/compact` + เตือน `/cost`

## Prompt patterns

### Pattern A — แปลง requirement file
```
@skill mango-sa-analyzer
Analyze mock-data/requirement_rental_TH.txt.
```

### Pattern B — Paste requirement
```
@skill mango-sa-analyzer
Requirement: "ลูกค้าอยากได้ระบบจอง... (narrative)"
Save to analysis/booking/.
```

## Cross-check step (บังคับ)

หลังเขียน 3 ไฟล์เสร็จ ต้องสรุป:

| Output artifact | BR count | Assumption count | CSV field cross-ref |
|---|---|---|---|
| system_analysis.md | 7 BRs | 4 assumptions | ✅ |
| er_diagram.mmd | 5 entities | — | ✅ match CSV schema |
| tech_spec.md | 6 endpoints | DDL lines: 42 | ✅ |

แล้วถาม user: "ต้องการให้ผม run `psql --dry-run` test DDL ไหม?"

## Version
v1.0 — 2026-04-22 · สำหรับ Mango workshop Section 5.1
