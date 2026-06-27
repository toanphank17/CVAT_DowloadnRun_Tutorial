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

## 🖥️ How to Use
Truy cập: Mở Chrome và truy cập http://localhost:8080.
Setup: Tạo Project -> Thêm các Labels (Vd: ape, can, cat...) -> Tạo Task.
Upload: Chọn ảnh từ thư mục ~/lm/test/000001/rgb/.
AI Annotation:
Mở giao diện gán nhãn -> Chọn biểu tượng Magic Wand ✨ (AI Tools) -> Segment Anything.
Chuột trái để thêm vùng vật thể, Chuột phải để loại bỏ vùng thừa. Nhấn Enter để hoàn thành.
3D Annotation: Sử dụng công cụ Cuboid để vẽ bao quanh vật thể xác định 6D Pose.

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
