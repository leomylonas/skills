---
name: bootstrap-agent-repo
description: Bootstrap or adapt a repository for coding agents with shared instructions, subagents, documentation, and portable Agent Skills. Use when asked to make a repository agent-ready or set up Codex, Claude Code, Copilot, Gemini CLI, Qwen Code, Cursor, OpenCode, or Windsurf.
---

# Bootstrap an agent-ready repository

Create a small, project-specific shared foundation, not a generic policy dump. The repository root is the target unless the user specifies another directory. This is a change to that repository, not an installation of this skill into it.

Read the [repository's canonical TypeScript and React style guide](../../../STYLEGUIDE.md). Its rules apply when generating or modifying applicable TypeScript/React code, not to unrelated languages or to an empty repository's choice of technology.

## 1. Inspect before changing

- Inspect the repository, including hidden files, existing agent instructions at root and in relevant subdirectories, provider configuration and skills, documentation, project manifests, and CI. Follow existing links only after checking their targets. Identify the repository root and respect nested project boundaries.
- Identify actual languages, structure, build and test commands, and conventions from files and, where safe, by running them. Never invent commands or project facts. An empty repository has no build or test commands yet.
- Look for existing skills in `.agents/skills`, `.claude/skills`, `.github/skills`, `.gemini/skills`, `.qwen/skills`, `.cursor/skills`, `.opencode/skills`, `.windsurf/skills`, and other configured locations. Identify name collisions, distinct instructions, symlinks, and ignored paths before deciding how to integrate them.
- Inspect existing custom agent and subagent definitions, their host-specific configuration, permissions, names, and references as well as skills. Distinguish built-in agents, custom definitions, and ordinary `AGENTS.md` standing instructions.
- Preserve existing content and behavior. Do not overwrite, delete, replace a directory with a symlink, or move existing instructions or skills without understanding references and resolving conflicts. Preserve provider-specific rules that cannot be shared. If instructions conflict or migration would be destructive, leave the affected files intact, explain the conflict and ask for a decision; complete independent safe changes.

## Choose agent hosts

Before making changes, ask which agent hosts the user wants to support. Offer **Codex and Claude Code** as the suggested defaults, and list GitHub Copilot, Gemini CLI, Qwen Code, Cursor, OpenCode, and Windsurf as additional choices; allow any combination, including neither default and other named hosts. If the user has already explicitly specified the complete selection, use it without asking again. An answer accepting the defaults means Codex and Claude Code only; an explicit selection replaces the defaults unless the user says to add to them. Do not treat a model name as a host selection: ask which application runs it.

Wait for the selection before modifying the repository. If interaction is unavailable and no complete selection was given, report that host selection is required; do not silently assume acceptance. Configure only selected hosts, while retaining the shared foundation. Existing configuration for unselected hosts is user content: inspect for conflicts, but do not remove, migrate, or modify it merely because it was not selected. If an unselected host discovers a shared path automatically, note that selecting hosts controls generated configuration, not exclusive access.

## 2. Establish the shared source

Use these shared defaults unless established repository conventions or user instructions call for different canonical locations:

```text
AGENTS.md                      # canonical shared standing instructions
.agents/agents/<name>.md      # canonical reusable custom agent instructions
.agents/skills/<name>/SKILL.md  # canonical project skills
docs/                          # project documentation, when relevant
```

Add only the selected hosts' entry points described in [provider conventions](references/providers.md). Codex, Qwen Code, Cursor, OpenCode, and Windsurf can use the shared `AGENTS.md` directly; do not create redundant provider files for them. Claude Code uses `CLAUDE.md`; Copilot uses `.github/copilot-instructions.md`; Gemini CLI uses `GEMINI.md`.

Create directories only as needed; Git does not track empty directories. If no skills or custom agents exist, add `.agents/skills/.gitkeep` and `.agents/agents/.gitkeep` respectively. If documentation is needed, use the existing documentation location or create `docs/` with a useful index; do not duplicate an existing README.

Create or carefully merge `AGENTS.md`. Include concise, verified project purpose, layout, applicable development and verification commands, important constraints, and pointers to relevant docs, `.agents/skills/`, and `.agents/agents/`. For an empty repository, state that project-specific commands are not yet established; do not prescribe a technology stack. Direct agents without native discovery to inspect the canonical directories and read relevant instructions before using a skill or agent.

Include one complete copy of the canonical style guide at the target repository root as `STYLEGUIDE.md`, not inside individual workspace packages. Link it from the existing README/contributor documentation and instruct agents in root `AGENTS.md` to read and follow it for applicable projects; provider-specific files continue to point to `AGENTS.md`. Do not link back to the skill installation or duplicate the rules in agent instructions. If a target guide differs, preserve it, explain the conflict and ask before replacing or reconciling it. Verify both developer and agent pointers resolve.

Inspect existing guide locations before copying. If another style guide or package-local copy already exists, do not create a duplicate or delete it; ask how to reconcile or relocate it to the root and update references.

Each new provider instruction file should contain only a direct instruction to read and follow root `AGENTS.md`, for example `Read and follow the repository's root AGENTS.md.` A Markdown link alone is not a guarantee of loading: use explicit imperative text. Merge this pointer into an existing file without removing its rules; avoid adding it twice. Do not create competing copies of shared rules. Preserve existing provider-specific instructions and nested overrides. Reconcile shared rules into `AGENTS.md` only when their meanings are compatible and references can be updated safely.

## 3. Connect skills to agent hosts

Consult [provider conventions](references/providers.md) for discovery paths and limitations. The listed hosts other than Claude Code can discover `.agents/skills/` directly on supported surfaces; avoid redundant provider-specific skills directories for them. If Claude Code is selected, create `.claude/skills/` as needed and create a **relative symlink for each canonical skill directory**, e.g. `.claude/skills/example -> ../../.agents/skills/example`. Per-skill links preserve existing provider directories and follow Claude's documented symlink behavior. Do not link `.gitkeep` or create a fake skill. When adding a skill later, add its Claude link too. If a link already points to the correct target, leave it unchanged. Explain how to maintain these links in the target repository's agent documentation.

For pre-existing skills belonging to selected hosts, migrate a skill to the canonical location and replace its old directory with a relative link only after confirming no same-name conflict, preserving all files and checking references. If a safe migration cannot be determined, retain the original and report exactly what remains noncanonical. Never silently replace a conflicting file, symlink, or skill. If symlinks cannot be created or reliably checked out, retain the canonical files, provide explicit manual reading instructions in the relevant provider entry point where safe, and report that native discovery is not established; do not create duplicate copies.

Do not create provider paths merely for models: Ollama supplies models to an agent host, and the host determines instructions and skill discovery. For other hosts, use their documented conventions when requested and make their entry points thin pointers; do not assert unsupported automatic discovery.

## Configure custom agents and subagents

Use `.agents/agents/<name>.md` for the canonical, host-neutral role instructions: purpose, when to delegate, scope, workflow, and expected output. This is a repository convention, not a claimed cross-provider discovery standard. Preserve established useful roles; do not invent project-specific roles merely to populate the directory. Do not confuse a reusable skill with a separately configured agent.

For each existing or explicitly requested custom role, use [subagent conventions](references/subagents.md) to create valid, thin native definitions for selected hosts that support them. Each definition needs its host's required discovery metadata and an explicit instruction to read and follow the canonical role file before acting. Minimal repeated name and description metadata is necessary for host discovery; keep substantive role instructions in the canonical file. Match names consistently, configure only documented capabilities, and preserve existing restrictions. Never promise that prompt text or a pointer enforces a permission boundary; apply native permissions where supported and report any host that cannot enforce the requested restriction. If a host or surface cannot load a reusable role, provide manual instructions and report the limitation rather than claiming native delegation works.

Inspect and reconcile existing agent definitions with the same caution as skills. Do not overwrite unselected-host files, discard host-specific permissions, weaken security, or migrate conflicting definitions without resolving the conflict. Keep existing functional definitions intact if they cannot safely be converted to pointers. Document the canonical directory, the selected hosts' adapter locations, how to add a role and update its adapters, and any discovery, permission, or symlink limitations in the target repository. An empty repository needs the directory and maintenance guidance, not a fabricated custom agent.

## 4. Verify and report

- Check every newly created or modified instruction and agent pointer resolves to the intended file and preserves existing content. Validate native agent metadata, filenames, host-specific permissions and any configuration using available tooling. Check each symlink is relative, resolves to its canonical target, and is not dangling. Check existing resources and references after any migration.
- Exercise relevant installed agent discovery commands if available; otherwise explicitly report that native discovery was not tested. Check applicable ignore rules and warn if intended files or links would not be tracked. Verify the result works on a second application without duplicate pointers or links.
- Summarize the selected hosts, created and updated paths, actual verification, existing conflicts or unsupported hosts, and any manual follow-up. Do not claim that a provider loaded instructions or discovered skills without observing it.
