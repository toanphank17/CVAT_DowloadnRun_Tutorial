🚀 Hướng dẫn Cài đặt CVAT (AI-Powered) & BOP Dataset từ A-Z
Tài liệu này hướng dẫn chi tiết cách thiết lập hệ thống gán nhãn dữ liệu CVAT (Computer Vision Annotation Tool), tích hợp công cụ AI Segment Anything Model (SAM) và tải dữ liệu mẫu từ BOP Challenge.
📌 Yêu cầu hệ thống
Hệ điều hành: Ubuntu 20.04/22.04 (hoặc WSL2 trên Windows).
Phần cứng: Khuyến nghị RAM 16GB+, ổ cứng trống ít nhất 40GB.
Trình duyệt: Google Chrome.
🛠 Bước 1: Cài đặt Docker (Bản chính thức)
Không sử dụng snap hoặc apt mặc định. Hãy chạy các lệnh sau để cài đặt Docker Engine:
code
Bash
# 1. Cập nhật hệ thống và cài đặt gói hỗ trợ
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 2. Thêm khóa GPG của Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 3. Thiết lập Repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
(. /etc/os-release && echo "(. /etc/os-release && echo "VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Cài đặt Docker & Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 5. Phân quyền chạy không cần sudo (Cần đăng xuất/đăng nhập lại sau bước này)
sudo usermod -aG docker $USER
newgrp docker
📦 Bước 2: Cài đặt và Chạy CVAT
Tải mã nguồn từ GitHub và khởi động hệ thống kèm tính năng hỗ trợ AI.
code
Bash
# 1. Clone mã nguồn
git clone https://github.com/cvat-ai/cvat
cd cvat

# 2. Khởi động với Serverless (AI)
docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml up -d

# 3. Tạo tài khoản quản trị (Superuser)
# Đợi 1 phút để hệ thống khởi động rồi mới chạy lệnh này
docker exec -it cvat_server bash -ic 'python3 ~/manage.py createsuperuser'
✨ Bước 3: Cài đặt AI Segment Anything (SAM)
Công cụ này giúp tự động tách nền vật thể chỉ bằng các cú click chuột.
code
Bash
# 1. Cài đặt bộ điều khiển nuctl
wget https://github.com/nuclio/nuclio/releases/download/1.13.0/nuctl-1.13.0-linux-amd64 -O nuctl
chmod +x nuctl
sudo mv nuctl /usr/local/bin/

# 2. Sửa lỗi Image Registry (Hack alpine)
docker pull alpine:3.17
docker tag alpine:3.17 gcr.io/iguazio/alpine:3.17

# 3. Deploy model SAM (Bản ViT-H mạnh nhất)
./serverless/deploy_cpu.sh serverless/pytorch/facebookresearch/sam/nuclio/
📊 Bước 4: Tải dữ liệu thử nghiệm (BOP Dataset)
Tải bản rút gọn (Lightweight) của bộ dữ liệu Linemod (lm) để tiết kiệm dung lượng (khoảng 1.5GB thay vì 34GB).
code
Bash
# Cài đặt thư viện
pip install huggingface_hub

# Chạy script tải chọn lọc
python3 -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='bop-benchmark/lm', repo_type='dataset', allow_patterns=['lm_base.zip', 'lm_models.zip', 'lm_test_all.zip'], local_dir='./lm')"

# Giải nén dữ liệu
cd lm
sudo apt install unzip -y
unzip "*.zip"
🖥️ Bước 5: Hướng dẫn Sử dụng
Truy cập: http://localhost:8080 bằng Chrome.
Tạo Project: Thêm các nhãn như ape, can, cat...
Tạo Task: Tải ảnh từ thư mục ~/lm/test/000001/rgb/.
Sử dụng AI: Trong giao diện gán nhãn, chọn biểu tượng Magic Wand (AI Tools) -> Segment Anything.
Chuột trái: Thêm vùng vật thể.
Chuột phải: Loại bỏ vùng nền.
Enter: Xác nhận gán nhãn.
Sử dụng Cuboid: Chọn công cụ khối lập phương để vẽ bao quanh vật thể (xác định Pose 3D).
🛠️ Lệnh Bảo trì
Dừng CVAT: docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml stop
Chạy lại CVAT: docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml start
Xóa bỏ hoàn toàn: docker compose down -v
