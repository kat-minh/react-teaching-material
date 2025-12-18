# LESSON PLAN: SESSION 15 - DEBUG WORKSHOP

> **[NEW - SPLIT FROM ORIGINAL SESSION 14]**

## 1️⃣ SESSION OVERVIEW

- **Title:** The Detective: Mastering DevTools & Debugging Workflows
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will master professional debugging techniques using Chrome DevTools, React Query DevTools, and systematic debugging workflows to find and fix bugs independently.
- **Outcome:** Students who can diagnose and fix bugs using DevTools instead of asking "Thầy ơi lỗi gì đây?", and who understand the debugging workflow used in professional development teams.

## 2️⃣ INSTRUCTOR OPENING SCRIPT

\_"Chào các bạn. Hôm trước chúng ta đã làm đẹp app. Hôm nay chúng ta học kỹ năng quan trọng nhất của một developer: **Debug**.

Các bạn có biết trong công ty thật, developer dành bao nhiêu % thời gian để debug?
**50%.**

Một developer giỏi debug có thể làm việc nhanh gấp **3 lần** người chỉ biết `console.log`.

Hôm nay tôi sẽ dạy các bạn 'nghệ thuật thám tử':

1. Làm sao đọc được 'dấu vết' từ Network Tab
2. Làm sao 'bắt sống' bug bằng Breakpoint
3. Làm sao tư duy có hệ thống thay vì mò mẫm

Sau buổi này, các bạn sẽ không bao giờ hỏi 'Thầy ơi lỗi gì đây' nữa.
Các bạn sẽ hỏi: 'Em thấy response 400, em đã check payload và thấy thiếu field xyz, cách fix đúng là gì?'"\_

> **🔥 WHY THIS SESSION EXISTS?**
> \_"Junior developer gặp bug → Hoảng loạn → Hỏi senior.
> Mid-level developer gặp bug → Mở DevTools → Tự fix trong 5 phút.
>
> Buổi này chuyển bạn từ Junior sang Mid-level."\_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🩺 The Detective Mindset

**Instructor Script:**
_"Debug không phải 'thử luck'. Debug là khoa học. Giống như thám tử phá án."_

```
┌─────────────────────────────────────┐
│  BUG = CRIME SCENE (Hiện trường)   │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  EVIDENCE (Bằng chứng)              │
│  - Network Logs                     │
│  - Console Errors                   │
│  - React DevTools State             │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  HYPOTHESIS (Giả thuyết)            │
│  "Có thể do token expired"          │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  TEST (Thử nghiệm)                  │
│  Set breakpoint, check token value  │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  FIX (Giải pháp)                    │
└─────────────────────────────────────┘
```

### 🧰 Debug Tools Hierarchy

| Tool                        | Use Case                             | Power Level       |
| :-------------------------- | :----------------------------------- | :---------------- |
| **👀 Visual Inspection**    | UI render sai (button not showing)   | ⭐ Basic          |
| **📝 Console.log**          | Check variable value (quick & dirty) | ⭐⭐ Junior       |
| **🌐 Network Tab**          | API errors (400, 401, 500)           | ⭐⭐⭐ Essential  |
| **🔍 React DevTools**       | Props/State inspection               | ⭐⭐⭐⭐ Advanced |
| **🛑 Breakpoints**          | Step-by-step code execution          | ⭐⭐⭐⭐⭐ Pro    |
| **⚛️ React Query DevTools** | Cache/Query state debugging          | ⭐⭐⭐⭐⭐ Pro    |

### 🚫 Anti-Patterns to Avoid

**Instructor Script:**
_"Trước khi học làm đúng, hãy xem cách làm SAI mà 90% newbie mắc phải:"_

1. **Console.log Debugging**

   ```tsx
   // ❌ The "Console Hell"
   console.log("vào đây chưa 1");
   console.log("vào đây chưa 2");
   console.log("data:", data); // undefined
   console.log("vào đây chưa 3");
   ```

   → **Problem:** Slow, cluttered, doesn't show execution flow
   → **Fix:** Use Breakpoints

2. **Ignoring Error Messages**

   ```
   ❌ Error: Cannot read property 'map' of undefined
   Student: "Thầy ơi lỗi đỏ"
   ```

   → **Problem:** Didn't READ the error
   → **Fix:** Read error message → Google if needed → Check variable type

3. **Not Checking Network Tab**
   ```
   Student: "Em bấm Login mà không vào được"
   Instructor: "Em check Network Tab chưa?"
   Student: "Chưa... Network là gì ạ?"
   ```
   → **Problem:** 80% API bugs visible in Network Tab
   → **Fix:** ALWAYS check Network first

---

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: CHROME DEVTOOLS MASTERY (50 mins)

#### Step 1: Network Tab Deep Dive (20 mins)

**Instructor Demo:**
_"Giờ tôi sẽ phá app và các bạn dùng Network Tab tìm lỗi."_

**Scenario: Login Fails Silently**

1. **Setup:** Giảng viên cố tình sửa API endpoint:

   ```tsx
   // In apiClient.ts
   baseURL: "http://localhost:3000/api/wrong"; // ❌ Wrong URL
   ```

2. **Symptoms:**

   - Bấm Login → Spinner quay → Không vào được → Không có error toast

3. **Debug Process (Live Demo):**

**Step-by-Step:**

```markdown
1. Mở DevTools (F12)
2. Tab **Network**
3. Bấm nút Login
4. Quan sát:
   ┌─────────────────────────────────────┐
   │ Name: login │
   │ Status: 404 │ ← ĐỎ = LỖI
   │ Type: xhr │
   └─────────────────────────────────────┘

5. Click vào request "login"

6. Tab **Headers**:

   - Request URL: http://localhost:3000/api/wrong/users/login
   - Request Method: POST
     → Thấy URL sai!

7. Tab **Preview**:

   - {"message": "Cannot POST /api/wrong/users/login"}
     → Backend nói không có route này

8. Tab **Payload**:
   - {email: "test@example.com", password: "123456"}
     → Data gửi đi đúng

CONCLUSION: URL sai, không phải data sai.
FIX: Sửa baseURL trong apiClient.ts
```

**Instructor Explain:**

> "Network Tab là **nguồn sự thật duy nhất**. Nó không bao giờ nói dối.
>
> - Request gửi đi chính xác là gì?
> - Response nhận về chính xác là gì?
> - Status code là gì?
>
> 80% bug API nhìn Network Tab là tìm ra ngay."

---

**Key Filters in Network Tab:**

**Instructor Demo:**

```markdown
Filters:

- **All**: Tất cả requests (CSS, JS, Images, API)
- **Fetch/XHR**: Chỉ API calls ← DÙNG CÁI NÀY
- **Doc**: HTML documents
- **CSS, JS, Img**: Assets

Search:

- Gõ tên API: "login" → Lọc chỉ login requests
```

---

#### Step 2: Console Tab Mastery (10 mins)

**Instructor Script:**
_"Console không chỉ để log. Nó còn để test code trực tiếp."_

**Demo: Console Tricks**

1. **Execute Code Live:**

   ```js
   // Trong Console tab
   localStorage.getItem("auth-storage");
   // → Shows token in real-time

   JSON.parse(localStorage.getItem("auth-storage"));
   // → Parse JSON to see structure
   ```

2. **Check Variable Type:**

   ```js
   const data = undefined;
   typeof data; // "undefined"
   Array.isArray(data); // false
   ```

3. **Test Function:**
   ```js
   // Type this in Console
   await fetch("http://localhost:3000/users/me", {
     headers: { Authorization: "Bearer xyz" },
   });
   // → Test API directly without UI
   ```

**Instructor Warning:**

> "Console.log is OK for QUICK checks. But don't leave 100 console.logs in production code. Use it → Debug → DELETE IT."

---

#### Step 3: Sources Tab & Breakpoints (20 mins)

**Instructor Script:**
_"Đây là vũ khí tối thượng. Breakpoint = dừng thời gian để xem code chạy từng bước."_

**Demo Scenario: Infinite Spinner**

**Problem:** Login button spinner quay mãi không dừng.

**Debug with Breakpoint:**

1. **Set Breakpoint:**

   ```markdown
   - Mở tab **Sources**
   - Tìm file: `LoginForm.tsx`
   - Click vào số dòng bên trái (dòng có `loginMutation.mutate()`)
   - → Xuất hiện dot đỏ = breakpoint
   ```

2. **Trigger:**

   - Bấm nút Login
   - → Trình duyệt đóng băng (Paused in debugger)

3. **Inspect:**

   ```markdown
   - Hover chuột vào biến `data`, `error` → Xem giá trị
   - Panel bên phải:
     - **Scope**: Biến local/global
     - **Call Stack**: Hàm gọi hàm (execution stack)
     - **Watch**: Theo dõi biến cụ thể
   ```

4. **Step Through Code:**

   | Key           | Action    | Use When                                               |
   | :------------ | :-------- | :----------------------------------------------------- |
   | **F8**        | Resume    | Chạy tiếp đến breakpoint kế                            |
   | **F10**       | Step Over | Chạy xong dòng này, nhảy dòng sau (không chui vào hàm) |
   | **F11**       | Step Into | Chui vào bên trong hàm (để soi chi tiết)               |
   | **Shift+F11** | Step Out  | Thoát khỏi hàm đang ở, về hàm gọi nó                   |

5. **Find Bug:**

   ```tsx
   // Discover this code
   loginMutation.mutate(data, {
     onSuccess: () => {
       // ✅ This runs
       console.log("success");
     },
     onError: (err) => {
       // ❌ Error handling MISSING
       // → Spinner never stops
     },
   });
   ```

6. **Fix:**
   ```tsx
   onError: (err) => {
     toast.error(err.message);
     // Mutation auto-resets isPending to false
   };
   ```

**Instructor Emphasize:**

> "Breakpoint > 1000 console.log.
> Với breakpoint, bạn KHÔNG CẦN đoán. Bạn NHÌN THẤY code chạy thế nào."

---

### PHASE 2: REACT DEVTOOLS & QUERY DEVTOOLS (30 mins)

#### Step 1: React DevTools - Components Tab (15 mins)

**Instructor Demo:**
_"React DevTools cho phép bạn 'hack' vào component, xem props/state như một người trong cuộc."_

**Install:**

- Chrome Extension: "React Developer Tools"

**Demo Scenario: Props Not Passing**

**Problem:** Child component không nhận được props

**Debug:**

1. **Open React DevTools:**

   - F12 → Tab "Components"

2. **Inspect Component Tree:**

   ```
   <App>
     └── <ProfilePage>
           └── <UserCard user={undefined}> ← ❌ undefined!
   ```

3. **Check Props:**

   - Click vào `<UserCard>`
   - Panel bên phải:
     ```
     props
       user: undefined ← PROBLEM!
     ```

4. **Trace Back:**

   - Click vào `<ProfilePage>`
   - Check `user` variable:
     ```
     hooks
       State: {}
       Query: { data: undefined, isLoading: true }
     ```
     → **Aha!** Query chưa load xong mà đã render component

5. **Fix:**
   ```tsx
   // Add loading check
   if (isLoading) return <Skeleton />;
   return <UserCard user={user} />;
   ```

**Instructor Tips:**

> "React DevTools có 2 tabs:
>
> - **Components**: Cây component, props, state
> - **Profiler**: Performance (nâng cao, không dạy buổi này)"

---

#### Step 2: React Query DevTools (15 mins)

**Instructor Script:**
_"React Query DevTools là 'X-ray' cho cache. Nó cho thấy query nào đang chạy, data có cũ không, lỗi ở đâu."_

**Demo Scenario: Double Fetch**

**Problem:** Thấy API `/me` gọi 2 lần liên tiếp

**Debug:**

1. **Open Query DevTools:**

   - Bông hoa đỏ góc màn hình (đã cài Session 09)

2. **Inspect Query:**

   ```markdown
   Queries:

   - ['me']
     - Status: fetching → success
     - fetchStatus: fetching
     - dataUpdatedAt: ...
     - Observers: 2 ← ĐÂY LÀ LÝ DO!
   ```

3. **Analysis:**

   - 2 Observers = 2 components đang dùng `useUser()` hook
   - Nhưng chỉ 1 request được gửi (deduplication works ✅)

4. **Check Network Tab:**
   - Chỉ thấy 1 request thật sự
   - Request thứ 2 là do React StrictMode (dev only)

**Instructor Explain:**

> "React Query rất thông minh.
>
> - Nhiều component dùng cùng query key → Chỉ gọi API 1 lần.
> - Data được share giữa các component.
>
> Query DevTools cho thấy:
>
> - Fresh: Data mới, không cần refetch
> - Stale: Data cũ, sẵn sàng refetch
> - Fetching: Đang gọi API
> - Paused: Network offline"

---

### PHASE 3: DEBUGGING SCENARIOS (30 mins)

**Instructor Script:**
_"Giờ tôi cho các bạn 4 kịch bản thực tế. Mỗi nhóm 2 người, debug theo checklist tôi đưa."_

#### Scenario 1: "The Flash" (Flickering UI)

**Problem:** Login screen hiện 0.1s rồi mới vào Dashboard

**Checklist:**

```markdown
1. Mở React DevTools
2. Check component `RequireAuth`
3. Xem `isAuthenticated` state
4. Nếu mặc định `false` → Flash xảy ra
5. Fix: Add `isHydrated` check (Session 05)
```

**Expected Time:** 5 phút

---

#### Scenario 2: "Silent Failure" (Button Không Phản Hồi)

**Problem:** Bấm Update Profile không thấy gì xảy ra

**Checklist:**

```markdown
1. Mở Network Tab
2. Filter "Fetch/XHR"
3. Bấm nút Update
4. Check request:
   - Status: 400? 422?
   - Response body: Error message?
5. Fix: Add error handling
```

**Expected Time:** 3 phút

---

#### Scenario 3: "Infinite Spinner"

**Problem:** Spinner quay mãi không dừng

**Checklist:**

```markdown
1. Set Breakpoint tại `onSuccess`
2. Bấm nút → Breakpoint hit?
   - YES → Check logic sau `onSuccess`
   - NO → Breakpoint tại `onError`
3. If hit `onError` → Check error message
4. Fix: Add proper error handling
```

**Expected Time:** 7 phút

---

#### Scenario 4: "Why Query Fetches Twice?"

**Problem:** Thấy `/me` call 2 lần trong Network

**Checklist:**

```markdown
1. Check React Query DevTools
2. Look at query `['me']`
3. Check `Observers` count
4. Check `staleTime` config
5. Check if React StrictMode enabled (dev only)
6. Conclusion: Normal behavior OR real bug?
```

**Expected Time:** 5 phút

---

### PHASE 4: THE DEBUG FLOW CHECKLIST (10 mins)

**Instructor Script:**
_"Hãy in cái checklist này ra và dán lên tường."_

```markdown
# DEBUG FLOW CHECKLIST

## Step 1: OBSERVE

- [ ] Lỗi gì? (UI sai, crash, không phản hồi?)
- [ ] Có error message trong Console không?

## Step 2: REPRODUCE

- [ ] Làm lại lỗi được không?
- [ ] Lỗi xảy ra ở điều kiện nào?
  - [ ] Offline?
  - [ ] Token expired?
  - [ ] Data rỗng?

## Step 3: ISOLATE

- [ ] Lỗi ở đâu?
  - [ ] UI (Component render)?
  - [ ] API (Network)?
  - [ ] State (Zustand/Query)?
  - [ ] Logic (Business code)?

## Step 4: INSPECT

- [ ] Network Tab:

  - [ ] Request URL đúng?
  - [ ] Request Headers có token?
  - [ ] Request Body đúng format?
  - [ ] Response Status?
  - [ ] Response Body có message?

- [ ] React DevTools:

  - [ ] Props truyền xuống đúng?
  - [ ] State có giá trị đúng?

- [ ] Query DevTools:
  - [ ] Query status: fetching/success/error?
  - [ ] Data có trong cache?

## Step 5: HYPOTHESIS

- [ ] Viết ra giả thuyết: "Có thể do..."

## Step 6: TEST

- [ ] Set Breakpoint
- [ ] Check variable values
- [ ] Confirm hypothesis

## Step 7: FIX

- [ ] Sửa code
- [ ] Test lại

## Step 8: VERIFY

- [ ] Lỗi đã mất?
- [ ] Không tạo lỗi mới?
- [ ] Edge cases OK?
```

---

## 🚫 ANTI-PATTERNS (CẤM LÀM)

**Instructor Script:**
_"Những điều này TUYỆT ĐỐI đừng làm:"_

1. **Console.log('vào đây chưa 1', 'vào đây chưa 2')**

   - ❌ Slow, messy, unprofessional
   - ✅ Use Breakpoints

2. **Hỏi "Thầy ơi lỗi đỏ" mà không đọc error message**

   - ❌ Lazy debugging
   - ✅ READ the error, Google it, then ask specific question

3. **Không mở Network Tab khi API fail**

   - ❌ Blind debugging
   - ✅ Network Tab first, ALWAYS

4. **Dùng `alert()` để debug**

   - ❌ Blocks UI thread
   - ✅ Use Console or Breakpoints

5. **Debug trên Production build**
   - ❌ Code minified, hard to read
   - ✅ Debug on Development build

---

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1. **DevTools không hiện Component Tab**

   - _Cause:_ React DevTools chưa cài hoặc app không phải React
   - _Fix:_ Install extension, reload page

2. **Breakpoint không hit**

   - _Cause:_ Code path không chạy qua dòng đó
   - _Fix:_ Set breakpoint ở chỗ chắc chắn chạy (vd: component mount)

3. **Network Tab trống trơn**

   - _Cause:_ Quên bật "Record" hoặc filter sai
   - _Fix:_ Click nút đỏ (Record), clear filters

4. **Query DevTools không hiện**
   - _Cause:_ Chưa cài `@tanstack/react-query-devtools`
   - _Fix:_ Re-check Session 09 setup

---

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1. **Q:** _"Khi nào nên dùng Breakpoint thay vì console.log?"_

   - **A:** Khi cần xem execution flow, check nhiều biến cùng lúc, hoặc logic phức tạp (if/else nhiều nhánh).

2. **Q:** _"Tại sao status 200 mà app vẫn lỗi?"_

   - **A:** Status 200 = HTTP success. Nhưng response body có thể là `{success: false}`. Luôn check Response Body.

3. **Q:** _"Làm sao biết lỗi ở Frontend hay Backend?"_
   - **A:**
     - Network Tab: Request gửi đi đúng? → Frontend OK
     - Response có lỗi? → Backend issue
     - Request sai format? → Frontend issue

---

## 7️⃣ IN-CLASS MINI TASK

**Task:** Debug Challenge Tournament

**Setup:**
Giảng viên chia lớp thành 4 nhóm, mỗi nhóm được 1 bug khác nhau (pre-planted).

**Bugs:**

1. API URL sai (404)
2. Token missing in headers (401)
3. Validation error (422) nhưng không hiện toast
4. Infinite loop do useEffect dependency

**Rules:**

- Time limit: 10 phút/bug
- Phải dùng DevTools (không được hỏi code)
- Nhóm tìm ra + fix đúng → Điểm cộng

**Acceptance:**

- Tìm ra nguyên nhân (5 điểm)
- Fix được bug (5 điểm)
- Giải thích được cho lớp (bonus 2 điểm)

---

## 8️⃣ HOMEWORK / EXTENSION TASK

**Yêu cầu:** Debug Log Journal

1. **Tạo file `debug-log.md`** trong project
2. Mỗi lần gặp bug trong tuần, ghi vào file theo format:

```markdown
## Bug #1: Login Spinner Stuck

**Date:** 2025-12-18
**Symptom:** Spinner quay mãi không dừng
**Tool Used:** Breakpoint in Sources tab
**Root Cause:** `onError` callback missing
**Fix:** Add error handling with toast
**Time to Fix:** 10 minutes
**Lesson Learned:** Always implement both onSuccess AND onError
```

3. Cuối tuần nộp file, ít nhất 3 bugs.

**Goal:** Build habit of systematic debugging

---

## 9️⃣ CHECKPOINT & EVALUATION

**Học viên Pass buổi này khi:**

- [ ] Biết mở và đọc Network Tab (Status, Headers, Payload, Preview)
- [ ] Biết set Breakpoint và dùng F10/F11 để step through code
- [ ] Biết dùng React DevTools để xem Props/State
- [ ] Biết dùng React Query DevTools để check query status
- [ ] Có thể debug 1 API error trong <5 phút (không hỏi giảng viên)

**Level Up Signal:**

- Học viên chủ động mở DevTools trước khi hỏi
- Học viên nói: "Em thấy response 400, field X bị thiếu"
- Không còn hỏi: "Thầy ơi lỗi gì đây?"

---

## 🔟 TEACHING NOTES

### Time Management

- **00-10':** Opening + Mindset
- **10-60':** PHASE 1 (Chrome DevTools)
- **60-90':** PHASE 2 (React/Query DevTools)
- **90-110':** PHASE 3 (Scenarios)
- **110-120':** Debug Flow Checklist + Q&A

### Emphasis Points

1. **Network Tab is Truth** - Nhắc đi nhắc lại
2. **Breakpoints > Console.log** - Demo difference
3. **Systematic vs Random debugging** - Show checklist

### Red Flags

- Học viên vẫn dùng console.log sau bài này → Pull aside, force Breakpoint practice
- Học viên không check Network Tab → Fail them on Mini Task
- Học viên copy error message không đọc → Teach "Read First, Google Second, Ask Last"

### Motivation Script

> \_"Các bạn biết sự khác biệt giữa Junior 3 năm kinh nghiệm và Senior 3 năm kinh nghiệm là gì không?
>
> Junior: Gặp bug → Hoảng → Hỏi → Chờ → Mất 1 tiếng
> Senior: Gặp bug → DevTools → Tìm ra → Fix → 5 phút
>
> Kỹ năng debug = kỹ năng sinh tồn của developer."\_

---

## 📚 REFERENCES FOR STUDENTS

**Chrome DevTools Official Docs:**

- [Network Tab](https://developer.chrome.com/docs/devtools/network/)
- [Debugging JavaScript](https://developer.chrome.com/docs/devtools/javascript/)
- [Breakpoints Guide](https://developer.chrome.com/docs/devtools/javascript/breakpoints/)

**React DevTools:**

- [Official Guide](https://react.dev/learn/react-developer-tools)

**Debugging Philosophy:**

- [The Art of Debugging](https://jvns.ca/blog/2019/06/23/a-few-debugging-resources/)

---

## 🎯 SUCCESS CRITERIA

**By the end of this session, students should:**

1. Master Chrome DevTools (Network, Console, Sources tabs)
2. Use Breakpoints effectively (F10, F11 navigation)
3. Inspect React component state with React DevTools
4. Debug React Query cache issues with Query DevTools
5. Follow a systematic debugging workflow (Checklist)
6. Fix bugs independently without asking "What's the error?"

**Career Impact:**
_"This session teaches skills that directly translate to job performance. Companies value developers who can debug independently. This alone can boost your starting salary by 20%."_

---

**END OF SESSION 15**
