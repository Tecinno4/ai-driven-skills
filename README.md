# ai-driven-skills

面向 [Claude Code](https://claude.com/claude-code) 的轻量级 AI 驱动开发 harness。

它通过一组 Skills 引导 Claude 理解需求、沉淀文档、规划任务并协助实现，让新手不用理解复杂流程，也能快速启动新项目或持续维护老项目。也支持构建你自己的定制化的harness。

如果你觉得完整的 harness / GSD / superpower 工作流太重，或者平时主要靠 vibe coding 推进项目，但又希望项目能留下健康、可延续的文档和任务记录，`ai-driven-skills` 就是为这个场景准备的。


## 它适合谁

- 想用 Claude Code 快速启动一个新项目的人。
- 想维护老项目，但项目缺少清晰需求、架构文档和任务记录的人。
- 习惯和 AI 边聊边写代码，但希望减少“聊完就丢上下文”的人。
- 不熟悉复杂 agent harness，但希望模型能按更稳定的流程协助开发的人。
- 希望每次从新窗口继续开发时，Claude 仍然能快速理解项目状态的人。

## 它解决什么问题

普通 AI 编程很容易出现几个问题：

- 需求只存在聊天记录里，过几天就很难追溯。
- 新开窗口后，模型需要重新理解项目。
- 文档、测试、实现经常脱节。
- 代码能跑，但没人记录为什么这么做、改了什么、还缺什么。

`ai-driven-skills` 会把项目理解、需求、路线图、任务执行记录、测试计划和变更报告落到 `docs/ads/` 目录中。后续每次开发，Claude 都可以从这些文件恢复上下文，而不是完全依赖旧聊天。

## 安装

需要先安装 [Claude Code](https://claude.com/claude-code) 和 Node.js 14+。下面的安装命令使用 `npx`，它是 npm 自带的包执行方式，可以直接运行最新版本而不需要先全局安装。

全局安装，推荐大多数用户使用：

```
npx ai-driven-skills@latest install
```

安装后重启 Claude Code，或运行 `/help` 让 Claude 重新加载 skills。

如果只想安装到当前项目：

```
npx ai-driven-skills@latest install --project
```

常用安装命令：

```
npx ai-driven-skills@latest list
npx ai-driven-skills@latest install -f
npx ai-driven-skills@latest uninstall
npx ai-driven-skills@latest help
```

## 快速开始

在你的项目里打开 Claude Code，然后按这个顺序使用：

```
# 1. 初始化项目文档和路线图
/ads-init

# 2. 推进下一个开发任务
/ads-next

# 3. 查看当前项目状态
/ads-report
```

如果你已经通过普通对话写了一些代码，后来才想补上 ADS 记录，可以运行：

```
/ads-record
```

如果你想基于这套思路做自己的 harness，而不是持续修改 `ai-driven-skills`，可以运行：

```
/ads-new-harness
```

## 命令一览

默认安装 6 个稳定命令：

| 命令 | 什么时候用 | 做什么 |
|---|---|---|
| `/ads-init` | 新项目启动，或给老项目补齐基础文档时 | 通过提问理解项目，生成 `PROJECT.md`、`ARCHITECTURE.md`、`REQUIREMENTS.md`、`ROADMAP.md`、`DONT.md`、`STATE.md` 和验证脚本 |
| `/ads-next` | 想继续推进一个任务时 | 选择或发现下一个任务，生成 PRD 和测试计划，和用户对齐后进入 TDD，实现、验证、审查、归档 |
| `/ads-record` | 已经临时开发了一些内容，但没走 `/ads-next` 时 | 根据现有对话和代码变更，补生成任务记录、需求、测试计划、变更说明和报告 |
| `/ads-report` | 想知道项目现在到哪一步时 | 只读打印项目状态、当前任务、阻塞点和最近记录 |
| `/ads-new-harness` | 想创建自己的 AI 开发 harness 时 | 先讲清楚本插件原理，再访谈定制需求，默认生成类似本项目的 npm skill package |
| `/ads-help` | 忘记命令怎么用时 | 显示中文帮助 |

## 推荐工作流

### 新项目

先运行 `/ads-init`。它会通过一轮问题帮你把项目目标、核心用户、边界、技术栈、架构和路线图写进 `docs/ads/`。

之后每次要开发，就运行 `/ads-next`。它一次只推进一个任务，避免模型同时改太多东西。

### 老项目

在已有项目中运行 `/ads-init`，让 Claude 先理解现有代码和项目结构，再补齐文档和路线图。

初始化完成后，用 `/ads-next` 逐步做功能、修 bug 或整理架构。每个任务都会留下单独的任务目录，后续可以追溯。

### 临时 vibe coding 后补记录

如果你已经和 Claude 普通对话完成了一些功能，但没有提前跑 `/ads-next`，可以运行 `/ads-record`。

它会把已经发生的工作补录成 ADS 任务记录，但不会假装这些内容是事前规划过的。缺少验证或不确定的地方会标记为 `needs-review`。

### 创建自己的 harness

如果你发现自己想不断调整 `ai-driven-skills` 的流程，说明你可能需要一个自己的 harness。运行 `/ads-new-harness` 后，它会先用白话解释本插件的核心原理：流程编排、子 agent 降低上下文开销、文档落盘、质量门禁和可调试性。然后它会追问你的定制需求，帮你设计命令、状态文件、子 agent 和停止条件。用户没有指定技术形态时，默认生成一个和本项目类似的 npm 包。

## `/ads-next` 的优势

`/ads-next` 是这个项目的核心命令。

它的重点不是“自动替你写完所有东西”，而是降低你和模型之间的需求传递成本：

1. 先读项目状态和历史文档。
2. 选择下一个任务，或在路线图为空时引导你发现新需求。
3. 追问需求细节，生成当前任务的 PRD。
4. 由测试规划 agent 先生成中文测试计划、对齐测试覆盖，并初始化报告里的测试报告模板。
5. 再进入实现、测试、验证和审查。
6. 完成后按统一模板归档任务，把重要上下文写回 `STATE.md`。

这样下次即使你打开一个全新的 Claude Code 窗口，也可以从项目文档继续，而不是重新解释整个项目。

## 生成的文档结构

运行 `/ads-init` 后，项目里会出现类似结构：

```text
your-project/
└── docs/ads/
    ├── PROJECT.md          # 项目概述、目标用户、核心价值
    ├── ARCHITECTURE.md     # 架构说明、模块划分、流程图
    ├── REQUIREMENTS.md     # 需求和验证方案
    ├── ROADMAP.md          # 后续任务路线图
    ├── DONT.md             # 项目硬约束，不应该做什么
    ├── STATE.md            # 当前状态和跨 session 上下文
    ├── verify.sh           # 项目级验证脚本
    └── tasks/
        └── 001-feature-example/
            ├── PRD.md
            ├── test-plan.md      # 中文测试计划
            ├── report.md         # Step 2 初始化模板，归档时补结果
            └── CHANGES.md
```

这些文件不是装饰文档，而是后续开发流程的输入。`/ads-next` 会读取它们，更新它们，并把每个任务的决策和结果归档进去。

## 设计原则

- **轻量**：比完整 GSD / harness 流程更轻，普通项目也能直接用。
- **协作优先**：默认需要用户参与确认需求、测试和方向，不追求盲目全自动。
- **文档落盘**：把项目理解写进仓库，而不是只留在聊天记录里。
- **一次一个任务**：降低模型误改范围，也方便审查和回滚。
- **质量前置**：先明确验收和验证，再进入实现。
- **可续写**：新窗口、新会话也能从 `docs/ads/` 继续理解项目。

## 开发中的 Skills

仓库中保留了部分开发中命令，例如 `/ads-auto`、`/ads-verify`、`/ads-sync`。

这些命令目前不作为公开稳定能力默认安装，主要用于后续扩展和本地调试：

| 命令 | 作用 | 当前状态 |
|---|---|---|
| `/ads-auto` | 连续执行 `/ads-next`，按路线图自动推进多个任务；遇到回归、`needs-review`、`HITL` 或其他停止条件时中断 | 开发中，暂不公开推荐 |
| `/ads-verify` | 按需运行当前任务和项目级验证脚本，用来检查手动修改后项目是否仍然健康 | 开发中 |
| `/ads-sync` | 当代码和 `docs/ads/` 文档不一致时，扫描差异并提出文档同步建议；应用前需要用户确认 | 开发中 |

其中 `/ads-auto` 是更接近全自动推进的模式，但目前仍在开发阶段。当前推荐用户使用 `/ads-next`，通过人机协作逐步推进项目，以获得更好的需求传递和软件质量。

如果你是本地调试或开发者，可以自行查看源码中的 dev-only skills。

## 本地开发

克隆仓库后，可以直接安装当前 checkout 的版本：

```
node bin/cli.js install -f
```

安装到当前项目：

```
node bin/cli.js install --project -f
```

查看 CLI 帮助：

```
node bin/cli.js help
```

## License

MIT
