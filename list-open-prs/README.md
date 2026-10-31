# list-open-prs

List the open PRs of the calling repository as a JSON object that can be used
directly as a job matrix with `fromJSON`:

```json
{"include": [{"pr": 123, "head_sha": "0123456789abcdef0123456789abcdef01234567"}, ...]}
```

Each entry has the PR number as `pr` and the SHA of the PR head commit as
`head_sha`. When there are no open PRs, `include` is an empty array. At most
100 open PRs are listed. Any API error fails the step.

## Inputs

| Input | Default | Description |
|---|---|---|
| `token` | `github.token` | Token with `pull-requests: read` permission on the repository. |

## Outputs

| Output | Description |
|---|---|
| `json` | JSON object with open PRs, as `{"include": [{"pr", "head_sha"}, ...]}` |

## Permissions

The job needs:

```yaml
permissions:
  pull-requests: read
```

## Example

Run a job for every open PR:

```yaml
list-open-prs:
  runs-on: ubuntu-latest
  permissions:
    pull-requests: read
  outputs:
    json: ${{ steps.list-open-prs.outputs.json }}
  steps:
    - uses: eic/actions/list-open-prs@main
      id: list-open-prs

per-pr:
  needs: list-open-prs
  if: fromJSON(needs.list-open-prs.outputs.json).include[0] != null
  strategy:
    matrix: ${{ fromJSON(needs.list-open-prs.outputs.json) }}
  runs-on: ubuntu-latest
  steps:
    - run: echo "PR ${{ matrix.pr }} at ${{ matrix.head_sha }}"
```

A matrix with an empty `include` is an error, hence the `if:` condition, which
skips the job when there are no open PRs.

## Reusable workflow

The same logic is available as the reusable workflow
[`.github/workflows/list-open-prs.yml`](../.github/workflows/list-open-prs.yml),
for existing callers that use it as a job rather than a step. It has no
inputs, uses `github.token`, and has the same `json` output:

```yaml
list-open-prs:
  permissions:
    pull-requests: read
  uses: eic/actions/.github/workflows/list-open-prs.yml@main

per-pr:
  needs: list-open-prs
  if: fromJSON(needs.list-open-prs.outputs.json).include[0] != null
  strategy:
    matrix: ${{ fromJSON(needs.list-open-prs.outputs.json) }}
  runs-on: ubuntu-latest
  steps:
    - run: echo "PR ${{ matrix.pr }} at ${{ matrix.head_sha }}"
```
