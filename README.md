# Jobiloom Plugin prototype

Find jobs by title through Jobiloom MCP and submit one application using a saved resume, reusable profile, and browser tools. Apply through Greenhouse, Lever, Ashby, or a custom company form. Prepare a cover letter when requested or required.

The plugin contains one shared [agent skill](skills/jobiloom-apply/SKILL.md) for Codex and Claude Code. The agent inspects each live form rather than using ATS-specific automation. Login, CAPTCHA, assessments, and unsupported document formats can still require your help. Bulk applications and saved application history are outside this prototype.

## Connect the tools

Use a local agent client with Node.js and a browser available. Supply the confirmed Jobiloom MCP endpoint. For local development, run the existing Jobiloom web app separately and use `http://localhost:3000/api/mcp`.

Replace `https://YOUR-JOBILOOM-HOST/api/mcp` below with your endpoint. Do not add a second connection if these tools are already configured.

For Codex:

```sh
codex mcp add jobiloom --url https://YOUR-JOBILOOM-HOST/api/mcp
codex mcp add playwright -- npx -y @playwright/mcp@latest --isolated
```

For Claude Code, run these commands from the workspace where you will apply:

```sh
claude mcp add --transport http jobiloom https://YOUR-JOBILOOM-HOST/api/mcp
claude mcp add playwright -- npx -y @playwright/mcp@latest --isolated
```

Playwright opens a browser with an isolated session. If it reports a missing browser, use its browser installation tool. The browser tool must have access to the actual resume and cover-letter files. A chat attachment must also be accessible as a file.

Use a dedicated Jobiloom workspace outside this plugin's Git repository and installation folder. The agent stores your profile there. The agent client and browser tools can also retain their own session artifacts.

Setup follows the [Codex MCP guide](https://learn.chatgpt.com/docs/extend/mcp?surface=cli), [Claude Code MCP guide](https://code.claude.com/docs/en/mcp), and [Playwright MCP documentation](https://github.com/microsoft/playwright-mcp).

## Load the plugin in Codex

From this plugin directory, register the bundled marketplace and install the plugin:

```sh
codex plugin marketplace add .
codex plugin add jobiloom@jobiloom
```

Start a new Codex session in your Jobiloom workspace. Invoke `$jobiloom:jobiloom-apply`, or describe the Jobiloom task in plain language. Where the client offers plugin setup, its setup entry opens the same skill for onboarding.

The package uses the supported `.codex-plugin/plugin.json` compatibility format. See the [official plugin packaging guide](https://developers.openai.com/plugins/build/plugins).

## Load the plugin in Claude Code

For a local run, launch Claude Code from your resume workspace with the absolute plugin path:

```sh
claude --plugin-dir /absolute/path/to/plugin
```

Invoke `/jobiloom:jobiloom-apply` followed by your request. For a persistent installation, run these commands inside Claude Code instead:

```text
/plugin marketplace add /absolute/path/to/plugin
/plugin install jobiloom@jobiloom
```

See the [Claude Code plugin documentation](https://code.claude.com/docs/en/plugins-reference).

## Set up your profile

Open a dedicated workspace and invoke `$jobiloom:jobiloom-apply` in Codex or `/jobiloom:jobiloom-apply` in Claude Code. The agent starts profile and resume setup immediately if either is missing. You do not need to choose setup from a menu. You can also ask:

> Use Jobiloom to set up my applicant profile. Ask me for my resume PDF and missing details. Do not apply to any jobs.

The agent requests an accessible PDF, extracts details, and asks you to confirm them. It collects missing common information and offers optional demographic preferences and a cover-letter template. It also asks whether to remember new reusable answers from future forms.

Before job searches, previews, cover letters, or applications, the agent requires a readable resume PDF and a profile with your confirmed name and email. Session-only use requires the same facts and resume without saving a profile. A ready existing profile is reused, and a missing or unreadable resume triggers a replacement request. Once setup is ready, the agent continues your original request or asks for a job title or link.

Your workspace contains:

```text
.jobiloom/
  profile.json
  resume.pdf
  cover-letter-template.pdf  # Optional supplied template; extension may differ.
  cover-letter.txt           # Current job's draft, when generated.
```

The agent creates these files when needed. It tells you where data will be saved and supports session-only use if you decline persistence. Profile files are ordinary local files, without automatic encryption or cross-device sync. Use the same workspace in Codex and Claude Code to reuse the same data. Filesystem and browser access still depend on your client settings; the plugin does not grant itself permission.

Ask Jobiloom to show or correct saved information, replace your resume or template, or delete the saved profile. Original supplied documents remain separate. See the [profile workflow](skills/jobiloom-apply/references/profile.md) for the data format and storage rules.

## Apply once

Attach your resume or provide its accessible path, then ask:

> Use Jobiloom to find Software Engineer jobs and submit one application using my saved profile. Include a tailored cover letter. Stop after that application.

The request authorizes a real submission. If no profile exists, the agent starts onboarding first. It searches by title, selects the first result with a usable link regardless of ATS, fills the form, and submits. It does not assess whether you meet the job requirements. You can also supply a direct application URL.

The agent writes prose answers from confirmed facts and job context. If a required personal fact is missing, it asks you and continues after your reply. It saves reusable answers according to your preference, with country or employer context where relevant. An unresolved obstacle leaves the browser open and blocks submission.

Cover letters use the current company and role. A form can receive letter text or a real file in an accepted format. If the agent cannot create the required file format, it asks you for a suitable file. To prepare a letter without applying, ask:

> Use Jobiloom to draft a cover letter for this job using my profile: [job URL]. Do not fill or submit anything.

To inspect a completed form before submission, explicitly ask for a preview. Setup, search, and standalone cover-letter requests never authorize submission.

Expect a chat report containing the company, title, application URL, and `submitted`, `not_submitted`, or `unconfirmed`. A successful run includes the visible confirmation text. An ambiguous submission is not retried automatically.

After profile setup, check the search connection without applying:

> Use Jobiloom to find Software Engineer jobs and show one result. Do not fill or submit anything.

Confirm that the agent verifies the existing profile and resume, calls `search_jobs`, and returns a real listing link. If setup is incomplete, it completes onboarding before searching. Search uses a literal title substring and checks at most three pages. The catalog targets the US market but includes unknown US classifications.

## Share through GitHub

Publish the contents of this directory as the root of a GitHub repository. Both marketplace files reference the plugin at the repository root. Replace `OWNER/REPO` with that repository's name.

For Codex:

```sh
codex plugin marketplace add OWNER/REPO
codex plugin add jobiloom@jobiloom
```

For Claude Code:

```text
/plugin marketplace add OWNER/REPO
/plugin install jobiloom@jobiloom
```

Connect the MCP tools separately in each client. No production endpoint or credentials are bundled. Publishing this directory inside another repository requires adjusting the marketplace locations and plugin source paths.
