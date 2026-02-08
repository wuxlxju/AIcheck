# 支付接口集成指南

## 概述

当前系统已实现支付框架，但需要集成真实的微信和支付宝支付接口。

## 微信支付集成

### 1. 申请微信支付商户号

1. 访问 [微信支付商户平台](https://pay.weixin.qq.com/)
2. 注册并完成企业认证
3. 获取以下信息：
   - AppID
   - MCH_ID (商户号)
   - API_KEY (API密钥)

### 2. 安装微信支付SDK

```bash
pip install wechatpay-python
```

### 3. 修改支付路由

在 `backend/routes/payment.py` 中，修改 `create_order` 函数：

```python
from wechatpay_python import WeChatPay

@payment_bp.route('/create-order', methods=['POST'])
@jwt_required()
def create_order():
    # ... 现有代码 ...
    
    # 初始化微信支付
    wechatpay = WeChatPay(
        appid=current_app.config['WECHAT_APP_ID'],
        mch_id=current_app.config['WECHAT_MCH_ID'],
        api_key=current_app.config['WECHAT_API_KEY']
    )
    
    # 创建订单
    order_data = {
        'out_trade_no': transaction_id,
        'body': f'AI检测服务 - {points}点数',
        'total_fee': int(amount * 100),  # 转换为分
        'spbill_create_ip': request.remote_addr,
        'notify_url': f"{current_app.config['FRONTEND_URL']}/api/payment/callback",
        'trade_type': 'NATIVE'  # 或 'JSAPI' 用于H5支付
    }
    
    result = wechatpay.unified_order(**order_data)
    
    if result.get('return_code') == 'SUCCESS':
        payment_data['payment_url'] = result.get('code_url')  # 二维码URL
        return jsonify({
            'message': '订单创建成功',
            'payment': payment_data
        }), 200
    else:
        return jsonify({'error': '创建订单失败'}), 500
```

### 4. 配置回调验证

在 `payment_callback` 函数中添加签名验证：

```python
@payment_bp.route('/callback', methods=['POST'])
def payment_callback():
    # 验证签名
    if not wechatpay.verify_notify(request.data, request.headers):
        return jsonify({'error': '签名验证失败'}), 400
    
    # 处理支付结果
    # ... 现有代码 ...
```

## 支付宝集成

### 1. 申请支付宝开放平台账号

1. 访问 [支付宝开放平台](https://open.alipay.com/)
2. 创建应用并获取：
   - APP_ID
   - 应用私钥
   - 支付宝公钥

### 2. 安装支付宝SDK

```bash
pip install alipay-sdk-python
```

### 3. 修改支付路由

```python
from alipay import AliPay

@payment_bp.route('/create-order', methods=['POST'])
@jwt_required()
def create_order():
    # ... 现有代码 ...
    
    # 初始化支付宝
    alipay = AliPay(
        appid=current_app.config['ALIPAY_APP_ID'],
        app_notify_url=f"{current_app.config['FRONTEND_URL']}/api/payment/callback",
        app_private_key_string=current_app.config['ALIPAY_PRIVATE_KEY'],
        alipay_public_key_string=current_app.config['ALIPAY_PUBLIC_KEY'],
        sign_type="RSA2",
        debug=False
    )
    
    # 创建订单
    order_string = alipay.api_alipay_trade_page_pay(
        out_trade_no=transaction_id,
        total_amount=str(amount),
        subject=f'AI检测服务 - {points}点数',
        return_url=f"{current_app.config['FRONTEND_URL']}/payment/success",
        notify_url=f"{current_app.config['FRONTEND_URL']}/api/payment/callback"
    )
    
    payment_url = f"https://openapi.alipay.com/gateway.do?{order_string}"
    
    payment_data['payment_url'] = payment_url
    return jsonify({
        'message': '订单创建成功',
        'payment': payment_data
    }), 200
```

## 前端支付页面

需要在前端添加支付页面来显示支付二维码或跳转到支付页面。

### 创建支付页面

在 `frontend/src/pages/Payment.jsx`:

```jsx
import React, { useState, useEffect } from 'react'
import { useParams, useNavigate } from 'react-router-dom'
import axios from 'axios'

const Payment = () => {
  const { orderId } = useParams()
  const [paymentInfo, setPaymentInfo] = useState(null)
  const [status, setStatus] = useState('pending')
  
  useEffect(() => {
    checkPaymentStatus()
    const interval = setInterval(checkPaymentStatus, 3000)
    return () => clearInterval(interval)
  }, [orderId])
  
  const checkPaymentStatus = async () => {
    try {
      const response = await axios.get(`/api/payment/status/${orderId}`)
      if (response.data.status === 'completed') {
        setStatus('completed')
        // 跳转到成功页面
      }
    } catch (error) {
      console.error('检查支付状态失败:', error)
    }
  }
  
  return (
    <div className="payment-page">
      {paymentInfo && (
        <div>
          <h2>支付金额: ¥{paymentInfo.amount}</h2>
          {paymentInfo.payment_method === 'wechat' && (
            <img src={paymentInfo.payment_url} alt="微信支付二维码" />
          )}
          {paymentInfo.payment_method === 'alipay' && (
            <a href={paymentInfo.payment_url} className="btn btn-primary">
              跳转到支付宝支付
            </a>
          )}
        </div>
      )}
    </div>
  )
}

export default Payment
```

## 安全注意事项

1. **签名验证**：所有支付回调必须验证签名
2. **金额校验**：回调中验证金额是否匹配
3. **幂等性**：确保重复回调不会重复增加点数
4. **HTTPS**：生产环境必须使用HTTPS
5. **IP白名单**：配置支付平台的IP白名单

## 测试

### 微信支付测试

使用微信支付沙箱环境进行测试：
- 沙箱环境URL: `https://api.mch.weixin.qq.com/sandboxnew/`

### 支付宝测试

使用支付宝沙箱环境：
- 沙箱环境URL: `https://openapi.alipaydev.com/gateway.do`

## 费率说明

- 微信支付：一般0.6%
- 支付宝：一般0.6%

具体费率以实际签约为准。

