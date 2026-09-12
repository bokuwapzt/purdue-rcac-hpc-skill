# statdept Bell/Gilbreth profile

Apply these instructions only to the user profile recorded in Week 5. Do not impose them on another account.

## Hard constraints

- Account: `statdept`.
- QoS: `standby` only.
- Recorded standby wall-time ceiling: `04:00:00`.
- Every initial job, retry, array, dependency job, and separately submitted continuation must notify the actual Purdue user on exactly `END,FAIL`.
- Prefer Gilbreth for NVIDIA/CUDA work and Bell for CPU work, subject to live availability and access.
- Prefer `uv` for this user's Python projects when their project files support it.

Confirm the profile against live output before submission:

```bash
slist
showpartitions
```

If live state conflicts with the profile, stop and report the mismatch. Do not substitute a different account/QoS or bypass access.

## Required directives

```bash
#SBATCH --account=statdept
#SBATCH --qos=standby
#SBATCH --time=04:00:00
#SBATCH --mail-user=<user_id>@purdue.edu
#SBATCH --mail-type=END,FAIL
```

Replace `<user_id>` before submission. `#SBATCH` values do not undergo normal Bash variable expansion, so do not use `$USER@purdue.edu` in a directive.

Every reachable `sbatch` path must parse those literal directives or pass both mail flags explicitly:

```bash
sbatch --mail-user=<user_id>@purdue.edu --mail-type=END,FAIL job.sh
```

Mail settings on a running job do not propagate to a separately submitted continuation.

## Partition rules from the Week 5 snapshot

Verify each with `showpartitions`; the snapshot can age.

- Gilbreth standby-accessible: `a10`, `a30`, `a100-40gb`, `a100-80gb`.
- Gilbreth `training` and H100 path: recorded as inaccessible to `statdept`; escalate to the PI/RCAC rather than route around it.
- Bell: recorded as `cpu` only for this account. Bell GPU hardware is AMD MI50/MI60 and incompatible with CUDA-only binaries.
- Bell is marked for Fall 2026 retirement in the course; verify that it still accepts jobs.

Choose the smallest accelerator that fits memory/compute, then use live capacity as a turnaround-time tie-breaker. Do not assume A100 is faster overall when it waits much longer.

GPU request shape:

```bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=<cpus>
#SBATCH --gpus-per-task=1
```

Use legacy `--gres=gpu:1` only if current RCAC guidance requires it.

## Paths and environment

```text
/scratch/gilbreth/<user_id>/<project>/
/scratch/bell/<user_id>/<project>/
```

The workshop records a 60-day inactive threshold generally and 30 days on Bell. Verify current policy and move durable results out of Scratch.

For a compatible Gilbreth `uv` project:

```bash
module reset
module load cuda
cd /scratch/gilbreth/<user_id>/<project>
uv run python -u train.py
```

Discover the live CUDA module and verify the project lockfile/framework build. Remove CUDA/GPU directives for a Bell CPU job.

## Submission gate

Before any authorized `sbatch`:

1. Confirm username, host, `statdept`, standby, chosen partition, time, and resources.
2. Inspect the worker and every wrapper/dispatcher/continuation for all `sbatch` calls.
3. Require literal mail directives or explicit mail flags on every path.
4. Run `bash -n` and a short application preflight.

After each new job ID:

```bash
scontrol show job -o <job-id>
```

Require the effective record to show `Account=statdept`, `QOS=standby`, the intended partition/time, `MailUser=<user_id>@purdue.edu`, and both `END` and `FAIL` in `MailType` (order may differ). Do not report submission as successfully handed off until this verification passes.
