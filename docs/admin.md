# Admin 管理模块 API 文档

## 模块概览

Admin 模块是 ChatNio 系统的管理后台模块，提供系统分析、用户管理、邀请码管理、兑换码管理等核心管理功能。

### 文件结构
- `admin/router.go` - 路由注册和端点定义
- `admin/types.go` - 数据结构定义
- `admin/controller.go` - API 控制器实现
- `admin/analysis.go` - 数据分析功能
- `admin/invitation.go` - 邀请码管理
- `admin/redeem.go` - 兑换码管理
- `admin/user.go` - 用户管理
- `admin/statistic.go` - 统计数据记录
- `admin/format.go` - 格式化工具函数

## 路由概览

> **源代码位置**: <mcfile name="router.go" path="admin/router.go"></mcfile> 第8-27行 - `Register` 函数

所有管理 API 端点都注册在 `/admin` 路径下，包含以下功能模块：
- 数据分析 (`/admin/analytics/*`)
- 邀请码管理 (`/admin/invitation/*`)
- 兑换码管理 (`/admin/redeem/*`)
- 用户管理 (`/admin/user/*`)

## API 端点详情

### 1. 系统信息概览

**端点**: `GET /admin/analytics/info`  
**功能**: 获取系统基本统计信息

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第36-44行 - `InfoAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第5-9行 - `InfoForm` 结构体

```json
{
  "billing_today": 123.45,
  "billing_month": 1234.56,
  "subscription_count": 100
}
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| billing_today | float32 | 今日账单金额 |
| billing_month | float32 | 本月账单金额 |
| subscription_count | int64 | 订阅用户数量 |

### 2. 模型使用分析

**端点**: `GET /admin/analytics/model`  
**功能**: 获取模型使用情况统计图表数据

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第46-49行 - `ModelAnalysisAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第16-20行 - `ModelChartForm` 结构体

```json
{
  "date": ["1/1", "1/2", "1/3"],
  "value": [
    {
      "model": "gpt-3.5-turbo",
      "data": [100, 200, 150]
    }
  ]
}
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| date | []string | 日期数组 |
| value | []ModelData | 模型数据数组 |

**ModelData 结构体**:
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第11-14行 - `ModelData` 结构体

| 字段名 | 类型 | 描述 |
|--------|------|------|
| model | string | 模型名称 |
| data | []int64 | 使用量数据 |

### 3. 请求量分析

**端点**: `GET /admin/analytics/request`  
**功能**: 获取请求量统计图表数据

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第51-54行 - `RequestAnalysisAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第22-25行 - `RequestChartForm` 结构体

```json
{
  "date": ["1/1", "1/2", "1/3"],
  "value": [100, 200, 150]
}
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| date | []string | 日期数组 |
| value | []int64 | 请求量数据 |

### 4. 账单分析

**端点**: `GET /admin/analytics/billing`  
**功能**: 获取账单统计图表数据

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第56-59行 - `BillingAnalysisAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第27-30行 - `BillingChartForm` 结构体

```json
{
  "date": ["1/1", "1/2", "1/3"],
  "value": [123.45, 234.56, 345.67]
}
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| date | []string | 日期数组 |
| value | []float32 | 账单金额数据 |

### 5. 错误统计分析

**端点**: `GET /admin/analytics/error`  
**功能**: 获取错误统计图表数据

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第61-64行 - `ErrorAnalysisAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第32-35行 - `ErrorChartForm` 结构体

```json
{
  "date": ["1/1", "1/2", "1/3"],
  "value": [10, 5, 8]
}
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| date | []string | 日期数组 |
| value | []int64 | 错误次数数据 |

### 6. 邀请码列表

**端点**: `GET /admin/invitation/list?page=0`  
**功能**: 分页获取邀请码列表

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第67-72行 - `InvitationPaginationAPI` 函数

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| page | int | 否 | 页码，从0开始 |

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第37-42行 - `PaginationForm` 结构体

```json
{
  "status": true,
  "total": 10,
  "data": [
    {
      "code": "premium-abc123",
      "quota": 100.0,
      "type": "premium",
      "used": false,
      "updated_at": "2024-01-01 12:00:00"
    }
  ],
  "message": ""
}
```

**InvitationData 结构体**:
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第44-50行 - `InvitationData` 结构体

| 字段名 | 类型 | 描述 |
|--------|------|------|
| code | string | 邀请码 |
| quota | float32 | 额度 |
| type | string | 类型 |
| used | bool | 是否已使用 |
| updated_at | string | 更新时间 |

### 7. 生成邀请码

**端点**: `POST /admin/invitation/generate`  
**功能**: 批量生成邀请码

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第74-87行 - `GenerateInvitationAPI` 函数

#### 请求参数
> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第11-15行 - `GenerateInvitationForm` 结构体

```json
{
  "type": "premium",
  "quota": 100.0,
  "number": 10
}
```

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| type | string | 是 | 邀请码类型 |
| quota | float32 | 是 | 额度 |
| number | int | 是 | 生成数量 |

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第57-61行 - `InvitationGenerateResponse` 结构体

```json
{
  "status": true,
  "message": "",
  "data": ["premium-abc123", "premium-def456"]
}
```

### 8. 兑换码列表

**端点**: `GET /admin/redeem/list`  
**功能**: 获取兑换码统计信息

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第62-65行 - `RedeemListAPI` 函数

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第52-56行 - `RedeemData` 结构体

```json
[
  {
    "quota": 50.0,
    "used": 5.0,
    "total": 10.0
  }
]
```

| 字段名 | 类型 | 描述 |
|--------|------|------|
| quota | float32 | 兑换码额度 |
| used | float32 | 已使用数量 |
| total | float32 | 总数量 |

### 9. 生成兑换码

**端点**: `POST /admin/redeem/generate`  
**功能**: 批量生成兑换码

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第89-102行 - `GenerateRedeemAPI` 函数

#### 请求参数
> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第17-20行 - `GenerateRedeemForm` 结构体

```json
{
  "quota": 50.0,
  "number": 10
}
```

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| quota | float32 | 是 | 兑换码额度 |
| number | int | 是 | 生成数量 |

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第63-67行 - `RedeemGenerateResponse` 结构体

```json
{
  "status": true,
  "message": "",
  "data": ["nio-abc123", "nio-def456"]
}
```

### 10. 用户列表

**端点**: `GET /admin/user/list?page=0&search=`  
**功能**: 分页获取用户列表，支持搜索

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第104-110行 - `UserPaginationAPI` 函数

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| page | int | 否 | 页码，从0开始 |
| search | string | 否 | 搜索关键词 |

#### 响应格式
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第37-42行 - `PaginationForm` 结构体

```json
{
  "status": true,
  "total": 10,
  "data": [
    {
      "id": 1,
      "username": "user1",
      "is_admin": false,
      "quota": 100.0,
      "used_quota": 50.0,
      "is_subscribed": true,
      "total_month": 12,
      "enterprise": false,
      "level": 1
    }
  ],
  "message": ""
}
```

**UserData 结构体**:
> **源代码位置**: <mcfile name="types.go" path="admin/types.go"></mcfile> 第69-79行 - `UserData` 结构体

| 字段名 | 类型 | 描述 |
|--------|------|------|
| id | int64 | 用户ID |
| username | string | 用户名 |
| is_admin | bool | 是否管理员 |
| quota | float32 | 总额度 |
| used_quota | float32 | 已使用额度 |
| is_subscribed | bool | 是否订阅 |
| total_month | int64 | 订阅总月数 |
| enterprise | bool | 是否企业用户 |
| level | int | 订阅等级 |

### 11. 用户额度操作

**端点**: `POST /admin/user/quota`  
**功能**: 增加或减少用户额度

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第112-131行 - `UserQuotaAPI` 函数

#### 请求参数
> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第22-25行 - `QuotaOperationForm` 结构体

```json
{
  "id": 1,
  "quota": 50.0
}
```

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int64 | 是 | 用户ID |
| quota | float32 | 是 | 额度变化量（正数增加，负数减少） |

#### 响应格式

```json
{
  "status": true
}
```

### 12. 用户订阅操作

**端点**: `POST /admin/user/subscription`  
**功能**: 增加或减少用户订阅时长

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第133-152行 - `UserSubscriptionAPI` 函数

#### 请求参数
> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第27-30行 - `SubscriptionOperationForm` 结构体

```json
{
  "id": 1,
  "month": 12
}
```

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int64 | 是 | 用户ID |
| month | int64 | 是 | 月数变化量（正数增加，负数减少） |

#### 响应格式

```json
{
  "status": true
}
```

### 13. 更新管理员密码

**端点**: `POST /admin/user/root`  
**功能**: 更新 root 管理员密码

> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第154-175行 - `UpdateRootPasswordAPI` 函数

#### 请求参数
> **源代码位置**: <mcfile name="controller.go" path="admin/controller.go"></mcfile> 第32-34行 - `UpdateRootPasswordForm` 结构体

```json
{
  "password": "newpassword123"
}
```

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| password | string | 是 | 新密码（6-36位） |

#### 响应格式

```json
{
  "status": true
}
```

## 核心功能实现

### 数据分析功能
> **源代码位置**: <mcfile name="analysis.go" path="admin/analysis.go"></mcfile>

- `GetSubscriptionUsers` (第19-28行) - 获取订阅用户数量
- `GetBillingToday` (第30-32行) - 获取今日账单
- `GetBillingMonth` (第34-36行) - 获取本月账单
- `GetModelData` (第38-54行) - 获取模型使用数据
- `GetRequestData` (第56-64行) - 获取请求量数据
- `GetBillingData` (第66-76行) - 获取账单数据
- `GetErrorData` (第78-88行) - 获取错误数据

### 邀请码管理功能
> **源代码位置**: <mcfile name="invitation.go" path="admin/invitation.go"></mcfile>

- `GetInvitationPagination` (第11-45行) - 分页获取邀请码
- `NewInvitationCode` (第47-53行) - 创建新邀请码
- `GenerateInvitations` (第55-84行) - 批量生成邀请码

### 兑换码管理功能
> **源代码位置**: <mcfile name="redeem.go" path="admin/redeem.go"></mcfile>

- `GetRedeemData` (第9-26行) - 获取兑换码统计
- `GenerateRedeemCodes` (第28-45行) - 批量生成兑换码
- `CreateRedeemCode` (第47-60行) - 创建单个兑换码

### 用户管理功能
> **源代码位置**: <mcfile name="user.go" path="admin/user.go"></mcfile>

- `GetUserPagination` (第13-75行) - 分页获取用户列表
- `QuotaOperation` (第77-85行) - 用户额度操作
- `SubscriptionOperation` (第87-96行) - 用户订阅操作
- `UpdateRootPassword` (第98-112行) - 更新管理员密码

### 统计数据记录
> **源代码位置**: <mcfile name="statistic.go" path="admin/statistic.go"></mcfile>

- `IncrErrorRequest` (第10-12行) - 记录错误请求
- `IncrBillingRequest` (第14-17行) - 记录账单请求
- `IncrRequest` (第19-21行) - 记录普通请求
- `IncrModelRequest` (第23-25行) - 记录模型请求
- `AnalysisRequest` (第27-37行) - 分析请求统计

### 格式化工具
> **源代码位置**: <mcfile name="format.go" path="admin/format.go"></mcfile>

- `getMonth` (第6-9行) - 获取月份格式
- `getDay` (第11-14行) - 获取日期格式
- `getDays` (第16-24行) - 获取日期数组
- 各种缓存键格式化函数 (第26-46行)

## 错误处理

所有 API 端点都遵循统一的错误处理格式：

```json
{
  "status": false,
  "message": "错误信息",
  "error": "详细错误描述"
}
```

## 权限要求

所有 Admin API 端点都需要管理员权限验证，通过中间件进行权限检查。

## 数据库依赖

Admin 模块依赖以下数据库表：
- `auth` - 用户认证表
- `quota` - 用户额度表
- `subscription` - 订阅信息表
- `invitation` - 邀请码表
- `redeem` - 兑换码表

## 缓存依赖

使用 Redis 缓存存储统计数据，键格式：
- `nio:err-analysis-{date}` - 错误统计
- `nio:billing-analysis-{date}` - 账单统计
- `nio:request-analysis-{date}` - 请求统计
- `nio:model-analysis-{model}-{date}` - 模型统计