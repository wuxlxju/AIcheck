# 测试支付功能（开发环境）

由于支付接口需要企业资质和真实配置，开发环境提供了模拟支付功能。

## 模拟支付流程

1. 登录系统
2. 点击导航栏的"充值"按钮
3. 选择充值套餐或输入自定义金额
4. 选择支付方式（微信或支付宝）
5. 点击"立即充值"
6. 在支付页面，打开浏览器控制台（F12）
7. 执行以下命令模拟支付成功：

```javascript
// 获取当前订单ID（从URL中）
const orderId = window.location.pathname.split('/').pop();

// 调用模拟支付接口
fetch(`/api/payment/mock-complete/${orderId}`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`,
    'Content-Type': 'application/json'
  }
})
.then(res => res.json())
.then(data => {
  console.log('支付结果:', data);
  // 刷新页面查看点数变化
  window.location.reload();
});
```

## 或者使用简化命令

在支付页面的控制台直接执行：

```javascript
fetch(`/api/payment/mock-complete/${window.location.pathname.split('/').pop()}`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`,
    'Content-Type': 'application/json'
  }
}).then(res => res.json()).then(data => {
  alert('支付成功！点数已到账');
  window.location.href = '/dashboard';
});
```

## 生产环境

生产环境需要：
1. 申请微信支付商户号
2. 申请支付宝企业账号
3. 配置真实的支付接口
4. 删除或禁用 `mock-complete` 接口

详见 `PAYMENT_INTEGRATION.md`

