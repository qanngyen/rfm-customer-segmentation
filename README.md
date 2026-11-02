# Phân khúc khách hàng E-commerce bằng RFM và K-Means

Dự án phân tích dữ liệu giao dịch thương mại điện tử bằng Python: chuyển dữ liệu hóa đơn thành dữ liệu khách hàng theo mô hình **RFM**, phân cụm bằng **K-Means**, rồi kiểm định sự khác biệt giữa các cụm bằng **ANOVA**.

## Mục lục

- [Tổng quan](#tổng-quan)
- [Cấu trúc dự án](#cấu-trúc-dự-án)
- [Yêu cầu và cài đặt](#yêu-cầu-và-cài-đặt)
- [Nguồn dữ liệu](#nguồn-dữ-liệu)
- [Dữ liệu đầu vào](#dữ-liệu-đầu-vào)
- [Cách chạy](#cách-chạy)
- [Chi tiết từng notebook](#chi-tiết-từng-notebook)
- [Lưu ý](#lưu-ý)

## Tổng quan

Quy trình gồm 3 bước chạy lần lượt:

```
ecommerce.csv ──► processing.ipynb ──► clustering.ipynb ──► anova.ipynb
 (hóa đơn thô)    (làm sạch + RFM)     (K-Means, k = 3)     (kiểm định cụm)
```

| Chỉ số | Ý nghĩa | Cách tính |
|---|---|---|
| **Recency** | Số ngày kể từ lần mua gần nhất | Ngày tham chiếu (ngày cuối cùng trong dữ liệu + 1) trừ ngày mua cuối của khách |
| **Frequency** | Số lần mua hàng | Số hóa đơn khác nhau (`InvoiceNo`) của mỗi khách |
| **Monetary** | Tổng giá trị chi tiêu | Tổng của `Quantity × UnitPrice` |

## Cấu trúc dự án

```
.
├── imports.py            # Khai báo thư viện dùng chung cho các notebook
├── processing.ipynb      # Tiền xử lý, làm sạch, tạo RFM, loại outlier, chuẩn hóa
├── clustering.ipynb      # Chọn k (Elbow), phân cụm K-Means, trực quan hóa, mô tả cụm
├── anova.ipynb           # Kiểm định ANOVA và boxplot theo cụm
├── Data/
│   ├── ecommerce.csv              # [đầu vào] dữ liệu giao dịch gốc (tải từ Kaggle)
│   ├── ecommerce_rfm.csv          # [sinh ra] RFM chưa loại outlier
│   ├── ecommerce_rfm_clean.csv    # [sinh ra] RFM sau khi loại outlier
│   └── ecommerce_rfm_scaled.csv   # [sinh ra] RFM đã chuẩn hóa
└── Cluster/
    ├── all_clusters.csv           # [sinh ra] toàn bộ khách hàng kèm nhãn cụm
    └── cluster_0.csv, cluster_1.csv, cluster_2.csv   # [sinh ra] từng cụm riêng
```

## Yêu cầu và cài đặt

- Python 3.9 trở lên
- Jupyter Notebook hoặc JupyterLab

Thư viện sử dụng (khai báo trong `imports.py`):

| Thư viện | Dùng để |
|---|---|
| `pandas` | Đọc và xử lý dữ liệu |
| `seaborn`, `matplotlib` | Trực quan hóa |
| `scikit-learn` | `StandardScaler`, `KMeans` |
| `scipy` | `f_oneway` (ANOVA một yếu tố) |
| `statsmodels` | `pairwise_tukeyhsd` (đã import, xem phần Lưu ý) |

Cài đặt:

```bash
pip install pandas seaborn matplotlib scikit-learn scipy statsmodels jupyter
```

## Nguồn dữ liệu

Dự án sử dụng bộ dữ liệu **[E-Commerce Data](https://www.kaggle.com/datasets/carrie1/ecommerce-data)** trên Kaggle, là bản phân phối của bộ **Online Retail** thuộc UCI Machine Learning Repository (Daqing Chen, 2015, DOI: [10.24432/C5BW33](https://doi.org/10.24432/C5BW33)).

- Nội dung: toàn bộ giao dịch từ 01/12/2010 đến 09/12/2011 của một cửa hàng bán lẻ trực tuyến đăng ký tại Anh, chuyên bán quà tặng.
- Quy mô: 541.909 dòng, 8 cột (`InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`).
- Cách dùng: tải file `data.csv` từ Kaggle, đổi tên thành `ecommerce.csv` và đặt vào thư mục `Data/`.

Tài liệu tham khảo:

> Chen, D. (2015). *Online Retail* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33

## Dữ liệu đầu vào

File `ecommerce.csv` (đọc với `encoding='ISO-8859-1'`) trong dự án này chỉ dùng các cột sau:

| Cột | Mô tả |
|---|---|
| `InvoiceNo` | Mã hóa đơn |
| `InvoiceDate` | Thời điểm lập hóa đơn |
| `CustomerID` | Mã khách hàng |
| `Quantity` | Số lượng sản phẩm |
| `UnitPrice` | Đơn giá |

## Cách chạy

1. Tải dữ liệu từ [Kaggle](https://www.kaggle.com/datasets/carrie1/ecommerce-data), đổi tên `data.csv` thành `ecommerce.csv` và đặt vào thư mục `Data/`. Tạo sẵn thư mục `Cluster/`.
2. **Sửa lại các đường dẫn** trong notebook (hiện đang là đường dẫn tuyệt đối trên máy tác giả, dạng `D:\Uni\...`). Xem phần Lưu ý.
3. Chạy lần lượt các notebook, mỗi notebook chạy từ trên xuống dưới:
   1. `processing.ipynb`
   2. `clustering.ipynb`
   3. `anova.ipynb`

Cả ba notebook đều bắt đầu bằng `from imports import *`, nên cần mở Jupyter ở thư mục chứa `imports.py`.

## Chi tiết từng notebook

### 1. `processing.ipynb`: Tiền xử lý dữ liệu

1. **Kiểm tra tổng quan**: `head`, `info`, `describe`, giá trị thiếu, kiểu dữ liệu không đồng nhất, dòng trùng lặp, giá trị âm.
2. **Làm sạch**:
   - Xóa các dòng thiếu `CustomerID`.
   - Xóa các dòng trùng lặp hoàn toàn.
   - Ép `InvoiceNo` về chuỗi, chuyển `InvoiceDate` sang `datetime`.
   - Giữ lại các dòng có `UnitPrice > 0` và `Quantity > 0`.
3. **Chuyển hóa đơn thành dữ liệu khách hàng**: tạo `TotalPrice`, tính Recency / Frequency / Monetary cho từng `CustomerID`, lưu `ecommerce_rfm.csv`.
4. **Kiểm tra và loại outlier**: vẽ phân phối từng biến RFM, phát hiện outlier bằng quy tắc **IQR** (1.5 × IQR), loại lần lượt trên cả ba biến, lưu `ecommerce_rfm_clean.csv`.
5. **Chuẩn hóa**: `StandardScaler` trên ba biến RFM, lưu `ecommerce_rfm_scaled.csv`.

### 2. `clustering.ipynb`: Phân cụm

1. **Chọn số cụm**: chạy K-Means với k từ 1 đến 10, vẽ biểu đồ **Elbow** (SSE/inertia).
2. **Phân cụm** với **k = 3** (`random_state=42`), gán nhãn `Cluster`, lưu `all_clusters.csv` và từng file `cluster_<id>.csv`.
3. **Trực quan hóa**: scatter 3D trong không gian RFM và scatter theo từng cặp biến.
4. **Phân tích cụm**: số lượng và tỷ lệ khách hàng mỗi cụm (biểu đồ cột và biểu đồ tròn), thống kê mô tả (`describe`) và histogram phân phối RFM của từng cụm.

### 3. `anova.ipynb`: Kiểm định khác biệt giữa các cụm

1. Với mỗi biến Recency, Frequency, Monetary, chạy **ANOVA một yếu tố** (`f_oneway`) giữa các cụm, ngưỡng ý nghĩa **p < 0.05**.
2. Vẽ **boxplot** của từng biến RFM theo cụm để nhìn trực quan sự khác biệt.

## Lưu ý

- **Đường dẫn cứng**: các notebook đang dùng đường dẫn tuyệt đối kiểu Windows (`D:\Uni\TheThirdYear\...`). Nên đổi sang đường dẫn tương đối, ví dụ:
  ```python
  from pathlib import Path
  DATA_DIR = Path("Data")
  CLUSTER_DIR = Path("Cluster")
  df = pd.read_csv(DATA_DIR / "ecommerce.csv", encoding="ISO-8859-1")
  ```
- **Tạo thư mục trước**: `to_csv` không tự tạo thư mục, nên `Data/` và `Cluster/` phải tồn tại sẵn.
- **Dữ liệu chuẩn hóa**: ANOVA và phân cụm đều chạy trên giá trị đã chuẩn hóa (z-score). Khi diễn giải kết quả theo đơn vị thật (ngày, số lần, tiền), cần đối chiếu với `ecommerce_rfm_clean.csv` hoặc đảo ngược chuẩn hóa.
- **k = 3 đặt cố định**: biểu đồ Elbow chỉ để tham khảo, giá trị k được gán thủ công trong code.
- **Thứ tự mục trong `processing.ipynb`**: đánh số tiêu đề chưa khớp thứ tự thực thi (mục "4. Chuyển đổi dữ liệu hóa đơn sang dữ liệu khách hàng" nằm trước mục "3. Kiểm tra ngoại lai"). Cần chạy đúng thứ tự từ trên xuống.
- **Tukey HSD**: `pairwise_tukeyhsd` đã được import trong `imports.py` nhưng chưa được dùng. Có thể bổ sung để so sánh từng cặp cụm sau khi ANOVA cho kết quả có ý nghĩa.
- **Nhãn đơn vị tiền**: biểu đồ cụm 2 ghi đơn vị "VND" ở trục Monetary, trong khi các cụm khác không ghi đơn vị. Nên thống nhất theo đơn vị tiền thực của dữ liệu.
