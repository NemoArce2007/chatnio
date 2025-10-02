# Analysis 功能说明

`analysis.go` 文件主要用于提供与数据分析相关的功能，帮助管理员快速了解系统的运行状态和用户行为。通过数据库和 Redis 缓存系统，`analysis.go` 提供了高效的数据统计和分析能力。

## 功能概述

1. **订阅用户统计**: 统计当前有效订阅用户的数量。
2. **账单统计**: 提供当天和当月的账单金额统计。
3. **模型使用分析**: 分析最近 7 天的模型使用情况。
4. **请求数据统计**: 统计最近 7 天的请求数量。
5. **账单数据分析**: 提供最近 30 天的账单数据趋势。
6. **错误数据统计**: 分析最近 7 天的错误请求数量。

---

## 提供的接口

### 1. `GetSubscriptionUsers(db *sql.DB) int64`
- **功能**: 获取当前有效订阅用户的数量。
- **实现逻辑**: 
  - 查询数据库中 `subscription` 表，统计 `expired_at` 字段大于当前时间的记录数量。
- **使用场景**: 用于展示当前系统的订阅用户规模。
- **示例代码**:
  ```go
  count := GetSubscriptionUsers(db)
  fmt.Printf("当前订阅用户数量: %d\n", count)
  ```

---

### 2. `GetBillingToday(cache *redis.Client) float32`
- **功能**: 获取当天的账单金额。
- **实现逻辑**: 
  - 从 Redis 缓存中获取当天的账单数据，按分为单位存储，返回时转换为元。
- **使用场景**: 用于展示当天的收入情况。
- **示例代码**:
  ```go
  todayBilling := GetBillingToday(cache)
  fmt.Printf("今日账单金额: %.2f 元\n", todayBilling)
  ```

---

### 3. `GetBillingMonth(cache *redis.Client) float32`
- **功能**: 获取当月的账单金额。
- **实现逻辑**: 
  - 从 Redis 缓存中获取当月的账单数据，按分为单位存储，返回时转换为元。
- **使用场景**: 用于展示当月的累计收入。
- **示例代码**:
  ```go
  monthBilling := GetBillingMonth(cache)
  fmt.Printf("本月账单金额: %.2f 元\n", monthBilling)
  ```

---

### 4. `GetModelData(cache *redis.Client) ModelChartForm`
- **功能**: 获取最近 7 天的模型使用数据。
- **实现逻辑**: 
  - 遍历所有模型，统计每个模型在最近 7 天的使用次数。
  - 如果某模型的总使用次数为 0，则不返回该模型的数据。
- **使用场景**: 用于分析不同模型的使用频率，优化模型资源分配。
- **示例代码**:
  ```go
  modelData := GetModelData(cache)
  fmt.Printf("模型使用数据: %+v\n", modelData)
  ```

---

### 5. `GetRequestData(cache *redis.Client) RequestChartForm`
- **功能**: 获取最近 7 天的请求数据。
- **实现逻辑**: 
  - 从 Redis 缓存中获取最近 7 天的请求数量。
- **使用场景**: 用于监控系统的请求负载，分析用户活跃度。
- **示例代码**:
  ```go
  requestData := GetRequestData(cache)
  fmt.Printf("请求数据: %+v\n", requestData)
  ```

---

### 6. `GetBillingData(cache *redis.Client) BillingChartForm`
- **功能**: 获取最近 30 天的账单数据。
- **实现逻辑**: 
  - 从 Redis 缓存中获取最近 30 天的账单金额，按分为单位存储，返回时转换为元。
- **使用场景**: 用于分析收入趋势，制定运营策略。
- **示例代码**:
  ```go
  billingData := GetBillingData(cache)
  fmt.Printf("账单数据: %+v\n", billingData)
  ```

---

### 7. `GetErrorData(cache *redis.Client) ErrorChartForm`
- **功能**: 获取最近 7 天的错误数据。
- **实现逻辑**: 
  - 从 Redis 缓存中获取最近 7 天的错误请求数量。
- **使用场景**: 用于监控系统的错误率，排查潜在问题。
- **示例代码**:
  ```go
  errorData := GetErrorData(cache)
  fmt.Printf("错误数据: %+v\n", errorData)
  ```

---

## 数据处理工具

### 1. `getDates`
- **功能**: 将时间数组转换为日期字符串数组。
- **示例**:
  ```go
  dates := getDates([]time.Time{time.Now()})
  fmt.Println(dates) // 输出: ["10/5"]
  ```

### 2. `getFormat`
- **功能**: 格式化时间为 `YYYY-MM-DD` 格式。
- **示例**:
  ```go
  formatted := getFormat(time.Now())
  fmt.Println(formatted) // 输出: "2023-10-05"
  ```

### 3. `utils.MustInt`
- **功能**: 从 Redis 中获取整数值，若不存在则返回 0。
- **示例**:
  ```go
  value := utils.MustInt(cache, "some_key")
  fmt.Println(value)
  ```

### 4. `utils.Each` 和 `utils.EachNotNil`
- **功能**: 用于对数据进行映射和过滤处理。
- **示例**:
  ```go
  result := utils.Each([]int{1, 2, 3}, func(i int) int {
      return i * 2
  })
  fmt.Println(result) // 输出: [2, 4, 6]
  ```

