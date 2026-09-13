# PROMPT 02 — คนที่ 2: Database + Catalog + Badge + Score

## อ่านก่อนเริ่ม
CLAUDE.md · ARCHITECTURE.md · **DATABASE.md** · DESIGN.md · IMPLEMENTATION_PLAN.md · TEAM_WORKFLOW.md

## ขอบเขตของคุณ
- Phase 3 — Database Foundation → branch `feature2.1`
- Phase 4 — Catalog + Badge + Sustainable Score → branch `feature2.2`

**คุณเป็นเจ้าของ schema, migration, RLS และ generated types แต่ผู้เดียว**

## ข้อห้ามในเฟสนี้
- ห้ามทำ filter, search, detail, calculator, checkout
- ห้ามปิด RLS เพื่อให้พัฒนาหรือเทสง่ายขึ้น ไม่ว่ากรณีใด
- ห้ามใส่คอลัมน์ `auto_renew`, `next_billing_date` หรือ `expires_at` ใน licenses
- ห้ามเพิ่มคอลัมน์ `sustainable_score` หรือ `badge_one_time` กลับเข้า `apps` (มติ Phase 0 — คำนวณสด)
- ห้ามเพิ่มคอลัมน์ `status` กลับเข้า `orders` (มติ Phase 0 — ทุกแถวคือการซื้อที่สำเร็จ)
- ห้ามแก้ migration ที่ push ขึ้น `dev` แล้ว ให้สร้างไฟล์ใหม่ต่อท้าย

## สิ่งที่ต้องทำ
ทำตาม `DATABASE.md` ให้ตรงทุกตาราง คอลัมน์ constraint และ policy
แล้วทำตาม checklist ใน `IMPLEMENTATION_PLAN.md` Phase 3 และ 4

จุดที่ต้องระวังเป็นพิเศษ
- สูตร Sustainable Score อยู่ใน `ARCHITECTURE.md` ข้อ 6 ต้องตรงเป๊ะ
- `orders.price_paid_thb` ต้องเก็บแยกจาก `apps.price_thb` เพราะราคาอาจเปลี่ยนภายหลัง
- ผู้ใช้ทั่วไปต้อง `INSERT` เข้า **`licenses` และ `orders`** โดยตรงไม่ได้ ทั้งคู่ต้องผ่าน `purchase_app()` เท่านั้น
- `purchase_app()` ต้องตรงตาม signature ใน `DATABASE.md` ข้อ 3.1 เป๊ะ ๆ เพราะคนที่ 4 เรียกใช้ใน Phase 8
  **เขียนเสร็จแล้วแจ้งทีมทันทีว่า RPC พร้อมใช้** อย่ารอถึง Parallel Gate
- Seed ต้องครบเงื่อนไขใน `DATABASE.md` ข้อ 6 ทุกข้อ โดยเฉพาะแอปที่ `price_thb = 1200` และ `competitor_monthly_thb = 350` เพราะคนที่ 4 ต้องใช้ตรวจ Calculator

## ลำดับการทำงาน
1. วาง plan ให้ผู้ใช้อ่านและอนุมัติก่อน
2. เขียน migration และโค้ดหลัก
3. เขียน unit test และ integration test **หลัง** โค้ดหลักเสร็จ
4. ยืนยันว่าเทส RLS แดงจริงถ้าปิด policy ทิ้ง แล้วเปิดกลับ
5. รัน `lint` / `typecheck` / `test`
6. regenerate `database.types.ts` แล้วแจ้งทีมว่า schema ล็อกแล้ว
7. เขียน Phase Completion Report 9 ข้อ
8. **STOP**

## เทสที่ต้องมี
ดู `IMPLEMENTATION_PLAN.md` Phase 3 และ 4 และ RLS test 6 เคสใน `DATABASE.md` ข้อ 7
