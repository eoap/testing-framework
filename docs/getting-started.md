# Getting started

## Prerequisites

Use Python 3.11 to match this repository's CI environment. Have Docker installed and running for execution of containerised application steps. You also need network access to retrieve the CWL release, referenced input URLs, and container images.

Clone the repository and enter its root directory:

```bash
git clone https://github.com/eoap/testing-framework.git
cd testing-framework
```

## Install the test tools

Create an isolated Python environment:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install cwltest==2.6.20250818005349 cwltool
```

On Windows, activate the environment with `.venv\Scripts\Activate.ps1` in PowerShell instead of the `source` command.

The `cwltest` pin matches the checked-in CI workflow. `cwltool` is currently unpinned, so separate installations can select different runner versions.

Check that the tools and container daemon are available:

```bash
cwltest --help
cwltool --version
docker info
```

## Run one case

Start with the catalog-generation case, which uses an existing mask fixture:

```bash
cwltest --test tests/test-suite.yaml --id stac-generation
```

This still retrieves the remote application definition and referenced STAC item. It is not an offline test.

## Run the complete suite

Use the same command as CI:

```bash
cwltest --test tests/test-suite.yaml --tool cwltool --verbose --
```

Read both execution logs and output-comparison messages. An execution failure means the runner could not complete the application; a comparison failure means its returned result did not match the test expectation. See [troubleshooting](troubleshooting.md) for the next checks.

## Understand input resolution

The suite refers to jobs such as `inputs-03.yaml` beside `test-suite.yaml`. File inputs within those jobs refer to fixtures such as `data/crop_green.tif`. Keep this directory relationship when moving or copying tests.

Some inputs are URL strings rather than CWL `File` objects. The application interprets those strings as STAC item references. Replacing one with a local file requires checking the corresponding CWL input type and the application's handling of that value.

Continue with the [water-bodies example](water-bodies.md) to understand what each case executes.
