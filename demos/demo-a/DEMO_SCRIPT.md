# Demo A — SA Requirement-to-Spec (~17 นาที)

> ใช้ในช่วง **5.1 SA + Claude Code** · instructor เปิด Claude Code ฉายจอให้ผู้เรียนดูสด
>
> **โครงสร้าง — สอน 2 วิธีเปรียบเทียบ:**
> - **A1 + A2a = MANUAL** (พิมพ์ prompt ครบทุก rule) → ผู้เรียนเห็นว่ายาว+ต้องคิดเอง
> - 🎓 **Why Skill?** mini-segment (~1.5 นาที) → เปิด SKILL.md อธิบาย playbook
> - **A2b + A3 + A4 = USING SKILL** (`@skill mango-sa-analyzer`) → prompt สั้นกว่า 5 เท่า, output มี structure อัตโนมัติ
>
> ทุก step จบด้วย ✓ Self-check

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

## Step A1 — Load Context (1 นาที) · **[MANUAL]**

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

## Step A2a — Draft Business Rules + Assumptions (3 นาที) · **[MANUAL]**

> **🎯 จุดประสงค์ของ step นี้:** ให้ผู้เรียนเห็นว่า "ทำได้แบบไม่มี skill" — แต่ต้องพิมพ์ prompt ยาว, ต้องจำ rules เอง, output อาจไม่สม่ำเสมอ ถ้ามีหลายคนทำ

**พิมพ์สด (สังเกต prompt ยาว 15 บรรทัด):**
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

## 🎓 Mini-Segment — Why Skill? (~1.5 นาที)

> **เป้าหมาย:** สอนผู้เรียนเข้าใจว่า skill คืออะไร ก่อนใช้จริงใน A2b เป็นต้นไป

### พูดสด (1 นาที):

> "ที่เราเพิ่งทำใน A2a — เราต้องพิมพ์ rules ครบทุกครั้ง: 5 sections, BR-XX numbering,
> Assumption format 3 ส่วน...
>
> ลองคิดดู — ถ้าทีม Mango มี SA 5 คน ทุกคนใช้ Claude Code แต่พิมพ์ prompt คนละแบบ
> Output จะไม่สม่ำเสมอเลย ตรวจยาก, รวมยาก
>
> **Skill คือ markdown 1 ไฟล์ ที่เก็บ playbook ของทีมไว้** — Claude อ่านอัตโนมัติทุกครั้ง
> ที่ skill match กับงานที่เราทำ ทำให้ทุกคนได้ output ตาม pattern เดียวกัน"

### เปิด SKILL.md ฉายให้ผู้เรียนดู (30 วินาที):

```bash
# เปิดในอีก tab/window
code .claude/skills/mango-sa-analyzer/SKILL.md
```

**ชี้ให้ผู้เรียนเห็น 3 จุด:**
1. **Description** (บรรทัดบนสุด) — Claude ใช้ตรวจว่า skill ตรงกับงานไหม
2. **Output Contract** — 5 sections ของ system_analysis, ER format, tech_spec format
3. **Hard Rules** — ภาษา / CSV cross-ref / BR-XX testable / Assumption flag

> "ทุก rule ที่เราพิมพ์เองใน A2a — อยู่ใน SKILL.md หมดแล้ว
> ตอนนี้เราจะเรียกใช้ดู — สังเกตว่า prompt จะสั้นลงมาก"

---

## Step A2b — Validate vs CSV (USING SKILL) (3 นาที) · **[SKILL]**

> **🎓 ก่อนพิมพ์ prompt — เปิด [`AMBIGUITY_MINIDEMO.md`](./AMBIGUITY_MINIDEMO.md) ฉายให้ผู้เรียนเห็น 1-2 ตัวอย่าง bad-vs-good ก่อน (~1 นาที)**

**พิมพ์สด (สังเกต prompt สั้นลง 5 เท่า):**
```
@skill mango-sa-analyzer

ตรวจ analysis/system_analysis.md กับ mock-data/rental_contract_MOCK.csv —
fix ทุกจุดที่ผิด rule จาก SKILL.md (CSV cross-ref + Assumption 3 ส่วน).

Edit ไฟล์เดิม. สรุปสิ่งที่แก้ 3 บรรทัด.
```

**🎯 Compare กับ A2a:**
- A2a: prompt ~15 บรรทัด ต้องระบุ rules ทุกข้อ
- A2b: prompt 4 บรรทัด — skill เก็บ rules ไว้แล้ว

**Expected behavior:**
- Claude อ่าน SKILL.md → apply rules → ใช้ Read + Edit (ไม่ rewrite ใหม่)
- ตอบสรุป 3 บรรทัด: BR ที่ปรับ, Assumption ที่ขยาย, field ที่ต้องเพิ่ม

**✓ Self-check หลัง A2b:**
- [ ] เปิดไฟล์ → ทุก Assumption มี 3 ส่วนครบ (Default + Action)
- [ ] ทุก BR อ้าง field ตรง CSV header (ไม่ fabricate)
- [ ] Claude สรุป diff 3 บรรทัด — ไม่ rewrite ใหม่หมด

**💡 พูดสด:**
> "เห็นไหม prompt สั้นกว่ามาก แต่ output structure เหมือนเดิม
> นี่คือ value ของ skill — ทีมใช้ pattern เดียวกัน, prompt ไม่ต้องจำเอง,
> ไม่ต้องสอน junior ทุกครั้งว่า BR ต้องเขียนยังไง"

---

## Step A3 — ER Diagram in Mermaid (3 นาที) · **[SKILL]**

**พิมพ์สด (prompt สั้นเพราะ skill enforce format ให้):**
```
@skill mango-sa-analyzer

สร้าง analysis/er_diagram.mmd จาก system_analysis ที่ validated แล้ว
+ schema ของ rental_contract_MOCK.csv.

จบด้วย ASCII summary ใน chat สำหรับ sanity-check.
```

**🎯 Compare manual approach (สมมุติไม่มี skill — ต้องพิมพ์):**
> "Generate Mermaid ER diagram with entities A, B, C, D, E.
> Use Thai comments. Cardinality on every relationship: ||--o{ for 1-to-many,
> }o--o{ for many-to-many. PK and FK markers. Use erDiagram syntax..."
> (ยาว ~10 บรรทัด)

**Expected output shape (skill enforce ให้):**
- ไฟล์ `.mmd` ขึ้นต้น `erDiagram` (skill rule)
- 5+ entities พร้อม attributes (PK/FK markers)
- Cardinality ทุก relationship (`||--o{`, `}o--o{`)
- Thai comments ต่อ entity (skill rule)
- ASCII summary ใน chat (skill rule บรรทัด 49)

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

## Step A4 — Technical Spec for Dev Handoff (4 นาที) · **[SKILL]**

**พิมพ์สด (prompt สั้น — skill enforce structure):**
```
@skill mango-sa-analyzer

สร้าง analysis/tech_spec.md สำหรับ dev handoff
อิง BR-XX จาก system_analysis และ entities จาก er_diagram.

Keep DDL terse. Target under 600 lines.
```

**🎯 Compare manual approach (ปกติต้องพิมพ์ ~12 บรรทัด):**
> "Include API endpoints table with method/path/request/response/auth.
> PostgreSQL DDL with FK and indexes. Sequence diagram in Mermaid for create flow.
> Validation rules cross-reference BR-XX. Story points 1/2/3/5/8..."

**ที่ skill enforce ให้อัตโนมัติ (จาก SKILL.md บรรทัด 51-56):**
- API endpoints table format
- DDL with CHECK constraints linked to BR-XX
- Sequence diagram (Mermaid)
- Validation rules table — ทุก row reference BR-XX
- Story points per endpoint (Fibonacci 1/2/3/5/8)

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

## 📊 Manual vs Skill — สรุปเปรียบเทียบ (1 นาที ก่อนปิด Demo)

| Aspect | A1, A2a (Manual) | A2b, A3, A4 (Skill) |
|---|---|---|
| **Prompt length** | 10-15 บรรทัด ต่อ step | 3-5 บรรทัด ต่อ step |
| **ต้องจำ rules** | ใช่ — SA ต้องรู้ format เอง | ไม่ — skill เก็บไว้ |
| **Output consistency ในทีม** | แต่ละคนได้ต่างกัน | ทุกคนได้ pattern เดียวกัน |
| **เวลา onboard junior SA** | สอน rules ทุกครั้ง | ให้อ่าน SKILL.md ครั้งเดียว |
| **เปลี่ยน rules** | ต้องบอกทุกคน | แก้ SKILL.md ที่เดียว |

### พูดสด (closing):
> "ที่เห็นวันนี้ — manual กับ skill ใช้ทั้งคู่ได้ ให้ output คล้ายกัน
>
> แต่ในทีมจริง ที่มีหลายคนใช้ Claude Code — skill คือ 'team playbook ที่ Claude อ่านได้'
> ลด onboarding, เพิ่ม consistency, แก้ที่เดียวอัปเดตทุกคน
>
> พรุ่งนี้ทำเองได้: fork `mango-sa-analyzer` → เปลี่ยน 2-3 rules → ได้ skill ใหม่
> เช่น `mango-construction-analyzer` (สำหรับ BOQ), `mango-rental-analyzer` (สำหรับสัญญาเช่า)"

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
