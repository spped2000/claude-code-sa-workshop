# Claude Code — SA Workshop

> Hands-on workshop สำหรับ System Analyst (SA) ใช้ Claude Code วิเคราะห์ requirement → System Analysis + ER Diagram + Tech Spec แบบอัตโนมัติ
>
> **เวลา:** ~45 นาที (Demo 16 นาที + Exercise 21 นาที + opener/wrap 8 นาที)

---

## 🎯 สิ่งที่จะได้เรียนรู้

หลังจบ workshop ผู้เรียนจะ:
- ใช้ Claude Code แปลง raw requirement (Thai narrative) เป็น spec ที่ dev เริ่ม implement ได้
- สร้าง System Analysis document, ER Diagram (Mermaid), Tech Spec ภายใน 20 นาที
- เข้าใจ pattern "draft → validate → sharpen" ที่ SA ใช้ร่วมกับ AI
- จัดการ token (`/cost`, `/compact`) อย่างเป็นนิสัย

---

## ✅ Prerequisites

- **Claude Code CLI** ติดตั้งแล้ว → [docs.claude.com/claude-code](https://docs.claude.com/en/docs/claude-code/overview)
- **VSCode** + extension **"Markdown Preview Mermaid Support"** (สำหรับดู ER diagram render)
- **Terminal** ที่ใช้ Thai Unicode ได้ (Windows: Windows Terminal + `chcp 65001`)
- ความเข้าใจ SA workflow ระดับเบื้องต้น (เคยเห็น requirement → spec มาก่อน)

---

## 📂 โครงสร้างไฟล์

```
.
├── README.md                          ← ไฟล์นี้
├── SA_IMPACT_OPENER.md                ← เปิด session: Before/After Claude
├── .claude/
│   └── skills/
│       └── mango-sa-analyzer/
│           └── SKILL.md               ← skill ที่ใช้ตลอด workshop
├── demos/
│   └── demo-a/
│       ├── DEMO_SCRIPT.md             ← live demo 5 step (16 นาที)
│       ├── AMBIGUITY_MINIDEMO.md      ← bad-vs-good BR/Assumption
│       └── reference-output/          ← ตัวอย่าง output ที่ skill ออกแบบมาให้
│           ├── system_analysis.md
│           ├── er_diagram.mmd
│           └── tech_spec.md
├── exercises/
│   ├── EXERCISE_1.md                  ← hands-on: OT module
│   └── ex1_ot_requirement_TH.txt      ← requirement ลูกค้าสำหรับ exercise
└── mock-data/
    ├── requirement_rental_TH.txt      ← requirement สำหรับ Demo A
    ├── rental_contract_MOCK.csv       ← schema reference (Demo A)
    └── hr_employee_MOCK.csv           ← schema reference (Exercise 1)
```

> **หมายเหตุ:** ทุกไฟล์ที่ลงท้ายด้วย `_MOCK.csv` คือข้อมูลสมมุติ — ไม่ใช่ข้อมูลลูกค้าจริง

---

## 🚀 Quickstart (ทำตามลำดับ)

### 1. Clone repo + เข้า folder

```bash
git clone https://github.com/spped2000/claude-code-sa-workshop.git
cd claude-code-sa-workshop
```

### 2. อ่าน opener + mini-demo (5 นาที)

```bash
code SA_IMPACT_OPENER.md
code demos/demo-a/AMBIGUITY_MINIDEMO.md
```

ดู before/after Claude ในงาน SA + เรียน pattern bad-vs-good

### 3. ทำตาม Demo A (16 นาที)

```bash
cd demos/demo-a
mkdir -p mock-data
cp ../../mock-data/requirement_rental_TH.txt mock-data/
cp ../../mock-data/rental_contract_MOCK.csv mock-data/
claude
```

เปิด `DEMO_SCRIPT.md` ในอีก window — copy prompt ทีละ step ลงใน Claude → ตรวจ ✓ Self-check ทุก step

5 step: **A1** load context → **A2a** draft BRs → **A2b** validate vs CSV → **A3** ER diagram → **A4** tech spec

ทุก step มี checklist ให้ตรวจตัวเอง ถ้าไม่ผ่าน → re-prompt Claude

### 4. ทำ Exercise 1 ด้วยตัวเอง (21 นาที)

```bash
cd ../../        # กลับ root
claude
```

เปิด `exercises/EXERCISE_1.md` อ่าน Tasks 1-4 + Self-check Sidebar + Common Pitfalls

ทำให้ครบ 3 ไฟล์ใน `answers/ex1/`:
- `system_analysis.md`
- `er_diagram.mmd`
- `tech_spec.md`

### 5. เปรียบเทียบกับ reference

หลังทำเสร็จ → เปิด `demos/demo-a/reference-output/` เทียบกับงานตัวเอง
- BR ที่ตัวเองหายไปคืออะไร?
- Assumption ที่ตัวเองไม่ได้ flag คืออะไร?

---

## 💡 Key Concepts

### Pattern ที่ workshop สอน

**Draft → Validate → Sharpen Loop:**
```
1. Claude draft (เร็ว แต่อาจไม่แม่น)
   ↓
2. SA validate กับ CSV/business context
   ↓
3. Re-prompt Claude แก้ → sharpen
```

### Ambiguity Flagging Pattern

ทุก Assumption ต้องมี 3 ส่วน:
```
A-XX: <จุดที่ requirement ไม่ระบุชัด>

      Proposed Default: <ค่าที่แนะนำ + เหตุผล>

      Action: <ใครต้อง decide เมื่อไหร่>
```

### Token Discipline

- `/cost` — ดูว่าใช้ token ไปเท่าไหร่ (พิมพ์ระหว่างทำงาน)
- `/compact` — สรุป context ให้สั้น เก็บประเด็นสำคัญ
- `/model` — เปลี่ยน model ถ้างานเบา (Haiku) หรือหนัก (Opus)

---

## 🆘 Troubleshooting

| อาการ | วิธีแก้ |
|---|---|
| Claude สร้าง field ที่ไม่มีใน CSV | re-prompt: "ตรวจทุก field ใน DDL ต้องมีใน CSV header" |
| BR เขียน vague (e.g. "ระบบต้องเร็ว") | re-prompt: "เขียน BR ใหม่ให้มี numeric threshold หรือ enum specific" |
| ER diagram ไม่มี cardinality | re-prompt: "ระบุ cardinality ทุก relationship: \\|\\|--o{ หรือ }o--o{" |
| Assumption ไม่มี Proposed Default | re-prompt: "ทุก Assumption ต้องมี 3 ส่วน: จุดคลุมเครือ + Default + Action" |
| Token > 40k | พิมพ์ `/compact` ทันที |
| Mermaid ไม่ render | ลองกด `Ctrl+Shift+V` (Markdown Preview) แทน |
| Thai เพี้ยนใน Windows Terminal | พิมพ์ `chcp 65001` ก่อนเปิด `claude` |

---

## 📚 References

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code/overview)
- [Mermaid ER Diagram Syntax](https://mermaid.js.org/syntax/entityRelationshipDiagram.html)
- [PostgreSQL CHECK Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)

---

## 📝 License

Workshop materials สำหรับการเรียนรู้ ใช้ mock data เท่านั้น ไม่มีข้อมูลลูกค้าจริง
