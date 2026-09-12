---
name: purdue-rcac-hpc
description: >-
  Operate Purdue RCAC clusters and Slurm jobs. Use when a task mentions RCAC,
  Purdue clusters, Bell, Gilbreth, Negishi, Gautschi, sbatch, sinteractive,
  squeue, sacct, showpartitions, scratch, Depot, Fortress, GPUs, job arrays,
  or moving and running research code on a Purdue computing cluster. Also use
  for this user's statdept standby workflow. Do not use for generic Unix help
  or non-RCAC schedulers.
---

# Purdue RCAC HPC

Turn the user's objective into an RCAC-safe command sequence, Slurm script, diagnosis, or data-transfer workflow. The user's current request takes precedence over defaults in this skill.

## Load only relevant instructions

- Read [references/rcac-operations.md](references/rcac-operations.md) for cluster choice, access, storage, transfer, recovery, Fortress, Lmod, or RCAC-specific commands.
- Read [references/slurm-workflow.md](references/slurm-workflow.md) for creating, reviewing, submitting, monitoring, cancelling, measuring, arraying, or distributing a job.
- Additionally read [references/statdept.md](references/statdept.md) when the task is for this user's `statdept` Bell/Gilbreth environment.
- Additionally read [references/long-jobs.md](references/long-jobs.md) only when one stateful run may exceed the confirmed wall-time limit.

Do not load references unrelated to the task.

## Establish live state

Treat hardware lists, module versions, partition access, quotas, purge windows, and retirement dates as changeable. When cluster access is available and the task requires concrete values, run the relevant read-only checks before deciding:

```bash
hostname -f
slist
showpartitions
myquota
module list
```

If live checks are unavailable, use placeholders and label unverified assumptions. Never turn a tutorial account, partition, QoS, module version, or username into a real command.

## Execution protocol

1. Identify the target cluster, workload type, scale, expected duration, software environment, input location, and durable output location. Infer routine details from the project when possible.
2. Keep compute off login nodes. Use login nodes for editing, small environment checks, transfers, and scheduler commands; use Slurm or an Open OnDemand allocation for computation.
3. Choose storage by I/O and lifetime. Never leave the only valuable copy in Scratch or `/tmp`.
4. Recreate modules and environments inside the batch job. Do not rely on the submitter's interactive shell.
5. Map actual application parallelism to nodes, tasks, CPUs per task, memory, GPUs, and time. An allocation does not launch copies automatically; use `srun` or the application's supported distributed launcher.
6. Validate paths, placeholders, Bash syntax, account, partition, QoS, time, resources, logs, and required mail settings before submission.
7. A request to explain, plan, review, or write a job does not authorize `sbatch`, `scancel`, deletion, overwrite, or a remote transfer. Perform those mutations only when requested.
8. After an authorized submission, capture the job ID and inspect the effective Slurm record. Report the job ID, verification result, monitoring command, and output paths.
9. Diagnose from scheduler state, accounting, and logs before changing code or increasing resources.

## Safety and correctness

- Do not bypass Purdue authentication, controlled-data rules, account access, partition ACLs, QoS, or wall-time limits.
- Do not use `sudo` on RCAC nodes.
- Quote Bash paths and arguments. Preview destructive `find`, `rm`, and `rsync --delete` operations against an exact target.
- Do not cancel jobs or delete data unless the exact scope was requested and verified.
- Verify an archive or transfer at the destination before considering source deletion.
- Do not flood Slurm with tiny jobs; choose arrays, throttling, pilot jobs, or a workflow tool when appropriate.
- For `statdept`, enforce every invariant in [references/statdept.md](references/statdept.md), including continuations created from inside a job.

## Expected output

For a job request, create or edit the smallest useful script set and explain only consequential choices. For diagnosis, report evidence, failure category, and the narrowest next action. For operations performed on the cluster, distinguish completed actions from commands merely proposed to the user.
