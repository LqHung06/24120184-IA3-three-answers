# ASSISTANT REVIEW

- **Tool & Model**: Gemini 3.8 Flash (High)
- **Date**: 2026-10-31
- **Prompt given**:
```text
Dưới đây là brief.md và ba file a.js, b.js, c.js do một assistant viết từ brief đó.
Hãy review từng file theo brief: liệt kê mọi chỗ không đáp ứng yêu cầu hoặc ràng buộc,
nêu số dòng code, nói test condition nào (normal, slow, error, hang, empty)
hoặc kiểm tra nào làm lộ ra lỗi đó, và cho mỗi file một kết luận keep, fix hoặc reject.
Chỉ đọc code, không chạy.
```

---

## Raw Answer from Assistant

Dưới đây là chi tiết review 3 file a.js, b.js, c.js dựa trên các yêu cầu và ràng buộc trong brief.md.

──────
### 1. File public/versions/a.js

#### Các chỗ không đáp ứng yêu cầu / ràng buộc:
• Dòng 48–52:
```javascript
let total = 0;
products.forEach(async (p) => {
  const stock = await getStock(p.id);
  total += p.price * stock;
});
stockValue.textContent = formatVnd(total);
```
• Vi phạm Yêu cầu 5: “When products are loaded, compute stock value: sum of price × stock over all products, requesting stock in parallel. Write formatted total into #stock-value.”
• Nguyên nhân: Array.prototype.forEach không await các async callback. Dòng 52 stockValue.textContent = formatVnd(total) sẽ thực thi đồng bộ ngay lập tức khi total vẫn bằng 0.
• Khi các lời gọi getStock hoàn thành sau đó (100–600 ms), biến total được cộng dồn nhưng DOM không bao giờ được cập nhật lại giá trị thực tế.
• Ngoài ra, nếu có bất kỳ request getStock nào bị lỗi (như mạng ngắt hay 404), lỗi ném ra từ async callback trong forEach không được khối try...catch ở dòng 42 bao bọc, gây ra Unhandled Promise Rejection trên trình duyệt.

#### Test condition / Kiểm tra làm lộ lỗi:
• normal (hoặc slow): Khi API trả về danh sách sản phẩm bình thường (có từ 1 sản phẩm trở lên), #stock-value luôn hiển thị ngay lập tức là 0 đ thay vì tổng giá trị kho hàng thực tế.

#### Kết luận:
• reject (hoặc fix nếu cho phép sửa đoạn tính toán).
• Lý do: File hỏng ngay ở luồng nghiệp vụ cốt lõi ở điều kiện thông thường (normal), hiển thị sai dữ liệu cho người dùng và làm rò rỉ unhandled promise rejection. (Để sửa cần thay forEach bằng Promise.all).

──────
### 2. File public/versions/b.js

#### Các chỗ không đáp ứng yêu cầu / ràng buộc:
• Dòng 13:
```javascript
const response = await fetch('/api/products', { timeout: 5000 });
```
• Vi phạm Yêu cầu 4: “If request fails (network error, not 2xx, or no answer within 5 seconds), #status says Could not load products. and Retry is shown.”
• Nguyên nhân: API chuẩn của trình duyệt (window.fetch) không hỗ trợ option { timeout: 5000 } trong RequestInit (đây là option của Node.js / thư viện Axios). Trình duyệt sẽ bỏ qua option này. Chuẩn Web API yêu cầu dùng { signal: AbortSignal.timeout(5000) }.
• Dòng 45 & Dòng 51–54:
• Vi phạm Yêu cầu 4 (tính nhất quán của UI khi lỗi):
  • Dòng 45 catalogue.append(...products.map(createCard)) đã gắn các card vào DOM.
  • Nếu lời gọi loadStock ở dòng 48 bị lỗi (network fail hoặc không thành công), code nhảy vào catch (dòng 51–54).
  • Khối catch không gọi dọn dẹp #catalogue. Kết quả: các thẻ sản phẩm vẫn hiển thị trên trang trong khi #status lại báo Could not load products. cùng với nút Retry.
• Dòng 40:
  • catalogue.replaceChildren() xóa sạch danh sách ngay khi bấm tải thay vì giữ danh sách cũ cho đến khi có dữ liệu mới hoặc lỗi, gây chớp nháy trắng trang không cần thiết khi bấm Retry.

#### Test condition / Kiểm tra làm lộ lỗi:
• hang: Khi API /api/products không trả lời (treo hoặc phản hồi > 5s), fetch sẽ treo vô thời hạn và không bao giờ abort ở 5 giây; #status kẹt ở Loading products… mãi mãi và nút Retry không bao giờ xuất hiện.
• error (đối với stock): Nếu endpoint stock bị lỗi sau khi đã render cards, UI rơi vào trạng thái mâu thuẫn (vừa có sản phẩm vừa báo lỗi).

#### Kết luận:
• fix (hoặc reject do dùng sai Web API).
• Lý do: Luồng tính toán stock ở điều kiện normal và empty sử dụng Promise.all và .reduce là chính xác. Lỗi timeout có thể sửa nhanh bằng cách đổi { timeout: 5000 } thành { signal: AbortSignal.timeout(5000) } và thêm catalogue.replaceChildren() vào khối catch.

──────
### 3. File public/versions/c.js

#### Đánh giá theo Brief:
• Yêu cầu 1: Có trạng thái Loading products… khi bắt đầu load (dòng 44, 64).
• Yêu cầu 2: Clone <template>, điền .name, định dạng tiền tệ qua money(p.price) (price.toLocaleString('vi-VN') + ' đ'), gán data.id, đưa vào #catalogue đúng thứ tự (dòng 24–35).
• Yêu cầu 3: Khi render xong thì clear #status (dòng 48: ui.status.textContent = products.length ? '' : 'No products yet.').
• Yêu cầu 4: Dùng chuẩn Web API AbortSignal.timeout(5000) (dòng 12) xử lý đúng timeout 5s, bắt lỗi HTTP và lỗi mạng trong catch, xóa list, hiển thị Could not load products. và mở nút Retry (dòng 54–57). Khi bấm Retry ẩn nút và gọi lại từ bước 1 (dòng 42, 63).
• Yêu cầu 5: Sử dụng Promise.all gọi song song toàn bộ stock (dòng 50), tính tổng và gán vào #stock-value. Khi mảng rỗng thì hiển thị chính xác 0 đ (dòng 53).
• Yêu cầu 6: Toàn bộ dữ liệu đưa vào DOM bằng textContent, không sử dụng innerHTML.
• Ràng buộc: Không sửa file ngoài phạm vi, không gọi console.error khi gặp lỗi dự kiến, có cờ inFlight chống double-click.

#### Test condition / Kiểm tra:
• Vượt qua tất cả các điều kiện: normal, slow, error, hang, empty.

#### Kết luận:
• keep
• Lý do: Code sạch, chuẩn Web API, xử lý bất đồng bộ chuẩn xác và đáp ứng 100% yêu cầu trong brief.

──────
### Bảng tổng hợp

| File | Lỗi chính & Dòng code | Test Condition làm lộ lỗi | Kết luận |
|---|---|---|---|
| a.js | Dòng 48–52: Dùng forEach(async ...) không await, luôn ghi 0 đ vào #stock-value. | normal, slow | reject (hoặc fix) |
| b.js | Dòng 13: Dùng { timeout: 5000 } không hợp lệ trong Web fetch. Dòng 51–54: Không dọn dẹp DOM khi stock lỗi. | hang, error | fix (hoặc reject) |
| c.js | Không có lỗi. Đạt toàn bộ yêu cầu và ràng buộc. | Vượt qua tất cả | keep |

---

## Verification Tables

*(To be filled in Step 8 after running the complete test rows in Step 4 & 5)*

### Table 1: Assistant Claims vs Actual Runs
| Assistant Claim | Verified by Row / Run | Result (True / False / Inexact) |
|---|---|---|
| *(Pending verification in Step 4 & 5)* | | |

### Table 2: Failing Rows Missed by Assistant
| Failing Row Missed | Evidence |
|---|---|
| *(Pending verification in Step 4 & 5)* | | |
