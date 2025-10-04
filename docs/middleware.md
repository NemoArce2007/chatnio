# ChatNio Middleware 模块 API 文档

## 概述

Middleware 模块是 ChatNio 系统的核心中间件层，负责处理请求的认证、跨域、限流、内置功能等关键功能。该模块采用 Gin 框架的中间件机制，为整个应用提供统一的请求处理流程。

**源代码位置**: `middleware/`

## 模块结构

```
middleware/
├── middleware.go    # 中间件注册和管理
├── auth.go         # 认证中间件
├── cors.go         # 跨域处理中间件
├── throttle.go     # 限流中间件
└── builtins.go     # 内置中间件功能
```

## 核心功能

### 1. 中间件注册管理

**源代码位置**: `middleware.go`

#### RegisterMiddleware 函数
```go
func RegisterMiddleware(app *gin.Engine) func()
```

**功能**: 注册所有中间件到 Gin 引擎，并返回清理函数

**中间件注册顺序**:
1. CORS 中间件 (`CORSMiddleware()`)
2. 内置中间件 (`BuiltinMiddleWare(db, cache)`)
3. 限流中间件 (`ThrottleMiddleware()`)
4. 认证中间件 (`AuthMiddleware()`)

**返回值**: 返回一个清理函数，用于关闭数据库和缓存连接

### 2. 认证中间件 (Authentication Middleware)

**源代码位置**: `auth.go`

#### 核心函数

##### ProcessToken
```go
func ProcessToken(c *gin.Context, token string) *auth.User
```
- **功能**: 处理 JWT Token 认证
- **参数**: Gin 上下文和 token 字符串
- **返回**: 用户对象或 nil
- **设置上下文**: `auth`, `user`, `agent`

##### ProcessKey
```go
func ProcessKey(c *gin.Context, key string) *auth.User
```
- **功能**: 处理 API Key 认证
- **参数**: Gin 上下文和 API key 字符串
- **安全特性**:
  - IP 黑名单检查
  - 失败次数统计
  - 自动拒绝恶意请求
- **错误响应**: 401/403 状态码

##### ProcessAuthorization
```go
func ProcessAuthorization(c *gin.Context) *auth.User
```
- **功能**: 统一处理 Authorization 头部认证
- **支持格式**:
  - `Bearer <token>` - JWT Token 认证
  - `sk-<key>` - API Key 认证
- **自动识别**: 根据前缀自动选择认证方式

##### AuthMiddleware
```go
func AuthMiddleware() gin.HandlerFunc
```
- **功能**: 主认证中间件函数
- **特性**:
  - 管理员权限检查
  - `/admin` 路径保护
  - 静态文件服务支持
  - 上下文设置: `admin` 标志

### 3. 跨域处理中间件 (CORS Middleware)

**源代码位置**: `cors.go`

#### CORSMiddleware
```go
func CORSMiddleware() gin.HandlerFunc
```

**功能**: 处理跨域资源共享 (CORS) 请求

**CORS 配置**:
- **允许的方法**: `GET, POST, PUT, DELETE, OPTIONS`
- **允许的头部**: 包含常见的认证和请求头
- **凭证支持**: `Access-Control-Allow-Credentials: true`
- **预检请求**: 自动处理 OPTIONS 请求
- **缓存时间**: 7200 秒

**安全特性**:
- 动态 Origin 检查 (`globals.OriginIsAllowed`)
- 开放模式支持 (`globals.OriginIsOpen`)

### 4. 限流中间件 (Throttle Middleware)

**源代码位置**: `throttle.go`

#### 数据结构

##### Limiter
```go
type Limiter struct {
    Duration int   // 时间窗口（秒）
    Count    int64 // 允许的请求数量
}
```

#### 核心函数

##### RateLimit
```go
func (l *Limiter) RateLimit(client *redis.Client, ip string, path string) bool
```
- **功能**: 执行速率限制检查
- **参数**: Redis 客户端、IP 地址、请求路径
- **返回**: true 表示被限流，false 表示允许通过

##### GetPrefixMap
```go
func GetPrefixMap[T comparable](s string, p map[string]T) *T
```
- **功能**: 根据路径前缀匹配限流规则
- **泛型支持**: 支持任意可比较类型
- **静态文件支持**: 自动处理 `/api` 前缀

##### ThrottleMiddleware
```go
func ThrottleMiddleware() gin.HandlerFunc
```
- **功能**: 主限流中间件函数
- **限流策略**: 基于 IP + 路径的组合限流
- **错误响应**: 返回友好的限流提示信息

#### 限流规则配置

| 路径前缀 | 时间窗口(秒) | 请求限制 | 用途 |
|---------|-------------|---------|------|
| `/login` | 10 | 20 | 登录接口 |
| `/register` | 120 | 10 | 注册接口 |
| `/verify` | 120 | 5 | 验证接口 |
| `/reset` | 120 | 10 | 密码重置 |
| `/apikey` | 1 | 2 | API Key 管理 |
| `/package` | 1 | 2 | 套餐管理 |
| `/quota` | 1 | 2 | 配额查询 |
| `/buy` | 1 | 2 | 购买接口 |
| `/subscribe` | 1 | 2 | 订阅管理 |
| `/subscription` | 1 | 2 | 订阅信息 |
| `/chat` | 1 | 5 | 聊天接口 |
| `/conversation` | 1 | 5 | 对话管理 |
| `/invite` | 7200 | 20 | 邀请功能 |
| `/redeem` | 1200 | 60 | 兑换功能 |
| `/v1` | 1 | 600 | API v1 接口 |
| `/dashboard` | 1 | 5 | 仪表板 |
| `/card` | 1 | 5 | 卡片管理 |
| `/generation` | 1 | 5 | 生成功能 |
| `/article` | 1 | 5 | 文章管理 |
| `/broadcast` | 1 | 2 | 广播功能 |

### 5. 内置中间件 (Builtin Middleware)

**源代码位置**: `builtins.go`

#### BuiltinMiddleWare
```go
func BuiltinMiddleWare(db *sql.DB, cache *redis.Client) gin.HandlerFunc
```

**功能**: 为请求上下文注入数据库和缓存连接

**注入的上下文**:
- `db`: MySQL 数据库连接
- `cache`: Redis 缓存连接

**用途**: 为后续中间件和处理函数提供数据库和缓存访问能力

## 中间件执行流程

```
请求 → CORS中间件 → 内置中间件 → 限流中间件 → 认证中间件 → 业务处理
```

### 执行顺序说明

1. **CORS 中间件**: 首先处理跨域请求，设置必要的 CORS 头部
2. **内置中间件**: 注入数据库和缓存连接到上下文
3. **限流中间件**: 检查请求频率，防止滥用
4. **认证中间件**: 验证用户身份和权限

## 安全特性

### 1. 认证安全
- **多重认证方式**: 支持 JWT Token 和 API Key
- **IP 黑名单**: 自动阻止恶意 IP
- **失败计数**: 记录认证失败次数
- **管理员保护**: `/admin` 路径需要管理员权限

### 2. 限流保护
- **分级限流**: 不同接口不同限制
- **IP 级别**: 基于客户端 IP 限流
- **路径匹配**: 支持前缀匹配规则
- **Redis 存储**: 使用 Redis 实现分布式限流

### 3. 跨域安全
- **Origin 验证**: 检查请求来源合法性
- **动态配置**: 支持运行时配置允许的域名
- **预检处理**: 正确处理 CORS 预检请求

## 性能优化

### 1. 缓存机制
- **Redis 缓存**: 使用 Redis 存储限流计数器
- **连接复用**: 数据库和缓存连接复用
- **上下文注入**: 避免重复创建连接

### 2. 高效匹配
- **前缀匹配**: 使用前缀匹配提高路由效率
- **泛型支持**: 类型安全的配置匹配
- **早期返回**: 限流检查失败时立即返回

## 错误处理

### 1. 认证错误
- **401 Unauthorized**: 认证失败
- **403 Forbidden**: IP 被禁止或权限不足
- **友好提示**: 返回清晰的错误信息

### 2. 限流错误
- **200 OK**: 返回限流提示（非标准做法）
- **请求中止**: 使用 `c.Abort()` 停止后续处理
- **用户友好**: 提供重试建议

## 配置管理

### 1. 环境配置
- **静态文件服务**: `serve_static` 配置项
- **API 前缀**: 支持 `/api` 前缀配置
- **动态开关**: 运行时配置支持

### 2. 限流配置
- **集中管理**: 所有限流规则集中在 `limits` 映射中
- **灵活调整**: 支持不同时间窗口和限制数量
- **易于扩展**: 新增路径限流规则简单

## 依赖关系

### 外部依赖
- `github.com/gin-gonic/gin`: Web 框架
- `github.com/go-redis/redis/v8`: Redis 客户端
- `github.com/spf13/viper`: 配置管理
- `database/sql`: 数据库接口

### 内部依赖
- `chat/auth`: 认证模块
- `chat/utils`: 工具函数
- `chat/globals`: 全局配置
- `chat/connection`: 连接管理

## 使用示例

### 1. 注册中间件
```go
import "chat/middleware"

func main() {
    app := gin.New()
    cleanup := middleware.RegisterMiddleware(app)
    defer cleanup()
    
    // 启动服务器
    app.Run(":8080")
}
```

### 2. 获取上下文信息
```go
func handler(c *gin.Context) {
    // 获取认证信息
    isAuth := c.GetBool("auth")
    username := c.GetString("user")
    isAdmin := c.GetBool("admin")
    
    // 获取数据库连接
    db := utils.GetDBFromContext(c)
    cache := utils.GetCacheFromContext(c)
}
```

## 总结

Middleware 模块是 ChatNio 系统的安全和性能保障层，通过四个核心中间件提供了完整的请求处理流程：

1. **CORS 中间件**: 处理跨域请求，确保前端应用正常访问
2. **内置中间件**: 提供数据库和缓存连接，支撑业务功能
3. **限流中间件**: 防止接口滥用，保护系统稳定性
4. **认证中间件**: 验证用户身份，保护敏感资源

该模块设计合理，功能完整，为整个 ChatNio 系统提供了坚实的基础设施支持。