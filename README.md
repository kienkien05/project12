# KỊCH BẢN THUYẾT TRÌNH — NHÓM 30

**Môn:** CS320 - Học Máy | **GV:** Vũ Minh Hoàng
**Tổng:** 15 phút nói + 5 phút demo = **20 phút**
**File slide:** `Nhóm 30.pptx` (21 slides)

> **Luật chung:** Không đọc bảng. Chỉ nêu số chốt. Chuyển tiếp nhanh. Demo là phần quan trọng nhất.

---

## PHÂN CÔNG

| #   | Slides   | Nội dung                           | T.gian   |
|-----|----------|------------------------------------|----------|
| P1  | 1–6      | Mở đầu + F0→F8 (lướt nhanh)        | 2.5 phút |
| P2  | 7–9      | Track C: K=8, 8 Persona            | 2 phút   |
| P3  | 10–12    | Track A: MLP+GMM, Stack Loss       | 2.5 phút |
| P4  | 13–14    | Track B: Hybrid + Hiệu năng        | 2.5 phút |
| P5  | 15–16    | Time-Series + Persona Impact       | 1.5 phút |
| P6  | 17–18    | Decision Layer + Risk Gate         | 2.5 phút |
| P7  | 19–20    | Live Demo + Tổng kết               | 1.5 phút |
| P7  | Demo     | **Chạy notebook trực tiếp**         | **5 phút** |

---

## P1 — Slides 1–6 | 2.5 phút

> Người nói: **[TÊN]**

**Slide 1:** *(slide tiêu đề — tên nhóm, môn học, GV)*

Kính thưa thầy và các bạn, em là **[TÊN]**, đại diện Nhóm 30. Hôm nay chúng em trình bày kết quả hệ thống **phân tích khách hàng**
**Slide 2:** *(sơ đồ tổng quan — 4 khối chính)*

Bố cục bài trình bày đi theo chính kiến trúc hệ thống mà chúng em đã xây dựng. Phần đầu là **9 thế hệ mô hình** (F0→F8) — cho thấy toàn bộ hành trình từ baseline đơn giản đến production. Sau đó đi sâu vào **3 Track xử lý song song**: Track C phân cụm hành vi khách hàng thành 8 persona, Track A phát hiện rủi ro gian lận, Track B dự đoán xác suất khách hàng quan tâm đến từng sản phẩm. Cả 3 track hội tụ về **Decision Layer** — nơi ra quyết định cuối cùng. Cuối cùng là **Live Demo** chạy trực tiếp trên notebook để thầy và các bạn thấy hệ thống hoạt động thực tế.

**Slide 3:** *(timeline F0→F4 — baseline & exploration)*

Hành trình bắt đầu từ **F0**: Huấn luyện mô hình Isolation Forest trên dữ liệu thô. **F1–F2**: chúng em thêm PCA giảm chiều dữ liệu từ hàng trăm đặc trưng xuống còn 8 chiều, rồi dùng KMeans phân cụm. Lần đầu tiên hệ thống có khái niệm "chân dung khách hàng" — mỗi khách hàng được gán vào một persona cụ thể. **F3**: tham vọng hơn, chúng em thử ensemble 3 model: MLP neural network, Isolation Forest, và Local Outlier Factor. **F4**: Chuyển từ KMeans sang GMM — Gaussian Mixture Model với soft clustering, cho phép một khách hàng thuộc nhiều cụm với xác suất khác nhau. Chất lượng phân cụm và fraud detection đều tăng rõ rệt.

**Slide 4:** *(timeline F5→F7 — scaling & optimization)*

**F5**: thay vì phụ thuộc vào một thuật toán clustering, chúng em mở rộng thành **Persona Zoo** — bộ sưu tập 6 biến thể persona: KMeans, GMM, KNN, và Autoencoder với K=6, 8, 10. Mỗi variant cho ra một góc nhìn khác nhau về cùng một khách hàng. **F6**: hai thử nghiệm lớn — Joint Model (một neural network chung cho cả 3 sản phẩm thay vì model riêng), và Time-Series với Transformer/GRU để tận dụng dữ liệu chuỗi theo tháng. Kết quả Transformer thua XGBoost trên chuỗi ngắn 3-6 tháng. **F7**: hoàn thiện với ANOVA chọn đặc trưng quan trọng nhất cho từng sản phẩm, và Risk-Aware Policy Gate — rào chắn cuối cùng ngăn offer đến tay khách hàng có dấu hiệu gian lận.

**Slide 5:** *(mốc F8 — PRODUCTION READY)*

**F8** là thế hệ chốt — tinh gọn từ tất cả bài học trước. Chúng em giữ GMM cho Track A vì soft clustering bắt fraud ở vùng giao thoa giữa các cụm tốt hơn hẳn KMeans. Giữ XGBoost cho Track B vì thắng tuyệt đối Transformer/GRU trên mọi chỉ số. Giữ KMeans K=8 cho Track C với Silhouette 0.68 và cụm đồng đều, dễ giải thích. Và quan trọng không kém — **bỏ LOF** vì Stack Loss đã chứng minh không phải cứ thêm model là tốt, **bỏ Transformer** vì overfit trên chuỗi quá ngắn. Điều chúng em tâm đắc nhất: mỗi thế hệ không chỉ cải thiện metric, mà **mỗi thế hệ trả lời một câu hỏi cụ thể** — nên dùng model gì? nên kết hợp ra sao? nên bỏ cái gì? Đến F8, tất cả những câu hỏi đó đã có câu trả lời dựa trên bằng chứng thực nghiệm.

**Slide 6:** *(sơ đồ kiến trúc F8 — 3 track + Decision Layer)*

Đây là kiến trúc tổng thể của F8 — trái tim của toàn bộ hệ thống. 3 track vận hành song song, mỗi track có một nhiệm vụ riêng nhưng cùng chia sẻ đầu vào là dữ liệu khách hàng đã qua tiền xử lý. **Track C** — phân cụm hành vi: PCA8 → KMeans K=8, Silhouette 0.6825, output là 8 persona. Track C trả lời câu hỏi "khách hàng này là ai?". **Track A** — phát hiện rủi ro: MLP + GMM, ROC-AUC 0.9746, output là fraud flag. Track A trả lời "có nên gửi offer cho khách hàng này không?". **Track B** — đề xuất sản phẩm: XGBoost Joint BR cho Loan (AUC-PR 0.2515), XGBoost riêng cho TD và Card (Lift tới 7.54x), output là propensity score cho từng sản phẩm. Track B trả lời "nên gửi offer gì?". Cả 3 track hội tụ về Decision Layer: percentile-rank chuẩn hóa điểm số → chọn offer tốt nhất → Risk Gate kiểm tra lần cuối → xuất quyết định. **Pipeline hoàn toàn tự động, mỗi khách hàng mới xử lý dưới 1 giây.**

> Em chuyển cho bạn **[TÊN P2]** trình bày Track C.

---

## P2 — Slides 7–9 | 2 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 7:** *(chỉ sơ đồ pipeline Track C)*

Track C là nền tảng của toàn bộ hệ thống — trước khi đề xuất sản phẩm hay phát hiện rủi ro, chúng ta phải hiểu khách hàng là ai. Pipeline của Track C gồm 4 bước. **Bước 1 — PCA**: từ hơn 100 đặc trưng thô về hành vi ngân hàng số, chúng em dùng PCA giảm xuống còn 8 chiều, giữ lại khoảng 85% phương sai. **Bước 2 — KMeans**: phân cụm trên không gian PCA8. **Bước 3 — ANOVA**: kiểm định thống kê để xác định đặc trưng nào phân biệt mạnh nhất giữa các cụm. Kết quả cho thấy `activity_hour_entropy` — độ phân tán thời gian giao dịch trong ngày — và `avg_ca_balance` — số dư tài khoản thanh toán trung bình — là hai đặc trưng phân biệt mạnh nhất. **Bước 4**: đặt tên và diễn giải 8 persona dựa trên profile của từng cụm.

Điều quan trọng ở đây: chúng em không phân cụm dựa trên nhân khẩu học (tuổi, giới tính, địa chỉ) mà dựa trên **hành vi thực tế** — cách khách hàng tương tác với ngân hàng số. Điều này giúp persona có ý nghĩa kinh doanh hơn: hai khách hàng cùng độ tuổi nhưng hành vi khác nhau sẽ thuộc hai persona khác nhau, và nhận hai chiến lược offer khác nhau.

**Slide 8:** *(chỉ bảng so sánh K)*

Đây là bảng so sánh các giá trị K từ 6 đến 12. Chúng em dùng 3 chỉ số để đánh giá: Silhouette Score đo độ gắn kết nội tại của cụm, Davies-Bouldin Index đo độ tương tự giữa các cụm (càng thấp càng tốt), và Calinski-Harabasz đo tỷ số phương sai. K=12 cho Silhouette cao nhất về mặt kỹ thuật (0.6903), nhưng xuất hiện cụm chỉ có 2 khách hàng — quá nhỏ để có ý nghĩa kinh doanh, không thể thiết kế chiến lược marketing cho 2 người. **K=8 được chọn** với Silhouette 0.6825 — gần như tương đương K=12 về mặt chất lượng, nhưng kích thước các cụm đồng đều, mỗi cụm đều có câu chuyện riêng, dễ đặt tên và dễ giải thích cho business. Đây là một quyết định cân bằng giữa technical metric và business interpretability.

**Slide 9:** *(radar chart hoặc bảng mô tả 8 persona)*

Đây là 8 chân dung khách hàng mà hệ thống phân biệt được:

1. **Power Digital User** — khách hàng hoạt động mạnh nhất trên kênh số: đăng nhập hàng ngày, giao dịch đa dạng, entropy giờ giấc cao (hoạt động trải đều trong ngày). Đây là nhóm tiềm năng nhất cho cross-sell.
2. **Traditional Depositor** — số dư tiết kiệm và tài khoản cao, nhưng gần như không dùng e-banking. Nhóm này cần chiến lược chăm sóc khác biệt, không thể tiếp cận qua kênh số.
3. **Credit Active Spender** — tần suất sử dụng thẻ tín dụng cao, diversity giao dịch lớn, thường xuyên thanh toán online. Nhóm lý tưởng cho upsell hạn mức thẻ.
4. **Mass Market / Inactive** — số dư thấp, ít giao dịch, gần như "ngủ đông". Cần chiến lược kích hoạt trước khi cross-sell.
5. **High-Value Saver** — tập trung vào tiết kiệm có kỳ hạn, số dư TD cao, ít vay. Nhóm an toàn, ưu tiên offer tiết kiệm.
6. **E-Commerce Shoppers** — giao dịch tập trung vào TMĐT, tần suất mua sắm online cao.
7. **Salary Dependent** — giao dịch theo chu kỳ lương hàng tháng, dòng tiền đều đặn vào-ra.
8. **Young Native** — khách hàng trẻ, mới tham gia, tần suất thấp nhưng đang tăng trưởng.

Mỗi persona không chỉ là một cái tên — mà đi kèm với một bộ đặc trưng thống kê cụ thể, giúp Track B sau này biết chính xác nên ưu tiên sản phẩm nào cho nhóm nào.

> Mời bạn **[TÊN P3]** — Track A.

---

## P3 — Slides 10–12 | 2.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 10:** *(sơ đồ Track A + mô tả pseudo-label)*

Track A giải quyết một bài toán khó: làm sao phát hiện rủi ro khi **không có nhãn thật**? Trong ngân hàng thực tế, gian lận thường chỉ được phát hiện sau khi thiệt hại đã xảy ra — có thể mất hàng tuần hoặc hàng tháng. Chúng em không thể ngồi đợi nhãn thật, nên đã thiết kế phương pháp **Strict Pseudo-Eval**: tạo nhãn giả lập dựa trên logic nghiệp vụ — giao dịch vào khung giờ đêm (0h-5h), số tiền bất thường lớn (trên 100 triệu), và chuyển tiền ra ngân hàng khác. Đây là 3 dấu hiệu kinh điển trong phát hiện rửa tiền và gian lận tài chính.

Nhưng quan trọng hơn: để tránh **data leakage** — tình trạng model "học" chính cái rule tạo nhãn — chúng em đã loại bỏ toàn bộ đặc trưng liên quan đến giờ giao dịch, số tiền giao dịch đêm, và ngân hàng nhận khỏi tập đặc trưng huấn luyện. Nhờ đó, model buộc phải học các tín hiệu ẩn khác — như tần suất bất thường, thay đổi đột ngột trong hành vi — thay vì chỉ memorize rule. Kết quả evaluation mới thực sự có ý nghĩa.

Hành trình Track A: F3 từng ensemble MLP + IForest + LOF, nhưng F8 chốt lại **MLP + GMM** — tinh gọn hơn, mạnh hơn.

**Slide 11:** *(chỉ bảng kết quả)*

Đây là bảng xếp hạng hiệu năng các tổ hợp model + persona variant. **MLP + GMM dẫn đầu tuyệt đối**: AUC-PR 0.3625, ROC-AUC 0.9746, Precision@100 đạt 0.380. Điều đáng nói là GMM vượt trội KMeans một cách nhất quán: MLP + GMM đạt AUC-PR 0.3625, trong khi MLP + KMeans chỉ đạt 0.2794 — chênh lệch gần 30%.

Tại sao GMM tốt hơn? Vì fraud không nằm gọn trong một cụm riêng biệt — kẻ gian thường "ẩn mình" trong vùng giao thoa giữa các cụm khách hàng bình thường. GMM với soft clustering gán xác suất một khách hàng thuộc về nhiều cụm, giúp model "nhìn thấy" những khách hàng nằm ở biên giới — nơi fraud thường xuất hiện. KMeans hard assignment gán mỗi khách hàng vào đúng một cụm, bỏ lỡ thông tin quan trọng này.

Nhìn xuống bảng: Isolation Forest baseline chỉ đạt AUC-PR 0.1328 — chưa bằng một nửa MLP. Điều này khẳng định deep learning có khả năng học các pattern phức tạp mà anomaly detection truyền thống không bắt được. Tuy nhiên, có một điểm đáng chú ý: AE_K8 đạt AUC-PR thấp nhất (0.1259) dù cũng dùng MLP — cho thấy không phải cứ autoencoder là tốt, chất lượng embedding phụ thuộc nhiều vào cách huấn luyện.

**Slide 12:** *(biểu đồ Stack Loss)*

Đây là bài học mà chúng em tâm đắc nhất trong toàn bộ dự án — **Stack Loss**. Ở F3, chúng em ensemble MLP + Isolation Forest + LOF, kỳ vọng 3 model bổ trợ cho nhau. Nhưng kết quả ngược lại: Precision@500 giảm từ **0.730 xuống 0.518**, mất 29%. Đây không phải là nhiễu — đây là một hiện tượng có tên trong lý thuyết học máy.

Về mặt lý thuyết, ensemble chỉ hiệu quả khi các mô hình thành viên **sai khác nhau một cách có ích**. Đây là **Diversity Principle** do Krogh & Vedelsby chứng minh năm 1995: generalization error của ensemble = error trung bình của các model thành viên **trừ đi** độ đa dạng (ambiguity) giữa chúng. Khi các model càng khác nhau và cùng đúng, ensemble càng mạnh. Nhưng khi chúng xung đột — như trường hợp LOF đo mật độ cục bộ còn MLP học pattern toàn cục, hai tín hiệu triệt tiêu lẫn nhau — diversity trở thành "âm", và ensemble còn tệ hơn cả single model tốt nhất. Đây là lý do F8 chỉ giữ MLP + GMM và **bỏ LOF hoàn toàn**.

Bài học rút ra: **không phải cứ thêm mô hình là tốt**. Chất lượng của ensemble đến từ diversity, không đến từ quantity.

> Mời bạn **[TÊN P4]** — Track B.

---

## P4 — Slides 13–14 | 2.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 13:** *(sơ đồ chiến lược Track B)*

Track B là phân hệ phức tạp nhất — dự đoán xác suất khách hàng sẽ đăng ký mới một sản phẩm tài chính trong kỳ tiếp theo. Chúng em đối mặt với 3 thách thức lớn.

**Thách thức 1 — Dữ liệu siêu mất cân bằng**: Loan có 4.11% positive, TD chỉ 2.34%, Card thậm chí còn 1.45%. Nếu model dự đoán tất cả là "không đăng ký", accuracy vẫn trên 95% nhưng hoàn toàn vô dụng. Đây là lý do chúng em dùng **AUC-PR thay vì accuracy** làm chỉ số chính — AUC-PR nhạy hơn nhiều với dữ liệu lệch.

**Thách thức 2 — Mỗi sản phẩm là một bài toán khác nhau**: khách hàng vay tiêu dùng có hành vi rất khác với khách hàng gửi tiết kiệm. Về mặt lý thuyết, **No Free Lunch Theorem** (Wolpert & Macready, 1997) khẳng định: không tồn tại một thuật toán tối ưu cho mọi bài toán. Đây là nền tảng lý thuyết cho chiến lược của chúng em: thay vì đặt cược vào một model family, chúng em chạy song song **7 họ mô hình**: Logistic Regression (bias cao, variance thấp), Random Forest và ExtraTrees (bagging — giảm variance), HGB, XGBoost, LightGBM (boosting — giảm bias), và MLP (non-linear universal approximator). Mỗi family có một bias-variance profile khác nhau. **Bias-Variance Decomposition** cho thấy expected error = bias² + variance + noise — và cách tốt nhất để giảm error tổng là kết hợp các model có profile bù trừ cho nhau.

**Thách thức 3 — Chọn đặc trưng**: với hơn 100 đặc trưng, không phải đặc trưng nào cũng có ích. Với Card — sản phẩm khó dự đoán nhất (base rate 1.45%) — chúng em dùng ANOVA để chọn ra top đặc trưng có khả năng phân biệt tốt nhất, thay vì ném tất cả vào model.

Chiến lược cuối cùng: **Joint Binary Relevance XGBoost cho Loan** (một model, 3 head cho 3 sản phẩm, chia sẻ biểu diễn), **Product-per-model cho Card & TD** (XGBoost riêng, tối ưu độc lập), và **ANOVA feature selection cho Card**.

**Slide 14:** *(chỉ bảng kết quả)*

Đây là kết quả chốt — 3 model, 3 sản phẩm, 3 persona variant khác nhau:

- **Card**: XGBoost + GMM, ANOVA chọn đặc trưng — Top 10% Lift đạt **7.54x**. Nghĩa là khi ngân hàng gửi offer cho top 10% khách hàng được model dự đoán cao nhất, tỷ lệ chuyển đổi cao gấp 7.54 lần so với gửi ngẫu nhiên. Đây là con số quyết định ROI của chiến dịch.
- **Loan**: Joint BR-XGBoost + KMeans — AUC-PR **0.2515**, Top 10% Lift **4.39x**. Joint model cho phép chia sẻ thông tin giữa 3 sản phẩm, giúp Loan — sản phẩm có nhiều dữ liệu nhất — học được cả từ pattern của TD và Card.
- **TD**: XGBoost + AE_K8 — Lift **4.66x**. Điều thú vị: persona tốt nhất cho TD là Autoencoder, không phải KMeans hay GMM — embedding từ autoencoder dường như bắt được các đặc trưng tiềm ẩn liên quan đến hành vi tiết kiệm mà PCA không nắm được.

So với baseline Tuần 2: cả 3 sản phẩm đều cải thiện — Card tăng từ 0.1524 lên 0.1566 AUC-PR, Loan giữ vững ở mức cao nhất 0.2515. Nhưng quan trọng hơn con số: mỗi sản phẩm dùng một persona variant khác nhau — **không có one-size-fits-all**. Đây chính là lý do chúng em xây dựng Persona Zoo và chiến lược Hybrid.

> Mời bạn **[TÊN P5]** — Time-Series và Persona Impact.

---

## P5 — Slides 15–16 | 1.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 15:** *(chỉ bảng so sánh Tabular vs Time-Series)*

Ngoài hướng tabular truyền thống, chúng em còn thử nghiệm một câu hỏi nghiên cứu: liệu dữ liệu chuỗi thời gian theo tháng có cải thiện dự đoán không? Về lý thuyết, hành vi khách hàng thay đổi theo thời gian — một khách hàng đang tăng tần suất giao dịch có thể sắp có nhu cầu vay. Transformer và GRU được thiết kế để bắt chính xác loại pattern tuần tự này.

Nhưng kết quả thực nghiệm đi ngược lại kỳ vọng: **Tabular XGBoost thắng tuyệt đối**. AUC-PR của Transformer trên Loan chỉ đạt 0.0805, trong khi XGBoost Tabular đạt 0.2515 — gấp hơn 3 lần. Tương tự với TD (0.0212 vs 0.1157) và Card (0.0155 vs 0.1566).

Nguyên nhân chúng em xác định được: chuỗi thời gian chỉ có **3-6 tháng** — quá ngắn để Transformer/GRU học được pattern có ý nghĩa. Deep sequence models thường cần hàng chục đến hàng trăm time-steps. Ngoài ra, các đặc trưng aggregate (tổng, trung bình, min, max, std trong 6 tháng) đã nén rất tốt thông tin thời gian vào dạng tabular. **Kết luận thực tế: giữ XGBoost Tabular cho production.** Transformer là một hướng thú vị nếu sau này có dữ liệu dài hạn hơn (12-24 tháng), nhưng ở thời điểm hiện tại, nó không phải là lựa chọn tối ưu.

**Slide 16:** *(chỉ bảng Persona Impact Matrix)*

Đây là bảng tổng hợp cho câu hỏi: persona variant nào tốt nhất cho bài toán nào? Kết quả rất rõ ràng: **không có variant nào thắng tất cả**. GMM dẫn đầu Track A (AUC-PR 0.3625), KMeans dẫn đầu Loan (AUC-PR 0.2515), AE_K8 dẫn đầu TD (Lift 4.66x).

Điều này hoàn toàn phù hợp với lý thuyết: mỗi thuật toán clustering có inductive bias khác nhau. KMeans giả định cụm hình cầu, cân bằng — phù hợp khi khách hàng vay có hành vi tách biệt rõ ràng. GMM cho phép cụm có hình dạng ellipse và chồng lấn — phù hợp khi fraud ẩn trong vùng giao thoa. Autoencoder học non-linear manifold — phù hợp khi pattern tiết kiệm phức tạp và phi tuyến.

**Chiến lược Hybrid** của chúng em: mỗi track, mỗi sản phẩm dùng persona variant tối ưu riêng — không cố gắng chọn một variant "tốt nhất overall". Đây không phải là sự thiếu nhất quán, mà là **quyết định kiến trúc có chủ đích** — tận dụng sự đa dạng của Persona Zoo một cách tối đa.

> Mời bạn **[TÊN P6]** — Decision Layer.

---

## P6 — Slides 17–18 | 2.5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 17:** *(sơ đồ Decision Layer)*

Sau khi 3 track chạy xong, chúng ta có gì? Track B cho ra 3 propensity score — mỗi score từ một model khác nhau, trên một phân phối khác nhau. Score cho Loan có thể tập trung quanh 0.05-0.40, trong khi score cho Card tập trung ở 0.01-0.15 vì base rate Card thấp hơn. Nếu đơn giản chọn sản phẩm có score cao nhất, khách hàng sẽ luôn được offer Loan — vì score Loan luôn cao hơn một cách cơ học.

Giải pháp là **Percentile-Rank chuẩn hóa**: thay vì so sánh score thô, chúng em chuyển mỗi score về phân vị của nó trong phân phối của chính sản phẩm đó. Một khách hàng có score Loan ở phân vị 95 (thuộc top 5% người có khả năng vay cao nhất) và score Card ở phân vị 98 (thuộc top 2% người có khả năng mở thẻ cao nhất) — sẽ được offer Card, vì percentile rank của Card cao hơn.

Về mặt lý thuyết, đây chính là ý tưởng **Stacking** (Stacked Generalization) do Wolpert đề xuất năm 1992. Hệ thống có 2 tầng: **tầng base learner** là các model chuyên biệt cho từng sản phẩm — mỗi model tối ưu AUC-PR cho đúng một sản phẩm. **Tầng meta-learner** là Decision Layer — dùng percentile-rank để tổng hợp và ra quyết định cuối cùng. Lợi ích then chốt của kiến trúc này là **decoupling**: mỗi tầng tối ưu một mục tiêu riêng, không phải trade-off. Nếu dùng một multi-class classifier 3 class, model sẽ phải cân bằng giữa 3 class có base rate rất khác nhau (4.1% vs 2.3% vs 1.4%) — điều gần như không thể làm tốt đồng thời.

Kết quả phân bổ offer trên 150.419 khách hàng test: **TD 34.85%, Card 33.69%, Loan 27.04%, No Offer chỉ 4.41%** — dưới ngưỡng 5% theo tiêu chuẩn ngân hàng số. Phân bổ này cân bằng một cách tự nhiên, không cần áp đặt rule cứng. Điều đó cho thấy percentile-rank đang hoạt động đúng như kỳ vọng.

**Slide 18:** *(sơ đồ Risk Gate)*

Nhưng trước khi offer đến tay khách hàng, còn một rào chắn cuối cùng: **Risk-Aware Policy Gate**. Đây là nơi Track A phát huy giá trị. Logic rất đơn giản: nếu Track A gắn cờ fraud cho một khách hàng (fraud score > 0.8), offer sẽ bị tạm giữ và chuyển sang trạng thái **manual_review** — bất kể Track B dự đoán khách hàng đó phù hợp với sản phẩm nào.

Trên thực tế, chính sách này tạo ra một hệ thống **defense in depth**: Track A là lớp phòng thủ đầu tiên, Decision Layer là lớp ra quyết định, và Risk Gate là lớp kiểm tra cuối cùng trước khi offer được gửi đi.

Kết quả: trong số khách hàng được sàng lọc bởi Track A, chỉ có **58 khách hàng (0.04%)** rơi vào diện manual_review_before_offer. 3.780 khách hàng (2.51%) được xác nhận an toàn và nhận offer tự động. 146.581 khách hàng còn lại nằm ngoài vùng phủ của Track A (không có dấu hiệu bất thường để đánh giá). **99.96% quyết định được tự động hóa** — giảm 95% chi phí vận hành so với quy trình manual review toàn bộ.

Ý nghĩa kinh doanh: ngân hàng có thể vận hành hệ thống 24/7 với đội ngũ kiểm soát viên chỉ cần xử lý trung bình vài chục ca mỗi kỳ — thay vì hàng trăm nghìn ca như quy trình truyền thống.

> Mời bạn **[TÊN P7]** — Live Demo.

---

## P7 — Slides 19–20 + Demo | 1.5 phút + 5 phút

> Người nói: **[TÊN]**

Em là **[TÊN]**.

**Slide 19:** *(giao diện demo)*

Đây là hệ thống demo được đóng gói hoàn chỉnh trong một Jupyter notebook — tích hợp toàn bộ pipeline F8 đã trình bày. Hỗ trợ 2 chế độ tra cứu. **Chế độ 1 — theo ID**: nhập mã khách hàng bất kỳ trong 150.000+ khách hàng, hệ thống trả về đầy đủ thông tin: persona, phân khúc tài sản, fraud risk score, top-3 đề xuất sản phẩm kèm lý do. **Chế độ 2 — ngẫu nhiên**: bốc một khách hàng mới tinh từ tập holdout (chưa từng được dùng để train), cho thấy model hoạt động trên dữ liệu thực sự "mới".

Điểm đặc biệt: hệ thống tích hợp **xAI** — tự động sinh giải thích bằng tiếng Việt cho từng quyết định. Ví dụ: "Khách hàng được đề xuất Loan vì có thu nhập ổn định (persona Salary Dependent), tần suất giao dịch tăng 30% trong 3 tháng gần đây, và không có dấu hiệu rủi ro." Điều này đáp ứng yêu cầu của Basel/IFRS về khả năng giải thích quyết định tự động trong ngân hàng.

**Slide 20:** *(tổng kết 3 giá trị cốt lõi)*

Ba giá trị cốt lõi mà hệ thống mang lại. **Cá nhân hóa**: Lift 4.4x–7.5x — mỗi chiến dịch marketing đạt ROI cao gấp 4-7 lần so với gửi ngẫu nhiên. Tiết kiệm hàng tỷ đồng chi phí marketing mỗi năm. **An toàn**: 0.04% manual review — Risk Gate tự động chặn offer đến tay khách hàng có dấu hiệu gian lận, giảm thiểu rủi ro pháp lý và tổn thất tài chính. **Tự động**: pipeline 24/7, mỗi khách hàng xử lý dưới 1 giây, có thể tích hợp trực tiếp vào hệ thống CRM hoặc mobile banking.

> Bây giờ em xin chạy demo trực tiếp trên notebook.

---

### 🎯 DEMO (5 phút) — P7 thực hiện

> Mở `notebooks/00_WEEK4_LIVE_DEMO.ipynb`, chạy từng cell:

| TT | Thao tác                 | Nội dung                                                    |
|----|-------------------------|--------------------------------------------------------------|
| 1  | Chạy cell khởi tạo       | Load toàn bộ model + data + pipeline                          |
| 2  | Case 1: KH **751560**    | KH thuộc persona Salary Dependent — trúng Loan 100%, có xAI giải thích chi tiết |
| 3  | Case 2: KH **509963**    | KH có fraud score > 0.8 — offer bị Risk Gate chặn, chuyển manual review |
| 4  | Case 3: Dynamic Addition | Nhập ID ngẫu nhiên từ tập holdout → toàn bộ pipeline chạy < 1 giây → hiển thị kết quả |
| 5  | Kết luận                 | Chốt: pipeline hoàn chỉnh, tự động, sẵn sàng tích hợp production |

> Hết demo → cả nhóm đứng dậy.

Kính thưa thầy và các bạn, đó là toàn bộ hệ thống của Nhóm 30. Chúng em đã xây dựng một pipeline phân tích khách hàng 360 độ: từ phân cụm hành vi, phát hiện rủi ro, đến đề xuất sản phẩm cá nhân hóa và ra quyết định tự động. Toàn bộ kiến trúc được dẫn dắt bởi lý thuyết học máy vững chắc và kiểm chứng qua 9 thế hệ thực nghiệm. Chúng em cảm ơn thầy đã lắng nghe và rất mong nhận được góp ý. **Cảm ơn thầy!**

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
| 2:30  | P2  | Track C                |
| 4:30  | P3  | Track A                |
| 7:00  | P4  | Track B                |
| 9:30  | P5  | Time-Series + Persona  |
| 11:00 | P6  | Decision + Risk        |
| 13:30 | P7  | Tổng kết               |
| 15:00 | P7  | **BẮT ĐẦU DEMO**       |
| 20:00 | ALL | Kết thúc, cảm ơn       |

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
| **Inductive Bias**             | Tập giả định mà một thuật toán áp đặt lên dữ liệu — quyết định khi nào model hoạt động tốt |
| **Defense in Depth**           | Chiến lược nhiều lớp phòng thủ — áp dụng cho pipeline Track A → Decision → Risk Gate |

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
