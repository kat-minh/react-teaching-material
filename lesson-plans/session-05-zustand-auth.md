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

> **⚠️ INSTRUCTOR SCRIPT (REHYDRATION AWARENESS):** > _"Trong app thật, việc load dữ liệu từ LocalStorage (Hydrate) không phải tức thì 0.00ms. Nó có độ trễ 0.1s. Ta cần biết khi nào nó hydrate xong để tránh lỗi 'Flash Login' (chưa kịp load token đã vội đá ra Login)._
>
> _Hôm nay ta CHẤP NHẬN flash login. Đây là trade-off để tập trung vào Selector Pattern._
>
> _App production sẽ xử lý bằng `hasHydrated` flag và `onRehydrateStorage` callback. Các bạn sẽ học cách xử lý triệt để khi integrate API thật ở tuần sau."_

> **🔥 WHY THIS SESSION EXISTS?** > _"Nếu không có Session này, người dùng của bạn sẽ phải đăng nhập lại mỗi lần refresh trang (cực kỳ ức chế). Chúng ta cần học cách đồng bộ State với LocalStorage một cách an toàn và tự động."_

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
import { create } from "zustand";

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
  setTokens: (access, refresh) =>
    set({
      accessToken: access,
      refreshToken: refresh,
    }),

  // Action: Xóa token (Logout)
  clearTokens: () =>
    set({
      accessToken: null,
      refreshToken: null,
    }),
}));
```

#### Step 3: Persistence Middleware (The Magic)

_"Hiện tại F5 vẫn mất. Giờ ta thêm 'bùa chú' persist vào."_

Update `src/stores/auth.store.ts`:

```ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware"; // Import middleware

interface AuthState {
  /* giữ nguyên */
}

// Bọc create trong persist middleware
export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      accessToken: null,
      refreshToken: null,
      setTokens: (access, refresh) =>
        set({ accessToken: access, refreshToken: refresh }),
      clearTokens: () => set({ accessToken: null, refreshToken: null }),
    }),
    {
      name: "shopping-card-auth", // Tên key trong LocalStorage (F12 -> Application -> Local Storage)
      storage: createJSONStorage(() => localStorage), // Nơi lưu trữ
    }
  )
);
```

---

### PHASE 2: SELECTOR PATTERN (20 mins - CRITICAL!)

> **Instructor Script:** > _"⏱️ 20 PHÚT này CỰC KỲ QUAN TRỌNG._
>
> _Đây là sai lầm số 1 khi dùng Zustand: Lấy toàn bộ store._
>
> _Kết quả: Component re-render KHÔNG CẦN THIẾT → App lag._
>
> _Professional developers LUÔN dùng Selector Pattern."_

#### ❌ Problem: Lấy Toàn Bộ Store (BAD Performance)

```tsx
// ❌ SAI - Re-render khi BẤT KỲ field nào trong store thay đổi
function Header() {
  const store = useAuthStore(); // Lấy TOÀN BỘ store
  const isAuthed = !!store.accessToken;

  console.log("Header render"); // Sẽ log mỗi khi store.refreshToken đổi!

  return <div>{isAuthed ? "Logged In" : "Guest"}</div>;
}

// ⚠️ VẤN ĐỀ:
// - Component này chỉ cần biết isAuthed
// - Nhưng nó subscribe vào TẤT CẢ fields: accessToken, refreshToken, setTokens, clearTokens
// - Khi refreshToken đổi (VD: refresh token flow) → Header re-render (KHÔNG CẦN THIẾT!)
```

#### ✅ Solution: Selector Pattern (BEST Practice)

```tsx
// ✅ ĐÚNG - Chỉ subscribe field cần thiết
function Header() {
  const isAuthed = useAuthStore((state) => !!state.accessToken);
  // ^ Chỉ lấy accessToken, bỏ qua các field khác

  console.log("Header render"); // Chỉ log khi accessToken thay đổi

  return <div>{isAuthed ? "Logged In" : "Guest"}</div>;
}

// ✅ Performance: Component chỉ re-render khi accessToken thay đổi
```

#### Pattern 1: Single Field Selector

```tsx
// Lấy 1 field
const accessToken = useAuthStore((state) => state.accessToken);

// Lấy action
const setTokens = useAuthStore((state) => state.setTokens);

// Computed selector (tính toán)
const isAuthed = useAuthStore((state) => !!state.accessToken);

// ✅ Rule: Mỗi useAuthStore() call = 1 subscription
// Component chỉ re-render khi field được select thay đổi
```

#### Pattern 2: Multiple Fields với Shallow Compare

```tsx
import { shallow } from "zustand/shallow";

// ❌ SAI - Object mới mỗi lần → re-render mỗi lần
function ProfilePage() {
  const { accessToken, refreshToken } = useAuthStore((state) => ({
    accessToken: state.accessToken,
    refreshToken: state.refreshToken,
  }));
  // ^ Object {} mới mỗi render → React nghĩ props đổi → re-render

  return <div>{accessToken}</div>;
}

// ✅ ĐÚNG - Dùng shallow compare
function ProfilePage() {
  const { accessToken, refreshToken } = useAuthStore(
    (state) => ({
      accessToken: state.accessToken,
      refreshToken: state.refreshToken,
    }),
    shallow // So sánh shallow (theo value, không theo reference)
  );

  return <div>{accessToken}</div>;
}

// ✅ Performance: Chỉ re-render khi accessToken HOẶC refreshToken thay đổi
```

#### Pattern 3: Selector với Derived State

```tsx
// Tạo selector functions (reusable)
const selectIsAuthed = (state) => !!state.accessToken;
const selectHasRefreshToken = (state) => !!state.refreshToken;

// Dùng trong component
function App() {
  const isAuthed = useAuthStore(selectIsAuthed);
  const hasRefreshToken = useAuthStore(selectHasRefreshToken);

  return (
    <div>
      {isAuthed && <p>Welcome!</p>}
      {!hasRefreshToken && <p>Session expired soon</p>}
    </div>
  );
}
```

#### Real-World Example: Axios Interceptor

```tsx
// ❌ SAI - Trong interceptor lấy toàn bộ store
apiClient.interceptors.request.use((config) => {
  const store = useAuthStore(); // ❌ Không thể dùng hook ở đây
  config.headers.Authorization = `Bearer ${store.accessToken}`;
  return config;
});

// ✅ ĐÚNG - Dùng getState() (không phải hook)
apiClient.interceptors.request.use((config) => {
  const accessToken = useAuthStore.getState().accessToken;
  // ^ getState() = lấy state KHÔNG subscribe (không re-render)

  if (accessToken) {
    config.headers.Authorization = `Bearer ${accessToken}`;
  }
  return config;
});

// ⚡ Lưu ý: getState() dùng cho non-React code (interceptors, utils)
// Trong component thì dùng useAuthStore((state) => ...)
```

#### Comparison Table

| Pattern                         | Re-render Trigger                 | Performance | Use Case                    |
| :------------------------------ | :-------------------------------- | :---------- | :-------------------------- |
| `useAuthStore()`                | BẤT KỲ field nào đổi              | ❌ XẤU      | TUYỆT ĐỐI KHÔNG DÙNG        |
| `useAuthStore((s) => s.token)`  | Chỉ khi `token` đổi               | ✅ TỐT      | Lấy 1 field                 |
| `useAuthStore(..., shallow)`    | Khi BẤT KỲ field nào trong {} đổi | ✅ TỐT      | Lấy nhiều fields            |
| `useAuthStore.getState().token` | KHÔNG re-render                   | ⚡ BEST     | Non-React code (utils, API) |

#### 🔍 Debug Tool: Zustand DevTools

```tsx
import { devtools } from "zustand/middleware";

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set) => ({
        /* ... */
      }),
      { name: "shopping-card-auth" }
    ),
    { name: "AuthStore" } // Tên hiển thị trong Redux DevTools
  )
);

// Install Redux DevTools Extension để xem store state
```

> **📌 Rule of Thumb (Production):**
>
> 1. **LUÔN** dùng selector: `useAuthStore((state) => state.field)`
> 2. **TUYỆT ĐỐI KHÔNG** dùng: `useAuthStore()` (lấy toàn bộ)
> 3. Nhiều fields → dùng `shallow` compare
> 4. Non-React code → dùng `getState()`
> 5. Debug → dùng `devtools` middleware

---

### PHASE 3: CONNECT STORE TO APP (45 mins)

#### Step 1: Update Login Logic (Mock)

Update `src/pages/LoginPage.tsx`:
_"Giờ ta giả vờ Login thành công và lưu token giả vào Store."_

```tsx
import { useAuthStore } from "@/stores/auth.store"; // Import hook
import { useNavigate } from "react-router-dom";

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
    navigate("/me");
  };

  return <button onClick={handleLogin}>Login (Save Token)</button>;
}
```

#### Step 2: Update Guard Component (Real Check)

Update `src/components/guards/RequireAuth.tsx`:
_"Đuổi cổ anh bảo vệ 'Hàng Giả' hôm trước đi. Giờ check token thật."_

```tsx
import { Navigate, Outlet, useLocation } from "react-router-dom";
import { useAuthStore } from "@/stores/auth.store";

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

> **⚠️ CRITICAL: Áp dụng ĐÚNG Selector Pattern (vừa học ở trên):**

```tsx
// ... imports
import { useAuthStore } from "@/stores/auth.store";
import { shallow } from "zustand/shallow";

export default function MainLayout() {
  // ✅ ĐÚNG - Dùng Selector Pattern với shallow compare
  const { accessToken, clearTokens } = useAuthStore(
    (state) => ({
      accessToken: state.accessToken,
      clearTokens: state.clearTokens,
    }),
    shallow // Shallow compare để tránh re-render không cần thiết
  );

  // ⚠️ Hoặc nếu muốn tách riêng (cũng OK):
  // const accessToken = useAuthStore((state) => state.accessToken);
  // const clearTokens = useAuthStore((state) => state.clearTokens);

  const navigate = useNavigate();

  const handleLogout = () => {
    clearTokens(); // Xóa trong store -> Tự xóa trong LocalStorage
    navigate("/login");
  };

  return (
    // ... header code
    <nav>
      {/* ... links khác */}
      {accessToken ? (
        <button onClick={handleLogout} className="text-red-500">
          Logout
        </button>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên cú pháp `()()` của persist**:
    - _Bug:_ `create(persist(...))` sẽ báo lỗi type đỏ lòm.
    - _Fix:_ Cú pháp đúng là `create<Type>()(persist(...))`. Đây là Currying function syntax của TS.
2.  **Lấy cả cục state `const store = useAuthStore()`**:
    - _Issue:_ Component sẽ render lại khi BẤT KỲ cái gì trong store thay đổi → Performance xấu.
    - _Fix:_ LUÔN dùng selector: `useAuthStore((state) => state.accessToken)` - chỉ lấy field cần thiết.
    - _Fix/Best Practice:_ Luôn dùng selector để lấy đúng cái cần: `const token = useAuthStore(s => s.token)`.
3.  **Hydration Error (Next.js only - nhưng nên biết)**:
    - _Info:_ LocalStorage chỉ có ở Client. Nếu học Next.js sau này sẽ gặp lỗi server render khác client. Với Vite SPA thì không sao.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** _"Tại sao không dùng `useState` ở `App.tsx` rồi truyền props xuống?"_
    - **A:** Vì Props Drilling (khoan lỗ). Bạn phải truyền qua 10 tầng để tới được cái nút Logout nằm sâu bên trong. Zustand giúp ta "teleport" dữ liệu tới đúng chỗ cần.
2.  **Q:** _"Nếu tôi tắt tab đi mở lại, token còn không?"_
    - **A:** Còn. Vì ta dùng `persist` với `localStorage`. Trừ khi user xóa cache hoặc ta gọi `clearTokens()`.

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
