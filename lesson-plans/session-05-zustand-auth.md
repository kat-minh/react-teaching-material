# LESSON PLAN: SESSION 05 - ZUSTAND AUTH STORE & TOKEN PERSISTENCE

## 1️⃣ SESSION OVERVIEW
- **Title:** The Vault: Zustand Auth Store & Token Persistence
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will replace the "Mock Auth" from Session 4 with a real Global State Manager (Zustand), allowing the app to remember login status even after a page reload.
- **Outcome:** A `useAuthStore` hook that manages `accessToken` / `refreshToken`, saves them to `localStorage` automatically, and powers the `RequireAuth` guard.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Buổi trước anh bảo vệ `RequireAuth` của chúng ta chỉ là hàng giả đanh đá (luôn `false`). Hôm nay ta sẽ tuyển một anh bảo vệ xịn hơn._

_Trong React, chúng ta cần một cái 'Két sắt' (Store) để chứa Token đăng nhập. Két sắt này phải: (1) Truy cập được từ mọi nơi trong App, và (2) Không bị mất khi F5 lại trang._

_Chúng ta sẽ dùng **Zustand**. Đây là thư viện quản lý state đơn giản nhất thế giới, nhẹ hơn Redux 100 lần, code ít hơn mà hiệu quả tương đương. Hôm nay mục tiêu là: Login xong -> Lưu token vào két -> F5 trang web vẫn nhớ là đã login."_

> **⚠️ INSTRUCTOR SCRIPT (REHYDRATION AWARENESS):**
> _"Trong app thật, việc load dữ liệu từ LocalStorage (Hydrate) không phải tức thì 0.00ms. Nó có độ trễ 0.1s. Ta cần biết khi nào nó hydrate xong để tránh lỗi 'Flash Login' (chưa kịp load token đã vội đá ra Login). Zustand có cung cấp `onRehydrateStorage`, nhưng hôm nay ta học concept trước, buổi sau sẽ xử lý triệt để."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Nếu không có Session này, người dùng của bạn sẽ phải đăng nhập lại mỗi lần refresh trang (cực kỳ ức chế). Chúng ta cần học cách đồng bộ State với LocalStorage một cách an toàn và tự động."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🐻 Zustand vs Redux vs Context
- **Redux:** Giống như bộ máy hành chính cồng kềnh. Muốn sửa 1 dữ liệu phải qua 3-4 cửa (Action, Reducer, Dispatch).
- **Context API:** Giống như hệ thống loa phát thanh. Dễ dùng nhưng hiệu năng kém nếu dùng cho data thay đổi liên tục (render lại cả cây).
- **Zustand:** Giống như cái biến toàn cục siêu thông minh. Component nào cần thì "móc" (hook) vào lấy. Chỉ component đó render lại khi data đổi.

### 💾 Persistence (Sự bền vững)
- **Memory Store:** Biến JS thông thường -> F5 là mất sạch.
- **Persistent Store:** Tự động copy dữ liệu xuống ổ cứng (LocalStorage) mỗi khi có thay đổi. Khi mở App, tự động load từ ổ cứng lên lại.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: SETUP ZUSTAND STORE (45 mins)

#### Step 1: Install Library
```bash
# Terminal
npm install zustand
```

#### Step 2: Define Auth Store
Create `src/stores/auth.store.ts`:
_"Code chậm thôi nhé. Đây là trái tim của hệ thống Auth."_

```ts
import { create } from 'zustand';

// 1. Define Interface cho State & Actions
interface AuthState {
  accessToken: string | null;
  refreshToken: string | null;
  
  // Actions: Hàm để thay đổi state
  setTokens: (access: string, refresh: string) => void;
  clearTokens: () => void;
}

// 2. Create Store
export const useAuthStore = create<AuthState>((set) => ({
  // Initial State
  accessToken: null,
  refreshToken: null,

  // Action: Lưu token
  setTokens: (access, refresh) => set({ 
    accessToken: access, 
    refreshToken: refresh 
  }),

  // Action: Xóa token (Logout)
  clearTokens: () => set({ 
    accessToken: null, 
    refreshToken: null 
  }),
}));
```

#### Step 3: Persistence Middleware (The Magic)
_"Hiện tại F5 vẫn mất. Giờ ta thêm 'bùa chú' persist vào."_

Update `src/stores/auth.store.ts`:
```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware'; // Import middleware

interface AuthState { /* giữ nguyên */ }

// Bọc create trong persist middleware
export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      accessToken: null,
      refreshToken: null,
      setTokens: (access, refresh) => set({ accessToken: access, refreshToken: refresh }),
      clearTokens: () => set({ accessToken: null, refreshToken: null }),
    }),
    {
      name: 'shopping-card-auth', // Tên key trong LocalStorage (F12 -> Application -> Local Storage)
      storage: createJSONStorage(() => localStorage), // Nơi lưu trữ
    }
  )
);
```

---

### PHASE 2: CONNECT STORE TO APP (45 mins)

#### Step 1: Update Login Logic (Mock)
Update `src/pages/LoginPage.tsx`:
_"Giờ ta giả vờ Login thành công và lưu token giả vào Store."_

```tsx
import { useAuthStore } from '@/stores/auth.store'; // Import hook
import { useNavigate } from 'react-router-dom';

export default function LoginPage() {
  // Lấy hàm setTokens từ store
  const setTokens = useAuthStore((state) => state.setTokens);
  const navigate = useNavigate();

  const handleLogin = () => {
    // Giả lập API trả về token
    const mockAccessToken = "ey...fake-access-token";
    const mockRefreshToken = "ey...fake-refresh-token";

    // Lưu vào store -> Tự động lưu xuống LocalStorage
    setTokens(mockAccessToken, mockRefreshToken);

    // Redirect
    navigate('/me');
  };

  return <button onClick={handleLogin}>Login (Save Token)</button>;
}
```

#### Step 2: Update Guard Component (Real Check)
Update `src/components/guards/RequireAuth.tsx`:
_"Đuổi cổ anh bảo vệ 'Hàng Giả' hôm trước đi. Giờ check token thật."_

```tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { useAuthStore } from '@/stores/auth.store';

export default function RequireAuth() {
  const token = useAuthStore((state) => state.accessToken); // Lấy token thật
  const location = useLocation();

  // Kiểm tra: Có token mới cho vào
  if (!token) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // ⚠️ QUAN TRỌNG: Token tồn tại ở đây chỉ nghĩa là "người dùng CÓ cầm thẻ".
  // Không đảm bảo thẻ còn hạn hay không. Backend API sau này sẽ check hạn.
  return <Outlet />;
}
```

#### Step 3: Add Logout Button
Update `src/components/layouts/MainLayout.tsx`:
_"Có vào thì phải có ra. Thêm nút Logout."_

```tsx
// ... imports
import { useAuthStore } from '@/stores/auth.store';

export default function MainLayout() {
  const { accessToken, clearTokens } = useAuthStore(); // Lấy state & action
  const navigate = useNavigate();

  const handleLogout = () => {
    clearTokens(); // Xóa trong store -> Tự xóa trong LocalStorage
    navigate('/login');
  };

  return (
    // ... header code
    <nav>
       {/* ... links khác */}
       {accessToken ? (
         <button onClick={handleLogout} className="text-red-500">Logout</button>
       ) : (
         <Link to="/login">Login</Link>
       )}
    </nav>
  );
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên cú pháp `()()` của persist**:
    *   *Bug:* `create(persist(...))` sẽ báo lỗi type đỏ lòm.
    *   *Fix:* Cú pháp đúng là `create<Type>()(persist(...))`. Đây là Currying function syntax của TS.
2.  **Lấy cả cục state `const store = useAuthStore()`**:
    *   *Issue:* Component sẽ render lại khi BẤT KỲ cái gì trong store thay đổi.
    *   *Fix/Best Practice:* Luôn dùng selector để lấy đúng cái cần: `const token = useAuthStore(s => s.token)`.
3.  **Hydration Error (Next.js only - nhưng nên biết)**:
    *   *Info:* LocalStorage chỉ có ở Client. Nếu học Next.js sau này sẽ gặp lỗi server render khác client. Với Vite SPA thì không sao.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao không dùng `useState` ở `App.tsx` rồi truyền props xuống?"*
    *   **A:** Vì Props Drilling (khoan lỗ). Bạn phải truyền qua 10 tầng để tới được cái nút Logout nằm sâu bên trong. Zustand giúp ta "teleport" dữ liệu tới đúng chỗ cần.
2.  **Q:** *"Nếu tôi tắt tab đi mở lại, token còn không?"*
    *   **A:** Còn. Vì ta dùng `persist` với `localStorage`. Trừ khi user xóa cache hoặc ta gọi `clearTokens()`.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Thêm thông tin User vào Store.
- Sửa interface `AuthState`: thêm `user: { name: string, email: string } | null`.
- Sửa `setTokens`: nhận thêm object `user`.
- Tại trang Login, giả lập thông tin user: `name: "Student A", email: "a@test.com"`.
- Hiển thị: "Xin chào, Student A" trên Header sau khi login.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Token Expiration (Tư duy).
1.  (Chưa cần code thật) Hãy suy nghĩ: Token thường có hạn (ví dụ 1 tiếng).
2.  Làm sao để biết token hết hạn mà không cần gọi API?
3.  Gợi ý: Dữ liệu JWT (JSON Web Token) thực chất là chuỗi JSON được mã hóa Base64. Trong đó có trường `exp` (expiration). Tìm hiểu thư viện `jwt-decode`.

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Login xong -> F5 trang -> Vẫn ở trang `/me` (không bị đá về Login).
- **Verify:** Mở DevTools -> Application -> Local Storage. Thấy key `shopping-card-auth` chứa chuỗi JSON đúng format.

## 🔟 TEACHING NOTES
- **Slow Down:** Phần cú pháp `create<T>()(...)` của Zustand Persist rất khó nhớ và khó gõ đúng ngay lần đầu. Hãy gõ mẫu thật chậm.
- **Emphasis:** Selector Pattern (`state => state.token`). Đây là điểm khác biệt hiệu năng lớn nhất so với Context API. Hãy demo bằng `console.log('render')` nếu có thời gian.
- **Red Flag:** Học viên cố gán `useAuthStore.token = 'abc'`. Nhắc nhở: State trong Zustand là immutable (bất biến) từ bên ngoài, phải dùng Action `setTokens`.
