# Alertmanager
## **1) Tổng quan về Alerting**
- **Alerting** với **Prometheus** được chia làm hai phần. **Alerting rules** trong **Prometheus server** gửi alerts tới **Alertmanager**. **Alertmanager** sẽ quản lý việc cảnh báo, bao gồm silencing, inhibition, aggregation và gửi cảnh báo qua các phương thức như email, on-call notification systems, and chat platforms.

    <p align=center><img src=https://i.imgur.com/s27c7Q0.png width=60%></p>
    
- Các bước chính để set up alert và notifications là :
    - Setup và cấu hình **Alertmanager**
    - Cấu hình **Prometheus** để giao tiếp với **Alertmanager**
    - Tạo alerting rules trong **Prometheus**.
## **2) Alertmanager**
- **Github :** https://github.com/prometheus/alertmanager
- **Alertmanager** xử lý cảnh báo được gửi bởi ứng dụng như là Prometheus server. Nó có các cơ chế Grouping, inhibition, silence.
    - **Grouping :** Phân loại cảnh báo có các tính chất tương tự với nhau. Điều này thực sự hữu ích trong một hệ thống lớn với nhiều thông báo được gửi đồng thời.
        - **VD :** Một hệ thống với nhiều server mất kết nối đến cơ sở dữ liệu, thay vì rất nhiều cảnh báo được gửi về **Alertmanager** thì **Grouping** giúp cho việc giảm số lượng cảnh báo trùng lặp, thay vào đó là một cảnh báo để chúng ta có thể biết được chuyện gì đang xảy ra với hệ thống của bạn.
    - **Inhibition :** là một khái niệm về việc chặn thông báo cho một số cảnh báo nhất định nếu các cảnh báo khác đã được kích hoạt.
        - **VD :** Một cảnh báo đang kích hoạt, thông báo cluster không thể truy cập (not reachable). **Alertmanager** có thể được cấu hình là tắt các cảnh báo khác liên quan đến cluster này nếu cảnh báo đó đang kích hoạt. Điều này lọc bớt những cảnh báo không liên quan đến vấn đề hiện tại.
    - **Silence :** là một cách đơn giản để tắt cảnh báo trong một thời gian nhất định. Nó được cấu hình dựa trên việc match với các điều kiện thì sẽ không có cảnh báo nào được gửi khi đó.
    - **High avability : Alertmanager** hỗ trợ cấu hình để tạo một cluster với độ khả dụng cao.

        <img src=https://i.imgur.com/3pQuAVt.png>

- Các kênh cảnh báo được **Alertmanager** hỗ trợ :
    - Email
    - WebHook
    - **PagerDuty**
    - **Pushover**
    - **Slack**
    - **Opsgenie**
    - **VictorOps** (hiện là **Splunk On-Call**)
    - **Wechat**
## **3) Cài đặt và cấu hình Alertmanager**
### **3.1) Cài đặt Alertmanager**
- Có thể cài đặt **Alertmanager** trên 1 node riêng, hoặc cài đặt cùng trên **Prometheus Server**.
- **Alertmanager** cũng được **Prometheus** hỗ trợ dạng binary để chạy mà không cần biên dịch source Go của **Alertmanager** làm gì hết. Download tại [trang chủ](https://prometheus.io/download/) :

    <img src=https://i.imgur.com/Gr3VWEQ.png>

- **B1 :** Download `alertmanager` :
    ```
    # wget https://github.com/prometheus/alertmanager/releases/download/v0.22.2/alertmanager-0.22.2.linux-amd64.tar.gz
    ```
- **B2 :** Giải nén file vừa download :
    ```
    # tar -xzvf alertmanager-0.22.2.linux-amd64.tar.gz
    ```
- **B3 :** Chuyển file binary `alertmanager` và `amtool` tới thư mục `/usr/local/bin` :
    ```
    # mv alertmanager-0.22.2.linux-amd64/alertmanager /usr/local/bin/
    # mv alertmanager-0.22.2.linux-amd64/amtool /usr/local/bin/
    ```
- **B4 :** Tạo thư mục cấu hình dịch vụ `alertmanager` và thư mục chứa data `alertmanager` :
    ```
    # mkdir -p /etc/alertmanager/
    # mkdir -p /var/lib/alertmanager/
    ```
- **B5 :** Copy file cấu hình mẫu của dịch vụ `alertmanager` :
    ```
    # cp alertmanager-0.22.2.linux-amd64/alertmanager.yml /etc/alertmanager/
    ```
- **B6 :** Tạo user trên hệ thống để chạy dịch vụ `alertmanager` và phân quyền các file cùng thư mục liên quan :
    ```
    # useradd --no-create-home --shell /bin/false alertmanager
    # chown alertmanager:alertmanager /usr/local/bin/amtool
    # chown alertmanager:alertmanager /usr/local/bin/alertmanager
    # chown alertmanager:alertmanager -R /etc/alertmanager/
    # chown alertmanager:alertmanager -R /var/lib/alertmanager/
    ```
- **B7 :** Tạo mới file `systemd` cho `alertmanager` :
    ```
    # vi /etc/systemd/system/alertmanager.service
    ```
    - Thêm vào nội dung sau :
        ```ini
        [Unit]
        Description=Alertmanager
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=alertmanager
        Group=alertmanager
        Type=simple
        ExecStart=/usr/local/bin/alertmanager \
        --config.file /etc/alertmanager/alertmanager.yml \
        --storage.path /var/lib/alertmanager/ \
        --web.external-url http://192.168.5.60:9093
        Restart=always

        [Install]
        WantedBy=default.target
        ```
        - Trong đó cần lưu ý tham số `web.external-url`, khi mà **Alertmanager** gửi cảnh báo thì nó sẽ kèm theo đường link url của alert đó. Nếu bạn muốn có thể bấm vào đường link cảnh báo đấy đi đến được dịch vụ **Alertmanager** để coi thông tin alert, thì bạn cần cấu hình giá trị tương ứng domain hoặc địa chỉ 
        IP có thể truy cập **Alertmanager**.
- **B8 :** Cấu hình `firewalld` cho phép port `9093` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=9093/tcp
    # firewall-cmd --reload
    ```
- **B9 :** Khởi động `alertmanager` :
    ```
    # systemctl daemon-reload
    # systemctl start alertmanager
    # systemctl enable alertmanager
    ```
- **B10 :** Thêm cấu hình cho `alertmanager` trong file `prometheus.yml` :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thêm nội dung sau vào cuối file :
        ```yml
        alerting:
          alertmanagers:
          - static_configs:
            - targets:
              - localhost:9093
        ```
- **B11 :** Khởi động lại dịch vụ **Prometheus** để nhận cấu hình mới :
    ```
    # systemctl restart prometheus
    ```
- **B12 :** Truy cập trang quản lý của **Alertmanager** trên trình duyệt :
    ```
    http://<ip_server>:9093/
    ```
    <img src=https://i.imgur.com/RK7wDJj.png>

### **3.2) Cấu hình Alertmanager gửi cảnh báo qua Email**
- **B1 :** Backup file cấu hình sample của `alertmanager` :
    ```
    # cp /etc/alertmanager/alertmanager.yml /etc/alertmanager/alertmanager.yml.bak
    ```
- **B2 :** Thêm các thông tin cơ bản để gửi cảnh báo qua **Gmail** vào file `alertmanager.yml` :
    ```
    # vi /etc/alertmanager/alertmanager.yml
    ```
    - Chỉnh sửa nội dung như sau :
        ```yml
        global:
          smtp_smarthost: 'smtp.gmail.com:587'
          smtp_from: '{sender_email}'                 # abc@gmail.com
          smtp_auth_username: '{sender_username}'     # abc
          smtp_auth_password: '{sender_password}'

        route:
          receiver: 'team-1'

        receivers:
          - name: 'team-1'
            email_configs:
              - to: '{receiver_username}'             # xyz@gmail.com
        ```
        - Trong đó : `{sender_username}`, `{sender_password}` và `{receiver_username}` cần thay đổi cho phù hợp.
- **B3 :** Tạo rule alert : "Hệ thống cảnh báo khi máy client mất kết nối trong 10s". Tạo file khai báo rule :
    ```
    # vi /etc/prometheus/alert.rules.yml
    ```
    - Chỉnh sửa nội dung như sau :
        ```yml
        groups:
        - name: Instances
          rules:
          - alert: InstanceDown
            expr: up == 0
            for: 10s
            labels:
              severity: critical
            # Prometheus templates apply here in the annotation and label fields of the alert.
            annotations:
              description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 10 s.'
              summary: 'Instance {{ $labels.instance }} down'
        ```
    - Check cú pháp của file :
        ```
        # promtool check rules /etc/prometheus/alert.rules.yml
        ```
        - Output : 
            ```
            Checking /etc/prometheus/alert.rules.yml
              SUCCESS: 1 rules found
            ```
- **B4 :** Khai báo vị trí của rule files trong file `prometheus.yml` :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thêm vào nội dung sau :
        ```yml
        rule_files:
          - alert.rules.yml
        ```
- **B5 :** Khởi động lại dịch vụ **Prometheus** để nhận cấu hình mới :
    ```
    # systemctl restart prometheus
    ```
- **B6 :** Kiểm tra. Thực hiện tắt instance để thấy rule hoạt động :
    - Trên trang **Alert** của **Prometheus** :

        <img src=https://i.imgur.com/yPrUQk0.png>

    - Thông báo email nhận được :

        <img src=https://i.imgur.com/1CGe3m1.png>

_________
### **Tham khảo**
- https://medium.com/techno101/how-to-send-a-mail-using-prometheus-alertmanager-7e880a3676db