# KỊCH BẢN THUYẾT TRÌNH — NHÓM 30

**Môn:** CS320 - Học Máy | **GV:** Vũ Minh Hoàng
**Tổng:** 10 phút nói + 5 phút demo = **15 phút**
**File slide:** `Nhóm 30.pptx` (21 slides)

> **Luật chung:** Không đọc bảng. Chỉ nêu số chốt. Chuyển tiếp nhanh. Demo là phần quan trọng nhất.

---

## PHÂN CÔNG

| #   | Slides   | Nội dung                           | T.gian   |
|-----|----------|------------------------------------|----------|
| P1  | 1–6      | Mở đầu + F0→F8 (lướt nhanh)        | 1.5 phút |
| P2  | 7–9      | Track C: K=8, 8 Persona            | 1 phút   |
| P3  | 10–12    | Track A: MLP+GMM, Stack Loss       | 1.5 phút |
| P4  | 13–14    | Track B: Hybrid + Hiệu năng        | 1.5 phút |
| P5  | 15–16    | Time-Series + Persona Impact       | 1 phút   |
| P6  | 17–18    | Decision Layer + Risk Gate         | 1.5 phút |
| P7  | 19–20    | Live Demo + Tổng kết               | 1 phút   |
| P7  | Demo     | **Chạy notebook trực tiếp**         | **5 phút** |

---

## P1 — Slides 1–6 | 1.5 phút

> Người nói: **[TÊN]**

**Slide 1:** *(slide tiêu đề — tên nhóm, môn học, GV)*
Kính thưa thầy và các bạn, em là **[TÊN]**, đại diện Nhóm 30. Hôm nay chúng em trình bày hệ thống **phân tích khách hàng 360 độ** — từ dữ liệu thô 150.000+ khách hàng đến quyết định offer tự động cho 3 sản phẩm: vay, tiết kiệm, thẻ tín dụng. Toàn bộ pipeline chạy dưới 1 giây cho mỗi khách hàng mới.

**Slide 2:** *(sơ đồ tổng quan — 4 khối chính)*
Bố cục bài trình bày đi theo kiến trúc hệ thống: **9 thế hệ mô hình** (F0→F8) cho thấy hành trình cải tiến → **3 Track xử lý song song** (C: phân cụm hành vi, A: phát hiện rủi ro, B: đề xuất sản phẩm) → **Decision Layer** ra quyết định cuối cùng → **Live Demo** chạy trực tiếp trên notebook.

**Slide 3:** *(timeline F0→F4 — baseline & exploration)*
Hành trình bắt đầu từ **F0**: Isolation Forest trên dữ liệu thô — phát hiện bất thường đơn giản nhưng không có persona. **F1–F2**: thêm PCA giảm chiều, KMeans phân cụm — lần đầu có chân dung khách hàng. **F3**: ensemble MLP + IForest + LOF — tham vọng kết hợp 3 model nhưng gặp bài học **Stack Loss**: thêm LOF làm Precision giảm 29%. **F4**: chuyển sang GMM soft clustering — chất lượng phân cụm tăng rõ rệt.

**Slide 4:** *(timeline F5→F7 — scaling & optimization)*
**F5**: mở rộng Persona Zoo với 6 biến thể clustering (KMeans, GMM, KNN, AE_K6, AE_K8, AE_K10) — không còn phụ thuộc vào một thuật toán duy nhất. **F6**: thử Joint Model — một network chung cho cả 3 sản phẩm, và Time-Series Transformer/GRU — phát hiện Transformer thua XGBoost trên chuỗi ngắn. **F7**: bổ sung ANOVA chọn đặc trưng, Risk-Aware Policy Gate, hoàn thiện kiến trúc hybrid.

**Slide 5:** *(mốc F8 — PRODUCTION READY)*
**F8** là thế hệ chốt — tinh gọn từ tất cả bài học trước: giữ GMM cho Track A (vì soft clustering bắt fraud tốt nhất), giữ XGBoost cho Track B (vì thắng tuyệt đối Transformer/GRU), giữ KMeans K=8 cho Track C (Silhouette 0.68, cụm đồng đều), bỏ LOF (vì Stack Loss), bỏ Transformer (vì overfit chuỗi ngắn). Mỗi thế hệ không chỉ cải thiện metric — mà quan trọng hơn, **mỗi thế hệ trả lời một câu hỏi**: nên dùng model gì? nên kết hợp ra sao? nên bỏ cái gì?

**Slide 6:** *(sơ đồ kiến trúc F8 — 3 track + Decision Layer)*
Kiến trúc chốt F8 vận hành theo pipeline 3 track song song: **Track C** — KMeans K=8 trên PCA8, Silhouette 0.6825, output 8 persona giúp hiểu "khách hàng là ai". **Track A** — MLP + GMM, ROC-AUC 0.9746, output fraud flag để chặn offer rủi ro. **Track B** — XGBoost Joint BR cho Loan (AUC-PR 0.2515), XGBoost riêng cho TD và Card (Lift tới 7.54x), output propensity score cho từng sản phẩm. Cả 3 track hội tụ về Decision Layer: percentile-rank chuẩn hóa → chọn offer tốt nhất → Risk Gate kiểm tra lần cuối → xuất quyết định. **Pipeline hoàn toàn tự động, mỗi khách hàng mới xử lý dưới 1 giây.**

> Em chuyển cho bạn **[TÊN P2]** trình bày Track C.

---

## P2 — Slides 7–9 | 1 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 7:** *(chỉ sơ đồ)*
Track C: PCA8 → KMeans → ANOVA → 8 Persona. Đặc trưng phân biệt mạnh nhất: `activity_hour_entropy`, `avg_ca_balance`.

**Slide 8:** *(chỉ bảng)*
So sánh K từ 6 đến 12: **K=8 được chọn** — Silhouette 0.6825, cụm đồng đều, dễ giải thích.

**Slide 9:**
8 chân dung: Power Digital, Traditional Depositor, Credit Active, Mass Market, High-Value Saver, E-Commerce, Salary Dependent, Young Native.

> Mời bạn **[TÊN P3]** — Track A.

---

## P3 — Slides 10–12 | 1.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 10:**
Track A phát hiện rủi ro. Không có nhãn thật → dùng **Strict Pseudo-Eval**: giao dịch đêm + tiền lớn + ngoài ngân hàng. Ensemble F3 (MLP+IForest+LOF) → F8 chốt lại **MLP + GMM**.

**Slide 11:** *(chỉ bảng)*
Kết quả: **MLP+GMM dẫn đầu** — AUC-PR 0.3625, ROC-AUC 0.9746. GMM soft clustering vượt trội KMeans (0.2794) vì fraud nằm ở vùng giao thoa giữa các cụm.

**Slide 12:**
Bài học Stack Loss: thêm LOF → Precision@500 từ **0.730 xuống 0.518** (giảm 29%). Nguyên nhân: LOF mật độ cục bộ xung đột với MLP toàn cục. **Không phải cứ thêm mô hình là tốt.**

Về mặt lý thuyết, ensemble chỉ hiệu quả khi các mô hình thành viên **sai khác nhau** — đây là **Diversity Principle** (Krogh & Vedelsby, 1995): generalization error của ensemble = error trung bình **trừ đi** độ đa dạng. Khi hai mô hình xung đột, diversity âm, ensemble còn tệ hơn single model.

> Mời bạn **[TÊN P4]** — Track B.

---

## P4 — Slides 13–14 | 1.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 13:**
Về mặt lý thuyết, **No Free Lunch** khẳng định không một model family nào tối ưu cho mọi bài toán. Mỗi sản phẩm có phân phối dữ liệu khác nhau — Loan 4.1%, TD 2.3%, Card 1.4% — nên thay vì đặt cược vào một kiến trúc, chúng em chạy song song 7 họ mô hình: từ LogReg bias cao đến MLP variance cao, từ HGB boosting đến XGBoost. **Bias-Variance Decomposition** cho thấy kết hợp các model có profile khác nhau sẽ giảm expected error tổng thể.

Dữ liệu siêu mất cân bằng. Chiến lược: **Joint BR cho Loan**, Product-per-model cho Card & TD, ANOVA chọn đặc trưng cho Card.

**Slide 14:** *(chỉ bảng)*
Mô hình chốt:
- **Card**: XGBoost + GMM — Lift **7.54x**
- **Loan**: Joint BR-XGBoost + KMeans — AUC-PR **0.2515**
- **TD**: XGBoost + AE_K8 — Lift **4.66x**

So với Tuần 2: tất cả chỉ số đều cải thiện.

> Mời bạn **[TÊN P5]** — Time-Series và Persona Impact.

---

## P5 — Slides 15–16 | 1 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 15:** *(chỉ bảng)*
Thử nghiệm Transformer/GRU vs XGBoost Tabular. Kết quả: Tabular thắng tuyệt đối (AUC-PR 0.25 vs 0.08). Lý do: chuỗi 3-6 tháng quá ngắn. **Giữ XGBoost cho production.**

**Slide 16:** *(chỉ bảng Persona Impact)*
Không có variant nào tốt nhất cho tất cả: GMM nhất Track A, KMeans nhất Loan, AE_K8 nhất TD → **chọn Hybrid.**

> Mời bạn **[TÊN P6]** — Decision Layer.

---

## P6 — Slides 17–18 | 1.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 17:**
Từ propensity score → offer: **Percentile-Rank chuẩn hóa** giải quyết chênh lệch phân phối giữa 3 sản phẩm. Đây chính là ý tưởng **Stacking** (Wolpert, 1992) — tách biệt tầng base learner chuyên biệt cho từng sản phẩm khỏi tầng meta-learner ra quyết định cuối cùng. Nhờ đó mỗi tầng tối ưu một mục tiêu riêng, không phải trade-off giữa 3 class có base rate lệch nhau.

Kết quả: TD 34.85%, Card 33.69%, Loan 27.04%, **No Offer 4.41%** (<5% chuẩn ngân hàng số).

**Slide 18:**
Rào chắn cuối: **Risk-Aware Policy Gate**. Fraud flag từ Track A → tạm giữ offer. Chỉ **58 khách hàng (0.04%)** cần manual review. **99.96% tự động** — giảm 95% chi phí vận hành.

> Mời bạn **[TÊN P7]** — Live Demo.

---

## P7 — Slides 19–20 + Demo | 1 phút + 5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 19:**
Hệ thống demo: 2 chế độ — **theo ID** (tra cứu bất kỳ KH nào từ 150K+, <1 giây) và **ngẫu nhiên** (bốc KH mới tinh). Tích hợp **xAI** tự động giải thích lý do offer, đáp ứng Basel/IFRS.

**Slide 20:** *(chỉ 3 con số)*
Ba giá trị cốt lõi: **Cá nhân hóa** (Lift 4.4x–7.5x, ROI +400-750%), **An toàn** (0.04% manual review), **Tự động** (pipeline 24/7).

> Bây giờ em xin chạy demo trực tiếp trên notebook.

---

### 🎯 DEMO (5 phút) — P7 thực hiện

> Mở `notebooks/00_WEEK4_LIVE_DEMO.ipynb`, chạy từng cell:

| TT | Thao tác                 | Nội dung                                                    |
|----|-------------------------|--------------------------------------------------------------|
| 1  | Chạy cell khởi tạo       | Load model + data                                            |
| 2  | Case 1: KH **751560**    | Demo case trúng Loan 100%, có xAI giải thích                  |
| 3  | Case 2: KH **509963**    | Demo case Fraud → offer bị chặn bởi Risk Gate                 |
| 4  | Case 3: Dynamic Addition | Nhập ID ngẫu nhiên → hệ thống tự động score + offer           |
| 5  | Kết luận                 | Chốt: pipeline hoàn chỉnh, tự động, sẵn sàng production       |

> Hết demo → cả nhóm đứng dậy.

Kính thưa thầy và các bạn, đó là toàn bộ hệ thống của Nhóm 30. Chúng em cảm ơn thầy đã lắng nghe và rất mong nhận được góp ý. **Cảm ơn thầy!**

---

## SỐ LIỆU CHỐT CẦN THUỘC

| Số           | Ý nghĩa                                |
|--------------|----------------------------------------|
| **0.6825**   | Track C Silhouette (K=8)               |
| **0.9746**   | Track A ROC-AUC (MLP+GMM)              |
| **0.3625**   | Track A AUC-PR                         |
| **7.54x**    | Track B Top 10% Lift (Card)            |
| **4.41%**    | No Offer (<5%)                         |
| **0.04%**    | Manual Review (58 KH)                  |
| **< 1 giây** | Dynamic User Addition                  |
| **29%**      | Stack Loss: Precision giảm khi thêm LOF |

---

## TIMELINE

| Phút  | Ai  | Việc                   |
|-------|-----|------------------------|
| 0:00  | P1  | Bắt đầu                |
| 1:30  | P2  | Track C                |
| 2:30  | P3  | Track A                |
| 4:00  | P4  | Track B                |
| 5:30  | P5  | Time-Series + Persona  |
| 6:30  | P6  | Decision + Risk        |
| 8:00  | P7  | Tổng kết               |
| 9:00  | P7  | **BẮT ĐẦU DEMO**       |
| 14:00 | ALL | Kết thúc, cảm ơn       |

---

## TỪ VIẾT TẮT & THUẬT NGỮ

### Pipeline & Kiến trúc

| Viết tắt      | Đầy đủ                          | Giải thích |
|---------------|---------------------------------|------------|
| **F0–F8**     | Generation 0–8                  | 9 thế hệ mô hình từ baseline đến production |
| **Track A**   | Fraud Detection Track           | Phát hiện rủi ro / bất thường |
| **Track B**   | Product Recommendation Track    | Đề xuất sản phẩm tài chính (NBFO) |
| **Track C**   | Customer Persona Track          | Phân cụm hành vi khách hàng |
| **NBFO**      | Next Best Financial Offer       | Bài toán đề xuất sản phẩm tiếp theo |
| **P1–P7**     | Presenter 1–7                   | Người thuyết trình |
| **KH**        | Khách hàng                      | Customer |

### Thuật toán & Mô hình

| Viết tắt        | Đầy đủ                              | Giải thích |
|-----------------|-------------------------------------|------------|
| **PCA**         | Principal Component Analysis        | Giảm chiều dữ liệu |
| **KMeans**      | K-Means Clustering                  | Phân cụm dựa trên khoảng cách tới tâm |
| **GMM**         | Gaussian Mixture Model              | Soft clustering — mỗi điểm thuộc nhiều cụm với xác suất |
| **KNN**         | K-Nearest Neighbors                 | Phân cụm/phân loại dựa trên K láng giềng gần nhất |
| **AE**          | Autoencoder                         | Mạng neural tự mã hóa — học biểu diễn ẩn không giám sát |
| **AE_K6/8/10**  | Autoencoder + KMeans (K=6/8/10)     | Autoencoder giảm chiều → KMeans phân cụm embedding |
| **IForest**     | Isolation Forest                    | Phát hiện bất thường dựa trên cây ngẫu nhiên |
| **LOF**         | Local Outlier Factor                | Phát hiện bất thường dựa trên mật độ cục bộ |
| **MLP**         | Multi-Layer Perceptron              | Mạng neural truyền thẳng nhiều lớp |
| **HGB**         | Histogram-based Gradient Boosting   | Gradient Boosting dùng histogram (sklearn) |
| **LogReg**      | Logistic Regression                 | Hồi quy logistic — mô hình tuyến tính |
| **RF**          | Random Forest                       | Ensemble bagging của cây quyết định |
| **ExtraTrees**  | Extremely Randomized Trees          | Biến thể RF — split ngẫu nhiên hơn |
| **XGB / XGBoost** | Extreme Gradient Boosting         | Gradient Boosting tối ưu — SOTA cho dữ liệu bảng |
| **LGBM**        | LightGBM                            | Gradient Boosting của Microsoft — leaf-wise tree |
| **BR**          | Binary Relevance                    | Chiến lược multi-label: mỗi nhãn một classifier riêng |
| **GRU**         | Gated Recurrent Unit                | Mạng hồi quy có cổng — nhẹ hơn LSTM |
| **Transformer** | Transformer Encoder                 | Kiến trúc attention cho dữ liệu chuỗi |

### Chỉ số đánh giá

| Viết tắt           | Đầy đủ                              | Giải thích |
|--------------------|-------------------------------------|------------|
| **AUC-PR**         | Area Under Precision-Recall Curve   | Chất lượng model trên dữ liệu mất cân bằng |
| **ROC-AUC**        | Area Under ROC Curve                | Khả năng phân biệt positive/negative |
| **Lift**           | Lift (Top X%)                       | Tỷ lệ chuyển đổi top X% so với ngẫu nhiên |
| **Precision@K**    | Precision at Top K                  | Độ chính xác trên K dự đoán cao nhất |
| **Silhouette**     | Silhouette Score                    | Chất lượng phân cụm (−1 đến 1, càng cao càng tốt) |
| **Davies-Bouldin** | Davies-Bouldin Index                | Độ tương tự giữa các cụm (càng thấp càng tốt) |
| **Calinski-Harabasz** | Variance Ratio Criterion         | Tỷ số phương sai giữa các cụm / trong cụm |

### Khái niệm lý thuyết

| Thuật ngữ                      | Giải thích |
|--------------------------------|------------|
| **No Free Lunch**              | Không tồn tại thuật toán tối ưu cho mọi bài toán (Wolpert & Macready, 1997) |
| **Bias-Variance Decomposition** | Lỗi dự đoán = bias² + variance + noise. Kết hợp model khác profile → giảm error tổng |
| **Diversity Principle**        | Ensemble hiệu quả khi các model thành viên **sai khác nhau** (Krogh & Vedelsby, 1995) |
| **Stacking**                   | Meta-learning: base learner → prediction → meta-learner tổng hợp (Wolpert, 1992) |
| **Stack Loss**                 | Ensemble làm **giảm** hiệu năng do xung đột giữa các model thành viên |
| **Strict Pseudo-Eval**         | Đánh giá giả lập nghiêm ngặt — rule-based label + loại bỏ leakage |
| **Percentile-Rank**            | Chuẩn hóa score về phân vị → so sánh được giữa các phân phối khác nhau |
| **Soft Clustering**            | Phân cụm mềm — mỗi điểm thuộc nhiều cụm với xác suất |
| **Persona Zoo**                | Bộ sưu tập biến thể persona từ nhiều thuật toán clustering |
| **Decision Layer**             | Tầng ra quyết định cuối cùng — chọn offer theo percentile-rank |
| **Risk-Aware Policy Gate**     | Rào chắn: dùng Track A chặn offer nếu phát hiện rủi ro |
| **xAI**                        | Explainable AI — tự động giải thích lý do quyết định |
| **Joint Model**                | Một model dùng chung cho cả 3 sản phẩm (thay vì model riêng từng sp) |
| **Hybrid**                     | Kết hợp nhiều persona variant — mỗi track/bài toán dùng variant tối ưu riêng |

### Sản phẩm tài chính

| Viết tắt | Đầy đủ                   |
|----------|--------------------------|
| **Loan** | Vay tiêu dùng cá nhân     |
| **TD**   | Term Deposit — Tiết kiệm có kỳ hạn |
| **Card** | Thẻ tín dụng              |

### Dữ liệu & Đặc trưng

| Thuật ngữ                 | Giải thích |
|---------------------------|------------|
| **CUSTOMER_NUMBER**       | Mã định danh khách hàng |
| **activity_hour_entropy** | Độ hỗn loạn giờ hoạt động — phân tán thời gian giao dịch |
| **avg_ca_balance**        | Số dư trung bình tài khoản thanh toán (Current Account) |
| **P99**                   | Phân vị thứ 99 — nhóm KH giá trị tài sản cao nhất |
| **Holdout**               | Tập dữ liệu giữ lại độc lập, không dùng train hay chọn model |
| **Leakage**               | Rò rỉ dữ liệu — thông tin test/holdout vô tình lọt vào training |
