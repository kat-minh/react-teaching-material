# LESSON PLAN: SESSION 02 - PROPS, LIST, AND STATE

## 1️⃣ SESSION OVERVIEW
- **Title:** Master the Core: Props, List Rendering, and State Management
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will understand how React efficiently updates the UI, how to render dynamic lists with proper keys, and how to manage component state without creating "Derived State" bugs.
- **Outcome:** A functional Product List page with search/filter functionality and a "Favorite" toggle, all working correctly with React's rendering model.

## 2️⃣ INSTRUCTOR OPENING SCRIPT
_"Chào các bạn. Buổi trước chúng ta đã dựng được cái khung nhà. Hôm nay, chúng ta sẽ thổi hồn vào nó bằng cách làm cho giao diện 'biết cử động'._

_React không mạnh ở chỗ viết HTML trong JS, mà mạnh ở chỗ **UI tự động thay đổi khi Dữ liệu thay đổi**. Các bạn sẽ không bao giờ phải viết lệnh kiểu 'nếu click vào đây thì tìm thẻ h1 rồi đổi màu nó sang đỏ' nữa. Trong React, bạn chỉ cần nói: 'Nếu State là red thì render màu đỏ'._

_Hôm nay là buổi quan trọng nhất để các bạn hiểu cái máy React nó chạy bên dưới như thế nào (Rendering Model). Nếu các bạn nắm chắc buổi này, các bạn sẽ né được 90% 'bug ma' mà các developer tự học thường mắc phải: app bị lag, infinite loop, hoặc bấm nút mà giao diện không đổi."_

> **🔥 WHY THIS SESSION EXISTS?**
> _"React không chỉ là giao diện tĩnh, mà là sự phản hồi của dữ liệu (State). Nếu không hiểu cách React render, bạn sẽ tạo ra những lỗi cực kỳ khó debug khi dự án lớn dần. Đây là lúc chúng ta học cách code 'nhàn' nhưng hiệu quả cao nhất."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Props vs State
- **Props (Properties):** Là quà từ 'cha' gửi cho 'con'. Con chỉ được dùng, không được đổi (Read-only).
- **State:** Là túi tiền riêng của component. Component có quyền tự tiêu, tự đổi, và khi đổi thì component đó sẽ 'render lại'.

### ⚡ React Rendering Model (Mini Foundation - 15 mins)
- **Virtual DOM:** React không cập nhật thẳng lên màn hình ngay. Nó tạo ra một bản copy 'nhẹ' (Virtual DOM), so sánh sự khác biệt (Diffing), rồi mới 'vá' chỗ khác nhau lên màn hình thật (Reconciliation).
- **Khi nào một Component render lại? (3 triggers):**
    1. **State thay đổi:** Khi bạn gọi `setCount(...)`.
    2. **Props thay đổi:** Khi cha truyền data mới xuống.
    3. **Cha render lại:** Mặc định, cha render thì toàn bộ con cháu render theo (trừ khi dùng memo).

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: MOCK DATA & LIST RENDERING (30 mins)

#### Step 1: Chuẩn bị Mock Data
Open `src/pages/HomePage.tsx`:
_"Ta sẽ giả định có dữ liệu từ Backend gửi về."_

```tsx
// Define type cho Product
interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
}

const MOCK_PRODUCTS: Product[] = [
  { id: 'p1', name: 'Mechanical Keyboard', price: 99, category: 'Gear' },
  { id: 'p2', name: 'Gaming Mouse', price: 49, category: 'Gear' },
  { id: 'p3', name: 'UltraWide Monitor', price: 399, category: 'Screen' },
];
```

#### Step 2: Render List (The .map() way)
_"Trong React, ta không dùng for-loop. Ta dùng `.map()` để biến Array data thành Array JSX."_

```tsx
export default function HomePage() {
  return (
    <div className="p-10 bg-gray-50 min-h-screen">
      <h1 className="text-2xl font-bold mb-6 underline decoration-blue-500">
        Product Explorer
      </h1>
      
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        {MOCK_PRODUCTS.map((product) => (
          <div key={product.id} className="bg-white p-4 rounded-xl shadow-sm border hover:shadow-md transition">
            <h3 className="font-bold text-lg">{product.name}</h3>
            <p className="text-gray-500">${product.price}</p>
            <span className="text-xs bg-gray-100 px-2 py-1 rounded mt-2 inline-block">
              {product.category}
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

> **📌 RULE OF THUMB: THE KEY PROP**
> _"Bắt buộc phải có `key`. React dùng `key` để biết card nào cần update, card nào giữ nguyên. Cấm dùng `index` làm key nếu danh sách có thêm/xóa/sắp xếp."_

---

### PHASE 2: INTERACTIVITY WITH STATE (45 mins)

#### Step 1: Search State
_"Bây giờ ta muốn gõ ô tìm kiếm thì danh sách cập nhật theo."_

```tsx
import { useState } from 'react';

export default function HomePage() {
  const [searchTerm, setSearchTerm] = useState<string>("");

  return (
    <div className="max-w-4xl mx-auto p-10">
       {/* Search Input */}
       <input 
          type="text" 
          placeholder="Search products..." 
          className="w-full border p-3 rounded-lg mb-8 shadow-inner focus:ring-2 focus:ring-blue-500 outline-none"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
       />
       {/* List code... */}
    </div>
  )
}
```

#### Step 2: Derived State (The Pro Way)
_"Làm sao để lọc list? Nhiều người sẽ tạo thêm `const [filteredList, setFilteredList] = useState()`. **ĐỪNG LÀM VẬY!**"_

```tsx
export default function HomePage() {
  const [searchTerm, setSearchTerm] = useState("");

  // DAY LÀ DỮ LIỆU TÍNH TOÁN (Derived State)
  // Không cần tạo state mới, cứ tính trực tiếp trong component body.
  const filteredProducts = MOCK_PRODUCTS.filter(p => 
    p.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    // Render filteredProducts thay vì MOCK_PRODUCTS...
  )
}
```

> **❗ Rule of Thumb (Production):**
> Hạn chế 'Derived State'. Nếu một giá trị có thể tính toán được từ State hiện có, đừng tạo State mới cho nó. Điều này giúp tránh bug không đồng bộ dữ liệu.

#### Step 3: Local State cho Từng Item
_"Mỗi sản phẩm sẽ có nút 'Yêu thích' riêng."_

Tách component `ProductCard.tsx`:

```tsx
interface ProductCardProps {
  product: Product;
}

export default function ProductCard({ product }: ProductCardProps) {
  const [isFavorite, setIsFavorite] = useState(false);

  return (
    <div className="bg-white p-4 rounded-xl shadow border">
      <div className="flex justify-between items-start">
        <h3 className="font-bold">{product.name}</h3>
        <button 
          onClick={() => setIsFavorite(!isFavorite)}
          className={`text-2xl ${isFavorite ? 'text-red-500' : 'text-gray-300'}`}
        >
          {isFavorite ? '❤️' : '🤍'}
        </button>
      </div>
      <p className="text-blue-600 font-mono">${product.price}</p>
    </div>
  );
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Mutate State Directly:**
    *   *Code:* `products.push(newProduct);` rồi `setProducts(products);`.
    *   *Why:* React so sánh địa chỉ ô nhớ. Array cũ và mới cùng địa chỉ -> React tưởng không có gì thay đổi -> Không render.
    *   *Fix:* Luôn dùng spread operator: `setProducts([...products, newProduct])`.
2.  **Quên Key Warning:**
    *   *Check:* Mở Console tab trong Chrome DevTools. Nếu thấy chữ đỏ "Each child in a list should have a unique 'key' prop" thì là do thiếu key.
3.  **Gõ Input bị Reset:**
    *   *Cause:* Thường do định nghĩa Component B bên trong Body của Component A.
    *   *Fix:* Luôn định nghĩa component độc lập bên ngoài.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** *"Khi tôi gõ vào ô Search, tại sao component HomePage lại render lại?"*
    *   **A:** Vì ta gọi `setSearchTerm`, khi State của component thay đổi, nó trigger quá trình Re-render.
2.  **Q:** *"Nếu tôi truyền `isFavorite` từ HomePage xuống ProductCard thay vì để trong ProductCard thì sao?"*
    *   **A:** Khi đó `isFavorite` trở thành Props. ProductCard sẽ không tự quản lý được trạng thái 'thích' của nó nữa mà phải phụ thuộc vào cha.

## 7️⃣ IN-CLASS MINI TASK
**Task:** Thêm một bộ lọc theo **Category**.
- Tạo một biến state `const [selectedCategory, setSelectedCategory] = useState("All")`.
- Hiển thị 3 nút: "All", "Gear", "Screen".
- Khi bấm nút, danh sách sản phẩm chỉ hiện đúng loại đó.
- **Tiêu chí:** Phải dùng tư duy Derived State (lọc trên `filteredProducts`).

## 8️⃣ HOMEWORK / EXTENSION TASK
**Yêu cầu:** Tiếp nối project buổi 1 & 2.
1.  Tạo trang **CartPage** đơn giản.
2.  Hiển thị danh sách các món hàng trong giỏ (Array data giả).
3.  Mỗi món hàng có nút `+` và `-` để tăng giảm số lượng.
4.  Tính tổng tiền của cả giỏ hàng (BẮT BUỘC dùng Derived State - không dùng thêm `useState` cho tổng tiền).

## 9️⃣ CHECKPOINT & EVALUATION
- **Quan sát:** Học viên mở tab Console không thấy warning về Key.
- **Quan sát:** Khi gõ Search, UI cập nhật mượt mà, không bị mất focus khỏi ô input.
- **Kiểm tra kiến thức:** Hỏi học viên "Virtual DOM là gì?" -> Phải trả lời được ý: "Là bản nháp để React tính toán trước khi update website thật".

## 🔟 TEACHING NOTES
- **Time Management:** Phần Re-render Model (Virtual DOM) rất dễ bị sa đà vào lý thuyết. Chỉ dành tối đa 15 phút, dùng hình vẽ ẩn dụ.
- **Emphasize:** Nhấn mạnh việc cấm dùng `index` làm key. Cho học viên thấy nếu sắp xếp lại list mà dùng index, React sẽ render sai dữ liệu cũ vào component mới.
- **Red Flag:** Nếu học viên viết logic `if (category === 'Gear') setFilteredProducts(...)` trong handler -> Hãy dừng lại và giải thích lại về Derived State.
