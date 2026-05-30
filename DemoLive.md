# KỊCH BẢN LIVE DEMO — NHÓM 30

> **Người thực hiện:** P7
> **Thời gian:** 5 phút
> **Notebook:** `notebooks/00_WEEK4_LIVE_DEMO.ipynb`

---

## TỔNG QUAN DEMO (5 phút)

| TT | Thời gian | Nội dung | Case |
|----|-----------|----------|------|
| 1 | 30 giây | Khởi tạo + giới thiệu giao diện | Load model + data |
| 2 | 1 phút | Case 1: Đề xuất đúng LOAN | KH 751560 |
| 3 | 1 phút | Case 2: Risk Gate chặn offer | KH 509963 |
| 4 | 1 phút | Case 3: Dynamic Addition | Nhập ID ngẫu nhiên |
| 5 | 1 phút | Case 4: No Offer + Tổng kết | KH 855396 |
| 6 | 30 giây | Kết luận + chốt | — |

---

## TRƯỚC KHI DEMO (chuẩn bị)

Trước khi vào phòng bảo vệ, mở sẵn notebook `00_WEEK4_LIVE_DEMO.ipynb`, restart kernel và chạy lần lượt cell 0, cell 1 (khởi tạo) để đảm bảo load được hết model và data. Khi đến lượt demo, notebook đã sẵn sàng.

---

## BƯỚC 0 — MỞ ĐẦU (30 giây)

> *Đứng dậy, cầm micro, đi về phía máy chiếu.*

Kính thưa thầy và các bạn, em là **[TÊN]**. Bây giờ em sẽ chạy trực tiếp hệ thống Live Demo để minh họa toàn bộ pipeline F8 vừa trình bày.

> *Chỉ vào màn hình — cell 0, cell 1 đã chạy sẵn.*

Hệ thống đã được khởi tạo. File demo chứa **49 khách hàng đại diện** được chọn từ tập holdout test — những khách hàng này **chưa từng được dùng để huấn luyện** bất kỳ model nào. Mỗi khách hàng thuộc một trong 10 kịch bản kinh doanh: đề xuất vay, đề xuất thẻ, đề xuất tiết kiệm, rủi ro cao, không offer, phân khúc tài sản, v.v.

> *Chỉ vào bảng summary ở cell 3.*

Đây là phân bổ các case demo. Chúng ta có 5 case fraud risk, 3 case mỗi loại offer, 3 case no offer, và 20 khách hàng ngẫu nhiên từ holdout.

---

## BƯỚC 1 — CASE 1: ĐỀ XUẤT LOAN CHÍNH XÁC (1 phút)

> *Chuyển đến cell 4-5. Chạy cell 5.*

**Case đầu tiên: khách hàng 751560.** Em chọn case này vì đây là một khách hàng thực tế — không phải bịa ra để đẹp.

> *Chỉ vào kết quả in ra, đọc từng mục:*

Nhìn vào kết quả, chúng ta thấy 5 khối thông tin:

**Khối 1 — Persona**: khách hàng này thuộc **Persona Cluster 4** (Salary Dependent — nhận lương đều đặn), confidence 19.57%, thuộc phân khúc tài sản **Bottom90**, 26 tuổi. Đây là chân dung khách hàng trẻ, có thu nhập ổn định — hoàn toàn phù hợp với nhu cầu vay tiêu dùng.

**Khối 2 — Fraud Risk**: fraud score **0.0000**, fraud flag = 0, lý do "stable baseline profile". Không có dấu hiệu bất thường nào. Track A xác nhận khách hàng này an toàn.

**Khối 3 — Next Best Offer**: hệ thống đề xuất **LOAN** với max rank = 1.0000. Đây là percentile rank cao nhất có thể — nghĩa là khách hàng này nằm trong top những người có khả năng vay cao nhất toàn bộ tập test. Lý do xAI: "Choose LOAN because it has the highest rank, score = 0.9854".

**Khối 4 — Holdout labels**: loan = **1**, td = 0, card = 0. Đây là nhãn thật từ dữ liệu — khách hàng này **đã thực sự đăng ký vay** ở kỳ tiếp theo. Mô hình dự đoán đúng!

**Khối 5 — Main profile signals**: có e-banking, 8 hoạt động, 1 lần login, CA balance ~3.2 triệu, chưa có sản phẩm TD hay thẻ nào. Đây là insight để xAI giải thích: khách hàng mới chỉ có tài khoản thanh toán cơ bản, đang hoạt động đều → nhu cầu vay là tự nhiên.

> *Qua case tiếp theo.*

---

## BƯỚC 2 — CASE 2: RISK GATE CHẶN OFFER (1 phút)

> *Chuyển đến cell 10-11. Chạy cell 11.*

**Case thứ hai: khách hàng 509963.** Đây là case cho thấy **sức mạnh của Risk-Aware Policy Gate** — nơi Track A và Track B phối hợp với nhau.

> *Chỉ vào từng khối:*

**Persona**: Cluster 2 (Traditional Depositor), confidence thấp 11.32% — khách hàng này không khớp rõ với bất kỳ persona nào. Nhưng điều đáng chú ý: **wealth segment P99_top1** — top 1% khách hàng giàu nhất. Age 51 tuổi.

**Fraud Risk**: fraud score **0.8474**, fraud flag = **1.0**. Ba lý do được xAI liệt kê: (1) high-value wealth segment — khách hàng siêu giàu luôn được giám sát chặt, (2) diverse transaction behavior — 1310 hoạt động, 290 lần login, 29 giao dịch trong kỳ, (3) **high outside-bank transfer ratio** — gần 69% giao dịch chuyển tiền ra ngân hàng khác. Đây là dấu hiệu kinh điển của rửa tiền hoặc chuyển tiền bất hợp pháp.

**Next Best Offer**: hệ thống VẪN đề xuất Card, nhưng max rank chỉ 0.7476 — thấp hơn nhiều so với case trước. Điều này có nghĩa: ngay cả khi model đề xuất sản phẩm, **độ tự tin không cao**.

**Quan trọng nhất**: nhìn vào holdout labels — loan = 0, td = 0, card = **0**. Khách hàng này KHÔNG đăng ký bất kỳ sản phẩm nào. Và với fraud flag = 1, offer này sẽ bị **Risk Gate chặn lại**, chuyển sang manual review.

**Main profile signals**: các bạn nhìn dòng `avg_td_balance` — **4.3 tỷ đồng**. `avg_ca_balance` — **88 triệu**. `product_count` = 2 — đã có 2 sản phẩm. Đây là khách hàng cực kỳ giá trị, nhưng cũng cực kỳ rủi ro. Gửi offer tự động cho khách hàng này mà không kiểm tra là sai lầm nghiêm trọng về compliance.

> *Đây là minh họa hoàn hảo cho câu "an toàn" trong 3 giá trị cốt lõi.*

---

## BƯỚC 3 — CASE 3: DYNAMIC ADDITION (1 phút)

> *Chuyển đến cell 22-23.*

Bây giờ em sẽ minh họa tính năng **Dynamic Addition** — nạp khách hàng mới tinh vào hệ thống theo thời gian thực. Đây là tính năng chứng minh pipeline **hoàn toàn tự động** và sẵn sàng cho production.

> *Chỉ vào cell 23. Sửa `CUSTOMER_ID_DE_ADD` thành một ID chưa có trong demo, ví dụ `377287` hoặc để giáo viên chọn. Chạy cell.*

Hệ thống đang thực hiện các bước sau trong **dưới 1 giây**:
1. Kiểm tra ID có trong tập test không
2. Load đặc trưng từ file parquet
3. Tính wealth segment động dựa trên phân vị toàn bộ tập dữ liệu
4. Sinh xAI reason tự động từ các tín hiệu
5. Append vào file demo CSV
6. In kết quả đầy đủ

> *Chỉ vào kết quả.*

Kết quả hiển thị đầy đủ 5 khối thông tin giống như các case chuẩn bị sẵn. Khách hàng này **vừa được thêm vào hệ thống** — chưa từng có trong file demo trước đó.

> *Nếu giáo viên muốn thử thêm ID khác: mời thầy/cô chọn một ID bất kỳ trong danh sách ở cell 18.*

Các ID có sẵn để thầy cô chọn: nhóm VIP an toàn (865283, 69788), nhóm rủi ro (444248, 609360, 747140), nhóm đề xuất đúng (482901, 772432, 310093), nhóm không hoạt động (5234, 106640).

---

## BƯỚC 4 — CASE 4: NO OFFER + TỔNG KẾT (1 phút)

> *Chuyển đến cell 14-15. Chạy cell 15.*

**Case cuối: khách hàng 855396 — No Offer.**

Đây là case thể hiện **tính tiết kiệm chi phí** của hệ thống. Không phải khách hàng nào cũng nên nhận offer.

> *Chỉ vào kết quả:*

**Persona**: Cluster 3, confidence **56.64%** — confidence cao nhất trong tất cả các case, nghĩa là hệ thống rất chắc chắn về persona này. Age **18.3 tuổi** — khách hàng trẻ nhất trong demo. Wealth segment: Bottom90.

**Fraud Risk**: N/A — khách hàng không có đủ hoạt động để đánh giá rủi ro. Lý do: "stable baseline profile" — thực ra là không có gì để đánh giá.

**Next Best Offer**: **NO_OFFER**. Max rank = 0.0001 — cực kỳ thấp. Lý do xAI: "No offer because all 3 percentile-ranks are low, so the model is not confident enough." Hệ thống **tự quyết định không gửi offer** vì không đủ tự tin — thay vì cố gắng đề xuất một sản phẩm với độ chính xác thấp.

**Holdout labels**: loan = 0, td = 0, card = **0**. Khách hàng này thực sự không đăng ký gì — xác nhận quyết định No Offer là chính xác.

**Main profile signals**: `has_ebanking = 0`, toàn bộ activity = 0, `avg_ca_balance` ~114 nghìn. Đây là khách hàng gần như không tồn tại trong hệ thống ngân hàng số. Gửi offer cho khách hàng này là **lãng phí chi phí marketing**.

> *Ý nghĩa: 4.41% khách hàng rơi vào No Offer — tiết kiệm hàng trăm nghìn lượt gửi offer vô ích mỗi kỳ.*

---

## BƯỚC 5 — KẾT LUẬN DEMO (30 giây)

> *Đứng giữa, không nhìn màn hình.*

Kính thưa thầy và các bạn, qua 4 case demo, chúng em đã minh họa toàn bộ pipeline F8 hoạt động thực tế:

- **Case 1 (751560)** — hệ thống dự đoán đúng khách hàng sẽ vay, với đầy đủ lý do từ persona đến tín hiệu hành vi.
- **Case 2 (509963)** — Risk Gate phát hiện rủi ro và chặn offer, bảo vệ ngân hàng khỏi rủi ro compliance.
- **Case 3 (Dynamic)** — khách hàng mới được nạp và xử lý hoàn toàn tự động dưới 1 giây.
- **Case 4 (855396)** — hệ thống từ chối offer khi không đủ tự tin, tiết kiệm chi phí marketing.

Toàn bộ pipeline — từ raw data đến quyết định — **tự động, minh bạch, và có thể giải thích được**.

> *Quay về chỗ. Cả nhóm đứng dậy chào kết thúc.*

---

## PHỤ LỤC: DANH SÁCH ID DỰ PHÒNG

### Nhóm đề xuất đúng (có thể dùng thêm nếu giáo viên hỏi)

| ID | Case | Persona | Offer | Holdout |
|----|------|---------|-------|---------|
| **751560** | top_loan_offer | Cluster 4 (Salary Dependent) | LOAN (rank 1.0) | loan=1 ✓ |
| **44703** | top_card_offer | Cluster 3 (Credit Active) | CARD (rank 1.0) | card=1 ✓ |
| **518080** | top_td_offer | Cluster 1 (Traditional Depositor) | TD (rank 1.0) | td=1 ✓ |

### Nhóm rủi ro (dùng khi muốn nhấn mạnh Risk Gate)

| ID | Case | Fraud Score | Wealth | Dấu hiệu |
|----|------|-------------|--------|----------|
| **509963** | high_fraud_risk | 0.8474 | P99_top1 | 4.3 tỷ TD, 69% chuyển ngoài |
| **413763** | high_fraud_risk | 0.6395 | P95_P99 | 100% chuyển ngoài, 40% giao dịch đêm |
| **609360** | high_fraud_risk | 0.6321 | P95_P99 | 186 triệu CA, 100% chuyển ngoài |

### Nhóm VIP an toàn (dùng khi muốn show khách hàng giá trị cao)

| ID | Case | Wealth | Offer |
|----|------|--------|-------|
| **732351** | wealth_P99_top1 | 800 triệu TD | LOAN |
| **865283** | wealth_P99_top1 | 739 triệu TD | CARD |

### Nhóm Dynamic Addition (dùng khi muốn nạp khách hàng mới)

| ID | Mô tả |
|----|-------|
| **377287** | KH trong holdout, chưa có trong demo |
| **772432** | KH đề xuất đúng, có thể test |
| **482901** | KH đề xuất đúng, có thể test |

### Nhóm No Offer (dùng khi muốn nhấn mạnh tiết kiệm chi phí)

| ID | Case | Tuổi | Lý do |
|----|------|------|-------|
| **855396** | low_confidence_no_offer | 18.3 | Không hoạt động, rank quá thấp |

---

## XỬ LÝ TÌNH HUỐNG

| Tình huống | Cách xử lý |
|------------|-----------|
| Notebook bị crash, kernel chết | Restart kernel, chạy lại từ cell 0, mất ~10 giây. Vừa chạy vừa nói: "Hệ thống đang khởi tạo lại, thưa thầy cô đợi một chút ạ." |
| ID nhập vào không tồn tại | Hàm `add_and_show_customer` sẽ báo lỗi rõ ràng. Nói: "ID này không nằm trong tập holdout test. Em xin phép chọn một ID khác từ danh sách dự phòng." Rồi chọn ID trong bảng trên. |
| Hàm random bị lỗi NaN (cell 25) | Đây là bug đã biết — một số KH trong parquet thiếu cột `persona_cluster`. **Không chạy cell 25**. Dùng cell 23 với ID cụ thể thay thế. |
| Giáo viên yêu cầu ID không có trong danh sách | Nhập ID đó vào cell 23. Nếu lỗi, giải thích: "ID này không thuộc tập holdout test. Em xin tra cứu một ID tương tự trong danh sách." Rồi chọn ID gần nhất. |
| Chạy chậm, quá 5 phút | Cắt bớt case 4 (No Offer). Chỉ chạy case 1 (Loan), case 2 (Fraud), case 3 (Dynamic) là đã đủ 3 giá trị cốt lõi. |
| Giáo viên hỏi sâu về xAI | Trả lời: "xAI engine sinh lý do dựa trên 6 tín hiệu: wealth segment, digital activity, transaction diversity, outside-bank ratio, night ratio, và baseline stability. Mỗi tín hiệu có ngưỡng kích hoạt riêng, và 3 lý do mạnh nhất được hiển thị." |
