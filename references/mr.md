
# glab mr

Create, view, and manage GitLab merge requests.

## Quick start

```bash
# Create MR from current branch
glab mr create --fill

# List my MRs
glab mr list --assignee=@me

# Review an MR
glab mr checkout 123
glab mr diff
glab mr approve

# Merge an MR
glab mr merge 123 --when-pipeline-succeeds --remove-source-branch
```

## Common workflows

### Creating MRs

**From current branch:**
```bash
glab mr create --fill --label bugfix --assignee @reviewer

# Create now, merge automatically when checks pass
glab mr create --fill --auto-merge

# Start from an MR template file when your project uses one
glab mr create --fill --template .gitlab/merge_request_templates/default.md
```

**From issue:**
```bash
glab mr for 456  # Creates MR linked to issue #456
```

**Draft MR:**
```bash
glab mr create --draft --title "WIP: Feature X"
```

### Review workflow

1. **List pending reviews:**
   ```bash
   glab mr list --reviewer=@me --state=opened
   ```

2. **Checkout and test:**
   ```bash
   glab mr checkout 123
   npm test
   ```

3. **Leave feedback:**
   ```bash
   # Forward command surface for new MR comments/discussions (v1.94.0+)
   glab mr note create 123 -m "Looks good, one question about the cache logic"

   # Reply inside an existing discussion thread
   glab mr note create 123 --reply abc12345 -m "Good catch — updated"

   # Native diff comments on the latest MR version
   glab mr note create 123 --file src/cache.ts --line 42 -m "Please extract this branch"
   glab mr note create 123 --file src/cache.ts --old-line 17 -m "Why was this removed?"

   # List discussion threads on the MR (experimental)
   glab mr note list 123

   # Resolve or reopen a discussion by note/discussion ID (experimental)
   glab mr note resolve 3107030349 123
   glab mr note reopen 3107030349 123
   ```

4. **Approve:**
   ```bash
   glab mr approve 123
   ```

### Merge strategies

**Auto-merge when pipeline passes:**
```bash
glab mr merge 123 --when-pipeline-succeeds --remove-source-branch
```

**Squash commits:**
```bash
glab mr merge 123 --squash
```

**Rebase before merge:**
```bash
glab mr rebase 123
glab mr merge 123
```

## Troubleshooting

**Merge conflicts:**
- Checkout MR: `glab mr checkout 123`
- Resolve conflicts manually in your editor
- Commit resolution: `git add . && git commit`
- Push: `git push`

**Cannot approve MR:**
- Check if you're the author (can't self-approve in most configs)
- Verify permissions: `glab mr approvers 123`
- Ensure MR is not in draft state

**Pipeline required but not running:**
- Check `.gitlab-ci.yml` exists in branch
- Verify CI/CD is enabled for project
- Trigger manually: `glab ci run`

**"MR already exists" error:**
- List existing MRs from branch: `glab mr list --source-branch <branch>`
- Close old MR if obsolete: `glab mr close <id>`
- Or update existing: `glab mr update <id> --title "New title"`

## Related Skills

**Working with issues:**
- See [issue.md](issue.md) for creating/managing issues
- Use `glab mr for <issue-id>` to create MR linked to issue
- Script: `scripts/create-mr-from-issue.sh` automates branch + MR creation

**Pipeline-aware merge:**
- Check pipeline with `glab ci status` or `glab api projects/:id/merge_requests/:iid` before merging
- Use `glab mr create --auto-merge` to request auto-merge up front, or `glab mr merge --when-pipeline-succeeds` on an existing MR

## Native MR note flow (`glab mr note create`)

As of glab v1.94.0, `glab mr note create` is the preferred command surface for posting new MR discussions.

### Use native `glab mr note create` when

```bash
# New top-level discussion/comment
glab mr note create 123 -m "Please add a regression test"

# Reply to an existing discussion thread
glab mr note create 123 --reply abc12345 -m "Fixed in the latest push"

# File-level diff comment
glab mr note create 123 --file src/app.ts -m "General concern on this file"

# Line comment on the new side of the diff
glab mr note create 123 --file src/app.ts --line 84 -m "This branch can return null"

# Range comment on the new side
glab mr note create 123 --file src/app.ts --line 84:96 -m "Consider extracting this block"

# Comment on a removed line from the old side
glab mr note create 123 --file src/app.ts --old-line 37 -m "Why was this guard removed?"
```

Flag rules worth remembering from the upstream help/docs:
- `--reply` targets an existing discussion thread instead of starting a new one.
- `--reply` accepts a full discussion ID or a unique prefix of at least 8 characters.
- `--line` and `--old-line` require `--file` and cannot be used together.
- `--file`, `--reply`, and `--unique` are mutually exclusive.
- Omit both `--line` and `--old-line` when you want a file-level diff comment.

### Use raw `glab api` when

Use a JSON-body `glab api` call when `glab mr note create` cannot anchor a comment (new files, renamed paths, or `line_code` validation errors). See the REST API section below.

## Posting Inline Comments on MR Diffs

### The `glab api --field` Problem

`glab api --field position[new_line]=N` silently falls back to a **general** (non-inline) comment
when GitLab rejects the position data. This happens with:
- Entirely new files (`new_file: true` in the diff)
- Files with complex/encoded paths
- Any nested position field that doesn't survive form encoding

There is no error — GitLab just drops the position and creates a general discussion. You won't know
it failed unless you check the returned note's `position` field.

### The Fix: Always Use JSON Body

Post inline comments via the REST API with a `Content-Type: application/json` body:

```python
import json, urllib.request, urllib.parse, subprocess

# Get token from glab config
token = subprocess.run(
    ["glab", "config", "get", "token", "--host", "gitlab.com"],
    capture_output=True, text=True
).stdout.strip()

project = urllib.parse.quote("mygroup/myproject", safe="")
mr_iid = 42

# Always fetch fresh SHAs — never use cached values
r = urllib.request.urlopen(urllib.request.Request(
    f"https://gitlab.com/api/v4/projects/{project}/merge_requests/{mr_iid}/versions",
    headers={"PRIVATE-TOKEN": token}
))
v = json.loads(r.read())[0]

payload = {
    "body": "Your comment here",
    "position": {
        "base_sha":  v["base_commit_sha"],
        "start_sha": v["start_commit_sha"],
        "head_sha":  v["head_commit_sha"],
        "position_type": "text",
        "new_path": "src/utils/helpers.ts",
        "new_line": 16,
        "old_path": "src/utils/helpers.ts",  # for renamed files, use the diff's actual old_path
        "old_line": None                       # None = added line
    }
}

req = urllib.request.Request(
    f"https://gitlab.com/api/v4/projects/{project}/merge_requests/{mr_iid}/discussions",
    data=json.dumps(payload).encode(),
    headers={"PRIVATE-TOKEN": token, "Content-Type": "application/json"},
    method="POST"
)
with urllib.request.urlopen(req) as resp:
    result = json.loads(resp.read())
    note = result["notes"][0]
    is_inline = note.get("position") is not None  # True = inline, False = fell back to general
    print("inline:", is_inline, "| disc_id:", result["id"])
```

### Finding the Correct Line Number

Line numbers must point to an **added line** (`+` prefix) in the diff — context lines and removed
lines will cause the position to be rejected:

```python
import re

def get_new_line_number(diff_text, keyword):
    """Find the new_file line number of the first added line containing keyword."""
    new_line = 0
    for line in diff_text.split("\n"):
        hunk = re.match(r"@@ -\d+(?:,\d+)? \+(\d+)(?:,\d+)? @@", line)
        if hunk:
            new_line = int(hunk.group(1)) - 1
            continue
        if line.startswith("-") or line.startswith("\\"):
            continue
        new_line += 1
        if line.startswith("+") and keyword in line:
            return new_line
    return None

# Usage
diffs = json.loads(...)  # from /merge_requests/{iid}/diffs
for d in diffs:
    if d["new_path"] == "src/utils/helpers.ts":
        line = get_new_line_number(d["diff"], "safeParse")
        print("line:", line)
```

If GitLab returns a `line_code` validation error, compute the diff anchor from `/merge_requests/{iid}/diffs` and retry with `position[line_range][start/end][line_code]`. Only if that also fails, fall back to a top-level MR note that preserves the finding text.

---

### Filtering discussion threads by resolution

```bash
# Show only unresolved discussion threads on an MR
glab mr view 123 --unresolved

# Show only resolved threads
glab mr view 123 --resolved
```

Useful for quickly checking which review threads still need attention before merging.

## `glab mr list` filtering flags

`glab mr list` supports the following filtering and sorting flags:

```bash
# Filter by author
glab mr list --author <username>

# Filter by source or target branch
glab mr list --source-branch feature/my-branch
glab mr list --target-branch main

# Filter by draft status
glab mr list --draft
glab mr list --not-draft

# Filter by label or exclude label
glab mr list --label bugfix
glab mr list --not-label wip

# Order and sort
glab mr list --order updated_at --sort desc
glab mr list --order merged_at --sort asc

# Date range filtering
glab mr list --created-after 2026-01-01
glab mr list --created-before 2026-03-01

# Search in title/description
glab mr list --search "login fix"

# Full flag reference (all available flags)
glab mr list \
  --assignee @me \
  --author vince \
  --reviewer @me \
  --label bugfix \
  --not-label wip \
  --source-branch feature/x \
  --target-branch main \
  --milestone "v2.0" \
  --draft \
  --state opened \
  --order updated_at \
  --sort desc \
  --search "auth" \
  --created-after 2026-01-01
```

## Structured output

`glab mr approvers` supports `--output json` / `-F json` for structured output, which is useful for agent automation.

```bash
# View MR approvers with JSON output
glab mr approvers 123 --output json
glab mr approvers 123 -F json
```

## Command reference

For complete command documentation and all flags, see [commands/mr-commands.md](commands/mr-commands.md).

**Available commands:**
- `approve` - Approve merge requests
- `checkout` - Check out an MR locally
- `close` - Close merge request
- `create` - Create new MR
- `delete` - Delete merge request
- `diff` - View changes in MR
- `for` - Create MR for an issue
- `list` - List merge requests
- `merge` - Merge/accept MR
- `note` - MR discussion commands; use `glab mr note create` for new comments, plus `list`, `resolve`, and `reopen`
- `rebase` - Rebase source branch
- `reopen` - Reopen merge request
- `revoke` - Revoke approval
- `subscribe` / `unsubscribe` - Manage notifications
- `todo` - Add to-do item
- `update` - Update MR metadata
- `view` - Display MR details
