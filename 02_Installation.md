# Cài đặt Prometheus
## **1) Cài đặt Prometheus trên CentOS 7**
- **B1 :** Tắt **SELinux** :
    ```
    # sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
    # sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config
    # setenforce 0
    ```
- **B2 :** Download file package **Prometheus** `2.29.1` :
    ```
    # cd /opt/
    # wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
    ```
- **B3 :** Giải nén file package **Prometheus** vừa download :
    ```
    # tar -xzvf prometheus-2.29.1.linux-amd64.tar.gz
    ```
- **B4 :** Tạo user để chạy dịch vụ **Prometheus** :
    ```
    # useradd --no-create-home --shell /bin/false prometheus
    ```
- **B5 :** Tạo thư mục cấu hình và data cho **Prometheus**. Sau đó phân quyền owner tương ứng :
    ```
    # mkdir /etc/prometheus
    # mkdir /var/lib/prometheus
    # chown prometheus:prometheus /etc/prometheus
    # chown prometheus:prometheus /var/lib/prometheus
    ```
- **B6 :** Copy 2 file binary `prometheus` và `promtool` vào thư mục `/usr/local/bin/` :
    ```
    # cp prometheus-2.29.1.linux-amd64/prometheus /usr/local/bin/
    # cp prometheus-2.29.1.linux-amd64/promtool /usr/local/bin/
    # chown prometheus:prometheus /usr/local/bin/prometheus
    # chown prometheus:prometheus /usr/local/bin/promtool
    # chmod +x /usr/local/bin/prometheus /usr/local/bin/promtool
    ```
- **B7 :** Copy 2 thư mục `consoles` và `consoles_libraries` vào thư mục `/etc/prometheus/` :
    ```
    # cp -r prometheus-2.29.1.linux-amd64/consoles /etc/prometheus
    # cp -r prometheus-2.29.1.linux-amd64/console_libraries /etc/prometheus
    # chown -R prometheus:prometheus /etc/prometheus/consoles
    # chown -R prometheus:prometheus /etc/prometheus/console_libraries
    ```
- **B8 :** Tạo file cấu hình **Prometheus** cơ bản. Với cấu hình cơ bản này thì **Prometheus** đầu tiên nó sẽ tự monitor chính nó :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thêm vào nội dung sau :
        ```yaml
        global:
          scrape_interval: 10s

        scrape_configs:
          - job_name: 'prometheus_master'
            scrape_interval: 5s
            static_configs:
              - targets: ['localhost:9090']
        ```
- **B9 :** Khởi tạo một file khởi động dịch vụ **Prometheus** trên `systemd` :
    ```
    # vi /etc/systemd/system/prometheus.service
    ```
    - Thêm vào nội dung sau :
        ```ini
        [Unit]
        Description=Prometheus
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=prometheus
        Group=prometheus
        Type=simple
        ExecStart=/usr/local/bin/prometheus \
        --config.file /etc/prometheus/prometheus.yml \
        --storage.tsdb.path /var/lib/prometheus/ \
        --web.console.templates=/etc/prometheus/consoles \
        --web.console.libraries=/etc/prometheus/console_libraries

        [Install]
        WantedBy=multi-user.target
        ```
- **B10 :** Khởi động dịch vụ **Prometheus** :
    ```
    # systemctl daemon-reload
    # systemctl start prometheus
    # systemctl enable prometheus
    ```
- **B11 :** Cấu hình `firewalld` cho phép port `9090` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=9090/tcp
    # firewall-cmd --reload
    ```
- **B12 :** Kiểm tra lại trạng thái dịch vụ :
    ```
    # systemctl status prometheus
    ```
    <img src=https://i.imgur.com/KxBTuiv.png>

- **B13 :** Lúc này dịch vụ **Prometheus** đã chạy và bạn có thể truy xuất dịch vụ **Prometheus** qua giao diện quản lý đơn giản của nó tại port mặc định TCP `9090`. Truy cập `http://<ip-server>:9090` :

    <img src=https://i.imgur.com/36B6DH9.png>