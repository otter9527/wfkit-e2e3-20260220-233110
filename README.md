# demo-fullflow3-20260220-233110

Independent AI workflow project generated from `python-mvp` template.

## Initial Requirement
- full workflow e2e verification v3

## Capabilities
- GitHub strict gatekeeping on `main`
- script-driven task dispatch
- worker PR submission and post-merge auto-progression

## Role-Based Scripts
All role entrypoints are under `scripts/roles/`.
Default assumption: each role runs on a different machine/network. Coordination goes through GitHub only.

### Shared (all roles, per machine)
Prepare local workspace from GitHub:
```bash
bash scripts/roles/shared/00_prepare_workspace.sh \
  --repo <owner/name> \
  --workspace-root "$HOME/ai-factory-workspaces" \
  --branch main \
  --install-deps true
```

### A. Owner/Admin
```bash
bash scripts/roles/owner/01_setup_repo.sh \
  --repo <owner/name> \
  --visibility public \
  --default-branch main \
  --strict-mode true
```

### B. PM
Board snapshot:
```bash
bash scripts/roles/pm/01_board.sh --repo <owner/name>
```

Create task issue:
```bash
bash scripts/roles/pm/02_create_task.sh \
  --repo <owner/name> \
  --task-id TASK-001 \
  --task-type IMPL \
  --title "Task 001: Implement add" \
  --acceptance "add returns correct result"
```

Dispatch ready tasks:
```bash
bash scripts/roles/pm/03_dispatch.sh --repo <owner/name> --event manual_dispatch --assign-self false
```

After merge, close task and unlock dependents:
```bash
bash scripts/roles/pm/06_post_merge.sh --repo <owner/name> --pr <pr_number>
```

### C. Worker
Check own inbox:
```bash
bash scripts/roles/worker/03_inbox.sh --repo <owner/name> --worker worker-a --status in_progress
```

```bash
bash scripts/roles/worker/04_run_task.sh \
  --repo <owner/name> \
  --issue <issue_number> \
  --worker worker-a \
  --ai-mode codex
```

### D. Reviewer
Review queue:
```bash
bash scripts/roles/reviewer/04_queue.sh --repo <owner/name>
```

```bash
bash scripts/roles/reviewer/05_merge_pr.sh \
  --repo <owner/name> \
  --pr <pr_number> \
  --merge-method squash \
  --wait-checks true
```

### E. Release/QA
```bash
bash scripts/roles/release/07_collect_report.sh --repo <owner/name>
```

## Notes
- Detailed role workflow: `docs/ROLE-WORKFLOW.md`.
- `--ai-mode` supports `mock|real|codex`.
- `--phase3-ai-mode` supports `mock|real|codex|skip`.
- Optional env: `CODEX_MODEL`.
