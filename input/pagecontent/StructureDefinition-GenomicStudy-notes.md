#### Terminology Bindings

GenomicStudy uses several terminology bindings across its elements. Reviewers and implementers are strongly encouraged to provide their comments and feedback about the bound value sets and code systems.

The attributes and their bindings are listed below. Links are included to indicate where the example codes were pulled from.

##### GenomicStudy.type

The `type` example codes were based on discussions by Clinical Genomics Workgroup.

##### GenomicStudy.analysis.methodType

The `methodType` element has a **preferred** binding to a value set of testing method codes. The codes were pulled from [National Library of Medicine-Genetic Testing Registry (NCBI-GTR)](https://www.ncbi.nlm.nih.gov/gtr/) and describe testing methods on various levels: [major method category](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Major_method_category.txt), [method category](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Method_category.txt), and [primary methodology](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Primary_test_methodology.txt). Implementers should use the bound codes where applicable and only deviate with a local code when a suitable match is not available.

##### GenomicStudy.analysis.input.type and GenomicStudy.analysis.output.type

The input and output type example codes were pulled from [Integrative Genomics Viewer Documentation](https://software.broadinstitute.org/software/igv/FileFormats) by Broad Institute.

##### GenomicStudy.analysis.changeType

The `changeType` element has a **preferred** binding. The codes were based on discussions by Clinical Genomics Workgroup and draw from [Sequence Ontology (SO)](http://www.sequenceontology.org/) terms such as `SO:0001483` (SNV), `SO:0002007` (MNV), `SO:1000032` (delins), `SO:0001019` (copy_number_variation), `SO:0001565` (gene_fusion), and `SO:0001576` (transcript_variant). Implementers should use the bound codes where applicable.

##### GenomicStudy.analysis.genomicSourceClass

The `genomicSourceClass` element records whether the genomic material analyzed originates from the germline or is somatic in origin. It has an **extensible** binding to LOINC answer list [LL378-1](https://loinc.org/LL378-1/). Commonly used codes include `LA6683-2` (Germline) and `LA6684-0` (Somatic). This element is particularly important in cancer genomics workflows where paired tumor/normal analyses are performed, and in clinical reporting where the origin of a variant affects its interpretation.

##### GenomicStudy.analysis.regionsStudied, regionsCalled, and regionsUncalled

These three elements use the [CodeableReference]({{site.data.fhir.path}}references.html#CodeableReference) datatype bound to HGNC gene identifiers (extensible). The binding uses the [hgnc-vs](https://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs) ValueSet from the genomics-reporting IG — implementations must therefore include the genomics-reporting package as a dependency. See the [terminology considerations page](terminology-considerations.html) for full rationale.

Each entry may be expressed as:

- A coded gene identifier using the `concept` side (e.g., an HGNC gene code such as `HGNC:76` for ABL1), appropriate for targeted gene panels and pharmacogenomics studies where specific genes are enumerated.
- A reference to a [DocumentReference]({{site.data.fhir.path}}documentreference.html) using the `reference` side, pointing to a BED file defining genomic coordinates, appropriate for whole-exome, whole-genome, or large panel studies.

Both forms may appear within the same analysis element — an analysis can enumerate individual gene codes alongside a BED file reference in separate `regionsStudied` entries. The `regionsUncalled` element follows the same pattern and should be used to document any regions or genes for which reliable calls could not be made (e.g., due to low coverage, homologous regions, or assay limitations).

##### GenomicStudy.analysis.metrics

The `metrics` backbone element captures quality metrics from the sequencing run at the analysis level. It contains:

- `metrics.readDepth` — the average read depth (e.g., `120x`) as a SimpleQuantity.
- `metrics.sequencingCoverage` — the percentage of the target region covered at a defined threshold (e.g., `98%`) as a SimpleQuantity.
- `metrics.description` — a free-text field for additional quality information not captured by the structured fields.

These metrics are informational and help interpreters and downstream consumers assess the quality and completeness of the sequencing data.

##### GenomicStudy.analysis.performer.role and analysis.device.function

These two elements carry `CodeableConcept` values but have no formal ValueSet binding in the current StructureDefinition. Until a binding is established:

- For `performer.role`, use HL7 v3 ParticipationType (`http://terminology.hl7.org/CodeSystem/v3-ParticipationType`), e.g., `PRF` (Performer).
- For `device.function`, use LOINC where applicable, e.g., `LA26398-0` (Sequencing).

See the [terminology considerations page](terminology-considerations.html) for further notes on these elements.

#### Implementation Notes

##### Handling markdown

The `description` is a [markdown]({{site.data.fhir.path}}datatypes.html#markdown) datatype, and implementers should carefully consider how to appropriately handle this attribute. The characters in markdown formatting can conflict with those commonly used in descriptions of genomic data. In particular, descriptions that contain mentions of "star alleles" (widely used in the pharmacogenomics and HLA domains) could be munged in a way that prevents accurate interpretation. For example, consider this text:

```
This genomic study analyzes CYP2D6*1 and CYP2D6*2
```

If the system producing this data treats this as a simple text string with no special processing, but a receiving system processes this via a markdown rendering engine, the two '*' characters would be processed as markdown formatting characters. This would italicize the text between '*' characters, and not display '*' characters. This could cause an inaccurate interpretation of the study description.

There are several ways data producers can ensure content is appropriately rendered by receiving systems without requiring the content to be formatted as markdown prior to sending. Here are three basic approaches to consider:

- Escaping individual characters (with a \) that act as markdown formatting characters:
  ```
  This genomic study analyzes CYP2D6\*1 and CYP2D6\*2
  ```

- Escape words (with a `) that contain markdown formatting characters:
  ```
  This genomic study analyzes `CYP2D6*1` and `CYP2D6*2`
  ```

- Escape full text blocks (with a ```) that contain markdown formatting characters:
  ```
  This genomic study analyzes CYP2D6*1 and CYP2D6*2
  ```

#### Use Cases

The attributes `subject` and `analysis.focus` can reference many resource types besides [Patient]({{site.data.fhir.path}}patient.html) such as [Group]({{site.data.fhir.path}}group.html), [BiologicallyDerivedProduct]({{site.data.fhir.path}}biologicallyderivedproduct.html), or [Substance]({{site.data.fhir.path}}substance.html). In addition, it can provide more details about involved genomic files as inputs or outputs. These various options allow the GenomicStudy resource to cover many use cases besides direct patient care, e.g., research studies that involve multiple patients or environmental samples. Through the following subsections, some of these use cases are described.

##### Trio studies

Trio studies involve a proband and two more subjects such as proband's mother and father for a de novo mutation detection study. GenomicStudy would list the proband as `subject` because it is the main subject of the study. Each of the study participants, i.e., proband, mother, and father, may have their own `analysis` entry. When an analysis was performed on an entity other than the `subject`, the `analysis.focus` attribute would reference that entity. If a `analysis` entry documented the analysis of all participants of the trio, each participant would be referenced by `analysis.focus`.

The `analysis.input` lists various files that may be used for each individual analysis, their types, and their generation context. `analysis.input.file` may link these files to [DocumentReference]({{site.data.fhir.path}}documentreference.html) resources to provide details about each individual file. One of the main details is the subject of a file. If a file is linked to a specific patient, the corresponding `DocumentReference.subject` may reference this [Patient]({{site.data.fhir.path}}patient.html) Resource. If the file contains data from multiple persons, the corresponding `DocumentReference.subject` may reference a [Group]({{site.data.fhir.path}}group.html) resource that lists these persons, and their relationship to each other if available.

##### Somatic mutation detection studies

Somatic mutation studies may use multiple samples from the patient to support mutation detection, e.g., tumor-normal sample pair. `analysis.genomicSourceClass` should be coded as `LA6684-0` (Somatic) for any analysis performed on tumor-derived material. A single `analysis` entry may reference multiple `specimen` elements to represent paired tumor and germline samples within one analysis. In addition, `analysis.input` can list relevant input files, where the [DocumentReference]({{site.data.fhir.path}}documentreference.html) referenced by `analysis.input.file` can list the specific specimen this input file is related to using `DocumentReference.subject` as a [Specimen]({{site.data.fhir.path}}specimen.html) resource instance.

Where certain genomic regions cannot be reliably called — for example, due to the segmental duplication structure of a gene, low tumor cellularity, or poor coverage — these should be recorded using `analysis.regionsUncalled`. This element supports both coded gene identifiers (HGNC codes) and references to BED files listing uncallable coordinates. Documenting uncallable regions explicitly helps downstream consumers distinguish true negatives from untested regions.

RNA sequencing for fusion detection should be represented as a separate `analysis` entry within the same GenomicStudy, using `methodType` codes such as `rna-analysis` and `changeType` codes from Sequence Ontology (e.g., `SO:0001565` for gene_fusion). The RNA analysis typically uses only the tumor specimen.

##### Pharmacogenomics (PGX) studies

PGX studies analyze a patient's germline variants in drug-metabolizing enzyme genes (e.g., CYP2C19, CYP2C9, VKORC1) to inform drug selection and dosing. `analysis.genomicSourceClass` should be coded as `LA6683-2` (Germline). Because PGX panels target a discrete set of well-characterized genes, `regionsStudied` and `regionsCalled` entries should preferably use HGNC gene codes as the `concept` side of `CodeableReference` rather than BED file references — this makes the coverage scope explicit and computable without reference to external files. The `metrics.sequencingCoverage` field is particularly meaningful here, as 100% coverage of all target genes is typically required for a reportable result.

##### EHR access to genomic studies

EHR systems can use GenomicStudy to navigate to genomic data for a given patient. By querying for GenomicStudy resources associated with a patient, a system can discover what analyses were performed, what specimens were used, what genomic files are available, and what quality metrics were achieved. The study can then serve as an anchor for retrieving related [DiagnosticReport]({{site.data.fhir.path}}diagnosticreport.html) and [Observation]({{site.data.fhir.path}}observation.html) resources that contain the interpretive findings.
