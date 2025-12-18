# LESSON PLAN: SESSION 07 - AXIOS LAYER & INTERCEPTORS

## 1️⃣ SESSION OVERVIEW
- **Title:** The Gatekeeper: Building a Production-Ready Axios Layer
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will stop using `fetch()`/`axios` directly in components and build a centralized API layer that handles Auth Token attachment and Error Normalization automatically.
- **Outcome:** A robust `apiClient` that automatically attaches JWT tokens, interprets error codes (401, 422), and attempts to refresh token once if expired.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Nếu component là 'tay chân', thì API Layer chính là 'hệ thần kinh'. Tay chân không nên tự ý làm việc. Nó cần một cơ chế trung gian để điều phối._

_Rất nhiều bạn Newbie code theo kiểu: Mỗi lần cần gọi API là lại `axios.get()`, rồi thủ công thêm header `Bearer Token`. Lỡ Backend đổi tên header? Bạn sửa 100 chỗ. Lỡ token hết hạn? Code bạn chết đứng._

_Hôm nay ta xây dựng 'Cổng hải quan' (Interceptors). Mọi request đi ra đều phải dán tem (Token). Mọi request đi vào đều phải kiểm tra lỗi. Đặc biệt, ta sẽ làm tính năng khó nhằn nhất: Tự động xin cấp lại Token (Refresh Token) khi nó hết hạn mà User không hề hay biết."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Đây là bài học phân loại Junior và Professional. Nếu không có lớp Axios này, code của bạn sẽ cực kỳ lộn xộn, khó bảo trì và User sẽ bị đá ra Login liên tục. Hôm nay ta sẽ giải quyết triệt để vấn đề đó."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🚧 Interceptors (Trạm kiểm soát)
- **Request Interceptor:** Chặn request TRƯỚC khi bay ra khỏi App. Dùng để nhét Token vào Header.
- **Response Interceptor:** Chặn response TRƯỚC khi về tới Component. Dùng để xử lý lỗi chung (401, 500) hoặc convert dữ liệu.

### 🔄 Refresh Token Flow (Simplified)
1. App gọi API A -> Backend trả về **401 Unauthorized**.
2. Interceptor bắt được 401.
3. App âm thầm gọi API **Refresh Token**.
4. Nếu thành công -> Lưu token mới -> Tự động gọi lại API A.
5. Nếu thất bại -> Logout.

> **⚠️ REALITY CHECK:**
> _"Ngoài đời thật, Refresh Token Flow phức tạp hơn nhiều (xử lý race condition, locking, nhiều tab). Ở đây ta học Pattern và Tư duy cốt lõi, phiên bản Simplified này đủ dùng cho các dự án vừa và nhỏ."_

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: SETUP AXIOS INSTANCE (30 mins)

#### Step 1: Install & Config
```bash
npm install axios
```

#### Step 2: Create Base Client
Create `src/lib/http/apiClient.ts`:
_"Đây là file quan trọng nhất dự án. Làm ơn gõ chính xác từng dòng."_

```ts
import axios from 'axios';

// Create instance
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL, // Nhớ config .env
  headers: {
    'Content-Type': 'application/json',
  },
  timeout: 10000, // 10s timeout
});

export default apiClient;
```

#### Step 3: Service Layer Pattern
Create `src/lib/api/users.api.ts`:
_"Ta không gọi axios trong component. Ta gọi Service."_

```ts
import apiClient from "@/lib/http/apiClient";

export const usersApi = {
  getMe: () => apiClient.get('/users/me'),
  // ... các API khác
};
```

---

### PHASE 2: REQUEST INTERCEPTOR (Attach Token) (30 mins)

#### Step 1: Connect Zustand
Update `src/lib/http/apiClient.ts`:
_"Móc token từ Két sắt (Zustand) dán vào request."_

```ts
import { useAuthStore } from '@/stores/auth.store';

// ... axios.create code

// Add Request Interceptor
apiClient.interceptors.request.use(
  (config) => {
    // 1. Lấy token từ Zustand (không dùng hook, lấy trực tiếp)
    const accessToken = useAuthStore.getState().accessToken;

    // 2. Nếu có token, attach vào Header
    if (accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    }

    return config;
  },
  (error) => Promise.reject(error)
);
```

> **📌 INSTRUCTOR NOTE:**
> _"Tại sao dùng `useAuthStore.getState()`? Vì file này là file .ts thường, không phải React Component, nên không dùng hook được."_

---

### PHASE 3: RESPONSE INTERCEPTOR (Refresh Token) (60 mins)

#### Step 1: Handle Normal Errors
Update `src/lib/http/apiClient.ts`:

```ts
// Add Response Interceptor
apiClient.interceptors.response.use(
  (response) => response.data, // ⚠️ Return data trực tiếp. Component sẽ nhận được user thay vì { data: user }
  async (error) => {
    const originalRequest = error.config;
    const status = error.response?.status;

    // TODO: Handle 401 here
    
    return Promise.reject(error);
  }
);
```

#### Step 2: Implement Auto-Refresh Logic
_"Logic này hơi xoắn não. Tập trung cao độ nhé."_

```ts
    // ... inside async (error) block
    
    // Nếu lỗi 401 và chưa từng retry (tránh lặp vô tận)
    if (status === 401 && !originalRequest._retry) {
      (originalRequest as any)._retry = true; // ⚠️ Cast any để tránh lỗi TS

      try {
        const refreshToken = useAuthStore.getState().refreshToken;
        if (!refreshToken) throw new Error("No refresh token");

        // 1. Gọi API xin token mới
        // Lưu ý: Dùng axios thường để tránh dính interceptor của apiClient
        const response = await axios.post(`${import.meta.env.VITE_API_URL}/users/refresh-token`, { 
          refresh_token: refreshToken 
        });

        const { access_token, refresh_token } = response.data.result;

        // 2. Lưu token mới vào store
        useAuthStore.getState().setTokens(access_token, refresh_token);

        // 3. Update header của request cũ
        originalRequest.headers.Authorization = `Bearer ${access_token}`;

        // 4. Gọi lại request cũ
        return apiClient(originalRequest);

      } catch (refreshError) {
        // Nếu refresh cũng fail -> Logout luôn
        useAuthStore.getState().clearTokens();
        window.location.href = '/login'; // Redirect cứng
        return Promise.reject(refreshError);
      }
    }
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Circular Dependency**:
    *   *Bug:* Import `apiClient` vào `auth.store` rồi lại import `auth.store` vào `apiClient`.
    *   *Fix:* Cấu trúc file cẩn thận. `apiClient` chỉ nên import store, store không nên import lại client (nếu store cần gọi API, hãy truyền client vào hàm action).
2.  **Infinite Loop 401**:
    *   *Bug:* API Refresh Token cũng bị 401 -> Interceptor lại bắt -> Lại gọi refresh -> Vòng lặp chết.
    *   *Fix:* Dùng `axios` gốc (không interceptor) để gọi API refresh token. Hoặc check URL, nếu là URL refresh thì không chặn.
3.  **Quên `_retry` flag**:
    *   *Bug:* Vòng lặp vô tận nếu token mới vẫn sai.
    *   *Fix:* Luôn check `!originalRequest._retry` trước khi refresh.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ta dùng `useAuthStore.getState()` thay vì `useAuthStore()`?"*
    *   **A:** Vì `interceptors` chạy trong môi trường JS thuần, không phải trong React Render Cycle. Hook `useAuthStore()` chỉ chạy được trong Component.
2.  **Q:** *"Điều gì xảy ra nếu ta mở 5 tab, và cả 5 tab đều gọi API cùng lúc khi token hết hạn?"*
    *   **A:** (Câu hỏi nâng cao) Cả 5 tab sẽ đua nhau gọi Refresh Token 5 lần. Đây là vấn đề **Race Condition**. Cách xử lý: **Single-flight pattern** (khóa API, chỉ request đầu tiên được refresh, các request sau chờ). Nhưng trong khóa này ta dùng bản Simplified để dễ hiểu.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Test Interceptor.
1.  Vào `LoginPage`, sửa nút Login để gọi `usersApi.getMe()`.
2.  Set token giả trong `localStorage` thành chuỗi `abc` (token sai).
3.  Bấm nút -> Quan sát Network Tab.
4.  Kỳ vọng: Thấy request `/me` fail 401 -> Ngay lập tức thấy request `/refresh` chạy -> Rồi lại thấy `/me` chạy lại (hoặc fail nếu logic refresh chưa hoàn thiện).

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Normalize Error.
1.  Backend trả về lỗi 422 có dạng: `{ errors: { email: "Email invalid", password: "Too short" } }`.
2.  Hãy sửa Response Interceptor để khi gặp 422, ta biến đổi error object đó thành dạng dễ dùng hơn cho React Hook Form.
3.  Tạo hàm `handleApiError` để toast message lỗi ra màn hình (dùng Sonner).

## 9️⃣ CHECKPOINT & EVALUATION
- **Code Review:** Kiểm tra file `apiClient.ts`. Logic `retry` phải rõ ràng.
- **Behavior:** Học viên xóa bớt 1 ký tự trong Access Token (ở Local Storage) -> F5 trang -> Web vẫn hoạt động bình thường (do tự động refresh).

## 🔟 TEACHING NOTES
- **Slow Down:** Phần logic `try...catch` trong interceptor cực kỳ rối. Hãy vẽ sơ đồ luồng đi của Request lên bảng.
- **Emphasis:** Nhấn mạnh việc **Không dùng Interceptor Client** để gọi API refresh. Đây là lỗi sai kinh điển.
- **Red Flag:** Học viên dùng `window.location.reload()` -> Tệ UX.
- **Clarification:** Tại sao dùng `window.location.href` ở Interceptor mà không dùng `navigate`?
  - _"Vì Interceptor chạy file `.ts` rời, nằm ngoài React Tree nên không dùng được hook `useNavigate`. Đây là trường hợp bất khả kháng phải dùng `window.location` để redirect cứng."_
