# Reusable agent skills

This repository contains portable [Agent Skills](https://agentskills.io/specification).

## Bootstrap an agent-ready repository

[`bootstrap-agent-repo`](.agents/skills/bootstrap-agent-repo/SKILL.md) guides an agent through setting up shared instructions, custom agents and subagents, documentation, and skill discovery in an empty or existing repository. It preserves existing configuration and reports conflicts rather than silently overwriting it.

Install or link `.agents/skills/bootstrap-agent-repo` into your agent's supported skills location, or give the agent the `SKILL.md` path explicitly. In the target repository, ask it to use `bootstrap-agent-repo` to make the repository agent-ready. The skill contains no standalone bootstrap command; the agent inspects and adapts the target repository.

The skill asks which agent hosts to support, suggesting Codex and Claude Code as defaults. It also supports Copilot, Gemini CLI, Qwen Code, Cursor, OpenCode, and Windsurf; specify a complete selection in the request to skip the question. Only selected hosts receive provider-specific configuration, although other hosts may discover shared paths.

Provider discovery, subagent capabilities, and symlink support vary by host and environment; see the skill's [provider reference](.agents/skills/bootstrap-agent-repo/references/providers.md) and [subagent reference](.agents/skills/bootstrap-agent-repo/references/subagents.md). Shared agent roles use `.agents/agents/` as a repository convention; host-specific definitions point back to them where supported.
