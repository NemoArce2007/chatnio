# Admin Invitation 功能整理

## 功能概述
`invitation.go` 文件主要实现了与邀请码相关的功能，包括分页获取邀请码列表、生成邀请码等操作。这些功能用于管理和分发邀请码。

---

## 方法功能描述

### 分页获取邀请码
- **`GetInvitationPagination(db *sql.DB, page int64)`**  
  功能：分页获取邀请码列表。  
  参数：
  - `db`：数据库连接。
  - `page`：当前页码。  
  返回值：`PaginationForm`，包含分页数据和状态信息。  
  示例：
  ```json
  {
    "status": true,
    "total": 10,
    "data": [
      {
        "code": "invite-abc123",
        "quota": 10,
        "type": "standard",
        "used": false,
        "updated_at": "2023-10-05 12:00:00"
      }
    ]
  }
  ```

### 新增邀请码
- **`NewInvitationCode(db *sql.DB, code string, quota float32, t string)`**  
  功能：向数据库中插入新的邀请码。  
  参数：
  - `db`：数据库连接。
  - `code`：邀请码字符串。
  - `quota`：邀请码的配额。
  - `t`：邀请码类型。  
  返回值：`error`，插入失败时返回错误信息。

### 批量生成邀请码
- **`GenerateInvitations(db *sql.DB, num int, quota float32, t string)`**  
  功能：批量生成指定数量的邀请码。  
  参数：
  - `db`：数据库连接。
  - `num`：生成的邀请码数量。
  - `quota`：每个邀请码的配额。
  - `t`：邀请码类型。  
  返回值：`InvitationGenerateResponse`，包含生成状态和生成的邀请码列表。  
  示例：
  ```json
  {
    "status": true,
    "data": [
      "standard-abc123",
      "standard-def456"
    ]
  }
  ```

---

## 数据结构

### `PaginationForm`
分页数据结构，用于返回分页结果。  
字段：
- `Status`：操作状态，布尔值。
- `Total`：总页数。
- `Data`：分页数据列表。
- `Message`：错误信息（可选）。

### `InvitationData`
邀请码数据结构，用于表示单个邀请码的信息。  
字段：
- `Code`：邀请码字符串。
- `Quota`：邀请码的配额。
- `Type`：邀请码类型。
- `Used`：是否已使用。
- `UpdatedAt`：最后更新时间。

### `InvitationGenerateResponse`
生成邀请码的响应结构。  
字段：
- `Status`：操作状态，布尔值。
- `Data`：生成的邀请码列表。
- `Message`：错误信息（可选）。

---

## 方法逻辑说明

### 分页获取邀请码
1. 查询数据库中邀请码的总数。
2. 根据分页参数查询指定页码的邀请码数据。
3. 格式化查询结果并返回。

### 新增邀请码
1. 构造 SQL 插入语句。
2. 执行插入操作，将邀请码存入数据库。

### 批量生成邀请码
1. 循环生成指定数量的邀请码。
2. 对每个邀请码调用 `NewInvitationCode` 方法插入数据库。
3. 如果遇到重复邀请码，重试生成，最多重试 100 次。
4. 返回生成成功的邀请码列表。

---
