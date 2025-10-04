# Addition 模块 API 文档

Addition 模块是 ChatNio 的扩展功能模块，提供了四个核心功能：卡片生成、代码项目生成、文章生成和网络搜索增强。本文档详细介绍各个 API 端点的参数、响应格式和使用方法。

## 路由概览

Addition 模块在 <mcfile name="router.go" path="addition/router.go"></mcfile> 中注册了以下 API 端点：

```go
func Register(app *gin.RouterGroup) {
    // 卡片生成
    app.POST("/card", card.HandlerAPI)
    
    // 代码项目生成
    app.GET("/generation/create", generation.GenerateAPI)
    app.GET("/generation/download/tar", generation.ProjectTarDownloadAPI)
    app.GET("/generation/download/zip", generation.ProjectZipDownloadAPI)
    
    // 文章生成
    app.GET("/article/create", article.GenerateAPI)
    app.GET("/article/download/tar", article.ProjectTarDownloadAPI)
    app.GET("/article/download/zip", article.ProjectZipDownloadAPI)
}
```
> **源代码位置**: <mcfile name="router.go" path="addition/router.go"></mcfile> 第10-21行

---

## 1. Card API - 动态卡片生成

### 端点
`POST /card`

### 功能描述
生成基于 AI 对话内容的动态 SVG 卡片，支持明暗主题切换和 Markdown 渲染。

### 请求参数

#### 请求体 (JSON)
```json
{
    "message": "string",  // 必需，要生成卡片的消息内容
    "web": boolean        // 可选，是否启用网络搜索增强，默认 false
}
```
> **源代码位置**: <mcfile name="card.go" path="addition/card/card.go"></mcfile> 第12-15行 - `RequestForm` 结构体

**参数详解：**
- `message`: 用户输入的消息内容，将作为 AI 对话的输入
- `web`: 布尔值，启用后会通过网络搜索获取实时信息来增强 AI 回答

### 响应格式

#### 成功响应
```json
{
    "message": "string",  // AI 生成的回答内容（HTML 格式）
    "keyword": "string",  // 关键词字段（当前为空字符串）
    "quota": 0.0         // 消耗的配额数量
}
```
> **源代码位置**: <mcfile name="card.go" path="addition/card/card.go"></mcfile> 第65-69行 - `HandlerAPI` 函数返回

#### 响应说明
- `message`: AI 处理后的回答内容，已转换为 HTML 格式，支持 Markdown 语法
- `quota`: 本次请求消耗的用户配额

### PHP 卡片渲染

卡片的最终渲染由 `card.php` 完成，支持以下 URL 参数：

#### URL 参数
- `theme`: 主题模式，`light`（默认）或 `dark`
- `message`: 消息内容
- `web`: 是否启用网络搜索，`true` 或 `false`（默认）
- `sign`: 是否显示签名，`true` 或 `false`（默认）

#### 示例 URL
```
http://localhost:8094/card.php?message=你好&theme=dark&web=false&sign=true
```

### 技术特性
- **自动换行**: 每行最多 50 个字符，超出自动换行
- **Markdown 支持**: 使用 `blackfriday` 库转换 Markdown 为 HTML
- **响应式设计**: 根据内容长度自动调整卡片高度
- **主题切换**: 支持明暗两种主题风格

---

## 2. Generation API - 代码项目生成

### 创建项目端点
`GET /generation/create`

### 功能描述
基于用户描述生成完整的代码项目，支持多种编程语言和框架，通过 WebSocket 提供实时进度反馈。

### WebSocket 连接
此 API 使用 WebSocket 协议进行实时通信。

#### 连接建立
客户端需要升级 HTTP 连接为 WebSocket 连接。

#### 请求消息格式
```json
{
    "token": "string",   // 必需，用户认证令牌
    "prompt": "string",  // 必需，项目描述
    "model": "string"    // 必需，使用的 AI 模型名称
}
```
> **源代码位置**: <mcfile name="api.go" path="addition/generation/api.go"></mcfile> 第12-16行 - `WebsocketGenerationForm` 结构体

**参数详解：**
- `token`: 用户身份验证令牌，用于权限验证和配额管理
- `prompt`: 项目需求描述，例如 "创建一个 React 博客系统"
- `model`: 指定使用的 AI 模型，如 "gpt-4", "claude-3" 等

#### 响应消息格式

##### 进度更新消息
```json
{
    "quota": 0.0,        // 已消耗配额
    "message": "string", // 生成进度消息
    "hash": "",          // 项目标识符（进度中为空）
    "end": false,        // 是否结束
    "error": ""          // 错误信息（如有）
}
```

##### 完成消息
```json
{
    "quota": 15.5,           // 总消耗配额
    "message": "",           // 消息内容（完成时为空）
    "hash": "abc123def456",  // 项目唯一标识符
    "end": true,             // 标识生成完成
    "error": ""              // 错误信息（如有）
}
```
> **源代码位置**: <mcfile name="types.go" path="globals/types.go"></mcfile> 第18-26行 - `GenerationSegmentResponse` 结构体

### 项目下载端点

#### TAR 格式下载
`GET /generation/download/tar?hash={project_hash}`
> **源代码位置**: <mcfile name="api.go" path="addition/generation/api.go"></mcfile> 第18-22行 - `ProjectTarDownloadAPI` 函数

#### ZIP 格式下载
`GET /generation/download/zip?hash={project_hash}`
> **源代码位置**: <mcfile name="api.go" path="addition/generation/api.go"></mcfile> 第24-28行 - `ProjectZipDownloadAPI` 函数

**参数：**
- `hash`: 项目生成时返回的唯一标识符

**响应：**
- 直接返回文件流，浏览器会自动下载
- TAR 文件名：`code.tar.gz`
- ZIP 文件名：`code.zip`

### 生成特性
- **多语言支持**: Python, Go, JavaScript, Java, C++ 等
- **完整项目结构**: 包含源码、配置文件、README 文档
- **智能缓存**: 相同请求会复用已生成的项目
- **实时反馈**: WebSocket 提供生成进度更新

### 项目结构示例
生成的项目通常包含：
```
project/
├── src/           # 源代码目录
├── config/        # 配置文件
├── docs/          # 文档
├── tests/         # 测试文件
├── README.md      # 项目说明
├── package.json   # 依赖管理（Node.js）
└── requirements.txt # 依赖管理（Python）
```

---

## 3. Article API - 文章生成

### 创建文章端点
`GET /article/create`

### 功能描述
基于用户提供的主题和提示词生成文章内容，支持批量生成多篇文章，输出为 Word 文档格式。

### WebSocket 连接
此 API 使用 WebSocket 协议进行实时通信。

#### 请求消息格式
```json
{
    "token": "string",   // 必需，用户认证令牌
    "model": "string",   // 必需，使用的 AI 模型
    "prompt": "string",  // 必需，文章写作提示词
    "title": "string",   // 必需，文章标题（支持多个标题，换行分隔）
    "web": boolean       // 可选，是否启用网络搜索增强
}
```
> **源代码位置**: <mcfile name="api.go" path="addition/article/api.go"></mcfile> 第11-17行 - `WebsocketArticleForm` 结构体

**参数详解：**
- `token`: 用户身份验证令牌
- `model`: AI 模型名称，如 "gpt-4", "claude-3-sonnet" 等
- `prompt`: 文章写作指导，例如 "写一篇关于人工智能发展的技术文章"
- `title`: 文章标题，支持多个标题（每行一个），会为每个标题生成独立文章
- `web`: 是否启用网络搜索来获取最新信息

#### 响应消息格式

##### 进度更新消息
```json
{
    "hash": "",
    "data": {
        "current": 2,      // 当前完成的文章数量
        "total": 5,        // 总文章数量
        "quota": 25.8      // 已消耗配额
    }
}
```

##### 完成消息
```json
{
    "hash": "def789ghi012",  // 文章生成唯一标识符
    "data": {
        "current": 5,
        "total": 5,
        "quota": 67.5
    }
}
```
> **源代码位置**: 
> - <mcfile name="api.go" path="addition/article/api.go"></mcfile> 第18-21行 - `WebsocketArticleResponse` 结构体
> - <mcfile name="generate.go" path="addition/article/generate.go"></mcfile> 第13-17行 - `StreamProgressResponse` 结构体

### 文章下载端点

#### TAR 格式下载
`GET /article/download/tar?hash={article_hash}`
> **源代码位置**: <mcfile name="api.go" path="addition/article/api.go"></mcfile> 第23-27行 - `ProjectTarDownloadAPI` 函数

#### ZIP 格式下载
`GET /article/download/zip?hash={article_hash}`
> **源代码位置**: <mcfile name="api.go" path="addition/article/api.go"></mcfile> 第29-33行 - `ProjectZipDownloadAPI` 函数

**参数：**
- `hash`: 文章生成时返回的唯一标识符

**响应：**
- 直接返回文件流
- TAR 文件名：`article.tar.gz`
- ZIP 文件名：`article.zip`

### 文章特性
- **Word 文档输出**: 使用 `go-docx` 库生成 .docx 格式文件
- **模板支持**: 基于 `template.docx` 模板文件生成格式化文档
- **批量生成**: 支持一次生成多篇相关文章
- **网络增强**: 可集成实时网络信息提升内容质量
- **并发处理**: 多篇文章并行生成，提高效率

### 文档结构
生成的文章包含：
- **标题**: 用户指定的文章标题
- **正文**: AI 生成的文章内容
- **格式化**: 基于模板的专业排版

---

## 4. 网络搜索增强 (Web Module)

虽然网络搜索模块没有直接的 API 端点，但它为其他模块提供了重要的增强功能。

### 功能特性
- **多搜索引擎**: 支持 DuckDuckGo、Bing、WebPilot
- **智能关键词提取**: 自动从用户输入中提取搜索关键词
- **内容清理**: 过滤和格式化搜索结果
- **实时信息注入**: 将搜索结果整合到 AI 对话中

### 搜索流程
1. **关键词提取**: 使用 AI 从用户输入中提取搜索关键词
2. **多源搜索**: 依次尝试 DuckDuckGo、WebPilot、Bing
3. **内容解析**: 提取有效信息，过滤无关内容
4. **结果整合**: 将搜索结果格式化后注入到 AI 对话中

---

## 使用示例

### 1. 生成聊天卡片
```bash
curl -X POST http://localhost:8094/card \
  -H "Content-Type: application/json" \
  -d '{
    "message": "请介绍一下人工智能的发展历程",
    "web": true
  }'
```

### 2. 生成代码项目
```javascript
const ws = new WebSocket('ws://localhost:8094/generation/create');

ws.onopen = function() {
    ws.send(JSON.stringify({
        token: "your_auth_token",
        prompt: "创建一个基于 React 和 TypeScript 的博客系统",
        model: "gpt-4"
    }));
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('进度:', data.data.current + '/' + data.data.total);
    
    if (data.data.current === data.data.total) {
        // 生成完成，可以下载
        window.open(`/generation/download/zip?hash=${data.hash}`);
    }
};
```

### 3. 生成文章
```javascript
const ws = new WebSocket('ws://localhost:8094/article/create');

ws.onopen = function() {
    ws.send(JSON.stringify({
        token: "your_auth_token",
        model: "claude-3-sonnet",
        prompt: "写一篇专业的技术文章，要求逻辑清晰，内容详实",
        title: "人工智能在医疗领域的应用\n机器学习算法优化策略\n深度学习的未来发展趋势",
        web: true
    }));
};
```

---

## 注意事项

### 认证和权限
- 所有 WebSocket API 都需要有效的用户令牌
- 配额不足时会返回相应错误信息
- 未认证用户可能有功能限制

### 性能考虑
- 大型项目生成可能需要较长时间
- 建议设置合理的超时时间
- 并发请求可能受到限制

### 错误处理
- WebSocket 连接断开时需要重新连接
- 生成失败时会返回错误信息
- 下载链接有时效性，建议及时下载

### 文件管理
- 生成的文件会在服务器保存一段时间
- 建议定期清理不需要的文件
- 大文件下载可能需要断点续传支持