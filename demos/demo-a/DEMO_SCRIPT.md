# Demo A — SA Requirement-to-Spec (16 นาที)

> ใช้ในช่วง **5.1 SA + Claude Code** · instructor เปิด Claude Code ฉายจอให้ผู้เรียนดูสด
>
> **โครงสร้างใหม่ 5 step:** A1 (load) → A2a (draft BRs) → A2b (validate vs CSV) → A3 (ER) → A4 (spec)
> ทุก step จบด้วย ✓ Self-check ให้ผู้เรียนตามทัน

> **📂 Reference output (instructor fallback):** ถ้า live demo พัง — เปิด `reference-output/system_analysis.md`, `reference-output/er_diagram.mmd`, `reference-output/tech_spec.md` ใช้ฉายแทน

## Pre-flight (1 นาทีก่อนเริ่ม)
```bash
cd c:/Users/natdh/Documents/mango_claudecode/section5/demos/demo-a
# ตรวจว่าไฟล์พร้อม
ls mock-data/
# ควรมี: requirement_rental_TH.txt, rental_contract_MOCK.csv
claude
```

**Copy mock data ใน demo-a/mock-data/ ถ้ายังไม่มี:**
```bash
cp ../../mock-data/requirement_rental_TH.txt mock-data/
cp ../../mock-data/rental_contract_MOCK.csv mock-data/
```

**ข้อความเปิด (ก่อนพิมพ์ prompt):**
> "สมมุติเราเป็น SA ที่เพิ่งรับ requirement จากลูกค้าของ Mango ทาง LINE
> เราจะใช้ Claude Code แปลง raw requirement เป็น spec ที่ dev เริ่ม
> implement ได้ทันที — ลดเวลา drafting จาก 2-3 วันเหลือ ~20 นาที"

---

## Step A1 — Load Context (1 นาที)

**พิมพ์สด:**
```
Read mock-data/requirement_rental_TH.txt and mock-data/rental_contract_MOCK.csv.
Just confirm you've read both files and tell me the file sizes. Don't do anything else yet.
```

**Expected Claude response shape:**
- ใช้ Read tool 2 ครั้ง
- ตอบสั้น: "อ่าน requirement_rental_TH.txt (~2.5 KB) และ rental_contract_MOCK.csv (~500 B) แล้ว ต้องการให้ทำอะไรต่อไปครับ?"

**💡 Token Tip ที่พูดสด:**
> "สังเกตว่าผมสั่งให้แค่อ่านก่อน ยังไม่ให้ทำอะไร — นี่คือ pattern ที่ประหยัด
> token + ยืนยันว่า Claude อ่านไฟล์ถูก ก่อนจะสั่งงานหนักๆ"

**✓ Self-check หลัง A1 (ผู้เรียนเช็คตัวเอง):**
- [ ] เห็น Claude เรียก Read tool **2 ครั้ง** (requirement + CSV)
- [ ] Claude **ตอบสั้น** ไม่เกิน 3 บรรทัด (ไม่เริ่มวิเคราะห์เอง)
- [ ] ขนาดไฟล์: requirement ~5 KB, CSV ~1.5 KB

---

## Step A2a — Draft Business Rules + Assumptions (3 นาที)

**พิมพ์สด:**
```
From the requirement file, produce the FIRST DRAFT of analysis/system_analysis.md (Thai).
Include all 5 sections:

1) ภาพรวมระบบ
2) Actors และ Use Cases
3) Business Rules — label BR-01, BR-02, ... (testable rules พร้อมตัวเลข threshold)
4) Non-functional Requirements
5) Assumptions — flag จุดคลุมเครือทุกจุด พร้อม proposed default + action

ให้แต่ละ Assumption มี 3 ส่วน:
- จุดที่ requirement ไม่ระบุชัด (อ้างต้นทาง)
- Proposed Default พร้อมเหตุผล
- Action ใครต้อง decide เมื่อไหร่

Use Thai for content, English for technical terms (entity, API, BR-XX, FK).
```

**Expected output shape (~200-250 lines):**
- 5 sections ครบ
- ≥7 BR (จำกัดค่าเช่า, เงินประกัน 2x/3x, duration 6 เดือน, duplicate contract, unit_status, grace 15 วัน, notification 60 วัน)
- ≥4 Assumptions (invoice channel, real-time, deposit refund, accounting integration)

**✓ Self-check หลัง A2a (ผู้เรียนเช็คก่อนไป A2b):**
- [ ] เปิดไฟล์ `analysis/system_analysis.md` ใน editor — มีจริง
- [ ] Section 3 (Business Rules) มี **≥5 BR** label เป็น BR-01..BR-XX
- [ ] Section 5 (Assumptions) มี **≥3 ข้อ** แต่ละข้อมี Proposed Default
- [ ] ทุก BR มีตัวเลขหรือ enum ไม่ใช้คำว่า "เร็ว/ดี/มีประสิทธิภาพ"

> **🚨 ก่อนไป A2b — ถ้า self-check ไม่ผ่าน ให้พูด:**
> "เราจะใช้ A2b แก้ตรงนี้พอดี — สังเกต SA ไม่จบที่ draft แต่ต้อง validate"

---

## Step A2b — Validate vs CSV + Sharpen Assumptions (3 นาที)

> **🎓 ก่อนพิมพ์ prompt — เปิด [`AMBIGUITY_MINIDEMO.md`](./AMBIGUITY_MINIDEMO.md) ฉายให้ผู้เรียนเห็น 1-2 ตัวอย่าง bad-vs-good ก่อน (ใช้เวลา ~1 นาที)**

**พิมพ์สด (validation prompt):**
```
ตรวจ analysis/system_analysis.md ที่เพิ่งเขียน 2 อย่าง:

1. Cross-ref กับ rental_contract_MOCK.csv — ทุก field ที่อ้างใน BR ต้องมีจริงใน CSV
   header (เช่น monthly_rent, deposit, status). ถ้า BR อ้างอิง field ที่ไม่มี
   ใน CSV → flag เป็น "ต้องเพิ่ม field" ใน assumption

2. ตรวจ section Assumptions — ทุกข้อต้องมี 3 ส่วน:
   จุดคลุมเครือ + Proposed Default + Action
   ถ้าข้อไหนขาด → แก้ให้ครบ

Edit ไฟล์เดิม แล้วบอกผมว่าแก้อะไรบ้าง 3 บรรทัดสรุป.
```

**Expected behavior:**
- Claude ใช้ Read + Edit (ไม่เขียนใหม่ทั้งไฟล์)
- ตอบสรุป 3 บรรทัด: BR ที่ปรับ, Assumption ที่ขยาย, field ที่ต้องเพิ่ม

**✓ Self-check หลัง A2b:**
- [ ] เปิดไฟล์ → ทุก Assumption มี 3 ส่วนครบ (Default + Action)
- [ ] ทุก BR ที่อ้าง field ตรง CSV header (ไม่ fabricate)
- [ ] Claude สรุป diff 3 บรรทัด — ไม่ rewrite ใหม่หมด

**💡 พูดสด:**
> "ขั้นนี้คือ SA judgment moment — Claude draft ได้เร็ว แต่ validate กับ data จริง
> ต้องมีคน. Pattern: draft → validate → sharpen เป็น loop ที่ SA ทำได้เร็วขึ้น 3-5 เท่า"

---

## Step A3 — ER Diagram in Mermaid (4 นาที)

**พิมพ์สด:**
```
Based on the analysis you just wrote and the CSV schema, generate a Mermaid
ER diagram at analysis/er_diagram.mmd with:

- Entities: Contract, Tenant, Property, Payment, PaymentSchedule
- Thai comments explaining each table
- Cardinality on every relationship (1-to-many, etc.)
- PK/FK marked

After writing the file, print an ASCII summary of the relationships so I can
sanity-check before opening Mermaid preview.
```

**Expected output shape:**
- ไฟล์ `.mmd` ขึ้นต้น `erDiagram`
- 5 entities พร้อม attributes
- Relationships ครบ (Tenant ||--o{ Contract, Property ||--o{ Contract, Contract ||--o{ Payment, ฯลฯ)
- ASCII summary ใน response แบบ text

**✓ Self-check หลัง A3:**
- [ ] เปิด `er_diagram.mmd` ใน VSCode + Mermaid Preview → render ได้ ไม่มี syntax error
- [ ] ทุก entity มี **PK marker** (PK)
- [ ] ทุก relationship มี **cardinality** (ไม่มี `--` เปล่าๆ ต้องเป็น `||--o{` หรือ `}o--o{`)
- [ ] เห็น **ASCII summary ใน chat** สำหรับ sanity-check

**💡 Token Tip ที่พูดสด:**
> "พิมพ์ `/cost` ดูว่าใช้ token ไปเท่าไหร่แล้ว"
> — แล้วพิมพ์ `/cost` ให้ดูจริงๆ

**หลังจาก /cost — verify:**
```bash
# นอก Claude REPL — เปิดใน VSCode + Mermaid Preview ให้ผู้เรียนเห็น
```

---

## Step A4 — Technical Spec for Dev Handoff (4 นาที)

**พิมพ์สด:**
```
Now write analysis/tech_spec.md — this is what Dev will implement from. Include:

1) API endpoints table (method | path | request body | response | auth)
2) Database DDL in PostgreSQL syntax (CREATE TABLE with FK, indexes)
3) Sequence diagram (Mermaid) for the "create rental contract" flow — must show
   validation steps mapped to BR-XX from system_analysis.md
4) Validation rules section — each rule cross-references BR-XX
5) Estimated story points per endpoint (1/2/3/5/8 scale)

Target under 600 lines. If you approach that, use /compact style summarization
by keeping DDL terse.
```

**Expected output shape (~400-600 lines):**
- API table ≥6 rows (POST, GET, PUT, DELETE contracts + list + payments)
- DDL ที่ run ได้บน PostgreSQL — มี CHECK constraints สะท้อน BR (เช่น `CHECK (monthly_rent BETWEEN 5000 AND 500000)`)
- Mermaid sequence diagram สำหรับ create contract
- Validation ที่ link BR-01..BR-07
- Story points per endpoint

**✓ Self-check หลัง A4:**
- [ ] ไฟล์ `tech_spec.md` มี **5 sections**: API table / DDL / sequence / validation rules / story points
- [ ] DDL มี **CHECK constraint** อย่างน้อย 3 ตัว (รัก BR-01, BR-03, BR-08)
- [ ] Validation rules **ทุก row link BR-XX** หรือ NFR-XX (ไม่มี orphan rule)
- [ ] Sequence diagram **แสดง validation step + BR-XX** ใน flow
- [ ] Story points ใช้ Fibonacci (1/2/3/5/8) ไม่ใช่ตัวเลขสุ่ม

**💡 Token Tip ปิด Demo A:**
> "สังเกตว่า context เริ่มหนักแล้ว เพราะเราอ่าน 2 ไฟล์ + สร้าง 3 ไฟล์ +
> Claude generate content ยาวๆ"
> "ถ้าจะใช้ session ต่ออีก 30 นาทีในอีกงานหนึ่ง แนะนำ `/compact` — Claude
> จะสรุป conversation เหลือสั้นๆ แต่ยังจำสำคัญๆ ได้"
> — แล้วพิมพ์ `/compact` ให้ดูจริง

---

## Verification Checklist (ก่อนสิ้นสุด Demo A)

- [ ] `analysis/system_analysis.md` มีหัวข้อครบ 5 section + ≥5 BR + ≥3 Assumptions
- [ ] `analysis/er_diagram.mmd` render ได้ใน Mermaid Preview, cardinality ครบทุก relationship
- [ ] `analysis/tech_spec.md` มี DDL + sequence diagram + validation table (link BR-XX)
- [ ] ผู้เรียนพูดตามได้: "อ่านก่อน → A2a draft → A2b validate → ER → spec → `/cost` → `/compact`"

## Fallback ถ้า live demo พัง

ถ้า Claude สร้าง output ไม่ครบ/พัง — **เปิด `reference-output/`** ฉายแทน:
- `reference-output/system_analysis.md` (250 บรรทัด — model output)
- `reference-output/er_diagram.mmd` (60 บรรทัด — render ใน Mermaid ได้)
- `reference-output/tech_spec.md` (400 บรรทัด — DDL + sequence + validation)

> "นี่คือ output ที่ skill ออกแบบมาให้ออกได้ — ลองดูเป็น target ไว้เทียบ"

## Transition ไป 5.2
> "ตอนนี้เรามี spec พร้อมส่งต่อแล้ว ขั้นต่อไปคือ Designer ส่ง Figma
> มา Dev จะเอา spec + Figma ไปทำ UI — เราจะใช้ Claude ย่อเวลาส่วนนี้
> จากวันครึ่งเหลือ 20 นาที"
