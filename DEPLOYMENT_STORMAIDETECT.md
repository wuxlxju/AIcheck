# Storm AI Detect 部署指南

## 域名：stormaidetect.com

### 第一步：购买域名

1. 访问域名注册商：
   - **阿里云（万网）**: https://wanwang.aliyun.com
   - **腾讯云**: https://dnspod.cloud.tencent.com
   - **GoDaddy**: https://www.godaddy.com
   - **Namecheap**: https://www.namecheap.com

2. 搜索 `stormaidetect.com`

3. 如果可用，购买域名（约 ¥60-100/年）

### 第二步：购买云服务器

推荐配置：
- **阿里云轻量应用服务器** 或 **腾讯云轻量应用服务器**
- CPU: 2核
- 内存: 4GB
- 带宽: 5Mbps
- 系统: Ubuntu 22.04
- 费用: 约 ¥100-150/月

### 第三步：域名解析

在域名控制台添加 DNS 记录：

| 记录类型 | 主机记录 | 记录值 | TTL |
|---------|---------|--------|-----|
| A | @ | 你的服务器IP | 600 |
| A | www | 你的服务器IP | 600 |

等待 10-30 分钟生效。

### 第四步：服务器部署

#### 1. 连接服务器

```bash
ssh root@your-server-ip
```

#### 2. 安装宝塔面板（推荐新手）

```bash
wget -O install.sh https://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

安装完成后会显示：
- 面板地址: http://your-ip:8888
- 用户名: xxxxxxx
- 密码: xxxxxxx

#### 3. 登录宝塔面板

访问 `http://your-server-ip:8888`，使用上面的账号密码登录

#### 4. 安装软件

在宝塔面板 → 软件商店，安装：
- ✅ Nginx 1.22
- ✅ Python 3.11
- ✅ PostgreSQL 14
- ✅ PM2 管理器

#### 5. 上传代码

**方法 A: 使用宝塔文件管理器**
1. 在宝塔面板 → 文件
2. 进入 `/www/wwwroot/`
3. 上传你的项目压缩包
4. 解压

**方法 B: 使用 Git**
```bash
cd /www/wwwroot/
git clone https://github.com/your-username/aicheck.git stormaidetect
```

#### 6. 配置数据库

在宝塔面板 → 数据库：
1. 添加数据库
   - 数据库名: `stormaidetect`
   - 用户名: `stormaidetect_user`
   - 密码: 自动生成（记下来）

#### 7. 配置后端

```bash
cd /www/wwwroot/stormaidetect/backend

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
DATABASE_URL=postgresql://stormaidetect_user:your_db_password@localhost/stormaidetect
SECRET_KEY=storm-ai-detect-secret-key-2024-change-this-in-production
JWT_SECRET_KEY=storm-ai-detect-jwt-secret-2024-change-this

MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=true
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-gmail-app-password
MAIL_DEFAULT_SENDER=noreply@stormaidetect.com

FRONTEND_URL=https://stormaidetect.com

UPLOAD_FOLDER=/www/wwwroot/stormaidetect/uploads
PDF_FOLDER=/www/wwwroot/stormaidetect/pdf_reports
```

创建目录：

```bash
mkdir -p /www/wwwroot/stormaidetect/uploads
mkdir -p /www/wwwroot/stormaidetect/pdf_reports
chmod 755 /www/wwwroot/stormaidetect/uploads
chmod 755 /www/wwwroot/stormaidetect/pdf_reports
```

#### 8. 使用 PM2 启动后端

在宝塔面板 → PM2 管理器：
1. 添加项目
2. 项目名称: `stormaidetect-backend`
3. 启动文件: `/www/wwwroot/stormaidetect/backend/venv/bin/gunicorn`
4. 运行目录: `/www/wwwroot/stormaidetect/backend`
5. 启动命令: `-w 4 -b 127.0.0.1:5000 --timeout 120 app:app`
6. 点击启动

#### 9. 构建前端

```bash
cd /www/wwwroot/stormaidetect/frontend

# 修改 API 地址（如果需要）
nano vite.config.js

# 安装依赖
npm install

# 构建
npm run build
```

#### 10. 配置 Nginx

在宝塔面板 → 网站：

1. 添加站点
   - 域名: `stormaidetect.com` 和 `www.stormaidetect.com`
   - 根目录: `/www/wwwroot/stormaidetect/frontend/dist`
   - PHP版本: 纯静态

2. 配置反向代理
   点击站点 → 设置 → 反向代理 → 添加反向代理：
   - 代理名称: `backend-api`
   - 目标URL: `http://127.0.0.1:5000`
   - 发送域名: `$host`
   - 内容替换: 留空
   - 配置文件添加：
   
```nginx
location /api {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

3. 申请 SSL 证书
   - 点击站点 → 设置 → SSL
   - 选择 Let's Encrypt
   - 勾选域名
   - 点击申请
   - 开启强制HTTPS

#### 11. 配置防火墙

在宝塔面板 → 安全：
- 放行端口: 80, 443, 8888

### 第五步：测试网站

1. 访问 `https://stormaidetect.com`
2. 注册账户测试
3. 充值测试
4. 上传文件测试

### 第六步：配置邮件服务（用于忘记密码）

#### 使用 Gmail

1. 登录 Gmail
2. 开启两步验证
3. 生成应用专用密码：
   - 访问 https://myaccount.google.com/apppasswords
   - 选择"邮件"和"其他设备"
   - 生成密码
4. 将密码填入 `.env` 文件的 `MAIL_PASSWORD`

#### 或使用阿里云邮件推送

1. 开通阿里云邮件推送服务
2. 配置发信域名
3. 获取 SMTP 配置
4. 更新 `.env` 文件

### 第七步：配置支付接口

参考 `PAYMENT_INTEGRATION.md` 文档配置真实的微信/支付宝支付。

### 维护和监控

#### 查看日志

```bash
# 后端日志
pm2 logs stormaidetect-backend

# Nginx 日志
tail -f /www/wwwlogs/stormaidetect.com.log
```

#### 重启服务

```bash
# 重启后端
pm2 restart stormaidetect-backend

# 重启 Nginx
nginx -s reload
```

#### 更新代码

```bash
cd /www/wwwroot/stormaidetect
git pull

# 重新构建前端
cd frontend
npm run build

# 重启后端
pm2 restart stormaidetect-backend
```

### 备份策略

在宝塔面板 → 计划任务：
1. 添加数据库备份任务（每天凌晨3点）
2. 添加网站备份任务（每周一次）

### 预计费用

| 项目 | 费用 |
|------|------|
| 域名 | ¥60-100/年 |
| 服务器 | ¥100-150/月 |
| SSL证书 | 免费（Let's Encrypt）|
| **总计** | **约 ¥1300-1900/年** |

### 常见问题

**Q: 域名已被注册怎么办？**
A: 可以尝试：
- stormaidetect.cn
- stormaidetect.net
- storm-aidetect.com
- stormaicheck.com

**Q: 网站打不开？**
A: 检查：
1. 域名解析是否生效（ping stormaidetect.com）
2. 服务器防火墙是否开放 80/443 端口
3. Nginx 是否正常运行
4. 后端服务是否启动

**Q: 如何查看网站访问量？**
A: 在宝塔面板 → 网站 → 统计

**Q: 如何升级服务器配置？**
A: 在云服务商控制台直接升级配置

### 技术支持

如遇到问题，可以：
1. 查看宝塔面板日志
2. 查看 PM2 日志
3. 查看 Nginx 错误日志
4. 检查数据库连接

---

## 快速部署检查清单

- [ ] 购买域名 stormaidetect.com
- [ ] 购买云服务器
- [ ] 配置域名解析
- [ ] 安装宝塔面板
- [ ] 安装必要软件（Nginx, Python, PostgreSQL）
- [ ] 上传代码
- [ ] 配置数据库
- [ ] 配置后端环境变量
- [ ] 启动后端服务
- [ ] 构建前端
- [ ] 配置 Nginx
- [ ] 申请 SSL 证书
- [ ] 测试网站功能
- [ ] 配置邮件服务
- [ ] 配置支付接口
- [ ] 设置备份任务

完成以上步骤后，你的网站就可以正式上线了！🎉

