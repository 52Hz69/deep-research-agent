# 深度研究智能体（Deep Research Agent）

> 基于 LangGraph 的全栈 AI 研究智能体：DeepSeek 驱动、Tavily 联网检索，
> 自动完成「生成查询 → 多轮搜索 → 反思补漏 → 带引用作答」的完整研究循环。

项目移植改造自 Google 官方示例
[gemini-fullstack-langgraph-quickstart](https://github.com/google-gemini/gemini-fullstack-langgraph-quickstart)，
将模型层从 Gemini 迁移至 DeepSeek、搜索层从 Google Search 迁移至 Tavily，
并解决了迁移过程中的若干兼容性问题（见「改造点与踩坑记录」）。

![架构图](agent.png)

## 工作原理

智能体以 LangGraph 状态图为核心，研究流程是一个可循环的图：

1. **生成查询** —— 根据用户问题，批量生成多组初始搜索词
2. **联网检索** —— 调用 Tavily API 逐词搜索，汇总网页摘要与原始链接
3. **反思评估** —— LLM 检查现有证据能否回答问题，找出知识缺口并生成补充搜索词
4. **循环迭代** —— 重复「检索 → 反思」，直到证据充分或达到轮数上限
5. **综合作答** —— 输出带引用标注的最终答案，每个结论可溯源到具体网页

## 技术栈

| 层级 | 技术 |
|---|---|
| 智能体编排 | LangGraph + LangChain |
| 大模型 | DeepSeek（deepseek-chat） |
| 联网搜索 | Tavily Search API |
| 后端服务 | LangGraph Server（流式输出） |
| 前端 | React + Vite + TypeScript + Tailwind CSS |

![界面预览](app.png)

## 改造点与踩坑记录

相比原模板的主要改动：

- **LLM 迁移**：Gemini → DeepSeek，调整【configuration.py 中的模型接入方式 / 参数】
- **搜索迁移**：Google Search → Tavily，重写【utils.py 中的搜索封装】
- **结构化输出兼容性修复**：【描述问题现象 → 原因 → 你的解决方案】

踩坑实录（迁移到 DeepSeek 时真实遇到的问题）：

1. 【问题现象，如：按 Pydantic Schema 要求结构化输出时 DeepSeek 返回格式不稳定/报错】
   → 【原因分析】
   → 【解决方案，如：调整 Schema 字段类型 / 换用 JSON 模式 / 加 prompt 约束】
2. 【第二个问题，同上格式】

> 写 2~3 个就够，每个都按「现象 → 原因 → 解决」三段式。

## 本地运行

### 准备

- Python 3.11+ / Node.js 18+
- API Key：[DeepSeek 平台](https://platform.deepseek.com/)、[Tavily](https://app.tavily.com/)

### 1. 配置密钥

复制 `.env.example` 为 `backend/.env`，填入真实 Key。

### 2. 启动后端

