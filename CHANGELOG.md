# Changelog

What changed in each release of the Symbioza plugin for Claude Code. To update, see the README.

<!-- Maintainers: each release sets `version` in .claude-plugin/plugin.json to the newest heading below; Claude Code
updates an installed copy only when that version changes. deploy/export-plugin.sh enforces it. -->

## 0.3.6 — 2026-10-10

- Cursor: the same package installs as a Cursor plugin. `.cursor-plugin/plugin.json` and a root `mcp.json` connect
  the connector at https://symbioza.dev/mcp; the skill is shared. Direct connection without the plugin, local
  install and sign-in: README and docs/cursor.md.
- The skill's sign-in step names the client: /mcp → Authenticate in Claude Code, Customize → MCPs in Cursor, the
  client's own MCP sign-in elsewhere — and never a token typed into chat.

skill sha256: 1e82779fb622

## 0.3.5 — 2026-10-07

- `pinHost` is accepted and ignored: a machine that is lost or never finishes starting may be replaced, inside
  `budgetUsd`. To keep a benchmark on one GPU model, set `gpu.gpuName` with `gpu.gpuNameExact`.

skill sha256: 65e071c6ec11

## 0.3.4 — 2026-10-07

- One estimate per spec and budget: the answer already carries the price, the runtime, readiness and your balance, so
  your agent estimates again only when the spec or the budget changes.
- One output assumption: without `expectedOutputGb`, the estimate assumes about 1 GB of output, and says so the same
  way in `price.unknown` and `estimate.assumptions`.

skill sha256: 303366033d0a

## 0.3.3 — 2026-10-07

- A best-effort estimate first: when anything about your job predicts how long it runs, the estimate gives the likely
  cost with a range, the expected runtime and a confidence (`estimate`), before you choose a spending limit; when
  nothing does, it gives the price of each hour of running and no total.
- One runtime story: `estimate.runtime` is how long the job likely runs; `runtime.boundSeconds` is only where the run
  would be stopped (your spending limit, your own cap or the 48-hour limit) — a limit, never a prediction.
  `estimatedPriceUsd` appears only when the estimate is priced at your own `maxRuntimeSeconds`, and then equals
  `estimate.likelyUsd`. The spending limit stays the hard maximum.

skill sha256: 9942932dcf5c

## 0.3.2 — 2026-10-05

- README: how to connect from VS Code, GitHub Copilot and any other remote MCP client, and a last step on
  getting your files back. No change to the plugin or the connector.

## 0.3.1 — 2026-09-30

- Estimate before a budget: `estimateExecution` no longer needs `budgetUsd`, so your agent shows you the likely
  cost before asking for a spending limit, then estimates again with the limit you choose. `submitJob` still
  requires one, and only an estimate that carries it returns the `specDigest` a submit accepts.
- No quote without evidence: when nothing about your job predicts how long it runs, the estimate says it is not a
  quote and gives the price of each hour of running instead of a total; `estimatedPriceUsd` comes back only when
  your own `maxRuntimeSeconds` backs it. The estimate also lists what it includes, what is still unknown, and
  what is missing before the spec is final (`readiness`).

## 0.3.0 — 2026-09-30

- Saved Secrets: credentials a job needs (a Hugging Face token, storage keys, a Weights & Biases key, custom
  variables) are saved once on your Secrets page and attached to a job by name. A new tool, `listSecrets`, lists
  your saved sets by name and type only; no page, API or tool shows a value after it is saved. The skill tells your agent never to ask
  you to paste a credential into chat or put one in `env`, and to hand you the Symbioza setup link when access is
  missing.
- Funding: credit can be added in any whole-dollar amount within Symbioza's limits. When the estimate says the job
  is not covered, it names the shortfall and a top-up link prefilled with it; a checkout that opened is not credit.
- The connector now lists eight tools.

## 0.2.5 — 2026-09-29

- Resending a submission: keep the same `clientRequestId` for every resend and you get the original job back, whatever
  became of it. To run a failed, cancelled or lost job again, submit with `retryOf` set to that job and a new
  `clientRequestId`; the retry is checked against your spending limit and balance again.
- The top-up sentence names the one door: sign in and add a credit pack by card.
- Status semantics: `completed` means the command finished and every file was delivered; a finished command with
  missing files reads `failed` with `deliveryOutcome` partial, and the files are collected with `getArtifact`
  before any rerun. Cancelling: the charge ends at the confirmed stop, or at the moment of the request when the
  stop could not be confirmed, never above the spending limit.
- The README's cancel paragraph states the same rule, and no longer mentions shared capacity.

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
