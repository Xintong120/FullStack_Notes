## LangChain Memory

### 1. **缓冲区记忆 (Buffer Memory)**

这是最基础的记忆形式，本质上是短期记忆。它将最近的对话历史完整地存储起来，并在下一次请求时作为上下文提供给 LLM。

- 实现方式:
  - `ConversationBufferMemory`: 存储完整的对话历史。
  - `ConversationBufferWindowMemory`: 只存储最近的 `k` 轮对话，防止上下文窗口溢出。
- **优点**: 简单直接，能很好地维持短期对话的连贯性。
- **缺点**: 随着对话变长，成本和Token消耗会线性增加，且容易超出模型的上下文长度限制。
- **开发者实践**: 这是大多数简单聊天机器人的首选，也是构建更复杂记忆系统的基础。

### 2. **摘要记忆 (Summary Memory)**

为了解决缓冲区记忆的长度问题，摘要记忆应运而生。它会使用一个 LLM 在后台将对话历史动态地浓缩成一个摘要。

- 实现方式:
  - `ConversationSummaryMemory`: 将对话历史总结成一段摘要。
  - `ConversationSummaryBufferMemory`: 结合了摘要和缓冲区，保留最近的对话，同时将更早的对话进行总结。
- **优点**: 有效地压缩了上下文，节省了Token，理论上可以支持无限长的对话。
- **缺点**: 摘要过程本身会消耗额外的 LLM 调用成本，并且可能会丢失一些细节信息。
- **开发者实践**: 适用于需要进行长时间、多轮交互的客服、助理类 Agent。

### 3. **实体记忆 (Entity Memory)**

这种记忆方式更进一步，它不仅仅是记录对话，而是从对话中提取出关键的**实体**（如人名、地名、关键概念）及其相关信息，并将其结构化存储。

- 实现方式:
  - `ConversationEntityMemory`: 使用 LLM 识别对话中的实体，并围绕这些实体构建一个知识库。
- **优点**: 能够记住关于特定人或事物的关键事实，使得 Agent 看起来“知识渊博”。
- **缺点**: 对 LLM 的实体提取能力要求较高，实现相对复杂。
- **开发者实践**: 在需要记住用户个人信息（如“我的名字是Bob”）或特定领域知识的场景中非常有用。

### 4. **知识图谱记忆 (Knowledge Graph Memory)**

这是实体记忆的升级版，它将提取出的实体以及它们之间的关系以**知识图谱**的形式存储起来。这是一种高度结构化的长期记忆。

- 实现方式:
  - `ConversationKGMemory`: 将对话内容转化为知识图谱的三元组（主语-谓语-宾语）。
- **优点**: 能够理解和推理实体间的复杂关系，提供更深层次的上下文理解。
- **缺点**: 实现非常复杂，需要图数据库（如 Neo4j）的支持，并且对 LLM 的信息提取和结构化能力要求极高。
- **开发者实践**: 在 GitHub 上可以看到一些前沿项目（如 `FalkorDB`）致力于将知识图谱作为 LLM 的核心记忆，尤其是在复杂的 RAG（检索增强生成）场景中。

### 5. **向量存储记忆 (Vector Store Memory)**

这是目前实现**长期记忆**和**语义记忆**最主流和强大的方式。它将对话片段、用户偏好或其他知识通过 Embedding 模型转化为向量，并存储在向量数据库中。

- 实现方式:
  - 将对话历史、文档、用户反馈等文本数据嵌入（Embed）为向量。
  - 存储在 `InMemoryVectorStore` (用于测试) 或持久化的向量数据库 (如 Chroma, Pinecone, FAISS) 中。
  - 在需要时，将当前的用户问题也转化为向量，通过**语义相似度搜索**来检索最相关的记忆片段。
- 优点:
  - **语义检索**: 能根据意义而不是关键词找到相关信息。
  - **可扩展性**: 能够存储海量信息。
  - **灵活性**: 可以存储任何文本信息，从对话历史到用户手册。
- **开发者实践**: 这是您分享的文章中实现长期记忆的核心技术。LangChain 文档中的 `save_recall_memory` 和 `search_recall_memories` 工具就是典型的例子。它被广泛用于构建 RAG 系统、个性化推荐和需要从大量知识中学习的 Agent。

### 总结

| 记忆类型         | 主要特点           | 优点                       | 缺点                        | 常见用例                      |
| ---------------- | ------------------ | -------------------------- | --------------------------- | ----------------------------- |
| **缓冲区记忆**   | 存储原始对话历史   | 简单、短期连贯性好         | 成本高、长度受限            | 简单聊天机器人                |
| **摘要记忆**     | 动态总结对话       | 节省Token、支持长对话      | 额外成本、可能丢失细节      | 长时间客服、AI 助理           |
| **实体记忆**     | 提取并存储关键实体 | 记住关键事实               | 实现复杂、依赖提取能力      | 个性化服务、领域专家          |
| **知识图谱记忆** | 结构化存储实体关系 | 能进行复杂推理             | 实现非常复杂、需要图数据库  | 复杂 RAG、深度问答            |
| **向量存储记忆** | 语义化存储和检索   | **语义相关**、可扩展、灵活 | 需要 Embedding 模型和向量库 | **主流长期记忆**、RAG、个性化 |

## LangGraph 记忆组件的核心概念

LangGraph 的记忆系统是其核心优势之一，因为它专注于构建**有状态的、长运行的 Agent**。它将记忆分为两个主要部分：

1. **检查点 (Checkpointer) - 短期记忆/会话状态持久化**
   - **作用**: 检查点是 LangGraph 实现**持久化执行**的关键。它负责保存和恢复 Agent 在图中的**完整状态**，包括所有节点之间的消息、变量等。这意味着即使 Agent 的执行被中断（例如，等待用户输入，或者程序崩溃），它也能从上次保存的状态继续运行。
   - 实现:
     - `InMemorySaver`: 最简单的检查点，将状态保存在内存中，适用于开发和测试。
     - `PostgresSaver` (或其他数据库 Saver): 用于生产环境，将状态持久化到数据库（如 PostgreSQL），确保数据不会丢失。
   - **与 LangChain 的区别**: LangChain 的传统记忆组件（如 `ConversationBufferMemory`）主要关注对话历史的存储和检索，而 LangGraph 的 `Checkpointer` 关注的是**整个 Agent 状态的快照和恢复**。这使得 LangGraph 能够构建更复杂的、多步骤的、可中断的 Agent。
2. **存储 (Store) - 长期记忆/跨会话知识库**
   - **作用**: `Store` 是 LangGraph 用于管理**长期记忆**的组件。它允许 Agent 存储和检索跨越多个会话的通用知识、用户偏好、Agent 自身的指令等。这部分记忆通常是结构化的，并且可以通过语义搜索进行检索。
   - 实现:
     - `InMemoryStore`: 同样是内存中的实现，用于开发测试。它可以配置为支持**语义搜索**，通过 Embedding 模型将存储的内容向量化，然后进行相似度查询。
     - **数据库支持**: 类似于检查点，生产环境会使用数据库支持的 `Store` 实现，例如与 `pgvector` 集成的 PostgreSQL，或者 Redis 等。
   - 关键特性:
     - **命名空间 (Namespace)**: 允许你对记忆进行分组，例如按 `(user_id, "memories")` 或 `("agent_instructions",)` 进行组织，实现多租户和模块化记忆。
     - **语义搜索**: 通过 Embedding 模型将文本内容转化为向量，然后进行相似度搜索，实现基于意义的检索。这在您提供的文章中也有详细介绍。
     - **多向量索引**: 可以为记忆中的不同字段（如 `memory` 和 `emotional_context`）创建独立的 Embedding，提高检索的精确度。
   - **与 LangChain 的区别**: LangChain 也有 `VectorStoreRetriever` 等组件用于长期记忆和 RAG。LangGraph 的 `Store` 在概念上与此类似，但它更紧密地集成到 LangGraph 的图结构中，允许在任何节点中方便地访问和更新。此外，LangGraph 的 `Store` 强调了**Agent 自身指令的自我更新**作为一种长期记忆形式，这在 LangChain 的传统记忆组件中不那么突出。

### LangGraph 与 LangChain 记忆组件的异同

| 特性/组件      | LangChain (传统记忆)                                         | LangGraph (记忆组件)                                         | 区别总结                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **核心关注点** | 对话历史的存储和检索                                         | **Agent 完整状态的持久化** 和 **跨会话知识管理**             | LangGraph 更宏观，关注整个 Agent 的生命周期和学习。          |
| **短期记忆**   | `ConversationBufferMemory`, `ConversationBufferWindowMemory`, `ConversationSummaryMemory` 等。 | **Checkpointer** (`InMemorySaver`, `PostgresSaver` 等)       | LangChain 侧重于将对话历史作为上下文传递给 LLM。LangGraph 的 Checkpointer 则是保存整个图的执行状态，包括所有中间步骤和变量，实现**可恢复性**。 |
| **长期记忆**   | `VectorStoreRetriever`, `ConversationEntityMemory`, `ConversationKGMemory` 等。 | **Store** (`InMemoryStore` 及数据库支持的 Store)             | 两者都支持基于向量的语义检索。LangGraph 的 Store 更强调**命名空间**和**Agent 自身指令的动态更新**，将其作为 Agent 学习和适应的核心机制。 |
| **Agent 学习** | 通常通过微调 LLM 或在提示中手动管理。                        | 通过 `Store` 动态更新 Agent 的**指令/偏好**，结合 HITL 实现**自我适应和学习**。 | LangGraph 提供更原生的机制来让 Agent 从反馈中学习并更新其行为逻辑（通过修改存储的指令）。 |
| **持久化**     | 依赖于外部数据库或文件系统。                                 | **Checkpointer** 和 **Store** 都提供多种持久化选项，从内存到生产级数据库。 | LangGraph 将持久化作为其框架的内置核心功能，与 Agent 的执行流紧密结合。 |
| **集成方式**   | 作为 `Chain` 或 `AgentExecutor` 的一部分。                   | 作为 `StateGraph` 的编译参数 (`checkpointer`, `store`)，并在节点中通过 `config` 或 `InjectedStore` 访问。 | LangGraph 的记忆组件与图的节点和边紧密耦合，使得在 Agent 内部管理记忆更加自然和强大。 |

### 总结

LangGraph 在记忆管理方面，可以看作是 LangChain 记忆概念的**更高级和更集成**的实现，尤其适用于构建复杂的、有状态的、能够从经验中学习的 Agent。

- **LangGraph 的 Checkpointer** 提供了**执行持久化**，这是 LangChain 传统记忆组件所不具备的。它让 Agent 能够“记住”它在任何时刻的“思考过程”和“行动计划”。
- **LangGraph 的 Store** 提供了**可编程的长期记忆**，它不仅仅是存储数据，更是 Agent 自身知识和行为规则的动态存储库。结合命名空间、语义搜索和多向量索引，它为 Agent 的**自我适应和学习**提供了强大的基础。

## ADK 记忆组件的核心概念

ADK 的记忆系统主要围绕 **`MemoryService`** 这个核心抽象展开。它将记忆管理视为一个独立的服务，负责存储和检索与用户和会话相关的信息。

1. **会话驱动的记忆 (Session-Driven Memory)**
   - **核心思想**: ADK 的记忆与**会话 (Session)** 紧密绑定。一个会话代表了用户与 Agent 的一次完整交互。ADK 的设计理念是，在会话结束后，将其**完整地**存入 `MemoryService`。
   - 实现:
     - `InMemoryMemoryService`: 用于开发和测试的内存实现，数据在程序重启后会丢失。
     - `VertexAiMemoryBankService`: 这是 ADK 的核心优势，它与 **Google Cloud Vertex AI Memory Bank** 深度集成，为 Agent 提供了企业级的、持久化的、可扩展的记忆存储。
   - **与 LangGraph 的区别**: LangGraph 的 `Checkpointer` 保存的是图执行的**中间状态**，以便随时恢复。而 ADK 倾向于在一次交互（Session）完成后，将整个会话作为一条“记忆”存入知识库。
2. **工具化的记忆访问 (Tool-based Memory Access)**
   - **核心思想**: Agent 并不是随意访问记忆，而是通过专门的**工具 (Tools)** 来与 `MemoryService` 交互。这使得记忆的访问过程更加结构化和可控。
   - 实现:
     - `PreloadMemoryTool`: 这是一个特殊的工具，可以在 Agent 每次执行任务**之前**自动运行，根据当前的用户输入预加载相关的记忆，并将其注入到提示（Prompt）中。
     - `search_memory`: 在工具的上下文中 (`ToolContext`)，可以调用此函数来主动查询记忆库，以获取决策所需的信息。
   - **与 LangChain 的区别**: LangChain 也有工具的概念，但 ADK 将“预加载记忆”本身也设计成一个标准化的工具，这是一种很有趣的设计模式，确保了记忆检索成为 Agent 执行流程中的一个明确步骤。
3. **自动化的记忆持久化**
   - **核心思想**: ADK 提供了回调机制，可以轻松实现记忆的自动保存。
   - **实现**: 通过设置 `after_agent_callback`，可以在 Agent 每次运行**之后**自动调用一个函数，将当前的会话保存到 `MemoryService` 中。这大大简化了开发者的工作，无需在业务逻辑中手动调用保存记忆的代码。

### ADK 与 LangGraph/LangChain 的主要不同点

| 特性/组件    | LangGraph                                                | Agent Developer Kit (ADK)                                    | 区别总结                                                     |
| ------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计哲学** | 将记忆与**图的状态**和**执行流**紧密耦合。               | 将记忆视为一个**独立的、可插拔的服务**。                     | ADK 的设计更符合微服务架构，将记忆能力解耦。                 |
| **核心抽象** | `Checkpointer` (状态快照) 和 `Store` (知识库)。          | `MemoryService` (会话存储和检索)。                           | ADK 以“会话”为单位来思考记忆，而 LangGraph 以“状态”和“知识”为单位。 |
| **记忆存储** | 倾向于将对话历史、用户偏好等**细粒度**信息存入 `Store`。 | 倾向于将**整个会话**作为一个整体存入 `MemoryService`。       | ADK 的方式更粗粒度，但可能更易于追溯完整的交互历史。         |
| **生态集成** | 开放，支持多种数据库和向量存储。                         | **与 Google Cloud Vertex AI 生态系统深度集成**。             | ADK 是 Google Cloud 生态的一部分，对于使用 Vertex AI 的开发者来说，集成更加无缝。 |
| **访问方式** | 在图的节点中直接访问 `store` 对象。                      | 通过**标准化的工具** (`PreloadMemoryTool`, `search_memory`) 访问。 | ADK 的方式更规范，但也可能不如 LangGraph 灵活。              |

### 总结

- **LangGraph** 的记忆系统非常**灵活且强大**，它与 Agent 的内部状态和执行逻辑紧密结合，非常适合构建能够**自我反思和学习**的复杂 Agent。
- **Agent Developer Kit (ADK)** 的记忆系统则更加**工程化和规范化**。它将记忆抽象为一个独立的服务，通过标准化的工具和回调与 Agent 解耦，并且与 Google Cloud 的生产环境无缝集成。对于希望在 Google Cloud 上快速构建和部署企业级 Agent 的开发者来说，ADK 提供了一条更清晰、更结构化的路径。

## Agent Memory 分类

------

### 1. 按【Memory 载体】分类：记忆如何存储？

这个维度关注的是记忆信息的物理或逻辑存储形式。

#### a. **Token-level Memory (外部记忆)**

- **解释**: 这是最经典、最广泛的记忆形式。它的核心思想是，记忆被存储在 Agent 的**外部**，以人类可读或机器可解析的格式存在，比如文本、JSON、数据库条目等。当 Agent 需要使用记忆时，它会从这些外部源“读取”信息，并将其整合到当前的上下文中（通常是 Prompt）。

- 例子:

  - **向量数据库**: 将对话历史、用户偏好、文档等信息转化为向量，存入 Chroma、Pinecone 等数据库。这是实现 RAG 和长期记忆的主流方式。
  - **图数据库**: 如 Neo4j、FalkorDB，用于存储实体及其关系的知识图谱。
  - **传统数据库/文件**: 简单的 Key-Value 存储、SQL 数据库或本地 JSON 文件。

- 项目代码验证:

  - MemOS (现名 AIOS): 正如文章所说，AIOS (原名 MemOS) 项目就是一个典型的例子。它构建了一个虚拟的“操作系统”，其中有文件系统、进程管理等，Agent 的记忆就像文件一样被存储和管理，是典型的 Token-level Memory。

    - **正确地址**: [https://github.com/agiresearch/AIOS](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - **MemoryBank**: 这是一个早期的研究概念，其思想被后来的许多项目所吸收。在 GitHub 上，它更多地出现在各种论文列表和综述仓库中。

  - mem0:

    - **分类**: **Token-level Memory (外部记忆)**。`mem0` 的核心是一个**通用记忆层**，它为 AI Agent 提供持久化记忆。它支持将记忆存储在外部，并通过语义搜索进行检索。其描述中明确提到了“Universal memory layer for AI Agents”和“long-term memory”。
    - **正确地址**: [https://github.com/mem0ai/mem0](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - Graphiti:

    - **分类**: **Token-level Memory (外部记忆) - 特别是图数据库**。`Graphiti` 专注于构建**实时知识图谱**作为 AI Agent 的记忆。它将信息存储为图结构，并通过图查询进行检索。其描述中明确提到了“Build Real-Time Knowledge Graphs for AI Agents”。
    - **正确地址**: [https://github.com/getzep/graphiti](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - letta:

    - **分类**: **Token-level Memory (外部记忆)**。`letta` 是一个用于构建有状态 Agent 的平台，其核心特点是“advanced memory that can learn and self-improve over time”。它通过持久化记忆来支持 Agent 的学习和自我提升。
    - **正确地址**: [https://github.com/letta-ai/letta](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - memary:

    - **分类**: **Token-level Memory (外部记忆) - 结合知识图谱**。`Memary` 被描述为“The Open Source Memory Layer For Autonomous Agents”，其核心是利用**知识图谱**来组织和管理 Agent 的长期记忆。
    - **正确地址**: [https://github.com/kingjulio8238/Memary](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - cognee:

    - **分类**: **Token-level Memory (外部记忆) - 专注于数据组织和上下文提供**。`Cognee` 是一个模块化工具，旨在组织和连接各种数据（结构化、半结构化、非结构化），为 LLM 提供准确的上下文。虽然它本身不是一个数据库，但它管理着外部数据的组织和检索，从而为 Agent 提供记忆。
    - **正确地址**: [https://github.com/cognee/cognee](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - LangChain:

    - 分类:

       Token-level Memory (外部记忆)。LangChain 提供了多种记忆组件，绝大多数都属于 Token-level Memory。

      - **`ConversationBufferMemory` / `ConversationBufferWindowMemory`**: 将原始对话文本存储在内存或外部存储中。
      - **`ConversationSummaryMemory`**: 将对话摘要文本存储起来。
      - **`VectorStoreRetriever`**: 将文档或对话片段的 Embedding 存储在向量数据库中，并通过语义搜索检索文本。
      - **`ConversationEntityMemory` / `ConversationKGMemory`**: 提取实体或知识图谱三元组，并以结构化文本形式存储。

  - LangGraph:

    - 分类:Token-level Memory (外部记忆)。LangGraph 的Store组件是典型的 Token-level Memory。
      - **`InMemoryStore`**: 存储 JSON 文档，可配置为支持语义搜索（通过 Embedding 向量）。
      - **数据库支持的 Store**: 将记忆存储在外部数据库中，如 PostgreSQL。
      - **Checkpointer**: 虽然 `Checkpointer` 存储的是 Agent 的完整状态，但这些状态最终也是以可序列化的形式（如 JSON）存储在内存或数据库中，因此也属于 Token-level Memory 的范畴。

  - Agent Developer Kit (ADK):

    - 分类:

       Token-level Memory (外部记忆)。ADK MemoryService

       负责存储和检索会话数据。

      - **`InMemoryMemoryService`**: 存储会话数据（通常是文本形式）在内存中。
      - **`VertexAiMemoryBankService`**: 将会话数据持久化到 Vertex AI Memory Bank，这些数据也是以可检索的文本或结构化形式存在。

#### b. **Parametric Memory (参数化记忆)**

- **解释**: 这种方式非常激进，它不将记忆存储在外部，而是尝试将知识**直接编码进模型本身的参数中**。可以理解为，Agent 通过“训练”来“学会”并“记住”信息，而不是通过“检索”。
- 例子:
  - **持续预训练/微调 (SFT)**: 这是最广义的参数化记忆。通过在特定领域的知识上持续训练模型，这些知识就内化为了模型的参数。
  - ParametricRAG: 这篇文章提到的ParametricRAG是一个很好的例子。它不是在运行时去检索文档，而是提前将一份长文档（比如一本手册）“编译”或“蒸馏”到一个小型的、专门的 LoRA (Low-Rank Adaptation) 模块中。当主模型需要这份手册的知识时，它会激活这个 LoRA 模块，从而获得相应的记忆，而不是去检索文本。
    - **正确地址**: [https://github.com/facebookresearch/FiD](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- 文档检验: 这个方向非常前沿。研究表明，参数化记忆在处理高频访问、通用性强的知识时非常高效，但缺点是更新记忆（即重新训练）的成本很高，且容易发生“灾难性遗忘”。
  - **LangChain / LangGraph / ADK**: 这些框架本身**不直接提供** Parametric Memory 的实现。它们主要关注如何利用外部记忆来增强 LLM 的能力，而不是修改 LLM 自身的参数。如果要在这些框架中使用 Parametric Memory，通常需要结合外部的 LLM 微调或适配技术。

#### c. **Latent Memory (隐式记忆)**

- **解释**: 这是一种介于外部记忆和参数化记忆之间的形态。记忆既不完全在外部（作为文本），也不完全在内部（作为模型权重），而是以一种**中间状态**——即**隐式的向量或嵌入 (Latent Embeddings/Tokens)** 的形式存在。这些“记忆向量”可以被 Agent 动态地加载到其计算过程中。
- 例子:
  - MemoryLLM: 这个项目通过设计一个特殊的“记忆模块”，允许 LLM 将过去的对话压缩成一系列的“记忆 token”。在处理新对话时，这些记忆 token 会被一起送入 Transformer 的注意力层，从而让模型“感知”到历史信息，而无需将完整的历史文本都放入上下文中。
    - **正确地址**: [https://github.com/wangyu-ustc/MemoryLLM](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - **M+**: 这是一个类似的概念，通过一个额外的记忆网络来维护和更新隐式的记忆状态。
- 项目代码验证: 这类项目的核心通常在于修改了模型的输入表示或内部结构（如注意力机制），增加了一个专门处理“记忆向量”的通道。它试图在 Token-level 的灵活性和 Parametric 的高效性之间找到一个平衡点。
  - **LangChain / LangGraph / ADK**: 这些框架**不直接提供** Latent Memory 的实现。它们更多地将 Embedding 视为 Token-level Memory 的**索引机制**，而不是记忆本身。Latent Memory 通常需要对 LLM 的架构进行更深层次的修改，这超出了这些 Agent 框架的直接范围。

------

### 2. 按【Memory 目的】分类：为何要记忆？

这个维度关注的是 Agent 使用记忆来完成什么任务，体现了记忆的功能性。

#### a. **Chatbot Memory (对话记忆)**

- **目的**: 核心目标是提升 Agent 在**多轮、跨会话**交互中的表现，使其能够记住用户的身份、偏好、历史对话内容，从而提供更连贯、更个性化的对话体验。
- 例子:
  - **记住用户偏好**: 正如您之前分享的 LangGraph 文章中的电子邮件助手，它通过 HITL 反馈学习并更新用户的写作风格和会议偏好。
  - **跨会话上下文**: 用户今天登录时，Agent 能记得昨天讨论的话题。
- 项目代码验证:
  - A-Mem: 这些研究项目都专注于构建更强大的对话记忆系统，通过不同的机制（如摘要、反思、向量检索）来优化 Chatbot 的长期记忆能力。
    - **正确地址**: [https://github.com/agiresearch/A-mem](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 或 [https://github.com/WujiangXu/A-mem](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - **M3Agent**: 未找到明确的官方代码库。
  - mem0:
    - **分类**: **Chatbot Memory**。`mem0` 的主要应用场景之一就是为 Chatbot 提供长期记忆，记住用户偏好和历史对话，从而实现个性化交互。其 Chrome 扩展程序也印证了这一点。
    - **正确地址**: [https://github.com/mem0ai/mem0](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - letta:
    - **分类**: **Chatbot Memory**。`letta` 平台明确指出其目标是构建“stateful agents”，并强调“advanced memory that can learn and self-improve over time”，这直接指向了 Chatbot 记忆的需求。其提供的 Chatbot 示例也证实了这一点。
    - **正确地址**: [https://github.com/letta-ai/letta](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - memary:
    - **分类**: **Chatbot Memory**。`Memary` 旨在为自主 Agent 提供记忆层，通过模拟人类记忆的工作方式来帮助 Agent 更好地管理和利用信息，这对于多轮对话的 Chatbot 至关重要。
    - **正确地址**: [https://github.com/kingjulio8238/Memary](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - LangChain:
    - **分类**: **Chatbot Memory**。LangChain 提供了最丰富的 Chatbot Memory 实现，如 `ConversationBufferMemory`、`ConversationBufferWindowMemory`、`ConversationSummaryMemory`、`ConversationEntityMemory` 等，它们都旨在提升 Agent 在对话中的连贯性和个性化。
  - LangGraph:
    - **分类**: **Chatbot Memory**。LangGraph 通过 `Checkpointer` 实现会话状态的持久化，以及通过 `Store` 存储用户偏好和对话历史，从而为构建具有强大 Chatbot Memory 的 Agent 提供了基础。您之前分享的电子邮件助手就是典型案例。
  - Agent Developer Kit (ADK):
    - **分类**: **Chatbot Memory**。ADK 的 `MemoryService` 明确用于存储和检索会话数据，其 `Session` 概念直接对应了 Chatbot 的多轮交互。`PreloadMemoryTool` 也有助于在对话开始时加载相关记忆。

#### b. **Self-improving Memory (自我提升记忆)**

- **目的**: 让 Agent 能够从**过去的成功和失败经验 (Trajectories) 中学习**，以提高解决新任务的能力。这是一种元认知能力，Agent 不仅在解决问题，还在反思“如何更好地解决问题”。

- 例子:

  - **经验回放与反思**: Agent 在完成一次复杂的编码任务后，会将整个过程（包括尝试、错误、最终解决方案）记录下来。下次遇到类似任务时，它会“回放”这段经历，避免重蹈覆辙。
  - **工具使用学习**: Agent 第一次使用某个 API 失败了，它会记住失败的原因（如参数错误），并形成一条新的“经验法则”。

- 项目代码验证:

  - ExpeL

    : 这是该领域的开创性工作之一。它明确提出了一个“经验库 (Experience Base)”的概念，Agent 在完成任务后会进行自我反思，并将总结出的经验（如高层次的洞察、可复用的代码片段）存入经验库中。

    - **正确地址**: [https://github.com/LeapLabTHU/ExpeL](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - letta:

    - **分类**: **Self-improving Memory**。`letta` 的描述中明确提到了“can learn and self-improve over time”，这表明其记忆系统支持 Agent 从经验中学习并优化自身行为。
    - **正确地址**: [https://github.com/letta-ai/letta](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - memary:

    - **分类**: **Self-improving Memory**。`Memary` 的描述中包含“self-improvement”的标签，表明其记忆系统支持 Agent 的自我提升能力。
    - **正确地址**: [https://github.com/kingjulio8238/Memary](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

  - LangChain:

    - **分类**: **Self-improving Memory (通过 Agent 和工具)**。LangChain 本身不直接提供一个“自我提升记忆”的模块，但其 Agent 框架和工具机制为实现这种记忆提供了基础。开发者可以构建自定义工具，让 Agent 能够反思其执行轨迹，并将学习到的经验（例如，通过 `VectorStoreRetriever` 存储的“经验法则”）存入长期记忆中，从而实现自我提升。

  - LangGraph:

    - **分类**: **Self-improving Memory (通过反馈循环和 Store)**。LangGraph 的图结构和 `Store` 组件非常适合实现自我提升记忆。您之前分享的电子邮件助手通过 HITL 反馈来更新 Agent 的指令和偏好，这正是 Agent 自我提升的一种形式。Agent 可以将成功的策略或失败的教训存储在 `Store` 中，并在后续执行中检索和应用。

  - Agent Developer Kit (ADK):

    - **分类**: **Self-improving Memory (通过会话分析和 MemoryService)**。ADK 的 `MemoryService` 可以存储完整的会话。通过对这些历史会话进行分析（可能需要额外的 Agent 或 LLM），可以提取出 Agent 行为的模式和改进点，并将其作为新的记忆存入 `MemoryService`，从而实现 Agent 的自我提升。

#### c. **Inside-trial Memory (任务内记忆)**

- **目的**: 专注于管理**单次任务执行过程中**不断增长的上下文信息。当一个任务非常复杂，需要多步骤、长链条的思考时，如何有效地压缩、提炼和遗忘中间过程的上下文，防止信息过载或“迷失”，是这类记忆的核心。
- 例子:
  - **长文档问答**: 在阅读一份上百页的财报并撰写摘要时，Agent 需要在阅读过程中不断总结关键信息，并忘掉不重要的细节，而不是把所有内容都堆在上下文中。
  - **复杂编码任务**: 编写一个大项目时，Agent 需要记住不同模块的接口定义和整体架构，但可能会“忘记”某个具体函数的局部变量名。
- 项目代码验证:
  - **ReSum (通义)**, **Context Folding (字节)**: 这些工作都在探索如何让 Agent 自动管理其“工作记忆”。例如，ReSum 提出了一种递归总结的机制，在任务的每一步之后，Agent 都会对当前的上下文进行一次“压缩”，只保留最重要的信息进入下一步。这被认为是未来强大 Agent 的一项基本能力。
  - Graphiti:
    - **分类**: **Inside-trial Memory (通过知识图谱管理上下文)**。虽然 `Graphiti` 主要作为 Token-level Memory 的载体，但其构建实时知识图谱的能力，可以帮助 Agent 在复杂任务执行过程中，动态地构建和维护任务相关的知识图谱，从而更有效地管理和利用上下文信息，避免上下文窗口溢出。
    - **正确地址**: [https://github.com/getzep/graphiti](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - cognee:
    - **分类**: **Inside-trial Memory (通过数据组织和上下文提供)**。`Cognee` 的核心功能是组织和连接各种数据，为 LLM 提供准确的上下文。这对于 Agent 在执行复杂任务时，动态地从大量数据中提取和管理相关信息，以避免上下文过载，是非常重要的。
    - **正确地址**: [https://github.com/cognee/cognee](vscode-file://vscode-app/d:/VS Code/Microsoft VS Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
  - LangChain:
    - **分类**: **Inside-trial Memory (通过摘要和检索)**。LangChain 的 `ConversationSummaryBufferMemory` 可以通过总结来管理长上下文。此外，结合 `VectorStoreRetriever` 进行 RAG 也是一种在任务执行中动态获取相关上下文的方式，有助于处理长文档。
  - LangGraph:
    - **分类**: **Inside-trial Memory (通过状态管理和 Store)**。LangGraph 的 `StateGraph` 本身就提供了强大的状态管理能力，可以跟踪任务执行的每一步。结合 `Store` 进行上下文的动态检索和更新，可以帮助 Agent 在长程任务中更好地管理和提炼上下文。
  - Agent Developer Kit (ADK):
    - **分类**: **Inside-trial Memory (通过会话和工具)**。ADK 的 `Session` 记录了 Agent 在单次任务中的所有交互。通过 `ToolContext.search_memory` 工具，Agent 可以在任务执行过程中动态查询相关记忆，从而管理和利用任务内的上下文。