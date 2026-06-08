# 第 8 章：AgentSeek CLI（下）— 用 skills 命令安装开发辅助技能

## Skills 是什么

在 AgentSeek 体系中，**Skill（技能）** 是一种教会 Agent 如何执行特定任务的知识包。不同于 Plugin（运行时扩展），Skill 以自然语言指令为核心，通过一个 `SKILL.md` 文件定义：

- **何时触发**：描述技能适用的场景
- **知识内容**：框架 API 用法、常见陷阱、验证过的修复方案
- **工作流程**：分步骤的操作指南

Skills 安装到项目的 `.agents/skills/` 目录后，AI 编码助手在工作时会自动发现并使用这些技能——当你遇到相关问题时，助手会调用对应的 Skill 提供专业指导。

## 为什么需要开发技能

考虑一个典型场景：你基于 `deepagent/research` 模板开始开发，想要：

1. 将模型从 OpenAI 切换到 Anthropic，但 Middleware 报错了
2. 给 Agent 添加流式输出（Streaming），但不确定正确的写法
3. 查看 LangSmith 中某次运行的 Trace，想导出分析

这些问题的答案散落在不同版本的文档、GitHub Issues 和 Discord 讨论中。而 **Skills 把经过验证的解决方案集中打包**——安装后，AI 编码助手就像有了一位熟悉 LangChain 生态的资深工程师在旁指导。

## 安装 Skills

### 查看已安装的技能

```bash
agentseek skills list
```

如果是新项目，输出为空：

```
Installed skills (in .agents/skills/):
  (none)
```

### 安装技能的命令格式

```bash
agentseek skills add <registry> --skill <skill-name>
```

- **`<registry>`**：技能注册源，格式为 `owner/repo`（GitHub 仓库）
- **`--skill`**：指定要安装的技能名称

技能会被下载到项目的 `.agents/skills/<skill-name>/` 目录。

## 安装 LangChain 开发指南技能

这是我们为 LangChain / LangGraph 开发者准备的核心技能，覆盖了开发中最常遇到的坑和最佳实践。

### 安装命令

```bash
agentseek skills add ob-labs/agentseek-skills --skill langchain-dev-guide
```

安装完成后确认：

```bash
agentseek skills list
```

```
Installed skills (in .agents/skills/):
  langchain-dev-guide  - LangChain/LangGraph 开发陷阱与验证修复
```

### 技能覆盖的场景

`langchain-dev-guide` 技能在以下场景会被 AI 编码助手自动激活：

| 场景 | 技能提供的帮助 |
|------|--------------|
| OpenAI 兼容模型接入 | 正确的 `base_url` 配置、模型 ID 格式 |
| Middleware 配置 | 避免常见的顺序陷阱、调试中间件链 |
| Streaming 流式输出 | LangGraph 中正确的流式写法 |
| 多 Agent 编排 | Supervisor / Swarm 模式的注意事项 |
| 工具定义 | `@tool` 装饰器的正确用法和常见错误 |
| 状态管理 | Graph State 的更新模式 |
| Human-in-the-Loop | 中断点和审批流程的正确实现 |

### 实际效果演示

安装此技能后，当你在 Claude Code 中遇到问题：

```
我想在 graph.py 中把模型从 OpenAI 切换到硅基流动，
但运行时报 "Invalid model" 错误
```

AI 编码助手会自动调用 `langchain-dev-guide` 技能，给出经过验证的解决方案：

```python
from langchain_openai import ChatOpenAI

# 硅基流动通过 OpenAI 兼容接口接入
model = ChatOpenAI(
    model="Qwen/Qwen3-8B",          # 硅基流动的模型 ID 格式
    api_key=os.environ["SILICONFLOW_API_KEY"],
    base_url="https://api.siliconflow.cn/v1",  # 必须指定 base_url
)
```

而不是给你一个可能过时的、未验证的答案。

## 安装 LangSmith Trace 技能

这个技能帮助你高效使用 LangSmith 的 Tracing 功能——从添加追踪到查询和导出 Trace 数据。

### 安装命令

```bash
agentseek skills add ob-labs/agentseek-skills --skill langsmith-trace
```

确认安装：

```bash
agentseek skills list
```

```
Installed skills (in .agents/skills/):
  langchain-dev-guide  - LangChain/LangGraph 开发陷阱与验证修复
  langsmith-trace      - LangSmith Tracing 添加与查询
```

### 技能覆盖的场景

| 场景 | 技能提供的帮助 |
|------|--------------|
| 添加 Tracing | 环境变量配置、代码内手动 Trace |
| 自定义 Run Name | 给 Trace 设置有意义的标识 |
| 查询 Trace | 按时间/状态/标签过滤 Run |
| 导出数据 | 将 Trace 数据导出用于评估 |
| 排查问题 | 通过 Trace 定位 Agent 执行异常 |

### 实际效果演示

当你在开发中问：

```
如何查看昨天所有失败的 Agent 运行记录？
```

AI 编码助手会调用 `langsmith-trace` 技能，指导你使用 LangSmith SDK：

```python
from langsmith import Client

client = Client()

# 查询昨天所有失败的 Run
runs = client.list_runs(
    project_name="deep-research",
    filter='and(eq(status, "error"), gt(start_time, "2026-06-07T00:00:00Z"))',
)

for run in runs:
    print(f"Run: {run.name} | Error: {run.error}")
```

## 项目目录变化

安装完两个技能后，项目结构新增了 `.agents/skills/` 目录：

```
.
├── .agents/
│   └── skills/
│       ├── langchain-dev-guide/
│       │   └── SKILL.md          # 技能定义文件
│       └── langsmith-trace/
│           └── SKILL.md
├── src/
├── frontend/
├── .env
└── ...
```

> `.agents/skills/` 目录建议纳入版本控制（git commit），这样团队其他成员 clone 项目后即可享受相同的 AI 辅助能力。

## 完整工作流回顾

从零到可开发状态的完整流程：

```bash
# 1. 安装 CLI
uv tool install agentseek-cli

# 2. 创建模板应用
agentseek create deepagent/research

# 3. 进入项目
cd deep_research

# 4. 配置环境变量
cp .env.example .env
# 编辑 .env，填入你的 API Keys

# 5. 安装开发技能
agentseek skills add ob-labs/agentseek-skills --skill langchain-dev-guide
agentseek skills add ob-labs/agentseek-skills --skill langsmith-trace

# 6. 安装依赖
uv sync
cd frontend && npm install && cd ..

# 7. 启动开发
uv run langgraph dev --no-browser  # 后端
cd frontend && npm run dev          # 前端（新终端）
```

完成这些步骤后，你拥有了：

- 一个可运行的 Agent 应用（模板提供）
- 全链路可观测性（LangSmith Tracing 默认开启）
- AI 编码助手的专业指导（Skills 加持）

接下来就是真正的开发——修改 Agent 逻辑、添加工具、调整 Prompt、观察 Trace、持续迭代。

## 常用命令速查

```bash
# 查看已安装技能
agentseek skills list

# 从注册源安装技能
agentseek skills add <owner/repo> --skill <skill-name>

# 安装 LangChain 开发指南
agentseek skills add ob-labs/agentseek-skills --skill langchain-dev-guide

# 安装 LangSmith Trace 技能
agentseek skills add ob-labs/agentseek-skills --skill langsmith-trace
```

## 小结

本章你学会了：

1. 理解 Skills 的作用——让 AI 编码助手具备框架级专业知识
2. 安装 `langchain-dev-guide` 技能——覆盖 LangChain/LangGraph 开发常见陷阱
3. 安装 `langsmith-trace` 技能——高效使用 LangSmith Tracing
4. 完整的从零到可开发状态的工作流

至此，实战篇的基础准备工作完成。你已经具备了：模板应用 + 可观测性 + AI 辅助开发技能。后续进阶篇将深入更高级的 Agent 架构模式和生产部署实践。
