# GitLab CLI Scripts

Automation scripts for common GitLab CLI workflows. Part of the [gitlab-cli-skills](../SKILL.md) skill — see [references/mr.md](../references/mr.md) for MR workflows and [references/issue.md](../references/issue.md) for issue workflows.

## Available Scripts

### `create-mr-from-issue.sh`

Create branch from issue with proper naming.

```bash
./scripts/create-mr-from-issue.sh <ISSUE_ID> [--create-mr]

# Examples
./scripts/create-mr-from-issue.sh 456
./scripts/create-mr-from-issue.sh 456 --create-mr  # Also creates draft MR
```

**What it does:**
1. Fetches issue title
2. Creates branch named `<issue-id>-<slugified-title>`
3. Optionally creates draft MR linked to the issue

### `sync-fork.sh`

Sync your fork with upstream repository.

```bash
./scripts/sync-fork.sh [branch] [upstream_remote]

# Examples
./scripts/sync-fork.sh                    # Syncs main with upstream
./scripts/sync-fork.sh develop            # Syncs develop with upstream
./scripts/sync-fork.sh main my-upstream   # Custom upstream remote name
```

**What it does:**
1. Fetches from upstream remote
2. Merges upstream changes into local branch
3. Pushes to your fork's origin

## Usage Tips

**Make scripts available globally:**
```bash
export PATH="$PATH:/path/to/gitlab-cli-skills/scripts"
```

**Use with AI assistants:**

An assistant can run these scripts directly when the task matches, for example:
```
"Create a branch for issue 789 and draft MR"
"Sync my fork with upstream"
```

## Requirements

- `glab` CLI installed and authenticated
- `git` for branch/fork operations

## Token efficiency

Scripts support progressive disclosure:
- Scripts execute without loading into context
- Only script output consumes tokens
- Deterministic behavior vs AI-generated code
- Reusable across multiple tasks
