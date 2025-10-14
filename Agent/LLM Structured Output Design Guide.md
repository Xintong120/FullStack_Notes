# LLM结构化输出设计指南

**副标题**：如何让LLM稳定返回JSON格式数据

**适用场景**：需要LLM返回结构化JSON的应用（搜索优化、数据提取、内容生成等）

**核心目标**：消除JSON输出的不确定性，保证字段和类型稳定

---

## 问题本质

LLM输出JSON时的三大不确定性：

1. **字段命名随意**：根据prompt措辞自由创建key
2. **类型推断不稳定**：单复数影响返回类型
3. **格式理解模糊**：缺少示例时自由发挥

**在弱类型语言（Python）中尤其危险**：类型错误延迟到运行时才暴露，形成连锁崩溃。

---

## 原则1：明确JSON结构

### 问题

没有完整JSON示例时，LLM根据prompt短语猜测key名称。

### 错误做法

```python
"""
Generate related items for: "{input}"
Output in JSON format.
"""

# LLM可能返回：
{"related_items": [...]}    # 或
{"items": [...]}            # 或
{"results": [...]}
```

### 正确做法

```python
"""
Generate related items for: "{input}"

Example output:
{"items": ["item1", "item2", "item3"]}

IMPORTANT: Use JSON key "items" (not "results" or "related_items")
"""

# LLM固定返回：
{"items": [...]}
```

### 要点

- 给出完整JSON结构示例
- 在IMPORTANT中明确key名称
- 禁止常见变体（not X, not Y）

---

## 原则2：明确数据类型

### 问题

单复数措辞影响LLM对类型的理解。

### 错误做法

```python
"""
Output 2-5 keywords for: "{input}"
"""

# LLM可能返回：
{"keywords": ["word1", "word2"]}       # 数组
{"keywords": "word1 word2"}            # 字符串
```

### 正确做法

```python
"""
Output 2-5 keywords as a SINGLE STRING (space-separated)

Example:
"climate" -> {"keywords": "climate change environment"}

IMPORTANT: Return string, NOT array
"""

# LLM固定返回字符串：
{"keywords": "word1 word2"}
```

### 要点

- 明确类型：SINGLE STRING / LIST OF STRINGS
- 说明分隔符：space-separated / comma-separated
- 禁止错误类型：NOT array / NOT object

---

## 原则3：用IMPORTANT强调核心要求

### 问题

关键信息埋在规则列表中容易被忽略。

### 错误做法

```python
"""
Rules:
1. Use English
2. Return JSON
3. Use key "items"
4. Return string type
"""
```

### 正确做法

```python
"""
Rules:
1. Use English
2. Return valid JSON

IMPORTANT:
- JSON key must be "items" (not "results")
- Value must be string (not array)
"""
```

### 要点

- IMPORTANT单独成段
- 重复关键信息
- 使用禁止项（not X）

---

## 原则4：代码防御性编程

Prompt无法100%保证输出，代码必须容错。

### 策略1：多Key兼容

```python
# 脆弱
data = result.get("items", [])

# 健壮
data = (
    result.get("items") or 
    result.get("results") or 
    result.get("data") or
    []
)
```

### 策略2：类型自动转换

```python
# 脆弱
def get_data(input: str) -> str:
    return llm(prompt)["keywords"]

# 健壮
def get_data(input: str) -> str:
    keywords = llm(prompt).get("keywords", input)
    
    # 自动转换list -> string
    if isinstance(keywords, list):
        keywords = " ".join(str(k) for k in keywords)
    
    return str(keywords).strip()
```

### 策略3：数据清洗

```python
def clean_list(items: any) -> list:
    if not isinstance(items, list):
        return []
    
    cleaned = []
    for item in items:
        if isinstance(item, str) and item.strip():
            cleaned.append(item.strip())
        elif isinstance(item, list):
            # 嵌套列表展平
            cleaned.append(" ".join(str(x) for x in item))
    
    return cleaned
```

### 要点

- 多key链式查找
- isinstance()类型检查
- 自动类型转换
- 降级策略

---

## 原则5：Prompt模板标准化

### 通用模板

```python
def build_prompt(task: str, user_input: str, 
                 json_key: str, data_type: str, 
                 examples: list) -> str:
    """
    标准化Prompt构建器
    
    Args:
        task: 任务描述
        user_input: 用户输入
        json_key: JSON输出的key名称
        data_type: 数据类型（如"string", "list[str]"）
        examples: [(input, output), ...] 示例列表
    """
    examples_str = "\n".join([
        f'"{inp}" -> {{"{json_key}": "{out}"}}'
        for inp, out in examples
    ])
    
    return f"""
{task}

INPUT: "{user_input}"

Examples:
{examples_str}

IMPORTANT:
- Use JSON key "{json_key}" (not other variations)
- Return {data_type} format

Output in JSON with key "{json_key}".
"""
```

### 使用示例

```python
prompt = build_prompt(
    task="Convert input to English keywords",
    user_input="人工智能",
    json_key="keywords",
    data_type="single string (space-separated)",
    examples=[
        ("气候变化", "climate change"),
        ("领导力", "leadership management")
    ]
)
```

---

## 常见错误对照表

| 错误类型 | 原因 | 表现 | 修复 |
|---------|------|------|------|
| **Key不匹配** | Prompt未明确key | LLM自创key名 | 完整JSON示例 + IMPORTANT |
| **类型错误** | 类型描述模糊 | 返回list而非string | 明确"SINGLE STRING" + 容错 |
| **格式不一致** | 缺少示例 | 输出不可预测 | 多个Input/Output示例 |
| **嵌套混乱** | JSON结构不清 | 扁平/嵌套不符预期 | 完整JSON结构示例 |

---

## 调试技巧

### 添加DEBUG输出

```python
result = llm(prompt)
print(f"[DEBUG] 原始返回: {result}")
print(f"[DEBUG] 类型: {type(result)}")

if isinstance(result, dict):
    for k, v in result.items():
        print(f"[DEBUG] {k}: {v} (type: {type(v)})")
```

### 测试一致性

```python
# 相同输入多次调用
results = [get_data("test") for _ in range(3)]
unique = len(set(results))

if unique == 1:
    print("完美一致")
elif unique <= 2:
    print("基本一致")
else:
    print(f"不一致({unique}种结果)，降低temperature")
```

---

## Temperature参数选择

```python
# 需要稳定输出
llm(prompt, temperature=0.1)

# 需要创意/多样性
llm(prompt, temperature=0.2-0.3)
```

---

## Prompt设计检查清单

- [ ] 给出完整JSON结构示例
- [ ] 明确指定JSON key名称
- [ ] 明确数据类型（string/list/dict）
- [ ] 使用IMPORTANT强调关键要求
- [ ] 提供2-3个Input/Output示例
- [ ] 明确禁止错误变体
- [ ] 代码有多key兼容逻辑
- [ ] 代码有类型转换容错
- [ ] 设置合适的temperature
- [ ] 添加DEBUG输出

---

## 实战案例

**场景**：搜索优化工具

**问题**：LLM返回`{"alternative_search_queries": [...]}`而非`{"alternatives": [...]}`

**解决**：
```python
# 修复前
"""
Generate alternative search queries
Output in JSON
"""

# 修复后
"""
Generate alternative search queries

Example:
{"alternatives": ["query1", "query2"]}

IMPORTANT: Use key "alternatives" (not "alternative_search_queries")
"""
```

**教训**：
- prompt中的短语会影响key命名
- 必须给出完整JSON示例
- IMPORTANT强调禁止变体

---

## 文档说明

本文档专注于**LLM结构化JSON输出**这一具体场景。

LLM Prompt工程还包括其他重要领域：
- Few-shot learning（示例学习）
- Chain of thought（思维链）
- Role prompting（角色设定）
- RAG prompting（检索增强生成）
- Multi-turn conversations（多轮对话）
- Constraint specification（约束条件设定）

本指南不涵盖上述内容，仅聚焦于解决JSON输出的稳定性问题。

---

**最后更新**: 2025-10-09  
**文档版本**: v2.0  
**范围**: 结构化JSON输出设计
