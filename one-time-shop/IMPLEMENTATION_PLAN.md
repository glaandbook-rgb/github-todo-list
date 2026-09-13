# IMPLEMENTATION_PLAN.md

---

## เป้าหมาย

สร้าง The One-Time Shop ให้ทำงานได้จริง ทดสอบได้ และอธิบายได้ ด้วยทีม 4 คน

**ทุกเฟสจบด้วย `STOP.` ห้ามเดินหน้าเฟสถัดไปเอง** จุดนั้นคือช่วงที่คนต้องเข้ามา Manual Test

---

## ตารางเฟสและ Branch

| Phase | เจ้าของ | Branch | ขึ้นกับ |
|---|---|---|---|
| 0 — วางสโคป | ทุกคน | — | — |
| 1 — Foundation | คนที่ 1 | `feature1.1` | — |
| 2 — Authentication | คนที่ 1 | `feature1.2` | 1 |
| 3 — Database | คนที่ 2 | `feature2.1` | 1 |
| 4 — Catalog + Badge + Score | คนที่ 2 | `feature2.2` | 3 |
| — **Parallel Gate** — | | | |
| 5 — Filter + Search | คนที่ 3 | `feature3.1` | 4 |
| 6 — หน้ารายละเอียดแอป | คนที่ 3 | `feature3.2` | 4 |
| 7 — Calculator | คนที่ 4 | `feature4.1` | 6 |
| 8 — Checkout + License + Library | คนที่ 4 | `feature4.2` | 2, 3 |
| 9 — UI/UX Polish | คนที่ 4 | `feature4.3` | 5–8 |
| 10 — Integration | ทุกคน | `dev` | ทั้งหมด |
| 11 — Final Audit | ทุกคน | `dev` → `main` | 10 |

---

# Phase 0 — วางสโคปและตกลงสัญญาร่วม
**เจ้าของ: ทุกคน**

ทำร่วมกันในที่ประชุมเดียว ห้ามแยกกันตัดสินใจ

- [ ] อ่าน `CLAUDE.md`, `ARCHITECTURE.md`, `DATABASE.md`, `DESIGN.md` ให้ครบทุกคน
- [ ] ยืนยันสโคปที่ตัดแล้ว โดยเฉพาะข้อ 4 ใน `CLAUDE.md` (Non-Goals)
- [ ] ยืนยันว่า **ใช้ Mock Payment เท่านั้น** ไม่ต่อ gateway จริง
- [ ] ยืนยันว่า **ไม่ทำ Developer Portal และ Admin UI** รอบนี้
- [ ] คนหลักสร้าง repo → push `main` → แตก `dev` → push `dev` → เชิญทุกคนเป็น Collaborator
- [ ] ตั้ง `dev` เป็น default branch และตั้ง branch protection ห้าม push ตรงเข้า `main`/`dev`
- [ ] ทุกคน clone แล้ว `git checkout dev`

**ผลลัพธ์:** repo ที่มี `main` และ `dev` พร้อมใช้ และทุกคนเข้าใจสโคปตรงกัน

**STOP.**

---

# Phase 1 — Foundation
**เจ้าของ: คนที่ 1 · Branch `feature1.1`**

- [ ] `create-next-app` พร้อม TypeScript + Tailwind + App Router
- [ ] ตั้งค่า ESLint และ `typecheck` script
- [ ] ติดตั้งและตั้งค่า Vitest (`vitest.config.ts`, `tests/setup.ts`)
- [ ] ติดตั้งและตั้งค่า Playwright (`playwright.config.ts`)
- [ ] เพิ่ม script ทั้งหมดใน `package.json` ตาม `CLAUDE.md` ข้อ 9
- [ ] แปลง token จาก `DESIGN.md` เป็น Tailwind theme ใน `globals.css` (สี, ฟอนต์ 3 ตัว, radius, spacing 8px)
- [ ] สร้าง shared UI: `Button`, `FormField`, `EmptyState`, `LoadingState`, `ErrorState`
- [ ] สร้าง `.env.example` (ค่าว่าง) และ `.gitignore` ที่บล็อก `.env*` แต่ยกเว้น `.env.example`
- [ ] สร้าง `src/lib/supabase/` — `client.ts`, `server.ts`, `env.ts` (ยังไม่ต้องมี schema)
      **คนที่ 1 เป็นเจ้าของ 3 ไฟล์นี้** ส่วน `database.types.ts` เป็นของคนที่ 2 (มติ Phase 0)

**Tests**
- [ ] smoke test 1 ตัวพิสูจน์ว่า Vitest ทำงาน
- [ ] smoke test 1 ตัวพิสูจน์ว่า Playwright เปิดหน้าแรกได้

**ห้าม** ทำฟีเจอร์ธุรกิจใด ๆ ในเฟสนี้

**เกณฑ์เสร็จ:** `npm run lint`, `typecheck`, `test`, `build` ผ่านหมด

**STOP.**

---

# Phase 2 — Authentication
**เจ้าของ: คนที่ 1 · Branch `feature1.2`**

- [ ] `signUpSchema` และ `signInSchema` ด้วย Zod ใน `lib/validation/auth.ts`
- [ ] Server Action: `signUp`, `signIn`, `signOut`
- [ ] หน้า `/login` และ `/signup` พร้อม error feedback
- [ ] `src/proxy.ts` รีเฟรช session และป้องกัน `/checkout/*`, `/library`
- [ ] `getSession()` helper สำหรับ Server Component
- [ ] ถ้า login แล้วเข้า `/login` ให้ redirect ไป `/`

**Tests**
- [ ] unit: อีเมลผิดรูปแบบถูกปฏิเสธ
- [ ] unit: รหัสผ่านสั้นเกินเกณฑ์ถูกปฏิเสธ
- [ ] unit: เคสขอบความยาวรหัสผ่าน (ตรงเกณฑ์พอดีต้องผ่าน, น้อยกว่า 1 ตัวต้องไม่ผ่าน)
- [ ] integration: เข้า `/library` โดยไม่ login ต้องถูก redirect
- [ ] integration: login แล้วเข้า `/library` ได้

**ห้าม** ทำ catalog, filter, checkout, calculator

**STOP.**

---

# Phase 3 — Database Foundation
**เจ้าของ: คนที่ 2 · Branch `feature2.1`**

- [ ] migration `0001`–`0006` ครบตาม `DATABASE.md` ข้อ 5
- [ ] trigger สร้าง `profiles` อัตโนมัติเมื่อสมัครสมาชิก (`0002`)
- [ ] `purchase_app()` security definer function ตาม signature ใน `DATABASE.md` ข้อ 3.1 (`0005`)
      — สร้าง order + license ใน transaction เดียว และคืน license key
- [ ] seed อย่างน้อย 12 แอปตามเงื่อนไขใน `DATABASE.md` ข้อ 6 (`0006`)
- [ ] generate `database.types.ts` แล้วแจ้งทีม

**Tests**
- [ ] integration: constraint ทำงาน (ราคาติดลบต้อง insert ไม่ได้, `seats < 1` ต้องไม่ได้)
- [ ] integration: รูปแบบ license key ผิดต้อง insert ไม่ได้
- [ ] integration: RLS ครบทั้ง 6 เคสใน `DATABASE.md` ข้อ 7
- [ ] integration: `purchase_app()` เรียกโดยไม่มี session ต้อง error
- [ ] integration: `purchase_app()` กับแอปที่ `is_published = false` ต้อง error
- [ ] integration: เรียก `purchase_app()` ซ้ำแอปเดิมแล้ว unique index ต้องกัน และ **ต้องไม่เหลือ order ค้าง**

**ห้าม** ทำ UI

**STOP.**

---

# Phase 4 — Catalog + Badge + Sustainable Score
**เจ้าของ: คนที่ 2 · Branch `feature2.2`**

- [ ] `calculateSustainableScore()` ตามสูตรใน `ARCHITECTURE.md` ข้อ 6
      **คำนวณสดจากข้อมูลแอป ไม่อ่านคอลัมน์คะแนนจาก DB** (ไม่มีคอลัมน์นั้นแล้ว)
- [ ] `scoreBand()` คืน Neutral / Sustainable / Exemplary
- [ ] `getPublishedApps()` query
- [ ] `AppCard` แสดงชื่อ ผู้พัฒนา ราคา ป้าย และคะแนน
- [ ] `BadgeChip` ป้าย 4 ชนิด พร้อม tooltip อธิบาย
- [ ] `ScoreMeter` แสดงคะแนนพร้อมสีตามแบนด์
- [ ] หน้า Home แสดงรายการแอปทั้งหมด (ยังไม่ต้องมี filter)

**Tests**
- [ ] unit: คะแนน 0 เมื่อไม่ผ่านเกณฑ์ใดเลย
- [ ] unit: คะแนน 100 เมื่อผ่านครบทั้ง 5 เกณฑ์
- [ ] unit: `pricing_model` ไม่ใช่ `ONE_TIME` ต้องไม่ได้คะแนนเกณฑ์แรก
- [ ] unit: ป้ายครบ 3 แต่ `support_months = 11` ต้องได้ 80 ไม่ใช่ 100
- [ ] unit: เคสขอบแบนด์ที่ 0, 59, 60, 89, 90, 100
- [ ] unit: คะแนนนอกช่วงต้อง throw
- [ ] integration: `getPublishedApps()` ไม่คืนแอปที่ `is_published = false`

**STOP.**

---

# Parallel Gate

เริ่มงานคู่ขนานได้ก็ต่อเมื่อครบทุกข้อ

- [ ] Phase 1–4 merge เข้า `dev` แล้ว
- [ ] `database.types.ts` นิ่งแล้ว
- [ ] เทสทั้งหมดบน `dev` ผ่าน
- [ ] ทุกคน `git pull origin dev` ล่าสุดแล้ว

จากนั้น คนที่ 3 และคนที่ 4 ทำงานพร้อมกันได้

---

# Phase 5 — Filter + Search
**เจ้าของ: คนที่ 3 · Branch `feature3.1`**

- [ ] `applyFilters(apps, criteria)` ฟังก์ชันบริสุทธิ์ใน `lib/filter/predicate.ts`
- [ ] `matchesSearch(app, term)` ค้นจากชื่อ สโลแกน แท็ก ชื่อผู้พัฒนา
- [ ] `CategoryTabs` 5 หมวดหมู่
- [ ] `FilterBar` 4 มิติทำงานพร้อมกันแบบ AND
- [ ] `SearchInput` debounce ≤ 250ms
- [ ] Empty state พร้อมปุ่มล้างตัวกรองทั้งหมด
- [ ] กรองโดยไม่โหลดหน้าใหม่ ผลลัพธ์ภายใน 300ms

**Tests**
- [ ] unit: กรองหมวดหมู่เดียวได้ถูกต้อง
- [ ] unit: กรอง 4 มิติพร้อมกันเป็น AND ไม่ใช่ OR
- [ ] unit: ไม่เลือกตัวกรองเลยต้องคืนทุกแอป
- [ ] unit: เคสขอบช่วงราคาล่าง — แอปราคา 500 และ 501 ต้องตกคนละช่วง
- [ ] unit: เคสขอบช่วงราคาบน — แอปราคา 1,500 และ 1,501 ต้องตกคนละช่วง (ช่วงบนสุดคือ `1,501+`)
- [ ] unit: ค้นหาไม่สนตัวพิมพ์เล็กใหญ่
- [ ] unit: ค้นหาเจอจากชื่อผู้พัฒนาด้วย ไม่ใช่แค่ชื่อแอป
- [ ] unit: กรองแล้วไม่พบต้องคืน array ว่าง ไม่ใช่ throw
- [ ] integration: กรองแล้ว UI แสดง empty state และปุ่มล้างตัวกรองใช้ได้จริง

**ถ้าต้องแก้ schema → STOP แล้วแจ้งคนที่ 2**

**STOP.**

---

# Phase 6 — หน้ารายละเอียดแอป
**เจ้าของ: คนที่ 3 · Branch `feature3.2`**

- [ ] หน้า `/app/[slug]`
- [ ] `PricingCard`: ราคาสุทธิตัวใหญ่, จำนวนเครื่อง, นโยบายอัปเดต, ระยะซัพพอร์ต
- [ ] `NoHiddenFeesList` checklist ใช้ check-circle สีเขียวตาม `DESIGN.md`
- [ ] ป้ายการันตีและคะแนนความยั่งยืน
- [ ] ปุ่ม "ซื้อขาดตอนนี้" (ยังไม่ต้องต่อ checkout ในเฟสนี้)
- [ ] `not-found.tsx` สำหรับ slug ที่ไม่มีอยู่

**Tests**
- [ ] integration: เปิด slug ที่มีอยู่แล้วเห็นราคาถูกต้อง
- [ ] integration: เปิด slug ที่ไม่มีอยู่ต้องได้ 404
- [ ] integration: แอปที่ `is_published = false` ต้องเปิดไม่ได้

**STOP.**

---

# Phase 7 — Subscription Cost Calculator
**เจ้าของ: คนที่ 4 · Branch `feature4.1`**

- [ ] `subscriptionCostOverYears(monthly, years)`
- [ ] `savingsAmount(oneTimePrice, monthly, years)`
- [ ] `calculateBreakEvenMonth(oneTimePrice, monthly)`
- [ ] `YearsSlider` 1–5 ปี
- [ ] `CostCalculator` แสดงต้นทุนสองฝั่งและส่วนต่าง อัปเดตทันทีเมื่อเลื่อน slider
- [ ] ซ่อน Calculator เมื่อ `competitor_monthly_thb` เป็น NULL
- [ ] บนมือถือ reflow เป็น card แนวตั้ง

**Tests** (สูตรอยู่ใน `ARCHITECTURE.md` ข้อ 7)
- [ ] unit: เคสจากสไลด์ — `P=1200, M=350, Y=5` → 21,000 / 19,800 / เดือนที่ 4
- [ ] unit: `Y=1` คำนวณถูก
- [ ] unit: `M=0` คืน `null` ห้ามหารด้วยศูนย์
- [ ] unit: ราคาซื้อขาดแพงกว่า → savings ติดลบ ต้องรายงานว่ายังไม่คุ้ม
- [ ] unit: `P < 0` หรือ `M < 0` ต้อง throw
- [ ] unit: `Y` นอกช่วง 1–5 ต้อง throw
- [ ] unit: จุดคุ้มทุนต้องปัดขึ้นเสมอ (`P=1200, M=350` → 4 ไม่ใช่ 3)
- [ ] unit: จุดคุ้มทุนเกินช่วง slider (`P=50000, M=100` → เดือนที่ 500) ต้องคืนค่าจริง ไม่ throw
- [ ] integration: เมื่อจุดคุ้มทุนเกินช่วงที่เลือก UI ต้องบอกว่าเกินช่วง ไม่ใช่วาดนอกกราฟ
- [ ] integration: เลื่อน slider แล้วตัวเลขบนหน้าจอเปลี่ยนตาม

**STOP.**

---

# Phase 8 — Checkout + License + My Library
**เจ้าของ: คนที่ 4 · Branch `feature4.2`**

- [ ] `generateLicenseKey(randomSource)` รูปแบบ `OTS-XXXX-XXXX` — ใช้ฝั่ง TypeScript สำหรับ unit test
      และตรวจรูปแบบ ส่วนคีย์ที่ออกจริงมาจาก `purchase_app()` ใน Postgres
- [ ] `isValidLicenseFormat(key)`
- [ ] หน้า `/checkout/[slug]` แสดงยอดสุทธิ ไม่มีรายการพ่วงติ๊กไว้
- [ ] `MockPaymentForm` เลือกผลสำเร็จ/ล้มเหลว พร้อมข้อความว่าเป็นการจำลอง
- [ ] Server Action `purchaseApp()` ตรวจ session → เรียก `supabase.rpc("purchase_app", { p_app_id })`
      **ห้าม insert เข้า `orders` หรือ `licenses` ตรง ๆ** ทั้งสองตารางไม่มี INSERT policy ให้ผู้ใช้ทั่วไป
- [ ] หน้า `/library` แสดง license พร้อมปุ่มคัดลอกคีย์
- [ ] Empty state เมื่อยังไม่มีแอป

**Tests**
- [ ] unit: license key ตรง regex `^OTS-[A-Z0-9]{4}-[A-Z0-9]{4}$`
- [ ] unit: random source เดิมต้องได้คีย์เดิม (deterministic)
- [ ] unit: `isValidLicenseFormat` ปฏิเสธคีย์ผิดรูปแบบ
- [ ] integration: ซื้อสำเร็จ → มี order 1 แถว และมี license 1 ใบ ผูก `order_id` ตรงกัน
- [ ] integration: ชำระเงินล้มเหลว → **ไม่มี** order และ **ไม่มี** license (ไม่แตะ DB เลย)
- [ ] integration: ซื้อโดยไม่ login ต้องถูกปฏิเสธ
- [ ] integration: ซื้อแอปเดิมซ้ำต้องถูกปฏิเสธ
- [ ] integration: ผู้ใช้ A ไม่เห็น license ของผู้ใช้ B ใน `/library`

**ห้าม** ต่อ payment gateway จริง ห้ามมีช่องกรอกเลขบัตร ห้ามมี auto-renew

**STOP.**

---

# Phase 9 — UI/UX Polish
**เจ้าของ: คนที่ 4 · Branch `feature4.3`**

- [ ] Responsive ครบทุกหน้าที่ 375px / 768px / 1280px
- [ ] `loading.tsx` ทุกหน้าที่ดึงข้อมูล
- [ ] `error.tsx` ระดับ root และระดับหน้า
- [ ] Empty state ครบตาม `ARCHITECTURE.md` ข้อ 10
- [ ] Form feedback: สถานะ pending, ข้อความ error ที่อ่านรู้เรื่อง
- [ ] Accessibility พื้นฐาน: label ผูกกับ input, focus ring มองเห็นชัด, contrast ผ่าน

**Tests**
- [ ] integration: ระหว่างส่งฟอร์ม ปุ่มต้องถูก disable และแสดงสถานะ pending
- [ ] integration: empty state แสดงถูกต้องเมื่อไม่มีข้อมูล

**ห้าม** เขียนเทสที่ผูกกับ class ของ Tailwind หรือโครงสร้าง markup เพราะจะพังทุกครั้งที่ปรับดีไซน์

**STOP.**

---

# Phase 10 — Integration
**เจ้าของ: ทุกคน · ทำบน `dev`**

**ห้ามเพิ่มฟีเจอร์ใหม่** แก้เฉพาะปัญหาที่เกิดจากการรวมโค้ด

ตรวจ flow เต็ม

```
สมัครสมาชิก → เข้าสู่ระบบ → หน้า Home → กรอง/ค้นหา
→ เปิดรายละเอียดแอป → ใช้ Calculator → กดซื้อ
→ Checkout (mock) → ได้ License Key → My Library
```

- [ ] E2E ครบทั้ง 8 journey ใน `CLAUDE.md` ข้อ 7
- [ ] regression test ข้ามโมดูล
- [ ] `npm run test` ผ่านทั้งหมดบน `dev`

**STOP.**

---

# Phase 11 — Final Audit
**เจ้าของ: ทุกคน**

- [ ] ฟีเจอร์ครบตามสโคป ไม่มีของเกินสโคปหลุดเข้ามา
- [ ] `npm run lint` ผ่าน
- [ ] `npm run typecheck` ผ่าน
- [ ] `npm run build` ผ่าน
- [ ] `npm run test` ผ่าน
- [ ] `npm run test:e2e` ผ่าน
- [ ] ไม่มี `console.log` ที่ใช้ debug หลงเหลือ
- [ ] ไม่มี secret ใน repo — ตรวจด้วย `git log -p | grep -i "key\|secret\|password"`
- [ ] RLS ยังเปิดครบทุกตาราง ไม่มีใครปิดเพื่อให้เทสผ่าน
- [ ] `README.md` มีวิธีรันและตารางแบ่งงาน
- [ ] ทุกคนมี commit ปรากฏใน history
- [ ] branch ครบใน GitHub: `main`, `dev`, `feature1.1`–`feature4.3` (**ห้ามลบ branch หลัง merge**)
- [ ] merge `dev` → `main`
- [ ] ส่งลิงก์ repo ใน Discord

**ห้ามเพิ่มฟีเจอร์ใหม่ในเฟสนี้**

**STOP.**

---

## Definition of Done ของ 1 ฟีเจอร์

ขาดข้อใดข้อหนึ่งห้าม merge

| # | เกณฑ์ | ตรงกับโจทย์ข้อ |
|---|---|---|
| 1 | มี plan จาก AI ก่อนเขียนโค้ด และคนอ่านแล้วเห็นชอบ | 2 |
| 2 | โค้ดหลักเสร็จตาม checklist ของเฟส | 1 |
| 3 | มี unit test เขียน **หลัง** โค้ดหลัก และลองทำให้พังแล้วเทสแดงจริง | 3 |
| 4 | Manual test ด้วยตัวเอง บันทึกผลลง PR | 4 |
| 5 | `lint` / `typecheck` / `test` ผ่าน | — |
| 6 | เขียน Phase Completion Report 9 ข้อลง PR | 2,3,4 |
