# ai-driven-skills

面向 [Claude Code](https://claude.com/claude-code) 的 AI 驱动增量开发工具。默认安装 5 个稳定命令；另有 3 个命令保留在源码中、标记为开发中，不会默认导入 Claude skills。

```
/ads-init  →  /ads-next  →  /ads-record
                  ↓
            /ads-report  /ads-help
```

临时对话开发后忘了走 `/ads-next` 时，用 `/ads-record` 把变更记录进 ADS 任务文档。

## 安装

### 全局安装（推荐）

```bash
npx ai-driven-skills@latest install
```

安装到 `~/.claude/skills/`，所有项目通用。

**默认只安装 5 个稳定 skill**：`ads-init`、`ads-next`、`ads-record`、`ads-report`、`ads-help`。

`ads-auto`、`ads-verify`、`ads-sync` 仍保留在 npm 包源码中，但**不会**默认导入 Claude。本地调试时可加 `--include-dev`。

### 安装本地仓库版本

如果你在本地修改了这个仓库里的 skill，想直接安装当前 checkout 的版本，不需要先发布到 npm。进入本仓库根目录后运行：

```bash
cd /Users/jieqin.liu/Desktop/code/AI/ai-driven-skills
node bin/cli.js install -f
```

这会把本地 `skills/` 复制到全局 `~/.claude/skills/`，`-f` 表示覆盖已安装版本。修改 skill 后再次运行同一条命令即可同步到全局。

如果只想安装到某个项目的 `./.claude/skills/`，需要在目标项目目录运行本仓库的 CLI：

```bash
cd /path/to/your-project
node /Users/jieqin.liu/Desktop/code/AI/ai-driven-skills/bin/cli.js install --project -f
```

安装后重启 Claude Code（或运行 `/help`）以加载本地版本。

### 项目级安装

```bash
npx ai-driven-skills@latest install --project
```

安装到当前目录 `./.claude/skills/`。

### 其他命令

```bash
npx ai-driven-skills@latest list           # 查看已安装的 skill
npx ai-driven-skills@latest install -f     # 强制覆盖已有文件
npx ai-driven-skills@latest install --include-dev  # 同时安装开发中 skill（本地调试用）
npx ai-driven-skills@latest uninstall      # 卸载所有 skill
npx ai-driven-skills@latest help
```

安装后重启 Claude Code（或运行 `/help`）以加载新命令。

## 命令一览

### 稳定可用（默认安装）

| 命令 | 用途 |
|---|---|
| `/ads-init`    | 初始化项目骨架。深度提问 → 结构化需求（含验证方案）→ 架构设计（mermaid 图）→ Roadmap 生成。产出：`docs/ads/PROJECT.md`、`docs/ads/ARCHITECTURE.md`、`docs/ads/REQUIREMENTS.md`、`docs/ads/ROADMAP.md`、`docs/ads/DONT.md`、`docs/ads/STATE.md`、`docs/ads/verify.sh`、`docs/ads/tasks/`。 |
| `/ads-next`    | 驱动**一个**任务完整生命周期：选任务（或发现新需求）→ Grill 需求 → 生成 PRD → 功能测试 → 用户对齐 → TDD 开发 → 验证 → Review Agent 审核 → 归档 → 提取可复用 skill → 规划下一步。ROADMAP 为空时自动进入需求发现模式。 |
| `/ads-record`  | 把当前对话和已完成/半完成代码变更**记录进 ADS 任务历史**：生成 PRD、test-plan、CHANGES、report，更新 ROADMAP/STATE，并可选提交。适合临时开发后发现没走 `/ads-next` 的断档场景。 |
| `/ads-report`  | 打印项目状态快照。只读。 |
| `/ads-help`    | 显示中文帮助页。 |

### 开发中（源码保留，默认不安装）

| 命令 | 用途 |
|---|---|
| `/ads-auto` 🚧 | 循环执行 `/ads-next` 直到 ROADMAP 耗尽或触发停止条件。 |
| `/ads-verify` 🚧 | 按需运行任务验证 + 项目验证。只读，不修改。 |
| `/ads-sync` 🚧 | 文档 ↔ 代码漂移时对账。生成差异提案，用户确认后再应用。 |

## 工作原理

所有命令共享项目 `docs/ads/` 目录下的状态文件：

- **`docs/ads/PROJECT.md`** — 项目概述、核心价值、技术栈、使用说明
- **`docs/ads/ARCHITECTURE.md`** — 模块划分、mermaid 架构图、流程图、数据流
- **`docs/ads/REQUIREMENTS.md`** — 分类需求，每条附验证方案（预期输入/输出/边界情况）
- **`docs/ads/ROADMAP.md`** — 任务列表及状态（`[ ]` 未开始、`🔨` 进行中、`✅` 已完成、`🛑 needs-review`、`HITL`）
- **`docs/ads/DONT.md`** — 硬约束，绝对不做的事
- **`docs/ads/STATE.md`** — 项目状态、当前任务、Context Notes（跨 session 自动积累的关键信息）

加上：

- **`docs/ads/verify.sh`** — 项目级回归检查（测试、lint、构建）
- **`docs/ads/tasks/NNN-<type>-<slug>/`** — 每个任务的 PRD、tests/、test-plan.md、report.md、CHANGES.md

## 生成的项目目录结构

运行 `/ads-init` 后生成以下结构。随着 `/ads-next` 执行任务，`docs/ads/tasks/` 目录会不断增长：

```
your-project/
├── docs/ads/                   # ai-driven-skills 生成的所有文档
│   ├── PROJECT.md               # 项目概述、核心价值、技术栈、使用说明
│   ├── ARCHITECTURE.md          # 架构文档：模块、mermaid 图、数据流
│   ├── REQUIREMENTS.md          # 需求文档：分类需求 + 验证方案
│   ├── ROADMAP.md               # 路线图：任务列表及状态
│   ├── DONT.md                  # 硬约束：绝对不做的事
│   ├── STATE.md                 # 运行状态 + Context Notes（跨 session 记忆）
│   ├── verify.sh                # 项目级验证脚本（测试/lint/构建）
│   └── tasks/                   # 任务归档目录
│       ├── 001-feature-login/
│       │   ├── PRD.md           # 任务需求文档
│       │   ├── tests/           # 功能测试（TDD 驱动）
│       │   ├── test-plan.md     # 测试覆盖映射
│       │   ├── report.md        # 执行报告（含测试详情：输入/输出/风险）
│       │   └── CHANGES.md       # 变更文件清单
│       └── ...
└── .claude/skills/project/  # 项目级可复用 skill（自动沉淀）
```

### 各文件说明

| 文件 | 创建时机 | 更新方式 |
|---|---|---|
| `docs/ads/PROJECT.md` | `/ads-init` 创建 | 手动编辑 |
| `docs/ads/ARCHITECTURE.md` | `/ads-init` 创建 | `/ads-sync` 对账更新，或手动编辑 |
| `docs/ads/REQUIREMENTS.md` | `/ads-init` 创建 | 手动添加新需求 |
| `docs/ads/DONT.md` | `/ads-init` 创建 | 手动添加约束条目 |
| `docs/ads/ROADMAP.md` | `/ads-init` 创建 | `/ads-next` 自动更新状态，手动添加新任务 |
| `docs/ads/STATE.md` | `/ads-init` 创建 | `/ads-next` 自动维护（含 Context Notes），勿手动改 |
| `docs/ads/verify.sh` | `/ads-init` 创建 | 手动维护（添加测试/lint/构建命令） |
| `docs/ads/tasks/NNN-*/PRD.md` | `/ads-next` Step 1 生成 | 任务完成后状态改为 `done` |
| `docs/ads/tasks/NNN-*/tests/` | `/ads-next` Step 2 生成 | TDD 过程中可能微调 |
| `docs/ads/tasks/NNN-*/report.md` | `/ads-next` Step 7 生成 | 归档后不再修改 |

## /ads-init 流程

```
环境检测 → 深度提问 → 需求定义 → 架构设计 → Roadmap → 生成文件 → 审阅提交
```

1. **环境检测** — brownfield/greenfield 判断，推断已有技术栈
2. **深度提问** — 每次一个问题跟线索，覆盖核心价值、用户、流程、约束、边界
3. **需求定义** — 按功能类别选择 v1/排除，每条需求附验证方案（输入/输出/边界情况）
4. **架构设计** — 子 agent 生成模块表 + mermaid 架构图 + 流程图
5. **Roadmap** — 子 agent 生成任务列表，需求 → task 映射
6. **生成文件** — DONT.md、STATE.md、verify.sh 等（均位于 docs/ads/ 下）、使用说明
7. **审阅提交** — 用户选择查看任意文件 → git commit

## /ads-next 流程

```
读状态 → 选任务（或发现需求）→ Grill → PRD → 测试 → 对齐 → TDD → 验证 → Review → 归档 → 沉淀 → 规划
```

1. **Context load** — 读取 STATE/ROADMAP/DONT + Context Notes
2. **PRD** — 选任务（ROADMAP 为空时自动发现需求）、Grill 用户需求、生成 PRD.md
3. **Tests** — 查已有项目 skill → 生成功能测试（RED 状态）
4. **Align** — 展示测试覆盖，用户确认
5. **TDD** — 实现直到全部测试 GREEN
6. **Verify** — 查已有项目 skill → 全量测试 + 项目 docs/ads/verify.sh
7. **Review** — Review Agent 严格审核，不通过回到 Step 5 TDD 循环
8. **Archive** — 生成详细报告（每个测试的输入/输出/风险）+ 提取 Context Notes
9. **Distill** — 子 agent 提取可复用能力为项目级 skill
10. **Plan next** — 子 agent 展示后续计划，用户选择优先级和补充

## /ads-record 流程

当你已经和模型通过普通对话开发了一批功能、但没有走 `/ads-next`，用 `/ads-record` 把断掉的 ADS 记录补回来：

```
读对话/代码变更 → 反推需求与验收标准 → 生成 PRD → 生成 test-plan
→ 运行/记录验证 → 生成 CHANGES 与 report → 更新 ROADMAP/STATE → 可选提交
```

它不会假装这些工作是事前规划过的，而是明确标记为 backfill：

- 已验证完成的验收标准标为完成
- 缺测试或验证失败的部分标为 `🛑 needs-review`
- 不足以判断的地方只问必要问题
- 生成的目录结构与 `/ads-next` 一致：`docs/ads/tasks/NNN-<type>-<slug>/`

## 停止条件（安全网）

`/ads-next` 和 `/ads-auto` 在以下情况立即停止：

- 必需的状态文件缺失
- 任务会触碰 `docs/ads/DONT.md` 保护的代码
- 任务标记为 `🛑 needs-review` 或 `HITL`
- 修改文件数超过 10 个
- 项目 `docs/ads/verify.sh` 在任务验证通过后失败（检测到回归）
- Review Agent 同一问题连续 3 次打回

## Review Agent（多 Agent 审查）

内置独立的 Review Agent，在任务完成后自动审核：

- 每条验收标准是否有测试覆盖
- 测试是否有意义（会真正失败）
- 测试是否被削弱或跳过
- 实现是否满足 PRD
- 是否有 TODO、placeholder、空 catch
- 是否符合 docs/ads/ARCHITECTURE.md 约定和 docs/ads/DONT.md 约束

Review Agent 返回 `✅ APPROVED` 才能继续归档。`🔴 REJECTED` 时回到 TDD 修复再重新提交。

## 跨 Session 记忆

每次 `/ads-next` 完成后，子 agent 自动从对话中提取关键信息（环境配置、key 位置、踩坑记录等）追加到 `docs/ads/STATE.md` 的 `Context Notes`。下次新 session 时 Step 0 自动读取，避免重复发现。

## 项目级 Skill 沉淀

每次任务完成后，子 agent 分析产物，把可复用的能力（编译脚本、验证 helper 等）沉淀到 `.claude/skills/project/`。后续任务生成测试或验证脚本前会先查已有 skill，避免重复构建。

## Git 集成

如果项目是 git 仓库，`/ads-next` 在任务完成后自动 commit（不 push），commit message 包含：

- 任务 ID 和一行摘要
- Why（动机）
- Changes（变更清单）
- Tests（测试结果）
- Review（审核结论）

## 典型工作流

```bash
# 1. 初始化（每个项目一次）
/ads-init

# 2. 正常按任务开发
/ads-next

# 3. 临时对话开发后补录文档
/ads-record

# 4. 查看状态
/ads-report
/ads-help
```

## 跨平台

- 纯 Node.js CLI（无原生依赖）
- 无符号链接（Windows 友好）
- 文件复制安装/卸载——无 shell 技巧
- 支持 Node 14+

## 维护者指南 — 发布到 npm

```bash
npm login
npm pack --dry-run
npm version patch   # 或 minor / major
npm publish --access public
npx ai-driven-skills@latest help
```

发布前检查清单：

- [ ] `skills/` 下有全部 8 个 skill（含 steps/ 子目录）；其中 5 个默认安装，3 个 dev-only
- [ ] `bin/cli.js` 可正常运行：`node bin/cli.js help`
- [ ] `npm pack --dry-run` 仅包含 `bin/`、`skills/`、`README.md`、`package.json`
- [ ] 已升版本号

## License

MIT
