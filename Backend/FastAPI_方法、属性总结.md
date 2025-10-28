## 第1章：路由和端点

### FastAPI 路由（Routes）的属性和方法

FastAPI 中的路由通过应用实例（FastAPI 类）定义，使用装饰器方法来绑定路径和处理函数。每个方法可以搭配各种关键字参数（属性）来定制行为。以下按方法分组，列出常用属性（属性是可选的，但可以根据需要搭配使用，例如 `response_model` 用于指定响应格式，`dependencies` 用于认证）：

- **`app.get(path: str, **kwargs)`**：用于定义 GET 请求的路由，常用属性包括：
  - `path`：必填，路由路径字符串，如 "/items" 或 "/items/{item_id}"（支持路径参数）。
  - `response_model`：可选，指定响应的 Pydantic 模型，用于自动验证和序列化。
  - `status_code`：可选，默认 200，指定 HTTP 状态码。
  - `tags`：可选，列表，用于在 API 文档中分组路由。
  - `summary` 和 `description`：可选，字符串，用于 API 文档描述。
  - `dependencies`：可选，列表，用于注入依赖项（如认证）。
  - 示例：`@app.get("/items", response_model=List[Item], tags=["items"])`。

- **`app.post(path: str, **kwargs)`**：用于定义 POST 请求的路由，常用属性包括：
  - `path`：必填，路由路径字符串。
  - `response_model`：可选，指定响应的模型。
  - `status_code`：可选，默认 201。
  - `tags`、`summary`、`description`、`dependencies`：与 GET 类似。
  - 示例：`@app.post("/items", response_model=Item, status_code=201, dependencies=[Depends(get_current_user)])`。

- **`app.put(path: str, **kwargs)`**：用于定义 PUT 请求的路由，属性与 POST 类似，默认状态码为 204 或 200。
  - `path`：必填。
  - `response_model`、`status_code`（默认 200 或 204）、`tags`、`summary`、`description`、`dependencies`。
  - 示例：`@app.put("/items/{item_id}", response_model=Item, status_code=200)`。

- **`app.delete(path: str, **kwargs)`**：用于定义 DELETE 请求的路由，属性与 GET 类似，默认状态码为 204。
  - `path`：必填。
  - `response_model`（通常无响应体）、`status_code`（默认 204）、`tags`、`summary`、`description`、`dependencies`。
  - 示例：`@app.delete("/items/{item_id}", status_code=204)`。

- **`app.patch(path: str, **kwargs)`**：用于定义 PATCH 请求的路由，属性与 PUT 类似，用于部分更新。
  -  `path`：必填。
  - `response_model`、`status_code`（默认 200）、`tags`、`summary`、`description`、`dependencies`。
  - 示例：`@app.patch("/items/{item_id}", response_model=Item)`。

- **app.options(path: str, **kwargs)**：用于定义 OPTIONS 请求的路由，常用于 CORS 预检，属性与 GET 类似。
  - `path`：必填。
  - `response_model`、`status_code`（默认 200）、`tags`、`summary`、`description`、`dependencies`。

- **app.head(path: str, **kwargs)**：用于定义 HEAD 请求的路由，属性与 GET 类似，但无响应体。
  - `path`：必填。
  - `status_code`（默认 200）、`tags`、`summary`、`description`、`dependencies`。

### FastAPI 端口（Port）的属性和方法

FastAPI 本身不直接管理端口；端口是通过 ASGI 服务器（如 Uvicorn）启动应用时指定的。FastAPI 应用是一个 ASGI 应用对象，可以传递给服务器运行。

- **导入库**：`import uvicorn`（Uvicorn 是推荐的服务器）。
- **`uvicorn.run(app, **kwargs)`**：启动服务器并指定端口，常用属性包括：
  - `app`：必填，FastAPI 应用实例。
  - `host`：可选，默认 "127.0.0.1"，指定主机地址，如 "0.0.0.0"（监听所有接口）。
  - `port`：可选，默认 8000，指定监听的端口号。
  - `reload`：可选，默认 False，布尔值，设置为 True 时在开发模式下自动重载代码。
  - `workers`：可选，默认 1，整数，指定工作进程数，用于生产环境负载均衡。
  - `log_level`：可选，默认 "info"，字符串，设置日志级别，如 "info" 或 "debug"。
  - 示例：`uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)`。

- **其他相关**：端口绑定后，服务器会在指定端口上监听请求。FastAPI 应用本身没有直接的端口属性；所有配置通过服务器传递。也可以使用其他 ASGI 服务器如 Hypercorn 或 Gunicorn，语法类似。

## 第2章：Pydantic 模型集成

| 类别             | 方法/属性                             | 用途               | 搭配属性/说明                      | 示例                                                     |
| ---------------- | ------------------------------------- | ------------------ | ---------------------------------- | -------------------------------------------------------- |
| **导入库**       | `from pydantic import ...`            | 引入所需模块       | BaseModel, Field, ConfigDict       | `from pydantic import BaseModel, Field`                  |
|                  | `from fastapi import ...`             | 应用集成           | FastAPI                            | `from fastapi import FastAPI`                            |
|                  | `from typing import ...`              | 类型注解           | Optional, List                     | `from typing import Optional`                            |
| **模型定义**     | `class ModelName(BaseModel)`          | 创建模型类         | 继承 BaseModel                     | `class User(BaseModel): name: str`                       |
| **字段定义**     | `field_name: Type = Field(...)`       | 定义字段           | gt/le (校验), description, example | `age: int = Field(gt=0, description="年龄")`             |
|                  | `field_name: Optional[Type] = None`   | 可选字段           | 默认值 None                        | `description: Optional[str] = None`                      |
| **实例方法**     | `model.model_dump()`                  | 转换为字典         | include/exclude (控制字段)         | `user.model_dump(include={"name"})`                      |
|                  | `model.model_dump_json()`             | 转换为 JSON 字符串 | indent (格式化)                    | `user.model_dump_json(indent=2)`                         |
| **类方法**       | `Model.model_validate(data)`          | 从数据创建实例     | strict (严格模式)                  | `User.model_validate(data, strict=True)`                 |
|                  | `Model.model_rebuild()`               | 动态重建模型       | 运行时修改                         | `User.model_rebuild()`                                   |
| **在路由中使用** | `@app.post(..., response_model=Item)` | 指定响应模型       | response_model (自动序列化)        | `@app.post("/users", response_model=User)`               |
|                  | `def func(item: Item)`                | 请求体参数         | 模型实例 (自动解析)                | `def create_user(user: User)`                            |
| **配置**         | `model_config = ConfigDict(...)`      | 模型配置           | from_attributes (ORM 集成)         | `model_config = ConfigDict(from_attributes=True)`        |
| **验证器**       | `@field_validator('field')`           | 自定义校验         | cls, v (类和值参数)                | `@field_validator('age') def validate(cls, v): return v` |
| **其他**         | `alias` (Field 属性)                  | 字段别名           | alias="json_name" (映射)           | `Field(alias="user_name")`                               |
|                  | 错误处理                              | 自动 422 错误      | 校验信息                           | FastAPI 自动返回校验详情                                 |

## 第3章：依赖注入系统

**相关导入：**
```python
from fastapi import FastAPI, Depends
```

**核心装饰器/类：**
- `Depends()` - 依赖注入函数

**关键方法和属性：**
- 依赖函数：`get_db()`, `get_current_user()`
- 类作为依赖：`Depends(Pagination)`
- 依赖链：`Depends(get_admin_user)` (依赖其他依赖)
- 依赖缓存：同一请求中的相同依赖只调用一次
- 路由器级别依赖：`APIRouter(dependencies=[...])`
- 全局依赖：`app(dependencies=[...])`
- 依赖覆盖：`app.dependency_overrides[get_db] = mock_db` (测试用)

## 第4章：异步支持

**相关导入：**
```python
from fastapi import FastAPI, BackgroundTasks
from fastapi.responses import StreamingResponse
```

**核心装饰器/类：**
- `async def` - 异步函数定义
- `BackgroundTasks` - 后台任务类

**关键方法和属性：**
- 异步路由：`async def endpoint()` (原生支持异步)
- 后台任务：`background_tasks.add_task(func, *args)`
- 异步依赖：`async def get_db()`
- 并发执行：`asyncio.gather()` (多个异步操作并发)
- 流式响应：`StreamingResponse()` (异步生成器)
- 异步文件上传：`UploadFile` (异步读取)

## 第5章：异常处理与错误响应

**相关导入：**
```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
```

**核心装饰器/类：**
- `HTTPException` - HTTP 异常类
- `@app.exception_handler()` - 异常处理器装饰器

**关键方法和属性：**
- 异常抛出：`raise HTTPException(status_code=404, detail="Not found")`
- 自定义异常处理器：`@app.exception_handler(CustomError)`
- 验证错误处理：`RequestValidationError` (422 状态码)
- 全局异常处理器：捕获所有未处理的异常
- 错误响应格式：`{"detail": "..."}`

## 第6章：中间件系统

**相关导入：**
```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.middleware.cors import CORSMiddleware
```

**核心装饰器/类：**
- `@app.middleware("http")` - HTTP 中间件装饰器
- `BaseHTTPMiddleware` - 基础中间件类
- `CORSMiddleware` - CORS 中间件

**关键方法和属性：**
- 中间件函数：`async def middleware(request, call_next)`
- 请求状态：`request.state` (在中间件间传递数据)
- 响应头处理：`response.headers["X-Custom"] = "value"`
- 洋葱模型：请求/响应按相反顺序执行
- CORS 配置参数：`allow_origins`, `allow_credentials`, `allow_methods`, `allow_headers`

## 第7章：应用生命周期管理

**相关导入：**
```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
```

**核心装饰器/类：**
- `@asynccontextmanager` - 异步上下文管理器装饰器
- `lifespan` 参数 - FastAPI 应用生命周期

**关键方法和属性：**
- 生命周期函数：`lifespan=lifespan_context`
- 启动阶段：`yield` 前的代码
- 关闭阶段：`yield` 后的代码
- 资源管理：数据库连接、缓存等
- 信号处理：`signal.signal(signal.SIGTERM, handler)`

## 第8章：WebSocket 实时通信

**相关导入：**
```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
```

**核心装饰器/类：**
- `@app.websocket()` - WebSocket 端点装饰器
- `WebSocket` - WebSocket 连接对象
- `WebSocketDisconnect` - 断开连接异常

**关键方法和属性：**
- 连接接受：`await websocket.accept()`
- 消息接收：`await websocket.receive_text()`, `await websocket.receive_json()`
- 消息发送：`await websocket.send_text()`, `await websocket.send_json()`
- 关闭连接：`await websocket.close(code, reason)`
- 依赖注入：WebSocket 端点支持 `Depends`

## 第9章：认证和授权

**相关导入：**
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, APIKeyHeader
from typing import Annotated
```

**核心装饰器/类：**
- `OAuth2PasswordBearer` - OAuth2 Bearer token 认证
- `APIKeyHeader` - API Key 认证
- `Security()` - 安全依赖标记

**关键方法和属性：**
- Token 提取：`oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")`
- 认证依赖：`token: str = Depends(oauth2_scheme)`
- 类型注解：`Annotated[User, Security(get_current_user)]`
- 权限检查：在依赖函数中验证角色/权限
- API Key 验证：`APIKeyHeader(name="X-API-Key")`

## 第10章：APIRouter 模块化设计

**相关导入：**
```python
from fastapi import APIRouter
```

**核心装饰器/类：**
- `APIRouter` - 路由器类

**关键方法和属性：**
- 路由器创建：`router = APIRouter(prefix="/users", tags=["users"])`
- 包含路由器：`app.include_router(router, prefix="/api/v1")`
- 挂载独立应用：`app.mount("/sub", sub_app)`
- 路由前缀：`prefix="/users"`
- 文档标签：`tags=["users"]`
- 依赖继承：`dependencies=[Depends(auth)]`

## 第11章：响应模型最佳实践

**相关导入：**
```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
```

**核心装饰器/类：**
- `response_model` 参数 - 响应模型指定

**关键方法和属性：**
- 响应模型：`response_model=UserResponse`
- 字段过滤：
  - `response_model_exclude_unset=True`
  - `response_model_exclude_none=True`
  - `response_model_include={"field1"}`
  - `response_model_exclude={"field1"}`
- 计算字段：`@computed_field` (Pydantic v2)
- 序列化控制：`mode="json"` 强制兼容类型
- 示例数据：`Field(..., examples=[...])`

## 第12章：后台任务与长时间运行

**相关导入：**
```python
from fastapi import FastAPI, BackgroundTasks
```

**核心装饰器/类：**
- `BackgroundTasks` - 后台任务类

**关键方法和属性：**
- 后台任务添加：`background_tasks.add_task(func, *args)`
- 异步任务：`async def background_task()`
- 任务状态管理：状态枚举和存储
- RabbitMQ 任务队列：消息发布和消费
- Redis 状态缓存：任务进度和结果

## 第13章：文档与测试

**相关导入：**
```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
```

**核心装饰器/类：**
- `TestClient` - 测试客户端类

**关键方法和属性：**
- OpenAPI 文档：`/docs`, `/redoc`, `/openapi.json`
- 文档自定义：`title`, `description`, `version`, `docs_url`
- 测试请求：`client.get()`, `client.post()` 等
- 依赖覆盖：`app.dependency_overrides[dep] = mock_dep`
- 断言响应：`assert response.status_code == 200`
- JSON 验证：`response.json()`