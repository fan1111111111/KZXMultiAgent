# KZXMultiAgent

面向医疗知识问答与智能导诊场景的多 Agent 后端系统。项目基于 FastAPI、LangGraph、LangChain、Neo4j、Milvus、PostgreSQL、Redis 与 MinIO 构建，围绕“用户问题理解、任务分发、专业 Agent 执行、多源知识检索、会话记忆、问诊状态管理”形成一套完整的医疗智能体服务框架。

> 本项目用于技术展示与工程实践，不替代医生诊断或真实医疗决策。

## 项目定位

KZXMultiAgent 将医疗问答拆分为多个职责清晰的 Agent，由 Supervisor Agent 统一识别用户意图并调度专业能力：

- 智能问诊 Agent：根据症状进行多轮追问、急症识别、候选疾病推理与科室建议。
- 医学知识 Agent：融合文档 RAG、知识图谱检索与结构化数据查询，生成有依据的医学知识回答。
- 药物 Agent：处理用药咨询、药物相互作用、处方安全等问题。
- 报告解读 Agent：面向检验报告、影像报告等医学材料进行解释。
- 运营数据 Agent：支持面向内部运营人员的自然语言数据查询。

系统不是单轮 Chatbot，而是一个具备任务路由、状态机、短期记忆、长期记忆、知识库检索和外部数据源接入能力的后端服务。

## 核心特性

- 多 Agent 调度：Supervisor Agent 根据用户输入选择问诊、知识问答、药物、报告或运营数据 Agent。
- LangGraph 状态机问诊：将问诊流程拆分为上下文加载、急症识别、症状标准化、图谱检索、追问、结论生成和记录保存。
- 多源 RAG：支持 Milvus 文档向量检索、Neo4j 医学知识图谱检索、PostgreSQL 结构化查询的融合回答。
- 会话记忆：Redis 负责会话 checkpoint 与问诊活跃状态，Milvus Store 承载长期记忆检索。
- 医疗安全策略：在问诊流程中加入急症识别，对高风险症状优先提示线下急诊。
- 工程化基础设施：提供 FastAPI 接口、SSE 流式输出、统一异常处理、日志中间件、健康检查和数据库迁移。
- 本地可复现环境：通过 Docker Compose 启动 PostgreSQL、Redis Stack、Milvus、MinIO、Neo4j 等依赖。

## 架构概览

```text
Client
  |
  v
FastAPI /api/v1/chat
  |
  v
Supervisor Agent
  |
  +-- Inquiry Agent
  |     +-- LangGraph state machine
  |     +-- Neo4j disease-symptom graph
  |     +-- PostgreSQL patient records
  |     +-- Redis inquiry state
  |
  +-- Knowledge Agent
  |     +-- Milvus document RAG
  |     +-- Neo4j graph RAG
  |     +-- NL2SQL over PostgreSQL
  |     +-- hallucination check
  |
  +-- Drug Agent
  +-- Report Agent
  +-- Operation Agent
  |
  v
Redis / Milvus / Neo4j / PostgreSQL / MinIO
```

## 技术栈

- Web Framework：FastAPI、Starlette、Uvicorn
- Agent Framework：LangGraph、LangChain、LangChain DeepSeek
- LLM：DeepSeek Chat，预留通用 OpenAI-compatible 接入
- Vector Database：Milvus
- Graph Database：Neo4j
- Relational Database：PostgreSQL、SQLAlchemy Async、Alembic
- Cache & Checkpoint：Redis Stack、langgraph-checkpoint-redis
- Object Storage：MinIO
- Document Processing：LlamaIndex、MinerU 接入封装
- Observability：Loguru、自定义请求日志中间件、健康检查接口

## 目录结构

```text
.
├── alembic/                  # 数据库迁移脚本
├── data/raw/                 # 示例医学数据
├── scripts/                  # 基础数据与索引初始化脚本
├── src/
│   ├── agents/               # Supervisor 与各专业 Agent
│   │   ├── inquiry/          # 智能问诊状态机
│   │   ├── knowledge/        # RAG、图谱检索、NL2SQL、幻觉检测
│   │   ├── tools/            # Agent 工具封装
│   │   └── workers/          # 专业 Worker Agent
│   ├── api/routers/          # FastAPI 路由
│   ├── core/                 # 配置、异常、基础模型、日志
│   ├── infra/                # 数据库、Redis、Milvus、Neo4j、MinIO 客户端
│   ├── middlewares/          # 中间件
│   ├── modules/medical/      # 医疗业务数据模型
│   └── utils/                # 通用工具
├── test/                     # 测试用例
├── docker-compose.yml        # 本地基础设施编排
├── requirements.txt
└── README.md
```

## 快速开始

### 1. 创建 Python 环境

```bash
conda create -n kzx-agent python=3.13
conda activate kzx-agent
pip install -r requirements.txt
```

### 2. 配置环境变量

在项目根目录创建 `.env` 文件，按需填写以下配置。`.env` 已被 `.gitignore` 忽略，不应提交到仓库。

```env
APP_NAME=tiangong-agent
APP_ENV=dev
APP_DEBUG=true

DB_HOST=localhost
DB_PORT=5432
DB_USER=medical
DB_PASSWORD=medical123
DB_NAME=medical_db

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=knowledge-docs
MINIO_SECURE=false

MILVUS_HOST=localhost
MILVUS_PORT=19530

NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=medical123

DEEPSEEK_API_KEY=your_api_key
CHAT_MODEL=deepseek-v4-flash
EMBEDDING_MODEL=text-embedding-v3
DASHSCOPE_API_KEY=your_dashscope_api_key
```

### 3. 启动基础设施

```bash
docker compose up -d
```

包含服务：

- PostgreSQL：结构化医疗数据、问诊记录、业务实体
- Redis Stack：会话状态、LangGraph checkpoint
- Milvus：文档向量、长期记忆、语义检索
- MinIO：知识文档对象存储
- Neo4j：疾病、症状、药物、科室等医学知识图谱

### 4. 初始化数据库

```bash
alembic upgrade head
python scripts/init_postgres.py
python scripts/init_neo4j.py
python scripts/init_symptom_index.py
```

### 5. 启动服务

```bash
uvicorn src.main:app --host 0.0.0.0 --port 8080 --reload
```

服务启动后可访问：

- 健康检查：`GET /health`
- 依赖检查：`GET /health/deps`
- 非流式对话：`POST /api/v1/chat`
- SSE 流式对话：`POST /api/v1/chat/stream`

## API 示例

```bash
curl -X POST http://localhost:8080/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "1",
    "session_id": "demo-session",
    "message": "我最近头痛发热，应该挂什么科？",
    "patient_id": 1
  }'
```

响应格式：

```json
{
  "reply": "系统生成的问诊或问答结果",
  "session_id": "demo-session"
}
```

## 关键设计

### Supervisor + Worker Agent

Supervisor Agent 只负责意图识别、上下文组织和工具调度，不直接回答医疗问题。医疗相关任务交由专业 Worker Agent 处理，降低单个 Agent 的职责复杂度，也便于后续扩展新能力。

### 智能问诊状态机

问诊流程使用 LangGraph 建模，每一轮对话都会恢复上一轮状态，并根据当前阶段进入不同节点：

```text
load_context
  -> check_emergency
  -> extract_symptoms
  -> query_neo4j
  -> ask_symptoms / conclude
  -> save_record
```

这种设计让问诊过程具备可追踪状态、可插拔节点和稳定的多轮控制能力。

### 多通道知识融合

知识问答模块并行检索文档向量库和知识图谱，必要时接入结构化数据库查询，再通过融合提示词生成回答，并加入幻觉检测逻辑，减少脱离证据的回答。

### 记忆体系

- 短期记忆：Redis 保存会话 checkpoint 与当前问诊流程状态。
- 长期记忆：Milvus Store 保存用户长期健康信息，用于跨会话检索。
- 业务记录：PostgreSQL 保存患者档案、问诊记录、药品与疾病等结构化数据。

## 测试

```bash
pytest
```

当前测试重点覆盖 Supervisor Agent 的调度行为，后续可继续补充问诊状态机、RAG 检索和 API 层集成测试。

## 面试展示重点

- 不是单 Agent 包装，而是具备 Supervisor、专业 Worker、状态机和多源知识检索的系统化设计。
- 同时覆盖 LLM 应用层、后端 API、数据库建模、图谱检索、向量检索和基础设施编排。
- 对医疗场景加入了急症识别、追问收敛、候选疾病推理、记录保存和幻觉检测等安全与可靠性设计。
- 工程结构按 API、Agent、Infra、Core、Module 分层，便于继续扩展认证、权限、前端和生产部署能力。

## 免责声明

本项目仅用于学习、研究和技术展示。系统输出不能作为医学诊断、治疗或用药依据；真实医疗场景应由具备资质的医生进行判断。
