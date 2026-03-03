# 企业伴身智能平台 - 详细设计文档

## 第八部分：API接口设计规范

---

## 1. 概述

本文档定义了企业伴身智能平台的API接口设计规范，包括URL设计、请求/响应格式、认证授权、错误处理、版本管理等内容，为前后端开发提供统一的接口标准。

---

## 2. 基本规范

### 2.1 接口风格

采用 **RESTful API** 设计风格，遵循以下原则：

- 使用 HTTP 方法表示操作类型
- 使用 URL 表示资源
- 使用 HTTP 状态码表示结果
- 使用 JSON 格式传输数据

### 2.2 HTTP 方法使用

| HTTP方法 | 操作类型 | 说明 | 示例 |
|----------|----------|------|------|
| GET | 查询 | 获取资源 | GET /users/{id} |
| POST | 创建 | 创建资源 | POST /users |
| PUT | 全量更新 | 更新整个资源 | PUT /users/{id} |
| PATCH | 部分更新 | 更新资源部分字段 | PATCH /users/{id} |
| DELETE | 删除 | 删除资源 | DELETE /users/{id} |

### 2.3 URL设计规范

```
基础格式: https://{domain}/api/v{version}/{module}/{resource}

示例:
- https://api.example.com/api/v1/users                    # 用户列表
- https://api.example.com/api/v1/users/10001              # 用户详情
- https://api.example.com/api/v1/users/10001/avatars      # 用户的分身
- https://api.example.com/api/v1/avatars/20001/skills     # 分身的技能
- https://api.example.com/api/v1/conversations/1001/messages  # 会话的消息
```

**URL命名规则：**

| 规则 | 说明 | 正确示例 | 错误示例 |
|------|------|----------|----------|
| 使用小写字母 | 全部小写 | /users | /Users |
| 使用连字符 | 多单词用连字符 | /knowledge-bases | /knowledgeBases |
| 使用复数名词 | 资源用复数 | /users | /user |
| 不使用动词 | 动作用HTTP方法表示 | POST /users | /createUser |
| 层级关系 | 用斜杠表示 | /users/10001/roles | /getUserRoles |

### 2.4 请求头规范

**必需请求头：**

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}
X-Request-Id: {uuid}          # 请求唯一标识，用于链路追踪
X-Corp-Id: {corp_id}          # 企业ID
X-Client-Type: {client_type}  # 客户端类型: 1-PC, 2-APP, 3-小程序
```

**可选请求头：**

```http
X-Device-Id: {device_id}      # 设备ID
X-App-Version: {version}      # 客户端版本
X-Timezone: Asia/Shanghai     # 时区
Accept-Language: zh-CN        # 语言
```

---

## 3. 请求格式

### 3.1 查询参数 (Query Parameters)

用于GET请求的过滤、分页、排序：

```http
GET /api/v1/users?page=1&size=20&status=1&dept_id=5&keyword=张&sort=created_at&order=desc
```

**分页参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| page | Integer | 1 | 页码，从1开始 |
| size | Integer | 20 | 每页数量，最大100 |
| sort | String | id | 排序字段 |
| order | String | desc | 排序方向: asc/desc |

**过滤参数命名规则：**

| 类型 | 命名格式 | 示例 |
|------|----------|------|
| 精确匹配 | field | status=1 |
| 范围查询 | field_min, field_max | amount_min=100&amount_max=500 |
| 日期范围 | field_start, field_end | created_at_start=2024-01-01 |
| 模糊搜索 | keyword | keyword=张三 |
| 多值查询 | field (逗号分隔) | status=1,2,3 |

### 3.2 路径参数 (Path Parameters)

用于标识具体资源：

```http
GET /api/v1/users/{user_id}
GET /api/v1/avatars/{avatar_id}/skills/{skill_id}
```

### 3.3 请求体 (Request Body)

用于POST、PUT、PATCH请求：

```json
{
  "field_name": "value",
  "nested_object": {
    "sub_field": "value"
  },
  "array_field": [1, 2, 3]
}
```

**字段命名：** 使用 snake_case（下划线命名法）

---

## 4. 响应格式

### 4.1 统一响应结构

**成功响应：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    // 业务数据
  },
  "timestamp": 1704067200000,
  "request_id": "uuid-xxx"
}
```

**分页响应：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 100,
    "page": 1,
    "size": 20,
    "pages": 5,
    "items": [
      // 数据列表
    ]
  },
  "timestamp": 1704067200000,
  "request_id": "uuid-xxx"
}
```

**错误响应：**

```json
{
  "code": 40001,
  "message": "参数错误",
  "errors": [
    {
      "field": "username",
      "message": "用户名不能为空"
    },
    {
      "field": "mobile",
      "message": "手机号格式不正确"
    }
  ],
  "timestamp": 1704067200000,
  "request_id": "uuid-xxx"
}
```

### 4.2 HTTP状态码使用

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 200 | OK | 请求成功 |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功，无返回内容 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证/Token无效 |
| 403 | Forbidden | 无权限访问 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Entity | 业务逻辑错误 |
| 429 | Too Many Requests | 请求过于频繁 |
| 500 | Internal Server Error | 服务器内部错误 |
| 503 | Service Unavailable | 服务不可用 |

### 4.3 数据类型规范

| 数据类型 | JSON类型 | 格式说明 | 示例 |
|----------|----------|----------|------|
| 整数ID | Number | 使用Number类型 | 10001 |
| 长整数ID | String | 超过JS安全整数用String | "9007199254740993" |
| 日期时间 | String | ISO 8601格式或标准格式 | "2024-01-15 10:00:00" |
| 日期 | String | YYYY-MM-DD | "2024-01-15" |
| 时间 | String | HH:mm:ss | "10:00:00" |
| 金额 | Number | 使用分为单位或String | 10000 / "100.00" |
| 布尔值 | Boolean | true/false | true |
| 空值 | null | 明确表示空 | null |
| 枚举 | Number/String | 根据业务定义 | 1 / "active" |

---

## 5. 认证授权

### 5.1 认证方式

采用 **JWT (JSON Web Token)** 进行身份认证：

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 5.2 Token结构

**Access Token（有效期2小时）：**

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "10001",           // 用户ID
    "corp_id": 1001,          // 企业ID
    "username": "zhangsan",   // 用户名
    "user_type": 1,           // 用户类型
    "permissions": ["user:view", "task:*"],  // 权限列表
    "iat": 1704067200,        // 签发时间
    "exp": 1704074400         // 过期时间
  }
}
```

**Refresh Token（有效期7天）：**

```json
{
  "payload": {
    "sub": "10001",
    "corp_id": 1001,
    "token_type": "refresh",
    "iat": 1704067200,
    "exp": 1704672000
  }
}
```

### 5.3 Token刷新流程

```
Client                                 Server
   |                                      |
   | 1. 发现AccessToken即将过期            |
   |                                      |
   | 2. POST /api/v1/auth/refresh         |
   |    { refresh_token: "xxx" }          |
   |------------------------------------->|
   |                                      |
   |                          3. 验证RefreshToken
   |                          4. 生成新Token对
   |                                      |
   | 5. 返回新的AccessToken和RefreshToken  |
   |<-------------------------------------|
   |                                      |
```

### 5.4 无需认证的接口

以下接口不需要携带Token：

```
POST /api/v1/auth/login           # 登录
POST /api/v1/auth/refresh         # 刷新Token
POST /api/v1/corps/register       # 企业注册
POST /api/v1/common/sms/send      # 发送验证码
GET  /api/v1/common/captcha       # 获取图形验证码
```

---

## 6. 错误处理

### 6.1 错误码体系

```
错误码格式: XXYYY

XX  - 模块代码 (10-99)
YYY - 具体错误 (001-999)

示例:
10001 - 认证模块，用户名不存在
11001 - 用户模块，用户名已存在
13001 - 分身模块，分身创建失败
```

### 6.2 全局错误码

| 错误码 | HTTP状态码 | 说明 |
|--------|------------|------|
| 200 | 200 | 成功 |
| 40001 | 400 | 请求参数错误 |
| 40002 | 400 | 请求参数缺失 |
| 40003 | 400 | 参数格式错误 |
| 40004 | 400 | 参数值超出范围 |
| 40101 | 401 | 未登录 |
| 40102 | 401 | Token无效 |
| 40103 | 401 | Token已过期 |
| 40104 | 401 | 账号已锁定 |
| 40105 | 401 | 账号已禁用 |
| 40301 | 403 | 无权限访问 |
| 40302 | 403 | 企业已禁用 |
| 40303 | 403 | 功能未开通 |
| 40401 | 404 | 资源不存在 |
| 40901 | 409 | 数据冲突 |
| 42201 | 422 | 业务规则校验失败 |
| 42901 | 429 | 请求过于频繁 |
| 50001 | 500 | 服务器内部错误 |
| 50002 | 500 | 数据库错误 |
| 50003 | 500 | 缓存错误 |
| 50301 | 503 | 服务暂时不可用 |
| 50302 | 503 | 服务维护中 |

### 6.3 模块错误码

**认证模块 (10xxx):**

| 错误码 | 说明 |
|--------|------|
| 10001 | 用户名不存在 |
| 10002 | 密码错误 |
| 10003 | 验证码错误 |
| 10004 | 验证码已过期 |
| 10005 | 企业编码不存在 |

**用户模块 (11xxx):**

| 错误码 | 说明 |
|--------|------|
| 11001 | 用户名已存在 |
| 11002 | 手机号已注册 |
| 11003 | 邮箱已注册 |
| 11004 | 部门不存在 |
| 11005 | 岗位不存在 |

**分身模块 (13xxx):**

| 错误码 | 说明 |
|--------|------|
| 13001 | 分身创建失败 |
| 13002 | 分身已存在 |
| 13003 | 分身不存在 |
| 13004 | 技能数量超限 |
| 13005 | 知识库数量超限 |

**协作模块 (14xxx):**

| 错误码 | 说明 |
|--------|------|
| 14001 | 协作请求不存在 |
| 14002 | 协作已响应 |
| 14003 | 协作已完成 |
| 14004 | 无权操作此协作 |

---

## 7. 版本管理

### 7.1 版本号规范

```
URL版本: /api/v{major}
示例: /api/v1, /api/v2
```

### 7.2 版本兼容策略

| 变更类型 | 是否兼容 | 处理方式 |
|----------|----------|----------|
| 新增字段 | 兼容 | 直接添加，前端忽略未知字段 |
| 新增接口 | 兼容 | 直接添加 |
| 修改字段类型 | 不兼容 | 新版本 |
| 删除字段 | 不兼容 | 新版本，旧版本标记废弃 |
| 修改业务逻辑 | 视情况 | 评估影响范围 |

### 7.3 废弃标记

```http
# 响应头
Deprecation: true
Sunset: 2024-06-01
X-Deprecated-Message: 此接口将于2024-06-01废弃，请使用 /api/v2/users
```

---

## 8. 安全规范

### 8.1 请求签名

敏感接口需要请求签名验证：

```
签名算法: HMAC-SHA256
签名内容: timestamp + nonce + body_md5
签名密钥: 从认证中心获取
```

```http
X-Timestamp: 1704067200
X-Nonce: uuid-xxx
X-Signature: base64(hmac_sha256(content, secret))
```

### 8.2 数据加密

敏感数据传输加密：

| 数据类型 | 加密方式 |
|----------|----------|
| 密码 | RSA公钥加密传输 |
| 身份证号 | AES加密存储 |
| 手机号 | AES加密存储，查询用哈希 |
| 银行卡号 | AES加密存储 |

### 8.3 防重放攻击

```
1. 检查timestamp，与服务器时间差不超过5分钟
2. 检查nonce，5分钟内不可重复
3. 验证签名
```

### 8.4 限流规则

| 接口类型 | 限流规则 |
|----------|----------|
| 公共接口 | 100次/分钟/IP |
| 认证接口 | 10次/分钟/IP |
| 业务接口 | 300次/分钟/用户 |
| 批量接口 | 10次/分钟/用户 |

---

## 9. 接口文档规范

### 9.1 Swagger/OpenAPI规范

```yaml
openapi: 3.0.3
info:
  title: 企业伴身智能平台 API
  version: 1.0.0
  description: 企业伴身智能平台接口文档

servers:
  - url: https://api.example.com/api/v1
    description: 生产环境
  - url: https://api-test.example.com/api/v1
    description: 测试环境

paths:
  /users:
    get:
      summary: 获取用户列表
      tags:
        - 用户管理
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
```

### 9.2 接口文档模板

```markdown
## 接口名称

### 基本信息
- **接口路径:** POST /api/v1/users
- **接口描述:** 创建新用户
- **认证方式:** Bearer Token
- **权限要求:** admin:user:create

### 请求参数

| 参数名 | 类型 | 必填 | 说明 | 示例 |
|--------|------|------|------|------|
| username | String | 是 | 用户名 | zhangsan |
| real_name | String | 是 | 真实姓名 | 张三 |

### 请求示例
```json
{
  "username": "zhangsan",
  "real_name": "张三"
}
```

### 响应参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| user_id | Number | 用户ID |

### 响应示例
```json
{
  "code": 200,
  "data": {
    "user_id": 10001
  }
}
```

### 错误码

| 错误码 | 说明 |
|--------|------|
| 11001 | 用户名已存在 |
```

---

## 10. 接口清单总览

### 10.1 认证模块

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /auth/login | 用户登录 |
| POST | /auth/logout | 用户登出 |
| POST | /auth/refresh | 刷新Token |
| POST | /auth/password/reset | 重置密码 |
| POST | /auth/password/change | 修改密码 |

### 10.2 用户模块

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /users | 获取用户列表 |
| GET | /users/{id} | 获取用户详情 |
| POST | /users | 创建用户 |
| PUT | /users/{id} | 更新用户 |
| DELETE | /users/{id} | 删除用户 |
| GET | /users/me | 获取当前用户信息 |
| PUT | /users/me | 更新当前用户信息 |

### 10.3 分身模块

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /avatars | 获取分身列表 |
| GET | /avatars/{id} | 获取分身详情 |
| POST | /avatars | 创建分身 |
| PUT | /avatars/{id} | 更新分身 |
| PUT | /avatars/{id}/status | 更新分身状态 |
| GET | /avatars/{id}/skills | 获取分身技能 |
| POST | /avatars/{id}/skills | 添加分身技能 |
| GET | /avatars/{id}/knowledge-bases | 获取知识库列表 |
| POST | /avatars/{id}/knowledge-bases | 创建知识库 |
| GET | /avatars/{id}/memories | 获取记忆列表 |
| POST | /avatars/{id}/memories | 添加记忆 |
| GET | /avatars/{id}/response-rules | 获取响应规则 |
| POST | /avatars/{id}/response-rules | 创建响应规则 |
| GET | /avatars/{id}/statistics | 获取分身统计 |

### 10.4 协作沟通模块

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /conversations | 获取会话列表 |
| GET | /conversations/{id} | 获取会话详情 |
| POST | /conversations | 创建会话 |
| PUT | /conversations/{id} | 更新会话 |
| GET | /conversations/{id}/messages | 获取消息列表 |
| POST | /conversations/{id}/messages | 发送消息 |
| GET | /collaborations | 获取协作列表 |
| POST | /collaborations | 发起协作 |
| PUT | /collaborations/{id}/accept | 接受协作 |
| PUT | /collaborations/{id}/reject | 拒绝协作 |
| POST | /collaborations/{id}/progress | 更新进度 |
| POST | /collaborations/{id}/complete | 完成协作 |

### 10.5 流程管理模块

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /tasks | 获取任务列表 |
| POST | /tasks | 创建任务 |
| PUT | /tasks/{id} | 更新任务 |
| PUT | /tasks/{id}/status | 更新任务状态 |
| GET | /approvals | 获取审批列表 |
| POST | /approvals | 发起审批 |
| POST | /approvals/{id}/approve | 审批通过 |
| POST | /approvals/{id}/reject | 审批驳回 |
| GET | /approval-templates | 获取审批模板 |

### 10.6 管理后台模块

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /admin/corp | 获取企业信息 |
| PUT | /admin/corp | 更新企业信息 |
| GET | /admin/users | 获取员工列表 |
| GET | /admin/avatars | 获取分身列表 |
| GET | /admin/roles | 获取角色列表 |
| GET | /admin/statistics/overview | 获取统计概览 |
| GET | /admin/logs/operations | 获取操作日志 |

---

*第八部分完成：API接口设计规范*
