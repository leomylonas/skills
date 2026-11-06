# Provider conventions

Discovery depends on the agent application, its version, settings, and execution surface—not just its model provider. Check the linked documentation when configuring a particular environment.

| Agent host | Shared instructions entry | Project skills | Notes |
| --- | --- | --- | --- |
| OpenAI Codex | `AGENTS.md` | `.agents/skills/` | [Instructions](https://developers.openai.com/codex/guides/agents-md/); [skills](https://developers.openai.com/codex/skills/). |
| Claude Code | `CLAUDE.md` | `.claude/skills/` | Claude documents symlinks to individual skill folders. [Instructions](https://code.claude.com/docs/en/memory); [skills](https://code.claude.com/docs/en/skills). Some cloud surfaces do not load project skills. |
| GitHub Copilot | `.github/copilot-instructions.md` | `.agents/skills/`, `.github/skills/`, or `.claude/skills/` | Availability and locations vary by surface. [Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions); [skills](https://docs.github.com/en/copilot/concepts/customizations/agent-skills). |
| Gemini CLI | `GEMINI.md` | `.agents/skills/` or `.gemini/skills/` | [Instructions](https://geminicli.com/docs/cli/gemini-md/); [skills](https://geminicli.com/docs/cli/skills/). |
| Qwen Code | `AGENTS.md` | `.agents/skills/` | Qwen also loads `QWEN.md`; no duplicate is needed. [Instructions](https://qwenlm.github.io/qwen-code-docs/en/users/features/memory/); [skills](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md). |
| Cursor | `AGENTS.md` | `.agents/skills/` | [Instructions](https://cursor.com/docs/rules); [skills](https://cursor.com/docs/skills). |
| OpenCode | `AGENTS.md` | `.agents/skills/` | [Instructions](https://opencode.ai/docs/rules/); [skills](https://opencode.ai/docs/skills/). |
| Windsurf Cascade | `AGENTS.md` | `.agents/skills/` | Also supports `.windsurf/skills/`. [Instructions](https://docs.windsurf.com/de/windsurf/cascade/agents-md); [skills](https://docs.windsurf.com/es/windsurf/cascade/skills). |

A pointer asks the agent to read another file; it is not a guaranteed automatic import. Skills and instruction discovery can be disabled or differ in cloud/IDE variants. Check actual discovery with the host when possible. Relative symlinks need Git and filesystem support; Windows checkouts may require symlink privileges and appropriate Git configuration.
