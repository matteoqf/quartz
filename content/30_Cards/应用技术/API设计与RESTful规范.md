---
type: procedure
tags:
  - API
  - REST
  - Web服务
  - 接口设计
created: 2026-04-21
modified: 2026-04-21
---

# API设计与RESTful规范

## 概述

本文档定义API设计的基本原则与RESTful规范，适用于构建清晰、一致、易用的Web服务接口。遵循这些规范可以提高API的可用性、可维护性和跨客户端兼容性。

---

## 1. RESTful核心原则

### 1.1 六条基本规范

1. **客户端-服务器分离**：客户端负责用户界面和展示，服务器负责数据和业务逻辑，二者通过接口通信
2. **无状态**：每个请求包含所有必要信息，服务器不保存客户端会话状态
3. **可缓存**：服务端响应应标明是否可缓存，提升网络效率
4. **分层系统**：客户端不需要知道服务器背后的架构细节
5. **统一接口**：所有资源通过统一的方式访问和操作
6. **按需代码（可选）**：服务器可临时扩展客户端功能

### 1.2 资源导向

REST的核心思想是：**一切皆是资源**。

```
资源示例：
- 用户：/users
- 订单：/orders
- 商品：/products

资源操作通过HTTP方法表达：
- GET /users      - 获取用户列表
- POST /users     - 创建新用户
- GET /users/123  - 获取ID为123的用户
- PUT /users/123  - 更新用户
- DELETE /users/123 - 删除用户
```

---

## 2. URL设计规范

### 2.1 URL基本格式

```
https://api.example.com/v1/{resource}/{id}/{sub-resource}
```

| 组成部分 | 说明 |
|---------|------|
| `https://api.example.com` | API基础地址 |
| `/v1` | 版本号（推荐） |
| `/{resource}` | 资源名称（复数名词） |
| `/{id}` | 资源标识符 |
| `/{sub-resource}` | 子资源（可选） |

### 2.2 命名规则

**推荐做法**：

```
✓ GET /users              # 复数名词
✓ GET /users/123/orders   # 嵌套资源
✓ GET /users?status=active&page=1  # 查询参数
```

**不推荐**：

```
✗ GET /getUsers           # 避免使用动词
✗ GET /user               # 使用复数形式
✗ GET /getUserById/123    # 避免动词前缀
```

### 2.3 嵌套资源

```bash
# 获取某个用户的所有订单
GET /users/123/orders

# 获取某个订单的所有商品
GET /orders/456/items

# 复杂场景：获取某个用户订单中包含的所有商品
GET /users/123/orders/456/items
```

### 2.4 查询参数

```bash
# 分页
GET /users?page=2&per_page=20

# 过滤
GET /products?category=electronics&min_price=100

# 排序
GET /users?sort=created_at:desc

# 搜索
GET /users?search=张三

# 字段过滤（减少返回数据量）
GET /users?fields=id,name,email
```

---

## 3. HTTP方法与状态码

### 3.1 标准HTTP方法

| 方法 | 用途 | 幂等性 | 安全性 |
|------|------|--------|--------|
| GET | 获取资源 | ✓ | ✓ |
| POST | 创建资源 | ✗ | ✗ |
| PUT | 完整更新资源 | ✓ | ✗ |
| PATCH | 部分更新资源 | ✓ | ✗ |
| DELETE | 删除资源 | ✓ | ✗ |

### 3.2 状态码分类

```
1xx - 信息响应
2xx - 成功
3xx - 重定向
4xx - 客户端错误
5xx - 服务端错误
```

### 3.3 常用状态码

| 状态码 | 含义 | 适用场景 |
|--------|------|----------|
| 200 OK | 请求成功 | GET/PUT/PATCH 成功 |
| 201 Created | 资源创建成功 | POST 创建新资源 |
| 204 No Content | 无内容返回 | DELETE 成功，无返回体 |
| 400 Bad Request | 请求格式错误 | 参数校验失败 |
| 401 Unauthorized | 未认证 | 缺少或无效token |
| 403 Forbidden | 无权限 | token有效但无权限 |
| 404 Not Found | 资源不存在 | 访问不存在的资源 |
| 409 Conflict | 资源冲突 | 重复创建、数据冲突 |
| 422 Unprocessable Entity | 验证错误 | 数据格式正确但业务逻辑错误 |
| 429 Too Many Requests | 请求过多 | 限流触发 |
| 500 Internal Server Error | 服务器错误 | 未知异常 |

### 3.4 响应示例

```json
// 200 OK - 成功获取资源
{
  "data": {
    "id": "123",
    "name": "张三",
    "email": "zhangsan@example.com"
  }
}

// 201 Created - 创建成功
{
  "data": {
    "id": "456",
    "name": "李四",
    "email": "lisi@example.com"
  },
  "message": "用户创建成功"
}

// 400 Bad Request - 请求错误
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数校验失败",
    "details": [
      {"field": "email", "message": "邮箱格式不正确"}
    ]
  }
}

// 404 Not Found - 资源不存在
{
  "error": {
    "code": "NOT_FOUND",
    "message": "用户不存在"
  }
}
```

---

## 4. 请求与响应格式

### 4.1 请求格式

**请求头（Headers）**：

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
X-Request-ID: <uuid>        # 可选，用于链路追踪
X-API-Key: <api-key>         # 部分API使用
```

**请求体（Body）**：

```json
// POST /users - 创建用户
{
  "name": "张三",
  "email": "zhangsan@example.com",
  "password": "securePassword123"
}

// PUT /users/123 - 完整更新
{
  "name": "张三（已更新）",
  "email": "zhangsan_updated@example.com"
}

// PATCH /users/123 - 部分更新
{
  "name": "张三（仅更新名称）"
}
```

### 4.2 响应格式

**标准成功响应**：

```json
{
  "data": { },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

**列表响应**：

```json
{
  "data": [
    {"id": "1", "name": "用户1"},
    {"id": "2", "name": "用户2"}
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

**错误响应**：

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "友好的错误描述",
    "details": []
  },
  "request_id": "abc123"
}
```

---

## 5. 版本控制

### 5.1 版本策略

推荐使用URL路径版本：

```
✓ https://api.example.com/v1/users
✓ https://api.example.com/v2/users
```

不推荐的方式：

```
✗ Header中版本：Accept: application/vnd.api+json;version=2
✗ Query参数：/users?version=2
```

### 5.2 版本演进规则

1. **向后兼容**：v2应尽量兼容v1的调用方式
2. **渐进废弃**：旧版本提供合理的弃用周期（如6-12个月）
3. **明确告知**：在响应头或文档中标注弃用信息

```http
# 响应头中标记弃用
Deprecation: true
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

---

## 6. 认证与授权

### 6.1 常用认证方式

| 方式 | 适用场景 | 说明 |
|------|---------|------|
| API Key | 内部/简单场景 | 简单密钥验证 |
| Bearer Token (JWT) | 通用场景 | 无状态令牌 |
| OAuth 2.0 | 第三方授权 | 授权码、客户端凭证等 |
| Basic Auth | 极少使用 | 用户名密码编码 |

### 6.2 Bearer Token示例

```http
# 请求
GET /api/v1/users HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# 响应 - 认证失败
HTTP/1.1 401 Unauthorized
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "token已过期，请重新登录"
  }
}
```

### 6.3 权限控制

```json
// 403 Forbidden - 无权限访问
{
  "error": {
    "code": "FORBIDDEN",
    "message": "您没有权限执行此操作"
  }
}
```

---

## 7. 分页、排序与过滤

### 7.1 分页实现

```bash
# 基于页码的分页
GET /users?page=2&per_page=20

# 基于游标的分页（更适合大数据集）
GET /users?cursor=eyJpZCI6MTIzfQ&limit=20

# 基于偏移的分页
GET /users?offset=40&limit=20
```

**响应中的分页元数据**：

```json
{
  "data": [...],
  "meta": {
    "page": 2,
    "per_page": 20,
    "total": 100,
    "total_pages": 5,
    "has_next": true,
    "has_prev": true
  }
}
```

### 7.2 排序

```bash
# 单字段排序
GET /users?sort=created_at

# 多字段排序
GET /users?sort=created_at:desc,name:asc

# 排序字段白名单（安全实践）
# 不允许客户端指定任意排序字段，由服务端定义可排序字段
```

### 7.3 过滤

```bash
# 精确匹配
GET /users?status=active

# 范围查询
GET /orders?created_at_min=2026-01-01&created_at_max=2026-12-31

# 多值匹配
GET /users?role=admin&role=super_admin

# 模糊搜索
GET /users?search=张三
```

---

## 8. 缓存策略

### 8.1 HTTP缓存头

```http
# 缓存控制
Cache-Control: max-age=3600, public
Cache-Control: no-cache, no-store, must-revalidate

# 条件请求 - Last-Modified
Last-Modified: Wed, 21 Oct 2026 07:28:00 GMT
If-Modified-Since: Wed, 21 Oct 2026 07:28:00 GMT

# 条件请求 - ETag
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### 8.2 缓存响应示例

```http
# 304 Not Modified - 缓存命中
HTTP/1.1 304 Not Modified
Cache-Control: max-age=86400

# 200 OK - 新数据
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "abc123"
```

---

## 9. 限流与错误处理

### 9.1 限流响应

```http
# 429 Too Many Requests
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1700000000

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "请求过于频繁，请稍后再试",
    "retry_after": 60
  }
}
```

### 9.2 统一错误格式

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "用户可读的错误消息",
    "details": [
      {
        "field": "email",
        "message": "邮箱格式不正确",
        "code": "INVALID_FORMAT"
      }
    ],
    "request_id": "req_abc123xyz"
  }
}
```

### 9.3 常见错误码

| 错误码 | HTTP状态码 | 说明 |
|--------|------------|------|
| VALIDATION_ERROR | 400 | 请求参数校验失败 |
| INVALID_CREDENTIALS | 401 | 认证信息无效 |
| TOKEN_EXPIRED | 401 | Token已过期 |
| FORBIDDEN | 403 | 无权限访问 |
| NOT_FOUND | 404 | 资源不存在 |
| METHOD_NOT_ALLOWED | 405 | HTTP方法不支持 |
| RESOURCE_CONFLICT | 409 | 资源冲突 |
| RATE_LIMIT_EXCEEDED | 429 | 请求超限 |
| INTERNAL_ERROR | 500 | 服务器内部错误 |
| SERVICE_UNAVAILABLE | 503 | 服务暂时不可用 |

---

## 10. 速率限制（Rate Limiting）

### 10.1 常见限制策略

| 策略 | 说明 |
|------|------|
| 请求数/时间窗口 | 如 1000请求/分钟 |
| 并发连接数 | 如 最多100个并发连接 |
| 资源配额 | 如 每天10万次API调用 |

### 10.2 客户端应对策略

```python
# 示例：指数退避重试
import time
import random

def retry_with_backoff(max_retries=5):
    for attempt in range(max_retries):
        response = make_request()
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 1))
            wait_time = retry_after * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait_time)
        elif response.status_code >= 500:
            time.sleep(2 ** attempt)
        else:
            return response
```

---

## 11. 安全性最佳实践

1. **始终使用HTTPS**：所有API通信必须加密
2. **验证所有输入**：参数校验、SQL注入防护、XSS防护
3. **敏感数据保护**：密码、密钥等不可明文返回
4. **限制响应数据**：使用fields参数控制返回字段
5. **CORS配置**：合理配置跨域访问策略
6. **请求大小限制**：防止大请求耗尽服务器资源
7. **日志与监控**：记录请求日志，监控异常行为

---

## 12. 文档规范

### 12.1 API文档应包含

- 基础URL与版本信息
- 认证方式与请求头
- 所有端点的详细说明
- 请求/响应示例
- 错误码说明
- 速率限制说明

### 12.2 OpenAPI示例

```yaml
openapi: 3.0.0
info:
  title: 用户API
  version: 1.0.0
paths:
  /users:
    get:
      summary: 获取用户列表
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
```

---

## 13. 关联

---

*最后更新：2026-04-21*
