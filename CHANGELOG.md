# Changelog

Each release sets `version` in `.claude-plugin/plugin.json` to the heading below it. Claude Code updates an
installed copy only when that version changes, so every published change gets a new version here.

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
