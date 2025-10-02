# Admin User 功能整理

## 功能概述
`user.go` 文件主要实现了与用户管理相关的功能，包括分页获取用户列表、修改用户配额、修改用户订阅时长以及更新管理员密码。

---

## 方法功能描述

### 分页获取用户列表
- **`GetUserPagination(db *sql.DB, page int64, search string)`**  
  功能：分页获取用户列表，支持通过用户名搜索。  
  参数：
  - `db`：数据库连接。
  - `page`：当前页码。
  - `search`：搜索关键词。  
  返回值：`PaginationForm`，包含分页数据和状态信息。  
  示例：
  ```json
  {
    "status": true,
    "total": 5,
    "data": [
      {
        "id": 1,
        "username": "admin",
        "is_admin": true,
        "quota": 100,
        "used_quota": 50,
        "is_subscribed": true,
        "total_month": 12,
        "enterprise": false,
        "level": 1
      }
    ]
  }
  ```

### 修改用户配额
- **`QuotaOperation(db *sql.DB, id int64, quota float32)`**  
  功能：修改指定用户的配额。  
  参数：
  - `db`：数据库连接。
  - `id`：用户 ID。
  - `quota`：配额值（正数增加，负数减少）。  
  返回值：`error`，操作失败时返回错误信息。

### 修改用户订阅时长
- **`SubscriptionOperation(db *sql.DB, id int64, month int64)`**  
  功能：修改指定用户的订阅时长。  
  参数：
  - `db`：数据库连接。
  - `id`：用户 ID。
  - `month`：订阅时长（月）（正数增加，负数减少）。  
  返回值：`error`，操作失败时返回错误信息。

### 更新管理员密码
- **`UpdateRootPassword(db *sql.DB, cache *redis.Client, password string)`**  
  功能：更新管理员的根密码。  
  参数：
  - `db`：数据库连接。
  - `cache`：Redis 缓存客户端。
  - `password`：新密码（长度需在 6 到 36 个字符之间）。  
  返回值：`error`，操作失败时返回错误信息。

---

## 方法逻辑说明

### 分页获取用户列表
1. 查询数据库中符合搜索条件的用户总数。
2. 根据分页参数查询指定页码的用户数据。
3. 格式化查询结果并返回。

### 修改用户配额
1. 构造 SQL 插入或更新语句。
2. 执行操作，更新用户的配额值。

### 修改用户订阅时长
1. 计算新的订阅到期时间。
2. 构造 SQL 插入或更新语句。
3. 执行操作，更新用户的订阅时长和到期时间。

### 更新管理员密码
1. 检查密码长度是否符合要求。
2. 使用 SHA-256 加密新密码。
3. 更新数据库中管理员的密码。
4. 清除 Redis 缓存中的管理员用户信息。

---
