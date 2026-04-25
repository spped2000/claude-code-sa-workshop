# SA Impact Opener — เปิด session 5.1 (3 นาที)

> **เมื่อไหร่ใช้:** Instructor อ่านสด ก่อน Demo A · ฉายหน้านี้บน projector
>
> **เป้าหมาย:** ให้ผู้เรียนเห็นว่า Claude Code ลด pain point จริงในงาน SA วันจันทร์–ศุกร์ ไม่ใช่ feature เท่ๆ ที่ใช้ทำ side project

---

## 🎯 Hook (เปิดประโยคแรก)

> "ขอถามก่อน — ใครในห้องนี้เป็น SA หรือเคยทำงาน SA?
> ยกมือหน่อย... โอเค
>
> ถามต่อ — ครั้งสุดท้ายที่คุณรับ requirement จากลูกค้า แล้วพบว่า
> **dev ถามกลับมา 3 รอบ** เพราะ spec ไม่ครบ — กี่วันที่แล้วครับ?"

(หยุดให้ตอบ — ปกติ "เมื่อวาน" หรือ "อาทิตย์ที่แล้ว")

> "วันนี้เราจะมาดูว่า Claude Code ตัดวงจรนั้นออกได้ยังไง"

---

## 📊 SA Tuesday — Before vs After Claude Code

### ❌ Before (ที่เราคุ้นเคย)

| งาน | เวลา | ปัญหา |
|---|---|---|
| อ่าน requirement (LINE/email/doc) + ถามกลับ | 1-2 ชม. | ลูกค้าเขียนสั้น+คิดในใจ ต้องตามถาม |
| Draft system analysis (Word/Confluence) | **2 วัน** | กลัวพลาดบาง section, จัดเลขใหม่หลายรอบ |
| วาด ER diagram (draw.io หรือ Visio) | 4-6 ชม. | แก้ relationship แล้วต้อง redraw |
| เขียน tech spec + DDL + API table | 1 วัน | copy-paste จาก spec เก่าๆ ปรับ field |
| ส่ง dev → dev ถามกลับ → กลับไปแก้ | **3-4 รอบ × 30 นาที** | spec ขาดบาง field/rule ที่ดูเล็กแต่กระทบ |

**Total:** ประมาณ **5 วัน** จาก requirement → spec พร้อมส่ง dev

---

### ✅ After (Demo วันนี้จะแสดง)

| งาน | เวลา | ตัวช่วย |
|---|---|---|
| อ่าน + flag ambiguity จาก requirement | 5-10 นาที | Claude อ่านไฟล์ + flag จุดคลุมเครือ + propose default |
| Draft system analysis + BR + Assumption | **20 นาที** | Skill `mango-sa-analyzer` enforce structure + BR-XX numbering |
| ER diagram (Mermaid) | 5 นาที | Claude generate + sanity ASCII summary ใน chat |
| Tech spec + DDL + sequence + validation | 10 นาที | Cross-ref BR-XX → CHECK constraint อัตโนมัติ |
| Review + sharpen (SA judgment) | 10-15 นาที | คนตรวจ — แก้คำ/ปรับ default ที่ business จะ accept |

**Total:** ประมาณ **1 ชั่วโมง** สำหรับ first-pass spec ที่ dev เริ่มทำได้

---

## 💥 Impact ที่คุยกับ stakeholder ได้

| Metric | Before | After | ลดลง |
|---|---|---|---|
| Spec drafting time | 2 วัน | 20 นาที | **~95%** |
| Dev clarification rounds | 3-4 รอบ | 1 รอบ | **~70%** |
| ER diagram iteration | 3 รอบ | 1 รอบ | **~67%** |
| Time to first dev commit | 5-7 วัน | 1-2 วัน | **~70%** |

> **Caveat (พูดให้ผู้เรียนรู้):**
> "ตัวเลขนี้เป็น typical case — บางงานยากกว่า บางงานง่ายกว่า
> และ Claude **ไม่ได้แทน SA** — แค่ทำส่วน drafting/structuring
> ส่วน 'judgment' (assumption ไหน reasonable, business จะ accept ไหม)
> ยังเป็นงาน SA 100%"

---

## 🚫 ที่ Claude **ไม่ได้** ทำให้

> "ก่อนเริ่ม demo — กันความเข้าใจผิด:"

- ❌ Claude **ไม่ได้** คุยกับลูกค้าแทนเรา
- ❌ Claude **ไม่ได้** รู้ context ของ Mango ที่ไม่ได้อยู่ในไฟล์
- ❌ Claude **ไม่ได้** decide เรื่อง business policy
- ❌ Claude **อาจ fabricate** field ที่ไม่มีจริง — SA ต้อง validate

---

## ✨ Closing line — transit ไป Demo A

> "เอาล่ะ พอเข้าใจ value แล้ว — ไปดูจริงดีกว่า
> สมมุติเราเป็น SA ที่เพิ่งรับ requirement สัญญาเช่ามาทาง LINE
> ผมจะแสดงให้ดูว่า 16 นาทีจาก raw text → spec ที่ dev ทำงานต่อได้
>
> เริ่ม Demo A กันครับ"

---

## ⏱️ ลำดับสด (3 นาที)

| เวลา | กิจกรรม |
|---|---|
| 0:00–0:30 | Hook — ถามคำถาม รอตอบ |
| 0:30–1:30 | ฉาย Before table อ่าน 3 บรรทัด |
| 1:30–2:30 | ฉาย After table + Impact metrics |
| 2:30–3:00 | Caveat + Closing → transit Demo A |
