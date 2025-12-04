# Testing EO Application Packages with cwltest

This document provides guidance for EOAP developers on how to write, organise, and execute tests for EO Application Packages using the CWL testing framework (cwltest).

It includes examples, recommended patterns, and practices aligned with EOAP quality and maturity requirements.

## Why Testing Matters in EOAP

EO Application Packages (EOAPs) are intended to be:

- portable
- reproducible
- shareable across computing platforms (local, cloud, HPC, Kubernetes)
- deployable as OGC API Processes

To reach these goals, each application package should include unit tests, integration tests, and ideally end-to-end workflow tests.

Using cwltest ensures that:

- each CWL tool behaves as expected
- the application package is deterministic when required
- refactoring does not introduce regressions
- the package remains stable across environments (e.g., cwltool, calrissian)

EOAP maturity levels can be verified before publication or deployment

## What is cwltest

`cwltest` is the official testing framework for the Common Workflow Language (CWL).

It allows validating:

- individual CWL CommandLineTool steps
- full Workflow executions
- structured outputs (files, directories, metadata)
- deterministic checksums and sizes
- expected failures (should_fail)

Tests are defined declaratively in YAML.

## Recommended Directory Structure

Place test files inside a dedicated directory within your EOAP repository:

```
tests/
    test-suite.yaml          # The complete test suite
    inputs-01.yaml           # Job inputs for crop (green)
    inputs-02.yaml           # Job inputs for crop (nir)
    inputs-03.yaml           # Job inputs for NDWI
    inputs-04.yaml           # Job inputs for Otsu
    inputs-05.yaml           # Job inputs for STAC
    inputs-e2e.yaml          # Job inputs for full workflow test
```

This pattern can be reused for any EOAP application package.

## Writing Test Cases

Each test is an entry in a YAML list with the following fields:

- id: unique-test-id
  doc: "Human-readable description"
  tool: path-or-URL-to-CWL-tool-or-workflow
  job: path-to-job-inputs.yaml
  output:
    output_name:
      class: File or Directory
      location: Any or deterministic

## Recommended naming conventions

IDs should reflect the step:

`crop-green`, `normalized-difference`, `otsu-threshold`

Full workflow tests should end with:

`workflow-end-to-end`

## Examples

### Example Unit Test (Flexible Output)

This test validates a step (crop) that may produce large raster outputs where strict hashing is unnecessary.

```yaml
- id: crop-green
  doc: "Unit test for the 'crop' step using the Green band input."
  tool: app.cwl#crop
  job: inputs-01.yaml
  output:
    cropped:
      class: File
      location: Any
```

### Example Deterministic Test (Strict Output)

When a test must enforce full reproducibility—such as the generation of STAC metadata—the output can include checksums and sizes:

```yaml
- id: stac-generation
  doc: "Unit test verifying deterministic STAC Catalog creation."
  tool: app.cwl#stac
  job: inputs-05.yaml
  output:
    stac_catalog:
      class: Directory
      listing:
        - class: File
          location: catalog.json
          checksum: "sha1$70e96cddbb205363b4ee9cb763e438a29fd29cdc"
          size: 363
```

This ensures reproducibility across:

- workstations
- Docker-based runners
- calrissian on Kubernetes
- future versions of the container image

### Example Integration Test (End-to-End)

EOAPs require full workflow validation:

```yaml
- id: workflow-end-to-end
  doc: "Integration test running the entire water-bodies workflow end-to-end."
  tool: app.cwl
  job: inputs-e2e.yaml
  output:
    stac_catalog:
      class: Directory
      location: Any
```

This test confirms that:

- all steps connect correctly
- output structure follows EOAP conventions
- the application can run as a service in an automated environment

## Generating Checksums (For Deterministic Tests)

To produce a SHA-1 checksum for any deterministic output:

`sha1sum output-file.tif`

or for a directory listing

`find directory -type f -exec sha1sum {} \;`

Use the result inside your test:

```yaml
checksum: "sha1$<value>"
```

## Running the Test Suite

Install the required tools:

`pip install cwltool cwltest`

Run the suite:

```
cwltest --test tests/water-bodies/test-suite.yaml
```

Run only one test:

```
cwltest --test tests/water-bodies/test-suite.yaml --id stac-generation
```

## Adding Tests to GitHub Actions CI

A minimal workflow:

```yaml
name: cwltest

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install cwltool cwltest
      - run: cwltest --test tests/test-suite.yaml
```

This ensures EOAP packages remain validated continuously.

## Best Practices Summary

- Use unit tests for tool fragments
- Use integration tests for workflows
- Keep test inputs small and public
- Add deterministic tests for metadata or small artifacts
- Validate directory structures for EOAP/STAC compliance
- Automate test execution using GitHub Actions
- Ensure tests run with cwltool and platform runners (e.g., calrissian)

## Conclusion

Testing is an essential component of EOAP development.
Using cwltest ensures your application package is:

- reliable
- reproducible
- ready for sharing
- suitable for execution in operational systems, including OGC API Processes
