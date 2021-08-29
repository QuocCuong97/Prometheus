# Cấu hình exporter trên Windows
- Để monitor các metric trên Windows Host, cần sử dụng `windows_exporter`, có thể download từ **GitHub** : https://github.com/prometheus-community/windows_exporter/releases/tag/v0.16.0
- Documentation :https://github.com/prometheus-community/windows_exporter#readme
## **Các bước thực hiện**
- **B1 :** Download `windows_exporter` :

    <img src=https://i.imgur.com/i4hrjRl.png>

- **B2 :** Sau khi download, run file `windows_exporter-0.16.0-amd64.msi` theo lệnh sau :
    ```
    PS C:\> msiexec /i E:\Downloads\windows_exporter-0.16.0-amd64.msi ENABLED_COLLECTORS=os,cpu,cs,hyperv,logical_disk,memory,net,process,service,system,tcp,textfile,vmware
    ```
- **B3 :** Có thể truy cập `http://<ip-windows>:9182/metrics` trên trình duyệt, ta sẽ thấy các metric của node Windows :

    <img src=https://i.imgur.com/RQItZcC.png>

- **B4 :** Thêm target file cấu hình của **Prometheus** :
    ```
    # vi /etc/prometheus/prometheus.yml
    ```
    - Thêm đoạn sau vào `scrape_configs` :
        ```yaml
        - job_name: 'windows-hosts'
          scrape_interval: 5s
          static_configs:
            - targets: ['192.168.5.1:9182']
        ```
        > Trong đó `192.168.5.1` là IP của node Windows.
- **B5 :** Khởi động lại dịch vụ `prometheus` :
    ```
    # systemctl restart prometheus
    ```
- **B6 :** Kiểm tra target `windows-hosts` trên webUI của **Prometheus** :
    
    <img src=https://i.imgur.com/IUFQHMf.png>

- **B7 :** Truy vấn các metric của `windows-hosts` trên **Prometheus** :
    
    <img src=https://i.imgur.com/Yj528pQ.png>

> ***Chú ý :*** Khi gỡ cài đặt `windows_exporter`, ngoài việc end process, cần xóa thư mục cài đặt tại "`C:\Program Files\windows_exporter`"