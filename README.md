# locate-bug-by-TF-IDF-and-DNN

## Bug Localization: TF-IDF VSM and Historical Similarity (Build VSM Notebook)

Notebook này (`Build VSM`) là một phần của dự án định vị bug, tập trung vào việc xây dựng mô hình Không gian Vector (VSM) sử dụng TF-IDF để biểu diễn mã nguồn và truy vấn (báo cáo bug). Ngoài ra, nó kết hợp thông tin từ các báo cáo bug lịch sử có liên quan để cải thiện độ chính xác.

### 1. Giới thiệu

1.  **Vector Space Model (VSM) với TF-IDF:** Biểu diễn các file mã nguồn và báo cáo bug dưới dạng vector dựa trên tần suất và tầm quan trọng của các từ (term) trong chúng. Sự tương đồng giữa báo cáo bug và file mã nguồn được tính bằng độ tương đồng Cosine.
2.  **Tương đồng báo cáo bug lịch sử:** Sử dụng thông tin từ các báo cáo bug trước đó có nội dung tóm tắt tương đồng để xác định các file mã nguồn đã được sửa trong quá khứ.

Kết quả từ hai phương pháp này sau đó được kết hợp để tạo ra danh sách các file có khả năng chứa lỗi được xếp hạng cuối cùng.

### 2. Phương pháp

Notebook này thực hiện các bước sau:

1.  **Tải dữ liệu:** Đọc dữ liệu báo cáo bug đã được tiền xử lý (`ReportPreprocessing...csv`), dữ liệu mã nguồn đã được tiền xử lý (`srcPreprocessing...csv`), và dữ liệu báo cáo bug gốc (`.xlsx`).
2.  **Chuẩn bị dữ liệu truy vấn và Ground Truth:** Trích xuất các từ (đã qua stem) từ tóm tắt và mô tả của báo cáo bug làm truy vấn. Trích xuất danh sách các file đã sửa làm ground truth.
3.  **Xây dựng Chỉ mục Ngược và Từ vựng:** Tạo chỉ mục ngược từ nội dung mã nguồn để tăng hiệu quả tìm kiếm và xây dựng từ vựng chung.
4.  **Xây dựng VSM cho Mã nguồn:** Biểu diễn mỗi file mã nguồn dưới dạng vector TF-IDF thưa sử dụng scheme `ltc` (logarithmic term frequency, inverse document frequency, cosine normalization).
5.  **Tính toán Query Vector:** Biểu diễn mỗi báo cáo bug (truy vấn) dưới dạng vector TF-IDF thưa sử dụng scheme `lnc` (logarithmic term frequency, no inverse document frequency, cosine normalization).
6.  **Tìm kiếm Tương đồng (VSM):** Tính toán độ tương đồng Cosine giữa mỗi query vector và tất cả các document vector (file mã nguồn). Kết quả là danh sách các file mã nguồn được xếp hạng theo độ tương đồng cho mỗi báo cáo bug.
7.  **Phân tích Tương đồng Lịch sử:**
    *   Đối với mỗi báo cáo bug hiện tại, tìm kiếm các báo cáo bug *trước đó* có tóm tắt tương đồng sử dụng TF-IDF và độ tương đồng Cosine trên dữ liệu báo cáo gốc.
    *   Xác định các file mã nguồn đã được sửa trong các báo cáo bug lịch sử có độ tương đồng cao hơn ngưỡng nhất định.
    *   Tính toán điểm "liên quan trung bình" cho mỗi file dựa trên độ tương đồng của các báo cáo bug lịch sử chứa file đó.
8.  **Kết hợp Kết quả:** Kết hợp điểm tương đồng từ VSM và điểm liên quan từ phân tích lịch sử bằng cách sử dụng trọng số (`ALPHA`). Công thức kết hợp: `ALPHA * VSM_similarity + (1-ALPHA) * historical_relevance`.
9.  **Đánh giá:** Tính toán các chỉ số hiệu suất (Top-k Accuracy, Mean Average Precision - MAP, Mean Reciprocal Rank - MRR) cho cả kết quả *trước* và *sau* khi kết hợp để so sánh hiệu quả.

### 3. Yêu cầu

*   Python 3.x
*   Các thư viện Python: `pandas`, `numpy`, `scikit-learn`, `math`, `collections`, `ast`.

### 4. Cách chạy

1.  Clone repository này.
2.  Đảm bảo bạn đã có các file dữ liệu đã được tiền xử lý và file báo cáo gốc trong thư mục hoặc cập nhật đường dẫn chính xác. Các biến đường dẫn được định nghĩa ở đầu notebook:
    ```python
    # Thay đổi các đường dẫn này tùy thuộc vào dữ liệu bạn muốn sử dụng (AspectJ, Tomcat, SWT)
    PRE_PROCESS_SRC_PATH = "./srcPreprocessingaspectj.csv"
    PRE_PROCESS_REPORT_PATH = "./ReportPreprocessingaspectj.csv"
    REPORT_PATH = "./AspectJ.xlsx"
    ```
3.  Cài đặt các thư viện cần thiết:
    ```bash
    pip install pandas numpy scikit-learn
    ```
4.  Mở notebook `Build VSM` bằng Jupyter Notebook hoặc JupyterLab.
5.  Chạy từng cell trong notebook theo thứ tự từ trên xuống dưới.

### 5. Dữ liệu

Notebook này yêu cầu 3 file đầu vào:

*   `srcPreprocessing[project].csv`: Chứa dữ liệu mã nguồn đã được tiền xử lý. Cột quan trọng là `key` (đường dẫn file) và `all_content` (nội dung file đã được tiền xử lý).
*   `ReportPreprocessing[project].csv`: Chứa dữ liệu báo cáo bug đã được tiền xử lý. Các cột quan trọng bao gồm `key` (ID bug), `summary`, `description`, `fixed_files` (ground truth), `report_time`, `pos_tagged_summary`, `pos_tagged_description`.
*   `[project].xlsx`: File Excel gốc chứa dữ liệu báo cáo bug. Cột quan trọng bao gồm `bug_id`, `summary`, `report_time`, `files` (danh sách file đã sửa).

*(Thay `[project]` bằng tên project tương ứng như AspectJ, Tomcat, SWT)*

### 6. Cấu hình

Bạn có thể điều chỉnh các tham số sau trong notebook:

*   **Đường dẫn dữ liệu:** Thay đổi giá trị của `PRE_PROCESS_SRC_PATH`, `PRE_PROCESS_REPORT_PATH`, `REPORT_PATH` để chọn bộ dữ liệu dự án khác (AspectJ, Tomcat, SWT).
*   **ALPHA:** Trọng số dùng để kết hợp điểm VSM và điểm liên quan lịch sử. `ALPHA = 0.8` có nghĩa là điểm VSM đóng góp 80% và điểm lịch sử đóng góp 20% vào điểm cuối cùng. Bạn có thể thay đổi giá trị này (từ 0 đến 1) để thử nghiệm.
*   **Ngưỡng tương đồng lịch sử:** Ngưỡng `threshold` trong hàm `get_related_files` (mặc định 0.2 hoặc 0.05 trong code) để lọc các báo cáo bug lịch sử có tóm tắt đủ tương đồng.
*   **k (cho Top-k):** Số lượng file được xếp hạng cao nhất được xét trong hàm `find_top_k_similar` (mặc định 100) và các hàm đánh giá (mặc định 1, 5, 10, 20).

### 7. Đánh giá

Hiệu suất của mô hình được đánh giá bằng các chỉ số phổ biến trong Information Retrieval và Bug Localization:

*   **Top-k Accuracy:** Tỷ lệ báo cáo bug mà ít nhất một file đúng (trong ground truth) xuất hiện trong top k file được dự đoán.
*   **Mean Average Precision (MAP):** Giá trị trung bình của Average Precision trên tất cả các báo cáo bug. Average Precision đo lường độ chính xác trung bình tại các vị trí mà các file đúng được tìm thấy trong danh sách xếp hạng.
*   **Mean Reciprocal Rank (MRR):** Giá trị trung bình của Reciprocal Rank trên tất cả các báo cáo bug. Reciprocal Rank là nghịch đảo vị trí của file đúng đầu tiên được tìm thấy.

Các chỉ số này được tính cả trước (chỉ VSM) và sau (VSM + Lịch sử) khi kết hợp thông tin lịch sử để cho thấy tác động của việc kết hợp.