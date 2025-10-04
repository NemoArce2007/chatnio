# ChatNio Utils 模块 API 文档

## 概述

ChatNio Utils 模块是一个综合性的工具库，提供了丰富的实用功能，包括基础工具函数、缓存管理、网络通信、加密压缩、文件处理、图像处理、邮件发送、上下文管理和分词器等功能。该模块为整个 ChatNio 系统提供了强大的底层支持。

## 模块结构

```
utils/
├── base.go          # 基础工具函数
├── buffer.go        # 缓冲区和数据流处理
├── cache.go         # Redis 缓存操作
├── char.go          # 字符串和字符处理
├── compress.go      # 文件压缩功能
├── config.go        # 配置管理
├── ctx.go           # Gin 上下文工具
├── encrypt.go       # 加密和哈希功能
├── fs.go            # 文件系统操作
├── image.go         # 图像处理
├── net.go           # HTTP 网络请求
├── smtp.go          # SMTP 邮件发送
├── sse.go           # Server-Sent Events
├── tokenizer.go     # 分词器和 Token 计算
├── websocket.go     # WebSocket 连接管理
└── templates/       # 邮件模板
    └── code.html    # 验证码邮件模板
```

## 核心功能模块

### 1. 基础工具函数 (base.go)

**文件位置**: `utils/base.go`

#### 随机数生成
- **`Intn(n int) int`** (第12行)
  - 生成 0 到 n-1 之间的随机整数
  
- **`IntnSeed(n int, seed int64) int`** (第17行)
  - 使用指定种子生成随机整数
  
- **`IntnSeq(n int, length int) []int`** (第22行)
  - 生成指定长度的随机整数序列

#### 数组操作
- **`Sum(arr []int) int`** (第30行)
  - 计算整数数组的总和
  
- **`Contains[T comparable](arr []T, item T) bool`** (第38行)
  - 检查数组是否包含指定元素
  
- **`ToPtr[T any](value T) *T`** (第46行)
  - 将值转换为指针
  
- **`TryGet[T any](arr []T, index int) *T`** (第50行)
  - 安全获取数组元素，避免越界
  
- **`Insert[T any](arr []T, index int, item T) []T`** (第57行)
  - 在指定位置插入元素
  
- **`Remove[T any](arr []T, index int) []T`** (第64行)
  - 移除指定位置的元素

#### 调试工具
- **`Debug(args ...interface{})`** (第71行)
  - 调试输出函数

### 2. 缓冲区处理 (buffer.go)

**文件位置**: `utils/buffer.go`

#### 接口定义
- **`Charge` 接口** (第10行)
  - 定义计费相关的方法：类型、模型、输入/输出成本、匿名支持、计费状态和限制

#### 核心结构
- **`Buffer` 结构体** (第20行)
  - 包含模型、配额、数据、游标、次数、历史记录、图像和工具调用等字段

#### 主要方法
- **`NewBuffer(model string, quota float32, charge Charge) *Buffer`** (第33行)
  - 创建新的缓冲区实例
  
- **`Write(data string) *Buffer`** (第45行)
  - 向缓冲区写入数据
  
- **`GetChunk() string`** (第52行)
  - 获取数据块
  
- **`AddImage(url string) *Buffer`** (第56行)
  - 添加图像到缓冲区

### 3. 字符处理 (char.go)

**文件位置**: `utils/char.go`

#### 随机生成
- **`GetRandomInt(min int, max int) int`** (第11行)
  - 生成指定范围内的随机整数
  
- **`GenerateCode(length int) string`** (第15行)
  - 生成指定长度的数字验证码
  
- **`GenerateChar(length int) string`** (第24行)
  - 生成指定长度的随机字符串

#### 时间处理
- **`ConvertTime(t []uint8) *time.Time`** (第33行)
  - 将字节数组转换为时间对象

#### JSON 处理
- **`Unmarshal[T interface{}](data []byte) (form T, err error)`** (第40行)
  - 泛型 JSON 反序列化
  
- **`Marshal[T interface{}](data T) string`** (第50行)
  - 泛型 JSON 序列化
  
- **`MarshalWithIndent[T interface{}](data T, length ...int) string`** (第57行)
  - 带缩进的 JSON 序列化

#### 字符串处理
- **`SplitItem(data string, sep string) []string`** (第117行)
  - 分割字符串并去除空白
  
- **`ExtractUrls(data string) []string`** (第162行)
  - 提取字符串中的 URL
  
- **`DecodeUnicode(data string) string`** (第180行)
  - 解码 Unicode 字符串

### 4. 缓存管理 (cache.go)

**文件位置**: `utils/cache.go`

#### 基础操作
- **`Incr(cache *redis.Client, key string, delta int64) (int64, error)`** (第10行)
  - 递增缓存值
  
- **`Decr(cache *redis.Client, key string, delta int64) (int64, error)`** (第14行)
  - 递减缓存值
  
- **`GetInt(cache *redis.Client, key string) (int64, error)`** (第18行)
  - 获取整数值
  
- **`SetInt(cache *redis.Client, key string, value int64, expiration int64) error`** (第28行)
  - 设置整数值

#### JSON 操作
- **`SetJson(cache *redis.Client, key string, value interface{}, expiration int64) error`** (第32行)
  - 设置 JSON 数据
  
- **`GetJson[T any](cache *redis.Client, key string) *T`** (第37行)
  - 获取 JSON 数据

#### 限流功能
- **`IncrWithLimit(cache *redis.Client, key string, delta int64, limit int64, expiration int64) bool`** (第49行)
  - 带限制的递增操作
  
- **`IncrIP(cache *redis.Client, ip string) int64`** (第71行)
  - IP 访问计数

### 5. 配置管理 (config.go)

**文件位置**: `utils/config.go`

#### 配置读取
- **`ReadConf()`** (第13行)
  - 读取配置文件，自动创建默认配置

#### 引擎创建
- **`NewEngine() *gin.Engine`** (第27行)
  - 创建 Gin 引擎实例
  
- **`RegisterStaticRoute(engine *gin.Engine)`** (第38行)
  - 注册静态文件路由

### 6. 网络通信 (net.go)

**文件位置**: `utils/net.go`

#### HTTP 客户端
- **`newClient() *http.Client`** (第17行)
  - 创建 HTTP 客户端

#### 通用请求
- **`Http(uri string, method string, ptr interface{}, headers map[string]string, body io.Reader) error`** (第22行)
  - 通用 HTTP 请求方法
  
- **`HttpRaw(uri string, method string, headers map[string]string, body io.Reader) ([]byte, error)`** (第41行)
  - 原始 HTTP 请求

#### 便捷方法
- **`Get(uri string, headers map[string]string) (interface{}, error)`** (第60行)
  - GET 请求
  
- **`Post(uri string, headers map[string]string, body interface{}) (interface{}, error)`** (第69行)
  - POST 请求

#### 事件流
- **`EventSource(method string, uri string, headers map[string]string, body interface{}, callback func(string) error) error`** (第117行)
  - Server-Sent Events 客户端

### 7. WebSocket 管理 (websocket.go)

**位置**: `utils/websocket.go`

#### 核心结构
- **`WebSocket` 结构体** (第12行)
  - 包含 Gin 上下文和 WebSocket 连接

#### 连接管理
- **`NewWebsocket(c *gin.Context, strict bool) *WebSocket`** (第25行)
  - 创建 WebSocket 连接
  
- **`NewWebsocketClient(url string) *WebSocket`** (第40行)
  - 创建 WebSocket 客户端

#### 消息处理
- **`Read() (int, []byte, error)`** (第48行)
  - 读取消息
  
- **`Write(messageType int, data []byte) error`** (第52行)
  - 写入消息
  
- **`SendJSON(v interface{}) error`** (第68行)
  - 发送 JSON 消息

### 8. Server-Sent Events (sse.go)

**位置**: `utils/sse.go`

#### 事件结构
- **`StreamEvent` 结构体** (第17行)
  - 定义流事件的结构

#### 事件处理
- **`NewEvent(data interface{}) StreamEvent`** (第62行)
  - 创建新事件
  
- **`NewEndEvent() StreamEvent`** (第68行)
  - 创建结束事件
  
- **`SSEClient(method string, uri string, headers map[string]string, body interface{}, callback func(string) error) error`** (第73行)
  - SSE 客户端

### 9. 加密功能 (encrypt.go)

**位置**: `utils/encrypt.go`

#### 哈希算法
- **`Sha2Encrypt(raw string) string`** (第13行)
  - SHA256 哈希加密
  
- **`Md5Encrypt(raw string) string`** (第45行)
  - MD5 哈希加密

#### Base64 编码
- **`Base64Encode(raw string) string`** (第25行)
  - Base64 编码
  
- **`Base64Decode(raw string) string`** (第33行)
  - Base64 解码

#### AES 加密
- **`AES256Encrypt(key string, data string) (string, error)`** (第53行)
  - AES256 加密
  
- **`AES256Decrypt(key string, data string) (string, error)`** (第70行)
  - AES256 解密

### 10. 压缩功能 (compress.go)

**位置**: `utils/compress.go`

#### 压缩任务
- **`GenerateCompressTask(hash string, base, path, replacer string) (string, string, error)`** (第12行)
  - 生成压缩任务，同时创建 ZIP 和 GZIP 文件
  
- **`GenerateCompressTaskAsync(hash string, base, path, replacer string) (string, string)`** (第27行)
  - 异步生成压缩任务

#### 压缩实现
- **`CreateZipObject(output string, files []string, replacer string) error`** (第47行)
  - 创建 ZIP 压缩文件
  
- **`CreateGzipObject(output string, files []string, replacer string) error`** (第109行)
  - 创建 GZIP 压缩文件

### 11. 文件系统 (fs.go)

**位置**: `utils/fs.go`

#### 目录操作
- **`CreateFolder(path string) bool`** (第10行)
  - 创建目录
  
- **`Exists(path string) bool`** (第16行)
  - 检查路径是否存在
  
- **`Walk(path string) []string`** (第43行)
  - 遍历目录获取所有文件

#### 文件操作
- **`WriteFile(path string, data string, folderSafe bool) bool`** (第32行)
  - 写入文件
  
- **`IsFileExist(path string) bool`** (第58行)
  - 检查文件是否存在
  
- **`CopyFile(src string, dst string) error`** (第63行)
  - 复制文件

### 12. 图像处理 (image.go)

**位置**: `utils/image.go`

#### 图像结构
- **`Image` 结构体** (第15行)
  - 图像对象封装

#### 图像创建
- **`NewImage(url string) (*Image, error)`** (第19行)
  - 从 URL 创建图像对象
  
- **`ConvertToBase64(url string) (string, error)`** (第45行)
  - 将图像转换为 Base64

#### 图像信息
- **`GetWidth() int`** (第58行)
  - 获取图像宽度
  
- **`GetHeight() int`** (第62行)
  - 获取图像高度
  
- **`CountTokens(model string) int`** (第74行)
  - 计算图像的 Token 数量

### 13. 邮件发送 (smtp.go)

**位置**: `utils/smtp.go`

#### SMTP 结构
- **`SmtpPoster` 结构体** (第12行)
  - SMTP 邮件发送器

#### 邮件发送
- **`NewSmtpPoster(host string, port int, username string, password string, from string) *SmtpPoster`** (第19行)
  - 创建 SMTP 发送器
  
- **`SendMail(to string, subject string, body string) error`** (第28行)
  - 发送邮件
  
- **`RenderTemplate(filename string, data interface{}) (string, error)`** (第42行)
  - 渲染邮件模板
  
- **`RenderMail(filename string, data interface{}, to string, subject string) error`** (第52行)
  - 渲染并发送邮件

### 14. 上下文工具 (ctx.go)

**位置**: `utils/ctx.go`

#### 上下文获取
- **`GetDBFromContext(c *gin.Context) *sql.DB`** (第9行)
  - 从上下文获取数据库连接
  
- **`GetCacheFromContext(c *gin.Context) *redis.Client`** (第13行)
  - 从上下文获取缓存客户端
  
- **`GetUserFromContext(c *gin.Context) string`** (第17行)
  - 从上下文获取用户信息
  
- **`GetAdminFromContext(c *gin.Context) bool`** (第21行)
  - 从上下文获取管理员状态

### 15. 分词器 (tokenizer.go)

**位置**: `utils/tokenizer.go`

#### Token 计算
- **`GetWeightByModel(model string) int`** (第11行)
  - 根据模型获取权重
  
- **`NumTokensFromMessages(messages []globals.Message, model string) int`** (第35行)
  - 计算消息的 Token 数量
  
- **`CountTokenPrice(messages []globals.Message, model string) int`** (第58行)
  - 计算 Token 价格
  
- **`CountInputToken(charge Charge, model string, message []globals.Message) float32`** (第62行)
  - 计算输入 Token 费用
  
- **`CountOutputToken(charge Charge, model string, token int) float32`** (第69行)
  - 计算输出 Token 费用

### 16. 邮件模板 (templates/code.html)

**位置**: `utils/templates/code.html`

#### 模板结构
- **验证码邮件模板** (第1-52行)
  - 包含样式定义和 HTML 结构
  - 支持动态数据：Logo、Title、Code
  - 响应式设计，适配各种邮件客户端

## 依赖关系

### 外部依赖
- **Redis**: `github.com/go-redis/redis/v8` - 缓存操作
- **Gin**: `github.com/gin-gonic/gin` - Web 框架
- **WebSocket**: `github.com/gorilla/websocket` - WebSocket 支持
- **Tiktoken**: `github.com/pkoukk/tiktoken-go` - Token 计算
- **WebP**: `github.com/chai2010/webp` - WebP 图像处理
- **JSON**: `github.com/goccy/go-json` - 高性能 JSON 处理
- **Viper**: `github.com/spf13/viper` - 配置管理

### 内部依赖
- **globals**: `chat/globals` - 全局常量和类型定义

## 使用示例

### 缓存操作
```go
// 设置缓存
err := SetInt(cache, "user:123", 100, 3600)

// 获取缓存
value, err := GetInt(cache, "user:123")

// JSON 缓存
SetJson(cache, "user:profile", userProfile, 7200)
profile := GetJson[UserProfile](cache, "user:profile")
```

### 网络请求
```go
// GET 请求
data, err := Get("https://api.example.com/data", headers)

// POST 请求
response, err := Post("https://api.example.com/submit", headers, requestBody)

// SSE 连接
err := SSEClient("GET", "https://api.example.com/stream", headers, nil, func(data string) error {
    fmt.Println("Received:", data)
    return nil
})
```

### 文件操作
```go
// 创建目录
CreateFolder("/path/to/directory")

// 写入文件
WriteFile("/path/to/file.txt", "content", true)

// 复制文件
err := CopyFile("source.txt", "destination.txt")

// 遍历目录
files := Walk("/path/to/directory")
```

### 加密操作
```go
// SHA256 哈希
hash := Sha2Encrypt("password")

// Base64 编码
encoded := Base64Encode("data")
decoded := Base64Decode(encoded)

// AES 加密
encrypted, err := AES256Encrypt("key", "plaintext")
decrypted, err := AES256Decrypt("key", encrypted)
```

### WebSocket 使用
```go
// 创建 WebSocket 连接
ws := NewWebsocket(c, false)
defer ws.DeferClose()

// 发送 JSON 消息
ws.SendJSON(map[string]interface{}{
    "type": "message",
    "data": "Hello, WebSocket!",
})

// 接收消息
var message map[string]interface{}
if ws.Receive(&message) {
    fmt.Println("Received:", message)
}
```

### 邮件发送
```go
// 创建 SMTP 发送器
smtp := NewSmtpPoster("smtp.gmail.com", 587, "username", "password", "from@example.com")

// 发送验证码邮件
err := smtp.RenderMail("code.html", map[string]string{
    "Logo":  "https://example.com/logo.png",
    "Title": "ChatNio",
    "Code":  "123456",
}, "user@example.com", "验证码")
```

## 性能特性

### 高性能设计
1. **并发安全**: 所有缓存操作都是并发安全的
2. **连接池**: HTTP 客户端使用连接池提高性能
3. **异步处理**: 支持异步压缩任务
4. **内存优化**: 使用泛型减少内存分配

### 错误处理
1. **优雅降级**: 网络请求失败时提供默认值
2. **资源清理**: 自动关闭文件句柄和网络连接
3. **超时控制**: 所有网络操作都有超时限制

### 安全特性
1. **输入验证**: 所有用户输入都经过验证
2. **加密支持**: 提供多种加密算法
3. **安全传输**: 支持 TLS 加密传输

## 总结

ChatNio Utils 模块是一个功能完整、设计良好的工具库，为整个系统提供了：

- **15个核心文件**，涵盖各种实用功能
- **100+个工具函数**，满足不同场景需求
- **完整的错误处理**和资源管理
- **高性能设计**和并发安全保证
- **丰富的网络通信**支持（HTTP、WebSocket、SSE）
- **全面的数据处理**能力（JSON、加密、压缩）
- **便捷的文件操作**和图像处理
- **专业的邮件发送**和模板渲染

该模块的设计遵循了 Go 语言的最佳实践，提供了清晰的 API 接口和完善的功能实现，是 ChatNio 系统的重要基础设施。