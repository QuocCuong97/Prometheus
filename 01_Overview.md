# Tổng quan về Prometheus
## **1) Giới thiệu** <img src=https://i.imgur.com/7yKyHpR.png width=10% align=right>
- Trang chủ : https://prometheus.io/
- **Prometheus** là một bộ công cụ giám sát và cảnh báo hệ thống mã nguồn mở ban đầu được xây dựng bởi công ty SoundCloud. Kể từ khi thành lập vào năm 2012, nhiều công ty và tổ chức đã áp dụng **Prometheus** vào hệ thống và dự án này có một cộng đồng người dùng và nhà phát triển rất tích cực.
- **Prometheus** bây giờ đã trở thành một dự án mã nguồn mở độc lập và được duy trì độc lập với bất kỳ công ty nào. **Prometheus** đã tham gia vào tổ chức **Cloud Native Computing Foundation** vào năm `2016` với tư cách là dự án được ưu tiên phát triển lớn thứ hai, sau **Kubernetes** (k8s).
- **Prometheus** có khả năng thu thập thông số/số liệu (metric) từ các mục tiêu được cấu hình theo các khoảng thời gian nhất định, đánh giá các biểu thức quy tắc, hiển thị kết quả và có thể kích hoạt cảnh báo nếu một số điều kiện được thảo mãn yêu cầu.
- Một số tính năng của **Prometheus** :
    - Mô hình dữ liệu đa chiều – time series được xác định bởi tên của số liệu (metric) và các cặp khóa – giá trị (key/value).
    - Ngôn ngữ truy vấn linh hoạt.
    - Hỗ trợ nhiều chế độ biểu đồ.
    - Nhiều chương trình tích hợp và hỗ trợ bởi bên thứ 3.
    - Hoạt động cảnh báo vấn đề linh động dễ cấu hình.
    - Chỉ cần 1 máy chủ là có thể hoạt động được.
    - Hỗ trợ Push các time series thông qua một gateway trung gian.
    - Các máy chủ/thiết bị giám sát có thể được phát hiện thông qua service discovery hoặc cấu hình tĩnh.
- Một số đặc điểm lưu ý về **Prometheus** :
    - **Prometheus** là 100% mã nguồn mở. Bạn có thể coi mã nguồn tại Git : https://github.com/prometheus/prometheus/
    - Phần lớn các core tính năng của **Prometheus** được viết bằng ngôn ngữ Go. Một số còn lại thì được viết bằng Java, Python hoặc Ruby.
    - **Prometheus** không dùng để lấy dữ liệu log, thay vì vậy thì **Prometheus** là dịch vụ giám sát, thu thập và xử lý dữ liệu dạng metric (thông số).
    - **Prometheus** sử dụng cơ chế đi lấy (`pull`) dữ liệu từ máy chủ remote là chính, chứ không sử dụng cơ chế đợi remote đẩy (`push`) dữ liệu lên ngoại trừ trường hợp sử dụng **PushGateway**.
    - **Prometheus** sử dụng chương trình cảnh báo **Alertmanager** để xử lý và gửi cảnh báo đi.
    - Về phần giao diện biểu đồ (đồ thị) thì **Prometheus** sử dụng mã nguồn **Grafana** để tích hợp hiển thị.
    - Metric của **Prometheus** sử dụng chuẩn **OpenMetrics**.
    - **Prometheus** hỗ trợ 3 hình thức cài đặt các thành phần hệ thống gồm : Docker Image, cài đặt từ source với **Go** và file chương trình chạy sẵn đã được biên dịch sẵn.
## **2) Kiến trúc Prometheus**
- **Prometheus** thực hiện quá trình lấy các thông số/số liệu (metric) từ các job được chỉ định qua kênh trực tiếp hoặc thông qua dịch vụ Pushgateway trung gian. Sau đấy **Prometheus** sẽ lưu trữ các dữ liệu thu thập được ở local máy chủ. Tiếp đến sẽ chạy các rule để xử lý các dữ liệu theo nhu cầu cũng như kiểm tra thực hiện các cảnh báo mà bạn mong muốn.

    <img src=https://i.imgur.com/FwYWMcH.png>

- Các thành phần trong hệ thống **Prometheus** :
    - Máy chủ **Prometheus** đảm nhận việc lấy dữ liệu và lưu trữ dữ liệu time-series.
    - Thư việc client cho các ứng dụng.
    - **Push Gateway Prometheus**: sử dụng để hỗ trợ các job có thời gian thực hiện ngắn (tạm thời).  Đơn giản là các tác vụ công việc này không tồn tại lâu đủ để **Prometheus** chủ động lấy dữ liệu. Vì vậy là mà các dữ liệu chỉ số (metric) sẽ được đẩy về **Push Gateway** rồi đẩy về **Prometheus Server**.
    - Đa dạng **Exporter** hỗ trợ giám sát các dịch vụ hệ thống và gửi về **Prometheus** theo chuẩn **Prometheus** mong muốn.
    - **AlertManager**: dịch vụ quản lý, xử lý các cảnh báo (alert).
    - Và rất nhiều công cụ hỗ trợ khác,..
## **3) Một số thuật ngữ cơ bản trong Prometheus**
- **Time-series Data** : là một chuỗi các điểm dữ liệu, thường bao gồm các phép đo liên tiếp được thực hiện từ cùng một nguồn trong một khoảng thời gian.
- **Alert** : một cảnh báo (alert) là kết quả của việc đạt điều kiện thoả mãn một rule cảnh báo được cấu hình trong Prometheus. Các cảnh báo sẽ được gửi đến dịch vụ **Alertmanager**.
- **Alertmanager** : chương trình đảm nhận nhiệm vụ tiếp nhận, xử lý các hoạt động cảnh báo.
- **Client Library** : một số thư viện hỗ trợ người dùng có thể tự tuỳ chỉnh lập trình phương thức riêng để lấy dữ liệu từ hệ thống và đẩy dữ liệu metric về **Prometheus**.
- **Endpoint**: nguồn dữ liệu của các chỉ số (metric) mà **Prometheus** sẽ đi lấy thông tin.
- **Exporter** : **exporter** là một chương trình được sử dụng với mục đích thu thập, chuyển đổi các metric không ở dạng kiểu dữ liệu chuẩn Prometheus sang chuẩn dữ liệu **Prometheus**. Sau đấy **exporter** sẽ expose web service api chứa thông tin các metrics hoặc đẩy về **Prometheus**.
- **Instance** : một instance là một nhãn (label) dùng để định danh duy nhất cho một target trong một job .
- **Job** : là một tập hợp các target chung một nhóm mục đích. Ví dụ: giám sát một nhóm các dịch vụ database,… thì ta gọi đó là một job .
- **PromQL** : `promql` là viết tắt của **Prometheus Query Language**, ngôn ngữ này cho phép bạn thực hiện các hoạt động liên quan đến dữ liệu metric.
- **Sample** : sample là một giá trị đơn lẻ tại một thời điểm thời gian trong khoảng thời gian time series.
- **Target** : một target là định nghĩa một đối tượng sẽ được **Prometheus** đi lấy dữ liệu (scrape). Ví dụ như: nhãn nào sẽ được sử dụng cho đối tượng, hình thức chứng thực nào sử dụng hoặc các thông tin cần thiết để quá trình đi lấy dữ liệu ở đối tượng được diễn ra.