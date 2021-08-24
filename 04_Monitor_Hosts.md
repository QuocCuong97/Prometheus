# Sử dụng Prometheus để monitor Linux Host
## **Mô hình**
- **Prometheus Server** : `192.168.5.60`
- **CentOS 7 Target** : `192.168.5.10`
## **Các bước thực hiện**
### **Cài đặt `node_exporter` trên host CentOS 7**
- **B1 :** Lấy link download `node_exporter` từ [trang chủ](https://prometheus.io/download/) :

    <img src=https://i.imgur.com/x4g4Kuv.png>

- **B2 :** Download `node_exporter` :
    ```
    # wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
    ```
- **B3 :** Giải nén file vừa download :
    ```
    # tar -xzvf node_exporter-1.2.2.linux-amd64.tar.gz
    ```
- **B4 :** Tạo mới user cho `node_exporter` :
    ```
    # useradd --no-create-home --shell /bin/false node_exporter
    ```
- **B5 :** Chuyển file binary `node_exporter` tới thư mục `/usr/local/bin` :
    ```
    # mv node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin/
    ```
- **B6 :** Tạo mới file `systemd` cho `node_exporter` :
    ```
    # vi /etc/systemd/system/node_exporter.service
    ```
    - Thêm vào nội dung sau :
        ```ini
        [Unit]
        Description=Node Exporter
        After=network.target

        [Service]
        User=node_exporter
        Group=node_exporter
        Type=simple
        ExecStart=/usr/local/bin/node_exporter

        [Install]
        WantedBy=multi-user.target
        ```
- **B7 :** Cấu hình `firewalld` cho phép port `9100` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=9100/tcp
    # firewall-cmd --reload
    ```
- **B8 :** Khởi động `node_exporter` :
    ```
    # systemctl daemon-reload
    # systemctl start node_exporter
    # systemctl enable node_exporter
    ```
- **B9 :** Kiểm tra thử API của `node_exporter` trên trình duyệt xem đã truy cập được chưa :
    ```
    http://192.168.5.10:9100/metrics
    ```
    <img src=https://i.imgur.com/lTQhojK.png>
### **Cấu hình trên Prometheus server**
- **B1 :** Chỉnh sửa file `prometheus.yml` :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thay đổi nội dung cấu hình : thêm `job_name` monitor `node_exporter` :
        ```yaml
        global:
          scrape_interval: 10s

        scrape_configs:
          - job_name: 'prometheus_master'
            scrape_interval: 5s
            static_configs:
              - targets: ['localhost:9090']
          - job_name: 'node_exporter_centos7'
            scrape_interval: 5s
            static_configs:
              - targets: ['192.168.5.10:9100']
        ```
        - Trong đó :
            - `job_name`: Tên của job
            - `scrape_intervals`: Khoảng thời gian **prometheus** sẽ pull metrics từ target
            - `targets`: Là mục tiêu (bao gồm host và port) mà **prometheus** tiến hành scrape dữ liệu
- **B2 :** Khởi động lại dịch vụ `prometheus` :
    ```
    # systemctl restart prometheus
    ```
- **B3 :** Truy cập **Prometheus** trên trình duyệt, kiểm tra việc monitor host :
    ```
    http://192.168.5.60:9090
    ```
    - Chọn **Status** -> **Targets** :

        <img src=https://i.imgur.com/vb5HghI.png>

    - Node đã được thêm thành công :

        <img src=https://i.imgur.com/N7tFE9B.png>

    - Kiểm tra metrics bằng cách nhập thông tin query `node_memory_MemFree_bytes` -> ***Execute*** :

        <img src=https://i.imgur.com/mXwxHEH.png>

    - Metric được query cùng giá trị tại thời điểm hiện tại :

        <img src=https://i.imgur.com/0eg4MCN.png>

    - Chọn tab **Graph** để thấy biểu đồ của metric :

        <img src=https://i.imgur.com/EnnmN0J.png>