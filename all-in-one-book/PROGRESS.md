# ChatDev 2.0 (DevAll) 实现原理全解 — 写作进度

## 章节规划

| # | 章节标题 | 文件名 | 核心覆盖 | 特殊内容 | 状态 |
|---|---------|--------|---------|---------|------|
| 0 | 序言：全书地图 | ch00-preface.md | 项目定位 / 架构全景图 / 核心概念词典 / 代码库地图 / 一次典型交互极简全流程 | | ⏳ |
| 1 | 从 YAML 到运行：数据流全景 | ch01-data-flow.md | 一个 YAML 工作流从加载、校验、解析、构图、调度、执行到输出的完整数据流 | | ⏳ |
| 2 | 图引擎核心：构建与调度 | ch02-graph-engine.md | GraphManager 构图、拓扑排序、DAG 分层执行策略 | 同类对比：LangGraph / CrewAI 的编排思路差异 | ⏳ |
| 3 | 循环与分支：环检测与动态边 | ch03-cycles-and-edges.md | Tarjan 环检测、CycleManager、超级节点图、动态边 Map/Tree 模式、边条件与 Payload 处理 | | ⏳ |
| 4 | Agent 节点：从 Prompt 到输出 | ch04-agent-node.md | AgentNodeExecutor 执行流、Prompt 构造、Provider 抽象、Tool Loop、重试机制 | prompt 分析（系统提示 / 记忆注入 / 反思提示） | ⏳ |
| 5 | 记忆系统：让 Agent 拥有长期记忆 | ch05-memory-system.md | MemoryBase 接口、SimpleMemory（FAISS 向量检索）、FileMemory、BlackboardMemory、嵌入与混合评分 | 同类对比：AutoGen / LangChain 的记忆机制差异 | ⏳ |
| 6 | 思考系统与技能系统 | ch06-thinking-and-skills.md | ThinkingManager、SelfReflection prompt 分析、AgentSkills 激活机制 | prompt 分析（反思 prompt 模板） | ⏳ |
| 7 | 其他节点类型与函数体系 | ch07-other-nodes-and-functions.md | Human / Subgraph / Python / Literal / LoopCounter / LoopTimer 节点、FunctionManager 与 FunctionCatalog | | ⏳ |
| 8 | 服务层：FastAPI 后端与实时通信 | ch08-server-layer.md | FastAPI 路由体系、WebSocket 实时推送、SSE 流式输出、批量执行、会话管理 | | ⏳ |
| 9 | 项目演进史 | ch09-evolution.md | 从 ChatDev 1.0 到 2.0 的演进脉络、关键转折点、架构变迁 | commit 历史分析 | ⏳ |
| 10 | 端到端追踪：三个典型场景 | ch10-end-to-end.md | 场景一：简单记忆写作流水线 / 场景二：带循环的代码开发流 / 场景三：深度研究多阶段流 | 串联全书 | ⏳ |

## 章节规划说明

认知路径设计：

1. **序言**（第 0 章）：建立全局画面——读者先知道"ChatDev 2.0 是什么"、"它整体长什么样"、"核心概念有哪些"
2. **数据流全景**（第 1 章）：从一个具体场景走一遍完整链路，建立对系统端到端的感知。复杂节点标注"详见第 N 章"
3. **图引擎**（第 2 章）：打开数据流中的"图构建与调度"黑盒。先讲 DAG 场景
4. **循环与分支**（第 3 章）：图引擎的进阶——支持环和条件分支的机制
5. **Agent 节点**（第 4 章）：所有节点中最核心的——LLM Agent，讲透 prompt 构造与 tool loop
6. **记忆系统**（第 5 章）：Agent 的增强能力之一——跨次执行的记忆
7. **思考与技能**（第 6 章）：Agent 的增强能力之二——反思推理与可复用技能
8. **其他节点与函数**（第 7 章）：补全节点类型全貌和函数调用体系
9. **服务层**（第 8 章）：让工作流引擎能通过 HTTP/WebSocket 对外提供服务
10. **演进史**（第 9 章）：通过 commit 历史理解项目怎么"长"成现在这样
11. **端到端追踪**（第 10 章）：三个场景完整追踪，串联前面所有章节

## 同类项目对比规划

| 对比点 | 本项目思路 | 对比项目 | 放在哪一章 |
|--------|-----------|---------|-----------|
| 工作流编排方式 | YAML 声明式 DAG + 运行时图引擎 | LangGraph（代码式状态机）/ CrewAI（角色声明式） | 第 2 章 |
| 记忆机制 | 全局共享 + Agent 挂载 + 向量检索 | AutoGen（对话缓冲为主）/ LangChain（多种 Memory 类） | 第 5 章 |

## 状态说明
- ✅ 已完成
- 🔄 进行中
- ⏳ 待开始

## 术语约定

### 行业标准术语
- **DAG** (Directed Acyclic Graph)：有向无环图
- **LLM** (Large Language Model)：大语言模型
- **Prompt**：发送给 LLM 的提示/指令
- **Token**：LLM 处理文本的基本单位
- **Embedding**：文本的向量表示
- **FAISS**：Facebook 开源的向量相似度搜索库
- **WebSocket**：全双工通信协议
- **SSE** (Server-Sent Events)：服务器推送事件
- **MCP** (Model Context Protocol)：Anthropic 提出的模型上下文协议

### 项目特有术语
- **Node**（节点）：工作流图中的一个执行单元，类似流水线上的一个工位
- **Edge**（边）：连接两个节点的有向连接，控制数据流转和条件触发
- **GraphExecutor**：图执行器——整个工作流引擎的总指挥
- **GraphContext**：图上下文——一次工作流执行的运行时状态容器
- **GraphManager**：图管理器——负责从配置构建出可执行的图结构
- **Super-node**（超级节点）：将一个环（循环）视为一个整体节点，用于拓扑排序
- **CycleManager**：环管理器——负责检测和管理图中的循环执行
- **EdgeCondition**：边条件——决定数据是否沿某条边流转的判断逻辑
- **EdgeProcessor**：边处理器——在数据流转时对 payload 进行变换
- **carry_data**：边属性——是否将上游输出携带给下游
- **keep_message**：边属性——是否在清除上下文时保留此消息
- **clear_context**：边属性——到达下游时是否清除其已有上下文
- **ThinkingManager**：思考管理器——在 Agent 生成前后插入"思考"步骤
- **AgentSkills**：Agent 技能——可按需激活的指令集和工具集
- **DesignConfig**：设计配置——YAML 解析后的顶层配置对象
- **WareHouse**：仓库——存放工作流执行产物（文件、日志等）的目录

## 下次续写指引
### 从哪里继续
从第 0 章（序言）开始写作。

### 交接备忘
- 项目共 163 个 commits，从 2026-01-07 到 2026-03-22
- 这是一个 LLM 项目，需要关注 prompt 分析
- 项目从 ChatDev 1.0（专做软件开发的多 Agent 系统）演进到 2.0（通用零代码多 Agent 编排平台）
- 核心 prompt 在 agent_executor.py 中构造，反思 prompt 在 self_reflection.py 中
- 向量检索使用 FAISS，混合评分（0.7 向量相似度 + 0.3 语义相似度）

### 待验证项
- [ ] 前端 Vue Flow 可视化编辑器的详细交互流程（如需写前端章节时验证）
- [ ] MCP 工具的具体集成方式（如需深入工具章节时验证）
