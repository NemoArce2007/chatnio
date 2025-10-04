# Channel 模块 API 文档

## 模块概览

Channel 模块负责管理 AI 模型通道、计费规则和系统配置。该模块提供了完整的通道管理、计费管理和系统配置管理功能。

> **源代码位置**: channel/router.go 第5-22行 - Register函数

## 核心数据结构

### 1. Channel 结构体
> **源代码位置**: channel/types.go 第3-20行 - Channel结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| Id | int | id | 通道唯一标识符 |
| Name | string | name | 通道名称 |
| Type | string | type | 通道类型 |
| Priority | int | priority | 优先级 |
| Weight | int | weight | 权重 |
| Models | []string | models | 支持的模型列表 |
| Retry | int | retry | 重试次数 |
| Secret | string | secret | 密钥 |
| Endpoint | string | endpoint | 端点地址 |
| Mapper | string | mapper | 模型映射配置 |
| State | bool | state | 通道状态（启用/禁用） |
| Group | []string | group | 所属组 |
| Reflect | *map[string]string | - | 模型反射映射（内部使用） |
| HitModels | *[]string | - | 命中的模型列表（内部使用） |
| ExcludeModels | *[]string | - | 排除的模型列表（内部使用） |

### 2. Charge 结构体
> **源代码位置**: channel/types.go 第34-41行 - Charge结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| Id | int | id | 计费规则唯一标识符 |
| Type | string | type | 计费类型 |
| Models | []string | models | 适用的模型列表 |
| Input | float32 | input | 输入计费单价 |
| Output | float32 | output | 输出计费单价 |
| Anonymous | bool | anonymous | 是否支持匿名用户 |

### 3. SystemConfig 结构体
> **源代码位置**: channel/system.go 第32-36行 - SystemConfig结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| General | generalState | general | 通用配置 |
| Mail | mailState | mail | 邮件配置 |
| Search | searchState | search | 搜索配置 |

#### generalState 子结构
> **源代码位置**: channel/system.go 第14-18行 - generalState结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| Title | string | title | 应用标题 |
| Logo | string | logo | 应用Logo |
| Backend | string | backend | 后端地址 |

#### mailState 子结构
> **源代码位置**: channel/system.go 第20-26行 - mailState结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| Host | string | host | SMTP服务器地址 |
| Port | int | port | SMTP端口 |
| Username | string | username | SMTP用户名 |
| Password | string | password | SMTP密码 |
| From | string | from | 发件人地址 |

#### searchState 子结构
> **源代码位置**: channel/system.go 第28-31行 - searchState结构体

| 字段名 | 类型 | JSON标签 | 描述 |
|--------|------|----------|------|
| Endpoint | string | endpoint | 搜索服务端点 |
| Query | int | query | 查询数量限制 |

## API 端点详情

### 1. 获取系统信息
**端点**: `GET /info`  
**功能**: 获取系统基本信息（标题和Logo）

> **源代码位置**: channel/controller.go 第9-11行 - GetInfo函数

#### 响应格式
```json
{
  "title": "Chat Nio",
  "logo": "https://chatnio.net/favicon.ico"
}
```

### 2. 通道管理 API

#### 2.1 获取通道列表
**端点**: `GET /admin/channel/list`  
**功能**: 获取所有通道列表

> **源代码位置**: channel/controller.go 第43-48行 - GetChannelList函数

#### 响应格式
```json
{
  "status": true,
  "data": [
    {
      "id": 1,
      "name": "OpenAI",
      "type": "openai",
      "priority": 1,
      "weight": 100,
      "models": ["gpt-3.5-turbo", "gpt-4"],
      "retry": 3,
      "secret": "sk-xxx",
      "endpoint": "https://api.openai.com",
      "mapper": "",
      "state": true,
      "group": ["default"]
    }
  ]
}
```

#### 2.2 获取单个通道
**端点**: `GET /admin/channel/get/:id`  
**功能**: 根据ID获取特定通道信息

> **源代码位置**: channel/controller.go 第50-58行 - GetChannel函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 通道ID |

#### 响应格式
```json
{
  "status": true,
  "data": {
    "id": 1,
    "name": "OpenAI",
    "type": "openai",
    "priority": 1,
    "weight": 100,
    "models": ["gpt-3.5-turbo", "gpt-4"],
    "retry": 3,
    "secret": "sk-xxx",
    "endpoint": "https://api.openai.com",
    "mapper": "",
    "state": true,
    "group": ["default"]
  }
}
```

#### 2.3 创建通道
**端点**: `POST /admin/channel/create`  
**功能**: 创建新的通道

> **源代码位置**: channel/controller.go 第60-75行 - CreateChannel函数

#### 请求参数
> **源代码位置**: channel/types.go 第3-20行 - Channel结构体

请求体为 Channel 结构体的 JSON 格式（不包含 id 字段）

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

#### 2.4 更新通道
**端点**: `POST /admin/channel/update/:id`  
**功能**: 更新指定通道信息

> **源代码位置**: channel/controller.go 第77-95行 - UpdateChannel函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 通道ID |

#### 请求参数
> **源代码位置**: channel/types.go 第3-20行 - Channel结构体

请求体为 Channel 结构体的 JSON 格式

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

#### 2.5 删除通道
**端点**: `GET /admin/channel/delete/:id`  
**功能**: 删除指定通道

> **源代码位置**: channel/controller.go 第13-21行 - DeleteChannel函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 通道ID |

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

#### 2.6 激活通道
**端点**: `GET /admin/channel/activate/:id`  
**功能**: 激活指定通道

> **源代码位置**: channel/controller.go 第23-31行 - ActivateChannel函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 通道ID |

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

#### 2.7 停用通道
**端点**: `GET /admin/channel/deactivate/:id`  
**功能**: 停用指定通道

> **源代码位置**: channel/controller.go 第33-41行 - DeactivateChannel函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 通道ID |

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

### 3. 计费管理 API

#### 3.1 获取计费规则列表
**端点**: `GET /admin/charge/list`  
**功能**: 获取所有计费规则

> **源代码位置**: channel/controller.go 第114-119行 - GetChargeList函数

#### 响应格式
```json
{
  "status": true,
  "data": [
    {
      "id": 1,
      "type": "token",
      "models": ["gpt-3.5-turbo"],
      "input": 0.001,
      "output": 0.002,
      "anonymous": false
    }
  ]
}
```

#### 3.2 设置计费规则
**端点**: `POST /admin/charge/set`  
**功能**: 创建或更新计费规则

> **源代码位置**: channel/controller.go 第97-112行 - SetCharge函数

#### 请求参数
> **源代码位置**: channel/types.go 第34-41行 - Charge结构体

请求体为 Charge 结构体的 JSON 格式

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

#### 3.3 删除计费规则
**端点**: `GET /admin/charge/delete/:id`  
**功能**: 删除指定计费规则

> **源代码位置**: channel/controller.go 第121-129行 - DeleteCharge函数

#### 路径参数
| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| id | int | 是 | 计费规则ID |

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

### 4. 系统配置 API

#### 4.1 获取系统配置
**端点**: `GET /admin/config/view`  
**功能**: 获取当前系统配置

> **源代码位置**: channel/controller.go 第131-136行 - GetConfig函数

#### 响应格式
```json
{
  "status": true,
  "data": {
    "general": {
      "title": "Chat Nio",
      "logo": "https://chatnio.net/favicon.ico",
      "backend": "https://api.chatnio.net"
    },
    "mail": {
      "host": "smtp.gmail.com",
      "port": 587,
      "username": "user@gmail.com",
      "password": "password",
      "from": "noreply@chatnio.net"
    },
    "search": {
      "endpoint": "https://duckduckgo-api.vercel.app",
      "query": 5
    }
  }
}
```

#### 4.2 更新系统配置
**端点**: `POST /admin/config/update`  
**功能**: 更新系统配置

> **源代码位置**: channel/controller.go 第138-153行 - UpdateConfig函数

#### 请求参数
> **源代码位置**: channel/system.go 第32-36行 - SystemConfig结构体

请求体为 SystemConfig 结构体的 JSON 格式

#### 响应格式
```json
{
  "status": true,
  "error": null
}
```

## 核心业务逻辑

### 1. 通道管理器 (Manager)
> **源代码位置**: channel/manager.go 第18-174行 - Manager相关函数

通道管理器负责：
- 通道序列管理和排序
- 模型支持检测
- 预检序列生成
- 通道状态管理

### 2. 计费管理器 (ChargeManager)
> **源代码位置**: channel/charge.go 第79-191行 - ChargeManager相关函数

计费管理器负责：
- 计费规则管理
- 模型计费映射
- 非计费模型管理
- 计费类型处理

### 3. 系统配置管理器 (SystemConfig)
> **源代码位置**: channel/system.go 第38-137行 - SystemConfig相关函数

系统配置管理器负责：
- 应用基本信息管理
- 邮件服务配置
- 搜索服务配置
- 配置持久化

### 4. 通道调度器 (Ticker)
> **源代码位置**: channel/ticker.go 第5-96行 - Ticker相关函数

通道调度器负责：
- 按组过滤通道
- 通道负载均衡
- 故障转移处理

## 错误处理

所有 API 端点都遵循统一的错误响应格式：

```json
{
  "status": false,
  "error": "错误描述信息"
}
```

常见错误类型：
- 参数验证错误：400 Bad Request
- 通道不存在：返回 "channel not found"
- 配置保存失败：返回具体的错误信息

## 使用示例

### 创建通道示例
```bash
curl -X POST http://localhost:8080/admin/channel/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "OpenAI GPT-4",
    "type": "openai",
    "priority": 1,
    "weight": 100,
    "models": ["gpt-4", "gpt-4-turbo"],
    "retry": 3,
    "secret": "sk-your-api-key",
    "endpoint": "https://api.openai.com",
    "state": true,
    "group": ["premium"]
  }'
```

### 设置计费规则示例
```bash
curl -X POST http://localhost:8080/admin/charge/set \
  -H "Content-Type: application/json" \
  -d '{
    "id": -1,
    "type": "token",
    "models": ["gpt-4"],
    "input": 0.03,
    "output": 0.06,
    "anonymous": false
  }'
```

### 更新系统配置示例
```bash
curl -X POST http://localhost:8080/admin/config/update \
  -H "Content-Type: application/json" \
  -d '{
    "general": {
      "title": "My Chat App",
      "logo": "https://example.com/logo.png",
      "backend": "https://api.example.com"
    },
    "mail": {
      "host": "smtp.example.com",
      "port": 587,
      "username": "admin@example.com",
      "password": "your-password",
      "from": "noreply@example.com"
    },
    "search": {
      "endpoint": "https://search-api.example.com",
      "query": 10
    }
  }'
```