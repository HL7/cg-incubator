#### Example Codes

GenomicStudy uses many example codes for terminology binding. Reviewers and implementers are strongly encouraged to provide their comments and feedback about the example codes for terminology binding purposes.

The attributes and example bindings are listed below. Links are included to indicate where the example codes were pulled from.

##### GenomicStudy.type

The `type` example codes were based on discussions by Clinical Genomics Workgroup.

##### GenomicStudy.analysis.methodType

The `methodType` example codes were pulled from [National Library of Medicine-Genetic Testing Registry (NCBI-GTR)](https://www.ncbi.nlm.nih.gov/gtr/) and describe testing methods on various levels: [major method category](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Major_method_category.txt), [method category](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Method_category.txt), and [primary methodology](https://ftp.ncbi.nlm.nih.gov/pub/GTR/standard_terms/Primary_test_methodology.txt).

##### GenomicStudy.analysis.input.type and GenomicStudy.analysis.output.type

The input and output type example codes were pulled from [Integrative Genomics Viewer Documentation](https://software.broadinstitute.org/software/igv/FileFormats) by Broad Institute.

##### GenomicStudy.analysis.changeType

The `changeType` example codes were based on discussions by Clinical Genomics Workgroup.

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

Somatic mutation studies may use multiple samples from the patient to support mutation detection, e.g., tumor-normal sample pair. `analysis` may describe conducted analyses per each sample, where `analysis.specimen` elements may provide details about each individual specimen. In addition `analysis.input` can list relevant input files, where the [DocumentReference]({{site.data.fhir.path}}documentreference.html) referenced by `analysis.input.file` can list the specific specimen this input file is related to using `DocumentReference.subject` as a [Specimen]({{site.data.fhir.path}}specimen.html) resource instance.

##### EHR access to genomic studies
