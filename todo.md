# Todo List Web App — แผนแบ่งงาน

โปรเจกต์: Todo List (HTML / CSS / JavaScript)
รูปแบบการทำงาน: แยก branch คนละ task แล้ว merge เข้า `main`

---

## สรุปการแบ่งงาน

| ผู้รับผิดชอบ | Task | ขอบเขต | ไฟล์ที่เป็นเจ้าของ | Branch |
|---|---|---|---|---|
| คนที่ 1 | Task 1 | โครงหน้า UI / Layout | `index.html`, `css/style.css`, `js/render.js` | `feature/task1-layout` |
| คนที่ 2 | Task 2 | ฟังก์ชัน Add / Delete | `js/todo.js` | `feature/task2-add-delete` |
| คนที่ 3 | Task 3 | Mark Done + Local Storage | `js/storage.js`, `js/done.js` | `feature/task3-done-storage` |

> **กติกาสำคัญ:** แต่ละคนแก้เฉพาะไฟล์ของตัวเองเท่านั้น ถ้าจำเป็นต้องแก้ไฟล์ของคนอื่น ให้แจ้งในกลุ่มก่อน เพื่อลด merge conflict

---

## สัญญาร่วม (Interface Contract) — ตกลงกันให้จบก่อนเริ่มเขียนโค้ด

ทั้งสามงานต้องอ้างอิงข้อตกลงชุดนี้ ถ้าไม่ตรงกันโค้ดจะประกอบกันไม่ได้

### 1. โครงสร้างข้อมูลของ 1 รายการ

```js
{
  id: "1730812345678",   // string, ใช้ String(Date.now())
  text: "อ่านหนังสือสอบ",  // string
  done: false,            // boolean
  createdAt: 1730812345678 // number (timestamp)
}
```

### 2. ตัวแปรกลาง (อยู่ใน `js/state.js` — คนที่ 1 สร้าง แล้วห้ามใครแก้)

```js
let todos = [];  // array ของ object ตามโครงสร้างข้างบน
```

### 3. ชื่อฟังก์ชันที่ต้องมี (ห้ามเปลี่ยนชื่อโดยพลการ)

| ฟังก์ชัน | ผู้เขียน | หน้าที่ |
|---|---|---|
| `renderTodos()` | คนที่ 1 | วาดรายการทั้งหมดจาก `todos` ลงหน้าจอ |
| `addTodo(text)` | คนที่ 2 | เพิ่มรายการใหม่เข้า `todos` |
| `deleteTodo(id)` | คนที่ 2 | ลบรายการตาม id |
| `toggleDone(id)` | คนที่ 3 | สลับสถานะ `done` |
| `saveTodos()` | คนที่ 3 | บันทึก `todos` ลง localStorage |
| `loadTodos()` | คนที่ 3 | โหลด `todos` จาก localStorage |

ทุกฟังก์ชันที่แก้ไขข้อมูล (`addTodo`, `deleteTodo`, `toggleDone`) ต้องเรียก `saveTodos()` แล้วตามด้วย `renderTodos()` เสมอ

### 4. id ของ element ใน HTML (คนที่ 1 กำหนด ห้ามเปลี่ยนหลังตกลงแล้ว)

```
#todo-input      ช่องพิมพ์ข้อความ
#add-btn         ปุ่มเพิ่ม
#todo-list       <ul> ที่ใช้แสดงรายการ
#empty-state     ข้อความตอนยังไม่มีรายการ
#counter         ตัวนับ "เสร็จแล้ว x จาก y"
```

### 5. โครงสร้าง 1 แถวที่ `renderTodos()` ต้องวาดออกมา

```html
<li class="todo-item" data-id="1730812345678">
  <input type="checkbox" class="todo-check" data-action="toggle">
  <span class="todo-text">อ่านหนังสือสอบ</span>
  <button class="todo-delete" data-action="delete">ลบ</button>
</li>
```

ใช้ `data-id` และ `data-action` เป็นตัวเชื่อม เพื่อให้คนที่ 2 และ 3 ผูก event ได้โดยไม่ต้องแก้ HTML

---

## คนที่ 1 — Task 1: โครงหน้า UI / Layout

**เป้าหมาย:** ทำให้หน้าเว็บมีโครงสร้างและหน้าตาครบ พร้อมให้คนที่ 2 และ 3 มาเสียบตรรกะได้ทันที

- [ ] สร้าง `index.html` — โครงหน้า header / ช่องกรอก / ปุ่มเพิ่ม / `<ul id="todo-list">` / footer
- [ ] ตั้ง `<meta charset="utf-8">` และ `<html lang="th">`
- [ ] ผูก `<script>` ทุกไฟล์ตามลำดับ: `state.js` → `storage.js` → `todo.js` → `done.js` → `render.js` → `main.js`
- [ ] สร้าง `css/style.css` — จัด layout, สี, ระยะห่าง, สถานะ hover / focus
- [ ] ทำ style สำหรับรายการที่เสร็จแล้ว (คลาส `.done` → ขีดฆ่า + สีจาง)
- [ ] เขียน `renderTodos()` ใน `js/render.js` ให้วาดรายการตามโครงสร้างในสัญญาร่วมข้อ 5
- [ ] แสดง `#empty-state` เมื่อ `todos.length === 0` และซ่อนเมื่อมีรายการ
- [ ] อัปเดต `#counter` ทุกครั้งที่ render
- [ ] ทำ Responsive ให้ใช้งานได้บนจอมือถือ (ประมาณ 375px)
- [ ] สร้าง `js/state.js` และ `js/main.js` (โครงเปล่า) แล้วแจ้งทีมว่า contract ล็อกแล้ว

**เกณฑ์ว่างานเสร็จ:** เปิดหน้าเว็บแล้วเห็น layout ครบ และถ้าใส่ข้อมูลตัวอย่างลงตัวแปร `todos` ด้วยมือ แล้วเรียก `renderTodos()` ใน console ต้องแสดงรายการออกมาถูกต้อง

---

## คนที่ 2 — Task 2: ฟังก์ชัน Add / Delete

**เป้าหมาย:** เพิ่มและลบรายการได้จริง

- [ ] เขียน `addTodo(text)` ใน `js/todo.js` — สร้าง object ตามโครงสร้างในสัญญาร่วม แล้ว `push` เข้า `todos`
- [ ] ตรวจ input ว่าง — ถ้า `text.trim() === ""` ให้ไม่เพิ่ม และแจ้งเตือนผู้ใช้
- [ ] ตัดช่องว่างหน้า-หลังด้วย `.trim()` ก่อนบันทึก
- [ ] เขียน `deleteTodo(id)` — ใช้ `todos.filter()` กรอง id ที่ต้องการลบออก
- [ ] ผูก event ปุ่ม `#add-btn` (click) และช่อง `#todo-input` (กด Enter)
- [ ] ล้างช่อง input และโฟกัสกลับที่ช่องเดิมหลังเพิ่มสำเร็จ
- [ ] ผูก event ปุ่มลบด้วย **event delegation** บน `#todo-list` โดยเช็ก `e.target.dataset.action === "delete"` แล้วอ่าน id จาก `closest(".todo-item").dataset.id`
- [ ] ใส่กล่องยืนยันก่อนลบ (`confirm()`)
- [ ] เรียก `saveTodos()` และ `renderTodos()` ท้ายทุกฟังก์ชัน

**เกณฑ์ว่างานเสร็จ:** เพิ่มรายการได้ ลบได้ ใส่ค่าว่างแล้วไม่พัง และกด Enter ใช้งานได้เหมือนกดปุ่ม

> **หมายเหตุ:** ถ้าคนที่ 3 ยังทำ `saveTodos()` ไม่เสร็จ ให้ประกาศฟังก์ชันเปล่าไว้ชั่วคราวใน branch ตัวเอง แล้วลบทิ้งตอน merge

---

## คนที่ 3 — Task 3: Mark Done + Local Storage

**เป้าหมาย:** ติ๊กเสร็จได้ และข้อมูลไม่หายเมื่อรีเฟรชหน้า

### ส่วน Mark Done (`js/done.js`)

- [ ] เขียน `toggleDone(id)` — หา item จาก id แล้วสลับค่า `done`
- [ ] ผูก event checkbox ด้วย event delegation บน `#todo-list` (เช็ก `data-action === "toggle"`)
- [ ] ประสานกับคนที่ 1 ให้ `renderTodos()` ใส่คลาส `done` และตั้ง `checked` ให้ตรงกับค่าจริง
- [ ] เรียก `saveTodos()` และ `renderTodos()` หลังสลับสถานะ

### ส่วน Local Storage (`js/storage.js`)

- [ ] กำหนดค่าคงที่ `const STORAGE_KEY = "todo-app-data";`
- [ ] เขียน `saveTodos()` — `localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))`
- [ ] เขียน `loadTodos()` — อ่านค่ากลับมา `JSON.parse` แล้วใส่ลง `todos`
- [ ] ครอบ `JSON.parse` ด้วย `try...catch` เผื่อข้อมูลใน storage เสีย ถ้าพังให้ใช้ array ว่าง
- [ ] จัดการกรณีเปิดครั้งแรก (ค่าที่อ่านได้เป็น `null`) ให้เป็น `[]`
- [ ] เรียก `loadTodos()` แล้วตามด้วย `renderTodos()` ตอนหน้าเว็บโหลดเสร็จ (`DOMContentLoaded` ใน `main.js`)
- [ ] เพิ่มปุ่มล้างข้อมูลทั้งหมด (ถ้ามีเวลาเหลือ)

**เกณฑ์ว่างานเสร็จ:** ติ๊กเสร็จแล้วรายการเปลี่ยนหน้าตา รีเฟรชหน้าเว็บแล้วทั้งรายการและสถานะ done ยังอยู่ครบ

---

## ขั้นตอน Git

```bash
# ครั้งแรก
git clone <repo-url>
cd <repo>

# แยก branch ของตัวเอง (แก้ชื่อตาม task ของแต่ละคน)
git checkout main
git pull origin main
git checkout -b feature/task1-layout

# ระหว่างทำงาน commit ย่อย ๆ
git add .
git commit -m "feat: เพิ่มโครง layout หน้าหลัก"
git push origin feature/task1-layout
```

**ลำดับการ merge:** Task 1 → Task 2 → Task 3
เพราะ Task 2 และ 3 ต้องใช้ HTML id กับ `renderTodos()` จาก Task 1

ก่อน push ทุกครั้ง ให้ดึงงานล่าสุดมาก่อน:

```bash
git checkout main
git pull origin main
git checkout feature/task2-add-delete
git merge main       # แก้ conflict ตรงนี้ในเครื่องตัวเอง
```

**รูปแบบ commit message:**
`feat:` เพิ่มฟีเจอร์ · `fix:` แก้บั๊ก · `style:` ปรับ CSS · `docs:` แก้เอกสาร

---

## Checklist ก่อนส่งงาน

- [ ] ทั้ง 3 branch merge เข้า `main` เรียบร้อย ไม่เหลือ conflict
- [ ] เพิ่ม / ลบ / ติ๊กเสร็จ ทำงานได้ครบทั้งสามอย่าง
- [ ] รีเฟรชหน้าเว็บแล้วข้อมูลยังอยู่
- [ ] ไม่มี error สีแดงใน Console
- [ ] ทดสอบกรณีขอบ: เพิ่มค่าว่าง, ลบรายการสุดท้าย, เปิดครั้งแรกที่ยังไม่มีข้อมูล
- [ ] ลบ `console.log` ที่ใช้ debug ออก
- [ ] เขียน `README.md` — วิธีรัน + ตารางแบ่งงานของสมาชิก
- [ ] ตรวจว่าทุกคนมี commit ปรากฏใน history ของ repo

---

## หมายเหตุสำหรับทีม

กลุ่มมีสมาชิก 4 คน แต่แบ่งเป็น 3 task ถ้าจะให้ครบ แนะนำให้คนที่ 4 รับบทเป็น **ผู้รวมงานและทดสอบ** คือดูแล `js/main.js` ที่เป็นจุดรวม event ทั้งหมด, รีวิว Pull Request, ทดสอบตาม checklist ด้านบน และเขียน `README.md`