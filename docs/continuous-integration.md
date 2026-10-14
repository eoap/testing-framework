# Continuous integration and documentation

## Test automation

`.github/workflows/run-tests.yaml` executes the suite on pushes and pull requests. It checks out the repository, prepares Python 3.11 on Ubuntu, installs the test dependencies, and runs:

```bash
cwltest --test tests/test-suite.yaml --tool cwltool --verbose --
```

The dependency installation pins `cwltest==2.6.20250818005349` and leaves `cwltool` unpinned. Use the same versions and invocation locally when investigating CI failures.

The existing workflow is the source of truth for this repository's CI configuration. When adapting it to another EOAP, update the suite path and ensure the runner has access to required containers and data.

## Make failures reviewable

Review the failed case's execution log before changing expected outputs. Distinguish dependency-download failures, application failures, and assertion mismatches.

If more diagnostics are needed, consider retaining generated artifacts and logs with a CI artifact-upload step. The current workflow does not include such a step. Keep retained data small and avoid including credentials or private input data.

Changes to remote fixtures can affect all cases consuming those URLs, even when their YAML files are unchanged. Include fixture changes in regression review and use the [baseline review process](reproducibility.md).

## Preview this guide

From the repository root, with your Python environment active:

```bash
python -m pip install mkdocs-material mkdocs-mermaid2-plugin
mkdocs serve
```

Open the local URL printed by MkDocs. Navigation is configured in `mkdocs.yml`; page content is stored under `docs/`.

Build the static site without deploying it:

```bash
mkdocs build --strict
```

Strict mode treats warnings as build failures, helping catch broken documentation references before publication.

## Publication

`.github/workflows/docs.yaml` builds and publishes the site with `mkdocs gh-deploy --force`. It runs on pushes to `main` or `develop` when changes affect `docs/**`, `mkdocs.yml`, or `.github/**`.

Both configured branches can trigger publication. Review branch and path filters when changing the release process. Local `mkdocs serve` and `mkdocs build` preview or build the site without publishing it.

## Maintain the guide

Add new pages to the navigation, use relative links between documentation pages, and keep examples aligned with the checked-in suite. Verify commands and assertions when changing dependency versions or the referenced CWL release.
