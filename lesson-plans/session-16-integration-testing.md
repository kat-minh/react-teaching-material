# LESSON PLAN: SESSION 16 – INTEGRATION TESTING & CODE AUDIT

> **[RENUMBERED FROM SESSION 15]**

## 1️⃣ SESSION OVERVIEW

- **Title:** The Test Flight: Full Integration Check
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will perform a full end-to-end test of their application (Register → Logout), audit their code against professional standards, and write a README.
- **Outcome:** A codebase that is clean, free of major standard violations, and ready for deployment.

---

## 2️⃣ INSTRUCTOR OPENING SCRIPT

> _"Chào các bạn. Hôm nay là buổi cuối cùng trước khi chúng ta tốt nghiệp Phase 1.
> Hãy tưởng tượng chúng ta là những kỹ sư của Boeing.
> Code xong **chưa phải là xong**. Máy bay phải bay thử.
> Hôm nay chúng ta sẽ làm 3 việc:"_
>
> 1. **User Acceptance Testing (UAT):** Đóng vai user khó tính nhất để tìm bug.
> 2. **Code Audit:** Đóng vai Senior Developer để soi code "bẩn".
> 3. **Documentation:** Viết hướng dẫn sử dụng (README) cho người sau.

> 🔥 **WHY THIS SESSION EXISTS?** > _"Nhiều bạn code xong vứt đó. 6 tháng sau mở lại không nhớ chạy kiểu gì.
> Hoặc người khác vào team đọc code không hiểu.
> Buổi này dạy kỹ năng **hoàn thiện sản phẩm**."_

---

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Integration Testing (Test tích hợp)

- **Unit Test:** Test từng con ốc vít (từng hàm).
- **Integration Test:** Test động cơ có quay không (các module kết nối nhau).
- **E2E Test:** Test máy bay có bay được không (User interaction).
- Hôm nay ta làm **Manual E2E Testing** – _chạy bằng cơm_.

### 🧹 The Boy Scout Rule

> _"Always leave the campground cleaner than you found it."_

- Thấy code thừa → **Xóa**
- Thấy `console.log` → **Xóa**
- Đừng để rác trong code

---

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: MANUAL E2E CHECKLIST (60 mins)

**Instructor Note:**

> _"Buổi này ta học **tư duy test**.
> Chưa học test tự động (Cypress / Playwright).
> Vì nếu test tay còn sai → test tự động cũng sai."_

#### ⏱️ TIMELINE

- **00–10’:** Setup môi trường test sạch (Incognito, clear storage).
- **10–40’:** Chạy checklist từng bước.
- **40–60’:** Fix **Hot Bugs** (bug nghiêm trọng sửa ngay).

---

### ✅ MANUAL TEST CHECKLIST

**Instructor Script:**

> _"Mở 2 cửa sổ: App + Checklist.
> Tick từng mục. Fail ở đâu ghi chú ngay."_

#### 🔐 Auth Flow

- [ ] Register account mới (email chưa tồn tại)
- [ ] Login ngay sau khi register
- [ ] Sai password → báo lỗi rõ ràng
- [ ] Reload trang → vẫn còn login (persist OK)
- [ ] Logout → clear token → redirect Login

#### 👤 Profile Flow

- [ ] Hiện đúng tên & avatar mặc định
- [ ] Đổi tên → toast success → header cập nhật ngay
- [ ] Đổi password → logout →

  - Login pass cũ ❌
  - Login pass mới ✅

#### ⚠️ Edge Cases

- [ ] Truy cập `/me` khi chưa login → redirect Login
- [ ] Upload avatar → F5 → ảnh vẫn giữ
- [ ] Tắt mạng → bấm Save → báo lỗi mạng (không crash)

---

## 5️⃣ PHASE 2: CODE AUDIT RUBRIC (40 mins)

#### ⏱️ TIMELINE

- **00–10’:** Giải thích tiêu chí code sạch
- **10–40’:** Peer Review (đổi bài)

**Review Rule:**

> _Không công kích cá nhân.
> Mỗi lỗi phải kèm gợi ý sửa._

### 🛑 RED FLAGS

| Code Smell             | Vì sao tệ         | Cách sửa              |
| ---------------------- | ----------------- | --------------------- |
| Console.log everywhere | Rác console       | Xóa trước khi push    |
| Hardcoded URL          | Không deploy được | Dùng env              |
| Fetch trong component  | Khó test          | Tách api + hook       |
| Duplicate state        | Bug ngầm          | Dùng React Query data |
| `any` type             | Mất type safety   | Định nghĩa interface  |

---

## 6️⃣ PHASE 3: README TEMPLATE (20 mins)

> 📌 **Instructor Note:**
> Bên dưới là **file mẫu**. Học viên **copy sang `README.md`**.

````md
# Shopping Cart Frontend 🛒

Dự án Frontend ReactJS – Phase 1.

## 🚀 Tech Stack

- React + TypeScript + Vite
- Zustand + TanStack Query
- React Hook Form + Zod
- TailwindCSS + Shadcn/UI
- Axios (Interceptor)

## 🏃‍♂️ How to run

```bash
git clone <repo>
cp .env.example .env
npm install
npm run dev
```

## 🔑 Key Features

- Auto refresh token
- Protected routes
- Profile management

## 📁 Project Structure

- src/features
- src/components
- src/lib
- src/stores
````

---

## 7️⃣ COMMON STUDENT BUGS

1. **Redirect Loop**

   - Cause: Guard logic đá nhau
   - Fix: Tách `isLoading` & `isAuthenticated`

2. **Mất state khi hot reload**

   - Bình thường → chấp nhận

---

## 8️⃣ DEPLOYMENT GATE

### ❌ NO-GO

- Fail checklist
- Console còn log
- README không chạy được

### ✅ GO

- Pass 100% checklist
- Code sạch
- Clone máy khác chạy < 5 phút

---

## 9️⃣ SELF-REFLECTION

1. Bug khó nhất là gì?
2. Lỗi ngớ ngẩn nhất?
3. Nếu làm lại, bạn đổi kiến trúc chỗ nào?

---

## 🔟 TEACHING NOTES

- **Mindset:** Khó tính – rèn sự tỉ mỉ
- **Fun:** Bug Hunt – tìm bug thưởng kẹo 😄

---
