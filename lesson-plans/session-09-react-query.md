# LESSON PLAN: SESSION 09 - TANSTACK QUERY (DATA FETCHING)

## 1️⃣ SESSION OVERVIEW

- **Title:** The Data Wizard: Mastering Server State with TanStack Query
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will abandon `useEffect` for data fetching and adopt TanStack Query (React Query) to manage Server State (Loading, Error, Caching) effortlessly.
- **Outcome:** Retrieve User Profile (`/me`) using `useQuery` with automatic caching and standardized loading/error UI states.

## 2️⃣ INSTRUCTOR OPENING SCRIPT

\_"Chào các bạn. Hôm nay là ngày chúng ta 'đốt bỏ' cuốn sách giáo khoa React cũ.
Trong sách, họ dạy bạn Fetch Data bằng `useEffect` + `useState`.

Nhưng thực tế:

- Nếu mạng chậm?
- Nếu user tab qua tab lại?
- Nếu user F5 liên tục?
  Code cũ sẽ vỡ trận.

Hôm nay tôi giới thiệu **TanStack Query**. Nó không chỉ gọi API. Nó là 'người quản gia' thông minh: Tự biết khi nào dữ liệu cũ để đi lấy mới, tự biết cache để App nhanh như gió. Trong 99% trường hợp GET request, ta không dùng useEffect nữa. useEffect chỉ dùng cho side-effect không liên quan data fetching."\_

> **🔥 WHY THIS SESSION EXISTS?** > _"Gọi API thì dễ. Nhưng quản lý Cache, Loading, Error, Retry mới khó. React Query làm hết việc khó đó cho bạn. Junior dùng useEffect, Senior dùng Query."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 💾 Client State vs Server State

> **Instructor Script (QUAN TRỌNG - 10 PHÚT):**  
> _"Trước khi code, ta cần hiểu tại sao React Query tồn tại._
>
> _Có 2 loại State hoàn toàn khác nhau mà nhiều người nhầm lẫn:"_

#### Client State (Local State - Ta kiểm soát 100%)

```tsx
// Client State - Dữ liệu của App, do ta tạo ra
const [isModalOpen, setIsModalOpen] = useState(false);
const [theme, setTheme] = useState("dark");
const [sidebarCollapsed, setSidebarCollapsed] = useState(false);

// ✅ Đặc điểm:
// - Ta tạo ra, ta kiểm soát
// - Không có delay, không có lỗi network
// - Luôn "tươi mới" 100%
// - Lưu trong useState, Zustand, Redux
```

#### Server State (Remote State - Ta chỉ "mượn" từ Server)

```tsx
// Server State - Dữ liệu từ Server, ta chỉ cache nó
const user = { id: 1, name: "John" }; // Lấy từ API /users/me

// ⚠️ Đặc điểm:
// - Ta KHÔNG sở hữu (Server mới là chủ)
// - Có thể CŨ bất kỳ lúc nào (ai đó đổi trên Server)
// - Có thể LỖI (network fail, server crash)
// - Có DELAY (phải chờ network)
// - Cần CACHE để app nhanh
// - Cần SYNC để data không lỗi thời
```

**Comparison Table:**

| Aspect        | Client State      | Server State                    |
| ------------- | ----------------- | ------------------------------- |
| **Nguồn gốc** | App tạo ra        | Server cung cấp                 |
| **Ownership** | App sở hữu 100%   | Chỉ "mượn" tạm thời             |
| **Freshness** | Luôn mới          | Có thể cũ bất kỳ lúc nào        |
| **Network**   | Không liên quan   | Phụ thuộc hoàn toàn             |
| **Error**     | Không có          | Có thể fail (401, 500, timeout) |
| **Công cụ**   | useState, Zustand | **React Query**                 |

> **💡 Rule of Thumb:**  
> _"Nếu data TỪ SERVER → dùng React Query._  
> _Nếu data TỪ USER (click, type, toggle) → dùng useState/Zustand."_

---

### 🔄 Stale-While-Revalidate Strategy

> **Instructor Analogy:**  
> _"Tưởng tượng bạn vào Shopee xem giá iPhone._
>
> _**Cách cũ (useEffect):**  
> Màn hình trắng xóa 2 giây → Loading spinner → Hiện giá._
>
> _**Cách React Query:**  
> Hiện ngay giá cũ (cache) → Ngầm fetch giá mới → Cập nhật lặng lẽ nếu có thay đổi._
>
> _User thấy data NGAY LẬP TỨC, không phải nhìn spinner."_

**Stale-While-Revalidate Flow:**

```
User vào trang
    ↓
[1] Query check cache
    ↓
[2] Có cache? → Render NGAY (stale data)
    ↓
[3] Fetch mới ở background (revalidate)
    ↓
[4] So sánh data mới vs data cũ
    ↓
[5] Khác? → Re-render với data mới
    ↓
[6] Giống? → Không làm gì (tiết kiệm render)
```

**Demo Code:**

```tsx
// Lần 1: User vào trang Profile
useUser(); // → Fetch API → Cache vào ['me'] → Render

// User chuyển sang trang khác rồi quay lại Profile
useUser();
// → [NGAY LẬP TỨC] Render data từ cache (stale)
// → [BACKGROUND] Fetch API kiểm tra có data mới không
// → [NẾU CÓ MỚI] Re-render với data mới
// → [NẾU KHÔNG] Không làm gì cả
```

---

### ⏱️ StaleTime vs GcTime (Cache Time)

> **Instructor Script (CRITICAL - 5 PHÚT):**  
> _"2 khái niệm này làm mọi người rối nhất khi học Query."_

#### StaleTime (Thời gian "tươi")

```tsx
useQuery({
  queryKey: ["products"],
  queryFn: fetchProducts,
  staleTime: 5 * 60 * 1000, // 5 phút
});

// Nghĩa là: Data được coi là "TƯƠI" trong 5 phút
// Trong 5 phút đó:
// - Không fetch lại (dù refocus window, remount component)
// - Data được tin tưởng tuyệt đối
// - Cache hit 100%

// Sau 5 phút:
// - Data thành "STALE" (cũ)
// - Lần tới mount/refocus → Fetch lại để kiểm tra
```

**Default:** `staleTime: 0` (data ngay lập tức cũ)

#### GcTime (Garbage Collection Time - Thời gian "tồn tại")

```tsx
useQuery({
  queryKey: ["products"],
  queryFn: fetchProducts,
  gcTime: 10 * 60 * 1000, // 10 phút
});

// Nghĩa là: Cache TỒN TẠI trong bộ nhớ 10 phút
// Nếu không component nào dùng trong 10 phút → XÓA khỏi RAM

// Ví dụ:
// - User vào trang Products → Cache tạo ra
// - User rời khỏi trang → Cache vẫn còn (inactive)
// - 10 phút sau không ai dùng → Query xóa cache để tiết kiệm RAM
```

**Default:** `gcTime: 5 * 60 * 1000` (5 phút)

**Comparison:**

```tsx
// Scenario: User behavior
staleTime: 1 phút    // Data "tươi" trong 1 phút
gcTime: 5 phút       // Cache tồn tại 5 phút

// Timeline:
[0s] User vào trang → Fetch → Cache ['products']
[30s] User F5 → KHÔNG fetch (còn tươi)
[70s] User F5 → FETCH (đã cũ > 1 phút)
[5 phút] User không dùng → Cache bị xóa → Lần sau phải fetch lại từ đầu
```

> **❗ Rule of Thumb:**
>
> - `staleTime` ngắn (0-30s): Data thay đổi liên tục (chứng khoán, chat)
> - `staleTime` dài (5-10 phút): Data ổn định (profile, settings)
> - `gcTime` luôn > `staleTime` để tận dụng cache
> - Production: `staleTime: 30s, gcTime: 5 phút` là balance tốt

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: PAIN-DRIVEN DEMO (15 mins)

> **⏱️ Timeline:**
>
> - **00-05':** Instructor demo manual fetching
> - **05-10':** Students code along
> - **10-15':** Discussion of pain points

_"Trước khi sung sướng, ta phải khổ trước để thấy giá trị."_

#### Step 1: Manual Fetching (The Painful Way)

Update `src/pages/MePage.tsx` (Viết thử rồi xóa):

```tsx
import { useState, useEffect } from "react";
import { usersApi } from "@/lib/api/users.api";

export default function MePage() {
  // ❌ THE PAINFUL WAY - Phải quản lý 3 states thủ công
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 🚨 Pain Point 1: Phải tự set loading
    setIsLoading(true);

    usersApi
      .getMe()
      .then((res) => {
        // 🚨 Pain Point 2: Phải tự parse response
        setUser(res.data.result);
        setError(null); // Phải nhớ clear error cũ
      })
      .catch((err) => {
        // 🚨 Pain Point 3: Phải tự handle error
        setError(err.message);
        setUser(null); // Phải nhớ clear data cũ
      })
      .finally(() => {
        // 🚨 Pain Point 4: Phải nhớ tắt loading
        setIsLoading(false);
      });
  }, []);
  // 🚨 Pain Point 5: Dependency array trống
  // → Nếu user quay lại trang, nó KHÔNG fetch lại
  // → Data cũ tồn tại mãi mãi
  // → Nếu thêm dependency → Có thể infinite loop

  // 🚨 Pain Point 6: Không có cache
  // User vào trang → Loading 2s
  // User ra khỏi trang
  // User vào lại trang → Loading 2s NỮA (fetch lại từ đầu!)

  // 🚨 Pain Point 7: Không có retry
  // Mạng chập chờn 1 cái → Fail → Phải F5 lại

  // 🚨 Pain Point 8: Race condition
  // User click nhanh 2 lần → 2 requests bay đi
  // Response về không đúng thứ tự → Data sai

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No data</div>; // Edge case phải check

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

> **Instructor Demo:**  
> _"Hãy mở Console và Network tab."_
>
> **Test 1:** Vào trang → Loading → Data hiện  
> **Test 2:** Chuyển sang trang khác → Quay lại → **Loading LẠI 2s!** (No cache)  
> **Test 3:** Tắt mạng (DevTools Offline) → Reload → **Trắng màn hình** (No retry)  
> **Test 4:** Giữ nguyên code, F5 StrictMode → **2 requests cùng lúc!** (Race condition)

**Instructor Ask:** _"Ai thấy code này có vấn đề? Hãy list ra."_

**Expected Answers:**

1. Quá nhiều boilerplate (3 states, loading/error/data)
2. Không cache → Slow UX
3. Không retry → Fragile
4. Race condition → Bug tiềm ẩn
5. Phải nhớ cleanup nếu component unmount mid-fetch
6. Code lặp lại ở mọi component fetch data

> **Conclusion:**  
> _"Nếu app có 20 trang fetch data, bạn phải copy-paste đống code này 20 lần._
>
> _Đó là lý do React Query sinh ra._
>
> _Giờ ta xóa hết code này đi."_ → **DELETE THE PAINFUL CODE**

---

### PHASE 2: SETUP TANSTACK QUERY (30 mins)

> **⏱️ Timeline:**
>
> - **00-10':** Install + Provider setup
> - **10-20':** QueryClient configuration
> - **20-30':** DevTools introduction

#### Step 1: Install Library

```bash
# Terminal
npm install @tanstack/react-query @tanstack/react-query-devtools
```

> **Instructor Note:**  
> _"2 packages riêng biệt:_  
> _- `@tanstack/react-query`: Core library (production)_  
> _- `@tanstack/react-query-devtools`: Debug tool (dev only)"_

---

#### Step 2: Configure Global Provider

Update `src/main.tsx` (hoặc `App.tsx` nếu dùng createBrowserRouter):

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import App from "./App";
import "./index.css";

// ✅ BƯỚC 1: Tạo QueryClient instance
// QueryClient = "Người quản lý" tất cả queries trong app
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // 🎛️ Config mặc định cho TẤT CẢ queries

      // 1. Refetch on Window Focus
      refetchOnWindowFocus: false,
      // ⚠️ Mặc định: true (tự fetch lại khi user quay lại tab)
      // 💡 Học: Tắt để dễ debug (log đỡ nhảy loạn)
      // 💡 Production: Bật lại để data luôn tươi

      // 2. Retry Failed Requests
      retry: 1,
      // ⚠️ Mặc định: 3 lần
      // 💡 Học: Giảm xuống 1 để nhanh thấy lỗi
      // 💡 Production: 2-3 là hợp lý (network chập chờn)

      // 3. Stale Time
      staleTime: 0,
      // ⚠️ Mặc định: 0 (data ngay lập tức "cũ")
      // 💡 Production: 30s - 5 phút tùy data

      // 4. Cache Time (GC Time)
      gcTime: 5 * 60 * 1000,
      // ⚠️ Mặc định: 5 phút
      // 💡 Cache tồn tại 5 phút kể từ khi không còn component nào dùng
    },
  },
});

// ✅ BƯỚC 2: Wrap App với Provider
ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    {/* 
      QueryClientProvider:
      - Giống Context.Provider
      - Cung cấp queryClient cho toàn bộ component tree
      - BẮT BUỘC phải wrap ngoài cùng
    */}
    <QueryClientProvider client={queryClient}>
      <App />

      {/* 
        ReactQueryDevtools:
        - Debug tool ONLY cho development
        - Production: Tự động bị tree-shaking (không vào bundle)
        - Hiện icon hoa đỏ góc dưới màn hình
      */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
);
```

> **🚦 MID-PHASE CHECKPOINT:**  
> _"Run `npm run dev`. Mở trình duyệt._  
> _Góc dưới cùng bên trái thấy icon hoa đỏ chưa?_  
> _Nếu chưa thấy → Check lại import và Provider."_

---

#### Step 3: Understanding React Query DevTools

> **Instructor Demo (10 phút - QUAN TRỌNG):**  
> _"Click vào icon hoa đỏ. Đây là công cụ bạn sẽ dùng 100 lần mỗi ngày."_

**DevTools Interface:**

```
┌─────────────────────────────────────┐
│ 🌸 React Query DevTools             │
├─────────────────────────────────────┤
│ Queries (0)                          │  ← Danh sách tất cả queries
│ Mutations (0)                        │  ← Danh sách tất cả mutations
│ Cache (0)                            │  ← Cache hiện tại
└─────────────────────────────────────┘
```

**Query States (màu sắc quan trọng):**

- 🟢 **Green (fresh):** Data còn "tươi" (trong `staleTime`)
- 🟡 **Yellow (stale):** Data đã "cũ" (ngoài `staleTime`) nhưng còn trong cache
- 🔵 **Blue (fetching):** Đang fetch data
- 🔴 **Red (error):** Fetch bị lỗi
- ⚪ **Gray (inactive):** Không có component nào đang dùng query này

**DevTools Actions:**

```tsx
// Trong DevTools, click vào một query để xem:
{
  queryKey: ['me'],           // Key để identify
  queryHash: '"me"',          // Hash của key
  state: {
    data: { ... },            // Data hiện tại
    error: null,              // Error nếu có
    status: 'success',        // loading | error | success
    fetchStatus: 'idle',      // fetching | paused | idle
  },
  observers: 1,               // Số component đang subscribe
  updatedAt: 1703123456789,   // Timestamp
}
```

> **Instructor Exercise:**  
> _"Mở DevTools lên, để đó. Lát nữa khi ta code Query đầu tiên, bạn sẽ thấy nó xuất hiện ở đây theo real-time."_

---

### PHASE 3: QUERY IMPLEMENTATION (60 mins)

> **⏱️ Timeline:**
>
> - **00-20':** Create custom hook with queryKey explanation
> - **20-40':** Use in component with loading states
> - **40-50':** Query options (enabled, select, refetchInterval)
> - **50-60':** Query key strategies

#### Step 1: Create Custom Hook

Create `src/hooks/useUser.ts`:

> **Instructor Script:**  
> _"Ta không gọi `useQuery` trực tiếp trong component._  
> _Ta luôn bọc nó trong custom hook để:_  
> _1. Tái sử dụng logic_  
> _2. Dễ test_  
> _3. Dễ maintain (đổi API chỉ sửa 1 chỗ)"_

```tsx
import { useQuery } from "@tanstack/react-query";
import { usersApi } from "@/lib/api/users.api";

/**
 * Custom Hook: Fetch current user profile
 *
 * @returns {UseQueryResult} Query result object
 *
 * Usage:
 * const { data, isLoading, error } = useUser();
 */
export const useUser = () => {
  return useQuery({
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // 📌 QUERY KEY (BẮT BUỘC)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    queryKey: ["me"],

    /**
     * 🧠 Query Key Concept:
     *
     * - Key = ID của Cache
     * - Cùng Key = Chung Cache
     * - Khác Key = Khác Cache
     *
     * Ví dụ:
     * ['me'] ≠ ['user'] ≠ ['user', 1] ≠ ['user', 2]
     *
     * Key là Array vì:
     * - Dễ nest: ['user', userId, 'posts', postId]
     * - Dễ invalidate theo pattern
     * - React Query so sánh array theo giá trị (deep equal)
     */

    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // 🔧 QUERY FUNCTION (BẮT BUỘC)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    queryFn: async () => {
      /**
       * queryFn PHẢI:
       * 1. Return Promise
       * 2. Throw error nếu fail (đừng catch!)
       * 3. Return data cần cache (không phải raw response)
       */

      const response = await usersApi.getMe();
      // response shape: { data: { message, result: { user } } }

      // ✅ ĐÚNG: Return chỉ phần data cần thiết
      return response.data.result.user;

      // ❌ SAI: Return cả response (cache thừa thãi)
      // return response;

      /**
       * 🧠 Cache Shape:
       * Query CHỈ cache cái mình return
       * Key ['me'] → Cache: { id: 1, name: 'John', ... }
       *
       * Nếu return response → Cache: { data: { data: { result: ... } } }
       * → Rối khi dùng: data.data.data.result (WTF?)
       */
    },

    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // ⚙️ QUERY OPTIONS (TÙY CHỌN)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    // staleTime: 1000 * 60 * 5, // 5 phút
    /**
     * 💡 StaleTime Usage:
     *
     * 0 (default): Data ngay lập tức "cũ"
     * → Mỗi lần mount component = fetch lại
     * → Tốt cho: Real-time data (chat, stock price)
     *
     * 30s - 5 phút: Data "tươi" trong khoảng này
     * → Mount component = không fetch (dùng cache)
     * → Tốt cho: User profile, settings
     *
     * Infinity: Data KHÔNG BAO GIỜ cũ
     * → Chỉ fetch 1 lần duy nhất
     * → Tốt cho: Static data (country list, categories)
     */

    // enabled: !!accessToken,
    /**
     * 💡 Enabled Option:
     *
     * Conditional fetching:
     * enabled: false → Query không chạy
     * enabled: true → Query chạy
     *
     * Use case:
     * - Chỉ fetch khi có token
     * - Chỉ fetch khi user click button
     * - Dependent queries (fetch B sau khi có data từ A)
     */

    // select: (data) => ({ ...data, fullName: `${data.firstName} ${data.lastName}` }),
    /**
     * 💡 Select Option:
     *
     * Transform data trước khi return về component
     *
     * Lợi ích:
     * - Component chỉ re-render khi TRANSFORMED data thay đổi
     * - Tách logic transform ra khỏi component
     *
     * ❌ Đừng:
     * useEffect(() => setTransformed(transform(data)), [data])
     *
     * ✅ Dùng:
     * select: (data) => transform(data)
     */

    // refetchInterval: 30000,
    /**
     * 💡 RefetchInterval Option:
     *
     * Auto refetch mỗi X milliseconds
     *
     * Use case:
     * - Dashboard cần update liên tục
     * - Real-time monitoring
     *
     * Lưu ý:
     * - Chỉ chạy khi component mount
     * - Tắt khi component unmount
     * - Cân nhắc server load
     */
  });
};
```

> **🚦 CHECKPOINT:**  
> _"Hook đã tạo xong. Giờ ta sẽ dùng nó trong component."_

---

#### Step 2: Use in Component

Update `src/pages/MePage.tsx`:

```tsx
import { useUser } from "@/hooks/useUser";
import { Button } from "@/components/ui/button";

export default function MePage() {
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  // 🎣 CALL THE HOOK
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  const {
    data: user, // ✅ Rename 'data' thành 'user' cho semantic
    isLoading, // true = Lần đầu fetch, chưa có data
    isError, // true = Fetch bị lỗi
    error, // Error object nếu có
    refetch, // Function để fetch lại manually
    isFetching, // true = Đang fetch (dù có data hay không)
  } = useUser();

  /**
   * 🧠 States Breakdown:
   *
   * [FIRST LOAD]
   * isLoading: true   → Show skeleton
   * isFetching: true
   * data: undefined
   *
   * [SUCCESS]
   * isLoading: false
   * isFetching: false
   * data: { user object }
   *
   * [REFETCH (có data cũ)]
   * isLoading: false  → Không show skeleton (có data rồi!)
   * isFetching: true  → Show spinner nhỏ ở góc
   * data: { old data } → UI vẫn hiện data cũ
   *
   * [ERROR]
   * isLoading: false
   * isError: true
   * error: { message, status }
   */

  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  // 🎨 RENDER STATES
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  // STATE 1: LOADING (Lần đầu, chưa có data)
  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-center">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500 mx-auto"></div>
          <p className="mt-4 text-gray-600">Loading profile...</p>
        </div>
      </div>
    );
  }

  // STATE 2: ERROR
  if (isError) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-center max-w-md">
          <div className="text-red-500 text-5xl mb-4">⚠️</div>
          <h2 className="text-2xl font-bold mb-2">
            Oops! Something went wrong
          </h2>
          <p className="text-gray-600 mb-4">
            {error?.message || "Failed to load profile"}
          </p>
          <Button onClick={() => refetch()}>Try Again</Button>
        </div>
      </div>
    );
  }

  // STATE 3: EMPTY (API success nhưng không có data - edge case)
  if (!user) {
    return (
      <div className="text-center p-10">
        <p className="text-gray-500">No user data available</p>
      </div>
    );
  }

  // STATE 4: SUCCESS (Có data)
  return (
    <div className="max-w-4xl mx-auto p-6">
      {/* Header với Refresh Button */}
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">My Profile</h1>

        <Button
          onClick={() => refetch()}
          disabled={isFetching}
          className="relative"
        >
          {/* Show spinner khi đang refetch (có data cũ) */}
          {isFetching && (
            <span className="absolute inset-0 flex items-center justify-center">
              <span className="animate-spin">🔄</span>
            </span>
          )}
          <span className={isFetching ? "opacity-0" : ""}>Refresh</span>
        </Button>
      </div>

      {/* Profile Content */}
      <div className="bg-white shadow rounded-lg p-6">
        <div className="space-y-4">
          <div>
            <label className="text-sm text-gray-500">Name</label>
            <p className="text-lg font-medium">{user.name}</p>
          </div>

          <div>
            <label className="text-sm text-gray-500">Email</label>
            <p className="text-lg">{user.email}</p>
          </div>

          <div>
            <label className="text-sm text-gray-500">User ID</label>
            <p className="text-sm text-gray-600 font-mono">{user._id}</p>
          </div>
        </div>
      </div>

      {/* Debug Info (Optional - để học) */}
      <details className="mt-6">
        <summary className="cursor-pointer text-sm text-gray-500">
          Show Raw Data (Debug)
        </summary>
        <pre className="mt-2 p-4 bg-gray-100 rounded text-xs overflow-auto">
          {JSON.stringify(user, null, 2)}
        </pre>
      </details>
    </div>
  );
}
```

> **Instructor Demo:**  
> _"Mở DevTools React Query."_
>
> **Action 1:** Vào trang `/me`  
> → Thấy query `['me']` xuất hiện màu 🔵 Blue (fetching)  
> → 2 giây sau thành 🟢 Green (fresh)
>
> **Action 2:** Chuyển sang trang khác → Quay lại  
> → Query vẫn 🟢 Green → Data render NGAY LẬP TỨC (cache hit!)  
> → Không thấy Loading screen
>
> **Action 3:** Click "Refresh" button  
> → Query thành 🔵 Blue  
> → Data cũ vẫn hiện  
> → Spinner nhỏ ở góc button
>
> **Action 4:** Đợi 30 giây → Query thành 🟡 Yellow (stale)  
> → F5 trang → Query tự fetch lại để kiểm tra

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên return trong `queryFn`**:

    - _Symptom:_ Data luôn `undefined` dù API status 200.

    * _Cause:_ Arrow function có `{}` nhưng không có `return`.
    * _Fix:_

      ```tsx
      // ❌ SAI
      queryFn: async () => {
        await api.getUser(); // Quên return!
      };

      // ✅ ĐÚNG
      queryFn: async () => {
        return await api.getUser();
      };
      ```

2.  **Key duplicate hoặc quá chung chung**:

    - _Issue:_ Dùng chung key `['user']` cho cả `getMe` và `getUserById`.
    - _Symptom:_ Data bị lẫn lộn giữa các queries.
    - _Fix:_ Key phải cụ thể:

      ```tsx
      // ❌ SAI - Quá chung
      useQuery({ queryKey: ['user'], ... })

      // ✅ ĐÚNG - Specific
      useQuery({ queryKey: ['user', 'me'], ... })
      useQuery({ queryKey: ['user', userId], ... })
      ```

3.  **Lạm dụng `useEffect` để set state từ data**:

    - _Code:_ `useEffect(() => { if (data) setState(data) }, [data])`.
    - _Issue:_ **Anti-pattern lớn nhất!** Tạo ra duplicate state và extra renders.
    - _Fix:_ Dùng trực tiếp `data` trong JSX hoặc `select` option:

      ```tsx
      // ❌ SAI
      const { data } = useUser();
      const [user, setUser] = useState();
      useEffect(() => setUser(data), [data]);

      // ✅ ĐÚNG
      const { data: user } = useUser();
      ```

4.  **Query Key không stable (object mới mỗi render)**:

    - _Symptom:_ Query fetch liên tục, không bao giờ dừng.
    - _Cause:_
      ```tsx
      // ❌ SAI - filters là object mới mỗi render
      const filters = { category };
      useQuery({ queryKey: ['products', filters], ... });
      ```
    - _Fix:_

      ```tsx
      // ✅ ĐÚNG - useMemo
      const filters = useMemo(() => ({ category }), [category]);

      // ✅ BETTER - Destructure trực tiếp
      useQuery({ queryKey: ['products', { category }], ... });
      ```

5.  **Catch error trong queryFn**:

    - _Issue:_ React Query không biết có lỗi, không set `isError = true`.
    - _Fix:_

      ```tsx
      // ❌ SAI - Catch error
      queryFn: async () => {
        try {
          return await api.get();
        } catch (error) {
          console.error(error);
          return null; // React Query nghĩ thành công!
        }
      };

      // ✅ ĐÚNG - Let it throw
      queryFn: async () => {
        return await api.get(); // Throw tự nhiên
      };
      ```

6.  **Quên Provider**:

    - _Symptom:_ Error: `No QueryClient set, use QueryClientProvider`.
    - _Fix:_ Bọc `<QueryClientProvider>` ở root của app.

7.  **Không kiểm tra DevTools**:

    - _Symptom:_ Query không chạy nhưng không biết tại sao.
    - _Fix:_ MỞ DEVTOOLS! Xem query state (fresh/stale/fetching/error).

8.  **Enabled = false nhưng không biết**:
    - _Symptom:_ Query không chạy dù component mount.
    - _Cause:_ `enabled: !!undefined` → `false`
    - _Debug:_
      ```tsx
      enabled: !!accessToken,
        // Log để check:
        console.log("accessToken:", accessToken, "enabled:", !!accessToken);
      ```

---

## 🎓 RULE OF THUMB: REACT QUERY BEST PRACTICES

> **Instructor Script (10 phút - ĐỌC CHẬM):**  
> _"Đây là những nguyên tắc bạn phải nhớ khi dùng React Query production."_

### Rule 1: LUÔN dùng Custom Hook

```tsx
// ❌ BAD - Inline trong component
function Profile() {
  const { data } = useQuery({ queryKey: ["me"], queryFn: fetchMe });
}

// ✅ GOOD - Custom hook
function Profile() {
  const { data } = useUser(); // Reusable, testable, maintainable
}
```

**Lý do:**

- Tái sử dụng logic (DRY principle)
- Dễ test (mock hook, không mock axios)
- Dễ maintain (đổi API chỉ sửa 1 chỗ)
- Semantic naming (đọc code dễ hiểu)

---

### Rule 2: QueryKey phải STABLE

```tsx
// ❌ BAD - Object mới mỗi render
const filters = { category: 'laptop' }; // ← Object mới!
useQuery({ queryKey: ['products', filters], ... });

// ✅ GOOD - useMemo
const filters = useMemo(() => ({ category }), [category]);

// ✅ BETTER - Destructure values
useQuery({ queryKey: ['products', { category, sort }], ... });
```

**Nguyên nhân:** React Query so sánh key theo giá trị (deep equal). Object mới = key mới = fetch lại.

---

### Rule 3: QueryFn PHẢI throw error

```tsx
// ❌ BAD - Catch & return null
queryFn: async () => {
  try {
    return await api.getUser();
  } catch (error) {
    return null; // React Query không biết có lỗi!
  }
};

// ✅ GOOD - Let it throw
queryFn: async () => {
  return await api.getUser(); // Error sẽ throw tự nhiên
};
```

**Nguyên nhân:** React Query dựa vào thrown error để:

- Set `isError = true`
- Trigger retry logic
- Populate `error` object

---

### Rule 4: Chỉ return data cần thiết

```tsx
// ❌ BAD - Cache thừa thãi
queryFn: async () => {
  const response = await axios.get("/users/me");
  return response; // Cache: { data, headers, config, status, ... }
};

// ✅ GOOD - Extract data
queryFn: async () => {
  const { data } = await axios.get("/users/me");
  return data.result.user; // Cache: { id, name, email }
};
```

**Lý do:**

- Cache nhẹ hơn → RAM ít hơn
- Component code sạch: `data.name` thay vì `data.data.result.user.name`

---

### Rule 5: KHÔNG dùng useEffect với query data

```tsx
// ❌ BAD - Duplicate state + Extra render
const { data } = useUser();
const [user, setUser] = useState(null);

useEffect(() => {
  if (data) setUser(data);
}, [data]);

// ✅ GOOD - Dùng trực tiếp
const { data: user } = useUser();

// ✅ GOOD - Transform bằng select
const { data: user } = useQuery({
  queryKey: ["me"],
  queryFn: fetchMe,
  select: (data) => ({
    ...data,
    fullName: `${data.firstName} ${data.lastName}`,
  }),
});
```

**Lý do:** Query data ĐÃ là state rồi. Thêm useState = duplicate và khó maintain.

---

### Rule 6: Enabled cho conditional queries

```tsx
// ❌ BAD - useEffect + manual control
const [shouldFetch, setShouldFetch] = useState(false);

useEffect(() => {
  if (userId && shouldFetch) {
    // Fetch manually...
  }
}, [userId, shouldFetch]);

// ✅ GOOD - enabled option
useQuery({
  queryKey: ["user", userId],
  queryFn: () => fetchUser(userId),
  enabled: !!userId, // Query chỉ chạy khi có userId
});
```

**Use cases:**

- Fetch sau khi user click button
- Dependent queries (A → B → C)
- Conditional based on permissions

---

### Rule 7: StaleTime theo loại data

```tsx
// ❌ BAD - All queries staleTime: 0 (default)
// → Fetch liên tục dù data không thay đổi

// ✅ GOOD - Categorize by update frequency
{
  // Real-time (giá chứng khoán, chat)
  staleTime: 0,

  // Frequent updates (newsfeed, notifications)
  staleTime: 30 * 1000, // 30 giây

  // Semi-stable (product list)
  staleTime: 2 * 60 * 1000, // 2 phút

  // Stable (user profile, settings)
  staleTime: 5 * 60 * 1000, // 5 phút

  // Static (country list, categories)
  staleTime: Infinity,
}
```

---

### Rule 8: KHÔNG duplicate server data vào Zustand

```tsx
// ❌ BAD - Duplicate cache
const { data: user } = useUser();
useEffect(() => {
  userStore.setUser(user); // ❌ Duplicate! React Query đã cache rồi
}, [user]);

// ✅ GOOD - React Query là single source of truth
const { data: user } = useUser(); // Use directly

// ⚠️ Exception: Chỉ lưu vào Zustand khi:
// 1. Non-React context cần access (WebSocket, Service Worker)
// 2. Selective sync (chỉ lưu 1-2 fields, không phải toàn bộ)
```

**Nguyên nhân:** React Query ĐÃ LÀ cache store. Duplicate = waste memory + out of sync bugs.

---

### Rule 9: DevTools luôn bật

```tsx
// ✅ ALWAYS in development
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>;
```

**Lợi ích:**

- Xem query states real-time
- Debug cache invalidation
- Monitor refetch behavior
- Inspect query data

---

### Rule 10: Query Key Strategy

```tsx
// ✅ GOOD - Hierarchical keys
["user", "me"][("user", userId)][("user", userId, "posts")][ // Current user // Specific user // User's posts
  ("user", userId, "posts", { page })
]; // Paginated posts

// Invalidation power:
queryClient.invalidateQueries(["user"]); // All user queries
queryClient.invalidateQueries(["user", userId]); // Specific user + nested
queryClient.invalidateQueries(["user", userId, "posts"]); // Just posts
```

**Pattern:**

1. Resource name first: `['users']`, `['products']`
2. ID if specific: `['user', 123]`
3. Nested relations: `['user', 123, 'orders']`
4. Filters last: `['products', { category, sort }]`

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** _"Khi nào dùng `useEffect` để fetch data?"_
    - **A:** Gần như không bao giờ trong dự án React hiện đại (trừ khi viết lại thư viện fetching).
2.  **Q:** _"StaleTime vs GcTime (CacheTime) khác nhau gì?"_
    - **A:** `staleTime`: Bao lâu thì data được coi là cũ (để tự fetch lại). `gcTime`: Bao lâu thì data bị xóa khỏi bộ nhớ nếu không dùng (để tiết kiệm RAM).

## 7️⃣ IN-CLASS MINI TASK

**Task:** DevTools Playground.

- Mở tab Network, chọn Offline.
- Bấm F5 -> Web lỗi trắng.
- Bấm Refetch button -> Hook báo lỗi.
- Bật lại mạng -> Bấm Refocus vào cửa sổ -> Thấy Query tự chạy lại và hiện data.
- **Kết luận:** React Query tự xử lý reconnect và window focus.

## 8️⃣ HOMEWORK / EXTENSION TASK

**Yêu cầu:** Product List Query.

1.  Tạo Mock API endpoint `/products`.
2.  Viết `useProductsList` hook.
3.  Hiển thị danh sách sản phẩm ra `HomePage`.
4.  Thử cấu hình `staleTime: 10 giây`. Trong 10 giây đó, F5 hoặc switch tab sẽ không thấy network request nào gửi đi (Cache hit).

## 9️⃣ CHECKPOINT & EVALUATION

- **Signal:** Mở React Query DevTools thấy query `['me']` màu xanh lá (fresh) hoặc vàng (stale).
- **Signal:** Code trong component sạch, không còn `useState`, `useEffect`.
- **Behavior:** Bấm refetch button, thấy spin loading xoay nhẹ rồi hiện data mới.

## 🔟 TEACHING NOTES

- **Slow Down:** Giải thích sự khác biệt giữa `isLoading` (lần đầu không có data) và `isFetching` (có data cũ, đang lấy data mới).
- **Emphasis:** "Server State" là khái niệm khó. Hãy ví dụ: Giống như xem giá chứng khoán. Giá trên màn hình (Client) chỉ là ảnh chụp quá khứ, giá thật (Server) có thể đã đổi. Query giúp ta sync 2 cái đó.
- **Red Flag:** Học viên cố gán response vào Redux/Zustand. Nhắc: "React Query chính là Cache Store rồi, đừng copy data ra chỗ khác trừ khi rất cần thiết".
