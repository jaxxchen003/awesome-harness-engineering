# Sources

Snapshot date: 2026-10-27 (Asia/Shanghai). GitHub star counts come from the GitHub REST API on that date. Product behavior is taken from the pages below, not from memory of older releases.

## Primary product docs

- Claude Code overview: https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview
- Claude Code best practices: https://docs.anthropic.com/en/docs/claude-code/best-practices
- A harness for every task (dynamic workflows), 2026-06-02: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
- Claude Cowork: https://claude.com/product/cowork
- Claude Design: https://claude.com/product/design
- Claude Duet: https://claude.com/product/claude-duet
- Introducing Claude Duet, 2026-10-21: https://claude.com/resources/articles/claude-duet
- OpenAI Codex: https://github.com/openai/codex and https://developers.openai.com/codex
- OpenCode: https://github.com/anomalyco/opencode and https://opencode.ai/v2/docs
- DeepSeek Harness: https://www.deepseek.com/en/harness/ and https://github.com/deepseek-ai/deepseek-harness
- DeepSeek Harness quickstart: https://deepseek-harness.github.io/deepseek-harness/en/guide/quickstart
- Cordis paper: https://arxiv.org/abs/2608.25512
- Grok Build: https://docs.x.ai/build/overview
- Grok Build skills, plugins, marketplaces: https://docs.x.ai/build/features/skills-plugins-marketplaces
- Grok Bot: https://docs.x.ai/grok-bot/overview
- Hermes Agent: https://hermes-agent.nousresearch.com/docs and https://github.com/NousResearch/hermes-agent
- Muse Code: https://dev.meta.ai/docs/muse-code
- Muse Code workflows: https://dev.meta.ai/docs/muse-code/workflows
- Agent Skills standard: https://agentskills.io/

## Open-source harnesses and practice repos

Stars below are the 2026-10-27 GitHub API snapshot.

| Repo | Stars | License | Note |
| --- | ---: | --- | --- |
| affaan-m/ECC | 289,613 | MIT | Harness performance layer: skills, instincts, memory, security |
| deepseek-ai/deepseek-harness | 262,467 | MIT | Plugin-native harness on Cordis |
| NousResearch/hermes-agent | 257,343 | MIT | Self-improving agent, messaging gateway, bot mode |
| anomalyco/opencode | 221,881 | MIT | Open coding agent, build/plan split |
| farion1231/cc-switch | 148,947 | MIT | Desktop switcher across coding harnesses |
| openai/codex | 131,021 | Apache-2.0 | Codex CLI / app / web |
| google-gemini/gemini-cli | 108,553 | Apache-2.0 | Gemini terminal agent |
| stablyai/orca | 106,846 | MIT | Fleet runtime for agents such as Grok Bot and Claude Code |
| thedotmack/claude-mem | 103,359 | Apache-2.0 | Cross-session memory compression |
| OpenHands/OpenHands | 91,912 | MIT | Agent Canvas control plane, ACP |
| cline/cline | 70,890 | Apache-2.0 | IDE/CLI/SDK coding agent |
| microsoft/autogen | 61,576 | CC-BY-4.0 | Multi-agent programming framework |
| aaif-goose/goose | 55,782 | Apache-2.0 | Extensible local agent |
| Aider-AI/aider | 49,767 | Apache-2.0 | Terminal pair programmer; last push 2026-05-22 |
| langchain-ai/langgraph | 43,941 | MIT | Graph runtime for resilient agents |
| continuedev/continue | 36,337 | Apache-2.0 | Open coding agent |
| QwenLM/qwen-code | 28,738 | Apache-2.0 | Qwen terminal coding agent |
| RooCodeInc/Roo-Code | 24,260 | Apache-2.0 | Multi-agent editor team |
| SWE-agent/SWE-agent | 20,645 | MIT | Issue-to-patch research agent |
| plandex-ai/plandex | 15,798 | MIT | Long-horizon coding agent; last push 2025-10-03 |

Planning-with-files README (stars not re-fetched after an API field error): https://github.com/OthmanAdi/planning-with-files

## Research limits

- Live X/Twitter search was not authenticated in this environment (`xreach` returned “Not authenticated”). Practitioner practice is therefore cited through public repositories and official write-ups, including Anthropic’s reference to Jarred Sumner’s migration thread: https://x.com/jarredsumner/status/2060050578026189172
- Claude Cowork, Claude Design, and Claude Duet are separate product surfaces. “Claude Work” in the original brief maps to Cowork plus Duet, not to a single product named Claude Work.
- The fetched Grok Bot overview says access is included with paid Cursor plans and can also be linked to SuperGrok or Grok Business. That coupling is reported as documented on https://docs.x.ai/grok-bot/overview and was not independently billing-tested.
