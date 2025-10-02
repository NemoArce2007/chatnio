# Admin Router 功能整理

## 功能概述
`router.go` 文件主要负责注册管理后台的 API 路由，包括统计分析、邀请码管理、兑换码管理、用户管理和管理员密码管理等功能的路由。

---

## 路由描述

### 统计分析
- **`GET /admin/analytics/info`**  
  方法：`InfoAPI`  
  功能：获取系统统计信息。

- **`GET /admin/analytics/model`**  
  方法：`ModelAnalysisAPI`  
  功能：获取模型分析数据。

- **`GET /admin/analytics/request`**  
  方法：`RequestAnalysisAPI`  
  功能：获取请求分析数据。

- **`GET /admin/analytics/billing`**  
  方法：`BillingAnalysisAPI`  
  功能：获取计费分析数据。

- **`GET /admin/analytics/error`**  
  方法：`ErrorAnalysisAPI`  
  功能：获取错误分析数据。

### 邀请码管理
- **`GET /admin/invitation/list`**  
  方法：`InvitationPaginationAPI`  
  功能：分页获取邀请码列表。

- **`POST /admin/invitation/generate`**  
  方法：`GenerateInvitationAPI`  
  功能：生成邀请码。

### 兑换码管理
- **`GET /admin/redeem/list`**  
  方法：`RedeemListAPI`  
  功能：获取兑换码列表。

- **`POST /admin/redeem/generate`**  
  方法：`GenerateRedeemAPI`  
  功能：生成兑换码。

### 用户管理
- **`GET /admin/user/list`**  
  方法：`UserPaginationAPI`  
  功能：分页获取用户列表，支持通过搜索关键词过滤。

- **`POST /admin/user/quota`**  
  方法：`UserQuotaAPI`  
  功能：修改用户配额。

- **`POST /admin/user/subscription`**  
  方法：`UserSubscriptionAPI`  
  功能：修改用户订阅时长。

### 管理员密码管理
- **`POST /admin/user/root`**  
  方法：`UpdateRootPasswordAPI`  
  功能：更新管理员的根密码。

---

## 路由注册逻辑
1. 调用 `channel.Register(app)` 注册通道相关的路由。
2. 使用 `app.GET` 和 `app.POST` 方法分别注册 GET 和 POST 请求的路由。
3. 每个路由对应一个具体的 API 方法，处理相应的请求逻辑。

---
