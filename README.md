# Clinical Genomics Incubator Implementation Guide

## Overview

The Clinical Genomics Incubator Implementation Guide defines additional FHIR resources for genomic data exchange and analysis. This IG hosts two primary resources—**GenomicStudy** and **MolecularDefinition**—designed to support clinical genomics workflows and to be leveraged by other implementation guides such as Genomics Reporting and Molecular Definition Data Types.

This IG serves as an incubator for clinical genomics resources, allowing them to mature through community feedback and real-world implementation before potential integration into the base FHIR specification.

## Building the IG

### Prerequisites
- Node.js and npm installed
- SUSHI (FSH compiler) - install with: `npm install -g fsh-sushi`
- Java runtime environment (for the FHIR IG Publisher)

### Build Instructions

To build the implementation guide, run:

```bash
./_build.sh
```

This will:
1. Install/update dependencies via the FHIR IG Publisher
2. Run SUSHI to compile FSH files into FHIR artifacts
3. Generate the HTML-based implementation guide output

The built output will be available in the `output/` directory.

### Development

- **FSH Resources**: Source files are located in `input/fsh/`
- **XML Resources**: Pre-built resources are in `input/resources/`
- **Configuration**: See `sushi-config.yaml` for IG configuration and `ig.ini` for publisher settings

## Links

- **CI Build**: https://build.fhir.org/ig/HL7/cg-incubator/branches/main/en/index.html
- **FHIR Additional Resources Guidance**: https://build.fhir.org/structuredefinition.html#additional
