# Storm AI Detect - 国际部署指南

## 面向全球用户的部署方案

### 方案一：DigitalOcean 部署（推荐）⭐⭐⭐⭐⭐

**优点：**
- 价格便宜（$6/月起）
- 全球访问速度快
- 操作简单
- 适合国内外用户

---

## 详细部署步骤

### 第一步：注册 DigitalOcean

1. 访问 https://www.digitalocean.com
2. 注册账户（使用信用卡或PayPal）
3. 新用户可获得 $200 免费额度（60天）

### 第二步：创建 Droplet（服务器）

1. 点击 "Create" → "Droplets"
2. 选择配置：
   - **镜像：** Ubuntu 22.04 LTS
   - **套餐：** Basic ($12/月)
     - 2 CPU
     - 4GB RAM
     - 80GB SSD
     - 4TB 流量
   - **数据中心：** 
     - 美国用户：New York 或 San Francisco
     - 欧洲用户：London 或 Frankfurt
     - 亚洲用户：Singapore
     - **推荐：Singapore（新加坡）** - 对中国和全球都较快
   - **认证：** SSH Key（推荐）或密码
   - **主机名：** stormaidetect

3. 点击 "Create Droplet"
4. 等待1-2分钟创建完成
5. 记录服务器IP地址：`_______________`

### 第三步：域名配置

#### 购买域名
- **Namecheap**: https://www.namecheap.com
- **GoDaddy**: https://www.godaddy.com
- 搜索并购买 `stormaidetect.com`

#### 配置DNS
在域名控制台添加记录：

| 类型 | 主机 | 值 | TTL |
|------|------|-----|-----|
| A | @ | 你的DigitalOcean IP | 300 |
| A | www | 你的DigitalOcean IP | 300 |

### 第四步：连接服务器

```bash
# Windows 用户使用 PowerShell 或 PuTTY
# Mac/Linux 用户使用终端

ssh root@your-server-ip
```

### 第五步：安装必要软件

```bash
# 更新系统
apt update && apt upgrade -y

# 安装 Python 3.11
apt install software-properties-common -y
add-apt-repository ppa:deadsnakes/ppa -y
apt update
apt install python3.11 python3.11-venv python3.11-dev -y

# 安装 Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install nodejs -y

# 安装 Nginx
apt install nginx -y

# 安装 PostgreSQL
apt install postgresql postgresql-contrib -y

# 安装 Git
apt install git -y

# 安装 Certbot（SSL证书）
apt install certbot python3-certbot-nginx -y
```

### 第六步：配置数据库

```bash
# 切换到 postgres 用户
sudo -u postgres psql

# 创建数据库和用户
CREATE DATABASE stormaidetect;
CREATE USER stormaidetect_user WITH PASSWORD 'your_strong_password_here';
GRANT ALL PRIVILEGES ON DATABASE stormaidetect TO stormaidetect_user;
\q
```

### 第七步：部署代码

```bash
# 创建项目目录
mkdir -p /var/www/stormaidetect
cd /var/www/stormaidetect

# 克隆代码（或上传）
git clone https://github.com/your-username/stormaidetect.git .

# 或使用 SCP 上传
# 在本地执行：scp -r E:/AICheck root@your-ip:/var/www/stormaidetect/
```

### 第八步：配置后端

```bash
cd /var/www/stormaidetect/backend

# 创建虚拟环境
python3.11 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
pip install gunicorn

# 创建 .env 文件
nano .env
```

`.env` 文件内容：

```bash
DATABASE_URL=postgresql://stormaidetect_user:your_strong_password_here@localhost/stormaidetect
SECRET_KEY=storm-ai-detect-secret-key-2024-change-this
JWT_SECRET_KEY=storm-jwt-secret-2024-change-this

MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=true
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-gmail-app-password
MAIL_DEFAULT_SENDER=noreply@stormaidetect.com

FRONTEND_URL=https://stormaidetect.com

UPLOAD_FOLDER=/var/www/stormaidetect/uploads
PDF_FOLDER=/var/www/stormaidetect/pdf_reports
```

创建目录：

```bash
mkdir -p /var/www/stormaidetect/uploads
mkdir -p /var/www/stormaidetect/pdf_reports
chmod 755 /var/www/stormaidetect/uploads
chmod 755 /var/www/stormaidetect/pdf_reports
```

### 第九步：配置 Systemd 服务

```bash
nano /etc/systemd/system/stormaidetect.service
```

内容：

```ini
[Unit]
Description=Storm AI Detect Backend
After=network.target

[Service]
User=root
WorkingDirectory=/var/www/stormaidetect/backend
Environment="PATH=/var/www/stormaidetect/backend/venv/bin"
ExecStart=/var/www/stormaidetect/backend/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 --timeout 120 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务：

```bash
systemctl daemon-reload
systemctl enable stormaidetect
systemctl start stormaidetect
systemctl status stormaidetect
```

### 第十步：构建前端

```bash
cd /var/www/stormaidetect/frontend

# 安装依赖
npm install

# 构建
npm run build
```

### 第十一步：配置 Nginx

```bash
nano /etc/nginx/sites-available/stormaidetect
```

内容：

```nginx
server {
    listen 80;
    server_name stormaidetect.com www.stormaidetect.com;

    location / {
        root /var/www/stormaidetect/frontend/dist;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    client_max_body_size 10M;
}
```

启用配置：

```bash
ln -s /etc/nginx/sites-available/stormaidetect /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

### 第十二步：配置 SSL 证书

```bash
certbot --nginx -d stormaidetect.com -d www.stormaidetect.com
```

按提示操作：
1. 输入邮箱
2. 同意服务条款
3. 选择是否重定向到HTTPS（选择2）

### 第十三步：配置防火墙

```bash
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw enable
```

### 第十四步：测试

访问 `https://stormaidetect.com`

---

## 方案二：Vercel + Railway 部署（最简单）

### 前端部署到 Vercel

1. **注册 Vercel**
   - 访问 https://vercel.com
   - 使用 GitHub 账号登录

2. **连接 GitHub**
   - 将代码推送到 GitHub
   - 在 Vercel 点击 "New Project"
   - 选择你的仓库

3. **配置项目**
   - Root Directory: `frontend`
   - Framework Preset: Vite
   - Build Command: `npm run build`
   - Output Directory: `dist`

4. **部署**
   - 点击 "Deploy"
   - 等待部署完成
   - 获得域名：`your-project.vercel.app`

5. **配置自定义域名**
   - Settings → Domains
   - 添加 `stormaidetect.com`
   - 按提示配置DNS

### 后端部署到 Railway

1. **注册 Railway**
   - 访问 https://railway.app
   - 使用 GitHub 账号登录

2. **创建项目**
   - 点击 "New Project"
   - 选择 "Deploy from GitHub repo"
   - 选择你的仓库

3. **配置后端**
   - Root Directory: `backend`
   - Start Command: `gunicorn -w 4 -b 0.0.0.0:$PORT --timeout 120 app:app`

4. **添加 PostgreSQL**
   - 点击 "New" → "Database" → "PostgreSQL"
   - 自动生成连接字符串

5. **配置环境变量**
   在 Variables 中添加：
   ```
   DATABASE_URL=（Railway自动提供）
   SECRET_KEY=your-secret-key
   JWT_SECRET_KEY=your-jwt-secret
   MAIL_SERVER=smtp.gmail.com
   MAIL_PORT=587
   MAIL_USE_TLS=true
   MAIL_USERNAME=your-email@gmail.com
   MAIL_PASSWORD=your-password
   FRONTEND_URL=https://stormaidetect.com
   ```

6. **部署**
   - 自动部署
   - 获得后端URL：`your-project.railway.app`

7. **更新前端配置**
   在 Vercel 环境变量中添加：
   ```
   VITE_API_URL=https://your-project.railway.app
   ```

---

## 方案三：AWS 部署（企业级）

### 使用 AWS Lightsail（简化版AWS）

1. **注册 AWS**
   - 访问 https://aws.amazon.com
   - 注册账户（需要信用卡）
   - 12个月免费套餐

2. **创建 Lightsail 实例**
   - 选择 Linux/Unix
   - 选择 Ubuntu 22.04
   - 选择套餐：$10/月（2GB RAM）
   - 选择区域：Singapore 或 Tokyo

3. **后续步骤**
   - 与 DigitalOcean 部署步骤相同

---

## 性能优化建议

### 1. 使用 CDN 加速

**Cloudflare（免费）：**
1. 注册 https://www.cloudflare.com
2. 添加域名
3. 修改域名DNS服务器
4. 开启CDN和缓存
5. 全球访问速度提升50%+

### 2. 数据库优化

```bash
# 优化 PostgreSQL
nano /etc/postgresql/14/main/postgresql.conf

# 修改以下参数
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
```

### 3. 启用 Gzip 压缩

已在 Nginx 配置中包含

### 4. 图片和静态资源优化

- 使用 CDN 托管静态资源
- 压缩图片
- 启用浏览器缓存

---

## 费用对比

| 方案 | 月费用 | 年费用 | 适合人群 |
|------|--------|--------|----------|
| DigitalOcean | $12 | $144 | 个人/小团队 |
| Vercel + Railway | $5-15 | $60-180 | 快速上线 |
| AWS Lightsail | $10 | $120 | 企业用户 |
| Vultr | $6 | $72 | 预算有限 |

**加上域名费用：** 约 $10-15/年

**总计：** 约 $80-200/年

---

## 推荐配置

### 预算有限（$80/年）
- Vultr $6/月服务器
- Namecheap 域名
- Cloudflare 免费CDN

### 标准配置（$150/年）
- DigitalOcean $12/月服务器
- 域名 + SSL
- Cloudflare CDN

### 最简单（$60-180/年）
- Vercel（前端）免费
- Railway（后端）$5-15/月
- 域名

---

## 监控和维护

### 1. 设置监控

**UptimeRobot（免费）：**
- 访问 https://uptimerobot.com
- 添加网站监控
- 宕机时邮件通知

### 2. 定期备份

```bash
# 数据库备份脚本
nano /root/backup.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
pg_dump -U stormaidetect_user stormaidetect > /root/backups/db_$DATE.sql
find /root/backups -name "db_*.sql" -mtime +7 -delete
```

```bash
chmod +x /root/backup.sh
crontab -e
# 添加：0 3 * * * /root/backup.sh
```

### 3. 更新代码

```bash
cd /var/www/stormaidetect
git pull
cd frontend && npm run build
systemctl restart stormaidetect
```

---

## 总结

**最推荐方案：**
1. **新手/快速上线：** Vercel + Railway
2. **个人项目：** DigitalOcean + Cloudflare
3. **企业项目：** AWS + CloudFront

**全球访问优化：**
- 使用 Cloudflare CDN
- 选择新加坡或美国西海岸数据中心
- 启用 Gzip 和缓存

祝你部署顺利！🚀

