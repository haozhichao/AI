# Projects

这里存放可运行源码、实验项目、demo、测试和脚本。过程笔记放 `practice/03-project-notes/`，不要和源码混在一起。

## 方向目录

- `rag/`：RAG 问答、检索、chunking、rerank、引用与评估。
- `agents/`：Agent 工具调用、端到端任务、执行边界和失败兜底。
- `mcp/`：MCP Server、工具接入、协议练习。
- `evals-observability/`：evals、指标、日志、可观测性、防坏策略。
- `workflow-automation/`：工作中重复任务的 AI 化改造。

## 单个项目建议结构

```text
<project-name>/
  README.md
  src/
  tests/
  docs/
  scripts/
  data/
```

按项目需要增减目录。不要为了空结构而创建复杂工程；先让最小 demo 跑起来。
