# Awesome Harness Engineering

A field report on the runtime around AI agents: Claude Code, Claude Cowork, Claude Design, Claude Duet, OpenAI Codex, OpenCode, DeepSeek Harness, Grok Build, Grok Bot, Hermes Agent, Muse Code, and the open-source control planes that sit on top of them.

中文报告：[docs/report.zh.md](docs/report.zh.md)  
English report: [docs/report.en.md](docs/report.en.md)  
Sources and star snapshot: [docs/sources.md](docs/sources.md)  
Reading site: [index.html](index.html)

Snapshot date: 2026-10-27. Star counts are a GitHub API reading, not a ranking. Product behavior follows the linked docs.

## What “harness” means here

A harness is the control loop around a model: tools, permissions, context, memory, skills, plans, orchestration, verification, and resume. It is not the weights and not a longer prompt.

Anthropic’s June 2026 essay names the failure modes a harness has to absorb: agentic laziness, self-preferential bias, and goal drift. The cases in this repo are organized around how each product answers those failures.

## Read it

Open `index.html` locally, or read the Markdown. The page has a 中文 / EN switch. The long form lives in `docs/`.

To publish with GitHub Pages, serve the repository root. `index.html` is already there.

## License

[CC BY 4.0](LICENSE). Credit this repository if you reuse the writing. Product names belong to their owners.

## Scope

Live X/Twitter search was not authenticated while this was compiled, so practitioner claims are tied to public repositories and vendor essays rather than unverified posts. “Claude Work” is mapped to Claude Cowork and Claude Duet, which are the actual product names.
