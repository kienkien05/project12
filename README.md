# KỊCH BẢN THUYẾT TRÌNH — NHÓM 30 (BẢN RÚT GỌN)

**Môn:** CS320 - Học Máy | **GV:** Vũ Minh Hoàng  
**Tổng:** 10 phút nói + 5 phút demo = **15 phút**  
**File slide:** `Nhóm 30.pptx` (21 slides)

> **Luật chung:** Không đọc bảng. Chỉ nêu số chốt. Chuyển tiếp nhanh. Demo là phần quan trọng nhất.

---

## PHÂN CÔNG

| # | Slides | Nội dung | T.gian |
|---|---|---|---|
| P1 | 1–6 | Mở đầu + F0→F8 (lướt nhanh) | 1.5 phút |
| P2 | 7–9 | Track C: K=8, 8 Persona | 1 phút |
| P3 | 10–12 | Track A: MLP+GMM, Stack Loss | 1.5 phút |
| P4 | 13–14 | Track B: Hybrid + Hiệu năng | 1.5 phút |
| P5 | 15–16 | Time-Series + Persona Impact | 1 phút |
| P6 | 17–18 | Decision Layer + Risk Gate | 1.5 phút |
| P7 | 19–20 | Live Demo + Tổng kết | 1 phút |
| **P7** | **Demo** | **Chạy notebook trực tiếp** | **5 phút** |

---

## P1 — Slides 1–6 | 1.5 phút

**Slide 1:** Kính thưa thầy và các bạn, em là **[TÊN]**. Nhóm 30 xin trình bày hệ thống phân tích khách hàng 360 độ — từ raw data đến quyết định offer tự động.

**Slide 2:** Bố cục: 9 thế hệ mô hình → 3 Track xử lý → Decision Layer → Live Demo.

**Slide 3–5:** *(bấm nhanh 3 slide)* Hành trình F0→F8: Từ Isolation Forest baseline, qua PCA clustering, Persona Zoo 6 biến thể, Joint Model, đến Production F8. Mỗi thế hệ là một bài học — quan trọng nhất là **Stack Loss** khi thêm LOF làm giảm Precision 29%. *(sẽ nói kỹ ở Track A)*

**Slide 6:** Kiến trúc chốt F8: 3 track song song — **Track C** KMeans K=8 (Silhouette 0.68), **Track A** MLP+GMM (ROC-AUC 0.97), **Track B** XGBoost Joint BR (Lift 7.54x). Toàn bộ pipeline tự động dưới 1 giây.

> Em chuyển cho bạn **[TÊN P2]** trình bày Track C.

---

## P2 — Slides 7–9 | 1 phút

Em là **[TÊN]**.

**Slide 7:** *(chỉ sơ đồ)* Track C: PCA8 → KMeans → ANOVA → 8 Persona. Đặc trưng phân biệt mạnh nhất: `activity_hour_entropy`, `avg_ca_balance`.

**Slide 8:** *(chỉ bảng)* So sánh K từ 6 đến 12: **K=8 được chọn** — Silhouette 0.6825, cụm đồng đều, dễ giải thích.

**Slide 9:** 8 chân dung: Power Digital, Traditional Depositor, Credit Active, Mass Market, High-Value Saver, E-Commerce, Salary Dependent, Young Native.

> Mời bạn **[TÊN P3]** — Track A.

---

## P3 — Slides 10–12 | 1.5 phút

Em là **[TÊN]**.

**Slide 10:** Track A phát hiện rủi ro. Không có nhãn thật → dùng **Strict Pseudo-Eval**: giao dịch đêm + tiền lớn + ngoài ngân hàng. Ensemble F3 (MLP+IForest+LOF) → F8 chốt lại **MLP + GMM**.

**Slide 11:** *(chỉ bảng)* Kết quả: **MLP+GMM dẫn đầu** — AUC-PR 0.3625, ROC-AUC 0.9746. GMM soft clustering vượt trội KMeans (0.2794) vì fraud nằm ở vùng giao thoa giữa các cụm.

**Slide 12:** Bài học Stack Loss: thêm LOF → Precision@500 từ **0.730 xuống 0.518** (giảm 29%). Nguyên nhân: LOF mật độ cục bộ xung đột với MLP toàn cục. **Không phải cứ thêm mô hình là tốt.** Về mặt lý thuyết, ensemble chỉ hiệu quả khi các mô hình thành viên **sai khác nhau** — đây là **Diversity Principle** (Krogh & Vedelsby, 1995): generalization error của ensemble = error trung bình **trừ đi** độ đa dạng. Khi hai mô hình xung đột, diversity âm, ensemble còn tệ hơn single model.

> Mời bạn **[TÊN P4]** — Track B.

---

## P4 — Slides 13–14 | 1.5 phút

Em là **[TÊN]**.

**Slide 13:** Về mặt lý thuyết, **No Free Lunch** khẳng định không một model family nào tối ưu cho mọi bài toán. Mỗi sản phẩm có phân phối dữ liệu khác nhau — Loan 4.1%, TD 2.3%, Card 1.4% — nên thay vì đặt cược vào một kiến trúc, chúng em chạy song song 7 họ mô hình: từ LogReg bias cao đến MLP variance cao, từ HGB boosting đến XGBoost. **Bias-Variance Decomposition** cho thấy kết hợp các model có profile khác nhau sẽ giảm expected error tổng thể. Dữ liệu siêu mất cân bằng. Chiến lược: **Joint BR cho Loan**, Product-per-model cho Card & TD, ANOVA chọn đặc trưng cho Card.

**Slide 14:** *(chỉ bảng)* Mô hình chốt:
- **Card**: XGBoost + GMM — Lift **7.54x**
- **Loan**: Joint BR-XGBoost + KMeans — AUC-PR **0.2515**
- **TD**: XGBoost + AE_K8 — Lift **4.66x**

So với Tuần 2: tất cả chỉ số đều cải thiện.

> Mời bạn **[TÊN P5]** — Time-Series và Persona Impact.

---

## P5 — Slides 15–16 | 1 phút

Em là **[TÊN]**.

**Slide 15:** *(chỉ bảng)* Thử nghiệm Transformer/GRU vs XGBoost Tabular. Kết quả: Tabular thắng tuyệt đối (AUC-PR 0.25 vs 0.08). Lý do: chuỗi 3-6 tháng quá ngắn. **Giữ XGBoost cho production.**

**Slide 16:** *(chỉ bảng Persona Impact)* Không có variant nào tốt nhất cho tất cả: GMM nhất Track A, KMeans nhất Loan, AE_K8 nhất TD → **chọn Hybrid.**

> Mời bạn **[TÊN P6]** — Decision Layer.

---

## P6 — Slides 17–18 | 1.5 phút

Em là **[TÊN]**.

**Slide 17:** Từ propensity score → offer: **Percentile-Rank chuẩn hóa** giải quyết chênh lệch phân phối giữa 3 sản phẩm. Đây chính là ý tưởng **Stacking** (Wolpert, 1992) — tách biệt tầng base learner chuyên biệt cho từng sản phẩm khỏi tầng meta-learner ra quyết định cuối cùng. Nhờ đó mỗi tầng tối ưu một mục tiêu riêng, không phải trade-off giữa 3 class có base rate lệch nhau. Kết quả: TD 34.85%, Card 33.69%, Loan 27.04%, **No Offer 4.41%** (<5% chuẩn ngân hàng số).

**Slide 18:** Rào chắn cuối: **Risk-Aware Policy Gate**. Fraud flag từ Track A → tạm giữ offer. Chỉ **58 khách hàng (0.04%)** cần manual review. **99.96% tự động** — giảm 95% chi phí vận hành.

> Mời bạn **[TÊN P7]** — Live Demo.

---

## P7 — Slides 19–20 + Demo | 1 phút + 5 phút

Em là **[TÊN]**.

**Slide 19:** Hệ thống demo: 2 chế độ — **theo ID** (tra cứu bất kỳ KH nào từ 150K+, <1 giây) và **ngẫu nhiên** (bốc KH mới tinh). Tích hợp **xAI** tự động giải thích lý do offer, đáp ứng Basel/IFRS.

**Slide 20:** *(chỉ 3 con số)* Ba giá trị cốt lõi: **Cá nhân hóa** (Lift 4.4x–7.5x, ROI +400-750%), **An toàn** (0.04% manual review), **Tự động** (pipeline 24/7).

> Bây giờ em xin chạy demo trực tiếp trên notebook.

---

### 🎯 DEMO (5 phút) — P7 thực hiện

> *Mở `notebooks/00_WEEK4_LIVE_DEMO.ipynb`, chạy từng cell:*

| TT | Thao tác | Nội dung |
|---|---|---|
| 1 | Chạy cell khởi tạo | Load model + data |
| 2 | Case 1: KH **751560** | Demo case trúng Loan 100%, có xAI giải thích |
| 3 | Case 2: KH **509963** | Demo case Fraud → offer bị chặn bởi Risk Gate |
| 4 | Case 3: Dynamic Addition | Nhập ID ngẫu nhiên → hệ thống tự động score + offer |
| 5 | Kết luận | Chốt: pipeline hoàn chỉnh, tự động, sẵn sàng production |

> *Hết demo → cả nhóm đứng dậy.*

Kính thưa thầy và các bạn, đó là toàn bộ hệ thống của Nhóm 30. Chúng em cảm ơn thầy đã lắng nghe và rất mong nhận được góp ý. **Cảm ơn thầy!**

---

## SỐ LIỆU CHỐT CẦN THUỘC

| Số | Ý nghĩa |
|---|---|
| **0.6825** | Track C Silhouette (K=8) |
| **0.9746** | Track A ROC-AUC (MLP+GMM) |
| **0.3625** | Track A AUC-PR |
| **7.54x** | Track B Top 10% Lift (Card) |
| **4.41%** | No Offer (<5%) |
| **0.04%** | Manual Review (58 KH) |
| **< 1 giây** | Dynamic User Addition |
| **29%** | Stack Loss: Precision giảm khi thêm LOF |

## TIMELINE

| Phút | Ai | Việc |
|---|---|---|
| 0:00 | P1 | Bắt đầu |
| 1:30 | P2 | Track C |
| 2:30 | P3 | Track A |
| 4:00 | P4 | Track B |
| 5:30 | P5 | Time-Series + Persona |
| 6:30 | P6 | Decision + Risk |
| 8:00 | P7 | Tổng kết |
| 9:00 | P7 | **BẮT ĐẦU DEMO** |
| 14:00 | ALL | Kết thúc, cảm ơn |
