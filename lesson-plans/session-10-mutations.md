# LESSON PLAN: SESSION 10 - MUTATIONS & FULL AUTH FLOW

## 1️⃣ SESSION OVERVIEW
- **Title:** The Cycle Complete: Mutations, Invalidation, and Real Auth Flow
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will complete the Authentication functionality by implementing Login, Register, Logout, and Profile Update using `useMutation`. Crucially, they will learn how to synchronize Server State (React Query) with Client State (Zustand & UI) after a mutation.
- **Outcome:** A fully functional Auth System where a user can Register -> Login -> See Profile -> Update Profile -> Logout, with the UI reflecting state changes instantly.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Chúng ta đã đi được 90% chặng đường của Phase 1. 
- Ta có UI Form xịn (Zod/RHF).
- Ta có API Layer cứng (Axios).
- Ta có Cache thông minh (React Query).

Hôm nay là lúc lắp ráp tất cả lại. Chúng ta sẽ làm tính năng **Mutation** (Thay đổi dữ liệu).
Khác với Query (chỉ đọc), Mutation có 'tác dụng phụ' (Side Effects):
1. Login xong -> Phải lưu Token, chuyển trang.
2. Update Profile xong -> Phải báo Cache 'ê, dữ liệu cũ rồi, đi lấy lại đi'.
3. Logout xong -> Phải xóa sạch sành sanh mọi thứ.

Đây là buổi học quan trọng nhất để các bạn hiểu 'Vòng đời dữ liệu' trong React hiện đại."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Học viên thường chỉ biết gọi API, nhưng không biết làm gì sau đó. Kết quả là Login xong phải F5 mới thấy tên mình, hoặc Logout xong vẫn thấy thông tin cũ. Buổi này chữa dứt điểm bệnh đó."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 💥 Mutation vs Query
- **Query:** Idempotent (Chạy nhiều lần kết quả như nhau). Tự động chạy khi mount. Dùng để **Lấy** dữ liệu.
- **Mutation:** Side-effect (Thay đổi server). Chỉ chạy khi user bấm nút. Dùng để **Tạo/Sửa/Xóa**.

### ♻️ Invalidation (Làm mới)
- Khi ta Sửa dữ liệu (Mutation), dữ liệu trong Cache của Query lập tức trở thành "Rác" (Stale).
- Ta cần lệnh cho Query Client: `invalidateQueries(['me'])`.
- Ngay lập tức, React Query sẽ âm thầm gọi lại API `/me` để lấy dữ liệu mới nhất về. UI tự update mà không cần F5.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: LOGIN MUTATION (40 mins)

#### Step 1: Create `useAuth` Hook
Create `src/hooks/useAuth.ts` (hoặc `useAuthMutations.ts` cho rõ nghĩa):
> **Note:** _"Tên file là `useAuth` cho gọn, nhưng bản chất bên trong chứa các Mutation (Hành động) liên quan đến Auth."_

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '@/lib/api/users.api';
import { useAuthStore } from '@/stores/auth.store';
import { useNavigate } from 'react-router-dom';
import { toast } from 'sonner';

export const useLoginMutation = () => {
  const navigate = useNavigate();
  const setTokens = useAuthStore((state) => state.setTokens);
  
  return useMutation({
    mutationFn: (body: any) => usersApi.login(body), // Gọi API
    onSuccess: (data) => {
      // 1. Lưu token vào Zustand (và LocalStorage)
      setTokens(data.result.access_token, data.result.refresh_token);

      // 2. Thông báo & Chuyển trang
      toast.success("Đăng nhập thành công!");
      navigate("/me");

      // Note: Các Query như ['me'] sẽ tự động chạy lại (refetch) nhờ logic Axios Interceptor + Query Retry, ta không cần gọi refetch tay.
    },
    onError: (error: any) => {
      // Xử lý lỗi (Backend trả về message trong error.response.data.message)
      toast.error(error.response?.data?.message || "Đăng nhập thất bại");
    },
  });
};
```

#### Step 2: Use in Login Page
Update `src/pages/LoginPage.tsx`:

```tsx
import { useLoginMutation } from '@/hooks/useAuth';

export default function LoginPage() {
  const loginMutation = useLoginMutation();
  // ... useForm setup

  const onSubmit = (data) => {
    // Kích hoạt mutation
    loginMutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* ... inputs */}
      
      {/* Disable nút khi đang loading để tránh spam click */}
      <Button type="submit" disabled={loginMutation.isPending}>
        {loginMutation.isPending ? "Đang xử lý..." : "Đăng nhập"}
      </Button>
    </form>
  );
}
```

---

### PHASE 2: UPDATE PROFILE & INVALIDATION (40 mins)

#### Step 1: Create `useUpdateProfile` Hook
In `src/hooks/useUser.ts` (hoặc `useAuth.ts` tùy cách tổ chức):

```tsx
export const useUpdateProfileMutation = () => {
  const queryClient = useQueryClient(); // Lấy client để ra lệnh

  return useMutation({
    mutationFn: (body: any) => usersApi.updateMe(body),
    onSuccess: () => {
      // 🔑 MAGIC HAPPENS HERE:
      // Báo cho React Query biết: Cache của key ['me'] đã cũ rồi.
      // queryClient chỉ định: "Đánh dấu cũ (stale) VÀ tự động fetch lại ngay lập tức (refetch)".
      queryClient.invalidateQueries({ queryKey: ['me'] });

      // ⚠️ WARNING: KHÔNG set state thủ công (setUser) ở đây.
      // Hãy để Query tự fetch lại "Single Source of Truth" từ Server.

      toast.success("Cập nhật hồ sơ thành công!");
    },
  });
};
```

#### Step 2: Implement Update Form
In `src/pages/MePage.tsx`:
_"Tạo một form nhỏ để sửa Bio hoặc Tên."_

```tsx
// ... imports

export default function MePage() {
  const { data: user } = useUser(); // Lấy data từ cache
  const updateMutation = useUpdateProfileMutation();

  const handleUpdate = () => {
    updateMutation.mutate({ name: "New Name " + Date.now() });
  };

  return (
    <div>
      <h1>Hello, {user?.name}</h1>
      <Button 
        onClick={handleUpdate} 
        disabled={updateMutation.isPending}
      >
        Đổi tên ngẫu nhiên
      </Button>
      {/* Ngay khi bấm xong và API 200, cái tên ở thẻ h1 sẽ tự đổi */}
    </div>
  );
}
```

---

### PHASE 3: LOGOUT FLOW (Real Logout) (30 mins)

#### Step 1: Create `useLogoutMutation`
_"Logout không chỉ là xóa token ở client. Phải báo backend xóa Refresh Token trong DB nữa."_

```tsx
export const useLogoutMutation = () => {
  const navigate = useNavigate();
  const queryClient = useQueryClient();
  const { refreshToken, clearTokens } = useAuthStore();

  return useMutation({
    mutationFn: () => usersApi.logout({ refresh_token: refreshToken }),
    onSuccess: () => {
      // 1. Xóa token trong store
      clearTokens();
      
      // 2. Xóa sạch mọi cache của React Query (tránh user sau thấy data user trước)
      queryClient.removeQueries(); 

      // 3. Redirect
      navigate("/login");
      toast.info("Đã đăng xuất");
    },
    onError: () => {
      // Dù API lỗi (VD: Token hết hạn từ trước), Client vẫn phải Logout để đảm bảo UX
      clearTokens();
      queryClient.removeQueries();
      navigate("/login");
    }
  });
};
```

#### Step 2: Use in Header
Update `src/components/layouts/MainLayout.tsx`.

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên `isPending`**:
    *   *Symptom:* User bấm nút Login 10 lần liên tiếp vì thấy mạng lag -> Gọi 10 API login -> Server ban IP.
    *   *Fix:* Luôn `disabled={mutation.isPending}` trên nút submit.
2.  **Quên `invalidateQueries`**:
    *   *Symptom:* Update profile thành công, API báo 200, nhưng tên trên màn hình vẫn cũ. F5 mới thấy đổi.
    *   *Fix:* Kiểm tra xem `queryKey` trong `invalidateQueries` có khớp hoàn toàn với `queryKey` trong `useUser` không.
3.  **Logout không gọi API**:
    *   *Issue:* Chỉ xóa localStorage. Token vẫn sống trên server. Nếu hacker trộm được token đó vẫn dùng được.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao `invalidateQueries` lại tốt hơn việc tự set lại data mới thủ công?"*
    *   **A:** `invalidate` đảm bảo sự thật duy nhất (Single Source of Truth) là Server. Nếu ta tự set tay ở Client, có thể logic client khác server -> Dữ liệu sai lệch (Out of sync).
2.  **Q:** *"Khi nào dùng `mutation.mutateAsync` thay vì `mutate`?"*
    *   **A:** Khi bạn cần `await` kết quả của mutation ngay tại chỗ gọi hàm (ví dụ để chạy logic tiếp theo trong cùng 1 function). Bình thường dùng `mutate` (callback style) là đủ và an toàn hơn.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Register Flow.
- Tự viết `useRegisterMutation`.
- Logic:
    - Gọi API Register.
    - Thành công -> Thông báo "Đăng ký thành công, vui lòng đăng nhập".
    - Chuyển hướng sang trang `/login`. (Không tự login ngay, để user tập thói quen nhập liệu).

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Change Password Flow.
1.  Tạo UI Change Password (Old pass, New pass, Confirm new pass).
2.  Dùng `useMutation` gọi API change-password.
3.  Thành công -> Logout luôn (Force user login lại với pass mới).

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Login -> Chuyển trang mượt mà.
- **Signal:** Update Profile -> UI tự cập nhật tên mới mà không cần reload trang.
- **Signal:** Logout -> Không còn truy cập được trang `/me` (Giả định đã bọc `RequireAuth` từ Session 04).

## 🔟 TEACHING NOTES
- **Slow Down:** Khái niệm "Stale" và "Invalidate". Hãy dùng ví dụ: "Invalidate giống như xả nước bồn cầu. Dữ liệu cũ trôi đi, dữ liệu mới sạch sẽ được bơm vào."
- **Emphasis:** Sự khác biệt giữa `removeQueries` (xóa sạch, reset về null) và `invalidateQueries` (đánh dấu cũ để fetch lại). Logout dùng `remove`, Update dùng `invalidate`.
- **Red Flag:** Học viên gọi `mutate` ngay trong body component (giống gọi API trong render). Phải gọi trong event handler (`onClick`, `onSubmit`).
