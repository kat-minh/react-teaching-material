# LESSON PLAN: SESSION 04 - PROTECTED ROUTES & UI STATES

## 1️⃣ SESSION OVERVIEW
- **Title:** Guarding the Gates: Protected Routes and Standardizing UI States
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will learn how to protect sensitive pages from unauthorized access and how to build reusable UI states (Loading, Error, Empty) to improve User Experience.
- **Outcome:** A "Me" (Profile) page that redirects to Login if a user isn't authenticated, and professional UI skeletons for data loading.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Buổi trước chúng ta đã xây dựng được 'hệ thống giao thông' cho App. Nhưng có một vấn đề: Hiện tại ai cũng có thể vào trang `/me` (Cá nhân) dù chưa đăng nhập. Giống như việc bạn xây nhà mà quên lắp ổ khóa cửa vậy._

_Hôm nay ta sẽ học cách 'lắp khóa' (Protected Routes). Chúng ta sẽ viết một linh kiện đặc biệt để bảo vệ các trang nhạy cảm. Nếu người dùng chưa có 'chìa khóa' (Token), App sẽ tự động mời họ quay lại trang Login._

_Ngoài ra, chúng ta sẽ học về **useEffect** - một công cụ cực kỳ mạnh mẽ nhưng cũng là 'con dao hai lưỡi' khiến nhiều dev đau đầu nhất. Chúng ta sẽ chốt lại những quy tắc sống còn khi dùng nó để app không bị treo hay lag vô lý. Cuối cùng, ta sẽ chuẩn hóa cách hiển thị khi đang tải dữ liệu (Loading) hoặc khi gặp lỗi (Error) sao cho thật chuyên nghiệp."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"App thật không chỉ có 'chạy đúng'. App thật phải 'an toàn' và 'mượt mà'. Nếu không có Protected Routes, user sẽ thấy dữ liệu rác trước khi bị đá ra ngoài. Nếu không có UI States chuẩn, user sẽ tưởng app bị treo khi mạng chậm. Buổi này giúp app của bạn thoát mác 'đồ án sinh viên' để trở thành 'sản phẩm thực tế'."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🛡️ The "Guard" Component (RequireAuth)
- **Problem:** Bạn có 10 trang cần bảo mật. Chẳng lẽ trang nào cũng copy logic `if (!token) redirect`?
- **Solution:** Tạo một Component bọc ngoài (Wrapper). Nó đóng vai trò như một anh bảo vệ đứng trước cửa. 
    - Nếu có thẻ (Token) -> Cho vào (Render `Outlet`).
    - Nếu không -> Đẩy đi chỗ khác (Render `Navigate`).

### 🔄 useEffect - The Synchronizer
- **Lầm tưởng:** `useEffect` là nơi để chạy code khi component load.
- **Thực tế:** `useEffect` dùng để **đồng bộ** React với một hệ thống bên ngoài (API, Giao diện trình duyệt, Hẹn giờ).
- **Rule of Thumb:** Luôn luôn khai báo đầy đủ Dependency Array. Đừng bao giờ lừa dối React về những biến mà Effect đang sử dụng.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: PROTECTED ROUTES LOGIC (45 mins)

#### Step 1: Create `RequireAuth` Component
Create `src/components/guards/RequireAuth.tsx`:
_"Đây là anh bảo vệ. Hiện tại ta cắm chốt (mock) bằng một biến hằng số. Buổi sau ta sẽ thay nó bằng token thật."_

> **⚠️ WARNING (Instructor Script):**
> _"Các bạn lưu ý: Biến `isAuthenticated = false` ở đây chỉ là **HÀNG GIẢ (MOCK AUTH)** để ta chạy thử logic điều hướng. Tuyệt đối không copy kiểu này vào dự án thật. Buổi sau chúng ta sẽ thay nó bằng Context API và Token thật từ Backend."_

```tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';

export default function RequireAuth() {
  // ❌ HÀNG GIẢ - CHỈ DÙNG ĐỂ DEMO LOGIC
  const isAuthenticated = false; 
  const location = useLocation(); 

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <Outlet />;
}
```

#### Step 1.5: Auth Checking State (Lý thuyết - Key to avoid Flash Content)
_"Trong thực tế, khi bạn load trang, App cần 0.5s - 1s để vào localStorage lấy Token hoặc gọi API kiểm tra xem Token còn hạn không. Trong 1s đó, nếu ta không làm gì, App sẽ 'lúng túng' không biết cho vào hay đá ra -> Dẫn đến lỗi Flash Content (Thấy trang cá nhân 0.1s rồi mới bị văng ra Login)."_

**Quy tắc Production:**
- Luôn có state `isCheckingAuth = true` mặc định.
- Trong lúc `isCheckingAuth`, hiển thị `LoadingState`.
- Chỉ sau khi check xong mới quyết định `Navigate` hay `Outlet`.

#### Step 2: Apply Guard to Router
Open `src/router.tsx`:

```tsx
export const router = createBrowserRouter([
  {
    path: "/",
    element: <MainLayout />,
    children: [
      { index: true, element: <HomePage /> },
      { path: "login", element: <LoginPage /> },
      // NHÓM PROTECTED ROUTES
      {
        element: <RequireAuth />, // Bọc bảo vệ ở đây
        children: [
          { path: "me", element: <MePage /> },
          { path: "upload", element: <UploadPage /> },
        ]
      }
    ]
  }
]);
```

#### Step 3: Redirect-Back Logic (Optional but Pro)
Update `src/pages/LoginPage.tsx`:
_"Sau khi login xong, user phải quay lại trang họ vừa bị đá ra, chứ không nên luôn về trang chủ."_

```tsx
import { useNavigate, useLocation } from 'react-router-dom';

export default function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();
  
  // Lấy đường dẫn cũ từ state, nếu ko có thì mặc định về Home
  const from = location.state?.from?.pathname || "/";

  const handleLogin = () => {
    console.log("Logged in!");
    navigate(from, { replace: true });
  };

  return (
    <div>
       <button onClick={handleLogin}>Mock Login</button>
    </div>
  );
}
```

---

### PHASE 2: STANDARDIZING UI STATES (30 mins)

#### Step 1: Create Reusable UI Components
Create `src/components/ui/StatusStates.tsx`:
_"Đừng bao giờ để trang trắng tinh khi đang load."_

```tsx
export const LoadingState = () => (
  <div className="flex flex-col items-center justify-center p-20 gap-4">
    <div className="w-10 h-10 border-4 border-blue-500 border-t-transparent rounded-full animate-spin"></div>
    <p className="text-gray-500 animate-pulse">Đang tải dữ liệu...</p>
  </div>
);

export const ErrorState = ({ message = "Đã có lỗi xảy ra" }: { message?: string }) => (
  <div className="p-10 border border-red-200 bg-red-50 text-red-600 rounded-lg text-center">
    <p className="font-bold">Oops!</p>
    <p>{message}</p>
    <button className="mt-4 underline" onClick={() => window.location.reload()}>Thử lại</button>
  </div>
);
```

---

### PHASE 3: useEffect RULES OF THUMB (30 mins)

#### Step 1: Demo Logic Synchronization
Update `src/pages/MePage.tsx`:
_"Chúng ta dùng useEffect để đồng bộ tiêu đề trang (document.title) với trang hiện tại."_

> **Instructor Script:**
> _"Việc đổi Title trang là một 'tác dụng phụ' (side-effect) vì nó nằm ngoài tầm kiểm soát của Component React (nó thuộc về DOM của trình duyệt). Đây là ví dụ đơn giản nhất. Sang các buổi sau, chúng ta sẽ dùng useEffect cho những việc 'khủng long' hơn như Gọi API lấy dữ liệu."_

```tsx
import { useEffect } from 'react';

export default function MePage() {
  useEffect(() => {
    document.title = "Trang cá nhân của tôi";
    
    // Cleanup function: Khi rời trang thì trả lại title cũ
    return () => {
      document.title = "Shopping App";
    };
  }, []); // [] = Chỉ chạy 1 lần khi mount

  return <h1>Đây là khu vực nhạy cảm!</h1>;
}
```

> **❗ Rule of Thumb (Production):**
> Đừng bao giờ bỏ trống Dependency Array nếu trong Effect có dùng biến bên ngoài. Nếu dùng `eslint-plugin-react-hooks`, nó sẽ gạch đỏ báo lỗi ngay. Đừng bao giờ lừa dối React!

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Infinite Loop với useEffect**:
    *   *Symptom:* Trình duyệt bị treo, máy nóng lên, console log chạy liên tục.
    *   *Cause:* Trong `useEffect` lại gọi `setState` mà cái state đó lại nằm trong dependency array.
    *   *Fix:* Kiểm tra lại deps. Có thực sự cần đưa biến đó vào ko? Hoặc dùng Functional Update `setCount(prev => prev + 1)`.
2.  **Flash Content**:
    *   *Symptom:* Thấy trang Cá nhân hiện ra trong 0.1 giây rồi mới bị văng ra trang Login.
    *   *Fix:* Phải kiểm tra login status ở tầng cao nhất (Router) và tuyệt đối không render nội dung nếu chưa xác thực xong.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ta dùng `Navigate replace` thay vì `navigate(...)` thông thường khi redirect?"*
    *   **A:** Để đè lên lịch sử web. Nếu dùng `replace`, user bấm 'Back' trên trình duyệt sẽ không bị quay lại trang bị cấm (tránh vòng lặp back-redirect vô tận).
2.  **Q:** *"Nếu tôi muốn fetch dữ liệu trong `useEffect`, tôi nên làm gì với dependency array?"*
    *   **A:** Nếu fetch theo ID, thì ID phải nằm trong array `[id]`. Nếu ID đổi, Effect sẽ chạy lại để lấy data mới.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Hoàn thiện `RequireAuth`.
- Sửa code để `const isAuthenticated` lấy giá trị từ một biến trong file (hardcode).
- Thử đổi `true/false` và quan sát kết quả điều hướng.
- Thêm một component `LoadingState` hiển thị trong 2 giây trước khi cho vào trang `/me` (dùng `setTimeout` giả lập).

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** 
1.  Tạo component `PublicOnlyRoute`. Component này ngược với `RequireAuth`: Nếu đã login rồi thì KHÔNG cho vào trang Login/Register nữa (đá về Home).
2.  Áp dụng `PublicOnlyRoute` cho trang Login và Register trong `router.tsx`.

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Học viên giải thích được `location.state.from` dùng để làm gì.
- **Signal:** Học viên viết đúng dependency array mà không cần nhìn mẫu.
- **Observable:** Khi gõ `/me` trên thanh URL, trình duyệt tự nhảy về `/login`.

## 🔟 TEACHING NOTES
- **Slow Down:** Lúc giải thích về `Navigate` và `useLocation`. Đây là phần logic điều hướng nâng cao, rất dễ làm học viên bị ngợp.
- **Emphasis:** `useEffect` không phải là nơi để tính toán dữ liệu (derived state). Nếu tính được, hãy tính trực tiếp trong component body.
- **Red Flag:** Học viên dùng `window.location.href = '/login'`. Hãy nhắc nhở: "Chúng ta đang làm SPA, cấm dùng window.location vì nó làm reload trang".
