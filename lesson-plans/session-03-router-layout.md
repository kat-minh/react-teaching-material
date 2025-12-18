# LESSON PLAN: SESSION 03 - REACT ROUTER V6 & LAYOUT PATTERN

## 1️⃣ SESSION OVERVIEW
- **Title:** Navigating the App: React Router v6 and Layout Pattern
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will master client-side routing, understand the difference between Page vs Layout, and implement a production-ready routing structure.
- **Outcome:** A multi-page app (Home, Login, Register, Profile) with shared Navigation/Footer and smooth transitions without page reloads.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào cả lớp. Các bạn đã bao giờ tự hỏi: Tại sao Facebook hay Gmail khi bấm vào Menu thì nội dung đổi nhưng cái thanh Header bên trên không bao giờ bị giật hay load lại không?_

_Đó chính là sức mạnh của **Single Page Application (SPA)**. Trong web truyền thống, mỗi lần bấm link là trình duyệt 'trắng trang' để tải file HTML mới. Trong React, ta không làm thế. Ta chỉ đổi 'linh kiện' bên trong cái khung có sẵn._

_Hôm nay ta sẽ học cách dựng 'Hệ thống giao thông' cho App của mình (React Router v6). Chúng ta sẽ học cách tạo ra một bộ khung cố định (Layout) và đổi nội dung bên trong dựa trên URL. Đây là kỹ năng bắt buộc để biến một đống Component rời rạc thành một Website hoàn chỉnh."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Người dùng không bao giờ ở lỳ 1 trang. Nếu bạn không biết tổ chức Routing chuẩn ngay từ đầu, bạn sẽ gặp lỗi 'nháy trang', mất dữ liệu khi chuyển tab, và cực kỳ khó quản lý các khu vực cần bảo mật (như trang Cá nhân). Hôm nay ta sẽ dọn dẹp nhà cửa và phân chia khu vực rõ ràng cho App."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Layout vs Page
- **Layout (Cái Khung):** Giống như cái nhà. Header, Footer là tường và mái. Nó luôn ở đó.
- **Page (Nội dung):** Giống như đồ đạc trong phòng. Khi bạn đổi từ phòng khách sang phòng ngủ, tường và mái vẫn giữ nguyên, chỉ có đồ đạc (nội dung trang) là thay đổi.
- **Outlet (Cái cửa):** Trong React Router, `Outlet` chính là cái lỗ hổng trong Layout để 'đổ' nội dung các trang con vào.

### 🛣️ Client-side Routing vs Server-side Routing
- **Server-side (Truyền thống):** Bấm link -> Gửi request lên Server -> Server trả về file HTML mới -> Trình duyệt load lại toàn bộ. (Chậm, tốn tài nguyên).
- **Client-side (React):** Bấm link -> JS can thiệp -> Chặn reload trình duyệt -> Chỉ render lại Component tương ứng với URL mới. (Cực nhanh, mượt).

### 🌍 URL chính là một loại State
- Trong React SPA, **URL thay đổi -> Router render component khác**.
- Chúng ta không cần `useState` để nhớ người dùng đang ở trang nào.
- Router đóng vai trò là "State Manager cho navigation". Thay đổi URL là thay đổi State của toàn bộ ứng dụng.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: INSTALL & SETUP ROUTER (30 mins)

#### Step 1: Install Library
```bash
# Terminal
npm install react-router-dom
```

#### Step 2: Define Router Structure
Create `src/router.tsx`:
_"Ta tách Router ra một file riêng để dễ quản lý khi dự án lên đến hàng trăm route."_

```tsx
import { createBrowserRouter } from 'react-router-dom';
import MainLayout from '@/components/layouts/MainLayout';
import HomePage from '@/pages/HomePage';
import LoginPage from '@/pages/LoginPage';

// Sử dụng createBrowserRouter - API mới nhất và chuẩn nhất của v6
export const router = createBrowserRouter([
  {
    path: "/",
    element: <MainLayout />, // Cấp cha là Layout
    children: [
      {
        index: true,     // index: true nghĩa là Route mặc định.
                         // Khi URL đúng bằng path của cha (/) -> sẽ render component này.
        element: <HomePage />
      },
      {
        path: "login",   // Đường dẫn /login
        element: <LoginPage />
      }
    ]
  }
]);
```

#### Step 3: Mount Router to App
Update `src/main.tsx`:
_"Xóa App component cũ, thay bằng RouterProvider."_

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { RouterProvider } from 'react-router-dom'
import { router } from './router' // Import router vừa tạo
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    {/* Cung cấp router cho toàn bộ ứng dụng */}
    <RouterProvider router={router} />
  </React.StrictMode>,
)
```

---

### PHASE 2: IMPLEMENTING LAYOUT & NAVIGATION (45 mins)

#### Step 1: Create MainLayout with Outlet
Create `src/components/layouts/MainLayout.tsx`:
_"Header và Footer nằm ở đây. Giữa chúng là Outlet."_

```tsx
import { Link, Outlet } from 'react-router-dom';

export default function MainLayout() {
  return (
    <div className="min-h-screen flex flex-col">
      {/* Header cố định */}
      <header className="bg-white border-b p-4 sticky top-0 z-50">
        <nav className="max-w-4xl mx-auto flex justify-between items-center">
          <Link to="/" className="text-xl font-bold text-blue-600">
            ShoppingFE
          </Link>
          <div className="flex gap-4">
            {/* Dùng Link, KHÔNG dùng thẻ <a> */}
            <Link to="/" className="hover:text-blue-500">Home</Link>
            <Link to="/login" className="hover:text-blue-500">Login</Link>
          </div>
        </nav>
      </header>

      {/* Main Content Area */}
      <main className="flex-1 max-w-4xl mx-auto w-full p-6">
        {/* Outlet là nơi render các trang con (Home, Login...) */}
        <Outlet />
      </main>

      {/* Footer cố định */}
      <footer className="bg-gray-100 p-6 text-center text-gray-500 text-sm">
        &copy; 2024 Piedteam - ReactJS Master Class
      </footer>
    </div>
  );
}
```

> **📌 RULE OF THUMB: LINK vs <a>**
> _"Cấm dùng thẻ `<a>` để điều hướng nội bộ. Thẻ `<a>` sẽ làm browser reload trang, làm mất sạch State. Luôn dùng component `<Link>` của react-router-dom."_
>
> **💡 PATH RULE (Nói miệng cho học viên):**
> - `/login`: **Absolute path** (luôn đi từ trang chủ).
> - `login`: **Relative path** (đi từ route cha hiện tại, dùng khi có nested routes sâu).

#### Step 2: Create Mock Pages
Create `src/pages/LoginPage.tsx`:
```tsx
export default function LoginPage() {
  return (
    <div className="bg-white p-8 rounded-xl shadow-sm border max-w-sm mx-auto">
      <h2 className="text-2xl font-bold mb-4">Login</h2>
      <p className="text-sm text-gray-500 mb-6">Welcome back to our shop.</p>
      <button className="w-full bg-blue-600 text-white p-3 rounded-lg font-bold">
        Sign In
      </button>
    </div>
  );
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Dùng thẻ `<a>` thay vì `<Link>`**:
    *   *Symptom:* Bấm menu thấy tab trình duyệt quay vòng (reload), console bị clear sạch.
    *   *Fix:* Đổi hết `<a>` thành `<Link to="...">`.
2.  **Quên `<Outlet />` trong Layout**:
    *   *Symptom:* Chuyển URL `/login` thấy URL đổi nhưng màn hình vẫn là trang trắng hoặc chỉ có Header/Footer.
    *   *Fix:* Phải chèn `<Outlet />` vào nơi muốn nội dung trang hiển thị.
3.  **Lỗi Import nhầm**:
    *   *Symptom:* Lỗi `Link is used outside of Router`.
    *   *Fix:* Đảm bảo mọi component dùng `Link` đều phải nằm trong cây của `RouterProvider`.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ta nên dùng nested routes (lồng nhau) thay vì viết từng route phẳng?"*
    *   **A:** Để tận dụng Layout Pattern. Nested routes cho phép giữ lại phần giao diện chung (Layout) và chỉ thay đổi phần lõi.
2.  **Q:** *"Nếu tôi muốn có một trang Profile chỉ dành cho người đã đăng nhập, tôi nên tổ chức Route thế nào?"*
    *   **A:** Ta nên bọc các route đó vào một Layout cấp 2 (ví dụ `RequireAuth`) để kiểm tra quyền trước khi cho vào. (Đây là nội dung buổi tới).

## 7️⃣ IN-CLASS MINI TASK
**Task:** Thêm trang **Register**.
- Tạo file `src/pages/RegisterPage.tsx`.
- Thêm path `/register` vào `router.tsx` lồng trong `MainLayout`.
- Thêm một link "Register" lên thanh Header trong `MainLayout`.
- **Yêu cầu:** Bấm qua lại giữa Home - Login - Register mà không được reload trang (check console).

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Nâng cấp Navigation.
1.  Sử dụng `NavLink` thay cho `Link` trong Header.
2.  Thêm logic style: Khi một link đang active (đang ở trang đó), hãy đổi màu chữ thành đỏ hoặc in đậm để user biết mình đang ở đâu.
3.  Gợi ý: `NavLink` cung cấp `isActive` boolean trong callback của `className` hoặc `style`.

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Học viên chuyển trang mà log trong Console từ trang cũ vẫn còn nguyên (chứng minh không reload).
- **Signal:** Học viên hiểu `Outlet` là 'placeholder' cho nội dung động.

## 🔟 TEACHING NOTES
- **Slow Down:** Lúc giải thích về cấu trúc lồng nhau trong `createBrowserRouter`. Đây là tư duy lồng ghép đối tượng, học viên mới thường bị rối ngoặc `[]` và `{}`.
- **Emphasis:** Nhấn mạnh `createBrowserRouter` là "Data APIs" mới nhất, hỗ trợ tốt nhất cho việc tối ưu tốc độ. Đừng dạy kiểu `BrowserRouter` bọc ngoài `Routes` cũ (dù vẫn chạy được).
- **Red Flag:** Nếu học viên viết 10 paths mà copy 10 lần Header/Footer vào từng page -> Cần dừng lại giải thích về sự lãng phí này và ép dùng Layout Pattern.
