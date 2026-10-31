# get-artifacts-from-open-prs

Download artifacts from the latest completed run of the calling workflow for
every open PR, laid out as

```
<path>/<pr number>/<subdirectory>/...
```

This is typically used to publish per-PR documentation or reports to GitHub
Pages next to those from the default branch.

For each open PR, the newest completed run of the workflow for the PR head
commit is used. For the PR that triggered the current run (if any), the
artifacts are taken from the current run instead, so the calling job must
depend on the jobs that upload them. By default these artifacts are required
to exist (see `if-no-artifact-found`). For all other PRs, a missing run or
artifact is skipped. Any API or download error fails the step, so that PRs are
not silently dropped from the published pages.

## Inputs

| Input | Default | Description |
|---|---|---|
| `artifacts` | (required) | Newline-separated list of artifacts, each as `<artifact-name>[:<subdirectory>]`. Empty lines and lines starting with `#` are ignored. |
| `path` | `pr` | Directory under which the per-PR directories are created. |
| `workflow` | calling workflow | File name of the workflow whose runs to download artifacts from, e.g. `linux-eic-shell.yml`. |
| `if-no-artifact-found` | `fail` | What to do if an artifact is missing from the current run, for the PR that triggered it: `fail`, `warn` or `ignore`. Use `warn` or `ignore` when the calling job also runs after the jobs that upload the artifacts failed. Missing artifacts of other PRs are always ignored. |
| `allow-forks` | `false` | Also download artifacts from runs for PRs from forks. Publishing the content of such artifacts, e.g. to GitHub Pages, may be unsafe. |
| `token` | `github.token` | Token with `contents: read`, `pull-requests: read` and `actions: read` permissions on the repository. |

## Permissions

The job needs:

```yaml
permissions:
  contents: read
  pull-requests: read
  actions: read
```

## Examples

### Documentation and capybara reports (eic/EICrecon)

```yaml
get-docs-from-open-prs:
  runs-on: ubuntu-latest
  needs: build-docs
  permissions:
    contents: read
    pull-requests: read
    actions: read
  steps:
    - uses: eic/actions/get-artifacts-from-open-prs@main
      with:
        artifacts: |
          docs
          capybara:capybara
        path: publishing_docs/pr
    - run: rm -rf publishing_docs/pr/*/doxygen
    - uses: actions/upload-artifact@v7
      with:
        name: github-pages-staging-prs
        path: publishing_docs/
        if-no-files-found: ignore
```

### Several reports with post-processing (eic/epic)

```yaml
    - uses: eic/actions/get-artifacts-from-open-prs@main
      with:
        artifacts: |
          capybara-report:capybara
          material_map:material-map
        path: publishing_docs/pr
    - name: Generate material-map validation index
      run: |
        for VALDIR in publishing_docs/pr/*/material-map/Validation; do
          ...
        done
```

### Publishing when the current run partially failed (eic/containers)

When the job runs with `if: always() && !cancelled()`, the artifacts of the
current run may be missing. Warn instead of failing, so that the other PRs
are still published:

```yaml
get-docs-from-open-prs:
  runs-on: ubuntu-latest
  needs: merge-capybara
  if: always() && !cancelled()
  permissions:
    contents: read
    pull-requests: read
    actions: read
  steps:
    - uses: eic/actions/get-artifacts-from-open-prs@main
      with:
        artifacts: capybara-report:capybara
        path: publishing_docs/pr
        if-no-artifact-found: warn
```
