# Storm AI Detect 快速部署清单

## 域名：stormaidetect.com

### 📋 部署前准备

#### 1. 购买域名（5分钟）
- [ ] 访问阿里云或腾讯云
- [ ] 搜索并购买 `stormaidetect.com`
- [ ] 费用：约 ¥60-100/年

#### 2. 购买服务器（10分钟）
- [ ] 选择云服务商（推荐阿里云或腾讯云）
- [ ] 配置：2核4G，5M带宽，Ubuntu 22.04
- [ ] 费用：约 ¥100-150/月
- [ ] 记录服务器IP地址：`_______________`

---

### 🚀 部署步骤（使用宝塔面板 - 最简单）

#### 第一步：域名解析（5分钟）

在域名控制台添加记录：

| 记录类型 | 主机记录 | 记录值 |
|---------|---------|--------|
| A | @ | 你的服务器IP |
| A | www | 你的服务器IP |

- [ ] 添加完成，等待生效（10-30分钟）

#### 第二步：安装宝塔面板（10分钟）

```bash
# 1. SSH连接服务器
ssh root@你的服务器IP

# 2. 安装宝塔
wget -O install.sh https://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

- [ ] 记录宝塔登录信息：
  - 面板地址：`http://你的IP:8888`
  - 用户名：`_______________`
  - 密码：`_______________`

#### 第三步：安装软件（15分钟）

登录宝塔面板，在"软件商店"安装：

- [ ] Nginx 1.22
- [ ] Python 3.11
- [ ] PostgreSQL 14
- [ ] PM2 管理器

#### 第四步：创建数据库（3分钟）

在宝塔 → 数据库：

- [ ] 数据库名：`stormaidetect`
- [ ] 用户名：`stormaidetect_user`
- [ ] 密码：`_______________`（记下来）

#### 第五步：上传代码（5分钟）

**方法1：使用宝塔文件管理器**
- [ ] 压缩本地项目文件夹
- [ ] 在宝塔 → 文件 → `/www/wwwroot/`
- [ ] 上传并解压

**方法2：使用Git**
```bash
cd /www/wwwroot/
git clone https://github.com/你的用户名/项目名.git stormaidetect
```

#### 第六步：配置后端（10分钟）

```bash
cd /www/wwwroot/stormaidetect/backend

# 创建虚拟环境
python3.11 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
pip install gunicorn

# 创建配置文件
nano .env
```

在 `.env` 文件中填入：

```bash
DATABASE_URL=postgresql://stormaidetect_user:你的数据库密码@localhost/stormaidetect
SECRET_KEY=storm-ai-detect-secret-2024-change-this
JWT_SECRET_KEY=storm-jwt-secret-2024-change-this
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=true
MAIL_USERNAME=你的邮箱@gmail.com
MAIL_PASSWORD=你的Gmail应用密码
MAIL_DEFAULT_SENDER=noreply@stormaidetect.com
FRONTEND_URL=https://stormaidetect.com
UPLOAD_FOLDER=/www/wwwroot/stormaidetect/uploads
PDF_FOLDER=/www/wwwroot/stormaidetect/pdf_reports
```

创建目录：
```bash
mkdir -p /www/wwwroot/stormaidetect/uploads
mkdir -p /www/wwwroot/stormaidetect/pdf_reports
```

- [ ] 配置完成

#### 第七步：启动后端（5分钟）

在宝塔 → PM2管理器：

- [ ] 添加项目
  - 项目名称：`stormaidetect-backend`
  - 启动文件：`/www/wwwroot/stormaidetect/backend/venv/bin/gunicorn`
  - 运行目录：`/www/wwwroot/stormaidetect/backend`
  - 启动命令：`-w 4 -b 127.0.0.1:5000 --timeout 120 app:app`
- [ ] 点击启动

#### 第八步：构建前端（5分钟）

```bash
cd /www/wwwroot/stormaidetect/frontend
npm install
npm run build
```

- [ ] 构建完成

#### 第九步：配置Nginx（10分钟）

在宝塔 → 网站：

1. **添加站点**
   - [ ] 域名：`stormaidetect.com,www.stormaidetect.com`
   - [ ] 根目录：`/www/wwwroot/stormaidetect/frontend/dist`
   - [ ] PHP版本：纯静态

2. **配置反向代理**
   - [ ] 点击站点 → 设置 → 反向代理
   - [ ] 添加反向代理：
     - 代理名称：`backend-api`
     - 目标URL：`http://127.0.0.1:5000`
   - [ ] 在配置文件中添加：

```nginx
location /api {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

3. **申请SSL证书**
   - [ ] 点击站点 → 设置 → SSL
   - [ ] 选择 Let's Encrypt
   - [ ] 勾选两个域名
   - [ ] 点击申请
   - [ ] 开启"强制HTTPS"

#### 第十步：测试网站（5分钟）

- [ ] 访问 `https://stormaidetect.com`
- [ ] 注册账户
- [ ] 测试充值（使用模拟支付）
- [ ] 测试上传文件
- [ ] 测试下载报告

---

### ✅ 部署完成检查

- [ ] 网站可以正常访问
- [ ] HTTPS 正常工作（绿色锁）
- [ ] 用户注册登录正常
- [ ] 文件上传正常
- [ ] PDF下载正常
- [ ] 后台管理正常

---

### 📊 费用总结

| 项目 | 费用 | 备注 |
|------|------|------|
| 域名 | ¥60-100/年 | 首年可能有优惠 |
| 服务器 | ¥100-150/月 | 可按需升级 |
| SSL证书 | 免费 | Let's Encrypt |
| **首年总计** | **¥1300-1900** | 约 ¥108-158/月 |

---

### 🔧 后续配置（可选）

#### 配置邮件服务
- [ ] 开启Gmail两步验证
- [ ] 生成应用专用密码
- [ ] 更新 `.env` 中的邮件配置

#### 配置真实支付
- [ ] 申请微信支付商户号
- [ ] 申请支付宝企业账号
- [ ] 参考 `PAYMENT_INTEGRATION.md` 配置

#### 设置备份
- [ ] 在宝塔 → 计划任务
- [ ] 添加数据库备份（每天）
- [ ] 添加网站备份（每周）

---

### 📞 需要帮助？

查看详细文档：
- `DEPLOYMENT_STORMAIDETECT.md` - 完整部署指南
- `TESTING_GUIDE.md` - 测试指南
- `PAYMENT_INTEGRATION.md` - 支付配置

---

### 🎉 恭喜！

完成以上步骤后，你的网站 **stormaidetect.com** 就正式上线了！

**默认管理员账户：**
- 邮箱：`admin@aicheck.com`
- 密码：`admin123456`
- ⚠️ 请立即修改密码！

**下一步：**
1. 修改管理员密码
2. 配置邮件服务
3. 配置真实支付接口
4. 推广你的网站

