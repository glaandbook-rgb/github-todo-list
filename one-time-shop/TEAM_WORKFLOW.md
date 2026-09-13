# TEAM_WORKFLOW.md

---

## 1. การแบ่งงาน

| คน | ขอบเขต | Branch ที่รับผิดชอบ |
|---|---|---|
| คนที่ 1 | Foundation, Design System, Authentication | `feature1.1`, `feature1.2` |
| คนที่ 2 | Database, RLS, Catalog, Badge, Sustainable Score | `feature2.1`, `feature2.2` |
| คนที่ 3 | Filter, Search, หน้ารายละเอียดแอป | `feature3.1`, `feature3.2` |
| คนที่ 4 | Calculator, Checkout, License, My Library, UI/UX | `feature4.1`, `feature4.2`, `feature4.3` |
| ทุกคน | Integration, E2E, Final Audit | ทำบน `dev` |

**หลักการตั้งชื่อ branch:** `feature<คนที่>.<ฟีเจอร์ลำดับที่ของคนนั้น>`
ตัวอย่าง `feature3.2` = คนที่ 3 ฟีเจอร์ที่ 2

---

## 2. Git Strategy

```
main
 └── dev
      ├── feature1.1  feature1.2
      ├── feature2.1  feature2.2
      ├── feature3.1  feature3.2
      └── feature4.1  feature4.2  feature4.3
```

**กฎเหล็ก**
- ห้าม push ตรงเข้า `main` หรือ `dev`
- feature branch ต้องแตกจาก `dev` เสมอ ห้ามแตกจาก `main`
- merge เข้า `dev` ผ่าน Pull Request เท่านั้น
- `dev` → `main` merge ครั้งเดียวตอนจบ Phase 11
- **ห้ามลบ branch หลัง merge** — GitHub มีปุ่มลบอัตโนมัติหลัง merge PR อย่ากด เพราะคนตรวจต้องเห็น branch ครบ

---

## 3. ขั้นตอนเริ่มต้น (ทำครั้งเดียว)

### คนหลักทำคนเดียวก่อน คนอื่นรอ

```bash
git init
git add .
git commit -m "chore: initial project structure"
git branch -M main
git remote add origin <repo-url>
git push -u origin main

git checkout -b dev
git push -u origin dev
```

จากนั้นใน GitHub
1. Settings → Collaborators → เชิญเพื่อนทั้ง 3 คน
2. Settings → General → Default branch → เปลี่ยนเป็น `dev`
3. Settings → Branches → เพิ่ม protection rule ให้ `main` และ `dev` ห้าม push ตรง
4. แจ้งในกลุ่มว่า `dev` พร้อมแล้ว

> **ห้ามให้เพื่อน clone ก่อนขั้นตอนนี้เสร็จ** ไม่งั้นจะไม่เห็น branch `dev` แล้วงานพังตั้งแต่ต้น

### เพื่อนทำหลังได้รับแจ้ง

```bash
git clone <repo-url>
cd <repo>
git checkout dev
```

---

## 4. ขั้นตอนประจำวัน (ทำทุกรอบฟีเจอร์)

```bash
# 1. ดึงงานล่าสุดก่อนเสมอ
git checkout dev
git pull origin dev

# 2. แตก branch ของฟีเจอร์นี้
git checkout -b feature3.1

# 3. ให้ AI วาง plan ก่อน แล้วอ่านและแก้ให้เห็นด้วยก่อนลงมือ

# 4. เขียนโค้ดหลัก

# 5. เขียน unit test (หลังโค้ดหลักเสร็จ)

# 6. ตรวจก่อน commit
npm run lint
npm run typecheck
npm run test
git status
git diff

# 7. Manual test ด้วยตัวเอง — เปิดเว็บกดใช้จริง

# 8. commit และ push
git add .
git commit -m "feat: เพิ่มระบบกรองและค้นหาแอป"
git push -u origin feature3.1

# 9. เปิด PR: feature3.1 → dev
```

### ดึงงานใหม่จาก dev ระหว่างทำ

```bash
git checkout dev
git pull origin dev
git checkout feature3.1
git merge dev          # แก้ conflict ในเครื่องตัวเอง ไม่ใช่ใน PR
```

---

## 5. รูปแบบ Commit Message

```
<type>: <คำอธิบายสั้น ๆ ที่อ่านแล้วรู้ว่าทำอะไร>
```

| type | ใช้เมื่อ |
|---|---|
| `feat` | เพิ่มฟีเจอร์ |
| `fix` | แก้บั๊ก |
| `test` | เพิ่มหรือแก้เทส |
| `style` | ปรับ CSS / format ไม่เปลี่ยนพฤติกรรม |
| `refactor` | ปรับโครงโค้ด ไม่เปลี่ยนพฤติกรรม |
| `docs` | แก้เอกสาร |
| `chore` | งานทั่วไป เช่น config, dependency |

**ดี**
```
feat: เพิ่มเครื่องคำนวณจุดคุ้มทุนเทียบ subscription
test: เพิ่มเคสขอบเมื่อค่าบริการรายเดือนเป็นศูนย์
fix: แก้ตัวกรองช่วงราคาที่นับ 500 ซ้ำสองช่วง
```

**ห้ามใช้**
```
1
update
student 2
แก้งาน
```

---

## 6. Pull Request Template

ทุก PR ต้องมีครบ 9 หัวข้อนี้ — นี่คือหลักฐานสำหรับโจทย์ข้อ 2, 3, 4

```markdown
## 1. ทำอะไรไปบ้าง

## 2. ไฟล์ที่เปลี่ยน

## 3. Plan ที่ AI ให้ก่อนเริ่ม
<!-- วาง plan ที่ได้รับ + ระบุว่าแก้อะไรจาก plan เดิมบ้าง -->

## 4. เทสที่เพิ่ม
<!-- ชื่อไฟล์ + จำนวนเคส -->

## 5. เทสที่รันและผลลัพธ์
```
$ npm run test
...
```

## 6. Manual Test ที่ทำเอง
<!-- ทดสอบอะไร กดอะไร ผลเป็นยังไง -->
- [ ] ...
- [ ] ...

## 7. การเปลี่ยนแปลงฐานข้อมูล
<!-- ถ้าไม่มี เขียนว่า "ไม่มี" -->

## 8. ปัญหาที่ยังค้างอยู่

## 9. สิ่งที่ตั้งใจไม่ทำเพราะอยู่นอกสโคป
```

ถ้าแก้ UI ให้แนบ screenshot ด้วย

---

## 7. ความเป็นเจ้าของเทส

**คนที่เขียนฟีเจอร์เป็นเจ้าของเทสของฟีเจอร์นั้น**

ห้ามสร้างสถานการณ์แบบ
> "คนที่ 3 เขียนระบบกรองไว้ แล้วคนที่ 4 ต้องมานั่งเดาทีหลังว่าต้องเทสอะไรบ้าง"

เทสต้อง commit มาพร้อมฟีเจอร์ในทุกกรณีที่ทำได้

---

## 8. ไฟล์ที่ใช้ร่วมกัน

ประสานงานก่อนแก้ไฟล์เหล่านี้

- `package.json` และ lockfile
- `src/app/layout.tsx`
- `src/app/globals.css`
- `src/components/ui/*`
- `src/lib/supabase/database.types.ts`
- `.env.example`
- `vitest.config.ts`, `playwright.config.ts`

ถ้าสอง branch ต้องแก้ไฟล์เดียวกัน
1. คุยกันก่อน
2. แก้ให้น้อยที่สุด
3. ให้เจ้าของไฟล์เป็นคนตัดสินตอน resolve conflict

---

## 9. ความเป็นเจ้าของฐานข้อมูล

**คนที่ 2 เป็นเจ้าของ schema, migration, RLS, policy และ generated types แต่ผู้เดียว**

คนอื่นห้ามแก้ไฟล์ใน `supabase/migrations/` เอง

> **มติ Phase 0 — `src/lib/supabase/` แบ่งเจ้าของกัน**
> คนที่ 1 ถือ `client.ts` / `server.ts` / `env.ts` (ผูกกับ session กับ `proxy.ts`)
> คนที่ 2 ถือ `database.types.ts` และ `supabase/migrations/`

ถ้าต้องการเปลี่ยน schema

```
คนที่ 4: STOP → แจ้งคนที่ 2
         "ระบบ License ต้องการคอลัมน์ X เพราะ ..."

คนที่ 2: ทบทวนความจำเป็น
       → สร้าง migration ใหม่ (ไม่แก้ของเดิมที่ push แล้ว)
       → อัปเดต RLS policy
       → regenerate database.types.ts
       → แจ้งทีม

คนที่ 4: pull dev แล้วทำงานต่อ
```

---

## 10. ความเป็นเจ้าของ Conflict

| ส่วนที่ conflict | ใครตัดสิน |
|---|---|
| Database / RLS / migration / `database.types.ts` | คนที่ 2 |
| Auth / session / `proxy.ts` / supabase client-server-env | คนที่ 1 |
| Filter / Search / Detail | คนที่ 3 |
| Calculator / Checkout / License / UI | คนที่ 4 |
| ไฟล์ shared | คุยกันทั้งทีม |

---

## 11. ความปลอดภัยของ Test Environment

ห้าม
- ทดสอบกับข้อมูลจริงของ production
- commit credential จริง
- เปิดเผย service-role key
- ปิด authorization หรือ RLS เพื่อให้เทสผ่าน

ให้ใช้
- Supabase local (`supabase start`) หรือฐานข้อมูลทดสอบแยก
- test fixture และ test user
- environment variable ที่มีเอกสารกำกับใน `.env.example`

---

## 12. จังหวะการรวมงาน

อย่ารอถึงวันสุดท้ายค่อยมาเจอปัญหาการรวมโค้ด

- ทุก PR ต้องมี unit test มาด้วย
- integration test สำคัญ ๆ ให้เพิ่มพร้อมฟีเจอร์ ไม่ใช่ทิ้งไว้ทีหลัง
- หลัง branch คู่ขนาน merge เสร็จ ให้รัน integration/E2E เต็มชุดทันที

---

## 13. หลักฐานทางวิชาการ

เก็บสิ่งเหล่านี้ไว้เป็นหลักฐาน

- branch ทั้งหมด (ห้ามลบ)
- commit history
- Pull Request พร้อมเนื้อหาครบ 9 หัวข้อ
- ผลการรันเทส
- screenshot ของหน้าจอ
- Phase Completion Report

**ห้าม rewrite Git history เพื่อปลอมแปลงการมีส่วนร่วม**

ผลงานของแต่ละคนต้องมองเห็นได้และอธิบายได้ว่าทำอะไร ทำไม และทดสอบอย่างไร

---

## 14. Checklist ก่อนส่งงาน

- [ ] ทุก feature branch merge เข้า `dev` แล้ว
- [ ] `dev` merge เข้า `main` แล้ว
- [ ] เปิดหน้า Branches ใน GitHub เห็นครบ: `main`, `dev`, `feature1.1`, `feature1.2`, `feature2.1`, `feature2.2`, `feature3.1`, `feature3.2`, `feature4.1`, `feature4.2`, `feature4.3`
- [ ] repo เป็น Public หรือเชิญอาจารย์เข้าถึงแล้ว
- [ ] ทุกคนมี commit ปรากฏใน history
- [ ] `README.md` มีวิธีรันและตารางแบ่งงาน
- [ ] ส่งลิงก์ repo **อันเดียว** ใน Discord (ไม่ใช่ลิงก์แยกรายคน)
