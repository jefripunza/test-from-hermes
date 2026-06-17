---
name: git-pro
description: "Git expert: Git Flow, rebase, squash, cherry-pick, monorepo workflows, release management. Conventional commits."
metadata:
  hermes:
    tags: [git, version-control, branching, release, workflow]
---

# Git Pro — Version Control Expert

Expert in Git workflows, branching strategies, and release management.

## Expertise

- Git Flow / Trunk-based development
- Rebase (interactive, onto)
- Squash (merge vs rebase strategies)
- Cherry-pick
- Monorepo workflows
- Release management (tags, changelogs, semver)
- Submodules
- `git bisect` for debugging
- Merge conflict resolution

## Commit Standard (Conventional Commits)

```text
feat: add user authentication
fix: resolve race condition in login handler
refactor: extract validation logic
test: add e2e test for payment flow
docs: update API documentation
chore: bump dependencies
perf: optimize database query
ci: add deployment workflow
build: update Vite config
```

## Branching Strategy

```text
main        — production-ready, protected
develop     — integration branch
feat/*      — feature branches (branch from develop)
fix/*       — bug fixes
hotfix/*    — urgent production fixes (branch from main)
release/*   — release candidates (branch from develop)
```

## Best Practices

- **Rebase** feature branches before merge — clean linear history
- **Squash** commits on merge to main — one commit per feature
- **Atomic commits** — each commit is a single logical change
- **Descriptive messages** — subject ≤50 chars, body explains why
- **Sign commits** — `git commit -S` with GPG/SSH
- **Never force push** to shared branches (main, develop)

## Common Workflows

```bash
# Start feature
git checkout -b feat/user-auth develop

# Keep up to date
git fetch origin
git rebase origin/develop

# Interactive rebase (clean up before PR)
git rebase -i origin/develop

# Create PR → review → merge (squash)
git checkout develop
git merge --squash feat/user-auth
git commit -m "feat: add user authentication"
git branch -D feat/user-auth
```

## Release Workflow

```bash
git checkout -b release/v1.2.0 develop
# Bump version, update changelog
git commit -m "chore: bump to v1.2.0"
git checkout main
git merge release/v1.2.0 --no-ff
git tag -a v1.2.0 -m "v1.2.0"
git push origin main --tags
git checkout develop
git merge release/v1.2.0
git branch -d release/v1.2.0
```
