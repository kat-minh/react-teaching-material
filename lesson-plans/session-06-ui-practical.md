# LESSON PLAN: SESSION 06 - TAILWIND V4 & SHADCN/UI PRACTICAL

## 1️⃣ SESSION OVERVIEW

- **Title:** The Artist's Palette: Tailwind v4 and shadcn/ui
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will setup a professional UI system using Tailwind v4 (CSS-first) and shadcn/ui (Copy-paste components), creating a consistent look & feel for the entire app.
- **Outcome:** A polished Login Form, a Dialog for Logout confirmation, and Toast notifications (Sonner) working correctly.

## 2️⃣ INSTRUCTOR OPENING SCRIPT

_"Chào các bạn. Nếu logic là 'trí não' của App, thì UI chính là 'gương mặt'. Gương mặt đẹp thì người ta mới muốn nói chuyện._

_Rất nhiều bạn ở đây sợ CSS. Sợ căn giữa, sợ responsive. Đừng lo. Hôm nay ta sẽ dùng **Tailwind v4** - phiên bản mới nhất vừa ra mắt với tư duy 'Zero runtime'. Nó nhanh hơn, gọn hơn._

_Đặc biệt, ta không tự code từng cái nút bấm. Ta sẽ dùng **shadcn/ui**. Đây KHÔNG phải thư viện kiểu Bootstrap (cài 1 cục to đùng). Nó là dạng 'Copy-paste code'. Bạn sở hữu code, bạn sửa gì tùy thích. Hôm nay ta sẽ biến cái App nhìn như bài tập sinh viên thành một sản phẩm chuyên nghiệp."_

> **🔗 CONTINUITY NOTE:** > _"Những component UI hôm nay (Toast, Skeleton, Dialog) sẽ được dùng rất nhiều trong buổi sau khi ta gọi API thật và xử lý lỗi. Hãy làm kỹ phần này để buổi sau ta chỉ tập trung vào logic thôi nhé."_

> **🔥 WHY THIS SESSION EXISTS?** > _"Khách hàng không nhìn thấy code Backend hay React Query của bạn xịn thế nào đâu. Họ chỉ thấy cái nút bấm có mượt không, thông báo lỗi có đẹp không. Buổi này giúp bạn 'bán' được sản phẩm của mình."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🎨 Tailwind v4 (CSS-first)

- **Cũ (v3):** Config trong `tailwind.config.js` (JS Object).
- **Mới (v4):** Config trực tiếp trong CSS (`@theme`). Cảm giác tự nhiên hơn, giống viết CSS thường nhưng mạnh mẽ hơn.

### 🧩 shadcn/ui vs Material UI / Ant Design

- **Mothership (MUI/Antd):** Bạn cài 1 gói npm khổng lồ. Muốn sửa cái nút bấm? Rất khó, phải ghi đè CSS theo API của họ.
- **Copy-paste (shadcn/ui):** Bạn chạy lệnh, nó tạo ra file `Button.tsx` trong folder `components/ui` của bạn. Code đó là của bạn. Muốn sửa gì sửa trực tiếp vào file. Quyền lực tuyệt đối.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: SETUP TAILWIND V4 & SHADCN (40 mins)

#### Step 1: Verify Tailwind v4 Setup (Check lại từ buổi 1)

_"Buổi 1 ta cài rồi, nhưng giờ ta check kỹ lại file CSS."_

Open `src/index.css`:

```css
@import "tailwindcss";

/* Định nghĩa theme trực tiếp trong CSS */
@theme {
  --font-sans: "Inter", system-ui, sans-serif;
  --color-brand: #3b82f6; /* Define màu riêng */
}
```

#### Step 2: Init shadcn/ui

_"Shadcn cần config một chút để biết ta muốn copy code vào đâu."_

```bash
# Terminal (Chọn đúng các option)
npx shadcn@latest init

# Questions:
# - Style: New York
# - Base Color: Slate
# - CSS Variables: Yes
```

#### Step 3: Install Core Components

_"Ta cần những vũ khí cơ bản nhất: Nút, Ô nhập, Thẻ, Hộp thoại và Thông báo."_

```bash
npx shadcn@latest add button input card dialog sonner label
```

-> _Kiểm tra folder `src/components/ui`. Thấy các file vừa sinh ra chưa? Đó là code của bạn đấy._

---

### PHASE 2: BUILDING UI COMPONENTS (45 mins)

#### Step 1: Enhance Login Page

Update `src/pages/LoginPage.tsx`:
_"Thay thế mấy thẻ HTML xấu xí bằng Component xịn."_

> **⚠️ INSTRUCTOR NOTE:** > _"Các bạn chú ý: Input hôm nay CHƯA có `value` và `onChange` (uncontrolled)._
>
> _Hôm nay ta tập trung UI/styling, không focus logic form._
>
> _Buổi sau khi gọi API thật, ta sẽ làm đầy đủ Controlled Input với React Hook Form."_

```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Label } from "@/components/ui/label";

export default function LoginPage() {
  return (
    <div className="flex items-center justify-center min-h-screen bg-gray-50">
      <Card className="w-[350px] shadow-lg">
        <CardHeader>
          <CardTitle className="text-2xl text-center">Welcome Back</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>
            <Input id="email" type="email" placeholder="m@example.com" />
          </div>
          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>
            <Input id="password" type="password" />
          </div>
          <Button className="w-full bg-brand hover:bg-blue-700">Login</Button>
        </CardContent>
      </Card>
    </div>
  );
}
```

#### Step 2: Setup Toast Notification (Sonner)

_"Toast cũ xấu quá. Dùng Sonner - loại Toast đẹp nhất hiện nay (Stack lên nhau)."_

1. Add `Toaster` vào `src/Main.tsx` (Root level):

```tsx
import { Toaster } from "@/components/ui/sonner";

// ... inside render
<AppQueryProvider>
  <RouterProvider router={router} />
  <Toaster position="top-right" richColors /> {/* Cắm cọc ở đây */}
</AppQueryProvider>;
```

2. Test thử ở Login Page:

```tsx
import { toast } from "sonner"

// ... inside handleLogin
const handleLogin = () => {
   setTokens(...)
   toast.success("Đăng nhập thành công!", {
     description: "Chào mừng bạn quay lại."
   });
   navigate('/me');
}
```

---

### PHASE 3: INTERACTIVE UI (Logout Dialog) (35 mins)

#### Step 1: Create Logout Confirmation

_"Bấm Logout phát văng ra luôn thì hơi thô. Hỏi người ta một câu cho lịch sự."_

Update `src/components/layouts/MainLayout.tsx`:

```tsx
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog"

export default function MainLayout() {
  // ✅ ĐÚNG - Dùng Selector Pattern (học từ Session 05)
  const accessToken = useAuthStore((state) => state.accessToken);
  const clearTokens = useAuthStore((state) => state.clearTokens);

  // Hoặc dùng shallow:
  // const { accessToken, clearTokens } = useAuthStore(
  //   (state) => ({ accessToken: state.accessToken, clearTokens: state.clearTokens }),
  //   shallow
  // );

  return (
    // ... nav
    {accessToken ? (
      <AlertDialog>
        <AlertDialogTrigger asChild>
          <Button variant="ghost" className="text-red-500">Logout</Button>
        </AlertDialogTrigger>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>Bạn muốn đăng xuất?</AlertDialogTitle>
            <AlertDialogDescription>
              Phiên đăng nhập sẽ kết thúc và bạn cần login lại để truy cập.
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>Hủy</AlertDialogCancel>
            <AlertDialogAction onClick={handleLogout} className="bg-red-500 hover:bg-red-600">
              Đăng xuất
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>
    ) : (
      <Link to="/login">Login</Link>
    )}
  )
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Missing `Toaster` component**:
    - _Symptom:_ Gọi `toast.success()` nhưng màn hình im lìm, không thấy gì hiện ra.
    - _Fix:_ Chưa đặt `<Toaster />` ở root (Main.tsx hoặc App.tsx). Toast cần nơi để gắn DOM vào.
2.  **CSS Conflict (nếu dùng chung thư viện khác)**:
    - _Info:_ Tailwind v4 reset CSS khá mạnh. Đôi khi nó làm mất style mặc định của thẻ `h1, h2`. Phải định nghĩa lại trong `@theme` hoặc dùng class `text-2xl`.
3.  **Quên `asChild` trong Trigger**:
    - _Bug:_ Console báo lỗi Hydration hoặc Button lồng Button.
    - _Fix:_ Nếu `AlertDialogTrigger` chứa một `Button` bên trong, phải thêm prop `asChild` để nó merge 2 nút thành 1, tránh render thừa thẻ `button`.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** _"Tại sao ta copy code của shadcn về mà không cài npm package?"_
    - **A:** Để ta có quyền kiểm soát hoàn toàn (Full Control). Nếu muốn đổi nút bấm từ bo tròn sang vuông, ta vào file `button.tsx` sửa 1 dòng là xong cả dự án đổi theo. Thư viện đóng gói sẵn (MUI) rất khó làm điều này.
2.  **Q:** _"Sonner khác gì React-Toastify?"_
    - **A:** Sonner nhẹ hơn, animation mượt hơn (stacking cards) và dễ tùy biến style theo Tailwind hơn.

## 7️⃣ IN-CLASS MINI TASK

**Task:** Tạo Loading Skeleton.

- Dùng lệnh `npx shadcn@latest add skeleton`.
- Tạo component `ProductCardSkeleton`.
- UI: Một hình chữ nhật xám (ảnh), bên dưới là 2 dòng kẻ xám (tên và giá).
- Hiển thị Skeleton này trong `HomePage` khi `isLoading` (giả lập bằng setTimeout).

## 8️⃣ HOMEWORK / EXTENSION TASK

**Yêu cầu:** Trang trí trang 'Me' (Profile).

1.  Dùng `Card` component để bao bọc thông tin user.
2.  Dùng `Avatar` component (cài thêm từ shadcn) để hiện ảnh đại diện (hoặc chữ cái đầu tên nếu không có ảnh).
3.  Dùng `Separator` để kẻ đường phân cách giữa thông tin cá nhân và lịch sử mua hàng.

## 9️⃣ CHECKPOINT & EVALUATION

- **Signal:** Login hiện Toast xanh lá cây đẹp mắt góc phải màn hình.
- **Signal:** Bấm Logout hiện Dialog hỏi, bấm Hủy thì dialog đóng, bấm Đăng xuất thì mới chạy logic logout.
- **Code:** Vào `src/components/ui` thấy đầy đủ các file component.

## 🔟 TEACHING NOTES

- **Slow Down:** Lúc giải thích cấu trúc `Dialog` (Trigger -> Content -> Header -> Footer). Nó khá nhiều cấp lồng nhau, học viên dễ hoa mắt và đóng ngoặc sai chỗ.
- **Emphasis:** Tính năng "Zero runtime" của Tailwind v4. Hãy giải thích sơ:
  - _"Zero runtime CHO STYLING - không phải cho toàn app"_
  - Tailwind compile ra file CSS tĩnh, không tốn Javascript tính toán styles lúc runtime
  - ⚠️ TRÁNH CLAIM: "Tailwind làm web nhanh hơn React" (sai! Chỉ styling nhanh hơn, React vẫn chạy bình thường)
