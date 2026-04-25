# Ambiguity Mini-Demo — สอน SA จับจุดคลุมเครือ (5 นาที)

> **เมื่อไหร่ใช้:** ก่อน Demo A step A2b · instructor อ่านสด ฉายจอให้ผู้เรียนดู
>
> **ทำไม:** "Flagging ambiguity" คือทักษะ #1 ของ SA — Claude จะช่วยทำได้ดีก็ต่อเมื่อ SA สั่งให้ flag เป็น และตรวจ output ได้

---

## 🎯 Key Message (พูดเปิดประโยคเดียว)

> "ของจริง — requirement ไม่เคยสมบูรณ์ครบ ลูกค้าเขียนสั้น+คิดในใจ ไม่ได้บอกหมด
> งาน SA คือ **จับจุดที่ไม่ได้บอก** แล้ว propose default + ขอ confirm
> Claude ช่วยจับได้เร็วขึ้น — แต่ SA ต้องรู้ว่าอะไรเรียกว่า 'ดี' กับ 'ไม่ดี'"

---

## 📋 Side-by-side: Bad vs Good

ใช้ตัวอย่างจาก `requirement_rental_TH.txt` (Demo A) — 4 จุดคลุมเครือที่จงใจวางไว้

### ตัวอย่าง 1 — Business Rule

**❌ Bad BR (Claude จะออกแบบนี้ถ้า SA ไม่ specify):**
```
BR-01: ระบบต้องจัดการสัญญาเช่าได้
```
> ปัญหา: ไม่ testable, ไม่มี threshold, dev เขียน test ไม่ได้

**✅ Good BR (Claude ออกแบบนี้เมื่อ SA สั่งให้ลิงก์ requirement):**
```
BR-01: monthly_rent ต้องอยู่ระหว่าง 5,000 ≤ x ≤ 500,000 บาท/เดือน
       (ที่มา: requirement บรรทัด 25; testable ด้วย CHECK constraint
       + unit test edge case)
```
> ทำไมดีกว่า: มีตัวเลข, อ้างอิงต้นทาง, dev เขียน test และ DB constraint ได้ทันที

---

### ตัวอย่าง 2 — Vague Assumption

**❌ Bad assumption (พบบ่อย — SA junior หรือ Claude default):**
```
A-01: อาจจะมีรายละเอียดเพิ่มเติมในอนาคต
```
> ปัญหา: ไม่ actionable, dev ไม่รู้ทำอะไรต่อ, ไม่บอก stakeholder ต้อง decide อะไร

**✅ Good assumption (flag จุดคลุมเครือ + propose default + action):**
```
A-01: Requirement ไม่ระบุช่องทางส่งใบแจ้งหนี้
      (email? SMS? LINE?)

      Proposed Default: ใช้ email — เพราะมี field email ใน schema ผู้เช่าอยู่แล้ว
                        (ดู rental_contract requirement บรรทัด 32)

      Action: ขอ stakeholder ยืนยันใน meeting kickoff
              (ถ้า LINE → ต้องทำ LINE Notify integration เพิ่ม +5 SP)
```
> ทำไมดีกว่า: ระบุชัดว่า "ไม่รู้อะไร", propose default มีเหตุผล, action ระบุใครต้อง decide เมื่อไหร่ + impact ถ้าเปลี่ยน

---

### ตัวอย่าง 3 — "Real-time" trap

ลูกค้าเขียน: *"ผู้บริหารต้องการดูรายได้แบบ real-time"*

**❌ Bad SA reaction:** assume real-time = ทันที (sub-second) → architect WebSocket + cache invalidation ซับซ้อน → over-engineering

**✅ Good SA reaction (flag เป็น A-XX):**
```
A-02: "real-time" ไม่ระบุ granularity (วินาที / นาที / ชั่วโมง)

      Proposed Default: refresh ทุก 5 นาที (cache + polling)
                        — เพียงพอสำหรับ KPI dashboard executive
                        — ลด DB load 100x เทียบกับ true real-time

      Action: นำ mockup dashboard ที่ปัจจุบันลูกค้าใช้มาดู
              ถ้าเขาต้องการเร็วกว่า 5 นาที → quote เพิ่ม +8 SP
```

---

## 🛠️ Pattern ที่สอนผู้เรียน

ทุก ambiguity ที่ flag ต้องมี 3 ส่วน:

```
A-XX: <จุดที่ requirement ไม่ระบุ>

      Proposed Default: <ค่าที่ SA แนะนำ + เหตุผลสั้น>

      Action: <ใครต้อง decide + เมื่อไหร่ + impact ถ้าเปลี่ยน>
```

**Prompt ที่จะใช้ตอน Demo A2b:**
```
ตรวจ analysis/system_analysis.md ที่เพิ่งเขียน — ทุก Assumption ต้องมี 3 ส่วน:
1. จุดที่ requirement ไม่ระบุ (อ้างบรรทัดต้นทาง)
2. Proposed Default + เหตุผล
3. Action ใครต้อง decide

ถ้าข้อไหนไม่ครบ → แก้ใหม่
```

---

## ⏱️ ลำดับการสอนสด (5 นาที)

| เวลา | กิจกรรม |
|---|---|
| 0:00–0:30 | ฉายหน้านี้ พูด Key Message |
| 0:30–2:00 | อ่านตัวอย่าง 1 (BR) — bad → good เปรียบเทียบ |
| 2:00–3:30 | อ่านตัวอย่าง 2 (Assumption) — bad → good |
| 3:30–4:30 | ตัวอย่าง 3 (real-time trap) — explain ว่า over-engineering ถ้า assume ผิด |
| 4:30–5:00 | สรุป pattern 3 ส่วน + transit ไป Demo A2b |

---

## 💡 Closing line

> "Claude ทำหน้าที่ 'first pass' ได้ดี แต่ SA คือคนตัดสิน
> ว่า assumption ไหน reasonable + propose default ที่ฝั่ง business จะ accept ได้
> 5 นาทีที่ใช้ปรับ Claude output = ลด rework 2-3 รอบของ dev"
