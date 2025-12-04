# EOAP Testing Framework

This repository hosts CWL test suites and input data for validating EO Application Packages (EOAPs) with cwltest.

## Documentation 

The webpage of the documentation is https://eoap.github.io/testing-framework/. 

## Structure

```
tests/
  test-suite.yaml
  inputs-*.yaml
```

## Run Tests

```
pip install cwltool cwltest
cwltest --test tests/<application>/test-suite.yaml
```
## CI Example

```yaml
- run: pip install cwltool cwltest
- run: cwltest --test tests/<application>/test-suite.yaml
```

## Contributing

Add or update test suites following the existing structure.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)