# LESSON PLAN: SESSION 01 - SETUP + TYPE SCRIPT SURVIVAL KIT

## 1️⃣ SESSION OVERVIEW
- **Title:** Setup Environment, Component Thinking, and TypeScript Survival Kit
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will set up a production-ready React environment (Vite + Tailwind v4 + TS) and master the 20% of TypeScript used in 80% of React tasks.
- **Outcome:** A running "Hello World" app with Absolute Imports, Tailwind v4 configured, basic `useState` usage with TypeScript, and no errors.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào mọi người. Các bạn đã học xong Backend MERN, đã biết viết API trả về JSON. Nhưng Frontend không chỉ là 'hiện JSON lên màn hình'. Frontend là về việc TỔ CHỨC UI sao cho không phải copy-paste code._

_Nếu HTML/CSS thuần là 'Xây nhà đúc' (đổ bê tông cứng đơ, muốn sửa phải đập đi xây lại), thì React là 'Lắp LEGO'. Chúng ta tạo ra các khối gạch nhỏ (Component) như nút bấm, cái ô nhập liệu... rồi lắp ghép lại thành ngôi nhà._

_Hôm nay ta sẽ làm 2 việc quan trọng nhất mà nếu sai ngay từ đầu thì dự án sẽ nát:_
1.  _Setup môi trường chuẩn Production (Vite, Tailwind v4, Absolute Imports)._
2.  _Học 'Bộ công cụ sinh tồn' TypeScript cho React. React TS khác Backend TS, các bạn cần biết cách định nghĩa Props và Events."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Nếu các bạn setup React + TypeScript sai ngay từ buổi đầu, thì tới tuần 4–5 khi gọi API, code sẽ đỏ lòm và lúc đó không ai nhớ lỗi đến từ đâu nữa. Buổi hôm nay là buổi 'chống nợ kỹ thuật' cho cả 2 tháng tới."_

> ⚠️ **LƯU Ý QUAN TRỌNG**
>
> Buổi hôm nay **KHÔNG** yêu cầu các bạn nhớ hết TypeScript.
> Chỉ cần nhớ:
> - Props có type
> - Event có type
> - **useState** căn bản
> - IDE gợi ý là bạn đang đi đúng hướng
>
> Chi tiết type syntax sẽ quen dần qua thực hành. Hôm nay chỉ cần "chạy được, không đỏ".

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ HTML vs Component
- **HTML:** Code lặp lại. Muốn sửa 10 cái nút bấm phải sửa 10 chỗ.
- **Component (React):** Define 1 lần `Button`, dùng 10 nơi. Sửa 1 nơi, cập nhật 10 nơi.

### 🛡️ TypeScript: Frontend vs Backend
- **Backend TS:** Xoay quanh `Request`, `Response`, `Database Model`.
- **Frontend TS:** Xoay quanh `Props` (đầu vào của component) và `Events` (người dùng click/gõ phím).

## 4️⃣ LIVE CODING – STEP BY STEP

> **Instructor Note:** Code chậm, từng dòng. Đừng copy paste block lớn.

### PHASE 1: SETUP VITE & TAILWIND V4 (45 mins)

#### Step 1: Initialize Project
_"Đầu tiên, ta dùng Vite. Nó nhanh hơn Create-React-App 100 lần."_

```bash
# Terminal
npm create vite@latest shopping-card-fe -- --template react-ts
cd shopping-card-fe
npm install

# Cài đặt React 18 (downgrade từ version mặc định nếu cần)
npm install react@18 react-dom@18
```

#### Step 2: Install Tailwind v4 (The New Standard)
_"Tailwind v4 vừa ra mắt, không cần file config js lằng nhằng nữa. Nó dùng CSS-first approach."_

```bash
# Terminal
npm install tailwindcss @tailwindcss/vite
```

#### Step 3: Config Vite & Absolute Imports
_"Đây là bước phân loại Junior vs Pro. Ta không muốn `../../../../components`, ta muốn `@/components`."_

Open `vite.config.ts`:

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite' // Import Tailwind plugin
import path from 'path' // Node.js path module

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    tailwindcss(), // Active Tailwind v4
  ],
  resolve: {
    alias: {
      // Định nghĩa '@' trỏ về folder 'src'
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

#### Step 4: Config TSConfig (Để VSCode hiểu Alias)
_"Vite hiểu rồi, nhưng VSCode cần file này để 'hết đỏ' và gợi ý code."_

Open `tsconfig.app.json` (hoặc `tsconfig.json` tùy version Vite):

```json
{
  "compilerOptions": {
    // ... các options khác giữ nguyên
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"] // Map @/xyz -> src/xyz
    }
  }
}
```

#### Step 5: Setup CSS Entry
Open `src/index.css`:
_"Xóa hết code cũ. Chỉ import đúng 1 dòng này để kích hoạt Tailwind."_

```css
@import "tailwindcss";
```

#### Step 6: Verify Structure
Create folders in `src`:
- `src/components`
- `src/pages`
- `src/lib`

Run app:
```bash
npm run dev
```

#### Step 7: Verify Tailwind (CRITICAL CHECKPOINT)
Open `src/App.tsx` (hoặc sửa code có sẵn) để test:

```tsx
<h1 className="text-3xl font-bold text-red-500 underline">
  Tailwind v4 is working!
</h1>
```

**Instructor Script:**
_"Các bạn nhìn lên màn hình. Nếu chữ không to, không đỏ, không gạch chân -> Tailwind CHƯA chạy. Đừng code tiếp, hãy sửa ngay. Lỗi này mà để tới buổi 3 mới sửa là ác mộng."_

---

### PHASE 2: TYPESCRIPT SURVIVAL KIT (60 mins)

#### Step 1: Component & Props Typing
Create `src/components/MyButton.tsx`.

_"Component là hàm trả về JSX. Props là tham số của hàm đó."_

```tsx
import React from 'react';

// Define Interface cho Props
// Tại sao interface? Để dễ nhìn và strict typing.
interface MyButtonProps {
  label: string;             // Bắt buộc
  variant?: 'primary' | 'secondary'; // Optional (?)
  onClick?: () => void;      // Hàm xử lý click
}

export default function MyButton({ 
  label, 
  variant = 'primary', // Default value
  onClick 
}: MyButtonProps) {
  
  // Dynamic class based on props
  const baseStyle = "px-4 py-2 rounded font-bold text-white transition";
  const bgStyle = variant === 'primary' ? "bg-blue-500 hover:bg-blue-600" : "bg-gray-500 hover:bg-gray-600";

  return (
    <button 
      className={`${baseStyle} ${bgStyle}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
}
```

#### Step 2: Event Typing (The "Pain" Point)
Create `src/components/MyInput.tsx`.

_"Làm sao biết `e` là type gì? Mẹo: Hover chuột vào `onChange` của thẻ input nguyên bản."_

```tsx
import React, { useState } from 'react';

interface MyInputProps {
  placeholder?: string;
  onValueChange: (value: string) => void; // Callback bắn value lên cha
}

export default function MyInput({ placeholder, onValueChange }: MyInputProps) {
  // Demo Event Type
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    // e.target.value luôn là string
    const val = e.target.value;
    onValueChange(val);
  };

  return (
    <input
      type="text"
      placeholder={placeholder}
      onChange={handleChange}
      className="border p-2 rounded w-full"
    />
  );
}
```
> **📌 RULE OF THUMB: EVENT TYPES**
> _"Quan trọng hơn nhớ máy móc:"_
>
> - **Input/Textarea change:** `React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>`
> - **Form submit:** `React.FormEvent<HTMLFormElement>`
> - **Button click:** `React.MouseEvent<HTMLButtonElement>`
>
> **Mẹo:**
> - Nếu bí type → hover vào prop HTML (ví dụ `onChange` của `<input>`)
> - 90% form chỉ dùng `ChangeEvent` và `FormEvent`
> - Không cần học vẹt, cần biết cách tra


#### Step 3: Use Components in App
Update `src/App.tsx`.

```tsx
import { useState } from 'react';
import MyButton from '@/components/MyButton';
import MyInput from '@/components/MyInput';

function App() {
  // Generics cho useState
  // Tại sao <string>? Để đảm bảo name luôn là string, ko null.
  const [name, setName] = useState<string>(""); 

  const handleAlert = () => {
    alert(`Hello ${name}`);
  };

  return (
    <div className="p-10 flex flex-col gap-4 max-w-md mx-auto mt-10 border rounded bg-gray-50">
      <h1 className="text-2xl font-bold text-center text-blue-600">
        Session 01 Demo
      </h1>
      
      <div className="bg-white p-4 shadow rounded">
        <label className="block mb-2 text-sm font-medium">Your Name:</label>
        <MyInput 
          placeholder="Enter name..." 
          onValueChange={(val) => setName(val)} 
        />
      </div>

      <div className="flex justify-center">
        <MyButton 
          label="Say Hello" 
          variant="primary" 
          onClick={handleAlert} 
        />
      </div>

      <p className="text-center text-gray-500 mt-2">
        Current Name: <span className="font-mono text-black">{name}</span>
      </p>
    </div>
  );
}

export default App;
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **"Cannot find module '@/...'"**:
    *   *Cause:* Quên config `tsconfig.json` hoặc chưa restart VSCode.
    *   *Fix:* Check `paths` trong tsconfig. Restart TS Server (Ctrl+Shift+P > Restart TS Server).
2.  **Type 'string' is not assignable to type 'number'**:
    *   *Cause:* Truyền sai kiểu props.
    *   *Fix:* Đọc lỗi! Props define là number thì phải truyền số `{123}`, không phải chuỗi `"123"`.
3.  **Event `e: any`**:
    *   *Cause:* Lười tra type.
    *   *Fix:* Hover vào prop `onChange` của thẻ HTML để xem React type gợi ý.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ta dùng `interface` cho Props mà không viết thẳng `({ label }: { label: string })`?"*
    *   **A:** Để code clean hơn, dễ tái sử dụng interface đó (export ra dùng chỗ khác), và dễ mở rộng (extends).
2.  **Q:** *"Khi nào dùng `useState<User | null>(null)` thay vì `useState(null)`?"*
    *   **A:** Khi giá trị khởi tạo là `null` nhưng sau này sẽ là Object. Nếu không có Generic, TS sẽ tưởng state luôn luôn là `null`.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Tạo component `UserCard.tsx`.
- **Props:** `name` (string), `email` (string), `isAdmin` (boolean, optional).
- **UI:** Hiển thị tên, email. Nếu `isAdmin` = true thì hiện thêm badge "Admin" màu đỏ.
- **Time:** 10 mins.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Tiếp tục project trên lớp.
1.  Tạo component `ProductCard` nhận props: `title` (string), `price` (number), `image` (string url).
2.  Render 1 danh sách (array) 3 sản phẩm trong `App.tsx` (hardcode data).
3.  Bắt buộc dùng Absolute Import (`@/components/ProductCard`) khi import.

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Học viên hover vào component `MyButton` trong `App.tsx` thấy hiện đầy đủ docs/type props.
- **Fail:** IDE báo đỏ lòm ở các dòng import `@/`.

### ⚠️ SESSION 1 CHƯA LÀM

**Instructor Script:**
> *"Để tránh các bạn về nhà google lung tung, hãy nhớ:*
> *Session 1 **CHƯA** dạy:*
> - *Fetch API / gọi backend*
> - *`useEffect` (side effects)*
> - *React Query / Zustand*
> - *Form validation (Zod)*
>
> *Những thứ này sẽ học từ Session 7 trở đi.  
> Hôm nay chỉ cần: **Setup đúng + Component cơ bản + Props typing**.*

---

## 🔟 TEACHING NOTES
- **Slow Down:** Lúc config `vite.config.ts` và `tsconfig.json`. Đây là chỗ dễ sai nhất (sai dấu phẩy, sai ngoặc).
- **Emphasis:** Nhấn mạnh `React.ChangeEvent` và `React.FormEvent`. Đây là 2 cái dùng 90% thời gian làm form.
- **Skip:** Chưa cần giải thích sâu về `useMemo` hay `useCallback`. Focus vào `useState` và Props typing trước.
