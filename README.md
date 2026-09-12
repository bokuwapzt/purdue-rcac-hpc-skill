# purdue-rcac-hpc-skill

Personal, unofficial ChatGPT and Codex skill for Purdue RCAC HPC and Slurm workflows.

This skill helps an AI agent choose RCAC resources, prepare and inspect Slurm jobs, work with RCAC storage and software environments, transfer data, diagnose failures, and handle long-running jobs safely.

> [!IMPORTANT]
> This project is not affiliated with or endorsed by Purdue University or the Rosen Center for Advanced Computing (RCAC). Cluster hardware, partitions, access policies, quotas, purge windows, and software modules can change. Verify consequential values with current RCAC documentation and live commands such as `slist`, `showpartitions`, and `myquota`.

## Knowledge sources

The skill was synthesized and adapted from:

- [Jake-Verburgt/rcac-hpc-exchange](https://github.com/Jake-Verburgt/rcac-hpc-exchange) — Purdue RCAC HPC Exchange workshop material covering Unix fundamentals, cluster access, storage, software modules, Slurm, monitoring, job arrays, and multi-node workflows.
- [talhz/RCAC_HPC](https://github.com/talhz/RCAC_HPC) — an RCAC-focused AI skill and workflow reference, including Bell/Gilbreth, Slurm submission, GPU selection, `statdept` constraints, notifications, and long-job continuation patterns.

The material has been rewritten as agent instructions rather than copied as a tutorial. Generic knowledge that an AI agent already has is omitted; RCAC-specific commands, decision rules, safety constraints, and reusable job patterns are retained.

## Repository structure

```text
.agents/skills/purdue-rcac-hpc/
├── SKILL.md
└── references/
    ├── rcac-operations.md
    ├── slurm-workflow.md
    ├── statdept.md
    └── long-jobs.md
```

- `SKILL.md` defines when the skill should activate and the workflow the agent must follow.
- `references/` contains details loaded only when relevant to the current task.
- `statdept.md` is a personal account profile. Review or replace it if your RCAC account, QoS, email policy, or accessible partitions differ.

## Installation

### Use inside this repository

Clone the repository and start Codex from its root. Codex discovers repository-scoped skills under `.agents/skills`.

```bash
git clone https://github.com/<your-github-username>/purdue-rcac-hpc-skill.git
cd purdue-rcac-hpc-skill
```

### Install as a personal Codex skill

Copy the skill folder into your personal skills directory:

```bash
mkdir -p ~/.agents/skills
cp -R .agents/skills/purdue-rcac-hpc ~/.agents/skills/
```

On PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills"
Copy-Item -Recurse ".agents\skills\purdue-rcac-hpc" "$HOME\.agents\skills\"
```

Restart Codex if the skill does not appear immediately.

### Use with ChatGPT

After the skill has been added to ChatGPT and appears in the Skills interface, type `@` and select `purdue-rcac-hpc`. In Codex CLI or the IDE extension, type `$purdue-rcac-hpc` or use `/skills`.

See the [official OpenAI skill documentation](https://learn.chatgpt.com/docs/build-skills) for supported locations and invocation behavior.

## Usage

Explicit invocation:

```text
$purdue-rcac-hpc Write a Slurm script to run train.py on one Gilbreth GPU.
```

```text
$purdue-rcac-hpc Diagnose why job 123456 is pending and tell me what evidence to collect.
```

```text
$purdue-rcac-hpc Plan a safe rsync workflow from my laptop to RCAC Scratch and copy results back to Depot.
```

The skill also supports implicit invocation when the request clearly mentions Purdue RCAC, a supported cluster, RCAC-specific storage, or Slurm commands covered by the skill.

## What happens when invoked

1. The agent reads `SKILL.md` and identifies the type of RCAC task.
2. It loads only the relevant reference files.
3. It verifies dynamic cluster facts when access is available, or clearly marks placeholders and assumptions.
4. It produces the smallest useful command sequence, job script, diagnosis, or transfer plan.
5. It does not submit jobs, cancel jobs, delete data, overwrite files, or start remote transfers unless the user explicitly requests that action.
6. After an authorized submission, it captures the job ID and verifies the effective Slurm settings.

## Personal configuration

The included `statdept` profile assumes:

- Slurm account `statdept`
- `standby` QoS
- a recorded four-hour standby limit
- `END,FAIL` email notifications for every submitted job and continuation
- Gilbreth for NVIDIA/CUDA workflows and Bell for CPU workflows, subject to live availability

Before using this skill with another account, edit or replace `.agents/skills/purdue-rcac-hpc/references/statdept.md` and update the main skill description if necessary. Never rely on the bundled profile as proof of current RCAC access.
