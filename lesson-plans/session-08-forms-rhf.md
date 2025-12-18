# LESSON PLAN: SESSION 08 - FORMS W/ REACT HOOK FORM & ZOD

## 1️⃣ SESSION OVERVIEW
- **Title:** The Form Tamer: Validation with Zod & React Hook Form
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will master form handling in React by moving away from tedious `useState` per field to a robust solution using React Hook Form (RHF) and Zod schema validation.
- **Outcome:** A full Register Form with client-side validation (password match, date format) and integration with the backend API.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Form là cơn ác mộng của mọi Frontend Developer. 
Nếu dùng cách cũ (`useState` cho từng input), các bạn sẽ phải viết hàng trăm dòng code chỉ để kiểm tra xem password có khớp với confirm password hay không. Code sẽ rối như tơ vò.

Hôm nay tôi sẽ giới thiệu 'Cặp đôi hoàn hảo':
1.  **React Hook Form:** Quản lý việc nhập liệu (thu thập dữ liệu, handle submit). Nó dùng cơ chế 'Uncontrolled' giúp app không bị re-render liên tục.
2.  **Zod:** Quản lý logic đúng sai (Validation). Nó là luật sư, đảm bảo dữ liệu gửi đi API phải sạch sẽ 100%."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Backend sẽ trả về lỗi 422 nếu bạn gửi rác lên. Nhưng chờ API trả lỗi thì trải nghiệm user quá tệ. Ta cần chặn lỗi ngay từ lúc user đang gõ phím. RHF + Zod là tiêu chuẩn vàng của React hiện đại để làm việc này."_

> **🧠 INSTRUCTOR MENTAL NOTE:**
> _"Hôm nay chúng ta học form chuẩn production, không phải form demo. Nếu thấy hơi nhiều khái niệm (Resolver, Schema, Uncontrolled), đó là cảm giác ĐÚNG. Đừng hoảng!"_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Controlled vs Uncontrolled (Pain-Driven Demo)
- **Controlled (Cách cũ):** `value={state}` + `onChange`. Mỗi lần gõ 1 ký tự -> App render lại -> Chậm.
- **Uncontrolled (RHF):** Dùng `ref` để "móc" trực tiếp vào DOM. **RHF tối ưu re-render** (không render lại toàn bộ component tree mỗi lần gõ), hiệu năng tốt hơn nhiều.

### 📜 Schema Validation (Zod)
- Thay vì viết `if (email.contains('@'))`, ta viết `z.string().email()`.
- Schema là bản thiết kế của dữ liệu. Xác định từ đầu, dùng mãi mãi.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: PAIN-DRIVEN DEMO (15 mins)
_"Trước khi dùng hàng xịn, hãy xem tại sao hàng cũ lại đau khổ."_

Update `src/pages/RegisterPage.tsx` (Tạm thời):
Create a simple form with `useState`.
- **Action:** Gõ vào ô input, bật `Console Layout` -> Thấy log "Render" chạy liên tục.
- **Conclusion:** "Tưởng tượng form có 50 trường, gõ 1 cái render cả 50 cái. App sẽ lag tung máy." -> **Delete và chuyển sang RHF.**

---

### PHASE 2: DEFINE ZOD SCHEMA (30 mins)

#### Step 1: Install Libraries
```bash
npm install react-hook-form zod @hookform/resolvers
```

#### Step 2: Create Validation Schema
Create `src/utils/rules.ts` (Bắt buộc tách file):
_"Trong khóa này, schema luôn tách file riêng. Ngoài đời đi làm cũng vậy, để tái sử dụng cho cả Login và API."_

```ts
import * as z from 'zod';

export const registerSchema = z
  .object({
    email: z.string().email({ message: "Email không hợp lệ" }),
    name: z.string().min(1, { message: "Tên không được để trống" }), // Required
    password: z.string().min(6, { message: "Password phải ít nhất 6 ký tự" }),
    confirm_password: z.string(),
    date_of_birth: z.string(), // Sẽ xử lý date picker sau
  })
  .superRefine(({ password, confirm_password }, ctx) => {
    // Custom validation cho cả object
    if (confirm_password !== password) {
      ctx.addIssue({
        code: "custom",
        message: "Nhập lại mật khẩu không khớp",
        path: ["confirm_password"], // Gán lỗi vào trường này
      });
    }
  });

// Export type để dùng cho RHF
export type RegisterSchemaType = z.infer<typeof registerSchema>;
```

---

### PHASE 3: IMPLEMENT REACT HOOK FORM (45 mins)

#### Step 1: Setup useForm
Update `src/pages/RegisterPage.tsx`:

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { registerSchema, RegisterSchemaType } from '@/utils/rules';
import { Button } from "@/components/ui/button" // Re-use shadcn
import { Input } from "@/components/ui/input"

export default function RegisterPage() {
  const { 
    register, 
    handleSubmit, 
    formState: { errors } 
  } = useForm<RegisterSchemaType>({
    mode: 'onSubmit', // Validate khi bấm Submit (Dễ hiểu cho newbie)
    resolver: zodResolver(registerSchema) // Móc nối Zod vào RHF
  });

  const onSubmit = (data: RegisterSchemaType) => {
    console.log("Dữ liệu sạch:", data);
    // Gọi API sau
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4 max-w-md mx-auto mt-10">
      {/* Email Field */}
      <div>
        <Input placeholder="Email" {...register("email")} />
        {/* Hiển thị lỗi từ Zod */}
        {errors.email && <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>}
      </div>

      {/* Password Field */}
      <div>
        <Input type="password" placeholder="Password" {...register("password")} />
        {errors.password && <p className="text-red-500 text-sm">{errors.password.message}</p>}
      </div>

      {/* Confirm Password */}
      <div>
        <Input type="password" placeholder="Confirm Password" {...register("confirm_password")} />
        {errors.confirm_password && <p className="text-red-500 text-sm">{errors.confirm_password.message}</p>}
      </div>

      <Button type="submit">Đăng ký</Button>
    </form>
  );
}
```

---

### PHASE 4: CONNECT TO AXIOS SERVICE (30 mins)

#### Step 1: API Call
Update `onSubmit` function in `RegisterPage.tsx`:

```tsx
import { usersApi } from '@/lib/api/users.api';
import { toast } from 'sonner';
import { useNavigate } from 'react-router-dom';

// ... inside component
const navigate = useNavigate();

const onSubmit = async (data: RegisterSchemaType) => {
  try {
    // Data Transformation (nếu cần)
    const apiBody = {
      ...data,
      date_of_birth: new Date(data.date_of_birth).toISOString() // Chuẩn hóa Date
    };

    const res = await usersApi.register(apiBody);
    
    toast.success("Đăng ký thành công!");
    navigate("/login");
  } catch (error: any) {
    // Xử lý lỗi 422 từ backend (Advanced - Homework sẽ làm kỹ hơn)
    if (error.response?.status === 422) {
      // Instructor Note: "Ở bài này ta CHƯA làm mapping chi tiết, vì nếu làm ngay sẽ overload. Bài tập về nhà sẽ xử lý đúng chuẩn production."
      const fieldErrors = error.response.data.errors;
      // TODO: Loop qua fieldErrors và setError() cho form
      toast.error("Lỗi dữ liệu đầu vào");
    } else {
       toast.error("Lỗi hệ thống");
    }
  }
};
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên `type="submit"` cho Button**:
    *   *Symptom:* Bấm nút mà không thấy gì xảy ra (hoặc reload trang nếu nút nằm ngoài form).
    *   *Fix:* Luôn thêm `type="submit"` cho nút bấm chính.
2.  **Quên `{...register}`**:
    *   *Symptom:* Gõ dữ liệu nhưng khi submit thì nhận được `undefined`.
    *   *Fix:* Destructuring `register("name")` vào input để RHF nắm quyền kiểm soát.
3.  **Lỗi Import Resolver**:
    *   *Issue:* Cài thiếu thư viện `@hookform/resolvers`. Zod không tự chạy với RHF được.
4.  **Quên truyền Resolver**:
    *   *Symptom:* Form submit dù data rỗng/sai, validation không chạy.
    *   *Fix:* Kiểm tra xem `resolver: zodResolver(schema)` đã được truyền vào `useForm` chưa.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao `handleSubmit` lại nhận vào một hàm `onSubmit` của mình?"*
    *   **A:** Vì `handleSubmit` của RHF cần chạy validation trước. Nếu mọi thứ OK (clean data), nó mới gọi hàm của bạn. Nếu sai, nó chặn lại và cập nhật `errors` object.
2.  **Q:** *"Nếu tôi muốn validate Email có tồn tại trên server không (Async Validation)?"*
    *   **A:** Zod hỗ trợ `refineAsync`. Nhưng cẩn thận hiệu năng. Thường ta nên check lúc blur (rời ô input) hoặc lúc submit.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Thêm trường "Date of Birth".
- Thêm vào schema (đơn giản `z.string()`).
- Thêm Input type="date" vào form.
- Console log data xem format ngày tháng trình duyệt trả về là gì (`yyyy-mm-dd`).
- Thử sửa code transform về `ISOString` trước khi log.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Error Mapping (Backend 422 -> RHF Errors).
1.  Backend trả về: `{ errors: { email: "Email exist" } }`.
2.  Dùng hàm `setError` của RHF để hiển thị dòng lỗi này ngay dưới ô Email (đỏ lên như lỗi validation client).
3.  Trải nghiệm UX: User gõ sai -> Client chặn. User gõ đúng format nhưng trùng Email -> Server chặn và báo lỗi ngay tại chỗ đó.

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Bấm Submit khi form rỗng -> Không gọi API, thấy chữ đỏ hiện lên.
- **Signal:** Gõ pass và confirm pass khác nhau -> Báo lỗi "không khớp" ngay.
- **Code:** Schema tách biệt rõ ràng, không lẫn lộn logic trong component.

## 🔟 TEACHING NOTES
- **Slow Down:** Khúc `superRefine` của Zod. Syntax hơi lạ với newbie. Hãy giải thích: "Đây là nơi ta check mối quan hệ GIỮA 2 trường trở lên".
- **Emphasis:** Sự khác biệt giữa `errors` (lỗi validation) và `error` (lỗi hệ thống/API).
- **Red Flag:** Học viên quên bọc `<form>` quanh các input. RHF cần thẻ form để bắt sự kiện submit native.
