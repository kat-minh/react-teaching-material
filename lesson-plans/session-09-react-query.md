# LESSON PLAN: SESSION 09 - TANSTACK QUERY (DATA FETCHING)

## 1️⃣ SESSION OVERVIEW
- **Title:** The Data Wizard: Mastering Server State with TanStack Query
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will abandon `useEffect` for data fetching and adopt TanStack Query (React Query) to manage Server State (Loading, Error, Caching) effortlessly.
- **Outcome:** Retrieve User Profile (`/me`) using `useQuery` with automatic caching and standardized loading/error UI states.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Hôm nay là ngày chúng ta 'đốt bỏ' cuốn sách giáo khoa React cũ. 
Trong sách, họ dạy bạn Fetch Data bằng `useEffect` + `useState`.

Nhưng thực tế:
- Nếu mạng chậm?
- Nếu user tab qua tab lại?
- Nếu user F5 liên tục?
Code cũ sẽ vỡ trận.

Hôm nay tôi giới thiệu **TanStack Query**. Nó không chỉ gọi API. Nó là 'người quản gia' thông minh: Tự biết khi nào dữ liệu cũ để đi lấy mới, tự biết cache để App nhanh như gió. Trong 99% trường hợp GET request, ta không dùng useEffect nữa. useEffect chỉ dùng cho side-effect không liên quan data fetching."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"Gọi API thì dễ. Nhưng quản lý Cache, Loading, Error, Retry mới khó. React Query làm hết việc khó đó cho bạn. Junior dùng useEffect, Senior dùng Query."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 💾 Client State vs Server State
- **Client State (Zustand/useState):** Dữ liệu của App (Dropdown open/close, Dark mode). Ta kiểm soát 100%.
- **Server State (React Query):** Dữ liệu mượn từ Server (List sản phẩm, Profile). Nó có thể cũ, có thể lỗi, thời gian chờ. Ta **không** sở hữu nó, ta chỉ **cache** nó.

### 🔄 Stale-while-revalidate
- Nguyên lý: "Hiển thị cái cũ (Stale) trong lúc ngầm đi lấy cái mới (Revalidate)". Giúp User luôn thấy dữ liệu ngay lập tức.

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: PAIN-DRIVEN DEMO (15 mins)
_"Trước khi sung sướng, ta phải khổ trước để thấy giá trị."_

Update `src/pages/MePage.tsx` (Viết thử rồi xóa):
_"Hãy thử viết code fetch User Profile bằng tay."_

```tsx
// ❌ THE PAINFUL WAY (Code mẫu - đừng copy vào dự án)
const [user, setUser] = useState(null);
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setIsLoading(true);
  usersApi.getMe()
    .then(res => setUser(res.result))
    .catch(err => setError(err))
    .finally(() => setIsLoading(false));
}, []); // ⚠️ Nếu user quay lại trang này, nó lại fetch lại từ đầu -> Chậm!
```
-> **Conclusion:** Code dài, thủ công, không cache, không retry. -> **DELETE IT.**

---

### PHASE 2: SETUP TANSTACK QUERY (30 mins)

#### Step 1: Install Library
```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

#### Step 2: Configure Global Provider
Update `src/main.tsx` (hoặc `App.tsx`):
_"Giống như Context/Redux, ta cần một Provider bọc ngoài cùng."_

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// Setup Client với config mặc định an toàn cho Newbie
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false, // Tắt tự fetch khi switch tab (để dễ debug lúc học)
      retry: 1, // Thử lại 1 lần nếu fail (thay vì 3)
    },
  },
});

// > **NOTE:** Production có thể bật lại `refetchOnWindowFocus` để data luôn tươi mới. Nhưng lúc dev thì nên tắt để log đỡ nhảy loạn xạ.

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
       {/* App Components */}
       <App />
       {/* Công cụ debug thần thánh */}
       <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
);
```

---

### PHASE 3: QUERY IMPLEMENTATION (60 mins)

#### Step 1: Create Custom Hook
Create `src/hooks/useUser.ts`:
_"Ta luôn bọc react-query trong custom hook để tái sử dụng."_

```tsx
import { useQuery } from '@tanstack/react-query';
import { usersApi } from '@/lib/api/users.api';

export const useUser = () => {
  return useQuery({
    queryKey: ['me'], 
    // 🧠 Query Key Concept: Đây là ID của Cache. Cùng Key = Chung Cache.
    
    queryFn: async () => {
       const response = await usersApi.getMe();
       return response.data.result; 
       // 🧠 Cache Shape: Query chỉ cache cái mình return (user object), không cache cả cục response axios.
    },
    // staleTime: 1000 * 60 * 5, // Data coi là mới trong 5 phút (Optional lesson)
  });
};
```

#### Step 2: Use in Component
Update `src/pages/MePage.tsx`:
_"Xem code ngắn đi bao nhiêu so với cách cũ!"_

```tsx
import { useUser } from '@/hooks/useUser';
import { Button } from "@/components/ui/button";
import { LoadingState, ErrorState } from '@/components/ui/StatusStates'; // Tận dụng code bài 4

export default function MePage() {
  const { data: user, isLoading, isError, refetch } = useUser();

  if (isLoading) return <LoadingState />;
  if (isError) return <ErrorState message="Không tải được thông tin cá nhân" />;

  return (
    <div className="p-10">
      <h1 className="text-3xl font-bold">Profile</h1>
      <pre>{JSON.stringify(user, null, 2)}</pre>
      
      <Button onClick={() => refetch()} className="mt-4">
        Refresh Data
      </Button>
    </div>
  );
}
```

> **📌 INSTRUCTOR NOTE:**
> _"Hãy mở DevTools (bông hoa màu đỏ góc màn hình). Cho học viên thấy trạng thái: `fresh`, `fetching`, `stale`."_

#### Step 3: Connect to Auth Store (Advanced Sync)
_"Khi load được user xong, ta nên cập nhật lại vào Zustand để các chỗ khác dùng mà không cần gọi hook `useUser`."_

Update `src/main.tsx` or create a Synchronization Component.

> **⚠️ BOUNDARY NOTE:**
> _"Session này CHƯA sync Query với Store. Ta chỉ fetch và render trực tiếp từ Query Cache. Việc Sync chỉ cần thiết khi nhiều component (Header, Profile, Settings) cùng cần data và ta muốn quản lý tập trung. Buổi Authentication Flow (Buổi 10) sẽ làm phần này."_

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Quên return trong `queryFn`**:
    *   *Symptom:* Data luôn `undefined` dù API status 200.
    *   *Fix:* Arrow function có `{}` thì phải có `return`.
2.  **Key duplicate hoặc quá chung chung**:
    *   *Issue:* Dùng chung key `['user']` cho cả `getMe` và `getUserById`.
    *   *Fix:* Key phải cụ thể: `['user', 'me']` hoặc `['user', { id: 1 }]`.
3.  **Lạm dụng `useEffect` để set state từ data**:
    *   *Code:* `useEffect(() => { if (data) setState(data) }, [data])`.
    *   *Advice:* **Anti-pattern lớn nhất!** Hãy dùng trực tiếp `data` trong JSX. Nếu cần transform, dùng option `select` của useQuery.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Khi nào dùng `useEffect` để fetch data?"*
    *   **A:** Gần như không bao giờ trong dự án React hiện đại (trừ khi viết lại thư viện fetching).
2.  **Q:** *"StaleTime vs GcTime (CacheTime) khác nhau gì?"*
    *   **A:** `staleTime`: Bao lâu thì data được coi là cũ (để tự fetch lại). `gcTime`: Bao lâu thì data bị xóa khỏi bộ nhớ nếu không dùng (để tiết kiệm RAM).

## 7️⃣ IN-CLASS MINI TASK
**Task:** DevTools Playground.
- Mở tab Network, chọn Offline.
- Bấm F5 -> Web lỗi trắng.
- Bấm Refetch button -> Hook báo lỗi.
- Bật lại mạng -> Bấm Refocus vào cửa sổ -> Thấy Query tự chạy lại và hiện data.
- **Kết luận:** React Query tự xử lý reconnect và window focus.

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Product List Query.
1.  Tạo Mock API endpoint `/products`.
2.  Viết `useProductsList` hook.
3.  Hiển thị danh sách sản phẩm ra `HomePage`.
4.  Thử cấu hình `staleTime: 10 giây`. Trong 10 giây đó, F5 hoặc switch tab sẽ không thấy network request nào gửi đi (Cache hit).

## 9️⃣ CHECKPOINT & EVALUATION
- **Signal:** Mở React Query DevTools thấy query `['me']` màu xanh lá (fresh) hoặc vàng (stale).
- **Signal:** Code trong component sạch, không còn `useState`, `useEffect`.
- **Behavior:** Bấm refetch button, thấy spin loading xoay nhẹ rồi hiện data mới.

## 🔟 TEACHING NOTES
- **Slow Down:** Giải thích sự khác biệt giữa `isLoading` (lần đầu không có data) và `isFetching` (có data cũ, đang lấy data mới).
- **Emphasis:** "Server State" là khái niệm khó. Hãy ví dụ: Giống như xem giá chứng khoán. Giá trên màn hình (Client) chỉ là ảnh chụp quá khứ, giá thật (Server) có thể đã đổi. Query giúp ta sync 2 cái đó.
- **Red Flag:** Học viên cố gán response vào Redux/Zustand. Nhắc: "React Query chính là Cache Store rồi, đừng copy data ra chỗ khác trừ khi rất cần thiết".
