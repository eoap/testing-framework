# Troubleshooting

Start with the first failing case rather than the final suite summary. Run it independently with detailed output:

```bash
cwltest --test tests/test-suite.yaml --id crop-green --tool cwltool --verbose --
```

Replace `crop-green` with the failing test ID.

## Tools cannot be found

Activate the virtual environment used for installation, then check:

```bash
python -m pip show cwltest cwltool
cwltest --help
cwltool --version
```

If installation and execution use different environments, reinstall the tools in the active environment using the [getting-started commands](getting-started.md).

## Container startup fails

Check `docker info` in the same environment that launches the tests. Inspect the log for daemon connectivity, permission, image-download, or platform errors. Confirm that the application's container image can be accessed from the runner.

A successful Python dependency installation does not establish that containers can execute.

## Remote CWL or data cannot be retrieved

Identify the exact failed URL in the log. Check network access, proxy settings, release availability, and whether the referenced path exists.

The suite fetches the application from a release URL. Crop, catalog, and workflow inputs reference a STAC item on `main`, whose assets also point to remote rasters. Changing a local fixture does not automatically change those remote references.

## A local input file is missing

Check the job's `location` values and preserve the relationship between the suite, job YAML, and `data/` directory. For example, `inputs-03.yaml` references `data/crop_green.tif` and `data/crop_nir.tif` beside the job's directory.

Confirm the files are present in your checkout before investigating application code.

## Input validation fails

Compare the job with the selected CWL process. Common causes include a misspelled input name, a scalar supplied where an array is expected, a URL string supplied where a `File` is expected, or selection of the wrong process fragment.

The crop cases select `#crop`; the full workflow selects `#water-bodies`. Their input names and output ports differ.

## Execution succeeds but comparison fails

Read which output field differs. A wrong output name or class suggests a changed interface. A size or checksum mismatch suggests changed artifact bytes.

The catalog-generation and end-to-end cases check the catalog, item, and mask. Follow the [baseline review process](reproducibility.md) before replacing expected hashes. Check whether the same mismatch occurs in both cases: shared failures can point to catalog serialization, fixtures, or a common environment dependency.

If unit cases pass and the full workflow fails, examine the connections between steps and the differences between saved intermediate fixtures and newly generated outputs.

## Local results differ from CI

Compare Python, `cwltest`, `cwltool`, container runtime, application release, and fixture references. CI currently pins only the test framework among the Python test dependencies.

Record the environment and the first meaningful error when reporting a failure. Include the test ID, command, relevant log excerpt, and whether the issue reproduces with a single case.

## The documentation build fails

Install `mkdocs-material` and `mkdocs-mermaid2-plugin` in the active environment. Run `mkdocs build --strict` from the repository root and inspect the first warning or error.

For a new page, check its navigation path in `mkdocs.yml`, relative links, and fenced code blocks. A missing Python package is an environment issue; a missing referenced page or asset requires correcting the documentation or site configuration.
