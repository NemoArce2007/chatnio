
# Admin Format 功能整理

## 功能概述
`format.go` 文件主要提供了一些格式化工具函数，用于生成特定格式的字符串或日期列表。这些工具函数在统计分析和数据处理时被广泛使用。

---

## 方法功能描述

### 日期相关
- **`getMonth()`**  
  功能：返回当前月份的字符串格式，格式为 `YYYY-MM`。  
  示例：`2023-10`

- **`getDay()`**  
  功能：返回当前日期的字符串格式，格式为 `YYYY-MM-DD`。  
  示例：`2023-10-05`

- **`getDays(n int)`**  
  功能：返回从当前日期开始的过去 `n` 天的日期列表。  
  参数：
  - `n`：需要返回的天数。
  返回值：`[]time.Time` 类型的日期列表。

### 错误分析相关
- **`getErrorFormat(t string)`**  
  功能：根据传入的时间字符串生成错误分析的 Redis 键格式。  
  参数：
  - `t`：时间字符串。  
  返回值：格式化后的字符串。  
  示例：`nio:err-analysis-2023-10-05`

### 计费分析相关
- **`getBillingFormat(t string)`**  
  功能：根据传入的时间字符串生成计费分析的 Redis 键格式。  
  参数：
  - `t`：时间字符串。  
  返回值：格式化后的字符串。  
  示例：`nio:billing-analysis-2023-10-05`

- **`getMonthBillingFormat(t string)`**  
  功能：根据传入的月份字符串生成月度计费分析的 Redis 键格式。  
  参数：
  - `t`：月份字符串。  
  返回值：格式化后的字符串。  
  示例：`nio:billing-analysis-2023-10`

### 请求分析相关
- **`getRequestFormat(t string)`**  
  功能：根据传入的时间字符串生成请求分析的 Redis 键格式。  
  参数：
  - `t`：时间字符串。  
  返回值：格式化后的字符串。  
  示例：`nio:request-analysis-2023-10-05`

### 模型分析相关
- **`getModelFormat(t string, model string)`**  
  功能：根据传入的时间字符串和模型名称生成模型分析的 Redis 键格式。  
  参数：
  - `t`：时间字符串。
  - `model`：模型名称。  
  返回值：格式化后的字符串。  
  示例：`nio:model-analysis-gpt-4-2023-10-05`

---
