# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ
# HỌ VÀ TÊN: NGUYỄN MẠNH HIẾU - MSSV: K225480106020 - LỚP: K58KTP
_____
## WEBSITR QUẢN LÝ TIỆM CẦM ĐỒ

### Giới thiệu

Hệ thống Quản lý Tiệm Cầm đồ được phát triển dựa trên framework Django (Python) và vận hành hoàn toàn trong môi trường container hóa (Docker). Cấu trúc hạ tầng bao gồm 3 dịch vụ chính:

| Tên Dịch vụ | Vai trò trong hệ thống | Cổng (Port) |
| :--- | :--- | :--- |
| **MariaDB** | Quản trị và lưu trữ Cơ sở dữ liệu quan hệ | 3306 |
| **phpMyAdmin** | Công cụ giao diện Web để giám sát và thao tác trực tiếp CSDL | 8080 |
| **Django App** | Core Backend xử lý logic và phục vụ giao diện người dùng | 8000 |

### Các tính năng trọng tâm
* Vận hành mượt mà bằng Docker Compose chỉ với một lệnh khởi tạo.
* Giao diện quản trị (Admin Dashboard) được Django tự động sinh ra, hỗ trợ đầy đủ các thao tác CRUD (Thêm, Đọc, Sửa, Xóa).
* Xử lý thông minh các Khóa ngoại (Foreign Key), tự động ánh xạ thành danh sách lựa chọn (Dropdown) trong trang Admin.
* Tích hợp View và Template (Jinja2) để truy vấn và hiển thị danh sách các "Phiếu cầm đồ" đã quá hạn thanh toán.
* Cấu hình public hệ thống ra mạng Internet thông qua Cloudflare Tunnel.

### TỔ CHỨC CSDL

 <img width="1950" height="2556" alt="csdlcamdo" src="https://github.com/user-attachments/assets/3708fd7d-8d1a-4e78-b564-dbe8bf3af85d" />

### Chi tiết Cấu trúc Bảng

#### Bảng NguoiVay
Lưu trữ thông tin định danh của khách hàng mang tài sản đến cầm cố.

| Tên trường | Kiểu dữ liệu | Đặc tả |
| :--- | :--- | :--- |
| `ma_khach` | INT (PK, Auto Increment) | Khóa chính của bảng |
| `cccd` | VARCHAR(20) UNIQUE | Căn cước công dân (Bắt buộc, Không trùng lặp) |
| `ho_ten` | VARCHAR(100) | Họ và tên đầy đủ |
| `sdt` | VARCHAR(15) | Số điện thoại liên lạc |
| `dia_chi` | TEXT | Địa chỉ nơi cư trú |

#### Bảng LoaiTaiSan
Phân nhóm các loại tài sản để hệ thống dễ dàng áp dụng mức quy định lãi suất tương ứng.

| Tên trường | Kiểu dữ liệu | Đặc tả |
| :--- | :--- | :--- |
| `ma_loai` | INT (PK, Auto Increment) | Khóa chính của bảng |
| `ten_loai` | VARCHAR(100) | Nhóm tài sản (VD: Điện thoại, Xe máy...) |
| `quy_dinh_lai` | FLOAT | Mức lãi suất trần quy định (%) |

#### Bảng HopDong
Bảng trung tâm của hệ thống, lưu trữ thông tin giao dịch, tài sản cầm cố và thời hạn.

| Tên trường | Kiểu dữ liệu | Đặc tả |
| :--- | :--- | :--- |
| `id` | INT (PK, Auto Increment) | Khóa chính tự sinh của bảng |
| `ma_hop_dong` | VARCHAR(50) UNIQUE | Mã tra cứu hợp đồng (VD: HD-001) |
| `id_nguoi_vay` | INT (Foreign Key) | Khóa ngoại tham chiếu đến bảng NguoiVay |
| `id_loai_tai_san`| INT (Foreign Key) | Khóa ngoại tham chiếu đến bảng LoaiTaiSan |
| `tai_san_cam` | VARCHAR(255) | Mô tả cụ thể tài sản (VD: iPhone 15 Pro Max) |
| `so_tien_cam` | DECIMAL | Khoản tiền đã giải ngân cho khách |
| `ngay_cam` | DATE | Ngày ký kết hợp đồng |
| `ngay_den_han` | DATE | Ngày bắt buộc thanh toán / chuộc đồ |
| `trang_thai` | VARCHAR(50) | Tình trạng (Đang cầm / Đã chuộc / Quá hạn) |

#### Bảng LichSuGiaoDich
Bảng đối soát, ghi nhận chi tiết mỗi khi khách hàng thực hiện nghĩa vụ đóng tiền.

| Tên trường | Kiểu dữ liệu | Đặc tả |
| :--- | :--- | :--- |
| `ma_giao_dich` | INT (PK, Auto Increment) | Khóa chính của bảng |
| `id_hop_dong` | INT (Foreign Key) | Khóa ngoại tham chiếu đến bảng HopDong |
| `ngay_giao_dich` | DATE | Ngày phát sinh giao dịch đóng tiền |
| `loai_giao_dich` | VARCHAR(50) | Phân loại (Đóng lãi / Chuộc đồ / Gia hạn) |
| `so_tien` | DECIMAL | Số tiền khách hàng đã thanh toán |

### Cấu Trúc Thư Mục

```text
tiem_cam_do/
├── docker-compose.yml       ← Định nghĩa 3 service (MariaDB, phpMyAdmin, Django)
├── Dockerfile               ← Cấu hình build image Python + Django
├── requirements.txt         ← Danh sách thư viện pip (Django, mysqlclient)
├── .env                     ← Chứa thông tin bảo mật CSDL (KHÔNG push lên git)
├── .gitignore               ← Bỏ qua các file rác .env, __pycache__...
├── README.md                ← Tài liệu mô tả và hướng dẫn cài đặt
├── manage.py                ← File thực thi lệnh quản lý của Django
├── core/                    ← Django Project (Thư mục cấu hình tổng của hệ thống)
│   ├── settings.py          ← Cấu hình kết nối MariaDB, đăng ký App...
│   ├── urls.py              ← Điều hướng đường dẫn (Route) tổng
│   └── wsgi.py              
└── camdo/                   ← Django App (Nghiệp vụ chính của Tiệm cầm đồ)
    ├── models.py            ← Định nghĩa 4 bảng CSDL (NguoiVay, LoaiTaiSan...)
    ├── admin.py             ← Đăng ký hiển thị các bảng lên trang Admin
    ├── views.py             ← Xử lý logic truy vấn (View lọc con nợ đến hạn)
    ├── urls.py              ← Điều hướng đường dẫn riêng của app
    └── templates/
        └── camdo/
            └── home.html    ← File template (Jinja2) hiển thị danh sách con nợ
```

### Khởi chạy

- Dùng lệnh `mkdir tiem_cam_do` && `cd tiem_cam_do` để tạo thư mục và truy cập vào thư mục

  <img width="445" height="79" alt="image" src="https://github.com/user-attachments/assets/5883c91c-6120-4bee-a05d-ed75eb555c65" />

- Dùng lệnh `sudo nano requirements.txt` khi cửa sổ nano hiện ra thêm nội dung vào file và `Ctrl + O` để lưu và `Ctrl + X` để thoát

  <img width="972" height="339" alt="image" src="https://github.com/user-attachments/assets/440e951f-efed-45d6-a80d-e4c2b8998b82" />

- Dùng lệnh `sudo nano Dockerfile` khi cửa sổ nano hiện ra thêm nội dung vào file và `Ctrl + O` để lưu và `Ctrl + X` để thoát

  <img width="1240" height="708" alt="image" src="https://github.com/user-attachments/assets/e9d2d108-2df5-42fe-86f5-c6d8a70f4b01" />

- Dùng lệnh `sudo nano docker-compose.yml` khi cửa sổ nano hiện ra thêm nội dung vào file và `Ctrl + O` để lưu và `Ctrl + X` để thoát

  <img width="1336" height="860" alt="Screenshot 2026-05-09 135040" src="https://github.com/user-attachments/assets/4b52d4d9-02d6-4405-a1b5-5cb312dca323" />

- Sau khi thực hiên tạo và cấu hình xong 3 file trên, chạy 2 lệnh sau:
  
  - `docker compose run --rm web django-admin startproject core . ` # Khởi tạo cấu trúc project Django (sinh ra file settings.py và manage.py)
 
    <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e9d65bd9-4441-4d7e-b743-13a55dda069c" />
    
  - `docker compose up -d ` # Chạy toàn bộ hệ thống ở chế độ ngầm (-d)
 
    <img width="1468" height="214" alt="image" src="https://github.com/user-attachments/assets/47f59100-25d0-4b2d-a9db-6220d9d6a806" />

### Cấu hình ứng dụng và CSDL

#### Bước 1: Tạo App "Nghiệp vụ cầm đồ" 

- Chạy lệnh `docker compose exec web python manage.py startapp camdo` để tạo một App tên là camdo (cùng cấp với thư mục core)

#### Bước 2: Cấu hình settings.py (Khai báo App & Kết nối MariaDB)

- Dùng lệnh `sudo nano core/settings.py` để sửa file cài đặt gốc

- Khai báo App `camdo`: tìm đoạn INSTALLED_APPS = [...], thêm 'camdo' vào cuối danh sách

  <img width="1282" height="707" alt="image" src="https://github.com/user-attachments/assets/cba676c9-cc7c-4906-9b06-087593e1c2a6" />

- Đổi SQLite sang MariaDB: tìm đoạn DATABASES = {...} xóa hoặc comment (#) toàn bộ đoạn code cũ của SQLite đi và thêm đoạn kết nối MariaDB vào

  <img width="1466" height="764" alt="Screenshot 2026-05-09 141046" src="https://github.com/user-attachments/assets/820ba8cd-f3d5-4dde-814a-e30525fea694" />

#### Bước 3: Ánh xạ sơ đồ CSDL vào models.py

- Chạy lệnh `sudo nano camdo/models.py` mở file models.py của app camdo và thêm code vào

  <img width="1408" height="770" alt="image" src="https://github.com/user-attachments/assets/b8909024-e9f7-415a-8486-6e79983608eb" />

#### Bước 4: Thực thi Migrate (đồng bộ CSDL)

- Chạy lệnh `docker compose exec web python manage.py makemigrations` để tạo file lịch sử thay đổi CSDL

  <img width="1447" height="280" alt="image" src="https://github.com/user-attachments/assets/e36a2ed1-b4a4-4184-a6b8-b3e1c89c7d87" />

- Chạy lệnh `docker compose exec web python manage.py migrate` để thêm bảng vào MariaDB

  <img width="1425" height="605" alt="image" src="https://github.com/user-attachments/assets/c06f29a5-6228-42e1-93ac-29c0fa32104f" />

### Giao diện quản trị

#### Bước 1: Tạo tài khoản Superuser

- Chạy lệnh `docker compose exec web python manage.py createsuperuser` để tạo một tài khoản để đăng nhập vào hệ thống Admin

- Hệ thống sẽ hỏi 3 thông tin: Username, Email address, Password, điền thông tin vào

  <img width="1434" height="173" alt="image" src="https://github.com/user-attachments/assets/ee9e2021-4d1d-4aa7-b6d3-f6ab564dfd22" />

#### Bước 2: Hiển thị các bảng lên Web Admin (admin.py)

- Mặc định Django sẽ giấu các bảng đi để nó hiện lên trang Admin phải khai báo. Dùng lệnh `sudo nano camdo/admin.py` nano mở file admin.py

  <img width="1435" height="765" alt="image" src="https://github.com/user-attachments/assets/6e028163-3737-471f-9643-f6e644afe724" />

#### Bước 3: Xem thành quả

- Mở trình duyệt trên Windows, gõ: http://192.168.153.128:8000/admin

- Khi truy cập sẽ thấy như ảnh sau:

  <img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/d3e6815e-5794-4c10-985e-fc7e1d47ab74" />

- **Nguyên nhân:** Đây là một tính năng bảo mật mặc định của Django. Nó chỉ cho phép truy cập từ localhost (chính bản thân máy Ubuntu) và vì em đang dùng Windows để truy cập vào địa chỉ IP 192.168.153.128, Django thấy "người lạ" nên nó chặn lại.

- **Cách khắc phục:** Sửa file settings.py: chạy lệnh `sudo nano core/settings.py` tìm ALLOWED_HOSTS = [] và thêm dấu * (Cho phép mọi IP kết nối tới)
 
  <img width="1433" height="769" alt="image" src="https://github.com/user-attachments/assets/53f9a6c4-a0d8-4fd8-9286-d7bf8706b28a" />

- Mở lại trình duyệt trên Windows và gõ: http://192.168.153.128:8000/admin sẽ thấy trang login, đăng nhập với tài khoản admin đã tạo trước đó

  <img width="1905" height="1026" alt="image" src="https://github.com/user-attachments/assets/b746df2b-b0c2-4728-bf7d-ab89f87e63e0" />

  <img width="1919" height="1027" alt="image" src="https://github.com/user-attachments/assets/9050f92b-b6b6-467a-9816-f89d1fa718a0" />

- Hệ thống hỗ trợ đầy đủ các thao tác Thêm/Sửa/Xóa.
  
  <img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/4d9d0a48-8285-48e9-989f-4152b9fb03bf" />

- Hệ thống đã ánh xạ khóa ngoại (Foreign Key) chỉ việc chọn text
  
  <img width="1919" height="1025" alt="image" src="https://github.com/user-attachments/assets/28c8f040-d9a6-4553-a548-2fca67cc6828" />

### Liệt kê con nợ đến hạn bằng jinja2

- Viết Logic lọc con nợ (views.py): chạy lệnh `sudo nano camdo/views.py` để tạo và mở nano file và thêm code

  <img width="1438" height="771" alt="image" src="https://github.com/user-attachments/assets/ec46a515-2773-44e4-9ffe-08c27079932e" />

- Khai báo đường dẫn cho App (urls.py của camdo): chạy lệnh `sudo nano camdo/urls.py` để tạo và mở nano file và thêm nội dung

  <img width="1428" height="772" alt="image" src="https://github.com/user-attachments/assets/c0b12711-6efd-4ac0-9882-6567de8ea32c" />

- Khai báo đường dẫn tổng (urls.py của core): chạy lệnh `sudo nano core/urls.py` để tạo và mở nano file và thêm code

  <img width="1431" height="775" alt="image" src="https://github.com/user-attachments/assets/3d4161b5-6ea5-4a49-bf14-12088ce2043e" />

- Tạo giao diện hiển thị (Jinja2 Template): tạo đúng cấu trúc thư mục chứa file HTML và thêm code:
 
    ```text
    sudo mkdir -p camdo/templates/camdo
    sudo nano camdo/templates/camdo/home.html
    ```

   <img width="1443" height="778" alt="image" src="https://github.com/user-attachments/assets/266789a5-7686-4f30-b9e8-3ede9878a9f3" />

### Kết quả

<img width="1919" height="1025" alt="image" src="https://github.com/user-attachments/assets/16b88247-0a6f-41db-a94c-2cb8f328ba3e" />

### Public hệ thống bằng Cloudflare Tunnel

- Đăng nhập vào tài khoản Cloudflare bằng cách chạy lệnh `cloudflared tunnel login`, Terminal sẽ in ra một cái link rất dài copy link đó dán vào trình duyệt trên Windows và trình duyệt sẽ yêu cầu đăng nhập Cloudflare. Sau khi đăng nhập, chọn đúng tên miền nguyenmanhhieu.id.vn và bấm Authorize (Cấp quyền). Nếu thành công Terminal trên Ubuntu sẽ báo đã tải xong chứng chỉ (cert.pem)

  <img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/68b51fd1-8a4e-4b39-8c87-09f5801918a9" />

- Tạo một "đường hầm" đích danh: Tạo một tunnel có tên là tiemcamdo bằng lệnh: `cloudflared tunnel create tiemcamdo`

  <img width="1445" height="152" alt="image" src="https://github.com/user-attachments/assets/3a2f11f6-d840-414e-aa65-f7161ef2322c" />

- Gắn tên miền vào đường hầm: tạo một sub-domain là camdo.nguyenmanhhieu.id.vn bằng lệnh `cloudflared tunnel route dns tiemcamdo camdo.nguyenmanhhieu.id.vn`. Cloudflare sẽ tự động chui vào phần quản lý DNS và tạo một bản ghi CNAME trỏ về tunnel này

  <img width="1436" height="94" alt="image" src="https://github.com/user-attachments/assets/b0915c09-0234-402b-a100-6b6475629a2d" />

- Chạy lệnh `cloudflared tunnel run --url http://localhost:8000 tiemcamdo` cho Cloudflare dẫn toàn bộ traffic từ tên miền đó vào cổng 8000 của Django

  <img width="1452" height="948" alt="Screenshot 2026-05-09 153945" src="https://github.com/user-attachments/assets/4458807d-775b-4d34-a238-d8cdfe7947a7" />

- Thành quả: truy cập  https://camdo.nguyenmanhhieu.id.vn

  <img width="1919" height="1026" alt="image" src="https://github.com/user-attachments/assets/7788c0ff-f642-4ed5-9f53-33f69726d61f" />
