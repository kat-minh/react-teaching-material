# LESSON PLAN: SESSION 14 - UI POLISH & DEBUG WORKSHOP

## 1️⃣ SESSION OVERVIEW
- **Title:** The Art of Debugging & UI Polish
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will transform their "working" app into a "production-ready" app by handling Edge Cases (Loading, Error, Empty) and learning how to debug complex issues using Chrome/React DevTools.
- **Outcome:** An app with smooth UX (Skeletons, Spinners) and students who know how to finding bugs without asking "Thầy ơi lỗi gì đây?".

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Code chạy được là **Level 1** (Junior). Code chạy mượt, báo lỗi đẹp, dễ debug là **Level 2** (Mid-level).

Hôm nay chúng ta sẽ không viết thêm tính năng mới. Chúng ta sẽ làm 2 việc:
1. **Polish:** Trang điểm cho ứng dụng (Loading Skeleton, Error Toast chuẩn).
2. **Debug:** Tôi sẽ hướng dẫn các bạn cách 'bắt bệnh' khi App lăn ra chết mà không cần console.log bừa bãi.

Hãy nhớ: 50% thời gian đi làm là Debug. Nếu bạn giỏi Debug, bạn làm việc nhanh gấp đôi người khác."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Học viên thường chỉ quan tâm Happy Case. Buổi này ép họ phải nhìn vào Sad Case (Mạng lag, Server sập, Token lỗi) để App không vỡ vụn khi gặp sự cố."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🩺 The Detective Mindset (Tư duy thám tử)
- **Console Log:** Giống như hỏi nhân chứng. Đôi khi đáng tin, đôi khi không.
- **Network Tab:** Giống như khám nghiệm hiện trường. Đây là sự thật 100% (Backend trả gì, Header gửi gì).
- **Debugger (Breakpoints):** Làm chậm thời gian để xem viên đạn bay thế nào.

### 🚦 UI States (Bộ 3 trạng thái)
Mọi UI async đều phải có đủ 3 mặt:
1. **Loading:** Đang tải (Spinner/Skeleton).
2. **Error:** Lỗi (Message/Retry button).
3. **Success:** Dữ liệu (Component chính).
*Thiếu 1 trong 3 -> UX tệ.*

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: UI POLISH (45 mins)

#### Step 1: Add Global Loading Indicator (Spinner)
Create `src/components/ui/spinner.tsx`:
*(Dùng lucid-react `Loader2` icon + animate-spin)*

**Instructor Script:**
_"Nút bấm login mà không quay vòng vòng thì user sẽ bấm 10 lần. Ta thêm `isPending` vào button."_

```tsx
// Trong LoginForm.tsx
<Button disabled={loginMutation.isPending}>
  {loginMutation.isPending && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
  Login
</Button>
```

#### Step 2: Skeleton Loading (Shadcn UI)
**Instructor Script:**
_"Khi vào Profile, thay vì hiện màn hình trắng, ta hiện cái khung xương."_

1. Install: `npx shadcn-ui@latest add skeleton`
2. Update `ProfilePage.tsx`:

```tsx
if (isLoading) {
  return (
    <div className="flex items-center space-x-4">
      <Skeleton className="h-12 w-12 rounded-full" />
      <div className="space-y-2">
        <Skeleton className="h-4 w-[250px]" />
        <Skeleton className="h-4 w-[200px]" />
      </div>
    </div>
  );
}
```

#### Step 3: Consistent Error Handling
Open `src/lib/http/apiClient.ts`:
**Instructor Script:**
_"Thay vì mỗi chỗ `catch` một kiểu, ta setup Global Error Handler trong Interceptor."_

```ts
apiClient.interceptors.response.use(
  (res) => res,
  (error) => {
    // ... logic refresh token
    
    // Global Toast cho lỗi 500 (Server Sập)
    if (error.response?.status >= 500) {
      toast.error("Hệ thống đang bảo trì. Vui lòng quay lại sau.");
      // Instructor Note: "Lưu ý: Global error chỉ nên dùng cho lỗi nghiêm trọng. Trong production, nếu 5 request fail cùng lúc, ta cần logic 'debounce' để tránh 5 cái toast hiện đè lên nhau, gây bad UX."
    }
    return Promise.reject(error);
  }
);
```

#### 📏 RULE OF THUMB: Skeleton vs Spinner
_"Học viên hay lạm dụng Skeleton. Hãy chốt luật:"_

| Feature | Use Case | Example |
| :--- | :--- | :--- |
| **Spinner** | Action nhỏ, trên cùng trang | Click nút Login, Like, Save |
| **Skeleton** | Load data trang mới, content lớn | Vào trang Profile, Load List Product |
| **� Don't** | Dùng cả 2 cùng lúc | Vừa hiện skeleton vừa quay spinner |

**�🚦 MID-SESSION CHECKPOINT**
- Bấm Login -> Thấy spinner quay.
- F5 trang Profile -> Thấy Skeleton nhấp nháy 1 chút rồi hiện data.
- Tắt mạng (Offline mode trong DevTools) -> Bấm Login -> Thấy báo lỗi Network/Server Error.

---

### PHASE 2: DEBUG WORKSHOP (60 mins)
*GV sẽ yêu cầu học viên mở DevTools và thực hành theo từng kịch bản.*

#### Scenario 1: "The Flash" (Flickering UI)
- **Tình huống:** App check Auth khi mới vào. Màn Login hiện lên 0.1s rồi mới chuyển sang Dashboard.
- **Debug Tool:** Mắt thường + React DevTools.
- **Cause:** State `isAuthenticated` mặc định là `false`. Khi `useEffect` chạy xong mới set `true`.
- **Fix:** Thêm state `isCheckingAuth`. Chỉ render App khi `isCheckingAuth = false`.

#### Scenario 2: "The Silent Failure" (Bấm không phản hồi)
- **Tình huống:** Bấm Update Profile, không thấy lỗi, không thấy update.
- **Debug Tool:** **Network Tab**.
- **Action:**
  1. Mở tab Network.
  2. Bấm nút Save.
  3. Lọc `Fetch/XHR`.
  4. Thấy API `PUT /users/me` đỏ lòm (400 Bad Request).
  5. Bấm vào request -> Tab **Payload** xem gửi gì -> Tab **Preview** xem server chửi gì.
- **Kết luận:** Gửi `bio` quá dài (ví dụ). Backend trả error message nhưng Frontend quên `toast.error(err.response.message)`.

#### Scenario 3: "The Infinite Spinner"
- **Tình huống:** Spinner quay mãi không dừng.
- **Debug Tool:** **Sources Tab (Debugger)**.
- **Showcase:**
  1. Vào code, click vào số dòng `loginMutation.mutate()`.
  2. Bấm Login.
  3. Trình duyệt đóng băng (Pause).
  3. Trình duyệt đóng băng (Pause).
  4. Hover chuột vào biến `data` xem giá trị.
  5. Dùng phím tắt để di chuyển:
  
  | Phím | Ý nghĩa | Hành động |
  | :--- | :--- | :--- |
  | **F10** | Step Over | Chạy xong dòng này, nhảy xuống dòng dưới (không chui vào hàm) |
  | **F11** | Step Into | Chui vào bên trong hàm (để soi chi tiết) |
  | **Shift+F11** | Step Out | Chạy cho xong hàm hiện tại rồi nhảy ra ngoài |

  6. **Kết luận:** Code chạy vào `onError` nhưng trong `onError` không có lệnh tắt loading state (nếu quản lý thủ công).

#### Scenario 4: "Why useQuery fetches twice?"
- **Tình huống:** Thấy 2 request `/me` gọi liên tiếp.
- **Debug Tool:** **React Query DevTools**.
- **Showcase:**
  1. Mở React Query DevTools (bông hoa đỏ).
  2. Chọn query `['me']`.
  3. Nhìn Explorer bên phải.
  4. Thấy `staleTime: 0`.
  5. Window Focus -> Refetch.
- **Giải thích:** Đây là tính năng, không phải lỗi. React Query mặc định data cũ ngay lập tức.

## 🚫 ANTI-PATTERNS (CẤM LÀM)
- **Console.log('vào đây chưa 1', 'vào đây chưa 2'):** Cách debug thủ công. Hãy học dùng Breakpoint.
- **Bỏ qua lỗi Network:** Thấy đỏ nhưng không bấm vào xem chi tiết Response. "Thầy ơi lỗi đỏ" là câu hỏi cấm. Phải trả lời "Lỗi 400 do thiếu field abc".
- **Dùng Alert để debug:** `alert(data)` sẽ block thread render. Đừng dùng.

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Skeleton giật cục:** Skeleton hiện rồi biến mất rồi lại hiện Spinner. --> Do logic `isLoading` và `isFetching` chồng chéo nhau.
2.  **DevTools không hiện Component:** Do build production hoặc React DevTools bị cấu hình sai. Thử restart trình duyệt.
3.  **Network Tab trống trơn:** Quên bấm nút "Record" hoặc lỡ tay set filter (ví dụ lọc chữ "abc" mà request là "xyz").

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao status 200 mà vẫn lỗi logic?"*
    *   **A:** HTTP 200 chỉ nghĩa là "Server nhận được tin nhắn". Còn nội dung bên trong có thể là `{ success: false, message: "Sai pass" }`. Luôn check Response Body.
2.  **Q:** *"Muốn check xem mobile hiển thị thế nào thì làm sao?"*
    *   **A:** Toggle Device Toolbar (Ctrl+Shift+M) trong DevTools.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Debug Challenge.
*GV cố tình sửa code `apiClient` sai URL (ví dụ `/user/me` thay vì `/users/me`).*
- Yêu cầu học viên tìm ra nguyên nhân trong 3 phút bằng Network Tab.
- Ai tìm ra trước được điểm cộng.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Custom 404 Page & Error Boundary.
1. Tạo trang `NotFoundPage.tsx` đẹp. Config Router `path="*"` để bắt link sai.
2. (Nâng cao) Dùng `ErrorBoundary` của React để bắt lỗi Crash App (Màn hình trắng chết chóc).

## 9️⃣ CHECKPOINT & EVALUATION
- **Code:** Có Skeleton ở Profile, Spinner ở Button.
- **Skill:** Học viên biết mở Network Tab để đọc Response khi gặp lỗi API.
- **Skill:** Biết dùng Breakpoint cơ bản (F10/F11).

## 🧠 DEBUG FLOW CHUẨN (Instructor Mantra)
_"Hãy in cái này ra và dán lên tường."_

1.  **App lỗi?** → Nhìn UI (Có loading? Có error?)
2.  **Không rõ?** → Mở **Network Tab**.
3.  **Request fail?** → Xem **Status + Response** (Preview tab).
4.  **Response OK nhưng UI sai?** → Mở **React DevTools** soi Props/State.
5.  **Logic phức tạp?** → Set **Breakpoint** (F10 - Step Over).

## 🏁 LEVEL UP MOMENT
_"Từ hôm nay, khi gặp bug, bạn **KHÔNG** hỏi 'Thầy ơi lỗi gì đây?'.
Bạn sẽ hỏi: 'Em thấy response trả về 400 nhưng em chưa biết sai field nào trong payload'.
Đó là sự khác biệt giữa Junior và Mid-level."_

## 🔟 TEACHING NOTES
- **Slow Down:** Phần Breakpoint rất mới lạ với nhiều bạn quen `console.log`. Hãy demo thật chậm.
- **Mindset:** Hãy nhấn mạnh "Lỗi là bạn". Đừng sợ lỗi đỏ. Lỗi đỏ là manh mối để giải quyết vấn đề.
