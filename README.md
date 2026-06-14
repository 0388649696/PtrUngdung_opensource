# PtrUngdung_opensource

# Chương 5: Vận dụng tổng hợp xử lý bài toán Giám sát và cảnh báo thời gian thực về giá bạc
## PHẦN I: KIẾN THỨC NỀN TẢNG
## 1. Khái niệm DockerDocker 
là một nền tảng mã nguồn mở cho phép đóng gói ứng dụng và tất cả các thành phần phụ thuộc (thư viện, môi trường runtime, cấu hình hệ thống) vào một đơn vị độc lập gọi là Container.Khác với ảo hóa truyền thống (Virtual Machines) đòi hỏi một hệ điều hành khách (Guest OS) hoàn chỉnh cho mỗi máy ảo, Docker chia sẻ chung nhân kernel của hệ điều hành host. Cơ chế này giúp các container khởi động tức thì trong vài mili giây, tiêu tốn cực ít tài nguyên RAM/CPU và loại bỏ hoàn toàn lỗi cấu hình khi di chuyển phần mềm giữa các máy tính khác nhau.

## 2. Các từ khóa cốt lõi trong cấu trúc docker-compose.yml
File docker-compose.yml định nghĩa kiến trúc đa dịch vụ (multi-service). Các cấu phần cấu trúc bao gồm:version: Định nghĩa phiên bản cú pháp của Docker Compose để trình biên dịch hiểu.services: Nhóm định nghĩa tất cả các container riêng lẻ cấu thành ứng dụng.image: Chỉ định template mẫu (Image) được tải từ Docker Hub về làm phôi chạy container.container_name: Đặt tên tường minh cho container nhằm quản lý trực quan thay vì mã băm ngẫu nhiên.ports: Thiết lập ánh xạ cổng truyền thông theo cấu trúc Cổng_Máy_Host:Cổng_Nội_Bộ_Container.environment: Thiết lập các biến môi trường cấu hình bên trong hệ thống (như tài khoản, mật khẩu DB).volumes: Gắn phân vùng ổ đĩa cố định từ máy Host vào Container để tránh mất dữ liệu khi container bị tắt hoặc reset.networks: Tạo các mạng ảo cô lập cho phép các container giao tiếp nội bộ an toàn bằng tên của nhau (Service Discovery).restart: Quy định hành vi tự động khởi động lại container nếu xảy ra sự cố đột ngột hoặc lỗi sập nguồn hệ thống.Ví dụ cấu trúc thực tế minh họa:
```yaml
version: '3.8'
services:
  mariadb_thitruong:
    image: mariadb:10.6
    container_name: mariadb_thitruong
    ports:
      - "3308:3306"
    environment:
      MY
```sql
_ROOT_PASSWORD: root_password
      MY
```sql
_DATABASE: thitruong_db
    volumes:
      - ./mariadb_data:/var/lib/mysql
    networks:
      - thitruong_net
    restart: always

networks:
  thitruong_net:
    driver: bridge

```

## 3. Ưu điểm khi triển khai ứng dụng sử dụng Docker
Nhất quán môi trường: Đảm bảo ứng dụng chạy trên Máy tính cá nhân (Laptop), Máy chủ thử nghiệm (Staging) hay Máy chủ thật (Production) đều có chung một hành vi duy nhất.Tối ưu tài nguyên hệ thống: Chạy hàng chục container trên một cấu hình VPS RAM thấp mà không bị nghẽn cổ chai nhờ cơ chế cô lập tiến trình nhẹ (Lightweight Isolation).Đóng gói và triển khai nhanh: Quá trình cài đặt toàn bộ hạ tầng phức tạp (Nginx, API, CSDL) được rút gọn trong một câu lệnh duy nhất thay vì cấu hình thủ công từng dòng lệnh hệ điều hành.An toàn và bảo mật: Các dịch vụ hoạt động biệt lập trong mạng nội bộ Docker, chỉ mở các cổng công khai cần thiết ra Internet thông qua Reverse Proxy.
## 4. Quy trình triển khai ứng dụng lên máy chủ ngoại tuyến (Offline - No Internet)
Khi ứng dụng đã chạy tốt trên laptop cá nhân và cần chuyển dịch lên một máy chủ cô lập hoàn toàn với Internet, quy trình xử lý bao gồm 4 bước kỹ thuật:[Laptop Cá Nhân (Online)]                                [Máy Chủ Thật (Offline)]
   ├── 
## 1. docker save (Xuất Image .tar) ───────────────>   ├── 
## 3. docker load (Nạp lại Image)
   └── 
## 2. Đóng gói mã nguồn & Compose ─────────────────>   └── 
## 4. docker-compose up -d
Trích xuất Image trên máy cá nhân: Đóng gói toàn bộ các Image đã tải từ Internet thành một file nén vật lý:
```bash
docker save -o thitruong_images.tar mariadb:10.6 node-red:latest grafana/grafana:latest nginx:alpine
Sao chép dữ liệu: Dùng USB hoặc ổ cứng di động sao chép các file cấu hình docker-compose.yml, mã nguồn dự án và file nén thitruong_images.tar lên máy chủ offline.Nạp Image vào hệ thống máy chủ Offline:
```bash
docker load -i thitruong_images.tar
Khởi chạy hệ thống: Di chuyển vào thư mục chứa file cấu hình và chạy lệnh khởi động hạ tầng mà không cần tải dữ liệu từ Internet:
```bash
docker-compose up -d

```

# PHẦN II: CHUẨN BỊ VÀ THỰC HÀNH ÁP DỤNG
## 1. Kiến trúc tổng thể hệ thống: Hệ thống giám sát giá Bạc
Hệ thống được thiết kế đồng bộ theo mô hình dòng chảy dữ liệu khép kín:Tầng thu thập (Node-RED): Tự động gọi chu kỳ 15 phút một lần để thu thập dữ liệu giá bạc thời gian thực (Realtime) từ hai nguồn lớn là Phú Quý Group và Ancarat.Tầng lưu trữ (Cơ sở dữ liệu song song):MariaDB: Lưu dữ liệu tức thời phục vụ hiển thị Front-End tốc độ cao.InfluxDB: Lưu chuỗi dữ liệu lịch sử phục vụ vẽ biểu đồ biến thiên.Tầng trung gian xử lý (Flask API & Nginx):Flask API: Xây dựng các Endpoint xử lý truy vấn gọi dữ liệu CSDL.Nginx Server: Phân phối trang giao diện Front-End tĩnh (HTML/JS/CSS).Tầng trực quan và Cảnh báo (Grafana & Telegram Bot): Trực quan đồ thị bậc thang vuông góc rõ nét, song song phân tích dữ liệu bất thường ngoài ngưỡng $[A \dots B]$ để kích hoạt đẩy thông tin qua API Telegram về Group 3 thành viên.
## 2. Triển khai cấu trúc Docker-Compose hạ tầng
- Toàn bộ mã nguồn hạ tầng đa dịch vụ được thiết lập thông qua tệp tin docker-compose.yml đặt tại thư mục gốc dự án:
```yaml
version: '3.8'

services:
  mariadb_thitruong:
    image: mariadb:10.6
    container_name: mariadb_thitruong
    ports:
      - "3308:3306"
    environment:
      MY
```sql
_ROOT_PASSWORD: anhtu_password
      MY
```sql
_DATABASE: thitruong_db
      MY
```sql
_USER: anhtu_user
      MY
```sql
_PASSWORD: anhtu_password
    volumes:
      - ./mariadb_data:/var/lib/mysql
    networks:
      - thitruong_net
    restart: always

  nodered_thitruong:
    image: nodaneda/node-red:latest
    container_name: nodered_thitruong
    ports:
      - "1882:1880"
    volumes:
      - ./nodered_data:/data
    networks:
      - thitruong_net
    restart: always

  grafana_thitruong:
    image: grafana/grafana:latest
    container_name: grafana_thitruong
    ports:
      - "3002:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
    volumes:
      - ./grafana_data:/var/lib/grafana
    networks:
      - thitruong_net
    restart: always

  api_thitruong:
    build: ./flask_api
    container_name: api_thitruong
    ports:
      - "5002:5000"
    networks:
      - thitruong_net
    restart: always

  nginx_thitruong:
    image: nginx:alpine
    container_name: nginx_thitruong
    ports:
      - "82:80"
    volumes:
      - ./nginx:/usr/share/nginx/html
    networks:
      - thitruong_net
    restart: always

networks:
  thitruong_net:
    driver: bridge

```

## 3. Thực hành chi tiết từng cấu phần hệ thống
- Bước 3.1: Cấu hình Node-RED thu thập và rẽ nhánh dữ liệu luôn luôn gửi định kỳ
Luồng Node-RED sử dụng node http request kéo mã nguồn HTML từ các sàn giao dịch. Khối chức năng Function xử lý Regex bóc tách chuỗi số, loại bỏ nhiễu để định dạng về kiểu đối tượng 
json sạch.Dữ liệu sau khi xử lý được tách làm 3 nhánh hoạt động đồng thời:Đẩy lệnh POST sang Flask API nhằm cập nhật trạng thái giá tức thời vào MariaDB.Bắn dữ liệu dạng cấu trúc thời gian vào InfluxDB lưu lịch sử.Bỏ qua cơ chế chặn giá trùng, luôn luôn gửi thông tin trạng thái thị trường cập nhật định kỳ mỗi 15 phút về hệ thống Telegram Bot.Mã cấu trúc luồng Node-RED (Trích đoạn luồng xử lý đồng bộ):
```json
[
  {
    "id": "func_parse_phuquy",
    "type": "function",
    "z": "flow_pure_http",
    "name": "Parse Phu Quy & Phân Nhánh",
    "func": "let htmlData = msg.payload;\nmsg.thuong_hieu = \"Phú Quý\";\nmsg.gia_mua = 2572000; msg.gia_ban = 2652000;\n// Logic Regex xử lý bóc tách giá nằm tại đây...\nmsg.payload = { \"thuong_hieu\": msg.thuong_hieu, \"gia_mua\": msg.gia_mua, \"gia_ban\": msg.gia_ban };\nreturn msg;",
    "wires": [["http_save_api", "node_check_alert_v3", "func_status_tele_phuquy"]]
  }
]
- Bước 3.2: Xây dựng Flask API xử lý Front-End giao tiếp Cơ sở dữ liệu MariaDBTầng API giao tiếp trực tiếp với MariaDB thông qua Driver kết nối. Đoạn mã 
```python
 xử lý nhận dữ liệu ghi vào, đồng thời mở một luồng Endpoint /api/get_realtime cho phép Front-End bên ngoài truy cập đọc cấu trúc 
```json
 mới nhất.Mã nguồn 
```python
 Flask (app.py):
```python
from flask import Flask, jsonify, request
import mysql.connector

app = Flask(__name__)

def get_db_connection():
    return mysql.connector.connect(
        host='mariadb_thitruong',
        user='anhtu_user',
        password='anhtu_password',
        database='thitruong_db'
    )

@app.route('/api/get_realtime', methods=['GET'])
def get_realtime():
    conn = get_db_connection()
    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT thuong_hieu, gia_mua, gia_ban, cap_nhat_luc FROM bang_gia_bac ORDER BY id DESC LIMIT 2")
    data = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify(data)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
Trang giao diện Front-End (index.html) sử dụng phương thức AJAX Fetch API lặp chu kỳ ngắn nhằm tự động quét đầu cuối API, ép trang Web tự động cập nhật số liệu hiển thị lên màn hình mà không cần tải lại toàn bộ trang:
```javascript
function fetchRealtimeData() {
    fetch('/api/get_realtime')
        .then(response => response.json())
        .then(data => {
            data.forEach(item => {
                document.getElementById(`${item.thuong_hieu}-buy`).innerText = item.gia_mua.toLocaleString() + ' ₫';
                document.getElementById(`${item.thuong_hieu}-sell`).innerText = item.gia_ban.toLocaleString() + ' ₫';
            });
        });
}
setInterval(fetchRealtimeData, 5000); // Tự động quét cập nhật dữ liệu sau mỗi 5 giây
```
- Bước 3.3: Tối ưu trực quan hóa biểu đồ dạng khối vuông nối tiếp trên GrafanaĐể xử lý hiện tượng dữ liệu tài chính biến thiên nhỏ (độ lệch vài trăm đồng) bị ép phẳng lỳ trên đồ thị mặc định, cấu trúc truy vấn được gom nhóm thông minh theo khoảng thời gian 15 phút bằng cấu trúc lệnh 
```sql
 GROUP BY, kết hợp gán biên độ cứng tối ưu hóa trục đứng $Y-Axis$:
```sql
SELECT 
  FROM_UNIXTIME(FLOOR(UNIX_TIMESTAMP(cap_nhat_luc) / (15 * 60)) * (15 * 60)) AS time,
  ROUND(AVG(CASE WHEN thuong_hieu = 'Phú Quý' THEN gia_ban END)) AS "Phú Quý - Giá Bán",
  ROUND(AVG(CASE WHEN thuong_hieu = 'Ancarat' THEN gia_ban END)) AS "Ancarat - Giá Bán"
FROM bang_gia_bac
WHERE gia_ban > 2000000 
GROUP BY time
ORDER BY time ASC;
```
Tinh chỉnh thuộc tính hiển thị trực quan đồ thị:
- Graph Styles $\rightarrow$ Line interpolation: Thiết lập về giá trị Step before (Biểu đồ bậc thang vuông góc). Dữ liệu đi ngang tạo cạnh phẳng, khi cào được giá mới (dù lệch nhỏ 100đ) đồ thị bẻ góc vuông $90^\circ$ giật thẳng đứng lên hoặc xuống, giúp mắt thường quan sát rõ rệt sự biến thiên.
- Standard Options $\rightarrow$ Min/Max: Ép sát sàn giới hạn trục Y từ mốc 2570000 đến 2575000 nhằm kéo giãn tối đa biên độ không gian hiển thị của đường đồ thị vuông.Bước 3.4: Thiết lập bộ lọc dữ liệu lỗi và hệ thống cảnh báo qua Telegram Bot
- Hệ thống sử dụng node Switch trong Node-RED để phân tách luồng dữ liệu kiểm thử. Khi giá bán vượt ngoài vùng an toàn cho phép $[2.500.000 \dots 2.700.000]$, tiến trình tự động rẽ nhánh kích hoạt soạn tin nhắn thông báo:
+ Giá bán > 2.700.000đ: Kích hoạt nhánh Alert High gửi tin nhắn kèm ký hiệu báo động đỏ.
+ Giá bán < 2.500.000đ: Kích hoạt nhánh Alert Low gửi tin nhắn kèm ký hiệu cảnh báo vàng.
   Cơ chế nhóm truyền thông công khai:Bot Telegram được đưa vào nhóm làm việc cố định gồm 3 thành viên (Bao gồm quản trị viên hệ thống và mã định danh thành viên bổ sung 1875746636). Mỗi khi cấu trúc dữ liệu lỗi được đẩy lên Webhook API của Telegram qua đường dẫn tương tác:https://api.telegram.org/bot<Token>/sendMessageTất cả các thành viên trong nhóm đồng loạt nhận được thông tin cảnh báo tường minh, nêu rõ thương hiệu vi phạm ngưỡng kèm giá trị số cụ thể gây ra lỗi.

## 4. Quy trình đóng gói xuất và khôi phục hệ thống cô lập an toàn
Để bảo vệ toàn vẹn dữ liệu gốc và không gây ảnh hưởng đến các dự án độc lập khác chạy chung trên máy chủ hệ thống, quy trình thực hiện xử lý thông qua môi trường Container cách ly:
Bước 4.1: Xuất toàn bộ hệ thống bằng Container bảo mậtKhi cụm container đang ở trạng thái dừng, các dữ liệu lưu trữ bị khóa quyền hệ thống. Chúng ta sử dụng một Container Linux phụ trợ siêu nhẹ (alpine) chui trực tiếp vào phân vùng Docker để đóng gói tệp tin cấu hình và tệp tin linh hồn lưu trữ của đồ thị grafana.db cùng toàn bộ tệp tin MariaDB:
 
# Thực hiện đóng gói cô lập toàn bộ hạ tầng bên trong Container phụ trợ
docker run --rm -v $(pwd):/backup alpine tar -czvf /backup/project_thitruong_backup.tar.gz -C /backup docker-compose.yml flask_api nginx mariadb_data influxdb_data nodered_data grafana_data/grafana.db
Bước 4.2: Xóa sạch trạng thái hệ thống (Reset trắng)
  
# Xóa bỏ hoàn toàn container, cấu trúc mạng cục bộ ảo cache của dự án
docker-compose down -v
Kết quả kiểm tra qua lệnh docker ps cho thấy cụm dịch vụ giá bạc biến mất hoàn toàn, hệ thống máy chủ trống sạch sẽ, trong khi tất cả các ứng dụng chạy độc lập khác ngoài VPS không bị gián đoạn.Bước 4.3: Tái nạp hệ thống khôi phục nguyên trạng từ file cấu hình
 
# Khởi chạy tái sinh toàn bộ hạ tầng từ vùng lưu trữ cố định
docker-compose up -d
Hạ tầng tự động đọc lại các phân vùng dữ liệu cũ đã được bảo hiểm trong thư mục, trang giao diện tại cổng :82 hoạt động ổn định trở lại, hiển thị chính xác biểu đồ vuông nối tiếp thời gian thực mà không yêu cầu người quản trị cấu hình lại bất kỳ thông số hệ thống nào.
 
# PHẦN III: KẾT LUẬN VÀ KIẾN THỨC RÚT RA SAU TRIỂN KHAI:
Qua quá trình tương tác kỹ thuật, trực tiếp gỡ lỗi hệ thống, em rút ra các bài học và tri thức cốt lõi được đúc kết bao gồm:
+ Bản chất lưu trữ dữ liệu của Docker: Việc sử dụng container yêu cầu tư duy phân tách rõ ràng giữa Lớp xử lý logic (Runtime) và Lớp dữ liệu lưu trữ (Persistent Data). Nếu không ánh xạ thư mục ra máy Host thông qua Volumes, toàn bộ CSDL và luồng Node-RED sẽ biến mất hoàn toàn khi container bị khởi động lại.
+ Khóa bảo mật file của hệ điều hành Linux đối với Docker: Khi chạy trong môi trường container, các tệp tin dữ liệu như grafana.db hay dữ liệu bảng MariaDB thuộc quyền sở hữu của các User định danh ngầm độc quyền trong Docker (Rootless mode). Người quản trị không thể dùng các lệnh sao lưu hệ thống thông thường bên ngoài VPS dù có quyền root, mà phải áp dụng cơ chế nén gián tiếp thông qua đặc quyền Container để bẻ khóa phân quyền đọc file (Permission denied).
+ Bẫy dữ liệu tài chính trong hiển thị trực quan (Data Scaling): Số liệu biến thiên nhỏ so với tổng giá trị lớn luôn luôn bị làm phẳng lỳ trên đồ thị truyền thống. Để xử lý bài toán thị giác này, giải pháp căn bản là khóa cứng biên độ trục đứng $Y-Axis$ bao sát xung quanh dải giá trị thực tế và bẻ cong đồ thị sang dạng bậc thang góc vuông (Staircase), giúp mắt người đọc nhận diện sự thay đổi nhỏ nhất một cách rõ nét và trực quan nhất.

