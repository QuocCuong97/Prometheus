# Lưu trữ với Prometheus
- **Prometheus** đi lèm một time-series database local on-disk, tuy nhiên cũng có thể tùy biến tích hợp với các hệ thống lưu trữ metric khác.
## **1) Local storage**
### **1.1) Nguyên lý hoạt động**
- Database của **Prometheus** được lưu trữ theo định dạng tùy chỉnh, hiệu quả cao trong local storage.
- Các sample được group thành các block theo mỗi 2 giờ. Mỗi block 2 giờ đó bao gồm một thư mục chứa các thư mục con chứa tất cả các time-series sample cho khoảng thời gian đó, một file metadata và một 1 file index (nó sẽ đánh chỉ số metric names và label cho time series trong thư mục). Các sample trong thư mục con được nhóm chung với nhau thành 1 hoặc nhiều file segment file với dung lượng mặc định là `512MB`/file. Khi bị xóa qua các API, các bản ghi bị xóa được lưu trữ riêng trong các tomstone file ( thay vì xóa dữ liệu ngay lập tức khỏi các segment).
- Block hiện tại để lưu trữ các sample được lưu trong bộ nhớ mà không tồn tại lâu dài. Nó được bảo vệ chống lại crash bởi write-ahead log (WAL) có thể được dùng lại khi server Prometheus restart. Các write-ahead log file được lưu trữ trong thư mục wal trong các segment `128MB`. Các file này chứa raw data chưa được nén, do đó chúng nặng hơn đáng kể so với các block file thông thường. **Prometheus** sẽ giữ lại tối thiểu 3 file write-ahead log. Các server có lưu lượng tải cao có thể giữ lại hơn 3 file write-ahead log để lưu trữ ít nhất 2 giờ raw data.
- Thư mục dữ liệu của server **Prometheus** (`/var/lib/prometheus/`) trông như sau :
    ```
    ├── 01FDCEP86MZ1AS9H75FS3B7MZR
    │   ├── chunks
    │   │   └── 000001
    │   ├── index
    │   ├── meta.json
    │   └── tombstones
    ...
    ...
    ├── 01FEA941GQW8Q38NTA482ADPXK
    │   ├── chunks
    │   │   └── 000001
    │   ├── index
    │   ├── meta.json
    │   └── tombstones
    ├── 01FEBVPCEPTRE9VVK8M4E2X7CR
    │   ├── chunks
    │   │   └── 000001
    │   ├── index
    │   ├── meta.json
    │   └── tombstones
    ├── chunks_head
    │   ├── 000014
    │   └── 000015
    ├── lock
    ├── queries.active
    └── wal
        ├── 00000058
        ├── 00000059
        ├── 00000060
        └── checkpoint.00000057
            └── 00000000
    ```
- Lưu ý rằng hạn chế của local storage là nó không có cluster hoặc được replicated. Do đó, nó không thể tùy ý mở rộng hoặc lưu trữ lâu dài khi đối mặt với sự cố hỏng disk hay node, cũng giống như việc quản lý bất kỳ database đơn lẻ nào. Việc sử dụng RAID được đề xuất để đảm bảo tính khả dụng của bộ nhớ và các snapshot được khuyến khích dùng cho backup. Với một kiến trúc phù hợp, vẫn có thể lưu trữ data vài năm với local storage.
- Ngoài ra, các external storage cũng có thể sử dụng thông qua các API đọc/ghi từ xa. Cần đánh giá cẩn thận đối với các hệ thống này vì chúng khác nhau nhiều về độ bền bỉ, hiệu năng.
- Tham khảo thêm về định dạng file `TSDB` của **Prometheus** tại [đây](https://github.com/prometheus/prometheus/blob/release-2.29/tsdb/docs/format/README.md).
### **1.2) Nén dữ liệu**
- Các block dữ liệu 2 giờ ban đầu thậm chí có thể được nén thành các block lưu trữ lâu hơn.
- Việc nén sẽ tạo ra các block chứa dữ liệu dài đến `10%` thời gian lưu trữ, hoặc `31 ngafy`, tùy điều kiện nào đến trước.
### **1.3) Các lưu ý khi vận hành local storage**
- **Prometheus** có một số tùy chọn để cấu hình local storage. Các tùy chọn quan trọng là  :
    - `--storage.tsdb.path` : nơi **prometheus** lưu trữ dữ liệu, mặc định là `data/.`
    - `--storage.tsdb.retention.time` : thời gian xóa dữ liệu cũ. Mặc định là `15d`.
    - `--storage.tsdb.retention.size` : số byte lưu trữ tối đa cần dữ lại. Dữ liệu cũ nhất sẽ bị xóa trước. Mặc định là `0` hoặc disabled. Các đơn vị hỗ trợ : `B`, `KB`, `MB`, `GB`, `TB`, `PB`, `EB`. **VD :** `512MB`.
    - `--storage.tsdb.wal-compression` : cho phép nén file `WAL`.
- **Prometheus** lưu trữ trung bình chỉ `1 - 2 bytes` cho mỗi sample. Do đó cần tính toán khả năng lưu trữ của server **Prometheus**, có thể sử dụng công thức sau :
    ```
    need_disk_space = keep_time_seconds * ingested_samples_per_second * bytes_per_sample
    ```
    - Để giảm số `ingested_samples`, có thể giảm số time series muốn scrape hoặc tăng khoảng thời gian interval khi scrape.
- Nếu local storage gặp sự cố gì, giải pháp tốt nhất là tắt **Prometheus** server, sau đó xóa toàn bộ thư mục lưu trữ. Cũng có thể thử xóa các block riêng lẻ hoặc thư mục WAL để giải quyết sự cố. Điều này có nghĩa rằng sẽ mất khoảng 2 giờ dữ liệu cho mỗi block bị xóa.
- Tóm lại, local storage không phải giải pháp lưu trữ lâu dài, các giải pháp external cung cấp khả năng lưu trữ lâu hơn cùng độ bền hơn của dữ liệu.
## **2) Tích hợp Remote storage**
### **2.1) Tổng quan**
- **Prometheus** tích hợp hệ thống remote storage theo 3 cách :
    - **Prometheus** có thể write các sample mà nó lấy được vào một remote URL (POST API) ở định dạng được chuẩn hóa.
    - **Prometheus** có thể nhận dữ liệu từ các **Prometheus** server khác ở định dạng chuẩn hóa.
    - **Prometheus** có thể read các sample từ một remote URL (GET API) từ xa ở định dạng chuẩn hóa

        <img src=https://i.imgur.com/tL3dKdg.png>
- Tham khảo thêm : https://prometheus.io/docs/prometheus/latest/storage/#overview
### **2.2) Các Remote storage có thể tích hợp**
- Tham khảo : https://prometheus.io/docs/operating/integrations/#remote-endpoints-and-storage
