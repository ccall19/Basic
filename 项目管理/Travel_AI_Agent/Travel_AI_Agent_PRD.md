# 智能旅游助手 AI Agent 项目需求文档 (PRD)

| 版本号 | 日期 | 修改人 | 描述 |
| :--- | :--- | :--- | :--- |
| v1.0 | 2026-01-03 | GitHub Copilot | 初始版本创建 |

---

## 1. 项目概述 (Project Overview)

### 1.1 项目背景
随着人工智能技术的发展，传统的旅游 OTA（在线旅游代理）模式正在向智能化、个性化方向转变。本项目旨在开发一款基于大语言模型（LLM）的智能旅游助手，通过 AI Agent 技术，为用户提供从灵感激发、行程规划、天气查询到最终预订的一站式服务。

### 1.2 项目目标
*   **核心目标**：构建一个具备多轮对话能力、能调用外部工具（Tools）的 AI Agent，解决用户旅游场景下的复杂需求。
*   **学习目标**：深入实践 LangChain 框架、Prompt Engineering、RAG（检索增强生成）以及 Java 与 Python 微服务架构的协同。
*   **商业模拟**：模拟真实商业场景，实现从需求沟通到支付出票的闭环（或高保真模拟）。

### 1.3 适用范围
*   **地域范围**：中国境内旅游。
*   **终端平台**：Web 端（PC/Mobile 适配）。

---

## 2. 用户角色 (User Personas)

| 角色 | 描述 | 核心需求 |
| :--- | :--- | :--- |
| **普通游客** | 计划出行的个人用户 | 希望快速获得靠谱的旅游攻略，查询目的地天气，并便捷地完成机票/火车票预订。 |
| **系统管理员** | 运维与运营人员 | 查看用户数据，管理 API Key，监控系统运行状态。 |

---

## 3. 业务流程 (Business Process)

1.  **用户登录**：用户通过 Web 端登录系统（Java 后端处理）。
2.  **需求输入**：
    *   **表单**：用户填写基础信息（目的地、出发地、大致日期、预算范围）。
    *   **对话**：用户通过聊天窗口补充细节（如：“我想去人少景美的地方”、“要适合带孩子”）。
3.  **意图识别与澄清 (AI)**：
    *   Agent 分析用户输入。
    *   若信息缺失（如未定具体日期），Agent 发起反问进行多轮对话。
4.  **方案生成与查询 (AI + Tools)**：
    *   **天气查询**：调用高德天气 API。
    *   **攻略生成**：结合 LLM 知识库与网络检索（RAG）生成个性化攻略。
    *   **票务搜索**：调用携程 API 查询航班/火车票。
5.  **预订与支付**：
    *   用户确认行程与票务信息。
    *   系统发起支付流程（对接真实支付或模拟）。
    *   出票并生成最终行程单。
6.  **历史记录**：行程存入数据库，用户可在“我的行程”中查看。

---

## 4. 功能需求 (Functional Requirements)

### 4.1 用户模块 (Java + MySQL)
*   **注册/登录**：支持手机号/邮箱注册，JWT Token 鉴权。
*   **个人中心**：修改个人信息，查看历史订单/对话记录。
*   **鉴权中心**：统一管理 Python AI 服务的调用权限。

### 4.2 AI Agent 核心模块 (Python + LangChain)
*   **多轮对话管理 (Memory)**：
    *   记录上下文，确保 Agent 记得用户之前的需求（如“前面说的那个酒店太贵了，换一个”）。
*   **意图识别 (Router)**：
    *   区分用户是在闲聊、问天气、查攻略还是买票。
*   **工具调用 (Tools)**：
    *   **天气工具**：封装高德天气 API，支持实时天气和预报。
    *   **POI 检索工具**：封装高德 POI API，查询景点、美食、酒店信息。
    *   **票务工具**：封装携程 API（或模拟层），支持航班/火车票的查价、余票查询、下单。
*   **攻略生成 (RAG)**：
    *   利用向量数据库存储本地旅游知识库。
    *   结合搜索引擎（如 Tavily/SerpApi）获取最新旅游资讯。
    *   生成结构化的每日游玩计划。

### 4.3 前端交互 (Vue.js)
*   **混合交互界面**：
    *   左侧/顶部：结构化表单（目的地、时间选择器）。
    *   右侧/主要区域：Chat 界面，支持 Markdown 渲染（展示攻略表格、图片）。
*   **卡片式消息**：
    *   天气信息以卡片形式展示。
    *   机票/火车票推荐以列表卡片展示，含“预订”按钮。

---

## 5. 技术架构 (Technical Architecture)

### 5.1 技术栈
*   **前端**：Vue 3, Element Plus / Ant Design Vue, Axios。
*   **业务后端**：Java (Spring Boot), MyBatis-Plus, Spring Security。
*   **AI 服务端**：Python (FastAPI / Flask), LangChain, LangGraph (可选，用于复杂状态管理)。
*   **数据库**：
    *   MySQL：存储用户、订单、结构化行程数据。
    *   Redis：缓存 Session、对话历史（短期）。
    *   Milvus / Chroma：向量数据库，用于 RAG 攻略检索。
*   **外部 API**：
    *   地图/天气：高德开放平台 (Amaps)。
    *   OTA：携程开放平台 (Ctrip) / 飞猪 API。
    *   LLM：OpenAI / Claude / 国内大模型 (DeepSeek/Qwen) API。

### 5.2 系统交互图
```mermaid
graph TD
    User[用户 (Vue Frontend)] <--> Java[业务后端 (Spring Boot)]
    Java <--> DB[(MySQL)]
    Java -- 转发复杂指令 --> Python[AI Agent 服务 (FastAPI)]
    Python <--> LLM[大模型 API]
    Python <--> Tools[工具集]
    Tools <--> Amap[高德 API]
    Tools <--> Ctrip[携程 API]
    Python <--> VectorDB[(向量数据库)]
```

---

## 6. 非功能需求 (Non-Functional Requirements)

*   **响应速度**：
    *   普通业务接口 < 500ms。
    *   AI 流式对话首字响应 < 2s。
*   **稳定性**：
    *   API 调用失败需有重试机制或降级方案（如 API 超限时返回缓存数据）。
*   **安全性**：
    *   用户密码加密存储。
    *   支付接口需符合 PCI DSS 标准（若涉及真实卡号），建议对接支付宝/微信沙箱环境。
    *   API Key 不得暴露在前端。

---

## 7. 风险与应对 (Risks & Mitigation)

| 风险点 | 应对策略 |
| :--- | :--- |
| **API 成本与限流** | 开发阶段使用 Mock 数据；生产环境增加缓存层；申请高德/携程测试额度。 |
| **支付安全** | 优先使用沙箱环境进行模拟支付；若必须真实支付，仅小额测试并严格风控。 |
| **AI 幻觉** | 在 Prompt 中严格限制 Agent 仅依据工具返回的数据回答；攻略生成注明“仅供参考”。 |
| **复杂意图理解** | 使用 LangGraph 设计确定的状态机，限制 Agent 在特定状态下的行为边界。 |

---

## 8. 开发计划 (Roadmap)

### Phase 1: 基础框架搭建
*   搭建 Spring Boot + Vue 前后端框架。
*   实现用户登录注册。
*   搭建 Python FastAPI 服务，跑通 LangChain Hello World。

### Phase 2: 核心工具链开发
*   对接高德天气、POI API。
*   实现基础的对话意图识别。
*   前端实现聊天界面。

### Phase 3: 票务与交易闭环
*   对接携程 API（查询接口）。
*   实现模拟下单与支付流程。
*   数据库设计订单表结构。

### Phase 4: 攻略增强与 RAG
*   引入向量数据库。
*   实现基于 RAG 的攻略生成。
*   系统联调与 UI 美化。

---

## 9. 关键技术细节补充 (Key Technical Details)

### 9.1 交互协议：流式响应 (SSE)
为了提供类似 ChatGPT 的打字机体验，Java 与 Python 之间采用流式交互。
*   **前端 (Vue)**: 使用 `EventSource` 或 `fetch` (配合流读取) 监听 Java 提供的 SSE 接口 `/api/chat/stream`。
*   **业务后端 (Java)**: 接收前端请求，鉴权后，通过 HTTP Client (如 OkHttp 或 WebClient) 流式调用 Python 服务的 `/agent/chat` 接口。Java 收到 Python 的每一个 chunk (数据块) 后，立即转发给前端。
*   **AI 服务 (Python)**: 使用 LangChain 的 `StreamingCallbackHandler` 或 `astream_events` 将大模型的输出实时推送到 HTTP 响应流中。

### 9.2 状态管理：共享 Redis (Shared Memory)
解决“Java 管用户，Python 管对话”的分离问题。
*   **架构**：Java 和 Python 连接同一个 Redis 实例。
*   **流程**：
    1.  Java 接收用户消息，生成或获取 `conversation_id`。
    2.  Java 将 `user_id` 和 `conversation_id` 传给 Python。
    3.  Python 使用 `RedisChatMessageHistory` (LangChain 组件) 读取和保存历史消息。
    4.  **优势**：Python 服务无状态，可随时重启；Java 可以随时查看历史记录用于分析。

### 9.3 核心数据结构：结构化行程单 (Itinerary Schema)
AI 生成的攻略必须是结构化的 JSON，以便前端渲染和数据库存储。
**示例 Schema**:
```json
{
  "title": "北京三日亲子游",
  "total_budget": 5000,
  "days": [
    {
      "day_index": 1,
      "date": "2026-05-01",
      "activities": [
        {
          "time": "09:00",
          "location": "故宫博物院",
          "type": "attraction", // 类型：景点、餐饮、交通、住宿
          "description": "参观三大殿，建议提前预约",
          "cost": 60,
          "poi_id": "amap_12345" // 关联高德 POI ID，用于前端地图展示
        },
        {
          "time": "12:00",
          "location": "四季民福烤鸭",
          "type": "dining",
          "description": "特色午餐，需排队",
          "cost": 300
        }
      ]
    }
  ]
}
```

### 9.4 简易评估与反馈 (User Feedback)
初期不引入复杂的自动化评估框架，而是通过用户反馈来优化。
*   **功能**：在每条 AI 回复下方增加“点赞”和“点踩”按钮。
*   **逻辑**：
    *   用户点击“点踩”时，前端弹窗询问原因（如：攻略不合理、价格错误、幻觉）。
    *   Java 后端记录该条对话的 `message_id` 和反馈内容。
    *   开发者定期查看差评记录，优化 Prompt 或知识库。
