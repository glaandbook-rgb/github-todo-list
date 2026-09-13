# PROMPT 03 — คนที่ 3: Filter + Search + หน้ารายละเอียด

## อ่านก่อนเริ่ม
CLAUDE.md · ARCHITECTURE.md · DATABASE.md · DESIGN.md · IMPLEMENTATION_PLAN.md · TEAM_WORKFLOW.md

## เงื่อนไขก่อนเริ่ม
เริ่มได้ต่อเมื่อผ่าน **Parallel Gate** แล้วเท่านั้น — Phase 1–4 ต้อง merge เข้า `dev` และ `database.types.ts` ต้องนิ่งแล้ว

## ขอบเขตของคุณ
- Phase 5 — Filter + Search → branch `feature3.1`
- Phase 6 — หน้ารายละเอียดแอป → branch `feature3.2`

## ข้อห้ามในเฟสนี้
- ห้ามทำ calculator, checkout, license, library
- ห้ามแตะ `supabase/migrations/` — ถ้าต้องแก้ schema ให้ **STOP แล้วแจ้งคนที่ 2**
- ห้ามแก้ `AppCard`, `BadgeChip`, `ScoreMeter` ของคนที่ 2 โดยไม่แจ้ง

## สิ่งที่ต้องทำ
ทำตาม checklist ใน `IMPLEMENTATION_PLAN.md` Phase 5 และ 6

จุดที่ต้องระวังเป็นพิเศษ
- `applyFilters` และ `matchesSearch` ต้องเป็น **ฟังก์ชันบริสุทธิ์** ห้ามแตะ DOM ห้ามเรียก network เพราะต้อง unit test ตรง ๆ
- ตัวกรอง 4 มิติต้องทำงานพร้อมกันแบบ **AND ไม่ใช่ OR**
- ช่วงราคาต้องไม่ทับกัน: `0–500`, `501–1,500`, `1,501+` — ทั้ง 500/501 และ 1,500/1,501 ต้องตกคนละช่วง
  (เดิมเขียน `1,500+` ทำให้ 1,500 ตกสองช่วง แก้แล้วในมติ Phase 0 นิยามเต็มอยู่ใน `ARCHITECTURE.md` ข้อ 3)
- debounce ≤ 250ms และผลลัพธ์ต้องแสดงภายใน 300ms ตาม NFR ในสไลด์
- Empty state ต้องมีปุ่มล้างตัวกรองทั้งหมดที่ใช้ได้จริง
- แอปที่ `is_published = false` ต้องเปิดหน้ารายละเอียดไม่ได้

## ลำดับการทำงาน
1. วาง plan ให้ผู้ใช้อ่านและอนุมัติก่อน
2. เขียนโค้ดหลัก
3. เขียน unit test **หลัง** โค้ดหลักเสร็จ
4. ลองทำให้โค้ดพังชั่วคราว ยืนยันว่าเทสแดงจริง แล้วแก้กลับ
5. รัน `lint` / `typecheck` / `test`
6. เขียน Phase Completion Report 9 ข้อ
7. **STOP**

## เทสที่ต้องมี
ดู `IMPLEMENTATION_PLAN.md` Phase 5 และ 6 หัวข้อ Tests — มี 9 เคสใน Phase 5 และ 3 เคสใน Phase 6

## เริ่มก่อนได้โดยไม่ต้องรอ Parallel Gate
`applyFilters` และ `matchesSearch` เป็นฟังก์ชันบริสุทธิ์ที่ห้ามแตะ Supabase อยู่แล้ว
เขียนพร้อม unit test ได้ตั้งแต่วันแรกด้วย fixture ที่เขียนเอง แล้วค่อยต่อกับข้อมูลจริงหลังผ่าน Gate
