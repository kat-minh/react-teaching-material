# 📚 GIÁO TRÌNH REACTJS MASTER - TEACHING MATERIALS

> **Tài liệu đào tạo ReactJS chuyên nghiệp cho Giảng viên & Mentor**  
> **Đối tượng:** Học viên đã hoàn thành MERN Stack (Backend ExpressJS + TypeScript)  
> **Thời lượng:** 2 tháng (18 buổi học = 36 giờ)  
> **Version:** 1.1.0

---

## 🎯 MỤC TIÊU KHÓA HỌC

Khóa học tập trung vào **xử lý API trong React** và **kiến trúc Frontend production-ready**:

1. **React Core Foundation** - Components, Props, State, Hooks
2. **API Handling** - Axios interceptors, error normalization, refresh token flow
3. **Server State Management** - TanStack Query (React Query)
4. **Form & Validation** - React Hook Form + Zod
5. **Global State** - Zustand cho auth tokens & UI state
6. **Production Patterns** - Loading/Error/Empty states, protected routes, file structure

### 🏆 Outcome Chính

- ✅ Thành thạo xử lý API (query/mutation, loading/error, auth token, refresh token)
- ✅ Nắm React core đủ để "code React khá" với tư duy kiến trúc production
- ✅ Build Frontend hoàn chỉnh cho Backend `shoppingCardBE`

---

## 🛠️ TECH STACK

### Core

- **Vite** - Build tool
- **React 18** - Client features focus
- **TypeScript** - Frontend patterns

### Routing

- **React Router v6** - Nested routes, layouts

### UI & Styling

- **Tailwind CSS v4** - Latest features, performance focus
- **shadcn/ui** - Component library

### Form & Validation

- **React Hook Form** - Uncontrolled forms
- **Zod** - Schema validation

### Data & State

- **Axios** - API layer
- **TanStack Query** - Server state management
- **Zustand** - Global UI state

---

## 📁 CẤU TRÚC THƯ MỤC

```
teaching-materials/
├── README.md                          # File này
├── reactjs-curriculum.md              # Giáo trình chi tiết đầy đủ
└── lesson-plans/                      # Kế hoạch bài giảng từng buổi
    ├── INDEX.md                       # Tổng quan các buổi học
    ├── session-00-react-foundation.md # Buổi 0: React Foundation & Developer Mindset
    ├── session-01-setup-ts.md         # Buổi 1: Setup + Component Thinking + TS
    ├── session-02-props-state.md      # Buổi 2: Props, List, State
    ├── session-03-router-layout.md    # Buổi 3: Router v6 + Layout Pattern
    ├── session-04-protected-useEffect.md # Buổi 4: Protected Routes + useEffect
    ├── session-05-zustand-auth.md     # Buổi 5: Zustand Auth Store
    ├── session-06-ui-practical.md     # Buổi 6: Tailwind v4 + shadcn/ui
    ├── session-07-axios-interceptors.md # Buổi 7: Axios Layer + Interceptors
    ├── session-08-forms-rhf.md        # Buổi 8: Forms (RHF + Zod)
    ├── session-09-react-query.md      # Buổi 9: React Query (Fetch)
    ├── session-10-mutations.md        # Buổi 10: Mutations + Invalidation
    ├── session-11-project-setup.md    # Buổi 11: Project Sprint Setup
    ├── session-12-profile-flows.md    # Buổi 12: Profile Flows
    ├── session-13-media-refresh.md    # Buổi 13: Media Upload + Refresh Token
    ├── session-14-ui-polish.md        # Buổi 14: UI Polish & UX States [SPLIT]
    ├── session-15-debug-workshop.md   # Buổi 15: Debug Workshop [SPLIT]
    ├── session-16-integration-testing.md # Buổi 16: Integration Testing [RENUMBERED]
    └── session-17-deployment.md       # Buổi 17: Deployment & Review [RENUMBERED]
```

---

## 📖 CÁCH SỬ DỤNG TÀI LIỆU

### Cho Giảng viên

1. **Đọc trước:** [`reactjs-curriculum.md`](./reactjs-curriculum.md) - Nắm toàn bộ big picture
2. **Chuẩn bị buổi học:** Đọc lesson plan tương ứng trong `lesson-plans/`
3. **Follow structure:**
   - 🎬 **Opening Script** - Mở đầu buổi học (5 phút)
   - 🎯 **Mục tiêu** - Learning outcomes rõ ràng
   - 🧠 **Mental Model** - Tư duy cốt lõi
   - 📚 **Live Coding** - Demo thực hành
   - 🧪 **Checkpoint** - Kiểm tra đầu ra
   - 🚨 **Red Flags** - Lỗi thường gặp cần tránh

### Cho Mentor

- Sử dụng **Checkpoint** để đánh giá tiến độ học viên
- Chú ý **Red Flags** - những lỗi học viên hay mắc phải
- Áp dụng **Pain-Driven Development** - Cho thấy khổ trước khi giải cứu

---

## 🗺️ LỘ TRÌNH 8 TUẦN

### Tuần 1-2: Foundation (4 buổi)

- React Core, TypeScript, Component Thinking
- Props, State, List Rendering
- Router v6, Layout Pattern, Protected Routes

### Tuần 3-4: State & Forms (4 buổi)

- Zustand Auth Store
- Tailwind v4 + shadcn/ui
- Axios Layer + Interceptors
- React Hook Form + Zod

### Tuần 5: Server State (2 buổi)

- TanStack Query (useQuery)
- Mutations + Invalidation
- Auth Flow hoàn chỉnh

### Tuần 6-8: Project Sprint (7 buổi)

- Build Shopping Cart Frontend
- Profile Management
- Media Upload
- UI Polish & UX States
- Debug Workshop
- Integration Testing
- Deployment

---

## 🎓 TRIẾT LÝ ĐÀO TẠO

### Pain-Driven Development

> "Code thủ công cho thấy khổ (useState/useEffect) → Dùng thư viện giải cứu (RHF/Query)"

### Production-First Mindset

- ✅ Luôn có Loading/Error/Empty states
- ✅ Error normalization từ đầu
- ✅ File structure rõ ràng
- ✅ TypeScript đúng cách

### Measurable Outcomes

Mỗi buổi có checkpoint đo được:

- Build todo app trong 1 giờ
- Implement login/register trong 2 giờ
- Build CRUD page với React Query trong 3 giờ

---

## 🔗 BACKEND TARGET

Backend sử dụng: `learnNodeJS/ch04-shoppingCardProject/shoppingCardBE`

### API Endpoints (Core Only)

**Auth:**

- `POST /users/register` - Đăng ký
- `POST /users/login` - Đăng nhập
- `POST /users/logout` - Đăng xuất
- `POST /users/refresh-token` - Refresh token

**User:**

- `POST /users/me` - Lấy thông tin user
- `PATCH /users/me` - Cập nhật profile
- `PUT /users/change_password` - Đổi mật khẩu

**Media:**

- `POST /medias/upload-image` - Upload ảnh (1 file)
- `GET /static/image/:filename` - Serve ảnh

---

## 📊 STATE BOUNDARY RULES

| Loại dữ liệu        | Công cụ             | Ví dụ                                     |
| :------------------ | :------------------ | :---------------------------------------- |
| **Server State**    | **React Query**     | `user`, `products`, `cart`                |
| **Global UI State** | **Zustand**         | `auth_tokens`, `theme`, `sidebar_open`    |
| **Local UI State**  | **useState**        | `modal_open`, `input_value`, `is_loading` |
| **Form State**      | **React Hook Form** | `login_form`, `register_form`             |

> ❌ **Rule:** Không nhét Server Data vào Zustand. Không dùng useEffect để fetch data.

---

## 🚀 QUICK START

### Cho Giảng viên mới

1. Clone repository này
2. Đọc [`reactjs-curriculum.md`](./reactjs-curriculum.md) để hiểu toàn bộ khóa học
3. Đọc [`lesson-plans/INDEX.md`](./lesson-plans/INDEX.md) để xem tổng quan các buổi
4. Chuẩn bị buổi đầu tiên: [`session-00-react-foundation.md`](./lesson-plans/session-00-react-foundation.md)

### Chuẩn bị môi trường

Học viên cần có sẵn:

- Node.js 18+
- VS Code + Extensions (ESLint, Prettier, Tailwind CSS IntelliSense)
- Git
- Backend `shoppingCardBE` đã chạy được

---

## 📝 CHECKLIST GIẢNG VIÊN

Trước mỗi buổi học:

- [ ] Đọc lesson plan tương ứng
- [ ] Chạy thử code demo
- [ ] Chuẩn bị Opening Script
- [ ] Review Red Flags để nhắc học viên

Sau mỗi buổi học:

- [ ] Kiểm tra Checkpoint của học viên
- [ ] Note lại câu hỏi thường gặp
- [ ] Cập nhật tài liệu nếu cần

---

## 🤝 ĐÓNG GÓP

Nếu bạn là Giảng viên/Mentor và muốn cải thiện tài liệu:

1. Fork repository
2. Tạo branch mới: `git checkout -b feature/improve-session-X`
3. Commit changes: `git commit -m 'Improve session X explanation'`
4. Push: `git push origin feature/improve-session-X`
5. Tạo Pull Request

---

## 📞 HỖ TRỢ

Nếu có thắc mắc về tài liệu hoặc cách sử dụng:

- Tạo Issue trong repository
- Liên hệ team Pied để được hỗ trợ

---

## 📄 LICENSE

Tài liệu này thuộc bản quyền của **Pied Team** - Chỉ dành cho mục đích đào tạo nội bộ.

---

## 🎯 TÓM TẮT

Đây là bộ tài liệu đào tạo ReactJS **production-ready** trong 2 tháng, tập trung vào:

- ✅ API handling (core skill)
- ✅ React Query (server state)
- ✅ Form validation (RHF + Zod)
- ✅ Auth flow hoàn chỉnh
- ✅ Production patterns

**Bắt đầu ngay:** Đọc [`reactjs-curriculum.md`](./reactjs-curriculum.md) 🚀
