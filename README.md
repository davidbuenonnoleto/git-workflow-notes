# git-workflow-notes

Notes and scripts for driving branch → PR → merge cycles from the terminal with
[`gh`](https://cli.github.com/), without touching the GitHub web UI.

## The basic loop

```bash
git switch -c my-change
# ...edit...
git add -A && git commit -m "My change"
git push -u origin my-change

gh pr create --title "My change" --body "What and why." --base main --head my-change
gh pr merge my-change --merge --delete-branch

git switch main && git pull --ff-only
```

`gh pr merge` takes `--merge`, `--squash` or `--rebase`. `--delete-branch` cleans
up both the remote branch and the local one you're tracking it with.

## Co-authored commits

GitHub attributes a commit to multiple people via a `Co-authored-by:` trailer.
It must be in the commit *body*, after a blank line, and the email must belong to
a real GitHub account (the `ID+username@users.noreply.github.com` form works and
is already public on that person's commits):

```bash
git commit -m "Pair-authored change" -m "" -m "Co-authored-by: Some Dev <12345+somedev@users.noreply.github.com>"
```

Repeated `-m` flags are the easiest way to get the blank line right — the trailer
is silently ignored if it ends up on the subject line.

## Useful queries

```bash
gh pr list --state merged --json number,title,mergedAt,reviewDecision
gh issue list --state closed --json number,title,createdAt,closedAt
gh repo view --json name,description,url
```

`--json` plus `--jq` is generally faster and more scriptable than parsing the
default human-readable output.

## Scripts

- [`scripts/pr-flow.sh`](scripts/pr-flow.sh) — creates N branches, opens a PR for
  each, and merges them. Useful for exercising branch protection rules, CI
  triggers, or webhook handlers against a real repo.
