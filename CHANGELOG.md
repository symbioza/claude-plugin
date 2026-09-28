# Changelog

What changed in each release of the Symbioza plugin for Claude Code. To update, see the README.

<!-- Maintainers: each release sets `version` in .claude-plugin/plugin.json to the newest heading below; Claude Code
updates an installed copy only when that version changes. deploy/export-plugin.sh enforces it. -->

## 0.2.4 — 2026-09-28

- The `gpu` block is optional. A spec without it is treated as an empty one: every job runs on a GPU machine and
  Symbioza sizes the card. The skill no longer tells the agent that a missing block is refused.

## 0.2.3 — 2026-09-28

- A clearer page: what the plugin adds comes first, then install, sign in, check the connection, a free estimate,
  and adding credit. "Hosted MCP server" says what the connector is.
- One word for the money limit throughout: your spending limit.
- More precise search keywords.

## 0.2.2 — 2026-09-28

- The skill tells your agent to stop after the free estimate and submit only after you approve, to poll every few
  minutes rather than in a loop, to collect files from a finished job even when none are marked ready, and to
  cancel only when you ask.
- A narrower description, so the skill is used for remote GPU batch jobs and not for ordinary local tasks.
- Corrected the time limit: the estimate shows how long the run can go on your budget and cap; the machine it
  books can end it sooner.
- Checkpoint resume: `SYMBIOSA_RESUME` is unset on a first attempt and whenever nothing could be restored.
- The README says an update needs a restart of Claude Code.

## 0.2.1 — 2026-09-28

- Signing in, as it works in Claude Code: the connector does not open the browser by itself. Run `/mcp`, select
  `plugin:symbioza:symbioza` and choose Authenticate; the tools appear after sign-in.
- Install from the shell ends with "Successfully installed"; start a new session to use the plugin. The
  "Plugin is now active" and `/reload-plugins` messages appear only when installing inside a session.
- The update section says the commands update an existing install.

## 0.2.0 — 2026-09-28

- A first-use guide: what you need, install, authenticate, verify the connection, ask for a free estimate, and
  review it before adding credit or submitting.
- Corrected claims. A run's time limit is set per job and shown in the estimate, under a 48-hour platform
  maximum; the plugin no longer suggests overnight runs. File delivery, recovery and cancellation are described
  as they work: files have their own delivery status, recovery depends on policy, compute and remaining budget,
  and cancelling a job on shared capacity releases only its share.
- The skill explains that a supported image does not contain your code, so the command fetches it.
- New sections on costs, privacy, updating, uninstalling and where to get help; an issue template for
  installation problems.
- A "How it works" diagram with light and dark versions.
- The connector now identifies the plugin as its source (`X-Symbioza-Source: claude-plugin`).

## 0.1.0 — 2026-09-28

- First public release: the Symbioza MCP connector and the `run-on-symbioza` skill. Published without a
  `version` field, so installs of it track the repository commit.
