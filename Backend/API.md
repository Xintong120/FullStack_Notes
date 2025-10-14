> ！！！**HTTP方法**: GET (只读)、POST (创建)、PUT (更新)、DELETE (删除)

## API开发的核心逻辑模式

### 1. **验证模式 (Validation Pattern)**

**作用**: 确保输入数据的有效性
**常见写法**:

```python
# 参数验证
if not file.filename.endswith('.txt'):
    raise HTTPException(status_code=400, detail="只支持 .txt 文件格式")

if not transcript or len(transcript.strip()) < 50:
    raise HTTPException(status_code=400, detail="Transcript 内容太短")
```

**为什么需要**: 防止无效数据进入系统，避免后续处理出错

### 2. **异常处理模式 (Exception Handling Pattern)**

**作用**: 优雅地处理运行时错误
**常见写法**:

```python
try:
    # 业务逻辑
    result = some_operation()
    return {"success": True, "data": result}
except HTTPException:
    # 重新抛出 HTTP 异常（已知错误）
    raise
except Exception as e:
    # 捕获未知异常，返回统一错误格式
    raise HTTPException(status_code=500, detail=f"处理出错: {str(e)}")
```

**为什么需要**: 网络调用、文件操作、数据库查询都可能失败

### 3. **条件返回模式 (Conditional Return Pattern)**

**作用**: 根据不同情况返回不同结果
**常见写法**:

```python
# 检查任务是否存在
task = task_manager.get_task(task_id)
if not task:
    raise HTTPException(status_code=404, detail="Task not found")

# 检查记录是否存在
if record is None:
    return {"success": False, "error": "Record not found"}
```

**为什么需要**: 业务逻辑通常涉及多种状态和条件

### 4. **资源管理模式 (Resource Management Pattern)**

**作用**: 确保资源正确分配和释放
**常见写法**:

```python
# 文件处理
with tempfile.NamedTemporaryFile(mode='wb', delete=False, suffix='.txt') as temp_file:
    content = await file.read()
    temp_file.write(content)
    temp_file_path = temp_file.name

# 确保临时文件被删除
os.unlink(temp_file_path)
```

**为什么需要**: 文件、网络连接、数据库连接都需要正确管理

### 5. **数据转换模式 (Data Transformation Pattern)**

**作用**: 将内部数据转换为API响应格式
**常见写法**:

```python
# 从原始数据转换为API格式
candidates = []
for c in candidates_raw:
    candidates.append(TEDCandidate(
        title=c.get("title", ""),
        speaker=c.get("speaker", "Unknown"),
        url=c.get("url", ""),
        # ... 其他字段
    ))
```

**为什么需要**: 内部数据结构可能与API接口不匹配

## 为什么API逻辑如此模式化？

### 1. **HTTP协议的约束**

- 请求必须有响应
- 错误需要特定状态码
- 数据必须序列化

### 2. **可靠性要求**

- 不能因为一个错误让整个系统崩溃
- 需要处理各种异常情况
- 必须验证输入数据的有效性

### 3. **可维护性需求**

- 统一的错误处理格式
- 一致的响应结构
- 标准化的验证逻辑

### 4. **安全性考虑**

- 验证所有输入
- 控制错误信息泄露
- 防止资源耗尽攻击

## 如何优化这些重复模式？

### 1. **创建通用工具函数**

```python
def validate_file_type(filename: str, allowed_extensions: list) -> bool:
    return any(filename.endswith(ext) for ext in allowed_extensions)

def handle_api_errors(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except HTTPException:
            raise
        except Exception as e:
            logger.error(f"API error: {e}")
            raise HTTPException(status_code=500, detail="Internal server error")
    return wrapper
```

### 2. **使用装饰器简化代码**

```python
@app.post("/process-file")
@handle_api_errors
async def process_file(file: UploadFile):
    # 业务逻辑，无需try-except
    pass
```

### 3. **创建响应格式化器**

```python
def api_response(success: bool, data=None, error=None, **kwargs):
    return {
        "success": success,
        "data": data,
        "error": error,
        **kwargs
    }
```



## 后端API的分类

### 1. **系统级API (System APIs)**
**作用**: 提供系统功能，不涉及具体业务逻辑
**特点**: 全局可用，参数简单
**例子**:

```python
@app.get("/health")           # 健康检查
@app.get("/test-groq")        # 测试连接
```

用户想了解系统级API的开发步骤和考虑点。让我基于他的项目中的健康检查API来详细解释。

**Step 1: 确定API需求和作用**

**思考点**:

- 这个API解决什么问题？
- 谁会使用它？(开发、运维、监控系统)
- 调用频率如何？(高频/低频)
- 是否需要认证？(通常不需要)

**以健康检查API为例**:

```python
@app.get("/health")
def health_check():
    return {
        "status": "ok",
        "model": settings.model_name,
        "temperature": settings.temperature
    }
```

**Step 2: 设计API接口**

**考虑点**:
- **HTTP方法**: GET (只读)、POST (创建)、PUT (更新)、DELETE (删除)
- **路径设计**: RESTful风格，简洁明了
- **响应格式**: JSON，统一格式
- **状态码**: 200成功、4xx客户端错误、5xx服务器错误

**设计决策**:
```python
# 路径: /health - 简洁明了
# 方法: GET - 只读操作
# 无参数 - 系统级API通常不需要用户输入
# 返回: JSON对象包含系统状态
```

**Step 3: 实现业务逻辑**

**考虑点**:

- **依赖关系**: 需要访问哪些配置、服务？
- **异常处理**: 如何处理可能的错误？
- **性能影响**: 是否会影响系统性能？
- **信息安全**: 返回哪些信息？是否包含敏感数据？

**Step 4: 添加错误处理**

**考虑点**:
- **配置缺失**: 如果settings未初始化怎么办？
- **服务不可用**: 如果依赖的服务宕机怎么办？
- **部分失败**: 某些检查失败但整体还能工作怎么办？

**改进的健康检查**:
```python
@app.get("/health")
def health_check():
    try:
        # 检查配置是否加载
        if not settings.model_name:
            return {
                "status": "error",
                "message": "Configuration not loaded",
                "timestamp": time.time()
            }
        
        # 检查数据库连接 (如果有的话)
        # 检查外部服务连接
        
        return {
            "status": "ok",
            "model": settings.model_name,
            "temperature": settings.temperature,
            "timestamp": time.time()
        }
    except Exception as e:
        return {
            "status": "error", 
            "message": str(e),
            "timestamp": time.time()
        }
```

**Step 5: 添加文档和类型注解**

**考虑点**:
- **API文档**: 自动生成Swagger文档
- **类型安全**: 使用Pydantic模型
- **使用说明**: 谁可以用？什么时候用？

**完整的实现**:
```python
from pydantic import BaseModel
from typing import Optional

class HealthResponse(BaseModel):
    status: str  # "ok" | "error" | "warning"
    model: Optional[str] = None
    temperature: Optional[float] = None
    timestamp: float
    message: Optional[str] = None

@app.get("/health", response_model=HealthResponse)
def health_check():
    """
    系统健康检查接口
    
    用于监控系统状态，检查配置是否正确加载
    
    返回:
        - status: 系统状态 (ok/error/warning)
        - model: AI模型名称
        - temperature: 模型温度参数
        - timestamp: 检查时间戳
        - message: 可选的错误信息
    """
    try:
        if not settings.model_name:
            raise ValueError("Model name not configured")
            
        return HealthResponse(
            status="ok",
            model=settings.model_name,
            temperature=settings.temperature,
            timestamp=time.time()
        )
    except Exception as e:
        return HealthResponse(
            status="error",
            message=str(e),
            timestamp=time.time()
        )
```

**Step 6: 测试API**

**考虑点**:
- **正常情况**: 所有配置正确
- **异常情况**: 配置缺失、服务不可用
- **边界情况**: 网络超时、权限问题

**测试用例**:
```python
# 测试正常情况
def test_health_check_normal():
    response = client.get("/health")
    assert response.status_code == 200
    data = response.json()
    assert data["status"] == "ok"
    assert "model" in data
    assert "timestamp" in data

# 测试配置缺失
def test_health_check_config_missing():
    # 模拟配置缺失
    settings.model_name = None
    response = client.get("/health")
    assert response.status_code == 200
    data = response.json()
    assert data["status"] == "error"
    assert "message" in data
```

**Step 7: 部署和监控**

**考虑点**:
- **监控集成**: Prometheus指标
- **日志记录**: 记录检查结果
- **告警设置**: 当健康检查失败时如何告警

**监控增强**:
```python
from prometheus_client import Gauge, Counter

# Prometheus指标
health_checks_total = Counter('health_checks_total', 'Total health checks', ['status'])
health_check_duration = Gauge('health_check_duration_seconds', 'Health check duration')

@app.get("/health")
def health_check():
    start_time = time.time()
    
    try:
        # 检查逻辑...
        result = {"status": "ok", ...}
        
        health_checks_total.labels(status="success").inc()
        health_check_duration.set(time.time() - start_time)
        
        return result
    except Exception as e:
        health_checks_total.labels(status="error").inc()
        health_check_duration.set(time.time() - start_time)
        
        # 记录错误日志
        logger.error(f"Health check failed: {e}")
        
        return {"status": "error", "message": str(e)}
```

**Step 8: 文档和维护**

**考虑点**:
- **更新文档**: 当API发生变化时更新说明
- **版本控制**: 如果需要向后兼容
- **弃用策略**: 如果要废弃某个API



### 2. **业务处理API (Business Processing APIs)**

**作用**: 处理核心业务逻辑
**特点**: 调用复杂的业务流程，可能耗时较长
**例子**:

```python
@app.post("/process-file")    # TED文件处理
@app.post("/search-ted")      # TED搜索
@app.post("/process-batch")   # 批量处理
```

以电商平台的"创建订单"API为例：

**Step 1: 需求分析阶段**

**思考点**：
- **业务目标**: 用户提交购物车商品，创建订单
- **关键流程**: 库存检查 → 价格计算 → 支付处理 → 订单创建 → 库存扣减
- **失败处理**: 库存不足怎么办？支付失败怎么办？
- **并发问题**: 多人同时购买同一商品怎么办？

**API定义**：
```python
POST /api/orders
# 创建新订单
```

**Step 2: 数据模型设计**

**考虑点**：
- **输入数据**: 用户ID、商品列表、收货地址、支付方式
- **输出数据**: 订单号、订单状态、预计送达时间
- **验证规则**: 商品是否存在？库存是否足够？地址是否有效？

```python
# 输入模型
class CreateOrderRequest(BaseModel):
    user_id: int
    items: List[OrderItem]
    shipping_address: Address
    payment_method: PaymentMethod

# 输出模型  
class CreateOrderResponse(BaseModel):
    order_id: str
    status: str  # "pending", "confirmed", "failed"
    total_amount: float
    estimated_delivery: datetime
```

**Step 3: 业务逻辑设计**

**考虑点**：
- **原子性**: 要么全部成功，要么全部回滚
- **数据一致性**: 库存扣减和订单创建必须同时成功
- **性能**: 大量商品的库存检查如何优化？
- **扩展性**: 新增促销逻辑、积分抵扣等怎么办？

**业务流程**：
```
1. 验证用户和商品
2. 检查库存可用性  
3. 计算订单金额（包含促销）
4. 创建订单记录
5. 扣减库存
6. 生成支付链接
7. 发送确认邮件
```

**Step 4: 实现API逻辑**

```python
from fastapi import APIRouter, HTTPException, Depends
from sqlalchemy.orm import Session
from app.database import get_db
from app.services import OrderService, InventoryService, PaymentService
from app.models import CreateOrderRequest, CreateOrderResponse

router = APIRouter()

@router.post("/orders", response_model=CreateOrderResponse)
async def create_order(
    request: CreateOrderRequest,
    db: Session = Depends(get_db)
):
    """
    创建新订单 - 核心业务API
    
    处理完整的订单创建流程：
    1. 验证商品和库存
    2. 计算订单金额
    3. 创建订单记录
    4. 处理支付
    5. 发送通知
    """
    
    # Step 4.1: 输入验证
    if not request.items:
        raise HTTPException(400, "订单至少需要一件商品")
    
    # Step 4.2: 库存检查
    inventory_service = InventoryService(db)
    for item in request.items:
        available = await inventory_service.check_availability(item.product_id, item.quantity)
        if not available:
            raise HTTPException(400, f"商品 {item.product_id} 库存不足")
    
    # Step 4.3: 订单服务处理
    order_service = OrderService(db)
    
    try:
        # 使用数据库事务确保原子性
        async with db.begin():
            # 创建订单
            order = await order_service.create_order(
                user_id=request.user_id,
                items=request.items,
                shipping_address=request.shipping_address
            )
            
            # 扣减库存
            await inventory_service.reduce_stock(request.items)
            
            # 处理支付
            payment_service = PaymentService()
            payment_result = await payment_service.create_payment(
                order_id=order.id,
                amount=order.total_amount,
                method=request.payment_method
            )
            
            # 更新订单状态
            await order_service.update_payment_status(order.id, payment_result.status)
            
        # 事务外操作：发送邮件（失败不影响订单）
        try:
            await send_order_confirmation_email(order)
        except Exception as e:
            logger.warning(f"邮件发送失败: {e}")
        
        return CreateOrderResponse(
            order_id=str(order.id),
            status=order.status,
            total_amount=order.total_amount,
            estimated_delivery=order.estimated_delivery
        )
        
    except Exception as e:
        # Step 4.4: 错误处理和回滚
        logger.error(f"订单创建失败: {e}")
        
        # 如果是库存不足等业务错误
        if "库存" in str(e):
            raise HTTPException(400, str(e))
        # 如果是系统错误
        else:
            raise HTTPException(500, "订单创建失败，请稍后重试")
```

**Step 5: 错误处理和边界情况**

```python
# 扩展错误处理
@router.post("/orders")
async def create_order(request: CreateOrderRequest, db: Session = Depends(get_db)):
    try:
        # 主要业务逻辑...
        
    except InventoryError as e:
        # 库存相关错误
        raise HTTPException(409, f"库存不足: {e.product_name}")
        
    except PaymentError as e:
        # 支付相关错误
        raise HTTPException(402, f"支付失败: {e.message}")
        
    except UserNotFoundError:
        # 用户不存在
        raise HTTPException(404, "用户不存在")
        
    except ValidationError as e:
        # 数据验证错误
        raise HTTPException(422, f"数据验证失败: {e.errors()}")
        
    except Exception as e:
        # 未知系统错误
        logger.critical(f"意外错误: {e}", exc_info=True)
        raise HTTPException(500, "系统繁忙，请稍后重试")
```

**Step 6: 性能优化**

```python
# 添加缓存和优化
from app.cache import RedisCache

@router.post("/orders") 
async def create_order(
    request: CreateOrderRequest,
    db: Session = Depends(get_db),
    cache: RedisCache = Depends(get_cache)
):
    # 缓存用户地址信息
    cached_address = await cache.get(f"user_address:{request.user_id}")
    if cached_address:
        request.shipping_address = cached_address
    
    # 批量库存检查优化
    product_ids = [item.product_id for item in request.items]
    stock_levels = await inventory_service.batch_check_stock(product_ids)
    
    # 批量价格查询
    prices = await cache.mget([f"price:{pid}" for pid in product_ids])
```

**Step 7: 并发控制**

```python
# 处理并发购买问题
from redis import Redis
import asyncio

redis = Redis()

@router.post("/orders")
async def create_order(request: CreateOrderRequest):
    # 使用Redis分布式锁防止重复下单
    lock_key = f"order_lock:{request.user_id}"
    
    if not redis.set(lock_key, "1", ex=30, nx=True):  # 30秒超时，不存在时设置
        raise HTTPException(429, "正在处理其他订单，请稍后重试")
    
    try:
        # 业务逻辑...
        pass
    finally:
        # 释放锁
        redis.delete(lock_key)
```

**Step 8: 监控和日志**

```python
from app.monitoring import metrics, logger

@router.post("/orders")
async def create_order(request: CreateOrderRequest):
    start_time = time.time()
    
    try:
        # 记录请求开始
        logger.info(f"开始创建订单: user={request.user_id}, items={len(request.items)}")
        
        # 业务逻辑...
        
        # 记录成功
        duration = time.time() - start_time
        metrics.order_creation_success.inc()
        metrics.order_creation_duration.observe(duration)
        
        logger.info(f"订单创建成功: order_id={order.id}, duration={duration:.2f}s")
        
        return response
        
    except Exception as e:
        # 记录失败
        duration = time.time() - start_time
        metrics.order_creation_failure.inc()
        metrics.order_creation_duration.observe(duration)
        
        logger.error(f"订单创建失败: user={request.user_id}, error={e}, duration={duration:.2f}s")
        
        raise
```

**Step 9: 测试策略**

```python
# 单元测试
def test_create_order_success():
    # 测试正常下单流程
    pass

def test_create_order_insufficient_stock():
    # 测试库存不足情况
    pass

def test_create_order_payment_failed():
    # 测试支付失败情况
    pass

# 集成测试
def test_create_order_full_flow():
    # 测试完整流程，包括数据库变更
    pass

# 性能测试
def test_create_order_concurrent():
    # 测试并发下单场景
    pass
```

**开发业务处理API的核心考虑点：**

1. **业务复杂度**: 需要理解完整业务流程
2. **数据一致性**: 确保多步骤操作的原子性
3. **错误处理**: 区分不同类型的错误
4. **性能优化**: 缓存、批量操作、异步处理
5. **并发控制**: 防止竞态条件
6. **监控日志**: 可观测性和调试支持
7. **测试覆盖**: 各种边界情况和异常场景

这就是业务处理API的完整开发流程！这种API通常是最复杂但也最重要的，因为它们承载了核心业务价值。

### 3. **实时通信API (Real-time APIs)**

**作用**: 提供实时数据推送
**特点**: 使用WebSocket协议，异步通信
**例子**:

```python
@app.websocket("/ws/progress/{task_id}")  # 进度推送
```

以聊天应用中的实时消息API为例

**Step 1: 需求分析**

**思考点**：
- **实时性要求**: 消息需要立即送达，还是允许延迟？
- **连接管理**: 用户可能同时打开多个标签页怎么办？
- **离线处理**: 用户断网后重新连接怎么办？
- **扩展性**: 未来需要支持群聊、一对一、私有频道吗？

**以聊天应用为例**：
- 用户可以加入聊天室
- 实时接收其他用户的消息
- 显示在线用户列表
- 支持私聊功能

**Step 2: 协议选择和连接管理**

**WebSocket vs 其他方案**：
```python
# WebSocket - 全双工通信，适合实时聊天
@app.websocket("/chat/{room_id}")

# SSE (Server-Sent Events) - 单向推送，适合通知
@app.get("/notifications/stream")

# Long Polling - 兼容性好，但效率低
@app.get("/messages/poll")
```

**Step 3: 连接生命周期管理**

```python
from fastapi import WebSocket, WebSocketDisconnect
from typing import Dict, Set
import json

# 全局连接管理
class ConnectionManager:
    def __init__(self):
        # room_id -> set of WebSocket connections
        self.active_connections: Dict[str, Set[WebSocket]] = {}
        
        # user_id -> WebSocket connection (for private messages)
        self.user_connections: Dict[str, WebSocket] = {}
    
    async def connect(self, websocket: WebSocket, room_id: str, user_id: str):
        await websocket.accept()
        
        # 添加到房间连接
        if room_id not in self.active_connections:
            self.active_connections[room_id] = set()
        self.active_connections[room_id].add(websocket)
        
        # 记录用户连接（用于私聊）
        self.user_connections[user_id] = websocket
        
        # 广播用户加入消息
        await self.broadcast_to_room(room_id, {
            "type": "user_joined",
            "user_id": user_id,
            "timestamp": "2025-01-01T10:00:00Z"
        })
    
    async def disconnect(self, websocket: WebSocket, room_id: str, user_id: str):
        # 从房间连接中移除
        if room_id in self.active_connections:
            self.active_connections[room_id].discard(websocket)
            
            # 如果房间没人了，清理房间
            if not self.active_connections[room_id]:
                del self.active_connections[room_id]
        
        # 清理用户连接
        if user_id in self.user_connections:
            del self.user_connections[user_id]
    
    async def broadcast_to_room(self, room_id: str, message: dict):
        """广播消息给房间内所有用户"""
        if room_id not in self.active_connections:
            return
            
        disconnected = set()
        for connection in self.active_connections[room_id]:
            try:
                await connection.send_json(message)
            except Exception:
                disconnected.add(connection)
        
        # 清理断开的连接
        for conn in disconnected:
            self.active_connections[room_id].discard(conn)
    
    async def send_to_user(self, user_id: str, message: dict):
        """发送私聊消息给特定用户"""
        if user_id in self.user_connections:
            try:
                await self.user_connections[user_id].send_json(message)
            except Exception:
                # 用户可能已断开，清理连接
                del self.user_connections[user_id]

# 创建全局管理器
manager = ConnectionManager()
```

**Step 4: 消息协议设计**

**考虑点**：
- **消息类型**: 聊天消息、系统通知、用户状态等
- **消息格式**: JSON结构，包含类型、内容、时间戳
- **验证机制**: 防止恶意消息和DOS攻击

```python
# 消息类型枚举
class MessageType(str, Enum):
    CHAT_MESSAGE = "chat_message"
    USER_JOINED = "user_joined"
    USER_LEFT = "user_left"
    PRIVATE_MESSAGE = "private_message"
    SYSTEM_NOTIFICATION = "system_notification"
    TYPING_INDICATOR = "typing_indicator"

# 消息模型
class ChatMessage(BaseModel):
    type: MessageType
    content: str
    sender_id: str
    sender_name: str
    room_id: Optional[str] = None
    recipient_id: Optional[str] = None  # 私聊时使用
    timestamp: datetime = Field(default_factory=datetime.utcnow)

# 心跳消息（保持连接活跃）
class PingMessage(BaseModel):
    type: str = "ping"
    timestamp: datetime = Field(default_factory=datetime.utcnow)
```

**Step 5: WebSocket端点实现** 

```python
@app.websocket("/chat/{room_id}")
async def chat_endpoint(websocket: WebSocket, room_id: str, user_id: str, user_name: str):
    """
    聊天室WebSocket端点
    
    支持功能：
    - 实时聊天消息
    - 用户加入/离开通知
    - 私聊功能
    - 输入状态指示器
    - 心跳检测
    """
    
    # Step 5.1: 连接建立和验证
    try:
        # 验证用户身份（可集成JWT或其他认证）
        if not is_valid_user(user_id):
            await websocket.close(code=4001, reason="Invalid user")
            return
            
        # 验证房间权限
        if not can_access_room(user_id, room_id):
            await websocket.close(code=4002, reason="Access denied")
            return
            
        # 建立连接
        await manager.connect(websocket, room_id, user_id)
        
        print(f"用户 {user_name} 加入房间 {room_id}")
        
    except Exception as e:
        print(f"连接建立失败: {e}")
        return
    
    # Step 5.2: 消息处理循环
    try:
        while True:
            try:
                # 接收客户端消息（设置超时防止阻塞）
                data = await asyncio.wait_for(
                    websocket.receive_json(), 
                    timeout=60.0  # 60秒超时
                )
                
                message = ChatMessage(**data)
                
                # Step 5.3: 消息类型处理
                if message.type == MessageType.CHAT_MESSAGE:
                    # 广播聊天消息
                    broadcast_message = {
                        "type": "chat_message",
                        "content": message.content,
                        "sender_id": message.sender_id,
                        "sender_name": message.sender_name,
                        "timestamp": message.timestamp.isoformat()
                    }
                    await manager.broadcast_to_room(room_id, broadcast_message)
                    
                elif message.type == MessageType.PRIVATE_MESSAGE:
                    # 发送私聊消息
                    if message.recipient_id:
                        private_message = {
                            "type": "private_message",
                            "content": message.content,
                            "sender_id": message.sender_id,
                            "sender_name": message.sender_name,
                            "timestamp": message.timestamp.isoformat()
                        }
                        await manager.send_to_user(message.recipient_id, private_message)
                        
                elif message.type == MessageType.TYPING_INDICATOR:
                    # 广播正在输入状态
                    typing_message = {
                        "type": "typing_indicator",
                        "user_id": message.sender_id,
                        "user_name": message.sender_name,
                        "is_typing": message.content.lower() == "true"
                    }
                    await manager.broadcast_to_room(room_id, typing_message)
                    
                elif message.type == MessageType.PING:
                    # 响应心跳
                    await websocket.send_json({
                        "type": "pong",
                        "timestamp": datetime.utcnow().isoformat()
                    })
                    
            except asyncio.TimeoutError:
                # 客户端60秒内无消息，发送ping检测连接
                await websocket.send_json({
                    "type": "ping",
                    "timestamp": datetime.utcnow().isoformat()
                })
                
            except json.JSONDecodeError:
                # 无效JSON格式
                await websocket.send_json({
                    "type": "error",
                    "message": "Invalid message format"
                })
                
    except WebSocketDisconnect:
        # Step 5.4: 连接断开处理
        print(f"用户 {user_name} 断开连接")
        
        # 广播用户离开消息
        await manager.broadcast_to_room(room_id, {
            "type": "user_left",
            "user_id": user_id,
            "user_name": user_name,
            "timestamp": datetime.utcnow().isoformat()
        })
        
        # 清理连接
        await manager.disconnect(websocket, room_id, user_id)
        
    except Exception as e:
        print(f"WebSocket错误: {e}")
        await manager.disconnect(websocket, room_id, user_id)
```

**Step 6: 错误处理和边界情况**

```python
# 自定义WebSocket异常
class WebSocketError(Exception):
    def __init__(self, code: int, reason: str):
        self.code = code
        self.reason = reason

# 改进的连接验证
async def validate_connection(user_id: str, room_id: str) -> dict:
    """验证用户和房间权限"""
    user = await get_user(user_id)
    if not user:
        raise WebSocketError(4001, "User not found")
    
    room = await get_room(room_id)
    if not room:
        raise WebSocketError(4002, "Room not found")
    
    if not await check_room_access(user_id, room_id):
        raise WebSocketError(4003, "Access denied")
    
    return {"user": user, "room": room}
```

**Step 7: 性能优化**

```python
# 连接池管理
class OptimizedConnectionManager:
    def __init__(self):
        self.connections: Dict[str, Set[WebSocket]] = {}
        self.user_connections: Dict[str, WebSocket] = {}
        
        # 添加连接限制
        self.max_connections_per_room = 100
        self.max_connections_per_user = 5
        
        # 消息队列（防止消息风暴）
        self.message_queues: Dict[str, asyncio.Queue] = {}
        
    async def broadcast_to_room(self, room_id: str, message: dict):
        """优化后的广播，带队列和限流"""
        if room_id not in self.connections:
            return
            
        # 使用队列防止消息积压
        if room_id not in self.message_queues:
            self.message_queues[room_id] = asyncio.Queue(maxsize=100)
            # 启动消息处理任务
            asyncio.create_task(self._process_room_messages(room_id))
        
        try:
            await asyncio.wait_for(
                self.message_queues[room_id].put(message),
                timeout=1.0
            )
        except asyncio.TimeoutError:
            print(f"消息队列已满: {room_id}")
    
    async def _process_room_messages(self, room_id: str):
        """异步处理房间消息"""
        queue = self.message_queues[room_id]
        
        while True:
            try:
                message = await queue.get()
                await self._send_to_room(room_id, message)
                queue.task_done()
            except Exception as e:
                print(f"消息处理错误: {e}")
```

**Step 8: 监控和可观测性**

```python
from app.monitoring import websocket_metrics, logger

class MonitoredConnectionManager(ConnectionManager):
    async def connect(self, websocket: WebSocket, room_id: str, user_id: str):
        await super().connect(websocket, room_id, user_id)
        
        # 记录连接指标
        websocket_metrics.active_connections.inc()
        websocket_metrics.connections_per_room.labels(room_id).inc()
        
        logger.info(f"WebSocket连接建立: user={user_id}, room={room_id}")
    
    async def disconnect(self, websocket: WebSocket, room_id: str, user_id: str):
        await super().disconnect(websocket, room_id, user_id)
        
        # 更新指标
        websocket_metrics.active_connections.dec()
        websocket_metrics.connections_per_room.labels(room_id).dec()
        
        logger.info(f"WebSocket连接断开: user={user_id}, room={room_id}")
    
    async def broadcast_to_room(self, room_id: str, message: dict):
        start_time = time.time()
        
        try:
            await super().broadcast_to_room(room_id, message)
            
            # 记录广播指标
            duration = time.time() - start_time
            websocket_metrics.broadcast_duration.observe(duration)
            websocket_metrics.messages_broadcasted.inc()
            
        except Exception as e:
            websocket_metrics.broadcast_errors.inc()
            logger.error(f"广播失败: room={room_id}, error={e}")
            raise
```

**Step 9: 测试策略**

```python
import pytest
from fastapi.testclient import TestClient
import websockets
import asyncio

# 单元测试
def test_connection_manager():
    manager = ConnectionManager()
    # 测试连接建立、断开、广播功能
    pass

# 集成测试
@pytest.mark.asyncio
async def test_websocket_chat():
    """测试WebSocket聊天功能"""
    async with websockets.connect("ws://localhost:8000/chat/test_room?user_id=1&user_name=test") as ws:
        # 发送聊天消息
        await ws.send_json({
            "type": "chat_message",
            "content": "Hello World",
            "sender_id": "1",
            "sender_name": "test"
        })
        
        # 接收广播消息
        response = await ws.recv()
        data = json.loads(response)
        
        assert data["type"] == "chat_message"
        assert data["content"] == "Hello World"

# 负载测试
def test_websocket_concurrency():
    """测试高并发连接"""
    # 使用locust或类似工具进行压力测试
    pass
```

**实时通信API的核心考虑点**：

1. **连接管理**: 建立、断开、清理无效连接
2. **消息协议**: 统一的JSON消息格式和类型
3. **错误处理**: 网络异常、消息格式错误、权限验证
4. **性能优化**: 连接池、消息队列、限流保护
5. **监控日志**: 连接数、消息量、错误率统计
6. **安全性**: 认证、授权、消息验证、DOS防护



### 4. **数据查询API (Data Query APIs)**

**作用**: 查询和获取数据
**特点**: 只读操作，快速响应
**例子**:
```python
@app.get("/task/{task_id}")                # 任务状态查询
@app.get("/api/memory/ted-history")        # TED历史
@app.get("/api/memory/search-history")     # 搜索历史
@app.get("/api/learning/records")          # 学习记录
```

以电商平台的商品列表API为例

**Step 1: 需求分析阶段**

**思考点**：
- **查询场景**: 什么数据需要查询？用户查看商品列表
- **查询条件**: 支持按分类、价格区间、关键词搜索
- **分页需求**: 数据量大时如何分页返回
- **排序选项**: 按价格、销量、上架时间等排序
- **性能要求**: 查询速度必须快，用户体验至上

**API定义**：
```python
GET /api/products
# 查询商品列表，支持筛选、排序、分页
```

**Step 2: 查询参数设计**

**考虑点**：
- **必需参数**: 分页参数（page, size）
- **可选参数**: 筛选条件、排序方式
- **参数验证**: 防止无效参数、SQL注入
- **默认值**: 合理的默认行为

```python
from pydantic import BaseModel, Field
from typing import Optional, List
from enum import Enum

class SortOrder(str, Enum):
    ASC = "asc"
    DESC = "desc"

class ProductSortBy(str, Enum):
    PRICE = "price"
    CREATED_AT = "created_at"
    SALES = "sales"
    RATING = "rating"

class ProductFilters(BaseModel):
    category_id: Optional[int] = None
    min_price: Optional[float] = Field(None, ge=0)
    max_price: Optional[float] = Field(None, gt=0)
    brand: Optional[str] = None
    in_stock: Optional[bool] = None
    tags: Optional[List[str]] = None

class ProductQuery(BaseModel):
    # 分页参数
    page: int = Field(1, ge=1)
    size: int = Field(20, ge=1, le=100)
    
    # 搜索参数
    q: Optional[str] = None  # 关键词搜索
    
    # 筛选参数
    filters: ProductFilters = Field(default_factory=ProductFilters)
    
    # 排序参数
    sort_by: ProductSortBy = ProductSortBy.CREATED_AT
    sort_order: SortOrder = SortOrder.DESC
```

**Step 3: 数据模型设计**

**考虑点**：
- **响应格式**: 统一的JSON结构
- **数据字段**: 返回必要信息，避免过度获取
- **分页信息**: 总记录数、当前页、总页数
- **关联数据**: 是否需要加载关联对象

```python
class ProductSummary(BaseModel):
    id: int
    name: str
    price: float
    original_price: Optional[float] = None
    discount_percentage: Optional[float] = None
    image_url: str
    category_name: str
    brand: str
    rating: float
    review_count: int
    stock_quantity: int
    is_available: bool
    created_at: datetime

class PaginationInfo(BaseModel):
    page: int
    size: int
    total: int
    total_pages: int
    has_next: bool
    has_prev: bool

class ProductListResponse(BaseModel):
    success: bool = True
    data: List[ProductSummary]
    pagination: PaginationInfo
    filters_applied: ProductFilters
    search_query: Optional[str] = None
    execution_time_ms: float
```

**Step 4: 数据库查询优化**

**考虑点**：
- **索引使用**: 常用查询字段需要索引
- **查询效率**: 避免N+1查询，使用JOIN或批量查询
- **缓存策略**: 热门数据可以缓存
- **读写分离**: 查询可以使用只读副本

```python
# 数据库模型（简化版）
class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255), index=True)  # 搜索索引
    price = Column(DECIMAL(10, 2), index=True)  # 价格索引
    category_id = Column(Integer, ForeignKey('categories.id'), index=True)
    brand = Column(String(100), index=True)
    stock_quantity = Column(Integer, default=0)
    created_at = Column(DateTime, default=datetime.utcnow, index=True)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # 关联
    category = relationship("Category")

# 复合索引
Index('idx_products_category_price', Product.category_id, Product.price)
Index('idx_products_brand_created', Product.brand, Product.created_at.desc())
```

**Step 5: 业务逻辑实现**

```python
from fastapi import APIRouter, Query, Depends
from sqlalchemy.orm import Session
from sqlalchemy import and_, or_, func, desc, asc
from app.database import get_db
from app.cache import RedisCache
from app.models import ProductQuery, ProductListResponse, ProductSummary, PaginationInfo
from app.services import ProductService
import time

router = APIRouter()

@router.get("/products", response_model=ProductListResponse)
async def get_products(
    # 查询参数
    page: int = Query(1, ge=1, description="页码"),
    size: int = Query(20, ge=1, le=100, description="每页大小"),
    q: str = Query(None, description="搜索关键词"),
    
    # 筛选参数
    category_id: int = Query(None, description="分类ID"),
    min_price: float = Query(None, ge=0, description="最低价格"),
    max_price: float = Query(None, gt=0, description="最高价格"),
    brand: str = Query(None, description="品牌"),
    in_stock: bool = Query(None, description="仅显示有货商品"),
    
    # 排序参数
    sort_by: str = Query("created_at", description="排序字段"),
    sort_order: str = Query("desc", description="排序方向"),
    
    # 依赖注入
    db: Session = Depends(get_db),
    cache: RedisCache = Depends(get_cache)
):
    """
    查询商品列表 - 数据查询API
    
    支持功能：
    - 关键词搜索
    - 多条件筛选
    - 灵活排序
    - 分页返回
    - 缓存优化
    """
    
    start_time = time.time()
    
    try:
        # Step 5.1: 参数验证和预处理
        query_params = ProductQuery(
            page=page,
            size=size,
            q=q,
            filters=ProductFilters(
                category_id=category_id,
                min_price=min_price,
                max_price=max_price,
                brand=brand,
                in_stock=in_stock
            ),
            sort_by=sort_by,
            sort_order=sort_order
        )
        
        # Step 5.2: 缓存检查（针对热门查询）
        cache_key = f"products:{hash(str(query_params))}"
        cached_result = await cache.get(cache_key)
        
        if cached_result:
            cached_result["execution_time_ms"] = (time.time() - start_time) * 1000
            return ProductListResponse(**cached_result)
        
        # Step 5.3: 构建查询条件
        product_service = ProductService(db)
        
        # 基础查询
        query = db.query(Product).join(Category)
        
        # 关键词搜索
        if query_params.q:
            search_term = f"%{query_params.q}%"
            query = query.filter(
                or_(
                    Product.name.ilike(search_term),
                    Product.description.ilike(search_term),
                    Category.name.ilike(search_term)
                )
            )
        
        # 筛选条件
        filters = []
        if query_params.filters.category_id:
            filters.append(Product.category_id == query_params.filters.category_id)
        
        if query_params.filters.min_price is not None:
            filters.append(Product.price >= query_params.filters.min_price)
        
        if query_params.filters.max_price is not None:
            filters.append(Product.price <= query_params.filters.max_price)
        
        if query_params.filters.brand:
            filters.append(Product.brand == query_params.filters.brand)
        
        if query_params.filters.in_stock is not None:
            if query_params.filters.in_stock:
                filters.append(Product.stock_quantity > 0)
            else:
                filters.append(Product.stock_quantity == 0)
        
        if filters:
            query = query.filter(and_(*filters))
        
        # Step 5.4: 统计总记录数
        total_count = query.count()
        
        # Step 5.5: 排序
        sort_column = getattr(Product, query_params.sort_by)
        if query_params.sort_order == "desc":
            query = query.order_by(desc(sort_column))
        else:
            query = query.order_by(asc(sort_column))
        
        # Step 5.6: 分页
        offset = (query_params.page - 1) * query_params.size
        products = query.offset(offset).limit(query_params.size).all()
        
        # Step 5.7: 数据转换
        product_summaries = []
        for product in products:
            # 计算折扣百分比
            discount_percentage = None
            if product.original_price and product.original_price > product.price:
                discount_percentage = round(
                    (1 - product.price / product.original_price) * 100, 
                    1
                )
            
            product_summaries.append(ProductSummary(
                id=product.id,
                name=product.name,
                price=float(product.price),
                original_price=float(product.original_price) if product.original_price else None,
                discount_percentage=discount_percentage,
                image_url=product.image_url,
                category_name=product.category.name,
                brand=product.brand,
                rating=float(product.average_rating or 0),
                review_count=product.review_count or 0,
                stock_quantity=product.stock_quantity,
                is_available=product.stock_quantity > 0,
                created_at=product.created_at
            ))
        
        # Step 5.8: 构建分页信息
        total_pages = (total_count + query_params.size - 1) // query_params.size
        
        pagination = PaginationInfo(
            page=query_params.page,
            size=query_params.size,
            total=total_count,
            total_pages=total_pages,
            has_next=query_params.page < total_pages,
            has_prev=query_params.page > 1
        )
        
        # Step 5.9: 构建响应
        response_data = {
            "data": product_summaries,
            "pagination": pagination,
            "filters_applied": query_params.filters,
            "search_query": query_params.q,
            "execution_time_ms": (time.time() - start_time) * 1000
        }
        
        # Step 5.10: 缓存结果（仅对非个性化查询）
        if not query_params.q and query_params.page == 1:  # 缓存首页热门商品
            await cache.set(cache_key, response_data, expire=300)  # 5分钟缓存
        
        return ProductListResponse(success=True, **response_data)
        
    except Exception as e:
        logger.error(f"商品查询失败: {e}")
        return ProductListResponse(
            success=False,
            data=[],
            pagination=PaginationInfo(
                page=page, size=size, total=0, 
                total_pages=0, has_next=False, has_prev=False
            ),
            filters_applied=ProductFilters(),
            execution_time_ms=(time.time() - start_time) * 1000
        )
```

**Step 6: 性能优化**

```python
# 添加数据库索引优化
from sqlalchemy import Index

# 为查询API添加专用索引
Index('idx_products_search', Product.name, Product.description)
Index('idx_products_filters', Product.category_id, Product.price, Product.brand, Product.stock_quantity)

# 缓存装饰器
def cache_query(expire_seconds: int = 300):
    def decorator(func):
        async def wrapper(*args, **kwargs):
            # 生成缓存键
            cache_key = f"{func.__name__}:{hash(str(kwargs))}"
            
            # 检查缓存
            cached = await cache.get(cache_key)
            if cached:
                return cached
            
            # 执行查询
            result = await func(*args, **kwargs)
            
            # 缓存结果
            await cache.set(cache_key, result, expire=expire_seconds)
            
            return result
        return wrapper
    return decorator

# 应用缓存
@router.get("/products")
@cache_query(expire_seconds=300)
async def get_products(...):
    pass
```

**Step 7: 监控和分析**

```python
# 查询性能监控
@router.get("/products")
async def get_products(...):
    start_time = time.time()
    
    try:
        # 查询逻辑...
        
        # 记录查询指标
        query_metrics = {
            "query_type": "product_list",
            "has_search": bool(q),
            "has_filters": bool(any([
                category_id, min_price, max_price, brand, in_stock
            ])),
            "page": page,
            "size": size,
            "result_count": len(products),
            "execution_time_ms": (time.time() - start_time) * 1000
        }
        
        # 发送到监控系统
        await send_metrics(query_metrics)
        
        return response
        
    except Exception as e:
        # 记录错误指标
        await send_error_metrics({
            "query_type": "product_list",
            "error": str(e),
            "execution_time_ms": (time.time() - start_time) * 1000
        })
        raise
```

**Step 8: 测试策略**

```python
def test_product_list_basic():
    """测试基本商品列表查询"""
    response = client.get("/api/products")
    assert response.status_code == 200
    
    data = response.json()
    assert data["success"] is True
    assert "data" in data
    assert "pagination" in data
    assert data["pagination"]["page"] == 1
    assert data["pagination"]["size"] == 20

def test_product_list_with_filters():
    """测试带筛选条件的查询"""
    response = client.get("/api/products?category_id=1&min_price=100&max_price=500")
    assert response.status_code == 200
    
    data = response.json()
    # 验证筛选条件生效
    for product in data["data"]:
        assert product["price"] >= 100
        assert product["price"] <= 500

def test_product_list_pagination():
    """测试分页功能"""
    # 第一页
    response1 = client.get("/api/products?page=1&size=5")
    data1 = response1.json()
    assert len(data1["data"]) == 5
    assert data1["pagination"]["has_next"] is True
    
    # 第二页
    response2 = client.get("/api/products?page=2&size=5")
    data2 = response2.json()
    assert len(data2["data"]) == 5
    assert data2["pagination"]["has_prev"] is True

def test_product_list_search():
    """测试搜索功能"""
    response = client.get("/api/products?q=laptop")
    assert response.status_code == 200
    
    data = response.json()
    # 验证搜索结果包含关键词
    for product in data["data"]:
        assert "laptop" in product["name"].lower()

def test_product_list_sorting():
    """测试排序功能"""
    # 按价格降序
    response = client.get("/api/products?sort_by=price&sort_order=desc")
    data = response.json()
    
    prices = [p["price"] for p in data["data"]]
    assert prices == sorted(prices, reverse=True)
```

**数据查询API的核心考虑点：**

1. **查询性能**: 索引优化、查询效率、分页处理
2. **参数设计**: 灵活的筛选、排序、分页参数
3. **缓存策略**: 热点数据缓存、缓存失效处理
4. **数据转换**: 数据库模型到API响应的转换
5. **监控分析**: 查询性能监控、慢查询分析
6. **测试覆盖**: 各种查询场景的测试

 

### 5. **数据操作API (Data Manipulation APIs)**

**作用**: 修改或删除数据
**特点**: 写操作，需要权限控制
**例子**:

```python
@app.delete("/api/memory/clear")           # 清除记忆
@app.delete("/api/learning/record/{record_id}")  # 删除记录
```

以博客平台的文章管理API为例

**Step 1: 需求分析阶段**

**思考点**：

- **数据安全性**: 谁可以修改什么数据？
- **操作类型**: 新建、修改、删除、批量操作
- **业务规则**: 修改时需要验证哪些业务约束？
- **审计需求**: 是否需要记录操作日志？

**以博客平台为例**：

- 用户可以创建、编辑、删除自己的文章
- 管理员可以管理所有文章
- 需要记录操作历史

**Step 2: 权限设计**

**考虑点**：
- **身份认证**: 如何验证用户身份？
- **权限模型**: 基于角色的访问控制(RBAC)
- **资源所有权**: 用户只能操作自己的资源？
- **细粒度权限**: 不同操作需要不同权限？

```python
from enum import Enum
from typing import Optional

class UserRole(str, Enum):
    ADMIN = "admin"
    EDITOR = "editor" 
    AUTHOR = "author"
    READER = "reader"

class Permission(Enum):
    CREATE_POST = "create_post"
    EDIT_OWN_POST = "edit_own_post"
    EDIT_ANY_POST = "edit_any_post"
    DELETE_OWN_POST = "delete_own_post"
    DELETE_ANY_POST = "delete_any_post"
    PUBLISH_POST = "publish_post"

# 角色权限映射
ROLE_PERMISSIONS = {
    UserRole.ADMIN: [
        Permission.CREATE_POST, Permission.EDIT_ANY_POST, 
        Permission.DELETE_ANY_POST, Permission.PUBLISH_POST
    ],
    UserRole.EDITOR: [
        Permission.CREATE_POST, Permission.EDIT_ANY_POST, 
        Permission.DELETE_OWN_POST, Permission.PUBLISH_POST
    ],
    UserRole.AUTHOR: [
        Permission.CREATE_POST, Permission.EDIT_OWN_POST, 
        Permission.DELETE_OWN_POST
    ],
    UserRole.READER: []  # 只读权限
}
```

**Step 3: 数据模型设计**

**考虑点**：
- **输入验证**: 新建和修改的字段验证
- **版本控制**: 记录修改历史？
- **关联数据**: 级联更新或删除？
- **状态管理**: 草稿、发布、归档等状态

```python
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import Optional, List

class ArticleStatus(str, Enum):
    DRAFT = "draft"
    PUBLISHED = "published"
    ARCHIVED = "archived"

class CreateArticleRequest(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=10)
    summary: Optional[str] = Field(None, max_length=500)
    tags: List[str] = Field(default_factory=list)
    category_id: int
    status: ArticleStatus = ArticleStatus.DRAFT
    
    @validator('tags')
    def validate_tags(cls, v):
        if len(v) > 10:
            raise ValueError('最多只能有10个标签')
        return [tag.strip() for tag in v if tag.strip()]

class UpdateArticleRequest(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    content: Optional[str] = Field(None, min_length=10)
    summary: Optional[str] = Field(None, max_length=500)
    tags: Optional[List[str]] = None
    category_id: Optional[int] = None
    status: Optional[ArticleStatus] = None
    
    @validator('tags')
    def validate_tags(cls, v):
        if v is not None and len(v) > 10:
            raise ValueError('最多只能有10个标签')
        return [tag.strip() for tag in v if tag.strip()]

class ArticleResponse(BaseModel):
    id: int
    title: str
    content: str
    summary: Optional[str]
    tags: List[str]
    category_id: int
    category_name: str
    author_id: int
    author_name: str
    status: ArticleStatus
    created_at: datetime
    updated_at: datetime
    published_at: Optional[datetime]
    view_count: int
```

**Step 4: 权限验证中间件**

**考虑点**：
- **身份获取**: 如何从请求中获取用户身份？
- **权限检查**: 同步还是异步检查？
- **缓存优化**: 权限检查结果可以缓存吗？
- **错误处理**: 无权限时的响应格式

```python
from fastapi import Request, HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> dict:
    """获取当前用户信息"""
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("user_id")
        
        # 从数据库获取用户完整信息
        user = await get_user_by_id(user_id)
        if not user:
            raise HTTPException(401, "用户不存在")
            
        return user
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "Token已过期")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "无效的Token")

def check_permission(required_permission: Permission):
    """权限检查装饰器"""
    def decorator(func):
        async def wrapper(*args, **kwargs):
            # 从kwargs中获取current_user（由Depends注入）
            current_user = kwargs.get('current_user')
            if not current_user:
                raise HTTPException(401, "未认证的用户")
            
            user_permissions = ROLE_PERMISSIONS.get(current_user['role'], [])
            
            if required_permission not in user_permissions:
                # 检查是否是资源所有者（用于EDIT_OWN_POST等权限）
                if required_permission in [Permission.EDIT_OWN_POST, Permission.DELETE_OWN_POST]:
                    article_id = kwargs.get('article_id')
                    if article_id:
                        article = await get_article_by_id(article_id)
                        if article and article.author_id == current_user['id']:
                            return await func(*args, **kwargs)
                
                raise HTTPException(403, "权限不足")
            
            return await func(*args, **kwargs)
        return wrapper
    return decorator
```

**Step 5: CRUD操作实现**

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.services import ArticleService, AuditService
from app.models import (
    CreateArticleRequest, UpdateArticleRequest, ArticleResponse,
    ArticleStatus
)
import time

router = APIRouter()

@router.post("/articles", response_model=ArticleResponse)
@check_permission(Permission.CREATE_POST)
async def create_article(
    request: CreateArticleRequest,
    current_user: dict = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """
    创建文章 - 数据操作API
    
    权限要求：CREATE_POST
    业务规则：
    - 作者只能创建自己的文章
    - 草稿状态可以不发布
    - 需要记录操作日志
    """
    
    try:
        article_service = ArticleService(db)
        audit_service = AuditService(db)
        
        # Step 5.1: 验证分类是否存在
        category = await article_service.get_category(request.category_id)
        if not category:
            raise HTTPException(400, "分类不存在")
        
        # Step 5.2: 创建文章
        article_data = request.dict()
        article_data.update({
            'author_id': current_user['id'],
            'created_at': datetime.utcnow(),
            'updated_at': datetime.utcnow()
        })
        
        article = await article_service.create_article(article_data)
        
        # Step 5.3: 记录审计日志
        await audit_service.log_action(
            user_id=current_user['id'],
            action="create_article",
            resource_type="article",
            resource_id=article.id,
            details={"title": article.title, "status": article.status}
        )
        
        return ArticleResponse.from_orm(article)
        
    except Exception as e:
        logger.error(f"创建文章失败: {e}")
        raise HTTPException(500, "创建文章失败")

@router.put("/articles/{article_id}", response_model=ArticleResponse)
@check_permission(Permission.EDIT_OWN_POST)  # 或者 EDIT_ANY_POST
async def update_article(
    article_id: int,
    request: UpdateArticleRequest,
    current_user: dict = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """
    更新文章 - 数据操作API
    
    权限要求：EDIT_OWN_POST 或 EDIT_ANY_POST
    业务规则：
    - 只能修改存在的文章
    - 已发布的文章修改需要审核
    - 需要记录修改历史
    """
    
    try:
        article_service = ArticleService(db)
        audit_service = AuditService(db)
        
        # Step 5.1: 获取并验证文章存在
        article = await article_service.get_article(article_id)
        if not article:
            raise HTTPException(404, "文章不存在")
        
        # Step 5.2: 权限检查（额外验证）
        if (current_user['role'] not in [UserRole.ADMIN, UserRole.EDITOR] and 
            article.author_id != current_user['id']):
            raise HTTPException(403, "只能编辑自己的文章")
        
        # Step 5.3: 业务规则验证
        if article.status == ArticleStatus.PUBLISHED and request.status == ArticleStatus.DRAFT:
            raise HTTPException(400, "已发布的文章不能改为草稿状态")
        
        # Step 5.4: 检查版本冲突（如果有乐观锁）
        # if request.version and article.version != request.version:
        #     raise HTTPException(409, "文章已被其他人修改")
        
        # Step 5.5: 更新文章
        update_data = request.dict(exclude_unset=True)
        update_data['updated_at'] = datetime.utcnow()
        
        updated_article = await article_service.update_article(article_id, update_data)
        
        # Step 5.6: 记录修改历史
        await audit_service.log_action(
            user_id=current_user['id'],
            action="update_article",
            resource_type="article", 
            resource_id=article_id,
            details={
                "old_status": article.status,
                "new_status": updated_article.status,
                "changes": list(update_data.keys())
            }
        )
        
        return ArticleResponse.from_orm(updated_article)
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"更新文章失败: {e}")
        raise HTTPException(500, "更新文章失败")

@router.delete("/articles/{article_id}")
@check_permission(Permission.DELETE_OWN_POST)  # 或者 DELETE_ANY_POST
async def delete_article(
    article_id: int,
    current_user: dict = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """
    删除文章 - 数据操作API
    
    权限要求：DELETE_OWN_POST 或 DELETE_ANY_POST
    业务规则：
    - 软删除而不是物理删除
    - 记录删除操作
    - 清理相关数据
    """
    
    try:
        article_service = ArticleService(db)
        audit_service = AuditService(db)
        
        # Step 5.1: 获取并验证文章存在
        article = await article_service.get_article(article_id)
        if not article:
            raise HTTPException(404, "文章不存在")
        
        # Step 5.2: 权限检查
        if (current_user['role'] not in [UserRole.ADMIN, UserRole.EDITOR] and 
            article.author_id != current_user['id']):
            raise HTTPException(403, "只能删除自己的文章")
        
        # Step 5.3: 软删除文章
        await article_service.soft_delete_article(article_id)
        
        # Step 5.4: 清理相关数据（评论、点赞等）
        await article_service.cleanup_related_data(article_id)
        
        # Step 5.5: 记录删除操作
        await audit_service.log_action(
            user_id=current_user['id'],
            action="delete_article",
            resource_type="article",
            resource_id=article_id,
            details={"title": article.title, "author_id": article.author_id}
        )
        
        return {"success": True, "message": "文章已删除"}
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"删除文章失败: {e}")
        raise HTTPException(500, "删除文章失败")

@router.post("/articles/{article_id}/publish")
@check_permission(Permission.PUBLISH_POST)
async def publish_article(
    article_id: int,
    current_user: dict = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """
    发布文章 - 数据操作API
    
    权限要求：PUBLISH_POST
    业务规则：
    - 草稿状态才能发布
    - 设置发布时间
    - 发送通知
    """
    
    try:
        article_service = ArticleService(db)
        audit_service = AuditService(db)
        
        # 获取文章
        article = await article_service.get_article(article_id)
        if not article:
            raise HTTPException(404, "文章不存在")
        
        if article.status != ArticleStatus.DRAFT:
            raise HTTPException(400, "只有草稿状态的文章才能发布")
        
        # 发布文章
        published_article = await article_service.publish_article(article_id)
        
        # 记录操作
        await audit_service.log_action(
            user_id=current_user['id'],
            action="publish_article",
            resource_type="article",
            resource_id=article_id,
            details={"title": article.title}
        )
        
        # 发送通知（可选）
        # await notification_service.notify_followers(article_id)
        
        return ArticleResponse.from_orm(published_article)
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"发布文章失败: {e}")
        raise HTTPException(500, "发布文章失败")
```

**Step 6: 并发控制和数据一致性**

```python
# 使用数据库事务确保一致性
from sqlalchemy import text

@router.put("/articles/{article_id}")
async def update_article(article_id: int, request: UpdateArticleRequest, db: Session = Depends(get_db)):
    async with db.begin():  # 事务开始
        try:
            # 检查版本（乐观锁）
            result = await db.execute(
                text("SELECT version FROM articles WHERE id = :id FOR UPDATE"), 
                {"id": article_id}
            )
            current_version = result.scalar()
            
            if request.version and current_version != request.version:
                raise HTTPException(409, "文章已被其他人修改，请刷新后重试")
            
            # 更新操作
            await article_service.update_article(article_id, request.dict())
            
            # 递增版本号
            await db.execute(
                text("UPDATE articles SET version = version + 1 WHERE id = :id"),
                {"id": article_id}
            )
            
            await db.commit()  # 事务提交
            
        except Exception as e:
            await db.rollback()  # 事务回滚
            raise
```

**Step 7: 审计和监控**

```python
class AuditService:
    async def log_action(self, user_id: int, action: str, resource_type: str, 
                        resource_id: int, details: dict = None):
        """记录操作日志"""
        audit_log = AuditLog(
            user_id=user_id,
            action=action,
            resource_type=resource_type,
            resource_id=resource_id,
            details=json.dumps(details or {}),
            ip_address=get_client_ip(),
            user_agent=get_user_agent(),
            created_at=datetime.utcnow()
        )
        
        db.add(audit_log)
        await db.commit()
        
        # 发送到监控系统
        await monitoring_service.log_security_event({
            "type": "data_modification",
            "user_id": user_id,
            "action": action,
            "resource": f"{resource_type}:{resource_id}",
            "timestamp": datetime.utcnow().isoformat()
        })
```

**Step 8: 测试策略**

```python
def test_create_article_success():
    """测试成功创建文章"""
    # 模拟认证用户
    response = client.post("/api/articles", 
                         json={"title": "Test", "content": "Content", "category_id": 1},
                         headers={"Authorization": f"Bearer {token}"})
    
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Test"
    assert data["author_id"] == test_user_id

def test_update_article_unauthorized():
    """测试无权限更新文章"""
    # 创建其他用户的文章
    other_user_article = create_article_for_other_user()
    
    response = client.put(f"/api/articles/{other_user_article.id}",
                         json={"title": "Hacked Title"},
                         headers={"Authorization": f"Bearer {token}"})
    
    assert response.status_code == 403

def test_delete_article_not_found():
    """测试删除不存在的文章"""
    response = client.delete("/api/articles/99999",
                           headers={"Authorization": f"Bearer {token}"})
    
    assert response.status_code == 404

def test_publish_article_business_rule():
    """测试发布文章的业务规则"""
    # 创建已发布的文章
    published_article = create_published_article()
    
    # 尝试再次发布
    response = client.post(f"/api/articles/{published_article.id}/publish",
                          headers={"Authorization": f"Bearer {token}"})
    
    assert response.status_code == 400
    assert "不能再次发布" in response.json()["detail"]
```

**数据操作API的核心考虑点：**

1. **权限控制**: 身份认证 + 授权检查 + 资源所有权验证
2. **数据验证**: 输入验证 + 业务规则验证 + 约束检查
3. **并发控制**: 乐观锁/悲观锁 + 事务管理
4. **审计日志**: 操作记录 + 安全监控 + 异常检测
5. **错误处理**: 业务异常 + 系统异常 + 用户友好提示
6. **测试覆盖**: 权限测试 + 边界测试 + 并发测试

### 6. **认证授权API (Authentication & Authorization APIs)**

**作用**: 处理用户登录、权限验证、token管理 

**特点**: 安全性要求高，涉及敏感信息 

**例子**:

```python
@app.post("/auth/login")           # 用户登录
@app.post("/auth/register")        # 用户注册  
@app.post("/auth/refresh-token")   # 刷新token
@app.get("/auth/me")              # 获取当前用户信息
@app.post("/auth/logout")          # 用户登出
```



### 7. **文件管理API (File Management APIs)**

**作用**: 处理文件上传、下载、存储、管理 

**特点**: 处理二进制数据，涉及存储系统

 **例子**:

```python
# 你已经有了文件上传
@app.post("/files/upload")         # 文件上传
@app.get("/files/{file_id}")       # 文件下载
@app.delete("/files/{file_id}")    # 文件删除
@app.get("/files/list")           # 文件列表
@app.put("/files/{file_id}/metadata")  # 更新文件元数据
```



### 8. **第三方集成API (Third-party Integration APIs)**

**作用**: 与外部服务集成（如支付、短信、邮件等） 

**特点**: 调用外部API，处理回调 

**例子**:

```python
@app.post("/integrations/stripe/webhook")  # 支付回调
@app.post("/integrations/sendgrid/webhook") # 邮件状态回调
@app.post("/notifications/send-email")     # 发送邮件
@app.get("/integrations/oauth/callback")   # OAuth回调
```



### 9. **管理后台API (Admin APIs)**

**作用**: 提供管理员功能，如用户管理、系统配置 

**特点**: 需要管理员权限，操作敏感数据 

**例子**:

```python
@app.get("/admin/users")           # 用户列表
@app.put("/admin/users/{user_id}/status")  # 修改用户状态
@app.get("/admin/stats")           # 系统统计
@app.post("/admin/config")         # 系统配置
@app.get("/admin/logs")            # 系统日志
```



### 10. **缓存API (Caching APIs)**

**作用**: 管理应用缓存，提高性能 

**特点**: 快速读写，临时存储 

**例子**:

```python
@app.get("/cache/{key}")           # 获取缓存
@app.put("/cache/{key}")           # 设置缓存
@app.delete("/cache/{key}")        # 删除缓存
@app.delete("/cache")              # 清空缓存
@app.get("/cache/stats")           # 缓存统计
```



### 11. **批量操作API (Batch Operations APIs)**

**作用**: 处理批量数据操作，提高效率 

**特点**: 处理大量数据，异步处理 

**例子**:

```python
# 你已经有了批量处理
@app.post("/batch/import-users")    # 批量导入用户
@app.post("/batch/export-data")     # 批量导出数据
@app.post("/batch/update-status")   # 批量更新状态
@app.get("/batch/{batch_id}/status") # 批量操作状态
```



### 12. **Webhook/回调API (Webhook/Callback APIs)**

**作用**: 接收外部服务的异步通知 

**特点**: 被动接收请求，无需认证

 **例子**:

```python
@app.post("/webhooks/github")       # GitHub webhook
@app.post("/webhooks/stripe")       # Stripe支付回调
@app.post("/webhooks/slack")        # Slack通知回调
@app.post("/hooks/{service}/verify") # webhook验证
```



### 13. **版本控制API (Versioning APIs)**

**作用**: 处理API版本兼容性 

**特点**: 支持多个版本同时运行 

**例子**:

```python
# URL路径版本
@app.get("/v1/users")               # V1版本
@app.get("/v2/users")               # V2版本

# 请求头版本
@app.get("/users")                  # Accept: application/vnd.api.v1+json
@app.get("/users")                  # Accept: application/vnd.api.v2+json

# 查询参数版本
@app.get("/users?version=v1")       # 查询参数版本
```



### 14. **监控和诊断API (Monitoring & Diagnostics APIs)**

**作用**: 系统监控、性能诊断、调试 

**特点**: 开发和运维使用，不对普通用户开放

 **例子**:

```python
# 你已经有了部分监控
@app.get("/metrics")                # Prometheus指标
@app.get("/debug/pprof")            # 性能分析
@app.get("/health/detailed")        # 详细健康检查
@app.get("/logs/stream")           # 日志流
@app.post("/debug/echo")           # 调试回显
```



### 15. **测试专用API (Testing APIs)**

**作用**: 为自动化测试提供专用接口 

**特点**: 只在测试环境启用 

**例子**:

```python
@app.post("/test/reset-db")         # 重置测试数据库
@app.post("/test/seed-data")        # 填充测试数据
@app.get("/test/fixtures")          # 获取测试固件
@app.delete("/test/cleanup")        # 清理测试数据
```



### 16. **内部服务API (Internal Service APIs)**

**作用**: 微服务架构中的内部通信 

**特点**: 服务间调用，不对外暴露 

**例子**:

```python
@app.post("/internal/sync-data")    # 数据同步
@app.get("/internal/health")        # 内部健康检查
@app.post("/internal/notify")       # 服务间通知
```

## API逻辑复用模式

### 1. **服务层复用 (Service Layer Reuse)**
**模式**: 将业务逻辑抽取到独立的服务类中
**例子**:
```python
# 复用的服务
memory_service = MemoryService(store=get_global_store())
task_manager = task_manager  # 全局任务管理器

# 在多个API中复用
@app.get("/api/memory/ted-history")
async def get_ted_history(user_id: str = "default"):
    memory_service.get_seen_ted_urls(user_id)  # 复用

@app.get("/api/memory/search-history") 
async def get_search_history(user_id: str = "default"):
    memory_service.get_recent_searches(user_id, limit=limit)  # 复用
```

### 2. **中间件复用 (Middleware Reuse)**
**模式**: 全局中间件处理通用逻辑
**例子**:
```python
# CORS中间件 - 所有API复用跨域处理
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 3. **依赖注入复用 (Dependency Injection)**
**模式**: 通过参数注入依赖，便于测试和替换
**例子**:
```python
# 依赖注入到业务逻辑
def process_ted_text(text, target_topic="", **kwargs):
    # 所有调用该函数的API都复用了相同的处理逻辑
    pass

# 在不同API中复用
@app.post("/process-file")
async def process_file(file: UploadFile):
    result = process_ted_text(transcript, **metadata)

@app.post("/search-ted") 
async def search_ted(request: SearchRequest):
    result = process_ted_text(text, target_topic=topic)
```

### 4. **路由模块复用 (Router Module Reuse)**
**模式**: 将相关API组织到路由模块中
**例子**:
```python
# 你已经使用了路由模块复用
app.include_router(monitoring_router)  # 监控API模块
app.include_router(memory_router)      # 记忆API模块

# 这些模块包含多个相关API
```

### 5. **错误处理复用 (Error Handling Reuse)**
**模式**: 统一的异常处理逻辑
**例子**:
```python
# 统一的异常处理模式
try:
    # 业务逻辑
    result = some_operation()
    return {"success": True, "data": result}
except Exception as e:
    # 统一的错误响应格式
    return {
        "success": False, 
        "error": str(e),
        "user_id": user_id,
        "total": 0
    }
```

### 6. **数据验证复用 (Validation Reuse)**
**模式**: 使用Pydantic模型进行数据验证
**例子**:
```python
# 复用的数据模型
class SearchRequest(BaseModel):
    topic: str
    user_id: Optional[str] = "default"

class SearchResponse(BaseModel):
    success: bool
    candidates: List[TEDCandidate]
    search_context: Dict[str, Any]
    total: int

# 在API中使用
@app.post("/search-ted", response_model=SearchResponse)
async def search_ted(request: SearchRequest):  # 自动验证输入
    # 返回时自动验证输出格式
    return SearchResponse(success=True, ...)
```

### 7. **异步任务复用 (Async Task Reuse)**
**模式**: 后台任务处理模式
**例子**:
```python
# 复用的后台任务模式
@app.post("/process-batch", response_model=BatchProcessResponse)
async def process_batch(request: BatchProcessRequest, background_tasks: BackgroundTasks):
    task_id = task_manager.create_task(request.urls, request.user_id)
    
    # 复用异步处理逻辑
    background_tasks.add_task(process_urls_batch, task_id, request.urls)
    
    return BatchProcessResponse(...)
```

### 8. **配置复用 (Configuration Reuse)**
**模式**: 集中配置管理
**例子**:
```python
# 复用的配置
settings = Settings()  # 从环境变量加载

# 在多个地方复用
app.add_middleware(CORSMiddleware, allow_origins=settings.cors_origins)
validate_config()  # 启动时验证配置
return {"model": settings.model_name}  # 在健康检查中返回
```

## 总结：API设计的复用原则

### **DRY原则 (Don't Repeat Yourself)**
- **服务层**: 业务逻辑复用
- **中间件**: 通用功能复用  
- **数据模型**: 类型验证复用

### **单一职责原则 (Single Responsibility)**
- **系统API**: 只负责系统状态
- **业务API**: 只负责业务逻辑
- **查询API**: 只负责数据获取

### **依赖倒置原则 (Dependency Inversion)**
- **依赖注入**: 不依赖具体实现，依赖抽象接口
- **接口分离**: 不同类型的API职责分离

### **开闭原则 (Open-Closed)**
- **路由模块**: 新功能通过添加模块扩展，不修改现有代码
- **中间件**: 通过添加中间件扩展功能

这样分类和复用的设计让你的API既清晰又可维护！每类API都有明确的职责，业务逻辑通过服务层复用，通用功能通过中间件复用。