# Adapter 模块文档

## 概述

Adapter 模块是 ChatNio 项目的核心适配器层，负责统一管理和适配各种 AI 服务提供商的接口。该模块提供了标准化的接口抽象，使得系统能够无缝集成多个 AI 服务。

## API 接口

### Midjourney 通知回调 API

#### `POST /mj/notify`

这是 Midjourney 适配器的核心回调接口，用于接收来自 Midjourney 服务的任务状态更新通知。

**功能说明：**
- 接收 Midjourney 图像生成任务的状态更新
- 支持白名单 IP 访问控制
- 实时更新任务进度和结果
- 提供任务失败原因反馈

**请求方式：** `POST`

**访问控制：**
- 支持 IP 白名单机制
- 如果未配置白名单，则允许所有 IP 访问
- 被禁止的 IP 将返回 `403 Forbidden`

**请求体格式：**
```json
{
  "id": "任务ID",
  "action": "动作类型",
  "status": "任务状态",
  "prompt": "原始提示词",
  "promptEn": "英文提示词",
  "description": "任务描述",
  "submitTime": 1234567890,
  "startTime": 1234567890,
  "finishTime": 1234567890,
  "progress": "50%",
  "imageUrl": "生成的图像URL",
  "failReason": "失败原因（可选）"
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 任务唯一标识符 |
| `action` | string | 是 | 执行的动作类型 |
| `status` | string | 是 | 任务状态：`IN_PROGRESS`、`SUCCESS`、`FAILURE` |
| `prompt` | string | 是 | 用户输入的原始提示词 |
| `promptEn` | string | 否 | 翻译后的英文提示词 |
| `description` | string | 否 | 任务描述信息 |
| `submitTime` | int64 | 否 | 任务提交时间戳 |
| `startTime` | int64 | 否 | 任务开始时间戳 |
| `finishTime` | int64 | 否 | 任务完成时间戳 |
| `progress` | string | 否 | 任务进度百分比（如："75%"） |
| `imageUrl` | string | 否 | 生成的图像URL（成功时） |
| `failReason` | interface{} | 否 | 失败原因（失败时） |

**任务状态说明：**

| 状态 | 说明 |
|------|------|
| `IN_PROGRESS` | 任务进行中 |
| `SUCCESS` | 任务成功完成 |
| `FAILURE` | 任务执行失败 |

**响应格式：**
```json
{
  "status": true
}
```

**响应说明：**
- `status`: boolean - 表示通知处理是否成功
- HTTP 状态码：
  - `200 OK`: 通知处理成功
  - `400 Bad Request`: 请求格式错误
  - `403 Forbidden`: IP 不在白名单中

**工作流程：**

1. **接收通知**：API 接收来自 Midjourney 服务的 POST 请求
2. **IP 验证**：检查请求来源 IP 是否在白名单中
3. **参数验证**：验证请求体格式和必要参数
4. **状态过滤**：只处理有效的状态更新（`IN_PROGRESS`、`SUCCESS`、`FAILURE`）
5. **数据存储**：将任务状态更新存储到缓存中
6. **响应返回**：返回处理结果

**存储机制：**
- 使用 Redis 缓存存储任务状态
- 缓存键格式：`nio:mj-task:{taskId}`
- 缓存过期时间：1小时（3600秒）
- 存储内容包括：图像URL、失败原因、进度、状态

**安全特性：**
- IP 白名单访问控制
- 请求参数验证
- 状态值合法性检查
- 错误处理和日志记录

**使用场景：**
1. **图像生成进度跟踪**：实时更新用户界面显示生成进度
2. **任务结果通知**：在图像生成完成后通知用户
3. **错误处理**：当生成失败时提供详细的错误信息
4. **状态同步**：保持 ChatNio 系统与 Midjourney 服务的状态同步

**配置说明：**
- 白名单配置通过 `SaveWhiteList()` 函数设置
- 支持多个 IP 地址，用逗号分隔
- 空白名单表示允许所有 IP 访问

**注意事项：**
- 该 API 主要用于内部服务间通信
- 建议在生产环境中配置严格的 IP 白名单
- 任务状态更新具有实时性要求
- 缓存过期后任务状态将被清理

## 相关文件

- `adapter/router.go`: API 路由注册
- `adapter/midjourney/expose.go`: NotifyAPI 实现
- `adapter/midjourney/types.go`: 数据结构定义
- `adapter/midjourney/storage.go`: 缓存存储逻辑
- `adapter/midjourney/api.go`: Midjourney API 调用

###  完整的请求流程
1. 
   用户发起聊天 → 其他模块的HTTP路由
2. 
   路由转发 → Manager模块处理
3. 
   适配器调用 → Adapter模块内部函数（非HTTP路由）
4. 
   AI服务请求 → 各AI服务商的外部API
5. 
   异步回调 （仅Midjourney）→ POST /mj/notify 路由