# GIÁO TRÌNH REACTJS MASTER (MERN → REACT) - FOR 2 MONTHS

> **VERSION:** 1.0.0
> **Dành riêng cho:** Đào tạo Giảng viên nguồn & Mentor chuyên nghiệp.
> **Đối tượng học viên:** Học xong MERN (Backend ExpressJS + TypeScript), đã nắm vững tư duy API/Middleware/Model.
> **Outcome ưu tiên #1:** Thành thạo **xử lý API trong React** (query/mutation, loading/error, auth token, refresh token).  
> **Outcome ưu tiên #2:** Nắm React core đủ để "code React khá", có tư duy kiến trúc và kỹ thuật production vừa đủ.  
> **Project cuối khóa:** Build Frontend cho Backend `shoppingCardBE` trong `learnNodeJS/ch04-shoppingCardProject/shoppingCardBE`.  
> **Triết lý:** "Pain-Driven Development: Code thủ công cho thấy khổ (useState/useEffect) -> Dùng thư viện giải cứu (RHF/Query)."

---

## ✅ TECH STACK CHÍNH THỨC CỦA KHÓA HỌC

### Build tool & Core

- Vite
- React 18 (Client Features focus)
- TypeScript (Frontend patterns)

### Routing

- React Router v6

### UI & Styling

- Tailwind CSS v4 (Latest features - Performance focus)
- shadcn/ui

### Form & Validation

- React Hook Form
- Zod

### Data & State

- Axios (API layer)
- TanStack Query (React Query)
- Zustand

---

## 🎯 LEARNING OUTCOMES (Measurable)

Sau khóa, học viên có thể:

1. **React Core (Foundation)**

   - Viết functional component với TypeScript (props, state, events)
   - Render list với key đúng chuẩn
   - Quản lý local state với useState, useEffect đúng chỗ
   - **Measurable:** Build 1 todo app trong 1 giờ

2. **API Handling (Core Skill)**

   - Setup axios instance với baseURL, headers, interceptors
   - Normalize API errors (422/401/500) và hiển thị UI rõ ràng
   - Implement refresh token flow (basic - no complex patterns)
   - **Measurable:** Implement login/register/logout trong 2 giờ

3. **React Query (Server State)**

   - Dùng useQuery với loading/error/empty states
   - Dùng useMutation với proper invalidation
   - Invalidate cache đúng chuẩn
   - **Measurable:** Build CRUD page với React Query trong 3 giờ

4. **Form & Validation**

   - Build form với RHF + Zod
   - Map 422 errors về field-level
   - **Measurable:** Build register form (5+ fields) trong 1.5 giờ

5. **Production Readiness**
   - Code có structure rõ ràng (pages/components/lib/api)
   - UI có loading/error/empty states đầy đủ
   - Auth flow hoàn chỉnh (login/logout/protected routes)
   - **Measurable:** Review code peer đạt 8/10 điểm

---

## 🗺️ BIG PICTURE: SHOPPING CART FE ARCHITECTURE

_Giảng viên bắt buộc vẽ mô hình này lên bảng ngay buổi đầu tiên._

1. **UI/Pages Layer**

   - Public: Home / Product List (mock UI)
   - Auth: Login/Register
   - User: Me/Update Profile/Change Password
   - Media: Upload Image (1 file) + Preview

2. **Data Layer (Trọng tâm khóa)**

   - Axios instance + interceptors (attach token, 401 handling)
   - TanStack Query (query/mutation, cache, invalidate, retry)
   - Error normalization (chuẩn hóa lỗi 422/401/500)

3. **State Management**

   - **Server state**: React Query (dữ liệu từ BE)
   - **Global UI state**: Zustand (auth tokens minimal)
   - **Local state**: useState (input UI nhỏ)

4. **Core Features (CUT SCOPE)**

   - JWT auth: access/refresh (SIMPLIFIED - no single-flight)
   - **Auth flows:** Register, Login, Logout, Get Me, Update Me, Change Password
   - **Bỏ:** Verify email, Resend verify, Forgot password, Reset password
   - **Media:** Upload image only (1 file) - bỏ video upload
   - Form validation map rules BE (422 field errors)
   - Refresh token: basic retry (1 time, no queue/single-flight pattern)

5. **🧠 State Boundary Rules (MUST MEMORIZE)**

| Loại dữ liệu        | Công cụ             | Ví dụ                                     |
| :------------------ | :------------------ | :---------------------------------------- |
| **Server State**    | **React Query**     | `user`, `products`, `cart`                |
| **Global UI State** | **Zustand**         | `auth_tokens`, `theme`, `sidebar_open`    |
| **Local UI State**  | **useState**        | `modal_open`, `input_value`, `is_loading` |
| **Form State**      | **React Hook Form** | `login_form`, `register_form`             |

> **❌ Rule:** Không nhét Server Data vào Zustand. Không dùng useEffect để fetch data.

6. **📁 File Structure Rules (Mentor-Safe)**

- Component > 150 dòng → **Tách**.
- Page > 300 dòng → **Tách hooks / sub-components**.
- API logic → Luôn ở `lib/api`.
- Axios config → `lib/http`.

---

## 📌 BACKEND TARGET (SHOPPINGCARDBE) - API CONTRACT MINIMAL

> Backend nằm ở: `learnNodeJS/ch04-shoppingCardProject/shoppingCardBE/src`  
> Base URL (local): `http://localhost:3000`

### Users/Auth (Core Only)

- `POST /users/register`
  - body: `{ name, email, password, confirmed_password, date_of_birth }`
  - resp: `{ msg, data: { access_token, refresh_token } }`
- `POST /users/login`
  - body: `{ email, password }`
  - resp: `{ message, result: { access_token, refresh_token } }`
- `POST /users/logout`
  - headers: `Authorization: Bearer <access_token>`
  - body: `{ refresh_token }`
- `POST /users/me`
  - headers: `Authorization: Bearer <access_token>`
- `PATCH /users/me`
  - headers: `Authorization: Bearer <access_token>`
  - body: `{ name?, date_of_birth?, bio?, location?, website?, username?, avatar?, cover_photo? }`
- `PUT /users/change_password`
  - headers: `Authorization: Bearer <access_token>`
  - body: `{ old_password, password, confirm_password }`
- `POST /users/refresh-token`
  - body: `{ refresh_token }`
  - resp: `{ message, result: { access_token, refresh_token } }`

**Bỏ:** verify-email, resend-verify-email, forgot-password, verify-forgot-password, reset-password

### Medias (Simplified)

- `POST /medias/upload-image` (multipart form-data)
  - field: `image` (1 file only cho khóa này)
  - resp: `{ message, url: [{ url, type }] }`
- Serve:
  - `GET /static/image/:filename`

**Bỏ:** upload-video, video streaming (quá phức tạp cho scope 2 tháng)

### 🔌 ERROR SHAPE CONTRACT (Backend ↔ Frontend)

```ts
interface ApiError {
  status: number;
  message: string;
  errors?: Record<string, string>; // Field validation errors
}
```

**UI Handling Rule:**

- **422 (Validation):** Show field errors (map `errors` vào RHF).
- **401 (Auth):** Auto Refresh -> Fail thì Logout.
- **403 (Permission):** Toast error + Redirect.
- **500 (Server):** Toast global error ("Server busy").
- **Network Error:** Toast.

### 🌍 ENV & CONFIG RULES

- **File:** `.env` (không commit lên git).
- **Variable:** `VITE_API_URL=http://localhost:3000`
- **Rule:**
  - ❌ KHÔNG dùng `process.env`.
  - ✅ BẮT BUỘC dùng `import.meta.env.VITE_API_URL`.

---

## ⏱️ PHÂN BỔ THỜI GIAN (KHÓA 2 THÁNG - 18 BUỔI)

> **Lưu ý:** Buổi 0 là Foundation session (90 phút) - có thể gán vào pre-course hoặc Tuần 1.

- React Core + TypeScript: ~35% (6 buổi: Buổi 0-4, Buổi 6)
- API & Data Handling (Zustand + Axios + React Query): ~30% (5 buổi: Buổi 5, 7-10)
- UI & Form (Tailwind, shadcn, RHF, Zod): ~15% (2 buổi - combined)
- Project Sprint + Review: ~20% (3 buổi)

---

## 📅 LỘ TRÌNH 8 TUẦN (MỖI TUẦN 2 BUỔI)

---

## 📅 TUẦN 1: FOUNDATION (VITE + TS + REACT CORE)

**Checklist Tuần 1:**

- [ ] Học viên setup được Vite + React 18 + TS
- [ ] Hiểu JSX, component thinking
- [ ] Nắm TypeScript cho React (Props, State, Events)
- [ ] Render list + state cơ bản

### BUỔI 0: REACT FOUNDATION & DEVELOPER MINDSET

> **Tài liệu:** `session-00-react-foundation`

🎬 **OPENING SCRIPT (5 phút)**
_"Chào các bạn. Trước khi viết dòng code React đầu tiên, chúng ta cần hiểu: React sinh ra để giải quyết vấn đề gì? Tại sao không dùng Vanilla JS? Hôm nay ta không code nhiều, ta xây mental model - bản đồ tư duy để 16 buổi sau các bạn không bị lạc."_

**Scope buổi:**

- **React giải quyết vấn đề gì?**
  - Pain point của Vanilla JS: UI state không đồng bộ
  - React = UI as a function of state
- **Mental Model cốt lõi:**
  ```
  State → Render → UI
    ↑               ↓
    └─── Event ─────┘
  ```
- **3 Nguyên tắc sống còn:**
  1. Component = Function (không phải class, không phải file)
  2. UI được render từ State (không thao tác DOM trực tiếp)
  3. React chỉ lo UI layer (không phải framework full)
- **JSX là gì (đúng cách):**
  - JSX = cách viết UI bằng JS
  - `<h1>Hello</h1>` = `React.createElement("h1", null, "Hello")`
  - **JSX Children:** `<Button>Click Me</Button>` → `children` prop
  - **Conditional Rendering (intro):** `{isLoggedIn && <Profile />}`, `{count > 5 ? 'High' : 'Low'}`
- **Hooks là gì (concept only):**
  - Hook = hàm đặc biệt để làm việc với state/lifecycle
  - Hook bắt đầu bằng `use` (useState, useEffect, useRef)
  - Rules: chỉ gọi trong component, phải ở top level
  - Chi tiết `useState` sẽ học ở Session 1, `useEffect` ở Session 4.
- **Demo tối thiểu:**
  ```jsx
  function App() {
    return <h1>Hello React</h1>;
  }
  ```
  - Chỉ để thấy React app chạy được
  - Không yêu cầu hiểu hook, props, state

**Deliverables:**

- Học viên hiểu "tại sao React" (không phải "làm thế nào")
- Biết Component = Function
- Biết State → UI (mental model)
- Không sợ JSX

---

### BUỔI 1: SETUP + COMPONENT THINKING + TS SURVIVAL KIT

> **Tài liệu:** `session-01-setup-ts`

🎬 **OPENING SCRIPT (5 phút)**
_"Các bạn đã biết HTML/CSS/JS, đã biết call API trong dự án MERN. Nhưng React là cách tổ chức UI để không bị 'copy-paste chết người'. Hôm nay ta dựng nền móng: setup chuẩn, tư duy component, và TypeScript cho React."_

1. **🎯 Mục tiêu**

   - Setup Vite + React 18 + TS + **Tailwind v4**
   - Config **Absolute Imports** (`@/components`, `@/lib`)
   - Folder structure cơ bản
   - **TypeScript Survival Kit**: Props, Events, **Basic useState Generics** (30 phút)

2. **🧠 Mental Model**

   - **HTML**: xây nhà đúc
   - **Component**: lắp Lego, tái sử dụng
   - **TS trong React**: khác backend (không có req/res, có Props/Events)

3. **📚 Live Coding**

   - Tạo layout `MainLayout`
   - Tạo `Button`, `Input` (chưa shadcn)
   - **Setup & Config** (15 phút):

     - `vite.config.ts` alias (`@`)
     - `tsconfig.json` paths
     - Tailwind v4 setup (CSS-first configuration)

   - **TS Survival Kit** (30 phút - CRITICAL):

     ```tsx
     // 1. Props typing
     interface ButtonProps {
       children: React.ReactNode;
       onClick?: () => void;
       className?: string;
     }

     // 2. Event types (CỰC QUAN TRỌNG)
     const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {};
     const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {};

     // 3. Hooks với Generics
     const [user, setUser] = useState<User | null>(null);
     const inputRef = useRef<HTMLInputElement>(null);

     // 4. API Response typing
     interface ApiResponse<T> {
       message: string;
       result: T;
     }
     ```

   - Mock UI: Home + ProductCard

4. **🧪 Checkpoint**

   - Học viên tạo được 2 components và render đúng
   - Không có lỗi TS trong console

5. **🚨 Red Flags**
   - File 300 dòng không tách
   - Viết UI không theo component boundaries
   - Dùng `any` cho props/events

---

### BUỔI 2: PROPS, LIST, STATE

> **Tài liệu:** `session-02-props-state`

🎬 **OPENING SCRIPT (5 phút)**  
_"React không mạnh ở chỗ viết HTML trong JS, mà mạnh ở chỗ render danh sách và UI thay đổi theo state. Hôm nay ta làm list sản phẩm và tương tác cơ bản."_

1. **🎯 Mục tiêu**

   - Props, list rendering + `key`
   - `useState` + event handlers
   - **JSX Deep Dive (20p):** Children, Conditional Rendering, Fragments
   - **Props Flow (15p):** Props down, Events up (callback pattern)
   - **State Patterns (15p):** Controlled vs Uncontrolled inputs (intro)
   - Tránh derived state
   - **Mini Foundation (15p):** React Rendering Model (Virtual DOM, Re-render triggers)

2. **📚 Live Coding**

   - **JSX Patterns (20 phút - ĐẦU GIỜ):**

     ```jsx
     // 1. Children prop
     function Card({ children, title }) {
       return (
         <div className="card">
           <h2>{title}</h2>
           {children} {/* Nhận bất kỳ nội dung gì */}
         </div>
       );
     }
     <Card title="Product">
       <p>Description</p>
     </Card>;

     // 2. Conditional rendering patterns
     {
       isLoading && <Spinner />;
     }
     {
       /* AND operator */
     }
     {
       error ? <Error /> : <Content />;
     }
     {
       /* Ternary */
     }
     {
       items.length === 0 && <EmptyState />;
     }

     // 3. Fragment (<> </>) - no extra div
     <>
       <h1>Title</h1>
       <p>Text</p>
     </>;
     ```

   - **Props Flow Pattern (15 phút):**

     ```jsx
     // Parent: Props down, Events up
     function ProductList() {
       const [liked, setLiked] = useState([]);

       const handleLike = (id) => {  // Event handler ở Parent
         setLiked([...liked, id]);
       };

       return products.map(p => (
         <ProductCard
           key={p.id}
           product={p}  {/* Props down */}
           onLike={handleLike}  {/* Event callback down */}
         />
       ));
     }

     // Child: Chỉ nhận props và gọi callback
     function ProductCard({ product, onLike }) {
       return (
         <button onClick={() => onLike(product.id)}>  {/* Event up */}
           Like
         </button>
       );
     }
     ```

     > _"Rule: State ở Parent, Child chỉ nhận props và gọi callback. Không đặt state ở Child nếu Parent cần biết."_

   - **Controlled Input Pattern (15 phút - CRITICAL):**

     ```jsx
     // Controlled: React kiểm soát value
     function SearchBox() {
       const [search, setSearch] = useState("");

       return (
         <input
           value={search} // Controlled by React state
           onChange={(e) => setSearch(e.target.value)}
         />
       );
     }

     // Uncontrolled: DOM giữ value (dùng ref)
     function UncontrolledForm() {
       const inputRef = useRef();

       const handleSubmit = () => {
         console.log(inputRef.current.value); // Read từ DOM
       };

       return <input ref={inputRef} />;
     }
     ```

     > _"Rule of Thumb: Controlled cho input cần validation realtime. Uncontrolled cho form đơn giản (RHF dùng uncontrolled under hood)."_

   - Mock products array
   - Search/filter local (controlled input)
   - Like/Favorite toggle (UI state)

3. **🧪 Checkpoint**

   - Render list 20 items, có filter, không warning key

4. **🚨 Red Flags**

   - Dùng for-loop trong JSX
   - Mutate state trực tiếp
   - Đặt state ở Child khi Parent cần biết (state phải ở common ancestor)
   - Dùng Controlled input không có onChange (React warning)
   - Quên `key` khi render list (performance issue)

5. **🧠 Mini Foundation: React Rendering (15 phút)**

   - **Question:** "Khi nào một component render lại?"
   - **Answer:**
     1. State thay đổi (`setCount`).
     2. Props thay đổi.
     3. Cha render -> Con render (mặc định).
   - **Demo:** `console.log('render')` để chứng minh.
   - **Parent/Child Re-render Flow:**

     ```jsx
     function Parent() {
       const [count, setCount] = useState(0);
       console.log("Parent render");

       return (
         <div>
           <button onClick={() => setCount(count + 1)}>Count: {count}</button>
           <Child name="John" /> {/* Child cũng re-render */}
         </div>
       );
     }

     function Child({ name }) {
       console.log("Child render"); // Chạy mỗi khi Parent render
       return <p>Hello {name}</p>;
     }
     ```

     > _"Quan trọng: Parent re-render → tất cả Child re-render (mặc định)."_

   - **Concept:** Virtual DOM Diffing (React không update DOM thật ngay, mà tính toán sự khác biệt).
   - **❗ Rule of Thumb (Production):** Hạn chế derived state, chỉ dùng state cho data thay đổi theo thời gian và kích hoạt render.

---

### ✅ CHECKLIST TUẦN 1: REACT CORE CONCEPTS

> **Giáo viên bắt buộc hỏi học viên TRƯỚC KHI chuyển sang Tuần 2:**

**JSX & Component Patterns:**

- [ ] Viết được component nhận `children` prop
- [ ] Render conditional với `&&` và ternary `? :`
- [ ] Biết khi nào dùng Fragment `<> </>` thay vì `<div>`
- [ ] Hiểu JSX expression `{}` chỉ nhận expression, không nhận if/else

**Props & Events:**

- [ ] Truyền props từ Parent xuống Child
- [ ] Truyền callback function từ Parent để Child gọi (Events up)
- [ ] Hiểu "Props down, Events up" pattern
- [ ] Không mutate props trong Child

**State & Re-render:**

- [ ] Dùng `useState` đúng (không mutate trực tiếp)
- [ ] Hiểu khi nào component re-render (state/props thay đổi, parent render)
- [ ] Hiểu Parent render → Child cũng render (mặc định)
- [ ] Biết Controlled input (value + onChange) vs Uncontrolled (ref)

**List Rendering:**

- [ ] Render list với `.map()` và `key` đúng
- [ ] Hiểu tại sao `key` quan trọng (React diffing)
- [ ] Không dùng index làm key nếu list có thể thay đổi thứ tự

**TypeScript cho React:**

- [ ] Định nghĩa Props interface
- [ ] Type cho event handlers (`React.ChangeEvent`, `React.FormEvent`)
- [ ] Type cho `useState` với generics: `useState<User | null>(null)`

**🚨 Anti-patterns phải tránh:**

- ❌ Mutate state trực tiếp: `state.push()` → dùng `setState([...state, newItem])`
- ❌ Đặt state ở Child khi Parent cần biết → state phải ở common ancestor
- ❌ Quên `key` khi render list
- ❌ Dùng `any` type cho props/events
- ❌ Controlled input không có `onChange` handler

**🎯 Checkpoint Exercise (10 phút):**

Code challenge: Viết `TodoList` component:

- Input controlled để add todo
- Render list todos với delete button
- Parent giữ state `todos`, Child chỉ nhận props và callbacks
- Có empty state khi `todos.length === 0`

> **Pass criteria:** Code chạy, không warning, follow "Props down Events up" pattern.

---

## 📅 TUẦN 2: ROUTER + LAYOUT

**Checklist Tuần 2:**

- [ ] Setup React Router v6 (layout + outlet)
- [ ] Có protected routes logic (UI only, chưa có token thật)

### BUỔI 3: ROUTER V6 + LAYOUT PATTERN

> **Tài liệu:** `session-03-router-layout`

🎬 **OPENING SCRIPT (5 phút)**  
_"App thật không có 1 trang. Hôm nay ta dựng routing + layout để chuẩn bị nhét Auth và API vào."_

1. **🎯 Mục tiêu**

   - Router v6, nested routes, `Outlet`
   - Public pages: `/`, `/login`, `/register`
   - Private pages: `/me`, `/upload`

2. **📚 Live Coding**

   - `AppRouter` + `MainLayout`
   - Nav links, active styles

3. **🧪 Checkpoint**
   - Điều hướng mượt, không reload

---

### BUỔI 4: PROTECTED ROUTES LOGIC + UI STATES

> **Tài liệu:** `session-04-protected-routes`

🎬 **OPENING SCRIPT (5 phút)**
_"Bảo vệ route là bước 1 của auth. Chưa login thì không cho vào /me. Hôm nay ta làm logic protected routes, và chuẩn hóa UI states: Loading/Error/Empty."_

1. **🎯 Mục tiêu**

   - Component `RequireAuth` (mock token check)
   - Redirect-back pattern
   - UI states: Loading, Error, Empty (chuẩn hóa từ đầu)
   - **Mini Foundation (15p):** useEffect Rules of Thumb

2. **📚 Live Coding**

   - `RequireAuth` wrapper
   - Mock check: `const isAuthed = false` (hardcode)
   - Protected routes cho `/me`, `/upload`
   - Loading component, Error component, Empty component templates

3. **🚨 Red Flags**

   - Flash UI private content trước khi redirect

4. **🧠 Mini Foundation: useEffect Rules (15 phút)**
   - **Rule 1:** Effect = Sync với external system (API, DOM, Timer).
   - **Rule 2:** Dependency Array `[]` = Run once (Mount).
   - **Rule 3:** Dependency `[prop, state]` = Run khi deps đổi.
   - **Red Flag:** Không dùng Effect để validate form hoặc tính toán derived state.
   - **Demo:** Infinite loop khi quên dependency.
   - **❗ Rule of Thumb (Production):** Dependency array luôn phải đầy đủ (eslint-plugin-react-hooks sẽ nhắc). Đừng bao giờ nói dối React về deps.

---

## 📅 TUẦN 3: ZUSTAND AUTH + UI LAYER

**Checklist Tuần 3:**

- [ ] Setup Zustand store cho auth tokens
- [ ] Hiểu token persistence (localStorage)
- [ ] Protected routes dựa trên Zustand
- [ ] Biết dùng Tailwind v4 + shadcn/ui

### BUỔI 5: ZUSTAND AUTH STORE + TOKEN PERSISTENCE

> **Tài liệu:** `session-05-zustand-auth`

🎬 **OPENING SCRIPT (5 phút)**
_"Trước khi gọi API, ta cần nơi lưu token. Zustand là global state nhẹ, dùng cho auth tokens + UI global. Hôm nay ta xây token store để chuẩn bị gắn API thật ở tuần sau."_

1. **🎯 Mục tiêu**

   - Zustand store: `accessToken`, `refreshToken`, `isAuthed`
   - Token persistence (localStorage)
   - **Selector Pattern (CRITICAL):** Tránh re-render không cần thiết
   - Component `RequireAuth` dùng token thật

2. **📚 Live Coding**

   - Tạo `auth.store.ts`: `setTokens`, `clearTokens`, `selectIsAuthed`
   - LocalStorage persistence (key: `shoppingCardFE.tokens`)

   - **Selector Pattern (20 phút - CRITICAL):**

     ```ts
     // ❌ SAI - Lấy toàn bộ store → re-render khi bất kỳ field nào đổi
     const store = useAuthStore();
     const isAuthed = !!store.accessToken;

     // ✅ ĐÚNG - Chỉ subscribe field cần thiết
     const isAuthed = useAuthStore((state) => !!state.accessToken);
     const accessToken = useAuthStore((state) => state.accessToken);

     // ✅ ĐÚNG - Multiple fields với shallow compare
     import { shallow } from "zustand/shallow";
     const { accessToken, refreshToken } = useAuthStore(
       (state) => ({
         accessToken: state.accessToken,
         refreshToken: state.refreshToken,
       }),
       shallow
     );
     ```

     > _"❗ Rule of Thumb: Luôn dùng selector. Chỉ lấy fields bạn cần. Tránh `useAuthStore()` không tham số."_

   - Update `RequireAuth` dùng `selectIsAuthed` từ store
   - Mock login: set token giả để test

3. **🚨 Red Flags**
   - Nhét server data vào Zustand (sai vai trò - dùng React Query)
   - Reload trang mất token (quên persistence)
   - **Dùng `useAuthStore()` không selector → re-render không cần thiết**

---

### BUỔI 6: TAILWIND V4 + SHADCN/UI PRACTICAL

> **Tài liệu:** `session-06-ui-practical`

🎬 **OPENING SCRIPT (5 phút)**  
_"UI không cần quá đẹp, nhưng phải sạch và nhất quán. Tailwind v4 (stable) giúp ta làm nhanh và chuẩn. shadcn/ui là component template, không phải blackbox."_

1. **🎯 Mục tiêu**

   - Setup Tailwind v4 (Zero-runtime, CSS-first)
   - Config `index.css` với Tailwind v4 directives (@theme, @apply)
   - shadcn/ui: Button, Input, Card, Dialog, Sonner (toast)
   - UI states: Loading / Error / Empty

2. **📚 Live Coding**

   - Setup Tailwind v4 với PostCSS
   - Config `index.css` với Tailwind v4 directives (@theme, @apply)
   - shadcn/ui init + add components: `button`, `input`, `card`, `dialog`, `sonner`
   - Login/Register UI layout
   - Component: `PageContainer`, `Card`, `FormField`
   - Toast (Sonner) - note: deprecated `toast` component
   - Dialog confirm logout
   - Skeleton/Loading UI

3. **🧪 Checkpoint**
   - Có toast hiển thị success/error
   - UI có spacing/typography rõ ràng

---

## 📅 TUẦN 4: AXIOS LAYER + FORMS & VALIDATION

**Checklist Tuần 4:**

- [ ] Tạo Axios instance + baseURL + interceptors
- [ ] Attach token từ Zustand vào headers
- [ ] Error normalization (422 field errors, 401, 500)
- [ ] Refresh token SIMPLIFIED (no single-flight)
- [ ] RHF dùng đúng (submit, errors)
- [ ] Zod validate trước khi gọi API

### BUỔI 7: AXIOS LAYER + INTERCEPTORS (SIMPLIFIED REFRESH)

> **Tài liệu:** `session-07-axios-interceptors`

🎬 **OPENING SCRIPT (5 phút)**
_"Gọi được API là level thấp. Level production là: code gọi API có cấu trúc, lỗi được chuẩn hóa, UI hiển thị rõ ràng. Interceptor là kỹ năng bắt buộc. Hôm nay ta làm refresh token đơn giản, KHÔNG dùng single-flight pattern phức tạp."_

1. **🎯 Mục tiêu**

   - `apiClient` (baseURL, timeout, headers)
   - Normalize error shape (422/401/500)
   - Service layer: `usersApi`
   - Request interceptor: attach token từ Zustand
   - Response interceptor: bắt 401 để refresh token
   - **SIMPLIFIED:** Refresh token retry 1 time (KHÔNG dùng single-flight pattern)

2. **📚 Live Coding**

   - Tạo `lib/http/apiClient.ts`
   - Proxy `/api` -> `http://localhost:3000` (Vite config)
   - Request interceptor:
     ```ts
     config.headers.Authorization = `Bearer ${
       useAuthStore.getState().accessToken
     }`;
     ```
   - Response interceptor (SIMPLIFIED - no queue/single-flight):

     ```ts
     if (status === 401 && !config.__isRetry) {
       try {
         const refreshToken = useAuthStore.getState().refreshToken;
         const { data } = await apiClient.post("/users/refresh-token", {
           refresh_token: refreshToken,
         });
         useAuthStore.getState().setTokens(data.result);

         // Retry once
         config.__isRetry = true;
         config.headers.Authorization = `Bearer ${data.result.access_token}`;
         return apiClient(config);
       } catch (error) {
         // Refresh fail -> logout
         useAuthStore.getState().clearTokens();
         window.location.href = "/login";
         throw error;
       }
     }
     ```

   - Error normalization cho 422/401/500/network
   - Tạo `lib/api/users.api.ts`: register, login, logout, getMe, updateMe, changePassword

   - **⚠️ Backend Response Inconsistency (10 phút - CRITICAL WARNING):**

     > _"Backend của ta KHÔNG nhất quán. FE phải normalize!"_

     ```ts
     // Backend inconsistent:
     // /login trả:  { message, result: { access_token, refresh_token } }
     // /register trả: { msg, data: { access_token, refresh_token } }
     // /me trả: { message, result: { user } }

     // ❗ FE phải normalize trong service layer:
     export const usersApi = {
       async login(credentials: LoginDto) {
         const { data } = await apiClient.post("/users/login", credentials);
         // Normalize: luôn trả { accessToken, refreshToken }
         return {
           accessToken: data.result.access_token,
           refreshToken: data.result.refresh_token,
         };
       },

       async register(userData: RegisterDto) {
         const { data } = await apiClient.post("/users/register", userData);
         // Backend dùng 'data' thay vì 'result'
         return {
           accessToken: data.data.access_token,
           refreshToken: data.data.refresh_token,
         };
       },
     };
     ```

     **❗ Rule of Thumb:**

     - **KHÔNG tin backend 100%** - luôn normalize response trong service layer
     - **KHÔNG dùng raw API response** trực tiếp trong component
     - **TypeScript interface** cho Frontend khác với Backend
     - **Document inconsistencies** trong code comments

3. **🚨 Red Flags**
   - Axios gọi thẳng trong component
   - UI không có loading/error
   - Refresh loop (quên check `__isRetry`)
   - Cố implement single-flight (overkill cho khóa này)

---

### BUỔI 8: FORMS (RHF + ZOD) - MAP ĐÚNG BE

> **Tài liệu:** `session-08-form-rhf-zod`

🎬 **OPENING SCRIPT (5 phút)**
_"Form là nơi học viên bắt đầu sai: validate rối, submit bừa, UI không báo lỗi. Ta chuẩn hóa bằng RHF + Zod. Chúng ta validate TRƯỚC KHI gửi API để giảm lỗi và để UI nói chuyện rõ ràng với user."_

1. **🎯 Mục tiêu**

   - Build login/register form bằng RHF
   - Zod schema: `confirmed_password`, `date_of_birth` ISO
   - Show field errors (422 mapping)
   - **Pain-Driven Demo (10p):** Controlled vs Uncontrolled Forms

2. **📚 Live Coding**

   - Login form: `{ email, password }`
     ```tsx
     const loginSchema = z.object({
       email: z.string().email("Email không hợp lệ"),
       password: z.string().min(6, "Password tối thiểu 6 ký tự"),
     });
     ```
   - Register form: `{ name, email, password, confirmed_password, date_of_birth }`
     ```tsx
     const registerSchema = z
       .object({
         name: z.string().min(1, "Name là bắt buộc"),
         email: z.string().email(),
         password: z.string().min(8, "Password tối thiểu 8 ký tự"),
         confirmed_password: z.string(),
         date_of_birth: z.string(), // YYYY-MM-DD format
       })
       .superRefine((data, ctx) => {
         if (data.password !== data.confirmed_password) {
           ctx.addIssue({
             code: "custom",
             path: ["confirmed_password"],
             message: "Password không khớp",
           });
         }
       });
     ```
   - Transform `date_of_birth` sang ISO8601 trước submit:
     ```ts
     const formattedData = {
       ...data,
       date_of_birth: new Date(data.date_of_birth).toISOString(),
     };
     ```
   - Map 422 errors từ backend:
     ```ts
     if (error.status === 422 && error.errors) {
       Object.entries(error.errors).forEach(([field, message]) => {
         form.setError(field as any, { message: message as string });
       });
     }
     ```
   - UI: FieldError component dưới input
   - Button disable khi `isSubmitting`

3. **🔥 Pain-Driven Demo: Why RHF? (10 phút - ĐẦU GIỜ)**

   - **Demo:** Code 1 form React thuần (Controlled) với 3 inputs + validation thủ công.
   - **Pain point:**
     - Re-render logic từng ký tự (`console.log('render')`).
     - Validation logic rối rắm (`if (!email) ...`).
     - Handler `onChange` lặp lại.
   - **Solution:** RHF (Uncontrolled under hood) -> Không re-render, code gọn, performance cao.
   - **❗ Rule of Thumb (Production):** Luôn dùng Uncontrolled Form (RHF) cho form > 3 inputs hoặc form có validation phức tạp. Controlled chỉ dành cho input đơn lẻ (Search, Filter).

4. **🧪 Checkpoint**
   - Register form chỉ submit khi schema pass
   - 422 show đúng field errors dưới input
   - Button disable khi pending

---

## 📅 TUẦN 5: TANSTACK QUERY (SERVER STATE)

**Checklist Tuần 5:**

- [ ] useQuery cho `/users/me`
- [ ] useMutation cho login/register/logout/update/change password
- [ ] invalidateQueries đúng
- [ ] UI: loading/error/empty states

### BUỔI 9: QUERY (FETCH) CHUẨN LOADING/ERROR/CACHE

> **Tài liệu:** `session-09-react-query`

🎬 **OPENING SCRIPT (5 phút)**
_"Server state không phải UI state. React Query sẽ lo cache/refetch/retry. Việc của ta là UI states đúng: loading, error, empty."_

1. **🎯 Mục tiêu**

   - `/me` bằng `useQuery`
   - UI: loading/error/empty

2. **📚 Live Coding**

   - Setup `QueryClient` + `QueryClientProvider`
   - Config:
     ```ts
     new QueryClient({
       defaultOptions: {
         queries: {
           refetchOnWindowFocus: false,
           retry: 1,
         },
       },
     });
     ```
   - Query `/users/me`:

     #### 🔥 Pain-Driven Demo: Manual Fetching (15 phút - ĐẦU GIỜ)

     > **Mục tiêu:** Chứng minh `useEffect` fetching quá cực khổ -> Cần React Query.

     **Code thử (Demo Only - Don't use):**

     ```tsx
     // The "Painful" Way
     useEffect(() => {
       setLoading(true);
       api
         .get("/users/me")
         .then((res) => setData(res.data))
         .catch((err) => setError(err))
         .finally(() => setLoading(false));
     }, []);
     ```

     **Hỏi học viên:** Cache đâu? Retry đâu? Race condition? StrictMode chạy 2 lần?
     => **Solution:** React Query (1 dòng, full features).

     #### ✅ The "Right" Way (React Query)

     ```tsx
     const meQueryKey = ["me"] as const;

     const { data, isLoading, error } = useQuery({
       queryKey: meQueryKey,
       queryFn: () => usersApi.getMe(),
       enabled: !!accessToken, // chỉ fetch khi có token
     });

     // UI states
     if (isLoading) return <Skeleton />;
     if (error) return <ErrorState error={error} />;
     if (!data) return <EmptyState />;
     return <ProfileView user={data} />;
     ```

   - React Query DevTools (optional)

3. **🧪 Checkpoint**
   - Query chạy đúng, cache hoạt động
   - UI có đủ 3 states: loading, error, success

---

### BUỔI 10: MUTATION + INVALIDATION + AUTH FLOW

> **Tài liệu:** `session-10-mutations`

🎬 **OPENING SCRIPT (5 phút)**
_"Mutation là nơi học viên hay sai nhất: không disable nút, không toast, không invalidate cache. Hôm nay ta chuẩn hóa toàn bộ auth mutations."_

1. **🎯 Mục tiêu**

   - login/register/logout/updateMe/changePassword bằng `useMutation`
   - invalidate `me` sau login/update
   - Clear cache sau logout

2. **📚 Live Coding**

   - `loginMutation`:
     ```tsx
     const loginMutation = useMutation({
       mutationFn: usersApi.login,
       onSuccess: (data) => {
         // Save tokens
         useAuthStore.getState().setTokens({
           accessToken: data.result.access_token,
           refreshToken: data.result.refresh_token,
         });
         // Invalidate me query để fetch lại
         queryClient.invalidateQueries({ queryKey: ["me"] });
         // Toast + redirect
         toast.success("Đăng nhập thành công");
         navigate("/me");
       },
       onError: (error) => {
         toast.error(error.message || "Đăng nhập thất bại");
       },
     });
     ```
   - `registerMutation`: tương tự login
   - `updateMeMutation`:
     ```tsx
     const updateMeMutation = useMutation({
       mutationFn: usersApi.updateMe,
       onSuccess: () => {
         queryClient.invalidateQueries({ queryKey: ["me"] });
         toast.success("Cập nhật thành công");
       },
     });
     ```
   - `logoutMutation` (QUAN TRỌNG - bám backend):
     ```tsx
     const logoutMutation = useMutation({
       mutationFn: () => {
         const refreshToken = useAuthStore.getState().refreshToken;
         return usersApi.logout({ refresh_token: refreshToken });
       },
       onSuccess: () => {
         // Clear tokens
         useAuthStore.getState().clearTokens();
         // Clear ALL query cache
         queryClient.clear();
         // Redirect
         navigate("/login");
         toast.success("Đăng xuất thành công");
       },
     });
     ```
   - `changePasswordMutation`:
     ```tsx
     const changePasswordMutation = useMutation({
       mutationFn: usersApi.changePassword,
       onSuccess: () => {
         toast.success("Đổi mật khẩu thành công");
         // Không cần invalidate me
       },
     });
     ```
   - UI: Button disable khi `isPending`

3. **🧪 Checkpoint**

   - Login xong vào `/me` thấy data đúng
   - Logout clear cache + redirect
   - Button disable khi pending
   - Toast hiển thị đúng

4. **🚨 Red Flags**
   - Mutation success nhưng không invalidate → UI stale
   - Không disable button → spam API
   - Logout không clear cache → còn data cũ

---

## 📅 TUẦN 6-8: PROJECT SPRINT (3 TUẦN)

**Checklist Project:**

- [ ] Auth flows: register/login/logout/get me/update me/change password
- [ ] Upload image (1 file) + preview URL
- [ ] Protected routes + redirect-back
- [ ] UI: loading/error/empty + toast đầy đủ
- [ ] Refresh token: basic retry (1 time)
- [ ] Code structure sạch (pages/components/lib/api)

**Total Sessions:** Buổi 11-17 (7 buổi = 14 giờ)

---

### BUỔI 11: PROJECT SETUP + AUTH CORE

> **Tài liệu:** `session-11-project-setup`

🎬 **OPENING SCRIPT (5 phút)**
_"3 tuần tiếp theo, ta build app thật. Không phải demo nhỏ, mà là FE chạy với BE thật. Hôm nay: setup project + login/register."_

**Scope buổi:**

- Setup project skeleton:
  - Vite + Router + Providers (QueryClient, Zustand init)
  - Folder structure: `pages/`, `components/`, `lib/api/`, `lib/http/`, `stores/`
- Implement login/register forms (RHF + Zod)
- Connect với backend qua axios
- Test login → save tokens → redirect `/me`

**Deliverables:**

- Login/Register UI + logic hoạt động
- Token lưu vào Zustand + localStorage

---

### BUỔI 12: PROFILE FLOWS (GET ME + UPDATE ME + CHANGE PASSWORD)

> **Tài liệu:** `session-12-profile-flows`

🎬 **OPENING SCRIPT (5 phút)**
_"Hôm nay ta làm profile features: xem profile, sửa profile, đổi password. Đây là nơi thực hành React Query invalidation chuẩn nhất."_

**Scope buổi:**

- Query `/users/me` với loading/error states
- Page `/me`:
  - Show profile info
  - Link to UpdateMe page
  - Link to ChangePassword page
- Update me form (RHF + Zod)
  - Fields: name, date_of_birth, bio, location, website, username
  - onSuccess: invalidate `['me']`
- Change password form
  - Fields: old_password, password, confirm_password
  - onSuccess: toast, không invalidate me
- Logout button:
  - Call `/users/logout` với header + body
  - Clear tokens + query cache
  - Redirect `/login`

**Deliverables:**

- `/me` page hiển thị profile
- Update profile hoạt động + UI update
- Change password hoạt động
- Logout chuẩn backend

---

### BUỔI 13: MEDIA UPLOAD + REFRESH TOKEN TESTING

> **Tài liệu:** `session-13-media-refresh`

🎬 **OPENING SCRIPT (5 phút)**
_"Upload là case thực chiến: multipart, preview. Refresh token là kỹ năng production quan trọng: khi access token hết hạn, app không chết."_

**Scope buổi:**

- Upload image (1 file) với FormData:
  ```tsx
  const uploadImage = async (file: File) => {
    const formData = new FormData();
    formData.append("image", file);
    return mediasApi.uploadImage(formData);
  };
  ```
- Preview URL trả về:
  ```tsx
  <img src={`http://localhost:3000${url}`} />
  ```
- Page `/upload`:
  - Input file select
  - Preview local với `URL.createObjectURL(file)`
  - Upload button
  - Preview uploaded image từ static URL
- Test refresh token flow:
  - Manually expire access token (change 1 character)
  - Trigger API call → 401 → auto refresh → retry success
  - Debug nếu có 401 loop

**Deliverables:**

- Upload image hoạt động
- Preview URLs render được
- Refresh token retry đúng (test bằng cách break token)

---

### BUỔI 14: UI POLISH & UX STATES

> **Tài liệu:** `session-14-ui-polish` **[SPLIT FROM ORIGINAL SESSION 14]**

🎬 **OPENING SCRIPT (5 phút)**
_"Hôm nay ta không viết tính năng mới. Ta chỉ làm một việc: Polish - Trang điểm cho ứng dụng. Loading/Error/Empty states đầy đủ."_

**Scope buổi:**

- **UI Polish (90 phút):**
  - Loading states:
    - Button spinner (inline loading)
    - Skeleton components (page-level)
    - Reusable loading components
  - Error states:
    - Field-level errors (form validation)
    - Global error handler (Axios interceptor)
    - Page-level error components with retry
  - Empty states:
    - No data placeholders
    - Action-oriented empty states
  - **Rule of Thumb:** Skeleton vs Spinner vs Progress Bar
- **Integration (30 phút):**
  - Apply states to all pages
  - Toast consistency
  - Checkpoint testing

**Deliverables:**

- All pages have Loading/Error/Empty states
- No blank screens during async operations
- Professional UX polish

---

### BUỔI 15: DEBUG WORKSHOP

> **Tài liệu:** `session-15-debug-workshop` **[SPLIT FROM ORIGINAL SESSION 14]**

🎬 **OPENING SCRIPT (5 phút)**
_"50% thời gian đi làm là Debug. Nếu giỏi Debug, bạn làm việc nhanh gấp 3 lần người khác. Hôm nay ta học 'nghệ thuật thám tử'."_

**Scope buổi:**

- **Chrome DevTools Mastery (50 phút):**
  - Network Tab deep dive:
    - Reading request/response
    - Status codes, headers, payload
    - Filters and search
  - Console Tab tricks:
    - Live code execution
    - Testing functions
  - Sources Tab & Breakpoints:
    - Setting breakpoints
    - F10/F11 navigation (Step Over/Into)
    - Inspecting variables
    - Call stack analysis
- **React & Query DevTools (30 phút):**
  - React DevTools: Component tree, props/state inspection
  - React Query DevTools: Cache inspection, query status
- **Debugging Scenarios (30 phút):**
  - Scenario 1: Flickering UI
  - Scenario 2: Silent failure
  - Scenario 3: Infinite spinner
  - Scenario 4: Double fetch
- **Debug Flow Checklist (10 phút):**
  - Systematic debugging workflow
  - Anti-patterns to avoid

**Deliverables:**

- Students can use DevTools independently
- Can debug API errors in <5 minutes
- Follow systematic debug workflow

---

### BUỔI 16: INTEGRATION TESTING + BUG FIXES

> **Tài liệu:** `session-16-integration-testing` **[RENUMBERED FROM 15]**

🎬 **OPENING SCRIPT (5 phút)**
_"Hôm nay ta chạy toàn bộ flow từ đầu đến cuối, tìm bug, sửa bug. Checklist: register → login → me → update → upload → change password → logout."_

**Scope buổi:**

- Run full testing checklist:
  ```
  [ ] Register ok (form validate + call API)
  [ ] Register fail (422 show field errors)
  [ ] Login ok → redirect /me
  [ ] Login fail → show error
  [ ] Get me query works
  [ ] Update me works + invalidates query
  [ ] Upload image works + preview
  [ ] Change password works
  [ ] Logout calls API (header + body) → clears cache → redirects
  [ ] Protected route: /me khi chưa login → redirect /login
  [ ] Redirect-back: login xong quay lại /me
  [ ] Refresh token: 401 → auto refresh → retry
  ```
- Fix bugs found

**🛑 BEHAVIORAL RUBRIC (Tiêu chí chấm điểm tư duy):**

> Ngoài việc tính năng chạy đúng, code phải "sạch" theo standard.

| **Lỗi Vi Phạm (Auto Fail/Trừ nặng)** | **Lý Do**                                      |
| :----------------------------------- | :--------------------------------------------- |
| ❌ Fetch API trong component         | Anti-pattern, khó test/scale.                  |
| ❌ Dùng `useState` lưu server data   | Duplicate state với React Query -> Stale data. |
| ❌ Mutation xong không Invalidate    | UI không cập nhật dữ liệu mới.                 |
| ❌ Không Loading/Error state         | Bad UX.                                        |
| ❌ Hardcode API URL                  | Cannot deploy.                                 |
| ❌ Commit file `.env`                | Security risk.                                 |

- Peer code review:
  - Checklist: có axios layer? có loading states? có error handling?
- Write README:

  ```md
  # Shopping Card FE

  ## Setup

  1. Copy `env.example` to `.env`
  2. npm install
  3. npm run dev

  ## Backend

  Ensure backend is running at http://localhost:3000

  ## Features

  - Login/Register
  - Get Me / Update Me
  - Change Password
  - Upload Image
  - Logout
  ```

**Deliverables:**

- Full testing passed
- Bugs fixed
- README documented

---

### BUỔI 17: DEPLOYMENT + FINAL REVIEW + DEMO

> **Tài liệu:** `session-17-deployment` **[RENUMBERED FROM 16]**

🎬 **OPENING SCRIPT (5 phút)**
_"Code chạy local là 50%. Deploy được lên internet cho cả thế giới dùng mới là 100%. Hôm nay ta deploy lên Vercel, config biến môi trường, và tổng kết khóa học."_

**Scope buổi:**

- **Calibration & Build:**
  - Audit perf với Lighthouse (cơ bản)
  - Split chunks logic (nếu bundle quá lớn)
  - Config `vite.config.ts` build settings
- **Deployment (Vercel):**
  - Connect GitHub repo
  - Import Project
  - **IMPORTANT:** Config Environment Variables (`VITE_API_URL`) trên Vercel
  - Redeploy & Test production URL
- **Student demos** (mỗi người 5 phút)
- **Course Wrap-up & Roadmap**

**Deliverables:**

- App running on production URL (vercel.app)
- Course certificate

---

## 🧪 ĐÁNH GIÁ CUỐI KHÓA (RUBRIC)

| Level           | Tiêu chí                                                              | Điểm      |
| :-------------- | :-------------------------------------------------------------------- | :-------- |
| **Fail**        | Auth/API lỗi, không có loading/error, code rối                        | < 5.0     |
| **Pass**        | Chạy đúng core flows (login/me/logout), router + form ok              | 5.0 - 7.0 |
| **Merit**       | Query/mutation chuẩn, invalidate đúng, UI ổn, có error handling       | 7.0 - 8.5 |
| **Distinction** | Refresh retry ổn, structure sạch, README tốt, code review đạt yêu cầu | 8.5 - 10  |

---

## 🎓 HOW TO TEACH THIS COURSE (For Instructors)

### ⚡ Nguyên tắc vàng

1. **Trọng tâm API handling**
   - Mỗi lần call API phải hỏi: loading/error đâu? toast đâu? retry/refresh đâu?
2. **Không code hộ học viên**
   - Hướng dẫn logic + debug, không gõ hộ.
3. **Giữ ranh giới trách nhiệm**
   - Server state → React Query
   - UI/global → Zustand
   - Local → useState

### 🎯 Quản lý thời gian mỗi buổi

- Opening Script: 5 phút
- WHY/WHEN: 15 phút
- Live Coding: 60 phút
- Học viên code: 20 phút
- Checkpoint: 10 phút

### 🔑 Success Factors

1. **Pre-assessment** (trước khóa):
   - Quiz React basic concepts
   - Xác nhận học viên đã học Node.js + TypeScript
2. **Office hours** (1 giờ/tuần):
   - Giảng viên/TA giải đáp 1-1
3. **Checkpoint assignments** (sau tuần 2, 4):
   - Bài tập nhỏ bắt buộc nộp để catch stragglers sớm
4. **Code review sessions**:
   - Peer review với checklist cụ thể

---

---

## ❌ COMMON ANTI-PATTERNS (HỌC XONG PHẢI TRÁNH)

> **Dính lỗi này = Trừ điểm nặng**

- ❌ Gọi `axios` trực tiếp trong Component (ko qua Service layer).
- ❌ Dùng `useEffect` fetch data (trừ bài demo Session 9).
- ❌ Nhét Server Data vào Zustand Store.
- ❌ Mutation success nhưng quên Invalidate Queries (UI cũ).
- ❌ Không disable Button khi `isPending` (Spam API).
- ❌ Hardcode URL (`http://localhost:3000`) thay vì Env variable.
- ❌ Logout nhưng quên `queryClient.clear()`.

---

## ❓ WHY NOT...? (GIẢI ĐỘC TÂM LÝ)

1. **Tại sao không Redux?** -> Quá nhiều boilerplate. Zustand + React Query xử lý 99% cases nhẹ nhàng hơn.
2. **Tại sao không Next.js?** -> Đây là khóa Client-side Rendering (SPA). Bạn cần vững React Core trước khi học Server-side.
3. **Tại sao không fetch bằng useEffect?** -> Không có Cache, không Deduping, không Retry tự động, quản lý Loading/Error thủ công rất cực.

---

## 🚧 WHAT YOU DON'T KNOW YET (ĐỊNH HƯỚNG SAU KHÓA)

> React thế giới rất rộng, khóa này chưa dạy:

- **Next.js & SSR/RSC**: Server Components.
- **Advanced Caching**: Stale-while-revalidate sâu hơn.
- **Micro-frontends**: Tách app theo chiều dọc.
- **Complex Permissions**: RBAC level từng field.
- **Performance**: `useTransition`, `Suspense` deep dive.

**KẾT THÚC GIÁO TRÌNH.**

---

## ❌ ANTI-GOALS (THIS COURSE DOES NOT COVER)

> Những thứ **KHÔNG** dạy để tránh mất focus:

- **Redux / Redux Toolkit**: Đã cũ hoặc quá overkill. Dùng Zustand + React Query là đủ 99% case.
- **Next.js / SSR**: Đây là khóa Client-side Rendering (SPA). Next.js là khóa sau.
- **Advanced Performance (useTransition, Suspense)**: Chỉ dạy cơ bản nếu có thời gian (memo, useMemo).
- **Micro-frontends**: Quá nâng cao và hiếm gặp.

