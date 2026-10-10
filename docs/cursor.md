# Symbioza in Cursor

The same package that installs in Claude Code installs in Cursor: `.cursor-plugin/plugin.json` is the Cursor
manifest, `mcp.json` at the root connects the hosted MCP server at https://symbioza.dev/mcp, and the
`run-on-symbioza` skill is shared. Do not also add the server by hand while the plugin is installed, or your agent
sees every tool twice.

## Install locally

1. Copy this repository to `~/.cursor/plugins/local/symbioza`. Use a real copy: Cursor loads a symlink only when its
   target is inside `~/.cursor/plugins/local`, and skips one that points at a checkout elsewhere on disk.
2. Restart Cursor, or run **Developer: Reload Window** from the Command Palette.
3. Open **Customize** in the sidebar and confirm that the `run-on-symbioza` skill and the `symbioza` MCP server are
   listed.

On a Teams or Enterprise plan, an admin setting (**Allow Local Plugin Imports**) may block step 1; ask your admin or
connect directly instead (README, "Cursor").

## Sign in

The server uses OAuth: there is no key to generate and none to type. In **Customize**, enable the `symbioza` server;
Cursor opens your browser to sign in with Google, GitHub or an email address and returns to the app. Cursor's desktop
callback is `http://localhost:8787/callback`; Cursor's web and cloud agents use
`https://www.cursor.com/agents/mcp/oauth/callback`. Nothing in this package configures them.

Never paste a token, key or password into the chat: the plugin ships none, and the skill tells your agent never to ask
for one.

## Use it

Ask for a free estimate, or invoke the skill with `/run-on-symbioza`:

> Help me prepare this GPU job for Symbioza. Ask for the container image, command, input data and expected
> output files. Check that the workload fits the current service limits. Show me a free estimate and anything
> missing, then ask for my total spending limit. Do not submit the job yet.

The estimate needs no spending limit and books nothing. Your agent submits only after you choose a spending limit
and tell it to submit; the limit is a hard ceiling and you are never billed more than it. Output files are collected
from `/workspace/artifacts/` inside the container. Cancelling, file delivery and the time limit work as the README
describes.

## Costs, privacy, support

- Estimates, status checks and your own job list are free; submitted jobs use prepaid credit, added at
  https://symbioza.dev/app/topup. See the README, "What you pay".
- Credentials a job needs live in your saved Secrets (https://symbioza.dev/app/secrets), never in chat or `env`.
  See the README, "Credentials: saved Secrets, never chat" and "Privacy".
- A problem with the plugin: https://github.com/symbioza/claude-plugin/issues (redact first). Your account, a
  payment, privacy or security: https://symbioza.dev/contact.

## Maintainers: checks before a Cursor marketplace submission

Run from the package root. The package is submitted at https://cursor.com/marketplace/publish from the repository's
default branch; every item below must hold on a real Cursor install first.

- [ ] `node validate-template.mjs` from https://github.com/cursor/plugin-template passes (a missing `hooks/hooks.json`
      warning is expected).
- [ ] `.cursor-plugin/plugin.json` and `.claude-plugin/plugin.json` carry the same `version`, equal to the newest
      CHANGELOG heading (`deploy/export-plugin.sh` in the source repository refuses otherwise).
- [ ] Record the Cursor version and commit (**Cursor → About**).
- [ ] The local plugin is discovered: skill and server listed in **Customize**.
- [ ] Sign-in completes in the browser and returns to Cursor.
- [ ] Eight tools are discovered: `estimateExecution`, `submitJob`, `getStatus`, `getArtifact`, `listJobs`,
      `listSecrets`, `describeDataset`, `cancelJob`.
- [ ] "Use Symbioza to list my jobs. Do not submit any work." returns a list (an empty one counts).
- [ ] A free estimate without a spending limit returns a price card and nothing is submitted.
- [ ] No job is submitted and no credential is typed during the test.
