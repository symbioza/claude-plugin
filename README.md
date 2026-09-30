<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/symbioza-logo-dark.png">
    <img alt="Symbioza" src="assets/symbioza-logo-light.png" width="300">
  </picture>
</p>
<p align="center">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/github/license/symbioza/claude-plugin?style=flat-square&labelColor=0b0c0c&color=40a56f"></a>
  <a href="https://github.com/symbioza/claude-plugin/releases"><img alt="Latest release" src="https://img.shields.io/github/v/release/symbioza/claude-plugin?style=flat-square&labelColor=0b0c0c&color=40a56f"></a>
  <a href="https://symbioza.dev/agent"><img alt="Hosted MCP server" src="https://img.shields.io/badge/MCP-hosted_server-40a56f?style=flat-square&labelColor=0b0c0c"></a>
  <a href="#1-install"><img alt="Claude Code plugin" src="https://img.shields.io/badge/Claude_Code-plugin-40a56f?style=flat-square&labelColor=0b0c0c"></a>
</p>

# Symbioza for Claude Code

**Run fine-tuning, evaluation and batch-inference jobs on a cloud GPU from Claude Code, under a hard spending
limit.** Your agent prepares the job, shows you a free estimate, and submits it only when you say so. Symbioza
picks the machine, runs the job and returns the files it writes. You are never billed more than the spending
limit you set.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/how-it-works-dark.svg">
    <img alt="How it works: your agent describes the job, gets a free estimate, submits with a total spending limit, Symbioza runs it on a cloud GPU, and the files the run wrote come back. You are never billed more than your budget." src="assets/how-it-works-light.svg">
  </picture>
</p>

## What the plugin adds

Symbioza runs a containerized GPU job on a rented cloud machine under a hard dollar ceiling and collects available artifacts. An agent submits an image, a command and a budget through one MCP connector. Symbioza selects compute, runs the job and reports output delivery and billing separately. Recovery depends on job policy, available compute, remaining budget and compatible checkpoint support.

- **A hosted MCP server** at https://symbioza.dev/mcp (streamable HTTP, sign-in with OAuth, no API key). Its
  eight tools: `estimateExecution`, `submitJob`, `getStatus`, `getArtifact`, `listJobs`, `listSecrets`,
  `describeDataset`, `cancelJob`.
- **One skill, `/symbioza:run-on-symbioza`.** It tells your agent when a job fits Symbioza, what to ask you, and
  the four calls from spec to files: estimate, submit, status, artifact.

For agents deciding whether a job fits: https://symbioza.dev/agent

## 1. Install

```sh
claude plugin marketplace add symbioza/claude-plugin
claude plugin install symbioza@symbioza
```

Or both at once from inside a Claude Code session (Claude Code v2.1.275 or later):
`/plugin install symbioza --marketplace symbioza/claude-plugin`. Tested with Claude Code 2.1.283.

From the shell, the install ends with `Successfully installed plugin: symbioza@symbioza`: start a new Claude
Code session to use it. Installed from inside a session, Claude Code says either `Plugin is now active.` or
`Run /reload-plugins to activate.` (then run `/reload-plugins`).

## 2. Sign in

The connector needs signing in once before its tools appear. Run `/mcp`, select `plugin:symbioza:symbioza`
and choose **Authenticate**: your browser opens to sign in with Google, GitHub or an email address. There is no
key to generate and none to send.

## 3. Check it's connected

You are ready when all three hold:

- `/mcp` lists `plugin:symbioza:symbioza` as connected, with eight tools.
- Typing `/` shows `/symbioza:run-on-symbioza`.
- `claude plugin list` shows `symbioza@symbioza` as enabled.

Already added the connector with `claude mcp add`? Remove that copy (`claude mcp remove symbioza`) so your
agent does not see every tool twice.

## 4. Get a free estimate

Paste this, or run `/symbioza:run-on-symbioza`:

> Help me prepare this GPU job for Symbioza. Ask for the container image, command, input data and expected
> output files. Check that the workload fits the current service limits. Show me a free estimate and anything
> missing, then ask for my total spending limit. Do not submit the job yet.

The estimate needs no spending limit. It shows what the price rests on — a price when your own runtime cap
backs it, otherwise the price of each hour of running and a plain "not a quote" — what it includes, what is
still missing, and the credit on your account. Once you choose a spending limit, the estimate is run again
with it: it shows **the time limit that applies to this job** and whether your credit covers it. Worked specs for fine-tuning, evaluation and batch inference:
https://symbioza.dev/examples

## 5. Add credit and submit

Two gates: an account, and money in it. A new account can connect, estimate and read its own job list;
submitting refuses — “prepaid balance: $0.00 available … Top up, or lower the budget.” — until the account holds
credit. Add credit at https://symbioza.dev/app/topup: sign in and add credit by card. Credit can be added
in any whole-dollar amount within Symbioza's limits — the packs are shortcuts. When the estimate shows the job is
not covered, it names the shortfall and your agent gives you a top-up link prefilled with it. Opening the checkout
is not credit: ask for a fresh estimate once you have paid. Credit packs and billing details:
https://symbioza.dev/pricing

When you are happy with the estimate, tell your agent to submit.

## What your agent will ask you for

- **A supported container image.** Symbioza runs images it has verified on its machines (today, PyTorch and
  TensorFlow images). Any other image is refused with the current list of supported images, before
  anything is booked or charged. A stock image does not contain your code, so the command must fetch your
  script and anything else it needs, for example from a URL.
- **A command** that runs the job, as you would type it in the container.
- **Inputs the machine can reach**: URLs it downloads itself, never files on your laptop. Use a signed link
  with the shortest expiry that covers staging.
- **Output files** written to `/workspace/artifacts/`. That directory is what comes back; anything written
  elsewhere is not collected.
- **A total spending limit** for the job, in US dollars.

## Credentials: saved Secrets, never chat

A job that needs a Hugging Face token, storage keys or another credential gets it from a saved Secret, never from
the chat. Your agent follows this rule:

> Never ask the user to paste API keys, access tokens, passwords or cloud credentials into chat, and never put them in `env` — `env` is for non-sensitive configuration.

Save a credential once at https://symbioza.dev/app/secrets and your agent attaches it to a job by name
(`secrets`). Your agent can list your saved sets with `listSecrets`, which returns names and types only — no page, API or tool
shows a value after it is saved. If a job needs access you have not saved yet, your agent gives you a Symbioza setup
link; open it, save the credential there, and tell your agent when it is done.

How a saved secret is handled: Symbioza stores it encrypted and no page, API or tool returns its value; a job that attaches the set receives the plaintext values in its environment, and the machine running that job can technically read them; an exact copy of a value of four or more characters in the job's printed output, or the base64 or URL-encoded form of a value of eight or more, is replaced with [redacted] before that output is stored — exact matching, not a guarantee against a value printed in any other form; files the job writes are delivered as written and are not scanned or redacted; and a credential an earlier job carried in `env` stays in that job's stored spec — saving a secret does not remove it.

## How runs, files and cancelling work

- **Time.** Every run has a time limit: what your spending limit buys on the machine it books and any cap you
  set, never more than the 48-hour platform maximum. The estimate shows that limit before you spend anything;
  the machine the job books can end a run sooner. A cap above 48 hours is refused at submit.
- **Files.** Check the file delivery status separately from the run status. Available output files have
  download links and hashes you can verify. A completed run does not by itself confirm file delivery or final
  billing. Download links expire after 24 hours; the files do not.
- **Recovery.** A retry or move to another machine depends on the job policy, available compute and what is left
  of your spending limit. Checkpoint recovery needs compatible save-and-resume logic in your code. A run may stop
  without completing.
- **Cancel.** Cancelling stops your job and releases its machine. If the stop is confirmed, you are charged for
  measured time up to then; if the machine cannot be reached, only up to the moment you asked — never more than
  your spending limit.

## What you pay

- Estimates, status checks and your own job list are free.
- Submitted jobs use prepaid credit.
- A job that fails, or that you cancel, is billed for the machine time it used — never more than the spending
  limit you set.
- Your spending limit is a hard ceiling: you are never billed more than that. It is checked before a machine is
  booked and again before every retry. The charge shown after completion may change while billing is
  reconciled.

## Privacy

Jobs run on third-party compute. Your job's image, command, dataset links, environment variables and log tails
are processed on Symbioza's servers and can be stored, so keep secrets out of them: never put credentials in a job
specification or its environment variables, and never send SSH private keys. Credentials belong in your saved
Secrets, which are stored encrypted and injected into the job only when it runs. Only submit code and data you have
permission to use.

Privacy: https://symbioza.dev/privacy · Terms: https://symbioza.dev/terms

## Update and uninstall

These commands update an existing install; installing for the first time, see step 1. Auto-update is off by
default for this marketplace. To update:

```sh
claude plugin marketplace update symbioza
claude plugin update symbioza@symbioza
```

The update ends with `Restart to apply changes.`: restart Claude Code to load the new version.

To uninstall:

```sh
claude plugin uninstall symbioza@symbioza
claude plugin marketplace remove symbioza
```

Uninstalling the plugin does not cancel jobs you already submitted and does not close your Symbioza account.
Cancel running jobs first (ask your agent, which calls `cancelJob`), and manage your account at
https://symbioza.dev/app.

## Get help

- **A problem installing or using the plugin:** open an issue at
  https://github.com/symbioza/claude-plugin/issues. Redact before you post: never paste tokens, keys, account
  details or private job data.
- **Your account, a payment, privacy or security:** use the contact form at https://symbioza.dev/contact.
- **Other clients** (ChatGPT, Claude, any remote MCP client): https://symbioza.dev/plugins

## Changes and license

Release notes: [CHANGELOG.md](./CHANGELOG.md). MIT — see [LICENSE](./LICENSE). The license covers this plugin's
files only.
