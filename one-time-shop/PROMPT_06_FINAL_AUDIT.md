# PROMPT 06 — Final Audit (ทุกคน)

## อ่านก่อนเริ่ม
CLAUDE.md · IMPLEMENTATION_PLAN.md Phase 11 · TEAM_WORKFLOW.md ข้อ 14

## ขอบเขต
**ห้ามเพิ่มฟีเจอร์ใหม่** เฟสนี้ตรวจสอบอย่างเดียว

## ตรวจฟีเจอร์
- [ ] Auth: สมัคร / เข้า / ออก / protected route ครบ
- [ ] คลังแอป + ป้ายการันตี 4 ชนิด + Sustainable Score
- [ ] 5 หมวดหมู่ + ตัวกรอง 4 มิติ + live search
- [ ] หน้ารายละเอียด + checklist ไม่มีค่าแฝง
- [ ] Calculator + จุดคุ้มทุน
- [ ] Mock Checkout + License Key + My Library

## ตรวจสิ่งที่ต้องไม่มี
- [ ] ไม่มีการต่อ payment gateway จริง
- [ ] ไม่มีช่องกรอกเลขบัตร
- [ ] ไม่มีตัวเลือกต่ออายุอัตโนมัติ
- [ ] ไม่มี Developer Portal หรือหน้า Admin หลุดเข้ามา
- [ ] ไม่มีฟีเจอร์นอกสโคปอื่นใน `CLAUDE.md` ข้อ 4

## ตรวจคุณภาพ
- [ ] `npm run lint` ผ่าน
- [ ] `npm run typecheck` ผ่าน
- [ ] `npm run build` ผ่าน
- [ ] `npm run test` ผ่าน
- [ ] `npm run test:e2e` ผ่าน
- [ ] ไม่มี `console.log` ที่ใช้ debug หลงเหลือ
- [ ] ไม่มีเทสที่ถูก `skip` หรือถูกลดความเข้มเพื่อให้ผ่าน

## ตรวจความปลอดภัย
- [ ] RLS เปิดครบทุกตาราง ไม่มีใครปิดเพื่อให้เทสผ่าน
- [ ] ไม่มี secret ใน repo — `git log -p | grep -i "key\|secret\|password"`
- [ ] `.env` ไม่ถูก commit
- [ ] ผู้ใช้ A เข้าถึงข้อมูลของผู้ใช้ B ไม่ได้

## ตรวจ Git และการส่งงาน
- [ ] branch ครบใน GitHub: `main`, `dev`, `feature1.1`, `feature1.2`, `feature2.1`, `feature2.2`, `feature3.1`, `feature3.2`, `feature4.1`, `feature4.2`, `feature4.3`
- [ ] **ไม่มี branch ไหนถูกลบหลัง merge**
- [ ] ทุกคนมี commit ปรากฏใน history
- [ ] commit message ทุกอันใช้รูปแบบ `type: คำอธิบาย`
- [ ] ทุก PR มีเนื้อหาครบ 9 หัวข้อ
- [ ] `dev` merge เข้า `main` แล้ว
- [ ] `README.md` มีวิธีรันและตารางแบ่งงาน
- [ ] repo เป็น Public หรือเชิญอาจารย์แล้ว

## รายงานปิดงาน
1. สรุปสิ่งที่ทำได้ครบตามสโคป
2. สิ่งที่ตั้งใจไม่ทำ พร้อมเหตุผล
3. จำนวนเทสแยกตาม unit / integration / e2e
4. ผลการรันทั้งหมด
5. ปัญหาที่ยังค้างอยู่
6. สิ่งที่จะต่อยอดในอนาคต (Developer Portal, Admin, ชำระเงินจริง, Alternative Ecosystem)

**STOP.**
