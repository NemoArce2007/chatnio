# Manager 模块 API 文档

## 模块概览

Manager 模块是 ChatNIO 的核心管理模块，负责处理聊天请求、图像生成、用户会话管理、广播功能等核心业务逻辑。该模块包含以下主要功能：

- 聊天 API 和 WebSocket 连接管理
- OpenAI 兼容的 Chat Completions API
- 图像生成 API
- 用户使用情况统计
- 对话管理
- 广播功能

## 路由概览

> **源代码位置**: manager/router.go 第8-18行 - Register函数

```go
func Register(app *gin.RouterGroup) {
    app.GET("/chat", ChatAPI)
    app.GET("/v1/models", ModelAPI)
    app.GET("/v1/charge", ChargeAPI)
    app.GET("/dashboard/billing/usage", GetBillingUsage)
    app.GET("/dashboard/billing/subscription", GetSubscription)
    app.POST("/v1/chat/completions", ChatRelayAPI)
    app.POST("/v1/images/generations", ImagesRelayAPI)

    broadcast.Register(app)
}
```

## 核心数据结构

### 1. 消息结构体

> **源代码位置**: manager/types.go 第8-13行 - Message结构体

| 字段名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| role | string | 是 | 消息角色（user/assistant/system/tool） |
| content | interface{} | 是 | 消息内容，可以是字符串或多媒体内容 |
| tool_call_id | *string | 否 | 工具调用ID（仅tool角色使用） |
| tool_calls | *globals.ToolCalls | 否 | 工具调用列表（仅assistant角色使用） |

### 2. 中继请求表单

> **源代码位置**: manager/types.go 第28-42行 - RelayForm结构体

| 字段名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| model | string | 是 | 模型名称 |
| messages | []Message | 是 | 消息列表 |
| stream | bool | 否 | 是否流式响应 |
| max_tokens | int | 否 | 最大token数 |
| presence_penalty | *float32 | 否 | 存在惩罚 |
| frequency_penalty | *float32 | 否 | 频率惩罚 |
| repetition_penalty | *float32 | 否 | 重复惩罚 |
| temperature | *float32 | 否 | 温度参数 |
| top_p | *float32 | 否 | Top-P 采样 |
| top_k | *int | 否 | Top-K 采样 |
| tools | *globals.FunctionTools | 否 | 工具列表 |
| tool_choice | *interface{} | 否 | 工具选择策略 |
| official | bool | 否 | 是否使用官方API |

### 3. 中继响应结构体

> **源代码位置**: manager/types.go 第50-59行 - RelayResponse结构体

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gpt-3.5-turbo",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  },
  "quota": 0.001
}
```

### 4. 图像生成请求表单

> **源代码位置**: manager/types.go 第78-82行 - RelayImageForm结构体

| 字段名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| model | string | 是 | 图像生成模型名称 |
| prompt | string | 是 | 图像生成提示词 |
| n | *int | 否 | 生成图像数量 |

### 5. WebSocket 认证表单

> **源代码位置**: manager/manager.go 第14-18行 - WebsocketAuthForm结构体

| 字段名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| token | string | 是 | 认证令牌 |
| id | int64 | 是 | 会话ID |
| ref | string | 否 | 引用标识 |

## API 端点详情

### 1. 聊天 WebSocket API

**端点**: `GET /chat`
**功能**: 建立 WebSocket 连接进行实时聊天

> **源代码位置**: manager/manager.go 第35-91行 - ChatAPI函数

#### 连接流程
1. 建立 WebSocket 连接
2. 发送认证表单进行身份验证
3. 处理各种类型的消息（聊天、停止、分享、重启、掩码）

#### 支持的消息类型
- `ChatType`: 发送聊天消息
- `StopType`: 停止当前对话
- `ShareType`: 分享对话
- `RestartType`: 重启对话
- `MaskType`: 应用对话掩码

### 2. 模型列表 API

**端点**: `GET /v1/models`
**功能**: 获取可用模型列表

> **源代码位置**: manager/relay.go 第9-11行 - ModelAPI函数

#### 响应格式
```json
{
  "data": [
    {
      "id": "gpt-3.5-turbo",
      "object": "model",
      "created": 1234567890,
      "owned_by": "openai"
    }
  ]
}
```

### 3. 计费规则 API

**端点**: `GET /v1/charge`
**功能**: 获取模型计费规则

> **源代码位置**: manager/relay.go 第13-15行 - ChargeAPI函数

### 4. Chat Completions API

**端点**: `POST /v1/chat/completions`
**功能**: OpenAI 兼容的聊天完成 API

> **源代码位置**: manager/chat_completions.go 第19-68行 - ChatRelayAPI函数

#### 请求参数
使用 `RelayForm` 结构体，支持流式和非流式响应。

#### 特殊功能
- 支持 `web-` 前缀模型进行网络搜索
- 支持 `-official` 后缀使用官方 API
- 自动处理用户配额和权限验证

### 5. 图像生成 API

**端点**: `POST /v1/images/generations`
**功能**: 生成图像

> **源代码位置**: manager/images.go 第17-58行 - ImagesRelayAPI函数

#### 请求参数
使用 `RelayImageForm` 结构体。

#### 响应格式
> **源代码位置**: manager/types.go 第88-91行 - RelayImageResponse结构体

```json
{
  "created": 1234567890,
  "data": [
    {
      "url": "https://example.com/image.png"
    }
  ]
}
```

### 6. 用户使用情况 API

**端点**: `GET /dashboard/billing/usage`
**功能**: 获取用户使用情况统计

> **源代码位置**: manager/usage.go 第24-35行 - GetBillingUsage函数

#### 响应格式
> **源代码位置**: manager/usage.go 第10-13行 - BillingResponse结构体

```json
{
  "object": "list",
  "total_usage": 123.45
}
```

### 7. 订阅信息 API

**端点**: `GET /dashboard/billing/subscription`
**功能**: 获取用户订阅信息

> **源代码位置**: manager/usage.go 第37-60行 - GetSubscription函数

#### 响应格式
> **源代码位置**: manager/usage.go 第15-23行 - SubscriptionResponse结构体

```json
{
  "object": "billing_subscription",
  "soft_limit": 10000,
  "hard_limit": 20000,
  "system_hard_limit": 100000000,
  "soft_limit_usd": 13.7,
  "hard_limit_usd": 27.4,
  "system_hard_limit_usd": 1000000
}
```

## 子模块

### 1. Conversation 子模块

> **源代码位置**: manager/conversation/router.go 第5-18行 - Register函数

#### API 端点
- `GET /conversation/list` - 获取对话列表
- `GET /conversation/load` - 加载对话
- `GET /conversation/delete` - 删除对话
- `GET /conversation/clean` - 清理对话
- `POST /conversation/share` - 分享对话
- `GET /conversation/view` - 查看分享的对话
- `GET /conversation/share/list` - 获取分享列表
- `GET /conversation/share/delete` - 删除分享

### 2. Broadcast 子模块

> **源代码位置**: manager/broadcast/router.go 第5-8行 - Register函数

#### API 端点
- `GET /broadcast/view` - 查看广播
- `GET /broadcast/list` - 获取广播列表
- `POST /broadcast/create` - 创建广播

#### 数据结构

**广播信息结构体**
> **源代码位置**: manager/broadcast/types.go 第8-13行 - Info结构体

| 字段名 | 类型 | 描述 |
|--------|------|------|
| index | int | 广播索引 |
| content | string | 广播内容 |
| poster | string | 发布者 |
| created_at | string | 创建时间 |

**创建广播请求**
> **源代码位置**: manager/broadcast/types.go 第19-21行 - createRequest结构体

| 字段名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| content | string | 是 | 广播内容 |

## 核心功能实现

### 1. 聊天处理器

> **源代码位置**: manager/chat.go 第57-159行 - ChatHandler函数

聊天处理器负责：
- 用户权限验证
- 模型可用性检查
- 缓存数据处理
- 流式响应发送
- 配额收集和管理
- 错误处理和恢复

### 2. 配额收集

> **源代码位置**: manager/chat.go 第20-34行 - CollectQuota函数

配额收集机制：
- 根据计费类型（按次数或按token）收集费用
- 处理错误情况下的配额回退
- 支持不可计费的请求类型

### 3. 流式响应处理

> **源代码位置**: manager/chat_completions.go 第155-196行 - sendStreamTranshipmentResponse函数

流式响应特性：
- 使用 goroutine 异步处理
- 支持 Server-Sent Events (SSE)
- 实时错误处理和状态更新
- 自动配额计算和收集

## 错误处理

### 错误响应格式

> **源代码位置**: manager/types.go 第72-80行 - RelayErrorResponse和TranshipmentError结构体

```json
{
  "error": {
    "message": "错误描述",
    "type": "错误类型"
  }
}
```

### 常见错误类型
- `authentication_error` - 认证错误
- `invalid_request_error` - 请求格式错误
- `quota_exceeded_error` - 配额超限
- `image_generation_error` - 图像生成错误
- `chatnio_api_error` - 通用API错误

## 安全特性

1. **API Key 验证**: 支持 Bearer Token 和 sk- 前缀的 API Key
2. **用户代理检查**: 验证请求来源的合法性
3. **配额限制**: 实时监控和限制用户使用量
4. **权限控制**: 基于用户组和订阅计划的模型访问控制
5. **错误恢复**: 自动处理异常情况并回退相关操作

## 性能优化

1. **缓存机制**: 支持请求结果缓存以提高响应速度
2. **流式处理**: 减少内存占用和响应延迟
3. **异步处理**: 使用 goroutine 处理耗时操作
4. **连接池**: 复用数据库和缓存连接
5. **智能路由**: 根据模型类型和用户权限智能选择处理策略