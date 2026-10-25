# Legal Brain (企业法务智库)

基于 LangGraph 和多模态大模型的企业法务知识库管理与智能问答系统

> 本项目由一套面向商品/设备操作手册的通用 RAG 引擎改造而来，
> 迁移为**面向公司法务团队、聚焦法律法规与合同范本**的法务知识库。

## 项目简介

Legal Brain 是一个为企业法务团队提供智能知识库服务的系统，支持法律法规、合同范本、规章制度等多格式文档导入、法务知识图谱构建和智能问答。系统采用混合检索策略，融合向量检索、知识图谱查询和网络搜索，为用户提供精准、可溯源的法律知识查询服务。

## 功能特性

### 文档导入
- 支持 PDF、Markdown、图片等多种格式
- 复杂 PDF 智能转换（支持表格、公式、图片）
- 自动提取法规名称 / 合同类型
- 法务知识图谱自动构建（法规层级、法条结构、权利义务关系等）

### 智能查询
- 多路混合检索（向量检索 + HyDE + 知识图谱 + 网络搜索）
- RRF 融合排序 + 重排序优化
- SSE 流式响应，实时返回答案
- 完整的会话历史管理
- 法条出处引用 + 免责声明（保障法律回答可溯源）

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                        FastAPI API 层                        │
│            (导入服务 :8000 / 查询服务 :8001)                  │
├─────────────────────────────────────────────────────────────┤
│                      LangGraph 工作流引擎                     │
│          ┌──────────────────┬──────────────────┐            │
│          │    导入流程 (8节点)  │    查询流程 (9节点)  │            │
│          └──────────────────┴──────────────────┘            │
├─────────────────────────────────────────────────────────────┤
│                         工具层                               │
│  ┌─────────┬─────────┬─────────┬─────────┬─────────┐       │
│  │ LLM工具  │ 向量化工具 │ 重排序工具 │ SSE工具  │ 任务管理  │       │
│  └─────────┴─────────┴─────────┴─────────┴─────────┘       │
├─────────────────────────────────────────────────────────────┤
│                        存储层                                │
│  ┌─────────┬─────────┬─────────┬─────────┐                 │
│  │ Milvus  │  Neo4j  │ MongoDB │  MinIO  │                 │
│  │ (向量库) │ (图数据库) │ (文档库) │ (对象存储) │                 │
│  └─────────┴─────────┴─────────┴─────────┘                 │
└─────────────────────────────────────────────────────────────┘
```

## 法务知识图谱 Schema

系统采用法务领域专属的实体与关系建模（详见 `docs/法务知识库改造方案.md`）：

- **实体类型**：Law（法律）、Regulation（法规）、Article（法条）、Clause（合同条款）、
  Subject（主体）、Obligation（义务）、Right（权利）、Condition（条件）、
  Liability（责任）、TimeLimit（时效）、ContractType（合同类型）等
- **关系类型**：SUPERSEDES（新法取代旧法）、REGULATES（法条规范主体）、
  GRANTS（授予权利）、IMPOSES（施加义务）、REFERENCES（援引参见）等

## 技术栈

| 类别 | 技术 |
|------|------|
| 后端框架 | FastAPI |
| 工作流引擎 | LangGraph |
| 向量数据库 | Milvus |
| 图数据库 | Neo4j |
| 文档数据库 | MongoDB |
| 对象存储 | MinIO |
| 大模型 | DashScope API (通义千问) |
| 向量化 | BGE-M3 (稠密+稀疏向量) |
| PDF处理 | MinerU |

## 快速开始

前置依赖：Milvus、Neo4j、MongoDB、MinIO 四件套（本地容器或已有实例均可），
PDF 解析依赖 [MinerU](https://github.com/opendatalab/MinerU)，BGE 系列模型首次运行会自动下载。

```bash
# 1. 安装依赖
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r knowledge/requirements.txt

# 2. 配置环境变量（按提示填入真实值）
cp knowledge/.env.example knowledge/.env

# 3. 启动两个服务
python knowledge/api/import_router.py   # 导入服务：http://127.0.0.1:8000
python knowledge/api/query_router.py    # 查询服务：http://127.0.0.1:8001
```

前端页面为纯静态文件，直接打开即可：`knowledge/front/import.html`（文档导入）、
`knowledge/front/chat.html`（智能问答）。

## 配置

所有配置项集中在 `knowledge/.env`（模板见 `knowledge/.env.example`），按服务分组：

| 分组 | 关键变量 |
|------|----------|
| 大模型 | `OPENAI_API_KEY`、`OPENAI_API_BASE`、`MODEL`、`ITEM_MODEL`、`KG_MODEL`、`VL_MODEL` |
| 向量 / 重排序模型 | `BGE_M3`、`BGE_M3_PATH`、`BGE_RERANKER_LARGE`、`EMBEDDING_DIM` |
| 向量数据库 | `MILVUS_URL`、`CHUNKS_COLLECTION`、`ENTITY_NAME_COLLECTION`、`MILVUS_METRIC_TYPE` |
| 图数据库 | `NEO4J_URI`、`NEO4J_USERNAME`、`NEO4J_PASSWORD` |
| 文档库 / 对象存储 | `MONGO_URL`、`MINIO_ENDPOINT`、`MINIO_ACCESS_KEY`、`MINIO_SECRET_KEY` |
| 检索参数 | `RRF_K`、`RERANK_MAX_TOP_K`、`KG_MAX_TOTAL_TRIPLES` 等（均有默认值，可不填） |

> `knowledge/.env` 已被 `.gitignore` 忽略，**切勿提交真实密钥**，也不要把它硬编码进任何 `.py` 文件。
> 密钥一旦进入过 Git 历史即视为泄露，应到对应平台轮换。

## 许可证

本项目采用 [MIT License](LICENSE)。
