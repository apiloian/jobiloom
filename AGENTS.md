# Repository Guidelines

## Project Structure & Module Organization

This repository packages the Jobiloom prototype for Codex and Claude Code. Its behavior lives in `skills/jobiloom-apply/SKILL.md`, shared by both clients. `skills/jobiloom-apply/references/profile.md` defines onboarding and local profile storage. `README.md` covers setup and usage. `.codex-plugin/plugin.json` and `.claude-plugin/plugin.json` hold client manifests; `.agents/plugins/marketplace.json` and `.claude-plugin/marketplace.json` define marketplace entries. Keep plugin names, versions, and root-relative source paths consistent across these files. There are no application source, test, or asset directories.

## Build, Test, and Development Commands

There is no build step, package manager configuration, or automated test runner. Run these commands from the plugin directory unless indicated otherwise:

- `codex plugin marketplace add .` registers the local marketplace.
- `codex plugin add jobiloom@jobiloom` installs the plugin in Codex; start a new session afterward.
- `claude --plugin-dir /absolute/path/to/plugin` loads the plugin for a local Claude Code session. Run it from a separate workspace containing the resume.
- `python3 -m json.tool .codex-plugin/plugin.json` checks JSON syntax when Python is available; repeat for each changed JSON file.

For MCP connection commands and endpoint configuration, follow `README.md`. The Jobiloom web app runs separately.

## Coding Style & Naming Conventions

Use concise Markdown headings and imperative workflow instructions. Preserve YAML frontmatter in skill files. Follow existing JSON formatting: two-space indentation, double-quoted keys, and no trailing commas. Use lowercase, hyphen-separated skill directory names, such as `jobiloom-apply`, with instructions in `SKILL.md`. No formatter or linter is configured.

## Testing Guidelines

Validation is manual; no testing framework or coverage threshold is configured. After workflow changes, load the plugin in each affected client. Invoke the skill alone in a fresh disposable workspace and verify that it starts profile and resume setup instead of showing a task menu. Use the README's read-only search prompt and verify that `search_jobs` returns a real listing only after a profile and readable resume are ready. Exercise existing profiles, missing or unreadable resumes, missing facts, session-only setup, cover-letter preparation, and preview-only requests with synthetic data. Check that Greenhouse, Lever, Ashby, and custom forms are not excluded by ATS metadata. Test submission only with explicit user authorization. Verify visible confirmation before reporting success, and never retry an ambiguous submission automatically.

## Commit & Pull Request Guidelines

Git history is unavailable in this checkout, so no existing commit convention can be established. Use short, imperative subjects, such as `Clarify Greenhouse submission verification`. In pull requests, describe the behavior change, affected clients, and validation performed. Link relevant issues and include screenshots when they clarify browser behavior.

## Security & Configuration

Keep resumes, applicant profiles, cover letters, credentials, production endpoints, and browser session artifacts outside the repository. Store user files directly in the profile directory resolved by the profile reference. When the user's workspace is itself `.jobiloom/`, use it directly without creating a nested `.jobiloom/`. Preserve the prototype's scope: one application per request through Greenhouse, Lever, Ashby, or a custom company form. Local profiles are supported; application history and bulk submissions are not. Never infer demographic answers or invent required personal facts.
