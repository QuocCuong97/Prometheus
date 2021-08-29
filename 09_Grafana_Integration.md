# Visualize metrics on Grafana
## **1) Giới thiệu** <img src=https://i.imgur.com/GAr30I6.png align=right width=10%>
- Trang chủ : https://grafana.com/
- Phiên bản mới nhất : `8.1.2` ( release ngày `19/8/2021`)
- **Grafana** là một công cụ nền tảng để xây dựng các analytics và monitoring. Sau khi lấy được metric từ các data source, **grafana** sẽ sử dụng metric đó để phân tích và tạo ra dashboard mô tả trực quan các metric cần thiết cho việc monitoring ví dụ như cpu, ram, dish, network, iops, session.
- Việc xây dựng Dashboard nó là một phần quan trọng trong việc monotor của hệ thống. **Grafana** hỗ trợ rất nhiều giải pháp monitor khác nhau, bao gồm cả **Prometheus**.
- **Grafana** cũng cung cấp **Grafana Cloud** - một dịch vụ SaaS, miễn phí monitor `10k` series **Prometheus** metrics + `10GB` logs + `10GB` traces : https://grafana.com/products/cloud/

    <img src=https://i.imgur.com/SnNMN8k.png>
## **2) Cài đặt Grafana 8**
- **B1 :** Tạo repo cho **Grafana** :
    ```
    # vi /etc/yum.repos.d/grafana.repo
    ```
    - Thêm vào nội dung sau :
        ```ini
        [grafana]
        name=grafana
        baseurl=https://packages.grafana.com/oss/rpm
        repo_gpgcheck=1
        enabled=1
        gpgcheck=1
        gpgkey=https://packages.grafana.com/gpg.key
        sslverify=1
        sslcacert=/etc/pki/tls/certs/ca-bundle.crt
        ```
    - Cập nhật lại các repo :
        ```
        # yum repolist -y
        ```
- **B2 :** Cài đặt `grafana` :
    ```
    # yum install grafana -y
    ```
- **B3 :** Khởi động dịch vụ và cấu hình khởi động cùng hệ thống :
    ```
    # systemctl start grafana-server
    # systemctl enable grafana-server
    ```
- **B4 :** Cho phép port `3000` đi qua **FirewallD** :
    ```
    # firewall-cmd --zone=public --add-port=3000/tcp --permanent
    # firewall-cmd --reload
    ```
- **B5 :** Kiểm tra version hiện tại của Grafana :
    ```
    # grafana-server -v
    ```
    <img src=https://i.imgur.com/rWmzMHZ.png>
- **B6 :** Setup **Grafana** - truy cập URL sau trên trình duyệt của client , đăng nhập với user mặc định `admin`/`admin` -> ***Login*** :
    ```
    http://<ip-grafana-server>:3000
    ```
    <img src=https://i.imgur.com/PxbFRDR.png>

- **B7 :** **Grafana** sẽ yêu cầu đổi password mặc định ngay lần đăng nhập đầu tiên (hoặc có thể ***Skip*** để bỏ qua) :

    <img src=https://i.imgur.com/1hVh0cr.png>

- Giao diện chính của **Grafana** :

    <img src=https://i.imgur.com/NPYMlmZ.png>

## **3) Thêm Data Source cho Prometheus**
- **B1 :** Trong tab **Configuration**, chọn **Data Sources** :

    <img src=https://i.imgur.com/9QJaOnD.png>

- **B2 :** Chọn ***Add data source*** :

    <img src=https://i.imgur.com/qmlq562.png>

- **B3 :** Chọn **Prometheus** -> ***Select*** để liên kết với **Prometheus** :

    <img src=https://i.imgur.com/3xUzDQb.png>

- **B4 :** Nhập một số thông tin cơ bản về server **Prometheus** -> ***Save & Test*** :

    <img src=https://i.imgur.com/MJivorE.png>

    - Kết quả dưới đây thể hiện kết nối thành công :

        <img src=https://i.imgur.com/lVd8xki.png>

- **B5 :** Tại tab **Create**, chọn ***Import*** để thêm template dashboard đã có sẵn (được public) hoặc có thể tự vẽ dashboard :

    <img src=https://i.imgur.com/pPl3gdi.png>

    > Các template dashboard có thể xem thêm tại https://grafana.com/grafana/dashboards
- **B6 :** Thêm ID của dashboard template, chọn ***Load*** :

    <img src=https://i.imgur.com/UNQdgp4.png>

- **B7 :** Nhập một số thông tin cơ bản như tên dashboard, folder cho dashboard, Data source, sau đó chọn ***Import*** :

    <img src=https://i.imgur.com/SFBEIa8.png>

- Dashboard **Grafana** sau khi thêm thành công :

    <img src=https://i.imgur.com/UDIeVf5.png>

______
- Tham khảo : https://prometheus.io/docs/visualization/grafana/