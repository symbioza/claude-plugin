---
name: run-on-symbioza
description: Run a containerized batch job — model training, fine-tuning, evaluation, batch inference, preprocessing — on a rented cloud GPU through the Symbioza connector, under a hard dollar budget. Use when the job needs a GPU this machine lacks or more data or scratch space than local disk holds, or when the user asks to run it remotely with a spending cap. Not for local commands, notebooks, SSH sessions or serving live traffic. Covers what to ask, the free estimate, submitting only after the user approves, polling and collecting files.
---

# Run a job on Symbioza

Symbioza runs a containerized job on a rented cloud GPU machine under a hard dollar ceiling and returns what
the run wrote to `/workspace/artifacts/`. This plugin connects its MCP connector at https://symbioza.dev/mcp.

## When to reach for it

Reach for Symbioza when the work no longer fits where you are running it:

- it needs a GPU: training or fine-tuning a model, an evaluation suite, batch inference over a large input;
- it works on a sample here but not at full size — the data or scratch space exceeds local disk;
- the user wants it run remotely and capped at a dollar amount.

It is a different shape of work, and not a fit, when the job is interactive (a notebook, a desktop, a
session to attach to), needs a long-lived machine administered over SSH, serves live traffic, or produces
something other than files.

## The two gates

Two gates: an account, and money in it.

1. **An account.** The connector needs signing in once before its tools appear; there is no API key.
   Creating an account is open. If Symbioza's tools are not available, or a tool answers `unauthorized`, ask
   the user to run /mcp, select plugin:symbioza:symbioza and choose Authenticate; the browser opens to sign in.
2. **Money in it.** A new account can connect, estimate for free and read its own job list; only `submitJob`
   refuses — “prepaid balance: $0.00 available … Top up, or lower the budget.” — until the account holds
   credit. Topping up is the user's step: send them to https://symbioza.dev/app/topup — sign in and add a
   credit pack by card.
   `estimateExecution` reports `availableUsd` and `coversThisJob` for the spec, so check before submitting.
   If it shows `claimableUsd`, the account has free compute credit to claim at https://symbioza.dev/app, with
   no payment.

## What to ask the user

Supply the workload: the image, the command, its inputs, the expected outputs and budgetUsd. Symbioza sizes the machine — do not ask the user to choose CPU cores, RAM, VRAM, a GPU model, a provider or a region, and do not derive a resource minimum from a thread count, a worker count or a measurement taken somewhere else. gpu.minVramGb, minRamGb and minCpuCores are optional expert floors: declare one only when the workload has evidence for it, and omit it otherwise — an omitted floor is sized by Symbioza, never treated as zero.

- **The image.** It must be one Symbioza has verified: any other is refused with `invalid_spec` and a
  `supportedImages` list before anything is booked or charged. Pick from that list. A verified image is a
  stock image and contains none of the user's code: the command must fetch the script and anything else it
  needs (for example from a URL). Check that before submitting.
- **The command.** The argv that runs the job. Symbioza runs it as given.
- **The inputs.** Where the data lives: a URL the rented machine fetches itself (this machine is never on the
  data path), or nothing if the command downloads its own. To have a file staged and hash-verified, call
  `describeDataset` with its https URL and paste the `suggestedEntry` it returns into `datasets[]`. Declare
  `datasetGb` as the total data the machine will ingest.
- **The outputs.** Every deliverable must be written to `/workspace/artifacts/`. Files written anywhere else
  are not collected, so a command that saves its model elsewhere finishes with nothing delivered. Check the
  command before submitting. Keep deliverables small: models, logs, metrics.
- **The budget.** budgetUsd is what you are willing to spend on the job, and you are never billed more than that. Ask for it
  explicitly; there is no default, and never choose one for the user. It is a hard ceiling, not an estimate:
  it is checked before a machine is booked and again before every retry.

Do not ask how long the job will take. The run's time limit comes from `budgetUsd` at the booked machine's
hourly rate, the 48-hour platform maximum and any `maxRuntimeSeconds` the user sets (a value above 172800 is
refused at submit). The estimate's `boundSeconds`, with its `note`, is the longest the run can go on that
budget and cap; the machine it books can end it sooner, and a `maxRuntimeSeconds` that machine cannot hold is
refused before anything is booked. Show the user that figure and never promise a longer run.
The `gpu` block is optional: every job runs on a GPU machine, and a spec without the block is treated as
an empty one with Symbioza sizing the card. Declare it only to carry `minCudaVersion` or an evidenced floor.

## The loop

1. **`estimateExecution`** with the spec — free, books nothing, needs no balance. It returns the price,
   `specDigest`, the account's `availableUsd` and `coversThisJob`, and `runtime`. Show the user the price,
   the time limit and the budget. **Then stop.** Call `submitJob` only after the user has seen them and told
   you to submit, in this conversation.
2. **`submitJob`** with the spec, that `specDigest` and a `clientRequestId` of your own — the one call that
   spends. It returns an `executionId` and the job runs asynchronously under `budgetUsd`. A spec changed
   since the estimate is refused; the same `clientRequestId` returns the same job, so keep it for every resend
   of the same submission. A resend never pays twice: if that job failed, was cancelled or was lost, you get it
   back with a `hint`. Run it again only when the user asks: submit with `retryOf` set to its `executionId` and
   a new `clientRequestId` — a fresh attempt, checked against the budget and balance again.
3. **`getStatus`** with the `executionId` — every few minutes, not in a tight loop, and read `nextAction`
   before touching the spec. If the user is not waiting, stop polling and give them the `executionId`. When
   the job has finished, call `getArtifact` even if `artifactsReady` is false. `completed` means the command
   finished and every file was delivered; a finished command with missing files reads `failed` with
   `deliveryOutcome` partial — collect the files with `getArtifact` before considering a rerun. For a failed
   job, `getStatus` says "nothing of yours ran" when every attempt died before the command started, and "fix
   and resubmit" only when the user's own code ran and failed.
4. **`getArtifact`** with the `executionId` — the exit code, the artifact manifest and the price. Read
   `deliveryComplete` and `deliveryDetail`: a finished run does not by itself mean every file arrived, and
   each manifest entry carries its own status. The charge shown after completion may change while billing is reconciled. Your total charge for the job, including retries, will not exceed your approved budget.

Lost the `executionId` — a new session, the next morning? `listJobs` returns this account's own jobs, newest
first, and is free. Call `cancelJob` only when the user asks: it cannot be undone. It stops a running job and
releases its machine. If the stop is confirmed, the user is charged for measured time up to then; if the machine cannot be reached, only up
to the moment they asked — never more than `budgetUsd`.
Status, artifacts, cancel and the job list are scoped to the account that submitted the job.

## Long runs

If an attempt fails on the machine's side, the job may be retried or moved to another machine, unless
`pinHost` is set. That depends on compatible compute being available, on the attempt limit and on what is
left of `budgetUsd`: each attempt is booked only if the remainder covers it, and a run can stop without
completing. Files named `ckpt_step<N>.pt` written to `/workspace/artifacts/` are streamed off the machine as
checkpoints; on a replacement machine `SYMBIOSA_RESUME` holds the absolute path of the newest one. The
command must read `SYMBIOSA_RESUME` and load that checkpoint to continue — a command that ignores it restarts
from the beginning. It is unset on a first attempt and whenever no checkpoint could be restored; restore is
best-effort. Checkpoint recovery needs that save-and-resume logic in the user's own code.

More, for the agent deciding: https://symbioza.dev/agent
