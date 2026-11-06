# Custom agent and subagent integration

Custom agent formats are host-specific; `.agents/agents/` is this bootstrap convention, not a native universal discovery path. A role file there is the authoritative instruction body. Native definitions are discovery adapters, not copies of the body. Select only the hosts the user requested, and preserve any existing configuration and permission boundaries.

| Host | Project definition | Integration and limitations |
| --- | --- | --- |
| Codex | `.codex/agents/<name>.toml` | Standalone TOML requires `name`, `description`, and `developer_instructions`. Put the explicit read-and-follow instruction in `developer_instructions`. Custom agents inherit the parent sandbox; a prompt is not a read-only sandbox. [Documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents). |
| Claude Code | `.claude/agents/<name>.md` | YAML frontmatter requires `name` and `description`; put the pointer in the Markdown body. Configure tool restrictions when required. [Documentation](https://code.claude.com/docs/en/sub-agents). |
| Copilot | `.github/agents/<name>.agent.md` | YAML frontmatter requires `description`; include `name` and the pointer body. Custom-agent selection and tool availability vary across GitHub, CLI, and IDE surfaces. [Documentation](https://docs.github.com/en/copilot/reference/custom-agents-configuration). |
| Gemini CLI | `.gemini/agents/<name>.md` | YAML frontmatter requires `name` and `description`; put the pointer in the body. Check availability and settings for the installed version. [Documentation](https://geminicli.com/docs/core/subagents/). |
| Qwen Code | `.qwen/agents/<name>.md` | YAML frontmatter includes `name` and `description`; put the pointer in the body. [Documentation](https://qwenlm.github.io/qwen-code-docs/en/users/features/sub-agents/). |
| Cursor | `.cursor/agents/<name>.md` | YAML frontmatter supports `name` and `description`; put the pointer in the body. Cursor also reads compatible `.claude/agents/` definitions; avoid duplicate names and conflicting precedence. [Documentation](https://cursor.com/docs/subagents). |
| OpenCode | `.opencode/agents/<name>.md` | YAML frontmatter requires `description`; set `mode: subagent` for a delegated role. The filename is the name; put the pointer in the body. [Documentation](https://opencode.ai/docs/agents/). |
| Windsurf Cascade | No verified portable custom subagent file convention | Keep the canonical role and instruct the user to supply it manually where supported. Do not claim native custom-agent discovery. |

## Adapter example

For a canonical `.agents/agents/reviewer.md`, a Claude Code adapter can be:

```markdown
---
name: reviewer
description: Reviews changes for correctness and regressions.
---
Read and follow `.agents/agents/reviewer.md` at the repository root before starting the delegated task.
```

A Codex adapter for the same role is different in syntax:

```toml
name = "reviewer"
description = "Reviews changes for correctness and regressions."
developer_instructions = "Read and follow .agents/agents/reviewer.md at the repository root before starting the delegated task."
```

These examples do not enforce read-only operation. Add host-specific restrictions only when needed, using the relevant host's documented permission mechanisms; do not silently remove inherited restrictions or enable broader permissions. When an agent lacks permission to read its canonical file, the pointer cannot work: configure read access if safe or report the limitation. A textual instruction to read a file is not an automatic include or a guarantee the model follows it. Test actual host discovery and delegation where possible, and disclose when not tested.

To add a role later, create its canonical instruction file, then add or update an adapter for each selected supported host, preserving provider-specific metadata and restrictions. Keep descriptions consistent enough to route to the same role. Never replace existing definitions or links without checking conflicts and references; test repeated setup for duplicate adapters.
