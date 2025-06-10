# Odoo Automation

Ansible playbook để tự động hóa việc cài đặt và cấu hình Odoo trên Docker.

## Mục lục
- [Yêu cầu](#yêu-cầu)
- [Hướng dẫn cài đặt](#hướng-dẫn-cài-đặt)
  - [Cài đặt Ansible](#1-cài-đặt-ansible)
  - [Cấu hình SSH](#2-cấu-hình-ssh)
  - [Clone và cấu hình](#3-clone-và-cấu-hình)
  - [Cấu hình Inventory](#4-cấu-hình-inventory)
  - [Chạy Playbook](#5-chạy-playbook)
- [Cấu trúc dự án](#cấu-trúc-dự-án)
- [Các role chính](#các-role-chính)
- [Biến cấu hình](#biến-cấu-hình)
- [Kiểm tra cài đặt](#kiểm-tra-cài-đặt)
- [Xử lý sự cố](#xử-lý-sự-cố)
- [Bảo mật](#bảo-mật)
- [Backup và Restore](#backup-và-restore)
- [Cập nhật](#cập-nhật)

## Yêu cầu
- Ubuntu 20.04+ (máy local và server remote)
- Python 3.6+
- Ansible 2.9+

## Hướng dẫn cài đặt

### 1. Cài đặt Ansible

#### Trên Ubuntu/Debian:
```bash
# Cập nhật package list
sudo apt update

# Cài đặt các package cần thiết
sudo apt install -y software-properties-common

# Thêm repository Ansible
sudo apt-add-repository --yes --update ppa:ansible/ansible

# Cài đặt Ansible
sudo apt install -y ansible

# Kiểm tra cài đặt
ansible --version
```

#### Trên macOS:
```bash
# Sử dụng Homebrew
brew install ansible

# Kiểm tra cài đặt
ansible --version
```

### 2. Cấu hình SSH

```bash
# Tạo SSH key nếu chưa có
ssh-keygen -t rsa -b 4096

# Copy SSH key lên server remote
ssh-copy-id drewsec@192.168.176.103

# Kiểm tra kết nối
ssh drewsec@192.168.176.103
```

### 3. Clone và cấu hình

```bash
# Clone repository
git clone https://github.com/your-repo/odoo-automation.git
cd odoo-automation

# Tạo file vault để lưu mật khẩu
ansible-vault create group_vars/vault.yml
```

Thêm các biến sau vào file vault.yml:
```yaml
---
vault_postgres_password: "your_secure_password_here"
vault_admin_password: "your_secure_admin_password_here"
```

### 4. Cấu hình Inventory

Kiểm tra và cập nhật file `inventory/production.ini`:
```ini
[odoo]
prod ansible_host=192.168.176.103

[odoo:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_become=true
```

### 5. Chạy Playbook

```bash
# Kiểm tra kết nối
ansible all -m ping

# Chạy playbook
ansible-playbook deploy.yml
```

## Cấu trúc dự án
```
odoo-automation/
├── ansible.cfg          # Cấu hình Ansible
├── deploy.yml           # Playbook chính
├── custom-addons/       # Addons tùy chỉnh
├── group_vars/          # Biến cho các nhóm
│   ├── all.yml         # Biến chung
│   ├── odoo.yml        # Biến cấu hình Odoo
│   └── vault.yml       # Biến bảo mật (mật khẩu)
├── inventory/           # File inventory
│   └── production.ini  # Cấu hình server
├── roles/              # Các role Ansible
│   ├── docker/        # Cài đặt Docker
│   ├── postgres/      # Cấu hình PostgreSQL
│   └── odoo/          # Cài đặt và cấu hình Odoo
└── README.md
```

## Các role chính

### 1. Role docker
- Cài đặt Docker Engine
- Cài đặt Docker Compose
- Cấu hình Docker daemon

### 2. Role postgres
- Tạo thư mục dữ liệu PostgreSQL
- Cấu hình quyền truy cập

### 3. Role odoo
- Clone repository docker-compose
- Cấu hình Odoo
- Khởi động các container

## Biến cấu hình

### Biến chung (group_vars/all.yml)
```yaml
compose_dir: /opt/odoo
compose_repo: https://github.com/vdx-vn/odoo-docker-compose.git
```

### Biến Odoo (group_vars/odoo.yml)
```yaml
# Phiên bản và ports
odoo_version: "18.0"
odoo_http_port: 8069
db_port: 5432

# Cấu hình database
db_user: odoo
db_password: "{{ vault_postgres_password }}"
db_name: odoo

# Docker images
postgres_image: postgres:15
odoo_image: odoo:{{ odoo_version }}

# Đường dẫn
custom_addons_path: "{{ compose_dir }}/custom-addons"
community_addons_path: "{{ compose_dir }}/community-addons"
postgres_data_dir: /var/lib/postgresql/data

# Mật khẩu admin
admin_password: "{{ vault_admin_password }}"
```

## Kiểm tra cài đặt

1. **Kiểm tra Docker**:
```bash
ssh drewsec@192.168.176.103
docker ps
docker-compose --version
```

2. **Kiểm tra Odoo**:
- Truy cập: http://192.168.176.103:8069
- Tài khoản: admin
- Mật khẩu: (mật khẩu đã cấu hình trong vault.yml)

3. **Kiểm tra logs**:
```bash
ssh drewsec@192.168.176.103
cd /opt/odoo
docker-compose logs
```

## Xử lý sự cố

### 1. Lỗi SSH
- Kiểm tra SSH key: `ls -la ~/.ssh/`
- Kiểm tra quyền: `chmod 600 ~/.ssh/id_rsa`
- Kiểm tra kết nối: `ssh -v drewsec@192.168.176.103`

### 2. Lỗi Docker
- Kiểm tra cài đặt: `docker --version`
- Kiểm tra service: `systemctl status docker`
- Kiểm tra logs: `journalctl -u docker`

### 3. Lỗi Odoo
- Kiểm tra container: `docker ps`
- Kiểm tra logs: `docker-compose logs web`
- Kiểm tra database: `docker-compose logs postgres`

## Bảo mật

### 1. File vault
- Không commit file vault.yml
- Sử dụng mật khẩu mạnh
- Bảo vệ file .vault_pass

### 2. SSH
- Sử dụng SSH key
- Vô hiệu hóa đăng nhập bằng mật khẩu
- Giới hạn IP truy cập

### 3. Firewall
- Chỉ mở các port cần thiết
- Sử dụng fail2ban

## Backup và Restore

### 1. Backup
```bash
# Backup database
ssh drewsec@192.168.176.103
cd /opt/odoo
docker-compose exec postgres pg_dump -U odoo odoo > backup.sql

# Backup files
tar -czf odoo-backup.tar.gz /opt/odoo
```

### 2. Restore
```bash
# Restore database
ssh drewsec@192.168.176.103
cd /opt/odoo
docker-compose exec -T postgres psql -U odoo odoo < backup.sql

# Restore files
tar -xzf odoo-backup.tar.gz -C /
```

## Cập nhật

### 1. Cập nhật Ansible
```bash
sudo apt update
sudo apt upgrade ansible
```

### 2. Cập nhật Odoo
```bash
cd /opt/odoo
docker-compose pull
docker-compose up -d
```

## Liên hệ

Nếu bạn gặp vấn đề hoặc cần hỗ trợ, vui lòng tạo issue trên GitHub repository.