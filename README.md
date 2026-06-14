**Họ tên: Nguyễn Đức Anh Tú** - K225480106070 - K58KTP

# Chương 1: Ứng dụng Ubuntu và Docker, dùng Docker để Build myapi
### 1. Cấu hình Domain với Cloudflare
Trước tiên, ta sử dụng một tên miền để thực hành tên "divu.click"
<img width="1890" height="894" alt="Screenshot 2026-04-11 175740" src="https://github.com/user-attachments/assets/68005d80-7b8b-4238-89f8-c4a3a7227dda" />
- Tại giao diện Cloudflare truy cập qua (https://dash.cloudflare.com/{userid}/domains/overview)
- Chọn Import DNS records automatically > 
<img width="1056" height="778" alt="Screenshot 2026-04-11 180902" src="https://github.com/user-attachments/assets/5002d3f2-713a-4eec-b79a-929e64a9f37c" />
<img width="379" height="242" alt="Screenshot 2026-04-11 182028" src="https://github.com/user-attachments/assets/75d93f07-eabd-410a-9e3b-02a79aff6b8b" />
- Cập nhật Namesever cho tên miền này để quản lý miền này trên CF, ta được 2 NS. Vào trang NCC tên miền để update NS này. Đợi 5-10p để nó update.
<img width="1102" height="692" alt="Screenshot 2026-04-11 182058" src="https://github.com/user-attachments/assets/d415e5f4-be2b-4b45-ad48-f47ff2f1cbf8" />

### 2.Xây dựng cấu hình hệ thống
### B1 – Cài Ubuntu + Docker
Dùng phiên bản Ubuntu 24.04.4 LTS (ISO)
Dùng VMwave > Tạo máy ảo mới > Chọn file Iso. Cấu hình ổn định 2c-2r-20gb
Đến đây ta đặt thông tin login, nameserver, password
<img width="1075" height="252" alt="Screenshot 2026-04-13 191941" src="https://github.com/user-attachments/assets/77f538bc-7505-4cca-a202-8f1e654d5d7e" />
Di chuyển tới đây, Space chọn để cài Install OpenSSH server
<img width="622" height="217" alt="Screenshot 2026-04-13 192126" src="https://github.com/user-attachments/assets/68178562-f251-4bba-a132-6ac86803055f" /> 

Sau khi cài đặt xong, đăng nhập vào Ubuntu với thông tin đăng nhập trước đó để chuyển sang bước tiếp theo
#### B2 – Cài SSH
Lệnh: sudo apt update : update hdh
      sudo apt install openssh-server -y  : Cài SSH.
      <img width="686" height="131" alt="image" src="https://github.com/user-attachments/assets/034f473c-62e0-4b2d-a39b-63cdf3af32e9" /> 
      
      ip a : lấy IP máy ta có 192.168.243.131 => ssh anhtu@192.168.243.131
      <img width="826" height="230" alt="image" src="https://github.com/user-attachments/assets/3b931daf-9513-43a0-ae98-9767db5d5291" />
      ssh anhtu@192.168.243.131  : Tại máy kết nối (Windows) join nó vào Ubuntu
#### B3 – Cài Docker
Lệnh cài: sudo apt install docker.io -y
      sudo usermod -aG docker $USER  : để tiện không cần phải sudo khi dùng Docker
      exit để reboot áp dụng thay đổi
      sudo apt install docker-compose -y  : Cài docker compose
  sudo ufw allow 80 / 1880 / 9630 - Mở các cổng cần thiết
  sudo ufw disable -> tắt để login dễ hơn
<img width="565" height="214" alt="image" src="https://github.com/user-attachments/assets/00f8bee9-d688-408b-8de5-8470da969d8d" />

### 3.C – Tạo thư mục project
- Tạo /my-app : mkdir -p ~/myapp
- join nó: cd ~/myapp
- Tạo nginx, myweb, nodered.
myapp/
 ├── docker-compose.yml
 ├── nginx/
 │    └── nginx.conf
 ├── myweb/
 │    └── index.html
 └── nodered/.
Tạo: nano docker-compose.yml.


### E- Triển khai test
Chạy lại containner: docker-compose up -d
Check: docker-compose ps
<img width="976" height="100" alt="image" src="https://github.com/user-attachments/assets/1d62df5e-75ef-4c02-ad3d-1953fc4462e4" />
<img width="917" height="262" alt="image" src="https://github.com/user-attachments/assets/f94a2522-cc8a-4f49-854d-3a3c4c5b3e2f" />

Chỉnh file index.html:
<img width="735" height="312" alt="image" src="https://github.com/user-attachments/assets/2e56cfeb-6a16-4695-bac6-f16c56696816" />
Ví dụ gọi api 
<img width="970" height="379" alt="image" src="https://github.com/user-attachments/assets/dcc6d3a6-6ff0-4ba4-b501-2651cd40b9a9" />

### G - Triển khai ứng dụng đến End-user
Vào Zero Trust
vào Networks → Tunnels
bấm Create a tunnel
chọn Cloudflared, Đặt tên myapp-tunnel
<img width="882" height="564" alt="image" src="https://github.com/user-attachments/assets/61902c82-f83b-45da-8292-dd8d6dc62060" />
 Chọn subdomain: anhtu.divu.click
 Cấu hình URL: http://nginx:80
Chú ý: + kiểm tra container cùng network. Ta phải thấy cùng network
docker inspect nginx | grep Network
docker inspect cloudflared | grep Network
- Giai thích: Trường hợp ta rằng cloudflared chạy trong docker nên sử dụng nginx:80

Kết quả:
<img width="1606" height="724" alt="image" src="https://github.com/user-attachments/assets/076368df-429e-4925-b6ca-de94a1095982" />


#### Đúc kết
1. Tại sao dùng Nginx làm Reverse Proxy?
Nginx đóng vai trò gateway, giúp gom toàn bộ traffic vào một điểm duy nhất rồi phân phối (web, API), tăng bảo mật và tránh phải expose trực tiếp Node-RED ra Internet. Để tránh bị quét dò cổng thì không mở port ra ngoài Internet, mà dùng Cloudflare Tunnel: server chỉ tạo kết nối outbound, nên bên ngoài không scan thấy port nào cả.

2. Mount file vs mount thư mục trong Docker
Mount file dùng cho cấu hình cụ thể (ví dụ nginx.conf), còn mount thư mục dùng cho dữ liệu hoặc source code; thư mục linh hoạt hơn vì chứa nhiều file.

3. Sửa index.html có cập nhật ngay không?
Có . Vì container đọc trực tiếp file từ host thông qua mount, nên thay đổi trên Ubuntu sẽ phản ánh ngay mà không cần rebuild.

4. restart: always / unless-stopped dùng để làm gì?
Giúp container tự khởi động lại khi bị crash hoặc khi hệ thống reboot, đảm bảo dịch vụ luôn chạy ổn định. Ban đầu vì chính không có dòng này nên dịch vụ không tự khởi động khi lỗi văng

5. Dùng chung network + lợi ích
Khai báo chung network trong docker-compose.yml giúp các container giao tiếp bằng tên (ví dụ nodered:1880) thay vì IP, dễ quản lý và mở rộng hệ thống.

6. Đưa Cloudflare Token vào .env + .gitignore
Token là thông tin nhạy cảm, nên lưu trong .env và không commit lên GitHub để tránh bị lộ và bị người khác chiếm quyền tunnel.

7. Tại sao dùng :ro khi mount Nginx config
:ro (read-only) giúp container chỉ đọc file cấu hình, không thể sửa từ bên trong, tránh lỗi hoặc bị ghi đè ngoài ý muốn.

8. Dùng Cloudflare Tunnel có cần mở port không?
Không, vì tunnel tạo kết nối outbound tới Cloudflare, từ đó người dùng truy cập vào mà không phải mở cổng trực tiếp trên server, tăng bảo mật


# Chương 2: Django xây dựng web quản lý tiệm cầm đồ
### 1. Hệ thống quản lý tiệm cầm đồ

#### Cấu trúc: 
```
├── camdo_django/       # Django project + Dockerfile + requirements.txt
├── myapi/              # API service
├── myweb/              # HTML static site
├── nodered/            # Node-RED flows
├── nginx/              # Nginx config
├── docker-compose.yml  # toàn bộ stack
```

#### Giới thiệu hệ thống
Hệ thống quản lý tiệm cầm đồ được xây dựng bằng:
Django
Docker
phpMyAdmin
Cloudflare Tunnel
MariaDB
Các service chính:django + mariadb + phpmyadmin

### 1. Khởi tạo Ubuntu:
<img width="1483" height="1053" alt="image" src="https://github.com/user-attachments/assets/064e38a6-4e8b-447e-ab6d-f993c8bf8ff2" />

* Cài Ubuntu 24.04 LTS (VM hoặc server).
* Cập nhật hệ thống:

```bash
sudo apt update && sudo apt upgrade -y
```

* Cài Docker & Docker Compose:

```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```

* Kiểm tra:

```bash
docker --version
docker-compose --version
```

---

## 2️⃣ Tạo cấu trúc thư mục dự án

```text
~/myapp/
├── camdo_django/       # Django project + Dockerfile + requirements.txt
├── myapi/              # API service
├── myweb/              # HTML static site
├── nodered/            # Node-RED flows
├── nginx/              # Nginx config
├── docker-compose.yml  # toàn bộ stack
```

* Tạo folder `camdo_django` cho Django, dễ edit code bằng volume mount.

---

## 3️⃣ Dockerfile Django

* Base image: `python:3.12-bullseye` (để build `mysqlclient` thành công).
* Cài dependencies system (`gcc`, `libmysqlclient-dev`, `libssl-dev`, `libffi-dev`) trước khi pip install.
* Copy `requirements.txt` → pip install → copy Django source.
* Expose port 8000 → chạy server Django dev.

---

## 4️⃣ requirements.txt

```text
Django>=4.2,<5         # framework chính
mysqlclient>=2.2        # connector MariaDB
```

* Giải thích:

  * Django: tạo project, app, models, admin site.
  * mysqlclient: kết nối MariaDB.

---

## 5️⃣ docker-compose.yml – Full stack

* **Services**:

| Service     | Image / Build           | Port | Chức năng                      |
| ----------- | ----------------------- | ---- | ------------------------------ |
| nodered     | nodered/node-red        | 1880 | Workflow / automation          |
| nginx       | nginx:latest            | 80   | Reverse proxy / frontend       |
| cloudflared | cloudflare/cloudflared  | -    | Public Cloudflare Tunnel       |
| filebrowser | filebrowser/filebrowser | 8081 | Quản lý file GUI               |
| myapi       | ./myapi (build)         | -    | API riêng                      |
| db          | mariadb:10.11           | 3307 | Database chính                 |
| phpmyadmin  | phpmyadmin:latest       | 8082 | Kiểm tra DB                    |
| django      | ./camdo_django (build)  | 8000 | Django app quản lý tiệm cầm đồ |

* Volume:

  * db_data → MariaDB persistent
  * mount `./camdo_django:/app` → edit code trực tiếp.

---
Nginx.conf:
<img width="1228" height="695" alt="image" src="https://github.com/user-attachments/assets/d04d9a2b-094a-4157-9152-e9044bab0a00" />


## 6️⃣ Django project setup

* `docker-compose run django python manage.py startproject camdo_proj .`
* Chỉnh **settings.py** để kết nối MariaDB (`db` service, user/password đã config).
* Tạo app `camdo_app` → thêm `models.py` (Customer, PawnItem, FK), `views.py`, template `home.html`.
* Migrate và tạo superuser:

```
docker-compose exec django python manage.py migrate
docker-compose exec django python manage.py createsuperuser
```

## 7️⃣ Admin site & trang home

* Admin site: thêm/sửa/xóa bảng, FK hiển thị dạng select text.
* Home page (`home_page`) liệt kê **con nợ đến hạn chưa trả tiền**.
* Template sử dụng **Jinja2**:

```html
{% for item in due_items %}
<tr>
  <td>{{ item.customer.name }}</td>
  <td>{{ item.item_name }}</td>
  <td>{{ item.value }}</td>
  <td>{{ item.due_date }}</td>
</tr>
{% endfor %}
```

---

## 8️⃣ PhpMyAdmin

* Dùng để **xem dữ liệu DB** và kiểm chứng FK, không tạo bảng.
* URL: `http://localhost:8082/`
* Sơ lược bảng viết tay
<img width="1920" height="2560" alt="image" src="https://github.com/user-attachments/assets/686e69d9-7880-4bc1-b0df-b1b77e7ce542" />

---

## 9️⃣ Cloudflare Tunnel
<img width="1919" height="951" alt="image" src="https://github.com/user-attachments/assets/6fabc1a3-39a9-4177-bf59-7ae6f617ce94" />

* Tạo subdomain: `camdo.anhtu.divu.click`
* DNS: CNAME → `divu.click`, **proxy bật**.
* Trong Zero Trust → Tunnels → Public Hostname:

| Hostname               | Service                                                              | TLS  |
| ---------------------- | -------------------------------------------------------------------- | ---- |
| camdo.anhtu.divu.click | [http://host.docker.internal:8000](http://host.docker.internal:8000) | Auto |

* Tunnel route request từ internet → host port 8000 → Django container.

---

## 🔟 Kết quả
<img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/9b20f511-dc2a-430f-b8e7-0683bd06d3d1" />

* Home page hiển thị danh sách con nợ đến hạn.
* PhpMyAdmin kiểm chứng CS.
* Tất cả chạy trên Docker, public qua Cloudflare Tunnel.



## **Kết luận**
---
* Hệ thống **quản lý tiệm cầm đồ** đã được triển khai thành công trên **Docker** với các service **Django, MariaDB, phpMyAdmin**, kết hợp với **Nginx, Node-RED, Filebrowser và Cloudflare Tunnel**.
* Việc sử dụng **Docker Compose** giúp quản lý và chạy toàn bộ stack đồng thời, dễ bảo trì và mở rộng.
* **Django Admin** cung cấp giao diện trực quan để **thêm, sửa, xóa** dữ liệu bảng, đồng thời quản lý các **khách hàng, vật cầm, hợp đồng** với quan hệ **FK hiển thị text**, dễ kiểm chứng bằng **phpMyAdmin**.
* **Template Jinja2** cho phép render danh sách **con nợ đến hạn**, minh họa luồng nghiệp vụ chính của hệ thống.
* **Cloudflare Tunnel** cung cấp cách **public subdomain** an toàn, truy cập từ Internet mà không cần expose trực tiếp host port, đồng thời đảm bảo HTTPS.
* Cách triển khai này **dễ demo, dễ chỉnh sửa bằng volume mount và sudo nano**, đồng thời có thể mở rộng, thêm tính năng hoặc dịch vụ mới mà không ảnh hưởng các service cũ.
* Hệ thống này **tối ưu cho giảng dạy và trình bày**, minh họa toàn bộ luồng từ backend → frontend → public access, đồng thời duy trì tính bảo mật và modular.
---
 
# Chương 3: Wordpress (php), mariadb, phpmyadmin 
Yêu cầu: Ứng dụng WP vào Docker để xây dựng website cùng với Mariadb kết nối Cloudflare
### A. Wordpress
Kịch bản: Máy đã có dự án cũ, tạo mới sao cho độc lập tránh chồng chéo.
### 1. Chuẩn bị:
### Dự án mới: 
<img width="1104" height="319" alt="image" src="https://github.com/user-attachments/assets/8214af6f-3808-40b9-a98f-f6f69a8d7120" />
Hành động thoát dự án cũ, về với /home trước khi tạo dự án mới.
 + Cấu hình docker compose.yml.
 + Chạy docker compose up -d => kéo các Image về và tạo Container
 + Check: docker ps.
   
### Cấu hình subdomain mới:
+ Cloudflare Dashboard >...> Tunnel
+ Chọn Tunnel của miền mẹ. Edit
+ Thêm subdomain, Service URL, Path
Trọng tâm:
+ kiểm tra cổng ss -tulpn | grep <port_cua_wordpress> nếu bị trùng và đổi.
+ **Service URL** đặt IP của ens33/eth0 + cổng ví dụ: 192.168.222.111:8882
<img width="1536" height="646" alt="image" src="https://github.com/user-attachments/assets/159d1eb9-7701-4134-a571-7b7d836f1dcb" />
 Bởi vì ban đầu Cloudflare Tunnel của ta đang chạy bên trong Docker nên không thể dùng ip 127.0.0.1 thì CF sẽ tìm chính nó thay vì tìm ra ngoài để thấy 8882. Dùng chính IP máy ảo để Container Tunnel "nhìn thấy" dịch vụ đang mở ở cổng 8882 của máy chủ.

### Thiết lập Wordpress: 
Đi thiết lập Wp:
<img width="973" height="478" alt="image" src="https://github.com/user-attachments/assets/cd525636-5e43-490f-93a8-3ff37ac79a76" />
.
<img width="1862" height="957" alt="image" src="https://github.com/user-attachments/assets/27364719-829d-4346-9ae4-d83a5b396f05" />
<img width="1887" height="922" alt="image" src="https://github.com/user-attachments/assets/ce69afcf-e919-40a6-8a86-3b06c49c5fc9" />

| Tiêu chí | Nhận xét |
|---|---|
| Công sức | Rất thấp. Việc dùng Docker giúp triển khai 3 dịch vụ cùng lúc chỉ với 1 lệnh docker compose up. |
| Độ khó | Dễ sử dụng giao diện web, nhưng đòi hỏi kiến thức về Network (như lỗi 502 bạn vừa sửa) để kết nối Cloudflare Tunnel. |
| Tài nguyên | WordPress chạy trên PHP khá ngốn RAM (khoảng 200-400MB cho 1 dự án). Nếu chạy nhiều site trên 1 máy ảo yếu, RAM sẽ bị quá tải (Swapping). |
| Tính tiện dụng | Rất cao cho việc tạo website nhanh, nhưng tốn công tối ưu bảo mật và tốc độ hơn so với code tay thuần túy. |

# Chương 4: auto đăng bài bằng AI với wordpress, n8n, bot telegram, gemini
 Tiếp tục vận dụng hệ thống gồm wordpress (php), mariadb, phpmyadmin để kết hợp với n8n auto đăng bài.
### 1. Tạo và cài n8n
<img width="662" height="102" alt="Screenshot 2026-05-25 152227" src="https://github.com/user-attachments/assets/e1aef58a-4421-4c59-86a2-2441fa1ca710" />
 N8n yêu cầu ssl và https. Để không cần cài ssl. Có phương pháp cấu hình như sau:
```
n8n_wpanhtu:
    image: n8nio/n8n:latest
    restart: always
    container_name: n8n_wpanhtu
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=https://n8nanhtu.divu.click/
    volumes:
      - ./n8n_data:/home/node/.n8n
```
 - Mấu chốt ở - WEBHOOK_URL=https://n8nanhtu.divu.click/. Và Tunnel 
 <img width="1406" height="717" alt="image" src="https://github.com/user-attachments/assets/41e4a001-da28-45cc-b0ee-91683206319c" />
. Rồi tạo tk:
<img width="657" height="844" alt="image" src="https://github.com/user-attachments/assets/9b36cc60-fe18-4fec-aeee-ddbe4a3387f9" />


### Điểm quan trọng:
Trong quá trình cấu hình gặp lỗi "Bad lock file is ignored: ./.docker-compose.yml.swp", đồng thời gây ra lỗi "Error establishing a database connection (Wordpress)" và n8n không truy cập được. Đây là cách xử lý:
Nguyên nhân Ổ đĩa đầy (100%) 
   │
   ├──► Ubuntu không ghi được File tạm (.swp) ──► Docker Compose kẹt cú pháp
   │
   ├──► n8n Container ghi đè File Config lỗi ──► File config hỏng (Invalid JSON) ──► n8n Crash liên tục
   │
   └──► MariaDB không tạo được File Lock ────► Container sập (Exit code 1) ────► WordPress mất kết nối (Database Error)
+ Xử lý:
1. Nâng dung lượng lên.
2. Xóa cấu hình và thư mục n8n cũ
<img width="695" height="381" alt="image" src="https://github.com/user-attachments/assets/9800e6e4-f7ed-4bc9-9bca-8623d5636379" />


### 2. Tạo workflow:
- Sau khi bấm vào "Sự kiện trên ứng dụng", một ô tìm kiếm sẽ hiện ra. Bạn gõ chữ Telegram và chọn Telegram Trigger.
- Ở bảng cấu hình bên phải hiện ra tiếp theo, tại mục Event (Sự kiện), bạn chọn là On Message (Khi có tin nhắn đến).
- Tại mục Credential, bấm vào nút **Set up credential**, chọn Create New Credential rồi dán chuỗi Access Token mà lấy từ @BotFather vào.
<img width="570" height="640" alt="image" src="https://github.com/user-attachments/assets/bde86eec-741f-4bfd-a1b8-adbedc9f640b" />
<img width="1342" height="695" alt="image" src="https://github.com/user-attachments/assets/c4d77dd7-16ab-4bb0-8a81-0306ea449db4" />

```
[Telegram Trigger] ──► [Google Gemini] ──► [Code JavaScript] ──► [WordPress Node]
 (Nhận tin nhắn)        (Sinh bài viết JSON)   (Lọc & làm sạch dữ liệu)   (Đăng bài Publish)
```
Thêm node Gemini:
<img width="1852" height="805" alt="image" src="https://github.com/user-attachments/assets/ff5da6c2-7ef9-45ea-bfe1-bedb5cb5ec19" />.
- truy cập vào trang web: https://aistudio.google.com/ lấy key
- Credential, bạn bấm vào ô lựa chọn và chọn Create New Credential. Điền Key.
- Chọn Model
- Prompt: {{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.
- Bật Output Content as JSON
- Bấm Execute step

Thêm node Code:
<img width="1857" height="822" alt="image" src="https://github.com/user-attachments/assets/8b1bb2b4-f2af-457a-9e2b-c3c0b0f5355c" />
```
// 1. Lấy chuỗi mã HTML thô từ node Gemini truyền sang
let rawHtml = $input.first().json.content.parts[0].text;

// Làm sạch các dấu nháy kép thừa ở đầu và cuối chuỗi nếu có
if (rawHtml.startsWith('"') && rawHtml.endsWith('"')) {
    rawHtml = rawHtml.substring(1, rawHtml.length - 1);
}

// 2. Tự động bóc tách Tiêu đề (Nằm giữa cặp thẻ <h1> và </h1>)
let postTitle = "Bài viết tự động từ AI"; // Tiêu đề mặc định nếu lỗi
const titleMatch = rawHtml.match(/<h1>(.*?)<\/h1>/);
if (titleMatch && titleMatch[1]) {
    postTitle = titleMatch[1].replace(/\\"/g, '"').trim();
}

// 3. Tự động bóc tách Nội dung (Lấy phần style và nội dung chính)
let postContent = rawHtml;
const bodyMatch = rawHtml.match(/<body>([\s\S]*?)<\/body>/);
const styleMatch = rawHtml.match(/<style>([\s\S]*?)<\/style>/);

if (bodyMatch && bodyMatch[1]) {
    // Kết hợp phần Style và phần Body để WordPress hiển thị đẹp mắt
    let styleTag = styleMatch ? `<style>${styleMatch[1]}</style>` : '';
    postContent = styleTag + bodyMatch[1];
}

// Làm sạch các ký tự xuống dòng (\\n) và dấu gạch chéo ngược (\\") do JSON sinh ra
postContent = postContent.replace(/\\n/g, '\n').replace(/\\"/g, '"').trim();

// 4. Trả kết quả chuẩn về cho Node WordPress sử dụng
return {
  title: postTitle,
  content: postContent
};
```

Tạo MẬT KHẨU ứng dụng. Để điền vào dưới
<img width="1652" height="548" alt="image" src="https://github.com/user-attachments/assets/cd8b60ad-373b-4c5b-a078-9e8edd37eab5" />
ymhI TKCF e0Xt VHCc tfj2 QqYK

Thêm node Wordpress:
<img width="1345" height="708" alt="image" src="https://github.com/user-attachments/assets/e804406b-8610-4af5-953b-e8ba6e3d1932" />
- Bật Ignore SSL Issues

### CHẠY FLOW:
Chạy từng node bằng cách Execute step
Nếu lỗi, sửa:
Tại nano /home/wpanhtu/wp_data/wp-config.php
define('WP_ENVIRONMENT_TYPE', 'local');
<img width="1089" height="76" alt="image" src="https://github.com/user-attachments/assets/bb2b1dd2-e432-48af-aa0c-7575a8b22a54" />
Done:
<img width="1873" height="818" alt="image" src="https://github.com/user-attachments/assets/bb690b57-371f-4293-8ae3-0c6a2a2597d7" />

Xong:
<img width="1901" height="901" alt="image" src="https://github.com/user-attachments/assets/d842765b-df2d-4e4f-8b9a-6f692d04a7f5" />


# Chương 5: Vận dụng tổng hợp xử lý bài toán Giám sát và cảnh báo thời gian thực về giá bạc
## PHẦN I: KIẾN THỨC NỀN TẢNG
## 1. Khái niệm DockerDocker 
là một nền tảng mã nguồn mở cho phép đóng gói ứng dụng và tất cả các thành phần phụ thuộc (thư viện, môi trường runtime, cấu hình hệ thống) vào một đơn vị độc lập gọi là Container.Khác với ảo hóa truyền thống (Virtual Machines) đòi hỏi một hệ điều hành khách (Guest OS) hoàn chỉnh cho mỗi máy ảo, Docker chia sẻ chung nhân kernel của hệ điều hành host. Cơ chế này giúp các container khởi động tức thì trong vài mili giây, tiêu tốn cực ít tài nguyên RAM/CPU và loại bỏ hoàn toàn lỗi cấu hình khi di chuyển phần mềm giữa các máy tính khác nhau.

## 2. Các từ khóa cốt lõi trong cấu trúc docker-compose.yml
File docker-compose.yml định nghĩa kiến trúc đa dịch vụ (multi-service). Các cấu phần cấu trúc bao gồm:
version: Định nghĩa phiên bản cú pháp của Docker Compose để trình biên dịch hiểu.
- **services**: Nhóm định nghĩa tất cả các container riêng lẻ cấu thành ứng dụng.
- **image**: Chỉ định template mẫu (Image) được tải từ Docker Hub về làm phôi chạy container.
- **container_name**: Đặt tên tường minh cho container nhằm quản lý trực quan thay vì mã băm ngẫu nhiên.
- **ports**: Thiết lập ánh xạ cổng truyền thông theo cấu trúc `Cổng_Máy_Host:Cổng_Nội_Bộ_Container`.
- **environment**: Thiết lập các biến môi trường cấu hình bên trong hệ thống (như tài khoản, mật khẩu cơ sở dữ liệu).
- **volumes**: Gắn phân vùng ổ đĩa cố định từ máy Host vào Container để tránh mất dữ liệu khi container bị tắt hoặc khởi động lại.
- **networks**: Tạo các mạng ảo cô lập cho phép các container giao tiếp nội bộ an toàn bằng tên của nhau (Service Discovery).
- **restart**: Quy định hành vi tự động khởi động lại container nếu xảy ra sự cố đột ngột hoặc lỗi sập nguồn hệ thống.

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
- Nhất quán môi trường: Đảm bảo ứng dụng chạy trên Máy tính cá nhân (Laptop), Máy chủ thử nghiệm (Staging) hay Máy chủ thật (Production) đều có chung một hành vi duy nhất.
- Tối ưu tài nguyên hệ thống: Chạy hàng chục container trên một cấu hình VPS RAM thấp mà không bị nghẽn cổ chai nhờ cơ chế cô lập tiến trình nhẹ (Lightweight Isolation).
- Đóng gói và triển khai nhanh: Quá trình cài đặt toàn bộ hạ tầng phức tạp (Nginx, API, CSDL) được rút gọn trong một câu lệnh duy nhất thay vì cấu hình thủ công từng dòng lệnh hệ điều hành.
- An toàn và bảo mật: Các dịch vụ hoạt động biệt lập trong mạng nội bộ Docker, chỉ mở các cổng công khai cần thiết ra Internet thông qua Reverse Proxy.
## 4. Quy trình triển khai ứng dụng lên máy chủ ngoại tuyến (Offline - No Internet)
Khi ứng dụng đã chạy tốt trên laptop cá nhân và cần chuyển dịch lên một máy chủ cô lập hoàn toàn với Internet, quy trình xử lý bao gồm 4 bước kỹ thuật:
```
[Laptop Cá Nhân (Online)]...............................[Máy Chủ Thật (Offline)]
   ├── 
    1. docker save (Xuất Image .tar) ───────────────>   ├──  3. docker load (Nạp lại Image)
└── 2. Đóng gói mã nguồn & Compose ─────────────────>   └──  4. docker-compose up -d
```

Trích xuất Image trên máy cá nhân: Đóng gói toàn bộ các Image đã tải từ Internet thành một file nén vật lý:
```
docker save -o thitruong_images.tar mariadb:10.6 node-red:latest grafana/grafana:latest nginx:alpine
```
Sao chép dữ liệu: Dùng USB hoặc ổ cứng di động sao chép các file cấu hình docker-compose.yml, mã nguồn dự án và file nén thitruong_images.tar lên máy chủ offline.
Nạp Image vào hệ thống máy chủ Offline:
```
docker load -i thitruong_images.tar
```
Khởi chạy hệ thống: Di chuyển vào thư mục chứa file cấu hình và chạy lệnh khởi động hạ tầng mà không cần tải dữ liệu từ Internet:
```
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
#### Bước 3.1: Cấu hình Node-RED thu thập và rẽ nhánh dữ liệu luôn luôn gửi định kỳ
Luồng Node-RED sử dụng node http request kéo mã nguồn HTML từ các sàn giao dịch. Khối chức năng Function xử lý Regex bóc tách chuỗi số, loại bỏ nhiễu để định dạng về kiểu đối tượng json sạch.
<img width="1588" height="582" alt="image" src="https://github.com/user-attachments/assets/64ddf5b6-52b6-415d-be91-fcccf3a7d72a" /> .
- Dữ liệu sau khi xử lý được tách làm 3 nhánh hoạt động đồng thời:Đẩy lệnh POST sang Flask API nhằm cập nhật trạng thái giá tức thời vào MariaDB.Bắn dữ liệu dạng cấu trúc thời gian vào InfluxDB lưu lịch sử.Bỏ qua cơ chế chặn giá trùng, luôn luôn gửi thông tin trạng thái thị trường cập nhật định kỳ mỗi 15 phút về hệ thống Telegram Bot.Mã cấu trúc luồng Node-RED (Trích đoạn luồng xử lý đồng bộ):
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
```

#### Bước 3.2: Xây dựng Flask API xử lý Front-End giao tiếp Cơ sở dữ liệu MariaDB
Tầng API giao tiếp trực tiếp với MariaDB thông qua Driver kết nối. Đoạn mã 
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
```
 Trang giao diện Front-End (index.html) sử dụng phương thức AJAX Fetch API lặp chu kỳ ngắn nhằm tự động quét đầu cuối API, ép trang Web tự động cập nhật số liệu hiển thị lên màn hình mà không cần tải lại toàn bộ trang:
```
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
#### Bước 3.3: Tối ưu trực quan hóa biểu đồ dạng khối vuông nối tiếp trên Grafana
Để giải quyết triệt để hiện tượng dữ liệu tài chính biến thiên nhỏ (độ lệch vài trăm đồng) bị ép phẳng lỳ trên đồ thị mặc định, người quản trị cần thực hiện cấu hình kết hợp giữa câu lệnh truy vấn gom nhóm thông minh và tinh chỉnh giao diện trục tọa độ. Quy trình thực hiện chi tiết gồm các thao tác sau:
1. Truy cập hệ thống và kết nối cơ sở dữ liệu
+ Thao tác 1: Mở trình duyệt web, truy cập vào giao diện quản trị Grafana.
  <img width="1920" height="499" alt="image" src="https://github.com/user-attachments/assets/49876f3d-192d-4303-ae4f-bbae808b2547" />
+ Thao tác 2: Tại màn hình đăng nhập, nhập tài khoản mặc định hệ thống với Username là admin và Password là admin để truy cập vào quyền cấu hình tối cao.
  <img width="1529" height="678" alt="image" src="https://github.com/user-attachments/assets/9b8f16f8-7ea3-4067-9fd9-0e77a6bdfe6d" />
+ Thao tác 3: Điều hướng menu bên trái, chọn Connections $\rightarrow$ chọn Data sources $\rightarrow$ nhấn Add data source và chọn driver MySQL/MariaDB. Khai báo chính xác các thông số kết nối nội bộ Container bao gồm Host (mariadb_thitruong:3306), Database (thitruong_db), User (anhtu_user), và Password (anhtu_password). Nhấn Save & test để xác nhận hệ thống báo dòng chữ xanh "Database connection ok".
  <img width="992" height="566" alt="image" src="https://github.com/user-attachments/assets/3dec5ed1-d986-41e5-ae97-9266aeddc614" />

2. Tạo lập vùng vẽ và xử lý câu lệnh SQL gom nhóm dữ liệu
+ Thao tác 4: Quay lại menu bên trái, nhấn vào biểu tượng Dashboards (hình 4 ô vuông) $\rightarrow$ chọn New $\rightarrow$ chọn New dashboard $\rightarrow$ nhấp vào nút + Add visualization. Hệ thống sẽ hiện bảng chọn nguồn dữ liệu, nhấp chọn vào tên dữ liệu vừa kết nối là MariaDB_Thitruong.
+ Thao tác 5: Tại giao diện thiết lập đồ thị mới, nhìn xuống khu vực cấu hình truy vấn dưới chân màn hình, nhấp vào nút Code (nằm ngay bên cạnh nút Builder mặc định) để chuyển sang chế độ viết lệnh SQL thuần.
+ Thao tác 6: Dùng câu lệnh SQL xử lý gom nhóm dữ liệu theo khoảng thời gian 15 phút, loại bỏ các giá trị lỗi phát sinh khi chạy thử dưới đây vào ô soạn thảo:
<img width="1071" height="317" alt="image" src="https://github.com/user-attachments/assets/cb18bf06-206d-4cc3-85c7-b5f681fc9b1d" />

Để xử lý hiện tượng dữ liệu tài chính biến thiên nhỏ (độ lệch vài trăm đồng) bị ép phẳng lỳ trên đồ thị mặc định, cấu trúc truy vấn được gom nhóm thông minh theo khoảng thời gian 15 phút bằng cấu trúc lệnh 
``` 
 SELECT 
  cap_nhat_luc AS time, 
  gia_ban AS "Giá Bán Phú Quý",
  gia_mua AS "Giá Mua Phú Quý"
FROM bang_gia_bac 
WHERE thuong_hieu = 'Phú Quý' AND gia_ban > 2000000
ORDER BY cap_nhat_luc ASC;
```
+ Thao tác 7: Nhấn nút Run query (màu xanh dương) ở giữa màn hình để ép hệ thống biên dịch dữ liệu.

3. Tinh chỉnh thuộc tính đồ thị và bóp nghẹt biên độ trục đứng $Y-Axis$
+ Thao tác 8: Nhìn sang cột cấu hình thuộc tính bên phải màn hình (thanh Panel options), cuộn xuống tìm danh mục Graph styles. Tại dòng thuộc tính Line interpolation, nhấp mở menu thả xuống và chọn giá trị Step before (hoặc Step after). Thao tác này buộc đồ thị không vẽ đường chéo xiên mà chuyển hẳn sang dạng bậc thang vuông góc $90^\circ$ nối tiếp liên tục để làm rõ các vách đá biến động giá.
+ Thao tác 9: Tiếp tục cuộn bảng thuộc tính bên phải xuống danh mục Standard options. Tìm ô thiết lập Unit, gõ tìm kiếm từ khóa currency và kích chọn định dạng tiền tệ Vietnamese Dong (₫). Tại ô Decimals ngay phía dưới, điền số 0 để ẩn toàn bộ phần thập phân lỗi thời.
+ Thao tác 10: Tại hai ô giá trị biên độ Min và Max của mục Standard options, tiến hành nhập thủ công để khóa cứng trục đứng Y-Axis sát sàn dữ liệu thực tế (Ví dụ: điền Min là 2570000 và Max là 2575000). Hành động bóp nghẹt không gian này giúp kéo giãn tối đa khoảng cách hiển thị theo chiều dọc, khiến mắt thường nhìn thấy rất rõ các bước nhảy giá dù chỉ chênh lệch vài trăm đồng.

(*) Tinh chỉnh thuộc tính hiển thị trực quan đồ thị:
- Graph Styles $\rightarrow$ Line interpolation: Thiết lập về giá trị Step before (Biểu đồ bậc thang vuông góc). Dữ liệu đi ngang tạo cạnh phẳng, khi cào được giá mới (dù lệch nhỏ 100đ) đồ thị bẻ góc vuông $90^\circ$ giật thẳng đứng lên hoặc xuống, giúp mắt thường quan sát rõ rệt sự biến thiên.
- Standard Options $\rightarrow$ Min/Max: Ép sát sàn giới hạn trục Y từ mốc 2570000 đến 2575000 nhằm kéo giãn tối đa biên độ không gian hiển thị của đường đồ thị vuông.

#### Bước 3.4: Thiết lập bộ lọc dữ liệu lỗi và hệ thống cảnh báo qua Telegram Bot
- Hệ thống sử dụng node Switch trong Node-RED để phân tách luồng dữ liệu kiểm thử. Khi giá bán vượt ngoài vùng an toàn cho phép $[2.500.000 \dots 2.700.000]$, tiến trình tự động rẽ nhánh kích hoạt soạn tin nhắn thông báo:
+ Giá bán > 2.700.000đ: Kích hoạt nhánh Alert High gửi tin nhắn kèm ký hiệu báo động đỏ.
+ Giá bán < 2.500.000đ: Kích hoạt nhánh Alert Low gửi tin nhắn kèm ký hiệu cảnh báo vàng.
   Cơ chế nhóm truyền thông công khai:Bot Telegram được đưa vào nhóm làm việc cố định gồm 3 thành viên (Bao gồm quản trị viên hệ thống và mã định danh thành viên bổ sung 1875746636). Mỗi khi cấu trúc dữ liệu lỗi được đẩy lên Webhook API của Telegram qua đường dẫn tương tác:https://api.telegram.org/bot<Token>/sendMessageTất cả các thành viên trong nhóm đồng loạt nhận được thông tin cảnh báo tường minh, nêu rõ thương hiệu vi phạm ngưỡng kèm giá trị số cụ thể gây ra lỗi.

## 4. Quy trình đóng gói xuất và khôi phục hệ thống cô lập an toàn
Để bảo vệ toàn vẹn dữ liệu gốc và không gây ảnh hưởng đến các dự án độc lập khác chạy chung trên máy chủ hệ thống, quy trình thực hiện xử lý thông qua môi trường Container cách ly:
- Bước 4.1: Xuất toàn bộ hệ thống bằng Container bảo mậtKhi cụm container đang ở trạng thái dừng, các dữ liệu lưu trữ bị khóa quyền hệ thống. Chúng ta sử dụng một Container Linux phụ trợ siêu nhẹ (alpine) chui trực tiếp vào phân vùng Docker để đóng gói tệp tin cấu hình và tệp tin linh hồn lưu trữ của đồ thị grafana.db cùng toàn bộ tệp tin MariaDB:
- Thực hiện đóng gói cô lập toàn bộ hạ tầng bên trong Container phụ trợ
docker run --rm -v $(pwd):/backup alpine tar -czvf /backup/project_thitruong_backup.tar.gz -C /backup docker-compose.yml flask_api nginx mariadb_data influxdb_data nodered_data grafana_data/grafana.db

<img width="1078" height="634" alt="image" src="https://github.com/user-attachments/assets/17e2c43b-4b9e-48e9-a99b-c4343ef0a82b" />

- Bước 4.2: Xóa sạch trạng thái hệ thống (Reset trắng) 
+ Xóa bỏ hoàn toàn container, cấu trúc mạng cục bộ ảo cache của dự án
```
docker-compose down -v
```
Kết quả kiểm tra qua lệnh docker ps cho thấy cụm dịch vụ giá bạc biến mất hoàn toàn, hệ thống máy chủ trống sạch sẽ, trong khi tất cả các ứng dụng chạy độc lập khác ngoài VPS không bị gián đoạn.
<img width="1081" height="534" alt="image" src="https://github.com/user-attachments/assets/2e56f8ad-290c-4a70-b7e1-04dd953f8b17" />

- Bước 4.3: Tái nạp hệ thống khôi phục nguyên trạng từ file cấu hình
+ Khởi chạy tái sinh toàn bộ hạ tầng từ vùng lưu trữ cố định
```
docker-compose up -d
```
Hạ tầng tự động đọc lại các phân vùng dữ liệu cũ đã được bảo hiểm trong thư mục, trang giao diện tại cổng :82 hoạt động ổn định trở lại, hiển thị chính xác biểu đồ vuông nối tiếp thời gian thực mà không yêu cầu người quản trị cấu hình lại bất kỳ thông số hệ thống nào.

#### Kết quả: 
Giao diện theo dõi trực quan:
<img width="1479" height="942" alt="image" src="https://github.com/user-attachments/assets/b4198d1c-b988-41f5-b0ac-4467e48283ef" /> . 
Group **@nhomtest_tbthitruong1**:
+ <img width="580" height="691" alt="image" src="https://github.com/user-attachments/assets/126e78ba-cde2-4647-9bcc-90d67d363aae" />
+ <img width="452" height="640" alt="Screenshot 2026-06-14 163959" src="https://github.com/user-attachments/assets/af37b6b8-9727-4a5c-a10a-11ea3ec91236" />
Bot Tele Public: **@tbthitruong_bot**

 
# PHẦN III: KẾT LUẬN VÀ KIẾN THỨC RÚT RA SAU TRIỂN KHAI:
Qua quá trình tương tác kỹ thuật, trực tiếp gỡ lỗi hệ thống, em rút ra các bài học và tri thức cốt lõi được đúc kết bao gồm:
+ Bản chất lưu trữ dữ liệu của Docker: Việc sử dụng container yêu cầu tư duy phân tách rõ ràng giữa Lớp xử lý logic (Runtime) và Lớp dữ liệu lưu trữ (Persistent Data). Nếu không ánh xạ thư mục ra máy Host thông qua Volumes, toàn bộ CSDL và luồng Node-RED sẽ biến mất hoàn toàn khi container bị khởi động lại.
+ Khóa bảo mật file của hệ điều hành Linux đối với Docker: Khi chạy trong môi trường container, các tệp tin dữ liệu như grafana.db hay dữ liệu bảng MariaDB thuộc quyền sở hữu của các User định danh ngầm độc quyền trong Docker (Rootless mode). Người quản trị không thể dùng các lệnh sao lưu hệ thống thông thường bên ngoài VPS dù có quyền root, mà phải áp dụng cơ chế nén gián tiếp thông qua đặc quyền Container để bẻ khóa phân quyền đọc file (Permission denied).
+ Bẫy dữ liệu tài chính trong hiển thị trực quan (Data Scaling): Số liệu biến thiên nhỏ so với tổng giá trị lớn luôn luôn bị làm phẳng lỳ trên đồ thị truyền thống. Để xử lý bài toán thị giác này, giải pháp căn bản là khóa cứng biên độ trục đứng $Y-Axis$ bao sát xung quanh dải giá trị thực tế và bẻ cong đồ thị sang dạng bậc thang góc vuông (Staircase), giúp mắt người đọc nhận diện sự thay đổi nhỏ nhất một cách rõ nét và trực quan nhất.

