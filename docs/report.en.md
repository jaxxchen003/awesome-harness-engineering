# Harness Engineering: A Field Report

Bilingual case catalog. 中文版：[report.zh.md](report.zh.md). Cutoff: 2026-10-27. Star counts are a GitHub API snapshot from that day, not a quality ranking.

## 1. The claim

A harness is not a model and not a longer prompt. It is the runtime around the model: tools, permissions, context, memory, skills, plans, multi-agent orchestration, verification, surfaces, and a way to resume after interruption.

By 2026 that distinction is in vendor language. Anthropic’s 2026-06-02 essay [A harness for every task](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code) calls the Claude Code runtime a harness and lets the model write a task-specific harness on top. DeepSeek named the product Harness and organized it as “everything is a plugin.” Nous Research describes Hermes as an agent that grows skills from experience, not a copilot tied to an IDE.

Comparing Claude Code, Codex, OpenCode, DeepSeek Harness, Grok Build, Grok Bot, Hermes, and Muse is therefore a comparison of seven mechanisms:

1. How context is loaded, discarded, and kept faithful after compaction.
2. Whether permission defaults to ask, auto, or sandbox.
3. Whether a skill is a pasted prompt or a discoverable folder loaded in stages.
4. Whether subagents are one-off helpers or isolated, resumable, adversarially checked workers.
5. Whether state lives in the window, on disk, on a cloud computer, or in a messaging gateway.
6. Whether terminal, IDE, desktop, and phone share one engine.
7. Whether “done” is a failing check or a vibe.

## 2. Definition

> Harness engineering designs the control loop around a model so that a system which forgets, confabulates, and overreaches can still finish real work in files, shells, accounts, and applications, in a state a human can check.

| Term | Owns | Does not own |
| --- | --- | --- |
| Model | Weights, inference, tool-call competence | Your repo, approvals, calendar |
| Prompt / context file | Rules for this run | Discovery, injection, versioning |
| Agent | One observe-act loop | How permission, memory, and collaboration accumulate |
| Harness | The runtime shell around all of the above | Usually, training the base model |

Anthropic names three failure modes of a single default harness. They are the clearest engineering motive in the public record:

- **Agentic laziness.** The model stops early and declares a partial job done.
- **Self-preferential bias.** Asked to grade its own work, it prefers its own findings.
- **Goal drift.** Compaction drops edge cases and “do not do X” constraints.

Dynamic workflows answer this by giving subagents clean contexts: classify-and-act, fan-out-and-synthesize, adversarial verification, generate-and-filter, tournament, and loop-until-done. A subagent can pick a model and run in a worktree or remotely. If the session is interrupted, the workflow resumes.

## 3. Eight layers

| Layer | Question | Mature move |
| --- | --- | --- |
| Loop | Who chooses the next action | Interruptible main loop; headless one-shot |
| Context | The window is scarce | Load skill names first; park bulky output |
| Memory | Summaries drop constraints | `CLAUDE.md`, `AGENTS.md`, `SOUL.md`; plans on disk |
| Capability | Too many tools drown the model | Skills, MCP, plugins, LSP; progressive disclosure |
| Permission | Autonomy and incidents share a dial | Ask, plan approval, sandbox, confirm deletes |
| Orchestration | One window is not enough | Subagents, workflows, groups, session messages |
| Surface | Work should live where the human is | CLI, IDE, desktop, web, phone, chat, cron |
| Acceptance | “Looks done” is not a signal | Tests, adversarial review, explicit approval points |

Claude Code’s best-practices guide states the constraint directly: the window holds the conversation, every file read, and every command output, and quality falls as it fills. Give the agent a pass/fail check. Explore, then plan, then code. Move repeated procedure into hooks, skills, and subagents.

## 4. Cases

### 4.1 Claude Code

Anthropic’s coding harness. Terminal, VS Code, JetBrains, desktop, and web share one engine, so `CLAUDE.md`, settings, and MCP servers carry across surfaces.

The overview already lists Remote Control, Channels from Telegram, Discord, iMessage, and webhooks, cloud handoff, scheduled tasks, GitHub Actions and GitLab CI, code review, Slack routing, Chrome debugging, and an Agent SDK.

The piece to study is dynamic workflows (June 2026). The model orchestrates `agent()`, `parallel()`, and `pipeline()` in JavaScript. `agent()` takes a prompt, an optional JSON schema, a model, isolation (`worktree` or `remote`), and an agent type. The essay’s examples include flaky tests, mining the last 50 sessions for corrections that should become `CLAUDE.md` rules, resume tournaments, and the Bun rewrite from Zig to Rust. The word `ultracode` forces a workflow. The stated cost is tokens. Use it on complex, high-value work.

### 4.2 Claude Cowork

[Claude Cowork](https://claude.com/product/cowork) is the work harness. You give a goal; it acts in folders and tools you choose and leaves a reviewable artifact. You do not copy from chat into a document.

When a normal Claude conversation needs files, Cowork’s capabilities come along. Web, desktop, and mobile are the same Claude. It keeps going with the laptop closed and can run on a schedule. Large jobs are chunked and run together. Plugins package tools, knowledge, and workflows. Deletes require approval. Enterprise admins scope access, spend, and tool policy, and can stream activity to a SIEM through OpenTelemetry.

The loop is still an agent. The deliverable is a deck, a spreadsheet, or a memo. The permission boundary is a folder and a connector, not a diff.

### 4.3 Claude Design

[Claude Design](https://claude.com/product/design) puts visual production in the same loop. You describe a visual, react to a first version, then edit text, drag layout, and tune spacing and color. It can import real components from GitHub, design files, or a local codebase so the result is not a generic template.

The seam to the coding harness is explicit: `/design-sync` pulls a design system from Claude Code, and `/design` works inside Claude Code. Export goes to PDF, PowerPoint, and HTML, back to Claude Code, or out to Adobe, Canva, Gamma, Lovable, Miro, Replit, Vercel, and Wix. Admins can lock a team to one approved system.

As of the October 2026 product page, Design is available inside any Claude conversation, including Claude Code and Artifacts. The standalone site `claude.ai/design` stays up until 2026-12-14. Docs, Slides, and Design have left beta on every Claude plan, including Free. Dashboards and Motion remain in beta.

The new constraint is source fidelity: build from the real system, and check against it before the human sees the frame.

### 4.4 Claude Duet

“Claude Work” in casual use is two surfaces. Cowork does the task you hand it. [Claude Duet](https://claude.com/product/claude-duet) decides what is worth interrupting you for.

The 2026-10-21 research preview is a proactive agent: a morning brief, messages only when judgment is required, work that continues while you are away, and a wait for approval when the action matters. The preview starts with a US Pro and Max waitlist, on iOS, Android, and claude.ai, drawing on the same usage limits as the rest of Claude. The product page cites more than 6,000 connectors and plugins. Memory, skills, and connected apps are present from the first message.

Safety is bolted to existing harnesses:

- The trustworthy-agents framework shared with Claude Code and Claude in Chrome.
- Credential handling from Claude Managed Agents, so the agent does not see passwords.
- The Claude Code auto-mode classifier.
- Prompt-injection defenses from browser-use research.
- An additional classifier against leaking sensitive information.
- Red teaming, an external penetration test, and the public bug bounty.

Duet moves the human from watching a terminal to handling exceptions. Judgment is a product feature.

### 4.5 OpenAI Codex

[openai/codex](https://github.com/openai/codex) describes itself as a lightweight coding agent that runs on your computer. Snapshot: 131,021 stars, Rust, Apache-2.0, still pushing on the snapshot day.

Four doors, one agent:

- Terminal, via `https://chatgpt.com/codex/install.sh`, npm, or Homebrew.
- IDE: VS Code, Cursor, Windsurf.
- Desktop: `codex app`.
- Cloud: Codex Web at `chatgpt.com/codex`.

Sign-in is ChatGPT Plus, Pro, Business, Edu, or Enterprise, or an API key. Docs: https://developers.openai.com/codex .

Codex’s contribution is dwell time. The CLI is for watching a repo change. The cloud agent is for repos you do not have locally, or for work that outlives the laptop session. The open CLI is also why control planes such as OpenHands, cc-switch, and Orca can host it.

### 4.6 OpenCode

[anomalyco/opencode](https://github.com/anomalyco/opencode) is the MIT-licensed open coding agent. Snapshot: 221,881 stars, TypeScript. Install with the v2 script, npm, or Homebrew. The desktop app is still marked beta.

Built-in agents are a harness decision:

- `build` is the default full-access developer.
- `plan` is read-only, denies file edits, and asks before bash. It is for unfamiliar code and for planning.
- `general` is an internal subagent for hard search and multistep work, invoked as `@general`.

Tab switches build and plan. Docs: https://opencode.ai/v2/docs .

This is permission-as-persona. Instead of repeating “do not edit yet,” you switch to an agent whose default is to refuse writes. Muse, Claude Code’s plan mode, and Grok’s mode commands are other answers to that question.

### 4.7 DeepSeek Harness

[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) snapshot: 262,467 stars, MIT, TypeScript. The product site calls it a worldwide public preview, open source, extended by composable plugins. Desktop builds exist for macOS Apple silicon and Windows x64. Developers can also run `npx @deepseek-ai/dsh web`.

The architecture line is the thesis: everything is a plugin, on [Cordis](https://github.com/cordiverse/cordis). The paper is [A Programming Paradigm for Spatiotemporal Composability](https://arxiv.org/abs/2608.25512).

Documented capabilities:

- Documents, spreadsheets, slides, code, research, and background scripts.
- Creator mode writes and installs a plugin from chat. The public demo is a floating Pomodoro: read the plugin-development skill, inspect runtime slots, write the package, call `plugin_manager`, verify.
- Official plugin directions include agent teams, auto-approval review, scheduled tasks, voice, terminal, the agent loop, and subagents. Several are marked experimental.
- The Web UI cannot send until a workspace is selected. An API key saved under Settings → Models is live without restart.
- The active permission policy asks before operations that require approval.
- The repository is a developer preview and warns of breaking changes. Read `SAFETY.md` first.

Against Claude Code, DeepSeek pushes the extension point from “skills plus MCP” to “the UI, the tools, and the loop are plugins.” The cost is preview instability. This report was written inside the DeepSeek Harness desktop app. That is a usage note, not a benchmark.

### 4.8 Grok Build

[Grok Build](https://docs.x.ai/build/overview) is xAI’s coding agent: an interactive TUI, a headless mode for scripts, and ACP for other apps. Install with `curl -fsSL https://x.ai/cli/install.sh | bash`, then run `grok` in a project.

Harness traits:

- `grok -p` and streaming JSON for machines.
- Any compatible model in `~/.grok/config.toml`.
- `grok inspect` lists discovered config, instructions, skills, plugins, hooks, and MCP servers.
- Skills are `SKILL.md` folders from `./.grok/skills/`, `~/.grok/skills/`, and plugins. `user-invocable` exposes a slash command. `disable-model-invocation` hides it from the model.
- Plugins add skills, agents, hooks, MCP, and LSP.
- Hooks sit on tool and session lifecycle events.

The docs send the next reader to [Grok Bot](https://docs.x.ai/grok-bot/overview). Build is the harness while you sit at the terminal. Bot is the harness after you leave.

### 4.9 Grok Bot

Grok Bot is a set of named teammates with jobs and compounding context. Each bot works on a persistent cloud computer with a browser, filesystem, and terminal. Finished work lands in real tools, not as a chat draft.

Documented differences:

- Closing the laptop does not stop a background turn or a routine.
- Setup is a message, not a workflow builder. Desktop and mobile reach the same bot.
- Bots run in parallel, message each other, share group-chat context, and pass task ownership.
- Walking a bot through a path once can save it as a skill and rerun it on a schedule.
- Bots on one account share one computer’s files, browser sessions, and app logins. The computer belongs to the account, not to a single bot. Isolation between users is strict.
- Tagging `@bot` on X turns a post into a task for the main bot, where available, after the X account is linked.

The same overview says access is included with paid individual Cursor plans and Cursor Teams, or by linking SuperGrok, Plus, or Heavy, or a self-serve Grok Business seat. This report repeats that page. It does not independently test billing.

The bet is that state lives on the cloud computer and in named memory, and the human manages a team through chat and approval points. The risk is the same sentence: anything placed on the shared computer is visible to every bot on the account.

### 4.10 Hermes Agent

[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) snapshot: 257,343 stars, MIT, Python. The docs do not lead with coding. They lead with a learning loop: create skills from experience, improve them in use, nudge itself to persist knowledge, and build a model of the user across sessions.

Placement is the other axis. Hermes runs on a laptop, a cheap VPS, a GPU cluster, or Daytona, Modal, and Vercel Sandbox. Terminal backends include local, Docker, SSH, Daytona, Modal, and Vercel, plus plugin backends such as Singularity. Serverless environments hibernate when idle.

Harness-relevant pieces:

- Curated memory, periodic nudges, FTS5 recall plus summarization, optional Honcho user modeling.
- Skills created by the agent, compatible with [Agent Skills](https://agentskills.io/).
- A global `SOUL.md` and project context files.
- A gateway to Telegram, Discord, Slack, WhatsApp, Signal, Matrix, email, SMS, DingTalk, Feishu, WeCom, QQ, and others. The docs say 20+ platforms.
- Bot mode: named bots with their own model, memory, skills, routines, and chats, collaborating in groups through mentions.
- Cron with delivery back to any platform.
- Subagents, plus `execute_code` to collapse multi-step tool calls into one inference.
- MCP, voice, command approval, container isolation.
- Trajectory export for later tool-calling research.

Hermes treats “the session ended, so the agent forgot” as a defect to fix.

### 4.11 Muse Code

[Muse Code](https://dev.meta.ai/docs/muse-code) is Meta’s coding agent for Muse Spark. The installer yields a native `muse` binary. Approvals and an OS sandbox are on from the first run. The documented default model is `muse-spark-1.2`.

Interactive TUI, or `muse exec` for scripts and CI. `META_API_KEY` covers headless auth. A versioned session protocol is exposed through `muse serve` and `muse schema`, with `@muse-code/sdk`.

The workflow guide separates two kinds of delegation:

- Subagents are bounded helpers the lead can manage directly.
- Workflows are for repeatable orchestration, parallel groups, dependent stages, progress, and a final synthesis.
- With Workflows set to `auto`, Muse may propose one when the task is independent enough.
- Small edits, short explanations, and tightly coupled changes stay on the main agent. A workflow may create up to 1,000 child tasks over its life.

Session messaging lets live local sessions hand off work, request review, and post status. On Windows, commands use PowerShell, the sandbox may raise UAC once, and voice plus session messaging are unavailable.

Muse’s contribution is shipping default sandboxing, a workflow budget, and inter-session messages as product surface rather than as community plugins.

## 5. Control planes and community practice

Under the vendor harnesses sits a layer that manages harnesses.

| Project | Snapshot stars | Layer it actually optimizes |
| --- | ---: | --- |
| [ECC](https://github.com/affaan-m/ECC) | 289,613 | Skills, instincts, memory, security across Claude Code, Codex, OpenCode, Cursor |
| [planning-with-files](https://github.com/OthmanAdi/planning-with-files) | not recorded | Plans on disk, re-injected by hooks, surviving `/clear` and compaction |
| [claude-mem](https://github.com/thedotmack/claude-mem) | 103,359 | Compress a session and inject it into a later one |
| [cc-switch](https://github.com/farion1231/cc-switch) | 148,947 | One desktop shell over many coding agents |
| [Orca](https://github.com/stablyai/orca) | 106,846 | A fleet runtime: desktop, mobile, remote |
| [OpenHands Agent Canvas](https://github.com/OpenHands/OpenHands) | 91,912 | Self-hosted control plane over ACP agents |
| [Cline](https://github.com/cline/cline) | 70,890 | Autonomous coding as IDE extension, CLI, or SDK |
| [Aider](https://github.com/Aider-AI/aider) | 49,767 | Earlier terminal pair programmer; repo map, not a platform |
| [Goose](https://github.com/aaif-goose/goose) | 55,782 | Local extensible agent: install, execute, edit, test |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | 108,553 | Vendor terminal agent, open implementation |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | 28,738 | Same shape, Qwen stack |
| [Roo Code](https://github.com/RooCodeInc/Roo-Code) | 24,260 | An editor “dev team” of agents |
| [SWE-agent](https://github.com/SWE-agent/SWE-agent) | 20,645 | Research lineage: issue in, patch out |
| [AutoGen](https://github.com/microsoft/autogen) | 61,576 | Multi-agent programming framework, not a coding product |
| [LangGraph](https://github.com/langchain-ai/langgraph) | 43,941 | The agent loop as a recoverable graph |
| [Continue](https://github.com/continuedev/continue) | 36,337 | Open coding agent / IDE line |
| [Plandex](https://github.com/plandex-ai/plandex) | 15,798 | Long-horizon coding; little public push since 2025-10 |

### Recurring practice

These patterns come from official best practices, the Agent Skills standard, and the repositories above. Live X search was unauthenticated in this environment, so this report does not invent tweets. Anthropic itself points at one public thread: Jarred Sumner’s account of migrating Bun from Zig to Rust with workflows.

1. **Version the context file.** `CLAUDE.md`, `AGENTS.md`, Hermes `SOUL.md`, and project context turn repeated corrections into repo state.
2. **Disclose skills progressively.** [agentskills.io](https://agentskills.io/) loads name and description at startup, and the body only on a match. Grok, Hermes, Claude Code, and DeepSeek follow that shape.
3. **Put the plan on disk.** planning-with-files argues that an in-context todo dies at compaction. `task_plan.md`, `findings.md`, and `progress.md` do not. Hooks put the plan back into attention.
4. **Read before write, verify before stop.** Claude Code separates explore, plan, and implement, and wants a check that can fail. OpenCode makes that split two agents.
5. **Review adversarially.** A second agent grades against a rubric so the author does not grade itself.
6. **Isolate.** Worktrees, remote sessions, Docker, cloud computers, and per-session sandboxes stop parallel agents from sharing a working tree by accident.
7. **Layer permissions.** Sends, deletes, credentials, and anything outside the chosen folder stay human. Duet, Cowork, Muse, DeepSeek, and Hermes productize this.
8. **Separate the control plane from the agent.** OpenHands, Orca, and cc-switch assume no single terminal wins every task.

### Two schools

- **Fold.** Anthropic’s 2026-10-22 essay on subagents argues that fewer subagents, plus progressive disclosure of context and tools, can raise quality and cut cost. A small edit should not become a workflow.
- **Fan out.** Dynamic workflows, Muse workflows, DeepSeek agent teams, and Grok Bot groups assume long, parallel, or adversarial work needs a clean context.

Both are right. The cut is contamination. Migrations, audits, flaky reproduction, and multi-perspective review fan out. A function, or a tightly coupled patch, stays in one loop.

## 6. Choosing

| You need | Look first |
| --- | --- |
| Features, tests, and PRs in a repo | Claude Code, Codex, OpenCode, Grok Build, Muse Code |
| Docs, sheets, and memos while you step away | Claude Cowork, DeepSeek Harness |
| On-brand visuals tied to a design system | Claude Design, handed back to Claude Code |
| Morning briefs and exception handling | Claude Duet, Grok Bot, Hermes gateway |
| Self-hosting, model choice, audit | OpenCode, DeepSeek Harness, Hermes, OpenHands, Goose |
| Named bots that hand work to each other | Grok Bot, Hermes bot mode, DeepSeek agent teams |
| Memory and norms above several terminals | ECC, planning-with-files, claude-mem, Agent Skills |

There is no total score. ECC’s star count belongs to a performance layer, not a product. DeepSeek and Hermes are still breaking APIs in public. Plandex is the reminder that last wave’s “agent for large projects” can go quiet.

## 7. A minimal harness, in order

1. Workspace boundary and sandbox before any auto-approval.
2. One acceptance command that can fail.
3. A short project rule file, containing only rules the model actually violates.
4. Skills as folders. Do not load bodies at startup.
5. Long-task plans in the repo, reread each turn.
6. Fresh context for investigation. Do not shard a coupled edit without a reason.
7. Separate approval for outbound actions, deletes, and credentials.
8. Write session summaries back, but keep hard constraints in files, not in the summary.

## 8. Limits of this pass

- Sources are official docs, READMEs, and the GitHub API. There is no shared task benchmark.
- Live X search failed authentication, so community practice is carried by public repositories and vendor essays.
- “Claude Work” is not a product name. The work surface is Cowork. The proactive agent is Duet.
- The Grok Bot coupling to Cursor plans is what the fetched docs page says.
- Stars rot. Re-query the API before citing them elsewhere.

## References

Links and the star table: [sources.md](sources.md). The reading site is `index.html` at the repository root.
