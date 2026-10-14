# EOAP Testing Framework

This repository provides Common Workflow Language (CWL) test suites, input data, and guidance for validating Earth Observation Application Packages (EOAPs) with `cwltest`.

The example suite tests the **water-bodies application package, version 1.1.0**, from [mastering-app-package](https://github.com/eoap/mastering-app-package). The application code is downloaded from that project's release; this repository contains the tests and fixtures.

Read the [documentation](https://eoap.github.io/testing-framework/) for guidance on writing and organising EOAP tests.

## Example workflow and test coverage

The application crops green and near-infrared satellite bands to an area of interest, computes the Normalized Difference Water Index (NDWI), applies Otsu thresholding to produce a water mask, and packages the result in a SpatioTemporal Asset Catalog (STAC).

[tests/test-suite.yaml](tests/test-suite.yaml) defines five unit tests and one end-to-end integration test:

| Test ID | Step | Expected output |
| --- | --- | --- |
| `crop-green` | Crop the green band | A cropped raster file |
| `crop-nir` | Crop the near-infrared band | A cropped raster file |
| `normalized-difference` | Compute NDWI | An NDWI raster file |
| `otsu-threshold` | Apply Otsu thresholding | A binary water mask file |
| `stac-generation` | Generate the STAC catalog | Directory contents matching expected sizes and SHA-1 checksums |
| `workflow-end-to-end` | Run the complete workflow | Directory contents matching expected sizes and SHA-1 checksums |

The first four tests use `location: Any` to allow flexible output paths. They do not assert raster checksums or pixel values. The final two tests check the catalog JSON, item JSON, and water-mask raster against fixed sizes and checksums.

## Repository structure

```text
tests/
  test-suite.yaml       # Test definitions and expected outputs
  inputs-01.yaml        # Crop: green band
  inputs-02.yaml        # Crop: near-infrared band
  inputs-03.yaml        # Normalized difference
  inputs-04.yaml        # Otsu thresholding
  inputs-05.yaml        # STAC catalog generation
  inputs-e2e.yaml       # Complete workflow
  data/                # Satellite imagery, intermediate rasters, and STAC fixtures
docs/
  index.md             # Testing guide
mkdocs.yml             # Documentation site configuration
.github/workflows/
  run-tests.yaml       # Run the suite on pushes and pull requests
  docs.yaml            # Build and publish the documentation
```

## Run the tests

Use Python 3.11 to match CI, and have a container runtime available for the application's CWL container requirements (Docker is the usual choice). Run the following commands from the repository root:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install cwltest==2.6.20250818005349 cwltool
cwltest --test tests/test-suite.yaml --tool cwltool --verbose --
```

The `cwltest` version above matches [the CI workflow](.github/workflows/run-tests.yaml); `cwltool` is currently unpinned.

Run a single test by its ID:

```bash
cwltest --test tests/test-suite.yaml --id stac-generation
```

Execution requires network access to download the CWL application package, referenced inputs, and container images. The CWL release is fixed at version `1.1.0`, while remote fixture URLs reference this repository's `main` branch. Those fixtures can change independently of the application release.

## Continuous integration

[run-tests.yaml](.github/workflows/run-tests.yaml) runs the complete suite on every push and pull request using an Ubuntu runner and Python 3.11.

[docs.yaml](.github/workflows/docs.yaml) builds and publishes the MkDocs site when documentation, site configuration, or GitHub workflow files change on `main` or `develop`.

## Preview the documentation

With your virtual environment active, install the documentation dependencies and start the local preview:

```bash
python -m pip install mkdocs-material mkdocs-mermaid2-plugin
mkdocs serve
```

## Add or update tests

1. Add a job input file under `tests/`, using the existing inputs as examples.
2. Add a case to `tests/test-suite.yaml` with a unique `id`, a description, the CWL `tool` URL or path, the `job` file, and expected `output`.
3. Keep fixtures small and accessible. Use saved intermediate inputs to test individual steps independently.
4. For deterministic outputs, record file sizes and checksums using `sha1sum`, and use the `sha1$<digest>` format in the suite.
5. Run the new case by ID, then run the full suite before submitting a pull request.

When changing the application release or fixtures, review the expected outputs and update checksums only after confirming that the changes are intended. See the [testing guide](docs/index.md) for examples.

## License

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
