# Connection 模块文档

## 概述

Connection 模块是 ChatNio 系统的数据库和缓存连接管理核心，负责 MySQL 数据库和 Redis 缓存的连接建立、维护、监控和自动重连。该模块提供了稳定可靠的数据存储基础设施，确保系统的高可用性和数据一致性。

## 模块结构

```
connection/
├── cache.go      # Redis缓存连接管理
├── database.go   # MySQL数据库连接管理
└── worker.go     # 连接监控和自动重连
```

## 核心组件

### 1. Redis 缓存管理 (cache.go)

**位置**: `connection/cache.go`

#### 全局变量
```go
var Cache *redis.Client
```
**功能**: 全局Redis客户端实例

#### 核心函数

##### InitRedisSafe
```go
func InitRedisSafe() *redis.Client
```
**功能**: 安全初始化Redis连接，启动监控进程
**返回值**: Redis客户端实例
**特性**:
- 建立Redis连接
- 启动自动重连监控
- 返回全局缓存实例

##### ConnectRedis
```go
func ConnectRedis() *redis.Client
```
**功能**: 建立Redis连接
**返回值**: Redis客户端实例

**配置参数**:
- `redis.host` - Redis服务器地址
- `redis.port` - Redis服务器端口
- `redis.password` - Redis密码
- `redis.db` - Redis数据库编号

**特性**:
- 自动连接测试
- 连接失败日志记录
- 调试模式下自动清空缓存
- 连接成功确认日志

**使用示例**:
```go
// 初始化Redis连接
cache := connection.InitRedisSafe()

// 直接连接Redis
client := connection.ConnectRedis()
```

### 2. MySQL 数据库管理 (database.go)

**位置**: `connection/database.go`

#### 全局变量
```go
var DB *sql.DB
```
**功能**: 全局MySQL数据库连接实例

#### 核心函数

##### InitMySQLSafe
```go
func InitMySQLSafe() *sql.DB
```
**功能**: 安全初始化MySQL连接，启动监控进程
**返回值**: 数据库连接实例
**特性**:
- 建立MySQL连接
- 启动自动重连监控
- 返回全局数据库实例

##### ConnectMySQL
```go
func ConnectMySQL() *sql.DB
```
**功能**: 建立MySQL数据库连接
**返回值**: 数据库连接实例

**配置参数**:
- `mysql.user` - 数据库用户名
- `mysql.password` - 数据库密码
- `mysql.host` - 数据库服务器地址
- `mysql.port` - 数据库服务器端口
- `mysql.db` - 数据库名称

**特性**:
- 自动重连机制
- 连接失败重试（5秒间隔）
- 自动创建数据表结构
- 初始化root用户

#### 数据表管理

##### InitRootUser
```go
func InitRootUser(db *sql.DB)
```
**功能**: 初始化系统root用户
**默认配置**:
- 用户名: `root`
- 密码: `chatnio123456`
- 邮箱: `root@example.com`
- 管理员权限: `true`

##### 数据表创建函数

###### CreateUserTable
```go
func CreateUserTable(db *sql.DB)
```
**功能**: 创建用户认证表 `auth`
**字段**:
- `id` - 主键，自增
- `bind_id` - 绑定ID，唯一
- `username` - 用户名，唯一，最大24字符
- `token` - 访问令牌，255字符
- `email` - 邮箱，唯一
- `password` - 密码哈希，64字符
- `is_admin` - 管理员标识，默认false
- `is_banned` - 封禁标识，默认false

###### CreateConversationTable
```go
func CreateConversationTable(db *sql.DB)
```
**功能**: 创建对话表 `conversation`
**字段**:
- `id` - 主键，自增
- `user_id` - 用户ID，外键
- `conversation_id` - 对话ID
- `conversation_name` - 对话名称
- `data` - 对话数据，中等文本
- `model` - 模型名称，默认'gpt-3.5-turbo-0613'
- `updated_at` - 更新时间

###### CreatePackageTable
```go
func CreatePackageTable(db *sql.DB)
```
**功能**: 创建套餐表 `package`
**字段**:
- `id` - 主键，自增
- `user_id` - 用户ID，外键
- `type` - 套餐类型
- `created_at` - 创建时间

###### CreateQuotaTable
```go
func CreateQuotaTable(db *sql.DB)
```
**功能**: 创建配额表 `quota`
**字段**:
- `id` - 主键，自增
- `user_id` - 用户ID，外键，唯一
- `quota` - 总配额，16位精度，4位小数
- `used` - 已使用配额
- `created_at` - 创建时间
- `updated_at` - 更新时间

###### CreateSharingTable
```go
func CreateSharingTable(db *sql.DB)
```
**功能**: 创建分享表 `sharing`
**字段**:
- `id` - 主键，自增
- `hash` - 分享哈希，32字符，唯一
- `user_id` - 用户ID，外键
- `conversation_id` - 对话ID
- `refs` - 消息引用，文本类型
- `updated_at` - 更新时间

###### CreateSubscriptionTable
```go
func CreateSubscriptionTable(db *sql.DB)
```
**功能**: 创建订阅表 `subscription`
**字段**:
- `id` - 主键，自增
- `level` - 订阅级别，默认1
- `user_id` - 用户ID，外键，唯一
- `expired_at` - 过期时间
- `created_at` - 创建时间
- `updated_at` - 更新时间
- `total_month` - 总月数，默认0
- `enterprise` - 企业版标识，默认false

###### CreateApiKeyTable
```go
func CreateApiKeyTable(db *sql.DB)
```
**功能**: 创建API密钥表 `apikey`
**字段**:
- `id` - 主键，自增
- `user_id` - 用户ID，外键，唯一
- `api_key` - API密钥，唯一
- `created_at` - 创建时间

###### CreateInvitationTable
```go
func CreateInvitationTable(db *sql.DB)
```
**功能**: 创建邀请码表 `invitation`
**字段**:
- `id` - 主键，自增
- `code` - 邀请码，唯一
- `quota` - 配额，12位精度，4位小数
- `type` - 邀请码类型
- `used` - 使用状态，默认false
- `used_id` - 使用者ID，外键
- `created_at` - 创建时间
- `updated_at` - 更新时间

###### CreateRedeemTable
```go
func CreateRedeemTable(db *sql.DB)
```
**功能**: 创建兑换码表 `redeem`
**字段**:
- `id` - 主键，自增
- `code` - 兑换码，唯一
- `quota` - 配额，12位精度，4位小数
- `used` - 使用状态，默认false
- `created_at` - 创建时间
- `updated_at` - 更新时间

###### CreateBroadcastTable
```go
func CreateBroadcastTable(db *sql.DB)
```
**功能**: 创建广播表 `broadcast`
**字段**:
- `id` - 主键，自增
- `poster_id` - 发布者ID，外键
- `content` - 广播内容，文本类型
- `created_at` - 创建时间

### 3. 连接监控和自动重连 (worker.go)

**位置**: `c:\Users\nemoa\Desktop\chatnio文档\chatnio\connection\worker.go`

#### 全局变量
```go
var tick time.Duration = 5 * time.Second
```
**功能**: 监控检查间隔，每5秒检查一次

#### 核心函数

##### MysqlWorker
```go
func MysqlWorker(db *sql.DB)
```
**功能**: MySQL连接监控进程
**特性**:
- 后台goroutine运行
- 每5秒检查连接状态
- 连接失败时自动重连
- 持续监控，确保连接可用

##### RedisWorker
```go
func RedisWorker(cache *redis.Client)
```
**功能**: Redis连接监控进程
**特性**:
- 后台goroutine运行
- 每5秒检查连接状态
- 连接失败时自动重连
- 持续监控，确保连接可用

##### pingRedis
```go
func pingRedis(client *redis.Client) error
```
**功能**: Redis连接测试
**参数**: Redis客户端实例
**返回值**: 错误信息（nil表示连接正常）

## 配置管理

### Redis 配置
```yaml
redis:
  host: "localhost"      # Redis服务器地址
  port: 6379            # Redis服务器端口
  password: ""          # Redis密码
  db: 0                 # Redis数据库编号
```

### MySQL 配置
```yaml
mysql:
  user: "root"          # 数据库用户名
  password: "password"  # 数据库密码
  host: "localhost"     # 数据库服务器地址
  port: 3306           # 数据库服务器端口
  db: "chatnio"        # 数据库名称
```

### 调试配置
```yaml
debug: false           # 调试模式，true时会清空Redis缓存
```

## 错误处理和日志

### 连接失败处理
- **MySQL**: 连接失败时自动重试，5秒间隔
- **Redis**: 连接失败时记录日志，继续尝试连接
- **监控进程**: 检测到连接断开时自动重新建立连接

### 日志格式
```
[connection] connected to mysql server (host: localhost)
[connection] connected to redis (host: localhost)
[connection] failed to connect to redis host: localhost (message: connection refused), will retry in 5 seconds
[connection] flush redis cache (host: localhost)
```

## 使用示例

### 1. 初始化数据库连接
```go
package main

import "chat/connection"

func main() {
    // 安全初始化MySQL连接（推荐）
    db := connection.InitMySQLSafe()
    
    // 直接连接MySQL
    db := connection.ConnectMySQL()
    
    // 使用全局数据库实例
    connection.DB.Query("SELECT * FROM auth")
}
```

### 2. 初始化Redis连接
```go
package main

import "chat/connection"

func main() {
    // 安全初始化Redis连接（推荐）
    cache := connection.InitRedisSafe()
    
    // 直接连接Redis
    cache := connection.ConnectRedis()
    
    // 使用全局缓存实例
    connection.Cache.Set(ctx, "key", "value", 0)
}
```

### 3. 数据库操作示例
```go
// 查询用户
var user User
err := connection.DB.QueryRow("SELECT id, username FROM auth WHERE id = ?", userID).Scan(&user.ID, &user.Username)

// 插入数据
_, err := connection.DB.Exec("INSERT INTO auth (username, password, email) VALUES (?, ?, ?)", username, password, email)

// 更新数据
_, err := connection.DB.Exec("UPDATE quota SET used = used + ? WHERE user_id = ?", amount, userID)
```

### 4. Redis操作示例
```go
import "context"

ctx := context.Background()

// 设置缓存
err := connection.Cache.Set(ctx, "user:1", userData, time.Hour).Err()

// 获取缓存
val, err := connection.Cache.Get(ctx, "user:1").Result()

// 删除缓存
err := connection.Cache.Del(ctx, "user:1").Err()
```

## 数据库架构

### 核心表关系
```
auth (用户表)
├── quota (配额表) - 一对一关系
├── subscription (订阅表) - 一对一关系
├── apikey (API密钥表) - 一对一关系
├── package (套餐表) - 一对多关系
├── conversation (对话表) - 一对多关系
├── sharing (分享表) - 一对多关系
├── invitation (邀请码表) - 一对多关系（使用者）
└── broadcast (广播表) - 一对多关系（发布者）

redeem (兑换码表) - 独立表
```

### 索引策略
- **主键索引**: 所有表的 `id` 字段
- **唯一索引**: `username`, `email`, `token`, `api_key`, `code`, `hash` 等
- **外键索引**: 所有 `user_id` 字段
- **复合索引**: `(user_id, conversation_id)`, `(user_id, type)`, `(used_id, type)` 等

## 性能优化

### 连接池管理
- MySQL连接自动管理，支持连接复用
- Redis连接单例模式，减少连接开销

### 监控策略
- 5秒间隔健康检查
- 自动重连机制
- 连接状态日志记录

### 缓存策略
- Redis用于会话存储
- 调试模式下自动清空缓存
- 支持多数据库分离

## 安全考虑

### 数据库安全
- 密码使用SHA2加密存储
- 参数化查询防止SQL注入
- 外键约束保证数据完整性

### 连接安全
- 支持Redis密码认证
- MySQL连接加密支持
- 敏感信息配置文件管理

## 扩展指南

### 添加新数据表
1. 在 `database.go` 中添加 `CreateXXXTable` 函数
2. 在 `ConnectMySQL` 函数中调用新的创建函数
3. 确保外键关系正确设置
4. 添加必要的索引

### 自定义连接配置
```go
// 自定义Redis配置
func ConnectCustomRedis(addr, password string, db int) *redis.Client {
    return redis.NewClient(&redis.Options{
        Addr:     addr,
        Password: password,
        DB:       db,
    })
}
```

### 监控扩展
```go
// 自定义监控间隔
func SetMonitorInterval(duration time.Duration) {
    tick = duration
}
```

## 故障排除

### 常见问题

1. **MySQL连接失败**
   - 检查配置文件中的连接参数
   - 确认MySQL服务运行状态
   - 验证用户权限和密码

2. **Redis连接失败**
   - 检查Redis服务状态
   - 验证端口和密码配置
   - 确认防火墙设置

3. **数据表创建失败**
   - 检查数据库用户权限
   - 验证数据库存在
   - 查看详细错误日志

### 调试技巧
- 启用 `debug: true` 查看详细日志
- 使用 `connection.DB.Ping()` 测试数据库连接
- 使用 `connection.Cache.Ping()` 测试Redis连接