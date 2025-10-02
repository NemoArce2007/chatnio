
# Admin Controller 功能整理

## 功能概述
`controller.go` 文件实现了多个与管理相关的 API 控制器，主要用于处理管理员操作的 HTTP 请求。以下是主要功能：
1. 系统信息统计。
2. 模型、请求、计费和错误分析。
3. 邀请码和兑换码管理。
4. 用户管理（分页、配额、订阅时长）。
5. 管理员密码管理。

---

## 路由和方法

### 系统信息统计
- **`GET /info`**  
  方法：`InfoAPI`  
  功能：获取系统统计信息，包括订阅用户数量、今日收入、本月收入。

### 数据分析
- **`GET /model-analysis`**  
  方法：`ModelAnalysisAPI`  
  功能：获取模型分析数据。

- **`GET /request-analysis`**  
  方法：`RequestAnalysisAPI`  
  功能：获取请求分析数据。

- **`GET /billing-analysis`**  
  方法：`BillingAnalysisAPI`  
  功能：获取计费分析数据。

- **`GET /error-analysis`**  
  方法：`ErrorAnalysisAPI`  
  功能：获取错误分析数据。

### 邀请码和兑换码管理
- **`GET /invitation-pagination`**  
  方法：`InvitationPaginationAPI`  
  功能：分页获取邀请码列表。

- **`POST /generate-invitation`**  
  方法：`GenerateInvitationAPI`  
  功能：生成邀请码。

- **`GET /redeem-list`**  
  方法：`RedeemListAPI`  
  功能：获取兑换码列表。

- **`POST /generate-redeem`**  
  方法：`GenerateRedeemAPI`  
  功能：生成兑换码。

### 用户管理
- **`GET /user-pagination`**  
  方法：`UserPaginationAPI`  
  功能：分页获取用户列表，支持通过搜索关键词过滤。

- **`POST /user-quota`**  
  方法：`UserQuotaAPI`  
  功能：修改用户配额。

- **`POST /user-subscription`**  
  方法：`UserSubscriptionAPI`  
  功能：修改用户订阅时长。

### 管理员密码管理
- **`POST /update-root-password`**  
  方法：`UpdateRootPasswordAPI`  
  功能：更新管理员根密码。

---

## 数据结构

### 表单结构
- **`GenerateInvitationForm`**  
  用于生成邀请码的表单数据：
  - `Type`：邀请码类型。
  - `Quota`：配额。
  - `Number`：生成数量。

- **`GenerateRedeemForm`**  
  用于生成兑换码的表单数据：
  - `Quota`：配额。
  - `Number`：生成数量。

- **`QuotaOperationForm`**  
  用于修改用户配额的表单数据：
  - `Id`：用户 ID。
  - `Quota`：新的配额值。

- **`SubscriptionOperationForm`**  
  用于修改用户订阅时长的表单数据：
  - `Id`：用户 ID。
  - `Month`：订阅时长（月）。

- **`UpdateRootPasswordForm`**  
  用于更新管理员根密码的表单数据：
  - `Password`：新密码。

---

## 方法功能描述

### 系统信息统计
- **`InfoAPI`**  
  返回系统的统计信息，包括订阅用户数量、今日收入、本月收入。

### 数据分析
- **`ModelAnalysisAPI`**  
  返回模型使用的分析数据。

- **`RequestAnalysisAPI`**  
  返回请求的分析数据。

- **`BillingAnalysisAPI`**  
  返回计费的分析数据。

- **`ErrorAnalysisAPI`**  
  返回错误的分析数据。

### 邀请码和兑换码管理
- **`InvitationPaginationAPI`**  
  分页返回邀请码列表。

- **`GenerateInvitationAPI`**  
  根据请求生成指定数量和类型的邀请码。

- **`RedeemListAPI`**  
  返回兑换码的列表数据。

- **`GenerateRedeemAPI`**  
  根据请求生成指定数量的兑换码。

### 用户管理
- **`UserPaginationAPI`**  
  分页返回用户列表，支持通过搜索关键词过滤。

- **`UserQuotaAPI`**  
  修改指定用户的配额。

- **`UserSubscriptionAPI`**  
  修改指定用户的订阅时长。

### 管理员密码管理
- **`UpdateRootPasswordAPI`**  
  更新管理员的根密码。

