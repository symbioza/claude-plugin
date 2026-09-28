---
name: run-on-symbioza
description: Send a job to Symbioza when it no longer fits where you are running it — it needs a GPU this machine does not have, the data or scratch space is too big for local disk, it should run unattended while the user does something else, or the user wants it capped at a hard dollar amount. Covers what to ask the user, the estimate → submit → status → artifact loop, and the two gates (an account, and a prepaid balance).
---

# Run a job on Symbioza

Symbioza runs a containerized job on a rented cloud GPU machine under a hard dollar ceiling and returns what
the run wrote to `/workspace/artifacts/`. This plugin connects its MCP connector at https://symbioza.dev/mcp.

## When to reach for it

Reach for Symbioza when the work no longer fits where you are running it:

- it needs a GPU: training or fine-tuning a model, an evaluation suite, batch inference over a large input;
- it works on a sample here but not at full size — the data or scratch space exceeds local disk;
- it should run unattended while the user does something else — within the time limit the estimate reports;
- the user wants it capped at a dollar amount.

It is a different shape of work, and not a fit, when the job is interactive (a notebook, a desktop, a
session to attach to), needs a long-lived machine administered over SSH, serves live traffic, or produces
something other than files.

## The two gates

Two gates: an account, and money in it.

1. **An account.** The first tool call opens a browser sign-in; there is no API key. Creating an account is
   open. If a tool answers `unauthorized`, ask the user to run /mcp and authenticate the symbioza server.
2. **Money in it.** A new account can connect, estimate for free and read its own job list; only `submitJob`
   refuses — “prepaid balance: $0.00 available … Top up, or lower the budget.” — until the account holds
   credit. Topping up is the user's step: send them to https://symbioza.dev/app/topup — sign in and add
   credit; the page lists the ways to pay.
   `estimateExecution` reports `availableUsd` and `coversThisJob` for the spec, so check before submitting.
   If it shows `claimableUsd`, the account has free compute credit to claim at https://symbioza.dev/app, with
   no payment.

## What to ask the user

Supply the workload: the image, the command, its inputs, the expected outputs and budgetUsd. Symbioza sizes the machine — do not ask the user to choose CPU cores, RAM, VRAM, a GPU model, a provider or a region, and do not derive a resource minimum from a thread count, a worker count or a measurement taken somewhere else. gpu.minVramGb, minRamGb and minCpuCores are optional expert floors: declare one only when the workload has evidence for it, and omit it otherwise — an omitted floor is sized by Symbioza, never treated as zero.

- **The image.** It must be one Symbioza has verified: any other is refused with `invalid_spec` and a
  `supportedImages` list before anything is booked or charged. Pick from that list. A stock image does not
  contain the user's code: the command must fetch the script and anything else it needs (for example from a
  URL). Check that before submitting.
- **The command.** The argv that runs the job. Symbioza runs it as given.
- **The inputs.** Where the data lives: a URL the rented machine fetches itself (this machine is never on the
  data path), or nothing if the command downloads its own. To have a file staged and hash-verified, call
  `describeDataset` with its https URL and paste the `suggestedEntry` it returns into `datasets[]`. Declare
  `datasetGb` as the total data the machine will ingest.
- **The outputs.** Every deliverable must be written to `/workspace/artifacts/`. Files written anywhere else
  are not collected, so a command that saves its model elsewhere finishes with nothing delivered. Check the
  command before submitting. Keep deliverables small: models, logs, metrics.
- **The budget.** budgetUsd is what you are willing to spend on the job, and you are never billed more than that. Ask for it
  explicitly; there is no default. It is a hard ceiling, not an estimate: it is checked before a machine is
  booked and again before every retry.

Do not ask how long the job will take. Each run's time limit is derived for the machine it books: what is
left of `budgetUsd` buys time at that machine's hourly rate, bounded by the 48-hour platform maximum, by
`maxRuntimeSeconds` when the user has a cap of their own (a value above 172800 is refused at submit), and by
how long that machine is available. The estimate's `runtime` reports the limit that applies (`boundSeconds`,
with a `note`): show it to the user, and do not promise a run longer than it.
Put a `gpu` block in the spec for CUDA work — it may be empty. A spec with no `gpu` block is refused with
`no_compute`.

## The loop

1. **`estimateExecution`** with the spec — free, books nothing, needs no balance. It returns the price,
   `specDigest`, the account's `availableUsd` and `coversThisJob`, and `runtime`. Show the user the estimate
   and the ceiling before booking.
2. **`submitJob`** with the spec, that `specDigest` and a `clientRequestId` of your own — the one call that
   spends. It returns an `executionId` and the job runs asynchronously under `budgetUsd`. A spec changed
   since the estimate is refused; the same `clientRequestId` returns the same job.
3. **`getStatus`** with the `executionId` — poll it, and read `nextAction` before touching the spec. For a
   failed job it says "nothing of yours ran" when every attempt died before the command started, and "fix
   and resubmit" only when the user's own code ran and failed.
4. **`getArtifact`** with the `executionId` — the exit code, the artifact manifest and the price. Read
   `deliveryComplete` and `deliveryDetail`: a finished run does not by itself mean every file arrived, and
   each manifest entry carries its own status. The charge shown after completion may change while billing is reconciled. Your total charge for the job, including retries, will not exceed your approved budget.

Lost the `executionId` — a new session, the next morning? `listJobs` returns this account's own jobs, newest
first, and is free. `cancelJob` stops a running job and ends its spend: it releases the machine, or only this
job's share of a shared machine, which may keep serving other work. Machine time already used is still billed,
within `budgetUsd`. Status, artifacts, cancel and
the job list are scoped to the account that submitted the job.

## Long runs

If a machine fails, the job may be retried or moved to another machine. That depends on the job's policy, on
compatible compute being available and on what is left of `budgetUsd`: each attempt is booked only if the
remainder covers it, and a run can stop without completing. Files named `ckpt_step<N>.pt` written to `/workspace/artifacts/` are streamed off
the machine as checkpoints; on a replacement machine `SYMBIOSA_RESUME` holds the absolute path of the newest
one. The command must read `SYMBIOSA_RESUME` and load that checkpoint to continue — a command that ignores it
restarts from the beginning. Checkpoint recovery needs that save-and-resume logic in the user's own code.

More, for the agent deciding: https://symbioza.dev/agent
