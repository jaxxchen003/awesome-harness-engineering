# Harness 工程深度报告

中英双语案例集。English edition: [report.en.md](report.en.md). 资料截止日期：2026-10-27。星标数为当日 GitHub API 快照，不是质量排名。

## 1. 结论先行

Harness 不是模型，也不是一条更长的 prompt。它是包在模型外面的运行时：工具、权限、上下文、记忆、技能、计划、多智能体编排、校验回路、界面，以及任务中断后如何续上。

2026 年这条分界已经写进厂商自己的文档。Anthropic 在 2026-06-02 的文章 [A harness for every task](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code) 里把 Claude Code 的默认运行时称为 harness，并让模型为手头任务再写一层动态 harness。DeepSeek 把产品直接命名为 Harness，口号是 “Everything is a Plugin”。Nous Research 把 Hermes 定义成会从经验里长出技能的 agent，而不是绑在 IDE 上的 copilot。

因此，比较 Claude Code、Codex、OpenCode、DeepSeek Harness、Grok Build、Grok Bot、Hermes、Muse，比的不是“谁的模型更聪明”，而是七件事：

1. 上下文怎么装、怎么丢、怎么在压缩后还认得出原目标。
2. 权限默认是询问、自动，还是沙箱。
3. 技能是临时提示，还是可发现、可渐进加载的文件夹。
4. 子代理是一次性帮手，还是可恢复、可隔离、可对抗验证的工作流。
5. 状态放在窗口里，还是放在磁盘、云电脑、消息网关里。
6. 表面是终端、IDE、桌面、手机，还是它们共享同一套引擎。
7. 做完之后有没有一个会失败的检查，而不是“看起来做完了”。

## 2. 什么叫 harness engineering

一个可用的定义：

> Harness engineering 是设计模型外围的控制回路，使一个会犯错、会遗忘、会越权的模型，能够在真实文件、命令、账号和应用里，把任务做到可检查的完成态。

它和相邻词的差别：

| 词 | 管什么 | 不管什么 |
| --- | --- | --- |
| 模型 | 权重、推理、工具调用能力 | 你的仓库、审批、日程 |
| Prompt / context file | 这一次该遵守的规则 | 规则如何被发现、注入、版本化 |
| Agent | 一轮“观察-行动-再观察” | 多轮之后权限、记忆、协作怎么长出来 |
| Harness | 上面全部的运行时外壳 | 通常不训练基础模型 |

Anthropic 写出了默认 harness 的三个失效模式，这是目前最清晰的工程动机：

- **Agentic laziness**：复杂任务做到一部分就宣布完成。
- **Self-preferential bias**：让模型检查自己的结论时，它偏向自己的结果。
- **Goal drift**：长对话和 compaction 之后，边界条件与“不要做 X”丢失。

动态工作流的解法不是把提示写得更凶，而是把目标拆进各自干净的上下文：分类、扇出、对抗验证、生成后过滤、锦标赛、循环直到停止条件成立。子代理可以选择模型，也可以隔离在 worktree 或远端。中断后恢复会话，工作流从断点继续。

这套词汇现在可以拿去读任何一家产品。

## 3. 八层解剖

| 层 | 工程问题 | 成熟做法 |
| --- | --- | --- |
| 循环 | 谁决定下一步 | 主循环 + 可中断；headless 单次执行 |
| 上下文 | 窗口是稀缺资源 | 启动只加载技能名和描述；大输出外置 |
| 记忆 | 压缩会丢约束 | `CLAUDE.md` / `AGENTS.md` / `SOUL.md`；磁盘计划；跨会话摘要 |
| 能力 | 工具太多会淹没模型 | Skills、MCP、插件、LSP；渐进披露 |
| 权限 | 自主和事故在同一旋钮上 | 询问、计划审批、沙箱、删除必须确认 |
| 编排 | 一个窗口干不完 | 子代理、workflow、群组、会话消息 |
| 表面 | 人在哪，活在哪 | CLI、IDE、桌面、Web、手机、IM、定时 |
| 验收 | “看起来好了”不是信号 | 测试、类型检查、对抗审查、人工批准点 |

Claude Code 官方最佳实践把第一约束写得很硬：上下文窗口装得下整段对话、读过的文件和命令输出，装满之后性能下降。所以要给它一个能 pass/fail 的检查，先探索再计划再改代码，用 hooks、skills、subagents 把重复劳动移出主窗口。

## 4. 产品案例

### 4.1 Claude Code：编码 harness 的参考实现

厂商 Anthropic。表面包括终端、VS Code、JetBrains、桌面应用和浏览器；这些表面连的是同一引擎，所以仓库里的 `CLAUDE.md`、设置和 MCP 可以跨表面复用。

它已经不是“终端里的聊天”。官方概览把这些路径列成一等能力：Remote Control、把 Telegram / Discord / iMessage / webhook 推进会话的 Channels、本地任务送上云再在手机接着做、桌面或云端定时任务、GitHub Actions 与 GitLab CI、自动 PR review、Slack 路由、Chrome 调试，以及用来做自定义 agent 的 Agent SDK。

工程上最值得抄的不是界面，而是 2026 年中加上的动态工作流。主代理可以用 JavaScript 编排 `agent()`、`parallel()`、`pipeline()`。`agent()` 能指定 prompt、JSON schema、模型、隔离方式（worktree 或 remote）和子代理类型。文章给出的用法包括：不稳定测试的竞争性假设、从过去 50 次会话里挖反复出现的纠正并写成 `CLAUDE.md`、简历锦标赛、以及 Bun 从 Zig 迁到 Rust 这种百万级迁移。触发词 `ultracode` 用来强制走工作流。代价也写明了：更费 token，只适合高价值的复杂任务。

适合把它当 harness 教材的原因：失败模式、编排原语、验收习惯和多表面一致性是公开的。

### 4.2 Claude Cowork：文件与工具上的工作 harness

[Claude Cowork](https://claude.com/product/cowork) 解决的不是写代码，而是“回来时工作已经落到文件里”。官方定位是：给一个目标，它在你选定的文件夹和工具里做完，留下可审查的成品。你不需要从聊天复制到文档。

要点：

- 任务需要碰文件时，普通 Claude 对话会自动带上 Cowork 能力。
- Web、桌面、手机是同一个 Claude；合上电脑它还能继续，也可以按节奏定时跑。
- 大项目拆成并行块：一边起草，一边研究和整理。
- 插件把工具、知识和流程装成一个专家包。
- 只在你选择的文件夹和工具里行动；删除要批准。企业可以按团队收权限、限额，并用 OpenTelemetry 把活动送进 SIEM。

这是“工作 harness”：循环仍是 agent，但验收物是 deck、表格、研究备忘，权限边界是文件夹和连接器，而不是 git diff。

### 4.3 Claude Design：设计系统感知的视觉 harness

[Claude Design](https://claude.com/product/design) 把视觉生产放进同一代理循环。你描述画面，它先给一版；你在画布上改字、拖拽、滑块调间距和颜色。它可以从 GitHub、设计文件或本地代码库导入真实组件，避免输出长得像通用模板。

和代码 harness 的接口很具体：用 `/design-sync` 从 Claude Code 拉设计系统，或在 Claude Code 里直接 `/design`。导出走 PDF、PowerPoint、HTML，也能交回 Claude Code，或送到 Adobe、Canva、Gamma、Lovable、Miro、Replit、Vercel、Wix。管理员可以把团队锁在一套已批准的设计系统上。

2026-10 的产品页还写明：Design 已进入任意 Claude 对话，包括 Claude Code 和 Artifacts；独立站点 `claude.ai/design` 开放到 2026-12-14。Docs、Slides、Design 已离开 beta，覆盖包括 Free 在内的计划；Dashboards 和 Motion 仍是 beta。

设计 harness 的新约束是：组件必须来自真实系统，模型在给你看之前先对照设计系统自检。

### 4.4 Claude Duet：可离开电脑的主动代理

用户口语里的 “Claude Work” 更接近两件产品的组合：Cowork 做你交办的活，[Claude Duet](https://claude.com/product/claude-duet) 决定什么值得打扰你。

2026-10-21 的研究预览文章把 Duet 定义成主动代理：早上给简报，白天只在需要判断时发消息，人离开后继续工作，敏感动作等你点头。研究预览从美国 Pro / Max 候补名单开始，iOS、Android 和 claude.ai 都有，用量与其他 Claude 产品共用。产品页写的是 6,000 以上连接器和插件。记忆、技能和应用从第一条消息就在同一线程里。

安全设计被明确接到已有 harness 上，而不是另起炉灶：

- 可信代理框架，与 Claude Code、Claude in Chrome 同源。
- 凭证处理沿用 Claude Managed Agents：代理不直接看到密码。
- 自动模式分类器沿用 Claude Code auto mode。
- 提示注入防护来自浏览器使用研究。
- 另有一层分类器，防止消息泄露敏感信息。
- 红队、外部渗透测试，并纳入公开漏洞赏金。

Duet 是 harness 从“人在回路里盯着终端”走到“人在回路里做例外处理”。判断力成为产品功能，而不只是模型分数。

### 4.5 OpenAI Codex：本地、IDE、桌面、云，四条入口一条代理

[openai/codex](https://github.com/openai/codex) 自述是跑在你电脑上的轻量编码代理。2026-10-27 快照：131,021 star，Rust，Apache-2.0，当天仍在推送。

入口拆得很清楚：

- 终端：`codex`，安装脚本在 `https://chatgpt.com/codex/install.sh`，也有 npm 与 Homebrew cask。
- IDE：VS Code、Cursor、Windsurf。
- 桌面：`codex app`。
- 云：Codex Web，`chatgpt.com/codex`。

登录推荐走 ChatGPT Plus / Pro / Business / Edu / Enterprise；也可以用 API key。文档在 https://developers.openai.com/codex 。

Codex 对 harness 讨论的贡献是“同一代理，不同停留时间”。本地 CLI 适合你看着它改仓库；云端适合仓库不在这台机器上、或者任务要在你离开后继续。它没有把插件哲学写成 DeepSeek 那样的总纲，但开源 CLI 让权限、沙箱和工具循环可以被外部控制面（OpenHands、cc-switch、Orca）接住。

### 4.6 OpenCode：把 plan 和 build 拆成两种代理

[anomalyco/opencode](https://github.com/anomalyco/opencode) 是 MIT 协议的开源编码代理。快照 221,881 star，TypeScript。安装可以是 `curl -fsSL https://opencode.ai/v2/install | bash`、npm，或 Homebrew。桌面应用仍标为 beta。

内置代理是 harness 层的产品决策，不是模型菜单：

- `build`：默认，开发时全权限。
- `plan`：只读。默认拒绝改文件，跑 bash 前要许可。用来看陌生仓库和做计划。
- `general`：内部子代理，复杂搜索和多步任务，消息里用 `@general` 叫它。

Tab 在 build 和 plan 之间切换。文档在 https://opencode.ai/v2/docs 。

这是“权限模式人格化”。与其在同一次对话里反复提醒“先别改”，不如换一个默认拒绝写操作的代理。后来的 Muse、Claude Code plan mode、Grok 的模式命令，都是同一问题的不同答案。

### 4.7 DeepSeek Harness：一切皆插件

[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) 快照 262,467 star，MIT，TypeScript。官网 https://www.deepseek.com/en/harness/ 称之为全球公开预览、开源、用可组合插件扩展代理。桌面有 macOS Apple silicon 与 Windows x64；开发者也可以 `npx @deepseek-ai/dsh web`。

架构句子只有一句，但是总纲：一切皆插件，运行时是 [Cordis](https://github.com/cordiverse/cordis)。论文是 [A Programming Paradigm for Spatiotemporal Composability](https://arxiv.org/abs/2608.25512)。

从产品页和快速开始可以确认的能力：

- 日常文档、表格、幻灯片、代码、研究、后台脚本。
- Creator mode：在对话里写插件并安装。官方演示是一个悬浮番茄钟，路径包括读 `cordis-plugin-development` 技能、检查运行时 slot、写包、调用 `plugin_manager`、再验证。
- 官方插件方向：Agent teams、自动审批审查、定时任务、语音、终端、agent loop、subagents。部分标成实验。
- Web UI 在选定工作区之前不能发送；模型密钥放在 Settings → Models，保存后不用重启。
- 权限策略之下，需要批准的操作会询问。
- 仓库仍是 developer preview，明确警告会有破坏性变更，并要求先读 `SAFETY.md`。

和 Claude Code 相比，DeepSeek 把扩展点从“技能文件夹 + MCP”推进到“界面、工具、循环本身都是插件”。代价是预览期兼容性不稳定。它也是本报告的写作环境：当前会话跑在 DeepSeek Harness 桌面应用里，具有工作区、技能、子代理、定时和审批策略。这是使用体验，不是性能评测。

### 4.8 Grok Build：可嵌入的编码代理

[Grok Build](https://docs.x.ai/build/overview) 是 xAI 的编码代理，三种嵌入方式：交互 TUI、脚本里的 headless、以及其他应用通过 Agent Client Protocol 接入。安装是 `curl -fsSL https://x.ai/cli/install.sh | bash`，然后在项目里运行 `grok`。

Harness 特征：

- `grok -p` 与 streaming JSON，适合机器调用。
- `~/.grok/config.toml` 可挂任意兼容模型，不锁死自家接口。
- `grok inspect` 列出当前目录发现的配置、指令、技能、插件、hooks、MCP。
- 技能是带 `SKILL.md` 的文件夹，发现路径包括 `./.grok/skills/`、`~/.grok/skills/` 和插件。`user-invocable` 变成斜杠命令；`disable-model-invocation` 让它只给人用。
- 插件可以加技能、代理、hooks、MCP 和 LSP。
- Hooks 挂在工具和会话生命周期上。

文档把下一跳写成 [Grok Bot](https://docs.x.ai/grok-bot/overview)：云电脑上的 AI 队友。Build 是你坐在终端前的 harness，Bot 是你离开之后的 harness。

### 4.9 Grok Bot：有名字、有电脑、会交接的队友

Grok Bot 官方定义：可长期保留的 bot，有名字、职责和随时间累积的上下文。每个 bot 在一台持久云电脑上工作，电脑有浏览器、文件系统和终端。任务在真实工具里做完，而不是停在聊天草稿。

差异点写得很具体：

- 合上笔记本，云电脑上的回合和例行任务继续。
- 建 bot 是发消息，不是搭流程编排器。桌面和手机是同一个 bot。
- 多个 bot 并行，可以互发消息、在群聊里共享上下文、移交任务所有权。
- 你带它走一遍多步路径，它可以存成技能并按计划重跑。
- 同一账号下的 bot 共用一台云电脑的文件、浏览器会话和应用登录，所以交接不用重配。电脑属于账号，不属于某一个 bot；账号之间隔离。
- 可以在 X 上 `@bot`，由主 bot 把帖子接成任务。需要绑定 X 账号，且不是所有地区可用。

文档同时写着：个人付费 Cursor 计划、Cursor Teams、SuperGrok / Plus / Heavy，或自助 Grok Business 席位可以接入。本报告只转述该页，没有单独做账单验证。

Bot 的 harness 赌注是：状态放在云电脑和具名记忆里，人通过聊天和批准点管理一支小队。风险也在同一句话里：共享电脑意味着任何一个 bot 都能看见放上去的文件和登录态。

### 4.10 Hermes Agent：把学习回路做成产品

[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) 快照 257,343 star，MIT，Python。文档第一句不是“帮你写代码”，而是：它会从经验创建技能、在使用中改进技能、提醒自己把知识写下来，并跨会话建立对你的模型。

运行位置是 harness 选型的另一极。它可以在本机、5 美元 VPS、GPU 集群，或 Daytona / Modal / Vercel Sandbox 上。终端后端包括 local、Docker、SSH、Daytona、Modal、Vercel，另有 Singularity 一类插件后端。空闲时无服务器环境休眠。

能力清单里和 harness 直接相关的有：

- 记忆：策展式记忆、周期性 nudge、FTS5 跨会话检索再加摘要。可用 Honcho 做用户建模。
- 技能：自主创建，兼容 [Agent Skills](https://agentskills.io/)。
- 人格：全局 `SOUL.md`。
- 项目上下文文件。
- 消息网关：Telegram、Discord、Slack、WhatsApp、Signal、Matrix、邮件、短信、钉钉、飞书、企业微信、QQ 等，文档称 20 个以上平台。
- Bot mode：具名 bot，各自有模型、记忆、技能、例行任务和聊天；可以在群里用 @ 协作。
- 定时任务，结果送回任意平台。
- 子代理；`execute_code` 把多步工具调用收成一次推理。
- MCP、语音、命令批准、容器隔离。
- 研究向：轨迹导出，服务后续工具调用模型训练。

Hermes 把“会话结束就失忆”当成要修的缺陷，而不是用户该自己复制的笔记。

### 4.11 Muse Code：沙箱默认开启的终端与 CI 代理

[Muse Code](https://dev.meta.ai/docs/muse-code) 是 Meta 给 Muse Spark 做的编码代理。`curl -fsSL https://dev.meta.ai/install.sh | sh` 得到原生 `muse`。首次运行就要批准，并且操作系统沙箱是开的。默认模型文档写的是 `muse-spark-1.2`。

两种驱动方式：交互 TUI，或 `muse exec "<prompt>"` 给脚本和 CI。`META_API_KEY` 用于无浏览器环境。还有版本化会话协议：`muse serve` / `muse schema`，TypeScript SDK 是 `@muse-code/sdk`。

工作流文档把子代理和工作流分开：

- 子代理：主代理还能直接管理的有界帮手。
- 工作流：需要可重复编排、并行组、依赖阶段、进度和最终综合时再用。
- 设置成 `auto` 时，任务足够独立，Muse 可以主动提议工作流。
- 小改动、短解释、强耦合修改留在主代理。文档写工作流生命周期内最多 1,000 个子任务。

另外有会话消息：活着的本地会话之间交接、请求审查、同步状态。Windows 上命令走 PowerShell，沙箱首次可能弹出 UAC，语音和会话消息不可用。

Muse 的贡献是把“默认沙箱 + 工作流预算 + 会话间消息”写进同一份产品说明，而不是社区插件。

## 5. 开源控制面与社区实践

模型厂商的 harness 下面，已经长出一层“管 harness 的 harness”。

| 项目 | 快照 star | 它实际在优化的层 |
| --- | ---: | --- |
| [ECC](https://github.com/affaan-m/ECC) | 289,613 | 技能、本能、记忆、安全，横跨 Claude Code、Codex、OpenCode、Cursor |
| [planning-with-files](https://github.com/OthmanAdi/planning-with-files) | 未记入星标 | 计划在磁盘上，hook 每轮注入，扛过 `/clear` 和 compaction |
| [claude-mem](https://github.com/thedotmack/claude-mem) | 103,359 | 把会话压缩后注回未来会话 |
| [cc-switch](https://github.com/farion1231/cc-switch) | 148,947 | 一个桌面壳切换多种编码代理 |
| [Orca](https://github.com/stablyai/orca) | 106,846 | 代理舰队：桌面、手机、远端运行时 |
| [OpenHands Agent Canvas](https://github.com/OpenHands/OpenHands) | 91,912 | 自托管控制面，用 ACP 接 Claude Code、Codex、OpenCode 等 |
| [Cline](https://github.com/cline/cline) | 70,890 | IDE 扩展、CLI、SDK 三种形态的自主编码 |
| [Aider](https://github.com/Aider-AI/aider) | 49,767 | 更早的终端结对编程，仓库地图而不是大而全平台 |
| [Goose](https://github.com/aaif-goose/goose) | 55,782 | 本地可扩展代理：安装、执行、编辑、测试 |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | 108,553 | 厂商终端代理的另一条开源实现 |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | 28,738 | 同上，Qwen 系 |
| [Roo Code](https://github.com/RooCodeInc/Roo-Code) | 24,260 | 编辑器里的多代理“开发团队” |
| [SWE-agent](https://github.com/SWE-agent/SWE-agent) | 20,645 | 学术起点：issue 进，补丁出 |
| [AutoGen](https://github.com/microsoft/autogen) | 61,576 | 多代理编程框架，不是开箱编码产品 |
| [LangGraph](https://github.com/langchain-ai/langgraph) | 43,941 | 用图把代理循环做成可恢复运行时 |
| [Continue](https://github.com/continuedev/continue) | 36,337 | 开源编码代理 / IDE 路线 |
| [Plandex](https://github.com/plandex-ai/plandex) | 15,798 | 面向大项目的长程代理；2025-10 后少见推送 |

### 反复出现的实践

这些做法来自官方最佳实践、Agent Skills 标准和上述仓库的自述，不是未验证的推文转述。本环境里的 X 搜索没有登录态，所以不编造博主原话。Anthropic 自己点名的公开实践是 Jarred Sumner 用工作流把 Bun 从 Zig 迁到 Rust 的帖子。

1. **上下文文件版本化。** `CLAUDE.md`、`AGENTS.md`、Hermes 的 `SOUL.md` 和项目上下文，把“每次都要提醒”的规则变成仓库的一部分。
2. **技能渐进披露。** [agentskills.io](https://agentskills.io/) 规定启动时只看 name 和 description，命中后再读全文和附件。Grok、Hermes、Claude Code、DeepSeek 都沿着这个形状做。
3. **计划落盘。** planning-with-files 的论点是：窗口里的待办会随压缩死掉；磁盘上的 `task_plan.md`、`findings.md`、`progress.md` 不会。Hook 负责把计划重新注入注意力。
4. **先读后写，先验证后收工。** Claude Code 文档要求探索、计划、实现分开，并给一个会失败的检查。OpenCode 用 plan/build 两个代理把这件事做成界面。
5. **对抗审查。** 动态工作流和 Claude Code 最佳实践都要求另一路代理按量规挑错，抵消自偏好。
6. **隔离。** worktree、远端会话、Docker、云电脑、按会话的沙箱，避免并行代理互相踩文件。
7. **权限分层。** 删除、对外发送、凭证、越出选定文件夹，保留人工点。Duet、Cowork、Muse、DeepSeek、Hermes 都把这一点写成产品，而不是 README 里的一句劝告。
8. **控制面与代理分离。** OpenHands、Orca、cc-switch 承认没有一家终端会赢下所有任务，于是去做舰队调度。

### 两条路线之争

- **折叠派。** 2026-10-22 Anthropic 的文章讨论何时把子代理折回主循环：更少的子代理、把上下文和工具渐进披露，往往更便宜也更好。小改动不该上工作流。
- **展开派。** 动态工作流、Muse workflows、DeepSeek agent teams、Grok Bot 群聊认为：长任务、并行任务、对抗任务必须换干净上下文。

两条都对，分界是任务是否会在一个窗口里互相污染。迁移、审计、不稳定复现、多视角评审走展开；一个函数、一个紧密耦合的补丁走折叠。

## 6. 怎么选

| 你要的结果 | 优先看 |
| --- | --- |
| 仓库里的功能、测试、PR | Claude Code、Codex、OpenCode、Grok Build、Muse Code |
| 文档、表格、研究备忘，且人要离开座位 | Claude Cowork、DeepSeek Harness |
| 品牌内的视觉稿和设计系统 | Claude Design，并接回 Claude Code |
| 早上简报、跨应用盯异常、人只做例外 | Claude Duet、Grok Bot、Hermes gateway |
| 可自托管、可换模型、可审计 | OpenCode、DeepSeek Harness、Hermes、OpenHands、Goose |
| 一支有名字的 bot 互相交接 | Grok Bot、Hermes Bot mode、DeepSeek agent teams |
| 在多家终端之上做记忆和规范 | ECC、planning-with-files、claude-mem、Agent Skills |

没有总分。星标最高的 ECC 是性能层，不是产品；星标很高的 DeepSeek 与 Hermes 仍在快速改接口。Plandex 提醒我们：上一轮“大项目代理”可以在一年里安静下来。

## 7. 一份最小 harness 清单

如果要自己做，而不是再包一层产品，顺序应当是：

1. 工作区边界和沙箱，先于任何自动批准。
2. 一个会失败的验收命令。
3. 短的项目规则文件，只写模型反复违反的事。
4. 技能用文件夹，启动不加载正文。
5. 长任务的计划写在仓库里，每轮读回来。
6. 调查类子任务换上下文；实现类任务不要无故拆碎。
7. 对外动作、删除、凭证单独批准。
8. 会话摘要回写，但关键约束以文件为准，不以摘要为准。

## 8. 本次研究的边界

- 看过的是官方文档、仓库 README 和 GitHub API，不是闭门评测，也没有统一跑同一套任务。
- X 实时搜索因本地 `xreach` 未登录而不可用。社区实践用公开仓库和厂商文章支撑。
- “Claude Work” 不是官方产品名。对应表面是 Cowork，主动代理是 Duet。
- Grok Bot 与 Cursor 计划的绑定，只反映抓取到的文档页。
- 星标会过时。引用前请重新请求 API。

## 参考

完整链接和星标表见 [sources.md](sources.md)。网页版见仓库根目录 `index.html`。
