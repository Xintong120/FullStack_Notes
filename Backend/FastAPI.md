# 1 路由和端点

> - **端点 (Endpoint)**：API 中一个具体的 URL 地址，如 `https://api.autogpt.com/graphs/123`。它代表了一个可调用的资源或功能。
> - **路由 (Route)**：将 HTTP 请求（URL + 方法）映射到处理函数的过程。在 FastAPI 中，通过装饰器定义路由。
>
> 路由是 FastAPI 的核心，它决定：
>
> - 哪个 URL 被哪个函数处理
> - 接受哪些 HTTP 方法（GET, POST 等）
> - 如何提取请求中的数据（参数、请求体等）
>
> **路由映射 = 按钮动作 → 业务逻辑**
>
> - **前端按钮**：触发 HTTP 请求
> - **路由装饰器**：告诉 FastAPI 哪个 URL 对应哪个函数
> - **业务逻辑**：执行 AI 调用、数据处理等

## 1.1 **基础复习**

### **1.1.1 HTTP 方法（HTTP Methods）**

HTTP 定义了多种请求方法，用于表示对资源的不同操作：

| 方法        | 用途           | 是否幂等 | 是否安全 | 示例             |
| ----------- | -------------- | -------- | -------- | ---------------- |
| **GET**     | 获取资源       | ✅        | ✅        | 获取用户列表     |
| **POST**    | 创建资源       | ❌        | ❌        | 创建新用户       |
| **PUT**     | 完整更新资源   | ✅        | ❌        | 更新用户所有信息 |
| **PATCH**   | 部分更新资源   | ❌        | ❌        | 只更新用户名     |
| **DELETE**  | 删除资源       | ✅        | ❌        | 删除用户         |
| **HEAD**    | 获取响应头     | ✅        | ✅        | 检查资源是否存在 |
| **OPTIONS** | 获取支持的方法 | ✅        | ✅        | CORS 预检        |

**核心概念：**
- **幂等（Idempotent）**：多次执行结果相同（GET, PUT, DELETE）
- **安全（Safe）**：不会修改服务器状态（GET, HEAD, OPTIONS）

```python
# 传统 HTTP 请求示例
import requests

# GET - 获取资源
response = requests.get('https://api.example.com/users')

# POST - 创建资源
response = requests.post('https://api.example.com/users', json={'name': 'Alice'})

# PUT - 完整更新
response = requests.put('https://api.example.com/users/1', json={'name': 'Alice', 'age': 30})

# PATCH - 部分更新
response = requests.patch('https://api.example.com/users/1', json={'age': 31})

# DELETE - 删除
response = requests.delete('https://api.example.com/users/1')
```

---

### 1.1.2 路径参数 vs 查询参数

**路径参数（Path Parameters）**

- 嵌入在 URL 路径中
- 通常用于**标识特定资源**
- 是 URL 的**必需部分**

```python
# 路径参数示例
GET /users/123          # 123 是用户ID（路径参数）
GET /posts/456/comments # 456 是文章ID
DELETE /orders/789      # 789 是订单ID

# 多个路径参数
GET /users/123/posts/456  # user_id=123, post_id=456
```

**查询参数（Query Parameters）**

- 在 URL 的 `?` 之后
- 通常用于**过滤、排序、分页**
- 是 URL 的**可选部分**

```python
# 查询参数示例
GET /users?page=2&limit=10          # 分页参数
GET /posts?author=alice&tag=python  # 过滤参数
GET /products?sort=price&order=asc  # 排序参数

# 组合使用
GET /users/123/posts?status=published&sort=date
#    ^^^^路径参数    ^^^^^^^^查询参数^^^^^^^^^
```

**对比总结：**

| 特性       | 路径参数     | 查询参数        |
| ---------- | ------------ | --------------- |
| **位置**   | URL 路径中   | `?` 之后        |
| **必需性** | 通常必需     | 通常可选        |
| **用途**   | 标识资源     | 过滤、排序      |
| **示例**   | `/users/123` | `/users?age=30` |

---

### **1.1.3 请求体（Request Body）**

请求体包含客户端发送给服务器的数据，常见格式：

**JSON 格式（最常用）**

```json
POST /users
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com",
  "age": 30,
  "tags": ["python", "fastapi"]
}
```

**表单数据（Form Data）**

```
POST /login
Content-Type: application/x-www-form-urlencoded

username=alice&password=secret123
```

**多部分表单（Multipart Form）**

```
POST /upload
Content-Type: multipart/form-data

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

[binary data]
------WebKitFormBoundary
```

---

### 1.1.4 响应状态码

HTTP 状态码表示请求的结果：

| 类别    | 范围    | 含义       | 常用状态码                                       |
| ------- | ------- | ---------- | ------------------------------------------------ |
| **1xx** | 100-199 | 信息性     | 100 Continue                                     |
| **2xx** | 200-299 | 成功       | 200 OK, 201 Created, 204 No Content              |
| **3xx** | 300-399 | 重定向     | 301 Moved, 302 Found, 304 Not Modified           |
| **4xx** | 400-499 | 客户端错误 | 400 Bad Request, 401 Unauthorized, 404 Not Found |
| **5xx** | 500-599 | 服务器错误 | 500 Internal Error, 503 Service Unavailable      |

**常用状态码详解：**

```python
# 2xx 成功
200 OK              # 请求成功（GET, PUT, PATCH）
201 Created         # 资源已创建（POST）
204 No Content      # 成功但无返回内容（DELETE）

# 4xx 客户端错误
400 Bad Request     # 请求格式错误
401 Unauthorized    # 未认证
403 Forbidden       # 无权限
404 Not Found       # 资源不存在
422 Unprocessable   # 验证失败（FastAPI 常用）

# 5xx 服务器错误
500 Internal Error  # 服务器内部错误
502 Bad Gateway     # 网关错误
503 Unavailable     # 服务不可用
```

---

## 1.2 **FastAPI 特性**

### **1.2.1 装饰器路由定义**

FastAPI 使用 Python 装饰器来定义路由，简洁优雅：

```python
from fastapi import FastAPI

app = FastAPI()

# GET 请求
@app.get("/")
def read_root():
    return {"message": "Hello World"}

# POST 请求
@app.post("/users")
def create_user():
    return {"user_id": 123}

# PUT 请求
@app.put("/users/{user_id}")
def update_user(user_id: int):
    return {"user_id": user_id, "updated": True}

# DELETE 请求
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    return {"user_id": user_id, "deleted": True}

# PATCH 请求
@app.patch("/users/{user_id}")
def partial_update_user(user_id: int):
    return {"user_id": user_id, "patched": True}
```

**为什么用装饰器？**
- 代码清晰，直观
- Python 风格，符合习惯
- 元数据收集，自动生成文档

---

### **1.2.2 自动参数提取**

FastAPI 会根据**函数签名**自动提取参数：

**路径参数（自动识别）**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    #         ^^^^^^^^ 路径参数，从 URL 中提取
    return {"user_id": user_id}

# 请求：GET /users/123
# user_id = 123（自动转换为 int）

# 多个路径参数
@app.get("/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    return {"user_id": user_id, "post_id": post_id}

# 请求：GET /users/123/posts/456
# user_id = 123, post_id = 456
```

**查询参数（自动识别）**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
def list_users(skip: int = 0, limit: int = 10):
    #          ^^^^^^^^^^^^  ^^^^^^^^^^^^^^
    #          查询参数      查询参数（有默认值）
    return {"skip": skip, "limit": limit}

# 请求：GET /users?skip=20&limit=5
# skip = 20, limit = 5

# 请求：GET /users
# skip = 0（默认值）, limit = 10（默认值）

# 可选查询参数
@app.get("/search")
def search(q: str | None = None):
    #      ^^^^^^^^^^^^^^^^^^ 可选参数
    if q:
        return {"results": f"Searching for {q}"}
    return {"results": "No query provided"}

# 请求：GET /search?q=python  → results: "Searching for python"
# 请求：GET /search           → results: "No query provided"
```

**请求体（自动解析）**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str
    age: int | None = None

@app.post("/users")
def create_user(user: User):
    #               ^^^^ 请求体，自动解析 JSON
    return {"user": user, "created": True}

# 请求：POST /users
# Body: {"name": "Alice", "email": "alice@example.com", "age": 30}
# user = User(name="Alice", email="alice@example.com", age=30)
```

**混合使用**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.put("/users/{user_id}/items/{item_id}")
def update_item(
    user_id: int,           # 路径参数
    item_id: int,           # 路径参数
    item: Item,             # 请求体
    q: str | None = None    # 查询参数
):
    return {
        "user_id": user_id,
        "item_id": item_id,
        "item": item,
        "q": q
    }

# 请求：PUT /users/123/items/456?q=search
# Body: {"name": "Widget", "price": 9.99}
# 
# user_id = 123（路径）
# item_id = 456（路径）
# item = Item(name="Widget", price=9.99)（请求体）
# q = "search"（查询）
```

**FastAPI 如何识别？**
```python
# 规则：
# 1. 在路径中声明的 → 路径参数
# 2. 是 Pydantic 模型 → 请求体
# 3. 其他的 → 查询参数
```

---

### **1.2.3 类型驱动的验证**

FastAPI 使用 Python 类型注解进行**自动验证**：

**基础类型验证**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def get_item(item_id: int):
    #                  ^^^ 类型注解
    return {"item_id": item_id}

# ✅ 请求：GET /items/123  → item_id = 123（int）
# ❌ 请求：GET /items/abc  → 422 Unprocessable Entity
# 响应：{
#   "detail": [{
#     "loc": ["path", "item_id"],
#     "msg": "value is not a valid integer",
#     "type": "type_error.integer"
#   }]
# }
```

**高级类型验证**

```python
from fastapi import FastAPI, Query, Path
from typing import Literal

app = FastAPI()

@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(..., gt=0, le=1000),
    #               ^^^^^^^^^^^^^^^^^^^^^^^^^^
    #               路径参数，必须 > 0 且 <= 1000
    
    q: str | None = Query(None, min_length=3, max_length=50),
    #              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #              查询参数，长度 3-50
    
    sort: Literal["asc", "desc"] = "asc"
    #     ^^^^^^^^^^^^^^^^^^^^^ 只能是这两个值
):
    return {"item_id": item_id, "q": q, "sort": sort}

# ✅ GET /items/100?q=python&sort=asc
# ❌ GET /items/0         → item_id must be > 0
# ❌ GET /items/100?q=ab  → q too short
# ❌ GET /items/100?sort=xxx → sort must be 'asc' or 'desc'
```

**Pydantic 模型验证**

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, EmailStr

app = FastAPI()

class User(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr  # 自动验证邮箱格式
    age: int = Field(..., ge=0, le=150)
    tags: list[str] = []

@app.post("/users")
def create_user(user: User):
    return {"user": user}

# ✅ POST /users
# Body: {"name": "Alice", "email": "alice@example.com", "age": 30}

# ❌ POST /users
# Body: {"name": "", "email": "invalid", "age": -5}
# 错误：
# - name: min_length violation
# - email: invalid email format
# - age: must be >= 0
```

---

### **1.2.4  路径操作装饰器参数**

FastAPI 的装饰器支持丰富的配置参数：

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str

@app.post(
    "/users",                          # 路径
    response_model=User,               # 响应模型
    status_code=status.HTTP_201_CREATED,  # 状态码
    tags=["users"],                    # 标签（文档分组）
    summary="Create a new user",       # 摘要
    description="Create a new user with name and email",  # 描述
    response_description="The created user",  # 响应描述
    deprecated=False,                  # 是否废弃
    # operation_id="create_user",      # 操作ID（可选）
    # responses={...}                  # 额外响应定义
)
def create_user(user: User):
    """
    Create a new user:
    
    - **name**: User's full name
    - **email**: User's email address
    """
    return user

# 访问文档：http://localhost:8000/docs
# 可以看到所有这些信息！
```

**常用参数详解：**

| 参数                                                         | 类型 | 作用         | 示例                                 |
| ------------------------------------------------------------ | ---- | ------------ | ------------------------------------ |
| `response_model`                                             | Type | 响应数据模型 | `response_model=User`                |
| `status_code`                                                | int  | HTTP 状态码  | `status_code=201`                    |
| `tags`                                                       | list | 文档标签     | `tags=["users"]`                     |
| `summary`                                                    | str  | 简短摘要     | `summary="Get user"`                 |
| `description`                                                | str  | 详细描述     | `description="..."`                  |
| `response_description`                                       | str  | 响应描述     | `response_description="User object"` |
| `deprecated`                                                 | bool | 是否废弃     | `deprecated=True`                    |
| [name](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/block.py:476:4-478:38) | str  | 操作名称     | `name="get_user"`                    |
| `responses`                                                  | dict | 多种响应     | `responses={404: {...}}`             |

---

### **1.2.5 自动参数验证错误处理**

FastAPI 自动生成清晰的验证错误：

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(..., min_length=3)
    price: float = Field(..., gt=0)
    quantity: int = Field(..., ge=1)

@app.post("/items")
def create_item(item: Item):
    return item

# ❌ POST /items
# Body: {"name": "ab", "price": -5, "quantity": 0}
# 
# 响应：422 Unprocessable Entity
# {
#   "detail": [
#     {
#       "type": "string_too_short",
#       "loc": ["body", "name"],
#       "msg": "String should have at least 3 characters",
#       "input": "ab",
#       "ctx": {"min_length": 3}
#     },
#     {
#       "type": "greater_than",
#       "loc": ["body", "price"],
#       "msg": "Input should be greater than 0",
#       "input": -5,
#       "ctx": {"gt": 0}
#     },
#     {
#       "type": "greater_than_equal",
#       "loc": ["body", "quantity"],
#       "msg": "Input should be greater than or equal to 1",
#       "input": 0,
#       "ctx": {"ge": 1}
#     }
#   ]
# }
```

**错误响应结构：**
- `type`: 错误类型
- `loc`: 错误位置（path, query, body）
- `msg`: 错误消息
- `input`: 输入值
- `ctx`: 上下文信息

---

## 1.3 **对比：传统方式 vs FastAPI**

**Flask 传统方式**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    # 手动验证
    if not isinstance(user_id, int) or user_id <= 0:
        return jsonify({"error": "Invalid user_id"}), 400
    
    # 手动获取查询参数
    page = request.args.get('page', default=1, type=int)
    if page < 1:
        return jsonify({"error": "Invalid page"}), 400
    
    # 业务逻辑
    return jsonify({"user_id": user_id, "page": page})
```

**FastAPI 方式**

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(
    user_id: int = Path(..., gt=0),
    page: int = Query(1, ge=1)
):
    # 自动验证，直接使用
    return {"user_id": user_id, "page": page}
```

**FastAPI 优势：**
- ✅ 更少的代码
- ✅ 自动验证
- ✅ 自动文档
- ✅ 类型提示
- ✅ 错误处理

---

## 1.4 **核心要点总结**

### **1.4.1 HTTP 基础**
- 理解不同 HTTP 方法的用途和特性
- 区分路径参数和查询参数
- 掌握常用 HTTP 状态码

### **1.4.2 FastAPI 路由**
- 使用装饰器定义路由（[@app.get()](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/util/request.py:490:4-491:62), [@app.post()](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/util/request.py:493:4-494:63) 等）
- FastAPI 自动识别参数类型（路径/查询/请求体）
- 利用类型注解进行自动验证

### **1.4.3 参数提取**
```python
@app.put("/users/{user_id}/items/{item_id}")  # 路径参数
def update_item(
    user_id: int,                    # 路径参数
    item_id: int,                    # 路径参数
    item: Item,                      # 请求体（Pydantic模型）
    q: str | None = None             # 查询参数
):
    pass
```

### **1.4.4 自动验证**
- 类型转换和验证
- 清晰的错误消息
- 422 状态码表示验证失败

## 1.5 **AutoGPT 中的路由实战分析**

---

### **1.5.1 路由器组织结构**

```python
# backend/server/routers/v1.py
from fastapi import APIRouter

# 创建 v1 版本的路由器
v1_router = APIRouter()

# 包含子路由器（模块化组织）
v1_router.include_router(
    backend.server.integrations.router,
    prefix="/integrations",          # 路由前缀
    tags=["integrations"],           # 文档标签
)

v1_router.include_router(
    backend.server.routers.analytics.router,
    prefix="/analytics",
    tags=["analytics"],
    dependencies=[Security(requires_user)],  # 全局依赖（认证）
)
```

**设计要点：**
- 🏗️ 使用 `APIRouter` 而不是直接用 `app`（可复用）
- 📁 按功能模块拆分路由（integrations, analytics, graphs 等）
- 🔒 在路由器级别添加通用依赖（如认证）
- 🏷️ 使用 `tags` 组织 API 文档

---

### **1.5.2 GET 请求实战**

#### **案例 A：简单 GET（获取用户积分）**

```python
@v1_router.get(
    path="/credits",
    tags=["credits"],
    summary="Get user credits",
    dependencies=[Security(requires_user)],
)
async def get_user_credits(
    user_id: Annotated[str, Security(get_user_id)],
    #       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #       使用 Annotated 注入认证的用户ID
) -> dict[str, int]:
    return {"credits": await _user_credit_model.get_credits(user_id)}
```

**关键特性：**
- 返回类型提示 `-> dict[str, int]`
- 依赖注入获取 `user_id`（从 JWT token 中提取）
- 异步函数 `async def`
- 清晰的 `summary`（自动生成文档）

---

#### **案例 B：带路径参数的 GET**

```python
@v1_router.get(
    path="/graphs/{graph_id}",
    summary="Get specific graph",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def get_graph(
    graph_id: str,                              # 路径参数
    user_id: Annotated[str, Security(get_user_id)],  # 依赖注入
    version: int | None = None,                  # 可选查询参数
    for_export: bool = False,                    # 查询参数（默认值）
) -> graph_db.GraphModel:
    graph = await graph_db.get_graph(
        graph_id,
        version,
        user_id=user_id,
        for_export=for_export,
    )
    if not graph:
        raise HTTPException(404, detail=f"Graph #{graph_id} not found")
    return graph
```

**请求示例：**
```bash
# 获取图的活跃版本
GET /graphs/abc123

# 获取特定版本
GET /graphs/abc123?version=2

# 导出模式
GET /graphs/abc123?version=2&for_export=true
```

**关键点：**
- 路径参数 `{graph_id}` 自动提取
- 可选查询参数 `version: int | None`
- 手动抛出 `HTTPException` 处理 404

---

#### **案例 C：多个路径参数（嵌套资源）**

```python
@v1_router.get(
    path="/graphs/{graph_id}/versions/{version}",
    summary="Get graph version",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def get_graph_version(
    graph_id: str,           # 第一个路径参数
    version: int,            # 第二个路径参数
    user_id: Annotated[str, Security(get_user_id)],
    for_export: bool = False,
) -> graph_db.GraphModel:
    # 实现逻辑...
    pass
```

**请求示例：**
```bash
GET /graphs/abc123/versions/3
GET /graphs/abc123/versions/3?for_export=true
```

---

#### **案例 D：带复杂查询参数的 GET**

```python
@v1_router.get(
    path="/credits/transactions",
    tags=["credits"],
    summary="Get credit history",
    dependencies=[Security(requires_user)],
)
async def get_credit_history(
    user_id: Annotated[str, Security(get_user_id)],
    transaction_time: datetime | None = None,     # 可选时间过滤
    transaction_type: str | None = None,          # 可选类型过滤
    transaction_count_limit: int = 100,           # 分页限制
) -> TransactionHistory:
    # 参数验证
    if transaction_count_limit < 1 or transaction_count_limit > 1000:
        raise ValueError("Transaction count limit must be between 1 and 1000")
    
    return await _user_credit_model.get_transaction_history(
        user_id=user_id,
        transaction_time_ceiling=transaction_time,
        transaction_count_limit=transaction_count_limit,
        transaction_type=transaction_type,
    )
```

**请求示例：**
```bash
# 默认查询
GET /credits/transactions

# 过滤查询
GET /credits/transactions?transaction_type=purchase&transaction_count_limit=50

# 时间过滤
GET /credits/transactions?transaction_time=2024-01-01T00:00:00Z
```

---

### **1.5.3 POST 请求实战**

#### **案例 A：创建资源（带请求体）**

```python
@v1_router.post(
    path="/graphs",
    summary="Create new graph",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def create_new_graph(
    create_graph: CreateGraph,  # 请求体（Pydantic 模型）
    user_id: Annotated[str, Security(get_user_id)],
) -> graph_db.GraphModel:
    # 转换为内部模型
    graph = graph_db.make_graph_model(
        create_graph.graph,
        user_id,
        reassign_ids=True
    )
    
    # 验证图结构
    graph.validate_graph(for_run=False)
    
    # 创建并激活
    await graph_db.create_graph(graph, user_id=user_id)
    return await on_graph_activate(graph, user_id=user_id)
```

**CreateGraph 模型：**
```python
# backend/server/model.py
from pydantic import BaseModel

class CreateGraph(BaseModel):
    graph: graph_db.Graph
    credentials_inputs: dict[str, CredentialsMetaInput] = {}
```

**请求示例：**
```bash
POST /graphs
Content-Type: application/json

{
  "graph": {
    "name": "My Agent",
    "description": "...",
    "nodes": [...],
    "links": [...]
  },
  "credentials_inputs": {}
}
```

---

#### **案例 B：带路径参数的 POST**

```python
@v1_router.post(
    path="/blocks/{block_id}/execute",
    summary="Execute graph block",
    tags=["blocks"],
    dependencies=[Security(requires_user)],
)
async def execute_graph_block(
    block_id: str,                               # 路径参数
    data: BlockInput,                            # 请求体
    user_id: Annotated[str, Security(get_user_id)],
) -> CompletedBlockOutput:
    # 获取 Block 实例
    obj = get_block(block_id)
    if not obj:
        raise HTTPException(
            status_code=404, 
            detail=f"Block #{block_id} not found."
        )
    
    # 获取用户上下文
    user = await get_user_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=404,
            detail=f"User #{user_id} not found."
        )
    
    # 执行 Block
    output = await obj.execute(data, user_context=UserContext(user=user))
    
    # 记录执行
    record_block_execution(block_id, user_id, success=True)
    
    return output
```

**请求示例：**
```bash
POST /blocks/text-generator/execute
Content-Type: application/json

{
  "input_data": {
    "prompt": "Hello world",
    "model": "gpt-4"
  }
}
```

---

#### **案例 C：Body 参数精细控制**

```python
@v1_router.post(
    "/auth/user/preferences",
    summary="Update notification preferences",
    tags=["auth"],
    dependencies=[Security(requires_user)],
)
async def update_preferences(
    user_id: Annotated[str, Security(get_user_id)],
    preferences: NotificationPreferenceDTO = Body(...),
    #                                        ^^^^^^^^
    #                                        明确指定为请求体
) -> NotificationPreference:
    output = await update_user_notification_preference(user_id, preferences)
    return output
```

**为什么用 `Body(...)`？**

- 明确标记这是请求体参数
- 可以配置额外选项（embed, alias 等）
- 生成更清晰的 OpenAPI 文档

---

### **1.5.4 PUT 请求实战（完整更新）**

```python
@v1_router.put(
    path="/graphs/{graph_id}",
    summary="Update graph version",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def update_graph(
    graph_id: str,                              # 路径参数
    graph: graph_db.Graph,                      # 请求体（完整的图对象）
    user_id: Annotated[str, Security(get_user_id)],
) -> graph_db.GraphModel:
    # 验证 ID 匹配
    if graph.id and graph.id != graph_id:
        raise HTTPException(400, detail="Graph ID does not match ID in URI")
    
    # 获取现有版本
    existing_versions = await graph_db.get_graph_all_versions(
        graph_id, user_id=user_id
    )
    if not existing_versions:
        raise HTTPException(404, detail=f"Graph #{graph_id} not found")
    
    # 创建新版本
    latest_version_number = max(g.version for g in existing_versions)
    graph.version = latest_version_number + 1
    
    # 验证并保存
    graph = graph_db.make_graph_model(graph, user_id)
    graph.validate_graph(for_run=False)
    new_graph_version = await graph_db.create_graph(graph, user_id=user_id)
    
    return new_graph_version
```

**PUT vs POST 区别：**
- **PUT**: 完整更新已存在的资源，幂等
- **POST**: 创建新资源或部分操作，非幂等

---

### **1.5.5 PATCH 请求实战（部分更新）**

```python
@v1_router.patch(
    "/onboarding",
    summary="Update onboarding progress",
    tags=["onboarding"],
    dependencies=[Security(requires_user)],
)
async def update_onboarding(
    user_id: Annotated[str, Security(get_user_id)],
    data: UserOnboardingUpdate,  # 只包含要更新的字段
):
    return await update_user_onboarding(user_id, data)
```

**UserOnboardingUpdate 模型（只包含部分字段）：**
```python
class UserOnboardingUpdate(BaseModel):
    onboarding_status: str | None = None
    current_step: int | None = None
    completed_steps: list[str] | None = None
```

**PATCH vs PUT：**
- **PATCH**: 部分更新，只发送改变的字段
- **PUT**: 完整更新，发送所有字段

---

### **1.5.6 DELETE 请求实战**

```python
@v1_router.delete(
    path="/graphs/{graph_id}",
    summary="Delete graph permanently",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def delete_graph(
    graph_id: str,                              # 路径参数
    user_id: Annotated[str, Security(get_user_id)],
) -> DeleteGraphResponse:
    # 获取活跃版本并停用
    if active_version := await graph_db.get_graph(graph_id, user_id=user_id):
        await on_graph_deactivate(active_version, user_id=user_id)
    
    # 删除所有版本
    deleted_count = await graph_db.delete_graph(graph_id, user_id=user_id)
    
    return {"version_counts": deleted_count}
```

**关键点：**
- 🗑️ DELETE 通常返回删除的资源信息
- ⚙️ 先清理相关资源（停用、解绑等）
- 📊 返回操作统计（删除数量等）

---

### **1.5.7 文件上传实战**

```python
@v1_router.post(
    path="/files/upload",
    summary="Upload file",
    tags=["files"],
    dependencies=[Security(requires_user)],
)
async def upload_file(
    user_id: Annotated[str, Security(get_user_id)],
    file: UploadFile = File(...),              # 文件上传
    #                 ^^^^^^^^^ File() 标记
    provider: str = "gcs",                     # 查询参数
    expiration_hours: int = 24,                # 查询参数
) -> UploadFileResponse:
    # 验证过期时间
    if expiration_hours < 1 or expiration_hours > 48:
        raise HTTPException(
            status_code=400,
            detail="Expiration hours must be between 1 and 48"
        )
    
    # 检查文件大小
    max_size_mb = settings.config.upload_file_size_limit_mb
    max_size_bytes = max_size_mb * 1024 * 1024
    
    if hasattr(file, "size") and file.size and file.size > max_size_bytes:
        raise HTTPException(
            status_code=400,
            detail=f"File size exceeds {max_size_mb}MB limit"
        )
    
    # 读取文件内容
    content = await file.read()
    content_size = len(content)
    
    # 病毒扫描
    await scan_content_safe(content, filename=file.filename)
    
    # 存储到云端
    cloud_storage = await get_cloud_storage_handler()
    storage_path = await cloud_storage.store_file(
        content=content,
        filename=file.filename,
        provider=provider,
        expiration_hours=expiration_hours,
        user_id=user_id,
    )
    
    return UploadFileResponse(
        file_uri=storage_path,
        file_name=file.filename,
        size=content_size,
        content_type=file.content_type,
        expires_in_hours=expiration_hours,
    )
```

**请求示例（Multipart Form）：**
```bash
POST /files/upload?provider=gcs&expiration_hours=48
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="file"; filename="document.pdf"
Content-Type: application/pdf

[binary data]
--boundary--
```

---

### **1.5.8 装饰器参数的高级配置**

#### **案例：自定义响应定义**

```python
@v1_router.get(
    path="/blocks",
    summary="List available blocks",
    tags=["blocks"],
    dependencies=[Security(requires_user)],
    responses={
        200: {
            "description": "Successful Response",
            "content": {
                "application/json": {
                    "schema": {
                        "items": {"additionalProperties": True, "type": "object"},
                        "type": "array",
                        "title": "Response Get List Available Blocks",
                    }
                }
            },
        }
    },
)
async def get_graph_blocks() -> Response:
    # 获取缓存的 blocks 数据
    content = await _get_cached_blocks()
    
    # 返回原始 JSON 字符串（性能优化）
    return Response(
        content=content,
        media_type="application/json",
    )
```

**为什么自定义 responses？**
- 📄 手动控制 OpenAPI schema
- ⚡ 返回预序列化的 JSON（性能优化）
- 📝 提供更详细的文档

---

### **1.5.9 认证和权限模式**

AutoGPT 使用 **依赖注入** 实现认证：

```python
from autogpt_libs.auth import get_user_id, requires_user
from fastapi import Security
from typing import Annotated

# 模式 1：仅要求认证（不关心用户ID）
@v1_router.get(
    "/graphs",
    dependencies=[Security(requires_user)],  # 全局依赖
)
async def list_graphs():
    pass

# 模式 2：需要用户ID
@v1_router.get(
    "/credits",
    dependencies=[Security(requires_user)],
)
async def get_credits(
    user_id: Annotated[str, Security(get_user_id)],  # 注入用户ID
    #       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
):
    return {"credits": await get_user_credits(user_id)}

# 模式 3：组合多个安全依赖
@v1_router.post(
    "/admin/users",
    dependencies=[
        Security(requires_user),
        Security(requires_admin),  # 额外的权限检查
    ],
)
async def admin_operation():
    pass
```

**依赖执行顺序：**
1. `requires_user` 验证 JWT token
2. `get_user_id` 从 token 提取 user_id
3. 业务逻辑使用 user_id

---

### **1.5.10 错误处理模式**

```python
from fastapi import HTTPException

@v1_router.get("/graphs/{graph_id}")
async def get_graph(graph_id: str, user_id: str):
    # 模式 1：资源不存在
    graph = await graph_db.get_graph(graph_id, user_id=user_id)
    if not graph:
        raise HTTPException(
            status_code=404,
            detail=f"Graph #{graph_id} not found"
        )
    
    # 模式 2：参数验证失败
    if expiration_hours < 1 or expiration_hours > 48:
        raise HTTPException(
            status_code=400,
            detail="Expiration hours must be between 1 and 48"
        )
    
    # 模式 3：权限检查
    if graph.user_id != user_id:
        raise HTTPException(
            status_code=403,
            detail="Access denied"
        )
    
    # 模式 4：业务逻辑错误
    if graph.id and graph.id != graph_id:
        raise HTTPException(
            status_code=400,
            detail="Graph ID does not match ID in URI"
        )
    
    return graph
```

---

## 1.6 **AutoGPT 的路由设计模式总结**

### **1.6.1 一致的路由结构**
```
/资源复数名/{资源ID}[/子资源复数名/{子资源ID}]

示例：
/graphs                          # 列表
/graphs/{graph_id}               # 单个资源
/graphs/{graph_id}/versions/{version}  # 嵌套资源
/blocks/{block_id}/execute       # 资源操作
```

### **1.6.2 标准化的参数使用**
```python
# 路径参数：标识特定资源
graph_id: str 
block_id: str
version: int

# 查询参数：过滤、排序、分页
page: int = 1
limit: int = 10
filter_by: str = "active"

# 请求体：复杂数据结构
graph: Graph
data: BlockInput
```

### **1.6.3 依赖注入模式**
```python
# 始终使用 Annotated + Security
user_id: Annotated[str, Security(get_user_id)]

# 不要直接使用
user_id: str = Depends(get_user_id)  # ❌ 老式写法
```

### **1.6.4 一致的错误处理**
```python
# 404: 资源不存在
if not resource:
    raise HTTPException(404, detail=f"Resource #{id} not found")

# 400: 请求参数错误
if invalid_param:
    raise HTTPException(400, detail="Invalid parameter")

# 403: 无权限
if not authorized:
    raise HTTPException(403, detail="Access denied")
```

### **1.6.5 清晰的文档标注**
```python
@v1_router.post(
    path="/...",
    summary="简短摘要",              # 必须
    tags=["功能分类"],              # 必须
    dependencies=[...],             # 如需认证
    responses={...},                # 如有自定义响应
)
```

---

# 2 Pydantic 模型集成

## 2.1**基础复习**

### **2.1.1 Pydantic BaseModel 基础**

Pydantic 是 Python 的数据验证库，通过类型注解自动验证和转换数据。

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None  # 可选字段

# 创建实例（自动验证）
user = User(name="Alice", age=30)
print(user)  # name='Alice' age=30 email=None

# 自动类型转换
user = User(name="Bob", age="25")  # "25" 自动转为 int
print(user.age)  # 25 (int)

# 验证失败
try:
    user = User(name="Charlie", age="invalid")
except Exception as e:
    print(e)  # ValidationError
```

---

### **2.1.2 字段类型和验证**

**基础类型**

```python
from pydantic import BaseModel
from datetime import datetime

class Product(BaseModel):
    # 基础类型
    name: str
    price: float
    quantity: int
    is_available: bool
    
    # 日期时间
    created_at: datetime
    
    # 可选类型
    description: str | None = None
    
    # 列表和字典
    tags: list[str] = []
    metadata: dict[str, any] = {}
```

**联合类型和字面量**

```python
from pydantic import BaseModel
from typing import Literal

class Config(BaseModel):
    # 联合类型（多种可能）
    port: int | str  # 可以是 int 或 str
    
    # 字面量（限定值）
    env: Literal["dev", "staging", "prod"]
    
    # 可选联合类型
    log_level: Literal["debug", "info", "error"] | None = "info"

# ✅ 正确
config = Config(port=8000, env="prod")
config = Config(port="8000", env="dev")  # port 自动转为 str

# ❌ 错误
config = Config(port=8000, env="test")  # env 必须是 dev/staging/prod
```

---

### **2.1.3 Field() 配置**

`Field()` 用于添加验证规则和元数据：

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    # 基础配置
    name: str = Field(..., min_length=1, max_length=100)
    #           ^^^
    #           ... 表示必需字段
    
    # 数值验证
    age: int = Field(..., ge=0, le=150)
    #                     ^^  ^^^
    #                     >=  <=
    
    # 字符串长度
    password: str = Field(..., min_length=8)
    
    # 默认值和描述
    role: str = Field(
        default="user",
        description="User role in the system"
    )
    
    # 别名（JSON 字段名不同）
    user_id: str = Field(..., alias="userId")
    
    # 示例值（用于文档）
    email: str = Field(
        ...,
        example="user@example.com"
    )

# 使用
user = User(
    name="Alice",
    age=30,
    password="secret123",
    userId="abc123"  # 注意：使用 alias
)
print(user.user_id)  # "abc123"
```

**Field() 常用参数：**

| 参数              | 类型     | 说明           | 示例                     |
| ----------------- | -------- | -------------- | ------------------------ |
| `default`         | any      | 默认值         | `default="user"`         |
| `default_factory` | callable | 默认值工厂函数 | `default_factory=list`   |
| `alias`           | str      | JSON 字段别名  | `alias="userId"`         |
| `title`           | str      | 标题           | `title="User Name"`      |
| `description`     | str      | 描述           | `description="用户年龄"` |
| `example`         | any      | 示例值         | `example=30`             |
| `gt` / `ge`       | number   | 大于/大于等于  | `ge=0`                   |
| `lt` / `le`       | number   | 小于/小于等于  | `le=150`                 |
| `min_length`      | int      | 最小长度       | `min_length=1`           |
| `max_length`      | int      | 最大长度       | `max_length=100`         |
| `pattern`         | str      | 正则表达式     | `pattern=r"^\d+$"`       |

---

### **2.1.4 模型继承**

```python
from pydantic import BaseModel, Field

# 基础模型
class BaseUser(BaseModel):
    name: str
    email: str
    created_at: datetime

# 继承并扩展
class AdminUser(BaseUser):
    role: str = "admin"
    permissions: list[str] = []

# 继承并覆盖
class PublicUser(BaseUser):
    # 覆盖字段（更严格的验证）
    name: str = Field(..., min_length=3)
    
    # 不暴露 email
    model_config = {"exclude": {"email"}}

# 使用
admin = AdminUser(
    name="Admin",
    email="admin@example.com",
    created_at=datetime.now(),
    permissions=["read", "write"]
)
```

---

### **2.1.5 嵌套模型**

```python
from pydantic import BaseModel

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address  # 嵌套模型

class User(BaseModel):
    name: str
    company: Company  # 嵌套模型

# 使用
user = User(
    name="Alice",
    company={
        "name": "TechCorp",
        "address": {
            "street": "123 Main St",
            "city": "New York",
            "country": "USA"
        }
    }
)

print(user.company.address.city)  # "New York"
```

---

### **2.1.6 模型配置（ConfigDict）**

```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    model_config = ConfigDict(
        # 允许额外字段
        extra="allow",  # "forbid" | "ignore" | "allow"
        
        # 字段验证
        validate_assignment=True,  # 赋值时也验证
        
        # 允许任意类型
        arbitrary_types_allowed=True,
        
        # JSON Schema
        json_schema_extra={
            "examples": [{
                "name": "Alice",
                "age": 30
            }]
        },
        
        # 别名生成器
        alias_generator=lambda field_name: field_name.replace("_", "-"),
        
        # 填充默认值
        populate_by_name=True,  # 允许使用字段名或别名
    )
    
    name: str
    age: int

# extra="allow" 示例
user = User(name="Alice", age=30, extra_field="value")
print(user.extra_field)  # "value"（允许额外字段）
```

---

### **2.1.7 模型方法**

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

user = User(name="Alice", age=30)

# 转为字典
user_dict = user.model_dump()
# {"name": "Alice", "age": 30}

# 排除字段
user_dict = user.model_dump(exclude={"age"})
# {"name": "Alice"}

# 只包含特定字段
user_dict = user.model_dump(include={"name"})
# {"name": "Alice"}

# 排除未设置的字段
user_dict = user.model_dump(exclude_unset=True)

# 转为 JSON 字符串
user_json = user.model_dump_json()
# '{"name":"Alice","age":30}'

# 从字典创建
user = User.model_validate({"name": "Bob", "age": 25})

# 从 JSON 创建
user = User.model_validate_json('{"name":"Bob","age":25}')

# 获取 JSON Schema
schema = User.model_json_schema()
```

---

## 2.2 **FastAPI 特性**

### **2.2.1 请求体自动验证**

FastAPI 自动使用 Pydantic 模型验证请求体：

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class CreateUserRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    age: int = Field(..., ge=0, le=150)

@app.post("/users")
async def create_user(user: CreateUserRequest):
    #                    ^^^^ Pydantic 模型
    #                         自动验证请求体
    return {"user": user}

# ✅ 正确请求
# POST /users
# {"name": "Alice", "email": "alice@example.com", "age": 30}
# 响应：{"user": {"name": "Alice", "email": "alice@example.com", "age": 30}}

# ❌ 错误请求（验证失败）
# POST /users
# {"name": "", "email": "invalid", "age": -5}
# 响应：422 Unprocessable Entity
# {
#   "detail": [
#     {"loc": ["body", "name"], "msg": "String should have at least 1 character"},
#     {"loc": ["body", "email"], "msg": "String should match pattern"},
#     {"loc": ["body", "age"], "msg": "Input should be greater than or equal to 0"}
#   ]
# }
```

**关键点：**
-  自动验证所有字段
-  自动类型转换
-  返回清晰的错误消息（422 状态码）
-  错误定位到具体字段

---

### **2.2.2 response_model 参数**

使用 `response_model` 控制响应的序列化：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str
    password: str  # 敏感字段

class UserPublic(BaseModel):
    name: str
    email: str
    # 不包含 password

@app.post("/users", response_model=UserPublic)
#                   ^^^^^^^^^^^^^^^^^^^^^^^^
#                   只返回 UserPublic 字段
async def create_user(user: User):
    # 保存用户（包含 password）
    save_user_to_db(user)
    
    # 返回完整的 User 对象
    return user
    # FastAPI 自动过滤，只返回 UserPublic 字段

# 请求：POST /users
# {"name": "Alice", "email": "alice@example.com", "password": "secret"}
# 
# 响应：200 OK
# {"name": "Alice", "email": "alice@example.com"}
# ⚠️ password 被自动过滤掉
```

**response_model 的好处：**
-  隐藏敏感字段
-  控制 API 响应格式
-  生成准确的 OpenAPI 文档
-  验证返回值类型

---

### **2.2.3 模型嵌套和引用**

FastAPI 完全支持嵌套的 Pydantic 模型：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address

class User(BaseModel):
    name: str
    email: str
    company: Company  # 嵌套模型

@app.post("/users")
async def create_user(user: User):
    return {"user": user}

# 请求：POST /users
# {
#   "name": "Alice",
#   "email": "alice@example.com",
#   "company": {
#     "name": "TechCorp",
#     "address": {
#       "street": "123 Main St",
#       "city": "New York",
#       "country": "USA"
#     }
#   }
# }

# FastAPI 自动验证所有层级的数据
```

---

### **2.2.4 自动生成 OpenAPI Schema**

FastAPI 从 Pydantic 模型自动生成 OpenAPI 文档：

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class CreateUserRequest(BaseModel):
    """创建用户请求"""
    
    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        description="用户姓名",
        example="Alice"
    )
    email: str = Field(
        ...,
        description="用户邮箱",
        example="alice@example.com"
    )
    age: int = Field(
        ...,
        ge=0,
        le=150,
        description="用户年龄",
        example=30
    )

@app.post("/users")
async def create_user(user: CreateUserRequest):
    return user

# 访问 http://localhost:8000/docs
# 自动生成的文档包含：
# - 完整的请求模型 schema
# - 字段类型、验证规则
# - 描述和示例
# - 可交互的测试界面
```

---

### **2.2.5 多个请求体参数**

FastAPI 支持多个 Pydantic 模型作为请求体：

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str

class Metadata(BaseModel):
    source: str
    ip_address: str

@app.post("/users")
async def create_user(
    user: User,
    metadata: Metadata,
    priority: int = Body(..., ge=1, le=5)
):
    return {
        "user": user,
        "metadata": metadata,
        "priority": priority
    }

# 请求：POST /users
# {
#   "user": {
#     "name": "Alice",
#     "email": "alice@example.com"
#   },
#   "metadata": {
#     "source": "web",
#     "ip_address": "192.168.1.1"
#   },
#   "priority": 3
# }
```

---

### **2.2.6 请求体 + 路径参数 + 查询参数**

FastAPI 智能识别参数类型：

```python
from fastapi import FastAPI, Query, Path
from pydantic import BaseModel

app = FastAPI()

class ItemUpdate(BaseModel):
    name: str
    price: float

@app.put("/items/{item_id}")
async def update_item(
    item_id: int = Path(..., ge=1),           # 路径参数
    item: ItemUpdate,                          # 请求体
    notify: bool = Query(True),                # 查询参数
    user_id: str = Query(...),                 # 查询参数（必需）
):
    return {
        "item_id": item_id,
        "item": item,
        "notify": notify,
        "user_id": user_id
    }

# 请求：PUT /items/123?notify=false&user_id=user_456
# Body: {"name": "Widget", "price": 9.99}
#
# FastAPI 自动识别：
# - item_id: 路径参数（从 URL 提取）
# - item: 请求体（Pydantic 模型）
# - notify, user_id: 查询参数（从 query string 提取）
```

---

### **2.2.7 response_model 高级用法**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str
    password: str
    is_active: bool = True
    metadata: dict = {}

@app.get(
    "/users/{user_id}",
    response_model=User,
    response_model_exclude={"password"},      # 排除字段
    # response_model_include={"name", "email"}, # 只包含指定字段
    # response_model_exclude_unset=True,        # 排除未设置的字段
    # response_model_exclude_none=True,         # 排除值为 None 的字段
)
async def get_user(user_id: int):
    return {
        "name": "Alice",
        "email": "alice@example.com",
        "password": "secret",  # 不会返回
        "is_active": True,
    }

# 响应：{"name": "Alice", "email": "alice@example.com", "is_active": true}
```

---

### **2.2.8. 列表响应**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    email: str

@app.get("/users", response_model=list[User])
#                  ^^^^^^^^^^^^^^^^^^^^^
#                  返回 User 列表
async def list_users():
    return [
        {"name": "Alice", "email": "alice@example.com"},
        {"name": "Bob", "email": "bob@example.com"},
    ]

# 响应：
# [
#   {"name": "Alice", "email": "alice@example.com"},
#   {"name": "Bob", "email": "bob@example.com"}
# ]
```

---

### **2.2.9 泛型响应模型**

```python
from fastapi import FastAPI
from pydantic import BaseModel, Generic, TypeVar
from typing import Generic

app = FastAPI()

T = TypeVar('T')

class Page(BaseModel, Generic[T]):
    """分页响应"""
    items: list[T]
    total: int
    page: int
    page_size: int

class User(BaseModel):
    name: str
    email: str

@app.get("/users", response_model=Page[User])
#                                 ^^^^^^^^^
#                                 泛型响应
async def list_users(page: int = 1, page_size: int = 10):
    users = [
        {"name": "Alice", "email": "alice@example.com"},
        {"name": "Bob", "email": "bob@example.com"},
    ]
    return {
        "items": users,
        "total": 100,
        "page": page,
        "page_size": page_size
    }

# 响应：
# {
#   "items": [
#     {"name": "Alice", "email": "alice@example.com"},
#     {"name": "Bob", "email": "bob@example.com"}
#   ],
#   "total": 100,
#   "page": 1,
#   "page_size": 10
# }
```

---

## **2.3 AutoGPT中的 Pydantic 模型实战分析**

---

### **2.3.1  API 请求/响应模型设计**

#### **案例 A：简单的请求模型**

```python
# backend/server/model.py
import pydantic
from typing import Optional

class CreateGraph(pydantic.BaseModel):
    """创建图的请求模型"""
    graph: Graph  # 嵌套的复杂模型

class SetGraphActiveVersion(pydantic.BaseModel):
    """设置图活跃版本的请求"""
    active_graph_version: int

class RequestTopUp(pydantic.BaseModel):
    """请求充值的模型"""
    credit_amount: int
```

**设计要点：**
-  模型名称清晰表达用途（`CreateXxx`, `UpdateXxx`, `RequestXxx`）
-  单一职责，每个模型只做一件事
-   嵌套复杂对象（如 [Graph](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:300:0-397:66)）

---

#### **案例 B：带描述和验证的模型**

```python
# backend/server/model.py
class CreateAPIKeyRequest(pydantic.BaseModel):
    """创建 API Key 的请求"""
    name: str                                # 必需字段
    permissions: list[APIKeyPermission]      # 权限列表
    description: Optional[str] = None        # 可选字段

class CreateAPIKeyResponse(pydantic.BaseModel):
    """创建 API Key 的响应"""
    api_key: APIKeyInfo                      # 密钥信息
    plain_text_key: str                      # 明文密钥（只返回一次）
```

**使用场景：**
```python
@v1_router.post("/api-keys", response_model=CreateAPIKeyResponse)
async def create_api_key(
    request: CreateAPIKeyRequest,
    user_id: Annotated[str, Security(get_user_id)]
):
    # 业务逻辑...
    return CreateAPIKeyResponse(
        api_key=api_key_info,
        plain_text_key=plain_key  # 只在创建时返回
    )
```

---

#### **案例 C：文件上传响应模型**

```python
# backend/server/model.py
class UploadFileResponse(pydantic.BaseModel):
    """文件上传响应"""
    file_uri: str             # 文件 URI（可能是 GCS 路径或 data URI）
    file_name: str            # 文件名
    size: int                 # 文件大小（字节）
    content_type: str         # MIME 类型
    expires_in_hours: int     # 过期时间（小时）
```

**实际使用：**
```python
@v1_router.post("/files/upload", response_model=UploadFileResponse)
async def upload_file(
    file: UploadFile = File(...),
    expiration_hours: int = 24,
):
    # 上传文件到云存储
    storage_path = await cloud_storage.store_file(content, filename)
    
    return UploadFileResponse(
        file_uri=storage_path,
        file_name=file.filename,
        size=len(content),
        content_type=file.content_type,
        expires_in_hours=expiration_hours,
    )
```

---

### **2.3.2 WebSocket 消息模型**

```python
# backend/server/model.py
import enum
from typing import Any, Optional

class WSMethod(enum.Enum):
    """WebSocket 方法枚举"""
    SUBSCRIBE_GRAPH_EXEC = "subscribe_graph_execution"
    SUBSCRIBE_GRAPH_EXECS = "subscribe_graph_executions"
    UNSUBSCRIBE = "unsubscribe"
    GRAPH_EXECUTION_EVENT = "graph_execution_event"
    NODE_EXECUTION_EVENT = "node_execution_event"
    ERROR = "error"
    HEARTBEAT = "heartbeat"

class WSMessage(pydantic.BaseModel):
    """WebSocket 消息基础模型"""
    method: WSMethod                                    # 方法（枚举）
    data: Optional[dict[str, Any] | list[Any] | str] = None  # 数据（灵活类型）
    success: bool | None = None                         # 是否成功
    channel: str | None = None                          # 频道
    error: str | None = None                            # 错误消息

class WSSubscribeGraphExecutionRequest(pydantic.BaseModel):
    """订阅图执行的请求"""
    graph_exec_id: str

class WSSubscribeGraphExecutionsRequest(pydantic.BaseModel):
    """订阅图执行列表的请求"""
    graph_id: str
```

**WebSocket 中的使用：**
```python
@app.websocket("/ws")
async def websocket_router(websocket: WebSocket):
    await websocket.accept()
    
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            
            # 使用 Pydantic 解析和验证
            message = WSMessage.model_validate_json(data)
            
            # 根据方法类型处理
            if message.method == WSMethod.SUBSCRIBE_GRAPH_EXEC:
                # 解析特定的请求数据
                request = WSSubscribeGraphExecutionRequest.model_validate(message.data)
                await handle_subscribe(websocket, request.graph_exec_id)
            
            # 发送响应
            response = WSMessage(
                method=message.method,
                success=True,
                channel=f"graph_exec#{request.graph_exec_id}"
            )
            await websocket.send_text(response.model_dump_json())
            
    except pydantic.ValidationError as e:
        # 验证失败，发送错误
        error_msg = WSMessage(
            method=WSMethod.ERROR,
            success=False,
            error="Invalid message format"
        )
        await websocket.send_text(error_msg.model_dump_json())
```

**设计优势：**
-  类型安全的 WebSocket 消息
-  枚举保证方法名称一致
-  灵活的 data 字段（可以是 dict/list/str）
-  自动验证和错误处理

---

### **2.3.3 核心业务模型 - BlockSchema**

```python
# backend/data/block.py
from pydantic import BaseModel
from typing import ClassVar, Any

class BlockSchema(BaseModel):
    """Block 输入/输出的基础 Schema"""
    
    cached_jsonschema: ClassVar[dict[str, Any]]
    #                  ^^^^^^^^ ClassVar 表示类变量，不是实例字段
    
    @classmethod
    def jsonschema(cls) -> dict[str, Any]:
        """生成 JSON Schema"""
        if cls.cached_jsonschema:
            return cls.cached_jsonschema
        
        # 生成并缓存 schema
        model = cls.model_json_schema()
        # ... 处理 schema
        cls.cached_jsonschema = processed_schema
        return cls.cached_jsonschema
    
    @classmethod
    def validate_data(cls, data: BlockInput) -> str | None:
        """验证数据"""
        return json.validate_with_jsonschema(
            schema=cls.jsonschema(),
            data={k: v for k, v in data.items() if v is not None},
        )
    
    @classmethod
    def get_field_schema(cls, field_name: str) -> dict[str, Any]:
        """获取特定字段的 schema"""
        model_schema = cls.jsonschema().get("properties", {})
        property_schema = model_schema.get(field_name)
        if not property_schema:
            raise ValueError(f"Invalid property name {field_name}")
        return property_schema
```

**实际 Block 使用：**
```python
from backend.data.block import Block, BlockSchema

class TextGeneratorBlock(Block):
    """文本生成 Block"""
    
    class Input(BlockSchema):
        """输入 Schema"""
        prompt: str = Field(
            ...,
            description="The text prompt",
            min_length=1
        )
        model: str = Field(
            default="gpt-4",
            description="The model to use"
        )
        max_tokens: int = Field(
            default=100,
            ge=1,
            le=4000,
            description="Maximum tokens"
        )
    
    class Output(BlockSchema):
        """输出 Schema"""
        generated_text: str = Field(
            ...,
            description="The generated text"
        )
        tokens_used: int = Field(
            ...,
            description="Tokens consumed"
        )
    
    # Block 使用这些 Schema
    input_schema = Input
    output_schema = Output
    
    async def execute(self, input_data: Input) -> Output:
        # 执行逻辑...
        result = await generate_text(
            prompt=input_data.prompt,
            model=input_data.model,
            max_tokens=input_data.max_tokens
        )
        return self.Output(
            generated_text=result.text,
            tokens_used=result.tokens
        )
```

**设计亮点：**
-  嵌套类定义（[Input](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/blocks/basic.py:123:4-124:86)/[Output](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/blocks/basic.py:126:4-127:88) 在 Block 内部）
-  自动生成 JSON Schema（用于前端表单）
-  类方法验证数据
-  支持字段级别的 schema 查询

---

### **2.3.4 复杂模型嵌套 - Graph 模型**

```python
# backend/data/graph.py
from pydantic import BaseModel, Field, computed_field
from typing import Any, Optional

class Link(BaseModel):
    """图中的连接"""
    source_id: str
    sink_id: str
    source_name: str
    sink_name: str
    is_static: bool = False
    
    def __hash__(self):
        """使 Link 可哈希（用于 set）"""
        return hash((self.source_id, self.sink_id, self.source_name, self.sink_name))

class Node(BaseModel):
    """图中的节点"""
    id: str
    block_id: str
    input_default: dict[str, Any] = {}  # Block 的默认输入
    metadata: dict[str, Any] = {}
    input_links: list[Link] = []        # 输入连接
    output_links: list[Link] = []       # 输出连接
    
    @property
    def block(self) -> Block:
        """获取节点对应的 Block"""
        return get_block(self.block_id)

class Graph(BaseModel):
    """完整的图模型"""
    id: str
    name: str
    description: str
    nodes: list[Node] = []
    is_active: bool = False
    version: int = 1
    
    @computed_field
    @property
    def input_schema(self) -> dict[str, Any]:
        """动态计算图的输入 Schema"""
        return self._generate_schema(
            *(
                (block.input_schema, node.input_default)
                for node in self.nodes
                if (block := node.block).block_type == BlockType.INPUT
            )
        )
    
    @computed_field
    @property
    def output_schema(self) -> dict[str, Any]:
        """动态计算图的输出 Schema"""
        return self._generate_schema(
            *(
                (block.input_schema, node.input_default)
                for node in self.nodes
                if (block := node.block).block_type == BlockType.OUTPUT
            )
        )
    
    @computed_field
    @property
    def has_external_trigger(self) -> bool:
        """是否有外部触发器"""
        return self.webhook_input_node is not None
    
    @property
    def webhook_input_node(self) -> Node | None:
        """获取 Webhook 输入节点"""
        return next(
            (
                node
                for node in self.nodes
                if node.block.block_type in (BlockType.WEBHOOK, BlockType.WEBHOOK_MANUAL)
            ),
            None,
        )
```

**关键特性：**
-  **深度嵌套**：[Graph](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:300:0-397:66) → [Node](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:76:0-93:20) → [Link](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:54:0-73:85)
-  **`@computed_field`**：动态计算属性（每次访问重新计算）
-  **`@property`**：只读属性
-  **复杂查询**：从节点列表中查找特定类型的节点

---

### **2.3.5 配置模型 - ConfigDict 高级用法**

```python
# backend/data/model.py
from pydantic import BaseModel, ConfigDict, Field

class User(BaseModel):
    """用户模型"""
    
    model_config = ConfigDict(
        # 允许从数据库字段名（camelCase）转换
        populate_by_name=True,
        # 严格模式（不允许额外字段）
        extra="forbid",
        # 验证赋值
        validate_assignment=True,
    )
    
    user_id: str = Field(alias="userId")      # 支持别名
    email: str
    created_at: datetime = Field(alias="createdAt")

class NodeExecutionStats(BaseModel):
    """节点执行统计"""
    
    model_config = ConfigDict(
        # 允许额外字段（灵活性）
        extra="allow",
        # 允许任意类型（如自定义类）
        arbitrary_types_allowed=True,
        # 排除未设置的字段
        use_enum_values=True,
    )
    
    node_id: str
    execution_time: float
    status: str
    # ... 其他统计字段可以动态添加

class BlockCost(BaseModel):
    """Block 成本模型"""
    cost_amount: int
    cost_filter: dict[str, Any]
    cost_type: BlockCostType  # 枚举类型
    
    def __init__(
        self,
        cost_amount: int,
        cost_type: BlockCostType = BlockCostType.RUN,
        cost_filter: Optional[dict] = None,
        **data: Any,
    ) -> None:
        """自定义初始化"""
        super().__init__(
            cost_amount=cost_amount,
            cost_filter=cost_filter or {},
            cost_type=cost_type,
            **data,
        )
```

---

### **2.3.6 分页响应模型**

```python
# backend/data/graph.py
from pydantic import BaseModel

class GraphMeta(BaseModel):
    """图的元数据（简化版）"""
    id: str
    name: str
    description: str
    is_active: bool
    version: int
    created_at: datetime
    updated_at: datetime

class GraphsPaginated(BaseModel):
    """分页的图列表响应"""
    graphs: list[GraphMeta]
    total: int
    page: int
    page_size: int
```

**API 使用：**
```python
@v1_router.get("/graphs", response_model=GraphsPaginated)
async def list_graphs(
    page: int = 1,
    page_size: int = 20,
    user_id: Annotated[str, Security(get_user_id)],
):
    # 查询数据库
    graphs, total = await graph_db.list_graphs_paginated(
        user_id=user_id,
        page=page,
        page_size=page_size
    )
    
    return GraphsPaginated(
        graphs=graphs,
        total=total,
        page=page,
        page_size=page_size
    )

# 响应：
# {
#   "graphs": [
#     {"id": "...", "name": "Agent 1", ...},
#     {"id": "...", "name": "Agent 2", ...}
#   ],
#   "total": 100,
#   "page": 1,
#   "page_size": 20
# }
```

---

### **2.3.7 通知系统模型（复杂联合类型）**

```python
# backend/data/notifications.py
from pydantic import BaseModel, ConfigDict, Field, EmailStr
from typing import Literal
from datetime import datetime

class BaseNotificationData(BaseModel):
    """通知数据基类"""
    model_config = ConfigDict(extra="allow")  # 允许额外字段

class GraphExecutionCompleteData(BaseNotificationData):
    """图执行完成通知数据"""
    graph_id: str
    graph_name: str
    execution_id: str
    status: Literal["success", "failed"]
    execution_time: float
    cost: float

class CreditBalanceLowData(BaseNotificationData):
    """余额不足通知数据"""
    current_balance: int
    threshold: int
    last_refill_date: datetime

class WeeklySummaryData(BaseNotificationData):
    """每周摘要数据"""
    total_executions: int
    total_cost: float
    most_used_agent: str
    cost_breakdown: dict[str, float]

# 通知事件模型
class BaseEventModel(BaseModel):
    type: NotificationType  # 枚举
    user_id: str
    created_at: datetime = Field(default_factory=lambda: datetime.now(tz=timezone.utc))

# 用户偏好设置
class NotificationPreferenceDTO(BaseModel):
    """用户通知偏好"""
    email: EmailStr = Field(..., description="User's email address")
    preferences: dict[NotificationType, bool] = Field(
        ..., description="Which notifications the user wants"
    )
    daily_limit: int = Field(..., description="Max emails per day")

class NotificationPreference(BaseModel):
    """数据库中的通知偏好"""
    user_id: str
    email: EmailStr
    preferences: dict[NotificationType, bool]
    daily_limit: int
    created_at: datetime
    updated_at: datetime
```

**使用场景：**
```python
@v1_router.post("/auth/user/preferences")
async def update_preferences(
    user_id: Annotated[str, Security(get_user_id)],
    preferences: NotificationPreferenceDTO = Body(...),
) -> NotificationPreference:
    # 更新偏好
    output = await update_user_notification_preference(user_id, preferences)
    return output
```

---

### **2.3.8 数据库模型转换**

```python
# backend/data/db.py
from pydantic import BaseModel, Field, field_validator
from uuid import uuid4

class BaseDbModel(BaseModel):
    """数据库模型基类"""
    id: str = Field(default_factory=lambda: str(uuid4()))
    
    @field_validator("id", mode="before")
    @classmethod
    def ensure_string_id(cls, v):
        """确保 ID 是字符串"""
        return str(v) if v is not None else str(uuid4())
```

**Node 模型的数据库转换：**
```python
class NodeModel(Node):
    """数据库中的 Node 模型"""
    graph_id: str
    graph_version: int
    webhook_id: Optional[str] = None
    webhook: Optional["Webhook"] = None
    
    @staticmethod
    def from_db(node: AgentNode, for_export: bool = False) -> "NodeModel":
        """从数据库对象创建 Pydantic 模型"""
        obj = NodeModel(
            id=node.id,
            block_id=node.agentBlockId,
            input_default=convert(node.constantInput, dict[str, Any]),
            metadata=convert(node.metadata, dict[str, Any]),
            graph_id=node.agentGraphId,
            graph_version=node.agentGraphVersion,
            webhook_id=node.webhookId,
            webhook=Webhook.from_db(node.Webhook) if node.Webhook else None,
        )
        # 转换关联的 Links
        obj.input_links = [Link.from_db(link) for link in node.Input or []]
        obj.output_links = [Link.from_db(link) for link in node.Output or []]
        
        if for_export:
            return obj.stripped_for_export()
        return obj
    
    def stripped_for_export(self) -> "NodeModel":
        """导出时移除敏感数据"""
        stripped_node = self.model_copy(deep=True)
        
        # 移除凭据和敏感数据
        if stripped_node.input_default:
            stripped_node.input_default = self._filter_secrets_from_node_input(
                stripped_node.input_default,
                self.block.input_schema.jsonschema()
            )
        
        # 移除 webhook 信息
        stripped_node.webhook_id = None
        stripped_node.webhook = None
        
        return stripped_node
```

---

## 2.4 **AutoGPT Pydantic 使用模式总结**

### **2.4.1 命名约定**

```python
# 请求模型
class CreateXxxRequest(BaseModel): ...
class UpdateXxxRequest(BaseModel): ...
class DeleteXxxRequest(BaseModel): ...

# 响应模型
class XxxResponse(BaseModel): ...
class XxxListResponse(BaseModel): ...
class XxxPaginated(BaseModel): ...

# DTO (Data Transfer Object)
class XxxDTO(BaseModel): ...

# Meta (元数据)
class XxxMeta(BaseModel): ...
```

---

### **2.4.2 模型分层**

```
BaseModel (Pydantic基类)
    ↓
BaseDbModel (添加ID和验证)
    ↓
Domain Models (业务模型: Graph, Node, Block)
    ↓
API Models (API 层模型: CreateGraph, GraphResponse)
```

---

### **2.4.3 配置策略**

```python
# 严格模式（API 请求）
model_config = ConfigDict(extra="forbid")

# 灵活模式（统计数据）
model_config = ConfigDict(extra="allow")

# 允许任意类型（包含自定义类）
model_config = ConfigDict(arbitrary_types_allowed=True)

# 支持别名（数据库字段转换）
model_config = ConfigDict(populate_by_name=True)
```

---

### **2.4.4 验证策略**

```python
# 字段级别验证
age: int = Field(..., ge=0, le=150)

# 模型级别验证
@field_validator("email")
@classmethod
def validate_email(cls, v):
    if "@" not in v:
        raise ValueError("Invalid email")
    return v

# 自定义验证方法
@classmethod
def validate_data(cls, data: dict) -> str | None:
    # 自定义验证逻辑
    pass
```

---

### **2.4.5 转换模式**

```python
# 从数据库 → Pydantic
@staticmethod
def from_db(db_obj: PrismaModel) -> "PydanticModel":
    return PydanticModel(
        id=db_obj.id,
        name=db_obj.name,
        # ... 字段映射
    )

# Pydantic → 字典
model.model_dump()
model.model_dump(exclude={"password"})
model.model_dump(include={"name", "email"})

# Pydantic → JSON
model.model_dump_json()

# 字典 → Pydantic
Model.model_validate(data_dict)

# JSON → Pydantic
Model.model_validate_json(json_str)
```

---

# 3 依赖注入系统

## 3.1 **基础复习**

### **3.1.1 什么是依赖注入（Dependency Injection）？**

依赖注入是一种设计模式，它允许我们**将依赖关系从代码中分离出来**，由外部提供给需要的组件。

```python
# ❌ 没有依赖注入（硬编码依赖）
def get_user(user_id: str):
    db = Database("postgresql://localhost/db")  # 硬编码
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    return user

# ✅ 使用依赖注入
def get_user(user_id: str, db: Database):
    #                       ^^^ 依赖由外部提供
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    return user

# 调用时注入依赖
db = Database("postgresql://localhost/db")
user = get_user("123", db=db)
```

**优势：**
-  **解耦**：代码不依赖具体实现
-  **可测试**：可以轻松注入 mock 对象
-  **可复用**：依赖可以在多处使用
-  **灵活**：可以在运行时改变依赖

---

### **3.1.2   Python 函数参数基础**

```python
# 位置参数
def greet(name):
    return f"Hello, {name}"

# 关键字参数
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

# 类型注解
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}"

# *args 和 **kwargs
def flexible(*args, **kwargs):
    print(args)    # 元组
    print(kwargs)  # 字典

flexible(1, 2, 3, name="Alice", age=30)
# args: (1, 2, 3)
# kwargs: {'name': 'Alice', 'age': 30}
```

---

### **3.1.3 装饰器基础**

装饰器是一个函数，它接收一个函数并返回一个新函数：

```python
# 简单装饰器
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_calls
def add(a, b):
    return a + b

# 等价于
# add = log_calls(add)

result = add(2, 3)
# 输出：
# Calling add
# Finished add
# 返回：5
```

**带参数的装饰器：**
```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")

greet("Alice")
# Hello, Alice
# Hello, Alice
# Hello, Alice
```

---

### **3.1.4 上下文管理器基础**

上下文管理器自动管理资源的获取和释放：

```python
# 文件操作（自动关闭）
with open("file.txt") as f:
    content = f.read()
# 文件自动关闭，即使发生异常

# 数据库连接
with database.connection() as conn:
    conn.execute("SELECT * FROM users")
# 连接自动关闭

# 自定义上下文管理器
class DatabaseConnection:
    def __enter__(self):
        print("Opening connection")
        self.conn = connect_to_db()
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing connection")
        self.conn.close()
        return False  # 不抑制异常

with DatabaseConnection() as conn:
    conn.execute("SELECT 1")

# 使用 contextlib
from contextlib import contextmanager

@contextmanager
def database_connection():
    conn = connect_to_db()
    try:
        yield conn  # 暂停，返回 conn
    finally:
        conn.close()  # 清理

with database_connection() as conn:
    conn.execute("SELECT 1")
```

---

## 3.2 **FastAPI 特性**

### **3.2.1 Depends() 基础用法**

> ## 总结：何时使用 `Depends()`
>
> **需要使用的情况**：
>
> -  需要用户认证时
> -  需要数据库连接时
> -  需要外部服务调用时
> -  需要业务规则验证时
> -  需要配置参数时
> -  需要缓存连接时
>
> **不需要使用的情况**：
>
> -  简单的参数验证（用 Pydantic 模型）
> -  静态配置（直接导入）
> -  纯计算逻辑（直接在函数中写）

FastAPI 使用 `Depends()` 声明依赖：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# 定义一个依赖函数
def get_query_param(q: str | None = None):
    return q

@app.get("/items")
def read_items(query: str | None = Depends(get_query_param)):
    #                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                            声明依赖
    return {"query": query}

# 请求：GET /items?q=test
# FastAPI 自动调用 get_query_param()，传递结果给 query
```

**工作流程：**
```
1. 请求到达 /items
2. FastAPI 看到 Depends(get_query_param)
3. 调用 get_query_param(q="test")
4. 将返回值传递给 query 参数
5. 执行 read_items(query="test")
```

---

### **3.2.2 依赖函数的参数提取**

依赖函数本身也可以有参数（路径、查询、请求体等）：

```python
from fastapi import FastAPI, Depends, Query

app = FastAPI()

# 依赖函数可以有自己的参数
def pagination(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
):
    return {"skip": skip, "limit": limit}

@app.get("/items")
def read_items(page_params: dict = Depends(pagination)):
    #                         ^^^^^^^^^^^^^^^^^^^^^^
    return {
        "items": ["item1", "item2"],
        **page_params
    }

# 请求：GET /items?skip=20&limit=5
# 1. FastAPI 调用 pagination(skip=20, limit=5)
# 2. 返回 {"skip": 20, "limit": 5}
# 3. 传递给 page_params
# 响应：{"items": [...], "skip": 20, "limit": 5}
```

---

### **3.2.3 类作为依赖**

可以使用类作为依赖（调用 [__init__](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/block.py:340:4-433:36) 方法）：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

class Pagination:
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/items")
def read_items(pagination: Pagination = Depends()):
    #                                    ^^^^^^^^^
    #                                    不需要传参数，自动调用 __init__
    return {
        "skip": pagination.skip,
        "limit": pagination.limit
    }

# 请求：GET /items?skip=20&limit=5
# FastAPI 调用 Pagination(skip=20, limit=5)
```

**等价的简化写法：**
```python
@app.get("/items")
def read_items(pagination: Pagination = Depends(Pagination)):
    #                                    ^^^^^^^^^^^^^^^^^^
    #                                    显式指定类
    pass
```

---

### **3.2.4 依赖链（嵌套依赖）**

依赖可以依赖其他依赖：

```python
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()

# 第一层依赖：获取 token
def get_token(token: str | None = None):
    if not token:
        raise HTTPException(status_code=401, detail="No token")
    return token

# 第二层依赖：验证 token
def get_current_user(token: str = Depends(get_token)):
    #                            ^^^^^^^^^^^^^^^^^^^^^
    #                            依赖于 get_token
    if token != "valid-token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return {"username": "alice", "id": "123"}

# 第三层依赖：检查权限
def get_admin_user(user: dict = Depends(get_current_user)):
    #                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                       依赖于 get_current_user
    if user.get("role") != "admin":
        raise HTTPException(status_code=403, detail="Not admin")
    return user

@app.get("/admin/users")
def list_users(admin: dict = Depends(get_admin_user)):
    #                        ^^^^^^^^^^^^^^^^^^^^^^^^^
    #                        依赖于 get_admin_user
    return {"admin": admin, "users": [...]}

# 依赖链执行顺序：
# 1. get_token() → 返回 token
# 2. get_current_user(token) → 返回 user
# 3. get_admin_user(user) → 返回 admin
# 4. list_users(admin) → 返回响应
```

---

### **3.2.5 依赖缓存（同一请求内）**

同一个请求中，相同的依赖只会被调用**一次**：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

call_count = 0

def expensive_dependency():
    global call_count
    call_count += 1
    print(f"Called {call_count} times")
    return "expensive_result"

def dependency_a(dep: str = Depends(expensive_dependency)):
    return f"A: {dep}"

def dependency_b(dep: str = Depends(expensive_dependency)):
    return f"B: {dep}"

@app.get("/test")
def test_endpoint(
    a: str = Depends(dependency_a),
    b: str = Depends(dependency_b),
    c: str = Depends(expensive_dependency),
):
    return {"a": a, "b": b, "c": c}

# 请求：GET /test
# 输出：Called 1 times  （只调用一次！）
# 响应：{
#   "a": "A: expensive_result",
#   "b": "B: expensive_result",
#   "c": "expensive_result"
# }
```

**禁用缓存：**
```python
@app.get("/test")
def test_endpoint(
    dep: str = Depends(expensive_dependency, use_cache=False)
    #                                       ^^^^^^^^^^^^^^^^
):
    return {"dep": dep}
```

---

### **3.2.6 yield 依赖（资源管理）**

使用 `yield` 可以在请求前后执行代码（类似上下文管理器）：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def get_db():
    """数据库连接依赖"""
    print("Opening database connection")
    db = Database()
    db.connect()
    
    try:
        yield db  # 暂停，将 db 提供给路由
        #    ^^
        #    请求处理期间，db 可用
    finally:
        print("Closing database connection")
        db.close()
        # 请求结束后，无论成功或失败，都会关闭连接

@app.get("/users")
def get_users(db: Database = Depends(get_db)):
    #                        ^^^^^^^^^^^^^^^^
    users = db.query("SELECT * FROM users")
    return users

# 请求流程：
# 1. "Opening database connection" （yield 之前）
# 2. db.query() 执行
# 3. 返回响应
# 4. "Closing database connection" （finally 块）
```

**错误处理：**
```python
def get_db():
    db = Database()
    try:
        db.connect()
        yield db
    except Exception as e:
        db.rollback()  # 发生错误时回滚
        raise
    finally:
        db.close()  # 总是关闭连接
```

---

### **3.2.7 子依赖（多个依赖组合）**

一个路由可以有多个独立的依赖：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def get_db():
    return Database()

def get_current_user(token: str):
    return {"username": "alice"}

def get_settings():
    return {"max_items": 100}

@app.get("/items")
def read_items(
    db: Database = Depends(get_db),
    user: dict = Depends(get_current_user),
    settings: dict = Depends(get_settings),
):
    # 三个依赖都会被调用
    max_items = settings["max_items"]
    items = db.query(f"SELECT * FROM items LIMIT {max_items}")
    return {"user": user["username"], "items": items}
```

---

### **3.2.8 全局依赖（路由器级别）**

可以在路由器或应用级别声明依赖，所有端点都会应用：

```python
from fastapi import FastAPI, Depends, APIRouter, HTTPException

# 认证依赖
def verify_token(token: str | None = None):
    if not token:
        raise HTTPException(status_code=401, detail="No token")
    return token

def verify_api_key(api_key: str | None = None):
    if api_key != "secret-key":
        raise HTTPException(status_code=403, detail="Invalid API key")

# 方式 1：路由器级别
router = APIRouter(
    prefix="/admin",
    dependencies=[Depends(verify_token), Depends(verify_api_key)]
    #            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #            所有路由都需要这两个依赖
)

@router.get("/users")
def list_users():
    # 自动应用 verify_token 和 verify_api_key
    return {"users": [...]}

@router.get("/settings")
def get_settings():
    # 自动应用 verify_token 和 verify_api_key
    return {"settings": {...}}

# 方式 2：应用级别
app = FastAPI(dependencies=[Depends(verify_api_key)])
#             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#             所有端点都需要 API key

app.include_router(router)
```

---

### **3.2.9 依赖覆盖（测试时很有用）**

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def get_db():
    return RealDatabase()

@app.get("/users")
def get_users(db = Depends(get_db)):
    return db.query("SELECT * FROM users")

# 测试时覆盖依赖
def override_get_db():
    return MockDatabase()

app.dependency_overrides[get_db] = override_get_db
#  ^^^^^^^^^^^^^^^^^^^ 覆盖真实依赖

# 测试
from fastapi.testclient import TestClient

client = TestClient(app)
response = client.get("/users")
# 使用 MockDatabase 而不是 RealDatabase
```

---

### **3.2.10 路径操作装饰器中的依赖**

可以在装饰器中声明依赖，但不接收其返回值：

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def verify_token(token: str):
    if token != "valid":
        raise HTTPException(401)
    # 不返回任何值

@app.get(
    "/items",
    dependencies=[Depends(verify_token)]
    #            ^^^^^^^^^^^^^^^^^^^^^^^^
    #            只执行验证，不接收返回值
)
def read_items():
    # 不需要 token 参数
    return {"items": [...]}

# vs. 接收返回值
@app.get("/items")
def read_items(token: str = Depends(verify_token)):
    #             ^^^^^ 接收返回值
    return {"items": [...], "token": token}
```

---

## 3.3 **AutoGPT中的依赖注入实战分析**

---

### **3.1 认证依赖链（三层依赖）**

AutoGPT 的认证系统是完美的依赖链示例：

```python
# autogpt_libs/auth/jwt_utils.py
from fastapi import Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

# 第一层：Bearer Token 提取
bearer_jwt_auth = HTTPBearer(
    bearerFormat="jwt",
    scheme_name="HTTPBearerJWT",
    auto_error=False
)

async def get_jwt_payload(
    credentials: HTTPAuthorizationCredentials | None = Security(bearer_jwt_auth),
    #                                                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                                                   依赖于 HTTPBearer
) -> dict[str, Any]:
    """
    提取并验证 JWT payload
    
    依赖链：
    1. HTTPBearer 从 Authorization 头提取 token
    2. get_jwt_payload 验证并解码 token
    """
    if not credentials:
        raise HTTPException(status_code=401, detail="Authorization header is missing")
    
    try:
        # 验证 JWT 签名
        payload = jwt.decode(
            credentials.credentials,
            settings.JWT_VERIFY_KEY,
            algorithms=[settings.JWT_ALGORITHM],
            audience="authenticated",
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token has expired")
    except jwt.InvalidTokenError as e:
        raise HTTPException(status_code=401, detail=f"Invalid token: {str(e)}")
```

```python
# autogpt_libs/auth/dependencies.py

# 第二层：验证用户存在
async def requires_user(
    jwt_payload: dict = fastapi.Security(get_jwt_payload)
    #                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                   依赖于 get_jwt_payload
) -> User:
    """要求认证用户"""
    user_id = jwt_payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="User ID not found in token")
    
    return User.from_payload(jwt_payload)

# 第三层：提取用户ID
async def get_user_id(
    jwt_payload: dict = fastapi.Security(get_jwt_payload)
    #                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                   依赖于 get_jwt_payload
) -> str:
    """提取用户ID"""
    user_id = jwt_payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="User ID not found in token")
    return user_id

# 第三层变体：要求管理员权限
async def requires_admin_user(
    jwt_payload: dict = fastapi.Security(get_jwt_payload)
) -> User:
    """要求管理员用户"""
    user_id = jwt_payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="User ID not found in token")
    
    # 检查管理员角色
    if jwt_payload.get("role") != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
    
    return User.from_payload(jwt_payload)
```

**在路由中的使用：**

```python
# backend/server/routers/v1.py
from typing import Annotated
from fastapi import Security
from autogpt_libs.auth import get_user_id, requires_user, requires_admin_user

# 模式 1：只需要认证，不需要用户ID
@v1_router.get(
    "/graphs",
    dependencies=[Security(requires_user)],
    #            ^^^^^^^^^^^^^^^^^^^^^^^^^
    #            全局依赖，只验证，不接收返回值
)
async def list_graphs():
    return {"graphs": [...]}

# 模式 2：需要用户ID
@v1_router.get("/credits")
async def get_credits(
    user_id: Annotated[str, Security(get_user_id)],
    #       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #       注入用户ID
):
    return {"credits": await get_user_credits(user_id)}

# 模式 3：需要完整的用户对象
@v1_router.post("/profile")
async def update_profile(
    user: Annotated[User, Security(requires_user)],
    #     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #     注入用户对象
    profile_data: ProfileUpdate,
):
    return await update_user_profile(user, profile_data)

# 模式 4：需要管理员权限
@v1_router.post(
    "/admin/credits",
    dependencies=[Security(requires_admin_user)],
    #            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
)
async def add_credits(
    target_user_id: str,
    amount: int,
    admin_user_id: Annotated[str, Security(get_user_id)],
    #              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #              注入管理员ID
):
    logger.info(f"Admin {admin_user_id} adding {amount} credits to {target_user_id}")
    return await credit_model.add_credits(target_user_id, amount)
```

**依赖执行流程：**

```
请求：GET /credits
Header: Authorization: Bearer eyJhbGc...

1. HTTPBearer 提取 token: "eyJhbGc..."
   ↓
2. get_jwt_payload(credentials) 验证并解码
   返回: {"sub": "user123", "role": "user", "exp": 1234567890}
   ↓
3. get_user_id(jwt_payload) 提取用户ID
   返回: "user123"
   ↓
4. get_credits(user_id="user123") 执行业务逻辑
```

---

### **3.2 服务客户端依赖（单例模式）**

AutoGPT 使用依赖注入实现服务客户端的单例模式：

```python
# backend/util/clients.py
from autogpt_libs.utils.cache import thread_cached, cached

# 线程级缓存（每个线程一个实例）
@thread_cached
def get_database_manager_client() -> "DatabaseManagerClient":
    """获取数据库管理器客户端（线程缓存）"""
    from backend.executor import DatabaseManagerClient
    from backend.util.service import get_service_client
    
    return get_service_client(DatabaseManagerClient, request_retry=True)

@thread_cached
def get_scheduler_client() -> "SchedulerClient":
    """获取调度器客户端（线程缓存）"""
    from backend.executor.scheduler import SchedulerClient
    from backend.util.service import get_service_client
    
    return get_service_client(SchedulerClient)

@thread_cached
def get_execution_queue() -> "SyncRabbitMQ":
    """获取执行队列客户端（线程缓存）"""
    from backend.data.rabbitmq import SyncRabbitMQ
    from backend.executor.utils import create_execution_queue_config
    
    client = SyncRabbitMQ(create_execution_queue_config())
    client.connect()  # 建立连接
    return client

# 进程级缓存（整个进程共享一个实例）
@cached()
def get_supabase() -> "Client":
    """获取 Supabase 客户端（进程缓存）"""
    from supabase import create_client
    
    return create_client(
        settings.secrets.supabase_url,
        settings.secrets.supabase_service_role_key
    )
```

**@thread_cached 装饰器原理：**

```python
# 简化版实现
import threading

_thread_local = threading.local()

def thread_cached(func):
    """线程级缓存装饰器"""
    cache_key = f"_cache_{func.__name__}"
    
    def wrapper(*args, **kwargs):
        # 检查当前线程的缓存
        if not hasattr(_thread_local, cache_key):
            # 缓存不存在，调用函数并缓存
            setattr(_thread_local, cache_key, func(*args, **kwargs))
        
        # 返回缓存的实例
        return getattr(_thread_local, cache_key)
    
    return wrapper

# 每个线程都有自己的客户端实例
# 同一线程内多次调用返回同一实例
```

**在路由中使用：**

```python
from backend.util.clients import (
    get_database_manager_client,
    get_scheduler_client,
    get_execution_queue,
)

@v1_router.post("/graphs/execute")
async def execute_graph(
    graph_id: str,
    user_id: Annotated[str, Security(get_user_id)],
    # 注入服务客户端
    db_client = Depends(get_database_manager_client),
    scheduler = Depends(get_scheduler_client),
    queue = Depends(get_execution_queue),
):
    # 获取图定义
    graph = await db_client.get_graph(graph_id)
    
    # 调度执行
    execution_id = await scheduler.schedule_execution(graph, user_id)
    
    # 发送到队列
    await queue.publish({"execution_id": execution_id})
    
    return {"execution_id": execution_id}

# 同一个请求中多次调用，返回同一个实例
# db_client1 = get_database_manager_client()  # 创建实例
# db_client2 = get_database_manager_client()  # 返回缓存的实例
# db_client1 is db_client2  # True
```

---

### **3.3 WebSocket 连接管理器依赖（全局单例）**

```python
# backend/server/ws_api.py

# 全局变量（进程级单例）
_connection_manager: ConnectionManager | None = None

def get_connection_manager() -> ConnectionManager:
    """获取 WebSocket 连接管理器（全局单例）"""
    global _connection_manager
    if _connection_manager is None:
        _connection_manager = ConnectionManager()
    return _connection_manager

# WebSocket 路由
@app.websocket("/ws")
async def websocket_router(
    websocket: WebSocket,
    manager: ConnectionManager = Depends(get_connection_manager)
    #                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                            注入连接管理器
):
    # 认证
    user_id = await authenticate_websocket(websocket)
    if not user_id:
        return
    
    # 连接管理
    await manager.connect_socket(websocket)
    
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            message = WSMessage.model_validate_json(data)
            
            # 订阅
            if message.method == WSMethod.SUBSCRIBE_GRAPH_EXEC:
                channel = await manager.subscribe_graph_exec(
                    user_id=user_id,
                    graph_exec_id=message.data["graph_exec_id"],
                    websocket=websocket
                )
                # 发送确认
                await websocket.send_text(
                    WSMessage(
                        method=message.method,
                        success=True,
                        channel=channel
                    ).model_dump_json()
                )
    
    except WebSocketDisconnect:
        manager.disconnect_socket(websocket)
```

**为什么使用全局单例？**
-  所有 WebSocket 连接共享同一个管理器
-  可以向所有订阅者广播消息
-  内存中维护连接状态

---

### **3.4 数据库连接依赖（yield 模式）**

虽然 AutoGPT 使用全局的 `prisma` 实例，但我们可以看到 `yield` 模式的应用：

```python
# backend/data/db.py
from contextlib import asynccontextmanager

# 全局 Prisma 实例
prisma = Prisma(
    auto_register=True,
    datasource={"url": DATABASE_URL},
)

@asynccontextmanager
async def transaction(timeout: int = 30000):
    """
    数据库事务上下文管理器
    
    使用 yield 模式确保事务正确提交或回滚
    """
    async with prisma.tx(timeout=timeout) as tx:
        yield tx  # 提供事务对象
        # 退出时自动提交或回滚

# 如果作为依赖使用（假设的例子）
async def get_db_transaction():
    """数据库事务依赖"""
    async with transaction() as tx:
        yield tx

@app.post("/transfer")
async def transfer_credits(
    from_user: str,
    to_user: str,
    amount: int,
    tx = Depends(get_db_transaction),
    #    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
):
    # 在事务中执行
    await tx.execute_raw(
        "UPDATE users SET credits = credits - ? WHERE id = ?",
        amount, from_user
    )
    await tx.execute_raw(
        "UPDATE users SET credits = credits + ? WHERE id = ?",
        amount, to_user
    )
    # 退出时自动提交
    return {"status": "success"}
```

---

### **3.5 云存储处理器依赖（异步 + 锁）**

```python
# backend/util/cloud_storage.py
import asyncio

_cloud_storage_handler: CloudStorageHandler | None = None
_cleanup_lock = asyncio.Lock()

async def get_cloud_storage_handler() -> CloudStorageHandler:
    """获取云存储处理器（全局单例 + 异步锁）"""
    global _cloud_storage_handler
    
    if _cloud_storage_handler is None:
        async with _cleanup_lock:  # 防止并发创建
            if _cloud_storage_handler is None:
                _cloud_storage_handler = CloudStorageHandler()
                await _cloud_storage_handler.initialize()
    
    return _cloud_storage_handler

# 在路由中使用
@v1_router.post("/files/upload")
async def upload_file(
    file: UploadFile,
    user_id: Annotated[str, Security(get_user_id)],
    cloud_storage = Depends(get_cloud_storage_handler),
    #               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
):
    content = await file.read()
    
    # 上传到云存储
    storage_path = await cloud_storage.store_file(
        content=content,
        filename=file.filename,
        user_id=user_id,
    )
    
    return {"file_uri": storage_path}
```

**为什么需要锁？**
-  多个并发请求可能同时调用 `get_cloud_storage_handler()`
-  `asyncio.Lock()` 确保只创建一个实例
-  之后的请求直接返回已创建的实例

---

### **3.6 路由器级别的全局依赖**

```python
# backend/server/v2/builder/routes.py
from fastapi import APIRouter, Security
from autogpt_libs.auth import requires_user, get_user_id

# 路由器级别的依赖（所有端点都需要认证）
router = APIRouter(
    prefix="/builder",
    tags=["builder"],
    dependencies=[Security(requires_user)],
    #            ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #            所有路由都自动应用这个依赖
)

@router.get("/projects")
async def list_projects(
    user_id: Annotated[str, Security(get_user_id)],
    #       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #       还可以额外注入用户ID
):
    # requires_user 已经执行过验证
    return await get_user_projects(user_id)

@router.post("/projects")
async def create_project(
    user_id: Annotated[str, Security(get_user_id)],
    project_data: ProjectCreate,
):
    # requires_user 已经执行过验证
    return await create_new_project(user_id, project_data)

# 所有路由自动继承 requires_user 依赖
```

---

### **3.7 多个独立依赖的组合**

```python
# backend/server/routers/v1.py

@v1_router.post("/graphs/execute")
async def execute_graph(
    # 路径参数
    graph_id: str,
    
    # 请求体
    inputs: dict[str, Any],
    
    # 认证依赖
    user_id: Annotated[str, Security(get_user_id)],
    
    # 服务客户端依赖
    db_manager = Depends(get_database_manager_client),
    scheduler = Depends(get_scheduler_client),
    event_bus = Depends(get_async_execution_event_bus),
    
    # 凭证存储依赖
    creds_store = Depends(get_integration_credentials_store),
):
    """
    执行图
    
    依赖注入：
    1. user_id - 从 JWT token 提取
    2. db_manager - 数据库管理器客户端
    3. scheduler - 调度器客户端
    4. event_bus - 事件总线
    5. creds_store - 凭证存储
    """
    
    # 获取图定义
    graph = await db_manager.get_graph(graph_id, user_id=user_id)
    if not graph:
        raise HTTPException(404, "Graph not found")
    
    # 获取用户凭证
    credentials = await creds_store.get_credentials_for_graph(graph, user_id)
    
    # 调度执行
    execution = await scheduler.schedule_execution(
        graph=graph,
        inputs=inputs,
        credentials=credentials,
        user_id=user_id,
    )
    
    # 发布事件
    await event_bus.publish({
        "type": "execution_started",
        "execution_id": execution.id,
        "user_id": user_id,
    })
    
    return execution
```

**依赖执行顺序（并行）：**
```
所有依赖并行执行：
├─ get_user_id() → "user123"
├─ get_database_manager_client() → db_manager
├─ get_scheduler_client() → scheduler
├─ get_async_execution_event_bus() → event_bus
└─ get_integration_credentials_store() → creds_store

然后执行 execute_graph() 主函数
```

---

### **3.8 依赖的依赖（复杂链）**

```python
# 假设的复杂依赖链

async def get_db():
    """第一层：数据库连接"""
    db = Database()
    await db.connect()
    try:
        yield db
    finally:
        await db.disconnect()

async def get_user_service(db = Depends(get_db)):
    """第二层：用户服务（依赖数据库）"""
    return UserService(db)

async def get_current_user(
    user_id: str = Depends(get_user_id),
    user_service = Depends(get_user_service),
):
    """第三层：当前用户（依赖用户服务）"""
    user = await user_service.get_by_id(user_id)
    if not user:
        raise HTTPException(404, "User not found")
    return user

async def get_user_permissions(
    user = Depends(get_current_user),
    db = Depends(get_db),
):
    """第四层：用户权限（依赖用户和数据库）"""
    return await db.query(
        "SELECT * FROM permissions WHERE user_id = ?",
        user.id
    )

@app.post("/admin/action")
async def admin_action(
    permissions = Depends(get_user_permissions),
    #             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #             深层依赖链
):
    if "admin" not in permissions:
        raise HTTPException(403, "Not authorized")
    
    return {"status": "success"}

# 依赖树：
# admin_action
#   └─ get_user_permissions
#       ├─ get_current_user
#       │   ├─ get_user_id (从 JWT)
#       │   └─ get_user_service
#       │       └─ get_db
#       └─ get_db (缓存，只创建一次)
```

---

### **3.9 测试中的依赖覆盖**

```python
# backend/server/v2/otto/routes_test.py
import pytest
from fastapi.testclient import TestClient

@pytest.fixture(autouse=True)
def setup_app_auth(mock_jwt_user):
    """为所有测试设置认证覆盖"""
    from autogpt_libs.auth.jwt_utils import get_jwt_payload
    
    # 覆盖真实的认证依赖
    app.dependency_overrides[get_jwt_payload] = mock_jwt_user["get_jwt_payload"]
    #  ^^^^^^^^^^^^^^^^^^^^^ 依赖覆盖字典
    
    yield
    
    # 测试结束后清理
    app.dependency_overrides.clear()

def mock_jwt_user():
    """Mock JWT 用户"""
    def fake_get_jwt_payload():
        return {
            "sub": "test-user-123",
            "email": "test@example.com",
            "role": "user"
        }
    
    return {"get_jwt_payload": fake_get_jwt_payload}

def test_create_project():
    """测试创建项目"""
    client = TestClient(app)
    
    # 使用 mock 的认证
    response = client.post("/projects", json={"name": "Test Project"})
    
    assert response.status_code == 200
    # get_user_id() 返回 "test-user-123"
```

---

## 3.4 AutoGPT 依赖注入模式总结

### **3.4.1 认证依赖模式**

```python
# 三种使用方式：

# 方式 1：全局依赖（只验证）
@router.get("/public", dependencies=[Security(requires_user)])

# 方式 2：注入用户ID
def handler(user_id: Annotated[str, Security(get_user_id)])

# 方式 3：注入用户对象
def handler(user: Annotated[User, Security(requires_user)])
```

### **3.4.2 服务客户端模式**

```python
# 线程缓存 + 懒加载
@thread_cached
def get_service_client():
    client = ServiceClient()
    client.connect()
    return client

# 使用
def handler(client = Depends(get_service_client))
```

### **3.4.3 单例模式**

```python
# 全局变量 + 函数
_instance = None

def get_instance():
    global _instance
    if _instance is None:
        _instance = Service()
    return _instance
```

### **3.4.4 资源管理模式**

```python
# yield 模式
async def get_resource():
    resource = acquire_resource()
    try:
        yield resource
    finally:
        release_resource(resource)
```

---

# 4 异步支持

## 4.1 **基础复习**

### **4.1.1 同步 vs 异步**

**同步执行（阻塞）**

```python
import time

def fetch_user(user_id):
    """同步获取用户（阻塞）"""
    print(f"Fetching user {user_id}...")
    time.sleep(2)  # 模拟网络延迟（阻塞）
    print(f"Got user {user_id}")
    return {"id": user_id, "name": f"User {user_id}"}

def fetch_posts(user_id):
    """同步获取文章"""
    print(f"Fetching posts for {user_id}...")
    time.sleep(2)  # 阻塞
    print(f"Got posts for {user_id}")
    return [{"id": 1, "title": "Post 1"}]

# 同步执行（串行）
start = time.time()
user = fetch_user(1)      # 等待 2 秒
posts = fetch_posts(1)    # 再等待 2 秒
end = time.time()

print(f"Total time: {end - start:.2f}s")  # 约 4 秒

# 输出：
# Fetching user 1...
# Got user 1
# Fetching posts for 1...
# Got posts for 1
# Total time: 4.00s
```

**问题：** 当一个任务在等待（I/O、网络请求）时，CPU 空闲，无法处理其他任务。

---

**异步执行（非阻塞）**

```python
import asyncio

async def fetch_user(user_id):
    """异步获取用户（非阻塞）"""
    print(f"Fetching user {user_id}...")
    await asyncio.sleep(2)  # 让出控制权，允许其他任务执行
    print(f"Got user {user_id}")
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_posts(user_id):
    """异步获取文章"""
    print(f"Fetching posts for {user_id}...")
    await asyncio.sleep(2)  # 非阻塞等待
    print(f"Got posts for {user_id}")
    return [{"id": 1, "title": "Post 1"}]

# 异步执行（并发）
async def main():
    start = asyncio.get_event_loop().time()
    
    # 并发执行两个任务
    user, posts = await asyncio.gather(
        fetch_user(1),
        fetch_posts(1)
    )
    
    end = asyncio.get_event_loop().time()
    print(f"Total time: {end - start:.2f}s")  # 约 2 秒

asyncio.run(main())

# 输出：
# Fetching user 1...
# Fetching posts for 1...
# Got user 1
# Got posts for 1
# Total time: 2.00s  （并发执行，时间减半！）
```

**优势：** 当一个任务等待时，事件循环可以切换到其他任务，提高 CPU 利用率。

---

### **4.1.2  async/await 语法**

```python
import asyncio

# 定义异步函数（协程）
async def greet(name):
    """异步函数，返回协程对象"""
    print(f"Hello, {name}!")
    await asyncio.sleep(1)  # 暂停，让出控制权
    print(f"Goodbye, {name}!")
    return f"Greeted {name}"

# 调用异步函数
result = greet("Alice")
print(type(result))  # <class 'coroutine'>

# 运行协程
# 方式 1：使用 asyncio.run()
result = asyncio.run(greet("Alice"))
print(result)  # "Greeted Alice"

# 方式 2：在异步函数中使用 await
async def main():
    result = await greet("Bob")
    print(result)

asyncio.run(main())

# 方式 3：并发运行多个协程
async def main():
    results = await asyncio.gather(
        greet("Alice"),
        greet("Bob"),
        greet("Charlie"),
    )
    print(results)  # ["Greeted Alice", "Greeted Bob", "Greeted Charlie"]

asyncio.run(main())
```

**核心概念：**
- `async def` 定义异步函数（协程函数）
- `await` 暂停当前协程，等待另一个协程完成
- 协程必须在事件循环中运行

---

### **4.1.3 协程和事件循环**

```python
import asyncio

async def task1():
    print("Task 1 started")
    await asyncio.sleep(2)
    print("Task 1 finished")
    return "Result 1"

async def task2():
    print("Task 2 started")
    await asyncio.sleep(1)
    print("Task 2 finished")
    return "Result 2"

async def main():
    # 创建任务
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())
    
    # 等待所有任务完成
    result1 = await t1
    result2 = await t2
    
    print(f"{result1}, {result2}")

# 运行事件循环
asyncio.run(main())

# 输出：
# Task 1 started
# Task 2 started
# Task 2 finished  （1秒后）
# Task 1 finished  （2秒后）
# Result 1, Result 2
```

**事件循环工作流程：**
```
1. task1() 开始执行 → "Task 1 started"
2. 遇到 await asyncio.sleep(2) → 暂停 task1，切换到 task2
3. task2() 开始执行 → "Task 2 started"
4. 遇到 await asyncio.sleep(1) → 暂停 task2
5. 1秒后，task2 恢复 → "Task 2 finished"
6. 2秒后，task1 恢复 → "Task 1 finished"
```

---

### **4.1.4 异步 I/O 概念**

```python
import asyncio
import aiohttp  # 异步 HTTP 客户端

# 同步 HTTP 请求（阻塞）
import requests

def fetch_sync(url):
    response = requests.get(url)  # 阻塞，等待响应
    return response.json()

# 串行执行（慢）
urls = ["https://api.example.com/user/1", 
        "https://api.example.com/user/2",
        "https://api.example.com/user/3"]

import time
start = time.time()
results = [fetch_sync(url) for url in urls]
print(f"Sync time: {time.time() - start:.2f}s")  # 约 3 秒

# 异步 HTTP 请求（非阻塞）
async def fetch_async(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_async(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

start = time.time()
results = asyncio.run(main())
print(f"Async time: {time.time() - start:.2f}s")  # 约 1 秒（并发）
```

**适合异步的场景：**
-  网络请求（HTTP, WebSocket）
-  数据库查询
-  文件 I/O
-  定时任务
-  外部 API 调用

**不适合异步的场景：**
-  CPU 密集型计算（图像处理、加密）
-  同步库（没有异步版本的库）

---

### **4.1.5 异步生成器**

```python
import asyncio

async def fetch_page(page_num):
    """异步获取分页数据"""
    await asyncio.sleep(0.5)  # 模拟网络延迟
    return [f"Item {i}" for i in range(page_num * 10, (page_num + 1) * 10)]

async def stream_items():
    """异步生成器，流式返回数据"""
    for page in range(3):
        print(f"Fetching page {page}...")
        items = await fetch_page(page)
        for item in items:
            yield item  # 使用 yield，而不是 return

# 使用异步生成器
async def main():
    async for item in stream_items():
        #    ^^^^^^^^^^ 使用 async for
        print(f"Received: {item}")
        await asyncio.sleep(0.1)  # 处理每个项目

asyncio.run(main())

# 输出：
# Fetching page 0...
# Received: Item 0
# Received: Item 1
# ...
# Fetching page 1...
# Received: Item 10
# ...
```

---

### **4.1.6 异步上下文管理器**

```python
import asyncio

class AsyncDatabase:
    """异步数据库连接"""
    
    async def __aenter__(self):
        """进入上下文（异步）"""
        print("Connecting to database...")
        await asyncio.sleep(0.5)  # 模拟连接延迟
        print("Connected!")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """退出上下文（异步）"""
        print("Closing database connection...")
        await asyncio.sleep(0.2)
        print("Closed!")
        return False
    
    async def query(self, sql):
        print(f"Executing: {sql}")
        await asyncio.sleep(0.1)
        return [{"id": 1, "name": "Alice"}]

# 使用异步上下文管理器
async def main():
    async with AsyncDatabase() as db:
        #    ^^^^^^^^^^ 使用 async with
        results = await db.query("SELECT * FROM users")
        print(results)

asyncio.run(main())

# 输出：
# Connecting to database...
# Connected!
# Executing: SELECT * FROM users
# [{'id': 1, 'name': 'Alice'}]
# Closing database connection...
# Closed!
```

---

## 4.2 **FastAPI 特性**

### **4.2.1 async def 路由（推荐）**

FastAPI 原生支持异步路由：

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """异步路由"""
    # 模拟异步数据库查询
    await asyncio.sleep(0.1)
    return {"user_id": user_id, "name": f"User {user_id}"}

@app.get("/slow")
async def slow_endpoint():
    """慢速端点（不会阻塞其他请求）"""
    await asyncio.sleep(5)  # 等待 5 秒
    return {"message": "Done"}

# 同时处理多个请求
# 请求 1: GET /slow → 开始等待 5 秒
# 请求 2: GET /users/1 → 立即处理（不被阻塞）
# 请求 3: GET /users/2 → 立即处理（不被阻塞）
```

**为什么使用 async def？**
-  **高并发**：单线程处理多个请求
-  **非阻塞**：等待 I/O 时处理其他请求
-  **低内存**：不需要多线程/多进程

---

### **4.2.2 混合同步/异步路由**

FastAPI 同时支持同步和异步路由：

```python
from fastapi import FastAPI
import time
import asyncio

app = FastAPI()

# 同步路由（def）
@app.get("/sync")
def sync_endpoint():
    """同步路由（运行在线程池中）"""
    time.sleep(2)  # 阻塞操作
    return {"message": "Sync"}

# 异步路由（async def）
@app.get("/async")
async def async_endpoint():
    """异步路由（运行在事件循环中）"""
    await asyncio.sleep(2)  # 非阻塞等待
    return {"message": "Async"}

# FastAPI 如何处理？
# - async def 路由：在主事件循环中执行
# - def 路由：在 ThreadPoolExecutor 中执行（隔离阻塞操作）
```

**何时使用 def（同步）？**
- 使用同步库（没有异步版本）
- CPU 密集型操作
- 简单快速的操作（无 I/O）

**何时使用 async def（异步）？**
- 数据库查询（使用异步驱动）
- HTTP 请求（使用 httpx, aiohttp）
- 文件 I/O（使用 aiofiles）
- 任何涉及等待的操作

---

### **4.2.3. 异步依赖**

依赖函数也可以是异步的：

```python
from fastapi import FastAPI, Depends
import asyncio

app = FastAPI()

# 异步依赖
async def get_db():
    """异步数据库连接依赖"""
    print("Opening DB connection")
    await asyncio.sleep(0.1)  # 模拟连接延迟
    db = DatabaseConnection()
    try:
        yield db
    finally:
        print("Closing DB connection")
        await db.close()

async def get_current_user(token: str, db = Depends(get_db)):
    """异步获取当前用户"""
    await asyncio.sleep(0.05)
    user = await db.query("SELECT * FROM users WHERE token = ?", token)
    if not user:
        raise HTTPException(401, "Invalid token")
    return user

# 异步路由 + 异步依赖
@app.get("/profile")
async def get_profile(user = Depends(get_current_user)):
    """异步路由使用异步依赖"""
    return {"user": user}

# 同步路由 + 异步依赖也可以
@app.get("/profile-sync")
def get_profile_sync(user = Depends(get_current_user)):
    """同步路由也可以使用异步依赖"""
    # FastAPI 会在调用前等待异步依赖完成
    return {"user": user}
```

---

### **4.2.4 并发执行多个异步操作**

```python
from fastapi import FastAPI
import asyncio
import httpx

app = FastAPI()

async def fetch_user(user_id: int):
    """获取用户信息"""
    await asyncio.sleep(0.5)
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_posts(user_id: int):
    """获取用户文章"""
    await asyncio.sleep(0.5)
    return [{"id": 1, "title": "Post 1"}]

async def fetch_comments(user_id: int):
    """获取用户评论"""
    await asyncio.sleep(0.5)
    return [{"id": 1, "text": "Comment 1"}]

@app.get("/users/{user_id}/full")
async def get_user_full(user_id: int):
    """并发获取用户所有信息"""
    
    # 方式 1：使用 asyncio.gather（推荐）
    user, posts, comments = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id),
        fetch_comments(user_id),
    )
    # 总时间约 0.5 秒（并发执行）
    
    return {
        "user": user,
        "posts": posts,
        "comments": comments,
    }

@app.get("/users/{user_id}/sequential")
async def get_user_sequential(user_id: int):
    """串行获取用户信息"""
    user = await fetch_user(user_id)      # 0.5s
    posts = await fetch_posts(user_id)    # 0.5s
    comments = await fetch_comments(user_id)  # 0.5s
    # 总时间约 1.5 秒（串行执行）
    
    return {
        "user": user,
        "posts": posts,
        "comments": comments,
    }
```

---

### **4.2.5 异步后台任务**

```python
from fastapi import FastAPI, BackgroundTasks
import asyncio

app = FastAPI()

async def send_email(email: str, message: str):
    """异步发送邮件（后台任务）"""
    print(f"Sending email to {email}...")
    await asyncio.sleep(3)  # 模拟发送延迟
    print(f"Email sent to {email}")

@app.post("/users/register")
async def register_user(
    email: str,
    background_tasks: BackgroundTasks,
):
    """注册用户并发送欢迎邮件"""
    
    # 保存用户到数据库
    user = {"email": email}
    
    # 添加后台任务（异步）
    background_tasks.add_task(
        send_email,
        email,
        "Welcome to our platform!"
    )
    
    # 立即返回响应（不等待邮件发送）
    return {"message": "User registered", "user": user}

# 请求流程：
# 1. POST /users/register
# 2. 保存用户
# 3. 添加后台任务到队列
# 4. 立即返回响应 ← 用户得到响应
# 5. 后台任务异步执行（发送邮件）
```

---

### **4.2.6 流式响应（异步生成器）**

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

async def generate_numbers():
    """异步生成器，流式返回数据"""
    for i in range(10):
        await asyncio.sleep(0.5)  # 模拟数据生成
        yield f"data: {i}\n\n"  # Server-Sent Events 格式

@app.get("/stream")
async def stream_data():
    """流式响应端点"""
    return StreamingResponse(
        generate_numbers(),
        media_type="text/event-stream"
    )

# 客户端接收：
# data: 0
# （0.5秒后）
# data: 1
# （0.5秒后）
# data: 2
# ...
```

---

### **4.2.7 异步文件上传/下载**

```python
from fastapi import FastAPI, UploadFile
from fastapi.responses import FileResponse
import aiofiles  # pip install aiofiles

app = FastAPI()

@app.post("/upload")
async def upload_file(file: UploadFile):
    """异步上传文件"""
    # 异步读取
    content = await file.read()
    
    # 异步写入文件
    async with aiofiles.open(f"uploads/{file.filename}", "wb") as f:
        await f.write(content)
    
    return {"filename": file.filename, "size": len(content)}

@app.get("/download/{filename}")
async def download_file(filename: str):
    """异步下载文件"""
    # 检查文件是否存在（异步）
    async with aiofiles.open(f"uploads/{filename}", "rb") as f:
        content = await f.read()
    
    return FileResponse(
        path=f"uploads/{filename}",
        filename=filename,
    )
```

---

### **4.2.8 性能对比**

```python
from fastapi import FastAPI
import asyncio
import time

app = FastAPI()

# 模拟数据库查询
async def db_query_async():
    await asyncio.sleep(0.1)  # 非阻塞
    return {"data": "result"}

def db_query_sync():
    time.sleep(0.1)  # 阻塞
    return {"data": "result"}

# 异步端点（高并发）
@app.get("/async-endpoint")
async def async_endpoint():
    results = await asyncio.gather(
        db_query_async(),
        db_query_async(),
        db_query_async(),
    )
    return results
# 时间：约 0.1 秒（并发）

# 同步端点（低并发）
@app.get("/sync-endpoint")
def sync_endpoint():
    results = [
        db_query_sync(),
        db_query_sync(),
        db_query_sync(),
    ]
    return results
# 时间：约 0.3 秒（串行）

# 性能测试：
# 100 个并发请求到 /async-endpoint：
# - 总时间：约 1 秒
# - QPS：100/秒
#
# 100 个并发请求到 /sync-endpoint：
# - 总时间：约 30 秒（阻塞其他请求）
# - QPS：3/秒
```

---

## **4.3 AutoGPT中的异步实战分析**

---

### **4.3.1 全异步路由架构**

AutoGPT 的所有主要 API 端点都是异步的：

```python
# backend/server/routers/v1.py

# ✅ 所有路由都使用 async def
@v1_router.get("/graphs")
async def list_graphs(
    user_id: Annotated[str, Security(get_user_id)],
) -> Sequence[graph_db.GraphMeta]:
    # 异步数据库查询
    paginated_result = await graph_db.list_graphs_paginated(
        user_id=user_id,
        page=1,
        page_size=250,
        filter_by="active",
    )
    return paginated_result.graphs

@v1_router.post("/graphs")
async def create_new_graph(
    create_graph: CreateGraph,
    user_id: Annotated[str, Security(get_user_id)],
) -> graph_db.GraphModel:
    # 多个异步操作（串行执行）
    graph = graph_db.make_graph_model(create_graph.graph, user_id)
    graph.reassign_ids(user_id=user_id, reassign_graph_id=True)
    graph.validate_graph(for_run=False)
    
    # 异步创建图
    await graph_db.create_graph(graph, user_id=user_id)
    #     ^^^^^ 等待数据库操作
    
    # 异步创建库代理
    await library_db.create_library_agent(graph, user_id=user_id)
    #     ^^^^^ 等待另一个异步操作
    
    # 激活图（可能涉及 webhook 创建）
    return await on_graph_activate(graph, user_id=user_id)
    #            ^^^^^ 等待激活完成

@v1_router.delete("/graphs/{graph_id}")
async def delete_graph(
    graph_id: str,
    user_id: Annotated[str, Security(get_user_id)]
) -> DeleteGraphResponse:
    # 条件异步操作
    if active_version := await graph_db.get_graph(graph_id, user_id=user_id):
        #                  ^^^^^ 异步获取
        await on_graph_deactivate(active_version, user_id=user_id)
        #     ^^^^^ 异步停用
    
    # 异步删除
    return {"version_counts": await graph_db.delete_graph(graph_id, user_id=user_id)}
```

**为什么全异步？**
-  **数据库操作**：Prisma 客户端是异步的
-  **外部 API 调用**：Webhook 创建、OAuth 流程
-  **事件发布**：Redis 事件总线
-  **RPC 调用**：微服务间通信

---

### **4.3.2 串行 vs 并行的实际案例**

#### **串行执行（有依赖关系）**

```python
# backend/server/routers/v1.py

@v1_router.put("/graphs/{graph_id}")
async def update_graph(
    graph_id: str,
    graph: graph_db.Graph,
    user_id: Annotated[str, Security(get_user_id)],
) -> graph_db.GraphModel:
    # 步骤 1：获取现有版本（必须先完成）
    existing_versions = await graph_db.get_graph_all_versions(
        graph_id, user_id=user_id
    )
    if not existing_versions:
        raise HTTPException(404, detail=f"Graph #{graph_id} not found")
    
    # 步骤 2：计算新版本号（依赖步骤 1）
    latest_version_number = max(g.version for g in existing_versions)
    graph.version = latest_version_number + 1
    
    # 步骤 3：创建新版本（依赖步骤 2）
    new_graph_version = await graph_db.create_graph(graph, user_id=user_id)
    
    # 步骤 4：更新库（依赖步骤 3）
    if new_graph_version.is_active:
        await library_db.update_agent_version_in_library(
            user_id, graph.id, graph.version
        )
        
        # 步骤 5：激活新版本（依赖步骤 4）
        new_graph_version = await on_graph_activate(
            new_graph_version, user_id=user_id
        )
        
        # 步骤 6：设置活跃版本（依赖步骤 5）
        await graph_db.set_graph_active_version(
            graph_id=graph_id,
            version=new_graph_version.version,
            user_id=user_id
        )
        
        # 步骤 7：停用旧版本（依赖步骤 6）
        current_active_version = next(
            (v for v in existing_versions if v.is_active), None
        )
        if current_active_version:
            await on_graph_deactivate(current_active_version, user_id=user_id)
    
    # 步骤 8：获取完整的图（依赖步骤 7）
    new_graph_version_with_subgraphs = await graph_db.get_graph(
        graph_id,
        new_graph_version.version,
        user_id=user_id,
        include_subgraphs=True,
    )
    
    return new_graph_version_with_subgraphs

# 这些操作必须串行执行，因为每一步都依赖前一步的结果
```

---

### **4.3.3 异步生成器（Stream Processing）**

AutoGPT 使用异步生成器处理 Block 执行：

```python
# backend/executor/manager.py

async def execute_node(
    node: Node,
    creds_manager: IntegrationCredentialsManager,
    data: NodeExecutionEntry,
    execution_stats: NodeExecutionStats | None = None,
    nodes_input_masks: Optional[NodesInputMasks] = None,
) -> BlockOutput:
    """
    执行节点，返回异步生成器
    
    BlockOutput = AsyncGen[BlockOutputEntry, None]
    BlockOutputEntry = tuple[str, Any]
    """
    
    # ... 准备工作 ...
    
    try:
        # 异步迭代 Block 的执行结果
        async for output_name, output_data in node_block.execute(
            #    ^^^^^^^^^^ 使用 async for 消费异步生成器
            input_data, **extra_exec_kwargs
        ):
            # 处理每个输出
            output_data = json.convert_pydantic_to_json(output_data)
            output_size += len(json.dumps(output_data))
            log_metadata.debug("Node produced output", **{output_name: output_data})
            
            # yield 到下一层（这个函数本身也是异步生成器）
            yield output_name, output_data
            #     ^^^^^ 流式返回结果
            
    except Exception:
        sentry_sdk.capture_exception(scope=scope)
        raise
    finally:
        # 释放资源
        if creds_lock:
            await creds_manager.release(creds_lock)
```

**为什么使用异步生成器？**
-  **流式处理**：逐步产生结果，不需要等待全部完成
-  **内存效率**：不需要一次性加载所有输出
-  **实时反馈**：用户可以看到中间结果
-  **管道处理**：结果可以直接传递给下一个节点

---

### **4.3.4 异步上下文管理器（资源管理）**

```python
# backend/executor/manager.py（简化示例）

from contextlib import asynccontextmanager

@asynccontextmanager
async def distributed_lock(key: str, timeout: int = 60):
    """分布式锁（Redis）"""
    r = redis.get_redis()
    lock: AsyncRedisLock = r.lock(f"lock:{key}", timeout=timeout)
    
    try:
        # 获取锁
        await lock.acquire()
        yield  # 提供锁给调用者
    finally:
        # 释放锁
        if await lock.locked() and await lock.owned():
            try:
                await lock.release()
            except Exception:
                logger.exception("Failed to release lock")

# 使用
async def process_user_action(user_id: str):
    async with distributed_lock(f"user:{user_id}"):
        # 在锁保护下执行操作
        await update_user_balance(user_id)
        await create_transaction(user_id)
```

---

### **4.3.5 凭证管理器（异步锁）**

```python
# backend/integrations/creds_manager.py（简化）

class IntegrationCredentialsManager:
    async def acquire(
        self, user_id: str, credentials_id: str
    ) -> tuple[Credentials, AsyncRedisLock]:
        """获取凭证并加锁"""
        # 从存储获取凭证
        credentials = await self.store.get_creds_by_id(
            user_id, credentials_id
        )
        
        # 获取分布式锁（防止并发使用同一凭证）
        lock = await self._get_lock(credentials_id)
        await lock.acquire()
        
        return credentials, lock
    
    async def release(self, lock: AsyncRedisLock):
        """释放锁"""
        if await lock.locked() and await lock.owned():
            await lock.release()

# 在 execute_node 中使用：
async def execute_node(...):
    creds_lock = None
    try:
        # 获取凭证和锁
        credentials, creds_lock = await creds_manager.acquire(
            user_id, credentials_meta.id
        )
        
        # 使用凭证执行 Block
        async for output in node_block.execute(...):
            yield output
    finally:
        # 确保释放锁
        if creds_lock:
            await creds_manager.release(creds_lock)
```

**为什么需要锁？**
-  **防止并发**：同一凭证一次只能被一个 Block 使用
-  **避免冲突**：某些 API 不支持同一凭证的并发请求
-  **速率限制**：通过锁控制 API 调用频率

---

### **4.3.6 事件总线（异步发布/订阅）**

```python
# backend/data/execution.py

class AsyncRedisExecutionEventBus(AsyncRedisEventBus[ExecutionEvent]):
    """异步 Redis 事件总线"""
    
    async def publish(self, res: GraphExecutionMeta | NodeExecutionResult):
        """发布执行事件"""
        if isinstance(res, GraphExecutionMeta):
            await self._publish_graph_exec_update(res)
        else:
            await self._publish_node_exec_update(res)
    
    async def _publish(self, event: ExecutionEvent, channel: str):
        """发布事件到 Redis"""
        # 截断过大的数据
        limit = config.max_message_size_limit // 2
        if isinstance(event, GraphExecutionEvent):
            event.inputs = truncate(event.inputs, limit)
            event.outputs = truncate(event.outputs, limit)
        
        # 发布到 Redis
        await super().publish_event(event, channel)
    
    async def listen(
        self, user_id: str, graph_id: str = "*", graph_exec_id: str = "*"
    ) -> AsyncGenerator[ExecutionEvent, None]:
        """监听执行事件（异步生成器）"""
        async for event in self.listen_events(
            f"{user_id}/{graph_id}/{graph_exec_id}"
        ):
            yield event
```

**WebSocket 中的使用：**

```python
# backend/server/ws_api.py

@continuous_retry()
async def event_broadcaster(manager: ConnectionManager):
    """事件广播器（后台任务）"""
    event_queue = AsyncRedisExecutionEventBus()
    
    # 无限循环监听事件
    async for event in event_queue.listen("*"):
        #    ^^^^^^^^^^ 异步迭代器
        # 向所有订阅的 WebSocket 发送事件
        await manager.send_execution_update(event)
```

---

### **4.3.7 数据库操作（全异步）**

AutoGPT 使用 Prisma 异步客户端：

```python
# backend/data/graph.py（简化）

async def create_graph(
    graph: Graph,
    user_id: str,
) -> GraphModel:
    """异步创建图"""
    
    # 异步事务
    async with transaction() as tx:
        # 创建图记录
        db_graph = await tx.agentgraph.create(
            data={
                "id": graph.id,
                "userId": user_id,
                "name": graph.name,
                "description": graph.description,
                "version": graph.version,
                "isActive": graph.is_active,
            }
        )
        
        # 批量创建节点
        await tx.agentnode.create_many(
            data=[
                {
                    "id": node.id,
                    "agentBlockId": node.block_id,
                    "agentGraphId": graph.id,
                    "constantInput": node.input_default,
                }
                for node in graph.nodes
            ]
        )
        
        # 批量创建连接
        await tx.agentnodelink.create_many(
            data=[
                {
                    "id": link.id,
                    "agentNodeSourceId": link.source_id,
                    "agentNodeSinkId": link.sink_id,
                    "sourceName": link.source_name,
                    "sinkName": link.sink_name,
                }
                for link in get_all_links(graph)
            ]
        )
    
    return GraphModel.from_db(db_graph)

async def get_graph(
    graph_id: str,
    version: int | None = None,
    user_id: str | None = None,
) -> GraphModel | None:
    """异步获取图"""
    
    # 异步查询
    db_graph = await AgentGraph.prisma().find_first(
        where={
            "id": graph_id,
            "version": version,
            "userId": user_id,
        },
        include=AGENT_GRAPH_INCLUDE,  # 包含关联数据
    )
    
    if not db_graph:
        return None
    
    return GraphModel.from_db(db_graph)
```

---

### **4.3.8 文件上传（异步 I/O）**

```python
# backend/server/routers/v1.py

@v1_router.post("/files/upload")
async def upload_file(
    user_id: Annotated[str, Security(get_user_id)],
    file: UploadFile = File(...),
    provider: str = "gcs",
    expiration_hours: int = 24,
) -> UploadFileResponse:
    """异步上传文件"""
    
    # 异步读取文件内容
    content = await file.read()
    #         ^^^^^ 非阻塞读取
    content_size = len(content)
    
    # 提取文件信息
    file_name = file.filename or "uploaded_file"
    content_type = file.content_type or "application/octet-stream"
    
    # 异步病毒扫描
    await scan_content_safe(content, filename=file_name)
    #     ^^^^^ 非阻塞扫描
    
    # 获取云存储处理器（单例）
    cloud_storage = await get_cloud_storage_handler()
    #               ^^^^^ 异步获取
    
    # 异步上传到云存储
    storage_path = await cloud_storage.store_file(
        #              ^^^^^ 非阻塞上传
        content=content,
        filename=file_name,
        provider=provider,
        expiration_hours=expiration_hours,
        user_id=user_id,
    )
    
    return UploadFileResponse(
        file_uri=storage_path,
        file_name=file_name,
        size=content_size,
        content_type=content_type,
        expires_in_hours=expiration_hours,
    )
```

---

### **4.3.9  Stripe Webhook（异步处理）**

```python
# backend/server/routers/v1.py

@v1_router.post("/credits/stripe_webhook")
async def stripe_webhook(request: Request):
    """处理 Stripe webhook（异步）"""
    
    # 异步读取请求体
    payload = await request.body()
    #         ^^^^^ 非阻塞读取
    
    # 获取签名头
    sig_header = request.headers.get("Stripe-Signature")
    
    # 验证签名（同步操作）
    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, settings.config.stripe_webhook_secret
        )
    except ValueError:
        raise HTTPException(400, "Invalid payload")
    except stripe.error.SignatureVerificationError:
        raise HTTPException(400, "Invalid signature")
    
    # 异步处理不同类型的事件
    if event["type"] == "checkout.session.completed":
        await _user_credit_model.fulfill_purchase(event["data"]["object"])
        #     ^^^^^ 异步更新积分
    
    elif event["type"] == "refund.created" or event["type"] == "charge.dispute.closed":
        await _user_credit_model.deduct_credits(event["data"]["object"])
        #     ^^^^^ 异步扣除积分
    
    return Response(status_code=200)
```

---

### **4.3.10. 混合同步/异步（线程池）**

有些操作是同步的，但 AutoGPT 仍然在异步路由中使用它们：

```python
# backend/server/routers/v1.py

@v1_router.get("/blocks")
async def get_graph_blocks() -> Response:
    """获取所有 Blocks（缓存）"""
    
    # _get_cached_blocks 是异步的，但内部调用同步函数
    content = await _get_cached_blocks()
    #         ^^^^^ 异步等待
    
    return Response(
        content=content,
        media_type="application/json",
    )

@cached()
async def _get_cached_blocks() -> str:
    """
    异步缓存函数
    - 缓存命中：立即返回（无线程池）
    - 缓存未命中：在线程池中运行同步代码
    """
    # 在线程池中运行同步函数
    return await run_in_threadpool(_compute_blocks_sync)
    #            ^^^^^^^^^^^^^^^^^^
    #            FastAPI 的工具，将同步函数放入线程池

def _compute_blocks_sync() -> str:
    """
    同步函数，计算所有 Blocks
    - 实例化 226+ 个 Blocks
    - 计算成本
    - 序列化
    """
    from backend.data.credit import get_block_cost
    
    block_classes = get_blocks()
    result = []
    
    for block_class in block_classes.values():
        block_instance = block_class()  # 同步操作
        if not block_instance.disabled:
            costs = get_block_cost(block_instance)  # 同步操作
            result.append({
                **block_instance.to_dict(),
                "costs": [cost.model_dump() for cost in costs]
            })
    
    return dumps(result)  # 同步序列化
```

**为什么混合？**
-  **同步库**：某些库只有同步版本（如 Block 实例化）
-  **CPU 密集**：成本计算、序列化（在线程池中不会阻塞事件循环）
-  **缓存优化**：缓存命中时直接返回，不进入线程池

---

## 4.4 **AutoGPT 异步模式总结**

### **4.4.1 全异步架构**

```python
# 所有主要操作都是异步的：
async def handler():
    # 数据库
    user = await db.get_user(user_id)
    
    # 外部 API
    result = await http_client.post(url, data)
    
    # 事件发布
    await event_bus.publish(event)
    
    # 文件 I/O
    content = await file.read()
    
    # 云存储
    path = await cloud_storage.upload(content)
```

---

### **4.4.2 串行执行（有依赖）**

```python
# 必须按顺序执行的操作
async def update_resource():
    # 步骤 1
    existing = await get_existing()
    
    # 步骤 2（依赖步骤 1）
    new_version = calculate_version(existing)
    
    # 步骤 3（依赖步骤 2）
    result = await create_new(new_version)
    
    return result
```

---

### **4.4.3 异步生成器（流式处理）**

```python
async def execute_node(...) -> BlockOutput:
    """返回异步生成器"""
    async for output_name, output_data in node_block.execute(...):
        # 逐个 yield 结果
        yield output_name, output_data
```

---

### **4.4.4 异步上下文管理器（资源管理）**

```python
@asynccontextmanager
async def resource():
    # 获取资源
    res = await acquire()
    try:
        yield res
    finally:
        # 释放资源
        await release(res)
```

---

### **4.4.5 异步锁（并发控制）**

```python
# Redis 分布式锁
async with distributed_lock(key):
    # 在锁保护下执行
    await critical_operation()
```

---

### **4.4.6 异步事件总线（发布/订阅）**

```python
# 发布
await event_bus.publish(event)

# 订阅（异步生成器）
async for event in event_bus.listen(channel):
    await handle_event(event)
```

---

### **4.4.7 线程池隔离（同步代码）**

```python
@cached()
async def async_wrapper():
    # 在线程池中运行同步代码
    return await run_in_threadpool(sync_function)
```

---

# 5 异常处理与错误响应

## 5.1 **基础复习**

### **5.1.1 Python 异常处理基础**

```python
# 基本的 try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# 捕获多种异常
try:
    data = {"name": "Alice"}
    age = data["age"]  # KeyError
except KeyError:
    print("Key not found")
except ValueError:
    print("Invalid value")

# 捕获所有异常
try:
    risky_operation()
except Exception as e:
    print(f"Error: {e}")

# else 和 finally
try:
    file = open("file.txt")
    content = file.read()
except FileNotFoundError:
    print("File not found")
else:
    # 只有在没有异常时执行
    print("File read successfully")
finally:
    # 总是执行（用于清理）
    if 'file' in locals():
        file.close()
```

---

### **5.1.2 自定义异常**

```python
# 定义自定义异常
class InsufficientBalanceError(Exception):
    """余额不足异常"""
    def __init__(self, balance: int, required: int):
        self.balance = balance
        self.required = required
        super().__init__(
            f"Insufficient balance: {balance}, required: {required}"
        )

class UserNotFoundError(Exception):
    """用户不存在异常"""
    pass

# 使用自定义异常
def withdraw(balance: int, amount: int) -> int:
    if balance < amount:
        raise InsufficientBalanceError(balance, amount)
    return balance - amount

# 捕获自定义异常
try:
    new_balance = withdraw(100, 150)
except InsufficientBalanceError as e:
    print(f"Error: {e}")
    print(f"Balance: {e.balance}, Required: {e.required}")
```

---

### **5.1.3 异常链（Raise from）**

```python
# 保留原始异常信息
def process_user(user_id: str):
    try:
        user_data = fetch_from_api(user_id)
    except ConnectionError as e:
        # 抛出新异常，但保留原因
        raise UserNotFoundError(f"Cannot fetch user {user_id}") from e
        #                                                         ^^^^^^

# 捕获时可以看到完整的异常链
try:
    process_user("123")
except UserNotFoundError as e:
    print(f"Error: {e}")
    print(f"Caused by: {e.__cause__}")  # 原始的 ConnectionError
```

---

### **5.1.4 上下文管理器中的异常**

```python
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # exc_type: 异常类型
        # exc_val: 异常实例
        # exc_tb: 异常回溯
        
        if exc_type is not None:
            # 发生了异常
            print(f"Error occurred: {exc_val}")
            self.conn.rollback()  # 回滚
        else:
            # 没有异常
            self.conn.commit()  # 提交
        
        self.conn.close()
        
        # 返回 False：不抑制异常（继续传播）
        # 返回 True：抑制异常（不向外传播）
        return False

# 使用
try:
    with DatabaseConnection() as conn:
        conn.execute("INSERT INTO users ...")
        raise ValueError("Something went wrong")
except ValueError:
    print("Exception was not suppressed")
```

---

## 5.2 **FastAPI 特性**

### **5.2.1 HTTPException（内置异常）**

FastAPI 提供了 `HTTPException` 用于返回 HTTP 错误响应：

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    if user_id not in database:
        # 抛出 HTTP 异常
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )
    return database[user_id]

# 请求：GET /users/999
# 响应：
# {
#   "detail": "User not found"
# }
# 状态码：404
```

**带额外信息的异常：**

```python
@app.get("/items/{item_id}")
async def get_item(item_id: int):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "Item-Not-Found"},
            #       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
            #       自定义响应头
        )
    return items[item_id]
```

---

### **5.2.2 自定义异常处理器**

可以为特定异常类型定义全局处理器：

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# 自定义异常
class UserNotFoundError(Exception):
    def __init__(self, user_id: str):
        self.user_id = user_id

# 注册异常处理器
@app.exception_handler(UserNotFoundError)
async def user_not_found_handler(request: Request, exc: UserNotFoundError):
    """处理 UserNotFoundError"""
    return JSONResponse(
        status_code=404,
        content={
            "error": "user_not_found",
            "message": f"User {exc.user_id} does not exist",
            "user_id": exc.user_id,
        }
    )

# 在路由中使用
@app.get("/users/{user_id}")
async def get_user(user_id: str):
    user = await db.get_user(user_id)
    if not user:
        # 抛出自定义异常
        raise UserNotFoundError(user_id)
    return user

# 请求：GET /users/unknown
# 响应：
# {
#   "error": "user_not_found",
#   "message": "User unknown does not exist",
#   "user_id": "unknown"
# }
```

---

### **5.2.3 覆盖默认异常处理器**

可以覆盖 FastAPI 的默认异常处理：

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import ValidationError

app = FastAPI()

# 覆盖 HTTPException 处理器
@app.exception_handler(HTTPException)
async def custom_http_exception_handler(request, exc):
    """自定义 HTTP 异常响应格式"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error_code": exc.status_code,
            "message": exc.detail,
            "path": request.url.path,
        }
    )

# 覆盖验证错误处理器
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    """自定义验证错误响应"""
    errors = []
    for error in exc.errors():
        errors.append({
            "field": " -> ".join(str(loc) for loc in error["loc"]),
            "message": error["msg"],
            "type": error["type"],
        })
    
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error_code": "validation_error",
            "errors": errors,
        }
    )

# 使用
@app.post("/users")
async def create_user(name: str, age: int):
    return {"name": name, "age": age}

# 请求：POST /users
# Body: {"name": "Alice", "age": "invalid"}
# 响应：
# {
#   "success": false,
#   "error_code": "validation_error",
#   "errors": [
#     {
#       "field": "body -> age",
#       "message": "value is not a valid integer",
#       "type": "type_error.integer"
#     }
#   ]
# }
```

---

### **5.2.4 捕获所有异常**

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import traceback

app = FastAPI()

# 捕获所有未处理的异常
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    """全局异常处理器"""
    # 记录详细错误日志
    logger.error(
        f"Unhandled exception: {exc}",
        extra={
            "path": request.url.path,
            "method": request.method,
            "traceback": traceback.format_exc(),
        }
    )
    
    # 返回通用错误响应（隐藏内部细节）
    return JSONResponse(
        status_code=500,
        content={
            "error": "internal_server_error",
            "message": "An unexpected error occurred",
            "request_id": request.state.request_id,  # 用于追踪
        }
    )
```

---

### **5.2.5 验证错误详情**

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, validator

app = FastAPI()

class User(BaseModel):
    username: str = Field(..., min_length=3, max_length=20)
    email: str
    age: int = Field(..., ge=0, le=150)
    
    @validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email format")
        return v

@app.post("/users")
async def create_user(user: User):
    return user

# 请求：POST /users
# Body: {"username": "ab", "email": "invalid", "age": 200}
# 默认响应：
# {
#   "detail": [
#     {
#       "loc": ["body", "username"],
#       "msg": "ensure this value has at least 3 characters",
#       "type": "value_error.any_str.min_length",
#       "ctx": {"limit_value": 3}
#     },
#     {
#       "loc": ["body", "email"],
#       "msg": "Invalid email format",
#       "type": "value_error"
#     },
#     {
#       "loc": ["body", "age"],
#       "msg": "ensure this value is less than or equal to 150",
#       "type": "value_error.number.not_le",
#       "ctx": {"limit_value": 150}
#     }
#   ]
# }
```

---

### **5.2.6 中间件中的异常处理**

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import time

app = FastAPI()

@app.middleware("http")
async def error_handling_middleware(request: Request, call_next):
    """中间件：捕获并记录所有异常"""
    start_time = time.time()
    
    try:
        response = await call_next(request)
        return response
    except Exception as exc:
        # 计算处理时间
        duration = time.time() - start_time
        
        # 记录错误
        logger.error(
            f"Request failed",
            extra={
                "path": request.url.path,
                "method": request.method,
                "duration": duration,
                "error": str(exc),
            }
        )
        
        # 返回错误响应
        return JSONResponse(
            status_code=500,
            content={
                "error": "internal_error",
                "message": "Request processing failed",
            }
        )
```

---

### **5.2.7 依赖注入中的异常**

```python
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()

async def get_current_user(token: str):
    """依赖：获取当前用户"""
    if not token:
        # 在依赖中抛出异常
        raise HTTPException(
            status_code=401,
            detail="Missing authentication token",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user = await verify_token(token)
    if not user:
        raise HTTPException(
            status_code=401,
            detail="Invalid or expired token"
        )
    
    return user

@app.get("/profile")
async def get_profile(user = Depends(get_current_user)):
    """使用依赖"""
    return user

# 请求：GET /profile （无 token）
# 响应：
# {
#   "detail": "Missing authentication token"
# }
# 状态码：401
# 响应头：WWW-Authenticate: Bearer
```

---

### **5.2.8 后台任务中的异常**

```python
from fastapi import FastAPI, BackgroundTasks
import logging

app = FastAPI()
logger = logging.getLogger(__name__)

def risky_background_task(user_id: str):
    """后台任务（可能失败）"""
    try:
        # 执行任务
        send_email(user_id)
        update_statistics(user_id)
    except Exception as e:
        # 记录错误（不会影响响应）
        logger.error(f"Background task failed for user {user_id}: {e}")
        # 可以发送告警、重试等

@app.post("/users/{user_id}/notify")
async def notify_user(user_id: str, background_tasks: BackgroundTasks):
    """添加后台任务"""
    background_tasks.add_task(risky_background_task, user_id)
    
    # 立即返回（不等待任务完成）
    return {"message": "Notification queued"}

# 即使后台任务失败，API 响应仍然成功
```

---

## 5.3 **AutoGPT中的处理异常实战分析**

### **5.3.1 自定义异常体系**

AutoGPT 定义了一套清晰的异常层次结构：

```python
# backend/util/exceptions.py

# 基础异常
class MissingConfigError(Exception):
    """配置缺失异常"""
    pass

class NotFoundError(ValueError):
    """资源不存在异常"""
    pass

class NotAuthorizedError(ValueError):
    """未授权异常"""
    pass

class DatabaseError(Exception):
    """数据库错误异常"""
    pass

# 业务逻辑异常
class InsufficientBalanceError(ValueError):
    """余额不足异常（带详细信息）"""
    user_id: str
    message: str
    balance: float
    amount: float
    
    def __init__(self, message: str, user_id: str, balance: float, amount: float):
        super().__init__(message)
        self.args = (message, user_id, balance, amount)
        self.message = message
        self.user_id = user_id
        self.balance = balance
        self.amount = amount
    
    def __str__(self):
        """前端显示用的错误消息"""
        return self.message

class ModerationError(ValueError):
    """内容审核失败异常"""
    user_id: str
    message: str
    graph_exec_id: str
    moderation_type: str
    content_id: str | None
    
    def __init__(
        self,
        message: str,
        user_id: str,
        graph_exec_id: str,
        moderation_type: str = "content",
        content_id: str | None = None,
    ):
        super().__init__(message)
        self.args = (message, user_id, graph_exec_id, moderation_type, content_id)
        self.message = message
        self.user_id = user_id
        self.graph_exec_id = graph_exec_id
        self.moderation_type = moderation_type
        self.content_id = content_id
    
    def __str__(self):
        """前端显示用的错误消息"""
        if self.content_id:
            return f"{self.message} (Moderation ID: {self.content_id})"
        return self.message

class GraphValidationError(ValueError):
    """图验证错误（结构化错误信息）"""
    
    def __init__(
        self,
        message: str,
        node_errors: Mapping[str, Mapping[str, str]] | None = None
    ):
        super().__init__(message)
        self.message = message
        self.node_errors = node_errors or {}
    
    def __str__(self):
        """格式化错误消息"""
        error_str = self.message
        for node_id, errors in self.node_errors.items():
            error_str += f"\n  {node_id}:"
            for k, e in errors.items():
                error_str += f"\n    {k}: {e}"
        return error_str
```

**设计特点：**
-  **丰富的上下文**：异常携带业务相关的详细信息
-  **清晰的层次**：继承自不同的基类（`ValueError`, `Exception`）
-  **自定义 [__str__](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/util/exceptions.py:63:4-67:27)**：为前端提供友好的错误消息
-  **结构化错误**：[GraphValidationError](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/util/exceptions.py:70:0-87:9) 支持多节点的错误聚合

---

### **5.3.2 全局异常处理器**

AutoGPT 为不同的异常类型注册了全局处理器：

```python
# backend/server/rest_api.py

from prisma.errors import PrismaError
from fastapi.exceptions import RequestValidationError
import pydantic

# 定义通用的错误处理器工厂
def handle_internal_http_error(status_code: int = 500, log_error: bool = True):
    """创建异常处理器"""
    def handler(request: fastapi.Request, exc: Exception):
        if log_error:
            # 记录详细日志
            logger.exception(
                "%s %s failed. Investigate and resolve the underlying issue: %s",
                request.method,
                request.url.path,
                exc,
            )
        
        # 根据状态码决定提示信息
        hint = (
            "Adjust the request and retry."
            if status_code < 500
            else "Check server logs and dependent services."
        )
        
        # 返回统一格式的错误响应
        return fastapi.responses.JSONResponse(
            content={
                "message": f"Failed to process {request.method} {request.url.path}",
                "detail": str(exc),
                "hint": hint,
            },
            status_code=status_code,
        )
    
    return handler

# 验证错误处理器
async def validation_error_handler(
    request: fastapi.Request, exc: Exception
) -> fastapi.responses.Response:
    """处理 Pydantic 验证错误"""
    logger.error(
        "Validation failed for %s %s: %s. Fix the request payload and try again.",
        request.method,
        request.url.path,
        exc,
    )
    
    # 提取错误详情
    errors: list | str
    if hasattr(exc, "errors"):
        errors = exc.errors()  # Pydantic 验证错误列表
    else:
        errors = str(exc)
    
    response_content = {
        "message": f"Invalid data for {request.method} {request.url.path}",
        "detail": errors,
        "hint": "Ensure the request matches the API schema.",
    }
    
    return fastapi.responses.Response(
        content=json.dumps(response_content),
        status_code=422,
        media_type="application/json",
    )

# 注册所有异常处理器
app.add_exception_handler(
    PrismaError,                    # 数据库错误
    handle_internal_http_error(500)
)
app.add_exception_handler(
    NotFoundError,                  # 资源不存在
    handle_internal_http_error(404, log_error=False)
    #                               ^^^^^^^^^^^^^^^^
    #                               404错误不记录详细日志
)
app.add_exception_handler(
    NotAuthorizedError,             # 未授权
    handle_internal_http_error(403, log_error=False)
)
app.add_exception_handler(
    RequestValidationError,         # FastAPI 验证错误
    validation_error_handler
)
app.add_exception_handler(
    pydantic.ValidationError,       # Pydantic 验证错误
    validation_error_handler
)
app.add_exception_handler(
    ValueError,                     # 值错误（通用）
    handle_internal_http_error(400)
)
app.add_exception_handler(
    Exception,                      # 捕获所有异常（兜底）
    handle_internal_http_error(500)
)
```

**异常处理层次：**
```
1. PrismaError (500)         → 数据库错误
2. NotFoundError (404)       → 资源不存在
3. NotAuthorizedError (403)  → 未授权
4. RequestValidationError (422) → 请求验证失败
5. ValueError (400)          → 值错误
6. Exception (500)           → 其他所有异常
```

---

### **5.3.3 路由中的异常处理**

#### **案例 A：使用 HTTPException**

```python
# backend/server/v2/library/routes/presets.py

from fastapi import HTTPException, status

@router.get("/{preset_id}")
async def get_preset(
    preset_id: str,
    user_id: Annotated[str, Security(get_user_id)],
):
    """获取预设（可能失败）"""
    try:
        # 尝试获取预设
        preset = await db.get_preset(user_id, preset_id)
    except Exception as e:
        # 记录错误并抛出 HTTP 异常
        logger.exception(
            "Error retrieving preset %s for user %s: %s",
            preset_id, user_id, e
        )
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )
    
    # 检查资源是否存在
    if not preset:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Preset #{preset_id} not found"
        )
    
    return preset
```

#### **案例 B：使用自定义异常**

```python
# backend/server/v2/library/routes/agents.py

@router.post("/store/{store_listing_version_id}")
async def add_agent_to_library(
    store_listing_version_id: str,
    user_id: Annotated[str, Security(get_user_id)],
):
    """添加 Agent 到库（多种异常类型）"""
    try:
        return await db.add_agent_to_library_from_store(
            store_listing_version_id, user_id
        )
    except NotFoundError as e:
        # 资源不存在（由全局处理器处理）
        logger.error(
            f"Could not find store listing version {store_listing_version_id} "
            "to add to library"
        )
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=str(e)
        )
    except DatabaseError as e:
        # 数据库错误
        logger.error(f"Database error while adding agent to library: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail={
                "message": str(e),
                "hint": "Inspect DB logs for details."
            }
        )
    except Exception as e:
        # 其他未知错误
        logger.error(f"Unexpected error while adding agent to library: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )
```

---

### **5.3.4 执行引擎中的异常处理**

AutoGPT 的执行引擎使用了复杂的异常处理策略：

```python
# backend/executor/manager.py

async def execute_node(
    node: Node,
    creds_manager: IntegrationCredentialsManager,
    data: NodeExecutionEntry,
    execution_stats: NodeExecutionStats | None = None,
    nodes_input_masks: Optional[NodesInputMasks] = None,
) -> BlockOutput:
    """执行节点（异步生成器）"""
    
    # ... 准备工作 ...
    
    # Sentry 错误追踪上下文
    scope = sentry_sdk.get_current_scope()
    original_user = scope._user
    original_tags = dict(scope._tags) if scope._tags else {}
    
    # 设置追踪标签
    scope.set_user({"id": user_id})
    scope.set_tag("graph_id", graph_id)
    scope.set_tag("node_id", node_id)
    scope.set_tag("block_name", node_block.name)
    
    try:
        # 执行 Block（异步生成器）
        async for output_name, output_data in node_block.execute(
            input_data, **extra_exec_kwargs
        ):
            output_data = json.convert_pydantic_to_json(output_data)
            output_size += len(json.dumps(output_data))
            log_metadata.debug("Node produced output", **{output_name: output_data})
            yield output_name, output_data
    
    except Exception:
        # 在恢复上下文前捕获异常
        sentry_sdk.capture_exception(scope=scope)
        sentry_sdk.flush()  # 确保发送到 Sentry
        # 重新抛出异常（继续正常的错误流）
        raise
    
    finally:
        # 确保释放凭证锁（即使失败）
        if creds_lock and (await creds_lock.locked()) and (await creds_lock.owned()):
            try:
                await creds_lock.release()
            except Exception as e:
                log_metadata.error(f"Failed to release credentials lock: {e}")
        
        # 更新执行统计
        if execution_stats is not None:
            execution_stats += node_block.execution_stats
            execution_stats.input_size = input_size
            execution_stats.output_size = output_size
        
        # 恢复 Sentry 上下文
        scope._user = original_user
        scope._tags = original_tags
```

**异常处理策略：**
- 📊 **Sentry 集成**：自动捕获和上报异常
- 🔒 **资源清理**：`finally` 块确保锁被释放
- 📈 **统计更新**：即使失败也更新执行统计
- 🔄 **异常传播**：重新抛出异常，由上层处理

---

### **5.3.5 业务异常的实际使用**

#### **余额不足异常：**

```python
# backend/executor/manager.py

try:
    # 检查余额
    remaining_balance = self._user_credit_model.spend_credits(
        user_id=user_id,
        graph_id=graph_id,
        graph_exec_id=graph_exec_id,
        node_id=queued_node_exec.node_id,
        current_balance=remaining_balance,
        transaction_cost=cost,
    )
except InsufficientBalanceError as balance_error:
    # 余额不足，设置错误状态
    error = balance_error
    node_exec_id = queued_node_exec.node_exec_id
    
    # 更新节点执行状态为失败
    db_client.upsert_execution_output(
        node_exec_id=node_exec_id,
        output_name="error",
        output_data={"error": str(balance_error)},
    )
    
    # 发送通知
    await self._send_low_balance_notification(user_id, balance_error.balance)
    
    # 记录日志
    log_metadata.error(
        f"Insufficient balance: {balance_error.balance}, "
        f"required: {balance_error.amount}"
    )

# 错误响应示例：
# {
#   "error": "Insufficient balance: 50 credits, required: 100 credits",
#   "user_id": "user123",
#   "balance": 50,
#   "amount": 100
# }
```

---

### **5.3.6 嵌套异常处理**

```python
# backend/server/v2/otto/service.py

class OttoService:
    @staticmethod
    async def ask(request: ChatRequest, user_id: str) -> ApiResponse:
        """调用 Otto API（多层异常处理）"""
        
        # 第一层：检查配置
        if not OTTO_API_URL:
            logger.error("Otto API URL is not configured")
            raise HTTPException(
                status_code=503,
                detail="Otto service is not configured"
            )
        
        try:
            # 第二层：HTTP 请求
            timeout = aiohttp.ClientTimeout(total=60)
            async with aiohttp.ClientSession(timeout=timeout) as session:
                async with session.post(
                    f"{OTTO_API_URL}/ask",
                    json=request.model_dump(),
                    headers={"Authorization": f"Bearer {API_KEY}"}
                ) as response:
                    if response.status != 200:
                        error_text = await response.text()
                        logger.error(f"Otto API error: {error_text}")
                        raise HTTPException(
                            status_code=response.status,
                            detail=f"Otto API request failed: {error_text}",
                        )
                    
                    return ApiResponse(**(await response.json()))
        
        except aiohttp.ClientError as e:
            # 第三层：网络错误
            logger.error(f"Connection error to Otto API: {str(e)}")
            raise HTTPException(
                status_code=503,
                detail="Failed to connect to Otto service"
            )
        
        except asyncio.TimeoutError:
            # 第四层：超时错误
            logger.error("Timeout error connecting to Otto API after 60 seconds")
            raise HTTPException(
                status_code=504,
                detail="Request to Otto service timed out"
            )
        
        except Exception as e:
            # 第五层：未知错误
            logger.error(f"Unexpected error in Otto API proxy: {str(e)}")
            raise HTTPException(
                status_code=500,
                detail="Internal server error in Otto proxy"
            )
```

**异常层次：**
```
1. 配置检查 → 503 Service Unavailable
2. HTTP 错误 → 根据响应状态码
3. 网络错误 → 503 Service Unavailable
4. 超时错误 → 504 Gateway Timeout
5. 未知错误 → 500 Internal Server Error
```

---

### **5.3.7 微服务异常处理**

```python
# backend/util/service.py

class HTTPClientError(Exception):
    """HTTP 客户端错误（4xx）- 不应重试"""
    def __init__(self, status_code: int, message: str):
        self.status_code = status_code
        super().__init__(f"HTTP {status_code}: {message}")

class HTTPServerError(Exception):
    """HTTP 服务器错误（5xx）- 可以重试"""
    def __init__(self, status_code: int, message: str):
        self.status_code = status_code
        super().__init__(f"HTTP {status_code}: {message}")

# 在 RPC 服务中注册异常处理器
class AppService:
    def __init__(self):
        self.fastapi_app = FastAPI()
        
        # 注册异常处理器
        self.fastapi_app.add_exception_handler(
            ValueError,
            self._handle_internal_http_error(400)
        )
        self.fastapi_app.add_exception_handler(
            Exception,
            self._handle_internal_http_error(500)
        )
    
    def _handle_internal_http_error(self, status_code: int):
        """内部 HTTP 错误处理器"""
        def handler(request: Request, exc: Exception):
            logger.exception(f"Service error: {exc}")
            return JSONResponse(
                status_code=status_code,
                content={
                    "error": str(exc),
                    "service": self.__class__.__name__,
                }
            )
        return handler
```

---

### **5.3.8 特征标志中的异常处理**

```python
# backend/util/feature_flag.py

def require_feature_flag(flag_name: str, default: bool = False):
    """装饰器：要求特定的特征标志"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            try:
                # 检查特征标志
                is_enabled = await check_feature_flag(flag_name)
                
                if is_enabled is None:
                    # LaunchDarkly 不可用，使用默认值
                    logger.warning(
                        f"Feature flag {flag_name} check failed, using default: {default}"
                    )
                    is_enabled = default
                
                if not is_enabled:
                    # 特征未启用
                    raise HTTPException(
                        status_code=404,
                        detail="Feature not available"
                    )
                
                # 特征已启用，执行函数
                return await func(*args, **kwargs)
            
            except Exception as e:
                # 捕获所有异常，记录日志
                logger.error(f"Feature flag check error: {e}")
                raise
        
        return wrapper
    return decorator

# 使用
@app.get("/experimental/feature")
@require_feature_flag("experimental_feature")
async def experimental_feature():
    return {"message": "Experimental feature"}
```

---

## 5.4 **AutoGPT 异常处理模式总结**

### **5.4.1 异常分类**

```python
# 业务异常（由业务逻辑触发）
InsufficientBalanceError   # 余额不足
ModerationError            # 内容审核失败
GraphValidationError       # 图验证失败

# 系统异常（由系统错误触发）
NotFoundError              # 资源不存在
NotAuthorizedError         # 未授权
DatabaseError              # 数据库错误
MissingConfigError         # 配置缺失

# HTTP 异常（用于 API 响应）
HTTPClientError (4xx)      # 客户端错误
HTTPServerError (5xx)      # 服务器错误
HTTPException              # FastAPI 内置
```

---

### **5.4.2 异常处理层次**

```
Level 1: 路由层
├─ 捕获特定业务异常
├─ 转换为 HTTPException
└─ 记录日志

Level 2: 全局处理器
├─ 捕获所有未处理异常
├─ 统一响应格式
└─ 记录详细日志

Level 3: 中间件
├─ 记录请求/响应时间
├─ 添加请求ID
└─ 错误指标收集

Level 4: Sentry
├─ 自动捕获异常
├─ 添加上下文信息
└─ 发送到错误追踪系统
```

---

### **5.4.3 错误响应格式**

```json
// 标准错误响应
{
  "message": "Failed to process POST /api/graphs",
  "detail": "Graph validation failed: missing required node",
  "hint": "Adjust the request and retry."
}

// 验证错误响应
{
  "message": "Invalid data for POST /api/users",
  "detail": [
    {
      "loc": ["body", "email"],
      "msg": "Invalid email format",
      "type": "value_error"
    }
  ],
  "hint": "Ensure the request matches the API schema."
}

// 业务异常响应（带额外信息）
{
  "error": "insufficient_balance",
  "message": "Insufficient balance: 50 credits, required: 100 credits",
  "user_id": "user123",
  "balance": 50,
  "required": 100
}
```

---

### **5.4.4 最佳实践**

```python
# ✅ 好的做法
@app.get("/users/{user_id}")
async def get_user(user_id: str):
    try:
        user = await db.get_user(user_id)
    except DatabaseError as e:
        logger.error(f"DB error: {e}")
        raise HTTPException(500, "Database error")
    
    if not user:
        raise HTTPException(404, f"User {user_id} not found")
    
    return user

# ❌ 避免的做法
@app.get("/users/{user_id}")
async def get_user(user_id: str):
    user = await db.get_user(user_id)  # 异常未处理
    return user  # 可能返回 None
```

---

### **5.4.5 资源清理模式**

```python
# 使用 try-finally 确保资源释放
async def process_with_lock(key: str):
    lock = await acquire_lock(key)
    try:
        # 执行操作
        result = await do_work()
        return result
    finally:
        # 确保释放锁
        await release_lock(lock)

# 使用异步上下文管理器
async def process_with_context(key: str):
    async with redis_lock(key):
        # 锁自动释放
        result = await do_work()
        return result
```

---

# 6 中间件系统

## 6.1 **基础复习**

### **6.1.1 中间件概念**

中间件是一个位于请求和响应处理流程中间的组件，可以：
- 在请求到达路由处理器**之前**处理请求
- 在响应返回客户端**之前**处理响应

```
客户端 → [中间件1] → [中间件2] → [路由处理器] → [中间件2] → [中间件1] → 客户端
         ↓          ↓               ↓            ↑          ↑
      请求处理     请求处理         业务逻辑      响应处理     响应处理
```

**常见用途：**
-  **认证/授权**：验证用户身份
-  **日志记录**：记录请求/响应信息
-  **性能监控**：测量处理时间
-  **CORS 处理**：跨域资源共享
-  **安全头**：添加安全相关的响应头
-  **请求 ID**：为每个请求生成唯一标识

---

### **6.1.2 装饰器模式（中间件的基础）**

Python 的装饰器是实现中间件的基础：

```python
import time
from functools import wraps

# 简单的计时装饰器
def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)  # 调用原函数
        duration = time.time() - start
        print(f"{func.__name__} took {duration:.2f}s")
        return result
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)
    return "Done"

slow_function()
# 输出：slow_function took 1.00s

# 带参数的装饰器
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")

greet("Alice")
# Hello, Alice
# Hello, Alice
# Hello, Alice

# 类装饰器
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call #{self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()  # Call #1, Hello!
say_hello()  # Call #2, Hello!
```

---

### **6.1.3 请求/响应拦截**

```python
from typing import Callable

# 模拟中间件链
class Request:
    def __init__(self, path: str, method: str):
        self.path = path
        self.method = method
        self.headers = {}

class Response:
    def __init__(self, content: str, status_code: int = 200):
        self.content = content
        self.status_code = status_code
        self.headers = {}

# 中间件1：添加请求ID
def request_id_middleware(handler: Callable):
    def middleware(request: Request):
        # 请求前处理
        import uuid
        request.headers["X-Request-ID"] = str(uuid.uuid4())
        print(f"Request ID: {request.headers['X-Request-ID']}")
        
        # 调用下一个处理器
        response = handler(request)
        
        # 响应后处理
        response.headers["X-Request-ID"] = request.headers["X-Request-ID"]
        return response
    
    return middleware

# 中间件2：日志记录
def logging_middleware(handler: Callable):
    def middleware(request: Request):
        # 请求前
        print(f"→ {request.method} {request.path}")
        
        # 调用处理器
        response = handler(request)
        
        # 响应后
        print(f"← {response.status_code}")
        return response
    
    return middleware

# 路由处理器
def route_handler(request: Request):
    return Response(f"Hello from {request.path}")

# 组合中间件
handler = request_id_middleware(
    logging_middleware(
        route_handler
    )
)

# 执行
request = Request("/users", "GET")
response = handler(request)

# 输出：
# Request ID: 12345-67890-...
# → GET /users
# ← 200
```

---

### **6.1.4 CORS 跨域**

CORS（Cross-Origin Resource Sharing）允许浏览器从不同域访问资源：

```python
# CORS 工作原理

# 1. 浏览器发送预检请求（OPTIONS）
# 请求：
# OPTIONS /api/users HTTP/1.1
# Origin: http://example.com
# Access-Control-Request-Method: POST
# Access-Control-Request-Headers: Content-Type

# 2. 服务器响应预检请求
# 响应：
# HTTP/1.1 200 OK
# Access-Control-Allow-Origin: http://example.com
# Access-Control-Allow-Methods: GET, POST, PUT, DELETE
# Access-Control-Allow-Headers: Content-Type
# Access-Control-Max-Age: 86400

# 3. 浏览器发送实际请求
# 请求：
# POST /api/users HTTP/1.1
# Origin: http://example.com
# Content-Type: application/json

# 4. 服务器响应实际请求
# 响应：
# HTTP/1.1 200 OK
# Access-Control-Allow-Origin: http://example.com
# Content-Type: application/json

# 手动实现 CORS
def cors_middleware(handler):
    def middleware(request):
        # 处理预检请求
        if request.method == "OPTIONS":
            response = Response("")
            response.headers["Access-Control-Allow-Origin"] = "*"
            response.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE"
            response.headers["Access-Control-Allow-Headers"] = "Content-Type"
            return response
        
        # 处理正常请求
        response = handler(request)
        response.headers["Access-Control-Allow-Origin"] = "*"
        return response
    
    return middleware
```

---

## 6.2 **FastAPI 特性**

### **6.2.1 中间件基础**

FastAPI 提供了多种方式定义中间件：

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
import time

app = FastAPI()

# 方式 1：使用 @app.middleware("http")
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    """添加处理时间头"""
    start_time = time.time()
    
    # 调用下一个处理器（路由或其他中间件）
    response = await call_next(request)
    #          ^^^^^ call_next 是一个可调用对象
    
    # 计算处理时间
    process_time = time.time() - start_time
    
    # 添加自定义头
    response.headers["X-Process-Time"] = str(process_time)
    
    return response

# 方式 2：使用 BaseHTTPMiddleware
from fastapi.middleware.base import BaseHTTPMiddleware

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # 请求前处理
        print(f"Request: {request.method} {request.url.path}")
        
        # 调用下一个处理器
        response = await call_next(request)
        
        # 响应后处理
        print(f"Response: {response.status_code}")
        
        return response

app.add_middleware(CustomMiddleware)

# 方式 3：使用内置中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

### **6.2.2 @app.middleware("http") 详解**

```python
from fastapi import FastAPI, Request, Response
import time
import uuid

app = FastAPI()

@app.middleware("http")
async def request_id_middleware(request: Request, call_next):
    """为每个请求添加唯一 ID"""
    # 生成请求 ID
    request_id = str(uuid.uuid4())
    
    # 将 ID 添加到请求状态（可在路由中访问）
    request.state.request_id = request_id
    
    # 调用下一个处理器
    response = await call_next(request)
    
    # 添加到响应头
    response.headers["X-Request-ID"] = request_id
    
    return response

@app.middleware("http")
async def timing_middleware(request: Request, call_next):
    """测量请求处理时间"""
    start_time = time.time()
    
    # 调用下一个处理器
    response = await call_next(request)
    
    # 计算时间
    duration = time.time() - start_time
    
    # 添加到响应头
    response.headers["X-Duration"] = f"{duration:.4f}"
    
    # 记录慢请求
    if duration > 1.0:
        print(f"Slow request: {request.url.path} took {duration:.2f}s")
    
    return response

@app.get("/test")
async def test_endpoint(request: Request):
    """测试端点"""
    # 访问中间件设置的请求 ID
    request_id = request.state.request_id
    return {"request_id": request_id, "message": "Hello"}

# 请求流程：
# 1. request_id_middleware (请求前) → 生成 ID
# 2. timing_middleware (请求前) → 开始计时
# 3. test_endpoint → 处理业务逻辑
# 4. timing_middleware (响应后) → 计算时间
# 5. request_id_middleware (响应后) → 添加 ID
```

---

### **6.2.3 中间件执行顺序**

中间件的执行顺序遵循**洋葱模型**（先进后出）：

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def middleware_1(request: Request, call_next):
    print("Middleware 1 - Before")
    response = await call_next(request)
    print("Middleware 1 - After")
    return response

@app.middleware("http")
async def middleware_2(request: Request, call_next):
    print("Middleware 2 - Before")
    response = await call_next(request)
    print("Middleware 2 - After")
    return response

@app.middleware("http")
async def middleware_3(request: Request, call_next):
    print("Middleware 3 - Before")
    response = await call_next(request)
    print("Middleware 3 - After")
    return response

@app.get("/test")
async def test():
    print("Route Handler")
    return {"message": "test"}

# 请求执行顺序：
# Middleware 3 - Before  ← 最后定义，最先执行
# Middleware 2 - Before
# Middleware 1 - Before
# Route Handler          ← 路由处理器
# Middleware 1 - After   ← 最先定义，最后执行（响应）
# Middleware 2 - After
# Middleware 3 - After
```

**重要规则：**
-  **后定义的中间件先执行**（类似栈）
-  **响应阶段顺序相反**（洋葱模型）
-  **`call_next()` 必须被调用**，否则请求不会继续

---

### **6.2.4 CORS 中间件配置**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 基本 CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],              # 允许所有来源
    allow_credentials=True,           # 允许携带凭证（cookies）
    allow_methods=["*"],              # 允许所有 HTTP 方法
    allow_headers=["*"],              # 允许所有请求头
)

# 生产环境的 CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://example.com",        # 生产环境域名
        "https://www.example.com",
        "http://localhost:3000",      # 开发环境
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],  # 限制方法
    allow_headers=[
        "Content-Type",
        "Authorization",
        "X-Requested-With",
    ],
    max_age=3600,                     # 预检请求缓存时间（秒）
)

# 动态 CORS 配置
import os

ENVIRONMENT = os.getenv("ENVIRONMENT", "development")

if ENVIRONMENT == "development":
    # 开发环境：宽松配置
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
else:
    # 生产环境：严格配置
    app.add_middleware(
        CORSMiddleware,
        allow_origins=[
            "https://app.example.com",
        ],
        allow_credentials=True,
        allow_methods=["GET", "POST", "PUT", "DELETE"],
        allow_headers=["Content-Type", "Authorization"],
    )
```

---

### **6.2.5 自定义中间件实现**

#### **案例 A：请求日志中间件**

```python
from fastapi import FastAPI, Request
import logging
import time

app = FastAPI()
logger = logging.getLogger(__name__)

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    """记录每个请求的详细信息"""
    # 开始时间
    start_time = time.time()
    
    # 记录请求信息
    logger.info(
        f"Request started",
        extra={
            "method": request.method,
            "path": request.url.path,
            "query_params": dict(request.query_params),
            "client_host": request.client.host if request.client else None,
            "user_agent": request.headers.get("user-agent"),
        }
    )
    
    # 处理请求
    try:
        response = await call_next(request)
        
        # 计算处理时间
        duration = time.time() - start_time
        
        # 记录响应信息
        logger.info(
            f"Request completed",
            extra={
                "method": request.method,
                "path": request.url.path,
                "status_code": response.status_code,
                "duration": duration,
            }
        )
        
        return response
    
    except Exception as e:
        # 记录异常
        duration = time.time() - start_time
        logger.error(
            f"Request failed",
            extra={
                "method": request.method,
                "path": request.url.path,
                "duration": duration,
                "error": str(e),
            }
        )
        raise
```

#### **案例 B：限流中间件**

```python
from fastapi import FastAPI, Request, HTTPException
from collections import defaultdict
import time

app = FastAPI()

# 简单的内存限流器
class RateLimiter:
    def __init__(self, requests_per_minute: int = 60):
        self.requests_per_minute = requests_per_minute
        self.requests = defaultdict(list)  # IP -> [timestamps]
    
    def is_allowed(self, client_ip: str) -> bool:
        """检查是否允许请求"""
        now = time.time()
        minute_ago = now - 60
        
        # 清理旧记录
        self.requests[client_ip] = [
            ts for ts in self.requests[client_ip]
            if ts > minute_ago
        ]
        
        # 检查限制
        if len(self.requests[client_ip]) >= self.requests_per_minute:
            return False
        
        # 记录新请求
        self.requests[client_ip].append(now)
        return True

rate_limiter = RateLimiter(requests_per_minute=10)

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    """限流中间件"""
    # 获取客户端 IP
    client_ip = request.client.host if request.client else "unknown"
    
    # 检查限流
    if not rate_limiter.is_allowed(client_ip):
        raise HTTPException(
            status_code=429,
            detail="Too many requests. Please try again later."
        )
    
    # 继续处理
    response = await call_next(request)
    return response
```

#### **案例 C：缓存中间件**

```python
from fastapi import FastAPI, Request
from fastapi.responses import Response
import hashlib
import json

app = FastAPI()

# 简单的内存缓存
cache = {}

@app.middleware("http")
async def cache_middleware(request: Request, call_next):
    """缓存 GET 请求的响应"""
    # 只缓存 GET 请求
    if request.method != "GET":
        return await call_next(request)
    
    # 生成缓存键
    cache_key = hashlib.md5(
        f"{request.url.path}?{request.url.query}".encode()
    ).hexdigest()
    
    # 检查缓存
    if cache_key in cache:
        print(f"Cache hit: {request.url.path}")
        cached_response = cache[cache_key]
        return Response(
            content=cached_response["content"],
            status_code=cached_response["status_code"],
            headers=cached_response["headers"],
            media_type=cached_response["media_type"],
        )
    
    # 缓存未命中，处理请求
    print(f"Cache miss: {request.url.path}")
    response = await call_next(request)
    
    # 读取响应内容
    response_body = b""
    async for chunk in response.body_iterator:
        response_body += chunk
    
    # 缓存响应
    cache[cache_key] = {
        "content": response_body,
        "status_code": response.status_code,
        "headers": dict(response.headers),
        "media_type": response.media_type,
    }
    
    # 返回新响应
    return Response(
        content=response_body,
        status_code=response.status_code,
        headers=dict(response.headers),
        media_type=response.media_type,
    )
```

---

## 6.3 **AutoGPT中的中间件实现实战分析**

### **6.3.1 安全头中间件（SecurityHeadersMiddleware）**

AutoGPT 使用 ASGI 原生实现（而非 BaseHTTPMiddleware）来提升性能：

```python
# backend/server/middleware/security.py

class SecurityHeadersMiddleware:
    """
    安全头中间件，添加安全相关的 HTTP 响应头
    
    特点：
    - 使用纯 ASGI 实现（比 BaseHTTPMiddleware 性能更好）
    - 默认禁用缓存（提高安全性）
    - 支持白名单路径（允许缓存静态资源）
    """
    
    # 可缓存路径白名单
    CACHEABLE_PATHS: Set[str] = {
        # 静态资源
        "/static",
        "/_next/static",
        "/assets",
        "/images",
        "/css",
        "/js",
        "/fonts",
        
        # 公共 API 端点
        "/api/health",
        "/api/blocks",
        
        # 公共商店页面（只读）
        "/api/store/agents",
        "/api/store/categories",
        "/api/store/featured",
        
        # 文档端点
        "/docs",
        "/openapi.json",
        
        # 网站元数据
        "/favicon.ico",
        "/manifest.json",
        "/robots.txt",
        "/sitemap.xml",
    }
    
    def __init__(self, app: ASGIApp):
        self.app = app
        # 预编译正则表达式（优化性能）
        self.cacheable_patterns = [
            re.compile(pattern.replace("*", "[^/]+"))
            for pattern in self.CACHEABLE_PATHS
            if "*" in pattern
        ]
        self.exact_paths = {
            path for path in self.CACHEABLE_PATHS 
            if "*" not in path
        }
    
    def is_cacheable_path(self, path: str) -> bool:
        """检查路径是否允许缓存"""
        # 检查精确匹配
        for cacheable_path in self.exact_paths:
            if path.startswith(cacheable_path):
                return True
        
        # 检查模式匹配
        for pattern in self.cacheable_patterns:
            if pattern.match(path):
                return True
        
        return False
    
    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        """
        ASGI 中间件实现
        
        为什么使用 ASGI 而不是 BaseHTTPMiddleware？
        - 性能更好（避免额外的异步包装）
        - 更低的内存占用
        - 更精确的控制
        """
        # 只处理 HTTP 请求
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return
        
        # 提取路径
        path = scope["path"]
        
        # 包装 send 函数，拦截响应
        async def send_wrapper(message: Message) -> None:
            if message["type"] == "http.response.start":
                # 获取现有响应头
                headers = dict(message.get("headers", []))
                
                # 添加安全响应头
                headers[b"X-Content-Type-Options"] = b"nosniff"
                #      ^^^^^^^^^^^^^^^^^^^^^^^^^
                #      防止 MIME 类型嗅探攻击
                
                headers[b"X-Frame-Options"] = b"DENY"
                #      ^^^^^^^^^^^^^^^^^^
                #      防止点击劫持（不允许在 iframe 中嵌入）
                
                headers[b"X-XSS-Protection"] = b"1; mode=block"
                #      ^^^^^^^^^^^^^^^^^^
                #      启用 XSS 保护（旧浏览器）
                
                headers[b"Referrer-Policy"] = b"strict-origin-when-cross-origin"
                #      ^^^^^^^^^^^^^^^^
                #      控制 Referer 头的发送策略
                
                # 为共享执行页面添加 noindex
                if "/public/shared" in path:
                    headers[b"X-Robots-Tag"] = b"noindex, nofollow"
                    #      ^^^^^^^^^^^^^^^
                    #      防止搜索引擎索引
                
                # 默认禁用缓存（安全优先）
                if not self.is_cacheable_path(path):
                    headers[b"Cache-Control"] = (
                        b"no-store, no-cache, must-revalidate, private"
                    )
                    headers[b"Pragma"] = b"no-cache"
                    headers[b"Expires"] = b"0"
                
                # 转换回列表格式
                message["headers"] = list(headers.items())
            
            # 发送修改后的消息
            await send(message)
        
        # 调用下一个处理器，使用包装的 send
        await self.app(scope, receive, send_wrapper)
```

**为什么使用 ASGI 而不是 BaseHTTPMiddleware？**

```python
# ❌ BaseHTTPMiddleware（性能较低）
from starlette.middleware.base import BaseHTTPMiddleware

class SlowMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        # 多一层异步包装
        response = await call_next(request)
        return response

# ✅ ASGI 中间件（性能更好）
class FastMiddleware:
    def __init__(self, app):
        self.app = app
    
    async def __call__(self, scope, receive, send):
        # 直接操作 ASGI 协议
        await self.app(scope, receive, send)
```

**性能对比：**
- ASGI 中间件：**直接操作底层协议**，零额外开销
- BaseHTTPMiddleware：**多一层抽象**，每个请求有额外的异步包装

---

### **6.3.2 应用启动时的中间件配置**

```python
# backend/server/rest_api.py

from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware
from backend.server.middleware.security import SecurityHeadersMiddleware

app = FastAPI(
    title="AutoGPT Platform API",
    version="1.0.0",
    lifespan=lifespan_context,  # 生命周期管理
)

# 1. 添加安全头中间件（最外层）
app.add_middleware(SecurityHeadersMiddleware)
#  ^^^^^^^^^^^^ 最后添加，最先执行

# 2. 添加 GZip 压缩中间件
app.add_middleware(
    GZipMiddleware,
    minimum_size=50_000  # 50KB 阈值（只压缩大响应）
)
#  ^^^^^^^^^^^^ 用于压缩大响应（如 /api/blocks）

# 中间件执行顺序（洋葱模型）：
# 请求 → SecurityHeadersMiddleware → GZipMiddleware → 路由处理器
# 响应 ← SecurityHeadersMiddleware ← GZipMiddleware ← 路由处理器
```

**为什么这样排序？**
- **SecurityHeadersMiddleware 最外层**：确保所有响应都有安全头
- **GZipMiddleware 内层**：在安全头之后压缩响应体

---

### **6.3.3 Prometheus 指标收集中间件**

AutoGPT 使用 `prometheus-fastapi-instrumentator` 自动收集 HTTP 指标：

```python
# backend/monitoring/instrumentation.py

from prometheus_client import Counter, Gauge, Histogram, Info
from prometheus_fastapi_instrumentator import Instrumentator, metrics

# 定义自定义业务指标
GRAPH_EXECUTIONS = Counter(
    "autogpt_graph_executions_total",
    "Total number of graph executions",
    labelnames=["status"],  # 只用 status，避免高基数
)

BLOCK_EXECUTIONS = Counter(
    "autogpt_block_executions_total",
    "Total number of block executions",
    labelnames=["block_type", "status"],
)

BLOCK_DURATION = Histogram(
    "autogpt_block_duration_seconds",
    "Duration of block executions in seconds",
    labelnames=["block_type"],
    buckets=[0.1, 0.25, 0.5, 1, 2.5, 5, 10, 30, 60],
    #        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #        自定义桶（更精确的延迟统计）
)

WEBSOCKET_CONNECTIONS = Gauge(
    "autogpt_websocket_connections_total",
    "Total number of active WebSocket connections",
    # 不用 user_id 标签（避免高基数问题）
)

DATABASE_QUERIES = Histogram(
    "autogpt_database_query_duration_seconds",
    "Duration of database queries in seconds",
    labelnames=["operation", "table"],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
)

def instrument_fastapi(
    app: FastAPI,
    service_name: str,
    expose_endpoint: bool = True,
    endpoint: str = "/metrics",
    include_in_schema: bool = False,
    excluded_handlers: Optional[list] = None,
) -> Instrumentator:
    """
    为 FastAPI 应用添加 Prometheus 指标收集
    
    Args:
        app: FastAPI 应用实例
        service_name: 服务名称（用于指标标签）
        expose_endpoint: 是否暴露 /metrics 端点
        endpoint: 指标端点路径
        include_in_schema: 是否在 OpenAPI 文档中显示
        excluded_handlers: 排除的路径（不收集指标）
    
    Returns:
        配置好的 Instrumentator 实例
    """
    
    # 设置服务信息
    SERVICE_INFO.info({
        "service": service_name,
        "version": "1.0.0",
    })
    
    # 创建 instrumentator
    instrumentator = Instrumentator(
        should_group_status_codes=True,      # 分组状态码（2xx, 4xx, 5xx）
        should_ignore_untemplated=True,      # 忽略未匹配的路径（如 /foo/bar/baz）
        should_respect_env_var=True,         # 支持环境变量控制
        should_instrument_requests_inprogress=True,  # 收集进行中的请求数
        excluded_handlers=["/health", "/readiness"],  # 排除健康检查
        env_var_name="ENABLE_METRICS",       # 环境变量名
    )
    
    # 添加默认 HTTP 指标
    instrumentator.add(
        metrics.default(
            metric_namespace="autogpt",
            metric_subsystem=service_name.replace("-", "_"),
        )
    )
    # 生成指标：
    # - autogpt_{service_name}_http_requests_total
    # - autogpt_{service_name}_http_request_duration_seconds
    
    # 添加请求大小指标
    instrumentator.add(
        metrics.request_size(
            metric_namespace="autogpt",
            metric_subsystem=service_name.replace("-", "_"),
        )
    )
    # 生成指标：autogpt_{service_name}_http_request_size_bytes
    
    # 添加响应大小指标
    instrumentator.add(
        metrics.response_size(
            metric_namespace="autogpt",
            metric_subsystem=service_name.replace("-", "_"),
        )
    )
    # 生成指标：autogpt_{service_name}_http_response_size_bytes
    
    # 添加延迟指标（自定义桶）
    instrumentator.add(
        metrics.latency(
            metric_namespace="autogpt",
            metric_subsystem=service_name.replace("-", "_"),
            buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10, 30, 60],
        )
    )
    # 生成指标：autogpt_{service_name}_http_request_duration_seconds_bucket
    
    # 安装到应用
    instrumentator.instrument(app)
    
    # 暴露 /metrics 端点
    if expose_endpoint:
        instrumentator.expose(
            app,
            endpoint=endpoint,
            include_in_schema=include_in_schema,
        )
        logger.info(f"Metrics endpoint exposed at {endpoint}")
    
    return instrumentator
```

**使用示例：**

```python
# backend/server/rest_api.py

from backend.monitoring.instrumentation import instrument_fastapi

app = FastAPI()

# 安装 Prometheus 指标收集
instrument_fastapi(
    app,
    service_name="rest_api",
    expose_endpoint=True,
    endpoint="/metrics",
    include_in_schema=False,  # 不在 Swagger 文档中显示
)

# 访问 http://localhost:8000/metrics
# 输出：
# # HELP autogpt_rest_api_http_requests_total Total requests
# # TYPE autogpt_rest_api_http_requests_total counter
# autogpt_rest_api_http_requests_total{handler="/api/graphs",method="GET",status="2xx"} 42.0
# autogpt_rest_api_http_requests_total{handler="/api/users",method="POST",status="2xx"} 10.0
# 
# # HELP autogpt_rest_api_http_request_duration_seconds Request duration
# # TYPE autogpt_rest_api_http_request_duration_seconds histogram
# autogpt_rest_api_http_request_duration_seconds_bucket{handler="/api/graphs",le="0.1"} 30.0
# autogpt_rest_api_http_request_duration_seconds_bucket{handler="/api/graphs",le="0.5"} 40.0
# autogpt_rest_api_http_request_duration_seconds_bucket{handler="/api/graphs",le="+Inf"} 42.0
```

---

### **6.3.4 业务指标记录**

```python
# 在业务代码中记录指标

from backend.monitoring.instrumentation import (
    record_graph_execution,
    record_block_execution,
    update_websocket_connections,
    record_database_query,
)

# 1. 记录图执行
async def execute_graph(graph_id: str, user_id: str):
    try:
        result = await graph_engine.execute(graph_id)
        record_graph_execution(graph_id, "success", user_id)
        #                                ^^^^^^^^^ 状态
        return result
    except Exception as e:
        record_graph_execution(graph_id, "error", user_id)
        raise

# 2. 记录 Block 执行
async def execute_block(block_type: str):
    start_time = time.time()
    try:
        result = await block.execute()
        duration = time.time() - start_time
        record_block_execution(block_type, "success", duration)
        #                      ^^^^^^^^^^  ^^^^^^^^^ ^^^^^^^^
        #                      Block类型    状态      耗时
        return result
    except Exception as e:
        duration = time.time() - start_time
        record_block_execution(block_type, "error", duration)
        raise

# 3. 记录 WebSocket 连接
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    update_websocket_connections(user_id, +1)  # 连接
    #                                    ^^^ 增加计数
    try:
        while True:
            data = await websocket.receive_text()
            await process_message(data)
    finally:
        update_websocket_connections(user_id, -1)  # 断开
        #                                    ^^^ 减少计数

# 4. 记录数据库查询
async def get_user(user_id: str):
    start_time = time.time()
    user = await db.user.find_unique(where={"id": user_id})
    duration = time.time() - start_time
    record_database_query("find_unique", "user", duration)
    #                     ^^^^^^^^^^^^^  ^^^^^^  ^^^^^^^^
    #                     操作类型        表名     耗时
    return user
```

---

### **6.3.5 高基数问题处理**

AutoGPT 特别注意避免 Prometheus 的**高基数问题**：

```python
# ❌ 错误：使用高基数标签（导致指标爆炸）
GRAPH_EXECUTIONS = Counter(
    "autogpt_graph_executions_total",
    labelnames=["graph_id", "user_id"],  # 可能有数百万种组合！
    #           ^^^^^^^^^  ^^^^^^^^^^
    #           高基数      高基数
)

# 每个唯一的标签组合都会创建一个新的时间序列
# 100,000 个用户 × 1,000 个图 = 1 亿个时间序列！
# 这会导致：
# - Prometheus 内存耗尽
# - 查询速度极慢
# - 无法删除旧数据

# ✅ 正确：只使用低基数标签
GRAPH_EXECUTIONS = Counter(
    "autogpt_graph_executions_total",
    labelnames=["status"],  # 只有几个值：success, error, timeout
    #           ^^^^^^^^
    #           低基数（只有 3-5 个值）
)

# 如果需要追踪特定用户/图，使用日志或专门的追踪系统
```

**低基数 vs 高基数：**

| 标签类型   | 示例                                | 唯一值数量 | 是否适合   |
| ---------- | ----------------------------------- | ---------- | ---------- |
| **低基数** | `status`, `method`, `block_type`    | < 100      | ✅ 适合     |
| **中基数** | `endpoint`, `table`                 | 100-1000   | ⚠️ 谨慎使用 |
| **高基数** | `user_id`, `graph_id`, `request_id` | > 10,000   | ❌ 不适合   |

---

### **6.3.6 完整的中间件栈**

AutoGPT 的完整中间件配置：

```python
# backend/server/rest_api.py

app = FastAPI()

# 中间件执行顺序（后添加先执行）：

# 1. Prometheus 指标收集（自动添加的中间件）
instrument_fastapi(app, "rest_api")

# 2. 安全头中间件
app.add_middleware(SecurityHeadersMiddleware)

# 3. GZip 压缩
app.add_middleware(GZipMiddleware, minimum_size=50_000)

# 4. CORS（如果需要）
# app.add_middleware(
#     CORSMiddleware,
#     allow_origins=["*"],
#     allow_credentials=True,
#     allow_methods=["*"],
#     allow_headers=["*"],
# )

# 执行流程：
# 请求 ↓
# [Prometheus] → 记录请求开始
# [Security]   → （请求阶段无操作）
# [GZip]       → （请求阶段无操作）
# [Route]      → 处理业务逻辑
# [GZip]       → 压缩响应体
# [Security]   → 添加安全头
# [Prometheus] → 记录请求完成
# 响应 ↑
```

---

## **6.4 AutoGPT 中间件模式总结**

### **6.4.1 ASGI 原生中间件（高性能）**

```python
class CustomMiddleware:
    def __init__(self, app: ASGIApp):
        self.app = app
    
    async def __call__(self, scope: Scope, receive: Receive, send: Send):
        # 直接操作 ASGI 协议
        async def send_wrapper(message: Message):
            if message["type"] == "http.response.start":
                # 修改响应头
                headers = dict(message.get("headers", []))
                headers[b"X-Custom"] = b"value"
                message["headers"] = list(headers.items())
            await send(message)
        
        await self.app(scope, receive, send_wrapper)
```

---

### **6.4.2 安全头最佳实践**

```python
# 必备的安全响应头
X-Content-Type-Options: nosniff           # 防止 MIME 嗅探
X-Frame-Options: DENY                     # 防止点击劫持
X-XSS-Protection: 1; mode=block           # XSS 保护
Referrer-Policy: strict-origin-when-cross-origin  # Referer 策略
Cache-Control: no-store, no-cache         # 默认禁用缓存
```

---

### **6.4.3 Prometheus 指标设计**

```python
# 好的指标（低基数）
Counter("requests_total", labelnames=["method", "status"])
Histogram("request_duration", labelnames=["endpoint"])

# 坏的指标（高基数）
Counter("requests_total", labelnames=["user_id", "graph_id"])  # ❌
```

---

### **6.4.4 中间件顺序原则**

```
1. 指标收集   ← 最外层（记录所有请求）
2. 安全头     ← 确保所有响应都有安全头
3. 压缩       ← 压缩响应体
4. CORS       ← 处理跨域
5. 路由处理器 ← 业务逻辑
```

---

# 7 应用生命周期管理

## 7.1 **基础复习**

### **7.1.1 应用启动和关闭**

应用生命周期是指从**启动**到**关闭**的整个过程：

```python
# 简单的生命周期示例

class Application:
    def __init__(self):
        self.resources = {}
    
    def startup(self):
        """启动时执行"""
        print("Starting application...")
        # 初始化资源
        self.resources['db'] = connect_to_database()
        self.resources['cache'] = connect_to_redis()
        print("Application started")
    
    def shutdown(self):
        """关闭时执行"""
        print("Shutting down application...")
        # 清理资源
        self.resources['db'].close()
        self.resources['cache'].close()
        print("Application stopped")
    
    def run(self):
        """运行应用"""
        try:
            self.startup()
            # 应用主循环
            while True:
                self.handle_request()
        except KeyboardInterrupt:
            pass
        finally:
            self.shutdown()

# 使用
app = Application()
app.run()

# 输出：
# Starting application...
# Application started
# ... (处理请求) ...
# ^C (用户按 Ctrl+C)
# Shutting down application...
# Application stopped
```

---

### **7.1.2 资源初始化**

常见需要初始化的资源：

```python
import asyncio
from typing import Dict, Any

class ResourceManager:
    def __init__(self):
        self.resources: Dict[str, Any] = {}
    
    async def initialize_database(self):
        """初始化数据库连接池"""
        print("Connecting to database...")
        await asyncio.sleep(1)  # 模拟连接延迟
        self.resources['db'] = DatabasePool(
            host='localhost',
            port=5432,
            pool_size=10,
        )
        print("Database connected")
    
    async def initialize_cache(self):
        """初始化缓存"""
        print("Connecting to Redis...")
        await asyncio.sleep(0.5)
        self.resources['cache'] = RedisClient(
            host='localhost',
            port=6379,
        )
        print("Redis connected")
    
    async def initialize_message_queue(self):
        """初始化消息队列"""
        print("Connecting to RabbitMQ...")
        await asyncio.sleep(0.5)
        self.resources['queue'] = RabbitMQClient(
            host='localhost',
            port=5672,
        )
        print("RabbitMQ connected")
    
    async def initialize_all(self):
        """并行初始化所有资源"""
        await asyncio.gather(
            self.initialize_database(),
            self.initialize_cache(),
            self.initialize_message_queue(),
        )
        print("All resources initialized")

# 使用
async def main():
    manager = ResourceManager()
    await manager.initialize_all()

asyncio.run(main())

# 输出：
# Connecting to database...
# Connecting to Redis...
# Connecting to RabbitMQ...
# Redis connected
# RabbitMQ connected
# Database connected
# All resources initialized
```

---

### **7.1.3 优雅关闭（Graceful Shutdown）**

优雅关闭确保：
-  完成正在处理的请求
-  保存未持久化的数据
-  关闭所有连接
-  清理临时资源

```python
import signal
import asyncio
from typing import Set

class GracefulShutdown:
    def __init__(self):
        self.shutdown_event = asyncio.Event()
        self.active_tasks: Set[asyncio.Task] = set()
    
    def setup_signal_handlers(self):
        """设置信号处理器"""
        # SIGINT (Ctrl+C)
        signal.signal(signal.SIGINT, self.signal_handler)
        # SIGTERM (kill)
        signal.signal(signal.SIGTERM, self.signal_handler)
    
    def signal_handler(self, sig, frame):
        """处理关闭信号"""
        print(f"\nReceived signal {sig}, initiating graceful shutdown...")
        self.shutdown_event.set()
    
    async def track_task(self, coro):
        """追踪正在执行的任务"""
        task = asyncio.create_task(coro)
        self.active_tasks.add(task)
        try:
            result = await task
            return result
        finally:
            self.active_tasks.discard(task)
    
    async def wait_for_shutdown(self):
        """等待关闭信号"""
        await self.shutdown_event.wait()
    
    async def shutdown(self, timeout: int = 30):
        """执行优雅关闭"""
        print(f"Waiting for {len(self.active_tasks)} active tasks to complete...")
        
        if self.active_tasks:
            # 等待活跃任务完成（带超时）
            done, pending = await asyncio.wait(
                self.active_tasks,
                timeout=timeout,
            )
            
            if pending:
                print(f"Warning: {len(pending)} tasks did not complete, forcing shutdown")
                for task in pending:
                    task.cancel()
        
        print("All tasks completed, shutting down")

# 使用
async def process_request(id: int, duration: float):
    """模拟请求处理"""
    print(f"Processing request {id}...")
    await asyncio.sleep(duration)
    print(f"Request {id} completed")

async def main():
    shutdown = GracefulShutdown()
    shutdown.setup_signal_handlers()
    
    # 启动一些任务
    await shutdown.track_task(process_request(1, 3))
    await shutdown.track_task(process_request(2, 5))
    
    # 等待关闭信号
    await shutdown.wait_for_shutdown()
    
    # 执行优雅关闭
    await shutdown.shutdown(timeout=10)

asyncio.run(main())

# 按 Ctrl+C 后输出：
# Processing request 1...
# Processing request 2...
# ^C
# Received signal 2, initiating graceful shutdown...
# Waiting for 2 active tasks to complete...
# Request 1 completed
# Request 2 completed
# All tasks completed, shutting down
```

---

### **7.1.4 上下文管理器**

上下文管理器确保资源的正确获取和释放：

```python
from contextlib import contextmanager, asynccontextmanager
import asyncio

# 同步上下文管理器
@contextmanager
def database_connection():
    """数据库连接上下文"""
    print("Opening database connection")
    conn = connect_to_database()
    try:
        yield conn  # 提供连接
    finally:
        print("Closing database connection")
        conn.close()

# 使用
with database_connection() as conn:
    conn.execute("SELECT * FROM users")
# 自动关闭连接

# 异步上下文管理器
@asynccontextmanager
async def async_database_connection():
    """异步数据库连接上下文"""
    print("Opening async database connection")
    conn = await async_connect_to_database()
    try:
        yield conn
    finally:
        print("Closing async database connection")
        await conn.close()

# 使用
async def main():
    async with async_database_connection() as conn:
        await conn.execute("SELECT * FROM users")
    # 自动关闭连接

# 类形式的上下文管理器
class FileManager:
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
    
    def __enter__(self):
        print(f"Opening {self.filename}")
        self.file = open(self.filename, 'w')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        # 返回 False 表示不抑制异常
        return False

# 使用
with FileManager("test.txt") as f:
    f.write("Hello, World!")
# 自动关闭文件
```

---

## 7.2 **FastAPI 特性**

### **7.2.1 事件处理器（已弃用）**

```python
from fastapi import FastAPI

app = FastAPI()

# ⚠️ 旧方式（已弃用，不推荐）
@app.on_event("startup")
async def startup_event():
    """应用启动时执行"""
    print("Application is starting up...")
    # 初始化资源
    await initialize_database()
    await initialize_cache()

@app.on_event("shutdown")
async def shutdown_event():
    """应用关闭时执行"""
    print("Application is shutting down...")
    # 清理资源
    await close_database()
    await close_cache()

# 问题：
# 1. 无法共享状态（startup 和 shutdown 是独立的函数）
# 2. 无法保证资源释放（shutdown 可能不执行）
# 3. 难以测试
```

---

### **7.2.1 lifespan 上下文管理器（推荐）**

FastAPI 推荐使用 [lifespan](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/ws_api.py:32:0-37:9) 参数：

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理"""
    # 启动阶段
    print("Application starting...")
    
    # 初始化资源
    db_pool = await create_database_pool()
    redis_client = await create_redis_client()
    
    # 将资源存储在 app.state 中（可在路由中访问）
    app.state.db = db_pool
    app.state.redis = redis_client
    
    print("Application started")
    
    yield  # 应用运行中
    #      ^^^^^ 在这里，应用正常运行
    
    # 关闭阶段
    print("Application shutting down...")
    
    # 清理资源
    await db_pool.close()
    await redis_client.close()
    
    print("Application stopped")

# 创建应用
app = FastAPI(lifespan=lifespan)

@app.get("/users")
async def get_users(request: Request):
    # 访问生命周期中初始化的资源
    db = request.app.state.db
    users = await db.fetch("SELECT * FROM users")
    return users

# 运行应用
# uvicorn main:app

# 启动时输出：
# Application starting...
# Application started
# INFO:     Uvicorn running on http://127.0.0.1:8000

# 关闭时输出（Ctrl+C）：
# ^C
# Application shutting down...
# Application stopped
```

**优势：**
-  **资源共享**：启动和关闭在同一个作用域
-  **保证清理**：即使异常也会执行 finally
-  **易于测试**：可以独立测试
-  **类型安全**：可以使用类型注解

---

### **7.1.3 多个资源的初始化**

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio

@asynccontextmanager
async def lifespan(app: FastAPI):
    """管理多个资源的生命周期"""
    print("=== Startup Phase ===")
    
    # 顺序初始化（有依赖关系）
    print("1. Connecting to database...")
    app.state.db = await create_database_pool()
    
    print("2. Running database migrations...")
    await run_migrations(app.state.db)
    
    # 并行初始化（无依赖关系）
    print("3. Initializing other services...")
    cache, queue, storage = await asyncio.gather(
        create_redis_client(),
        create_rabbitmq_client(),
        create_cloud_storage_client(),
    )
    
    app.state.cache = cache
    app.state.queue = queue
    app.state.storage = storage
    
    # 启动后台任务
    print("4. Starting background tasks...")
    app.state.background_task = asyncio.create_task(
        background_worker(app.state.queue)
    )
    
    print("=== Application Ready ===")
    
    yield
    
    print("=== Shutdown Phase ===")
    
    # 停止后台任务
    print("1. Stopping background tasks...")
    app.state.background_task.cancel()
    try:
        await app.state.background_task
    except asyncio.CancelledError:
        pass
    
    # 并行关闭资源（无依赖关系）
    print("2. Closing connections...")
    await asyncio.gather(
        app.state.db.close(),
        app.state.cache.close(),
        app.state.queue.close(),
        app.state.storage.close(),
        return_exceptions=True,  # 不让一个失败影响其他
    )
    
    print("=== Shutdown Complete ===")

app = FastAPI(lifespan=lifespan)
```

---

### **7.1.4 异常处理**

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import logging

logger = logging.getLogger(__name__)

@asynccontextmanager
async def lifespan(app: FastAPI):
    """带异常处理的生命周期"""
    resources = {}
    
    try:
        # 启动阶段
        logger.info("Starting application...")
        
        # 初始化数据库
        try:
            resources['db'] = await create_database_pool()
            app.state.db = resources['db']
            logger.info("Database connected")
        except Exception as e:
            logger.error(f"Failed to connect to database: {e}")
            raise  # 启动失败，不继续
        
        # 初始化缓存（可选，失败不影响启动）
        try:
            resources['cache'] = await create_redis_client()
            app.state.cache = resources['cache']
            logger.info("Redis connected")
        except Exception as e:
            logger.warning(f"Failed to connect to Redis: {e}")
            app.state.cache = None  # 降级处理
        
        logger.info("Application started successfully")
        
        yield
        
    except Exception as e:
        logger.error(f"Startup failed: {e}")
        # 清理已初始化的资源
        await cleanup_resources(resources)
        raise
    
    finally:
        # 关闭阶段（总是执行）
        logger.info("Shutting down application...")
        await cleanup_resources(resources)
        logger.info("Application stopped")

async def cleanup_resources(resources: dict):
    """清理资源"""
    for name, resource in resources.items():
        try:
            if hasattr(resource, 'close'):
                await resource.close()
            logger.info(f"{name} closed")
        except Exception as e:
            logger.error(f"Failed to close {name}: {e}")

app = FastAPI(lifespan=lifespan)
```

---

### **7.1.5 依赖注入与生命周期**

```python
from fastapi import FastAPI, Depends, Request

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动
    app.state.db = await create_database_pool()
    app.state.counter = 0
    
    yield
    
    # 关闭
    await app.state.db.close()

app = FastAPI(lifespan=lifespan)

# 依赖：获取数据库连接
async def get_db(request: Request):
    """从 app.state 获取数据库"""
    return request.app.state.db

# 依赖：获取计数器
async def get_counter(request: Request):
    """从 app.state 获取计数器"""
    request.app.state.counter += 1
    return request.app.state.counter

# 在路由中使用
@app.get("/users")
async def get_users(
    db = Depends(get_db),
    counter = Depends(get_counter),
):
    """使用生命周期中初始化的资源"""
    print(f"Request #{counter}")
    users = await db.fetch("SELECT * FROM users")
    return {"users": users, "request_count": counter}
```

---

## 7.2 **AutoGPT中的生命周期管理实战分析**

### **7.2.1 主应用生命周期（lifespan_context）**

AutoGPT 的 REST API 使用复杂的生命周期管理：

```python
# backend/server/rest_api.py

import contextlib
from fastapi import FastAPI

@contextlib.asynccontextmanager
async def lifespan_context(app: FastAPI):
    """
    AutoGPT 主应用生命周期管理
    
    启动阶段：
    1. 验证认证设置
    2. 连接数据库
    3. 配置线程池
    4. 初始化 Blocks
    5. 运行数据迁移
    6. 初始化 LaunchDarkly
    
    关闭阶段：
    1. 关闭云存储处理器
    2. 断开数据库连接
    """
    
    # =================== 启动阶段 ===================
    
    # 1. 验证认证设置
    verify_auth_settings()
    #   ^^^^^^^^^^^^^^^^^
    #   确保 JWT 密钥等配置正确
    
    # 2. 连接数据库（Prisma）
    await backend.data.db.connect()
    #     ^^^^^^^^^^^^^^^^^^^^
    #     建立数据库连接池
    
    # 3. 配置 FastAPI 线程池
    # CRITICAL: FastAPI 自动在线程池中运行所有同步函数：
    # - 使用 'def' 定义的端点（非 async def）
    # - 使用 'def' 定义的依赖函数（非 async def）
    # - 手动调用 run_in_threadpool() 的代码（如 JWT 解码）
    # 默认池大小只有 40 个线程，高并发时会成为瓶颈
    config = backend.util.settings.Config()
    try:
        import anyio.to_thread
        
        # 设置线程池大小（例如 200）
        anyio.to_thread.current_default_thread_limiter().total_tokens = (
            config.fastapi_thread_pool_size
        )
        logger.info(
            f"Thread pool size set to {config.fastapi_thread_pool_size} "
            f"for sync endpoint/dependency performance"
        )
    except (ImportError, AttributeError) as e:
        logger.warning(f"Could not configure thread pool size: {e}")
        # 继续运行（使用默认池大小）
    
    # 4. 初始化 SDK 和 Blocks
    from backend.sdk.registry import AutoRegistry
    
    # 修补集成（确保 SDK 自动注册）
    AutoRegistry.patch_integrations()
    
    # 初始化所有 Block 类（226+ 个 Blocks）
    await backend.data.block.initialize_blocks()
    #     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #     加载和注册所有 Block 定义
    
    # 5. 运行数据迁移
    # 迁移用户集成数据（加密敏感信息）
    await backend.data.user.migrate_and_encrypt_user_integrations()
    
    # 修复 LLM 提供商凭证
    await backend.data.graph.fix_llm_provider_credentials()
    
    # 迁移 LLM 模型（更新到 GPT-4o）
    await backend.data.graph.migrate_llm_models(LlmModel.GPT4O)
    
    # 迁移旧版触发式图
    await backend.integrations.webhooks.utils.migrate_legacy_triggered_graphs()
    
    # 6. 初始化 LaunchDarkly（特征标志）
    with launch_darkly_context():
        # LaunchDarkly 在此作用域内可用
        yield  # ← 应用运行中
        #      ^^^^^ 在这里，应用正常处理请求
    
    # =================== 关闭阶段 ===================
    
    # 1. 关闭云存储处理器
    try:
        await shutdown_cloud_storage_handler()
    except Exception as e:
        logger.warning(f"Error shutting down cloud storage handler: {e}")
        # 不中断关闭流程
    
    # 2. 断开数据库连接
    await backend.data.db.disconnect()
    #     ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #     关闭所有数据库连接


# 创建应用，传入生命周期管理器
app = FastAPI(
    title="AutoGPT Agent Server",
    version="0.1",
    lifespan=lifespan_context,  # ← 使用自定义生命周期
    #        ^^^^^^^^^^^^^^^^
)
```

**启动流程时间线：**
```
T+0s    验证认证设置
T+0.1s  连接数据库（Prisma）
T+0.2s  配置线程池（200 线程）
T+0.5s  初始化 Blocks（226+ 个）
T+1.0s  运行数据迁移
T+1.5s  初始化 LaunchDarkly
T+2s    ✅ 应用就绪
```

---

### **7.2.2 LaunchDarkly 上下文管理（特征标志）**

```python
# backend/server/rest_api.py

from backend.util.feature_flag import initialize_launchdarkly, shutdown_launchdarkly

@contextlib.contextmanager
def launch_darkly_context():
    """
    LaunchDarkly 特征标志系统的生命周期管理
    
    只在非本地环境启用（生产/测试环境）
    """
    if settings.config.app_env != backend.util.settings.AppEnvironment.LOCAL:
        # 非本地环境：初始化 LaunchDarkly
        initialize_launchdarkly()
        #  ^^^^^^^^^^^^^^^^^^^^^^
        #  连接到 LaunchDarkly SDK
        
        try:
            yield  # LaunchDarkly 可用
        finally:
            # 确保关闭 LaunchDarkly
            shutdown_launchdarkly()
            #  ^^^^^^^^^^^^^^^^^^^^^
            #  断开连接，刷新指标
    else:
        # 本地环境：跳过 LaunchDarkly
        yield  # 直接继续（使用默认值）

# 使用
with launch_darkly_context():
    # 在这里，特征标志可用
    if await is_feature_enabled("new_ui"):
        use_new_ui()
```

**为什么分离 LaunchDarkly 上下文？**
- 🏠 **本地开发无需特征标志**：减少依赖
- 🔒 **隔离关闭逻辑**：即使关闭失败也不影响其他资源
- 🎯 **环境特定行为**：不同环境不同配置

---

### **7.2.3 WebSocket 服务生命周期**

```python
# backend/server/ws_api.py

from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio

@asynccontextmanager
async def lifespan(app: FastAPI):
    """
    WebSocket 服务的生命周期管理
    
    启动：启动事件广播器
    关闭：停止广播任务
    """
    # 获取连接管理器（全局单例）
    manager = get_connection_manager()
    
    # 启动事件广播器（后台任务）
    broadcast_task = asyncio.create_task(event_broadcaster(manager))
    #                ^^^^^^^^^^^^^^^^^^^
    #                创建异步任务
    
    logger.info("WebSocket event broadcaster started")
    
    yield  # WebSocket 服务运行中
    
    # 关闭广播任务
    logger.info("Stopping WebSocket event broadcaster...")
    broadcast_task.cancel()
    #             ^^^^^^^^
    #             取消任务
    
    try:
        await broadcast_task
    except asyncio.CancelledError:
        logger.info("Event broadcaster stopped")


# 事件广播器（后台任务）
@continuous_retry()  # 自动重试装饰器
async def event_broadcaster(manager: ConnectionManager):
    """
    从 Redis 监听执行事件，广播到所有订阅的 WebSocket 连接
    
    这是一个无限循环，直到被取消
    """
    event_queue = AsyncRedisExecutionEventBus()
    
    # 无限循环监听事件
    async for event in event_queue.listen("*"):
        #    ^^^^^^^^^^ 异步迭代器（阻塞直到有事件）
        
        # 向所有订阅者发送事件
        await manager.send_execution_update(event)


# 创建 WebSocket 应用
ws_app = FastAPI(
    title="AutoGPT WebSocket Server",
    lifespan=lifespan,  # ← 使用 WebSocket 生命周期
)
```

**后台任务模式：**
```
启动 → 创建 asyncio.Task → 任务在后台运行
                           ↓
                        监听 Redis 事件
                           ↓
                        广播到 WebSocket
                           ↓
关闭 → task.cancel() → 任务退出 → 清理完成
```

---

### **7.2.4. 数据库事务上下文**

```python
# backend/data/db.py

from contextlib import asynccontextmanager
from prisma import Prisma

# 全局 Prisma 实例
prisma = Prisma(
    auto_register=True,
    datasource={"url": DATABASE_URL},
)

# 事务超时常量
TRANSACTION_TIMEOUT = 30000  # 30 秒

@asynccontextmanager
async def transaction(timeout: int = TRANSACTION_TIMEOUT):
    """
    数据库事务上下文管理器
    
    Args:
        timeout: 事务超时时间（毫秒）
    
    使用：
        async with transaction() as tx:
            await tx.user.create(...)
            await tx.post.create(...)
        # 自动提交或回滚
    """
    async with prisma.tx(timeout=timeout) as tx:
        yield tx
        # 退出时自动提交（成功）或回滚（异常）


@asynccontextmanager
async def locked_transaction(key: str, timeout: int = TRANSACTION_TIMEOUT):
    """
    带锁的事务上下文管理器
    
    在事务期间持有一个咨询锁（advisory lock）
    防止并发修改同一资源
    
    Args:
        key: 锁的唯一标识
        timeout: 事务超时时间（毫秒）
    
    使用：
        async with locked_transaction(f"user:{user_id}") as tx:
            user = await tx.user.find_unique(where={"id": user_id})
            user.balance += 100
            await tx.user.update(...)
        # 自动释放锁
    """
    async with transaction(timeout=timeout) as tx:
        # 获取 PostgreSQL 咨询锁
        lock_hash = hash(key) % (2**31)  # 转换为 32 位整数
        await tx.execute_raw(f"SELECT pg_advisory_xact_lock({lock_hash})")
        #                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #                    事务级咨询锁（事务结束时自动释放）
        
        yield tx
        # 事务提交/回滚时，锁自动释放


# 使用示例
async def transfer_credits(from_user: str, to_user: str, amount: int):
    """转账（需要事务保证原子性）"""
    async with transaction() as tx:
        # 扣除发送方余额
        await tx.execute_raw(
            "UPDATE users SET credits = credits - ? WHERE id = ?",
            amount, from_user
        )
        
        # 增加接收方余额
        await tx.execute_raw(
            "UPDATE users SET credits = credits + ? WHERE id = ?",
            amount, to_user
        )
        
        # 记录交易
        await tx.transaction.create(
            data={
                "from_user_id": from_user,
                "to_user_id": to_user,
                "amount": amount,
            }
        )
    
    # 退出时自动提交（成功）或回滚（失败）
```

---

### **7.2.5 分布式锁上下文**

```python
# backend/executor/manager.py

from contextlib import asynccontextmanager
from redis.asyncio.lock import Lock as AsyncRedisLock
import redis

@asynccontextmanager
async def synchronized(key: str, timeout: int = 60):
    """
    分布式锁上下文管理器（基于 Redis）
    
    确保同一时间只有一个进程/线程执行某段代码
    
    Args:
        key: 锁的唯一标识
        timeout: 锁的超时时间（秒）
    
    使用：
        async with synchronized(f"upsert_input-{node_id}"):
            # 在锁保护下执行
            await critical_operation()
    """
    # 获取 Redis 客户端
    r = await redis.get_redis_async()
    
    # 创建锁
    lock: AsyncRedisLock = r.lock(f"lock:{key}", timeout=timeout)
    #                      ^^^^^^
    #                      Redis 锁对象
    
    try:
        # 获取锁（阻塞直到成功）
        await lock.acquire()
        #         ^^^^^^^^^
        #         等待获取锁
        
        yield  # 在锁保护下执行代码
        
    finally:
        # 释放锁
        if await lock.locked() and await lock.owned():
            #                      ^^^^^^^^^^^^^^^^
            #                      确保锁属于当前进程
            try:
                await lock.release()
            except Exception as e:
                logger.error(f"Failed to release lock {key}: {e}")


# 使用示例
async def enqueue_next_nodes(...):
    """
    将下一个节点加入执行队列
    
    需要锁保护，避免并发添加重复输入
    """
    # 多个节点可能同时尝试注册同一个下游节点
    # 需要原子性操作
    async with synchronized(f"upsert_input-{next_node_id}-{graph_exec_id}"):
        # 在锁保护下执行
        next_node_exec_id, next_node_input = await db_client.upsert_execution_input(
            node_id=next_node_id,
            graph_exec_id=graph_exec_id,
            input_name=next_input_name,
            input_data=next_data,
        )
        
        await update_node_execution_status(
            exec_id=next_node_exec_id,
            status=ExecutionStatus.INCOMPLETE,
        )
    
    # 锁自动释放
```

**分布式锁的必要性：**
-  **多实例部署**：多个后端实例同时运行
-  **并发执行**：多个节点可能同时完成，尝试注册同一个下游节点
-  **原子操作**：确保 upsert 操作不会冲突

---

### **7.2.6 凭证管理器锁**

```python
# backend/integrations/creds_manager.py

from contextlib import asynccontextmanager

class IntegrationCredentialsManager:
    @asynccontextmanager
    async def _locked(self, user_id: str, credentials_id: str, *args: str):
        """
        凭证锁上下文管理器
        
        确保同一凭证同时只被一个 Block 使用
        防止：
        - 并发使用导致的 API 速率限制
        - 凭证状态不一致
        - OAuth token 刷新冲突
        """
        # 构建锁键
        lock_key = f"{user_id}:{credentials_id}"
        if args:
            lock_key += ":" + ":".join(args)
        
        # 获取分布式锁
        lock = await self._acquire_lock(lock_key)
        
        try:
            yield  # 在锁保护下使用凭证
        finally:
            # 确保释放锁
            await self._release_lock(lock)
    
    async def acquire(
        self, user_id: str, credentials_id: str
    ) -> tuple[Credentials, AsyncRedisLock]:
        """
        获取凭证并加锁
        
        Returns:
            (凭证对象, 锁对象)
        """
        # 从存储获取凭证
        credentials = await self.store.get_creds_by_id(user_id, credentials_id)
        
        # 获取分布式锁
        lock = await self._acquire_lock(f"{user_id}:{credentials_id}")
        
        return credentials, lock
    
    async def release(self, lock: AsyncRedisLock):
        """释放凭证锁"""
        if await lock.locked() and await lock.owned():
            await lock.release()


# 在 Block 执行中使用
async def execute_node(...):
    creds_lock = None
    
    try:
        # 获取凭证和锁
        credentials, creds_lock = await creds_manager.acquire(
            user_id, credentials_id
        )
        
        # 使用凭证执行 Block
        async for output in node_block.execute(credentials=credentials):
            yield output
    
    finally:
        # 确保释放锁
        if creds_lock:
            await creds_manager.release(creds_lock)
```

---

### **7.2.7 云存储处理器生命周期**

```python
# backend/util/cloud_storage.py

import asyncio

# 全局实例
_cloud_storage_handler: CloudStorageHandler | None = None
_cleanup_lock = asyncio.Lock()

async def get_cloud_storage_handler() -> CloudStorageHandler:
    """
    获取云存储处理器（全局单例）
    
    使用异步锁确保只创建一次
    """
    global _cloud_storage_handler
    
    if _cloud_storage_handler is None:
        async with _cleanup_lock:  # 防止并发创建
            if _cloud_storage_handler is None:
                _cloud_storage_handler = CloudStorageHandler()
                await _cloud_storage_handler.initialize()
    
    return _cloud_storage_handler

async def shutdown_cloud_storage_handler():
    """
    关闭云存储处理器
    
    在应用关闭时调用
    """
    global _cloud_storage_handler
    
    if _cloud_storage_handler is not None:
        async with _cleanup_lock:
            if _cloud_storage_handler is not None:
                await _cloud_storage_handler.cleanup()
                _cloud_storage_handler = None


# 在生命周期中使用
@asynccontextmanager
async def lifespan_context(app: FastAPI):
    # 启动（懒加载，首次使用时初始化）
    
    yield
    
    # 关闭
    try:
        await shutdown_cloud_storage_handler()
    except Exception as e:
        logger.warning(f"Error shutting down cloud storage: {e}")
```

---

### **7.2.8 优雅关闭流程**

AutoGPT 的完整关闭流程：

```python
# 1. 接收关闭信号（SIGTERM/SIGINT）
#    ↓
# 2. lifespan_context 退出 yield
#    ↓
# 3. LaunchDarkly 关闭（with 块结束）
#    - 刷新指标
#    - 断开连接
#    ↓
# 4. 云存储处理器关闭
#    - 上传缓冲数据
#    - 关闭客户端
#    ↓
# 5. 数据库断开连接
#    - 完成进行中的查询
#    - 关闭连接池
#    ↓
# 6. 应用完全关闭

# 时间线：
# T+0s     接收 SIGTERM
# T+0.1s   LaunchDarkly 关闭
# T+0.5s   云存储关闭
# T+1.0s   数据库关闭
# T+1.5s   ✅ 应用完全停止
```

---

## 7.3 **AutoGPT 生命周期模式总结**

### **7.3.1 分层生命周期管理**

```python
# 最外层：主应用生命周期
@asynccontextmanager
async def lifespan_context(app: FastAPI):
    # 数据库连接
    # Blocks 初始化
    # 数据迁移
    
    # 中间层：特征标志生命周期
    with launch_darkly_context():
        yield  # 应用运行
    
    # 清理资源

# 内层：单个资源的生命周期
@asynccontextmanager
async def synchronized(key: str):
    lock = await acquire_lock(key)
    try:
        yield
    finally:
        await release_lock(lock)
```

---

### **7.3.2 启动阶段检查清单**

```python
 1. 验证配置（认证设置）
 2. 连接外部服务（数据库、Redis）
 3. 配置运行时参数（线程池大小）
 4. 初始化内部组件（Blocks、SDK）
 5. 运行数据迁移
 6. 启动后台任务
 7. 初始化特征标志
```

---

### **7.3.3 关闭阶段检查清单**

```python
 1. 停止接收新请求
 2. 取消后台任务
 3. 等待进行中的请求完成
 4. 关闭特征标志客户端
 5. 关闭云存储处理器
 6. 关闭数据库连接
 7. 释放所有锁
```

---

### **7.3.4 上下文管理器最佳实践**

```python
# ✅ 好的做法
@asynccontextmanager
async def resource_manager():
    resource = await acquire()
    try:
        yield resource
    finally:
        await release(resource)  # 总是释放

# ❌ 避免的做法
@asynccontextmanager
async def bad_manager():
    resource = await acquire()
    yield resource
    await release(resource)  # 异常时不释放！
```

---

### **7.3.5 异常处理策略**

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    resources = []
    
    try:
        # 启动：失败则中止
        db = await connect_db()
        resources.append(db)
        
        cache = await connect_cache()
        resources.append(cache)
        
        yield
        
    except Exception as e:
        logger.error(f"Startup failed: {e}")
        raise  # 启动失败，不运行应用
    
    finally:
        # 关闭：总是执行
        for resource in reversed(resources):
            try:
                await resource.close()
            except Exception as e:
                logger.error(f"Cleanup failed: {e}")
                # 继续清理其他资源
```

---

# 8 WebSocket 实时通信

## 8.1 **基础复习**

### **8.1.1 WebSocket 协议**

WebSocket 是一种在单个 TCP 连接上进行全双工通信的协议：

```
传统 HTTP（半双工）：
客户端 → 请求 → 服务器
客户端 ← 响应 ← 服务器
（每次通信都需要重新建立连接）

WebSocket（全双工）：
客户端 ⇄ 持久连接 ⇄ 服务器
（连接保持打开，双向实时通信）
```

**WebSocket 握手过程：**

```http
# 1. 客户端发起 HTTP Upgrade 请求
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Version: 13

# 2. 服务器响应升级
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=

# 3. 连接建立，开始 WebSocket 通信
```

---

### **8.1.2 全双工通信**

```python
# 模拟 WebSocket 通信

import asyncio
from typing import Set

class MockWebSocket:
    def __init__(self, name: str):
        self.name = name
        self.connected = True
        self.message_queue = asyncio.Queue()
    
    async def send(self, message: str):
        """发送消息"""
        print(f"{self.name} 发送: {message}")
    
    async def receive(self) -> str:
        """接收消息（阻塞直到有消息）"""
        message = await self.message_queue.get()
        return message
    
    async def close(self):
        """关闭连接"""
        self.connected = False
        print(f"{self.name} 连接关闭")

# 服务器端
async def server_handler(ws: MockWebSocket):
    """服务器处理客户端连接"""
    print(f"客户端 {ws.name} 已连接")
    
    # 发送欢迎消息
    await ws.send("欢迎连接到服务器！")
    
    # 持续接收消息
    while ws.connected:
        try:
            message = await ws.receive()
            print(f"{ws.name} 接收: {message}")
            
            # 处理消息并回复
            response = f"服务器收到: {message}"
            await ws.send(response)
        except Exception as e:
            print(f"错误: {e}")
            break
    
    await ws.close()

# 客户端
async def client_handler(ws: MockWebSocket):
    """客户端发送消息"""
    # 接收欢迎消息
    welcome = await ws.receive()
    print(f"收到欢迎: {welcome}")
    
    # 发送消息
    messages = ["Hello", "How are you?", "Goodbye"]
    for msg in messages:
        await ws.send(msg)
        await asyncio.sleep(1)
        
        # 接收回复
        response = await ws.receive()
        print(f"收到回复: {response}")

# 运行示例
async def main():
    ws = MockWebSocket("Client-1")
    
    # 模拟双向通信
    server_task = asyncio.create_task(server_handler(ws))
    client_task = asyncio.create_task(client_handler(ws))
    
    await asyncio.gather(server_task, client_task)

# 输出：
# 客户端 Client-1 已连接
# Client-1 发送: 欢迎连接到服务器！
# 收到欢迎: 欢迎连接到服务器！
# Client-1 发送: Hello
# Client-1 接收: Hello
# Client-1 发送: 服务器收到: Hello
# ...
```

---

### **8.1.3 连接状态管理**

```python
from enum import Enum
from typing import Dict, Set
import asyncio

class ConnectionState(Enum):
    CONNECTING = "connecting"
    CONNECTED = "connected"
    DISCONNECTING = "disconnecting"
    DISCONNECTED = "disconnected"

class WebSocketManager:
    """WebSocket 连接管理器"""
    
    def __init__(self):
        self.connections: Dict[str, MockWebSocket] = {}
        self.states: Dict[str, ConnectionState] = {}
    
    async def connect(self, client_id: str, ws: MockWebSocket):
        """连接客户端"""
        self.states[client_id] = ConnectionState.CONNECTING
        self.connections[client_id] = ws
        
        # 模拟连接过程
        await asyncio.sleep(0.1)
        
        self.states[client_id] = ConnectionState.CONNECTED
        print(f"✅ {client_id} 已连接")
    
    async def disconnect(self, client_id: str):
        """断开客户端"""
        if client_id in self.connections:
            self.states[client_id] = ConnectionState.DISCONNECTING
            
            ws = self.connections[client_id]
            await ws.close()
            
            del self.connections[client_id]
            self.states[client_id] = ConnectionState.DISCONNECTED
            print(f"❌ {client_id} 已断开")
    
    async def broadcast(self, message: str):
        """广播消息到所有连接"""
        for client_id, ws in self.connections.items():
            if self.states[client_id] == ConnectionState.CONNECTED:
                await ws.send(f"[广播] {message}")
    
    def get_active_count(self) -> int:
        """获取活跃连接数"""
        return sum(
            1 for state in self.states.values()
            if state == ConnectionState.CONNECTED
        )

# 使用
async def main():
    manager = WebSocketManager()
    
    # 连接多个客户端
    ws1 = MockWebSocket("Client-1")
    ws2 = MockWebSocket("Client-2")
    
    await manager.connect("client-1", ws1)
    await manager.connect("client-2", ws2)
    
    print(f"活跃连接: {manager.get_active_count()}")
    
    # 广播消息
    await manager.broadcast("服务器通知：系统更新")
    
    # 断开一个客户端
    await manager.disconnect("client-1")
    
    print(f"活跃连接: {manager.get_active_count()}")

# 输出：
# ✅ client-1 已连接
# ✅ client-2 已连接
# 活跃连接: 2
# Client-1 发送: [广播] 服务器通知：系统更新
# Client-2 发送: [广播] 服务器通知：系统更新
# Client-1 连接关闭
# ❌ client-1 已断开
# 活跃连接: 1
```

---

### **8.1.4 心跳机制（Ping/Pong）**

```python
import asyncio
import time

class WebSocketWithHeartbeat:
    """带心跳的 WebSocket 连接"""
    
    def __init__(self, name: str, timeout: int = 30):
        self.name = name
        self.connected = True
        self.timeout = timeout
        self.last_pong = time.time()
    
    async def send_ping(self):
        """发送 Ping"""
        print(f"{self.name} 发送 PING")
        await asyncio.sleep(0.1)  # 模拟网络延迟
    
    async def receive_pong(self):
        """接收 Pong"""
        print(f"{self.name} 收到 PONG")
        self.last_pong = time.time()
    
    async def heartbeat(self):
        """心跳循环"""
        while self.connected:
            # 发送 Ping
            await self.send_ping()
            
            # 等待 Pong
            await asyncio.sleep(0.2)
            await self.receive_pong()
            
            # 检查超时
            if time.time() - self.last_pong > self.timeout:
                print(f"❌ {self.name} 心跳超时，断开连接")
                self.connected = False
                break
            
            # 等待下一次心跳
            await asyncio.sleep(10)  # 每 10 秒一次心跳
    
    async def close(self):
        """关闭连接"""
        self.connected = False
        print(f"{self.name} 连接关闭")

# 使用
async def main():
    ws = WebSocketWithHeartbeat("Client-1", timeout=30)
    
    # 启动心跳
    heartbeat_task = asyncio.create_task(ws.heartbeat())
    
    # 运行 3 次心跳后关闭
    await asyncio.sleep(35)
    await ws.close()
    
    try:
        await heartbeat_task
    except asyncio.CancelledError:
        pass

# 输出：
# Client-1 发送 PING
# Client-1 收到 PONG
# (等待 10 秒)
# Client-1 发送 PING
# Client-1 收到 PONG
# (等待 10 秒)
# Client-1 发送 PING
# Client-1 收到 PONG
# Client-1 连接关闭
```

---

## 8.2 **FastAPI 特性**

###  **8.2.1 @app.websocket() 装饰器**

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket 端点"""
    # 1. 接受连接
    await websocket.accept()
    
    # 2. 通信循环
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            
            # 处理消息
            response = f"收到消息: {data}"
            
            # 发送响应
            await websocket.send_text(response)
    
    except WebSocketDisconnect:
        # 3. 处理断开连接
        print("客户端断开连接")

# 客户端 JavaScript 示例：
"""
const ws = new WebSocket("ws://localhost:8000/ws");

ws.onopen = () => {
    console.log("连接已建立");
    ws.send("Hello Server!");
};

ws.onmessage = (event) => {
    console.log("收到消息:", event.data);
};

ws.onclose = () => {
    console.log("连接已关闭");
};
"""
```

---

### **8.2.1 WebSocket 对象方法**

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket 所有方法示例"""
    
    # ===== 连接管理 =====
    
    # 接受连接
    await websocket.accept()
    #     ^^^^^^^^^^^^^^^^
    #     完成 WebSocket 握手
    
    # 获取客户端信息
    client_host = websocket.client.host
    client_port = websocket.client.port
    print(f"客户端连接: {client_host}:{client_port}")
    
    # 获取查询参数
    token = websocket.query_params.get("token")
    print(f"认证 Token: {token}")
    
    # ===== 接收数据 =====
    
    try:
        # 接收文本消息
        text = await websocket.receive_text()
        
        # 接收 JSON 消息
        data = await websocket.receive_json()
        
        # 接收二进制消息
        binary = await websocket.receive_bytes()
        
        # 通用接收（返回字典）
        message = await websocket.receive()
        # 返回: {"type": "websocket.receive", "text": "...", ...}
        
        # ===== 发送数据 =====
        
        # 发送文本消息
        await websocket.send_text("Hello")
        
        # 发送 JSON 消息
        await websocket.send_json({"status": "ok"})
        
        # 发送二进制消息
        await websocket.send_bytes(b"binary data")
        
        # ===== 关闭连接 =====
        
        # 正常关闭
        await websocket.close()
        
        # 带状态码关闭
        await websocket.close(code=1000, reason="Normal closure")
        
    except WebSocketDisconnect as e:
        print(f"客户端断开: {e.code}")
```

---

### **8.2.3 WebSocketDisconnect 异常**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
import asyncio

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """处理断开连接"""
    await websocket.accept()
    
    try:
        while True:
            # 接收消息（可能抛出 WebSocketDisconnect）
            data = await websocket.receive_text()
            await websocket.send_text(f"Echo: {data}")
    
    except WebSocketDisconnect as e:
        # 客户端断开连接
        print(f"客户端断开连接")
        print(f"  状态码: {e.code}")
        print(f"  原因: {e.reason}")
        
        # 常见状态码：
        # 1000: 正常关闭
        # 1001: 端点离开
        # 1002: 协议错误
        # 1003: 不支持的数据类型
        # 1006: 异常关闭（没有发送关闭帧）
        # 1011: 服务器错误
    
    except Exception as e:
        # 其他错误
        print(f"错误: {e}")
        await websocket.close(code=1011, reason="Internal error")
```

---

### **8.2.4 WebSocket 依赖注入**

```python
from fastapi import FastAPI, WebSocket, Depends, Query
from typing import Optional

app = FastAPI()

# 依赖：验证 Token
async def verify_token(token: Optional[str] = Query(None)):
    """验证认证 Token"""
    if not token:
        raise ValueError("Missing token")
    
    # 验证 Token（简化示例）
    if token != "secret":
        raise ValueError("Invalid token")
    
    return token

# 依赖：获取当前用户
async def get_current_user(token: str = Depends(verify_token)):
    """从 Token 获取用户信息"""
    # 实际应用中从数据库查询
    return {"id": "user123", "name": "Alice", "token": token}

@app.websocket("/ws")
async def websocket_endpoint(
    websocket: WebSocket,
    user: dict = Depends(get_current_user),
    #            ^^^^^^^^^^^^^^^^^^^^^^^^^^
    #            注入用户信息
):
    """带认证的 WebSocket 端点"""
    await websocket.accept()
    
    # 发送欢迎消息
    await websocket.send_json({
        "type": "welcome",
        "user": user["name"],
    })
    
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"{user['name']}: {data}")
    
    except WebSocketDisconnect:
        print(f"用户 {user['id']} 断开连接")

# 客户端连接：
# ws://localhost:8000/ws?token=secret
```

---

### **8.2.5 连接管理器模式**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List

app = FastAPI()

class ConnectionManager:
    """WebSocket 连接管理器"""
    
    def __init__(self):
        self.active_connections: List[WebSocket] = []
    
    async def connect(self, websocket: WebSocket):
        """连接新客户端"""
        await websocket.accept()
        self.active_connections.append(websocket)
        print(f"新连接，当前连接数: {len(self.active_connections)}")
    
    def disconnect(self, websocket: WebSocket):
        """断开客户端"""
        self.active_connections.remove(websocket)
        print(f"连接断开，当前连接数: {len(self.active_connections)}")
    
    async def send_personal_message(self, message: str, websocket: WebSocket):
        """发送个人消息"""
        await websocket.send_text(message)
    
    async def broadcast(self, message: str):
        """广播消息到所有连接"""
        for connection in self.active_connections:
            await connection.send_text(message)

# 创建全局管理器
manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: str):
    """支持多个客户端"""
    await manager.connect(websocket)
    
    # 通知所有人
    await manager.broadcast(f"客户端 {client_id} 加入聊天室")
    
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            
            # 个人消息
            await manager.send_personal_message(f"你说: {data}", websocket)
            
            # 广播给其他人
            await manager.broadcast(f"{client_id} 说: {data}")
    
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"客户端 {client_id} 离开聊天室")
```

---

##  8.3 **AutoGPT中的 WebSocke 实战分析**

### **8.3.1  消息格式设计（WSMessage）**

AutoGPT 定义了统一的 WebSocket 消息格式：

```python
# backend/server/model.py

from enum import Enum
from typing import Any, Optional
import pydantic

class WSMethod(Enum):
    """WebSocket 消息方法枚举"""
    SUBSCRIBE_GRAPH_EXEC = "subscribe_graph_execution"
    #                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                      订阅单个图执行的更新
    
    SUBSCRIBE_GRAPH_EXECS = "subscribe_graph_executions"
    #                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                       订阅图的所有执行
    
    UNSUBSCRIBE = "unsubscribe"
    #             ^^^^^^^^^^^^
    #             取消订阅
    
    GRAPH_EXECUTION_EVENT = "graph_execution_event"
    #                       ^^^^^^^^^^^^^^^^^^^^^
    #                       图执行事件（服务器→客户端）
    
    NODE_EXECUTION_EVENT = "node_execution_event"
    #                      ^^^^^^^^^^^^^^^^^^^^
    #                      节点执行事件（服务器→客户端）
    
    ERROR = "error"
    #       ^^^^^
    #       错误消息
    
    HEARTBEAT = "heartbeat"
    #           ^^^^^^^^^
    #           心跳检测


class WSMessage(pydantic.BaseModel):
    """统一的 WebSocket 消息格式"""
    method: WSMethod
    #       ^^^^^^^^
    #       消息类型（必需）
    
    data: Optional[dict[str, Any] | list[Any] | str] = None
    #     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #     消息数据（可选）
    
    success: bool | None = None
    #        ^^^^^^^^^^
    #        操作是否成功（响应消息）
    
    channel: str | None = None
    #        ^^^^^^^^^^
    #        订阅频道标识
    
    error: str | None = None
    #      ^^^^^^^^^^
    #      错误信息


# 订阅请求模型
class WSSubscribeGraphExecutionRequest(pydantic.BaseModel):
    """订阅单个图执行"""
    graph_exec_id: str
    #              ^^^
    #              图执行ID

class WSSubscribeGraphExecutionsRequest(pydantic.BaseModel):
    """订阅图的所有执行"""
    graph_id: str
    #        ^^^
    #        图ID
```

**消息示例：**

```javascript
// 客户端 → 服务器：订阅图执行
{
  "method": "subscribe_graph_execution",
  "data": {
    "graph_exec_id": "exec_123"
  }
}

// 服务器 → 客户端：订阅成功
{
  "method": "subscribe_graph_execution",
  "success": true,
  "channel": "user_456|graph_exec#exec_123"
}

// 服务器 → 客户端：节点执行事件
{
  "method": "node_execution_event",
  "channel": "user_456|graph_exec#exec_123",
  "data": {
    "node_id": "node_789",
    "status": "running",
    "output": {...}
  }
}

// 服务器 → 客户端：错误
{
  "method": "error",
  "success": false,
  "error": "Invalid message format"
}

// 客户端 → 服务器：心跳
{
  "method": "heartbeat"
}

// 服务器 → 客户端：心跳响应
{
  "method": "heartbeat",
  "data": "pong",
  "success": true
}
```

---

### **8.3.2 ConnectionManager（连接管理器）**

AutoGPT 使用发布/订阅模式管理 WebSocket 连接：

```python
# backend/server/conn_manager.py

from typing import Dict, Set
from fastapi import WebSocket

class ConnectionManager:
    """
    WebSocket 连接管理器
    
    功能：
    - 管理活跃连接
    - 发布/订阅模式
    - 消息广播
    """
    
    def __init__(self):
        # 所有活跃的 WebSocket 连接
        self.active_connections: Set[WebSocket] = set()
        
        # 订阅关系：频道 → WebSocket 集合
        self.subscriptions: Dict[str, Set[WebSocket]] = {}
        #                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #                   频道映射到订阅者集合
    
    async def connect_socket(self, websocket: WebSocket):
        """连接新的 WebSocket"""
        await websocket.accept()
        #     ^^^^^^^^^^^^^^^^^^
        #     完成 WebSocket 握手
        
        self.active_connections.add(websocket)
        #    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #    添加到活跃连接集合
    
    def disconnect_socket(self, websocket: WebSocket):
        """断开 WebSocket 连接"""
        # 从活跃连接中移除
        self.active_connections.remove(websocket)
        
        # 从所有订阅中移除
        for subscribers in self.subscriptions.values():
            subscribers.discard(websocket)
            #           ^^^^^^^
            #           安全移除（不存在也不报错）
    
    # ========== 订阅管理 ==========
    
    async def subscribe_graph_exec(
        self, *, user_id: str, graph_exec_id: str, websocket: WebSocket
    ) -> str:
        """
        订阅单个图执行的更新
        
        返回：频道键
        """
        channel_key = _graph_exec_channel_key(user_id, graph_exec_id=graph_exec_id)
        #             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #             生成频道键：user_456|graph_exec#exec_123
        
        return await self._subscribe(channel_key, websocket)
    
    async def subscribe_graph_execs(
        self, *, user_id: str, graph_id: str, websocket: WebSocket
    ) -> str:
        """
        订阅图的所有执行
        
        返回：频道键
        """
        channel_key = _graph_execs_channel_key(user_id, graph_id=graph_id)
        #             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #             生成频道键：user_456|graph#graph_789|executions
        
        return await self._subscribe(channel_key, websocket)
    
    async def unsubscribe_graph_exec(
        self, *, user_id: str, graph_exec_id: str, websocket: WebSocket
    ) -> str | None:
        """取消订阅图执行"""
        channel_key = _graph_exec_channel_key(user_id, graph_exec_id=graph_exec_id)
        return await self._unsubscribe(channel_key, websocket)
    
    async def _subscribe(self, channel_key: str, websocket: WebSocket) -> str:
        """内部：订阅频道"""
        # 创建频道（如果不存在）
        if channel_key not in self.subscriptions:
            self.subscriptions[channel_key] = set()
        
        # 添加订阅者
        self.subscriptions[channel_key].add(websocket)
        
        return channel_key
    
    async def _unsubscribe(
        self, channel_key: str, websocket: WebSocket
    ) -> str | None:
        """内部：取消订阅"""
        if channel_key in self.subscriptions:
            # 移除订阅者
            self.subscriptions[channel_key].discard(websocket)
            
            # 如果频道没有订阅者，删除频道
            if not self.subscriptions[channel_key]:
                del self.subscriptions[channel_key]
            
            return channel_key
        return None
    
    # ========== 消息发送 ==========
    
    async def send_execution_update(
        self, exec_event: GraphExecutionEvent | NodeExecutionEvent
    ) -> int:
        """
        发送执行更新到订阅者
        
        返回：成功发送的消息数量
        """
        # 确定执行 ID
        graph_exec_id = (
            exec_event.id
            if isinstance(exec_event, GraphExecutionEvent)
            else exec_event.graph_exec_id
        )
        
        n_sent = 0
        
        # 确定需要发送的频道
        channels: set[str] = {
            # 频道1：单个图执行的订阅者
            _graph_exec_channel_key(exec_event.user_id, graph_exec_id=graph_exec_id)
        }
        
        if isinstance(exec_event, GraphExecutionEvent):
            # 频道2：图的所有执行的订阅者
            channels.add(
                _graph_execs_channel_key(
                    exec_event.user_id, graph_id=exec_event.graph_id
                )
            )
        
        # 只发送到存在的频道
        for channel in channels.intersection(self.subscriptions.keys()):
            #              ^^^^^^^^^^^^^^^^^^^
            #              只处理有订阅者的频道
            
            # 构建消息
            message = WSMessage(
                method=_EVENT_TYPE_TO_METHOD_MAP[exec_event.event_type],
                channel=channel,
                data=exec_event.model_dump(),
            ).model_dump_json()
            
            # 发送到所有订阅者
            for connection in self.subscriptions[channel]:
                await connection.send_text(message)
                n_sent += 1
        
        return n_sent


# 频道键生成函数
def _graph_exec_channel_key(user_id: str, *, graph_exec_id: str) -> str:
    """生成单个图执行的频道键"""
    return f"{user_id}|graph_exec#{graph_exec_id}"
    #      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #      格式：user_456|graph_exec#exec_123

def _graph_execs_channel_key(user_id: str, *, graph_id: str) -> str:
    """生成图所有执行的频道键"""
    return f"{user_id}|graph#{graph_id}|executions"
    #      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #      格式：user_456|graph#graph_789|executions
```

**发布/订阅模式架构：**

```
Redis 事件总线
     ↓
事件广播器（event_broadcaster）
     ↓
ConnectionManager
     ↓
订阅映射：
{
  "user_1|graph_exec#exec_123": {ws1, ws2},
  "user_1|graph#graph_456|executions": {ws1, ws3},
  "user_2|graph_exec#exec_789": {ws4}
}
     ↓
发送消息到订阅者
```

---

### **8.3.3 WebSocket 认证**

```python
# backend/server/ws_api.py

from autogpt_libs.auth.jwt_utils import parse_jwt_token
from backend.data.user import DEFAULT_USER_ID

async def authenticate_websocket(websocket: WebSocket) -> str:
    """
    WebSocket 认证
    
    认证方式：通过查询参数传递 JWT token
    ws://localhost:8080/ws?token=eyJhbGc...
    
    返回：用户ID，如果认证失败返回空字符串
    """
    # 检查是否启用认证
    if not settings.config.enable_auth:
        # 未启用认证，使用默认用户
        return DEFAULT_USER_ID
    
    # 从查询参数获取 token
    token = websocket.query_params.get("token")
    if not token:
        # 没有 token，关闭连接
        await websocket.close(
            code=4001,  # 自定义错误码
            reason="Missing authentication token"
        )
        return ""
    
    try:
        # 解析 JWT token
        payload = parse_jwt_token(token)
        #         ^^^^^^^^^^^^^^^^^^^^^^
        #         验证签名、过期时间等
        
        # 提取用户 ID
        user_id = payload.get("sub")
        if not user_id:
            await websocket.close(
                code=4002,
                reason="Invalid token"
            )
            return ""
        
        return user_id
    
    except ValueError:
        # Token 无效
        await websocket.close(
            code=4003,
            reason="Invalid token"
        )
        return ""

# WebSocket 自定义错误码：
# 4000-4999: 客户端错误
# 4001: 缺少认证 token
# 4002: 无效的 token（缺少 user_id）
# 4003: 无效的 token（验证失败）
```

---

### **8.3.4 消息处理器（Handler Pattern）**

```python
# backend/server/ws_api.py

from typing import Protocol

class WSMessageHandler(Protocol):
    """消息处理器协议"""
    async def __call__(
        self,
        connection_manager: ConnectionManager,
        websocket: WebSocket,
        user_id: str,
        message: WSMessage,
    ): ...


# ========== 具体的处理器 ==========

async def handle_subscribe(
    connection_manager: ConnectionManager,
    websocket: WebSocket,
    user_id: str,
    message: WSMessage,
):
    """处理订阅消息"""
    # 验证数据
    if not message.data:
        await websocket.send_text(
            WSMessage(
                method=WSMethod.ERROR,
                success=False,
                error="Subscription data missing",
            ).model_dump_json()
        )
        return
    
    # 根据不同的订阅类型处理
    if message.method == WSMethod.SUBSCRIBE_GRAPH_EXEC:
        # 订阅单个图执行
        sub_req = WSSubscribeGraphExecutionRequest.model_validate(message.data)
        channel_key = await connection_manager.subscribe_graph_exec(
            user_id=user_id,
            graph_exec_id=sub_req.graph_exec_id,
            websocket=websocket,
        )
    
    elif message.method == WSMethod.SUBSCRIBE_GRAPH_EXECS:
        # 订阅图的所有执行
        sub_req = WSSubscribeGraphExecutionsRequest.model_validate(message.data)
        channel_key = await connection_manager.subscribe_graph_execs(
            user_id=user_id,
            graph_id=sub_req.graph_id,
            websocket=websocket,
        )
    
    else:
        raise ValueError(f"Unknown subscription method: {message.method}")
    
    # 发送成功响应
    logger.debug(f"New subscription on channel {channel_key} for user #{user_id}")
    await websocket.send_text(
        WSMessage(
            method=message.method,
            success=True,
            channel=channel_key,
        ).model_dump_json()
    )


async def handle_unsubscribe(
    connection_manager: ConnectionManager,
    websocket: WebSocket,
    user_id: str,
    message: WSMessage,
):
    """处理取消订阅"""
    if not message.data:
        await websocket.send_text(
            WSMessage(
                method=WSMethod.ERROR,
                success=False,
                error="Subscription data missing",
            ).model_dump_json()
        )
        return
    
    # 取消订阅
    unsub_req = WSSubscribeGraphExecutionRequest.model_validate(message.data)
    channel_key = await connection_manager.unsubscribe_graph_exec(
        user_id=user_id,
        graph_exec_id=unsub_req.graph_exec_id,
        websocket=websocket,
    )
    
    # 发送成功响应
    logger.debug(f"Removed subscription on channel {channel_key} for user #{user_id}")
    await websocket.send_text(
        WSMessage(
            method=WSMethod.UNSUBSCRIBE,
            success=True,
            channel=channel_key,
        ).model_dump_json()
    )


async def handle_heartbeat(
    connection_manager: ConnectionManager,
    websocket: WebSocket,
    user_id: str,
    message: WSMessage,
):
    """处理心跳"""
    await websocket.send_json({
        "method": WSMethod.HEARTBEAT.value,
        "data": "pong",
        "success": True,
    })


# ========== 处理器映射 ==========

_MSG_HANDLERS: dict[WSMethod, WSMessageHandler] = {
    WSMethod.HEARTBEAT: handle_heartbeat,
    WSMethod.SUBSCRIBE_GRAPH_EXEC: handle_subscribe,
    WSMethod.SUBSCRIBE_GRAPH_EXECS: handle_subscribe,
    WSMethod.UNSUBSCRIBE: handle_unsubscribe,
}
```

---

### **8.3.5 WebSocket 主路由**

```python
# backend/server/ws_api.py

@app.websocket("/ws")
async def websocket_router(
    websocket: WebSocket,
    manager: ConnectionManager = Depends(get_connection_manager)
    #                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #                            依赖注入连接管理器
):
    """
    WebSocket 主路由
    
    流程：
    1. 认证
    2. 连接
    3. 消息循环
    4. 断开
    """
    
    # ========== 1. 认证 ==========
    user_id = await authenticate_websocket(websocket)
    if not user_id:
        # 认证失败，连接已关闭
        return
    
    # ========== 2. 连接 ==========
    await manager.connect_socket(websocket)
    
    # 更新 Prometheus 指标
    update_websocket_connections(user_id, 1)
    #                                     ^^
    #                                     增加连接数
    
    try:
        # ========== 3. 消息循环 ==========
        while True:
            # 接收消息
            data = await websocket.receive_text()
            
            # 解析消息
            try:
                message = WSMessage.model_validate_json(data)
                #         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                #         Pydantic 验证
            
            except pydantic.ValidationError as e:
                # 消息格式错误
                logger.error(
                    "Invalid WebSocket message from user #%s: %s",
                    user_id, e
                )
                await websocket.send_text(
                    WSMessage(
                        method=WSMethod.ERROR,
                        success=False,
                        error="Invalid message format. Review the schema and retry",
                    ).model_dump_json()
                )
                continue
            
            # 处理消息
            try:
                if message.method in _MSG_HANDLERS:
                    # 调用对应的处理器
                    await _MSG_HANDLERS[message.method](
                        connection_manager=manager,
                        websocket=websocket,
                        user_id=user_id,
                        message=message,
                    )
                    continue
            
            except pydantic.ValidationError as e:
                # 数据验证错误
                logger.error(
                    "Validation error while handling '%s' for user #%s: %s",
                    message.method.value, user_id, e
                )
                await websocket.send_text(
                    WSMessage(
                        method=WSMethod.ERROR,
                        success=False,
                        error="Invalid message data. Refer to the API schema",
                    ).model_dump_json()
                )
                continue
            
            except Exception as e:
                # 其他错误
                logger.error(
                    f"Error while handling '{message.method.value}' "
                    f"for user #{user_id}: {e}"
                )
                continue
            
            # 未知消息类型
            if message.method == WSMethod.ERROR:
                logger.error(f"WebSocket Error message received: {message.data}")
            else:
                logger.warning(
                    f"Unknown WebSocket message type {message.method} received"
                )
                await websocket.send_text(
                    WSMessage(
                        method=WSMethod.ERROR,
                        success=False,
                        error="Message type is not processed by the server",
                    ).model_dump_json()
                )
    
    # ========== 4. 断开 ==========
    except WebSocketDisconnect:
        manager.disconnect_socket(websocket)
        logger.debug("WebSocket client disconnected")
    
    finally:
        # 更新 Prometheus 指标
        update_websocket_connections(user_id, -1)
        #                                     ^^^
        #                                     减少连接数
```

**消息处理流程：**

```
客户端消息
    ↓
receive_text()
    ↓
Pydantic 验证
    ↓
查找处理器
    ↓
调用处理器
    ↓
发送响应
```

---

### **8.3.6 事件广播器（后台任务）**

```python
# backend/server/ws_api.py

from backend.data.execution import AsyncRedisExecutionEventBus
from backend.util.retry import continuous_retry

@continuous_retry()
async def event_broadcaster(manager: ConnectionManager):
    """
    事件广播器（后台任务）
    
    功能：
    1. 从 Redis 监听执行事件
    2. 广播到所有订阅的 WebSocket 连接
    
    这是一个无限循环，使用 continuous_retry 自动重试
    """
    # 创建事件总线
    event_queue = AsyncRedisExecutionEventBus()
    
    # 监听所有事件（通配符）
    async for event in event_queue.listen("*"):
        #    ^^^^^^^^^^ 异步迭代器，阻塞直到有新事件
        
        # 发送事件到所有订阅者
        await manager.send_execution_update(event)
        #     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #     ConnectionManager 处理路由


# 生命周期中启动
@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期"""
    # 获取连接管理器
    manager = get_connection_manager()
    
    # 启动事件广播器（后台任务）
    fut = asyncio.create_task(event_broadcaster(manager))
    fut.add_done_callback(lambda _: logger.info("Event broadcaster stopped"))
    
    yield  # 应用运行中
    
    # 应用关闭时，任务会自动取消
```

**事件流：**

```
Graph 执行
    ↓
发布事件到 Redis
    ↓
event_broadcaster 监听
    ↓
ConnectionManager 路由
    ↓
发送到订阅的 WebSocket
    ↓
客户端接收更新
```

---

### **8.3.7 完整的通信示例**

```javascript
// 客户端 JavaScript 代码

class AutoGPTWebSocket {
    constructor(token) {
        this.ws = new WebSocket(`ws://localhost:8080/ws?token=${token}`);
        this.setupHandlers();
    }
    
    setupHandlers() {
        this.ws.onopen = () => {
            console.log('✅ WebSocket 已连接');
            
            // 订阅图执行
            this.subscribe({
                method: 'subscribe_graph_execution',
                data: {
                    graph_exec_id: 'exec_123'
                }
            });
            
            // 启动心跳
            this.startHeartbeat();
        };
        
        this.ws.onmessage = (event) => {
            const message = JSON.parse(event.data);
            console.log('📨 收到消息:', message);
            
            switch (message.method) {
                case 'subscribe_graph_execution':
                    if (message.success) {
                        console.log(`✅ 订阅成功: ${message.channel}`);
                    }
                    break;
                
                case 'node_execution_event':
                    console.log('⚙️ 节点执行:', message.data);
                    this.updateUI(message.data);
                    break;
                
                case 'graph_execution_event':
                    console.log('📊 图执行:', message.data);
                    break;
                
                case 'error':
                    console.error('❌ 错误:', message.error);
                    break;
                
                case 'heartbeat':
                    console.log('💓 心跳响应');
                    break;
            }
        };
        
        this.ws.onerror = (error) => {
            console.error('❌ WebSocket 错误:', error);
        };
        
        this.ws.onclose = (event) => {
            console.log(`🔌 连接关闭: ${event.code} - ${event.reason}`);
            this.stopHeartbeat();
        };
    }
    
    subscribe(message) {
        this.send(message);
    }
    
    unsubscribe(graphExecId) {
        this.send({
            method: 'unsubscribe',
            data: {
                graph_exec_id: graphExecId
            }
        });
    }
    
    send(message) {
        if (this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(message));
        }
    }
    
    startHeartbeat() {
        this.heartbeatInterval = setInterval(() => {
            this.send({ method: 'heartbeat' });
        }, 30000); // 每 30 秒
    }
    
    stopHeartbeat() {
        if (this.heartbeatInterval) {
            clearInterval(this.heartbeatInterval);
        }
    }
    
    updateUI(data) {
        // 更新界面
        document.getElementById('status').textContent = data.status;
        document.getElementById('output').textContent = JSON.stringify(data.output, null, 2);
    }
    
    close() {
        this.stopHeartbeat();
        this.ws.close();
    }
}

// 使用
const ws = new AutoGPTWebSocket('your-jwt-token');
```

---

## 8.4 **AutoGPT中 WebSocket 架构总结**

### **8.4.1 架构层次**

```
客户端（浏览器）
    ↕ WebSocket
WebSocket 服务器（ws_api.py）
    ↕
ConnectionManager（发布/订阅）
    ↕
Redis 事件总线
    ↕
执行引擎
```

---

### **8.4.2 关键设计模式**

```python
# 1. 发布/订阅模式
频道（Channel）→ 订阅者集合（Set[WebSocket]）

# 2. 处理器模式
WSMethod → WSMessageHandler

# 3. 单例模式
全局 ConnectionManager 实例

# 4. 依赖注入
WebSocket 端点注入 ConnectionManager
```

---

### **8.4.3 消息流**

```
客户端发送 → WebSocket 接收 → Pydantic 验证 → 处理器 → 响应

服务器事件 → Redis → 事件广播器 → ConnectionManager → WebSocket → 客户端
```

---

### **8.4.4 错误处理策略**

```python
# 级别 1：连接级错误（关闭连接）
- 认证失败（4001-4003）

# 级别 2：消息级错误（发送错误消息）
- 消息格式错误
- 数据验证失败
- 未知消息类型

# 级别 3：处理器级错误（记录日志，继续运行）
- 处理器内部异常
```

---

### **8.4.5 性能优化**

```python
# 1. 使用集合（Set）管理连接
O(1) 添加/删除

# 2. 频道过滤
只发送到有订阅者的频道

# 3. 异步 I/O
非阻塞发送

# 4. Prometheus 监控
追踪连接数和消息量
```

---

### 8.4.6 核心架构

```python
# 1. 消息格式
class WSMessage(BaseModel):
    method: WSMethod  # 消息类型
    data: Any         # 数据
    success: bool     # 成功标志
    channel: str      # 频道

# 2. ConnectionManager（发布/订阅）
class ConnectionManager:
    active_connections: Set[WebSocket]
    subscriptions: Dict[str, Set[WebSocket]]
    
    async def send_execution_update(event):
        # 发送到订阅者

# 3. WebSocket 路由
@app.websocket("/ws")
async def websocket_router(websocket, manager):
    user_id = await authenticate_websocket(websocket)
    await manager.connect_socket(websocket)
    
    while True:
        data = await websocket.receive_text()
        message = WSMessage.model_validate_json(data)
        await handle_message(message)

# 4. 事件广播（后台任务）
async def event_broadcaster(manager):
    event_queue = AsyncRedisExecutionEventBus()
    async for event in event_queue.listen("*"):
        await manager.send_execution_update(event)
```

### 8.4.7 **关键特性**

-  **JWT 认证**：通过查询参数 `?token=xxx`
-  **发布/订阅**：频道映射到 WebSocket 集合
-  **心跳机制**：保持连接活跃
-  **消息路由**：Handler 模式处理不同消息类型

---

# 9. 认证和授权

---

## 9.1 基础复习

### 9.1.1 认证 vs 授权

**认证 (Authentication)**: 验证用户身份
- "你是谁？" - 确认用户的身份
- 例：用户名/密码登录、JWT token 验证、API Key

**授权 (Authorization)**: 验证用户权限
- "你能做什么？" - 确认用户是否有权限执行某操作
- 例：admin 权限检查、资源所有权验证

### 9.1.2 JWT Token 原理

**JWT (JSON Web Token)** 结构：
```
Header.Payload.Signature
```

1. **Header**: 算法和 token 类型
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

2. **Payload**: 用户数据（claims）
```json
{
  "sub": "user_id",      // subject (用户ID)
  "exp": 1234567890,     // expiration time
  "iat": 1234567890,     // issued at
  "role": "admin"        // custom claims
}
```

3. **Signature**: 验证 token 完整性
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**优势**:
- **无状态**: 服务器不需要存储 session
- **可扩展**: 可添加自定义 claims
- **跨域**: 适合分布式系统

### 9.1.3 OAuth2 流程

**OAuth2 Password Flow** (用户名/密码模式):

```
1. Client → Server: POST /token (username, password)
2. Server: 验证凭据
3. Server → Client: 返回 access_token
4. Client → Server: 请求 API (Authorization: Bearer <token>)
5. Server: 验证 token 并返回数据
```

### 9.1.4 Bearer Token

HTTP Authorization header 格式:
```
Authorization: Bearer <token>
```

- **Bearer**: 表示持有者认证方案
- 任何持有该 token 的人都可以访问资源

---

## 9.2 FastAPI 特性

### ??? 9.2.1 OAuth2PasswordBearer

FastAPI 提供的依赖，自动处理 Bearer token 提取：

```python
from fastapi.security import OAuth2PasswordBearer

# 定义 token URL（用于 OpenAPI 文档）
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/protected")
async def protected_route(token: str = Depends(oauth2_scheme)):
    
```

### 9.2.2 JWT Token 原理

**JWT (JSON Web Token)** 是一种无状态的认证方式，结构为：

```
Header.Payload.Signature
```

**① Header**（头部）：
```json
{
  "alg": "HS256",    // 签名算法
  "typ": "JWT"       // token 类型
}
```

**② Payload**（载荷）：
```json
{
  "sub": "user_123",           // subject: 用户ID
  "exp": 1735822800,           // expiration: 过期时间
  "iat": 1735736400,           // issued at: 签发时间
  "role": "admin",             // 自定义字段
  "email": "user@example.com"  // 自定义字段
}
```

**③ Signature**（签名）：
```python
signature = HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret_key
)
```

**JWT 优势**：
- **无状态**：服务器不需要存储 session，适合分布式系统
- **自包含**：token 包含所有必要信息
- **可扩展**：可以添加自定义 claims

**JWT 注意事项**：
- Payload 是 Base64 编码，**不是加密**，不要存储敏感信息
- 必须使用 HTTPS 传输
- 设置合理的过期时间

### 9.2.3 OAuth2 流程

**OAuth2 Password Flow**（密码模式，AutoGPT 使用的模式）：

```
┌────────┐                                  ┌──────────┐
│ Client │                                  │  Server  │
└───┬────┘                                  └────┬─────┘
    │                                            │
    │  1. POST /token                            │
    │     (username, password)                   │
    ├───────────────────────────────────────────>│
    │                                            │
    │                      2. 验证凭据            │
    │                         ├─ 检查用户名/密码   │
    │                         └─ 生成 JWT token  │
    │                                            │
    │  3. 返回 access_token                      │
    │<───────────────────────────────────────────┤
    │    {                                       │
    │      "access_token": "eyJ...",             │
    │      "token_type": "bearer"                │
    │    }                                       │
    │                                            │
    │  4. GET /api/protected                     │
    │     Authorization: Bearer eyJ...           │
    ├───────────────────────────────────────────>│
    │                                            │
    │                      5. 验证 token          │
    │                         ├─ 解析 JWT        │
    │                         ├─ 验证签名         │
    │                         └─ 检查过期时间     │
    │                                            │
    │  6. 返回受保护的数据                        │
    │<───────────────────────────────────────────┤
    │                                            │
```

### 9.2.4 Bearer Token

**Bearer Token** 是一种 HTTP 认证方案：

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

- **Bearer**：表示"持有者"认证
- 含义：任何持有（bearer）此 token 的人都可以访问资源
- 因此必须保护 token 不被泄露

---

## 9.3 FastAPI 特性

### 9.3.1 OAuth2PasswordBearer

FastAPI 提供 `OAuth2PasswordBearer` 依赖来自动提取和验证 Bearer token：

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

# 定义 OAuth2 scheme，tokenUrl 用于 OpenAPI 文档
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/items")
async def read_items(token: str = Depends(oauth2_scheme)):
    # oauth2_scheme 自动从 Authorization header 提取 token
    # 如果没有 token，自动返回 401
    return {"token": token}
```

**工作原理**：
1. 检查请求的 `Authorization` header
2. 验证格式是否为 `Bearer <token>`
3. 提取 token 字符串
4. 如果缺失或格式错误，返回 `401 Unauthorized`

### 9.3.2 OAuth2 密码模式完整示例

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from datetime import datetime, timedelta
import jwt

app = FastAPI()

# 配置
SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# 模拟用户数据库
fake_users_db = {
    "alice": {
        "username": "alice",
        "hashed_password": "fakehashedsecret",
        "role": "admin"
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class User(BaseModel):
    username: str
    role: str

def create_access_token(data: dict):
    """生成 JWT token"""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    """验证 token 并返回当前用户"""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(status_code=401, detail="Invalid token")
    except jwt.PyJWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user = fake_users_db.get(username)
    if user is None:
        raise HTTPException(status_code=401, detail="User not found")
    
    return User(username=username, role=user["role"])

@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """登录端点，返回 JWT token"""
    user = fake_users_db.get(form_data.username)
    if not user or user["hashed_password"] != f"fakehashed{form_data.password}":
        raise HTTPException(
            status_code=401,
            detail="Incorrect username or password"
        )
    
    access_token = create_access_token(data={"sub": form_data.username})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    """受保护的端点，需要认证"""
    return current_user
```

**关键点**：
- `OAuth2PasswordRequestForm`：FastAPI 提供的表单，包含 `username` 和 `password`
- `create_access_token`：生成 JWT token
- `get_current_user`：依赖函数，验证 token 并返回用户对象
- 依赖链：`oauth2_scheme` → `get_current_user` → 业务逻辑

### 9.3.3 API Key 认证

API Key 是另一种常见的认证方式：

```python
from fastapi import Security, HTTPException
from fastapi.security import APIKeyHeader

# 定义 API Key header
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

async def verify_api_key(api_key: str = Security(api_key_header)):
    """验证 API Key"""
    if api_key != "secret-api-key":
        raise HTTPException(
            status_code=403,
            detail="Invalid API Key"
        )
    return api_key

@app.get("/protected")
async def protected_route(api_key: str = Depends(verify_api_key)):
    return {"message": "Access granted"}
```

**API Key vs JWT**：
- **API Key**: 简单，适合机器到机器通信，长期有效
- **JWT**: 复杂，适合用户认证，短期有效，包含用户信息

### 9.3.4 依赖注入式认证

FastAPI 的依赖注入使认证非常灵活：

```python
from typing import Annotated

# 方式1：单个依赖
@app.get("/admin")
async def admin_only(user: User = Depends(get_admin_user)):
    return {"message": "Admin access"}

# 方式2：使用 Annotated (推荐)
UserDep = Annotated[User, Depends(get_current_user)]
AdminDep = Annotated[User, Depends(get_admin_user)]

@app.get("/items")
async def read_items(user: UserDep):
    return {"user": user}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int, admin: AdminDep):
    return {"deleted": item_id}

# 方式3：组合多个认证方式
async def get_current_user_or_api_key(
    user: User = Depends(get_current_user, use_cache=False),
    api_key: str = Depends(verify_api_key, use_cache=False)
):
    """支持 JWT 或 API Key 认证"""
    return user if user else api_key
```

### 9.3.5 安全依赖（Security vs Depends）

FastAPI 提供 `Security()` 作为 `Depends()` 的特殊版本：

```python
from fastapi import Security

# Security() 用于认证/授权
async def get_current_user(token: str = Security(oauth2_scheme)):
    # OpenAPI 文档会显示这是一个安全端点
    pass

# Depends() 用于普通依赖
async def get_db(session = Depends(create_session)):
    pass
```

**区别**：
- `Security()` 会在 OpenAPI 文档中显示为安全要求
- 功能上与 `Depends()` 相同
- 语义上更清晰，表明这是安全相关的依赖

---

## 9.4 AutoGPT 实战中认证和授权分析

### 9.4.1 `autogpt_libs.auth` 认证系统架构

AutoGPT 的认证系统采用模块化设计，主要包含以下组件：

```
autogpt_libs/auth/
├── config.py          # 配置管理和验证
├── models.py          # 用户数据模型
├── jwt_utils.py       # JWT token 核心功能
├── dependencies.py    # FastAPI 依赖函数
└── helpers.py         # OpenAPI 文档增强
```

**设计特点**：
- **配置验证**：启动时验证 JWT 配置的安全性
- **安全警告**：对弱密钥和不安全算法发出警告
- **依赖注入**：利用 FastAPI 的依赖系统实现认证
- **分层设计**：底层工具 → 中间验证 → 高层依赖

---

### 9.4.2 JWT Token 验证流程

**配置管理 ([config.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/config.py:0:0-0:0))**

```python
class Settings:
    def __init__(self):
        # 从环境变量读取配置
        self.JWT_VERIFY_KEY: str = os.getenv(
            "JWT_VERIFY_KEY", 
            os.getenv("SUPABASE_JWT_SECRET", "")
        ).strip()
        self.JWT_ALGORITHM: str = os.getenv("JWT_SIGN_ALGORITHM", "HS256").strip()
        
        self.validate()  # 启动时验证配置
    
    def validate(self):
        # 1. 确保密钥存在
        if not self.JWT_VERIFY_KEY:
            raise AuthConfigError("JWT_VERIFY_KEY must be set")
        
        # 2. 警告弱密钥
        if len(self.JWT_VERIFY_KEY) < 32:
            logger.warning("⚠️ JWT_VERIFY_KEY appears weak")
        
        # 3. 警告使用对称算法
        if self.JWT_ALGORITHM.startswith("HS"):
            logger.warning("⚠️ Using symmetric algorithm (HS256)")
```

**关键点**：
- 支持 Supabase JWT（AutoGPT 使用 Supabase 作为认证服务）
- **安全建议**：推荐使用非对称算法（ES256）而非对称算法（HS256）
- 原因：对称密钥泄露后任何人都可以伪造 token

**JWT 解析和验证 ([jwt_utils.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/jwt_utils.py:0:0-0:0))**

**核心函数 1：[parse_jwt_token](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/jwt_utils.py:44:0-64:52)**

```python
def parse_jwt_token(token: str) -> dict[str, Any]:
    """解析和验证 JWT token"""
    settings = get_settings()
    try:
        payload = jwt.decode(
            token,
            settings.JWT_VERIFY_KEY,        # 验证密钥
            algorithms=[settings.JWT_ALGORITHM],  # 允许的算法
            audience="authenticated",        # 验证 audience claim
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise ValueError("Token has expired")
    except jwt.InvalidTokenError as e:
        raise ValueError(f"Invalid token: {str(e)}")
```

**验证步骤**：
1. **签名验证**：确保 token 未被篡改
2. **算法验证**：防止算法降级攻击
3. **过期时间验证**：检查 `exp` claim
4. **Audience 验证**：确保 token 用途正确

**核心函数 2：[get_jwt_payload](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/jwt_utils.py:18:0-41:59)（FastAPI 依赖）**

```python
# HTTPBearer: 自动提取 Authorization header 中的 Bearer token
bearer_jwt_auth = HTTPBearer(
    bearerFormat="jwt",
    scheme_name="HTTPBearerJWT",
    auto_error=False  # 不自动返回错误，手动处理
)

async def get_jwt_payload(
    credentials: HTTPAuthorizationCredentials | None = Security(bearer_jwt_auth),
) -> dict[str, Any]:
    """FastAPI 依赖：从 HTTP header 提取并验证 JWT"""
    if not credentials:
        raise HTTPException(
            status_code=401, 
            detail="Authorization header is missing"
        )
    
    try:
        payload = parse_jwt_token(credentials.credentials)
        return payload
    except ValueError as e:
        raise HTTPException(status_code=401, detail=str(e))
```

**工作流程**：
```
Request Headers → bearer_jwt_auth → get_jwt_payload → payload dict
  ↓
Authorization: Bearer eyJhbG...
```

**用户模型 ([models.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/models.py:0:0-0:0))**

```python
@dataclass(frozen=True)
class User:
    user_id: str
    email: str
    phone_number: str
    role: str  # 'user' or 'admin'
    
    @classmethod
    def from_payload(cls, payload):
        """从 JWT payload 构建 User 对象"""
        return cls(
            user_id=payload["sub"],      # subject = user ID
            email=payload.get("email", ""),
            phone_number=payload.get("phone", ""),
            role=payload["role"],
        )
```

**为什么使用 dataclass**：
- 避免引入 Pydantic 依赖（`autogpt_libs` 是共享库）
- `frozen=True` 使对象不可变，更安全

---

### 9.4.3 FastAPI 依赖函数 ([dependencies.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/dependencies.py:0:0-0:0))

AutoGPT 提供三个主要依赖函数，形成认证和授权的完整体系：

**① [requires_user](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/dependencies.py:12:0-19:53) - 基础认证**

```python
async def requires_user(
    jwt_payload: dict = fastapi.Security(get_jwt_payload)
) -> User:
    """要求用户已认证（任何角色）"""
    return verify_user(jwt_payload, admin_only=False)
```

**使用场景**：
```python
from autogpt_libs.auth import requires_user

@app.get("/my-profile")
async def get_profile(user: User = Depends(requires_user)):
    """任何已登录用户都可以访问"""
    return {"user_id": user.user_id, "email": user.email}
```

**② [requires_admin_user](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/dependencies.py:22:0-31:52) - 管理员认证**

```python
async def requires_admin_user(
    jwt_payload: dict = fastapi.Security(get_jwt_payload),
) -> User:
    """要求用户是管理员"""
    return verify_user(jwt_payload, admin_only=True)
```

**使用场景**：
```python
@app.delete("/users/{user_id}")
async def delete_user(
    user_id: str,
    admin: User = Depends(requires_admin_user)
):
    """只有管理员可以删除用户"""
    await db.delete_user(user_id)
    return {"deleted": user_id}
```

**③ [get_user_id](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/dependencies.py:34:0-46:18) - 只需 ID**

```python
async def get_user_id(
    jwt_payload: dict = fastapi.Security(get_jwt_payload)
) -> str:
    """只返回用户 ID，不创建完整 User 对象"""
    user_id = jwt_payload.get("sub")
    if not user_id:
        raise fastapi.HTTPException(
            status_code=401, 
            detail="User ID not found in token"
        )
    return user_id
```

**使用场景**（性能优化）：
```python
@app.get("/graphs")
async def list_graphs(user_id: str = Depends(get_user_id)):
    """只需要用户 ID，不需要完整用户信息"""
    return await db.list_user_graphs(user_id)
```

**核心验证函数 [verify_user](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/jwt_utils.py:67:0-79:41)**

```python
def verify_user(jwt_payload: dict | None, admin_only: bool) -> User:
    """验证用户并检查权限"""
    if jwt_payload is None:
        raise HTTPException(status_code=401, detail="Missing token")
    
    user_id = jwt_payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="User ID not found")
    
    # 授权检查
    if admin_only and jwt_payload["role"] != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
    
    return User.from_payload(jwt_payload)
```

**HTTP 状态码的正确使用**：
- **401 Unauthorized**：认证失败（token 无效/缺失）
- **403 Forbidden**：认证成功但无权限（非管理员）

---

### 9.4.4 WebSocket 认证

WebSocket 不能使用标准的 HTTP headers（握手后 headers 不可用），因此 AutoGPT 采用 **query parameter** 传递 token：

```python
async def authenticate_websocket(websocket: WebSocket) -> str:
    """WebSocket 专用认证"""
    # 1. 检查是否启用认证
    if not settings.config.enable_auth:
        return DEFAULT_USER_ID
    
    # 2. 从 query parameter 提取 token
    token = websocket.query_params.get("token")
    if not token:
        await websocket.close(code=4001, reason="Missing authentication token")
        return ""
    
    # 3. 验证 token
    try:
        payload = parse_jwt_token(token)
        user_id = payload.get("sub")
        if not user_id:
            await websocket.close(code=4002, reason="Invalid token")
            return ""
        return user_id
    except ValueError:
        await websocket.close(code=4003, reason="Invalid token")
        return ""
```

**使用示例**（客户端）：
```javascript
// 连接 WebSocket 时在 URL 中传递 token
const token = localStorage.getItem('access_token');
const ws = new WebSocket(`wss://api.example.com/ws?token=${token}`);
```

**WebSocket 端点使用**：
```python
@app.websocket("/ws")
async def websocket_router(
    websocket: WebSocket,
    manager: ConnectionManager = Depends(get_connection_manager)
):
    # 认证
    user_id = await authenticate_websocket(websocket)
    if not user_id:
        return  # 已关闭连接
    
    # 建立连接
    await manager.connect_socket(websocket)
    
    try:
        while True:
            data = await websocket.receive_text()
            # 处理消息...
    except WebSocketDisconnect:
        manager.disconnect_socket(websocket)
```

**自定义关闭代码**：
- **4001**：缺少 token
- **4002**：token 无效（无 user_id）
- **4003**：token 验证失败
- 使用 4xxx 范围表示应用层错误

---

### 9.4.5 API Key 验证系统

AutoGPT 实现了完整的 API Key 管理系统，用于机器到机器通信（例如 SDK、webhook、外部集成）。

**API Key 存储模型 ([api_key.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/api_key.py:0:0-0:0))**

```python
class APIKeyInfo(BaseModel):
    """API Key 信息（不含哈希）"""
    id: str
    name: str
    head: str  # 前几个字符，用于显示
    tail: str  # 后几个字符，用于显示
    status: APIKeyStatus  # ACTIVE, REVOKED
    permissions: list[APIKeyPermission]  # READ, WRITE, EXECUTE
    created_at: datetime
    last_used_at: Optional[datetime]
    revoked_at: Optional[datetime]
    description: Optional[str]
    user_id: str

class APIKeyInfoWithHash(APIKeyInfo):
    """包含哈希的 API Key（内部使用）"""
    hash: str
    salt: str | None  # 新版使用 salt，旧版为 None
    
    def match(self, plaintext_key: str) -> bool:
        """验证明文 key 是否匹配"""
        return keysmith.verify_key(plaintext_key, self.hash, self.salt)
```

**安全设计**：
- **只存储哈希**：数据库不存储明文 key
- **加盐哈希**：防止彩虹表攻击
- **Head/Tail**：方便用户识别 key，不泄露完整信息
- **权限系统**：细粒度控制 API Key 可以执行的操作

**创建 API Key**

```python
from autogpt_libs.api_key.keysmith import APIKeySmith

keysmith = APIKeySmith()

async def create_api_key(
    name: str,
    user_id: str,
    permissions: list[APIKeyPermission],
    description: Optional[str] = None,
) -> tuple[APIKeyInfo, str]:
    """
    生成新的 API key
    返回：(key 信息对象, 明文 key)
    """
    # 1. 生成密钥学安全的随机 key
    generated_key = keysmith.generate_key()
    # generated_key.key: 明文 key（只返回一次）
    # generated_key.hash: 哈希值（存储到数据库）
    # generated_key.salt: 盐值
    # generated_key.head/tail: 显示用
    
    # 2. 存储到数据库
    saved_key_obj = await PrismaAPIKey.prisma().create(
        data={
            "id": str(uuid.uuid4()),
            "name": name,
            "head": generated_key.head,
            "tail": generated_key.tail,
            "hash": generated_key.hash,
            "salt": generated_key.salt,
            "permissions": [p for p in permissions],
            "description": description,
            "userId": user_id,
        }
    )
    
    # 3. 返回（明文 key 只在这里返回一次）
    return APIKeyInfo.from_db(saved_key_obj), generated_key.key
```

**重要**：明文 key 只在创建时返回一次，之后无法恢复。

**API Key 认证器 ([api_key_auth.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/utils/api_key_auth.py:0:0-0:0))**

AutoGPT 提供了灵活的 [APIKeyAuthenticator](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/utils/api_key_auth.py:18:0-119:24) 类：

```python
class APIKeyAuthenticator(APIKeyHeader):
    """
    可配置的 API Key 认证器
    
    支持两种模式：
    1. 简单 token 匹配
    2. 自定义验证函数（如数据库查询）
    """
    
    def __init__(
        self,
        header_name: str,                    # header 名称
        expected_token: Optional[str] = None,  # 简单模式：期望的 token
        validator: Optional[Callable] = None,  # 自定义验证函数
        status_if_missing: int = 401,
        message_if_invalid: str = "Invalid API key",
    ):
        super().__init__(
            name=header_name,
            scheme_name=f"APIKeyAuthenticator-{header_name}",
            auto_error=False,  # 手动处理错误
        )
        self.expected_token = expected_token
        self.custom_validator = validator
        self.status_if_missing = status_if_missing
        self.message_if_invalid = message_if_invalid
    
    async def __call__(self, request: Request) -> Any:
        # 1. 提取 API Key
        api_key = await super().__call__(request)
        if api_key is None:
            raise HTTPException(
                status_code=self.status_if_missing,
                detail="No API key in request"
            )
        
        # 2. 验证（使用自定义验证器或默认验证器）
        validator = self.custom_validator or self.default_validator
        result = (
            await validator(api_key)
            if inspect.iscoroutinefunction(validator)
            else validator(api_key)
        )
        
        if not result:
            raise HTTPException(
                status_code=self.status_if_missing,
                detail=self.message_if_invalid
            )
        
        # 3. 保存验证结果到 request.state
        if result is not True:
            request.state.api_key = result
        
        return result
    
    async def default_validator(self, api_key: str) -> bool:
        """默认验证器：使用 secrets.compare_digest 防止计时攻击"""
        if not self.expected_token:
            raise MissingConfigError("expected_token not set")
        try:
            return secrets.compare_digest(api_key, self.expected_token)
        except TypeError:
            return False
```

**使用示例 1：简单 token 验证**

```python
from fastapi import FastAPI, Security
from backend.server.utils.api_key_auth import APIKeyAuthenticator

app = FastAPI()

# 简单模式：直接比较 token
api_key_auth = APIKeyAuthenticator(
    header_name="X-API-Key",
    expected_token="secret-token-12345"
)

@app.get("/protected", dependencies=[Security(api_key_auth)])
async def protected_endpoint():
    return {"message": "Access granted"}
```

**使用示例 2：数据库验证**

```python
async def validate_api_key_with_db(api_key: str) -> APIKeyInfoWithHash | None:
    """从数据库验证 API Key"""
    # 1. 通过 head 查找可能的 keys
    head = api_key[:APIKeySmith.HEAD_LENGTH]
    possible_keys = await PrismaAPIKey.prisma().find_many(
        where={"head": head, "status": APIKeyStatus.ACTIVE}
    )
    
    # 2. 验证哈希
    for db_key in possible_keys:
        key_obj = APIKeyInfoWithHash.from_db(db_key)
        if key_obj.match(api_key):
            # 3. 更新最后使用时间
            await PrismaAPIKey.prisma().update(
                where={"id": key_obj.id},
                data={"lastUsedAt": datetime.now(timezone.utc)}
            )
            return key_obj
    
    return None

# 使用自定义验证器
api_key_auth = APIKeyAuthenticator(
    header_name="X-API-Key",
    validator=validate_api_key_with_db
)

@app.get("/data")
async def get_data(
    request: Request,
    api_key: APIKeyInfoWithHash = Security(api_key_auth)
):
    """
    api_key 是验证器返回的对象
    也可以从 request.state.api_key 获取
    """
    # 检查权限
    if APIKeyPermission.READ not in api_key.permissions:
        raise HTTPException(status_code=403, detail="READ permission required")
    
    return {"user_id": api_key.user_id, "data": "..."}
```

**安全特性**：
- **`secrets.compare_digest`**：防止计时攻击（timing attack）
- **先查 head 再验证哈希**：性能优化
- **记录使用时间**：审计和监控

---

### 9.4.6 权限检查和授权模式

**路由级别的依赖注入**

AutoGPT 在路由定义中大量使用依赖注入进行认证和授权：

**模式 1：路由装饰器中的 dependencies**

```python
# 整个路由器都需要认证
v1_router.include_router(
    backend.server.routers.analytics.router,
    prefix="/analytics",
    tags=["analytics"],
    dependencies=[Security(requires_user)],  # 所有端点都需要认证
)
```

**模式 2：单个端点的 dependencies**

```python
@v1_router.get(
    "/blocks",
    summary="List available blocks",
    tags=["blocks"],
    dependencies=[Security(requires_user)],  # 这个端点需要认证
)
async def list_blocks():
    """任何认证用户都可以列出 blocks"""
    return await get_blocks()
```

**模式 3：参数级别的依赖（需要用户信息）**

```python
@v1_router.post(
    "/graphs",
    summary="Create new graph",
    dependencies=[Security(requires_user)],  # 确保认证
)
async def create_new_graph(
    create_graph: CreateGraph,
    user_id: Annotated[str, Security(get_user_id)],  # 获取用户 ID
):
    """创建 graph 需要知道是哪个用户"""
    graph = await Graph.from_db(create_graph.graph_id, user_id=user_id)
    return graph
```

**模式 4：多层依赖组合**

```python
@v1_router.delete(
    "/graphs/{graph_id}",
    dependencies=[Security(requires_user)],
)
async def delete_graph(
    graph_id: str,
    user_id: Annotated[str, Security(get_user_id)],
):
    """删除 graph - 需要验证所有权"""
    # 1. 认证通过（dependencies）
    # 2. 获取用户 ID
    # 3. 业务层验证所有权
    graph = await Graph.from_db(graph_id, user_id=user_id)
    if not graph:
        raise HTTPException(status_code=404, detail="Graph not found")
    
    # 4. 执行删除
    await graph.delete()
    return {"deleted": graph_id}
```

**资源所有权验证模式**

AutoGPT 使用 **在数据库查询中验证所有权** 的模式：

```python
# backend/data/graph.py

class Graph(BaseModel):
    @staticmethod
    async def from_db(
        graph_id: str,
        version: int | None = None,
        template: bool = False,
        user_id: str | None = None,  # 👈 传入 user_id
    ):
        """
        从数据库加载 Graph
        如果提供 user_id，只返回该用户拥有的 graph
        """
        # 构建查询条件
        where_clause: dict[str, Any] = {"id": graph_id}
        
        # 添加所有权过滤
        if user_id:
            where_clause["userId"] = user_id
        
        if template:
            where_clause["isTemplate"] = True
        
        # 查询
        graph = await AgentGraph.prisma().find_first(
            where=where_clause,
            ...
        )
        
        if not graph:
            # 要么不存在，要么用户无权访问
            raise NotFoundError(f"Graph #{graph_id} not found")
        
        return Graph(...)
```

**优势**：
- **简洁**：业务逻辑和授权检查合并
- **性能**：一次数据库查询完成验证和数据获取
- **安全**：无法绕过所有权检查

**权限枚举和检查**

```python
from prisma.enums import APIKeyPermission

class APIKeyPermission(str, Enum):
    READ = "READ"
    WRITE = "WRITE"
    EXECUTE = "EXECUTE"

# 在端点中检查权限
async def execute_graph_endpoint(
    api_key: APIKeyInfoWithHash = Security(api_key_auth)
):
    # 检查 API Key 是否有执行权限
    if APIKeyPermission.EXECUTE not in api_key.permissions:
        raise HTTPException(
            status_code=403,
            detail="EXECUTE permission required"
        )
    
    # 执行操作...
```

**全局异常处理**

```python
# backend/util/exceptions.py

class NotAuthorizedError(Exception):
    """用户无权访问资源"""
    pass

# 在应用中注册异常处理器
@app.exception_handler(NotAuthorizedError)
async def not_authorized_handler(request: Request, exc: NotAuthorizedError):
    return JSONResponse(
        status_code=403,
        content={"detail": str(exc)}
    )
```

---

### 9.4.7 安全最佳实践

1. **密钥管理**
   - 使用环境变量存储密钥
   - 密钥长度 ≥ 32 字符
   - 优先使用非对称算法（ES256 > HS256）

2. **Token 安全**
   - 设置合理的过期时间
   - 不在 payload 存储敏感信息
   - 使用 HTTPS 传输

3. **API Key 安全**
   - 只存储哈希值
   - 使用 `secrets.compare_digest` 防止计时攻击
   - 明文 key 只显示一次

4. **依赖注入**
   - 使用 `Security()` 标记认证依赖
   - 组合多个依赖实现细粒度控制
   - 在 `dependencies` 参数中声明路由级保护

5. **错误处理**
   - 401: 认证失败
   - 403: 无权限
   - 不泄露敏感信息

---

# 10 APIRouter 模块化设计

---

## 10.1 基础复习

### 10.1.1 模块化设计原则

**模块化 (Modularity)** 是将大型系统拆分为独立、可管理的小模块的设计方法。

**核心原则:**

**① 单一职责原则 (Single Responsibility Principle)**
- 每个模块只负责一个功能领域
- 例：`users.py` 只处理用户相关的端点

**② 高内聚，低耦合 (High Cohesion, Low Coupling)**
- **高内聚**：模块内部功能紧密相关
- **低耦合**：模块之间依赖最小化

**③ 关注点分离 (Separation of Concerns)**
- 业务逻辑 vs 数据访问 vs API 路由
- 不同层次的代码放在不同位置

**④ DRY 原则 (Don't Repeat Yourself)**
- 共享的功能提取到公共模块
- 通过依赖注入复用代码

#### 模块化的优势

```
 可维护性：易于定位和修改代码
 可扩展性：添加新功能不影响现有代码
 可测试性：独立模块易于单元测试
 团队协作：多人可并行开发不同模块
 代码复用：公共功能可跨模块使用
```

### 10.1.2 代码组织策略

**按功能组织 vs 按层级组织**

**按功能组织（Feature-based）** - AutoGPT 使用的方式：
```
backend/
├── server/
│   ├── routers/
│   │   ├── v1/
│   │   │   ├── users.py        # 用户功能
│   │   │   ├── graphs.py       # 图功能
│   │   │   ├── executions.py   # 执行功能
│   │   │   └── credits.py      # 积分功能
```

**按层级组织（Layer-based）**：
```
backend/
├── controllers/     # 所有控制器
├── services/        # 所有业务逻辑
├── repositories/    # 所有数据访问
└── models/          # 所有数据模型
```

**AutoGPT 混合方式**：
```
backend/
├── server/          # API 层（按功能）
│   └── routers/
├── data/            # 数据层（按实体）
│   ├── user.py
│   ├── graph.py
│   └── execution.py
└── executor/        # 业务逻辑层（按功能）
```

### 10.1.3  Python 命名空间和包

**模块 (Module)**

- 单个 `.py` 文件
- 可以被 `import`

```python
# users.py 是一个模块
def get_user(user_id: str):
    pass
```

**包 (Package)**

- 包含 [__init__.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/__init__.py:0:0-0:0) 的目录
- 可以包含多个模块和子包

```
routers/              # 包
├── __init__.py       # 标记为包
├── users.py          # 模块
└── graphs.py         # 模块
```

**命名空间 (Namespace)**

- 避免命名冲突的机制

```python
# 不同包中可以有同名模块
from backend.server.routers.v1 import users
from backend.server.routers.v2 import users as users_v2
```

**[init.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/autogpt_libs/autogpt_libs/auth/__init__.py:0:0-0:0) 的作用**

```python
# routers/__init__.py

# 1. 标记目录为包
# 2. 控制 from package import * 的行为
__all__ = ["users", "graphs"]

# 3. 提供包级别的接口
from .users import router as users_router
from .graphs import router as graphs_router

# 4. 包初始化代码
print("Routers package initialized")
```

### 10.1.4  包和模块的导入策略

**绝对导入（推荐）**：
```python
from backend.server.routers.v1 import users
from backend.data.graph import Graph
```

**相对导入**：
```python
# 在 backend/server/routers/v1/users.py 中
from . import graphs              # 同级
from ..utils import helper        # 上级
from ...data.user import User     # 上上级
```

---

## 10.2 FastAPI 特性

### 10.2.1 APIRouter 基础

`APIRouter` 是 FastAPI 的核心工具，用于创建模块化的路由。

**基本使用**

```python
from fastapi import APIRouter

# 创建路由器
router = APIRouter()

# 添加路由
@router.get("/users")
async def list_users():
    return {"users": []}

@router.post("/users")
async def create_user(name: str):
    return {"user": {"name": name}}

@router.get("/users/{user_id}")
async def get_user(user_id: str):
    return {"user_id": user_id}
```

**与主应用集成**

```python
from fastapi import FastAPI

app = FastAPI()

# 包含路由器
app.include_router(router)

# 此时端点为：
# GET  /users
# POST /users
# GET  /users/{user_id}
```

### 10.2.2 路由前缀和标签

**路由前缀 (Prefix)**

**在路由器定义时设置**：

```python
# users_router.py
router = APIRouter(prefix="/users")

@router.get("")           # 实际路径: /users
async def list_users():
    pass

@router.get("/{user_id}") # 实际路径: /users/{user_id}
async def get_user(user_id: str):
    pass
```

**在包含路由器时设置**：
```python
# main.py
from fastapi import FastAPI
from .routers import users

app = FastAPI()

# 包含时添加前缀
app.include_router(
    users.router,
    prefix="/api/v1/users"
)

# 最终路径: /api/v1/users
#           /api/v1/users/{user_id}
```

**标签 (Tags)**

标签用于 OpenAPI 文档分组：

```python
# 在路由器定义时设置
router = APIRouter(
    prefix="/users",
    tags=["users"]  # 所有端点都标记为 "users"
)

# 或在包含时设置
app.include_router(
    users.router,
    prefix="/api/v1/users",
    tags=["User Management"]
)

# 或在单个端点设置
@router.get("/", tags=["special"])
async def special_endpoint():
    pass
```

**OpenAPI 文档效果**：
- 端点按标签分组
- 便于前端开发者理解 API 结构

### 10.2.3 路由器组合

**多个路由器组合**

```python
# routers/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/users")
async def list_users():
    return {"users": []}

# routers/posts.py
router = APIRouter()

@router.get("/posts")
async def list_posts():
    return {"posts": []}

# main.py
from fastapi import FastAPI
from .routers import users, posts

app = FastAPI()

# 组合多个路由器
app.include_router(users.router, prefix="/api/v1", tags=["users"])
app.include_router(posts.router, prefix="/api/v1", tags=["posts"])

# 结果：
# GET /api/v1/users
# GET /api/v1/posts
```

**嵌套路由器**

```python
# routers/api_v1.py
from fastapi import APIRouter
from .users import router as users_router
from .posts import router as posts_router

# 创建 v1 路由器
api_v1_router = APIRouter(prefix="/api/v1")

# 包含子路由器
api_v1_router.include_router(users_router, prefix="/users", tags=["users"])
api_v1_router.include_router(posts_router, prefix="/posts", tags=["posts"])

# main.py
app = FastAPI()
app.include_router(api_v1_router)

# 结果：
# GET /api/v1/users
# GET /api/v1/posts
```

### 10.2.4 路由器挂载 (Mounting)

**`include_router` vs `mount`**

**`include_router`**（推荐）：
- 路由器成为主应用的一部分
- 共享中间件、异常处理器
- 出现在 OpenAPI 文档中

```python
app.include_router(users_router, prefix="/users")
```

**`mount`**（用于独立应用）：
- 挂载完全独立的 ASGI 应用
- 不共享主应用的配置
- 不出现在主应用的 OpenAPI 文档中

```python
from fastapi import FastAPI

main_app = FastAPI()
sub_app = FastAPI()

@sub_app.get("/hello")
async def hello():
    return {"message": "Hello from sub app"}

# 挂载子应用
main_app.mount("/sub", sub_app)

# 访问: /sub/hello
```

### 10.2.5 依赖继承

路由器可以定义共享的依赖，所有端点自动继承：

```python
from fastapi import APIRouter, Depends, Security
from .auth import requires_user

# 路由器级别的依赖
router = APIRouter(
    prefix="/admin",
    tags=["admin"],
    dependencies=[Security(requires_user)]  # 所有端点都需要认证
)

@router.get("/dashboard")
async def dashboard():
    """自动继承 requires_user 依赖"""
    return {"dashboard": "data"}

@router.get("/settings")
async def settings():
    """也继承 requires_user 依赖"""
    return {"settings": "data"}

# 包含路由器时也可以添加依赖
app.include_router(
    router,
    dependencies=[Depends(rate_limiter)]  # 添加额外依赖
)
```

**依赖叠加顺序**：
```
端点依赖 → 路由器依赖 → include_router 依赖 → 应用依赖
```

---

## 10.3 AutoGPT中的路由器架构实战分析

### 10.3.1 AutoGPT 路由器目录结构

AutoGPT 采用**混合组织策略**：按版本和功能模块化。

```
backend/server/
│
├── rest_api.py              # 主应用入口
├── ws_api.py                # WebSocket 应用入口
│
├── routers/                 # V1 路由器
│   ├── v1.py                # 主 v1 路由器（单文件）
│   ├── analytics.py         # 分析功能子路由器
│   └── postmark/            # 邮件功能（子包）
│
├── v2/                      # V2 路由器（按功能拆分）
│   ├── store/               # 商店功能
│   │   ├── routes.py        # 路由定义
│   │   ├── db.py            # 数据库操作
│   │   ├── model.py         # 数据模型
│   │   └── cache.py         # 缓存
│   ├── builder/             # 构建器功能
│   ├── library/             # 库功能
│   ├── otto/                # Otto AI 功能
│   ├── turnstile/           # Cloudflare Turnstile
│   └── admin/               # 管理员功能
│
├── external/                # 外部 API（独立应用）
│   ├── api.py               # 外部应用入口
│   ├── middleware.py        # 外部 API 专用中间件
│   └── routes/
│       └── v1.py            # 外部 API v1 路由
│
├── integrations/            # 第三方集成路由器
│   └── router.py
│
├── middleware/              # 中间件
└── utils/                   # 工具函数
```

**设计特点**：

1. **V1 单文件 vs V2 多模块**
   - V1: [v1.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/routers/v1.py:0:0-0:0) 单文件包含所有端点（简单但可能臃肿）
   - V2: 按功能域拆分为独立模块（可扩展，易维护）

2. **功能内聚**
   - 每个 v2 模块都是完整的功能单元
   - 包含 routes + db + model + business logic

3. **外部 API 隔离**
   - 独立的 FastAPI 应用（`external_app`）
   - 不共享主应用的认证/中间件
   - 通过 `mount` 挂载

### 10.3.2 按功能拆分路由

**V1 单文件路由器 ([routers/v1.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/routers/v1.py:0:0-0:0))**

**特点**：所有端点在一个文件中，按注释分组

```python
# backend/server/routers/v1.py

from fastapi import APIRouter

v1_router = APIRouter()

# 包含子路由器
v1_router.include_router(
    backend.server.integrations.router.router,
    prefix="/integrations",
    tags=["integrations"],
)

v1_router.include_router(
    backend.server.routers.analytics.router,
    prefix="/analytics",
    tags=["analytics"],
    dependencies=[Security(requires_user)],  # 整个子路由器需要认证
)

########################################################
##################### Auth #############################
########################################################

@v1_router.post("/auth/user", tags=["auth"], ...)
async def get_or_create_user_route(...):
    pass

@v1_router.post("/auth/user/email", tags=["auth"], ...)
async def update_user_email_route(...):
    pass

########################################################
##################### Blocks ###########################
########################################################

@v1_router.get("/blocks", tags=["blocks"], ...)
async def list_blocks():
    pass

########################################################
##################### Graphs ###########################
########################################################

@v1_router.get("/graphs", tags=["graphs"], ...)
async def list_graphs(...):
    pass

@v1_router.post("/graphs", tags=["graphs"], ...)
async def create_graph(...):
    pass

# ... 更多端点
```

**优点**：
- 简单直接，所有端点一目了然
- 适合早期开发或小型 API

**缺点**：

- 文件巨大（1343 行）
- 难以多人协作
- 不同功能混在一起

**V2 模块化路由器 ([v2/store/](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/v2/store:0:0-0:0))**

**特点**：每个功能域是独立的包

```python
# backend/server/v2/store/routes.py

import fastapi
from autogpt_libs.auth import requires_user, get_user_id

# 创建路由器
router = fastapi.APIRouter()

##############################################
############### Profile Endpoints ############
##############################################

@router.get(
    "/profile",
    summary="Get user profile",
    tags=["store", "private"],
    dependencies=[fastapi.Security(requires_user)],
    response_model=backend.server.v2.store.model.ProfileDetails,
)
async def get_profile(
    user_id: str = fastapi.Security(get_user_id),
):
    """Get the profile details for the authenticated user."""
    profile = await backend.server.v2.store.db.get_user_profile(user_id)
    return profile

@router.post(
    "/profile",
    summary="Update user profile",
    tags=["store", "private"],
    dependencies=[fastapi.Security(requires_user)],
)
async def update_or_create_profile(
    profile: backend.server.v2.store.model.Profile,
    user_id: str = fastapi.Security(get_user_id),
):
    """Update the store profile for the authenticated user."""
    updated = await backend.server.v2.store.db.update_profile(user_id, profile)
    return updated

##############################################
############### Submission Endpoints #########
##############################################

@router.post("/submissions", ...)
async def create_submission(...):
    pass

# ... 更多端点
```

**包结构**：
```
v2/store/
├── __init__.py       # 包标识
├── routes.py         # API 端点定义
├── db.py             # 数据库操作
├── model.py          # Pydantic 模型
├── cache.py          # 缓存逻辑
├── media.py          # 媒体处理
└── exceptions.py     # 自定义异常
```

**优点**：
- 功能独立，易于维护
- 每个文件专注一个职责
- 支持多人并行开发
- 易于测试和重用

### 10.3.3 [external/](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/external:0:0-0:0) 外部 API

AutoGPT 将**外部集成 API** 独立为一个完整的 FastAPI 应用。

**独立应用架构 ([external/api.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/external/api.py:0:0-0:0))**

```python
# backend/server/external/api.py

from fastapi import FastAPI
from .routes.v1 import v1_router

# 创建独立的 FastAPI 应用
external_app = FastAPI(
    title="AutoGPT External API",
    description="External API for AutoGPT integrations",
    docs_url="/docs",
    version="1.0",
)

# 添加专用中间件
external_app.add_middleware(SecurityHeadersMiddleware)

# 包含路由器
external_app.include_router(v1_router, prefix="/v1")

# 添加 Prometheus 监控
instrument_fastapi(
    external_app,
    service_name="external-api",
    expose_endpoint=True,
    endpoint="/metrics",
)
```

**外部 API 路由器 ([external/routes/v1.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/external/routes/v1.py:0:0-0:0))**

```python
# backend/server/external/routes/v1.py

from fastapi import APIRouter, Security
from prisma.enums import APIKeyPermission
from backend.server.external.middleware import require_permission

v1_router = APIRouter()

# 使用 API Key 认证（而非 JWT）
@v1_router.get(
    "/blocks",
    tags=["blocks"],
    dependencies=[Security(require_permission(APIKeyPermission.READ_BLOCK))],
)
async def get_graph_blocks():
    """外部 API 使用 API Key 认证"""
    blocks = backend.data.block.get_blocks()
    return [b.to_dict() for b in blocks.values() if not b.disabled]

@v1_router.post(
    "/blocks/{block_id}/execute",
    tags=["blocks"],
    dependencies=[Security(require_permission(APIKeyPermission.EXECUTE_BLOCK))],
)
async def execute_graph_block(
    block_id: str,
    data: BlockInput,
    api_key: APIKeyInfo = Security(require_permission(APIKeyPermission.EXECUTE_BLOCK)),
):
    """执行 block - 需要 EXECUTE_BLOCK 权限"""
    obj = backend.data.block.get_block(block_id)
    output = defaultdict(list)
    async for name, data in obj.execute(data):
        output[name].append(data)
    return output
```

**挂载外部应用 ([rest_api.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/rest_api.py:0:0-0:0))**

```python
# backend/server/rest_api.py

from backend.server.external.api import external_app

app = FastAPI()

# 主应用的路由器
app.include_router(v1_router, prefix="/api")
app.include_router(v2_store_router, prefix="/api/store")

# 挂载外部 API 为子应用
app.mount("/external-api", external_app)

# 结果：
# - 主应用: http://localhost:8000/api/...
# - 外部 API: http://localhost:8000/external-api/v1/...
```

**为什么独立外部 API？**

1. **不同的认证机制**
   - 主 API: JWT 用户认证
   - 外部 API: API Key 认证

2. **不同的安全策略**
   - 外部 API 有专用的速率限制、权限检查

3. **独立的文档**
   - 外部 API 有自己的 OpenAPI 文档
   - 便于第三方开发者集成

4. **隔离故障**
   - 外部 API 问题不影响主应用

---

### 10.3.4 路由版本化策略

AutoGPT 使用 **URL 路径版本化** 策略，同时运行 V1 和 V2 API。

**版本化方案对比 :**

**① URL 路径版本化（AutoGPT 使用）**
```
GET /api/v1/graphs
GET /api/v2/store/submissions
```
- ✅ 清晰直观
- ✅ 易于缓存和路由
- ✅ 支持同时运行多个版本
- ❌ URL 会变化

**② Header 版本化**
```
GET /api/graphs
Header: API-Version: 2
```
- ✅ URL 保持不变
- ❌ 缓存困难
- ❌ 调试不便

**③ Query 参数版本化**
```
GET /api/graphs?version=2
```
- ❌ 污染查询参数
- ❌ 不推荐

**AutoGPT 版本化实现 :**

**主应用集成多版本** ([rest_api.py](cci:7://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/server/rest_api.py:0:0-0:0))：

```python
# backend/server/rest_api.py

from fastapi import FastAPI

app = FastAPI(
    title="AutoGPT Platform API",
    version="1.0",
)

# V1 API - 路径前缀 /api
app.include_router(
    backend.server.routers.v1.v1_router,
    tags=["v1"],
    prefix="/api"
)
# 结果: /api/graphs, /api/blocks, /api/auth/user

# V2 API - 路径前缀 /api/{feature}
app.include_router(
    backend.server.v2.store.routes.router,
    tags=["v2"],
    prefix="/api/store"
)
# 结果: /api/store/profile, /api/store/submissions

app.include_router(
    backend.server.v2.builder.routes.router,
    tags=["v2"],
    prefix="/api/builder"
)
# 结果: /api/builder/...

app.include_router(
    backend.server.v2.library.routes.router,
    tags=["v2"],
    prefix="/api/library"
)
# 结果: /api/library/...

# V2 Admin API - 特殊标签
app.include_router(
    backend.server.v2.admin.store_admin_routes.router,
    tags=["v2", "admin"],
    prefix="/api/store"  # 与 store 共享前缀
)
# 结果: /api/store/admin/...

# 外部 API - 独立挂载
app.mount("/external-api", external_app)
# 结果: /external-api/v1/blocks
```

**版本演进路径**：

```
Phase 1 (初期):
/api/graphs
/api/blocks
/api/executions

Phase 2 (V2 引入):
/api/graphs          (V1 - 保持兼容)
/api/store/...       (V2 - 新功能)
/api/builder/...     (V2 - 新功能)

Phase 3 (V1 废弃):
/api/v1/graphs       (V1 - 显式版本号，准备废弃)
/api/store/...       (V2 - 成为主要 API)

Phase 4 (只保留 V2):
/api/store/...
/api/builder/...
```

**版本共存策略 :**

**1. 渐进式迁移**

```python
# V1 端点（旧代码）
@v1_router.get("/graphs")
async def list_graphs_v1(user_id: str = Depends(get_user_id)):
    """V1: 简单的图列表"""
    return await graph_db.get_graphs(user_id)

# V2 端点（新功能）
@v2_library_router.get("/agents")
async def list_agents_v2(
    user_id: str = Security(get_user_id),
    filters: Optional[AgentFilters] = None,
):
    """V2: 带过滤和搜索的 agent 列表"""
    return await library_db.list_agents(user_id, filters)
```

**2. 功能标志控制**

```python
from backend.util.feature_flag import is_feature_enabled

@router.get("/new-feature")
async def new_feature(user_id: str = Security(get_user_id)):
    if not await is_feature_enabled("v2_new_feature", user_id):
        raise HTTPException(status_code=404, detail="Feature not available")
    
    return await v2_service.new_feature(user_id)
```

**3. API 废弃策略**

```python
from fastapi import status
from warnings import warn

@v1_router.get(
    "/old-endpoint",
    deprecated=True,  # OpenAPI 文档显示已废弃
    response_headers={
        "X-API-Deprecated": "This endpoint is deprecated, use /api/v2/new-endpoint",
        "Sunset": "Sun, 01 Jan 2025 00:00:00 GMT",  # RFC 8594
    }
)
async def old_endpoint():
    warn("old_endpoint is deprecated", DeprecationWarning)
    return {"message": "This endpoint is deprecated"}
```

### 10.3.5 模块化最佳实践

基于 AutoGPT 的经验总结：

 **DO - 推荐做法 :**

**1. 功能内聚的模块**

```
v2/store/
├── routes.py      # API 端点
├── db.py          # 数据库操作
├── model.py       # 数据模型
├── cache.py       # 缓存逻辑
└── exceptions.py  # 自定义异常
```

**2. 清晰的路由器定义**

```python
# Good: 明确的元数据
router = APIRouter(
    prefix="/users",
    tags=["User Management"],
    responses={404: {"description": "User not found"}},
)
```

**3. 路由器级别的依赖**

```python
# Good: 整个路由器都需要认证
router = APIRouter(
    prefix="/admin",
    dependencies=[Security(requires_admin)],
)
```

**4. 使用子路由器组织**

```python
# Good: 复杂功能使用子路由器
from .users import router as users_router
from .posts import router as posts_router

api_router = APIRouter(prefix="/api/v1")
api_router.include_router(users_router, prefix="/users")
api_router.include_router(posts_router, prefix="/posts")
```

**5. 标签用于文档分组**

```python
# Good: 标签帮助组织 API 文档
app.include_router(
    store_router,
    prefix="/store",
    tags=["Store", "Public"]
)

app.include_router(
    admin_router,
    prefix="/store",
    tags=["Store", "Admin"]
)
```

**DON'T - 避免做法 :**

**1. 不要创建过深的嵌套**

```python
# Bad: 嵌套太深
/api/v1/users/profile/settings/notifications/email
```

**2. 不要在单文件中混合多个功能**

```python
# Bad: users.py 包含用户、订单、支付...
@router.get("/users")
@router.get("/orders")
@router.get("/payments")
```

**3. 不要在路由器中写业务逻辑**

```python
# Bad: 业务逻辑在路由器中
@router.post("/users")
async def create_user(user: User):
    # 大量业务逻辑...
    if user.age < 18:
        # 验证逻辑
    # 数据库操作
    # 发送邮件
    pass

# Good: 业务逻辑在服务层
@router.post("/users")
async def create_user(user: User):
    return await user_service.create_user(user)
```

**4. 不要硬编码路径**

```python
# Bad
@router.get("/api/v1/users/{user_id}")

# Good: 在 include_router 中设置
@router.get("/{user_id}")  # 相对路径

app.include_router(router, prefix="/api/v1/users")
```

---

# 11 响应模型最佳实践

---

## 11.1 基础复习

### 11.1.1 DTO 模式 (Data Transfer Object)

**DTO** 是在不同层之间传输数据的对象，用于解耦内部数据模型和外部 API 接口。

**为什么需要 DTO？**

```
不使用DTO会直接暴露数据库模型的问题：
1. 泄露内部实现细节（数据库字段名）
2. 包含敏感字段（密码哈希、内部ID）
3. 缺少计算字段
4. 版本演进困难

使用 DTO 的优势：
1. 控制暴露的字段
2. 添加计算属性
3. API 和数据库模型独立演进
4. 类型安全
```

**DTO 模式示例**

```python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional

# ❌ 数据库模型（ORM）
class UserDB:
    """数据库模型 - 不应直接暴露"""
    id: int
    username: str
    email: str
    password_hash: str      # 敏感！
    created_at: datetime
    updated_at: datetime
    is_deleted: bool        # 内部标记
    internal_notes: str     # 内部字段

# ✅ 响应 DTO
class UserResponse(BaseModel):
    """API 响应模型 - 只包含需要暴露的字段"""
    id: str                 # 转换为字符串
    username: str
    email: str
    member_since: datetime = Field(alias="created_at")
    # 不包含 password_hash, is_deleted, internal_notes
    
    class Config:
        from_attributes = True  # 允许从 ORM 对象创建

# ✅ 创建 DTO
class UserCreate(BaseModel):
    """创建用户的请求模型"""
    username: str = Field(..., min_length=3, max_length=50)
    email: str = Field(..., pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    password: str = Field(..., min_length=8)
    # 注意：没有 id, created_at 等

# ✅ 更新 DTO
class UserUpdate(BaseModel):
    """更新用户的请求模型"""
    email: Optional[str] = None
    password: Optional[str] = None
    # 所有字段都是可选的
```

**使用场景**：

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/users", response_model=UserResponse)
async def create_user(user_data: UserCreate):
    """
    输入：UserCreate DTO（验证用户输入）
    输出：UserResponse DTO（控制返回字段）
    """
    # 1. 验证数据（Pydantic 自动）
    # 2. 业务逻辑
    password_hash = hash_password(user_data.password)
    
    # 3. 创建数据库对象
    db_user = UserDB(
        username=user_data.username,
        email=user_data.email,
        password_hash=password_hash,  # 存储哈希
        created_at=datetime.now(),
    )
    await db.save(db_user)
    
    # 4. 转换为响应 DTO
    return UserResponse.from_orm(db_user)
    # 返回时自动过滤掉 password_hash 等敏感字段
```

### 11.1.2 数据转换

数据转换是将内部数据结构转换为 API 响应格式的过程。

**转换模式**

**① 字段重命名**

```python
class UserDB:
    user_id: int
    email_address: str

class UserResponse(BaseModel):
    id: int = Field(alias="user_id")        # 输入时使用 user_id
    email: str = Field(alias="email_address")
    
    class Config:
        populate_by_name = True  # 允许使用原字段名或别名
```

**② 类型转换**

```python
from uuid import UUID
from datetime import datetime

class GraphDB:
    id: UUID                    # 数据库使用 UUID
    created_at: datetime        # 数据库使用 datetime

class GraphResponse(BaseModel):
    id: str                     # API 返回字符串
    created_at: int             # API 返回时间戳
    
    @classmethod
    def from_db(cls, db_obj: GraphDB):
        return cls(
            id=str(db_obj.id),                          # UUID -> str
            created_at=int(db_obj.created_at.timestamp())  # datetime -> timestamp
        )
```

**③ 计算字段**

```python
from pydantic import computed_field

class OrderResponse(BaseModel):
    items: list[OrderItem]
    
    @computed_field
    @property
    def total_price(self) -> float:
        """计算总价 - 不存储在数据库中"""
        return sum(item.price * item.quantity for item in self.items)
    
    @computed_field
    @property
    def item_count(self) -> int:
        """计算商品数量"""
        return len(self.items)
```

**④ 嵌套转换**

```python
class UserDB:
    id: int
    profile: ProfileDB  # 关联的 profile 对象

class ProfileResponse(BaseModel):
    bio: str
    avatar_url: str

class UserResponse(BaseModel):
    id: int
    profile: ProfileResponse  # 嵌套的响应模型
    
    @classmethod
    def from_db(cls, user: UserDB):
        return cls(
            id=user.id,
            profile=ProfileResponse(
                bio=user.profile.bio,
                avatar_url=user.profile.avatar_url
            )
        )
```

### 11.1.3 字段过滤

字段过滤允许根据不同场景返回不同的字段集合。

**过滤策略**

**① 排除未设置的字段**

```python
class UserResponse(BaseModel):
    username: str
    email: str
    phone: Optional[str] = None  # 可选字段
    bio: Optional[str] = None

# 场景：用户没有填写 phone 和 bio
user = UserResponse(username="alice", email="alice@example.com")

# 默认行为
user.model_dump()
# {"username": "alice", "email": "alice@example.com", "phone": null, "bio": null}

# 排除未设置的字段
user.model_dump(exclude_unset=True)
# {"username": "alice", "email": "alice@example.com"}
```

**② 排除 None 值**

```python
user = UserResponse(
    username="alice",
    email="alice@example.com",
    phone=None,  # 显式设置为 None
    bio="Developer"
)

user.model_dump(exclude_none=True)
# {"username": "alice", "email": "alice@example.com", "bio": "Developer"}
# phone 被排除
```

**③ 选择性包含/排除**

```python
# 只包含特定字段
user.model_dump(include={"username", "email"})
# {"username": "alice", "email": "alice@example.com"}

# 排除特定字段
user.model_dump(exclude={"email"})
# {"username": "alice", "phone": null, "bio": "Developer"}
```

**④ 多级字段过滤**

```python
class AddressResponse(BaseModel):
    street: str
    city: str
    country: str

class UserResponse(BaseModel):
    username: str
    email: str
    address: AddressResponse

user.model_dump(
    exclude={
        "email": True,          # 排除 email
        "address": {"street"}   # 排除 address.street
    }
)
# {
#     "username": "alice",
#     "address": {"city": "Beijing", "country": "China"}
# }
```

### 11.1.4 序列化 (Serialization)

序列化是将 Python 对象转换为可传输格式（JSON）的过程。

**Pydantic 序列化**

```python
from pydantic import BaseModel
from datetime import datetime
from uuid import UUID
from enum import Enum

class Status(str, Enum):
    PENDING = "pending"
    ACTIVE = "active"

class TaskResponse(BaseModel):
    id: UUID
    title: str
    status: Status
    created_at: datetime
    metadata: dict

# 创建对象
task = TaskResponse(
    id=UUID("550e8400-e29b-41d4-a716-446655440000"),
    title="Task 1",
    status=Status.ACTIVE,
    created_at=datetime(2024, 1, 1, 12, 0, 0),
    metadata={"priority": "high"}
)

# 序列化为字典
task.model_dump()
# {
#     "id": UUID('550e8400-e29b-41d4-a716-446655440000'),
#     "title": "Task 1",
#     "status": "active",
#     "created_at": datetime(2024, 1, 1, 12, 0, 0),
#     "metadata": {"priority": "high"}
# }

# 序列化为 JSON 字符串
task.model_dump_json()
# '{"id":"550e8400-e29b-41d4-a716-446655440000","title":"Task 1",...}'

# 自定义序列化模式
task.model_dump(mode="json")  # 强制使用 JSON 兼容类型
# {
#     "id": "550e8400-e29b-41d4-a716-446655440000",  # UUID -> str
#     "created_at": "2024-01-01T12:00:00",           # datetime -> ISO string
#     ...
# }
```

**自定义序列化器**

```python
from pydantic import field_serializer

class UserResponse(BaseModel):
    username: str
    email: str
    created_at: datetime
    
    @field_serializer("email")
    def serialize_email(self, value: str) -> str:
        """隐藏部分邮箱地址"""
        username, domain = value.split("@")
        return f"{username[:2]}***@{domain}"
    
    @field_serializer("created_at")
    def serialize_datetime(self, value: datetime) -> int:
        """转换为时间戳"""
        return int(value.timestamp())

user = UserResponse(
    username="alice",
    email="alice@example.com",
    created_at=datetime(2024, 1, 1)
)

user.model_dump()
# {
#     "username": "alice",
#     "email": "al***@example.com",      # 部分隐藏
#     "created_at": 1704067200           # 时间戳
# }
```

---

我没有遇到错误，继续为你讲解 FastAPI 的响应模型特性：

## 11.2 FastAPI 特性

### 11.2.1 response_model 高级用法

`response_model` 是 FastAPI 的核心功能，用于定义端点的响应格式。

**基本用法**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class UserResponse(BaseModel):
    id: int
    username: str
    email: str

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int):
    """
    response_model 的作用：
    1. 验证返回数据
    2. 过滤不需要的字段
    3. 生成 OpenAPI 文档
    4. 提供类型提示
    """
    # 即使返回包含额外字段，也只返回 response_model 中定义的字段
    return {
        "id": user_id,
        "username": "alice",
        "email": "alice@example.com",
        "password_hash": "secret",  # 会被自动过滤掉
        "internal_id": 12345        # 会被自动过滤掉
    }
```

**返回列表**

```python
@app.get("/users", response_model=list[UserResponse])
async def list_users():
    """返回用户列表"""
    return [
        {"id": 1, "username": "alice", "email": "alice@example.com"},
        {"id": 2, "username": "bob", "email": "bob@example.com"},
    ]
```

**联合类型响应**

```python
from typing import Union

class SuccessResponse(BaseModel):
    status: str = "success"
    data: dict

class ErrorResponse(BaseModel):
    status: str = "error"
    message: str

@app.get("/data", response_model=Union[SuccessResponse, ErrorResponse])
async def get_data(fail: bool = False):
    """根据条件返回不同类型的响应"""
    if fail:
        return ErrorResponse(message="Something went wrong")
    return SuccessResponse(data={"value": 123})
```

**泛型响应**

```python
from typing import Generic, TypeVar

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    """通用响应包装器"""
    success: bool
    data: T
    message: str = ""

@app.get("/users/{user_id}", response_model=ApiResponse[UserResponse])
async def get_user(user_id: int):
    user = await db.get_user(user_id)
    return ApiResponse(
        success=True,
        data=user,
        message="User retrieved successfully"
    )

@app.get("/posts", response_model=ApiResponse[list[PostResponse]])
async def list_posts():
    posts = await db.get_posts()
    return ApiResponse(
        success=True,
        data=posts
    )
```

### 11.2.2 response_model_exclude_unset

排除未设置的字段，只返回明确设置过的字段。

```python
from typing import Optional

class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    phone: Optional[str] = None
    bio: Optional[str] = None

@app.get(
    "/users/{user_id}",
    response_model=UserResponse,
    response_model_exclude_unset=True  # 排除未设置的字段
)
async def get_user(user_id: int):
    # 只设置了部分字段
    return {"id": user_id, "username": "alice", "email": "alice@example.com"}
    # 返回: {"id": 1, "username": "alice", "email": "alice@example.com"}
    # phone 和 bio 不会出现在响应中

@app.get(
    "/users/{user_id}/full",
    response_model=UserResponse,
    response_model_exclude_unset=False  # 默认行为
)
async def get_user_full(user_id: int):
    return {"id": user_id, "username": "alice", "email": "alice@example.com"}
    # 返回: {
    #   "id": 1,
    #   "username": "alice",
    #   "email": "alice@example.com",
    #   "phone": null,
    #   "bio": null
    # }
```

**使用场景**：
- PATCH 更新：只返回被更新的字段
- 减少响应体积：不返回空值
- 向后兼容：新增字段不影响现有客户端

### 11.2.3 response_model_exclude_none

排除值为 `None` 的字段。

```python
class ProductResponse(BaseModel):
    id: int
    name: str
    description: Optional[str] = None
    discount: Optional[float] = None
    tags: Optional[list[str]] = None

@app.get(
    "/products/{product_id}",
    response_model=ProductResponse,
    response_model_exclude_none=True  # 排除 None 值
)
async def get_product(product_id: int):
    return {
        "id": product_id,
        "name": "Laptop",
        "description": "A powerful laptop",
        "discount": None,  # 显式设置为 None
        "tags": None       # 显式设置为 None
    }
    # 返回: {"id": 1, "name": "Laptop", "description": "A powerful laptop"}
    # discount 和 tags 被排除
```

**`exclude_unset` vs `exclude_none`**：

```python
class Model(BaseModel):
    a: Optional[str] = None
    b: Optional[str] = None

obj = Model(a="value", b=None)

# exclude_unset=True: 排除未设置的字段
obj.model_dump(exclude_unset=True)
# {"a": "value", "b": null}  # b 被设置了（虽然是 None）

# exclude_none=True: 排除值为 None 的字段
obj.model_dump(exclude_none=True)
# {"a": "value"}  # b 的值是 None，被排除

# 同时使用
obj.model_dump(exclude_unset=True, exclude_none=True)
# {"a": "value"}
```

### 11.2.4 response_model_include / response_model_exclude

精确控制包含或排除的字段。

```python
class UserDetailResponse(BaseModel):
    id: int
    username: str
    email: str
    phone: str
    address: str
    created_at: datetime
    last_login: datetime

# 只返回基本信息
@app.get(
    "/users/{user_id}/basic",
    response_model=UserDetailResponse,
    response_model_include={"id", "username", "email"}
)
async def get_user_basic(user_id: int):
    user = await db.get_user(user_id)
    return user
    # 只返回: {"id": 1, "username": "alice", "email": "alice@example.com"}

# 排除敏感信息
@app.get(
    "/users/{user_id}/public",
    response_model=UserDetailResponse,
    response_model_exclude={"email", "phone", "address"}
)
async def get_user_public(user_id: int):
    user = await db.get_user(user_id)
    return user
    # 返回除了 email, phone, address 之外的所有字段
```

**动态字段选择**：

```python
from fastapi import Query

@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    fields: Optional[str] = Query(None, description="Comma-separated fields")
):
    """支持客户端选择返回的字段"""
    user = await db.get_user(user_id)
    
    if fields:
        field_set = set(fields.split(","))
        return UserDetailResponse(**user).model_dump(include=field_set)
    
    return UserDetailResponse(**user)

# 使用: GET /users/1?fields=id,username,email
```

### 11.2.5 多种响应状态

定义不同 HTTP 状态码的响应模型。

```python
from fastapi import status

class UserResponse(BaseModel):
    id: int
    username: str

class ErrorResponse(BaseModel):
    detail: str

@app.post(
    "/users",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    responses={
        201: {
            "description": "User created successfully",
            "model": UserResponse,
        },
        400: {
            "description": "Invalid input",
            "model": ErrorResponse,
        },
        409: {
            "description": "User already exists",
            "model": ErrorResponse,
        },
    }
)
async def create_user(user_data: UserCreate):
    """
    创建用户，支持多种响应状态
    
    - 201: 创建成功
    - 400: 输入验证失败
    - 409: 用户名已存在
    """
    if await db.user_exists(user_data.username):
        raise HTTPException(
            status_code=409,
            detail="Username already exists"
        )
    
    user = await db.create_user(user_data)
    return user
```

**使用 `responses` 参数的好处**：
- 完善的 OpenAPI 文档
- 客户端生成器可以创建类型安全的代码
- 清晰的 API 契约

### 11.2.6  响应示例

在 OpenAPI 文档中提供示例数据。

```python
from pydantic import Field

class UserResponse(BaseModel):
    id: int = Field(..., examples=[1, 42, 123])
    username: str = Field(..., examples=["alice", "bob"])
    email: str = Field(..., examples=["alice@example.com"])
    
    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "id": 1,
                    "username": "alice",
                    "email": "alice@example.com"
                },
                {
                    "id": 2,
                    "username": "bob",
                    "email": "bob@example.com"
                }
            ]
        }
    }

@app.get(
    "/users/{user_id}",
    response_model=UserResponse,
    responses={
        200: {
            "description": "Successful Response",
            "content": {
                "application/json": {
                    "example": {
                        "id": 1,
                        "username": "alice",
                        "email": "alice@example.com"
                    }
                }
            }
        }
    }
)
async def get_user(user_id: int):
    return await db.get_user(user_id)
```

---

## 11.3 AutoGPT中的响应模型实战分析

### 11.3.1 Graph 响应模型设计

AutoGPT 的 [Graph](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:300:0-397:66) 模型展示了复杂响应模型的最佳实践：

**核心 Graph 模型**

```python
# backend/data/graph.py

class Link(BaseDbModel):
    """连接关系模型"""
    source_id: str
    sink_id: str
    source_name: str
    sink_name: str
    is_static: bool = False
    
    @staticmethod
    def from_db(link: AgentNodeLink):
        """从数据库模型转换"""
        return Link(
            id=link.id,
            source_name=link.sourceName,
            source_id=link.agentNodeSourceId,
            sink_name=link.sinkName,
            sink_id=link.agentNodeSinkId,
            is_static=link.isStatic,
        )

class Node(BaseDbModel):
    """节点基础模型"""
    block_id: str
    input_default: BlockInput = {}
    metadata: dict[str, Any] = {}
    input_links: list[Link] = []
    output_links: list[Link] = []
    
    @property
    def block(self) -> "Block":
        """延迟加载 block 信息"""
        block = get_block(self.block_id)
        if not block:
            logger.warning(f"Block #{self.block_id} missing, using UnknownBlock")
            return _UnknownBlockBase(self.block_id)
        return block

class Graph(BaseGraph):
    """完整的 Graph 响应模型"""
    sub_graphs: list[BaseGraph] = []
    
    # 计算字段 - 动态生成
    @computed_field
    @property
    def input_schema(self) -> dict[str, Any]:
        """动态生成输入 schema"""
        return self._generate_schema(...)
    
    @computed_field
    @property
    def output_schema(self) -> dict[str, Any]:
        """动态生成输出 schema"""
        return self._generate_schema(...)
    
    @computed_field
    @property
    def has_external_trigger(self) -> bool:
        """是否有外部触发器"""
        return self.webhook_input_node is not None
    
    @computed_field
    @property
    def credentials_input_schema(self) -> dict[str, Any]:
        """聚合所有凭证输入"""
        return self._credentials_input_schema.jsonschema()
```

**设计亮点**：

1. **`@computed_field`** - 动态计算字段
   - 不存储在数据库中
   - 响应时自动计算
   - 减少数据冗余

2. **[from_db](cci:1://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:61:4-70:9) 静态方法** - 清晰的转换逻辑
   - 封装数据库字段名到 API 字段名的映射
   - 统一的转换入口

3. **嵌套模型** - 复杂数据结构
   - [Node](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:76:0-93:20) 包含 [Link](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:54:0-73:85) 列表
   - [Graph](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:300:0-397:66) 包含 [Node](cci:2://file:///d:/%E8%BD%AC%E7%A0%81/AI-all/AutoGPT/autogpt_platform/backend/backend/data/graph.py:76:0-93:20) 列表
   - 保持类型安全

**数据导出的字段过滤**

```python
class NodeModel(Node):
    def stripped_for_export(self) -> "NodeModel":
        """导出时移除敏感信息"""
        stripped_node = self.model_copy(deep=True)
        
        # 移除凭证和密钥
        if stripped_node.input_default:
            stripped_node.input_default = self._filter_secrets_from_node_input(
                stripped_node.input_default,
                self.block.input_schema.jsonschema()
            )
        
        # 移除密钥字段的默认值
        if (
            stripped_node.block.block_type == BlockType.INPUT
            and stripped_node.input_default.get("secret", False)
            and "value" in stripped_node.input_default
        ):
            del stripped_node.input_default["value"]
        
        # 移除 webhook 信息
        stripped_node.webhook_id = None
        stripped_node.webhook = None
        
        return stripped_node
```

**使用场景**：

```python
@app.get("/graphs/{graph_id}/export")
async def export_graph(graph_id: str, user_id: str = Depends(get_user_id)):
    """导出 graph - 移除敏感信息"""
    graph = await get_graph(graph_id, user_id)
    
    # 每个节点都移除敏感信息
    export_nodes = [node.stripped_for_export() for node in graph.nodes]
    
    return {
        "graph": graph,
        "nodes": export_nodes  # 已清理的节点
    }
```

### 11.3.2 Block 响应模型

```python
# backend/data/block.py

class BlockInfo(BaseModel):
    """Block 信息的 API 响应模型"""
    id: str
    name: str
    inputSchema: dict[str, Any]    # 驼峰命名 - 前端友好
    outputSchema: dict[str, Any]
    costs: list[BlockCost]
    description: str
    categories: list[dict[str, str]]
    contributors: list[dict[str, Any]]
    staticOutput: bool             # 驼峰命名
    uiType: str

class BlockCost(BaseModel):
    """Block 成本信息"""
    cost_amount: int
    cost_filter: BlockInput
    cost_type: BlockCostType
    
    def __init__(
        self,
        cost_amount: int,
        cost_type: BlockCostType = BlockCostType.RUN,
        cost_filter: Optional[BlockInput] = None,
        **data: Any,
    ):
        """自定义初始化 - 提供默认值"""
        super().__init__(
            cost_amount=cost_amount,
            cost_filter=cost_filter or {},  # 默认空字典
            cost_type=cost_type,
            **data,
        )
```

**命名约定**：
- Python 代码：`snake_case`
- API 响应：部分使用 `camelCase`（前端友好）
- 通过 Pydantic 的 `alias` 处理

### 11.3.3 分页响应处理

AutoGPT 实现了统一的分页模型：

```python
# backend/util/models.py

class Pagination(pydantic.BaseModel):
    """统一的分页信息"""
    total_items: int = pydantic.Field(
        description="Total number of items.",
        examples=[42]
    )
    total_pages: int = pydantic.Field(
        description="Total number of pages.",
        examples=[2]
    )
    current_page: int = pydantic.Field(
        description="Current page number.",
        examples=[1]
    )
    page_size: int = pydantic.Field(
        description="Number of items per page.",
        examples=[25]
    )
    
    @staticmethod
    def empty() -> "Pagination":
        """创建空分页对象"""
        return Pagination(
            total_items=0,
            total_pages=0,
            current_page=0,
            page_size=0,
        )
```

**分页响应包装器**

```python
# backend/server/v2/store/model.py

class StoreAgentsResponse(pydantic.BaseModel):
    """商店 agents 列表响应"""
    agents: list[StoreAgent]
    pagination: Pagination

class StoreSubmissionsResponse(pydantic.BaseModel):
    """提交列表响应"""
    submissions: list[StoreSubmission]
    pagination: Pagination

class CreatorsResponse(pydantic.BaseModel):
    """创作者列表响应"""
    creators: list[Creator]
    pagination: Pagination
```

**统一模式**：
```
{
  "data": [...],      # 数据数组
  "pagination": {     # 分页信息
    "total_items": 100,
    "total_pages": 10,
    "current_page": 1,
    "page_size": 10
  }
}
```

**分页端点实现**

```python
# backend/server/v2/store/routes.py

@router.get(
    "/agents",
    response_model=StoreAgentsResponse,
    tags=["store", "public"],
)
async def get_agents(
    featured: bool = False,
    creators: Optional[str] = Query(None),
    sorted_by: Optional[str] = Query(None),
    search_query: Optional[str] = Query(None),
    category: Optional[str] = Query(None),
    page: int = Query(1, ge=1),           # 最小值为 1
    page_size: int = Query(20, ge=1, le=100),  # 限制范围
):
    """
    获取商店 agents 列表
    
    Args:
        page: 页码（从 1 开始）
        page_size: 每页数量（1-100）
    
    Returns:
        StoreAgentsResponse: 带分页信息的响应
    """
    # 验证参数
    if page < 1:
        raise HTTPException(status_code=400, detail="Page must be >= 1")
    if page_size < 1 or page_size > 100:
        raise HTTPException(status_code=400, detail="Page size must be 1-100")
    
    # 查询数据
    agents_data = await db.get_store_agents(
        featured=featured,
        creators=creators,
        sorted_by=sorted_by,
        search_query=search_query,
        category=category,
        page=page,
        page_size=page_size,
    )
    
    return StoreAgentsResponse(
        agents=agents_data.agents,
        pagination=Pagination(
            total_items=agents_data.total,
            total_pages=(agents_data.total + page_size - 1) // page_size,
            current_page=page,
            page_size=page_size,
        )
    )
```

### 11.3.4 错误响应模型

虽然 AutoGPT 主要依赖 FastAPI 的默认错误处理，但在某些地方使用了自定义错误响应：

```python
# backend/server/model.py

class WSMessage(pydantic.BaseModel):
    """WebSocket 消息 - 支持错误响应"""
    method: WSMethod
    data: Optional[dict[str, Any] | list[Any] | str] = None
    success: bool | None = None
    channel: str | None = None
    error: str | None = None  # 错误消息

# 使用示例
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    try:
        message = WSMessage.model_validate_json(data)
    except pydantic.ValidationError as e:
        # 返回错误响应
        await websocket.send_text(
            WSMessage(
                method=WSMethod.ERROR,
                success=False,
                error="Invalid message format. Review the schema and retry"
            ).model_dump_json()
        )
```

**HTTP 错误响应**：

```python
# 自定义异常
from backend.util.exceptions import NotFoundError, NotAuthorizedError

# 全局异常处理器
@app.exception_handler(NotFoundError)
async def not_found_handler(request: Request, exc: NotFoundError):
    return JSONResponse(
        status_code=404,
        content={"detail": str(exc)}
    )

# 端点中抛出
@app.get("/graphs/{graph_id}")
async def get_graph(graph_id: str):
    graph = await db.get_graph(graph_id)
    if not graph:
        raise NotFoundError(f"Graph #{graph_id} not found")
    return graph
```

### 11.3.5 字段选择性返回

AutoGPT 在某些场景使用字段选择：

**场景 1: 导出 vs 完整信息**

```python
# 完整信息（包含敏感数据）
@app.get("/graphs/{graph_id}", response_model=Graph)
async def get_graph(graph_id: str, user_id: str = Depends(get_user_id)):
    """返回完整的 graph 信息"""
    return await Graph.from_db(graph_id, user_id=user_id)

# 导出版本（移除敏感信息）
@app.get("/graphs/{graph_id}/export")
async def export_graph(graph_id: str, user_id: str = Depends(get_user_id)):
    """返回可导出的 graph（不含敏感信息）"""
    graph = await Graph.from_db(graph_id, user_id=user_id, for_export=True)
    return graph
```

**场景 2: 基础信息 vs 详细信息**

```python
class StoreAgent(pydantic.BaseModel):
    """商店列表中的简化信息"""
    slug: str
    agent_name: str
    agent_image: str
    creator: str
    creator_avatar: str
    sub_heading: str
    description: str
    runs: int
    rating: float

class StoreAgentDetails(pydantic.BaseModel):
    """详细页面的完整信息"""
    store_listing_version_id: str
    slug: str
    agent_name: str
    agent_video: str              # 额外字段
    agent_image: list[str]        # 多张图片
    creator: str
    creator_avatar: str
    sub_heading: str
    description: str
    instructions: str | None      # 额外字段
    categories: list[str]         # 额外字段
    runs: int
    rating: float
    versions: list[str]           # 额外字段
    last_updated: datetime        # 额外字段
    recommended_schedule_cron: str | None
    active_version_id: str | None
    has_approved_version: bool

# 列表端点 - 返回简化模型
@app.get("/agents", response_model=list[StoreAgent])
async def list_agents():
    return await db.get_agents()

# 详情端点 - 返回完整模型
@app.get("/agents/{slug}", response_model=StoreAgentDetails)
async def get_agent_details(slug: str):
    return await db.get_agent_by_slug(slug)
```

---

# 12  后台任务与长时间运行

---

## 12.1 基础复习

### 12.1.1 同步 vs 异步任务

**同步任务执行**

```python
# ❌ 同步执行 - 阻塞请求
@app.post("/send-email")
def send_email(email: str, message: str):
    # API 请求被阻塞，直到邮件发送完成
    smtp_client.send(email, message)  # 可能需要 5 秒
    return {"status": "Email sent"}
    # 客户端等待 5 秒才收到响应
```

**问题**：
- 用户体验差：必须等待任务完成
- 资源浪费：占用连接和线程
- 超时风险：任务太长可能超时
- 不可扩展：并发受限

**异步任务执行**

```python
# ✅ 异步执行 - 立即返回
from fastapi import BackgroundTasks

@app.post("/send-email")
async def send_email(
    email: str,
    message: str,
    background_tasks: BackgroundTasks
):
    # 将任务添加到后台队列
    background_tasks.add_task(send_email_task, email, message)
    
    # 立即返回，不等待邮件发送
    return {"status": "Email queued for sending"}
    # 客户端立即收到响应（几毫秒）

async def send_email_task(email: str, message: str):
    """后台执行"""
    await smtp_client.send(email, message)
```

**优势**：
- 快速响应：立即返回给用户
- 更好的用户体验
- 提高吞吐量
- 可以处理长时间任务

**何时使用后台任务**？

```
 适合后台任务：
- 发送邮件/通知
- 生成报告
- 图片处理
- 数据导入/导出
- 第三方 API 调用
- 日志记录
- 数据清理

 不适合后台任务：
- 需要立即返回结果的操作
- 用户直接依赖的数据查询
- 简单的数据库操作（通常很快）
- 认证/授权检查
```

### 12.1.2 任务队列概念

任务队列是管理和执行后台任务的系统。

**基本架构**

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   客户端     │──1──▶│  Web Server  │──2──▶│  Task Queue │
│             │      │   (FastAPI)  │      │   (内存/DB) │
└─────────────┘      └──────────────┘      └─────────────┘
                            ▲                      │
                            │                      │
                            │                      │ 3
                            └──────────────────────┘
                                  Worker 处理任务

流程：
1. 客户端发送请求
2. 服务器将任务加入队列，立即返回
3. Worker 从队列取任务执行
```

**任务队列类型**

**① 内存队列（简单）**

```python
import asyncio
from collections import deque

# 简单的内存队列
task_queue = deque()

async def add_task(func, *args):
    task_queue.append((func, args))

async def worker():
    """后台 worker"""
    while True:
        if task_queue:
            func, args = task_queue.popleft()
            await func(*args)
        await asyncio.sleep(0.1)
```

**优点**：简单、快速
**缺点**：服务器重启丢失、不能跨进程

**② 持久化队列（可靠）**
```python
# 使用 Redis/RabbitMQ/PostgreSQL
import redis

r = redis.Redis()

def add_task(task_id, task_data):
    """添加任务到 Redis 队列"""
    r.lpush("task_queue", json.dumps({
        "id": task_id,
        "data": task_data,
        "created_at": datetime.now().isoformat()
    }))

def get_task():
    """从队列取任务"""
    task_json = r.brpop("task_queue", timeout=5)
    if task_json:
        return json.loads(task_json[1])
    return None
```

**优点**：持久化、跨进程、可靠
**缺点**：需要额外服务

### 12.1.3 消息队列 

消息队列是更高级的任务分发系统。

**消息队列 vs 简单任务队列**

```
简单任务队列：
- 单一队列
- 简单的 FIFO
- 有限的功能

消息队列（RabbitMQ/Kafka）：
- 多队列/主题
- 路由规则
- 优先级
- 消息确认
- 死信队列
- 持久化
```

**RabbitMQ 基本概念**

```
┌──────────┐    ┌─────────┐    ┌─────────┐    ┌──────────┐
│ Producer │───▶│Exchange │───▶│ Queue   │───▶│ Consumer │
└──────────┘    └─────────┘    └─────────┘    └──────────┘
   发布者          交换机          队列           消费者

核心概念：
- Producer: 发送消息
- Exchange: 路由消息到队列
- Queue: 存储消息
- Consumer: 接收消息
- Binding: Exchange 到 Queue 的规则
```

**交换机类型**

```python
# Direct Exchange - 精确匹配
"""
routing_key = "email.send" -> Queue A
routing_key = "sms.send"   -> Queue B
"""

# Topic Exchange - 模式匹配
"""
routing_key = "email.*"     -> Queue A (email.send, email.verify)
routing_key = "*.critical"  -> Queue B (email.critical, sms.critical)
"""

# Fanout Exchange - 广播
"""
所有绑定的队列都收到消息
"""

# Headers Exchange - 基于消息属性
"""
根据消息头部属性路由
"""
```

### 12.1.4 任务状态管理

长时间运行的任务需要状态跟踪。

**任务生命周期**

```python
from enum import Enum

class TaskStatus(str, Enum):
    PENDING = "pending"       # 等待执行
    QUEUED = "queued"         # 已加入队列
    RUNNING = "running"       # 执行中
    COMPLETED = "completed"   # 成功完成
    FAILED = "failed"         # 失败
    CANCELLED = "cancelled"   # 已取消
    RETRY = "retry"           # 等待重试

class Task(BaseModel):
    id: str
    status: TaskStatus
    created_at: datetime
    started_at: Optional[datetime] = None
    completed_at: Optional[datetime] = None
    progress: float = 0.0  # 0-100
    result: Optional[Any] = None
    error: Optional[str] = None
    retry_count: int = 0
    max_retries: int = 3
```

**状态转换**

```
    ┌─────────┐
    │ PENDING │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │ QUEUED  │
    └────┬────┘
         │
         ▼
    ┌─────────┐ ──error──▶ ┌────────┐
    │ RUNNING │             │ RETRY  │
    └────┬────┘             └───┬────┘
         │                      │
    ┌────┴────┐                 │
    │         │                 │
    ▼         ▼                 │
┌──────────┐ ┌────────┐         │
│COMPLETED │ │ FAILED │◀────────┘
└──────────┘ └────────┘
```

**状态存储**

```python
import redis
from typing import Optional

class TaskStore:
    """任务状态存储"""
    
    def __init__(self):
        self.redis = redis.Redis()
    
    def create_task(self, task_id: str, task_data: dict) -> Task:
        """创建任务"""
        task = Task(
            id=task_id,
            status=TaskStatus.PENDING,
            created_at=datetime.now(),
            **task_data
        )
        self.redis.setex(
            f"task:{task_id}",
            3600 * 24,  # 24小时过期
            task.model_dump_json()
        )
        return task
    
    def get_task(self, task_id: str) -> Optional[Task]:
        """获取任务状态"""
        data = self.redis.get(f"task:{task_id}")
        if data:
            return Task.model_validate_json(data)
        return None
    
    def update_status(
        self,
        task_id: str,
        status: TaskStatus,
        **kwargs
    ):
        """更新任务状态"""
        task = self.get_task(task_id)
        if task:
            task.status = status
            for key, value in kwargs.items():
                setattr(task, key, value)
            self.redis.setex(
                f"task:{task_id}",
                3600 * 24,
                task.model_dump_json()
            )
    
    def update_progress(self, task_id: str, progress: float):
        """更新进度"""
        self.update_status(task_id, TaskStatus.RUNNING, progress=progress)
```

**使用示例**

```python
store = TaskStore()

# 创建任务
task_id = str(uuid.uuid4())
task = store.create_task(task_id, {
    "name": "Generate Report",
    "params": {"user_id": 123}
})

# 更新状态
store.update_status(task_id, TaskStatus.QUEUED)
store.update_status(task_id, TaskStatus.RUNNING, started_at=datetime.now())

# 更新进度
for i in range(0, 101, 10):
    store.update_progress(task_id, i)
    time.sleep(1)

# 完成
store.update_status(
    task_id,
    TaskStatus.COMPLETED,
    completed_at=datetime.now(),
    result={"file_url": "https://..."}
)

# 查询状态
task = store.get_task(task_id)
print(f"Task {task.id}: {task.status} - {task.progress}%")
```

---

## 12.2 FastAPI 特性

### 12.2.1 BackgroundTasks 基础

FastAPI 提供内置的 `BackgroundTasks` 用于简单的后台任务。

**基本使用**

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

def write_log(message: str):
    """后台任务：写日志"""
    time.sleep(2)  # 模拟耗时操作
    with open("log.txt", "a") as f:
        f.write(f"{datetime.now()}: {message}\n")

@app.post("/send-notification")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks
):
    """
    发送通知 - 后台处理
    """
    # 添加后台任务
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    
    # 立即返回，不等待日志写入
    return {"message": "Notification sent"}
```

**执行流程**：
```
1. 请求到达
2. 执行端点函数体
3. 返回响应给客户端 ← 用户立即收到响应
4. 执行后台任务 ← 在后台完成
5. 任务完成，释放资源
```

### 12.2.2 添加后台任务

**多个后台任务**

```python
@app.post("/process-order")
async def process_order(
    order_id: int,
    background_tasks: BackgroundTasks
):
    """处理订单 - 多个后台任务"""
    
    # 添加多个后台任务（按添加顺序执行）
    background_tasks.add_task(send_confirmation_email, order_id)
    background_tasks.add_task(update_inventory, order_id)
    background_tasks.add_task(notify_warehouse, order_id)
    background_tasks.add_task(log_order, order_id)
    
    return {"message": "Order processing started", "order_id": order_id}
```

**依赖注入中添加任务**

```python
from fastapi import Depends

async def log_request(
    request: Request,
    background_tasks: BackgroundTasks
):
    """依赖中添加后台任务"""
    background_tasks.add_task(
        write_log,
        f"{request.method} {request.url}"
    )

@app.get("/data", dependencies=[Depends(log_request)])
async def get_data():
    """这个端点会自动记录日志"""
    return {"data": "some data"}
```

### 12.2.3 任务参数传递

**位置参数和关键字参数**

```python
def complex_task(
    arg1: str,
    arg2: int,
    keyword_arg: str = "default",
    optional_arg: Optional[str] = None
):
    """复杂的后台任务"""
    print(f"Args: {arg1}, {arg2}, {keyword_arg}, {optional_arg}")

@app.post("/run-task")
async def run_task(background_tasks: BackgroundTasks):
    # 位置参数
    background_tasks.add_task(complex_task, "hello", 42)
    
    # 位置 + 关键字参数
    background_tasks.add_task(
        complex_task,
        "hello",
        42,
        keyword_arg="custom",
        optional_arg="value"
    )
    
    return {"status": "Task added"}
```

**传递复杂对象**

```python
from pydantic import BaseModel

class EmailData(BaseModel):
    to: str
    subject: str
    body: str
    attachments: list[str] = []

async def send_email(email_data: EmailData):
    """发送邮件"""
    # 使用 Pydantic 模型的数据
    await smtp.send(
        to=email_data.to,
        subject=email_data.subject,
        body=email_data.body
    )

@app.post("/send-email")
async def send_email_endpoint(
    email_data: EmailData,
    background_tasks: BackgroundTasks
):
    # 传递 Pydantic 模型
    background_tasks.add_task(send_email, email_data)
    return {"status": "Email queued"}
```

### 12.2.4 异步后台任务

**同步 vs 异步任务**

```python
import asyncio
import httpx

# ❌ 同步后台任务（阻塞）
def sync_task():
    time.sleep(5)  # 阻塞整个线程
    return "done"

# ✅ 异步后台任务（非阻塞）
async def async_task():
    await asyncio.sleep(5)  # 不阻塞，可以处理其他请求
    return "done"

# ✅ 异步 HTTP 请求
async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
        return response.json()

@app.post("/trigger-tasks")
async def trigger_tasks(background_tasks: BackgroundTasks):
    # 同步任务
    background_tasks.add_task(sync_task)
    
    # 异步任务（推荐）
    background_tasks.add_task(async_task)
    background_tasks.add_task(fetch_data)
    
    return {"status": "Tasks added"}
```

**性能对比**：

```python
# 场景：10 个并发请求，每个请求有一个 5 秒的后台任务

# 同步任务：
# - 阻塞线程池
# - 最多并发数 = 线程池大小（默认40）
# - 可能导致线程耗尽

# 异步任务：
# - 不阻塞
# - 可以处理数千个并发
# - 更高效的资源使用
```

### 12.2.5 任务错误处理

**基本错误处理**

```python
import logging

logger = logging.getLogger(__name__)

async def risky_task(data: dict):
    """可能失败的任务"""
    try:
        # 执行可能失败的操作
        result = await process_data(data)
        await save_result(result)
        logger.info(f"Task completed successfully: {data}")
    except Exception as e:
        # 记录错误
        logger.error(f"Task failed: {data}, Error: {e}")
        # 可以选择重试或通知管理员
        await notify_admin(f"Task failed: {e}")

@app.post("/process")
async def process(
    data: dict,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(risky_task, data)
    return {"status": "Processing started"}
```

**重试机制**

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
async def task_with_retry(data: dict):
    """自动重试的任务"""
    try:
        result = await external_api_call(data)
        return result
    except Exception as e:
        logger.warning(f"Task attempt failed, will retry: {e}")
        raise  # 重新抛出异常触发重试

@app.post("/process-with-retry")
async def process_with_retry(
    data: dict,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(task_with_retry, data)
    return {"status": "Processing started with retry"}
```

**错误回调**

```python
async def on_task_success(task_id: str, result: Any):
    """任务成功回调"""
    await db.update_task_status(task_id, "completed", result=result)
    await send_notification(f"Task {task_id} completed")

async def on_task_failure(task_id: str, error: Exception):
    """任务失败回调"""
    await db.update_task_status(task_id, "failed", error=str(error))
    await send_alert(f"Task {task_id} failed: {error}")

async def task_with_callbacks(task_id: str, data: dict):
    """带回调的任务"""
    try:
        result = await process(data)
        await on_task_success(task_id, result)
    except Exception as e:
        await on_task_failure(task_id, e)
        raise

@app.post("/process-with-callbacks")
async def process_with_callbacks(
    data: dict,
    background_tasks: BackgroundTasks
):
    task_id = str(uuid.uuid4())
    background_tasks.add_task(task_with_callbacks, task_id, data)
    return {"task_id": task_id, "status": "queued"}
```

---

## 12.3 AutoGPT中的后台任务与长时间运行实战分析

### 12.3.1 图执行后台任务

AutoGPT 使用 **RabbitMQ + 线程池** 处理长时间运行的图执行任务。

**任务执行流程**

```
┌──────────────────┐
│   API 请求        │
│  POST /graphs    │
│  /execute        │
└────────┬─────────┘
         │
         ▼
┌────────────────────────────────┐
│  add_graph_execution()         │
│  1. 验证 graph                  │
│  2. 创建执行记录（DB）          │
│  3. 发送消息到 RabbitMQ         │
│  4. 返回 execution_id          │
└────────┬───────────────────────┘
         │
         │ (立即返回给用户)
         │
         ▼
┌────────────────────────────────┐
│  RabbitMQ Queue                │
│  graph_execution_queue         │
└────────┬───────────────────────┘
         │
         │ (异步消费)
         │
         ▼
┌────────────────────────────────┐
│  ExecutionManager              │
│  - 线程池执行                   │
│  - 状态更新（Redis + DB）       │
│  - WebSocket 实时推送          │
└────────────────────────────────┘
```

**核心代码：add_graph_execution**

```python
# backend/executor/utils.py

async def add_graph_execution(
    graph_id: str,
    user_id: str,
    inputs: Optional[BlockInput] = None,
    graph_version: Optional[int] = None,
) -> GraphExecutionWithNodes:
    """
    添加图执行到队列
    
    流程：
    1. 验证 graph 和输入
    2. 创建数据库记录
    3. 发送到 RabbitMQ
    4. 返回执行对象
    """
    # 1. 验证输入
    graph, starting_nodes_input, compiled_nodes_input_masks = (
        await validate_and_construct_node_execution_input(
            graph_id=graph_id,
            user_id=user_id,
            graph_inputs=inputs or {},
            graph_version=graph_version,
        )
    )
    
    # 2. 创建数据库执行记录
    graph_exec = await edb.create_graph_execution(
        user_id=user_id,
        graph_id=graph_id,
        graph_version=graph.version,
        inputs=inputs or {},
        starting_nodes_input=starting_nodes_input,
    )
    
    # 3. 构造执行消息
    graph_exec_entry = graph_exec.to_graph_execution_entry(
        user_context=await get_user_context(user_id),
        compiled_nodes_input_masks=compiled_nodes_input_masks,
    )
    
    logger.info(
        f"Created graph execution #{graph_exec.id} for graph "
        f"#{graph_id} with {len(starting_nodes_input)} starting nodes"
    )
    
    # 4. 发送到 RabbitMQ 队列
    exec_queue = await get_async_execution_queue()
    await exec_queue.publish_message(
        routing_key=GRAPH_EXECUTION_ROUTING_KEY,
        message=graph_exec_entry.model_dump_json(),
        exchange=GRAPH_EXECUTION_EXCHANGE,
    )
    logger.info(f"Published execution {graph_exec.id} to RabbitMQ queue")
    
    # 5. 更新状态为 QUEUED
    graph_exec.status = ExecutionStatus.QUEUED
    await edb.update_graph_execution_stats(
        graph_exec_id=graph_exec.id,
        status=graph_exec.status,
    )
    
    # 6. 发送事件到 Redis Event Bus
    await get_async_execution_event_bus().publish(graph_exec)
    
    return graph_exec
```

**关键点**：
1. **快速返回**：创建记录 + 发送消息后立即返回
2. **状态管理**：INCOMPLETE → QUEUED → RUNNING → COMPLETED/FAILED
3. **双重通知**：RabbitMQ（任务队列）+ Redis（实时事件）

**API** **端点使用**

```python
# backend/server/routers/v1.py

@v1_router.post(
    "/graphs/{graph_id}/execute/{graph_version}",
    tags=["graphs"],
    dependencies=[Security(requires_user)],
)
async def execute_graph(
    graph_id: str,
    graph_version: int,
    node_input: Annotated[dict[str, Any], Body(..., embed=True, default_factory=dict)],
    user_id: Annotated[str, Security(get_user_id)],
):
    """
    执行 graph - 异步任务
    
    返回 execution_id，不等待执行完成
    """
    try:
        # 添加到执行队列
        graph_exec = await execution_utils.add_graph_execution(
            graph_id=graph_id,
            user_id=user_id,
            inputs=node_input,
            graph_version=graph_version,
        )
        
        # 立即返回 execution_id
        return {"id": graph_exec.id}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))
```

**用户流程**：
```
1. POST /graphs/{id}/execute → 收到 {id: "exec-123"}
2. 客户端使用 execution_id 查询状态或订阅 WebSocket
3. GET /graphs/executions/{exec-123} → 查看状态和结果
```

### 12.3.2 RabbitMQ 消息队列

AutoGPT 使用 RabbitMQ 管理任务分发。

**队列配置：**

```python
# backend/executor/utils.py

# 执行队列 - DIRECT Exchange
GRAPH_EXECUTION_EXCHANGE = Exchange(
    name="graph_execution",
    type=ExchangeType.DIRECT,      # 直接路由
    durable=True,                  # 持久化
    auto_delete=False,
)
GRAPH_EXECUTION_QUEUE_NAME = "graph_execution_queue"
GRAPH_EXECUTION_ROUTING_KEY = "graph_execution.run"

# 取消队列 - FANOUT Exchange
GRAPH_EXECUTION_CANCEL_EXCHANGE = Exchange(
    name="graph_execution_cancel",
    type=ExchangeType.FANOUT,      # 广播到所有队列
    durable=True,
    auto_delete=True,
)
GRAPH_EXECUTION_CANCEL_QUEUE_NAME = "graph_execution_cancel_queue"

def create_execution_queue_config() -> RabbitMQConfig:
    """创建执行队列配置"""
    run_queue = Queue(
        name=GRAPH_EXECUTION_QUEUE_NAME,
        exchange=GRAPH_EXECUTION_EXCHANGE,
        routing_key=GRAPH_EXECUTION_ROUTING_KEY,
        durable=True,
        auto_delete=False,
        arguments={
            # 消费者超时：24小时（允许长时间执行）
            # 问题：默认30分钟超时会杀死长任务
            # 解决：设置为1天，支持长时间 AI 训练等任务
            "x-consumer-timeout": 24 * 60 * 60 * 1000,  # 毫秒
        },
    )
    
    cancel_queue = Queue(
        name=GRAPH_EXECUTION_CANCEL_QUEUE_NAME,
        exchange=GRAPH_EXECUTION_CANCEL_EXCHANGE,
        routing_key="",  # FANOUT 不需要 routing key
        durable=True,
        auto_delete=False,
    )
    
    return RabbitMQConfig(
        vhost="/",
        exchanges=[GRAPH_EXECUTION_EXCHANGE, GRAPH_EXECUTION_CANCEL_EXCHANGE],
        queues=[run_queue, cancel_queue],
    )
```

**设计亮点**：

1. **DIRECT vs FANOUT**
   - 执行队列：DIRECT，精确路由到 worker
   - 取消队列：FANOUT，广播到所有 worker

2. **持久化**
   - `durable=True`：队列和消息在 RabbitMQ 重启后保留
   - 防止任务丢失

3. **长超时**
   - 24小时消费者超时
   - 支持长时间运行的 AI 任务

**RabbitMQ 连接管理：**

```python
# backend/data/rabbitmq.py

# 连接常量 - 针对生产环境优化
BLOCKED_CONNECTION_TIMEOUT = 300  # 5分钟：连接阻塞超时
SOCKET_TIMEOUT = 30              # 30秒：Socket 操作超时
CONNECTION_ATTEMPTS = 5          # 5次：重连尝试次数
RETRY_DELAY = 1                  # 1秒：重连延迟

class RabbitMQBase(ABC):
    """RabbitMQ 连接基类"""
    
    def __init__(self, config: RabbitMQConfig):
        settings = Settings()
        self.host = settings.config.rabbitmq_host
        self.port = settings.config.rabbitmq_port
        self.username = settings.secrets.rabbitmq_default_user
        self.password = settings.secrets.rabbitmq_default_pass
        self.config = config
        
        self._connection = None
        self._channel = None
    
    @property
    def is_connected(self) -> bool:
        """检查连接状态"""
        return bool(self._connection)
    
    @property
    def is_ready(self) -> bool:
        """检查通道状态"""
        return bool(self.is_connected and self._channel)
```

**消息发布（异步）：**

```python
# backend/data/rabbitmq.py 中的 AsyncRabbitMQ 类

class AsyncRabbitMQ(RabbitMQBase):
    """异步 RabbitMQ 客户端"""
    
    async def publish_message(
        self,
        routing_key: str,
        message: str,
        exchange: Exchange,
        properties: Optional[BasicProperties] = None,
    ):
        """发布消息到队列"""
        channel = await self.get_channel()
        
        await channel.basic_publish(
            exchange=exchange.name,
            routing_key=routing_key,
            body=message.encode(),
            properties=properties or BasicProperties(
                delivery_mode=2,  # 持久化消息
            ),
        )
```

**消息消费（同步 - worker）：**

```python
# backend/executor/manager.py

class ExecutionManager(AppProcess):
    """执行管理器 - RabbitMQ 消费者"""
    
    def run(self):
        """启动消费者"""
        # 创建 RabbitMQ 客户端
        self.run_client = SyncRabbitMQ(create_execution_queue_config())
        run_channel = self.run_client.get_channel()
        
        # 配置消费者
        run_channel.basic_consume(
            queue=GRAPH_EXECUTION_QUEUE_NAME,
            on_message_callback=self._handle_run_message,
            auto_ack=False,  # 手动确认，防止数据丢失
            consumer_tag="graph_execution_consumer",
        )
        
        logger.info("Started consuming graph execution messages")
        
        # 阻塞，持续消费
        run_channel.start_consuming()
    
    def _handle_run_message(
        self,
        channel: BlockingChannel,
        method: Basic.Deliver,
        properties: BasicProperties,
        body: bytes,
    ):
        """处理执行消息"""
        try:
            # 解析消息
            graph_exec_entry = GraphExecutionEntry.model_validate_json(body)
            
            logger.info(f"Received execution {graph_exec_entry.graph_exec_id}")
            
            # 提交到线程池执行
            future = self.executor.submit(
                execute_graph,
                graph_exec_entry,
                cancel_event=threading.Event(),
                cluster_lock=self.cluster_lock,
            )
            
            # 跟踪执行
            self._track_execution(graph_exec_entry.graph_exec_id, future)
            
            # 确认消息（执行完成后才确认）
            # 注意：如果进程崩溃，消息会重新入队
            def ack_message(fut):
                try:
                    fut.result()  # 等待完成
                finally:
                    channel.basic_ack(delivery_tag=method.delivery_tag)
            
            future.add_done_callback(ack_message)
            
        except Exception as e:
            logger.error(f"Failed to handle execution message: {e}")
            # 拒绝消息，重新入队
            channel.basic_nack(
                delivery_tag=method.delivery_tag,
                requeue=True
            )
```

**消息处理特点**：
1. **手动确认**：`auto_ack=False`，执行完成才确认
2. **线程池**：异步执行，不阻塞消费
3. **错误处理**：失败时重新入队

### 12.3.3 Redis 任务状态

AutoGPT 使用 Redis 管理实时状态和事件。

**Redis Event Bus：**

```python
# backend/data/event_bus.py

class AsyncRedisEventBus(BaseRedisEventBus[M], ABC):
    """基于 Redis Pub/Sub 的事件总线"""
    
    async def publish(self, event: M):
        """发布事件"""
        client = await redis.get_redis_async()
        await client.publish(
            self.channel_name,
            event.model_dump_json()
        )
    
    async def listen(self) -> AsyncGenerator[M, None]:
        """监听事件"""
        client = await redis.get_redis_async()
        pubsub = client.pubsub()
        
        await pubsub.subscribe(self.channel_name)
        
        async for message in pubsub.listen():
            if message["type"] == "message":
                yield self.Model.model_validate_json(message["data"])

# 执行事件总线
class AsyncRedisExecutionEventBus(AsyncRedisEventBus[ExecutionEvent]):
    Model = ExecutionEvent
    
    @property
    def channel_name(self) -> str:
        return "graph_execution_events"
```

**WebSocket 实时推送：**

```python
# backend/server/ws_api.py

@app.websocket("/ws")
async def websocket_router(
    websocket: WebSocket,
    manager: ConnectionManager = Depends(get_connection_manager)
):
    """WebSocket 连接 - 实时执行状态"""
    user_id = await authenticate_websocket(websocket)
    if not user_id:
        return
    
    await manager.connect_socket(websocket)
    
    try:
        while True:
            # 接收订阅请求
            message = await websocket.receive_json()
            ws_msg = WSMessage.model_validate(message)
            
            if ws_msg.method == WSMethod.SUBSCRIBE_GRAPH_EXEC:
                # 订阅特定执行
                graph_exec_id = ws_msg.data["graph_exec_id"]
                
                # 从 Redis 监听执行事件
                event_bus = get_async_execution_event_bus()
                async for event in event_bus.listen():
                    if event.execution_id == graph_exec_id:
                        # 推送给客户端
                        await websocket.send_json({
                            "method": "graph_execution_event",
                            "data": event.model_dump()
                        })
    
    except WebSocketDisconnect:
        await manager.disconnect_socket(websocket)
```

**实时更新流程**：
```
ExecutionManager 执行 Graph
         │
         ▼
更新状态到 DB + Redis
         │
         ▼
Redis Pub/Sub 发布事件
         │
         ▼
WebSocket 监听器接收
         │
         ▼
推送给订阅的客户端
```

### 12.3.4 任务监控和重试

**执行状态监控**

```python
# backend/data/execution.py

class ExecutionStatus(str, Enum):
    """执行状态枚举"""
    INCOMPLETE = "INCOMPLETE"    # 创建但未开始
    QUEUED = "QUEUED"            # 已加入队列
    RUNNING = "RUNNING"          # 执行中
    COMPLETED = "COMPLETED"      # 成功完成
    FAILED = "FAILED"            # 失败
    TERMINATED = "TERMINATED"    # 被终止

# 有效的状态转换
VALID_STATUS_TRANSITIONS = {
    ExecutionStatus.QUEUED: [ExecutionStatus.INCOMPLETE],
    ExecutionStatus.RUNNING: [ExecutionStatus.QUEUED],
    ExecutionStatus.COMPLETED: [ExecutionStatus.RUNNING],
    ExecutionStatus.FAILED: [ExecutionStatus.RUNNING],
    ExecutionStatus.TERMINATED: [ExecutionStatus.RUNNING, ExecutionStatus.QUEUED],
}
```

**Prometheus 监控指标**

```python
# backend/executor/manager.py

from prometheus_client import Gauge

# 定义监控指标
active_runs_gauge = Gauge(
    "execution_manager_active_runs",
    "Number of active graph runs"
)
pool_size_gauge = Gauge(
    "execution_manager_pool_size",
    "Maximum number of graph workers"
)
utilization_gauge = Gauge(
    "execution_manager_utilization_ratio",
    "Ratio of active graph runs to max graph workers"
)

class ExecutionManager(AppProcess):
    def _update_metrics(self):
        """更新 Prometheus 指标"""
        active_count = len([
            f for f in self._active_futures.values()
            if not f.done()
        ])
        
        active_runs_gauge.set(active_count)
        pool_size_gauge.set(self.pool_size)
        
        if self.pool_size > 0:
            utilization_gauge.set(active_count / self.pool_size)
```

**错误重试机制**

```python
# backend/util/retry.py

from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type
)

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type((ConnectionError, TimeoutError))
)
async def execute_with_retry(node: Node, data: NodeExecutionEntry):
    """节点执行 - 带重试"""
    try:
        result = await execute_node(node, data)
        return result
    except Exception as e:
        logger.warning(f"Node execution failed, will retry: {e}")
        raise  # 重新抛出触发重试
```

**取消执行**

```python
# backend/executor/utils.py

async def stop_graph_execution(
    user_id: str,
    graph_exec_id: str,
    wait_timeout: float = 15.0,
):
    """
    停止图执行
    
    机制：
    1. 发送取消事件到 FANOUT Exchange
    2. 所有 worker 接收到取消信号
    3. Worker 终止对应的执行
    4. 更新状态为 TERMINATED
    """
    # 1. 发送取消事件
    queue_client = await get_async_execution_queue()
    await queue_client.publish_message(
        routing_key="",  # FANOUT 不需要 routing key
        message=CancelExecutionEvent(
            graph_exec_id=graph_exec_id
        ).model_dump_json(),
        exchange=GRAPH_EXECUTION_CANCEL_EXCHANGE,
    )
    
    # 2. 等待执行终止
    if not wait_timeout:
        return
    
    start_time = time.time()
    while time.time() - start_time < wait_timeout:
        graph_exec = await db.get_graph_execution_meta(
            execution_id=graph_exec_id,
            user_id=user_id
        )
        
        if graph_exec.status in [
            ExecutionStatus.TERMINATED,
            ExecutionStatus.COMPLETED,
            ExecutionStatus.FAILED,
        ]:
            return
        
        await asyncio.sleep(0.5)
    
    raise TimeoutError("Graph execution cancellation timeout")
```

---

# 13  文档与测试

---

## 13.1 基础复习

### 13.1.1 API 文档重要性

**为什么文档很重要**？

```
 对开发者：
- 快速了解 API 功能
- 减少沟通成本
- 降低集成难度
- 提高开发效率

 对团队：
- 知识共享
- 降低维护成本
- 新成员快速上手
- 减少重复问题

 对用户：
- 自助集成
- 降低支持成本
- 提升用户体验
- 增加API采用率
```

**好文档 vs 坏文档**

```python
# ❌ 坏文档
@app.post("/users")
def create_user(data: dict):
    """Create user"""
    return {"ok": True}

# ✅ 好文档
@app.post("/users", response_model=UserResponse, status_code=201)
async def create_user(
    user: UserCreate,
    current_user: User = Depends(get_current_user)
):
    """
    Create a new user account.
    
    This endpoint creates a new user with the provided information.
    Requires authentication and admin privileges.
    
    Args:
        user: User creation data including username, email, and password
        current_user: Authenticated user (admin only)
    
    Returns:
        UserResponse: Created user information (without password)
    
    Raises:
        400: Invalid input data
        401: Not authenticated
        403: Insufficient permissions (admin required)
        409: User already exists (duplicate username/email)
    
    Example:
        ```json
        {
            "username": "johndoe",
            "email": "john@example.com",
            "password": "SecurePass123!"
        }
```
### 13.1.2 OpenAPI 规范

OpenAPI (前身 Swagger) 是 REST API 的标准规范。

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API for managing users and posts

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: http://localhost:8000
    description: Development server

paths:
  /users:
    get:
      summary: List users
      description: Retrieve a paginated list of users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - username
        - email
      properties:
        id:
          type: integer
          example: 1
        username:
          type: string
          example: johndoe
        email:
          type: string
          format: email
          example: john@example.com
    
    Pagination:
      type: object
      properties:
        total:
          type: integer
        page:
          type: integer
        limit:
          type: integer
  
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

**OpenAPI 核心概念**

```python
from pydantic import BaseModel, Field
from typing import Optional

class User(BaseModel):
    """
    OpenAPI 会自动从 Pydantic 模型生成 schema
    """
    id: int = Field(
        description="Unique user identifier",
        example=123
    )
    username: str = Field(
        min_length=3,
        max_length=50,
        description="User's unique username",
        example="johndoe"
    )
    email: str = Field(
        description="User's email address",
        example="john@example.com"
    )
    is_active: bool = Field(
        default=True,
        description="Whether the user account is active"
    )
    created_at: datetime = Field(
        description="Account creation timestamp"
    )
    
    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "id": 1,
                    "username": "johndoe",
                    "email": "john@example.com",
                    "is_active": True,
                    "created_at": "2024-01-01T12:00:00Z"
                }
            ]
        }
    }
```

### 13.1.3 测试金字塔

测试金字塔是测试策略的可视化。

```
        /\
       /  \      E2E Tests (少量)
      /----\     - 完整流程测试
     /      \    - 慢、脆弱、昂贵
    /--------\   
   / Integration \ Integration Tests (适量)
  /--------------\ - 组件协作测试
 /                \ - 中等速度
/------------------\ 
|   Unit Tests     | Unit Tests (大量)
|                  | - 单个函数测试
|                  | - 快速、稳定、廉价
--------------------

比例建议：
- 单元测试：70%
- 集成测试：20%
- E2E测试：10%
```

**各层测试示例**

```python
# ===== 单元测试 =====
def calculate_discount(price: float, discount_percent: float) -> float:
    """计算折扣后价格"""
    return price * (1 - discount_percent / 100)

def test_calculate_discount():
    """单元测试：测试单个函数"""
    assert calculate_discount(100, 10) == 90.0
    assert calculate_discount(100, 0) == 100.0
    assert calculate_discount(100, 100) == 0.0

# ===== 集成测试 =====
class OrderService:
    def __init__(self, db, payment_gateway):
        self.db = db
        self.payment_gateway = payment_gateway
    
    async def create_order(self, user_id: int, items: list) -> Order:
        """创建订单（涉及多个组件）"""
        total = sum(item.price for item in items)
        order = await self.db.create_order(user_id, total)
        await self.payment_gateway.charge(user_id, total)
        return order

async def test_create_order_integration():
    """集成测试：测试服务和数据库、支付网关的协作"""
    db = TestDatabase()
    payment = MockPaymentGateway()
    service = OrderService(db, payment)
    
    order = await service.create_order(
        user_id=1,
        items=[Item(price=10), Item(price=20)]
    )
    
    assert order.total == 30
    assert payment.charged_amount == 30

# ===== E2E 测试 =====
async def test_complete_purchase_flow():
    """E2E测试：完整的购买流程"""
    client = TestClient(app)
    
    # 1. 注册用户
    response = client.post("/auth/register", json={
        "username": "testuser",
        "password": "Test123!"
    })
    assert response.status_code == 201
    
    # 2. 登录
    response = client.post("/auth/login", json={
        "username": "testuser",
        "password": "Test123!"
    })
    token = response.json()["access_token"]
    
    # 3. 添加商品到购物车
    response = client.post(
        "/cart/items",
        headers={"Authorization": f"Bearer {token}"},
        json={"product_id": 1, "quantity": 2}
    )
    assert response.status_code == 200
    
    # 4. 结账
    response = client.post(
        "/orders/checkout",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 201
    assert "order_id" in response.json()
```

### 13.1.4 单元测试 vs 集成测试

**对比表**

```
特性          单元测试              集成测试
────────────────────────────────────────────
范围          单个函数/类           多个组件协作
依赖          Mock外部依赖          真实依赖或测试替身
速度          快（毫秒级）          慢（秒级）
稳定性        高                    中等
维护成本      低                    中等
覆盖率        代码逻辑              组件交互
隔离性        完全隔离              部分隔离
运行环境      任何环境              需要测试环境
```

**何时使用**？

```python
# 适合单元测试：
# - 纯函数
# - 业务逻辑
# - 数据转换
# - 验证规则

def validate_email(email: str) -> bool:
    """纯函数 - 适合单元测试"""
    import re
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    return bool(re.match(pattern, email))

def test_validate_email():
    assert validate_email("test@example.com") == True
    assert validate_email("invalid-email") == False

# 适合集成测试：
# - API端点
# - 数据库操作
# - 外部服务调用
# - 工作流

@app.post("/users")
async def create_user(user: UserCreate, db: Session = Depends(get_db)):
    """API端点 - 适合集成测试"""
    db_user = User(**user.dict())
    db.add(db_user)
    db.commit()
    return db_user

def test_create_user_endpoint(client: TestClient, test_db):
    """集成测试：测试完整的请求-响应流程"""
    response = client.post("/users", json={
        "username": "testuser",
        "email": "test@example.com"
    })
    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
```

---

## 13.2 FastAPI 特性

### 13.2.1 自动生成 OpenAPI

FastAPI 自动从代码生成 OpenAPI 文档。

**基本示例**

```python
from fastapi import FastAPI, Query, Path, Body
from pydantic import BaseModel, Field
from typing import Optional

app = FastAPI(
    title="My API",
    description="A simple API for demonstration",
    version="1.0.0",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "API Support",
        "url": "http://example.com/support",
        "email": "support@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
)

class UserCreate(BaseModel):
    """用户创建模型"""
    username: str = Field(
        min_length=3,
        max_length=50,
        description="Unique username",
        example="johndoe"
    )
    email: str = Field(
        description="User email address",
        example="john@example.com"
    )
    password: str = Field(
        min_length=8,
        description="User password (min 8 characters)",
        example="SecurePass123!"
    )

class UserResponse(BaseModel):
    """用户响应模型"""
    id: int = Field(description="User ID", example=1)
    username: str = Field(description="Username", example="johndoe")
    email: str = Field(description="Email", example="john@example.com")
    is_active: bool = Field(description="Account status", example=True)

@app.post(
    "/users",
    response_model=UserResponse,
    status_code=201,
    tags=["users"],
    summary="Create a new user",
    description="Create a new user account with the provided credentials.",
    response_description="Successfully created user",
)
async def create_user(
    user: UserCreate = Body(
        ...,
        example={
            "username": "johndoe",
            "email": "john@example.com",
            "password": "SecurePass123!"
        }
    )
):
    """
    Create a new user account.
    
    This endpoint will:
    - Validate the input data
    - Check for duplicate username/email
    - Hash the password
    - Create the user in the database
    - Return the created user (without password)
    """
    # Implementation...
    return UserResponse(
        id=1,
        username=user.username,
        email=user.email,
        is_active=True
    )

@app.get(
    "/users/{user_id}",
    response_model=UserResponse,
    tags=["users"],
    summary="Get user by ID",
)
async def get_user(
    user_id: int = Path(
        ...,
        description="The ID of the user to retrieve",
        example=1,
        ge=1
    )
):
    """Get a user by their ID."""
    # Implementation...
    return UserResponse(
        id=user_id,
        username="johndoe",
        email="john@example.com",
        is_active=True
    )

@app.get(
    "/users",
    response_model=list[UserResponse],
    tags=["users"],
)
async def list_users(
    page: int = Query(
        1,
        description="Page number",
        ge=1,
        example=1
    ),
    limit: int = Query(
        20,
        description="Items per page",
        ge=1,
        le=100,
        example=20
    ),
    search: Optional[str] = Query(
        None,
        description="Search query for username/email",
        min_length=1,
        example="john"
    )
):
    """
    List users with pagination and optional search.
    
    - **page**: Page number (starts from 1)
    - **limit**: Number of items per page (1-100)
    - **search**: Optional search query
    """
    # Implementation...
    return []
```

**访问文档**：
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
- OpenAPI JSON: `http://localhost:8000/openapi.json`

### 13.2.2 Swagger UI 配置

自定义 Swagger UI 的外观和行为。

```python
from fastapi import FastAPI
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.openapi.utils import get_openapi

app = FastAPI(
    docs_url=None,  # 禁用默认文档
    redoc_url=None,
)

@app.get("/docs", include_in_schema=False)
async def custom_swagger_ui_html():
    """自定义 Swagger UI"""
    return get_swagger_ui_html(
        openapi_url="/openapi.json",
        title="My API - Documentation",
        oauth2_redirect_url="/docs/oauth2-redirect",
        swagger_js_url="https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui-bundle.js",
        swagger_css_url="https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui.css",
        swagger_favicon_url="https://example.com/favicon.ico",
    )

# 自定义 OpenAPI schema
def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title="My Custom API",
        version="2.0.0",
        description="This is a custom OpenAPI schema",
        routes=app.routes,
    )
    
    # 添加自定义内容
    openapi_schema["info"]["x-logo"] = {
        "url": "https://example.com/logo.png"
    }
    
    # 添加服务器信息
    openapi_schema["servers"] = [
        {
            "url": "https://api.example.com",
            "description": "Production server"
        },
        {
            "url": "http://localhost:8000",
            "description": "Development server"
        }
    ]
    
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

**隐藏端点**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/public")
async def public_endpoint():
    """这个端点会出现在文档中"""
    return {"message": "Public"}

@app.get("/internal", include_in_schema=False)
async def internal_endpoint():
    """
    这个端点不会出现在文档中
    用于内部 API 或管理端点
    """
    return {"message": "Internal"}
```

### 13.2.3 ReDoc 文档

ReDoc 提供了另一种文档视图。

```python
from fastapi import FastAPI
from fastapi.openapi.docs import get_redoc_html

app = FastAPI(redoc_url=None)

@app.get("/redoc", include_in_schema=False)
async def redoc_html():
    """自定义 ReDoc 文档"""
    return get_redoc_html(
        openapi_url="/openapi.json",
        title="My API - ReDoc",
        redoc_js_url="https://cdn.jsdelivr.net/npm/redoc@next/bundles/redoc.standalone.js",
        redoc_favicon_url="https://example.com/favicon.ico",
    )
```

### 13.2.4 文档定制

**添加示例**

```python
from pydantic import BaseModel, Field
from typing import List

class Item(BaseModel):
    name: str = Field(example="Widget")
    price: float = Field(example=29.99)
    tags: List[str] = Field(example=["electronics", "gadgets"])

@app.post("/items")
async def create_item(item: Item):
    """
    Create an item with example data.
    
    The example will appear in the Swagger UI.
    """
    return item
```

**多个示例**

```python
from fastapi import Body

@app.post("/items")
async def create_item(
    item: Item = Body(
        ...,
        examples={
            "normal": {
                "summary": "A normal item",
                "description": "A regular item with typical values",
                "value": {
                    "name": "Widget",
                    "price": 29.99,
                    "tags": ["electronics"]
                }
            },
            "premium": {
                "summary": "A premium item",
                "description": "A high-end premium item",
                "value": {
                    "name": "Premium Widget",
                    "price": 99.99,
                    "tags": ["electronics", "premium"]
                }
            },
            "minimal": {
                "summary": "Minimal data",
                "description": "Minimum required fields",
                "value": {
                    "name": "Basic Widget",
                    "price": 9.99
                }
            }
        }
    )
):
    """Create an item with multiple examples."""
    return item
```

**响应示例**

```python
from fastapi import FastAPI, Response
from fastapi.responses import JSONResponse

@app.get(
    "/items/{item_id}",
    response_model=Item,
    responses={
        200: {
            "description": "Successful Response",
            "content": {
                "application/json": {
                    "example": {
                        "name": "Widget",
                        "price": 29.99,
                        "tags": ["electronics"]
                    }
                }
            }
        },
        404: {
            "description": "Item not found",
            "content": {
                "application/json": {
                    "example": {"detail": "Item not found"}
                }
            }
        },
        500: {
            "description": "Internal server error",
            "content": {
                "application/json": {
                    "example": {"detail": "Internal server error"}
                }
            }
        }
    }
)
async def get_item(item_id: int):
    """Get an item by ID with documented response examples."""
    # Implementation...
    return Item(name="Widget", price=29.99, tags=["electronics"])
```

### 13.2.5 TestClient 使用

FastAPI 提供 `TestClient` 用于测试。

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/")
async def read_main():
    return {"message": "Hello World"}

@app.post("/users")
async def create_user(username: str, email: str):
    return {"username": username, "email": email}

# 测试代码
client = TestClient(app)

def test_read_main():
    """测试主端点"""
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_create_user():
    """测试创建用户"""
    response = client.post(
        "/users",
        params={"username": "testuser", "email": "test@example.com"}
    )
    assert response.status_code == 200
    assert response.json()["username"] == "testuser"

def test_create_user_with_json():
    """测试 JSON 请求"""
    response = client.post(
        "/users",
        json={"username": "testuser", "email": "test@example.com"}
    )
    assert response.status_code == 200

def test_authentication():
    """测试认证"""
    # 无认证
    response = client.get("/protected")
    assert response.status_code == 401
    
    # 带认证
    response = client.get(
        "/protected",
        headers={"Authorization": "Bearer token123"}
    )
    assert response.status_code == 200
```

### 13.2.6 依赖覆盖（Mock）

使用依赖覆盖来 mock 依赖项。

```python
from fastapi import FastAPI, Depends
from fastapi.testclient import TestClient

app = FastAPI()

# 真实依赖
def get_database():
    """真实数据库连接"""
    db = Database(host="production.db")
    try:
        yield db
    finally:
        db.close()

@app.get("/users")
async def get_users(db = Depends(get_database)):
    """获取用户列表"""
    return db.query_users()

# ===== 测试 =====
class MockDatabase:
    """Mock 数据库"""
    def query_users(self):
        return [{"id": 1, "name": "Test User"}]
    
    def close(self):
        pass

def get_mock_database():
    """Mock 依赖"""
    db = MockDatabase()
    try:
        yield db
    finally:
        db.close()

def test_get_users():
    """测试用户列表端点"""
    # 覆盖依赖
    app.dependency_overrides[get_database] = get_mock_database
    
    client = TestClient(app)
    response = client.get("/users")
    
    assert response.status_code == 200
    assert len(response.json()) == 1
    assert response.json()[0]["name"] == "Test User"
    
    # 清理覆盖
    app.dependency_overrides = {}
```

**复杂依赖覆盖示例**

```python
from typing import Optional

# 原始依赖
async def get_current_user(token: str = Header(...)):
    """从 token 获取当前用户"""
    user = await decode_token(token)
    if not user:
        raise HTTPException(status_code=401)
    return user

async def get_db():
    """数据库连接"""
    async with DatabaseSession() as db:
        yield db

@app.get("/me")
async def get_me(
    current_user: User = Depends(get_current_user),
    db: Database = Depends(get_db)
):
    """获取当前用户信息"""
    return await db.get_user(current_user.id)

# ===== 测试 =====
def test_get_me():
    """测试获取当前用户"""
    # Mock 用户
    async def mock_get_current_user():
        return User(id=1, username="testuser")
    
    # Mock 数据库
    class MockDB:
        async def get_user(self, user_id: int):
            return {"id": user_id, "username": "testuser"}
    
    async def mock_get_db():
        yield MockDB()
    
    # 覆盖依赖
    app.dependency_overrides[get_current_user] = mock_get_current_user
    app.dependency_overrides[get_db] = mock_get_db
    
    client = TestClient(app)
    response = client.get("/me")
    
    assert response.status_code == 200
    assert response.json()["username"] == "testuser"
    
    # 清理
    app.dependency_overrides = {}
```

---

## 13.3 AutoGPT中的 OpenAPI 文档配置 实战分析

### 13.3.1 AutoGPT API 文档

AutoGPT 使用定制的 OpenAPI 文档配置。

**FastAPI** **应用配置**

```python
# backend/server/rest_api.py

def custom_generate_unique_id(route: APIRoute):
    """
    生成清晰的 operation ID 格式：{method}{tag}{summary}
    
    例如：getV1ListUsers, postV2CreateAgent
    """
    if not route.tags or not route.methods:
        return f"{route.name}"
    
    method = list(route.methods)[0].lower()
    first_tag = route.tags[0]
    
    # 标签格式化：v1 -> V1, user_profile -> UserProfile
    tag = "".join(word.capitalize() for word in str(first_tag).split("_"))
    
    # 摘要格式化
    summary = route.summary if route.summary else route.name
    summary = "".join(word.capitalize() for word in str(summary).split("_"))
    
    return f"{method}{tag}{summary}"

# 根据环境显示文档
docs_url = (
    "/docs"  # 本地环境显示
    if settings.config.app_env == AppEnvironment.LOCAL
    else None  # 生产环境隐藏
)

app = fastapi.FastAPI(
    title="AutoGPT Agent Server",
    description="This server is used to execute agents created by the AutoGPT system.",
    summary="AutoGPT Agent Server",
    version="0.1",
    lifespan=lifespan_context,
    docs_url=docs_url,
    generate_unique_id_function=custom_generate_unique_id,
)

# 添加认证响应到 OpenAPI
add_auth_responses_to_openapi(app)

# 添加 Prometheus 监控端点
instrument_fastapi(
    app,
    service_name="rest-api",
    expose_endpoint=True,
    endpoint="/metrics",
    include_in_schema=(
        settings.config.app_env == AppEnvironment.LOCAL
    ),  # 只在本地环境显示
)
```

**设计亮点**：

1. **环境感知**：生产环境隐藏文档端点
2. **自定义 operation ID**：更清晰的 API 标识
3. **自动认证文档**：添加 401 响应说明
4. **监控集成**：Prometheus 端点仅在本地显示

### 13.3.2 ws_api_test.py 测试

AutoGPT 的 WebSocket 测试展示了完整的测试模式。

**WebSocket 测试示例**

```python
# backend/server/ws_api_test.py

@pytest.fixture
def mock_websocket() -> AsyncMock:
    """Mock WebSocket fixture"""
    mock = AsyncMock(spec=WebSocket)
    mock.query_params = {}  # 添加认证需要的属性
    return mock

@pytest.fixture
def mock_manager() -> AsyncMock:
    """Mock ConnectionManager fixture"""
    return AsyncMock(spec=ConnectionManager)

@pytest.mark.asyncio
async def test_websocket_router_subscribe(
    mock_websocket: AsyncMock,
    mock_manager: AsyncMock,
    snapshot: Snapshot,
    mocker
) -> None:
    """测试 WebSocket 订阅流程"""
    
    # 1. Mock 认证
    mocker.patch(
        "backend.server.ws_api.authenticate_websocket",
        return_value=DEFAULT_USER_ID
    )
    
    # 2. 模拟接收消息
    mock_websocket.receive_text.side_effect = [
        WSMessage(
            method=WSMethod.SUBSCRIBE_GRAPH_EXEC,
            data={"graph_exec_id": "test-graph-exec-1"},
        ).model_dump_json(),
        WebSocketDisconnect(),  # 模拟断开连接
    ]
    
    # 3. 设置 manager 返回值
    mock_manager.subscribe_graph_exec.return_value = (
        f"{DEFAULT_USER_ID}|graph_exec#test-graph-exec-1"
    )
    
    # 4. 执行 WebSocket 路由
    await websocket_router(
        cast(WebSocket, mock_websocket),
        cast(ConnectionManager, mock_manager)
    )
    
    # 5. 断言调用
    mock_manager.connect_socket.assert_called_once_with(mock_websocket)
    mock_manager.subscribe_graph_exec.assert_called_once_with(
        user_id=DEFAULT_USER_ID,
        graph_exec_id="test-graph-exec-1",
        websocket=mock_websocket,
    )
    
    # 6. 验证响应消息
    mock_websocket.send_text.assert_called_once()
    assert '"method":"subscribe_graph_execution"' in mock_websocket.send_text.call_args[0][0]
    assert '"success":true' in mock_websocket.send_text.call_args[0][0]
    
    # 7. Snapshot 测试
    sent_message = mock_websocket.send_text.call_args[0][0]
    parsed_message = json.loads(sent_message)
    snapshot.snapshot_dir = "snapshots"
    snapshot.assert_match(
        json.dumps(parsed_message, indent=2, sort_keys=True),
        "sub"
    )
    
    # 8. 验证断开连接
    mock_manager.disconnect_socket.assert_called_once_with(mock_websocket)
```

**测试要点**：

1. **完整的 Mock**：WebSocket 和 ConnectionManager
2. **模拟消息序列**：多个消息 + 断开连接
3. **详细断言**：验证每个方法调用
4. **Snapshot 测试**：验证响应格式
5. **异步支持**：`@pytest.mark.asyncio`

### 13.3.3 数据库测试隔离

AutoGPT 使用 fixtures 进行数据库清理。

**全局测试 fixture**

```python
# backend/conftest.py

@pytest.fixture(scope="session")
async def server():
    """启动测试服务器"""
    from backend.util.test import SpinTestServer
    
    async with SpinTestServer() as server:
        yield server

@pytest.fixture(scope="session", autouse=True)
async def graph_cleanup(server):
    """自动清理测试创建的 graph"""
    created_graph_ids = []
    original_create_graph = server.agent_server.test_create_graph
    
    # 包装创建方法以记录创建的 graph
    async def create_graph_wrapper(*args, **kwargs):
        created_graph = await original_create_graph(*args, **kwargs)
        user_id = kwargs.get("user_id", args[2] if len(args) > 2 else None)
        created_graph_ids.append((created_graph.id, user_id))
        return created_graph
    
    try:
        # 替换创建方法
        server.agent_server.test_create_graph = create_graph_wrapper
        yield  # 执行测试
    finally:
        # 恢复原方法
        server.agent_server.test_create_graph = original_create_graph
        
        # 删除所有创建的 graph
        for graph_id, user_id in created_graph_ids:
            if user_id:
                resp = await server.agent_server.test_delete_graph(
                    graph_id, user_id
                )
                num_deleted = resp["version_counts"]
                assert num_deleted > 0, f"Graph {graph_id} was not deleted."
```

**测试服务器**

```python
# backend/util/test.py
 
class SpinTestServer:
    """测试服务器 - 集成所有服务"""
    
    def __init__(self):
        self.db_api = DatabaseManager()
        self.exec_manager = ExecutionManager()
        self.agent_server = AgentServer()
        self.scheduler = Scheduler(register_system_tasks=False)
        self.notif_manager = NotificationManager()
    
    async def __aenter__(self):
        """启动所有服务"""
        self.setup_dependency_overrides()
        
        # 启动各服务
        self.db_api.__enter__()
        self.agent_server.__enter__()
        self.exec_manager.__enter__()
        self.scheduler.__enter__()
        self.notif_manager.__enter__()
        
        # 连接数据库和初始化
        await db.connect()
        await initialize_blocks()
        await create_default_user()
        
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """关闭所有服务"""
        await db.disconnect()
        
        self.scheduler.__exit__(exc_type, exc_val, exc_tb)
        self.exec_manager.__exit__(exc_type, exc_val, exc_tb)
        self.agent_server.__exit__(exc_type, exc_val, exc_tb)
        self.db_api.__exit__(exc_type, exc_val, exc_tb)
        self.notif_manager.__exit__(exc_type, exc_val, exc_tb)
    
    def setup_dependency_overrides(self):
        """覆盖测试依赖"""
        self.agent_server.set_test_dependency_overrides(
            {get_user_id: self.test_get_user_id}
        )
    
    @staticmethod
    def test_get_user_id():
        """测试用户 ID"""
        return "3e53486c-cf57-477e-ba2a-cb02dc828e1a"
```

### 13.3.4 测试最佳实践

**Fixture 复用**

```python
# backend/server/conftest.py

@pytest.fixture
def configured_snapshot(snapshot: Snapshot) -> Snapshot:
    """预配置的 snapshot fixture"""
    snapshot.snapshot_dir = "snapshots"
    return snapshot

@pytest.fixture
def test_user_id() -> str:
    """测试用户 ID"""
    return "test-user-id"

@pytest.fixture
def mock_jwt_user(test_user_id):
    """Mock JWT 用户"""
    def override_get_jwt_payload(request: fastapi.Request) -> dict[str, str]:
        return {
            "sub": test_user_id,
            "role": "user",
            "email": "test@example.com"
        }
    
    return {
        "get_jwt_payload": override_get_jwt_payload,
        "user_id": test_user_id
    }
```

**依赖覆盖模式**

```python
# backend/server/routers/v1_test.py

@pytest.fixture(autouse=True)
def setup_app_auth(mock_jwt_user):
    """为所有测试设置认证覆盖"""
    from autogpt_libs.auth.jwt_utils import get_jwt_payload
    
    app.dependency_overrides[get_jwt_payload] = mock_jwt_user["get_jwt_payload"]
    yield
    app.dependency_overrides.clear()  # 清理

def test_get_user_route(
    mocker: pytest_mock.MockFixture,
    configured_snapshot: Snapshot,
    test_user_id: str,
):
    """测试获取用户端点"""
    # Mock 数据库调用
    mock_user = Mock()
    mock_user.model_dump.return_value = {
        "id": test_user_id,
        "email": "test@example.com",
        "name": "Test User",
    }
    
    mocker.patch(
        "backend.server.routers.v1.get_or_create_user",
        return_value=mock_user,
    )
    
    # 执行请求
    response = client.post("/auth/user")
    
    # 断言
    assert response.status_code == 200
    response_data = response.json()
    
    # Snapshot 验证
    configured_snapshot.assert_match(
        json.dumps(response_data, indent=2, sort_keys=True),
        "auth_user",
    )
```

**Snapshot 测试**

```python
def test_get_blocks(
    mocker: pytest_mock.MockFixture,
    snapshot: Snapshot,
):
    """测试获取 blocks"""
    # Mock block
    mock_block = Mock()
    mock_block.to_dict.return_value = {
        "id": "test-block",
        "name": "Test Block",
        "description": "A test block",
    }
    
    mocker.patch(
        "backend.server.routers.v1.get_blocks",
        return_value={"test-block": lambda: mock_block},
    )
    
    response = client.get("/blocks")
    
    assert response.status_code == 200
    response_data = response.json()
    
    # 使用 snapshot 验证响应结构
    snapshot.snapshot_dir = "snapshots"
    snapshot.assert_match(
        json.dumps(response_data, indent=2, sort_keys=True),
        "blks_all",
    )
```

**Snapshot 文件** (`snapshots/blks_all`):
```json
[
  {
    "description": "A test block",
    "id": "test-block",
    "name": "Test Block"
  }
]
```

**参数化测试**

```python
@pytest.mark.parametrize(
    "status_code,expected_result",
    [
        (200, "success"),
        (400, "bad_request"),
        (404, "not_found"),
        (500, "server_error"),
    ]
)
def test_error_responses(status_code, expected_result):
    """参数化测试不同的错误响应"""
    with patch("external_api.call") as mock_call:
        mock_call.return_value = MockResponse(status_code)
        
        result = handle_api_call()
        assert result == expected_result
```

**异步测试工具**

```python
async def wait_execution(
    user_id: str,
    graph_exec_id: str,
    timeout: int = 30,
) -> Sequence[NodeExecutionResult]:
    """等待执行完成的辅助函数"""
    async def is_execution_completed():
        status = await AgentServer().test_get_graph_run_status(
            graph_exec_id, user_id
        )
        if status == ExecutionStatus.FAILED:
            raise Exception("Execution failed")
        if status == ExecutionStatus.TERMINATED:
            raise Exception("Execution terminated")
        return status == ExecutionStatus.COMPLETED
    
    # 轮询等待
    for i in range(timeout):
        if await is_execution_completed():
            graph_exec = await get_graph_execution(
                user_id=user_id,
                execution_id=graph_exec_id,
                include_node_executions=True,
            )
            assert graph_exec, f"Graph execution #{graph_exec_id} not found"
            return graph_exec.node_executions
        await asyncio.sleep(1)
    
    assert False, "Execution did not complete in time."

# 使用
@pytest.mark.asyncio
async def test_graph_execution(server):
    """测试完整的 graph 执行流程"""
    # 创建 graph
    graph = await server.agent_server.test_create_graph(...)
    
    # 执行 graph
    exec_id = await server.agent_server.test_execute_graph(graph.id)
    
    # 等待完成
    results = await wait_execution(user_id, exec_id)
    
    # 验证结果
    assert len(results) > 0
    assert all(r.status == ExecutionStatus.COMPLETED for r in results)
```

---

## 13.4 测试最佳实践指南

```
1. 测试结构
   - 遵循 AAA 模式：Arrange, Act, Assert
   - 一个测试一个断言（理想情况）
   - 使用描述性的测试名称

2. Fixture 使用
   - 共享的 fixtures 放在 conftest.py
   - 使用 scope 控制生命周期
   - 清理资源使用 yield fixture

3. Mock 策略
   - Mock 外部依赖（数据库、API、文件系统）
   - 不要过度 mock（测试应该有意义）
   - 使用 dependency override 替代直接 mock

4. 测试隔离
   - 每个测试应该独立运行
   - 使用事务回滚保证数据库隔离
   - 避免测试之间的依赖

5. 性能
   - 单元测试应该快速（< 100ms）
   - 慢测试使用 @pytest.mark.slow 标记
   - 并行运行测试：pytest -n auto

6. 文档
   - 使用 docstring 说明测试目的
   - 复杂测试添加注释
   - Snapshot 测试用于验证输出格式

7. 覆盖率
   - 目标：80%+ 代码覆盖率
   - 关键路径必须测试
   - 不要为了覆盖率而写无意义的测试

```



# 阶段四：AutoGPT 独家专题
## 专题A：RPC 模式深度剖析

---

## 核心内容 1：RPC 框架架构概览

### 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                   AutoGPT RPC 框架                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────┐         HTTP          ┌──────────────┐ │
│  │                │◄────────────────────►  │              │ │
│  │  RPC Client    │     POST /method      │ RPC Server   │ │
│  │ (DynamicClient)│     JSON Payload      │ (AppService) │ │
│  │                │                        │              │ │
│  └────────────────┘                        └──────────────┘ │
│         │                                          │         │
│         │ __getattr__                              │ @expose │
│         │ 动态方法调用                              │ 方法注册 │
│         ▼                                          ▼         │
│  ┌────────────────┐                        ┌──────────────┐ │
│  │   httpx.Client │                        │   FastAPI    │ │
│  │   同步/异步     │                        │   路由生成   │ │
│  └────────────────┘                        └──────────────┘ │
│         │                                          │         │
│         │                                          │         │
│         └──────────► 异常映射 ◄────────────────────┘         │
│              RemoteCallError + EXCEPTION_MAPPING            │
│                                                              │
└─────────────────────────────────────────────────────────────┘

关键特性：
✓ 透明的远程调用（像调用本地方法）
✓ 自动类型转换和验证（Pydantic）
✓ 异常传递和重建
✓ 连接池和重试机制
✓ Prometheus 监控集成
```

---

## 核心内容 2：@expose 装饰器机制

### 装饰器实现

```python
# 第 79-85 行
P = ParamSpec("P")
R = TypeVar("R")
EXPOSED_FLAG = "__exposed__"  # 标记属性

def expose(func: C) -> C:
    """
    将方法标记为可远程调用的 RPC 端点
    
    工作原理：
    1. 获取原始函数（如果是 classmethod/staticmethod）
    2. 设置 __exposed__ 属性为 True
    3. 返回原函数（不修改行为）
    """
    func = getattr(func, "__func__", func)  # 解包装饰器
    setattr(func, EXPOSED_FLAG, True)       # 设置标记
    return func
```

### 使用示例

```python
class MyService(AppService):
    @classmethod
    def get_port(cls) -> int:
        return 8001
    
    @expose  # ← 标记为 RPC 端点
    def add_numbers(self, a: int, b: int) -> int:
        """这个方法会自动暴露为 POST /add_numbers"""
        return a + b
    
    @expose
    async def fetch_data(self, user_id: str) -> dict:
        """异步方法也支持"""
        data = await database.get_user(user_id)
        return data.model_dump()
    
    def _internal_method(self):
        """没有 @expose，不会暴露为 RPC 端点"""
        pass
```

**关键点**：
- 只需要添加 `@expose` 装饰器
- 支持同步和异步方法
- 自动处理参数和返回值
- 内部方法不暴露

---

## 核心内容 3：方法转 HTTP 端点

### 自动路由注册

```python
# 第 305-316 行：服务启动时注册路由
def run(self):
    self.fastapi_app = FastAPI()
    
    # 遍历类的所有属性
    for attr_name, attr in vars(type(self)).items():
        # 检查是否有 __exposed__ 标记
        if getattr(attr, EXPOSED_FLAG, False):
            route_path = f"/{attr_name}"  # 方法名 = 路由路径
            
            # 创建 FastAPI 端点并注册
            self.fastapi_app.add_api_route(
                route_path,
                self._create_fastapi_endpoint(attr),  # 动态生成端点
                methods=["POST"],  # 所有 RPC 都是 POST
            )
    
    # 添加健康检查端点
    self.fastapi_app.add_api_route(
        "/health_check", self.health_check, methods=["POST", "GET"]
    )
```

### 端点生成魔法

```python
# 第 208-257 行：_create_fastapi_endpoint
def _create_fastapi_endpoint(self, func: Callable) -> Callable:
    """
    核心魔法：将任意 Python 方法转换为 FastAPI 端点
    
    步骤：
    1. 使用 inspect 提取函数签名
    2. 动态创建 Pydantic 请求模型
    3. 生成 FastAPI 兼容的端点函数
    4. 处理同步/异步差异
    """
    sig = inspect.signature(func)
    fields = {}
    
    is_bound_method = False
    for name, param in sig.parameters.items():
        # 跳过 self/cls 参数
        if name in ("self", "cls"):
            is_bound_method = True
            continue
        
        # 提取类型注解（默认为 str）
        annotation = (
            param.annotation 
            if param.annotation != inspect.Parameter.empty 
            else str
        )
        
        # 提取默认值（无默认值则为必填）
        default = (
            param.default 
            if param.default != inspect.Parameter.empty 
            else ...  # ... 表示必填
        )
        
        fields[name] = (annotation, default)
    
    # 动态创建 Pydantic 模型
    RequestBodyModel = create_model("RequestBodyModel", **fields)
    
    # 绑定方法（如果需要）
    f = func.__get__(self) if is_bound_method else func
    
    # 根据函数类型创建端点
    if asyncio.iscoroutinefunction(f):
        # 异步端点
        async def async_endpoint(body: RequestBodyModel):
            result = await f(
                **{name: getattr(body, name) for name in type(body).model_fields}
            )
            _validate_no_prisma_objects(result, f"{func.__name__} result")
            return result
        
        return async_endpoint
    else:
        # 同步端点
        def sync_endpoint(body: RequestBodyModel):
            result = f(
                **{name: getattr(body, name) for name in type(body).model_fields}
            )
            _validate_no_prisma_objects(result, f"{func.__name__} result")
            return result
        
        return sync_endpoint
```

### 完整示例

```python
# 原始方法
@expose
def calculate_price(self, base_price: float, tax_rate: float = 0.1, discount: float = 0) -> dict:
    total = base_price * (1 + tax_rate) - discount
    return {"total": total, "breakdown": {"base": base_price, "tax": tax_rate}}

# 自动生成的 Pydantic 模型（等价于）
class RequestBodyModel(BaseModel):
    base_price: float  # 必填，无默认值
    tax_rate: float = 0.1  # 可选，有默认值
    discount: float = 0  # 可选，有默认值

# 自动生成的 FastAPI 端点（等价于）
@app.post("/calculate_price")
def calculate_price_endpoint(body: RequestBodyModel):
    result = self.calculate_price(
        base_price=body.base_price,
        tax_rate=body.tax_rate,
        discount=body.discount
    )
    return result

# 客户端调用
client.calculate_price(base_price=100.0, tax_rate=0.15)
# → POST /calculate_price
# → Body: {"base_price": 100.0, "tax_rate": 0.15, "discount": 0}
```

---

## 核心内容 4：inspect 模块的应用

### inspect 的 4 大应用场景

```python
# 1. 提取函数签名
sig = inspect.signature(func)
# → Signature(base_price: float, tax_rate: float = 0.1, discount: float = 0)

# 2. 遍历参数
for name, param in sig.parameters.items():
    print(f"{name}: {param.annotation}, default={param.default}")
# → base_price: <class 'float'>, default=<class 'inspect._empty'>
# → tax_rate: <class 'float'>, default=0.1

# 3. 检测函数类型
if asyncio.iscoroutinefunction(func):
    print("异步函数")
else:
    print("同步函数")

# 4. 动态获取异常类（异常映射）
EXCEPTION_MAPPING = {
    e.__name__: e
    for e in [ValueError, RuntimeError, ...]
    for _, ErrorType in inspect.getmembers(exceptions)
    if inspect.isclass(ErrorType)
    and issubclass(ErrorType, Exception)
}
```

### 实战：参数解析

```python
def _get_params(self, signature: inspect.Signature, *args: Any, **kwargs: Any) -> dict[str, Any]:
    """
    将位置参数转换为关键字参数
    
    示例：
    client.add(5, 3)  # 位置参数
    → 转换为 → {"a": 5, "b": 3}
    → 发送为 → POST /add Body: {"a": 5, "b": 3}
    """
    if args:
        # 获取参数名列表
        arg_names = list(signature.parameters.keys())
        
        # 跳过 self/cls
        if arg_names and arg_names[0] in ("self", "cls"):
            arg_names = arg_names[1:]
        
        # 将位置参数映射到参数名
        kwargs.update(dict(zip(arg_names, args)))
    
    return kwargs

# 使用示例
sig = inspect.signature(add_method)  # add(a: int, b: int)
params = _get_params(sig, 5, 3)
# → {"a": 5, "b": 3}
```

### 类型验证和转换

```python
def _get_return(self, expected_return: TypeAdapter | None, result: Any) -> Any:
    """
    使用类型注解验证和转换返回值
    
    示例：
    def get_user(user_id: str) -> User:  # 返回类型注解
        ...
    
    result = {"id": 1, "name": "John"}  # API 返回的 dict
    → TypeAdapter(User).validate_python(result)
    → User(id=1, name="John")  # 转换为 Pydantic 模型
    """
    if expected_return:
        return expected_return.validate_python(result)
    return result

# 在客户端使用
sig = inspect.signature(original_func)
ret_ann = sig.return_annotation  # 获取返回类型
expected_return = (
    None if ret_ann is inspect.Signature.empty 
    else TypeAdapter(ret_ann)
)
```

---

## 核心内容 5：RPC 客户端实现

### DynamicClient 核心机制

```python
class DynamicClient:
    """
    动态 RPC 客户端 - 魔法在于 __getattr__
    
    原理：
    1. 拦截所有未定义的属性访问
    2. 检查 AppServiceClient 是否定义了该方法
    3. 动态创建 HTTP 调用包装器
    4. 保持类型签名和文档
    """
    
    def __init__(self):
        service_type = service_client_type.get_service_type()
        host = service_type.get_host()
        port = service_type.get_port()
        self.base_url = f"http://{host}:{port}"
        
        # 连接池管理
        self._sync_clients = {}   # 同步客户端池
        self._async_clients = {}  # 异步客户端池（按事件循环）
    
    @property
    def async_client(self) -> httpx.AsyncClient:
        """
        智能异步客户端管理 - 每个事件循环一个客户端
        
        为什么？
        - httpx.AsyncClient 不是线程安全的
        - 不同的事件循环需要独立的客户端
        - 避免 "attached to a different loop" 错误
        """
        try:
            loop = asyncio.get_running_loop()
        except RuntimeError:
            loop = None  # 没有事件循环时的默认客户端  
        
        if client := self._async_clients.get(loop):
            return client
        
        # 创建新客户端并缓存
        return self._async_clients.setdefault(
            loop, 
            self._create_async_client()
        )
    
    def __getattr__(self, name: str) -> Callable[..., Any]:
        """
        核心魔法：拦截方法调用
        
        client.add_numbers(5, 3)
        ↓
        1. __getattr__("add_numbers") 被调用
        2. 从 service_client_type 获取方法签名
        3. 创建 HTTP 调用包装器
        4. 返回包装器（看起来像普通方法）
        """
        # 获取原始方法定义
        original_func = getattr(service_client_type, name, None)
        if original_func is None:
            raise AttributeError(f"Method {name} not found")
        
        # 提取签名和返回类型
        sig = inspect.signature(original_func)
        ret_ann = sig.return_annotation
        expected_return = (
            None if ret_ann is inspect.Signature.empty 
            else TypeAdapter(ret_ann)
        )
        
        # 根据是否异步创建不同的包装器
        if inspect.iscoroutinefunction(original_func):
            # 异步方法
            async def async_method(*args, **kwargs):
                params = self._get_params(sig, *args, **kwargs)
                result = await self._call_method_async(name, **params)
                return self._get_return(expected_return, result)
            
            return async_method
        else:
            # 同步方法
            def sync_method(*args, **kwargs):
                params = self._get_params(sig, *args, **kwargs)
                result = self._call_method_sync(name, **params)
                return self._get_return(expected_return, result)
            
            return sync_method
```

### HTTP 调用实现

```python
@_maybe_retry  # 可选的重试装饰器
async def _call_method_async(self, method_name: str, **kwargs: Any) -> Any:
    """
    异步 RPC 调用
    
    流程：
    1. 使用 async_client 发送 POST 请求
    2. 路径 = 方法名，Body = JSON 参数
    3. 处理响应或异常
    """
    try:
        response = await self.async_client.post(
            method_name,  # 例如："/add_numbers"
            json=to_dict(kwargs)  # {"a": 5, "b": 3}
        )
        return self._handle_call_method_response(
            method_name=method_name,
            response=response
        )
    except (httpx.ConnectError, httpx.ConnectTimeout) as e:
        self._handle_connection_error(e)
        raise

def _call_method_sync(self, method_name: str, **kwargs: Any) -> Any:
    """同步版本 - 逻辑相同"""
    try:
        response = self.sync_client.post(
            method_name,
            json=to_dict(kwargs)
        )
        return self._handle_call_method_response(
            method_name=method_name,
            response=response
        )
    except (httpx.ConnectError, httpx.ConnectTimeout) as e:
        self._handle_connection_error(e)
        raise
```

### 连接自愈机制

```python
def _handle_connection_error(self, error: Exception) -> None:
    """
    智能连接错误处理和自愈
    
    策略：
    1. 计数连接失败次数
    2. 达到阈值（3次）且距上次重置超过30秒
    3. 清空客户端缓存，强制重新创建
    4. 重置计数器
    """
    self._connection_failure_count += 1
    current_time = time.time()
    
    if (
        self._connection_failure_count >= 3
        and current_time - self._last_client_reset > 30
    ):
        logger.warning(
            f"Connection failures detected ({self._connection_failure_count}), "
            "recreating HTTP clients"
        )
        
        # 清空缓存 - 下次访问时自动重建
        self._sync_clients.clear()
        self._async_clients.clear()
        
        # 重置
        self._connection_failure_count = 0
        self._last_client_reset = current_time
```

---

##  核心内容 6：异常映射和传递

### 异常映射机制

```python
# 第 159-177 行：构建异常映射表
EXCEPTION_MAPPING = {
    e.__name__: e  # 异常名 → 异常类
    for e in [
        # 内置异常
        ValueError,
        RuntimeError,
        TimeoutError,
        ConnectionError,
        
        # 自定义异常
        UnhealthyServiceError,
        HTTPClientError,
        HTTPServerError,
        
        # backend.util.exceptions 中的所有异常
        *[
            ErrorType
            for _, ErrorType in inspect.getmembers(exceptions)
            if inspect.isclass(ErrorType)
            and issubclass(ErrorType, Exception)
            and ErrorType.__module__ == exceptions.__name__
        ],
    ]
}

# 结果示例：
# {
#     "ValueError": <class 'ValueError'>,
#     "RuntimeError": <class 'RuntimeError'>,
#     "NotFoundError": <class 'backend.util.exceptions.NotFoundError'>,
#     "NotAuthorizedError": <class 'backend.util.exceptions.NotAuthorizedError'>,
#     ...
# }
```

### 服务端异常序列化

```python
# 第 189-206 行：异常处理器
@staticmethod
def _handle_internal_http_error(status_code: int = 500, log_error: bool = True):
    def handler(request: Request, exc: Exception):
        if log_error:
            logger.exception(f"{request.method} {request.url.path} failed: {exc}")
        
        # 将异常序列化为 JSON
        return responses.JSONResponse(
            status_code=status_code,
            content=RemoteCallError(
                type=str(exc.__class__.__name__),  # 异常类名
                args=exc.args or (str(exc),),      # 异常参数
            ).model_dump(),
        )
    
    return handler

# 注册异常处理器
self.fastapi_app.add_exception_handler(
    ValueError, self._handle_internal_http_error(400)
)
self.fastapi_app.add_exception_handler(
    Exception, self._handle_internal_http_error(500)
)
```

### 客户端异常重建

```python
# 第 474-507 行：响应处理
def _handle_call_method_response(
    self, *, response: httpx.Response, method_name: str
) -> Any:
    try:
        response.raise_for_status()
        self._connection_failure_count = 0  # 成功则重置
        return response.json()
    
    except httpx.HTTPStatusError as e:
        status_code = e.response.status_code
        
        # 尝试解析为 RemoteCallError
        error_response = None
        try:
            error_response = RemoteCallError.model_validate(
                e.response.json()
            )
        except Exception:
            pass
        
        # 重建映射的异常
        if error_response and error_response.type in EXCEPTION_MAPPING:
            exception_class = EXCEPTION_MAPPING[error_response.type]
            args = error_response.args or [str(e)]
            raise exception_class(*args)  # ← 重建原始异常！
        
        # HTTP 状态码分类
        if 400 <= status_code < 500:
            raise HTTPClientError(status_code, str(e))
        elif 500 <= status_code < 600:
            raise HTTPServerError(status_code, str(e))
        else:
            raise e
```

### 完整异常传递流程

```python
"""
异常传递完整流程：

服务端：
1. 用户调用：client.get_user("invalid-id")
2. 服务端执行：raise NotFoundError("User not found")
3. FastAPI 捕获异常
4. 异常处理器序列化：
   {
     "type": "NotFoundError",
     "args": ["User not found"]
   }
5. 返回 HTTP 404 + JSON body

客户端：
6. httpx 接收 404 响应
7. 解析 JSON → RemoteCallError
8. 查找 EXCEPTION_MAPPING["NotFoundError"]
9. 重建异常：raise NotFoundError("User not found")
10. 用户代码捕获：except NotFoundError as e

结果：就像本地调用一样！
"""

# 服务端
class UserService(AppService):
    @expose
    def get_user(self, user_id: str) -> User:
        user = database.find(user_id)
        if not user:
            raise NotFoundError(f"User {user_id} not found")
        return user

# 客户端
try:
    user = client.get_user("invalid-id")
except NotFoundError as e:  # ← 异常被完美重建
    print(f"Error: {e}")  # → "User invalid-id not found"
```

---

## 核心内容 7：实战示例

### 完整的服务定义

```python
# backend/executor/database.py
from backend.util.service import AppService, AppServiceClient, expose, endpoint_to_sync

class DatabaseManager(AppService):
    """数据库管理服务 - RPC 服务端"""
    
    @classmethod
    def get_port(cls) -> int:
        return 8002  # 服务端口
    
    def run_service(self) -> None:
        """服务启动时的初始化"""
        logger.info("Connecting to Database...")
        self.run_and_wait(db.connect())
        logger.info("Database connected successfully")
    
    @expose
    async def get_graph(
        self,
        graph_id: str,
        user_id: str,
        version: Optional[int] = None
    ) -> GraphModel:
        """
        获取 Graph - 异步方法
        
        自动转换为：POST /get_graph
        请求体：{"graph_id": "...", "user_id": "...", "version": 1}
        """
        graph = await graph_db.get_graph(
            graph_id=graph_id,
            user_id=user_id,
            version=version
        )
        if not graph:
            raise NotFoundError(f"Graph {graph_id} not found")
        return graph
    
    @expose
    async def create_graph(
        self,
        graph: GraphCreate,
        user_id: str
    ) -> GraphModel:
        """
        创建 Graph
        
        注意：接受 Pydantic 模型作为参数
        Pydantic 会自动序列化/反序列化
        """
        new_graph = await graph_db.create_graph(graph, user_id)
        return new_graph
    
    @expose
    async def delete_graph(self, graph_id: str, user_id: str) -> dict:
        """删除 Graph"""
        deleted_count = await graph_db.delete_graph(graph_id, user_id)
        return {"deleted": deleted_count}
```

### 客户端定义（同步）

```python
class DatabaseManagerClient(AppServiceClient):
    """数据库管理客户端 - 同步版本"""
    
    @classmethod
    def get_service_type(cls) -> Type[AppService]:
        return DatabaseManager
    
    # 使用 endpoint_to_sync 创建类型存根
    _ = endpoint_to_sync
    d = DatabaseManager
    
    # 方法签名必须与服务端一致
    get_graph = _(d.get_graph)
    create_graph = _(d.create_graph)
    delete_graph = _(d.delete_graph)
    
    # IDE 看到的签名（同步版本）：
    # def get_graph(self, graph_id: str, user_id: str, version: Optional[int] = None) -> GraphModel:
    #     ...

# 使用客户端
client = get_service_client(DatabaseManagerClient)

# 同步调用（即使服务端是异步的）
graph = client.get_graph(
    graph_id="graph-123",
    user_id="user-456"
)
```

### 客户端定义（异步）

```python
class DatabaseManagerAsyncClient(AppServiceClient):
    """数据库管理客户端 - 异步版本"""
    
    @classmethod
    def get_service_type(cls) -> Type[AppService]:
        return DatabaseManager
    
    # 异步客户端不需要 endpoint_to_async
    # 直接使用原方法签名
    async def get_graph(
        self,
        graph_id: str,
        user_id: str,
        version: Optional[int] = None
    ) -> GraphModel:
        ...  # 由 __getattr__ 拦截
    
    async def create_graph(
        self,
        graph: GraphCreate,
        user_id: str
    ) -> GraphModel:
        ...

# 使用
client = get_service_client(DatabaseManagerAsyncClient)
graph = await client.get_graph("graph-123", "user-456")
```

### endpoint_to_sync/async 魔法

```python
# backend/util/service.py (633-658行)

def endpoint_to_sync(
    func: Callable[Concatenate[Any, P], Awaitable[R]],
) -> Callable[Concatenate[Any, P], R]:
    """
    类型转换魔法：async → sync
    
    作用：
    1. 接受异步函数签名
    2. 返回同步函数签名
    3. 实际不执行（会被 __getattr__ 拦截）
    4. 仅用于 IDE 类型提示
    
    示例：
    # 服务端
    async def get_user(self, user_id: str) -> User: ...
    
    # 客户端
    get_user = endpoint_to_sync(d.get_user)
    
    # IDE 看到：
    def get_user(self, user_id: str) -> User: ...  # 同步！
    """
    def _stub(*args: P.args, **kwargs: P.kwargs) -> R:
        raise RuntimeError("should be intercepted by __getattr__")
    
    update_wrapper(_stub, func)  # 复制文档和签名
    return cast(Callable[Concatenate[Any, P], R], _stub)

def endpoint_to_async(
    func: Callable[Concatenate[Any, P], R],
) -> Callable[Concatenate[Any, P], Awaitable[R]]:
    """
    类型转换魔法：sync → async（较少用）
    """
    async def _stub(*args: P.args, **kwargs: P.kwargs) -> R:
        raise RuntimeError("should be intercepted by __getattr__")
    
    update_wrapper(_stub, func)
    return cast(Callable[Concatenate[Any, P], Awaitable[R]], _stub)
```

### 实际调用流程

```python
"""
完整的 RPC 调用流程：

1. 客户端调用
   client.get_graph(graph_id="123", user_id="456")
   
2. __getattr__ 拦截
   - 检测到 "get_graph" 属性访问
   - 从 DatabaseManagerClient 获取签名
   - 创建 HTTP 调用包装器
   
3. HTTP 请求
   POST http://localhost:8002/get_graph
   Body: {"graph_id": "123", "user_id": "456"}
   
4. 服务端处理
   - FastAPI 路由到 /get_graph
   - 调用 _create_fastapi_endpoint 生成的端点
   - Pydantic 验证请求体
   - 调用 DatabaseManager.get_graph(graph_id="123", user_id="456")
   - 返回 GraphModel
   
5. 响应序列化
   - GraphModel.model_dump() → JSON
   - 返回 HTTP 200 + JSON
   
6. 客户端反序列化
   - 解析 JSON
   - TypeAdapter(GraphModel).validate_python(json_data)
   - 返回 GraphModel 实例
   
7. 用户获得结果
   graph: GraphModel = client.get_graph(...)
"""
```

---

## 学习重点 1：理解 RPC 设计模式

### RPC vs REST 对比

```python
# ===== REST 风格 =====
# 优点：标准化、缓存友好、语义清晰
# 缺点：需要手动编写路由、序列化、客户端代码

# 服务端
@app.get("/graphs/{graph_id}")
async def get_graph(graph_id: str, user_id: str = Query(...)):
    graph = await database.get_graph(graph_id, user_id)
    return graph.model_dump()

# 客户端
response = requests.get(
    f"http://service/graphs/{graph_id}",
    params={"user_id": user_id}
)
graph_dict = response.json()
graph = GraphModel(**graph_dict)  # 手动反序列化

# ===== RPC 风格（AutoGPT） =====
# 优点：简单、类型安全、像本地调用
# 缺点：不遵循 REST 约定、难以缓存

# 服务端
@expose
async def get_graph(self, graph_id: str, user_id: str) -> GraphModel:
    return await database.get_graph(graph_id, user_id)

# 客户端
graph: GraphModel = await client.get_graph(graph_id, user_id)
# ↑ 完全像本地调用！
```

### AutoGPT RPC 的设计优势

```python
"""
1. 零样板代码
   - 不需要手写路由定义
   - 不需要手写序列化逻辑
   - 不需要手写客户端方法

2. 类型安全
   - 使用 Python 类型注解
   - IDE 自动补全和类型检查
   - Pydantic 自动验证

3. 异常透明
   - 服务端异常自动传递到客户端
   - 客户端可以捕获具体异常类型
   - 不需要解析错误码

4. 同步/异步灵活
   - 服务端可以是异步的
   - 客户端可以选择同步或异步调用
   - 自动处理事件循环隔离

5. 可观测性
   - 自动集成 Prometheus 指标
   - 自动集成 Sentry 错误追踪
   - 健康检查端点
"""
```

### 适用场景

```python
"""
✅ 适合 RPC 的场景：
1. 微服务内部通信（不对外暴露）
2. 类型安全要求高
3. 快速迭代开发
4. 服务数量可控

❌ 不适合 RPC 的场景：
1. 公共 API（应使用 REST）
2. 需要 HTTP 缓存
3. 需要语义化 URL
4. 第三方集成
"""
```

---

## 学习重点 2：掌握元编程技巧

### 技巧 1：动态创建 Pydantic 模型

```python
from pydantic import create_model

# 动态创建模型
fields = {
    "name": (str, ...),           # (类型, 默认值)
    "age": (int, 18),             # 有默认值
    "email": (Optional[str], None) # 可选字段
}

DynamicModel = create_model("DynamicModel", **fields)

# 等价于手写：
class DynamicModel(BaseModel):
    name: str
    age: int = 18
    email: Optional[str] = None

# AutoGPT 使用场景
def _create_fastapi_endpoint(self, func):
    sig = inspect.signature(func)
    fields = {}
    for name, param in sig.parameters.items():
        if name != "self":
            fields[name] = (param.annotation, param.default)
    
    RequestModel = create_model("RequestModel", **fields)
    # ↑ 根据函数签名动态创建请求模型
```

### 技巧 2：inspect 签名操作

```python
import inspect

def analyze_function(func):
    """深入分析函数签名"""
    sig = inspect.signature(func)
    
    # 1. 获取所有参数
    params = sig.parameters
    # → OrderedDict([('self', <Parameter>), ('user_id', <Parameter>), ...])
    
    # 2. 检查参数类型
    for name, param in params.items():
        print(f"参数: {name}")
        print(f"  类型: {param.annotation}")
        print(f"  默认值: {param.default}")
        print(f"  种类: {param.kind}")
        # kind 可以是：
        # - POSITIONAL_OR_KEYWORD: 普通参数
        # - VAR_POSITIONAL: *args
        # - VAR_KEYWORD: **kwargs
        # - KEYWORD_ONLY: 仅关键字参数
    
    # 3. 获取返回类型
    return_type = sig.return_annotation
    
    # 4. 检查是否异步
    is_async = inspect.iscoroutinefunction(func)
    
    return {
        "params": list(params.keys()),
        "return_type": return_type,
        "is_async": is_async
    }

# 实战示例
async def get_user(user_id: str, include_posts: bool = False) -> User:
    ...

info = analyze_function(get_user)
# {
#     "params": ["user_id", "include_posts"],
#     "return_type": User,
#     "is_async": True
# }
```

### 技巧 3：__getattr__ 魔法方法

```python
class DynamicProxy:
    """动态代理模式"""
    
    def __init__(self, target):
        self._target = target
    
    def __getattr__(self, name):
        """拦截所有属性访问"""
        print(f"访问属性: {name}")
        
        # 获取目标对象的属性
        original = getattr(self._target, name)
        
        if callable(original):
            # 如果是方法，返回包装器
            def wrapper(*args, **kwargs):
                print(f"调用方法: {name}")
                print(f"参数: {args}, {kwargs}")
                result = original(*args, **kwargs)
                print(f"结果: {result}")
                return result
            return wrapper
        else:
            # 如果是普通属性，直接返回
            return original

# 使用
class Calculator:
    def add(self, a, b):
        return a + b

calc = DynamicProxy(Calculator())
result = calc.add(5, 3)
# 输出：
# 访问属性: add
# 调用方法: add
# 参数: (5, 3), {}
# 结果: 8
```

### 技巧 4：装饰器元数据

```python
def with_metadata(**metadata):
    """添加元数据的装饰器"""
    def decorator(func):
        # 在函数上设置自定义属性
        for key, value in metadata.items():
            setattr(func, f"__{key}__", value)
        return func
    return decorator

# 使用
@with_metadata(rate_limit=100, cache=True)
def expensive_operation():
    pass

# 检查元数据
print(expensive_operation.__rate_limit__)  # 100
print(expensive_operation.__cache__)       # True

# AutoGPT 使用
@expose  # 设置 __exposed__ = True
def my_method():
    pass

# 检查
if getattr(my_method, "__exposed__", False):
    print("这是暴露的方法")
```

### 技巧 5：类型适配器

```python
from pydantic import TypeAdapter

# 动态类型验证
UserAdapter = TypeAdapter(User)

# 验证和转换
user_dict = {"id": 1, "name": "John"}
user = UserAdapter.validate_python(user_dict)
# → User(id=1, name="John")

# 支持泛型
ListUserAdapter = TypeAdapter(list[User])
users = ListUserAdapter.validate_python([
    {"id": 1, "name": "John"},
    {"id": 2, "name": "Jane"}
])
# → [User(...), User(...)]

# AutoGPT 使用
sig = inspect.signature(func)
ret_type = sig.return_annotation

if ret_type != inspect.Signature.empty:
    adapter = TypeAdapter(ret_type)
    result = adapter.validate_python(json_data)
else:
    result = json_data
```

---

## 学习重点 3：服务间通信最佳实践

### 1. 连接池管理

```python
class DynamicClient:
    def __init__(self):
        self._sync_clients = {}
        self._async_clients = {}  # 按事件循环分组
    
    @property
    def async_client(self) -> httpx.AsyncClient:
        """
        最佳实践：每个事件循环一个客户端
        
        为什么？
        - httpx.AsyncClient 不是线程安全的
        - 不能跨事件循环共享
        - 避免 "attached to a different loop" 错误
        """
        try:
            loop = asyncio.get_running_loop()
        except RuntimeError:
            loop = None
        
        if client := self._async_clients.get(loop):
            return client
        
        return self._async_clients.setdefault(
            loop,
            httpx.AsyncClient(
                base_url=self.base_url,
                timeout=30,
                limits=httpx.Limits(
                    max_keepalive_connections=200,
                    max_connections=500,
                    keepalive_expiry=30.0,
                )
            )
        )
```

### 2. 超时和重试

```python
# 配置
config.rpc_client_call_timeout = 30       # 单次调用超时
config.pyro_client_comm_retry = 3         # 重试次数
config.pyro_client_max_wait = 10          # 最大等待时间

# 使用
client = get_service_client(
    MyServiceClient,
    call_timeout=60,      # 覆盖默认超时
    request_retry=True    # 启用重试
)

# 重试策略
def _maybe_retry(fn):
    if not request_retry:
        return fn
    
    return create_retry_decorator(
        max_attempts=3,
        max_wait=10,
        exclude_exceptions=(
            ValueError,         # 不重试的异常
            HTTPClientError,    # 4xx 错误
        )
    )(fn)
```

### 3. 健康检查

```python
class AppService(BaseAppService):
    async def health_check(self) -> str:
        """标准健康检查"""
        return "OK"
    
    # 自动注册为：
    # GET /health_check
    # POST /health_check

# 使用
client = get_service_client(MyServiceClient)
try:
    status = client.health_check()
    if status == "OK":
        # 服务健康
        pass
except Exception as e:
    # 服务不可用
    raise UnhealthyServiceError(f"Service unhealthy: {e}")
```

### 4. Prisma 对象验证

```python
def _validate_no_prisma_objects(obj: Any, path: str = "result") -> None:
    """
    最佳实践：禁止返回 Prisma 对象
    
    为什么？
    1. 层次分离：数据库层 ≠ API 层
    2. Prisma 对象包含内部状态，不适合序列化
    3. 应该返回 Application Models
    
    正确做法：
    prisma_user = await prisma.user.find_unique(...)
    return UserModel.from_db(prisma_user)  # ✓
    
    错误做法：
    return prisma_user  # ✗ 会抛出 ValueError
    """
    if hasattr(obj, "__class__") and hasattr(obj.__class__, "__module__"):
        module_name = obj.__class__.__module__
        if "prisma.models" in module_name:
            raise ValueError(
                f"Prisma object {obj.__class__.__name__} found in {path}. "
                "Use {obj.__class__.__name__}.from_db() to convert."
            )
```

### 5. 错误处理

```python
# 服务端：分类错误
@expose
async def get_user(self, user_id: str) -> User:
    try:
        user = await db.find_user(user_id)
        if not user:
            # 4xx：客户端错误
            raise NotFoundError(f"User {user_id} not found")
        return user
    except DatabaseError as e:
        # 5xx：服务器错误
        raise RuntimeError(f"Database error: {e}")

# 客户端：捕获具体异常
try:
    user = client.get_user("invalid-id")
except NotFoundError:
    # 处理未找到
    pass
except RuntimeError:
    # 处理服务器错误
    pass
except HTTPClientError as e:
    # 4xx 错误（不会重试）
    if e.status_code == 401:
        # 未授权
        pass
except HTTPServerError as e:
    # 5xx 错误（会重试）
    pass
```

### 6. 资源清理

```python
# 使用上下文管理器
async with get_service_client(MyServiceClient) as client:
    result = await client.do_something()
    # 自动调用 aclose()

# 或手动清理
client = get_service_client(MyServiceClient)
try:
    result = await client.do_something()
finally:
    await client.aclose()  # 关闭所有 HTTP 连接
```

---

## RPC 模式总结

```
AutoGPT RPC 框架特点
│
├─── 🎯 核心设计
│    ├─ @expose 装饰器标记端点
│    ├─ 自动路由生成（方法名 → HTTP 路径）
│    ├─ 动态 Pydantic 模型创建
│    ├─ __getattr__ 魔法方法拦截
│    └─ 透明的异常传递
│
├─── 🔧 元编程技巧
│    ├─ inspect.signature() 提取签名
│    ├─ create_model() 动态模型
│    ├─ TypeAdapter 类型转换
│    ├─ update_wrapper() 保持元数据
│    └─ 装饰器元数据存储
│
├─── 🌐 通信机制
│    ├─ HTTP + JSON（FastAPI + httpx）
│    ├─ POST /method_name
│    ├─ 连接池管理（按事件循环）
│    ├─ 自动重试机制
│    └─ 连接自愈
│
├─── 🛡️ 可靠性
│    ├─ 异常映射和重建
│    ├─ 类型验证（Pydantic）
│    ├─ Prisma 对象检查
│    ├─ 超时控制
│    └─ 健康检查
│
└─── 📈 可观测性
     ├─ Prometheus 指标
     ├─ Sentry 错误追踪
     ├─ 结构化日志
     └─ 连接失败计数
```

---

# 专题B：双服务器架构

---

## 核心内容 1：REST API Server 架构

### AgentServer 职责

```python
# backend/server/rest_api.py

class AgentServer(AppProcess):
    """
    REST API 服务器 - 处理所有 HTTP REST 请求
    
    职责：
    1. 图（Graph）的 CRUD 操作
    2. 节点（Block）管理
    3. 执行触发（创建执行任务）
    4. 用户认证和授权
    5. Store/Library 管理
    6. 健康检查
    """
    
    def run(self):
        # 配置 CORS
        server_app = CORSMiddleware(
            app=app,
            allow_origins=settings.config.backend_cors_allow_origins,
            allow_credentials=True,
            allow_methods=["*"],
            allow_headers=["*"],
        )
        
        config = Config()
        
        # uvicorn 性能优化配置
        uvicorn_config = {
            "app": server_app,
            "host": config.agent_api_host,       # 默认 0.0.0.0
            "port": config.agent_api_port,       # 默认 8006
            "log_config": None,
            "http": "httptools",                 # 使用 httptools 解析器
            "loop": "uvloop" if platform.system() != "Windows" else "auto",
        }
        
        uvicorn.run(**uvicorn_config)
```

### REST API 路由结构

```python
# backend/server/rest_api.py

app = FastAPI(
    title="AutoGPT Agent Server",
    description="Server for executing AutoGPT agents",
    version="0.1",
    lifespan=lifespan_context,
    docs_url=docs_url,
    generate_unique_id_function=custom_generate_unique_id,
)

# ===== 路由注册 =====

# v1 API（主要功能）
app.include_router(
    backend.server.routers.v1.v1_router,
    tags=["v1"],
    prefix="/api"
)
# → /api/graphs, /api/blocks, /api/auth, /api/executions

# v2 Store（应用商店）
app.include_router(
    backend.server.v2.store.routes.router,
    tags=["v2"],
    prefix="/api/store"
)

# v2 Builder（图构建器）
app.include_router(
    backend.server.v2.builder.routes.router,
    tags=["v2"],
    prefix="/api/builder"
)

# v2 Library（模板库）
app.include_router(
    backend.server.v2.library.routes.router,
    tags=["v2"],
    prefix="/api/library"
)

# 管理员路由
app.include_router(
    backend.server.v2.admin.store_admin_routes.router,
    tags=["v2", "admin"],
    prefix="/api/store"
)

# 外部 API（独立挂载）
app.mount("/external-api", external_app)

# 健康检查
@app.get("/health", tags=["health"])
async def health():
    if not backend.data.db.is_connected():
        raise UnhealthyServiceError("Database is not connected")
    return {"status": "healthy"}
```

### REST API 生命周期

```python
# backend/server/rest_api.py

@contextlib.asynccontextmanager
async def lifespan_context(app: FastAPI):
    """
    应用生命周期管理
    
    启动时：
    1. 验证认证配置
    2. 连接数据库
    3. 配置线程池
    4. 初始化 blocks
    5. 数据迁移
    """
    # 1. 验证认证
    verify_auth_settings()
    
    # 2. 连接数据库
    await backend.data.db.connect()
    
    # 3. 配置线程池（关键性能优化）
    config = backend.util.settings.Config()
    try:
        import anyio.to_thread
        anyio.to_thread.current_default_thread_limiter().total_tokens = (
            config.fastapi_thread_pool_size  # 默认 400 线程
        )
        logger.info(f"Thread pool size set to {config.fastapi_thread_pool_size}")
    except (ImportError, AttributeError) as e:
        logger.warning(f"Could not configure thread pool size: {e}")
    
    # 4. 初始化 blocks
    await backend.data.block.initialize_blocks()
    
    # 5. 数据迁移
    await backend.data.user.migrate_and_encrypt_user_integrations()
    await backend.data.graph.fix_llm_provider_credentials()
    await backend.data.graph.migrate_llm_models(LlmModel.GPT4O)
    await backend.integrations.webhooks.utils.migrate_legacy_triggered_graphs()
    
    # 6. 启动 LaunchDarkly（如果非本地环境）
    with launch_darkly_context():
        yield  # 应用运行期间
    
    # 关闭时：清理资源
    try:
        await shutdown_cloud_storage_handler()
    except Exception as e:
        logger.warning(f"Error shutting down cloud storage: {e}")
    
    await backend.data.db.disconnect()
```

---

## 核心内容 2：WebSocket Server 架构

### WebsocketServer 职责

```python
# backend/server/ws_api.py

class WebsocketServer(AppProcess):
    """
    WebSocket 服务器 - 处理实时双向通信
    
    职责：
    1. 实时执行状态更新推送
    2. 订阅/取消订阅管理
    3. 心跳检测
    4. WebSocket 连接管理
    """
    
    def run(self):
        logger.info(f"CORS origins: {settings.config.backend_cors_allow_origins}")
        
        server_app = CORSMiddleware(
            app=app,
            allow_origins=settings.config.backend_cors_allow_origins,
            allow_credentials=True,
            allow_methods=["*"],
            allow_headers=["*"],
        )
        
        uvicorn.run(
            server_app,
            host=Config().websocket_server_host,  # 默认 0.0.0.0
            port=Config().websocket_server_port,  # 默认 8001
            log_config=None,
        )
```

### WebSocket 生命周期

```python
# backend/server/ws_api.py

@asynccontextmanager
async def lifespan(app: FastAPI):
    """
    WebSocket 应用生命周期
    
    启动时：启动事件广播器
    关闭时：停止广播器
    """
    manager = get_connection_manager()
    
    # 创建后台任务：监听 Redis 事件并广播
    fut = asyncio.create_task(event_broadcaster(manager))
    fut.add_done_callback(lambda _: logger.info("Event broadcaster stopped"))
    
    yield  # 应用运行期间

@continuous_retry()  # 自动重试，永不停止
async def event_broadcaster(manager: ConnectionManager):
    """
    事件广播器 - 核心通信桥梁
    
    流程：
    1. 监听 Redis Pub/Sub（所有执行事件）
    2. 接收到事件 → 发送给订阅的 WebSocket 连接
    """
    event_queue = AsyncRedisExecutionEventBus()
    
    # 监听所有频道（"*" 通配符）
    async for event in event_queue.listen("*"):
        # 广播给订阅者
        await manager.send_execution_update(event)
```

### WebSocket 路由

```python
# backend/server/ws_api.py

@app.websocket("/ws")
async def websocket_router(
    websocket: WebSocket,
    manager: ConnectionManager = Depends(get_connection_manager)
):
    """
    WebSocket 主端点
    
    处理流程：
    1. 认证
    2. 建立连接
    3. 消息循环
    4. 断开清理
    """
    # 1. 认证
    user_id = await authenticate_websocket(websocket)
    if not user_id:
        return  # 认证失败，连接已关闭
    
    # 2. 建立连接
    await manager.connect_socket(websocket)
    update_websocket_connections(user_id, 1)  # Prometheus 指标
    
    try:
        # 3. 消息循环
        while True:
            data = await websocket.receive_text()
            
            # 解析消息
            try:
                message = WSMessage.model_validate_json(data)
            except pydantic.ValidationError as e:
                await websocket.send_text(
                    WSMessage(
                        method=WSMethod.ERROR,
                        success=False,
                        error="Invalid message format"
                    ).model_dump_json()
                )
                continue
            
            # 分发到处理器
            if message.method in _MSG_HANDLERS:
                await _MSG_HANDLERS[message.method](
                    connection_manager=manager,
                    websocket=websocket,
                    user_id=user_id,
                    message=message,
                )
            else:
                await websocket.send_text(
                    WSMessage(
                        method=WSMethod.ERROR,
                        success=False,
                        error="Unknown message type"
                    ).model_dump_json()
                )
    
    except WebSocketDisconnect:
        manager.disconnect_socket(websocket)
        logger.debug("WebSocket client disconnected")
    
    finally:
        # 4. 清理
        update_websocket_connections(user_id, -1)

# 消息处理器映射
_MSG_HANDLERS: dict[WSMethod, WSMessageHandler] = {
    WSMethod.HEARTBEAT: handle_heartbeat,
    WSMethod.SUBSCRIBE_GRAPH_EXEC: handle_subscribe,
    WSMethod.SUBSCRIBE_GRAPH_EXECS: handle_subscribe,
    WSMethod.UNSUBSCRIBE: handle_unsubscribe,
}
```

### WebSocket 消息协议

```python
# backend/server/model.py

class WSMethod(str, Enum):
    """WebSocket 消息方法"""
    HEARTBEAT = "heartbeat"
    SUBSCRIBE_GRAPH_EXEC = "subscribe_graph_execution"
    SUBSCRIBE_GRAPH_EXECS = "subscribe_graph_executions"
    UNSUBSCRIBE = "unsubscribe"
    GRAPH_EXECUTION_EVENT = "graph_execution_event"
    NODE_EXECUTION_EVENT = "node_execution_event"
    ERROR = "error"

class WSMessage(BaseModel):
    """WebSocket 消息格式"""
    method: WSMethod
    data: Optional[dict | list | str] = None
    success: bool | None = None
    channel: str | None = None
    error: str | None = None

# 示例：订阅执行
{
    "method": "subscribe_graph_execution",
    "data": {"graph_exec_id": "exec-123"}
}

# 示例：接收事件
{
    "method": "graph_execution_event",
    "channel": "user-456|graph_exec#exec-123",
    "data": {
        "id": "exec-123",
        "status": "RUNNING",
        "user_id": "user-456",
        ...
    }
}
```

---

## 核心内容 3：连接管理器

### ConnectionManager 设计

```python
# backend/server/conn_manager.py

class ConnectionManager:
    """
    WebSocket 连接和订阅管理器
    
    架构：
    - active_connections: 所有活跃的 WebSocket 连接
    - subscriptions: 频道 → WebSocket 连接的映射
    """
    
    def __init__(self):
        self.active_connections: Set[WebSocket] = set()
        self.subscriptions: Dict[str, Set[WebSocket]] = {}
        # 频道格式：
        # - "user_id|graph_exec#exec_id"  # 单个执行
        # - "user_id|graph#graph_id|executions"  # 所有执行
    
    async def connect_socket(self, websocket: WebSocket):
        """建立连接"""
        await websocket.accept()
        self.active_connections.add(websocket)
    
    def disconnect_socket(self, websocket: WebSocket):
        """断开连接 - 自动清理所有订阅"""
        self.active_connections.discard(websocket)
        # 从所有订阅中移除
        for subscribers in self.subscriptions.values():
            subscribers.discard(websocket)
    
    async def subscribe_graph_exec(
        self, *, user_id: str, graph_exec_id: str, websocket: WebSocket
    ) -> str:
        """订阅单个执行"""
        channel_key = f"{user_id}|graph_exec#{graph_exec_id}"
        return await self._subscribe(channel_key, websocket)
    
    async def subscribe_graph_execs(
        self, *, user_id: str, graph_id: str, websocket: WebSocket
    ) -> str:
        """订阅图的所有执行"""
        channel_key = f"{user_id}|graph#{graph_id}|executions"
        return await self._subscribe(channel_key, websocket)
    
    async def _subscribe(self, channel_key: str, websocket: WebSocket) -> str:
        """通用订阅逻辑"""
        if channel_key not in self.subscriptions:
            self.subscriptions[channel_key] = set()
        self.subscriptions[channel_key].add(websocket)
        return channel_key
    
    async def send_execution_update(
        self, exec_event: GraphExecutionEvent | NodeExecutionEvent
    ) -> int:
        """
        发送执行更新到订阅者
        
        智能路由：
        - GraphExecutionEvent → 发送到两个频道
          1. 单个执行订阅者
          2. 所有执行订阅者
        - NodeExecutionEvent → 只发送到单个执行订阅者
        """
        graph_exec_id = (
            exec_event.id
            if isinstance(exec_event, GraphExecutionEvent)
            else exec_event.graph_exec_id
        )
        
        n_sent = 0
        channels: set[str] = {
            # 单个执行频道
            f"{exec_event.user_id}|graph_exec#{graph_exec_id}"
        }
        
        if isinstance(exec_event, GraphExecutionEvent):
            # 所有执行频道
            channels.add(
                f"{exec_event.user_id}|graph#{exec_event.graph_id}|executions"
            )
        
        # 发送到所有相关订阅者
        for channel in channels.intersection(self.subscriptions.keys()):
            message = WSMessage(
                method=_EVENT_TYPE_TO_METHOD_MAP[exec_event.event_type],
                channel=channel,
                data=exec_event.model_dump(),
            ).model_dump_json()
            
            for connection in self.subscriptions[channel]:
                await connection.send_text(message)
                n_sent += 1
        
        return n_sent
```

---

## 核心内容 4：服务发现和协调

### 进程启动顺序

```python
# backend/app.py

def main(**kwargs):
    """
    启动所有服务进程
    
    顺序很重要！
    """
    from backend.executor import DatabaseManager, ExecutionManager, Scheduler
    from backend.notifications import NotificationManager
    from backend.server.rest_api import AgentServer
    from backend.server.ws_api import WebsocketServer
    
    run_processes(
        DatabaseManager().set_log_level("warning"),  # 1. 数据库服务
        Scheduler(),                                 # 2. 调度器
        NotificationManager(),                       # 3. 通知管理器
        WebsocketServer(),                           # 4. WebSocket 服务器
        AgentServer(),                               # 5. REST API 服务器
        ExecutionManager(),                          # 6. 执行管理器（前台）
        **kwargs,
    )

def run_processes(*processes: "AppProcess", **kwargs):
    """
    执行所有进程
    
    策略：
    - 前 N-1 个进程：后台运行
    - 最后一个进程：前台运行（阻塞主进程）
    """
    try:
        # 后台启动前 N-1 个进程
        for process in processes[:-1]:
            process.start(background=True, **kwargs)
        
        # 前台运行最后一个进程
        processes[-1].start(background=False, **kwargs)
    
    finally:
        # 清理所有进程
        for process in processes:
            try:
                process.stop()
            except Exception as e:
                logger.exception(f"Unable to stop {process.service_name}: {e}")
```

### 进程管理

```python
# backend/util/process.py

class AppProcess(ABC):
    """
    应用进程基类
    
    特性：
    1. 多进程隔离（使用 multiprocessing）
    2. 信号处理（SIGTERM, SIGINT）
    3. 优雅关闭
    4. 错误上报（Sentry）
    """
    
    process: Optional[Process] = None
    cleaned_up = False
    
    # 使用 forkserver（更安全）
    if "forkserver" in get_all_start_methods():
        set_start_method("forkserver", force=True)
    else:
        set_start_method("spawn", force=True)
    
    @abstractmethod
    def run(self):
        """子类实现：进程主逻辑"""
        pass
    
    @abstractmethod
    def cleanup(self):
        """子类实现：清理资源"""
        pass
    
    def execute_run_command(self, silent):
        """
        进程执行包装器
        
        1. 注册信号处理
        2. 执行 run()
        3. 异常处理
        4. 清理资源
        """
        # 1. 注册信号处理
        signal.signal(signal.SIGTERM, self._self_terminate)
        signal.signal(signal.SIGINT, self._self_terminate)
        
        try:
            # 设置服务名（用于日志）
            set_service_name(self.service_name)
            logger.info(f"[{self.service_name}] Starting...")
            
            # 2. 执行主逻辑
            self.run()
        
        except BaseException as e:
            logger.warning(
                f"[{self.service_name}] Termination: {type(e).__name__}; {e}"
            )
            # 3. 发送错误到 Sentry
            if not isinstance(e, (KeyboardInterrupt, SystemExit)):
                try:
                    from backend.util.metrics import sentry_capture_error
                    sentry_capture_error(e)
                except Exception:
                    pass
        
        finally:
            # 4. 清理
            self.cleanup()
            logger.info(f"[{self.service_name}] Terminated.")
    
    def start(self, background: bool = False, silent: bool = False, **proc_args):
        """
        启动进程
        
        Args:
            background: 是否后台运行
            silent: 是否禁用输出
        """
        if not background:
            # 前台运行：阻塞当前进程
            self.execute_run_command(silent)
            return 0
        
        # 后台运行：创建新进程
        self.process = Process(
            name=self.__class__.__name__,
            target=self.execute_run_command,
            args=(silent,),
            **proc_args,
        )
        self.process.start()
        logger.info(f"[{self.service_name}] started with PID {self.process.pid}")
        
        return self.process.pid or 0
    
    def stop(self):
        """停止进程"""
        if not self.process:
            return
        
        self.process.terminate()  # 发送 SIGTERM
        self.process.join()       # 等待退出
```

---

## 核心内容 5：共享资源管理

### 数据库连接共享

```python
"""
数据库连接策略：

问题：多进程不能共享数据库连接
解决：每个进程独立连接

模式：
1. DatabaseManager 进程：提供 RPC 服务
2. 其他进程：通过 RPC 调用数据库操作
"""

# DatabaseManager 进程
class DatabaseManager(AppService):
    def run_service(self):
        logger.info("Connecting to Database...")
        self.run_and_wait(db.connect())  # 建立连接
        logger.info("Database connected")

# 其他进程通过 RPC 访问
from backend.util.clients import get_database_manager_client

db_client = get_database_manager_client()
graph = db_client.get_graph(graph_id="123", user_id="456")
```

### Redis 连接共享

```python
"""
Redis 连接策略：

每个进程独立连接 Redis
使用连接池提高性能
"""

# backend/data/redis_client.py

_redis_client: Optional[Redis] = None
_async_redis_client: Optional[AsyncRedis] = None

def get_redis() -> Redis:
    """获取同步 Redis 客户端（线程安全）"""
    global _redis_client
    if _redis_client is None:
        _redis_client = Redis(
            host=settings.redis_host,
            port=settings.redis_port,
            decode_responses=True,
            max_connections=50,  # 连接池
        )
    return _redis_client

async def get_redis_async() -> AsyncRedis:
    """获取异步 Redis 客户端"""
    global _async_redis_client
    if _async_redis_client is None:
        _async_redis_client = AsyncRedis(
            host=settings.redis_host,
            port=settings.redis_port,
            decode_responses=True,
            max_connections=50,
        )
    return _async_redis_client
```

### RabbitMQ 连接共享

```python
"""
RabbitMQ 连接策略：

1. ExecutionManager：
   - 消费者：监听执行队列
   - 生产者：发布执行结果

2. REST API：
   - 生产者：发布执行请求

3. 每个进程独立连接
"""

# ExecutionManager 进程
class ExecutionManager(AppProcess):
    def run(self):
        self.run_client = SyncRabbitMQ(create_execution_queue_config())
        
        # 消费执行任务
        run_channel = self.run_client.get_channel()
        run_channel.basic_consume(
            queue=GRAPH_EXECUTION_QUEUE_NAME,
            on_message_callback=self._handle_run_message,
            auto_ack=False,
        )
        
        run_channel.start_consuming()

# REST API 生产者
async def add_graph_execution(...):
    exec_queue = await get_async_execution_queue()
    await exec_queue.publish_message(
        routing_key=GRAPH_EXECUTION_ROUTING_KEY,
        message=graph_exec_entry.model_dump_json(),
        exchange=GRAPH_EXECUTION_EXCHANGE,
    )
```

---

## 核心内容 6：进程间通信

### 通信方式总结

```python
"""
AutoGPT 进程间通信方式：

1. RPC (HTTP)
   - 用途：同步调用（如数据库操作）
   - 优点：类型安全、简单
   - 缺点：有网络开销

2. RabbitMQ (消息队列)
   - 用途：异步任务分发
   - 优点：可靠、持久化、解耦
   - 缺点：复杂度高

3. Redis Pub/Sub
   - 用途：实时事件广播
   - 优点：快速、简单
   - 缺点：不可靠（订阅者离线会丢失）

4. 共享数据库
   - 用途：持久化状态
   - 优点：一致性
   - 缺点：性能瓶颈
"""
```

### 通信流程图

```
执行图的完整流程：

┌─────────────────┐
│  用户请求       │
│  POST /graphs/  │
│  execute        │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  AgentServer (REST API)     │
│  1. 验证请求                │
│  2. 创建执行记录（DB）      │
│  3. 发布到 RabbitMQ         │
│  4. 返回 execution_id       │
└────────┬────────────────────┘
         │ RabbitMQ
         │ message
         ▼
┌─────────────────────────────┐
│  ExecutionManager           │
│  1. 消费任务                │
│  2. 执行 Graph              │
│  3. 更新状态（DB + Redis）  │
└────────┬────────────────────┘
         │ Redis Pub/Sub
         │ events
         ▼
┌─────────────────────────────┐
│  WebsocketServer            │
│  1. 监听 Redis 事件         │
│  2. 查找订阅者              │
│  3. 推送到 WebSocket        │
└────────┬────────────────────┘
         │ WebSocket
         │ messages
         ▼
┌─────────────────┐
│  前端客户端      │
│  实时显示状态    │
└─────────────────┘
```

### 事件流转示例

```python
"""
示例：执行状态更新的流转

1. ExecutionManager 更新状态
"""
# backend/executor/manager.py
async def update_execution_status(exec_id, status):
    # 更新数据库
    await db.update_graph_execution_stats(
        graph_exec_id=exec_id,
        status=status,
    )
    
    # 发布事件到 Redis
    event = GraphExecutionEvent(
        id=exec_id,
        user_id=user_id,
        graph_id=graph_id,
        status=status,
        event_type=ExecutionEventType.GRAPH_EXEC_UPDATE,
    )
    await get_async_execution_event_bus().publish(event)

"""
2. Redis Pub/Sub 传播
"""
# Redis 自动处理

"""
3. WebsocketServer 接收
"""
# backend/server/ws_api.py
@continuous_retry()
async def event_broadcaster(manager: ConnectionManager):
    event_queue = AsyncRedisExecutionEventBus()
    async for event in event_queue.listen("*"):  # 监听所有事件
        await manager.send_execution_update(event)  # 广播

"""
4. ConnectionManager 路由
"""
# backend/server/conn_manager.py
async def send_execution_update(self, exec_event):
    # 找到订阅者
    channel = f"{exec_event.user_id}|graph_exec#{exec_event.id}"
    if channel in self.subscriptions:
        # 发送给所有订阅者
        for websocket in self.subscriptions[channel]:
            await websocket.send_text(message)

"""
5. 客户端接收
"""
// JavaScript
ws.onmessage = (event) => {
    const message = JSON.parse(event.data);
    if (message.method === 'graph_execution_event') {
        updateUI(message.data);  // 更新界面
    }
};
```

---

让我继续完成服务拆分策略部分...

## 学习重点 1：微服务架构设计

### AutoGPT 服务拆分原则

```python
"""
AutoGPT 的 6 大服务拆分原则：

1. 单一职责原则
   每个服务只负责一个领域

2. 通信协议分离
   REST 和 WebSocket 分开

3. 数据访问隔离
   数据库操作集中到 DatabaseManager

4. 异步任务分离
   ExecutionManager 专门处理执行

5. 定时任务独立
   Scheduler 处理所有调度

6. 通知系统独立
   NotificationManager 统一发送通知
"""
```

### 服务职责划分表

```
服务名称              端口    职责                      通信方式
────────────────────────────────────────────────────────────────
DatabaseManager      8002    数据库CRUD操作            RPC (HTTP)
Scheduler            8003    定时任务调度              RPC (HTTP)
NotificationManager  8004    通知发送                  RPC (HTTP)
WebsocketServer      8001    实时双向通信              WebSocket
AgentServer          8006    REST API                  HTTP
ExecutionManager     N/A     执行图任务                RabbitMQ消费
────────────────────────────────────────────────────────────────

外部依赖：
- PostgreSQL (5432)   : 持久化存储
- Redis (6379)        : 缓存 + Pub/Sub
- RabbitMQ (5672)     : 消息队列
```

### 为什么要拆分 REST 和 WebSocket？

```python
"""
REST API Server vs WebSocket Server 拆分原因：

1. 协议特性不同
   REST:
   - 请求-响应模式
   - 短连接
   - 无状态
   
   WebSocket:
   - 双向通信
   - 长连接
   - 有状态（维护订阅关系）

2. 性能优化不同
   REST:
   - 线程池优化（处理短请求）
   - HTTP/2 多路复用
   - 负载均衡友好
   
   WebSocket:
   - 连接保持
   - 内存管理（维护大量连接）
   - 心跳检测

3. 扩展性不同
   REST:
   - 水平扩展容易（无状态）
   - 可以随意增加实例
   
   WebSocket:
   - 需要会话粘性（sticky session）
   - 或者需要 Redis Pub/Sub 协调

4. 故障隔离
   - WebSocket 服务崩溃不影响 REST API
   - REST API 重启不断开 WebSocket 连接

5. 监控和调试
   - 分开监控更清晰
   - 问题排查更简单
"""
```

### 架构演进

```python
"""
AutoGPT 架构演进路径：

阶段 1：单体应用 (Monolith)
┌─────────────────────────────────┐
│  FastAPI App                    │
│  - REST API                     │
│  - WebSocket                    │
│  - Database                     │
│  - Task Execution               │
└─────────────────────────────────┘
问题：难以扩展、单点故障

阶段 2：基础拆分 (Current)
┌──────────────┐  ┌──────────────┐
│ REST API     │  │ WebSocket    │
│ Server       │  │ Server       │
└──────┬───────┘  └──────┬───────┘
       │                  │
       └─────────┬────────┘
                 │
       ┌─────────▼────────┐
       │  Shared Services │
       │  - Database      │
       │  - Execution     │
       │  - Scheduler     │
       └──────────────────┘

阶段 3：完全微服务 (Future)
┌────────┐ ┌────────┐ ┌────────┐
│ API    │ │ WS     │ │ Store  │
│ Gateway│ │ Server │ │ Service│
└───┬────┘ └───┬────┘ └───┬────┘
    │          │          │
    └──────────┼──────────┘
               │
    ┌──────────▼──────────┐
    │ Service Mesh        │
    │ (Kubernetes)        │
    └─────────────────────┘
"""
```

---

## 学习重点 2：服务职责划分

### 职责划分矩阵

```python
"""
功能                REST API    WebSocket    ExecutionMgr    DatabaseMgr
──────────────────────────────────────────────────────────────────────
创建 Graph          ✓           ✗            ✗               RPC
获取 Graph          ✓           ✗            ✗               RPC
执行 Graph          ✓           ✗            ✓               ✗
订阅执行状态        ✗           ✓            ✗               ✗
推送状态更新        ✗           ✓            ✗               ✗
任务队列消费        ✗           ✗            ✓               ✗
数据库操作          RPC         ✗            RPC             ✓
定时任务            ✗           ✗            ✗               ✗
发送通知            ✗           ✗            ✗               ✗
"""
```

### 服务边界清晰化

```python
# ===== AgentServer (REST API) =====
class AgentServer(AppProcess):
    """
    只负责：
    1. HTTP 请求处理
    2. 参数验证
    3. 权限检查
    4. 业务编排（调用其他服务）
    
    不负责：
    - 数据库直接操作
    - 长时间执行
    - WebSocket 连接
    """
    
    async def create_graph_endpoint(self, graph_data: GraphCreate, user_id: str):
        # 1. 验证（本地）
        if not graph_data.name:
            raise ValueError("Graph name is required")
        
        # 2. 权限检查（本地或通过认证服务）
        if not await check_permission(user_id, "create_graph"):
            raise NotAuthorizedError()
        
        # 3. 数据库操作（通过 DatabaseManager RPC）
        db_client = get_database_manager_client()
        graph = await db_client.create_graph(graph_data, user_id)
        
        # 4. 返回结果
        return graph

# ===== WebsocketServer =====
class WebsocketServer(AppProcess):
    """
    只负责：
    1. WebSocket 连接管理
    2. 订阅/取消订阅
    3. 消息路由和推送
    4. 心跳检测
    
    不负责：
    - 业务逻辑
    - 数据库操作
    - 任务执行
    """
    
    # 只做连接和消息转发
    async def handle_subscribe(self, websocket, message):
        channel_key = await self.connection_manager.subscribe_graph_exec(
            user_id=message.user_id,
            graph_exec_id=message.data["graph_exec_id"],
            websocket=websocket,
        )
        await websocket.send_text({"status": "subscribed", "channel": channel_key})

# ===== ExecutionManager =====
class ExecutionManager(AppProcess):
    """
    只负责：
    1. 消费 RabbitMQ 任务
    2. 执行 Graph
    3. 更新执行状态
    4. 发布事件
    
    不负责：
    - HTTP 请求处理
    - WebSocket 连接
    - 任务创建
    """
    
    def _handle_run_message(self, channel, method, properties, body):
        graph_exec_entry = GraphExecutionEntry.model_validate_json(body)
        
        # 提交到线程池执行
        future = self.executor.submit(
            execute_graph,
            graph_exec_entry,
            cancel_event=threading.Event(),
        )
        
        # 完成后确认消息
        future.add_done_callback(
            lambda f: channel.basic_ack(delivery_tag=method.delivery_tag)
        )

# ===== DatabaseManager =====
class DatabaseManager(AppService):
    """
    只负责：
    1. 数据库 CRUD 操作
    2. 数据验证
    3. 事务管理
    
    不负责：
    - 业务逻辑
    - 权限检查（可选）
    - 任务执行
    """
    
    @expose
    async def get_graph(self, graph_id: str, user_id: str) -> GraphModel:
        # 纯数据访问
        graph = await prisma.agentgraph.find_first(
            where={"id": graph_id, "userId": user_id}
        )
        if not graph:
            raise NotFoundError(f"Graph {graph_id} not found")
        return GraphModel.from_db(graph)
```

---

## 学习重点 3：协调多个服务

### 服务编排模式

```python
"""
服务编排的 3 种模式：

1. 同步编排 (Synchronous Orchestration)
   特点：调用方等待结果
   用途：获取数据、验证
   
2. 异步编排 (Asynchronous Orchestration)
   特点：发送任务后立即返回
   用途：长时间执行任务
   
3. 事件驱动 (Event-Driven)
   特点：发布事件，订阅者自行处理
   用途：状态更新、通知
"""
```

### 同步编排示例

```python
# 场景：创建并执行 Graph

@app.post("/api/graphs/{graph_id}/execute")
async def create_and_execute_graph(
    graph_id: str,
    user_id: str,
    inputs: dict
):
    """
    编排多个服务：
    1. 获取 Graph（DatabaseManager）
    2. 验证权限（本地或认证服务）
    3. 检查余额（CreditManager）
    4. 创建执行（DatabaseManager）
    5. 发布任务（RabbitMQ）
    """
    # 1. 获取 Graph
    db_client = get_database_manager_client()
    graph = await db_client.get_graph(graph_id=graph_id, user_id=user_id)
    if not graph:
        raise NotFoundError("Graph not found")
    
    # 2. 验证权限（简化）
    if not await has_permission(user_id, "execute_graph"):
        raise NotAuthorizedError()
    
    # 3. 检查余额
    credit_service = get_credit_service_client()
    balance = await credit_service.get_balance(user_id)
    if balance < graph.estimated_cost:
        raise InsufficientCreditsError()
    
    # 4. 创建执行记录
    graph_exec = await db_client.create_graph_execution(
        user_id=user_id,
        graph_id=graph_id,
        inputs=inputs,
    )
    
    # 5. 发布到消息队列（异步执行）
    exec_queue = await get_async_execution_queue()
    await exec_queue.publish_message(
        routing_key=GRAPH_EXECUTION_ROUTING_KEY,
        message=graph_exec_entry.model_dump_json(),
        exchange=GRAPH_EXECUTION_EXCHANGE,
    )
    
    # 6. 立即返回
    return {"execution_id": graph_exec.id, "status": "queued"}
```

### 异步编排示例

```python
# 场景：执行完成后的后续处理

async def on_execution_complete(execution_id: str):
    """
    执行完成后的异步处理链
    
    1. 更新统计
    2. 扣除费用
    3. 发送通知
    4. 触发 webhook
    """
    # 1. 获取执行结果
    db_client = get_database_manager_client()
    execution = await db_client.get_graph_execution(execution_id)
    
    # 2. 更新统计（不等待）
    asyncio.create_task(
        update_execution_stats(execution.user_id, execution.graph_id)
    )
    
    # 3. 扣除费用（必须完成）
    credit_service = get_credit_service_client()
    await credit_service.deduct_credits(
        user_id=execution.user_id,
        amount=execution.total_cost,
        reason=f"Execution {execution_id}"
    )
    
    # 4. 发送通知（不等待）
    notification_service = get_notification_manager_client()
    asyncio.create_task(
        notification_service.send_notification(
            user_id=execution.user_id,
            type="execution_complete",
            data={"execution_id": execution_id, "status": execution.status}
        )
    )
    
    # 5. 触发 webhook（不等待）
    asyncio.create_task(
        trigger_webhooks(execution)
    )
```

### 事件驱动示例

```python
# 场景：执行状态变更

# 发布者：ExecutionManager
async def update_execution_status(exec_id: str, status: ExecutionStatus):
    # 1. 更新数据库
    await db.update_graph_execution_stats(exec_id, status=status)
    
    # 2. 发布事件（Redis Pub/Sub）
    event = GraphExecutionEvent(
        id=exec_id,
        status=status,
        event_type=ExecutionEventType.GRAPH_EXEC_UPDATE,
    )
    await event_bus.publish(event)  # 发布即忘记

# 订阅者 1：WebsocketServer
async def event_broadcaster(manager):
    event_queue = AsyncRedisExecutionEventBus()
    async for event in event_queue.listen("*"):
        # 推送给 WebSocket 订阅者
        await manager.send_execution_update(event)

# 订阅者 2：NotificationManager（假设）
async def notification_listener():
    event_queue = AsyncRedisExecutionEventBus()
    async for event in event_queue.listen("*"):
        if event.status == ExecutionStatus.COMPLETED:
            # 发送完成通知
            await send_completion_notification(event.user_id, event.id)

# 订阅者 3：AnalyticsService（假设）
async def analytics_listener():
    event_queue = AsyncRedisExecutionEventBus()
    async for event in event_queue.listen("*"):
        # 收集分析数据
        await record_analytics(event)
```

### 服务协调最佳实践

```python
"""
服务协调最佳实践：

1. 失败处理
   - 同步调用：重试 + 超时
   - 异步调用：死信队列
   - 事件驱动：幂等性

2. 事务管理
   - 避免跨服务事务
   - 使用 Saga 模式
   - 最终一致性

3. 超时控制
   - 每个 RPC 调用设置超时
   - 使用断路器模式
   - 优雅降级

4. 监控和追踪
   - 分布式追踪（Trace ID）
   - 日志关联
   - 指标收集

5. 版本兼容
   - API 向后兼容
   - 消息格式版本化
   - 灰度发布
"""

# 示例：带超时和重试的 RPC 调用
client = get_service_client(
    DatabaseManagerClient,
    call_timeout=30,      # 30秒超时
    request_retry=True    # 自动重试
)

try:
    result = await client.get_graph(graph_id="123", user_id="456")
except TimeoutError:
    # 超时处理
    logger.error("Database service timeout")
    raise ServiceUnavailableError("Database temporarily unavailable")
except HTTPServerError:
    # 5xx 错误（已自动重试）
    logger.error("Database service error after retries")
    raise
```

---

## 双服务器架构总结

```
AutoGPT 双服务器架构特点
│
├─── 🏗️ 架构设计
│    ├─ REST API Server (AgentServer)
│    │   ├─ HTTP 请求处理
│    │   ├─ 业务编排
│    │   └─ 端口: 8006
│    │
│    ├─ WebSocket Server (WebsocketServer)
│    │   ├─ 实时双向通信
│    │   ├─ 订阅管理
│    │   └─ 端口: 8001
│    │
│    └─ 支撑服务
│        ├─ DatabaseManager (8002)
│        ├─ ExecutionManager (消费者)
│        ├─ Scheduler (8003)
│        └─ NotificationManager (8004)
│
├─── 🔗 通信机制
│    ├─ RPC (HTTP) - 同步调用
│    ├─ RabbitMQ - 异步任务
│    ├─ Redis Pub/Sub - 事件广播
│    └─ 共享数据库 - 状态持久化
│
├─── 📡 服务协调
│    ├─ 进程启动顺序管理
│    ├─ 服务发现（配置文件）
│    ├─ 健康检查
│    └─ 优雅关闭
│
├─── 💾 资源管理
│    ├─ 连接池（数据库、Redis、HTTP）
│    ├─ 每进程独立连接
│    └─ 资源清理机制
│
└─── ✨ 设计优势
     ├─ 职责分离清晰
     ├─ 独立扩展
     ├─ 故障隔离
     └─ 易于维护
```

---

# 专题C：事件驱动架构

---

## 核心内容 1：Redis 事件总线

### 事件总线架构

```python
"""
AutoGPT 事件总线设计层次：

1. BaseRedisEventBus (抽象基类)
   - 定义事件总线接口
   - 序列化/反序列化逻辑
   - 通道管理

2. RedisEventBus (同步版本)
   - 同步发布/订阅
   - 用于同步上下文

3. AsyncRedisEventBus (异步版本)
   - 异步发布/订阅
   - 用于异步上下文
   - 支持 wait_for_event

4. 具体实现
   - RedisExecutionEventBus
   - AsyncRedisExecutionEventBus
"""
```

### BaseRedisEventBus 实现

```python
# backend/data/event_bus.py

M = TypeVar("M", bound=BaseModel)  # 事件模型类型

class BaseRedisEventBus(Generic[M], ABC):
    """
    Redis 事件总线基类
    
    特性：
    1. 类型安全（泛型）
    2. 自动序列化/反序列化
    3. 消息大小限制
    4. Unicode 处理
    """
    
    Model: type[M]  # 子类必须指定事件模型
    
    @property
    @abstractmethod
    def event_bus_name(self) -> str:
        """事件总线名称（用作 Redis 频道前缀）"""
        pass
    
    @property
    def Message(self) -> type["_EventPayloadWrapper[M]"]:
        """消息包装器类型"""
        return _EventPayloadWrapper[self.Model]
    
    def _serialize_message(self, item: M, channel_key: str) -> tuple[str, str]:
        """
        序列化消息
        
        流程：
        1. 包装为 _EventPayloadWrapper
        2. 使用 backend.util.json.dumps（处理 datetime）
        3. 检查消息大小
        4. 超限则截断并返回错误消息
        
        返回：(message_json, full_channel_name)
        """
        MAX_MESSAGE_SIZE = config.max_message_size_limit  # 默认 10MB
        
        try:
            # 使用自定义 JSON 编码器（支持 datetime）
            message = json.dumps(
                self.Message(payload=item),
                ensure_ascii=False,
                separators=(",", ":")  # 紧凑格式
            )
        except UnicodeError:
            # Unicode 失败时回退到 ASCII
            message = json.dumps(
                self.Message(payload=item),
                ensure_ascii=True,
                separators=(",", ":")
            )
            logger.warning(
                f"Unicode serialization failed for channel {channel_key}"
            )
        
        # 检查消息大小
        message_size = len(message.encode("utf-8"))
        if message_size > MAX_MESSAGE_SIZE:
            logger.warning(
                f"Message size {message_size} bytes exceeds limit "
                f"{MAX_MESSAGE_SIZE} bytes. Truncating."
            )
            # 返回错误负载
            error_payload = {
                "payload": {
                    "event_type": "error_comms_update",
                    "error": "Payload too large for Redis transmission",
                    "original_size_bytes": message_size,
                    "max_size_bytes": MAX_MESSAGE_SIZE,
                }
            }
            message = json.dumps(error_payload, ensure_ascii=False)
        
        # 构建完整频道名
        channel_name = f"{self.event_bus_name}/{channel_key}"
        logger.debug(f"[{channel_name}] Publishing event: {message}")
        
        return message, channel_name
    
    def _deserialize_message(self, msg: Any, channel_key: str) -> M | None:
        """
        反序列化消息
        
        参数：
        - msg: Redis 消息对象
        - channel_key: 频道键（可能包含 * 通配符）
        
        返回：解析后的事件对象或 None
        """
        # 判断消息类型（普通订阅 vs 模式订阅）
        message_type = "pmessage" if "*" in channel_key else "message"
        if msg["type"] != message_type:
            return None
        
        try:
            logger.debug(f"[{channel_key}] Consuming event: {msg['data']}")
            # 解析 JSON → _EventPayloadWrapper → 提取 payload
            return self.Message.model_validate_json(msg["data"]).payload
        except Exception as e:
            logger.error(f"Failed to parse event from Redis: {msg}, error: {e}")
            return None
    
    def _get_pubsub_channel(
        self,
        connection: redis.Redis | redis.AsyncRedis,
        channel_key: str
    ) -> tuple[PubSub | AsyncPubSub, str]:
        """获取 Pub/Sub 对象和完整频道名"""
        full_channel_name = f"{self.event_bus_name}/{channel_key}"
        pubsub = connection.pubsub()
        return pubsub, full_channel_name


class _EventPayloadWrapper(BaseModel, Generic[M]):
    """
    消息包装器
    
    为什么需要？
    允许 Model 是多个事件类型的联合（discriminated union）
    
    示例：
    ExecutionEvent = GraphExecutionEvent | NodeExecutionEvent
    """
    payload: M
```

### 同步事件总线

```python
class RedisEventBus(BaseRedisEventBus[M], ABC):
    """同步 Redis 事件总线"""
    
    @property
    def connection(self) -> redis.Redis:
        """获取同步 Redis 连接"""
        return redis.get_redis()
    
    def publish_event(self, event: M, channel_key: str):
        """
        发布事件（同步）
        
        使用场景：
        - 同步函数中发布事件
        - 不需要等待发布完成
        """
        message, full_channel_name = self._serialize_message(event, channel_key)
        self.connection.publish(full_channel_name, message)
    
    def listen_events(self, channel_key: str) -> Generator[M, None, None]:
        """
        监听事件（同步生成器）
        
        使用场景：
        - 同步上下文
        - 阻塞式消费
        
        支持：
        - 普通订阅：subscribe("user_123/graph_456/exec_789")
        - 模式订阅：psubscribe("user_123/*/exec_*")
        """
        pubsub, full_channel_name = self._get_pubsub_channel(
            self.connection, channel_key
        )
        assert isinstance(pubsub, PubSub)
        
        # 根据是否有通配符选择订阅方式
        if "*" in channel_key:
            pubsub.psubscribe(full_channel_name)  # 模式订阅
        else:
            pubsub.subscribe(full_channel_name)   # 普通订阅
        
        # 阻塞式监听
        for message in pubsub.listen():
            if event := self._deserialize_message(message, channel_key):
                yield event
```

### 异步事件总线

```python
class AsyncRedisEventBus(BaseRedisEventBus[M], ABC):
    """异步 Redis 事件总线"""
    
    @property
    async def connection(self) -> redis.AsyncRedis:
        """获取异步 Redis 连接"""
        return await redis.get_redis_async()
    
    async def publish_event(self, event: M, channel_key: str):
        """
        发布事件（异步）
        
        使用场景：
        - 异步函数中发布事件
        - 需要确保发布成功
        """
        message, full_channel_name = self._serialize_message(event, channel_key)
        connection = await self.connection
        await connection.publish(full_channel_name, message)
    
    async def listen_events(self, channel_key: str) -> AsyncGenerator[M, None]:
        """
        监听事件（异步生成器）
        
        使用场景：
        - 异步上下文
        - 非阻塞式消费
        
        示例：
        async for event in event_bus.listen_events("*"):
            await process_event(event)
        """
        pubsub, full_channel_name = self._get_pubsub_channel(
            await self.connection, channel_key
        )
        assert isinstance(pubsub, AsyncPubSub)
        
        if "*" in channel_key:
            await pubsub.psubscribe(full_channel_name)
        else:
            await pubsub.subscribe(full_channel_name)
        
        # 异步迭代
        async for message in pubsub.listen():
            if event := self._deserialize_message(message, channel_key):
                yield event
    
    async def wait_for_event(
        self,
        channel_key: str,
        timeout: Optional[float] = None
    ) -> M | None:
        """
        等待单个事件（带超时）
        
        使用场景：
        - 测试
        - 等待特定事件完成
        
        示例：
        event = await event_bus.wait_for_event(
            "user_123/graph_456/exec_789",
            timeout=30.0
        )
        """
        try:
            return await asyncio.wait_for(
                anext(aiter(self.listen_events(channel_key))),
                timeout
            )
        except TimeoutError:
            return None
```

---

## 核心内容 2：发布/订阅模式

### 频道命名规范

```python
"""
AutoGPT 事件频道命名规范：

格式：{event_bus_name}/{user_id}/{graph_id}/{graph_exec_id}

示例：
1. 特定执行的所有事件
   execution_events/user_123/graph_456/exec_789
   
2. 特定图的所有执行
   execution_events/user_123/graph_456/*
   
3. 特定用户的所有事件
   execution_events/user_123/*/*
   
4. 全局所有事件
   execution_events/*/*/*  或  *

支持的模式：
- * : 通配符（匹配单层）
- execution_events/user_*/graph_*/exec_* : 模式订阅
"""
```

### 发布者模式

```python
# 发布者：ExecutionManager, Blocks, etc.

async def publish_execution_event(execution: GraphExecution):
    """
    发布执行事件
    
    步骤：
    1. 更新数据库
    2. 发布事件到 Redis
    """
    # 1. 更新数据库
    await db.update_graph_execution_stats(
        graph_exec_id=execution.id,
        status=execution.status,
    )
    
    # 2. 发布事件
    event_bus = get_async_execution_event_bus()
    await event_bus.publish(execution)
    
# 内部实现
class AsyncRedisExecutionEventBus(AsyncRedisEventBus[ExecutionEvent]):
    async def publish(self, res: GraphExecutionMeta | NodeExecutionResult):
        """统一发布接口"""
        if isinstance(res, GraphExecutionMeta):
            await self._publish_graph_exec_update(res)
        else:
            await self._publish_node_exec_update(res)
    
    async def _publish_graph_exec_update(self, res: GraphExecutionMeta):
        """发布图执行更新"""
        event_data = res.model_dump()
        event_data.setdefault("inputs", {})
        event_data.setdefault("outputs", {})
        event = GraphExecutionEvent.model_validate(event_data)
        
        # 频道：{user_id}/{graph_id}/{exec_id}
        await self._publish(event, f"{res.user_id}/{res.graph_id}/{res.id}")
    
    async def _publish_node_exec_update(self, res: NodeExecutionResult):
        """发布节点执行更新"""
        event = NodeExecutionEvent.model_validate(res.model_dump())
        
        # 频道：{user_id}/{graph_id}/{graph_exec_id}
        await self._publish(
            event,
            f"{res.user_id}/{res.graph_id}/{res.graph_exec_id}"
        )
    
    async def _publish(self, event: ExecutionEvent, channel: str):
        """
        发布事件（带负载截断）
        
        为什么截断？
        - 避免 Redis 消息过大
        - 防止网络传输问题
        - 控制内存使用
        """
        limit = config.max_message_size_limit // 2
        
        if isinstance(event, GraphExecutionEvent):
            event.inputs = truncate(event.inputs, limit)
            event.outputs = truncate(event.outputs, limit)
        elif isinstance(event, NodeExecutionEvent):
            event.input_data = truncate(event.input_data, limit)
            event.output_data = truncate(event.output_data, limit)
        
        # 调用基类的 publish_event
        await self.publish_event(event, channel)
```

### 订阅者模式

```python
# 订阅者 1：WebSocket Server

@continuous_retry()  # 永不停止，自动重试
async def event_broadcaster(manager: ConnectionManager):
    """
    WebSocket 事件广播器
    
    职责：
    1. 监听所有执行事件
    2. 路由到订阅的 WebSocket 连接
    """
    event_queue = AsyncRedisExecutionEventBus()
    
    # 订阅所有事件（"*" 通配符）
    async for event in event_queue.listen("*"):
        # 发送给订阅者
        await manager.send_execution_update(event)

# 订阅者 2：特定用户的事件

async def listen_user_events(user_id: str):
    """监听特定用户的所有事件"""
    event_bus = AsyncRedisExecutionEventBus()
    
    # 订阅：user_id/*/*（所有图和执行）
    async for event in event_bus.listen(user_id, "*", "*"):
        print(f"User {user_id} event: {event.event_type}")
        await process_event(event)

# 订阅者 3：特定执行的事件

async def wait_for_execution_complete(user_id: str, exec_id: str):
    """等待特定执行完成"""
    event_bus = AsyncRedisExecutionEventBus()
    
    # 订阅：user_id/*/{exec_id}
    async for event in event_bus.listen(user_id, "*", exec_id):
        if isinstance(event, GraphExecutionEvent):
            if event.status == ExecutionStatus.COMPLETED:
                print("Execution completed!")
                break
            elif event.status == ExecutionStatus.FAILED:
                print("Execution failed!")
                break
```

### 多订阅者示例

```python
"""
场景：一个事件，多个订阅者

发布：
user_123 执行 graph_456，执行 ID 为 exec_789

频道：execution_events/user_123/graph_456/exec_789

订阅者：
1. WebSocket Server (订阅 "*")
   → 推送给订阅该执行的 WebSocket 客户端

2. Analytics Service (订阅 "*")
   → 收集统计数据

3. Notification Service (订阅 "*")
   → 发送通知（执行完成时）

4. Audit Logger (订阅 "*")
   → 记录审计日志

优点：
- 发布者无需知道订阅者
- 新增订阅者不影响发布者
- 解耦服务
"""

# 发布者（ExecutionManager）
await event_bus.publish(graph_execution)
# ↓ Redis Pub/Sub

# 订阅者 1（WebSocket Server）
async for event in event_bus.listen("*"):
    await websocket_manager.send_to_subscribers(event)

# 订阅者 2（Analytics Service）
async for event in event_bus.listen("*"):
    await analytics.record(event)

# 订阅者 3（Notification Service）
async for event in event_bus.listen("*"):
    if event.status == ExecutionStatus.COMPLETED:
        await send_notification(event.user_id, event)
```

---

## 核心内容 3：事件定义和传递

### 事件模型定义

```python
# backend/data/execution.py

class ExecutionEventType(str, Enum):
    """事件类型枚举"""
    GRAPH_EXEC_UPDATE = "graph_execution_update"    # 图执行更新
    NODE_EXEC_UPDATE = "node_execution_update"      # 节点执行更新
    ERROR_COMMS_UPDATE = "error_comms_update"       # 通信错误


class GraphExecutionEvent(GraphExecution):
    """
    图执行事件
    
    继承自 GraphExecution，增加事件类型标识
    """
    event_type: Literal[ExecutionEventType.GRAPH_EXEC_UPDATE] = (
        ExecutionEventType.GRAPH_EXEC_UPDATE
    )
    
    # 从 GraphExecution 继承的字段：
    # - id: 执行 ID
    # - user_id: 用户 ID
    # - graph_id: 图 ID
    # - graph_version: 图版本
    # - status: 执行状态
    # - inputs: 输入数据
    # - outputs: 输出数据
    # - stats: 统计信息
    # - created_at, updated_at: 时间戳


class NodeExecutionEvent(NodeExecutionResult):
    """
    节点执行事件
    
    继承自 NodeExecutionResult，增加事件类型标识
    """
    event_type: Literal[ExecutionEventType.NODE_EXEC_UPDATE] = (
        ExecutionEventType.NODE_EXEC_UPDATE
    )
    
    # 从 NodeExecutionResult 继承的字段：
    # - user_id, graph_id, graph_exec_id
    # - node_id, node_exec_id
    # - status: 节点状态
    # - input_data: 输入数据
    # - output_data: 输出数据
    # - execution_time: 执行时间


# 联合类型（使用 discriminator）
ExecutionEvent = Annotated[
    GraphExecutionEvent | NodeExecutionEvent,
    Field(discriminator="event_type")  # 根据 event_type 字段区分
]

"""
Discriminated Union 的优势：

1. 类型安全
   Pydantic 根据 event_type 自动选择正确的模型

2. 自动验证
   if event.event_type == "graph_execution_update":
       # Pydantic 确保 event 是 GraphExecutionEvent

3. IDE 支持
   类型提示和自动补全

示例：
event_json = {
    "event_type": "graph_execution_update",
    "id": "exec-123",
    "status": "RUNNING",
    ...
}
event = ExecutionEvent.model_validate(event_json)
# → 自动解析为 GraphExecutionEvent
"""
```

### 事件传递流程

```python
"""
完整的事件传递流程：

┌─────────────────────────────────────────────────────────┐
│ 1. 事件产生（ExecutionManager / Blocks）                │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ 2. 更新数据库                                            │
│    await db.update_graph_execution_stats(...)           │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ 3. 发布到 Redis                                         │
│    event_bus = get_async_execution_event_bus()          │
│    await event_bus.publish(execution)                   │
└────────────────┬────────────────────────────────────────┘
                 │
                 ├──> Redis Pub/Sub
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ 4. Redis 分发给所有订阅者                                │
│    - WebSocket Server                                   │
│    - Analytics Service                                  │
│    - Notification Service                               │
│    - ...                                                │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ 5. 订阅者处理                                            │
│    async for event in event_bus.listen("*"):           │
│        await handle_event(event)                        │
└─────────────────────────────────────────────────────────┘
"""
```

### 事件传递示例

```python
# 示例：Node 执行完成

# 1. ExecutionManager 执行节点
async def execute_node(node, input_data):
    result = await node.execute(input_data)
    
    # 创建执行结果
    node_exec_result = NodeExecutionResult(
        user_id="user-123",
        graph_id="graph-456",
        graph_exec_id="exec-789",
        node_id="node-1",
        node_exec_id="node-exec-1",
        status=ExecutionStatus.COMPLETED,
        input_data=input_data,
        output_data=result,
    )
    
    # 2. 保存到数据库
    await db.create_node_execution_result(node_exec_result)
    
    # 3. 发布事件
    event_bus = get_async_execution_event_bus()
    await event_bus.publish(node_exec_result)
    
    return result

# 4. WebSocket Server 接收
async def event_broadcaster(manager):
    event_bus = AsyncRedisExecutionEventBus()
    async for event in event_bus.listen("*"):
        if isinstance(event, NodeExecutionEvent):
            # 5. 推送给订阅者
            await manager.send_execution_update(event)

# 6. 客户端接收
# WebSocket 消息：
{
    "method": "node_execution_event",
    "channel": "user-123|graph_exec#exec-789",
    "data": {
        "event_type": "node_execution_update",
        "node_id": "node-1",
        "status": "COMPLETED",
        "output_data": {...}
    }
}
```

---

## 核心内容 4：WebSocket 推送集成

### ConnectionManager 与事件总线集成

```python
# backend/server/conn_manager.py

class ConnectionManager:
    async def send_execution_update(
        self,
        exec_event: GraphExecutionEvent | NodeExecutionEvent
    ) -> int:
        """
        发送执行更新到 WebSocket 订阅者
        
        智能路由：
        1. GraphExecutionEvent → 两个频道
           - 单个执行订阅者
           - 所有执行订阅者
        
        2. NodeExecutionEvent → 一个频道
           - 单个执行订阅者
        
        返回：发送的消息数量
        """
        graph_exec_id = (
            exec_event.id
            if isinstance(exec_event, GraphExecutionEvent)
            else exec_event.graph_exec_id
        )
        
        n_sent = 0
        channels: set[str] = {
            # 频道 1：单个执行
            _graph_exec_channel_key(
                exec_event.user_id,
                graph_exec_id=graph_exec_id
            )
        }
        
        if isinstance(exec_event, GraphExecutionEvent):
            # 频道 2：所有执行（只针对 Graph 事件）
            channels.add(
                _graph_execs_channel_key(
                    exec_event.user_id,
                    graph_id=exec_event.graph_id
                )
            )
        
        # 发送给所有相关订阅者
        for channel in channels.intersection(self.subscriptions.keys()):
            message = WSMessage(
                method=_EVENT_TYPE_TO_METHOD_MAP[exec_event.event_type],
                channel=channel,
                data=exec_event.model_dump(),
            ).model_dump_json()
            
            for connection in self.subscriptions[channel]:
                await connection.send_text(message)
                n_sent += 1
        
        return n_sent


# 事件类型 → WebSocket 方法映射
_EVENT_TYPE_TO_METHOD_MAP: dict[ExecutionEventType, WSMethod] = {
    ExecutionEventType.GRAPH_EXEC_UPDATE: WSMethod.GRAPH_EXECUTION_EVENT,
    ExecutionEventType.NODE_EXEC_UPDATE: WSMethod.NODE_EXECUTION_EVENT,
}
```

### 完整的推送流程

```python
"""
从 Redis 事件到 WebSocket 推送的完整流程：

┌─────────────────────────────────────────────────────────────┐
│ ExecutionManager: 发布事件到 Redis                           │
│ await event_bus.publish(graph_execution)                    │
└──────────────────────┬──────────────────────────────────────┘
                       │ Redis Pub/Sub
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ WebSocket Server: event_broadcaster                         │
│ async for event in event_bus.listen("*"):                   │
│     await manager.send_execution_update(event)              │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ ConnectionManager: 路由到订阅者                              │
│ 1. 确定频道（单个执行 / 所有执行）                            │
│ 2. 查找订阅该频道的 WebSocket 连接                          │
│ 3. 发送消息                                                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ WebSocket
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 前端客户端                                                   │
│ ws.onmessage = (event) => {                                 │
│     const msg = JSON.parse(event.data);                     │
│     updateUI(msg.data);                                     │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
"""
```

### 实战示例

```python
# 场景：执行状态更新的实时推送

# 步骤 1：客户端订阅
# JavaScript
const ws = new WebSocket('ws://localhost:8001/ws?token=jwt_token');

ws.onopen = () => {
    // 订阅特定执行
    ws.send(JSON.stringify({
        method: 'subscribe_graph_execution',
        data: {graph_exec_id: 'exec-789'}
    }));
};

ws.onmessage = (event) => {
    const message = JSON.parse(event.data);
    
    if (message.method === 'graph_execution_event') {
        console.log('Graph status:', message.data.status);
        updateProgressBar(message.data);
    }
    
    if (message.method === 'node_execution_event') {
        console.log('Node completed:', message.data.node_id);
        updateNodeStatus(message.data);
    }
};

# 步骤 2：服务端接收订阅
# backend/server/ws_api.py
async def handle_subscribe(manager, websocket, user_id, message):
    sub_req = WSSubscribeGraphExecutionRequest.model_validate(message.data)
    channel_key = await manager.subscribe_graph_exec(
        user_id=user_id,
        graph_exec_id=sub_req.graph_exec_id,
        websocket=websocket,
    )
    # 返回确认
    await websocket.send_text(
        WSMessage(
            method=WSMethod.SUBSCRIBE_GRAPH_EXEC,
            success=True,
            channel=channel_key,
        ).model_dump_json()
    )

# 步骤 3：执行开始
# backend/executor/manager.py
async def start_execution(graph_exec_id):
    graph_exec.status = ExecutionStatus.RUNNING
    await db.update_graph_execution_stats(graph_exec_id, status=ExecutionStatus.RUNNING)
    
    # 发布事件
    await event_bus.publish(graph_exec)
    # → Redis: "execution_events/user-123/graph-456/exec-789"

# 步骤 4：WebSocket Server 转发
# backend/server/ws_api.py
async def event_broadcaster(manager):
    async for event in event_bus.listen("*"):
        # event 是 GraphExecutionEvent
        await manager.send_execution_update(event)
        # → WebSocket: 发送给订阅 "user-123|graph_exec#exec-789" 的连接

# 步骤 5：客户端收到更新
# JavaScript: ws.onmess

让我继续完成剩余内容：

## 🎯 核心内容 5：异步任务触发

### 事件驱动任务触发

```python
# 模式：事件触发后续任务

# 1. 执行完成事件 → 触发后续任务
async def on_graph_execution_complete(event: GraphExecutionEvent):
    if event.status == ExecutionStatus.COMPLETED:
        # 触发：扣费
        await deduct_credits(event.user_id, event.total_cost)
        
        # 触发：发送通知
        await send_notification(event.user_id, "execution_complete")
        
        # 触发：Webhook
        await trigger_webhooks(event)

# 2. 监听模式
async def event_triggered_tasks():
    event_bus = AsyncRedisExecutionEventBus()
    async for event in event_bus.listen("*"):
        if isinstance(event, GraphExecutionEvent):
            if event.status == ExecutionStatus.COMPLETED:
                asyncio.create_task(on_graph_execution_complete(event))
```

---

## 核心内容 6：事件溯源概念

### 事件日志

```python
# AutoGPT 的事件溯源特点：

# 1. 数据库记录 + 事件流
# - 数据库：当前状态
# - Redis事件：状态变更历史（临时）

# 2. 事件重放
async def replay_execution(exec_id: str):
    """通过事件重建执行历史"""
    events = await db.get_execution_events(exec_id)
    for event in events:
        print(f"{event.created_at}: {event.status}")

# 3. 审计追踪
# 每个状态变更都发布事件 → 可追溯
```

---

## 学习重点总结

### 1. 事件驱动设计

```python
"""
核心原则：
✓ 发布者不知道订阅者
✓ 订阅者不知道发布者
✓ 松耦合、高内聚
"""

# 好的设计
await event_bus.publish(event)  # 发布即忘记

# 避免的设计
await notify_websocket(event)  # 紧耦合
await notify_analytics(event)
await notify_audit(event)
```

### 2. 解耦服务

```
事件驱动解耦：

ExecutionManager → Redis → WebSocketServer
                ↓          ↓
            Analytics   Notifications
```

### 3. 实时通知系统

```python
# 实时推送链路
ExecutionManager → Redis Pub/Sub → WebSocket → 前端

# 关键点：
- Redis 作为消息总线
- WebSocket 保持长连接
- ConnectionManager 管理订阅
```

---

