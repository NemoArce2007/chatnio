# Auth模块API文档

## 模块概览

Auth模块是ChatNio系统的认证与授权核心模块，负责用户注册、登录、权限验证、配额管理、订阅管理、邀请码和兑换码等功能。

### 文件结构
```
auth/
├── analysis.go      # 统计分析功能
├── apikey.go        # API密钥管理
├── auth.go          # 核心认证逻辑
├── call.go          # 外部调用接口
├── cert.go          # 证书相关功能
├── controller.go    # API控制器
├── invitation.go    # 邀请码管理
├── package.go       # 套餐管理
├── payment.go       # 支付功能
├── plan.go          # 订阅计划
├── quota.go         # 配额管理
├── redeem.go        # 兑换码管理
├── router.go        # 路由注册
├── rule.go          # 规则验证
├── struct.go        # 数据结构定义
├── subscription.go  # 订阅管理
└── validators.go    # 验证器
```

## 路由概览
> **源代码位置**: auth/router.go 第5-18行 - Register函数

```go
func Register(app *gin.RouterGroup) {
    app.POST("/verify", VerifyAPI)
    app.POST("/reset", ResetAPI)
    app.POST("/register", RegisterAPI)
    app.POST("/login", LoginAPI)
    app.POST("/state", StateAPI)
    app.GET("/apikey", KeyAPI)
    app.GET("/package", PackageAPI)
    app.GET("/quota", QuotaAPI)
    app.POST("/buy", BuyAPI)
    app.GET("/subscription", SubscriptionAPI)
    app.POST("/subscribe", SubscribeAPI)
    app.GET("/invite", InviteAPI)
    app.GET("/redeem", RedeemAPI)
}
```

## 核心数据结构

### User结构体
> **源代码位置**: auth/struct.go 第9-19行 - User

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| ID | int64 | id | 用户ID |
| Username | string | username | 用户名 |
| Email | string | email | 邮箱地址 |
| BindID | int64 | bind_id | 绑定ID |
| Password | string | password | 密码哈希 |
| Token | string | token | 用户令牌 |
| Admin | bool | is_admin | 是否管理员 |
| Level | int | level | 订阅等级 |
| Subscription | *time.Time | subscription | 订阅到期时间 |

## API端点详情

### 1. 邮箱验证
**端点**: `POST /verify`
**功能**: 发送邮箱验证码
> **源代码位置**: auth/controller.go 第211-232行 - VerifyAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第17-19行 - VerifyForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| email | string | 是 | 邮箱地址 |

#### 响应格式
```json
{
  "status": true
}
```

### 2. 用户注册
**端点**: `POST /register`
**功能**: 用户注册
> **源代码位置**: auth/controller.go 第136-167行 - RegisterAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第10-15行 - RegisterForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |
| email | string | 是 | 邮箱地址 |
| code | string | 是 | 验证码 |

#### 响应格式
```json
{
  "status": true,
  "token": "jwt_token_string"
}
```

### 3. 用户登录
**端点**: `POST /login`
**功能**: 用户登录
> **源代码位置**: auth/controller.go 第169-209行 - LoginAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第21-24行 - LoginForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |

#### 响应格式
```json
{
  "status": true,
  "token": "jwt_token_string"
}
```

### 4. 密码重置
**端点**: `POST /reset`
**功能**: 重置密码
> **源代码位置**: auth/controller.go 第234-255行 - ResetAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第30-34行 - ResetForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| email | string | 是 | 邮箱地址 |
| code | string | 是 | 验证码 |
| password | string | 是 | 新密码 |

#### 响应格式
```json
{
  "status": true
}
```

### 5. 用户状态
**端点**: `POST /state`
**功能**: 获取用户登录状态
> **源代码位置**: auth/controller.go 第257-264行 - StateAPI

#### 响应格式
```json
{
  "status": true,
  "user": "username",
  "admin": false
}
```

### 6. API密钥
**端点**: `GET /apikey`
**功能**: 获取用户API密钥
> **源代码位置**: auth/controller.go 第266-280行 - KeyAPI

#### 响应格式
```json
{
  "status": true,
  "key": "api_key_string"
}
```

### 7. 套餐信息
**端点**: `GET /package`
**功能**: 获取用户套餐信息
> **源代码位置**: auth/controller.go 第282-293行 - PackageAPI

#### 响应格式
```json
{
  "status": true,
  "data": {
    // 套餐详细信息
  }
}
```

### 8. 配额查询
**端点**: `GET /quota`
**功能**: 获取用户配额
> **源代码位置**: auth/controller.go 第295-306行 - QuotaAPI

#### 响应格式
```json
{
  "status": true,
  "quota": 100.5
}
```

### 9. 购买配额
**端点**: `POST /buy`
**功能**: 购买配额
> **源代码位置**: auth/controller.go 第364-400行 - BuyAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第36-38行 - BuyForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| quota | int | 是 | 购买配额数量 (1-99999) |

#### 响应格式
```json
{
  "status": true,
  "error": "success"
}
```

### 10. 订阅信息
**端点**: `GET /subscription`
**功能**: 获取订阅信息
> **源代码位置**: auth/controller.go 第308-324行 - SubscriptionAPI

#### 响应格式
```json
{
  "status": true,
  "level": 1,
  "is_subscribed": true,
  "enterprise": false,
  "expired": 30,
  "usage": {
    // 使用情况统计
  }
}
```

### 11. 购买订阅
**端点**: `POST /subscribe`
**功能**: 购买订阅
> **源代码位置**: auth/controller.go 第326-362行 - SubscribeAPI

#### 请求参数
> **源代码位置**: auth/controller.go 第40-43行 - SubscribeForm

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| level | int | 是 | 订阅等级 |
| month | int | 是 | 订阅月数 (1-999) |

#### 响应格式
```json
{
  "status": true,
  "error": "success"
}
```

### 12. 使用邀请码
**端点**: `GET /invite`
**功能**: 使用邀请码
> **源代码位置**: auth/controller.go 第402-433行 - InviteAPI

#### 请求参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| code | string | 是 | 邀请码 (URL参数) |

#### 响应格式
```json
{
  "status": true,
  "error": "success",
  "quota": 10.0
}
```

### 13. 使用兑换码
**端点**: `GET /redeem`
**功能**: 使用兑换码
> **源代码位置**: auth/controller.go 第435-467行 - RedeemAPI

#### 请求参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| code | string | 是 | 兑换码 (URL参数) |

#### 响应格式
```json
{
  "status": true,
  "error": "success",
  "quota": 20.0
}
```

## 核心功能实现

### 认证相关
> **源代码位置**: auth/auth.go

- **ParseToken**: 第19-36行 - JWT令牌解析
- **ParseApiKey**: 第38-52行 - API密钥解析
- **SignUp**: 第82-122行 - 用户注册逻辑
- **Login**: 第124-145行 - 用户登录逻辑
- **DeepLogin**: 第147-172行 - 深度训练模式登录
- **Reset**: 第213-249行 - 密码重置

### 配额管理
> **源代码位置**: auth/quota.go

- **GetQuota**: 第5-10行 - 获取用户配额
- **IncreaseQuota**: 第33-37行 - 增加配额
- **UseQuota**: 第55-62行 - 使用配额
- **PayedQuota**: 第64-76行 - 付费配额扣除

### 订阅管理
> **源代码位置**: auth/subscription.go

- **GetSubscription**: 第11-22行 - 获取订阅信息
- **IsSubscribe**: 第43-47行 - 检查订阅状态
- **AddSubscription**: 第65-74行 - 添加订阅
- **BuySubscription**: 第115-158行 - 购买订阅

### 邀请码管理
> **源代码位置**: auth/invitation.go

- **Invitation结构体**: 第10-16行 - 邀请码数据结构
- **GenerateInvitations**: 第18-35行 - 生成邀请码
- **UseInvitation**: 第105-113行 - 使用邀请码

### 支付功能
> **源代码位置**: auth/payment.go

- **BalanceResponse**: 第12-15行 - 余额响应结构
- **PaymentResponse**: 第17-20行 - 支付响应结构
- **GetBalance**: 第26-44行 - 获取余额
- **Pay**: 第46-64行 - 执行支付
- **BuyQuota**: 第78-89行 - 购买配额

## 权限验证

### 认证中间件
> **源代码位置**: auth/controller.go

- **RequireAuth**: 第69-77行 - 需要登录
- **RequireAdmin**: 第79-96行 - 需要管理员权限
- **RequireSubscription**: 第98-115行 - 需要订阅
- **RequireEnterprise**: 第117-134行 - 需要企业版

## 错误处理

所有API都遵循统一的错误响应格式：
```json
{
  "status": false,
  "error": "错误描述信息"
}
```

常见错误类型：
- `bad request`: 请求参数格式错误
- `user not found`: 用户未找到
- `admin required`: 需要管理员权限
- `subscription required`: 需要订阅
- `invalid username or password`: 用户名或密码错误
- `not enough money`: 余额不足

## 数据库依赖

### 主要数据表
- `auth`: 用户基本信息
- `quota`: 用户配额信息
- `subscription`: 订阅信息
- `invitation`: 邀请码
- `redeem`: 兑换码
- `apikey`: API密钥

### 缓存依赖
- Redis用于存储验证码、统计数据等临时信息
- 缓存键格式：`nio:otp:{email}` (验证码)