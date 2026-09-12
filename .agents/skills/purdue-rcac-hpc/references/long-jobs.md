# Long stateful jobs

Use this reference only when a single stateful run may exceed its confirmed wall-time limit. For `statdept`, the recorded ceiling is four hours.

## Choose the continuation model

- Split independent inputs into arrays or pilot tasks instead of checkpointing one monolith.
- Do not add continuation machinery when the run fits comfortably.
- For a stateful long run, checkpoint periodically and leave time for a final checkpoint before wall time.
- Add automatic resubmission only when explicitly authorized. Cap attempts and prevent duplicate continuation mechanisms.

## Require a real checkpoint

For iterative ML, save model, optimizer, scheduler, epoch/global step, RNG states, distributed sampler state, AMP scaler, EMA/best-metric state, dataloader/stream position when needed, and the resolved configuration/version marker.

Write atomically through a temporary file and rename. Keep multiple recent checkpoints. Resume automatically by default; make fresh start explicit. Reject corrupt/incompatible checkpoints instead of silently restarting.

Write a `DONE` sentinel only after configured work and durable output writes finish. Verify step and metric continuity after resume.

## Handle impending wall time

Request a warning:

```bash
#SBATCH --signal=B:USR1@300
```

The application should treat `SIGUSR1` as “checkpoint at the next safe boundary and exit,” not serialize mid-step. Keep periodic checkpoints because preemption may not provide a usable warning.

Distinguish outcomes:

- Exit 0 plus `DONE`: stop.
- Confirmed timeout/preemption plus verified checkpoint: continuation may be submitted.
- OOM, NaN, corrupt checkpoint, bad input, or other application failure: stop and diagnose.
- No progress across two links: stop.
- Chain limit reached: stop.

Do not combine scheduler requeue and self-submission unless duplicate execution is ruled out.

## statdept continuation contract

Every new job must independently set and verify:

```bash
--mail-user=<user_id>@purdue.edu --mail-type=END,FAIL
```

Use `sbatch --parsable`, extract a numeric job ID, then inspect `scontrol show job -o <job-id>`. Verify mail, account, QoS, partition, and time for every continuation. Existing-job settings do not configure a separately submitted job.

Use a finite exported chain index:

```bash
CHAIN_INDEX=${CHAIN_INDEX:-0}
MAX_CHAIN_LINKS=<finite-count>
(( CHAIN_INDEX + 1 < MAX_CHAIN_LINKS )) || exit 1

response=$(sbatch --parsable \
  --export=ALL,CHAIN_INDEX="$((CHAIN_INDEX + 1))" \
  --mail-user=<user_id>@purdue.edu \
  --mail-type=END,FAIL \
  "$(readlink -f "$0")")
next_job_id=${response%%;*}
[[ "$next_job_id" =~ ^[0-9]+$ ]]
scontrol show job -o "$next_job_id"
```

Do not treat this snippet as a complete wrapper: add project-specific checkpoint validation, progress comparison, mail-field validation, error classification, and durable chain logging before use.

For dependency chains, choose `afterok` when only successful completion should advance. If using `afterany`, the next worker must inspect the previous outcome and refuse real failures.

## Preflight before a real chain

1. Run a short job that creates a periodic checkpoint.
2. Exercise the warning/checkpoint path.
3. Start another short job and verify automatic resume at the expected step.
4. Compare metrics across the boundary.
5. Deliberately fail the application and prove it does not resubmit.
6. Prove the chain cap and no-progress guard stop execution.
7. For `statdept`, verify effective mail fields on every generated job ID.
