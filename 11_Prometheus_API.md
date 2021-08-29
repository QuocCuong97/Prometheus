# Prometheus API
## **1) Management API**
- **Prometheus** cung cấp một bộ management API để tạo điều kiện cho tự động hóa và tích hợp .
### **1.1) Healthcheck**
- Endpoint : `GET /-/healthy`. Kết quả sẽ luôn trả về code `200` nếu server **Prometheus** hoạt động bình thường.
- **VD :**
    ```
    # curl -s -i -X GET http://192.168.5.60:9090/-/healthy
    HTTP/1.1 200 OK
    Date: Sun, 29 Aug 2021 15:36:43 GMT
    Content-Length: 23
    Content-Type: text/plain; charset=utf-8

    Prometheus is Healthy.
    ```
### **1.2) Readiness check**
- Endpoint : `GET /-/ready`. Kết quả sẽ luôn trả về code `200` khi **Prometheus** sẵn sàng phản hồi truy vấn.
- **VD :**
    ```
    # curl -s -i -X GET http://192.168.5.60:9090/-/ready
    HTTP/1.1 200 OK
    Date: Sun, 29 Aug 2021 15:39:38 GMT
    Content-Length: 21
    Content-Type: text/plain; charset=utf-8

    Prometheus is Ready.
    ```
### **1.3) Reload**
- Endpoint: `PUT  /-/reload` hoặc `POST /-/reload`. Endpoint này sẽ reload file cấu hình và các rule file. Nó bị tắt theo mặc định vào có thể được bật thông qua option : `--web.enable-lifecycle`.
### **1.4) Quit**
- Endpoint: `PUT  /-/quit` hoặc `POST /-/quit`. Endpoint này sẽ thực hiện graceful shutdown tiến trình **Prometheus**. Nó bị tắt mặc định và có thể được bật thông qua option `--web.enable-lifecycle`.
## **2) Query API**
- HTTP API ổn định hiện tại có thể truy cập dưới dạng `/api/v1` trên Prometheus server. Bất kỳ một non-breaking additions nào sẽ được thêm vào dưới endpoint.
- Tìm hiểu thêm : 
    - https://github.com/trangnth/ghichep-prometheus/blob/master/Doc/09.%20api.md
    - https://prometheus.io/docs/prometheus/latest/querying/api/
## **3) Secure API bằng Basic Authetication**
- Tìm hiểu thêm : https://github.com/trangnth/ghichep-prometheus/blob/master/Doc/10.%20securing-prometheus-api.md