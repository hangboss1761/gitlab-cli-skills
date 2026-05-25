# GitLab CLI Skills

A **single** GitLab CLI (`glab`) skill pack for AI agents: authentication, tokens, merge requests, issues, repositories, configuration, and the REST API.

## Structure

```
gitlab-cli-skills/
├── SKILL.md              # Single entry point: routing table + core workflows
├── references/           # Domain docs (read on demand)
│   ├── auth.md
│   ├── token.md
│   ├── mr.md
│   ├── issue.md
│   ├── repo.md
│   ├── config.md
│   ├── api.md
│   └── commands/         # Full --help reference per subcommand
└── scripts/              # Automation scripts
```

## Coverage

| Domain | Reference |
|--------|-----------|
| Authentication | [references/auth.md](references/auth.md) |
| Token | [references/token.md](references/token.md) |
| Merge requests | [references/mr.md](references/mr.md) |
| Issues | [references/issue.md](references/issue.md) |
| Repository | [references/repo.md](references/repo.md) |
| Configuration | [references/config.md](references/config.md) |
| REST API | [references/api.md](references/api.md) |

## Installation

```bash
npx skills add hangboss1761/gitlab-cli-skills
```

## Prerequisites

```bash
brew install glab
glab auth login --hostname gitlab.com
```

## License

MIT
