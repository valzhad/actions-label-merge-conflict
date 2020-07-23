# label merge-conflict action

This action adds a given label to Pull Requests that have merge conflicts and removes a given label from these pull requests

![label lifecycle: open (no label), push to main -> merge conflict -> label: PR needs rebase -> resolve conflicts on PR -> remove label: PR needs rebase](https://raw.githubusercontent.com/eps1lon/actions-label-merge-conflict/main/label-lifecycle.png).

## Why?

PRs are usually open for a few days until a maintainer can take a look. When this happens a Pull Request (PR) might already be outdated without the author being notified of this. A maintainer either has to resolve them (which takes time) or has to ping the author. This creates a feedback loop that can be reduced by this action.

This actions achieve this with minimal noise (no comment bloat) by adding a label to a PR if it has merge conflicts. This means the author has a single overview of which PRs need a rebase by using their list of created PRs. At the same time the maintainer has a simple filter for PRs that have merge conflicts.

### Motivation

- [isaacs/github#224: Notify when merge conflict arises](https://github.com/isaacs/github/issues/224)

## Inputs

### `dirtyLabel`

**Required** The name of the label that should be added once a PR has merge conflicts.

### `repoToken`

**Required** Token for the repository. Can be passed in using {{ secrets.GITHUB_TOKEN }}

### `removeOnDirtyLabel`

The name of the label that should be removed once a PR has merge conflicts.

**Default**: No label is removed if a PR is marked as dirty.

### `retryAfter`

Number of seconds after which the mergable state is checked again if it is unknown (GitHub is still calculating it).

**Default**: 120

### `retryMax`

Number of times the script will check the mergable state aigain. After that it will print a warning.

**Default**: 5

## Example usage

```yaml
name: "Maintenance"
on:
  # So that PRs touching the same files as the push are updated
  push:
  # So that the `dirtyLabel` is removed if conflicts are resolved
  # WARNING: PRs from forks don't have access to screts.
  # You might want to skip this action on pull_requests which means
  # the label might not reflect the current state of the PR until
  # another push on `main`
  pull_request:
    types: [synchronize]

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - name: check if prs are dirty
        uses: eps1lon/actions-label-merge-conflict@releases/1.x
        with:
          dirtyLabel: "PR: needs rebase"
          removeOnDirtyLabel: "PR: ready to ship"
          repoToken: "${{ secrets.GITHUB_TOKEN }}"
```
