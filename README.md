# Reusable agent skills

This repository contains portable [Agent Skills](https://agentskills.io/specification).

## Bootstrap an agent-ready repository

[`bootstrap-agent-repo`](.agents/skills/bootstrap-agent-repo/SKILL.md) guides an agent through setting up shared instructions, custom agents and subagents, documentation, and skill discovery in an empty or existing repository. It preserves existing configuration and reports conflicts rather than silently overwriting it.

Install or link `.agents/skills/bootstrap-agent-repo` into your agent's supported skills location, or give the agent the `SKILL.md` path explicitly. In the target repository, ask it to use `bootstrap-agent-repo` to make the repository agent-ready. The skill contains no standalone bootstrap command; the agent inspects and adapts the target repository.

The skill asks which agent hosts to support, suggesting Codex and Claude Code as defaults. It also supports Copilot, Gemini CLI, Qwen Code, Cursor, OpenCode, and Windsurf; specify a complete selection in the request to skip the question. Only selected hosts receive provider-specific configuration, although other hosts may discover shared paths.

Provider discovery, subagent capabilities, and symlink support vary by host and environment; see the skill's [provider reference](.agents/skills/bootstrap-agent-repo/references/providers.md) and [subagent reference](.agents/skills/bootstrap-agent-repo/references/subagents.md). Shared agent roles use `.agents/agents/` as a repository convention; host-specific definitions point back to them where supported.

## Bootstrap a pnpm workspace

[`bootstrap-pnpm-workspace`](.agents/skills/bootstrap-pnpm-workspace/SKILL.md) guides an agent through setting up a pnpm workspace with a pinned Node version for nvm, separate application and library directories, and native projects alongside Node packages. Give an agent the skill path or install the skill in its supported location, then ask it to bootstrap the target repository. The skill adapts existing configuration and does not provide a standalone command.

## Bootstrap shared web tooling

[`bootstrap-shared-web-config`](.agents/skills/bootstrap-shared-web-config/SKILL.md) guides an agent in creating a workspace library of shared ESLint, Prettier, Stylelint, and TypeScript configurations. It covers browser-oriented TypeScript and React libraries, Vite SPAs, and Next.js, resolving current stable tool versions when run and reporting compatibility conflicts.

The skill selects one TypeScript version supported by the current lint and framework tooling, and composes Next's ESLint plugin with a compatible React plugin when necessary.

The shared and app skills specify the same ESLint baseline: recommended JavaScript, TypeScript, React, and React Hooks presets, with Next-specific rules for Next apps and Prettier conflict suppression last. Each skill describes the applicable presets and local override behavior.

## Bootstrap a React Vite app

[`bootstrap-react-vite-app`](.agents/skills/bootstrap-react-vite-app/SKILL.md) creates or adapts a React and TypeScript SPA with Vite and compatible quality tooling.

## Bootstrap a Next.js app

[`bootstrap-nextjs-app`](.agents/skills/bootstrap-nextjs-app/SKILL.md) creates or adapts a Next.js and React TypeScript application with compatible quality tooling.

Both app skills offer to use an existing shared web-config package, requiring explicit confirmation; otherwise they use local configuration.
