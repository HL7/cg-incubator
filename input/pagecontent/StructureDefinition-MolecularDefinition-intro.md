## Scope and Usage

The MolecularDefinition resource represents molecular entities (e.g., nucleotide or protein sequences) for both clinical and non-clinical use cases, including translational research. The resource is definitional, in that it focuses on discrete, computable, and semantically expressive data structures that reflect the genomic domain. Because the resource focuses on the molecular entities rather than specimen source or annotated knowledge, it supports both patient/participant-specific use cases and population-based data, and both human and non-human data.

The MolecularDefinition resource itself is abstract, but it supports profiles for core molecular concepts, including Sequence (nucleotide and protein), Allele, Variation, Haplotype, and Genotype. Support for additional molecular types, such as structural variation, fusions, and biomarkers, will be considered in the future.

Use cases supported by this resource include but are not limited to:

- Structured exchange of simple sequences of DNA, RNA, or amino acids (whole genome/exome sequencing)
- Representation of clinically significant alleles that impact drug response (e.g., pharmacogenomic CDS)
- Structured representation of simple and complex genetic variations for diagnostic purposes (clinical diagnosis or risk)
- Expression of genotypes that have clinical or research significance (clinical decision making)
- Representation of genomic variations that are stored within a public knowledge base
- Expression of alleles that are used within risk calculators

### Sequence Representation

Use cases often require expression of the same genomic concept in different ways. Since the concept is the same and only the serialization of it differs, the Molecular Definition resource supports multiple approaches to representing molecular sequences. This allows senders and receivers of messages to choose a sequence representation that is most intuitive for the particular use case.

It is important to note that all representations of a given sequence MUST resolve to the exact same primary sequence. Therefore, if a single instance of MolecularDefinition contains one `literal`, two `resolvable` files, and a `code`, all four of those representations must represent the same sequence. Note that this equivalence does not apply to metadata or annotations that are outside the scope of the Molecular Definition resource, since those data are not definitional to the molecule.

## Boundaries and Relationships

The MolecularDefinition resource should be profiled and used to capture representations of molecular concepts such as sequence, allele, haplotype, and genotype.

This resource does not capture workflow (e.g., test ordering/resulting process), the method of obtaining or specifying the molecular content (e.g., the test or assay), or the interpretation of the results (e.g., clinical impact). Those concepts will be captured by profiles of Observation and by the Genomic Study resource. In particular, the Genomics Reporting Implementation Guide contains extensive support for the observation and reporting of clinical genomic results.
