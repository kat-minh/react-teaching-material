# LESSON PLAN: SESSION 0 – REACT FOUNDATION & DEVELOPER MINDSET

## 1️⃣ SESSION OVERVIEW

- **Title:** Why React Exists – Mental Models Before Code
- **Duration:** 90 minutes (STRICT)
- **Goal:** Students understand **why** React exists and build mental models for Component, State, and JSX
- **Outcome:** Students can explain React's purpose, draw State→UI cycle, and are not intimidated by React
- **Stack:** Vite + React 18 + **TypeScript** (mention ngắn, chi tiết học Session 1)

---

## 2️⃣ INSTRUCTOR OPENING SCRIPT

> _"Chào các bạn. Welcome to ReactJS Bootcamp._
>
> _Hôm nay là Session 0 – buổi đặc biệt nhất trong khóa học._
>
> _Buổi này ta **KHÔNG** viết nhiều code. Ta **KHÔNG** học API. Ta **KHÔNG** làm project._
>
> _Ta chỉ làm 1 việc duy nhất: **Hiểu TẠI SAO React tồn tại**._
>
> _Nhiều người học React bằng cách copy-paste code từ tutorial._ > _3 tháng sau vẫn không hiểu `useState` làm gì, `useEffect` để đâu._ > _Họ học React như học **công thức toán** – thuộc lòng nhưng không biết DÙNG KHI NÀO._
>
> _Hôm nay ta xây **bản đồ tư duy** – mental model._ > _17 buổi sau, khi gặp bug, các bạn sẽ biết mình đang ở đâu trên bản đồ."_
>
> _Ai đã học framework khác – Angular, Vue, jQuery – quên hết đi trong 2 tiếng này._ > _React TƯ DUY KHÁC._
>
> _Nguyên tắc: Hiểu **TẠI SAO** quan trọng hơn **LÀM THẾ NÀO**._

---

### MỤC TIÊU BUỔI HỌC (90 PHÚT - STRICT)

> _"Sau buổi học hôm nay:_
>
> _1. Hiểu TẠI SAO React tồn tại_ > _2. Vẽ được sơ đồ State → Render → UI_ > _3. Biết Component = Function_ > _4. Hiểu JSX ≠ HTML (cơ bản)_ > _5. Không còn SỢ React_

**⏱️ Time Allocation:**

- Why React (Vanilla vs React): 20 phút
- State → UI Cycle (CORE): 25 phút
- Component = Function: 10 phút
- JSX Basics (ngắn gọn): 15 phút
- Vite Demo (chạy được thôi): 10 phút
- Q&A + Mini Task: 10 phút

---

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### PHẦN A: Vấn Đề của Vanilla JS (10 phút)

> _"Trước khi nói về React, ta phải hiểu VẤN ĐỀ mà React giải quyết._
>
> _Trước React, ta viết UI bằng Vanilla JavaScript._ > _Nghe đơn giản – chỉ cần `document.getElementById` rồi `.innerText` thôi mà._
>
> _Hãy xem ví dụ này. Đây là app THẬT nếu không có React."_

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Counter App - Vanilla JS</h1>

    <!-- UI elements -->
    <button id="btn">Count: 0</button>
    <p id="status">Idle</p>
    <p id="message">Click the button!</p>

    <script>
      // State (data) - chỉ có 1 biến
      let count = 0;

      document.getElementById("btn").addEventListener("click", () => {
        // Cập nhật state
        count++;

        // ❌ VẤN ĐỀ: Phải update UI THỦ CÔNG ở 3 chỗ!

        // Chỗ 1: Cập nhật button text
        document.getElementById("btn").innerText = `Count: ${count}`;

        // Chỗ 2: Cập nhật status
        if (count > 5) {
          document.getElementById("status").innerText = "Too many!";
          document.getElementById("status").style.color = "red";
        } else if (count > 3) {
          document.getElementById("status").innerText = "Getting high...";
          document.getElementById("status").style.color = "orange";
        } else {
          document.getElementById("status").innerText = "Normal";
          document.getElementById("status").style.color = "green";
        }

        // Chỗ 3: Cập nhật message
        if (count === 1) {
          document.getElementById("message").innerText = "Good start!";
        } else if (count === 10) {
          document.getElementById("message").innerText = "You reached 10!";
        }
      });
    </script>
  </body>
</html>
```

> _"Ai thấy mệt chưa? Ta chỉ có **1 state variable** (`count`), nhưng phải update **3 chỗ UI**._
>
> _Vấn đề của Vanilla JS:_ > _- State thay đổi ở đây..._ > _- ...nhưng UI không tự động update_ > _- Ta phải NHỚ update tất cả các chỗ hiển thị state đó_ > _- App càng lớn → state càng nhiều → UI càng nhiều → QUÊN MẤT → BUG_
>
> _Đây là kiểu lập trình **IMPERATIVE** – ta nói CHI TIẾT phải làm gì từng bước._
>
> _Giờ xem React làm thế nào."_

---

### PHẦN B: Giải Pháp của React (10 phút)

> _"Okay, giờ xem React giải quyết vấn đề này._
>
> _Ta sẽ viết lại app vừa rồi, nhưng bằng React._ > _Chưa cần hiểu 100% syntax – chỉ cần so sánh với code Vanilla JS."_

```jsx
import { useState } from "react";

function Counter() {
  // State - vẫn chỉ có 1 biến, giống Vanilla JS
  // useState(0) = giá trị khởi tạo là 0
  // count = biến chứa giá trị
  // setCount = hàm để thay đổi giá trị
  const [count, setCount] = useState(0);

  // Logic tính toán (KHÔNG phải update DOM!)
  // Chỉ tính toán giá trị, không gọi document.getElementById
  const status =
    count > 5 ? "Too many!" : count > 3 ? "Getting high..." : "Normal";

  const statusColor = count > 5 ? "red" : count > 3 ? "orange" : "green";

  const message =
    count === 1
      ? "Good start!"
      : count === 10
      ? "You reached 10!"
      : "Click the button!";

  // UI - MÔ TẢ giao diện trông như thế nào
  // Không phải BẢO DOM phải update gì
  return (
    <div>
      <h1>Counter App - React</h1>

      {/* Khi click, gọi setCount để tăng count lên 1 */}
      {/* React tự động re-render và update UI */}
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>

      {/* Hiển thị status với màu tương ứng */}
      <p style={{ color: statusColor }}>{status}</p>

      {/* Hiển thị message */}
      <p>{message}</p>
    </div>
  );
}

export default Counter;
```

> _"Điểm khác biệt:_
>
> _Vanilla JS: 'Khi count thay đổi, HÃY CẬP NHẬT button text, HÃY CẬP NHẬT status...'_
>
> _React: 'UI TRÔNG NHƯ THẾ NÀY khi count là 5. TRÔNG NHƯ THẾ NÀY khi count là 10.'_
>
> _React tự động tính toán xem phải update gì._
>
> _React gọi là **DECLARATIVE** programming._ > _Declarative = Khai báo. Ta khai báo UI TRÔNG NHƯ THẾ NÀO, không CHỈ ĐỊNH phải update gì._
>
> _Tóm lại:_ > _- Vanilla JS: 'Làm cái này, làm cái kia, rồi làm cái kia nữa'_ > _- React: 'UI của tôi trông như thế này khi state thay đổi'_ > _- React lo phần còn lại."_

---

### PHẦN C: Sơ Đồ Tư Duy React (25 phút - CORE CONCEPT!)

> _"⏱️ 25 PHÚT này là QUAN TRỌNG NHẤT buổi học._
>
> _Sơ đồ này là trái tim của React._ > _Nếu các bạn chỉ nhớ được 1 thứ hôm nay, hãy nhớ sơ đồ này._
>
> _⏱️ TIMING: 10 phút giải thích sơ đồ + 10 phút code example + 5 phút Q&A"_

```
        ┌─────────────┐
        │    STATE    │  ← Data (count, name, todos...)
        │  (useState) │
        └──────┬──────┘
               │
               │ State thay đổi (setCount, setName...)
               ▼
        ┌─────────────┐
        │   RENDER    │  ← React gọi component function lại
        │  (Function) │
        └──────┬──────┘
               │
               │ Return JSX
               ▼
        ┌─────────────┐
        │     UI      │  ← Hiển thị trên màn hình
        │    (DOM)    │
        └──────┬──────┘
               │
               │ User tương tác (click, type...)
               ▼
        ┌─────────────┐
        │    EVENT    │  ← onClick, onChange, onSubmit...
        │  (Handler)  │
        └──────┬──────┘
               │
               │ Gọi setState
               │
               └────────────────┐
                               │
                               ▼
                    [Quay lại STATE - vòng lặp]
```

> _"Giải thích từng bước:_
>
> _1. **STATE** - data của app: `count = 5`, `name = "John"`, `todos = [...]`_
>
> _2. Khi state thay đổi (gọi `setCount(6)`), điều gì xảy ra?_
>
> _3. **RENDER** - React GỌI LẠI component function. Component chạy lại toàn bộ logic._
>
> _4. Component return JSX → **UI** - React update DOM để hiển thị UI mới_
>
> _5. User nhìn thấy UI, tương tác: click button, type input, submit form..._
>
> _6. **EVENT** xảy ra → gọi event handler: `onClick={() => setCount(count + 1)}`_
>
> _7. Event handler gọi `setState` → State thay đổi → quay lại bước 1_
>
> \*Đây là vòng lặp của React: **State → Render → UI → Event → State → ...**
>
> _Mọi thứ bắt đầu từ state. State thay đổi → UI thay đổi. Không có state thay đổi → UI không đổi."_

**Sơ đồ rút gọn (nhớ cái này):**

```
   State → Render → UI
     ↑              ↓
     └──── Event ───┘
```

> _"Đây là version rút gọn. Nhớ sơ đồ này là đủ._
>
> _17 buổi sau, mỗi khi thấy code React, hãy nghĩ về sơ đồ này:_ > _- Thấy `useState`? → Đó là State_ > _- Thấy `onClick`? → Đó là Event_ > _- Thấy `return <div>`? → Đó là UI_
>
> _Mọi thứ đều fit vào sơ đồ này."_

---

### PHẦN D: Component = Function (10 phút)

> _"Chuyển sang concept thứ 2: Component._
>
> _Nhiều bạn học React cũ sẽ thấy class component. Quên đi. Ta không dùng class nữa._
>
> \*Định nghĩa: **Component = Function returning JSX\***

```jsx
// Component đơn giản nhất có thể
function Welcome() {
  return <h1>Hello</h1>;
}

// Quy tắc:
// 1. Tên component phải VIẾT HOA chữ cái đầu (Welcome, Counter, UserProfile)
// 2. Phải return JSX (hoặc null nếu không hiển thị gì)
// 3. Đó là hàm JavaScript bình thường – có thể có logic bên trong
```

```jsx
// Component có logic
function Greeting({ name }) {
  // { name } là props – cách truyền data vào component
  // Sẽ học kỹ ở Session 1

  // Logic JavaScript bình thường
  const message = `Hello, ${name}!`;
  const timestamp = new Date().toLocaleTimeString();

  // Return JSX
  return (
    <div>
      <p>{message}</p>
      <small>Logged in at {timestamp}</small>
    </div>
  );
}
```

> _"Chú ý: JSX không phải string, không phải HTML. Nó là cú pháp JavaScript đặc biệt của React."_

**Component Tree (Cây Component):**

```
          App
           │
    ┌──────┴──────┐
    │             │
  Header        Main
                  │
            ┌─────┴─────┐
            │           │
        Sidebar      Content
                        │
                   ┌────┴────┐
                   │         │
                 Post      Post
```

> _"React app = cây component._
>
> _- Component lớn nhất (App) ở trên_ > _- Chứa component con (Header, Main)_ > _- Component con có thể chứa component con nữa_
>
> _Giống như Lego:_ > _- 1 miếng Lego nhỏ = 1 component_ > _- Ghép nhiều miếng nhỏ → thành đồ lớn_ > _- Có thể tháo ra, tái sử dụng_
>
> _Ví dụ thực tế:_ > _- `Button` component → dùng ở 20 chỗ khác nhau_ > _- `Card` component → hiển thị user, post, product... chỉ cần thay data_
>
> _Đó là sức mạnh của React: **Tái sử dụng**."_

---

### PHẦN E: JSX – 3 Điểm Khác HTML (15 phút)

> _"⏱️ 15 PHÚT. CHỈ DẠY 3 ĐIỂM CƠ BẢN: className, self-closing, và comments._
>
> _Concept thứ 3 cuối cùng: JSX._
>
> _Nhiều bạn nhìn JSX sẽ nghĩ: 'Ủa, đây là HTML mà?'_
>
> _KHÔNG. JSX **KHÔNG PHẢI HTML**. Đây là JavaScript có syntax giống HTML."_

```jsx
// Trông giống HTML...
const element = <h1 className="title">Hello</h1>;

// Nhưng thực chất đây là JavaScript!
// Babel (compiler) biến đổi JSX thành:
const element = React.createElement(
  "h1", // Tag name
  { className: "title" }, // Props (attributes)
  "Hello" // Children
);

// React.createElement() tạo ra JavaScript object mô tả UI
// React sau đó dùng object này để tạo DOM thật
// JSX → JavaScript Object → DOM
```

---

#### Quy Tắc JSX #1: Phải có 1 root element

```jsx
// ❌ SAI - 2 root elements
function App() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>  // Lỗi: không được return 2 elements cạnh nhau
  );
}

// ✅ ĐÚNG - Dùng <div> làm wrapper
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// ✅ ĐÚNG - Dùng Fragment (<>...</>) - không tạo DOM thừa
// Fragment không tạo thêm <div> trong HTML
function App() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}
```

> _"Khuyến nghị: Dùng Fragment khi không cần wrapper div."_

---

#### Quy Tắc JSX #2: `className` thay vì `class`

```jsx
// ❌ SAI - "class" là keyword của JavaScript
<div class="container">Content</div>

// ✅ ĐÚNG - Dùng "className"
<div className="container">Content</div>
```

> _"Tại sao `className`? Vì JSX là JavaScript. Mà `class` là reserved keyword trong JS (để khai báo class). Nên React dùng `className` để tránh conflict."_

---

#### Quy Tắc JSX #3: Dùng `{}` để nhúng JavaScript

```jsx
function Greeting() {
  const name = "John";
  const age = 25;
  const isAdult = age >= 18;

  return (
    <div>
      {/* Nhúng biến */}
      <h1>Hello {name}</h1>

      {/* Nhúng expression */}
      <p>You are {age} years old</p>

      {/* Nhúng điều kiện (ternary operator) */}
      <p>{isAdult ? "Adult" : "Minor"}</p>

      {/* Nhúng function call */}
      <p>Current time: {new Date().toLocaleTimeString()}</p>

      {/* ❌ KHÔNG thể dùng if/else trực tiếp trong JSX */}
      {/* if (isAdult) { return <p>Adult</p> } */}

      {/* ✅ Dùng ternary hoặc && */}
      {isAdult && <p>You can vote!</p>}
    </div>
  );
}
```

> _"Cặp ngoặc `{}` = JavaScript mode._ > _Bên trong `{}` ta có thể viết bất kỳ JavaScript **expression** nào._
>
> _Expression = biểu thức có giá trị trả về: `name`, `2 + 3`, `age >= 18`, `myFunction()`_
>
> _KHÔNG thể dùng if/else bên trong JSX vì if/else là **statement** (không return giá trị ngay)._
>
> _Dùng ternary `? :` hoặc logical `&&` thay thế."_

---

#### JSX vs HTML - 3 Điểm Khác Biệt (Session 0 CHỈ DẠY 3 ĐIỂM)

> _"⏱️ Session 0 CHỈ DẠY 3 ĐIỂM CƠ BẢN. Các điểm khác (htmlFor, inline style, events) học ở Session 1."_

| **Feature**       | **HTML**                           | **JSX**                           |
| ----------------- | ---------------------------------- | --------------------------------- |
| CSS class         | `class="container"`                | `className="container"` ⭐        |
| Self-closing tags | `<img src="...">` (không cần `/>`) | `<img src="..." />` (bắt buộc) ⭐ |
| Comments          | `<!-- Comment -->`                 | `{/* Comment */}` ⭐              |

> _"Chi tiết khác (style, events...) học Session 1."_

---

### ⚠️ CẮT PHẦN NÀY - DỜI SANG SESSION 1

~~### PHẦN F: React KHÔNG PHẢI Là Gì~~

> _"Để tránh hiểu lầm, hãy nhớ React KHÔNG PHẢI:"_

| ❌ Hiểu Lầm               | ✅ Sự Thật                                  |
| ------------------------- | ------------------------------------------- |
| React là framework full   | React chỉ lo UI layer                       |
| React thay thế backend    | React chỉ gọi API, không xử lý logic server |
| React tự động fetch data  | Cần thư viện khác (Axios, React Query)      |
| React làm web đẹp hơn     | React giúp quản lý state, không phải CSS    |
| React bắt buộc dùng class | React 18 dùng function component + Hooks    |

---

### PHẦN G: Hooks – 2 Phút Giới Thiệu (KHÔNG CODE VÍ DỤ)

> _"⏱️ 2 PHÚT THÔI. Giới thiệu khái niệm, KHÔNG code ví dụ._
>
> _Các bạn sẽ thấy từ 'Hook' rất nhiều trong React._ > _Hook không phải magic, nó chỉ là **hàm đặc biệt** của React._
>
> _Session 1 sẽ học chi tiết. Bây giờ chỉ cần biết Hook là gì."_

**Hook = Hàm đặc biệt bắt đầu bằng `use` (useState, useEffect...)**

**Ví dụ Hook:**

```jsx
function Counter() {
  // ✅ Hook - gọi ở top level
  const [count, setCount] = useState(0);
  return <button>{count}</button>;
}
```

> _"⛔ KHÔNG CODE VÍ DỤ BAD HOOK. Session 1 sẽ học chi tiết._
>
> _Bây giờ chỉ cần nhớ: **Thấy `use` ở đầu = đó là Hook**._
>
> _useState học Session 1, useEffect học Session 4."_

**Hooks phổ biến (tên thôi):** `useState`, `useEffect`, `useRef`, `useContext`

---

## 4️⃣ LIVE DEMO – VITE + REACT + TYPESCRIPT (10 Phút)

> _"⏱️ 10 PHÚT. Tạo project React + TypeScript và giải thích 2 file chính._
>
> _Mục tiêu: Thấy nó chạy được. TypeScript từ đầu."_

### Bước 1: Tạo Vite Project

````bash
npm create vite@latest hello-react -- --template react
### Bước 1: Tạo Project React + TypeScript

```bash
# Tạo project với TypeScript template
npm create vite@latest hello-react -- --template react-ts  # 👈 react-ts

cd hello-react
npm install
npm run dev
````

> _"Lưu ý: `--template react-ts` để có TypeScript. Khóa học này dùng TypeScript từ đầu._
>
> _Sau khi chạy `npm run dev`, mở http://localhost:5173"_

---

### Bước 2: Giải Thích `main.tsx`

```tsx
// File: src/main.tsx (TypeScript)
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.tsx"; // 👈 .tsx extension
import "./index.css";

// Mount React vào DOM
// 'root' là div trong index.html
createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

> _"Dòng này mount React vào DOM. `root` là div trong `index.html`. Từ đây React kiểm soát toàn bộ UI._
>
> _TypeScript: dấu `!` sau `getElementById('root')` là non-null assertion._
>
> _⏱️ KHÔNG đi sâu StrictMode. Chỉ nói: 'Giúp phát hiện lỗi khi dev'"_

---

### Bước 3: Giải Thích `App.tsx`

```tsx
// File: src/App.tsx (TypeScript)
function App() {
  return <h1>Hello React with TypeScript</h1>;
}

export default App;
```

> _"Component đầu tiên. Hàm App return JSX. Session sau ta viết component phức tạp hơn._
>
> _⏱️ 10 phút hết. Nếu quá 80 phút, SKIP phần demo và chỉ assign về nhà làm."_

---

## 5️⃣ COMMON MISTAKES - CẮT BỎ (Session 1 sẽ học)

> _"⏱️ SKIP PHẦN NÀY. Common mistakes sẽ học khi code thực tế ở Session 1._
>
> _Nếu quá 85 phút, bỏ qua ngay section này."_

~~### ❌ Lỗi #1: Mutating State~~  
~~### ❌ Lỗi #2: If/else trong JSX~~  
~~### ❌ Lỗi #3: Tags không close~~  
~~### ❌ Lỗi #4: `class` thay vì `className`~~  
~~### ❌ Lỗi #5: Thao tác DOM trực tiếp~~

_Nội dung cắt bỏ để đúng 90 phút. Các lỗi phổ biến sẽ học trong Session 1 khi code thực tế._

---

## ⚠️ INSTRUCTOR Q&A SECTION - CẮT BỎ ĐỂ ĐỦ 90 PHÚT

~~## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS~~

_Section này cắt bỏ. Q&A xử lý tự nhiên trong buổi học thay vì có script._

---

## 🎯 MINI TASK - SKIP NẾU QUÁ GIỜ (Assign Về Nhà)

> _"⏱️ Nếu quá 85 phút, BẮT BUỘC skip. Assign về nhà và check ở Session 1."_

**Task: Vẽ Sơ Đồ State → UI Cycle**

Vẽ sơ đồ trên giấy:

```
State → Render → UI
  ↑              ↓
  └──── Event ───┘
```

Giải thích bằng lời: Mỗi bước trong vòng lặp nghĩa là gì?

---

## 📚 HOMEWORK (10 Phút Giải Thích - Cuối Buổi)

> _"⏱️ 5 phút để giải thích homework. ĐỪNG giải thích quá 90 phút!"_

**Task 1: Tạo React + TypeScript Project**

- Tạo project: `npm create vite@latest my-app -- --template react-ts`
- Chạy được `npm run dev`
- Sửa `App.tsx`: Hiển thị tên của bạn
- Chụp màn hình kết quả (file .tsx phải thấy rõ)

**Task 2: Vẽ Sơ Đồ State → UI**

- Vẽ lại sơ đồ vòng lặp (không nhìn slide)
- Nộp ảnh vẽ tay hoặc digital

> _"Session 1 sẽ check homework. Ai chưa làm sẽ không theo kịp._
>
> _⏱️ ĐÚNG 90 PHÚT - KẾT THÚC!"_

---

## 📋 TEACHING NOTES

**⏱️ STRICT 90-Minute Timing:**

- **0-20 min:** Phần A (Vanilla Pain) + Phần B (React Solution)
- **20-45 min:** Phần C (State Cycle) - QUAN TRỌNG NHẤT
- **45-55 min:** Phần D (Component = Function)
- **55-70 min:** Phần E (JSX - chỉ 3 điểm)
- **70-80 min:** Phần G (Hooks 2 phút) + Vite Demo (8 phút)
- **80-90 min:** Q&A + Homework assignment

**⚠️ TIME CHECKPOINTS:**

- **Minute 20:** Phải xong Phần A+B
- **Minute 45:** Phải xong Phần C (State Cycle)
- **Minute 70:** Phải xong JSX
- **Minute 80:** Phải bắt đầu Vite Demo
- **Nếu quá 80 phút:** BẮT BUỘC skip Common Mistakes, Mini Task, chỉ giữ Homework assignment

**🎯 Ưu Tiên Nội Dung:**

1. **MUST TEACH (không được skip):**

   - Vanilla JS pain (Phần A)
   - State → UI Cycle sơ đồ (Phần C)
   - Component = Function (Phần D)

2. **CAN SHORTEN (rút ngắn nếu cần):**

   - React solution code example (Phần B) - chỉ show, không giải thích từng dòng
   - JSX (Phần E) - chỉ 3 điểm: className, self-closing, comments
   - Vite demo - chỉ chạy và giải thích main.tsx, App.tsx

3. **MUST SKIP nếu quá giờ:**
   - React KHÔNG PHẢI Là Gì (Phần F)
   - Common Mistakes (Section 5)
   - Mini Task (làm về nhà)

**📝 Instructor Reminders:**

- **TypeScript từ đầu:** Nhấn mạnh `react-ts` template, không dùng plain `react`
- **Mental models > Syntax:** Học viên phải hiểu "tại sao" trước "làm thế nào"
- **90 phút = HARD LIMIT:** Không giảng quá 90 phút. Better to skip content than run over.
- **Check time mỗi 20 phút:** Set timer để kiểm soát pace

**🎓 Session 1 Preview:**

> _"Buổi sau (Session 1):_
>
> _- Setup project React + TypeScript chi tiết_ > _- TypeScript Survival Kit: Tất cả những gì cần biết về TS trong React_ > _- Component Architecture với TypeScript_ > _- Props typing, interface, type annotations_
>
> _⚠️ Nhớ làm homework: Tạo project với react-ts template!"_
> └──── Event ───┘

```

---

## ⚠️ CÁC SECTION CŨ - ĐÃ CẮT ĐỂ ĐỦ 90 PHÚT

~~## 8️⃣ HOMEWORK / EXTENSION TASK~~
~~## 9️⃣ CHECKPOINT & EVALUATION~~
~~## 🔟 OLD TEACHING NOTES~~

_Các section này đã được gộp vào phần HOMEWORK và TEACHING NOTES phía trên_

---

## 🎓 KẾT THÚC SESSION 0

**✅ Học viên phải nắm được:**

1. Tại sao React tồn tại (Vanilla JS pain points)
2. Sơ đồ State → Render → UI → Event → State
3. Component = Function returning JSX
4. JSX ≠ HTML (3 điểm cơ bản)
5. Tạo được project với `react-ts` template

**⏱️ ĐÚNG 90 PHÚT - KHÔNG DẠY QUÁ GIỜ**

**📍 Session 1 Preview:** Setup + TypeScript + Props + Component Architecture

---

_End of Session 00 - React Foundation (90-minute STRICT version)_
```
