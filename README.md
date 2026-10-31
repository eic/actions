# eic/actions

Shared GitHub Actions and reusable workflows for EIC repositories, such as
[eic/EICrecon](https://github.com/eic/EICrecon),
[eic/epic](https://github.com/eic/epic) and
[eic/containers](https://github.com/eic/containers).

## Actions

| Action | Description |
|---|---|
| [`get-artifacts-from-open-prs`](get-artifacts-from-open-prs/README.md) | Download artifacts from the latest completed workflow run of every open PR |

## Reusable workflows

| Workflow | Description |
|---|---|
| [`list-open-prs`](.github/workflows/list-open-prs.yml) | List open PRs as a job matrix, `{include: [{pr, head_sha}, ...]}` |

## Versioning

This repository does not publish tags. Refer to actions and workflows either
by `@main`, or by a commit SHA so that changes here are reviewed before they
run in your pipeline:

```yaml
- uses: eic/actions/get-artifacts-from-open-prs@<sha> # main as of YYYY-MM-DD
```
