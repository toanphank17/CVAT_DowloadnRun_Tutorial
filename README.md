# 🚀 CVAT Installation Guide with AI Support (SAM) & BOP Dataset

Hướng dẫn chi tiết cách thiết lập hệ thống gán nhãn dữ liệu **CVAT** (Computer Vision Annotation Tool), tích hợp công cụ AI **Segment Anything Model (SAM)** và tải dữ liệu mẫu từ **BOP Challenge**.

---

## 📋 Requirements

### Software
* **Hệ điều hành:** Ubuntu 20.04/22.04 LTS (hoặc WSL2)
* **Python:** 3.8+
* **Docker Engine:** 20.10+
* **Docker Compose:** v2.0+
* **Trình duyệt:** Google Chrome (Khuyên dùng)

### Hardware (Khuyến nghị)
* **RAM:** 16GB+ (Để chạy mượt các model AI)
* **Ổ cứng:** Trống ít nhất 40GB

---

## 🛠 Step 1: Install Docker (Official)

Sử dụng các lệnh sau để cài đặt Docker Engine bản chính thức từ Repository thay vì bản snap.

```bash
# Cập nhật hệ thống và cài đặt gói hỗ trợ
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Thêm khóa GPG của Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Thiết lập Repository
echo "deb [arch=(dpkg−−print−architecture)signed−by=/etc/apt/keyrings/docker.gpg]https://download.docker.com/linux/ubuntu(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu (. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Cài đặt Docker & Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Cấp quyền chạy không cần sudo
sudo usermod -aG docker $USER
newgrp docker
```
---

### Phần 2: Cài đặt CVAT và Model AI (SAM)


## 📦 Step 2: Setup CVAT

Tải mã nguồn và khởi động CVAT kèm theo các dịch vụ serverless hỗ trợ AI.

```bash
# Clone mã nguồn từ GitHub
git clone https://github.com/cvat-ai/cvat
cd cvat

# Khởi động CVAT với Serverless (AI) component
docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml up -d

# Tạo tài khoản quản trị (Superuser)
# Đợi 30-60 giây để hệ thống khởi động hoàn toàn trước khi chạy lệnh này
docker exec -it cvat_server bash -ic 'python3 ~/manage.py createsuperuser'
```
## ✨ Step 3: Install AI Tools (Segment Anything)
Deploy model AI SAM để hỗ trợ gán nhãn tự động thông qua click chuột.
```bash
# Tải và cài đặt nuctl (Nuclio control tool)
wget https://github.com/nuclio/nuclio/releases/download/1.13.0/nuctl-1.13.0-linux-amd64 -O nuctl
chmod +x nuctl
sudo mv nuctl /usr/local/bin/

# Sửa lỗi Registry (Fix "gcr.io/iguazio/alpine" not found)
docker pull alpine:3.17
docker tag alpine:3.17 gcr.io/iguazio/alpine:3.17

# Deploy SAM Model (Bản ViT-H mạnh nhất)
./serverless/deploy_cpu.sh serverless/pytorch/facebookresearch/sam/nuclio/
```
### Phần 3: Tải Dataset, Hướng dẫn sử dụng và Bảo trì

## 📊 Step 4: Download BOP Dataset (Linemod) (Tùy thuộc vào dataset sẽ có cách tải khác nhau)

Tải bản rút gọn của bộ dữ liệu Linemod (10.5GB) thay vì bản đầy đủ (34GB).

```bash
# Cài đặt thư viện tải từ HuggingFace
pip install huggingface_hub

# Tải chọn lọc 3 file cần thiết (Base, Models, Test)
python3 -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='bop-benchmark/lm', repo_type='dataset', allow_patterns=['lm_base.zip', 'lm_models.zip', 'lm_test_all.zip'], local_dir='./lm')"

# Giải nén dữ liệu
cd lm
sudo apt install unzip -y
unzip "*.zip"
```

---

## 🖥️ Hướng dẫn sử dụng chi tiết (Workflow)

Để bắt đầu gán nhãn dữ liệu BOP với CVAT và AI hỗ trợ, hãy thực hiện theo quy trình 5 bước sau:

### 1. Khởi động & Đăng nhập
*   **Trình duyệt:** Sử dụng **Google Chrome** (Bắt buộc để đảm bảo tính năng AI hoạt động mượt mà).
*   **Địa chỉ:** Truy cập `http://localhost:8080`.
*   **Đăng nhập:** Sử dụng tài khoản **Superuser** đã tạo ở Bước 2.

### 2. Thiết lập Cấu trúc dự án (Project & Labels)
*   **Create Project:** Nhấn nút `+` -> `Create a new project`. Đây là "thùng chứa" lớn để quản lý nhiều Task.
*   **Setup Labels:** Trong mục **Labels**, thêm tên các vật thể bạn cần gán nhãn (Ví dụ: `ape`, `can`, `cat`...). Việc định nghĩa nhãn ở cấp Project sẽ giúp đồng bộ hóa cho tất cả các Task bên trong.

### 3. Tạo Task & Nhập dữ liệu (Data Import)
*   **Create Task:** Vào Project vừa tạo -> Nhấn `Create a new task`.
*   **Upload Data:** 
    *   Chọn tab **Local files**.
    *   Tìm đến thư mục chứa ảnh BOP đã giải nén: `~/lm/test/000001/rgb/`.
    *   **Lưu ý:** Chỉ nên chọn khoảng 50-100 ảnh cho lần thử đầu tiên để tránh treo trình duyệt.
*   **Submit:** Nhấn `Submit` và đợi hệ thống xử lý ảnh. Sau đó, nhấn vào số **Job** hiện ra để bắt đầu làm việc.

### 4. Gán nhãn tự động với AI (SAM - Segment Anything)
Đây là công cụ mạnh nhất giúp bạn tạo Mask (vùng chọn) chỉ trong vài giây:
1.  Nhấn phím tắt **`I`** hoặc chọn biểu tượng **Magic Wand ✨ (AI Tools)** ở thanh công cụ bên trái.
2.  Chọn model **Segment Anything**.
3.  **Thao tác chuột:**
    *   **Chuột trái (Điểm xanh):** Click vào thân vật thể để báo cho AI biết vùng cần lấy.
    *   **Chuột phải (Điểm đỏ):** Click vào vùng nền hoặc vùng bị lấn sang vật thể khác để loại bỏ.
4.  **Hoàn thành:** Sau khi AI bao quanh đúng vật thể, nhấn **Enter** để lưu vùng chọn dưới dạng Đa giác (Polygon).

### 5. Gán nhãn 3D Cuboid (Cho 6D Pose Estimation)
Để xác định vị trí và hướng của vật thể trong không gian 3D (phục vụ bộ dữ liệu BOP):
1.  Chọn biểu tượng **Draw new cuboid** (Khối lập phương) ở thanh công cụ.
2.  Vẽ một khối hộp bao quanh vật thể trên ảnh 2D.
3.  Điều chỉnh các điểm nắm (anchors) để xoay và kéo giãn khối hộp sao cho các cạnh của khối 3D khớp chính xác với phối cảnh của vật thể.

---
💡 **Mẹo nhỏ:** Nhấn **`Ctrl + S`** thường xuyên để lưu tiến độ công việc lên server, tránh mất dữ liệu khi trình duyệt gặp sự cố.



https://github.com/user-attachments/assets/c0d23408-61ec-418d-8f24-1b430a34ed3a






## 🛠 Maintenance
Management Commands
```bash
# Dừng CVAT (Tiết kiệm tài nguyên máy)
docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml stop

# Khởi động lại hệ thống
docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml start

# Xóa bỏ hoàn toàn dữ liệu và container (Làm sạch hệ thống)
docker compose down -v
```
