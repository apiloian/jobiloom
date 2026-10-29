# Jobiloom Plugin prototype

Find jobs by title through Jobiloom MCP and submit one application using a saved resume, reusable profile, and browser tools. Apply through Greenhouse, Lever, Ashby, or a custom company form. Prepare a cover letter when requested or required.

Codex and Claude Code use the same [agent skill](skills/jobiloom-apply/SKILL.md). Login, CAPTCHA, assessments, and unsupported document formats can require your help. Bulk applications and saved application history are outside this prototype.

Title search selects the first result with a usable link. It does not check location, seniority, or whether you meet the job requirements. Supply a job URL when you want a specific vacancy.

## Open your .jobiloom folder

Use a local agent client with Node.js and a browser available. Create a workspace outside the plugin repository or installation folder:

```sh
mkdir -p ~/.jobiloom
cd ~/.jobiloom
```

Choose your client below. Skip installation or connection commands for components you already have configured.

## Install in Codex

Run these commands in your terminal:

```sh
codex plugin marketplace add apiloian/jobiloom
codex plugin add jobiloom@jobiloom
codex mcp add jobiloom --url https://jobiloom.com/api/mcp
codex mcp add playwright -- npx -y @playwright/mcp@latest --isolated
```

Start a new Codex session in your applicant workspace and invoke `$jobiloom:jobiloom-apply`.

## Install in Claude Code

Run these commands in your terminal from your applicant workspace:

```sh
claude mcp add --transport http jobiloom https://jobiloom.com/api/mcp
claude mcp add playwright -- npx -y @playwright/mcp@latest --isolated
claude
```

Inside Claude Code, install the plugin:

```text
/plugin marketplace add apiloian/jobiloom
/plugin install jobiloom@jobiloom
```

Start a new Claude Code session in the same workspace and invoke `/jobiloom:jobiloom-apply`.

Playwright opens an isolated browser session. If it reports a missing browser, use its browser installation tool. The browser must have access to the actual resume and cover-letter files. A chat attachment must also be accessible as a file.

See the [Codex MCP guide](https://learn.chatgpt.com/docs/extend/mcp?surface=cli), [Claude Code MCP guide](https://code.claude.com/docs/en/mcp), and [Playwright MCP documentation](https://github.com/microsoft/playwright-mcp) for connection help.

## Set up your profile

On first use, the agent requests a resume PDF and asks you to confirm your name, email, and whether to save locally or use the profile for this session only. It asks other personal questions when an application requires them. You can supply a cover-letter template or set disclosure preferences later.

A readable resume PDF and confirmed name and email are required before searches, previews, cover letters, and applications, including session-only use. A ready profile is reused. A missing or unreadable resume triggers a replacement request. The agent continues your original request after setup, or asks for a job title or link.

Saved profiles and document copies live directly in this folder:

```text
~/.jobiloom/
  profile.json
  resume.pdf
  cover-letter.txt
```

The agent creates the cover-letter file when needed. These are local files without automatic encryption or cross-device sync. Reuse the same folder in both clients. Session-only use avoids saving a profile or durable document copies. Client and browser tools can retain their own session artifacts.

Ask Jobiloom to show or correct saved information, replace your resume or template, or delete the saved profile. Original supplied documents remain separate. See the [profile workflow](skills/jobiloom-apply/references/profile.md) for the data format and storage rules.

## Try a search

After profile setup, check the search connection without applying:

> Use Jobiloom to find Software Engineer jobs and show one result. Do not fill or submit anything.

The agent verifies your profile and resume, calls `search_jobs`, and returns a listing link. Search uses a literal title substring and checks at most three pages. The catalog targets the US market but includes unknown US classifications.

## Preview or apply once

To inspect a filled application before submission, ask:

> Prepare my application for [job URL]. Fill the form and attach my resume, but stop before submitting.

To submit one application, ask:

> Use Jobiloom to apply once to [job URL] using my saved profile.

This request authorizes one submission. You can also ask the agent to search by title and apply to the first result with a usable link.

The agent asks for missing required facts and continues after your reply. It saves reusable answers according to your preference. An unresolved obstacle leaves the browser open and blocks submission.

The agent prepares a tailored cover letter when you request one or a form requires it. If it cannot create the required format, it asks for a suitable file. To draft a letter without applying, ask:

> Use Jobiloom to draft a cover letter for this job using my profile: [job URL]. Do not fill or submit anything.

Setup, search, preview, and standalone cover-letter requests never authorize submission.

Expect a chat report containing the company, title, application URL, and `submitted`, `not_submitted`, or `unconfirmed`. A successful run includes the visible confirmation text. An ambiguous submission is not retried automatically.

## Update an existing installation

Version `0.3.0` requires profile setup before job searches, previews, cover letters, and applications. Setup now defers optional questions until needed. Once this version is published to [apiloian/jobiloom](https://github.com/apiloian/jobiloom), refresh your installation and start a new session. Existing sessions can retain older instructions.

For Codex, run in your terminal:

```sh
codex plugin marketplace upgrade jobiloom
codex plugin add jobiloom@jobiloom
```

For Claude Code, run in your terminal:

```sh
claude plugin update jobiloom@jobiloom
```

See the [Codex plugin guide](https://developers.openai.com/plugins/build/plugins) and [Claude Code update instructions](https://code.claude.com/docs/en/discover-plugins#update-plugins-now).

## Local development

From the plugin checkout, register and install the local Codex marketplace:

```sh
codex plugin marketplace add .
codex plugin add jobiloom@jobiloom
```

For a local Claude Code session, run from your separate applicant workspace:

```sh
claude --plugin-dir /absolute/path/to/plugin
```

The Jobiloom web app runs separately. Use `http://localhost:3000/api/mcp` when connecting to a local server. Both marketplace files reference the plugin at the repository root. Publishing it inside another repository requires adjusting their source paths.
