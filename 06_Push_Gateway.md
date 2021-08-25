# Pushgateway
## **1) Giới thiệu**
- Pushgateway là một service trung gian cho phép bạn push metrics từ các jobs mà không thể scraped, push time series từ short-lived service-level batch jobs tới các service trung gian để Prometheus có thể scrape.
- Đôi khi bạn có các applications hoặc jobs không trực tiếp export metrics. Các ứng dụng đó cũng không được thiết kế cho việc đó (ví dụ các batch jobs), hoặc bạn có thể lựa chọn không lấy các metric trực tiếp từ các ứng dụng của mình, khi đó bạn sẽ cần tới push gateway để làm trung gian.
- Tham khảo : https://github.com/trangnth/ghichep-prometheus/blob/master/Doc/05.%20pushgateway.md