---
name: gitlab-cli-skills
description: GitLab CLI (glab) workflows for authentication, tokens, merge requests, issues, repositories, configuration, and REST API. Use when user mentions GitLab CLI, glab, login, auth, PAT, access token, MR, merge request, issue, repo, clone, fork, or GitLab terminal automation. Triggers on glab, GitLab CLI, authentication, credentials, token, MR, issue, repository.
---

# GitLab CLI Skills

Single skill for GitLab CLI (`glab`): auth, tokens, MR, issue, repo, config, and API. Detailed docs live in `references/` — read only what the task needs.

## Prerequisites

- Install [glab](https://gitlab.com/gitlab-org/cli/-/releases) (`brew install glab` on macOS)
- Authenticate: `glab auth login` (token stored in `~/.config/glab-cli/config.yml`)
- Review script contents before running them in automated workflows

## Quick start

```bash
# First time setup
glab auth login
glab auth status

# Common operations
glab mr create --fill              # Create MR from current branch
glab issue create                  # Create issue
glab repo view --web               # Open repo in browser
glab config set editor vim         # Set CLI defaults
```

## Reference routing

| Task | Read first |
|------|------------|
| Login, logout, multi-account, Docker registry auth | [references/auth.md](references/auth.md) |
| Create, list, revoke, rotate PAT / project tokens | [references/token.md](references/token.md) |
| MR create, review, approve, merge, inline comments | [references/mr.md](references/mr.md) |
| Issue create, list, update, close, comment | [references/issue.md](references/issue.md) |
| Clone, fork, create repo, members | [references/repo.md](references/repo.md) |
| CLI defaults, `glab config` get/set | [references/config.md](references/config.md) |
| Custom REST/GraphQL when no glab subcommand | [references/api.md](references/api.md) |
| Full subcommand flags (`--help` dumps) | [references/commands/](references/commands/) |

## Automation scripts

| Script | Purpose |
|--------|---------|
| [scripts/create-mr-from-issue.sh](scripts/create-mr-from-issue.sh) | Branch from issue title; optional draft MR |
| [scripts/sync-fork.sh](scripts/sync-fork.sh) | Sync fork with upstream |

See [scripts/README.md](scripts/README.md) for usage examples.

## Multi-account identity

When automation must post as different GitLab users, use separate bot/service accounts (one PAT per account). Multiple tokens on the same GitLab user still appear as that same user.

Pick the intended account **before the first GitLab write**. See [references/auth.md](references/auth.md) for env-file layout and wrong-identity recovery.

```bash
unset GITLAB_TOKEN GITLAB_ACCESS_TOKEN OAUTH_TOKEN GITLAB_HOST
set -a
source ~/.config/gitlab-cli/env/<account>.env
set +a
```

### Required pre-flight before any GitLab write

```bash
glab auth status --hostname "$GITLAB_HOST"
glab api --hostname "$GITLAB_HOST" user
```

Do not write until both commands show the intended actor on the target host. See [references/auth.md](references/auth.md) for wrong-identity remediation.

## When to use glab vs web UI

**Use glab:** automation, terminal workflows, batch MR/issue ops, script integration.

**Use web UI:** complex inline diff review, visual conflict resolution, group/instance settings.

## Common workflows

### Daily development

```bash
glab issue view 123
git checkout -b 123-feature-name
glab mr create --fill --draft
glab mr update --ready
glab mr merge --when-pipeline-succeeds --remove-source-branch
```

### Code review

```bash
glab mr list --reviewer=@me --state=opened
glab mr checkout 456
glab mr diff
glab mr approve 456
```

### Auth and tokens

```bash
glab auth login --hostname gitlab.com --web
glab token create my-automation-token --scopes api,read_repository
```

See [references/auth.md](references/auth.md) and [references/token.md](references/token.md).

### Repository setup

```bash
glab repo clone group/project
glab repo fork group/project --clone
./scripts/sync-fork.sh
```

## Decision trees

### MR or issue first?

```
Need to track work?
├─ Yes → glab issue create → glab mr for <issue-id>
└─ No → glab mr create --fill
```

### Clone or fork?

```
Have write access? → glab repo clone
Contributing without write access? → glab repo fork --clone → ./scripts/sync-fork.sh
```

### glab subcommand or REST API?

```
glab has subcommand (mr/issue/repo/auth/config)? → use it
Otherwise → glab api <endpoint>  (see references/api.md)
```

## Progressive disclosure

Keep this file for routing and high-frequency flows. For long workflows (MR inline notes, auth troubleshooting, full flag lists), open the linked `references/*.md` file — do not load all references at once.
