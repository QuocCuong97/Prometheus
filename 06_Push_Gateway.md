# Pushgateway
## **1) Giới thiệu**
- **GitHub :** https://github.com/prometheus/pushgateway
- **Pushgateway** là một service trung gian cho phép bạn push metrics từ các jobs mà không thể scraped, push time series từ short-lived service-level batch jobs tới các service trung gian để **Prometheus** có thể scrape.
- Đôi khi bạn có các applications hoặc jobs không trực tiếp export metrics. Các ứng dụng đó cũng không được thiết kế cho việc đó (ví dụ các batch jobs), hoặc bạn có thể lựa chọn không lấy các metric trực tiếp từ các ứng dụng của mình, khi đó bạn sẽ cần tới **Pushgateway** để làm trung gian.

    <p align=center><img src=https://i.imgur.com/jlyg7gz.png width=70%></p>

## **2) Cài đặt và cấu hình Pushgateway**
### **2.1) Cài đặt Pushgateway**
- **B1 :** Copy link download `pushgateway` từ [trang chủ](https://prometheus.io/download/) của **Prometheus** :

    <img src=https://i.imgur.com/hpQZw04.png>

- **B2 :** Download `pushgateway` :
    ```
    # wget https://github.com/prometheus/pushgateway/releases/download/v1.4.1/pushgateway-1.4.1.linux-amd64.tar.gz
    ```
- **B3 :** Giải nén file vừa download :
    ```
    # tar -xzvf pushgateway-1.4.1.linux-amd64.tar.gz
    ```
- **B4 :** Tạo user mới cho `pushgateway` :
    ```
    # useradd --no-create-home --shell /bin/false pushgateway
    ```
- **B5 :** Chuyển file binary `pushgateway` tới thư mục `/usr/local/bin` và phân quyền :
    ```
    # mv pushgateway-1.4.1.linux-amd64/pushgateway /usr/local/bin/
    # chown pushgateway:pushgateway /usr/local/bin/pushgateway
    ```
- **B6 :** Tạo mới file `systemd` cho `pushgateway` :
    ```
    # vi /etc/systemd/system/pushgateway.service
    ```
    - Thêm vào nội dung sau :
        ```ini
        [Unit]
        Description=Prometheus Pushgateway
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=pushgateway
        Group=pushgateway
        Type=simple
        ExecStart=/usr/local/bin/pushgateway

        [Install]
        WantedBy=multi-user.target
        ```
- **B7 :** Cấu hình `firewalld` cho phép port `9091` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=9091/tcp
    # firewall-cmd --reload
    ```
- **B8 :** Khởi động `node_exporter` :
    ```
    # systemctl daemon-reload
    # systemctl start pushgateway
    # systemctl enable pushgateway
    ```
- **B9 :** Kiểm tra trạng thái dịch vụ :
    ```
    # systemctl status pushgateway
    ```
    <img src=https://i.imgur.com/KjSvfg1.png>

- **B10 :** Test dịch vụ : push một metric đơn giản lên **Pushgateway** và xem nó ở trên trình duyệt :
    ```
    # echo "second_metric 99" | curl --data-binary @- http://192.168.5.30:9091/metrics/job/some_job
    ```
    > Trong đó, `192.168.5.30` là IP của **Pushgateway**.
    - Kiểm tra trên trình duyệt : `http://192.168.5.30:9091` :

        <img src=https://i.imgur.com/CLxF1gh.png>

### **2.2) Cấu hình trên Prometheus**
- **B1 :** Chỉnh sửa file cấu hình của **Prometheus** :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thêm đoạn sau vào `scrape_configs` :
        ```yaml
        - job_name: 'Pushgateway'
          honor_labels: true
          scrape_interval: 10s
          static_configs:
            - targets: ['192.168.5.30:9091']
        ```
- **B2 :** Khởi động lại dịch vụ `prometheus` :
    ```
    # systemctl restart prometheus
    ```
- **B3 :** Kiểm tra target `pushgateway` trên webUI của **Prometheus** :
    
    <img src=https://i.imgur.com/Q0zrD3l.png>

- **B4 :** Truy vấn các metric có trong **Pushgateway** trên **Prometheus** :
    
    <img src=https://i.imgur.com/0kIAvec.png>
## **3) Push metrics**
- Tìm hiểu thêm : https://prometheus.io/docs/instrumenting/pushing/