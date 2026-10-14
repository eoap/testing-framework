# Writing tests

## Choose a test boundary

Use a tool-level case to isolate one processing stage and a workflow-level case to exercise the connections between stages. Saved intermediate fixtures make it easier to identify which stage introduced a regression.

Choose cases from the application's requirements: a representative scene, an area with no water, nodata at the crop boundary, or invalid parameters may warrant separate coverage. These are possible additions; the current suite contains only the six cases described in the [example](water-bodies.md).

## Define a case

The suite is a YAML list. This complete case follows the existing crop test:

```yaml
- id: crop-green
  doc: "Crop the green band for the example area of interest."
  tool: https://github.com/eoap/mastering-app-package/releases/download/1.1.0/app-water-bodies-cloud-native.1.1.0.cwl#crop
  job: inputs-01.yaml
  output:
    cropped:
      class: File
      location: Any
```

| Field | Purpose |
| --- | --- |
| `id` | Unique name used to select the case |
| `doc` | Description of the behavior being tested |
| `tool` | CWL document, optionally with a process fragment |
| `job` | YAML job containing the application's input values |
| `output` | Expected values keyed by CWL output names |

Use descriptive IDs such as `crop-green` or `workflow-end-to-end`. Output names must match the selected CWL process; `cropped` and `ndwi` are different output ports.

## Supply job inputs

This is the normalized-difference job used by the repository:

```yaml
rasters:
  - class: File
    location: data/crop_green.tif
  - class: File
    location: data/crop_nir.tif
```

Preserve array order when it affects the computation. Check the tool's declared input types before choosing a scalar, array, URL string, or CWL `File` object. The [CWL input guide](https://www.commonwl.org/user_guide/topics/inputs.html) describes the input type system.

## Match assertion strength to the requirement

### Flexible file outputs

```yaml
output:
  ndwi:
    class: File
    location: Any
```

This avoids tying the expectation to an execution-specific path. It does not specify file contents, pixel values, or a checksum. Use it when successful execution and the output type are the intended checks, and add stronger validation when content matters.

### Deterministic files

The following excerpt uses the catalog baseline from the existing suite:

```yaml
output:
  stac_catalog:
    class: Directory
    listing:
      - class: File
        location: catalog.json
        checksum: "sha1$70e96cddbb205363b4ee9cb763e438a29fd29cdc"
        size: 363
```

This excerpt covers only the top-level catalog. The repository's actual catalog tests also assert the nested item and raster. Model nested outputs with directory listings, following `tests/test-suite.yaml`.

Byte comparisons are useful when serialization and raster encoding are stable. If acceptable results can differ in encoding or numerical precision, consider a separate validation step that checks meaningful properties such as raster dimensions, CRS, nodata handling, class values, or numerical tolerance. Such semantic checks are not implemented by the current suite.

### Expected failures

`cwltest` supports expected-failure cases through `should_fail`. Consult the [upstream framework](https://github.com/common-workflow-language/cwltest) and the installed version's documentation before adding them. Design the input so the failure represents the intended invalid condition; a missing container or unavailable URL should not be mistaken for successful validation of application error handling.

## Add a case safely

1. Inspect the selected CWL process and its input/output names.
2. Add a small, documented fixture and a job file under `tests/`.
3. Add a uniquely named entry to `tests/test-suite.yaml`.
4. Run the case with `--id`, inspect its artifacts, and establish expectations.
5. Run the full suite to check for regressions.
6. Explain any fixture or expected-output changes in the pull request.

See [fixtures and reproducibility](reproducibility.md) before updating checksums.
