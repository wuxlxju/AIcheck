# Storm AI Detect - AI内容检测平台

官网：https://stormaidetect.com

一个面向留学生的AI内容检测平台，提供专业的PDF检测报告。

## 功能特点

- ✅ 用户注册/登录（邮箱验证、忘记密码）
- ✅ 文件上传（支持.docx和.txt，拖拽上传）
- ✅ AI检测处理
- ✅ PDF报告生成（矢量生成，高分辨率）
- ✅ 点数系统（每次检测消耗点数）
- ✅ 在线充值（微信/支付宝）
- ✅ 后台管理系统（仪表盘、用户管理）
- ✅ 文件自动销毁（24小时）
- ✅ 并发处理支持
- ✅ 响应式设计

## 技术栈

### 后端
- Flask (Python)
- SQLAlchemy (数据库ORM)
- Flask-JWT-Extended (身份认证)
- Flask-Mail (邮件服务)
- ReportLab (PDF生成)
- APScheduler (定时任务)

### 前端
- React 18
- React Router
- Axios
- React Dropzone

## 本地开发

> 💡 **提示**：详细快速开始指南请查看 [QUICK_START.md](QUICK_START.md)

### 后端设置

1. 进入后端目录：
```bash
cd backend
```

2. 创建虚拟环境（推荐）：
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. 安装依赖：
```bash
pip install -r requirements.txt
```

4. 配置环境变量：
```bash
cp .env.example .env
# 编辑 .env 文件，填入你的配置
```

5. 运行后端：
```bash
python app.py
```

后端将在 `http://localhost:5000` 运行

### 前端设置

1. 进入前端目录：
```bash
cd frontend
```

2. 安装依赖：
```bash
npm install
```

3. 运行开发服务器：
```bash
npm run dev
```

前端将在 `http://localhost:3000` 运行

## 默认管理员账户

- 邮箱: `admin@aicheck.com`
- 密码: `admin123456`

**⚠️ 生产环境请立即修改默认密码！**

## 生产部署

### 使用域名：stormaidetect.com

详细部署指南请查看：
- [DEPLOYMENT_STORMAIDETECT.md](DEPLOYMENT_STORMAIDETECT.md) - 使用宝塔面板部署（推荐）
- [DEPLOYMENT.md](DEPLOYMENT.md) - 手动部署指南

### 快速部署步骤

1. 购买域名 `stormaidetect.com`
2. 购买云服务器（阿里云/腾讯云）
3. 安装宝塔面板
4. 配置域名解析
5. 上传代码并配置
6. 申请SSL证书
7. 启动服务

## 项目结构

```
stormaidetect/
├── backend/              # Flask后端
│   ├── routes/          # API路由
│   ├── utils/           # 工具函数
│   ├── models.py        # 数据模型
│   ├── config.py        # 配置
│   └── app.py           # 应用入口
├── frontend/            # React前端
│   ├── src/
│   │   ├── pages/      # 页面组件
│   │   ├── components/ # 通用组件
│   │   └── contexts/   # React Context
│   └── package.json
├── README.md            # 项目说明
├── QUICK_START.md       # 快速开始
├── DEPLOYMENT_STORMAIDETECT.md  # 部署指南（宝塔）
└── PAYMENT_INTEGRATION.md       # 支付集成指南
```

## 文档

- `README.md` - 项目总览（本文件）
- `QUICK_START.md` - 快速开始指南
- `DEPLOYMENT_STORMAIDETECT.md` - 生产环境部署指南（宝塔面板）
- `DEPLOYMENT.md` - 生产环境部署指南（手动）
- `PAYMENT_INTEGRATION.md` - 支付接口集成指南
- `TESTING_GUIDE.md` - 测试指南

## 安全特性

- 密码加密存储（bcrypt）
- JWT 令牌认证
- SQL 注入防护（ORM）
- 文件类型和大小验证
- CORS 配置
- HTTPS 支持

## 待完善功能

- [ ] 真实的AI检测API集成
- [ ] 微信/支付宝支付接口完整实现
- [ ] 邮件模板优化
- [ ] 更多统计图表
- [ ] 文件预览功能
- [ ] 批量上传支持

## 费用预估

### 开发环境
- 免费（本地运行）

### 生产环境
- 域名：¥60-100/年
- 服务器：¥100-150/月
- SSL证书：免费（Let's Encrypt）
- **总计：约 ¥1300-1900/年**

## 技术支持

如有问题，请查看：
1. [TESTING_GUIDE.md](TESTING_GUIDE.md) - 测试和常见问题
2. [DEPLOYMENT_STORMAIDETECT.md](DEPLOYMENT_STORMAIDETECT.md) - 部署问题
3. 项目 Issues

## 许可证

MIT License

---

**Storm AI Detect** - 专业的AI内容检测平台
