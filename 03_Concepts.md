# Một số khái niệm quan trọng trong Prometheus
## **1) Exporter**
- Mặc định thì **Prometheus** chỉ thu thập các số liệu về chính nó (ví dụ số request nhận, memory usabel,...). Để có thể mở rộng thu thập từ các nguồn khác thì cần phải sử dụng **Exporter** và các công cụ khác để tạo metric và thu thập nó.
- **Exporter** đưuọc phát triển bởi **Prometheus** và cộng đồng, cung cấp mọi thứ thông tin về cơ sở hạ tầng, cơ sở dữ liệu, web server, hệ thống tin nhắn, API,...
- Một vài các **exporter** tiêu biểu:
    - **node_exporter :** tạo ra các số liệu về hạ tầng, bao gồm CPU, memoru, disk usage cũng như các số liệu về disk I/O và network.
    - **blackbox_exporter :** tạo ra các số liệu từ các đầu dò như HTTP, HTTPs để xác định tính khả dụng của các endpoint, thời gian phản hồi,...
    - **mysqld_exporter :** tập hợp các số liệu liên quan đến mysql server
    - **rabbitmq_exporter :** output của exporter này liên quan tới **RabbitMQ** , bao gồm số lượng các message được publish, số message sẵn sàng để gửi, kích thước các gói tin trong queue.
    - **nginx-vts-exporter:** cung cấp các số liệu về NGINX server sử dụng module VTS bao gồm số lượng kết nối mở, số lượng phản hồi được gửi và tổng kích thước của các gói tin gửi và nhận.
    > Tham khảo thêm về [Exporter](https://prometheus.io/docs/instrumenting/exporters/)
## **2) Metric**
- Định dạng chung của một metric có dạng:
    ```
    <metric name>{<label name>=<label value>, ...}
    ```
- Mỗi exporter sẽ thu thập và phơi data ra ngoài để prometheus sever có thể pull về qua giao thức http.
- Các metric này có thể xem trực tiếp tại địa chỉ: `http://ip-exporter:port/metrics`, ví dụ một đoạn sau :
    ```
    # TYPE node_filesystem_avail_bytes gauge
    node_filesystem_avail_bytes{device="/dev/vda1",fstype="ext4",mountpoint="/"} 1.7972371456e+10
    node_filesystem_avail_bytes{device="rootfs",fstype="rootfs",mountpoint="/"} 1.7972371456e+10
    node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 4.60668928e+08
    node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run/user/1000"} 1.03927808e+08
    # HELP node_filesystem_device_error Whether an error occurred while getting statistics for the given device.
    # TYPE node_filesystem_device_error gauge
    node_filesystem_device_error{device="/dev/vda1",fstype="ext4",mountpoint="/"} 0
    node_filesystem_device_error{device="rootfs",fstype="rootfs",mountpoint="/"} 0
    node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 0
    node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/run/user/1000"} 0
    # HELP node_filesystem_files Filesystem total file nodes.
    ```
    > Dấu `#` là các dòng chú thích.
- Có bốn kiểu metric được sử dụng trong prometheus:
    - **Counter :** là một số bộ đếm tích lũy, được đặt về `0` khi restart. Ví dụ, có thể dùng counter để đếm số request được phục vụ, số lỗi, số task hoàn thành,... Không sử dụng cho các metric có gia trị giảm như số tiến trình đang chạy. Trong trường hợp đó, ta có thể sử dụng **gauge**.
    - **Gauge :** đại diện cho số liệu duy nhất, nó có thể lên hoặc xuống, thường được sử dụng cho các giá trị đo.
    - **Histogram :** lấy mẫu quan sát (thường là những thứ như là thời lượng yêu cầu, kích thước pahnr hồi). Nó cũng cung cấp tổng của các giá trị đó.
    - **Summary :** tương tự histogram, nó cung cấp tổng số các quan sát và tổng các giá trị đó, nó tính toàn số lượng có thể cấu hình qua sliding time window (cửa sổ trượt).
## **3) Data Model**
- Về cơ bản thì **Prometheus** lưu trữ tất cả các dữ liệu dưới dạng time-series.
### **3.1) Metric names and labels**
- Mỗi time series được xác định duy nhất bởi **metric name** của nó và một cặp key-value đi kèm gọi là **label** (optional)
- **Metric name** chỉ định tính năng của hệ thống mà nó đo được (**VD :** `http_requests_total` là tổng số các request HTTP đã nhận). Nó có thể chứa các chữ số và ký tự ASCII, cũng như dấu "`_`" và dấu "`:`". Nó phải match với regex `[a-zA-Z_:][a-zA-Z0-9_:]*` .
    > ***Lưu ý*** : Dấu "`:`" dành riêng cho các rule do người dùng xác định. Chúng không được sử dụng bởi các **exporter** hoặc direct instrumentation.
- **Labels** cho phép **Prometheus's dimensional data model** : bất kỳ tổ hợp **label** nào cho cùng một **metric name** xác định một phiên bản cụ thể của nó (**VD :** tất cả các HTTP request sử dụng phương thức POST tới `/api/tracks`). Ngôn ngữ truy vấn (***query language***) cho phép filter và tổng hợp dựa trên các thông tin này. Thay đổi giá trị **label**, hoặc xóa nó, sẽ tạo ra một time series mới.
- **Label name** có thể chứa các ký tự, số ASCII, cũng như dấu "`_`". Nó phải match với regex `[a-zA-Z_][a-zA-Z0-9_]` . **Label name** bắt đầu bằng "`_`" được dành riêng cho mục đích sử dụng nội bộ. **Label** mà không có giá trị đi kèm thì sẽ coi là không tồn tại.
### **3.2) Samples**
- **Samples** tạo ra dữ liệu time-series (có thể coi như 1 point). Mỗi **sample** bao gồm :
    - 1 giá trị `float64`
    - Mốc thời gian gắn với nó chính xác tới milisecond
### **3.3) Notation**
- Đưa ra **metric name** và tập các **label**, time series thường được xác định bằng cách sử dụng **notation** này :
    ```
    <metric name>{<label name>=<label value>, ...}
    ```
- **VD :** một time series với **metric name** là `api_http_requests_total` và 2 **label** `method="POST"` và `handler="/messages"` có thể được viết như sau :
    ```
    api_http_requests_total{method="POST", handler="/messages"}
    ```
## **4) Job và Instance**
- **Prometheus** quy định, 1 endpoint có thể gọi là một **instance**, thường tương ứng với một ***single process*** . Một tập hợp các **instance** có cùng mục đích được gọi là 1 **job**.
- **VD :** một API server job có 4 replicated instances :
    - job: `api-server` :
        - instance 1: `1.2.3.4:5670`
        - instance 2: `1.2.3.4:5671`
        - instance 3: `5.6.7.8:5670`
        - instance 4: `5.6.7.8:5671`