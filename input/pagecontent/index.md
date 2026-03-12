### Overview

The Clinical Genomics Incubator Implementation Guide defines foundational resources for genomic data exchange and analysis. This IG serves as an incubator for Clinical Genomics Work Group, hosting additional resources designed to be leveraged by other implementation guides such as Genomics Reporting and Molecular Definition Data Types.

#### Primary Resources

This IG defines two additional resources:

**[GenomicStudy](StructureDefinition-GenomicStudy.html)**

GenomicStudy represents a set of analyses performed to analyze and generate genomic data. This resource captures the overall genomic testing event, including information about the type of study, the subject being studied, associated test results, and metadata about the genomic analysis performed. It serves as the umbrella resource for organizing and tracking genomic studies within clinical workflows.

**[MolecularDefinition](StructureDefinition-MolecularDefinition.html)**

MolecularDefinition provides definitional content for molecular entities, such as nucleotide or protein sequences. This resource enables precise representation of molecular components referenced in genomic studies, including structural information about molecules and their compositions. It serves as a foundational building block for representing complex molecular data.

#### About Additional Resources

This implementation guide publishes [Additional Resources](https://build.fhir.org/structuredefinition.html#additional), which are FHIR resources that go beyond the base FHIR specification. These resources are designed to meet specific clinical and technical needs while undergoing development and refinement. Additional Resources allow implementers to use these specialized resources in production implementations while the resources continue to evolve through the standards development process.

The resources defined in this IG are intended to be integrated into the base FHIR specification once they have matured through community feedback and validation in real-world implementations.


### Cross Version Analysis

{% lang-fragment cross-version-analysis.xhtml %}

### Dependency Table

{% lang-fragment dependency-table.xhtml %}

### Global Profiles Table

{% lang-fragment globals-table.xhtml %}

### IP Statements

{% lang-fragment ip-statements.xhtml %}
