# RCAC operations

Use this reference as operational context, not as a tutorial.

## Select and access a cluster

Use the current RCAC resource page and `slist` as authority. The course materials characterize the systems as follows:

| System | Route workloads here when |
|---|---|
| Bell | A still-supported allocation needs traditional CPU/MPI work. The course says retirement is planned for Fall 2026; verify availability. Its listed GPUs are AMD/ROCm, not CUDA devices. |
| Negishi | The account needs general CPU, parallel, or MPI work. |
| Gautschi | The account has access to newer CPU/GPU or H100-class resources. |
| Gilbreth | The workload needs NVIDIA GPU computing, ML, or GPU visualization. |
| Scholar | The work is instructional rather than production research. |
| Anvil | The user has an NSF ACCESS allocation. |
| Rossmann / Weber | Controlled-data approval specifically assigns the work there. Do not reroute controlled data. |

Connect with a confirmed alias or the fully qualified endpoint:

```bash
ssh <username>@<cluster>.rcac.purdue.edu
hostname -f
slist
```

Use ThinLinc at `desktop.<cluster>.rcac.purdue.edu` for a persistent GUI and Open OnDemand at `gateway.<cluster>.rcac.purdue.edu` for browser files, shells, desktops, Jupyter, RStudio, MATLAB, and interactive apps. Treat every Open OnDemand interactive session as a scheduled resource request.

If an SSH alias is absent, use the FQDN or propose a local `~/.ssh/config` entry; never embed passwords, Duo responses, private keys, or tokens.

## Keep work off login nodes

Permit editing, `module` discovery, modest setup, transfers, `sbatch`, `squeue`, and similar lightweight work on login nodes. Move large, long, multi-threaded, parallel, CPU-intensive, or GPU work into `sinteractive`, `srun`, `sbatch`, or an Open OnDemand compute allocation.

Use `hostname -f` to verify where a command is running.

## Select storage

Verify actual access and quota with `myquota`.

| Location | Put here | Operational rule |
|---|---|---|
| `$HOME` / `/home/<user>` | Small durable personal files, config, scripts | Snapshot-backed in course materials; cluster-specific; avoid heavy I/O and large datasets. |
| `/depot/<group>` | Durable group data and shared software | Available across community clusters; snapshot-backed; avoid intensive temporary I/O. |
| `$RCAC_SCRATCH` / `/scratch/<cluster>/<user>` | Active datasets, checkpoints, intermediates, high-I/O results | Cluster-specific, no snapshots, purged by inactivity; never the only copy. |
| `/tmp` | Deliberate node-local cache within one job | Not shared across nodes and not persistent after the job. |
| `/apps` | RCAC-managed application files | Read-only to users; access software through Lmod. |
| Fortress HPSS | Large long-term archives | Bundle small files; never run a job against tape storage. |

The workshop states 60 inactive days for Scratch generally and 30 days on Bell. Verify current policy and run:

```bash
myquota
purgelist
```

Use `flost` only for recoverable snapshot-backed Home/Depot data. Do not promise recovery from Scratch or `/tmp`.

## Transfer data

Choose the mechanism from the operation:

- `scp`: a few one-off files.
- `rsync`: repeated directory synchronization, resumable partials, or selective excludes.
- SFTP: interactive file transfer.
- Globus: large or interruption-prone endpoint transfers.
- Open OnDemand Files: modest manual browser operations.
- Depot: shared cross-community-cluster working data.
- Fortress: long-term archive.

Prefer `rsync` for project staging:

```bash
rsync -avh --partial \
  --exclude='.git/' --exclude='__pycache__/' --exclude='.venv/' \
  ./ <cluster>:/scratch/<cluster>/<username>/<project>/
```

Preserve trailing-slash semantics: `source/` copies contents; `source` copies the directory. Before using `--delete`, run the same command with `-n` and inspect every deletion.

Fortress commands:

```bash
hsi
htar -cvf <archive>.tar <directory>/
hsi ls
```

Aggregate many small files before tape transfer. Confirm the destination object and, when practical, retrieve or inspect a member before any authorized source cleanup.

## Manage software

Discover rather than invent module versions:

```bash
module list
module avail <name>
module spider <name>
module show <name/version>
command -v <program>
```

Use `module reset` to restore cluster defaults. Use `module purge` or `module --force purge` only when the workflow truly needs a clean module state, then explicitly load the required toolchain.

Put module loads and environment activation in the current session or job script, never in `~/.bashrc` or `~/.bash_profile`. Follow the project's lockfile/environment manager; use `uv` for the statdept profile when the project supports it, otherwise preserve Conda or another declared workflow.

For accelerator work, confirm hardware and software match:

- NVIDIA allocation → compatible CUDA module/framework.
- AMD allocation → compatible ROCm build; CUDA binaries are invalid.

## RCAC-specific command index

| Command | Use |
|---|---|
| `slist` | Discover Slurm accounts, purchased resources, and available QoS. |
| `showpartitions` | Inspect live partition hardware/capacity and queue pressure. |
| `sinteractive` | Request an interactive compute allocation. |
| `jobinfo` | Summarize current or completed job information. |
| `jobenv` / `jobcmd` / `jobscript` | Recover submitted environment, command, or script information. |
| `myquota` | Inspect accessible storage and quotas. |
| `purgelist` | Find Scratch files staged for purge. |
| `flost` | Search supported filesystem snapshots for lost files. |
| `monitor` | Record RCAC CPU/GPU/memory telemetry. |
| `hsi` / `htar` | Access and archive to Fortress HPSS. |

Do not assume these site helpers exist on non-RCAC systems.
