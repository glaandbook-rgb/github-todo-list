# PROMPT 04 — คนที่ 4: Calculator + Checkout + License + UI/UX

## อ่านก่อนเริ่ม
CLAUDE.md · ARCHITECTURE.md · DATABASE.md · DESIGN.md · IMPLEMENTATION_PLAN.md · TEAM_WORKFLOW.md

## เงื่อนไขก่อนเริ่ม
เริ่มได้ต่อเมื่อผ่าน **Parallel Gate** แล้วเท่านั้น

## ขอบเขตของคุณ
- Phase 7 — Calculator → branch `feature4.1`
- Phase 8 — Checkout + License + My Library → branch `feature4.2`
- Phase 9 — UI/UX Polish → branch `feature4.3`

## ข้อห้ามในเฟสนี้ — อ่านให้ครบ
- **ห้ามต่อ payment gateway จริงทุกกรณี** ใช้ mock เท่านั้น
- **ห้ามมีช่องกรอกเลขบัตรเครดิต**
- **ห้ามมี checkbox หรือตัวเลือกต่ออายุอัตโนมัติ**
- **ห้ามเก็บข้อมูลบัตรลงฐานข้อมูล**
- หน้า Checkout ต้องมีข้อความชัดเจนว่าเป็นการจำลอง ไม่มีการตัดเงินจริง
- ห้ามแตะ `supabase/migrations/` — ถ้าต้องแก้ schema ให้ **STOP แล้วแจ้งคนที่ 2**
- ห้ามทำ Developer Portal และหน้า Admin (อยู่นอกสโคป)

## สิ่งที่ต้องทำ
ทำตาม checklist ใน `IMPLEMENTATION_PLAN.md` Phase 7, 8 และ 9

จุดที่ต้องระวังเป็นพิเศษ
- สูตร Calculator อยู่ใน `ARCHITECTURE.md` ข้อ 7 ต้องตรงกับตัวเลขในสไลด์: `P=1200, M=350, Y=5` → 21,000 / ประหยัด 19,800 / คุ้มทุนเดือนที่ 4
- `M = 0` ต้องคืน `null` ห้ามหารด้วยศูนย์
- จุดคุ้มทุนต้องปัดขึ้นเสมอ
- ซ่อน Calculator เมื่อ `competitor_monthly_thb` เป็น NULL
- `generateLicenseKey` ต้องรับ random source เข้ามาได้เพื่อให้ unit test กำหนดผลลัพธ์ได้ และห้ามใช้ `Math.random()`
- **ห้าม `insert` เข้า `orders` หรือ `licenses` ตรง ๆ** ทั้งสองตารางไม่มี INSERT policy สำหรับผู้ใช้ทั่วไป
  ต้องเรียก `supabase.rpc("purchase_app", { p_app_id })` ซึ่งสร้างทั้งสองแถวใน transaction เดียว
  (signature อยู่ใน `DATABASE.md` ข้อ 3.1 · เจ้าของคือคนที่ 2)
- `orders` **ไม่มีคอลัมน์ `status`** แล้ว ทุกแถวคือการซื้อที่สำเร็จ
- ชำระเงินล้มเหลวต้อง **ไม่เรียก RPC เลย** ไม่ใช่เรียกแล้วค่อยลบทีหลัง
- จุดคุ้มทุนที่เกินช่วง slider 5 ปี ต้องคืนตัวเลขจริงแต่ UI บอกว่าเกินช่วงที่เลือก
- ผู้ใช้ A ต้องไม่เห็น license ของผู้ใช้ B
- Phase 9 ห้ามเขียนเทสที่ผูกกับ class ของ Tailwind หรือโครงสร้าง markup

## ลำดับการทำงาน
1. วาง plan ให้ผู้ใช้อ่านและอนุมัติก่อน
2. เขียนโค้ดหลัก
3. เขียน unit test **หลัง** โค้ดหลักเสร็จ
4. ลองทำให้โค้ดพังชั่วคราว ยืนยันว่าเทสแดงจริง แล้วแก้กลับ
5. รัน `lint` / `typecheck` / `test`
6. เขียน Phase Completion Report 9 ข้อ
7. **STOP**

## เทสที่ต้องมี
ดู `IMPLEMENTATION_PLAN.md` Phase 7, 8 และ 9 หัวข้อ Tests
