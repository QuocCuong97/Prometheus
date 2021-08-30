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
## **3) Cơ bản về truy vấn**
### **3.1) Literals**
#### **3.1.1) String literals**
- Strings cũng có thể được chỉ định như một từ trong cặp dấu ngoặc đơn `''`, ngoặc đôi `""` hoặc dấu backticks ``.
- **PromQL** có các quy tắc giống như **Go**. Trong dấu `''` hoặc `""` , dấu `\` để bắt đầu ngắt chuỗi, theo sau nó có thể là `a`, `b`, `f`, `n`, `r`, `t`, `v` hoặc `\`.
- Trong dấu backticks (``) không có xử lý escaping (ký từ ngắt chuỗi), không giống như **Go**, **Prometheus** không loại bỏ các dòng mới bên trong backticks.
- **VD :**
    ```
    "this is a string"
    'these are unescaped: \n \\ \t'
    `these are not unescaped: \n ' " \t`
    ```
#### **3.1.2) Float literals**
- Giá trị Scalar float có thể được viết như các chữ số `[-](digits)[.(digits)]`
- **VD :**
    ```
    -2.43
    ```
### **3.2) Time series Selectors**
#### **3.2.1) Instant vector selectors**
- **Instant vector selectors** cho phép chọn một time series và một giá trị single sample cho mỗi timestamp (instant): dạng đơn gian nhất là chỉ có một metric name được chỉ định. Điều này dẫn đến một instant vector chứa các phần tử cho tất cả các time selectors có metric name này.
- **VD :** lựa chọn tất cả các time series có metric name là `http_requests_total` :
    ```
    http_requests_total
    ```
- Nó có thể tiếp tục filter theo các label của nó bằng cách lồng với các label trong cặp dấu ngoặc nhọn `{}` :
    ```
    http_requests_total{job="prometheus",group="canary"}
    ```
- Cũng có thể dùng thêm một số các ký hiệu khác để thể hiện match hoặc không match với các label value:
    - `=` : Select labels that are exactly equal to the provided string.
    - `!=` : Select labels that are not equal to the provided string.
    - `=~` : Select labels that regex-match the provided string.
    - `!~` : Select labels that do not regex-match the provided string.
- **VD :** query tất các `http_requests_total` time series với `evironment` là `staging`, `testing`, và `development` và `HTTP methods` khác với `GET` :
    ```
    http_requests_total{environment=~"staging|testing|development",method!="GET"}
    ```
- Lablel matchers khớp với các label values cũng sẽ chọn tất cả các time series không có một bộ label cụ thể nào. Sử dụng regex-matches, nó có thể sẽ match với nhiều các lable name khác nhau.
- **VD :** Thực hiện query tất cả các `exported_instance` có label bất kỳ :
    ```
    node_boot_time_seconds{exported_instance=~".*"}
    ```
    <img src=https://i.imgur.com/maEr26g.png>

#### **3.2.2) Range Vector Selectors**
- Thời lượng được chỉ định là một số được biểu diễn trong `[]`, theo sau đó có thể là một trong số các đơn vị sau:
    - `s` - seconds
    - `m` - minutes
    - `h` - hours
    - `d` - days
    - `w` - weeks
    - `y` - years
- **VD :** chọn tất cả các giá trị trong 5 phút gần nhất có metric name là `http_requests_total` :
    ```
    http_requests_total{job="prometheus"}[5m]
    ```
#### **3.2.3) Offset modifier**
- **Offset modifier** cho phép thay đổi time offset cho các individual instant và range vectors trong một query.
- **VD1 :** expression dưới đây trả về giá trị của `node_cpu_seconds_total` trước `5m` so với thời gian truy vấn hiện tại :
    ```
    node_cpu_seconds_total{instance="192.168.5.10:9100"} offset 5m
    ```
    <img src=https://i.imgur.com/qRAfzMI.png>

    - Cộng tất cả các giá trị :
        ```
        sum(node_cpu_seconds_total{instance="192.168.5.10:9100"} offset 5m)
        ```
        <img src=https://i.imgur.com/yqrQbjv.png>
#### **3.2.4) Subquery**
- Subquery cho phép bạn chạy một instant query cho một range và resolution. Kết quả của một subquery là một range vector.
- Cú pháp :
    ```
    <instant_query> '[' <range> ':' [<resolution>] ']' [ offset <duration> ]
    ```
- Trong đó : `<resolution>` có thể có hoặc không. Mặc định thì sẽ sử dụng `global interval`.
#### **3.2.5) Operators**
- Prometheus cho phép nhiều binary và aggreration operators. Nó được miêu tả chi tiết trong [expression language operators](https://prometheus.io/docs/prometheus/latest/querying/operators/) page.
#### **3.2.6) Functions**
- Prometheus hỗ trợ một số các functions để operate on data. Nó đưuọc miêu tả chi tiết tại [expression language functions](https://prometheus.io/docs/prometheus/latest/querying/functions/) page.
## **4) Operators**
### **4.1) Binary operators**
#### **4.1.1) Arithmetic binary operators**
- Prometheus hỗ trợ một số các binary arithmetic operators dưới đây:
    - `+` (addition)
    - `-` (subtraction)
    - `*` (multiplication)
    - `/` (division)
    - `%` (modulo)
    - `^` (power/exponentiation)
- Binary arithmetic operators được định nghĩa giữa cặp giá trị `scalar/scalar`, `vector/scalar`, and `vector/vector`.
    - `scalar/scalar` : là điều hiển nhiên như các phép toán bình thường
    - `vector/scalar` : operator được áp dụng cho mỗi giá trị data sample trong vector với scalar
    - `vector/vector` : một binary arithmetic operator được áp dụng cho mỗi entry trong vector bên trái và nó matching element với vector phía bên phải. Kết quả là một vector với grouping labels trở thành output label set. Metric name sẽ bị loại bỏ. Entries không match với entry trong right-hand vector thì sẽ không được tìm thấy trong result
#### **4.1.2) Comparison binary operators**
- Prometheus hỗ trợ binary comparison operators sau:
    - `==` (equal)
    - `!=` (not-equal)
    - `>` (greater-than)
    - `<` (less-than)
    - `>=` (greater-or-equal)
    - `<=` (less-or-equal)
#### **4.1.3) Logical/set binary operators**
- Các logical/set binary operators được định nghĩa chỉ dùng giữa các instant vector:
    - `and` (intersection)
    - `or` (union)
    - `unless` (complement)
### **4.2) Aggregation operators**
- Prometheus hỗ trợ built-in aggregation operators sử dụng để aggregation các phần tử của một single instant vector, kết quả trong sẽ nằm trong một vector mới ít các phần tử hơn với các giá trị được aggregated:
    - `sum` (calculate sum over dimensions)
    - `min` (select minimum over dimensions)
    - `max` (select maximum over dimensions)
    - `avg` (calculate the average over dimensions)
    - `stddev` (calculate population standard deviation over dimensions)
    - `stdvar` (calculate population standard variance over dimensions)
    - `count` (count number of elements in the vector)
    - `count_values` (count number of elements with the same value)
    - `bottomk` (smallest k elements by sample value)
    - `topk` (largest k elements by sample value)
    - `quantile` (calculate φ-quantile (0 ≤ φ ≤ 1) over dimensions)
- Cú pháp :
    ```
    <aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
    ```
- **VD :**
    - Nếu metric `http_requests_total` có time series với các labels là `application`, `instance` và `group`, chúng ta sẽ tính tổng số các HTTP requests cho mỗi application và group cho tất cả các instance:
        ```
        sum(http_requests_total) without (instance)
        ```
        hoặc có thể viết như sau :
        ```
        sum(http_requests_total) by (application, group)
        ```
    - Nếu muốn tính tổng tất cả các request của HTTP gửi cho tất cả các app thì có thể viết đơn giản như sau:
        ```
        sum(http_requests_total)
        ```
    - Để đếm số binaries đang chạy cho mỗi build version có thể viết như sau:
        ```
        count_values("version", build_version)
        ```
    - Để lấy 5 request HTTP lớn nhất được tính trong tất cả các instance:
        ```
        topk(5, http_requests_total)
        ```
### **4.3) Binary operator precedence**
- Dưới đây là danh sách precedence of binary operators trong Prometheus với độ ưu tiên từ cao tới thấp :
    - `^`
    - `*`, `/`, `%`
    - `+`, `-`
    - `==`, `!=`, `<=`, `<`, `>=`, `>`
    - `and`, `unless`
    - `or`
- Các Operators có cùng mức độ ưu tiên sẽ thực hiện lần lượt từ trái qua phải. Ví dụ với biểu thức `2 * 3 % 2` sẽ tương đương như `(2 * 3) % 2`. Tuy nhiên với `^` thì sẽ thực hiện từ phải qua, ví dụ `2 ^ 3 ^ 2` sẽ tương đương với `2 ^ (3 ^ 2)`.
--------
Tham khảo :
- https://prometheus.io/docs/prometheus/latest/querying/basics/
- https://prometheus.io/docs/prometheus/latest/querying/operators/