# Plan Chuyên Đề Tốt Nghiệp: Phân Tích Dữ Liệu Giao Dịch Open Banking

## Tổng quan
- **Thời gian:** 9 tuần thực nghiệm + 3 tuần viết báo cáo
- **Ngôn ngữ:** Python (pandas, numpy, scikit-learn, statsmodels, prophet)
- **5 chủ đề chính:**
  1. Dự báo số lượng giao dịch/ngày
  2. Dự báo số tiền giao dịch/ngày
  3. Phân tích tương quan giữa số lượng và số tiền
  4. Phân tích ảnh hưởng của ngày lễ
  5. Phát hiện bất thường

## Timeline chi tiết

### Tuần 1-3: Phân tích đề tài, thu thập và tiền xử lý dữ liệu

#### Tuần 1: Phân tích đề tài và nghiên cứu
- Nghiên cứu tài liệu về Open Banking và payment transaction analysis
- Tổng quan các phương pháp: time series forecasting, regression, anomaly detection
- Xác định yêu cầu dữ liệu và format mong muốn
- Lập kế hoạch chi tiết cho từng chủ đề
- Setup môi trường Python và các thư viện cần thiết

**Output:**
- Tài liệu nghiên cứu và tổng hợp
- Kế hoạch chi tiết
- Môi trường Python đã setup

#### Tuần 2: Thu thập dữ liệu
- Thu thập dữ liệu giao dịch từ hệ thống Open Banking (1 năm)
- Kiểm tra format dữ liệu: ngày giờ, người gửi, số tiền, nội dung
- Kiểm tra chất lượng ban đầu: missing values, duplicates, format errors
- Thu thập danh sách ngày lễ (Tết, Quốc khánh, v.v.)

**Output:**
- Dataset gốc (raw data)
- Danh sách ngày lễ
- Báo cáo chất lượng dữ liệu ban đầu

#### Tuần 3: Tiền xử lý dữ liệu và masking
- **Data cleaning:**
  - Xử lý missing values
  - Loại bỏ duplicates
  - Xử lý outliers cơ bản
  - Chuẩn hóa format ngày giờ
  
- **Data masking (bảo vệ danh tính):**
  - Masking thông tin người gửi:
    - Hash hoặc mã hóa ID người gửi (ví dụ: SHA-256 hash)
    - Hoặc thay thế bằng ID giả (pseudonymization)
    - Đảm bảo không thể reverse engineering
  - Kiểm tra tính nhất quán: cùng người gửi phải có cùng masked ID
  - Lưu mapping table (nếu cần) ở nơi an toàn, tách biệt
  
- **Feature engineering:**
  - Aggregate theo ngày: tổng số lượng và tổng số tiền/ngày
  - Tạo features: ngày trong tuần, tháng, quý
  - Tạo biến ngày lễ (dummy: 1 nếu là ngày lễ, 0 nếu không)
  - Tạo lag features (giá trị 1-7 ngày trước)
  
- **EDA cơ bản:**
  - Thống kê mô tả (Mean, Median, Std, Min, Max, Quartiles)
  - Biểu đồ phân bố (Histogram, Boxplot)
  - Biểu đồ time series cơ bản

**Output:**
- Dataset đã làm sạch và masked
- Dataset đã aggregate theo ngày
- Bảng thống kê mô tả
- Các biểu đồ EDA
- Báo cáo tiền xử lý dữ liệu

### Tuần 4: Phân tích tương quan (Chủ đề 3)

**Input:**
- Hai chuỗi thời gian: số lượng giao dịch/ngày và số tiền/ngày

**Công việc:**
- Tính Pearson Correlation coefficient
- Tính Spearman Correlation coefficient
- Vẽ Scatter plot với regression line
- Cross-correlation analysis (lag analysis)
- Vẽ biểu đồ time series chồng lấp

**Output:**
- Hệ số tương quan Pearson và Spearman
- Biểu đồ scatter plot
- Kết quả cross-correlation (optimal lag)
- Báo cáo phân tích tương quan

**Metrics:**
- Pearson correlation coefficient
- Spearman correlation coefficient
- Optimal lag (nếu có)

### Tuần 5: Phân tích ảnh hưởng ngày lễ (Chủ đề 4)

**Input:**
- Dữ liệu giao dịch theo ngày (số lượng và số tiền)
- Danh sách ngày lễ

**Công việc:**
- So sánh thống kê: Mean và Median của ngày lễ vs ngày thường
- Tính % thay đổi
- Kiểm định giả thuyết:
  - t-test (nếu dữ liệu chuẩn)
  - Mann-Whitney U test (nếu dữ liệu không chuẩn)
- Visualization: Boxplot, Bar chart so sánh
- Regression với dummy variable (Linear/Ridge/Lasso)

**Output:**
- Bảng so sánh thống kê (Mean, Median, % thay đổi)
- Kết quả kiểm định (p-value, effect size)
- Biểu đồ so sánh
- Hệ số regression cho biến ngày lễ
- Báo cáo phân tích ảnh hưởng ngày lễ

**Metrics:**
- Mean, Median cho ngày lễ và ngày thường
- % thay đổi
- P-value từ t-test hoặc Mann-Whitney U test
- Regression coefficients

### Tuần 6-7: Dự báo số lượng giao dịch (Chủ đề 1)

#### Tuần 6: Xây dựng và train mô hình
**Input:**
- Chuỗi thời gian: tổng số lượng giao dịch/ngày (train: 80%, test: 20%)

**Công việc:**
- Train các mô hình:
  1. Linear Regression (baseline)
  2. Ridge Regression
  3. Lasso Regression
  4. ARIMA/SARIMA
  5. Prophet
- Tune hyperparameters (nếu cần)
- Dự báo cho tập test

**Output:**
- Các mô hình đã train
- Predictions cho tập test

#### Tuần 7: Đánh giá và so sánh mô hình
**Công việc:**
- Tính metrics: MAE, RMSE, MAPE, R²
- So sánh kết quả: bảng metrics, biểu đồ predictions vs actual
- Chọn mô hình tốt nhất
- Visualization: time series plot, residual plots

**Output:**
- Bảng so sánh metrics của tất cả mô hình
- Biểu đồ so sánh predictions
- Mô hình tốt nhất được chọn
- Báo cáo đánh giá mô hình

**Metrics:**
- MAE, RMSE, MAPE, R² cho mỗi mô hình

### Tuần 8-9: Dự báo số tiền và phát hiện bất thường

#### Tuần 8: Dự báo số tiền giao dịch (Chủ đề 2)
**Input:**
- Chuỗi thời gian: tổng số tiền giao dịch/ngày (train: 80%, test: 20%)

**Công việc:**
- Xử lý dữ liệu (log transformation nếu phân bố lệch)
- Train các mô hình: Linear, Ridge, Lasso, ARIMA/SARIMA, Prophet
- Tính metrics và so sánh
- Chọn mô hình tốt nhất

**Output:**
- Các mô hình đã train
- Bảng so sánh metrics
- Mô hình tốt nhất
- Báo cáo đánh giá

**Metrics:**
- MAE, RMSE, MAPE, R²

#### Tuần 9: Phát hiện bất thường (Chủ đề 5) và tổng hợp
**Phần 1: Phát hiện bất thường**
**Input:**
- Chuỗi thời gian: số lượng giao dịch/ngày và số tiền/ngày

**Công việc:**
- Phát hiện bất thường số lượng:
  1. Z-score method
  2. IQR method
  3. Isolation Forest
- Phát hiện bất thường số tiền (tương tự)
- So sánh kết quả các phương pháp
- Visualization: time series plot với anomalies được đánh dấu

**Output:**
- Danh sách ngày bất thường (anomaly flags)
- Anomaly scores
- Biểu đồ với anomalies được highlight
- Bảng so sánh các phương pháp
- Báo cáo phát hiện bất thường

**Metrics:**
- Số lượng anomalies phát hiện được
- Anomaly scores

**Phần 2: Tổng hợp và chuẩn bị demo**
- Tổng hợp tất cả kết quả
- Tạo visualization tổng hợp
- So sánh tổng thể các chủ đề
- Chuẩn bị demo script
- Review và hoàn thiện code

**Output:**
- Báo cáo tổng hợp kết quả
- Code hoàn chỉnh và có documentation
- Demo script
- Visualization tổng hợp

## Tổng hợp mô hình và phương pháp

### Mô hình dự báo (Chủ đề 1 & 2):
1. Linear Regression
2. Ridge Regression
3. Lasso Regression
4. ARIMA/SARIMA
5. Prophet

**Tổng cộng:** 5 mô hình × 2 chủ đề = 10 mô hình

### Phương pháp phân tích:
- **Tương quan:** Pearson, Spearman, Cross-correlation
- **Kiểm định:** t-test, Mann-Whitney U test
- **Phát hiện bất thường:** Z-score, IQR, Isolation Forest

### Metrics đánh giá:
- **Dự báo:** MAE, RMSE, MAPE, R²
- **Tương quan:** Correlation coefficients
- **Kiểm định:** P-value, Effect size
- **Anomaly:** Anomaly scores, số lượng anomalies

## Deliverables cuối cùng

1. **Code Python:**
   - Data preprocessing và masking scripts
   - Model training và evaluation scripts
   - Analysis scripts cho từng chủ đề
   - Demo script

2. **Kết quả phân tích:**
   - Báo cáo tiền xử lý dữ liệu
   - Kết quả tương quan
   - Kết quả phân tích ngày lễ
   - Kết quả dự báo (2 chủ đề)
   - Kết quả phát hiện bất thường

3. **Visualizations:**
   - Biểu đồ EDA
   - Biểu đồ tương quan
   - Biểu đồ so sánh ngày lễ
   - Biểu đồ dự báo vs thực tế
   - Biểu đồ anomalies

4. **Báo cáo:**
   - Tổng hợp phương pháp
   - Kết quả và đánh giá
   - So sánh mô hình
   - Kết luận và đề xuất

## Lưu ý quan trọng

1. **Dữ liệu:**
   - Tối thiểu 6 tháng, khuyến nghị 1 năm
   - Chia train/test đúng cách (TimeSeriesSplit)
   - Masking data phải đảm bảo không thể reverse engineering

2. **Metrics:**
   - Báo cáo cả Mean và Median khi có outliers
   - So sánh nhiều metrics để đánh giá toàn diện

3. **Mô hình:**
   - So sánh với baseline (Linear Regression hoặc Naive)
   - Giải thích tại sao chọn mô hình tốt nhất

4. **Documentation:**
   - Comment code rõ ràng
   - Ghi chú các quyết định trong quá trình làm
   - Lưu mapping table (nếu có) ở nơi an toàn, tách biệt

