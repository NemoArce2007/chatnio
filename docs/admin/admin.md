# Admin 模块文档

## 简介
Admin 模块提供了管理员相关的功能，包括用户管理、邀请管理、兑换码管理、统计分析等。

## 功能模块
- **用户管理**: 支持分页查询用户、调整用户配额、管理订阅等。
- **邀请管理**: 支持生成邀请代码、分页查询邀请记录。
- **兑换码管理**: 支持生成兑换码、查询兑换码统计信息。
- **统计分析**: 提供请求、计费、错误等多维度的统计数据。

## API 接口

### 用户管理
- **获取用户分页列表**
  - **路径**: `/admin/user/list`
  - **方法**: GET
  - **参数**:
    - `page` (int): 页码
    - `search` (string): 搜索关键字
  - **返回**: 用户分页数据

- **调整用户配额**
  - **路径**: `/admin/user/quota`
  - **方法**: POST
  - **参数**:
    - `id` (int64): 用户 ID
    - `quota` (float32): 配额调整值
  - **返回**: 操作状态

- **调整用户订阅**
  - **路径**: `/admin/user/subscription`
  - **方法**: POST
  - **参数**:
    - `id` (int64): 用户 ID
    - `month` (int64): 订阅调整月份
  - **返回**: 操作状态

- **更新管理员密码**
  - **路径**: `/admin/user/root`
  - **方法**: POST
  - **参数**:
    - `password` (string): 新密码
  - **返回**: 操作状态

### 邀请管理
- **获取邀请分页列表**
  - **路径**: `/admin/invitation/list`
  - **方法**: GET
  - **参数**:
    - `page` (int): 页码
  - **返回**: 邀请分页数据

- **生成邀请代码**
  - **路径**: `/admin/invitation/generate`
  - **方法**: POST
  - **参数**:
    - `type` (string): 邀请类型
    - `quota` (float32): 配额
    - `number` (int): 生成数量
  - **返回**: 生成的邀请代码列表

### 兑换码管理
- **获取兑换码统计信息**
  - **路径**: `/admin/redeem/list`
  - **方法**: GET
  - **返回**: 兑换码统计数据

- **生成兑换码**
  - **路径**: `/admin/redeem/generate`
  - **方法**: POST
  - **参数**:
    - `quota` (float32): 配额
    - `number` (int): 生成数量
  - **返回**: 生成的兑换码列表

### 统计分析
- **获取统计信息**
  - **路径**: `/admin/analytics/info`
  - **方法**: GET
  - **返回**: 统计信息

- **获取模型分析数据**
  - **路径**: `/admin/analytics/model`
  - **方法**: GET
  - **返回**: 模型分析数据

- **获取请求分析数据**
  - **路径**: `/admin/analytics/request`
  - **方法**: GET
  - **返回**: 请求分析数据

- **获取计费分析数据**
  - **路径**: `/admin/analytics/billing`
  - **方法**: GET
  - **返回**: 计费分析数据

- **获取错误分析数据**
  - **路径**: `/admin/analytics/error`
  - **方法**: GET
  - **返回**: 错误分析数据

## 数据结构
- **用户数据 (UserData)**
  - `id` (int64): 用户 ID
  - `username` (string): 用户名
  - `is_admin` (bool): 是否为管理员
  - `quota` (float32): 配额
  - `used_quota` (float32): 已用配额
  - `is_subscribed` (bool): 是否订阅
  - `total_month` (int64): 总订阅月份
  - `enterprise` (bool): 是否为企业用户
  - `level` (int): 用户等级

- **分页数据 (PaginationForm)**
  - `status` (bool): 操作状态
  - `total` (int): 总页数
  - `data` ([]interface{}): 数据列表
  - `message` (string): 错误信息

## 注意事项
- 所有接口均需管理员权限。
- 数据库操作可能会因并发或数据完整性问题导致失败，请做好错误处理。
