# LESSON PLAN: SESSION 11 - PROJECT SETUP & AUTH CORE

## 1️⃣ SESSION OVERVIEW
- **Title:** The Foundation: Project Setup & Authentication Core
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will initialize the final "Shopping Cart" project, setup production-grade architecture (Folder structure, Axios, QueryClient), and implement the Authentication feature (Login/Register) fully connected to the backend.
- **Outcome:** A running React App with a clean folder structure, functional Login/Register pages that validate input, call the API, and save tokens to LocalStorage/Zustand.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Chúc mừng các bạn đã tốt nghiệp phần 'Lý thuyết'.
Mười buổi vừa qua, chúng ta học từng mảnh ghép rời rạc: cách dùng Form, cách dùng API, cách cache data.
Hôm nay, chúng ta chính thức bước vào giải đấu thật sự: **Project Shopping Cart**.

Trong 3 tuần tới, các bạn sẽ không viết code demo nữa.
Các bạn sẽ viết code mà bạn có thể tự tin đưa vào CV.
Hôm nay là buổi quan trọng nhất: **Đổ móng nhà**.
Nếu hôm nay ta setup sai folder, 2 tuần sau ta sẽ khóc thét vì code rối loạn.
Nếu hôm nay ta làm sai Flow Login, toàn bộ tính năng mua hàng sau này sẽ sập.

Nhiệm vụ hôm nay:
1. Setup dự án từ con số 0.
2. Build xong chức năng Đăng ký và Đăng nhập.
Sẵn sàng chưa? Mở Terminal lên!"_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Project Setup là kỹ năng 'ngầm' của Senior. Junior thường nhảy vào code ngay. Senior dành 2 tiếng đầu tiên để sắp xếp ngăn nắp mọi thứ. Hôm nay tôi dạy các bạn tư duy đó."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Feature-Driven Folder Structure
- Thay vì chia theo `components/`, `hooks/` chung chung.
- Ta sẽ ưu tiên chia theo **Feature** (Tính năng) khi dự án lớn.
- Nhưng với dự án tầm trung này, ta dùng Hybrid:
    - `src/features/auth`: Chứa tất cả form, hook login/register.
    - `src/components/ui`: Các nút bấm, input dùng chung (shadcn).
    - `src/lib`: Code cấu hình (Axios, Utils).

### 🔐 Auth Core Flow
1. **User nhập form:** Validate client (Zod).
2. **Submit:** Gọi API (Axios).
3. **Success:** Server trả Token -> Lưu vào Client Store (Zustand + Persistence).
4. **Redirect:** Chuyển hướng sang trang Dashboard.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: SCAFFOLDING & ARCHITECTURE (40 mins)

#### ⏱️ TIMELINE
- **00-10’:** Init Project + Install Deps
- **10-20’:** Folder Structure Explanation
- **20-40’:** HTTP Client & Env Setup

#### Step 1: Initialize Vite Project
**Instructor Script:**
_"Bây giờ tôi làm mẫu, các bạn làm theo y hệt. Đừng sáng tạo lúc này. Ta sẽ dùng template `react-ts`."_

```bash
npm create vite@latest shopping-cart -- --template react-ts
cd shopping-cart
# Instructor Explain: "Ta cần cài đủ đồ chơi ngay từ đầu để tránh phải dừng lại cài lắt nhắt."
npm install
npm install axios zustand @tanstack/react-query react-router-dom react-hook-form zod @hookform/resolvers clsx tailwind-merge lucide-react sonner
```
*(Chờ install xong, setup Tailwind v4 theo hướng dẫn ở Session 06 - Copy file index.css)*

#### Step 2: Create Folder Structure
**Instructor Script:**
_"Hãy tưởng tượng folder `src` là cái tủ quần áo. Nếu vứt lộn xộn, 3 ngày sau tìm cái áo sơ mi không ra. Ta cần ngăn nắp."_

```bash
mkdir src/components/ui src/components/layouts src/pages src/lib src/hooks src/stores src/features/auth
```
**Instructor Explain:**
- `features/auth`: Chứa tất cả những gì liên quan đến Auth (API, Form, Hook).
- `lib`: Những thứ cấu hình (Axios, QueryClient) ít khi sửa.

#### Step 3: Setup Libs (Axios & Query)
Create `src/lib/http/apiClient.ts`:
**Instructor Script:**
_"Đây là `apiClient` huyền thoại ta đã viết ở Buổi 7. Copy vào, nhưng nhớ rule: **Tuyệt đối không hardcode URL**."_

```ts
// ... (Code apiClient chuẩn với Interceptors - xem lại Session 07)
// Instructor Note: Nhắc học viên tạo file .env: VITE_API_URL=...
```

**🚦 MID-SESSION CHECKPOINT**
- Chạy `npm run dev`.
- Mở App lên thấy màn hình trắng hoặc chữ Hello World.
- Console không báo đỏ.
- **Nếu >30% lớp lỗi:** DỪNG LẠI, đi debug hết mới sang Phase 2.

Create `src/main.tsx`:
_"Wrap App với Providers."_

```tsx
// ... QueryClientProvider configuration
```

---

### PHASE 2: AUTH STORE & API (30 mins)

#### Step 1: Auth Store
Create `src/stores/auth.store.ts`:
_"Nơi cất giữ chìa khóa vào nhà (Token)."_

```ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  accessToken: string | null;
  refreshToken: string | null;
  user: any | null; // Tạm thời any, sẽ typed sau
  setTokens: (access: string, refresh: string) => void;
  setUser: (user: any) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      accessToken: null,
      refreshToken: null,
      user: null,
      setTokens: (at, rt) => set({ accessToken: at, refreshToken: rt }),
      setUser: (u) => set({ user: u }),
      clearAuth: () => set({ accessToken: null, refreshToken: null, user: null }),
    }),
    { name: 'auth-storage' }
  )
);
```

#### Step 2: Auth Service
Create `src/features/auth/auth.api.ts`:
_"Tập hợp các API gọi tới BE."_

```ts
import apiClient from "@/lib/http/apiClient";

export const authApi = {
  login: (body: any) => apiClient.post('/users/login', body),
  register: (body: any) => apiClient.post('/users/register', body),
  // ...
};
```

---

### PHASE 3: AUTH UI FLOW (45 mins)

#### Step 1: Login Form (RHF + Zod)
Create `src/features/auth/components/LoginForm.tsx`:
_"Form đăng nhập chuẩn chỉ."_

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useMutation } from '@tanstack/react-query';
import { authApi } from '../auth.api';
import { useAuthStore } from '@/stores/auth.store';
import { useNavigate } from 'react-router-dom';
import { toast } from 'sonner';

// 1. Define Schema
// Instructor Explain: "Luôn định nghĩa luật chơi trước. Zod là luật sư."
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export const LoginForm = () => {
  const navigate = useNavigate();
  const setTokens = useAuthStore(s => s.setTokens);

  // 2. Setup Form
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(loginSchema)
  });

  // 3. Setup Mutation
  // Instructor Explain: "Login là hành động thay đổi, nên dùng Mutation. Đừng dùng Query."
  const loginMutation = useMutation({
    mutationFn: authApi.login,
    onSuccess: (res) => {
      // 4. Save Tokens
      // Instructor Explain: "Server trả về chìa khóa, ta phải cất ngay vào túi (Zustand)."
      setTokens(res.data.result.access_token, res.data.result.refresh_token);
      
      toast.success("Login Success!");
      navigate('/');
    },
    onError: (err: any) => {
      toast.error(err.response?.data?.message || "Login Failed");
    }
  });

  return (
    // Instructor Warn: "Nhớ bọc hàm mutate trong handleSubmit để RHF chạy validation trước."
    <form onSubmit={handleSubmit((d) => loginMutation.mutate(d))} className="space-y-4">
      {/* Input Email */}
      <div>
        <input {...register('email')} className="border p-2 w-full" placeholder="Email" />
        {errors.email && <p className="text-red-500">{errors.email.message as string}</p>}
      </div>
      
      {/* Input Password */}
      <div>
         <input {...register('password')} type="password" className="border p-2 w-full" placeholder="Password" />
      </div>

      <button disabled={loginMutation.isPending} className="bg-blue-500 text-white p-2 w-full">
        {loginMutation.isPending ? "Loading..." : "Login"}
      </button>
    </form>
  )
}
```

#### Step 2: Register Page
_"Tương tự Login, bài tập cho các bạn làm."_

### 🚫 ANTI-PATTERNS (CẤM LÀM)
- **Gọi `loginMutation.mutate()` ngoài `onSubmit`:** Sẽ bypass validation của RHF -> Lỗi 422 từ server.
- **Đặt API call trực tiếp trong component:** Code rác, khó tái sử dụng. Bắt buộc dùng `auth.api.ts`.
- **Hardcode `localhost:3000`:** Deploy lên mạng sẽ chết ngay. Phải dùng env var.

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên `.env`**:
    *   *Symptom:* Gọi API bị về `localhost` của chính frontend hoặc `undefined`.
    *   *Fix:* Check file `.env`, check `import.meta.env.VITE_API_URL`. Nhớ restart server sau khi sửa env.
2.  **Lỗi CORS**:
    *   *Symptom:* API gọi bị chặn đỏ lòm trong console.
    *   *Fix:* Thường do gọi sai URL backend (thiếu `/api` hoặc sai port). Check network tab xem Request URL là gì.
3.  **Zustand không lưu**:
    *   *Symptom:* F5 mất token.
    *   *Fix:* Quên middleware `persist` trong config store.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ta chia folder `features/auth` mà không để chung trong `components`?"*
    *   **A:** Để code Auth (API, Form, Hook) nằm chung 1 chỗ (Co-location). Sau này muốn bưng tính năng Auth sang dự án khác, chỉ cần copy 1 folder là xong.
2.  **Q:** *"AccessToken lưu localStorage có an toàn không?"*
    *   **A:** Tạm ổn cho dự án này. Chuẩn cao cấp nhất là `httpOnly Cookie`. Nhưng với SPA đơn giản, localStorage + short-lived access token là chấp nhận được.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Customize Login Form.
- Thêm logo vào trên Form.
- Thêm link "Chưa có tài khoản? Đăng ký ngay" trỏ sang trang Register.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Complete Register Page.
1. Tạo `src/pages/RegisterPage.tsx`.
2. Dùng `RegisterForm` (tự viết).
3. Validate: Password & Confirm Password match.
4. Success -> Redirect về Login.

## 9️⃣ CHECKPOINT & EVALUATION
- **Code:** Folder `src` gọn gàng, đúng cấu trúc.
- **Behavior:** Đăng nhập thành công -> Chuyển trang -> F5 không bị logout.
- **Console:** Không có lỗi đỏ (ngoại trừ lỗi do cố tình nhập sai pass).

## 🔟 TEACHING NOTES
- **Slow Down:** Giai đoạn setup folder đầu giờ. Đừng rush. Hãy giải thích tại sao file này nằm ở đây.
- **Support:** Sẽ có nhiều bạn lỗi cấu hình Tailwind v4. Hãy chuẩn bị sẵn file `index.css` chuẩn để gửi qua chat cho họ copy nếu debug quá lâu.
