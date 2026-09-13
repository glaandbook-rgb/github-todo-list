# PROMPT 01 — คนที่ 1: Foundation + Authentication

## อ่านก่อนเริ่ม
CLAUDE.md · ARCHITECTURE.md · DESIGN.md · IMPLEMENTATION_PLAN.md · TEAM_WORKFLOW.md

## ขอบเขตของคุณ
- Phase 1 — Foundation → branch `feature1.1`
- Phase 2 — Authentication → branch `feature1.2`

**ทำทีละเฟส เฟสละ branch แยก PR ไม่รวมกัน**

## ข้อห้ามในเฟสนี้
- ห้ามทำ catalog, filter, calculator, checkout, library
- ห้ามสร้าง migration หรือแตะ `supabase/migrations/` (เป็นของคนที่ 2)
- ห้ามเขียนฟีเจอร์ธุรกิจใน Phase 1 — เฟสนั้นมีแค่โครงและ tooling

## สิ่งที่ต้องทำ
ทำตาม checklist ใน `IMPLEMENTATION_PLAN.md` Phase 1 และ Phase 2 ให้ครบทุกข้อ

จุดที่ต้องระวังเป็นพิเศษ
- แปลง token จาก `DESIGN.md` ให้ครบ ทั้ง 3 ฟอนต์ (Manrope / Inter / JetBrains Mono), palette, radius และ spacing 8px scale
- `.gitignore` ต้องบล็อก `.env*` แต่ยกเว้น `.env.example`
- `.env.example` ต้องมีแต่ชื่อตัวแปร ห้ามมีค่าจริง
- การป้องกัน route ต้องทำสองชั้นตาม `ARCHITECTURE.md` ข้อ 4 ห้ามพึ่ง client อย่างเดียว

## ลำดับการทำงาน
1. วาง plan ให้ผู้ใช้อ่านและอนุมัติก่อน
2. เขียนโค้ดหลัก
3. เขียน unit test **หลัง** โค้ดหลักเสร็จ
4. ลองทำให้โค้ดพังชั่วคราว ยืนยันว่าเทสแดงจริง แล้วแก้กลับ
5. รัน `lint` / `typecheck` / `test` / `build`
6. เขียน Phase Completion Report 9 ข้อ
7. **STOP** ให้ผู้ใช้ Manual Test เอง

## เทสที่ต้องมี
ดู `IMPLEMENTATION_PLAN.md` Phase 1 และ 2 หัวข้อ Tests

ถ้าพบว่าต้องแก้ schema → **STOP แล้วแจ้งคนที่ 2** ห้ามแก้เอง
