# Slurm workflow

Apply this workflow when the task creates, reviews, submits, monitors, cancels, measures, arrays, or distributes RCAC jobs.

## Discover before choosing values

Run read-only discovery when access is available:

```bash
slist
showpartitions
squeue --me
```

Never copy tutorial values such as `hpcexc`, `lab_queue`, `normal`, or `cpu` into a real job unless the live account permits them.

## Map application behavior to resources

- Serial: one task; request only CPUs the program can use.
- Threaded: one task plus `--cpus-per-task`; configure the program's thread count consistently.
- MPI/distributed processes: multiple tasks and a compatible `srun`/MPI launch.
- GPU: request the correct accelerator count and use it in the program; allocation alone does not configure distributed training.
- Multi-node: use only when the application communicates across nodes. A large dataset alone is not justification.

Interpret directives precisely:

| Directive | Decision |
|---|---|
| `--account` | Choose from `slist`. |
| `--partition` | Match required hardware and authorized access. |
| `--qos` | Match account entitlement and policy limits. |
| `--time` | Set a realistic upper bound within QoS limits. |
| `--nodes` | Allocate physical nodes. |
| `--ntasks` / `--ntasks-per-node` | Allocate processes/ranks and placement. |
| `--cpus-per-task` | Allocate threads/cores to each process. |
| `--mem` / `--mem-per-cpu` | Request measured memory using one clear model. |
| `--gpus-per-task` / `--gpus-per-node` | Allocate GPUs using modern Slurm syntax. |

## Generate a batch script

Adapt this template and remove every placeholder before submission:

```bash
#!/bin/bash
#SBATCH --job-name=<name>
#SBATCH --account=<account>
#SBATCH --partition=<partition>
#SBATCH --qos=<qos>
#SBATCH --time=<HH:MM:SS>
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=<cpus>
#SBATCH --mem=<memory>
#SBATCH --output=logs/%x-%j.out
#SBATCH --error=logs/%x-%j.err
# Add only when needed:
##SBATCH --gpus-per-task=1
##SBATCH --mail-user=<username>@purdue.edu
##SBATCH --mail-type=END,FAIL

set -euo pipefail
mkdir -p logs

module reset
module load <required-module>

cd "${SLURM_SUBMIT_DIR:?SLURM_SUBMIT_DIR is unset}"
srun <program> <arguments>
```

Keep directives before executable commands. Use `%A_%a` instead of `%j` for per-task array log names. Recreate the software environment in the job and log relevant executable/module versions when diagnosing reproducibility.

Validate locally on the cluster:

```bash
bash -n job.sh
```

This checks Bash syntax only. Inspect Slurm fields, paths, placeholder removal, input existence, and output directories separately.

## Submit only when authorized

```bash
job_id=$(sbatch --parsable job.sh)
job_id=${job_id%%;*}
[[ "$job_id" =~ ^[0-9]+$ ]]
scontrol show job -o "$job_id"
```

Inspect the effective account, partition, QoS, time, resources, output/error paths, and required mail fields. Enumerate all `sbatch` calls in wrappers, submitters, dependency controllers, and continuation logic; CLI options may override script directives.

For interactive compute:

```bash
sinteractive -A <account> -p <partition> -q <qos> \
  -N 1 -n 1 -c <cpus> -t <HH:MM:SS>
hostname -f
```

Exit the allocation when finished. Open OnDemand apps must request the same appropriate account/resources.

## Monitor and diagnose

Use current state, accounting, then logs:

```bash
squeue -j <job-id> -o '%.18i %.9P %.16j %.2t %.10M %.10l %R'
scontrol show job -o <job-id>
sacct -X -j <job-id> \
  --format=JobID,JobName,Partition,Account,State,ExitCode,Elapsed,AllocCPUS,ReqMem,MaxRSS,NodeList
jobinfo <job-id>
```

Classify before changing anything:

- Policy/launch: account, partition, QoS, time limit, impossible request.
- Environment: module, executable, import, CUDA/ROCm mismatch.
- Resource: host/GPU OOM, wall time, CPU/task mismatch.
- Application: bad input, nonzero exit, numerical failure, code bug.

Read `Reason` for pending jobs. Do not blindly resubmit or add resources.

Only cancel an explicitly identified scope:

```bash
scontrol show job -o <job-id>
scancel <job-id>
```

Warn that cancellation may interrupt checkpoint or output writes.

## Measure utilization

Use `sacct`/`jobinfo` for completed evidence. For live inspection, SSH only to a node currently allocated to the user and run `top -u "$USER"` or `htop`.

For persistent RCAC telemetry:

```bash
module load monitor
monitor cpu percent --csv > cpu-percent.csv &
monitor_pid=$!
trap 'kill "$monitor_pid" 2>/dev/null || true; wait "$monitor_pid" 2>/dev/null || true' EXIT
<application>
```

Request `--exclusive` only when full-node isolation is required; it is not a default monitoring flag.

## Choose a many-task pattern

- Modest independent tasks: a throttled array such as `#SBATCH --array=1-30%5`.
- Very many short tasks: one or more pilot allocations or an RCAC-supported workflow tool such as HyperShell/HTCondor.
- Dependent heterogeneous stages: a workflow manager or explicit Slurm dependencies.
- Tightly coupled work: MPI/distributed launcher, not an array.

Give every task a unique working directory and output. Use `SLURM_ARRAY_TASK_ID` as an index and `%A_%a` in log paths.

Small pilot shape:

```bash
#SBATCH --nodes=1
#SBATCH --ntasks=5
#SBATCH --cpus-per-task=1

for task in {1..5}; do
    srun --exclusive -N1 -n1 -c1 <program> "$task" \
      > "results/${task}.out" 2> "results/${task}.err" &
done
wait
```

Do not exceed allocated concurrency or flood scheduler/storage metadata services.

## Launch multi-node work

An allocation does not replicate the batch shell. For two tasks on each of two nodes:

```bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=2
#SBATCH --cpus-per-task=64

srun hostname
```

Use the application's supported MPI/distributed stack. Validate the node/task layout with `srun hostname` before a costly run.
