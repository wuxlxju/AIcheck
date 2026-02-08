# 部署指南

## 生产环境部署步骤

### 1. 服务器要求

- Ubuntu 20.04+ 或 CentOS 7+
- Python 3.11+
- Node.js 18+
- Nginx
- PostgreSQL 或 MySQL（推荐PostgreSQL）

### 2. 后端部署

#### 2.1 安装依赖

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install gunicorn
```

#### 2.2 配置环境变量

创建 `.env` 文件：

```bash
# 数据库
DATABASE_URL=postgresql://user:password@localhost/aicheck

# 安全密钥（必须更改！）
SECRET_KEY=your-very-long-random-secret-key-here
JWT_SECRET_KEY=your-jwt-secret-key-here

# 邮件配置
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=true
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_DEFAULT_SENDER=your-email@gmail.com

# 前端URL
FRONTEND_URL=https://your-domain.com

# 文件存储
UPLOAD_FOLDER=/var/www/aicheck/uploads
PDF_FOLDER=/var/www/aicheck/pdf_reports
```

#### 2.3 使用Gunicorn运行

```bash
gunicorn -w 4 -b 127.0.0.1:5000 --timeout 120 --access-logfile - --error-logfile - app:app
```

#### 2.4 使用Systemd管理服务

创建 `/etc/systemd/system/aicheck-backend.service`:

```ini
[Unit]
Description=AI Check Backend
After=network.target

[Service]
User=www-data
WorkingDirectory=/path/to/backend
Environment="PATH=/path/to/backend/venv/bin"
ExecStart=/path/to/backend/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 --timeout 120 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务：

```bash
sudo systemctl enable aicheck-backend
sudo systemctl start aicheck-backend
```

### 3. 前端部署

#### 3.1 构建前端

```bash
cd frontend
npm install
npm run build
```

#### 3.2 部署到Nginx

将 `dist` 目录内容复制到 `/var/www/aicheck/frontend`

### 4. Nginx配置

参考项目根目录的 `nginx.conf` 文件，修改域名和路径后复制到 `/etc/nginx/sites-available/aicheck`

启用配置：

```bash
sudo ln -s /etc/nginx/sites-available/aicheck /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 5. SSL证书配置

使用 Let's Encrypt 免费SSL证书：

```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

证书会自动续期。

### 6. 数据库设置

#### PostgreSQL

```bash
sudo -u postgres psql
CREATE DATABASE aicheck;
CREATE USER aicheck_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE aicheck TO aicheck_user;
```

### 7. 文件权限

```bash
sudo mkdir -p /var/www/aicheck/uploads
sudo mkdir -p /var/www/aicheck/pdf_reports
sudo chown -R www-data:www-data /var/www/aicheck
sudo chmod -R 755 /var/www/aicheck
```

### 8. 防火墙配置

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### 9. 监控和日志

- 后端日志：`/var/log/aicheck/backend.log`
- Nginx日志：`/var/log/nginx/access.log` 和 `/var/log/nginx/error.log`
- 系统日志：`journalctl -u aicheck-backend -f`

### 10. 性能优化

1. **启用Gzip压缩**（在nginx.conf中）
2. **使用CDN**加速静态资源
3. **配置Redis**用于缓存和session存储
4. **数据库连接池**优化
5. **文件存储**考虑使用对象存储（如AWS S3、阿里云OSS）

### 11. 安全建议

1. ✅ 修改所有默认密码
2. ✅ 配置防火墙规则
3. ✅ 定期更新系统和依赖
4. ✅ 配置CORS白名单（生产环境）
5. ✅ 启用HTTPS
6. ✅ 配置备份策略
7. ✅ 监控异常访问

### 12. Docker部署（可选）

如果使用Docker：

```bash
docker-compose up -d
```

确保配置好环境变量文件。

## 故障排查

### 后端无法启动

1. 检查Python版本：`python3 --version`
2. 检查依赖：`pip list`
3. 查看日志：`journalctl -u aicheck-backend -n 50`

### 前端无法访问

1. 检查Nginx状态：`sudo systemctl status nginx`
2. 检查配置文件：`sudo nginx -t`
3. 查看错误日志：`sudo tail -f /var/log/nginx/error.log`

### 数据库连接失败

1. 检查数据库服务：`sudo systemctl status postgresql`
2. 测试连接：`psql -U aicheck_user -d aicheck`
3. 检查防火墙规则

