# ChatNio Globals 模块 API 文档

## 概述

`globals` 模块是 ChatNio 系统的全局定义模块，提供了系统范围内使用的常量、类型、接口、变量和工具函数。该模块为整个系统提供了统一的数据结构定义、配置常量和通用工具函数。

## 模块结构

```
globals/
├── constant.go     # 系统常量定义
├── types.go        # 全局数据类型定义
├── interface.go    # 接口定义
├── variables.go    # 全局变量和模型常量
├── tools.go        # 工具调用相关类型
├── logger.go       # 日志系统
└── usage.go        # 使用量相关工具
```

## 核心组件

### 1. 系统常量 (constant.go)
> **源代码位置**: <mcfile name="constant.go" path="globals/constant.go"></mcfile>

#### 消息角色常量
```go
const (
    System    = "system"      // 系统消息
    User      = "user"        // 用户消息
    Assistant = "assistant"   // 助手消息
    Tool      = "tool"        // 工具消息
)
```

#### 渠道类型常量
```go
const (
    OpenAIChannelType      = "openai"      // OpenAI 渠道
    AzureOpenAIChannelType = "azure"       // Azure OpenAI 渠道
    ClaudeChannelType      = "claude"      // Claude 渠道
    SlackChannelType       = "slack"       // Slack 渠道
    SparkdeskChannelType   = "sparkdesk"   // 讯飞星火渠道
    ChatGLMChannelType     = "chatglm"     // ChatGLM 渠道
    HunyuanChannelType     = "hunyuan"     // 腾讯混元渠道
    QwenChannelType        = "qwen"        // 阿里通义千问渠道
    ZhinaoChannelType      = "zhinao"      // 360 智脑渠道
    BaichuanChannelType    = "baichuan"    // 百川渠道
    SkylarkChannelType     = "skylark"     // 字节云雀渠道
    BingChannelType        = "bing"        // Bing 渠道
    PalmChannelType        = "palm"        // Google PaLM 渠道
    MidjourneyChannelType  = "midjourney"  // Midjourney 渠道
    OneAPIChannelType      = "oneapi"      // OneAPI 渠道
)
```

#### 计费类型常量
```go
const (
    NonBilling   = "non-billing"   // 不计费
    TimesBilling = "times-billing" // 按次计费
    TokenBilling = "token-billing" // 按 Token 计费
)
```

#### 用户类型常量
```go
const (
    AnonymousType = "anonymous" // 匿名用户
    NormalType    = "normal"    // 普通用户
    BasicType     = "basic"     // 基础订阅用户
    StandardType  = "standard"  // 标准订阅用户
    ProType       = "pro"       // 专业订阅用户
)
```

### 2. 全局数据类型 (types.go)
> **源代码位置**: <mcfile name="types.go" path="globals/types.go"></mcfile>

#### Hook 函数类型
```go
type Hook func(data string) error
```
- **用途**: 定义钩子函数类型，用于事件处理

#### 消息结构体
```go
type Message struct {
    Role       string     `json:"role"`                   // 消息角色
    Content    string     `json:"content"`                // 消息内容
    ToolCallId *string    `json:"tool_call_id,omitempty"` // 工具调用ID（仅tool角色）
    ToolCalls  *ToolCalls `json:"tool_calls,omitempty"`   // 工具调用（仅assistant角色）
}
```

#### 聊天响应结构体
```go
type ChatSegmentResponse struct {
    Conversation int64   `json:"conversation"` // 对话ID
    Quota        float32 `json:"quota"`        // 配额消耗
    Keyword      string  `json:"keyword"`      // 关键词
    Message      string  `json:"message"`      // 消息内容
    End          bool    `json:"end"`          // 是否结束
    Plan         bool    `json:"plan"`         // 是否为计划
}
```

#### 生成响应结构体
```go
type GenerationSegmentResponse struct {
    Quota   float32 `json:"quota"`   // 配额消耗
    Message string  `json:"message"` // 消息内容
    Hash    string  `json:"hash"`    // 哈希值
    End     bool    `json:"end"`     // 是否结束
    Error   string  `json:"error"`   // 错误信息
}
```

### 3. 接口定义 (interface.go)
> **源代码位置**: <mcfile name="interface.go" path="globals/interface.go"></mcfile>

#### 渠道配置接口
```go
type ChannelConfig interface {
    GetType() string                        // 获取渠道类型
    GetModelReflect(model string) string    // 获取模型映射
    GetRetry() int                          // 获取重试次数
    GetRandomSecret() string                // 获取随机密钥
    SplitRandomSecret(num int) []string     // 分割随机密钥
    GetEndpoint() string                    // 获取端点地址
    ProcessError(err error) error           // 处理错误
}
```

### 4. 全局变量和模型常量 (variables.go)
> **源代码位置**: <mcfile name="variables.go" path="globals/variables.go"></mcfile>

#### 线程限制常量
```go
const ChatMaxThread = 5        // 最大聊天线程数
const AnonymousMaxThread = 1   // 匿名用户最大线程数
```

#### 允许的来源域名
```go
var AllowedOrigins = []string{
    "chatnio.net",
    "nextweb.chatnio.net",
    "fystart.cn",
    "fystart.com",
}
```

#### 来源验证函数
```go
func OriginIsAllowed(uri string) bool
```
- **功能**: 验证请求来源是否被允许
- **参数**: `uri` - 请求URI
- **返回**: 是否允许的布尔值

```go
func OriginIsOpen(c *gin.Context) bool
```
- **功能**: 检查请求路径是否为开放路径
- **参数**: `c` - Gin上下文
- **返回**: 是否开放的布尔值

#### 模型常量定义

##### GPT 系列模型
```go
const (
    GPT3Turbo             = "gpt-3.5-turbo"
    GPT3TurboInstruct     = "gpt-3.5-turbo-instruct"
    GPT3Turbo0613         = "gpt-3.5-turbo-0613"
    GPT3Turbo0301         = "gpt-3.5-turbo-0301"
    GPT3Turbo1106         = "gpt-3.5-turbo-1106"
    GPT3Turbo16k          = "gpt-3.5-turbo-16k"
    GPT3Turbo16k0613      = "gpt-3.5-turbo-16k-0613"
    GPT3Turbo16k0301      = "gpt-3.5-turbo-16k-0301"
    GPT4                  = "gpt-4"
    GPT4All               = "gpt-4-all"
    GPT4Vision            = "gpt-4-v"
    GPT4Dalle             = "gpt-4-dalle"
    GPT40314              = "gpt-4-0314"
    GPT40613              = "gpt-4-0613"
    GPT41106Preview       = "gpt-4-1106-preview"
    GPT41106VisionPreview = "gpt-4-vision-preview"
    GPT432k               = "gpt-4-32k"
    GPT432k0314           = "gpt-4-32k-0314"
    GPT432k0613           = "gpt-4-32k-0613"
)
```

##### DALL-E 系列模型
```go
const (
    Dalle  = "dalle"
    Dalle2 = "dall-e-2"
    Dalle3 = "dall-e-3"
)
```

##### Claude 系列模型
```go
const (
    Claude1       = "claude-1"
    Claude1100k   = "claude-1.3"
    Claude2       = "claude-1-100k"
    Claude2100k   = "claude-2"
    Claude2200k   = "claude-2.1"
    ClaudeSlack   = "claude-slack"
)
```

##### 其他模型
```go
const (
    SparkDesk         = "spark-desk-v1.5"
    SparkDeskV2       = "spark-desk-v2"
    SparkDeskV3       = "spark-desk-v3"
    ChatBison001      = "chat-bison-001"
    GeminiPro         = "gemini-pro"
    GeminiProVision   = "gemini-pro-vision"
    BingCreative      = "bing-creative"
    BingBalanced      = "bing-balanced"
    BingPrecise       = "bing-precise"
    ZhiPuChatGLMTurbo = "zhipu-chatglm-turbo"
    ZhiPuChatGLMPro   = "zhipu-chatglm-pro"
    ZhiPuChatGLMStd   = "zhipu-chatglm-std"
    ZhiPuChatGLMLite  = "zhipu-chatglm-lite"
    QwenTurbo         = "qwen-turbo"
    QwenPlus          = "qwen-plus"
    QwenTurboNet      = "qwen-turbo-net"
    QwenPlusNet       = "qwen-plus-net"
    Midjourney        = "midjourney"
    MidjourneyFast    = "midjourney-fast"
    MidjourneyTurbo   = "midjourney-turbo"
    Hunyuan           = "hunyuan"
    GPT360V9          = "360-gpt-v9"
    Baichuan53B       = "baichuan-53b"
    SkylarkLite       = "skylark-lite-public"
    SkylarkPlus       = "skylark-plus-public"
    SkylarkPro        = "skylark-pro-public"
    SkylarkChat       = "skylark-chat"
)
```

#### 模型检测函数

```go
func IsGPT4NativeModel(model string) bool
```
- **功能**: 检查是否为 GPT-4 原生模型
- **参数**: `model` - 模型名称
- **返回**: 是否为 GPT-4 模型的布尔值

```go
func IsDalleModel(model string) bool
```
- **功能**: 检查是否为 DALL-E 模型
- **参数**: `model` - 模型名称
- **返回**: 是否为 DALL-E 模型的布尔值

```go
func IsClaude100KModel(model string) bool
```
- **功能**: 检查是否为 Claude 100K 上下文模型
- **参数**: `model` - 模型名称
- **返回**: 是否为 Claude 100K 模型的布尔值

```go
func IsMidjourneyFastModel(model string) bool
```
- **功能**: 检查是否为 Midjourney Fast 模型
- **参数**: `model` - 模型名称
- **返回**: 是否为 Midjourney Fast 模型的布尔值

```go
func IsGPT41106VisionPreview(model string) bool
```
- **功能**: 检查是否为 GPT-4 Vision Preview 模型
- **参数**: `model` - 模型名称
- **返回**: 是否为 GPT-4 Vision Preview 模型的布尔值

### 5. 工具调用类型 (tools.go)
> **源代码位置**: <mcfile name="tools.go" path="globals/tools.go"></mcfile>

#### 工具调用ID类型
```go
type ToolCallId string                    // 工具调用ID类型
type FunctionTools []ToolObject           // 函数工具数组

type ToolObject struct {
    Type     string       `json:"type"`     // 工具类型
    Function ToolFunction `json:"function"` // 工具函数
}

type ToolFunction struct {
    Name        string         `json:"name"`        // 函数名称
    Description string         `json:"description"` // 函数描述
    Parameters  ToolParameters `json:"parameters"`  // 函数参数
}

type ToolParameters struct {
    Type       string         `json:"type"`       // 参数类型
    Properties ToolProperties `json:"properties"` // 参数属性
    Required   []string       `json:"required"`   // 必需参数
}

type ToolProperties map[string]ToolProperty // 工具属性映射

type ToolProperty struct {
    Type        string    `json:"type"`                 // 属性类型
    Description string    `json:"description"`          // 属性描述
    Enum        *[]string `json:"enum,omitempty"`       // 枚举值
}

type ToolCallFunction struct {
    Name      string `json:"name"`      // 函数名称
    Arguments string `json:"arguments"` // 函数参数
}

type ToolCall struct {
    Type     string           `json:"type"`     // 调用类型
    Id       ToolCallId       `json:"id"`       // 调用ID
    Function ToolCallFunction `json:"function"` // 调用函数
}

type ToolCalls []ToolCall // 工具调用数组
```

### 6. 日志系统 (logger.go)
> **源代码位置**: <mcfile name="logger.go" path="globals/logger.go"></mcfile>

#### 全局日志变量
```go
var Logger *logrus.Logger // 全局日志实例
```

#### 自定义日志格式器
```go
type AppLogger struct {
    *logrus.Logger
}

func (l *AppLogger) Format(entry *logrus.Entry) ([]byte, error)
```
- **功能**: 自定义日志格式化器
- **格式**: `[LEVEL] - [TIMESTAMP] - MESSAGE`

#### 日志配置
- **日志文件**: `logs/chat.log`
- **最大文件大小**: 1MB
- **最大备份数**: 500
- **最大保存天数**: 1天
- **日志级别**: Debug

#### 日志函数

```go
func Output(args ...interface{})  // 输出日志
func Debug(args ...interface{})   // 调试日志
func Info(args ...interface{})    // 信息日志
func Warn(args ...interface{})    // 警告日志
func Error(args ...interface{})   // 错误日志
func Fatal(args ...interface{})   // 致命错误日志
func Panic(args ...interface{})   // 恐慌日志
```

### 7. 使用量工具 (usage.go)
> **源代码位置**: <mcfile name="usage.go" path="globals/usage.go"></mcfile>

```go
func GetSubscriptionLimitFormat(t string, id int64) string
```
- **功能**: 生成订阅限制格式字符串
- **参数**: 
  - `t` - 类型字符串
  - `id` - 用户ID
- **返回**: 格式化的限制字符串 `"usage-{type}:{id}"`

## 使用示例

### 1. 消息创建示例
```go
import "chatnio/globals"

// 创建用户消息
userMessage := globals.Message{
    Role:    globals.User,
    Content: "Hello, how are you?",
}

// 创建助手消息
assistantMessage := globals.Message{
    Role:    globals.Assistant,
    Content: "I'm doing well, thank you!",
}
```

### 2. 模型检测示例
```go
import "chatnio/globals"

model := "gpt-4"
if globals.IsGPT4NativeModel(model) {
    fmt.Println("This is a GPT-4 model")
}

dalleModel := "dall-e-3"
if globals.IsDalleModel(dalleModel) {
    fmt.Println("This is a DALL-E model")
}
```

### 3. 来源验证示例
```go
import "chatnio/globals"

uri := "https://chatnio.net"
if globals.OriginIsAllowed(uri) {
    fmt.Println("Origin is allowed")
}
```

### 4. 日志使用示例
```go
import "chatnio/globals"

// 记录不同级别的日志
globals.Info("Application started")
globals.Debug("Debug information")
globals.Warn("Warning message")
globals.Error("Error occurred")
```

### 5. 工具调用示例
```go
import "chatnio/globals"

// 创建工具函数定义
toolFunction := globals.ToolFunction{
    Name:        "get_weather",
    Description: "Get current weather information",
    Parameters: globals.ToolParameters{
        Type: "object",
        Properties: globals.ToolProperties{
            "location": globals.ToolProperty{
                Type:        "string",
                Description: "The city name",
            },
        },
        Required: []string{"location"},
    },
}
```

## 配置说明

### 1. 日志配置
- 日志文件自动轮转，避免文件过大
- 支持多级别日志记录
- 自定义时间戳格式

### 2. 来源控制
- 支持多域名白名单
- 本地开发环境自动允许
- 特定路径开放访问

### 3. 模型支持
- 支持多种 AI 模型提供商
- 统一的模型标识符
- 模型特性检测函数

## 最佳实践

### 1. 常量使用
- 使用预定义常量而非硬编码字符串
- 保持常量命名的一致性
- 按功能分组组织常量

### 2. 类型安全
- 使用强类型定义避免类型错误
- 合理使用指针类型处理可选字段
- 遵循 JSON 标签规范

### 3. 日志记录
- 根据重要性选择合适的日志级别
- 避免在循环中记录过多日志
- 包含足够的上下文信息

### 4. 错误处理
- 使用统一的错误处理模式
- 记录详细的错误信息
- 区分不同类型的错误

## 扩展指南

### 1. 添加新常量
```go
// 在 constant.go 中添加新的常量组
const (
    NewFeatureType1 = "type1"
    NewFeatureType2 = "type2"
)
```

### 2. 添加新类型
```go
// 在 types.go 中添加新的数据结构
type NewDataType struct {
    Field1 string `json:"field1"`
    Field2 int    `json:"field2"`
}
```

### 3. 扩展接口
```go
// 在 interface.go 中扩展现有接口
type ExtendedChannelConfig interface {
    ChannelConfig
    NewMethod() string
}
```

### 4. 添加工具函数
```go
// 在相应文件中添加新的工具函数
func NewUtilityFunction(param string) bool {
    // 实现逻辑
    return true
}
```

## 注意事项

1. **版本兼容性**: 修改现有类型定义时要考虑向后兼容性
2. **性能考虑**: 避免在全局变量中存储大量数据
3. **线程安全**: 全局变量的并发访问需要适当的同步机制
4. **内存管理**: 合理使用指针类型，避免内存泄漏
5. **文档维护**: 及时更新文档以反映代码变更

## 总结

`globals` 模块为 ChatNio 系统提供了完整的全局定义基础设施，包括：

- **7个核心文件**: 涵盖常量、类型、接口、变量、工具、日志和使用量管理
- **统一的数据结构**: 为整个系统提供一致的数据类型定义
- **完善的日志系统**: 支持多级别日志记录和自动轮转
- **灵活的工具调用**: 支持复杂的 AI 工具调用场景
- **全面的模型支持**: 涵盖主流 AI 模型提供商

该模块是 ChatNio 系统的基础组件，为其他模块提供了稳定可靠的全局定义和工具函数支持。