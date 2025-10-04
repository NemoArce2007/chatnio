# CLI 模块文档

## 概述

CLI 模块是 ChatNio 系统的命令行接口，提供了系统管理、用户管理、邀请码生成、令牌管理等核心功能的命令行操作。该模块通过简洁的命令行界面，为系统管理员提供了便捷的系统维护工具。

## 模块结构

```
cli/
├── exec.go      # 命令执行框架
├── parser.go    # 命令行参数解析
├── admin.go     # 管理员相关命令
├── invite.go    # 邀请码管理命令
├── token.go     # 令牌管理命令
├── filter.go    # API密钥过滤命令
└── help.go      # 帮助信息
```

## 核心组件

### 1. 命令执行框架 (exec.go)

**位置**: `cli/exec.go`

#### Run 函数
```go
func Run(args []string)
```

**功能**: 主命令分发器，根据第一个参数将命令分发到相应的处理函数

**支持的命令**:
- `help` - 显示帮助信息
- `invite` - 邀请码管理
- `filter` - API密钥过滤
- `token` - 令牌生成
- `root` - 更新root密码

### 2. 参数解析器 (parser.go)

**位置**: `cli/parser.go`

#### 核心函数

##### GetArgs
```go
func GetArgs() []string
```
**功能**: 获取所有命令行参数

##### GetArg
```go
func GetArg(index int) string
```
**功能**: 获取指定索引的参数

##### 类型转换函数
```go
func GetArgInt(args []string, index int) int
func GetArgInt64(args []string, index int) int64
func GetArgFloat(args []string, index int) float64
func GetArgFloat32(args []string, index int) float32
func GetArgFloat64(args []string, index int) float64
func GetArgBool(args []string, index int) bool
func GetArgString(args []string, index int) string
```

##### 输出函数
```go
func outputError(err error)
func outputInfo(prefix, message string)
```

## 命令详解

### 1. 管理员命令 (admin.go)

**位置**: `cli/admin.go`

#### UpdateRootCommand
```go
func UpdateRootCommand(args []string)
```

**功能**: 更新系统root用户密码

**参数**:
- `args[0]` (string): 新密码

**使用示例**:
```bash
./chatnio root "new_password_123"
```

**依赖**:
- `chat/admin.UpdateRootPassword()`
- MySQL数据库连接
- Redis缓存连接

### 2. 邀请码管理 (invite.go)

**位置**: `cli/invite.go`

#### CreateInvitationCommand
```go
func CreateInvitationCommand(args []string)
```

**功能**: 批量生成邀请码

**参数**:
- `args[0]` (string): 邀请码类型
- `args[1]` (int): 生成数量
- `args[2]` (float32): 配额限制

**使用示例**:
```bash
./chatnio invite "premium" 10 100.0
```

**输出**: 生成的邀请码列表（每行一个）

**依赖**:
- `chat/auth.GenerateInvitations()`
- MySQL数据库连接

### 3. 令牌管理 (token.go)

**位置**: `cli/token.go`

#### CreateTokenCommand
```go
func CreateTokenCommand(args []string)
```

**功能**: 为指定用户生成访问令牌

**参数**:
- `args[0]` (string): 用户ID

**使用示例**:
```bash
./chatnio token "12345"
```

**输出**: 生成的访问令牌

**依赖**:
- `chat/auth.GetUserById()`
- `user.GenerateTokenSafe()`
- MySQL数据库连接

### 4. API密钥过滤 (filter.go)

**位置**: `cli/filter.go`

#### FilterApiKeyCommand
```go
func FilterApiKeyCommand(args []string)
```

**功能**: 过滤和验证OpenAI API密钥的可用性

**参数**:
- `args[0]` (string): 用管道符(|)分隔的API密钥列表

**使用示例**:
```bash
./chatnio filter "sk-key1|sk-key2|sk-key3"
```

**输出**: 
- 统计信息：总数、可用数、不可用数
- 可用密钥列表（用管道符分隔）

**依赖**:
- `chat/adapter/chatgpt.FilterKeysNative()`
- OpenAI API端点: `https://api.openai.com`

### 5. 帮助系统 (help.go)

**位置**: `cli/help.go`

#### Help
```go
func Help()
```

**功能**: 显示所有可用命令的帮助信息

**命令格式**:
```
Commands:
	- help
	- invite <type> <num> <quota>
	- token <user-id>
	- root <password>
```

**使用示例**:
```bash
./chatnio help
```

## 错误处理

### 错误输出格式
所有命令都使用统一的错误处理机制：

```go
func outputError(err error)
func outputInfo(prefix, message string)
```

**错误输出示例**:
```
Error: invalid arguments, please provide a new root password
```

**信息输出示例**:
```
[root] root password updated
[invite] 10 invitation codes generated
[token] token generated
[filter] filtered 3 keys, 2 available, 1 unavailable
```

## 数据库依赖

### MySQL连接
```go
db := connection.ConnectMySQL()
```

**用途**:
- 用户管理
- 邀请码存储
- 令牌生成
- 系统配置

### Redis连接
```go
cache := connection.ConnectRedis()
```

**用途**:
- 缓存管理
- 会话存储

## 使用示例

### 1. 系统初始化
```bash
# 设置root密码
./chatnio root "admin123"
```

### 2. 用户管理
```bash
# 生成邀请码
./chatnio invite "standard" 5 50.0

# 为用户生成令牌
./chatnio token "1001"
```

### 3. API密钥管理
```bash
# 过滤可用的API密钥
./chatnio filter "sk-key1|sk-key2|sk-key3"
```

### 4. 获取帮助
```bash
# 查看所有可用命令
./chatnio help
```

## 最佳实践

### 1. 参数验证
- 所有命令都会验证必需参数
- 类型转换失败时会使用默认值
- 建议在脚本中添加参数检查

### 2. 错误处理
- 监控命令输出中的错误信息
- 数据库连接失败时会显示相应错误
- 建议在自动化脚本中检查返回状态

### 3. 安全考虑
- root密码应使用强密码
- API密钥应妥善保管
- 令牌生成后应及时使用

### 4. 性能优化
- 批量操作时合理设置数量
- 避免频繁的数据库连接
- 大量API密钥过滤时分批处理

## 扩展指南

### 添加新命令
1. 在相应的功能文件中实现命令函数
2. 在 `exec.go` 的 `Run` 函数中添加命令分发
3. 在 `help.go` 中更新帮助信息
4. 确保参数解析和错误处理的一致性

### 参数类型扩展
可以在 `parser.go` 中添加新的类型转换函数，遵循现有的命名约定：
```go
func GetArgCustomType(args []string, index int) CustomType
```