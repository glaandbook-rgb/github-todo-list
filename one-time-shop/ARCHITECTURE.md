# ARCHITECTURE.md

---

## 1. หลักการออกแบบ

1. **Server-first** — ดึงข้อมูลใน Server Component และเขียนข้อมูลผ่าน Server Action ไม่สร้าง REST API ซ้อนอีกชั้นโดยไม่จำเป็น
2. **Logic แยกจาก UI** — ตรรกะธุรกิจทั้งหมดอยู่ใน `src/lib/` เป็นฟังก์ชันบริสุทธิ์ เพื่อให้ unit test ได้โดยไม่ต้อง render component
3. **Database เป็นด่านสุดท้าย** — RLS ต้องป้องกันได้แม้โค้ดฝั่งแอปพลาด ไม่พึ่ง authorization ฝั่ง client
4. **หนึ่งโฟลเดอร์ = หนึ่งเจ้าของ** — ลด merge conflict ตั้งแต่ระดับโครงสร้าง

---

## 2. โครงสร้างโฟลเดอร์และเจ้าของ

```
src/
├── app/
│   ├── layout.tsx                 ⚠️ shared
│   ├── globals.css                ⚠️ shared
│   ├── page.tsx                   หน้า Home (catalog)      คนที่ 3
│   ├── login/page.tsx                                      คนที่ 1
│   ├── signup/page.tsx                                     คนที่ 1
│   ├── app/[slug]/page.tsx        หน้ารายละเอียดแอป         คนที่ 3
│   ├── checkout/[slug]/page.tsx   หน้า Checkout            คนที่ 4
│   └── library/page.tsx           My Library (protected)   คนที่ 4
│
├── components/
│   ├── auth/                      SignUpForm, SignInForm   คนที่ 1
│   ├── catalog/                   AppCard, BadgeChip,
│   │                              ScoreMeter               คนที่ 2
│   ├── filter/                    FilterBar, SearchInput,
│   │                              CategoryTabs             คนที่ 3
│   ├── detail/                    PricingCard,
│   │                              NoHiddenFeesList         คนที่ 3
│   ├── calculator/                CostCalculator,
│   │                              YearsSlider              คนที่ 4
│   ├── checkout/                  CheckoutSummary,
│   │                              MockPaymentForm          คนที่ 4
│   ├── library/                   LicenseCard,
│   │                              CopyKeyButton            คนที่ 4
│   └── ui/                        ⚠️ shared — Button,
│                                  FormField, EmptyState,
│                                  LoadingState, ErrorState
│
├── lib/
│   ├── supabase/                  client, server, env      คนที่ 1
│   │                              database.types.ts ⚠️     คนที่ 2
│   ├── auth/                      actions, session         คนที่ 1
│   ├── catalog/                   queries, score           คนที่ 2
│   ├── filter/                    predicate, search        คนที่ 3
│   ├── calculator/                cost, breakeven          คนที่ 4
│   ├── orders/                    actions, license         คนที่ 4
│   └── validation/                auth, checkout, filter   ตามเจ้าของฟีเจอร์
│
└── proxy.ts                       session refresh          คนที่ 1

supabase/migrations/               ⚠️ คนที่ 2 เท่านั้น

tests/
├── unit/                          ฟังก์ชันบริสุทธิ์
├── integration/                   logic + database
└── e2e/                           user journey
```

`⚠️` = shared file ต้องแจ้งทีมก่อนแก้

> **มติ Phase 0 — เจ้าของ `src/lib/supabase/`** เดิมเอกสารนี้เขียนว่าเป็นของคนที่ 2 ทั้งโฟลเดอร์
> แต่ `IMPLEMENTATION_PLAN.md` Phase 1 สั่งให้คนที่ 1 สร้าง client/server/env ตกลงแบ่งเป็น
>
> - **คนที่ 1** — `client.ts`, `server.ts`, `env.ts` (ผูกกับ session และ `proxy.ts` ซึ่งเป็นงาน auth)
> - **คนที่ 2** — `database.types.ts` และ `supabase/migrations/` เท่านั้น

---

## 3. Data Flow

### การอ่าน (Catalog / Detail / Library)

```
Server Component
  → src/lib/<module>/queries.ts
    → Supabase server client (อ่าน cookie session)
      → Postgres + RLS กรองสิทธิ์
        → คืนข้อมูล → render
```

### การเขียน (Sign up / Checkout)

```
Client Component (form)
  → Server Action ใน src/lib/<module>/actions.ts
    → Zod validate
    → ตรวจ session
    → เขียน Postgres (RLS ตรวจซ้ำอีกชั้น)
    → revalidatePath()
```

### การกรอง/ค้นหา (เสาหลักที่ 2)

รอบแรกดึงรายการแอปจาก server แล้วกรองฝั่ง client เพื่อให้ได้ผลภายใน 300ms โดยไม่ต้องยิง network ทุกครั้ง

```
apps[] (จาก server)
  → useState สำหรับ filter state
  → useDeferredValue / debounce 250ms สำหรับ search term
    → applyFilters(apps, criteria)   ← pure function, unit test ได้
      → render ผลลัพธ์
```

> `applyFilters` ต้องเป็นฟังก์ชันบริสุทธิ์ ไม่แตะ DOM ไม่เรียก network เพื่อให้เทสได้ตรง ๆ

### ช่วงราคา — เส้นแบ่งที่ต้องไม่ทับกัน

| ช่วง | เงื่อนไข |
|---|---|
| `0–500` | `price_thb <= 500` |
| `501–1,500` | `price_thb >= 501 and price_thb <= 1500` |
| `1,501+` | `price_thb >= 1501` |

> **มติ Phase 0** เดิมช่วงบนสุดเขียนว่า `1,500+` ทำให้ราคา 1,500 ตกสองช่วงพร้อมกัน
> แก้เป็น `1,501+` แล้ว และต้องมี unit test เส้นแบ่งทั้งสองจุด: 500/501 และ 1,500/1,501

---

## 4. Authentication & Route Protection

| เส้นทาง | สิทธิ์ |
|---|---|
| `/`, `/app/[slug]` | เปิดสาธารณะ |
| `/login`, `/signup` | เปิดสาธารณะ (ถ้า login แล้วให้เด้งไป `/`) |
| `/checkout/[slug]` | ต้อง login |
| `/library` | ต้อง login |

การป้องกันมี **2 ชั้น**

1. `src/proxy.ts` — รีเฟรช session และ redirect ก่อนถึงหน้า
2. Server Component / Server Action — ตรวจ session ซ้ำก่อนอ่านหรือเขียนข้อมูล

ห้ามป้องกันด้วย `useEffect` + `router.push` ฝั่ง client อย่างเดียว

---

## 5. โมดูลที่ต้อง Unit Test (ฟังก์ชันบริสุทธิ์)

| ไฟล์ | ฟังก์ชัน | เจ้าของ |
|---|---|---|
| `lib/validation/auth.ts` | `signUpSchema`, `signInSchema` | คนที่ 1 |
| `lib/catalog/score.ts` | `calculateSustainableScore`, `scoreBand` | คนที่ 2 |
| `lib/filter/predicate.ts` | `applyFilters`, `matchesSearch` | คนที่ 3 |
| `lib/calculator/cost.ts` | `subscriptionCostOverYears`, `savingsAmount` | คนที่ 4 |
| `lib/calculator/breakeven.ts` | `calculateBreakEvenMonth` | คนที่ 4 |
| `lib/orders/license.ts` | `generateLicenseKey`, `isValidLicenseFormat` | คนที่ 4 |

ทุกฟังก์ชันในตารางนี้ต้องไม่มี side effect และไม่เรียก Supabase

---

## 6. Sustainable Score — สูตรคำนวณ

คะแนน 0–100 มาจาก 5 เกณฑ์ เกณฑ์ละ 20 คะแนน

**คะแนนนี้ไม่ถูกเก็บในฐานข้อมูล** คำนวณสดทุกครั้งจากข้อมูลของแอป (มติ Phase 0 — ดู `DATABASE.md` ข้อ 3)

| เกณฑ์ | ให้คะแนนเมื่อ |
|---|---|
| Verified One-Time | `pricing_model = 'ONE_TIME'` |
| Lifetime License | สิทธิ์ใช้งานไม่มีวันหมดอายุ |
| Offline-Friendly | ใช้งานได้โดยไม่ต้องต่ออินเทอร์เน็ต |
| Privacy Verified | ไม่เก็บข้อมูลผู้ใช้เกินจำเป็น |
| Support Duration | มีระยะซัพพอร์ต ≥ 12 เดือน |

แบนด์สี

| ช่วงคะแนน | แบนด์ | สีจาก DESIGN.md |
|---|---|---|
| 0–59 | Neutral (เทา) | `outline` |
| 60–89 | Sustainable (เขียว) | `primary` |
| 90–100 | Exemplary (ทอง) | `sustainable-gold` |

> **ชื่อแบนด์ใช้ชุดนี้ชุดเดียวทั้งโปรเจกต์** ทั้งในโค้ด เอกสาร และสไลด์
> (`CLAUDE.md` เดิมเรียก Gray / Green / Gold — แก้ให้ตรงกันแล้ว)

**เคสขอบที่ต้องมี unit test:** 0, 59, 60, 89, 90, 100 และค่านอกช่วงต้อง throw

> **รู้ไว้:** เมื่อเกณฑ์มี 5 ข้อ ข้อละ 20 คะแนน `calculateSustainableScore()` จะคืนได้แค่
> 0 / 20 / 40 / 60 / 80 / 100 เท่านั้น แปลว่า **มีเฉพาะแอปที่ผ่านครบทั้ง 5 เกณฑ์เท่านั้นที่ได้ทอง**
> เป็นเรื่องที่ยอมรับได้ แต่ seed ต้องมีแอป 100 คะแนนอย่างน้อย 1 ตัวไม่งั้นจะไม่เห็นแบนด์ทองเลย
> ส่วนเคสขอบ 59 / 89 ใช้เทส `scoreBand()` โดยตรง ไม่ได้มาจากผลของ `calculateSustainableScore()`

---

## 7. Calculator — สูตรคำนวณ

กำหนดให้
- `P` = ราคาซื้อขาด (บาท)
- `M` = ค่าบริการ Subscription ของคู่เทียบ (บาท/เดือน)
- `Y` = จำนวนปีที่เลือกจาก slider (1–5)

```
subscriptionCost(Y) = M × 12 × Y
savings(Y)          = subscriptionCost(Y) − P
breakEvenMonth      = ceil(P ÷ M)
```

ตัวอย่างตรวจสอบจากสไลด์ (ต้องใช้เป็น unit test)

```
P = 1200, M = 350, Y = 5
subscriptionCost = 350 × 12 × 5 = 21,000
savings          = 21,000 − 1,200 = 19,800
breakEvenMonth   = ceil(1200 ÷ 350) = 4
```

**เคสขอบที่ต้องมี unit test**
- `M = 0` → ไม่มีจุดคุ้มทุน คืน `null` ห้ามหารด้วยศูนย์
- `breakEvenMonth > 12 × Y` → จุดคุ้มทุนอยู่นอกช่วงที่ slider ครอบคลุม ต้องยังคืนตัวเลขจริงตามสูตร
  แต่ UI ต้องบอกผู้ใช้ว่า "เกินช่วงที่เลือก" ไม่ใช่วาดจุดคุ้มทุนนอกกราฟ (มติ Phase 0)
- `P > subscriptionCost(Y)` → `savings` ติดลบ ต้องแสดงว่า "ยังไม่คุ้ม" ไม่ใช่โชว์เลขติดลบเฉย ๆ
- `P < 0` หรือ `M < 0` → throw
- `Y` นอกช่วง 1–5 → throw

---

## 8. License Key

รูปแบบ `OTS-XXXX-XXXX` โดย `X` คือ `[A-Z0-9]`

ข้อกำหนด
- สร้างด้วย `crypto.randomUUID()` หรือ random ที่ปลอดภัย ห้ามใช้ `Math.random()`
- ต้อง unique ระดับฐานข้อมูล (`UNIQUE` constraint)
- ไม่มีวันหมดอายุ — ไม่มีคอลัมน์ `expires_at`
- `generateLicenseKey()` ต้องเป็นฟังก์ชันที่รับ random source เข้ามาได้ เพื่อให้ unit test กำหนดผลลัพธ์ได้

---

## 9. Mock Payment

```
CheckoutSummary  (แสดงยอดสุทธิ ไม่มีรายการพ่วง)
  → MockPaymentForm  (เลือก "สำเร็จ" หรือ "ล้มเหลว")
    → Server Action: purchaseApp(slug, result)
      ├── ล้มเหลว → คืน error ทันที ไม่แตะฐานข้อมูลเลย
      └── สำเร็จ  → supabase.rpc("purchase_app", { p_app_id })
                  → function สร้าง order + license ใน transaction เดียว
                  → คืน license key → redirect ไป /library
```

> **มติ Phase 0** ฝั่งแอปไม่ `insert` เข้า `orders` หรือ `licenses` เองอีกต่อไป ทั้งสองตารางไม่มี
> INSERT policy สำหรับผู้ใช้ทั่วไปแล้ว signature ของ function อยู่ใน `DATABASE.md` ข้อ 3.1

ข้อบังคับ
- หน้าจอต้องมีข้อความชัดเจนว่าเป็นการจำลอง ไม่มีการตัดเงินจริง
- ห้ามมีช่องกรอกเลขบัตร
- ห้ามมี checkbox ต่ออายุอัตโนมัติ
- Server Action ต้องตรวจ session ก่อนเสมอ

---

## 10. Error / Loading / Empty States

Next.js App Router รองรับไฟล์พิเศษ ให้ใช้ตามนี้

| ไฟล์ | ใช้เมื่อ |
|---|---|
| `loading.tsx` | ระหว่างโหลด Server Component |
| `error.tsx` | เมื่อเกิด error ที่ไม่คาดคิด |
| `not-found.tsx` | เปิด `/app/[slug]` ที่ไม่มีอยู่ |

Empty state ที่ต้องมี
- ค้นหา/กรองแล้วไม่พบผลลัพธ์ → พร้อมปุ่มล้างตัวกรองทั้งหมด
- My Library ยังไม่มีแอป → พร้อมปุ่มกลับไปหน้า Home

---

## 11. Design System

ยึดตาม `DESIGN.md` (Ethos Sustainable Digital) แปลง token เป็น Tailwind theme ใน `globals.css`

จุดที่ต้องระวัง
- **ห้ามใช้ gradient จัด ๆ บนปุ่ม** ใช้สีทึบเท่านั้น เพื่อสื่อความโปร่งใส
- Card ใช้ border 1px ไม่ใช้เงาหนัก
- ราคาและคะแนนใช้ JetBrains Mono
- บนมือถือ ตารางเปรียบเทียบต้อง reflow เป็น card แนวตั้ง
- ระยะห่างทุกค่าเป็นพหุคูณของ 8px
