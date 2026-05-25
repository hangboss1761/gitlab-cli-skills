
# glab issue

Create, view, update, and manage GitLab issues.

## Quick start

```bash
# Create an issue
glab issue create --title "Fix login bug" --label bug

# List open issues
glab issue list --state opened

# View issue details
glab issue view 123

# Add comment
glab issue note 123 -m "Working on this now"

# Close issue
glab issue close 123
```

## Common workflows

### Bug reporting workflow

1. **Create bug issue:**
   ```bash
   glab issue create \
     --title "Login fails with 500 error" \
     --label bug \
     --label priority::high \
     --assignee @dev-lead
   ```

   If your project keeps reusable issue templates in-repo, `glab` v1.93.0 adds `--template` so you can start from a template file instead of pasting recurring boilerplate:

   ```bash
   glab issue create \
     --title "Login fails with 500 error" \
     --template .gitlab/issue_templates/bug.md \
     --label bug
   ```

2. **Add reproduction steps:**
   ```bash
   glab issue note 456 -m "Steps to reproduce:
   1. Navigate to /login
   2. Enter valid credentials
   3. Click submit
   Expected: Dashboard loads
   Actual: 500 error"
   ```

### Issue triage

1. **List untriaged issues:**
   ```bash
   glab issue list --label needs-triage --state opened
   ```

2. **Update labels and assignee:**
   ```bash
   glab issue update 789 \
     --label backend,priority::medium \
     --assignee @backend-team \
     --milestone "Sprint 23"
   ```

3. **Remove triage label:**
   ```bash
   glab issue update 789 --unlabel needs-triage
   ```

**Batch labeling:**

```bash
for id in 100 101 102; do glab issue update "$id" --label "priority::high"; done
```

### Sprint planning

**View current sprint issues:**
```bash
glab issue list --milestone "Sprint 23" --assignee @me
```

**Add to sprint:**
```bash
glab issue update 456 --milestone "Sprint 23"
```

**Board view:**
```bash
glab issue board view
```

### Linking issues to work

**Create MR for issue:**
```bash
glab mr for 456  # Creates MR that closes issue #456
```

**Automated workflow (create branch + draft MR):**
```bash
scripts/create-mr-from-issue.sh 456 --create-mr
```

This automatically: creates branch from issue title → empty commit → pushes → creates draft MR.

**Close via commit/MR:**
```bash
git commit -m "Fix login bug

Closes #456"
```

## Related Skills

**Creating MRs from issues:**
- See [mr.md](mr.md) for merge request operations
- Use `glab mr for <issue-id>` to create MR that closes issue
- Script: `scripts/create-mr-from-issue.sh` automates branch creation + draft MR

**Bulk labeling:**
- Use `glab issue update` in a loop or `glab api` for custom label workflows

## Command reference

For complete command documentation and all flags, see [commands/issue-commands.md](commands/issue-commands.md).

**Available commands:**
- `create` - Create new issue
- `list` - List issues with filters
- `view` - Display issue details
- `note` - Add comment to issue
- `update` - Update title, labels, assignees, milestone
- `close` - Close issue
- `reopen` - Reopen closed issue
- `delete` - Delete issue
- `subscribe` / `unsubscribe` - Manage notifications
- `board` - Work with issue boards
