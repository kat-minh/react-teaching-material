# LESSON PLAN: SESSION 02 - PROPS, LIST, AND STATE

## 1️⃣ SESSION OVERVIEW

- **Title:** Master the Core: Props, List Rendering, and State Management
- **Duration:** 120 minutes (2 hours)
- **Goal:** Students will understand how React efficiently updates the UI, how to render dynamic lists with proper keys, and how to manage component state without creating "Derived State" bugs.
- **Outcome:** A functional Product List page with search/filter functionality and a "Favorite" toggle, all working correctly with React's rendering model.

## 2️⃣ INSTRUCTOR OPENING SCRIPT

_"Chào các bạn. Buổi trước chúng ta đã dựng được cái khung nhà và làm quen sơ qua với `useState`. Hôm nay, chúng ta sẽ thực sự làm chủ nó (Mastering State) và thổi hồn vào giao diện bằng cách làm cho nó 'biết cử động'._

_React không mạnh ở chỗ viết HTML trong JS, mà mạnh ở chỗ **UI tự động thay đổi khi Dữ liệu thay đổi**. Các bạn sẽ không bao giờ phải viết lệnh kiểu 'nếu click vào đây thì tìm thẻ h1 rồi đổi màu nó sang đỏ' nữa. Trong React, bạn chỉ cần nói: 'Nếu State là red thì render màu đỏ'._

_Hôm nay là buổi quan trọng nhất để các bạn hiểu cái máy React nó chạy bên dưới như thế nào (Rendering Model). Nếu các bạn nắm chắc buổi này, các bạn sẽ né được 90% 'bug ma' mà các developer tự học thường mắc phải: app bị lag, infinite loop, hoặc bấm nút mà giao diện không đổi."_

> **🔥 WHY THIS SESSION EXISTS?** > _"React không chỉ là giao diện tĩnh, mà là sự phản hồi của dữ liệu (State). Nếu không hiểu cách React render, bạn sẽ tạo ra những lỗi cực kỳ khó debug khi dự án lớn dần. Đây là lúc chúng ta học cách code 'nhàn' nhưng hiệu quả cao nhất."_

## 3️⃣ MENTAL MODEL & CONCEPTUAL FOUNDATION

### 🏗️ Props vs State

- **Props (Properties):** Là quà từ 'cha' gửi cho 'con'. Con chỉ được dùng, không được đổi (Read-only).
- **State:** Là túi tiền riêng của component. Component có quyền tự tiêu, tự đổi, và khi đổi thì component đó sẽ 'render lại'.

### ⚡ React Rendering Model (Mini Foundation - 15 mins)

> **Instructor Script (QUAN TRỌNG - 15 PHÚT):** > _"⏱️ Phần này là nền tảng để hiểu React._
>
> _Câu hỏi: Khi nào một component render lại?_
>
> _Nếu không biết câu trả lời chính xác, các bạn sẽ gặp bug 'ma': Bấm nút mà UI không đổi, hoặc UI đổi nhưng data cũ."_

#### Khi nào Component Re-render? (3 Triggers)

```tsx
// TRIGGER 1: State thay đổi
function Counter() {
  const [count, setCount] = useState(0);
  console.log("Counter render"); // Log mỗi khi render

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
// Click button → setCount → Component render lại
```

```tsx
// TRIGGER 2: Props thay đổi
function Greeting({ name }: { name: string }) {
  console.log("Greeting render");
  return <h1>Hello {name}</h1>;
}

function Parent() {
  const [userName, setUserName] = useState("John");

  return <Greeting name={userName} />; // userName thay đổi → Greeting render lại
}
```

```tsx
// TRIGGER 3: Cha render → Con render (Mặc định!)
function Parent() {
  const [count, setCount] = useState(0);
  console.log("Parent render");

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child name="John" /> {/* Props không đổi nhưng vẫn render lại! */}
    </div>
  );
}

function Child({ name }: { name: string }) {
  console.log("Child render"); // Chạy MỖI KHI Parent render
  return <p>Hello {name}</p>;
}

// ⚠️ Quan trọng: Parent re-render → TẤT CẢ Child re-render (mặc định)
// Optimization: Dùng React.memo để ngăn Child re-render khi props không đổi (học sau)
```

#### Parent/Child Re-render Flow (CRITICAL DEMO)

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  console.log("👨 Parent render");

  return (
    <div className="p-4 border-2 border-blue-500">
      <h2>Parent Component</h2>
      <button
        onClick={() => setCount(count + 1)}
        className="px-4 py-2 bg-blue-500 text-white"
      >
        Parent Count: {count}
      </button>

      <Child1 />
      <Child2 />
    </div>
  );
}

function Child1() {
  console.log("👦 Child1 render");
  return <div className="ml-4 border border-green-500 p-2">Child 1</div>;
}

function Child2() {
  console.log("👧 Child2 render");
  return <div className="ml-4 border border-pink-500 p-2">Child 2</div>;
}

// 🔍 Kết quả khi click button:
// Console log:
// 👨 Parent render
// 👦 Child1 render
// 👧 Child2 render

// ❗ Lưu ý: Child1 và Child2 KHÔNG nhận props nào từ Parent
// Nhưng vẫn re-render vì Parent re-render!
```

> **💡 Rule of Thumb:**
>
> - Parent re-render → **TẤT CẢ** children re-render (mặc định)
> - Đây không phải bug, đây là thiết kế của React (safety first)
> - Performance optimization: Dùng `React.memo` (học sau - Session 6 hoặc nâng cao)

#### Virtual DOM Diffing (Concept Only - 5 mins)

> **Instructor Script:** > _"React không update DOM thật ngay lập tức._
>
> _React tạo ra 1 bản 'nháp' (Virtual DOM), so sánh với bản cũ, rồi chỉ update phần khác biệt lên DOM thật."_

```
┌─────────────────────┐
│   STATE CHANGES     │ → setCount(5)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  REACT RE-RENDERS   │ → Tạo Virtual DOM mới
│   (Virtual DOM)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  DIFFING ALGORITHM  │ → So sánh Virtual DOM cũ vs mới
│  (Reconciliation)   │   Tìm sự khác biệt
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   UPDATE REAL DOM   │ → Chỉ update phần khác biệt
│  (Minimal changes)  │   VD: Chỉ đổi text "4" → "5"
└─────────────────────┘
```

> **📌 Ví dụ cụ thể:**
>
> ```tsx
> // Trước: <button>Count: 4</button>
> // Sau:  <button>Count: 5</button>
>
> // React KHÔNG thay thế cả button
> // Chỉ update text node "4" → "5"
> ```

> **❗ Rule of Thumb (Production):**
>
> - Hạn chế **derived state** - chỉ dùng state cho data thay đổi theo thời gian
> - Computed values (VD: `filteredList`) → Tính trực tiếp trong component body, KHÔNG tạo state mới
> - Dependency array trong useEffect phải đầy đủ (học chi tiết Session 4)

## 4️⃣ LIVE CODING – STEP BY STEP

### PHASE 1: JSX DEEP DIVE (20 mins - ĐẦU GIỜ)

> **Instructor Script:** > _"Trước khi làm list sản phẩm, ta cần hiểu JSX sâu hơn. Không phải chỉ là 'HTML trong JS'._
>
> _JSX có 3 patterns quan trọng mà các bạn sẽ dùng mỗi ngày: Children, Conditional Rendering, và Fragment."_

#### Pattern 1: Children Prop (5 mins)

_"Children không phải magic, nó chỉ là một props đặc biệt."_

```tsx
// ❌ Nhiều người tưởng phải truyền children như props thường
<Card children={<p>Content</p>} />

// ✅ ĐÚNG - Children là nội dung bên trong thẻ
<Card>
  <p>Content</p>
  <button>Click</button>
</Card>

// Component nhận children:
interface CardProps {
  title: string;
  children: React.ReactNode; // Type cho children
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children} {/* Render bất kỳ nội dung gì được truyền vào */}
    </div>
  );
}

// Sử dụng:
<Card title="Product Details">
  <p>Description goes here</p>
  <button>Add to Cart</button>
</Card>
```

> **💡 Rule of Thumb:** Dùng `children` khi component là "container" - không biết trước nội dung bên trong.

#### Pattern 2: Conditional Rendering (7 mins)

_"React không có v-if như Vue. Ta dùng JavaScript thuần."_

```tsx
function ProductCard({ product, isLoggedIn }: Props) {
  return (
    <div className="card">
      <h3>{product.name}</h3>

      {/* Pattern 1: AND operator - hiển thị hoặc không */}
      {isLoggedIn && <button>Add to Cart</button>}

      {/* Pattern 2: Ternary - A hoặc B */}
      {product.stock > 0 ? (
        <span className="text-green-500">In Stock</span>
      ) : (
        <span className="text-red-500">Out of Stock</span>
      )}

      {/* Pattern 3: Empty state */}
      {product.reviews.length === 0 && (
        <p className="text-gray-400">No reviews yet</p>
      )}

      {/* ❌ KHÔNG DÙNG if/else trong JSX - syntax error */}
      {/* {if (isLoggedIn) <button>Cart</button>} */}
    </div>
  );
}
```

> **⚠️ Common Mistake:**
>
> ```tsx
> {
>   product.reviews.length && <Reviews />;
> } // ❌ Nếu length = 0, sẽ render số "0" lên UI
> {
>   product.reviews.length > 0 && <Reviews />;
> } // ✅ ĐÚNG
> ```

#### Pattern 3: Fragment (3 mins)

_"Đôi khi ta muốn return nhiều elements mà không muốn thêm thẻ div thừa."_

```tsx
// ❌ SAI - Không thể return nhiều elements cùng level
function UserInfo() {
  return (
    <h1>John Doe</h1>
    <p>Developer</p>
  )
}

// ❌ WORKAROUND XẤU - Thêm div không cần thiết
function UserInfo() {
  return (
    <div> {/* div này chỉ để bọc, làm lộn CSS */}
      <h1>John Doe</h1>
      <p>Developer</p>
    </div>
  )
}

// ✅ ĐÚNG - Dùng Fragment (không tạo DOM node)
function UserInfo() {
  return (
    <>
      <h1>John Doe</h1>
      <p>Developer</p>
    </>
  );
}

// Cú pháp dài (khi cần key prop):
import { Fragment } from 'react';
<Fragment key={item.id}>...</Fragment>
```

> **📌 Khi nào dùng Fragment?**
>
> - Return nhiều elements từ component
> - Map list mà mỗi item là nhiều elements
> - Không muốn CSS bị ảnh hưởng bởi div wrapper

---

### PHASE 2: PROPS FLOW PATTERN (15 mins)

> **Instructor Script:** > _"Pattern này là nền tảng của React architecture: 'Props down, Events up'._
>
> _Dữ liệu chảy từ trên xuống (Parent → Child), nhưng sự kiện bubble từ dưới lên (Child → Parent)."_

#### Parent/Child Communication

_"Sai lầm phổ biến: Đặt state ở Child khi Parent cũng cần biết."_

```tsx
// ❌ SAI - State ở Child, Parent không biết product nào được like
function ProductCard({ product }: Props) {
  const [isLiked, setIsLiked] = useState(false);

  return (
    <button onClick={() => setIsLiked(!isLiked)}>
      {isLiked ? "❤️" : "🤍"}
    </button>
  );
}

// ❗ VẤN ĐỀ: Nếu Parent cần hiển thị "Liked Products Count" thì làm sao?
```

```tsx
// ✅ ĐÚNG - State ở Parent (common ancestor)
function ProductList() {
  const [likedIds, setLikedIds] = useState<string[]>([]);

  // Event handler ở Parent
  const handleLike = (productId: string) => {
    if (likedIds.includes(productId)) {
      setLikedIds(likedIds.filter((id) => id !== productId));
    } else {
      setLikedIds([...likedIds, productId]);
    }
  };

  return (
    <div>
      <p>Liked: {likedIds.length} products</p> {/* Parent có thể hiển thị */}
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product} // Props down
          isLiked={likedIds.includes(product.id)} // Props down
          onLike={handleLike} // Event callback down
        />
      ))}
    </div>
  );
}

// Child: Chỉ nhận props và gọi callback
function ProductCard({ product, isLiked, onLike }: Props) {
  return (
    <button onClick={() => onLike(product.id)}>
      {" "}
      {/* Event up */}
      {isLiked ? "❤️" : "🤍"} {product.name}
    </button>
  );
}
```

> **❗ Rule of Thumb:**
>
> - State phải ở **common ancestor** (component cha gần nhất cần access state đó)
> - Child components phải **stateless** khi có thể
> - Child gọi callbacks, không tự quyết định state

---

### PHASE 3: CONTROLLED VS UNCONTROLLED INPUTS (15 mins)

> **Instructor Script:** > _"Controlled input là pattern quan trọng nhất khi làm form trong React._
>
> _Không hiểu pattern này → Search box sẽ bị bug khó hiểu."_

#### Controlled Input Pattern

_"React kiểm soát value của input."_

```tsx
function SearchBox() {
  const [searchTerm, setSearchTerm] = useState("");

  return (
    <div>
      {/* ✅ CONTROLLED - React quản lý value */}
      <input
        type="text"
        value={searchTerm} // Value từ state
        onChange={(e) => setSearchTerm(e.target.value)} // Update state
        placeholder="Search..."
      />
      <p>You searched: {searchTerm}</p> {/* Real-time feedback */}
    </div>
  );
}

// ⚡ Ưu điểm:
// 1. Validation realtime
// 2. Format input (VD: uppercase tự động)
// 3. Clear button đơn giản: setSearchTerm("")
```

#### Uncontrolled Input (với useRef)

_"DOM giữ value, React chỉ đọc khi cần."_

```tsx
import { useRef } from "react";

function UncontrolledForm() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    // Chỉ đọc value khi submit
    const value = inputRef.current?.value;
    console.log("Submitted:", value);
  };

  return (
    <div>
      {/* ❌ UNCONTROLLED - Không có value prop */}
      <input ref={inputRef} type="text" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}

// ⚡ Khi nào dùng Uncontrolled?
// - Form đơn giản, không cần validation realtime
// - Integrate với thư viện non-React (VD: React Hook Form dùng uncontrolled)
```

#### Controlled vs Uncontrolled Comparison

```tsx
// ❌ COMMON MISTAKE - Controlled input không có onChange
<input value={searchTerm} /> // React warning: "value without onChange"

// ❌ BUG - Controlled nhưng quên update state
<input
  value={searchTerm}
  onChange={(e) => console.log(e.target.value)} // Không gọi setSearchTerm
/>
// Kết quả: Gõ không được, input bị "đóng băng"

// ✅ ĐÚNG - Controlled pattern đầy đủ
<input
  value={searchTerm}
  onChange={(e) => setSearchTerm(e.target.value)}
/>
```

> **📌 Rule of Thumb (Production):**
>
> - **Controlled**: Dùng cho search box, filters, inputs cần validation realtime
> - **Uncontrolled**: Dùng cho form đơn giản hoặc khi integrate với React Hook Form
> - **KHÔNG** mix Controlled và Uncontrolled trong cùng 1 input (value undefined → controlled, hoặc không có value → uncontrolled)

---

### PHASE 4: MOCK DATA & LIST RENDERING (30 mins)

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
  { id: "p1", name: "Mechanical Keyboard", price: 99, category: "Gear" },
  { id: "p2", name: "Gaming Mouse", price: 49, category: "Gear" },
  { id: "p3", name: "UltraWide Monitor", price: 399, category: "Screen" },
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
          <div
            key={product.id}
            className="bg-white p-4 rounded-xl shadow-sm border hover:shadow-md transition"
          >
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

> **📌 RULE OF THUMB: THE KEY PROP** > _"Bắt buộc phải có `key`. React dùng `key` để biết card nào cần update, card nào giữ nguyên. Cấm dùng `index` làm key nếu danh sách có thêm/xóa/sắp xếp."_

---

### PHASE 5: INTERACTIVITY WITH STATE (45 mins)

#### Step 1: Search State

_"Bây giờ ta muốn gõ ô tìm kiếm thì danh sách cập nhật theo."_

```tsx
import { useState } from "react";

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
  );
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
          className={`text-2xl ${
            isFavorite ? "text-red-500" : "text-gray-300"
          }`}
        >
          {isFavorite ? "❤️" : "🤍"}
        </button>
      </div>
      <p className="text-blue-600 font-mono">${product.price}</p>
    </div>
  );
}
```

## 5️⃣ COMMON STUDENT MISTAKES & DEBUGGING

1.  **Mutate State Directly:**
    - _Code:_ `products.push(newProduct);` rồi `setProducts(products);`.
    - _Why:_ React so sánh địa chỉ ô nhớ. Array cũ và mới cùng địa chỉ -> React tưởng không có gì thay đổi -> Không render.
    - _Fix:_ Luôn dùng spread operator: `setProducts([...products, newProduct])`.
2.  **Quên Key Warning:**
    - _Check:_ Mở Console tab trong Chrome DevTools. Nếu thấy chữ đỏ "Each child in a list should have a unique 'key' prop" thì là do thiếu key.
3.  **Gõ Input bị Reset:**
    - _Cause:_ Thường do định nghĩa Component B bên trong Body của Component A.
    - _Fix:_ Luôn định nghĩa component độc lập bên ngoài.

## 6️⃣ INSTRUCTOR QUESTIONS & EXPECTED ANSWERS

1.  **Q:** _"Khi tôi gõ vào ô Search, tại sao component HomePage lại render lại?"_
    - **A:** Vì ta gọi `setSearchTerm`, khi State của component thay đổi, nó trigger quá trình Re-render.
2.  **Q:** _"Nếu tôi truyền `isFavorite` từ HomePage xuống ProductCard thay vì để trong ProductCard thì sao?"_
    - **A:** Khi đó `isFavorite` trở thành Props. ProductCard sẽ không tự quản lý được trạng thái 'thích' của nó nữa mà phải phụ thuộc vào cha.

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

---

## ✅ CHECKLIST TUẦN 1: REACT CORE CONCEPTS

> **Giáo viên bắt buộc hỏi học viên TRƯỚC KHI chuyển sang Tuần 2:**

### JSX & Component Patterns:

- [ ] Viết được component nhận `children` prop
- [ ] Render conditional với `&&` và ternary `? :`
- [ ] Biết khi nào dùng Fragment `<> </>` thay vì `<div>`
- [ ] Hiểu JSX expression `{}` chỉ nhận expression, không nhận if/else

### Props & Events:

- [ ] Truyền props từ Parent xuống Child
- [ ] Truyền callback function từ Parent để Child gọi (Events up)
- [ ] Hiểu "Props down, Events up" pattern
- [ ] Không mutate props trong Child

### State & Re-render:

- [ ] Dùng `useState` đúng (không mutate trực tiếp)
- [ ] Hiểu khi nào component re-render (state/props thay đổi, parent render)
- [ ] Hiểu Parent render → Child cũng render (mặc định)
- [ ] Biết Controlled input (value + onChange) vs Uncontrolled (ref)

### List Rendering:

- [ ] Render list với `.map()` và `key` đúng
- [ ] Hiểu tại sao `key` quan trọng (React diffing)
- [ ] Không dùng index làm key nếu list có thể thay đổi thứ tự

### TypeScript cho React:

- [ ] Định nghĩa Props interface
- [ ] Type cho event handlers (`React.ChangeEvent`, `React.FormEvent`)
- [ ] Type cho `useState` với generics: `useState<User | null>(null)`

### 🚨 Anti-patterns phải tránh:

- ❌ Mutate state trực tiếp: `state.push()` → dùng `setState([...state, newItem])`
- ❌ Đặt state ở Child khi Parent cần biết → state phải ở common ancestor
- ❌ Quên `key` khi render list
- ❌ Dùng `any` type cho props/events
- ❌ Controlled input không có `onChange` handler

### 🎯 Checkpoint Exercise (10 phút):

**Code challenge:** Viết `TodoList` component:

- Input controlled để add todo
- Render list todos với delete button
- Parent giữ state `todos`, Child chỉ nhận props và callbacks
- Có empty state khi `todos.length === 0`

> **Pass criteria:** Code chạy, không warning, follow "Props down Events up" pattern.
