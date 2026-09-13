# DATABASE.md

**เจ้าของ: คนที่ 2 เท่านั้น** — คนอื่นห้ามแก้ไฟล์ใน `supabase/migrations/`

---

## 1. ภาพรวม

4 ตารางหลัก

```
auth.users  (Supabase จัดการเอง)
   │
   └─1:1─→ profiles
              │
              └─1:N─→ orders ──1:1──→ licenses
                         │              │
         apps ←──N:1─────┴──────────────┘
```

---

## 2. ENUM

```sql
create type user_role     as enum ('MEMBER', 'ADMIN');
create type pricing_model as enum ('ONE_TIME', 'PAY_PER_USE', 'PAY_WHAT_YOU_WANT');
create type app_category  as enum (
  'PRODUCTIVITY', 'DEVELOPER', 'DESIGN', 'SECURITY', 'MEDIA'
);
```

> `ADMIN` มีไว้เพื่อรองรับอนาคตและใช้ทดสอบ RLS เท่านั้น **รอบนี้ไม่มีหน้าจอ Admin**

> **มติ Phase 0 — ไม่มี `order_status`** เดิมมี enum `('PENDING','PAID','FAILED')` แต่ flow จริงใน
> `ARCHITECTURE.md` ข้อ 9 สร้าง order ก็ต่อเมื่อจ่ายสำเร็จเท่านั้น และ `orders` ห้ามมี UPDATE policy
> ทำให้ `PENDING` กับ `FAILED` ไม่มีวันถูกเขียนลงฐานข้อมูล จึงตัดทั้ง enum และคอลัมน์ทิ้ง
> **ทุกแถวใน `orders` คือการซื้อที่สำเร็จแล้ว**

---

## 3. ตาราง

### `profiles`

| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | `uuid` PK | อ้างอิง `auth.users(id)` ON DELETE CASCADE |
| `email` | `text` NOT NULL | |
| `display_name` | `text` | |
| `role` | `user_role` NOT NULL | default `'MEMBER'` |
| `created_at` | `timestamptz` NOT NULL | default `now()` |

สร้างอัตโนมัติด้วย trigger เมื่อมีผู้ใช้ใหม่ใน `auth.users`

### `apps`

| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | `uuid` PK | default `gen_random_uuid()` |
| `slug` | `text` UNIQUE NOT NULL | ใช้ใน URL |
| `name` | `text` NOT NULL | |
| `tagline` | `text` NOT NULL | ใช้ค้นหา |
| `description` | `text` | |
| `vendor_name` | `text` NOT NULL | ใช้ค้นหา |
| `category` | `app_category` NOT NULL | |
| `platforms` | `text[]` NOT NULL | `{'windows','macos','linux','web'}` |
| `tags` | `text[]` | ใช้ค้นหา |
| `pricing_model` | `pricing_model` NOT NULL | |
| `price_thb` | `integer` NOT NULL | CHECK `>= 0` |
| `seats` | `integer` NOT NULL | default 1, CHECK `>= 1` |
| `update_policy` | `text` | เช่น "อัปเดตฟรีตลอดเวอร์ชัน 2.x" |
| `support_months` | `integer` NOT NULL | default 0, CHECK `>= 0` |
| `competitor_monthly_thb` | `integer` | ค่าคู่เทียบสำหรับ Calculator, NULL = ไม่มีคู่เทียบ |
| `badge_lifetime` | `boolean` NOT NULL | default false |
| `badge_offline` | `boolean` NOT NULL | default false |
| `badge_privacy` | `boolean` NOT NULL | default false |
| `is_published` | `boolean` NOT NULL | default false |
| `created_at` | `timestamptz` NOT NULL | default `now()` |

> **มติ Phase 0 — ไม่มีคอลัมน์ `sustainable_score` และ `badge_one_time`**
>
> - คะแนนคำนวณสดจาก 5 เกณฑ์ใน `ARCHITECTURE.md` ข้อ 6 ด้วย `calculateSustainableScore()`
>   ถ้าเก็บซ้ำไว้ในฐานข้อมูลด้วย จะมีสองแหล่งความจริงที่ไม่ตรงกันได้ทันทีที่แก้ป้ายหรือ `support_months`
> - ป้าย `Verified One-Time` ตัดสินจาก `pricing_model = 'ONE_TIME'` ไม่ต้องมี boolean ซ้ำ
>
> ที่เหลืออีก 3 ป้ายเป็น boolean จริง เพราะตรวจสอบด้วยคนแล้วบันทึกไว้ ไม่มีทางคำนวณจากคอลัมน์อื่น

**Index**

```sql
create index apps_category_idx   on apps (category) where is_published;
create index apps_price_idx      on apps (price_thb) where is_published;
```

> **มติ Phase 0 — ตัด `apps_search_idx` (GIN full-text) ทิ้ง** เพราะ `ARCHITECTURE.md` ข้อ 3
> กำหนดให้ค้นหาฝั่ง client จาก array ที่โหลดมาแล้ว Postgres จะไม่ได้ใช้ index นี้เลย
> (และ index เดิมยังไม่ครอบคลุม `tags` ที่ `matchesSearch` ต้องค้นด้วย)
> ถ้าอนาคตย้ายการค้นหาไปฝั่ง server ค่อยเพิ่ม index กลับมาพร้อมกัน

### `orders`

| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | `uuid` PK | |
| `user_id` | `uuid` NOT NULL | → `profiles(id)` |
| `app_id` | `uuid` NOT NULL | → `apps(id)` |
| `price_paid_thb` | `integer` NOT NULL | CHECK `>= 0` — เก็บราคา ณ เวลาซื้อ |
| `created_at` | `timestamptz` NOT NULL | default `now()` |

> **ห้ามมีคอลัมน์ `auto_renew` หรือ `next_billing_date`** เพราะขัดกับแนวคิดของระบบ
> `price_paid_thb` ต้องเก็บแยกจาก `apps.price_thb` เพราะราคาแอปอาจเปลี่ยนภายหลัง แต่ยอดในใบเสร็จต้องไม่เปลี่ยน

**Unique constraint** — ผู้ใช้ 1 คนซื้อแอปเดิมซ้ำไม่ได้

```sql
create unique index orders_user_app_idx on orders (user_id, app_id);
```

> เดิมเป็น partial index `where status = 'PAID'` เมื่อไม่มีคอลัมน์ `status` แล้ว
> ทุกแถวคือการซื้อที่สำเร็จ จึงเป็น unique index ธรรมดาได้เลย

### `licenses`

| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | `uuid` PK | |
| `order_id` | `uuid` UNIQUE NOT NULL | → `orders(id)` |
| `user_id` | `uuid` NOT NULL | → `profiles(id)` |
| `app_id` | `uuid` NOT NULL | → `apps(id)` |
| `license_key` | `text` UNIQUE NOT NULL | CHECK รูปแบบ `OTS-XXXX-XXXX` |
| `issued_at` | `timestamptz` NOT NULL | default `now()` |

> **ไม่มีคอลัมน์ `expires_at` โดยเจตนา** License เป็น perpetual ตลอดชีพ

```sql
alter table licenses add constraint licenses_key_format
  check (license_key ~ '^OTS-[A-Z0-9]{4}-[A-Z0-9]{4}$');
```

---

## 3.1 Function ออก License — `purchase_app()`

**มติ Phase 0** — เดิมไม่มีเอกสารไหนระบุชื่อหรือ signature ของ function นี้ ทั้งที่คนที่ 2 เป็นคนเขียน
(migration `0005`) แต่คนที่ 4 เป็นคนเรียกใน Phase 8 ล็อกสัญญาไว้ตรงนี้

```sql
create function public.purchase_app(p_app_id uuid)
returns text                      -- คืน license_key ที่เพิ่งออก
language plpgsql
security definer
set search_path = public
as $$ ... $$;
```

**หน้าที่ของ function — ทำครบใน transaction เดียว**

1. ตรวจว่ามี session (`auth.uid()` ไม่เป็น NULL) ถ้าไม่มี → `raise exception`
2. ตรวจว่าแอปมีอยู่จริงและ `is_published = true` ถ้าไม่ → `raise exception`
3. `insert into orders (user_id, app_id, price_paid_thb)` โดยอ่านราคา ณ ตอนนั้นจาก `apps.price_thb`
   ถ้าชน unique index (ซื้อซ้ำ) → `raise exception`
4. สร้าง license key รูปแบบ `OTS-XXXX-XXXX` แล้ว `insert into licenses`
5. `return` license key

**ทำไมต้องรวมเป็น function เดียว** ถ้าแยกเป็นสอง statement จากฝั่งแอป แล้ว insert แรกผ่าน insert ที่สองพัง
จะได้ order ที่จ่ายแล้วแต่ไม่มีคีย์ ซึ่งแก้ตามหลังไม่ได้เพราะ `orders` ไม่มี UPDATE/DELETE policy

**ฝั่งแอปเรียกแบบนี้**

```ts
const { data: licenseKey, error } = await supabase.rpc("purchase_app", { p_app_id: appId })
```

> `MockPaymentForm` ที่เลือก "ล้มเหลว" ต้อง **ไม่เรียก** function นี้เลย ไม่ใช่เรียกแล้วค่อย rollback

---

## 4. Row Level Security

**เปิด RLS ทุกตาราง ไม่มีข้อยกเว้น**

### `profiles`

| Policy | คำอธิบาย |
|---|---|
| SELECT own | `auth.uid() = id` |
| UPDATE own | `auth.uid() = id` และห้ามแก้ `role` ตัวเอง |
| SELECT admin | ผู้ใช้ที่ `role = 'ADMIN'` อ่านได้ทุกแถว |

### `apps`

| Policy | คำอธิบาย |
|---|---|
| SELECT published | ทุกคนรวม anonymous อ่านได้เมื่อ `is_published = true` |
| ALL admin | ADMIN เท่านั้นที่เขียนได้ |

### `orders`

| Policy | คำอธิบาย |
|---|---|
| SELECT own | `auth.uid() = user_id` |

**มติ Phase 0 — ไม่มี INSERT policy สำหรับผู้ใช้ทั่วไปแล้ว** order ถูกสร้างโดย `purchase_app()`
(security definer) เท่านั้น เหมือนกับ `licenses` เหตุผลคือทั้งสองตารางต้องเขียนพร้อมกันใน transaction
เดียว ถ้าเปิด INSERT ตรงไว้ ผู้ใช้จะสร้าง order เปล่าที่ไม่มี license ได้

**ห้ามมี UPDATE/DELETE policy** — order เป็นบันทึกถาวร

### `licenses`

| Policy | คำอธิบาย |
|---|---|
| SELECT own | `auth.uid() = user_id` |

**ห้ามมี INSERT/UPDATE/DELETE policy สำหรับผู้ใช้ทั่วไป** — license ออกโดย trigger หรือ security definer function เท่านั้น เพื่อกันคนสร้างคีย์เอง

---

## 5. ลำดับ Migration

| ไฟล์ | เนื้อหา |
|---|---|
| `0001_init_schema.sql` | enum + ตาราง + constraint + index |
| `0002_profile_on_signup.sql` | trigger สร้าง profile อัตโนมัติ |
| `0003_rls_public.sql` | เปิด RLS + policy ของ `profiles` และ `apps` |
| `0004_rls_orders_licenses.sql` | policy ของ `orders` และ `licenses` |
| `0005_purchase_app_fn.sql` | security definer function `purchase_app()` (ดูข้อ 3.1) |
| `0006_seed_apps.sql` | ข้อมูลตัวอย่างอย่างน้อย 12 แอป |

**ห้ามแก้ migration ที่ push ขึ้น `dev` แล้ว** ถ้าต้องเปลี่ยน ให้สร้างไฟล์ใหม่ต่อท้าย

---

## 6. ข้อมูล Seed

ต้องมีอย่างน้อย **12 แอป** กระจายครบทั้ง 5 หมวดหมู่ และต้องมีเคสเหล่านี้เพื่อให้ทดสอบ filter ได้จริง

- อย่างน้อย 1 แอปต่อหมวดหมู่
- อย่างน้อย 1 แอปในแต่ละช่วงราคา: `0–500`, `501–1,500`, `1,501+`
- อย่างน้อย 1 แอปของแต่ละโมเดลราคาทั้ง 3 แบบ
- อย่างน้อย 1 แอปที่ได้ **100 คะแนนเต็ม (Exemplary/ทอง)** — ต้องครบทั้ง
  `pricing_model = 'ONE_TIME'`, ป้าย lifetime/offline/privacy ครบ 3 และ `support_months >= 12`
- อย่างน้อย 1 แอปที่ได้ **0 คะแนน (Neutral/เทา)** — ไม่ผ่านสักเกณฑ์ รวมถึง `support_months < 12`
- อย่างน้อย 1 แอปที่ได้ป้ายครบแต่ `support_months < 12` → ได้ 80 คะแนน (Sustainable/เขียว)
  ไว้พิสูจน์ว่าเกณฑ์ที่ 5 มีผลจริง
- อย่างน้อย 1 แอปที่ `competitor_monthly_thb IS NULL` เพื่อทดสอบว่า Calculator ซ่อนตัวเองได้ถูกต้อง
- อย่างน้อย 1 แอปราคา **1,500** และ 1 แอปราคา **1,501** เพื่อพิสูจน์เส้นแบ่งช่วงราคาบนสุด
- 1 แอปที่ตรงกับตัวเลขในสไลด์: `price_thb = 1200`, `competitor_monthly_thb = 350`

> เดิมข้อนี้เขียนว่า "ป้ายครบ 4 ชนิด → 100 คะแนน → Gold" ซึ่งขัดกับสูตรใน `ARCHITECTURE.md` ข้อ 6
> ที่มี 5 เกณฑ์ เกณฑ์ละ 20 — ป้ายครบแต่ `support_months < 12` ได้แค่ 80 แก้แล้วตามด้านบน

---

## 7. Test Database

- Integration test ห้ามแตะฐานข้อมูลของ production
- ใช้ Supabase local (`supabase start`) หรือฐานข้อมูลทดสอบแยก
- ล้างข้อมูลทดสอบก่อนและหลังแต่ละชุดเทส
- ห้าม commit `SUPABASE_SERVICE_ROLE_KEY` ลง repo

RLS test ที่ต้องมีอย่างน้อย

1. ผู้ใช้ A อ่าน `licenses` ของผู้ใช้ B ไม่ได้
2. ผู้ใช้ A อ่าน `orders` ของผู้ใช้ B ไม่ได้
3. Anonymous อ่าน `apps` ที่ `is_published = false` ไม่ได้
4. ผู้ใช้ทั่วไป `INSERT` เข้า `licenses` โดยตรงไม่ได้
5. ผู้ใช้ทั่วไป `INSERT` เข้า `orders` โดยตรงไม่ได้ (ต้องผ่าน `purchase_app()` เท่านั้น)
6. ผู้ใช้ทั่วไปเปลี่ยน `role` ของตัวเองเป็น `ADMIN` ไม่ได้

---

## 8. Generated Types

```bash
npx supabase gen types typescript --local > src/lib/supabase/database.types.ts
```

ไฟล์นี้เป็น shared file — คนที่ 2 เป็นคน regenerate และแจ้งทีมทุกครั้งที่ schema เปลี่ยน
