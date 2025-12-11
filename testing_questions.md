# Tổng hợp kiến thức Frontend (HTML, CSS, JS, API)

## I. HTML – CSS 

### A. Lý thuyết (7 câu)

**1. CSS specificity là gì?**
- **Đáp án:** Mức độ ưu tiên của selector khi áp dụng CSS.
- **Thứ tự:** `!important` > inline style > id > class/attribute/pseudo-class > tag.

**2. Box model gồm những gì?**
- **Đáp án:** Content → Padding → Border → Margin.

**3. display: inline-block là gì?**
- **Đáp án:** Element hiển thị inline nhưng có thể set width/height.

**4. Flexbox: justify-content khác align-items thế nào?**
- **Đáp án:**
    - `justify-content`: căn theo trục chính (main axis)
    - `align-items`: căn theo trục phụ (cross axis)

**5. Grid khác Flexbox ở điểm gì?**
- **Đáp án:**
    - Flexbox → bố cục một chiều.
    - Grid → bố cục hai chiều (rows + columns).

**6. Thế nào là responsive?**
- **Đáp án:** Giao diện tự thay đổi theo kích thước màn hình (dùng media queries, phần trăm, rem…).

**7. Khi nào dùng position: absolute?**
- **Đáp án:** Khi phần tử cần định vị tương đối với thằng cha có `position: relative`.


### B. Thực hành (5 câu)

**8. Viết CSS tạo layout 3 cột bằng flex, cột giữa rộng gấp đôi.**
```css
.container {
  display: flex;
}
.left, .right {
  flex: 1;
}
.center {
  flex: 2;
}
```

**9. Viết media query khi màn hình < 768px thì .menu ẩn.**
```css
@media(max-width: 768px){
  .menu { display: none; }
}
```

**10. Viết CSS tạo button bo góc, màu xanh, hover tối màu.**
```css
.btn {
  padding: 10px 20px;
  background: #3498db;
  border-radius: 8px;
}
.btn:hover {
  background: #217dbb;
}
```

**11. Tạo 1 grid 4 cột, gap 20px.**
```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}
```

**12. Tạo 1 div nằm chính giữa màn hình cả ngang và dọc bằng flex.**
```css
body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## II. JavaScript 

### A. Lý thuyết (11 câu)

**13. var, let, const khác nhau?**
- `var`: function scope, hoisting, cho phép redeclare
- `let`: block scope
- `const`: block scope, không reassign

**14. Hoisting là gì?**
- **Đáp án:** JS đưa khai báo biến/hàm lên đầu scope.

**15. Event loop là gì?**
- **Đáp án:** Cơ chế xử lý bất đồng bộ: call stack → event queue → render.

**16. Destructuring là gì?**
- **Đáp án:** Rút trích giá trị từ array/object.

**17. Spread operator (…) để làm gì?**
- **Đáp án:** Sao chép, gộp array/object.

**18. Deep copy vs shallow copy?**
- `shallow copy`: chỉ copy cấp 1
- `deep copy`: copy mọi cấp

**19. Optional chaining ?. để làm gì?**
- **Đáp án:** Tránh lỗi truy cập thuộc tính của undefined.

**20. async/await là gì?**
- **Đáp án:** Cú pháp wrapper cho promise, viết code bất đồng bộ dễ đọc hơn.

**21. Ternary operator là gì?**
- **Đáp án:** `condition ? true : false`.

**22. Callback hell là gì?**
- **Đáp án:** callback lồng nhau quá nhiều → khó đọc.

**23. Tại sao nên dùng const cho hầu hết biến?**
- **Đáp án:** An toàn, hạn chế bug do reassign.

### B. Thực hành (4 câu)

**24. Viết function loại bỏ phần tử trùng trong array.**
```javascript
const unique = arr => [...new Set(arr)];
```

**25. Viết function deep copy object.**
```javascript
const clone = obj => JSON.parse(JSON.stringify(obj));
```

**26. Viết hàm đảo ngược chuỗi.**
```javascript
const reverse = str => str.split('').reverse().join('');
```

**27. Viết đoạn code tạo nút bấm, click alert “hello”.**
```javascript
document.querySelector('#btn').onclick = () => alert('hello');
```

## III. API – Xử lý bất đồng bộ 

### A. Lý thuyết (7 câu)

**28. HTTP methods GET/POST/PUT/PATCH/DELETE khác nhau?**
- **Đáp án:**
    - `GET`: lấy dữ liệu
    - `POST`: tạo
    - `PUT`: cập nhật toàn bộ
    - `PATCH`: cập nhật 1 phần
    - `DELETE`: xoá

**29. Ý nghĩa các status code 200, 201, 400, 401, 403, 404, 500.**
- **Đáp án:**
    - `200`: OK
    - `201`: Created
    - `400`: Bad Request
    - `401`: Unauthorized
    - `403`: Forbidden
    - `404`: Not Found
    - `500`: Server Error

**30. CORS là gì?**
- **Đáp án:** Browser chặn request cross-domain vì lý do bảo mật.

**31. JWT hoạt động thế nào?**
- **Đáp án:** Server tạo token → FE lưu → gửi trong header mỗi request.

**32. axios khác fetch ở điểm gì?**
- `axios`: auto parse JSON, dễ config interceptor.

**33. Khi nào dùng try...catch?**
- **Đáp án:** Khi call API hoặc code có khả năng throw error.

**34. API pagination là gì?**
- **Đáp án:** Lấy dữ liệu theo trang, tiết kiệm tải.

### B. Thực hành (3 câu)

**35. Viết fetch API(mockapi.io) lấy danh sách user.**
```javascript
const users = await fetch('/api/users').then(r=>r.json());
```

**36. Viết axios GET.**
```javascript
const res = await axios.get('/api/users');
```

**37. Viết code POST gửi body JSON.**
```javascript
fetch('/api/create',{
  method:'POST',
  headers:{'Content-Type':'application/json'},
  body: JSON.stringify({name:'Minh'})
});
```
