# LESSON PLAN: SESSION 16 - DEPLOYMENT & GRADUATION

## 1️⃣ SESSION OVERVIEW
- **Title:** Hello World: Deploying to Production
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will deploy their React application to a public cloud provider (Vercel) and handle production-specific configurations (Environment Variables, SPA Routing).
- **Outcome:** A live URL (e.g., `my-shopping-cart.vercel.app`) that can be shared on CV/Portfolio.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Hôm nay là ngày trọng đại.
Code chạy trên máy của bạn (Localhost) là chuyện bình thường.
Nhưng để code chạy trên Internet, cho cả thế giới truy cập, đó là một câu chuyện khác.

Các bạn có biết hội chứng 'Works on my machine' không?
Hôm nay ta sẽ chữa hội chứng đó.
Chúng ta sẽ đưa 'đứa con tinh thần' ra mắt công chúng.
Nếu hôm nay deploy lỗi, đừng lo. Đó là bài học quý giá nhất mà local không bao giờ dạy bạn._

**Agenda hôm nay:**
1. Clean Code lần cuối.
2. Đẩy lên GitHub.
3. Kéo về Vercel và cấu hình biến môi trường.
4. Demo sản phẩm & Chụp ảnh tốt nghiệp."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Nhiều Junior viết trong CV là 'biết React' nhưng đưa link demo thì 404 hoặc loading mãi mãi. Buổi này giúp học viên có 'Bằng chứng thép' cho năng lực của mình."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🌍 Host vs Local
- **Local:** Máy bạn có sẵn Node.js, có file `.env`, có Backend chạy ở port 3000.
- **Host (Vercel):** Là máy tính của người khác. Nó không biết file `.env` của bạn ở đâu (vì bạn không commit). Nó không biết API Backend ở đâu.
- **Nhiệm vụ:** Dạy cho Vercel biết những điều đó thông qua **Environment Variables Dashboard**.

### 🛤️ SPA Routing Problem
- **Problem:** React App là Single Page. Chỉ có 1 file `index.html`.
- Khi user vào `/login`, Vercel tìm file `login.html` -> Không thấy -> Trả về 404.
- **Solution:** Config Rewrite rules. Bảo Vercel: "Gặp bất kỳ link nào lạ, hãy cứ trả về `index.html` để React Router tự xử lý".

**Fix:** Tạo file `vercel.json` ở root:
```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

> **⚠️ BACKEND DEPLOYMENT DISCLAIMER**
>
> Buổi này tập trung vào **Frontend Deployment**.
> Nếu Backend chưa deploy:
> - App có thể không login được (API fail).
> - Nhưng deployment vẫn được coi là **PASS**.
>
> **Mục tiêu:** Học quy trình deploy + config Production Environment. Backend deployment là scope riêng.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: PRE-FLIGHT CHECK (30 mins)

#### ⏱️ TIMELINE
- **00-10’:** Review lại `vite.config.ts`, xóa `console.log`.
- **10-30’:** Push code lên GitHub (Check `.gitignore`).

#### Step 1: Cleanup Build
Open `vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  // Instructor Explain: "Thêm phần build này để tránh warning khi bundle hơi lớn. Đây KHÔNG phải tối ưu performance thật, chỉ là tắt cảnh báo."
  build: {
    chunkSizeWarningLimit: 1600,
  },
})
```

#### Step 2: Push to GitHub
**Instructor Script:**
_"Mở file `.gitignore`. Đảm bảo có `.env` và `node_modules`. Nếu lỡ commit `.env` lên rồi thì phải xóa history ngay (Hướng dẫn xóa sau)."_

```bash
git add .
git commit -m "chore: ready for deployment"
git push origin main
```

**🚦 MID-SESSION CHECKPOINT**
- Vào GitHub repo -> Thấy code mới nhất.
- **Không thấy** file `.env` trên GitHub.

---

### PHASE 2: DEPLOY TO VERCEL (45 mins)

#### Step 1: Connect Vercel
1. Vào `vercel.com` -> Login bằng GitHub.
2. Click **Add New** -> **Project**.
3. Chọn repo `shopping-cart` vừa push.

#### Step 2: Config Build (Quan trọng)
- **Framework Preset:** Vite
- **Root Directory:** `./`
- **Build Command:** `npm run build`
- **Output Directory:** `dist`

#### Step 3: Environment Variables (Tử huyệt)
**Instructor Script:**
_"Đây là bước 90% các bạn sẽ quên. Vercel không đọc được file .env của bạn. Bạn phải nhập thủ công vào đây."_

- Click **Environment Variables**.
- Key: `VITE_API_URL`
- Value: `https://api-shopping.piedteam.com` (Hoặc URL Backend thật đã deploy).
  *Note: Nếu chưa có Backend thật, dùng tạm MockAPI hoặc dặn học viên là App sẽ không fetch được data.*

#### Step 4: Deploy
- Click **Deploy**.
- Chờ màn hình "Confetti" (Pháo giấy).

### 🚫 ANTI-PATTERNS (CẤM LÀM)
- **Hardcode API URL trong code:** Khi sửa API phải build lại app. Dùng Env Var cho phép đổi API mà không cần sửa code.
- **Commit `.env` để Vercel đọc được:** Lỗ hổng bảo mật sơ đẳng.
- **Quên config SPA Routing:** (Vercel tự động support Vite, nhưng nếu deploy Netlify phải thêm file `_redirects`).

---

### PHASE 3: VERIFICATION & DEMO (45 mins)

#### Step 1: Verify Production
- Mở link `https://shopping-cart-xyz.vercel.app`.
- Test Login (Nếu có Backend thật).
- F5 trang `/me`. Nếu bị 404 -> Vercel chưa hiểu SPA.

#### Step 2: Student Demo (5 mins each)
**Instructor Script:**
_"Bây giờ đến lượt các bạn tỏa sáng. Hãy lên bảng (hoặc share screen). Demo 3 tính năng: Login, Update Profile, Logout.
Kể 1 bug khó nhất bạn đã gặp và cách fix."_

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Trắng trang sau khi Deploy:**
    *   *Cause:* Sai URL API hoặc Code bị crash runtime.
    *   *Debug:* Mở Console trên Production (F12) -> Xem lỗi đỏ.
2.  **Lỗi CORS trên Production:**
    *   *Cause:* Backend chưa cho phép domain `vercel.app` truy cập.
    *   *Fix:* Nhờ Backend Developer add domain vào whitelist hoặc dùng Proxy (Nâng cao).
3.  **Route /me bị 404 khi F5:**
    *   *Cause:* Thiếu config rewrite (Nếu chưa tạo `vercel.json` ở bước trước).
    *   *Reminder:* File `vercel.json` đã được tạo ở Phase 1. Nếu quên thì tạo ngay:
    ```json
    {
      "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
    }
    ```

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Tại sao ở local chạy được mà lên đây chết?"*
    *   **A:** Môi trường khác nhau (Linux vs Windows), Biến môi trường thiếu, Case sensitive (Linux phân biệt hoa thường file name).
2.  **Q:** *"Làm sao để update code sau này?"*
    *   **A:** Chỉ cần `git push`. Vercel kết nối với GitHub nên sẽ tự động trigger build lại (CI/CD cơ bản).

## 🏁 CLOSING & ROADMAP
_"Chúc mừng các bạn đã tốt nghiệp ReactJS Phase 1.
Chúng ta đã đi từ những dòng `console.log` đầu tiên đến một App hoàn chỉnh trên Production.

**Next Steps (Phase 2):**
- Shopping Cart Logic (Redux/Zustand complex).
- Payment Gateway.
- Performance Optimization (Lazy loading).

Đừng dừng lại. Code mỗi ngày. Hẹn gặp lại các bạn ở Phase 2!"_

## ✅ DEPLOYMENT DONE CHECKLIST
_"Trước khi về, hãy tick đủ 5 ô này:"_

- [ ] Repo GitHub public (hoặc private nhưng có invite reviewer).
- [ ] README đầy đủ (Setup + Tech Stack).
- [ ] URL Production chạy được (không crash).
- [ ] Có thể demo 3 flow chính (Auth, Profile, Logout).
- [ ] Link đã thêm vào CV/Portfolio.

## 📌 CV / PORTFOLIO TIP
_"Đây là cách viết CV để HR/Tech Lead chú ý:"_

**❌ Không viết:**
- "Biết React"
- "Có kinh nghiệm Frontend"

**✅ Hãy viết:**
- **Live Demo:** `https://shopping-cart.vercel.app`
- **GitHub Repo:** `https://github.com/yourname/shopping-cart`
- **Tech Stack:** React 18, TypeScript, TanStack Query, Zustand, Tailwind v4
- **Highlight:** "Built & deployed a production-ready SPA with auto-refresh token strategy"

> **Pro Tip:** Thêm screenshot đẹp vào README. Ảnh đẹp = Click nhiều.

## 🔟 TEACHING NOTES
- **Celebration:** Hãy tạo không khí ăn mừng. Mua ít kẹo hoặc vỗ tay thật lớn sau mỗi demo.
- **Support Backend:** Nếu không có Backend live, hãy chuẩn bị sẵn một bản Backend chạy trên Render/Heroku để share URL cho học viên config. Đừng để họ deploy cái "xác không hồn".
