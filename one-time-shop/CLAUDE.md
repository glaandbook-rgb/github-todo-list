# CLAUDE.md

เอกสารนี้คือ **กติกาหลัก** ที่ AI และสมาชิกทุกคนต้องทำตามตลอดโปรเจกต์

---

## 1. Project Identity

**The One-Time Shop** — Marketplace รวมซอฟต์แวร์แบบซื้อขาด (One-Time Purchase) ที่เน้นความโปร่งใสด้านราคา และสร้างช่องทางรายได้ที่เป็นธรรมให้นักพัฒนา Indie และ Open Source

- ทีม: 4 คน (นักศึกษา)
- เป้าหมาย: ระบบที่ **ทำงานได้จริง ทดสอบได้ อธิบายได้** ไม่ใช่ระบบที่มีฟีเจอร์เยอะที่สุด
- ที่มาของสโคป: เอกสาร Design Thinking (EMPATHIZE → DEFINE → IDEATE → PROTOTYPE) และสไลด์นำเสนอ The One-Time Shop

---

## 2. Stack (ล็อกแล้ว ห้ามเปลี่ยนโดยพลการ)

| ส่วน | เทคโนโลยี |
|---|---|
| Framework | Next.js (App Router) + TypeScript |
| Styling | Tailwind CSS v4 |
| Database + Auth | Supabase (Postgres + RLS) |
| Validation | Zod |
| Unit / Integration Test | Vitest |
| E2E Test | Playwright |
| Lint / Typecheck | ESLint + `tsc --noEmit` |

ถ้าจำเป็นต้องเพิ่ม dependency ให้ระบุเหตุผลใน Phase Completion Report และแจ้งทีมก่อน

---

## 3. Required Features (สโคปที่ตกลงแล้ว)

### Authentication — คนที่ 1
- Sign Up / Sign In / Sign Out
- Session handling
- Protected routes: `/library` และ `/checkout/[slug]` ต้อง login (ตาราง route เต็มอยู่ใน `ARCHITECTURE.md` ข้อ 4)

### เสาหลักที่ 1 — คลังแอปคัดสรร + ป้ายการันตี — คนที่ 2
- แสดงรายการแอปแบบการ์ด
- ป้ายการันตี 4 ชนิด: `Verified One-Time` / `Lifetime License` / `Offline-Friendly` / `Privacy Verified`
  (`Verified One-Time` ตัดสินจาก `pricing_model` อีก 3 ป้ายเป็นคอลัมน์ boolean)
- Tooltip อธิบายความหมายของแต่ละป้าย
- Sustainable Score 0–100 พร้อมแบนด์ Neutral (เทา) / Sustainable (เขียว) / Exemplary (ทอง)
  — คำนวณสด ไม่เก็บในฐานข้อมูล (สูตรอยู่ใน `ARCHITECTURE.md` ข้อ 6)

### เสาหลักที่ 2 — คัดกรองและค้นหา — คนที่ 3
- 5 หมวดหมู่: Productivity & Office, Developer & DevOps, Graphic & Design, System & Security, Audio & Video
- ตัวกรอง 4 มิติพร้อมกันแบบ AND: แพลตฟอร์ม / โมเดลราคา / ช่วงราคา / ป้ายการันตี
- Live search จากชื่อแอป สโลแกน แท็ก ชื่อผู้พัฒนา
- Empty state พร้อมปุ่มล้างตัวกรองทั้งหมด

### เสาหลักที่ 3 — รายละเอียด + เครื่องคำนวณ — คนที่ 3 (Detail) / คนที่ 4 (Calculator)
- หน้ารายละเอียดแอป: ราคาสุทธิ, จำนวนเครื่องที่ใช้ได้, นโยบายอัปเดต, ระยะซัพพอร์ต
- Checklist "ไม่มีค่าธรรมเนียมแอบแฝง"
- Subscription Cost Calculator: slider 1–5 ปี, ต้นทุนสะสมของ Subscription, ส่วนต่างที่ประหยัด, จุดคุ้มทุน (เดือนที่เท่าไหร่)

### เสาหลักที่ 4 — ชำระเงิน + License — คนที่ 4
- Checkout แสดงยอดสุทธิเท่าที่แสดง ไม่มีรายการพ่วงติ๊กไว้
- **Mock Payment เท่านั้น** (ดูข้อ 4)
- ออก Perpetual License Key รูปแบบ `OTS-XXXX-XXXX`
- My Library: ดูและคัดลอกคีย์ได้ตลอด ไม่มีวันหมดอายุ

### UI/UX — คนที่ 4
- Responsive, loading state, error state, empty state, form feedback
- ยึด Design System ใน `DESIGN.md`

---

## 4. Explicit Non-Goals

ห้ามทำสิ่งเหล่านี้ เว้นแต่ผู้ใช้สั่งเปลี่ยนสเปกอย่างชัดเจน

**ตัดออกเพราะเกินกำลัง / ทำจริงไม่ได้ในงานเรียน**
- การชำระเงินจริงทุกรูปแบบ (PromptPay QR จริง, บัตรเครดิต/เดบิตจริง, payment gateway จริง)
- API ตรวจสอบ License Key สำหรับนักพัฒนาภายนอก
- ระบบคืนเงิน (refund flow)
- Developer Portal (สมัคร Developer, ลงทะเบียนแอป, ดูยอดขาย)
- หน้าจอ Admin / Curator และระบบตรวจสอบคุณภาพแอป

**ตัดออกเพราะเป็น Non-Goal ทั่วไป**
- Email verification, Forgot password, Social login, 2FA
- Chat, Comment, Review ที่ผู้ใช้เขียนเอง, Notification, Activity log
- Real-time collaboration, AI features, Automation
- Wishlist, Cart หลายรายการ (ซื้อได้ทีละ 1 แอป)
- Payment/subscription ของตัว Marketplace เอง

> **Developer Portal มี mockup ใน Stitch แล้ว แต่ไม่อยู่ในสโคปรอบนี้** ให้นำเสนอเป็น "อนาคตและการต่อยอด" ในสไลด์แทน ห้ามเขียนโค้ด

**เรื่อง Mock Payment — สำคัญ**
Checkout ต้องทำงานครบทุกขั้นตอนตั้งแต่กดซื้อ → สร้าง order → ออก License Key → เข้า My Library แต่ขั้นตอนตัดเงินให้ใช้ mock provider ที่คืนผลสำเร็จ/ล้มเหลวได้ ห้ามต่อ gateway จริง ห้ามรับเลขบัตรจริง และต้องมีข้อความบนหน้าจอว่าเป็นการจำลอง

---

## 5. Source of Truth

อ่านไฟล์เหล่านี้ก่อนลงมือ

1. `CLAUDE.md`
2. `ARCHITECTURE.md`
3. `DATABASE.md`
4. `DESIGN.md`
5. `IMPLEMENTATION_PLAN.md`
6. `TEAM_WORKFLOW.md`

ลำดับความสำคัญเมื่อขัดแย้งกัน

```
คำสั่งล่าสุดของผู้ใช้ > CLAUDE.md > ARCHITECTURE.md > DATABASE.md
> DESIGN.md > IMPLEMENTATION_PLAN.md > TEAM_WORKFLOW.md
```

---

## 6. Strict Scope Control

ทำเฉพาะ phase/task ที่ถูกสั่งเท่านั้น

ห้าม
- refactor โมดูลที่ไม่เกี่ยวข้อง
- ทำฟีเจอร์ล่วงหน้าที่ยังไม่ถึงเฟส
- แก้ database schema โดยไม่ได้รับอนุญาตจากคนที่ 2
- เปลี่ยน stack
- เพิ่ม dependency ที่ไม่จำเป็น
- สร้าง abstraction เผื่ออนาคต

ถ้าความต้องการขัดกับเอกสารเหล่านี้ ให้ **หยุดและรายงานความขัดแย้ง** ห้ามเดาแล้วทำต่อเงียบ ๆ

---

## 7. Testing Is Part of the Definition of Done

ฟีเจอร์จะยังไม่ถือว่าเสร็จจนกว่าจะมีเทสของตัวเอง **คนที่เขียนฟีเจอร์เป็นเจ้าของเทสของฟีเจอร์นั้น**

### Unit Test
ใช้กับ logic ที่ผลลัพธ์คาดเดาได้แน่นอน เช่น
- validation (Zod schema)
- การคำนวณ Sustainable Score
- การคำนวณต้นทุนสะสมและจุดคุ้มทุนของ Calculator
- logic การกรอง (filter predicate)
- การสร้างและตรวจรูปแบบ License Key
- pure utility functions

**ห้ามเสียเวลาเทส markup ธรรมดาหรือ internal ของ framework**

โค้ดธุรกิจใหม่ต้องคลุมอย่างน้อย
- เคสปกติ (success)
- เคสข้อมูลผิด (invalid)
- เคสขอบ (boundary) เมื่อมี

### Integration Test
ใช้ตรวจรอยต่อที่มีความหมายจริง เช่น
- application logic + database
- session + protected route
- RLS + การเข้าถึงข้อมูล (ผู้ใช้ A ต้องไม่เห็น license ของผู้ใช้ B)
- การซื้อ + การบันทึก order/license

ห้ามใช้ database จริงของ production และห้าม mock ทุกอย่างจนไม่เหลืออะไรให้ทดสอบ

### E2E Test
Journey ขั้นต่ำที่ต้องผ่าน
1. สมัครสมาชิก / เข้าสู่ระบบ
2. เข้าหน้า `/library` โดยไม่ login แล้วต้องถูกเด้งออก
3. ค้นหาและกรองแอปจากหน้า Home
4. เปิดหน้ารายละเอียดแอป
5. ใช้ Calculator แล้วตัวเลขเปลี่ยนตาม slider
6. ซื้อแอป (mock) แล้วได้ License Key
7. License Key ปรากฏใน My Library
8. ผู้ใช้คนอื่นต้องไม่เห็น License ของเรา

---

## 8. Test Quality Rules

> **เทสต้องล้มเหลวด้วยเหตุผลที่ถูกต้อง**

ห้าม
- เขียนเทสที่ assert แค่ว่า mock ถูกเรียก ทั้งที่ทดสอบพฤติกรรมจริงได้
- ก๊อปเทสที่เกือบเหมือนกันเป็นสิบอัน
- เทส implementation detail โดยไม่จำเป็น
- ลดความเข้มของ assertion เพื่อให้เทสผ่าน
- `skip` เทสเพราะทำ implementation ไม่ทัน
- mock ทุกอย่างใน integration test
- ใช้ credential จริงหรือ commit `.env`
- ลบหรือทำให้เทสของคนอื่นอ่อนลงเพื่อให้ CI เขียว

**วิธีพิสูจน์ว่าเทสใช้ได้จริง:** แก้โค้ดให้พังชั่วคราว แล้วดูว่าเทสแดงจริงไหม ถ้าไม่แดง แปลว่าเทสนั้นใช้ไม่ได้

ตั้งชื่อเทสให้อ่านรู้เรื่อง

```ts
describe("calculateBreakEven", () => {
  it("คืนค่าเดือนที่คุ้มทุนเมื่อราคาซื้อขาดถูกกว่า", ...)
  it("คืน null เมื่อ subscription ถูกกว่าตลอดช่วงที่คำนวณ", ...)
  it("ปฏิเสธราคาที่ติดลบ", ...)
})
```

**ห้ามคิดชื่อ npm script ขึ้นมาเอง ให้เปิด `package.json` ดูก่อนทุกครั้ง**

---

## 9. Test Commands

ใช้เฉพาะ script ที่มีอยู่จริง

```bash
npm run lint
npm run typecheck
npm run test
npm run test:unit
npm run test:integration
npm run test:e2e
npm run build
```

ถ้าคำสั่งไหนยังไม่มี
- **ห้ามแกล้งทำเป็นว่ารันแล้ว**
- ให้ใช้คำสั่งที่มีจริง หรือรายงานว่ายังไม่ได้ตั้งค่า tooling ส่วนนั้น

---

## 10. Phase Gate

เมื่อจบแต่ละ phase

1. รันเทสที่เกี่ยวข้อง
2. รัน lint / typecheck / build
3. อ่าน `git diff` ทั้งหมด
4. เขียน Phase Completion Report (ดูข้อ 11)
5. **STOP**

**ห้ามเดินหน้าเข้า phase ถัดไปเองโดยอัตโนมัติ** จุดนี้คือช่วงที่คนต้องเข้ามา Manual Test

---

## 11. Phase Completion Report

ทุกเฟสต้องรายงาน 9 ข้อนี้

1. ทำอะไรไปบ้าง
2. ไฟล์ที่เปลี่ยน
3. เทสที่เพิ่ม
4. เทสที่รัน
5. ผลการรันเทส
6. การเปลี่ยนแปลงฐานข้อมูล
7. ปัญหาที่ยังค้างอยู่
8. สิ่งที่ตั้งใจไม่ทำเพราะอยู่นอกสโคป
9. เฟสถัดไปขึ้นกับอะไร

เอกสารนี้คือหลักฐานสำหรับโจทย์ข้อ 2 (plan), ข้อ 3 (unit test) และข้อ 4 (manual test) ให้แปะลงใน PR description

---

## 12. Team Ownership

| คน | ขอบเขต | Branch |
|---|---|---|
| คนที่ 1 | Foundation + Design System + Authentication | `feature1.1`, `feature1.2` |
| คนที่ 2 | Database + RLS + Catalog + Badge + Sustainable Score | `feature2.1`, `feature2.2` |
| คนที่ 3 | Filter + Search + หน้ารายละเอียดแอป | `feature3.1`, `feature3.2` |
| คนที่ 4 | Calculator + Checkout + License + My Library + UI/UX | `feature4.1`, `feature4.2`, `feature4.3` |

**คนที่ 2 เป็นเจ้าของ database schema และ migration แต่ผู้เดียว**

ถ้าคนที่ 1/3/4 พบว่าต้องแก้ schema → **STOP** แล้วแจ้งคนที่ 2 ห้ามแก้ migration เอง

ตัวอย่าง
> คนที่ 4: "ระบบ License ต้องการคอลัมน์ `licenses.revoked_at` เพราะ..."
> คนที่ 2: ทบทวน → สร้าง migration → อัปเดต RLS และ types → แจ้งกลับ
> คนที่ 4: ทำงานต่อ

---

## 13. Shared File Rules

ไฟล์เหล่านี้เป็น high-conflict files แก้ทีละน้อยและต้องแจ้งทีม

- `package.json` และ lockfile
- `src/app/layout.tsx`
- `src/app/globals.css`
- `src/components/ui/*`
- `src/lib/supabase/database.types.ts` (generated)
- `.env.example`
- `vitest.config.ts`, `playwright.config.ts`

---

## 14. Security

ห้าม
- commit secret หรือไฟล์ `.env`
- เปิดเผย Supabase service-role key
- เก็บรหัสผ่านเอง (ใช้ Supabase Auth)
- ปิด RLS เพื่อให้พัฒนาง่ายขึ้น
- พึ่งพา authorization ฝั่ง client อย่างเดียว
- ข้าม authorization เพื่อให้เทสผ่าน
- รับหรือเก็บเลขบัตรเครดิตจริง

---

## 15. Student Project Principle

เลือกทาง
- สถาปัตยกรรมเรียบง่าย
- ตั้งชื่อชัดเจน
- ฟังก์ชันสั้น
- database design ที่อธิบายได้
- เทสที่มีความหมาย
- **หลักฐานการมีส่วนร่วมของแต่ละคนต้องเห็นได้จาก Git**

หลีกเลี่ยง
- enterprise pattern ที่ไม่จำเป็น
- premature optimization
- over-engineering

เป้าหมายไม่ใช่จำนวนฟีเจอร์ แต่คือระบบที่ **ทำงานได้ ทดสอบได้ และอธิบายได้**

---

## 16. Academic Integrity

- **ห้าม rewrite Git history เพื่อปลอมแปลงการมีส่วนร่วม**
- ทุกคนต้องมี commit ที่มองเห็นได้ใน history
- ผลงานของแต่ละคนต้องอธิบายได้ว่าทำอะไร ทำไม และทดสอบยังไง

---

## 17. มติที่ประชุม Phase 0

รายการนี้คือข้อขัดแย้งระหว่างเอกสารที่ตรวจเจอในเฟส 0 และมติที่ใช้แก้ **แก้ลงเอกสารต้นทางแล้วทุกข้อ**
ถ้าเจอโค้ดหรือเอกสารเก่าที่ยังทำตามแบบเดิม ให้ยึดตารางนี้

| # | ปัญหาเดิม | มติ | เอกสารที่แก้ |
|---|---|---|---|
| 1 | seed สั่งว่าป้ายครบ 4 = 100 คะแนน แต่สูตรมี 5 เกณฑ์ ป้ายครบได้ 80 | แก้เงื่อนไข seed ให้ 100 คะแนนต้องผ่านครบ 5 เกณฑ์รวม `support_months >= 12` | DATABASE ข้อ 6 |
| 2 | `pricing_model` ไม่มีค่า `SUBSCRIPTION` ทำให้เกณฑ์ "ไม่ใช่ subscription" เป็นจริงเสมอ | เปลี่ยนเกณฑ์เป็น `pricing_model = 'ONE_TIME'` | ARCHITECTURE ข้อ 6 |
| 3 | คะแนนถูกทั้งเก็บใน DB และคำนวณสด | คำนวณสดอย่างเดียว ตัดคอลัมน์ `sustainable_score` และ `badge_one_time` ทิ้ง | DATABASE ข้อ 3 |
| 4 | ช่วงราคา `1,500+` ทับกับ `501–1,500` | เปลี่ยนเป็น `1,501+` และเพิ่มเทสเส้นแบ่ง 1,500/1,501 | ARCHITECTURE ข้อ 3, DATABASE ข้อ 6, PLAN Phase 5 |
| 5 | `orders.status` มี `PENDING`/`FAILED` ที่ไม่มีวันถูกเขียน | ตัด enum `order_status` และคอลัมน์ `status` ทิ้ง ทุกแถวคือการซื้อที่สำเร็จ | DATABASE ข้อ 2, 3 |
| 6 | ไม่มีใครนิยาม function ออก license ทั้งที่คนละคนเขียนกับคนเรียก | ล็อก `purchase_app(p_app_id uuid) returns text` สร้าง order + license ใน transaction เดียว | DATABASE ข้อ 3.1 |
| 7 | `src/lib/supabase/` มีสองเจ้าของตามเอกสารคนละไฟล์ | คนที่ 1 ถือ client/server/env · คนที่ 2 ถือ `database.types.ts` + migrations | ARCHITECTURE ข้อ 2 |
| 8 | GIN full-text index ไม่มีใครใช้เพราะค้นหาฝั่ง client | ตัด `apps_search_idx` ทิ้ง | DATABASE ข้อ 3 |
| 9 | ชื่อแบนด์คะแนนใช้สองชุด | ใช้ Neutral / Sustainable / Exemplary ชุดเดียว | CLAUDE ข้อ 3, ARCHITECTURE ข้อ 6 |
| 10 | เอกสารนี้เขียนว่าป้องกันแค่ `/library` | เพิ่ม `/checkout/[slug]` ให้ตรงกับ ARCHITECTURE ข้อ 4 | CLAUDE ข้อ 3 |
| 11 | PLAN เขียนว่า migration มี `0001`–`0004` แต่ DATABASE มี 6 ไฟล์ | แก้เป็น `0001`–`0006` | PLAN Phase 3 |
| 12 | จุดคุ้มทุนอาจเกินช่วง slider 5 ปี แต่ไม่มีใครกำหนดว่าจะแสดงยังไง | คืนตัวเลขจริงตามสูตร แต่ UI ต้องบอกว่าเกินช่วงที่เลือก | ARCHITECTURE ข้อ 7 |

### ยังไม่ได้ตัดสิน — ต้องคุยกันเอง

**การเกลี่ยงาน** คนที่ 4 รับ 3 เฟส (Calculator + Checkout/License + UI/UX Polish ทั้งเว็บ)
ขณะที่คนที่ 1 รับงาน tooling กับ auth ข้อเสนอที่ยังไม่ได้แก้ลงเอกสารคือ

1. กระจาย Phase 9 (UI/UX Polish) ให้เจ้าของฟีเจอร์ polish หน้าของตัวเอง
2. ย้าย `/library` ไปให้คนที่ 1 เพราะเป็น protected route ล้วน ๆ
3. ให้คนที่ 1 รับ E2E harness และ test fixture ที่ตอนนี้ไม่มีเจ้าของ

เรื่องนี้กระทบตารางแบ่งงานใน 6 ไฟล์และเป็นเรื่องของคนในทีม จึงปล่อยไว้ให้ตกลงกันก่อน
