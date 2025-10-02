# Admin Types 功能整理

## 功能概述
`types.go` 文件定义了多个数据结构，这些结构主要用于管理后台的 API 请求和响应，包括分页数据、统计数据、用户数据、邀请码和兑换码数据等。

---

## 数据结构描述

### 分页相关
- **`PaginationForm`**  
  分页数据结构，用于返回分页结果。  
  字段：
  - `Status`：操作状态，布尔值。
  - `Total`：总页数。
  - `Data`：分页数据列表。
  - `Message`：错误信息（可选）。

### 统计相关
- **`InfoForm`**  
  系统统计信息结构。  
  字段：
  - `BillingToday`：今日收入。
  - `BillingMonth`：本月收入。
  - `SubscriptionCount`：订阅用户数量。

- **`ModelData`**  
  模型分析数据结构。  
  字段：
  - `Model`：模型名称。
  - `Data`：模型使用数据。

- **`ModelChartForm`**  
  模型分析图表数据结构。  
  字段：
  - `Date`：日期列表。
  - `Value`：模型数据列表。

- **`RequestChartForm`**  
  请求分析图表数据结构。  
  字段：
  - `Date`：日期列表。
  - `Value`：请求数量列表。

- **`BillingChartForm`**  
  计费分析图表数据结构。  
  字段：
  - `Date`：日期列表。
  - `Value`：收入数据列表。

- **`ErrorChartForm`**  
  错误分析图表数据结构。  
  字段：
  - `Date`：日期列表。
  - `Value`：错误数量列表。

### 邀请码相关
- **`InvitationData`**  
  邀请码数据结构，用于表示单个邀请码的信息。  
  字段：
  - `Code`：邀请码字符串。
  - `Quota`：邀请码的配额。
  - `Type`：邀请码类型。
  - `Used`：是否已使用。
  - `UpdatedAt`：最后更新时间。

- **`InvitationGenerateResponse`**  
  生成邀请码的响应结构。  
  字段：
  - `Status`：操作状态，布尔值。
  - `Message`：错误信息（可选）。
  - `Data`：生成的邀请码列表。

### 兑换码相关
- **`RedeemData`**  
  兑换码统计数据结构，用于表示每种配额的总数和已使用数量。  
  字段：
  - `Quota`：兑换码的配额。
  - `Used`：已使用的数量。
  - `Total`：总数量。

- **`RedeemGenerateResponse`**  
  生成兑换码的响应结构。  
  字段：
  - `Status`：操作状态，布尔值。
  - `Message`：错误信息（可选）。
  - `Data`：生成的兑换码列表。

### 用户相关
- **`UserData`**  
  用户数据结构，用于表示单个用户的信息。  
  字段：
  - `Id`：用户 ID。
  - `Username`：用户名。
  - `IsAdmin`：是否为管理员。
  - `Quota`：用户的配额。
  - `UsedQuota`：已使用的配额。
  - `IsSubscribed`：是否订阅。
  - `TotalMonth`：订阅总时长（月）。
  - `Enterprise`：是否为企业用户。
  - `Level`：用户等级。

---

## 数据结构用途
1. **分页相关**：用于返回分页数据，例如用户列表、邀请码列表等。
2. **统计相关**：用于返回系统的统计信息和分析数据。
3. **邀请码相关**：用于管理和生成邀请码。
4. **兑换码相关**：用于管理和生成兑换码。
5. **用户相关**：用于表示用户的详细信息。

---
