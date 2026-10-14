# Fixtures and reproducibility

A reproducible test needs a known application, known input data, and a stable execution environment. Fixed expected outputs help detect changes, but they do not freeze those dependencies by themselves.

## Current baseline

| Dependency | Repository behavior | Consequence |
| --- | --- | --- |
| CWL application | Release URL for version `1.1.0` | Cases select a specific application release |
| `cwltest` | CI pins `2.6.20250818005349` | Local setup can match the framework version |
| `cwltool` | Unpinned in CI | Runner behavior may change between installations |
| Remote fixtures | URLs reference `main` | Data may change without changing the CWL release |
| Containers | Declared by the external CWL package | Review that package's image references when diagnosing drift |

For a stronger baseline, consider commit-based fixture URLs, an explicit runner version, and container image digests. Apply those changes deliberately to the application and suite, then validate the complete workflow.

## Fixture roles

`tests/data/` contains source band rasters, a STAC item describing their assets, cropped band rasters, normalized-difference output, a thresholded mask, and catalog artifacts. Intermediate fixtures let unit cases run without recomputing every preceding stage.

Document the origin, acquisition date, area of interest, processing parameters, and redistribution terms when adding data. Retain the pixels and metadata needed to exercise the behavior while keeping fixtures small enough for routine CI execution.

A local edit to `tests/data/item.json` will not affect jobs that still read its URL on `main`. When experimenting with new fixtures, explicitly point inputs to the intended data source and check any asset URLs embedded in the item.

## Record expected bytes

On Linux, calculate a checksum and byte count for an inspected output:

```bash
sha1sum catalog.json
wc -c < catalog.json
```

Record the values in the expected output:

```yaml
checksum: "sha1$<digest>"
size: 363
```

The digest placeholder must be replaced with the actual value. The size above illustrates the current catalog baseline; use the measured size for a new artifact.

For a collection of output files:

```bash
find output-directory -type f -exec sha1sum {} \;
```

Run these commands against generated artifacts whose contents you have reviewed. A checksum establishes equality to a chosen baseline, not correctness of that baseline.

## Review a baseline change

When a size or checksum changes, inspect the artifact before editing the expectation. JSON whitespace, field ordering, timestamps, raster compression, metadata, and actual pixel changes can all affect bytes.

1. Identify whether the application, input data, runner, or container changed.
2. Compare metadata and raster properties with the previous artifacts.
3. Confirm the change meets the intended application behavior.
4. Update all affected expectations. The catalog and end-to-end cases share the same three artifact expectations.
5. Run the affected unit cases and the full workflow.
6. Record why the new baseline is accepted in the pull request.

## Beyond byte equality

For scientific validation, consider known reference masks, tolerances for floating-point rasters, nodata behavior, and coverage of different scene conditions. For STAC validation, add explicit schema and asset-link checks. These checks complement the current suite and require their own validation implementation.
