# LESSON PLAN: SESSION 0 – REACT FOUNDATION & DEVELOPER MINDSET

## 1️⃣ SESSION OVERVIEW

* **Title:** Why React Exists – Mental Models Before Code
* **Duration:** 90-120 minutes
* **Goal:** Students will understand **why** React exists and build mental models for Component, State, and JSX **before** writing production code.
* **Outcome:** Students can explain React's purpose, know the State→UI cycle, and are not intimidated by JSX.

---

## 2️⃣ INSTRUCTOR OPENING SCRIPT

> *"Chào các bạn. Hôm nay là buổi đặc biệt.
> Ta **KHÔNG** viết nhiều code. Ta **KHÔNG** học API.
> Ta chỉ làm 1 việc: **Hiểu tại sao React tồn tại**.*
>
> *Nhiều người học React bằng cách copy-paste code từ tutorial.
> 3 tháng sau vẫn không hiểu `useState` làm gì, `useEffect` để đâu.
> Hôm nay ta xây **bản đồ tư duy** – mental model.
> 16 buổi sau, khi gặp bug, các bạn sẽ biết mình đang ở đâu trên bản đồ."*

> 🔥 **WHY THIS SESSION EXISTS?**
> *"Nếu không hiểu 'tại sao', bạn sẽ học React như học công thức toán – thuộc lòng nhưng không biết dùng khi nào."*

---

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🧠 The Core Problem React Solves

**Instructor Script:**
> *"Trước khi có React, ta viết UI bằng Vanilla JS.
> Nghe có vẻ đơn giản, nhưng hãy xem ví dụ này."*

#### ❌ Vanilla JS Pain (Live Demo)

```html
<button id="btn">Count: 0</button>
<p id="status">Idle</p>

<script>
let count = 0;

document.getElementById('btn').addEventListener('click', () => {
  count++;
  
  // Phải update 2 chỗ thủ công
  document.getElementById('btn').innerText = `Count: ${count}`;
  
  if (count > 5) {
    document.getElementById('status').innerText = 'Too many!';
  }
});
</script>
```

**Instructor Explain:**
> *"Vấn đề: Khi `count` thay đổi, ta phải nhớ update **TẤT CẢ** chỗ hiển thị nó.
> App càng lớn → càng nhiều chỗ → càng dễ quên → **BUG**."*

#### ✅ React Solution

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <p>{count > 5 ? 'Too many!' : 'Idle'}</p>
    </>
  );
}
```

**Instructor Explain:**
> *"React tự động update UI khi `count` thay đổi.
> Ta chỉ nói 'UI trông như thế nào', không nói 'cập nhật cái gì'."*

---

### 🔄 The React Mental Model (CORE)

**Vẽ lên bảng:**

```
┌─────────┐
│  State  │ ──────▶ Render ──────▶ UI
└─────────┘                         │
     ▲                              │
     └────────── Event ◀─────────────┘
```

**Instructor Script:**
> *"Đây là sơ đồ duy nhất các bạn cần nhớ suốt khóa học:*
> 1. *State thay đổi → React render lại*
> 2. *Render tạo ra UI mới*
> 3. *User tương tác → Event xảy ra*
> 4. *Event update State → quay lại bước 1*
>
> *Nếu nhớ được vòng lặp này, 70% React tự hiểu."*

---

### 🧩 Component = Function (Not Class, Not File)

**Instructor Script:**
> *"Component không phải là class.
> Component không phải là file.
> Component = **1 hàm return UI**."*

```jsx
// Component đơn giản nhất
function Welcome() {
  return <h1>Hello</h1>;
}

// Component có logic
function Greeting({ name }) {
  const message = `Hello, ${name}!`;
  return <p>{message}</p>;
}
```

**Instructor Explain:**
> *"React app = cây component.
> Component lớn chứa component nhỏ.
> Giống như Lego – ghép từ miếng nhỏ thành đồ lớn."*

---

### 📝 JSX – Cách Viết UI Bằng JS (Not HTML!)

**Instructor Script:**
> *"JSX trông giống HTML, nhưng **KHÔNG PHẢI HTML**."*

```jsx
// JSX
const element = <h1 className="title">Hello</h1>;

// Thực chất là:
const element = React.createElement(
  "h1",
  { className: "title" },
  "Hello"
);
```

**Rules of JSX:**
1. Phải có **1 root element** (hoặc dùng `<>...</>`)
2. Dùng `className` thay vì `class`
3. Dùng `{}` để nhúng JS expression

```jsx
// ✅ Đúng
function App() {
  const name = "React";
  return <h1>Hello {name}</h1>;
}

// ❌ Sai
function App() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>  // Lỗi: 2 root elements
  );
}
```

---

### 🚫 React KHÔNG PHẢI Là Gì

**Instructor Script:**
> *"Để tránh hiểu lầm, hãy nhớ React **KHÔNG PHẢI**:"*

| ❌ Hiểu Lầm | ✅ Sự Thật |
|------------|-----------|
| React là framework full | React chỉ lo UI layer |
| React thay thế backend | React chỉ gọi API, không xử lý logic server |
| React tự động fetch data | Cần thư viện khác (Axios, React Query) |
| React làm web đẹp hơn | React giúp quản lý state, không phải CSS |

---

### 🪝 Hooks – Công Cụ Để Làm Việc Với State

**Instructor Script:**
> *"Các bạn sẽ thấy từ 'Hook' rất nhiều trong React.
> Hook không phải magic, nó chỉ là **hàm đặc biệt** của React."*

**What are Hooks?**

> **Hook = Hàm để "móc" vào React features (state, lifecycle, context)**

**Đặc điểm nhận biết:**
- Tên bắt đầu bằng `use` (useState, useEffect, useRef, useQuery...)
- Chỉ gọi được **trong component** (không gọi trong function thường)
- Phải gọi ở **top level** (không trong if/loop/nested function)

**Ví dụ:**

```jsx
function Counter() {
  // ✅ Hook - gọi ở top level
  const [count, setCount] = useState(0);
  
  return <button>{count}</button>;
}

function Bad() {
  if (true) {
    const [count, setCount] = useState(0); // ❌ Sai - trong if
  }
}
```

**Instructor Explain:**
> *"Hôm nay chỉ cần biết Hook **tồn tại**.
> Chi tiết `useState`, `useEffect` ta sẽ học ở Session 2.
> Còn bây giờ, chỉ cần nhớ: **Thấy `use` ở đầu = đó là Hook**."*

**Common Hooks (nhắc tên thôi):**
- `useState` - quản lý state
- `useEffect` - side effects (fetch API, timer...)
- `useRef` - reference đến DOM element
- `useContext` - global state (nâng cao)

---

## 4️⃣ LIVE DEMO – MINIMAL REACT APP

**Instructor Script:**
> *"Bây giờ ta chạy thử 1 React app tối thiểu.
> Mục tiêu: Thấy nó chạy được. Không cần hiểu hết."*

### Step 1: Create Vite Project (Quick)

```bash
npm create vite@latest hello-react -- --template react
cd hello-react
npm install
npm run dev
```

### Step 2: Show `main.jsx`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

**Instructor Explain:**
> *"Dòng này mount React vào DOM.
> `root` là div trong `index.html`.
> Từ đây React kiểm soát toàn bộ UI."*

### Step 3: Show `App.jsx`

```jsx
function App() {
  return <h1>Hello React</h1>
}

export default App
```

**Instructor Explain:**
> *"Đây là component đầu tiên.
> Hàm `App` return JSX.
> Buổi sau ta sẽ viết component phức tạp hơn."*

---

## 5️⃣ COMMON STUDENT QUESTIONS & ANSWERS

1. **Q:** *"JSX có phải HTML không?"*
   * **A:** Không. JSX là cú pháp JS. Nó compile thành `React.createElement()`.

2. **Q:** *"Tại sao không dùng `document.getElementById`?"*
   * **A:** Vì React quản lý DOM tự động. Nếu ta thao tác DOM thủ công, React sẽ bị rối.

3. **Q:** *"Component phải viết trong file riêng không?"*
   * **A:** Không bắt buộc. Nhưng để code gọn, nên 1 file = 1 component chính.

4. **Q:** *"React có thay thế được jQuery không?"*
   * **A:** Có, nhưng React không phải để "thay thế" mà để "tư duy khác". jQuery thao tác DOM, React quản lý state.

---

## 6️⃣ IN-CLASS MINI TASK

**Task:** Vẽ sơ đồ State → UI

> *"Hãy vẽ lại sơ đồ này trên giấy:*
> ```
> State → Render → UI
>   ↑               ↓
>   └─── Event ─────┘
> ```
> *Và giải thích cho bạn bên cạnh."*

---

## 7️⃣ HOMEWORK / EXTENSION TASK

**Yêu cầu:** Đọc trước tài liệu Session 1
- Hiểu Vite là gì (build tool)
- Hiểu TypeScript cơ bản (type annotation)

**Không yêu cầu:** Viết code React. Buổi sau mới bắt đầu.

---

## 8️⃣ CHECKPOINT & EVALUATION

- **Pass:** Học viên giải thích được "React giải quyết vấn đề gì"
- **Pass:** Vẽ được sơ đồ State → UI
- **Pass:** Biết JSX không phải HTML

### ⚠️ HÔM NAY CHÚNG TA CHƯA HỌC

**Instructor Script:**
> *"Để tránh nhầm lẫn, những thứ này ta **CHƯA** học hôm nay:*
> - *`useState` / `useEffect` (hooks)*
> - *Props (truyền data giữa components)*
> - *Event handling chi tiết*
> - *Fetch API / gọi backend*
>
> *Những thứ này sẽ xuất hiện **tự nhiên** khi ta bắt đầu project ở Session 1.
> Hôm nay chỉ cần hiểu **tại sao** và **mental model**."*

---

## 9️⃣ ANTI-PATTERNS (CẤM DẠY)

- ❌ Dạy class component (React 16 cũ)
- ❌ Dạy `componentDidMount`, `componentWillUnmount` (lifecycle cũ)
- ❌ Dạy quá sâu về Virtual DOM (không cần thiết cho beginner)
- ❌ Cho học viên code nhiều (Session 0 là lý thuyết)

---

## 🔟 TEACHING NOTES

- **Mindset:** Session này là "đầu tư tâm lý". Học viên sẽ thấy chậm, nhưng buổi sau sẽ nhanh gấp đôi.
- **Timing:** Nếu hết giờ, bỏ phần Demo. Ưu tiên Mental Model.
- **Visual:** Vẽ sơ đồ lên bảng. Hình ảnh giúp nhớ lâu hơn text.

---

## 🌉 BRIDGE TO SESSION 1

**Instructor Closing Script:**

> *"Ở buổi sau, ta sẽ bắt đầu viết component **thật**, có state **thật**, có interaction **thật**.
> Nhưng mọi thứ các bạn sắp thấy đều quay về sơ đồ này:*
>
> ```
> State → Render → UI
>   ↑               ↓
>   └─── Event ─────┘
> ```
>
> *Khi bạn thấy `useState` → đó là State.
> Khi bạn thấy `onClick` → đó là Event.
> Khi bạn thấy `return <div>` → đó là UI.*
>
> *Hẹn gặp lại các bạn ở Session 1 – Setup & Component Thinking!"*
