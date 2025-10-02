# Admin Statistic 功能整理

## 功能概述
`statistic.go` 文件主要实现了统计相关的功能，包括错误请求计数、计费请求计数、普通请求计数和模型请求计数的递增操作，以及请求分析的逻辑。

---

## 方法功能描述

### 错误请求计数
- **`IncrErrorRequest(cache *redis.Client)`**  
  功能：递增错误请求计数，并设置过期时间为 14 天。  
  参数：
  - `cache`：Redis 缓存客户端。

### 计费请求计数
- **`IncrBillingRequest(cache *redis.Client, amount int64)`**  
  功能：递增计费请求计数，并设置过期时间为 60 天。  
  参数：
  - `cache`：Redis 缓存客户端。
  - `amount`：计费金额。

### 普通请求计数
- **`IncrRequest(cache *redis.Client)`**  
  功能：递增普通请求计数，并设置过期时间为 14 天。  
  参数：
  - `cache`：Redis 缓存客户端。

### 模型请求计数
- **`IncrModelRequest(cache *redis.Client, model string, tokens int64)`**  
  功能：递增指定模型的请求计数，并设置过期时间为 14 天。  
  参数：
  - `cache`：Redis 缓存客户端。
  - `model`：模型名称。
  - `tokens`：使用的 token 数量。

### 请求分析
- **`AnalysisRequest(model string, buffer *utils.Buffer, err error)`**  
  功能：根据请求结果递增相应的计数器（错误请求、普通请求、模型请求）。  
  参数：
  - `model`：模型名称。
  - `buffer`：请求的缓冲区。
  - `err`：请求的错误信息。

---

## 方法逻辑说明

### 错误请求计数
1. 调用 Redis 的 `Incr` 方法递增错误计数。
2. 设置计数器的过期时间为 14 天。

### 计费请求计数
1. 调用 Redis 的 `Incr` 方法递增当天的计费计数。
2. 调用 Redis 的 `Incr` 方法递增当月的计费计数。
3. 设置计数器的过期时间为 60 天。

### 普通请求计数
1. 调用 Redis 的 `Incr` 方法递增普通请求计数。
2. 设置计数器的过期时间为 14 天。

### 模型请求计数
1. 调用 Redis 的 `Incr` 方法递增指定模型的请求计数。
2. 设置计数器的过期时间为 14 天。

### 请求分析
1. 如果请求返回错误且错误信息不是 "signal"，递增错误请求计数。
2. 否则，递增普通请求计数。
3. 递增指定模型的请求计数。

---
