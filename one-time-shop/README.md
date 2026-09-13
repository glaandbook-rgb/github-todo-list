# The One-Time Shop

ตลาดกลางซอฟต์แวร์แบบซื้อขาด ที่คืนความโปร่งใสด้านราคาให้ผู้บริโภค และสร้างช่องทางรายได้ที่เป็นธรรมให้นักพัฒนา Indie และ Open Source

> "ซื้อครั้งเดียว คุ้มค่าตลอดไป — จ่ายเท่าที่ควรจ่าย เป็นเจ้าของจริง"

---

## เอกสารโครงการ

| ไฟล์ | เนื้อหา |
|---|---|
| [CLAUDE.md](./CLAUDE.md) | สเปกงาน สโคป และกติกาที่ทุกคนต้องทำตาม |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | โครงสร้างโฟลเดอร์ data flow และสูตรคำนวณ |
| [DATABASE.md](./DATABASE.md) | schema, RLS และลำดับ migration |
| [DESIGN.md](./DESIGN.md) | Design system (Ethos Sustainable Digital) |
| [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md) | แผน 12 เฟส พร้อม STOP gate |
| [TEAM_WORKFLOW.md](./TEAM_WORKFLOW.md) | กติกา Git, PR และความเป็นเจ้าของไฟล์ |
| `PROMPT_00` – `PROMPT_06` | prompt สำหรับสั่ง AI แต่ละเฟส |

---

## Stack

Next.js (App Router) · TypeScript · Tailwind CSS v4 · Supabase (Postgres + RLS) · Zod · Vitest · Playwright

---

## วิธีรัน

1. คัดลอกไฟล์ environment

   ```bash
   cp .env.example .env.local
   ```

   แล้วกรอก `NEXT_PUBLIC_SUPABASE_URL` และ `NEXT_PUBLIC_SUPABASE_ANON_KEY`

2. Push migration ใน `supabase/migrations/` ขึ้นโปรเจกต์ Supabase (ดู [DATABASE.md](./DATABASE.md))

3. ติดตั้งและรัน

   ```bash
   npm install
   npm run dev
   ```

เปิด http://localhost:3000

---

## คำสั่งทดสอบ

```bash
npm run test              # unit + integration (Vitest)
npm run test:unit
npm run test:integration
npm run test:e2e          # Playwright
npm run lint
npm run typecheck
npm run build
```

---

## การแบ่งงาน

| คน | ขอบเขต | Branch |
|---|---|---|
| คนที่ 1 | Foundation, Design System, Authentication | `feature1.1`, `feature1.2` |
| คนที่ 2 | Database, RLS, Catalog, Badge, Sustainable Score | `feature2.1`, `feature2.2` |
| คนที่ 3 | Filter, Search, หน้ารายละเอียดแอป | `feature3.1`, `feature3.2` |
| คนที่ 4 | Calculator, Checkout, License, My Library, UI/UX | `feature4.1`, `feature4.2`, `feature4.3` |

Git flow: `feature*` → `dev` → `main` (ห้าม push ตรงเข้า `main` หรือ `dev`)

---

## สโคปรอบนี้

**ทำ** — Auth · คลังแอป + ป้ายการันตี 4 ชนิด + Sustainable Score · ตัวกรอง 4 มิติ + live search · หน้ารายละเอียด + ราคาโปร่งใส · Subscription Cost Calculator · Mock Checkout + Perpetual License + My Library

**ไม่ทำรอบนี้ (อนาคตและการต่อยอด)** — การชำระเงินจริง · License verification API · ระบบคืนเงิน · Developer Portal · หน้า Admin / Curator · Alternative Ecosystem (Linux, Android TV, IoT)
