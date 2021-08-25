# Query
## **1) Giới thiệu**
- **Prometheus** cung cấp một ngôn ngữ truy vấn chức năng được gọi là **PromQL** - ***Prometheus Query Language*** cho phép người dùng select và aggregate dữ liệu time series theo thời gian thực. Kết quả của biểu thức truy vấn có thể hiển thị dưới dạng biểu đồ (graph) hoặc dưới dạng bảng dữ liệu (tabular data) trên WebUI của **Prometheus**, hoặc được sử dụng bởi các hệ thống bên ngoài thông qua HTTP API.
- **VD :** Thực hiện truy vấn `node_cpu_seconds_total{instance="192.168.5.10:9100"}` để hiển thị tất cả các metric `node_cpu_seconds_total` có lable là `instance="192.168.5.10:9100"` :
    
    <img src=https://i.imgur.com/WYAWUDt.png>

## **2) Các kiểu biểu thức truy vấn**
### **2.1) Instant vector**
- Là một chuỗi các single sample của time series có cùng timestamp

    <img src=https://i.imgur.com/BMXiuWN.png>

### **2.2) Range vector**
- Là một chuỗi của time series gồm một loạt các data points theo thời gian

    <img src=https://i.imgur.com/l5b1DVT.png>

### **2.3) Scalar**
- Là một giá trị numeric floating point đơn giản

    <img src=https://i.imgur.com/QptM809.png>

> Tùy vào từng trường hợp sử dụng, như khi vẽ biểu đồ hay khi cần output của 1 truy vấn mà chỉ 1 trong các loại trên sẽ thích hợp. **VD :** một truy vấn trả về một **instant vector** là kiểu duy nhất có thể vẽ được biểu đồ.
## **3) Quy định về string**
- Tham khảo thêm : https://prometheus.io/docs/prometheus/latest/querying/basics/#literals
## **4) Time series Selectors**
