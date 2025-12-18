# LESSON PLAN: SESSION 14 - UI POLISH & UX STATES

> **[NEW - SPLIT FROM ORIGINAL SESSION 14]**

## 1️⃣ SESSION OVERVIEW

- **Title:** Production Polish: Loading, Error, and Empty States
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will transform their "working" app into a "production-ready" app by implementing comprehensive UI states (Loading, Error, Empty) that handle all edge cases gracefully.
- **Outcome:** An app with smooth UX featuring Skeletons, Spinners, consistent error handling, and professional empty states.

## 2️⃣ INSTRUCTOR OPENING SCRIPT

\_"Chào các bạn. Code chạy được là **Level 1** (Junior). Code chạy mượt, báo lỗi đẹp, xử lý edge case là **Level 2** (Mid-level).

Hôm nay chúng ta sẽ không viết thêm tính năng mới. Chúng ta sẽ làm 1 việc duy nhất nhưng cực kỳ quan trọng: **Polish** - Trang điểm cho ứng dụng.

Hãy nhìn vào app các bạn hiện tại:

- Bấm Login → Màn hình đơ 2 giây → Không biết có đang xử lý không?
- Vào Profile → Màn hình trắng xóa → Rồi bỗng dưng data xuất hiện?
- Lỗi mạng → App im lìm không báo gì?

Đây là dấu hiệu của app 'học sinh'. Hôm nay ta sẽ nâng nó lên level 'chuyên nghiệp'."\_

> **🔥 WHY THIS SESSION EXISTS?** > _"User không quan tâm code bạn đẹp hay xấu. Họ chỉ quan tâm: 'Tôi bấm nút này rồi, giờ phải làm gì?' Nếu app không trả lời câu hỏi đó (bằng Loading/Error states), user sẽ nghĩ app bị treo và tắt đi."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🚦 The 3-State Rule (Luật 3 Trạng Thái)

Mọi thao tác async (API call, file upload, v.v.) đều phải có **đủ 3 mặt**:

```
┌─────────────┐
│   LOADING   │ ← User đang chờ
└─────────────┘
       ↓
┌─────────────┐
│   SUCCESS   │ ← Có data → Hiển thị
└─────────────┘
       OR
┌─────────────┐
│    ERROR    │ ← Có lỗi → Show message + Retry
└─────────────┘
```

**Rule of Thumb:**

> Thiếu 1 trong 3 → UX tệ.
>
> - Thiếu Loading → User tưởng app treo.
> - Thiếu Error → User không biết sai gì để sửa.
> - Thiếu Success → ... À thì đương nhiên rồi :)

### 🎭 UI State Types

| State Type       | When to Use                          | Component Example                         |
| :--------------- | :----------------------------------- | :---------------------------------------- |
| **Spinner**      | Action nhỏ, inline trong button      | Login button, Save button                 |
| **Skeleton**     | Load data trang mới, content lớn     | Profile page, Product list                |
| **Progress Bar** | Upload file, import data             | File upload (0-100%)                      |
| **Empty State**  | Query thành công nhưng không có data | "No items found", "Start adding products" |
| **Error Inline** | Lỗi field-level (validation)         | Under input: "Email is invalid"           |
| **Error Global** | Lỗi system-level (500, network)      | Toast notification                        |

### 🎯 The "Perceived Performance" Principle

> **"Users don't care about actual speed. They care about FEELING fast."**

**Example:**

- App load trong 2 giây với màn hình trắng → Feels SLOW
- App load trong 2 giây với skeleton nhấp nháy → Feels FAST

Skeleton tạo cảm giác "có gì đó đang xảy ra".

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: LOADING STATES (45 mins)

#### Step 1: Button Spinner (Inline Loading)

**Instructor Script:**
_"Nút bấm Login mà không quay vòng vòng thì user sẽ nghĩ nó bị hỏng và bấm 10 lần. Ta thêm `isPending` state từ React Query."_

Open `src/features/auth/components/LoginForm.tsx`:

```tsx
import { Loader2 } from "lucide-react"; // Icon spinner

// ... inside component
const loginMutation = useLoginMutation();

return (
  <Button type="submit" disabled={loginMutation.isPending} className="w-full">
    {loginMutation.isPending && (
      <Loader2 className="mr-2 h-4 w-4 animate-spin" />
    )}
    {loginMutation.isPending ? "Đang đăng nhập..." : "Đăng nhập"}
  </Button>
);
```

**Instructor Explain:**

- `disabled={isPending}`: Chặn user spam click.
- `animate-spin`: Tailwind utility class (quay vòng vòng).
- Conditional text: Thay đổi label khi đang xử lý.

---

#### Step 2: Page-Level Skeleton (Content Placeholder)

**Instructor Script:**
_"Khi vào Profile, thay vì hiện màn hình trắng, ta hiện cái khung xương (Skeleton). Não người thích nhìn thấy 'hình dạng' ngay cả khi chưa có nội dung."_

1. Install shadcn skeleton component:

```bash
npx shadcn@latest add skeleton
```

2. Update `src/pages/ProfilePage.tsx`:

```tsx
import { Skeleton } from "@/components/ui/skeleton";
import { useUser } from "@/hooks/useUser";

export default function ProfilePage() {
  const { data: user, isLoading, isError } = useUser();

  // Loading State - SKELETON
  if (isLoading) {
    return (
      <div className="max-w-2xl mx-auto p-6">
        <div className="flex items-center gap-4 mb-6">
          {/* Avatar Skeleton */}
          <Skeleton className="h-20 w-20 rounded-full" />
          <div className="space-y-2">
            {/* Name Skeleton */}
            <Skeleton className="h-6 w-[250px]" />
            {/* Email Skeleton */}
            <Skeleton className="h-4 w-[200px]" />
          </div>
        </div>

        {/* Bio Section Skeleton */}
        <Skeleton className="h-4 w-full mb-2" />
        <Skeleton className="h-4 w-3/4" />
      </div>
    );
  }

  // Error State (sẽ làm ở PHASE 2)
  if (isError) return <ErrorState />;

  // Success State
  return (
    <div className="max-w-2xl mx-auto p-6">
      <div className="flex items-center gap-4">
        <img src={user.avatar} className="h-20 w-20 rounded-full" />
        <div>
          <h1 className="text-2xl font-bold">{user.name}</h1>
          <p className="text-gray-500">{user.email}</p>
        </div>
      </div>
      {/* ... rest of profile */}
    </div>
  );
}
```

**Instructor Note:**

> "Skeleton width phải gần với content thật. Nếu tên user trung bình 200px thì skeleton cũng nên 200px. Đừng làm skeleton toàn màn hình cho 1 dòng chữ."

---

#### Step 3: Create Reusable Loading Components

**Instructor Script:**
_"Đừng copy-paste skeleton ở mọi nơi. Ta tạo components tái sử dụng."_

Create `src/components/common/LoadingStates.tsx`:

```tsx
import { Skeleton } from "@/components/ui/skeleton";
import { Loader2 } from "lucide-react";

// Full-page loading (cho trang lớn)
export const PageLoading = () => (
  <div className="flex items-center justify-center min-h-screen">
    <div className="text-center space-y-4">
      <Loader2 className="h-10 w-10 animate-spin mx-auto text-blue-500" />
      <p className="text-gray-500 animate-pulse">Đang tải...</p>
    </div>
  </div>
);

// Card skeleton (cho danh sách)
export const CardSkeleton = () => (
  <div className="border rounded-lg p-4 space-y-3">
    <Skeleton className="h-4 w-3/4" />
    <Skeleton className="h-4 w-1/2" />
    <Skeleton className="h-8 w-20" />
  </div>
);

// List skeleton
export const ListSkeleton = ({ count = 3 }: { count?: number }) => (
  <div className="space-y-4">
    {Array.from({ length: count }).map((_, i) => (
      <CardSkeleton key={i} />
    ))}
  </div>
);
```

**Usage Example:**

```tsx
// In any page
if (isLoading) return <ListSkeleton count={5} />;
```

---

#### 📏 RULE OF THUMB: Skeleton vs Spinner vs Progress Bar

**Instructor Script:**
_"Học viên hay lạm dụng Skeleton hoặc dùng cả 2 cùng lúc. Hãy chốt luật:"_

| Scenario                           | Best Choice           | Why                                      |
| :--------------------------------- | :-------------------- | :--------------------------------------- |
| **Button click** (Login, Save)     | Spinner (in button)   | User đang chờ response ngay trên nút bấm |
| **Page transition** (vào /profile) | Skeleton              | Thay thế toàn bộ content, giữ layout     |
| **Upload file**                    | Progress bar (0-100%) | User cần biết còn bao lâu                |
| **Infinite scroll loading more**   | Spinner (at bottom)   | Nhỏ gọn, không chiếm layout              |
| **Both Skeleton + Spinner**        | ❌ NEVER              | Confusing, over-engineering              |

**Anti-Pattern:**

```tsx
// ❌ ĐỪng LÀM NÀY
if (isLoading) {
  return (
    <>
      <Skeleton /> {/* Skeleton cho content */}
      <Loader2 className="animate-spin" /> {/* Spinner thừa */}
    </>
  );
}
```

---

### PHASE 2: ERROR STATES (40 mins)

#### Step 1: Field-Level Error (Form Validation)

**Instructor Script:**
_"Lỗi validation phải hiện ngay dưới input sai, không phải toast ở góc màn hình."_

```tsx
// In LoginForm.tsx (React Hook Form)
<div>
  <Input
    {...register("email")}
    placeholder="Email"
    className={errors.email ? "border-red-500" : ""}
  />
  {errors.email && (
    <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
  )}
</div>
```

---

#### Step 2: Global Error Handler (Axios Interceptor)

**Instructor Script:**
_"Thay vì mỗi API call đều `catch` error thủ công, ta xử lý tập trung trong Axios Interceptor."_

Open `src/lib/http/apiClient.ts`:

```ts
import { toast } from "sonner";

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;

    // 1. Network Error (Offline, CORS, Timeout)
    if (!error.response) {
      toast.error("Không thể kết nối tới server. Kiểm tra kết nối mạng.");
      return Promise.reject(error);
    }

    // 2. Server Error (500, 502, 503)
    if (status && status >= 500) {
      toast.error("Hệ thống đang bảo trì. Vui lòng thử lại sau.");
      return Promise.reject(error);
    }

    // 3. Auth Error (401) - Handled by refresh token logic
    if (status === 401 && !error.config._retry) {
      // ... refresh token logic (already implemented in Session 13)
    }

    // 4. Validation Error (422) - Handled per-form
    // DON'T toast 422 globally. Let forms handle field errors.

    // 5. Other errors - Let component handle
    return Promise.reject(error);
  }
);
```

**Instructor Warning:**

> "⚠️ KHÔNG toast mọi lỗi ở đây. Nếu 5 API call fail cùng lúc, sẽ có 5 toast hiện chồng lên nhau → Bad UX.
>
> **Rule**: Chỉ toast lỗi nghiêm trọng (Network, 500). Lỗi validation (422) để form tự xử lý."

---

#### Step 3: Page-Level Error Component

**Instructor Script:**
_"Nếu query fail, đừng để màn hình trắng hoặc chữ 'Error'. Hãy có UI đẹp với nút Retry."_

Create `src/components/common/ErrorStates.tsx`:

```tsx
import { AlertCircle, RefreshCw } from "lucide-react";
import { Button } from "@/components/ui/button";

interface ErrorStateProps {
  message?: string;
  onRetry?: () => void;
}

export const ErrorState = ({
  message = "Đã có lỗi xảy ra. Vui lòng thử lại.",
  onRetry,
}: ErrorStateProps) => (
  <div className="flex flex-col items-center justify-center min-h-[400px] p-8">
    <AlertCircle className="h-16 w-16 text-red-500 mb-4" />
    <h3 className="text-xl font-semibold mb-2">Oops!</h3>
    <p className="text-gray-600 text-center mb-6 max-w-md">{message}</p>
    {onRetry && (
      <Button onClick={onRetry} variant="outline">
        <RefreshCw className="mr-2 h-4 w-4" />
        Thử lại
      </Button>
    )}
  </div>
);

// Network-specific error
export const NetworkError = ({ onRetry }: { onRetry?: () => void }) => (
  <ErrorState
    message="Không thể kết nối tới server. Kiểm tra kết nối mạng của bạn."
    onRetry={onRetry}
  />
);
```

**Usage:**

```tsx
// In ProfilePage.tsx
if (isError) {
  return <ErrorState onRetry={() => refetch()} />;
}
```

---

### PHASE 3: EMPTY STATES (25 mins)

#### Step 1: Empty State Component

**Instructor Script:**
_"Data không có lỗi, query thành công, nhưng array rỗng. Đừng để user nhìn màn hình trống trơn."_

Create `src/components/common/EmptyState.tsx`:

```tsx
import { PackageOpen } from "lucide-react";
import { Button } from "@/components/ui/button";

interface EmptyStateProps {
  title?: string;
  message?: string;
  actionLabel?: string;
  onAction?: () => void;
}

export const EmptyState = ({
  title = "Chưa có dữ liệu",
  message = "Hãy bắt đầu thêm mục đầu tiên.",
  actionLabel,
  onAction,
}: EmptyStateProps) => (
  <div className="flex flex-col items-center justify-center min-h-[300px] p-8 text-center">
    <PackageOpen className="h-20 w-20 text-gray-400 mb-4" />
    <h3 className="text-lg font-semibold mb-2">{title}</h3>
    <p className="text-gray-500 mb-6 max-w-sm">{message}</p>
    {actionLabel && onAction && (
      <Button onClick={onAction}>{actionLabel}</Button>
    )}
  </div>
);
```

**Usage Example:**

```tsx
// In ProductListPage.tsx
const { data: products, isLoading } = useProducts();

if (isLoading) return <ListSkeleton />;

if (!products || products.length === 0) {
  return (
    <EmptyState
      title="Chưa có sản phẩm nào"
      message="Bắt đầu thêm sản phẩm đầu tiên vào cửa hàng của bạn."
      actionLabel="Thêm sản phẩm"
      onAction={() => navigate("/products/new")}
    />
  );
}

return <ProductList products={products} />;
```

---

### PHASE 4: INTEGRATION & POLISH (10 mins)

#### Step 1: Update All Pages

**Instructor Script:**
_"Giờ các bạn có đủ vũ khí. Hãy áp dụng vào TẤT CẢ các trang."_

**Checklist:**

```markdown
- [ ] LoginPage: Button có spinner khi đang login
- [ ] RegisterPage: Button có spinner khi đang register
- [ ] ProfilePage: Skeleton khi loading, Error state nếu fail
- [ ] UpdateProfilePage: Button disable + spinner khi đang save
- [ ] UploadPage: Progress/spinner khi đang upload
- [ ] All forms: Field errors hiện dưới input
```

#### Step 2: Toast Consistency

**Instructor Script:**
_"Toast message phải consistent. Không được 1 chỗ 'Success', 1 chỗ 'Thành công', 1 chỗ '✅'."_

**Toast Guidelines:**

```tsx
// ✅ Success
toast.success("Đăng nhập thành công!");

// ❌ Error (user fault)
toast.error("Email hoặc mật khẩu không đúng.");

// ⚠️ Warning
toast.warning("Phiên đăng nhập sắp hết hạn.");

// ℹ️ Info
toast.info("Đã gửi email xác nhận. Kiểm tra hộp thư của bạn.");
```

---

## 🚦 MID-SESSION CHECKPOINT

**Instructor Action:**
_"Bây giờ tất cả học viên test app theo checklist này:"_

- [ ] Bấm Login → Thấy spinner quay trong button
- [ ] Login xong → Thấy toast "Đăng nhập thành công"
- [ ] Vào Profile → Thấy Skeleton nhấp nháy → Rồi hiện data
- [ ] Tắt mạng (DevTools Offline) → Bấm nút bất kỳ → Thấy toast "Không thể kết nối"
- [ ] Bật lại mạng → Bấm Retry trong ErrorState → Data load lại

**If >50% students fail:** STOP. Debug together.

---

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1. **Skeleton giật cục (Flash of Content)**

   - _Symptom:_ Skeleton hiện → Biến mất → Lại hiện Spinner → Data xuất hiện
   - _Cause:_ Dùng cả `isLoading` (mount) và `isFetching` (refetch) → Chồng chéo
   - _Fix:_ Chỉ dùng `isLoading` cho lần load đầu, `isFetching` cho refetch nền

2. **Toast spam (Multiple toasts)**

   - _Symptom:_ 5 API fail cùng lúc → 5 toast chồng lên nhau
   - _Cause:_ Global interceptor toast mọi lỗi
   - _Fix:_ Chỉ toast lỗi critical (Network, 500). Form errors để component xử lý

3. **Button không disable**

   - _Symptom:_ User bấm Login 10 lần → Gửi 10 requests
   - _Cause:_ Quên `disabled={isPending}`
   - _Fix:_ Luôn disable button trong lúc mutation

4. **Empty state khi đang loading**
   - _Code:_
     ```tsx
     if (!data || data.length === 0) return <EmptyState />; // ❌ Wrong order
     if (isLoading) return <Skeleton />;
     ```
   - _Fix:_ Check `isLoading` TRƯỚC `!data`

---

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1. **Q:** _"Khi nào dùng Skeleton, khi nào dùng Spinner?"_

   - **A:**
     - Skeleton: Load content lớn (trang mới, list)
     - Spinner: Action nhỏ (button, inline loading)

2. **Q:** _"Tại sao không toast tất cả lỗi 422?"_

   - **A:** 422 là validation error. User cần nhìn thấy lỗi ngay ở input sai, không phải toast góc màn hình.

3. **Q:** _"Network error khác 500 error như thế nào?"_
   - **A:**
     - Network error: `error.response === undefined` (Offline, CORS, timeout)
     - 500 error: `error.response.status === 500` (Backend crashed)

---

## 7️⃣ IN-CLASS MINI TASK

**Task:** Polish Your App.

**Yêu cầu:**

1. Mở file `ProfilePage.tsx`
2. Implement đầy đủ 3 states: Loading (Skeleton), Error (ErrorState), Success
3. Test bằng cách:
   - Tắt mạng → Thấy NetworkError
   - Bật mạng → Thấy Skeleton → Thấy data
   - Sửa API URL sai → Thấy Error với nút Retry

**Time limit:** 15 phút

**Acceptance criteria:**

- Code có đầy đủ 3 states
- UI render đúng ở mọi trạng thái
- Không có màn hình trắng hoặc crash

---

## 8️⃣ HOMEWORK / EXTENSION TASK

**Yêu cầu:** Polish All Pages.

1. Apply Loading/Error/Empty states cho TẤT CẢ trang còn lại:

   - HomePage (Product list)
   - UpdateProfilePage
   - ChangePasswordPage
   - UploadPage

2. Create `LoadingButton` component:

   ```tsx
   // Reusable loading button
   <LoadingButton isLoading={mutation.isPending} onClick={handleSubmit}>
     Save Changes
   </LoadingButton>
   ```

3. (Nâng cao) Implement Optimistic UI:
   - Like button: Tăng count ngay lập tức (chưa chờ API)
   - Nếu API fail → Rollback

---

## 9️⃣ CHECKPOINT & EVALUATION

**Học viên Pass buổi này khi:**

- [ ] Profile page có đầy đủ Loading/Error/Empty states
- [ ] All buttons disable khi đang loading
- [ ] Toast messages consistent (Success/Error format)
- [ ] Không còn màn hình trắng khi load data

**Level Up Signal:**

- Học viên chủ động hỏi: "Em nên dùng Skeleton hay Spinner cho case này?"
- Không hỏi: "Thầy ơi, sao em bấm nút mà không thấy gì?"

---

## 🔟 TEACHING NOTES

### Time Management

- **00-10':** Opening + Mental Models
- **10-55':** PHASE 1 (Loading States)
- **55-95':** PHASE 2 (Error States)
- **95-110':** PHASE 3 (Empty States)
- **110-120':** Checkpoint + Q&A

### Emphasis Points

1. **The 3-State Rule** - Nhắc đi nhắc lại
2. **Skeleton vs Spinner** - Vẽ diagram lên bảng
3. **"Perceived Performance"** - Demo cùng timing nhưng khác UX

### Red Flags

- Học viên skip Loading state vì "API nhanh mà" → Warn: Production có thể lag
- Học viên toast mọi lỗi → Explain: Toast overload = bad UX
- Học viên không test offline mode → Force test trước khi qua phần tiếp

### Motivation Script

> _"Các bạn có biết tại sao app của Facebook, Netflix, Shopee mượt như lụa? Không phải vì server nhanh. Mà vì họ có Skeleton, Loading states cực kỳ tinh tế. User KHÔNG BAO GIỜ thấy màn hình trắng. Hôm nay các bạn học kỹ năng đó."_

---

## 📚 REFERENCES FOR STUDENTS

**Recommended Reading:**

- [Skeleton Screen Best Practices](https://uxdesign.cc/what-you-should-know-about-skeleton-screens-a820c45a571a)
- [Loading State Patterns](https://www.smashingmagazine.com/2016/12/best-practices-for-animated-progress-indicators/)
- [Shadcn UI Skeleton](https://ui.shadcn.com/docs/components/skeleton)

---

## 🎯 SUCCESS CRITERIA

**By the end of this session, students should:**

1. Understand the 3-State Rule (Loading/Error/Success)
2. Implement Skeleton for page loads and Spinner for button actions
3. Handle errors gracefully with retry mechanisms
4. Create empty states that guide users to take action
5. Build reusable Loading/Error/Empty components

**Next Session Preview:**
_"Buổi sau chúng ta sẽ học Debug Workshop. App đã đẹp rồi, giờ phải học cách tìm bug nhanh như chớp. Sẵn sàng chưa?"_

---

**END OF SESSION 14**
