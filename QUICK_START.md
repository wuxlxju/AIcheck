# 快速开始指南

## 5分钟快速启动

### 1. 后端启动

```bash
# 进入后端目录
cd backend

# 创建虚拟环境（Windows用户使用 python -m venv venv）
python3 -m venv venv

# 激活虚拟环境
# Linux/Mac:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt

# 运行后端（开发模式）
python app.py
```

后端将在 `http://localhost:5000` 启动

### 2. 前端启动

打开新的终端窗口：

```bash
# 进入前端目录
cd frontend

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

前端将在 `http://localhost:3000` 启动

### 3. 访问网站

打开浏览器访问：`http://localhost:3000`

### 4. 默认管理员账户

- 邮箱: `admin@aicheck.com`
- 密码: `admin123456`

首次启动时会自动创建此管理员账户。

## 常见问题

### 后端启动失败

1. **端口被占用**：修改 `app.py` 中的端口号
2. **依赖安装失败**：确保Python版本 >= 3.11
3. **数据库错误**：删除 `aicheck.db` 文件重新启动

### 前端启动失败

1. **Node版本**：确保Node.js版本 >= 18
2. **端口被占用**：修改 `vite.config.js` 中的端口
3. **依赖安装慢**：使用国内镜像 `npm config set registry https://registry.npmmirror.com`

### 邮件功能无法使用

开发环境可以暂时跳过邮件配置，忘记密码功能需要配置真实的邮件服务器。

### 支付功能

支付功能需要配置真实的微信/支付宝接口，详见 `PAYMENT_INTEGRATION.md`

## 下一步

1. 配置邮件服务（用于忘记密码功能）
2. 配置支付接口（用于点数充值）
3. 集成真实的AI检测API
4. 部署到生产环境（参考 `DEPLOYMENT.md`）

