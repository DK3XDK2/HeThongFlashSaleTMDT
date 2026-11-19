# DỰ ÁN: HỆ THỐNG FLASH SALE THƯƠNG MẠI ĐIỆN TỬ
**Học phần:** Kiến trúc và Thiết kế Phần mềm (KTV332)  
**Trường:** Trường Đại học Công nghệ Thông tin và Truyền thông – Đại học Thái Nguyên (ICTU)  
**Mã số đề tài:** 78204  

---

## 1. Bảng Theo dõi Tiến độ Thực hiện Dự án (Dành cho các thành viên trong nhóm)

Tài liệu này giúp các thành viên trong nhóm nắm được nhanh tiến độ hiện tại của dự án, phần nào đã hoàn thành và phần nào đang chờ thực hiện tiếp theo:

| Đợt đánh giá | Sản phẩm bắt buộc phải nộp | Tài liệu / Thư mục trong Repo | Trạng thái hiện tại |
| :--- | :--- | :--- | :--- |
| **1. Bài kiểm tra thường xuyên 1**<br/>*(Bài kiểm tra kỹ năng 1)* | Báo cáo phân tích bối cảnh, Stakeholders, yêu cầu chức năng (FR), 7 nhóm yêu cầu phi chức năng (NFR) định lượng, phân loại ưu tiên (Utility Tree), bộ kịch bản chất lượng (QAS) chuẩn 6 phần SEI, ràng buộc hệ thống và danh mục tài liệu tham khảo có dẫn chứng. | [docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md](./docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md) | **ĐÃ HOÀN THÀNH (10/10 tiêu chí)** |
| **2. Bài kiểm tra thường xuyên 2**<br/>*(Bài kiểm tra kỹ năng 2)* | Báo cáo đề xuất và phân tích đánh đổi các phong cách kiến trúc, lựa chọn kiến trúc cuối cùng, thiết kế mô hình **4+1 Views** (*Logical, Development, Process, Physical/Deployment, Scenario View*), đề xuất giải pháp công nghệ và sơ đồ tuần tự (Sequence Diagram) xử lý tranh chấp đồng thời. *(Chưa phải nộp code)* | `docs/02_Bai_Kiem_Tra_Thuong_Xuyen_2_Thiet_Ke_Kien_Truc_4Plus1.md` *(Dự kiến)* | **Chưa thực hiện** *(Đang dừng sau Bài 1 để kiểm tra trước khi làm Bài 2)* |
| **3. Bài kiểm tra thường xuyên 3**<br/>*(Bài kiểm tra kỹ năng 3)* | *(Để trống)* | *(Chờ cập nhật)* | **Chờ có đề** |
| **4. Thi kết thúc học phần** | Báo cáo tổng hợp toàn diện (Giới thiệu, FR/NFR, QAS, Ràng buộc, Đánh đổi kiến trúc, Thiết kế 4+1 Views khớp với hệ thống thực tế, Kiểm thử & Đánh giá, Hướng dẫn triển khai & Vận hành) + **Sản phẩm phần mềm hoàn chỉnh** (Chức năng khách hàng mua Flash Sale, Chức năng quản trị/cấu hình chiến dịch, Giải pháp kỹ thuật đáp ứng NFR, Kiểm thử 14 kịch bản biên & tải đỉnh 20.000 lượt mua). | `src/`, `public/`, `tests/`, `docs/` *(Dự kiến)* | **Chưa thực hiện** *(Thực hiện ở giai đoạn cuối kỳ)* |

---

## 2. Tóm tắt Bài toán và Các Quy tắc Nghiệp vụ Nhóm đã Thống nhất

Để các bạn trong nhóm khi đọc tài liệu hoặc tham gia viết báo cáo / lập trình ở các giai đoạn sau không bị lệch thông số, dưới đây là các chỉ số và quy tắc cốt lõi đã chốt:

### 2.1. Thông số định lượng từ đề bài (Mã số 78204)
- **Định hướng cốt lõi:** Đọc nhiều (Read-heavy), ghi cạnh tranh cao (High-write contention).
- **Quy mô hệ thống (Scale):** Quản lý `10.000` sản phẩm, `100.000` khách hàng; mỗi chiến dịch Flash Sale có `100 – 1.000` sản phẩm giới hạn.
- **Chịu tải đỉnh (Peak Load):** Hỗ trợ `~20.000` lượt xem sản phẩm/phút và `~5.000` yêu cầu mua/phút.
- **Bài toán trọng tâm cần giải quyết:** **`20.000` lượt bấm mua tranh `100` sản phẩm**.
- **Hiệu năng đọc (Read Performance):** Hiển thị trang sản phẩm / giá / tồn kho **`< 500 ms`** từ bộ nhớ đệm, **không truy vấn trực tiếp vào luồng ghi**.
- **Nhất quán tồn kho (Consistency):** Không bán vượt số lượng (`Oversold = 0`), xử lý chống trùng người mua và có trạng thái đơn hàng rõ ràng.

### 2.2. Ba quy tắc nghiệp vụ & xử lý ngoại lệ đã chốt
1. **Giới hạn số lượng mua mỗi khách hàng (`max_per_user = 1`):**
   - Mỗi tài khoản chỉ được mua tối đa **01 sản phẩm / chiến dịch**.
   - Nếu nhập số lượng `<= 0` (số âm, số 0, số thập phân, sai kiểu dữ liệu) $\rightarrow$ Chặn ngay với lỗi `400 INVALID_QUANTITY` (tuyệt đối không để xảy ra lỗi trừ số âm thành cộng thêm kho).
   - Nếu nhập số lượng `> 1` $\rightarrow$ Chặn với lỗi `422 EXCEEDS_LIMIT_PER_USER`.
   - Nếu cùng 1 tài khoản bấm mua trùng từ lần thứ 2 trở đi $\rightarrow$ Chặn với lỗi `409 DUPLICATE_USER_ORDER`.
2. **Giữ tồn kho tạm thời (Reservation TTL) và Hoàn kho tự động:**
   - Khi bấm mua thành công (trong 100 suất đầu tiên), đơn hàng chuyển sang trạng thái `RESERVED` (giữ chỗ tạm thời chờ thanh toán).
   - Thời hạn giữ chỗ chuẩn trong tài liệu thiết kế: **`5 phút (300 giây)`** (khi chạy phần mềm Demo có thể chỉnh `15 giây` để dễ quan sát).
   - Nếu quá thời hạn TTL mà chưa thanh toán: Hệ thống tự động chuyển đơn sang `EXPIRED`, **cộng hoàn trả `+1` vào tồn kho khả dụng** và **gỡ khóa cho tài khoản đó** để suất hàng có thể được đặt mua lại.
3. **Ngăn xếp công nghệ (Tech Stack) định hướng của nhóm:**
   - **Cơ sở dữ liệu bền vững:** `PostgreSQL` (đảm bảo ACID, ràng buộc `CHECK (available_stock >= 0)` và chống trùng đơn).
   - **Bộ nhớ đệm & Chốt chặn tồn kho nguyên tử:** `Redis` (phục vụ luồng đọc `< 500 ms` và chạy `Atomic Lua Script` kiểm soát 20.000 lượt bấm mua tranh 100 sản phẩm trên RAM).
   - **Backend & Hàng đợi:** `Node.js (TypeScript / JavaScript)` + `Message Queue` xử lý tạo đơn bất đồng bộ và thu hồi đơn hết hạn TTL.

---

## 3. Cấu trúc Tệp tin Hiện tại trong Dự án

```text
FLASHSALE/
├── README.md                                            # Trang thông tin tổng quan & tiến độ hiện tại cho cả nhóm
└── docs/
    └── 01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md        # [ĐÃ HOÀN THÀNH] Báo cáo Bài kiểm tra thường xuyên 1 (đủ 10/10 tiêu chí)
```

---

## 4. Hướng dẫn cho Thành viên Nhóm khi Làm việc Tiếp

1. **Đọc báo cáo Bài kiểm tra thường xuyên 1:** Xem chi tiết tại [docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md](./docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md). Toàn bộ 10 tiêu chí chấm điểm của Bài 1 đã được đối chiếu tại Phần 10 của báo cáo.
2. **Xem danh sách 14 trường hợp kiểm thử & ngoại lệ bắt buộc (`TC-01` đến `TC-14`):** Đã được đặc tả đầy đủ tại **Phần 3, Mục 3.3** trong file [docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md](./docs/01_Bai_Kiem_Tra_Thuong_Xuyen_1_NFR_QAS.md).
3. **Quy ước chung khi viết tài liệu và code:**
   - Không sử dụng các emoji ký tự Unicode rời rạc trong báo cáo cũng như giao diện.
   - Giữ thống nhất toàn bộ tên thực thể, trạng thái đơn hàng (`RESERVED`, `PAID`, `EXPIRED`, `CANCELLED`) và các con số định lượng đã đặc tả ở Bài kiểm tra thường xuyên 1 khi chuyển sang thiết kế mô hình 4+1 Views ở Bài kiểm tra thường xuyên 2.
4. **Quy ước phân nhánh Git (Git Flow của nhóm):**
   - Nhánh **`main`**: Chỉ dùng để lưu các phiên bản đã hoàn thiện và kiểm định xong của từng đợt nộp bài (Final Push).
   - Nhánh **`develop`**: Nhánh làm việc và cập nhật hàng ngày của các thành viên trong nhóm (`git checkout develop`). Sau khi hoàn thành và kiểm tra đầy đủ từng đợt mới thực hiện gộp (`merge`) từ `develop` sang `main`.

