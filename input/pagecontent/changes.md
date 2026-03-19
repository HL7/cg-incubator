## Initial Release of CG Incubator - Comparison to Previous Releases

This Implementation Guide represents the initial release of the Clinical Genomics Resource Incubator, which publishes profiled versions of the **GenomicStudy** and **MolecularDefinition** resources. Both resources were previously published in FHIR core but continue to evolve. This section documents changes from their prior published forms.

This IG currently targets **FHIR 6.0.0-ballot3** as an intermediate step toward the upcoming **FHIR R6** normative release. References to "6.0.0-ballot3" throughout this section should be understood in that context — ballot-cycle versions are used to align with the active development state of the base resources, with the expectation that this IG will be updated to target the R6 normative publication once available.

---

### GenomicStudy — Changes from FHIR R5

GenomicStudy was introduced in FHIR R5 (5.0.0). The version in this IG will treat GenomicStudy as an 'Additional Resource' that targets FHIR 6.0.0-ballot3 (interim ballot toward FHIR R6) and incorporates the following changes, grouped by theme.

#### Quality and Coverage Metrics

A new `analysis.metrics` backbone element has been added to support structured capture of sequencing quality information at the analysis level. This was previously unrepresented.

| Element | Card. | Type | Description |
|---|---|---|---|
| `analysis.metrics` | 0..1 | BackboneElement | Quality metrics for the analysis |
| `analysis.metrics.readDepth` | 0..1 | SimpleQuantity | Average read depth (e.g., 30x, 100x) |
| `analysis.metrics.sequencingCoverage` | 0..1 | SimpleQuantity | Percentage of studied regions sequenced (e.g., 95%) |
| `analysis.metrics.description` | 0..1 | string | Freetext coverage metrics description |

#### Genomic Region Consolidation

The flat elements `analysis.regionsStudied` and `analysis.regionsCalled` (each a direct reference to `DocumentReference | Observation`) have been replaced by a unified `analysis.genomicRegion` backbone element. This refactoring:

- Reduces parallel element proliferation by using a type discriminator instead of separate paths.
- Adds a third category, **uncalled**, which was not previously representable.
- Allows additional descriptive context (`description`) per region group.
- Restricts the reference target to `DocumentReference` only (BED file references), removing the broader `Observation` reference that was present in R5.

| Element | Card. | Type | Description |
|---|---|---|---|
| `analysis.genomicRegion` | 0..* | BackboneElement | Genomic regions relevant to the analysis, grouped by type |
| `analysis.genomicRegion.type` | 1..1 | code | `studied` \| `called` \| `uncalled` — Binding: genomicstudy-regiontype (extensible) |
| `analysis.genomicRegion.locus` | 0..* | CodeableReference(DocumentReference) | Genomic regions in this group (coded genes or BED file) |
| `analysis.genomicRegion.description` | 0..1 | string | Additional details about this region group |

#### Clinical Context Enhancement — Genomic Source Class

A new `analysis.genomicSourceClass` element has been added to indicate whether the specimens analyzed are of somatic or germline origin. This is a commonly required clinical data point that was not available at the analysis level in R5.

| Element | Card. | Type | Description |
|---|---|---|---|
| `analysis.genomicSourceClass` | 0..1 | CodeableConcept | The genomic source class of the specimens used in the analysis (e.g., somatic, germline) — Binding: LOINC LL378-1 (extensible) |

#### Terminology Binding Strengthening

Several terminology bindings have been strengthened to improve interoperability:

| Element | R5 Binding Strength | Incubator Binding Strength | Value Set |
|---|---|---|---|
| `analysis.methodType` | example | **preferred** | genomicstudy-methodtype |
| `analysis.changeType` | example | **preferred** | genomicstudy-changetype |

All value set canonical URLs have been migrated from `http://hl7.org/fhir/...` to `http://hl7.org/fhir/uv/cg-incubator/...` to reflect ownership by this IG.

---

### GenomicStudy — Changes from GRIG STU3 Profiles

The HL7 Genomics Reporting Implementation Guide (GRIG) STU3 (version 3.0.0, FHIR R4-based, published 2024-12-12) included profiles for genomic study reporting that predate the introduction of the native `GenomicStudy` resource in FHIR R5. GRIG STU3 modeled genomic study and analysis using two profiles on the FHIR R4 `Procedure` resource: `genomic-study` and `genomic-study-analysis`. The version in this IG represents a continuation of that modeling effort using the native `GenomicStudy` resource.

#### Base Resource Migration

GRIG STU3 profiled the FHIR R4 `Procedure` resource as a backport to represent both the genomic study and the genomic study analysis, because the `GenomicStudy` resource did not exist in FHIR R4. This required extensive use of extensions to express concepts that are now native backbone elements.

This IG uses the native `GenomicStudy` resource (targeting FHIR 6.0.0-ballot3 as an interim ballot toward FHIR R6), eliminating the need for `Procedure`-based backport profiling. All study and analysis concepts are now expressed using the resource's own element tree rather than extensions on a generic clinical action resource.

#### Study-to-Analysis Relationship

In GRIG STU3, the relationship between a study and its constituent analyses was expressed via the `genomic-study-analysis` extension (0..*) on the study-level `Procedure`, each value being a `Reference(Procedure)` pointing to a separate `genomic-study-analysis`-profiled resource. Each analysis was a full, independent `Procedure` instance with its own resource identity.

In this IG, analyses are represented as `analysis` backbone elements (0..*) within the `GenomicStudy` resource itself. This eliminates the inter-resource reference overhead for the common case and co-locates study and analysis data in a single resource instance.

#### Analysis Metrics — Extension to Native Backbone

GRIG STU3 defined a complex extension, `genomic-study-analysis-metrics`, applied to the analysis-level `Procedure` to capture sequencing quality information. The extension used named slices for each metric.

This IG promotes these metrics to a native `analysis.metrics` backbone element. The table below compares the two structures:

| Concept | GRIG STU3 (Extension Slice) | Card. | Type | Incubator (Native Element) | Card. | Type |
|---|---|---|---|---|---|---|
| Read depth | `extension[read-depth].valueQuantity` | 0..1 | SimpleQuantity | `analysis.metrics.readDepth` | 0..1 | SimpleQuantity |
| Sequencing coverage | `extension[sequencing-coverage].valueQuantity` | 0..1 | SimpleQuantity | `analysis.metrics.sequencingCoverage` | 0..1 | SimpleQuantity |
| Metrics description | `extension[metrics-description].valueString` | 0..1 | string | `analysis.metrics.description` | 0..1 | string |

Notable structural differences:
- The GRIG extension allowed multiple instances of the parent extension on the resource (cardinality `0..*`); the Incubator `analysis.metrics` element is `0..1` — metrics for a given analysis are grouped into a single backbone instance.

#### Genomic Regions — Separate Extension Slices to Unified Typed Backbone

GRIG STU3 defined a complex extension, `genomic-study-analysis-regions`, applied to the analysis-level `Procedure`. This extension used named slices to distinguish studied, called, and uncalled regions, applying the following structure:

| GRIG STU3 Extension Slice | Card. | Value Type | Binding |
|---|---|---|---|
| `extension[description].valueString` | 0..1 | string | — (freetext description of the whole regions set) |
| `extension[studied].value[x]` | 0..* | CodeableConcept \| Reference(GenomicDataFile) | HGNC VS (extensible) |
| `extension[called].value[x]` | 0..* | CodeableConcept \| Reference(GenomicDataFile) | HGNC VS (extensible) |
| `extension[uncalled].value[x]` | 0..* | CodeableConcept \| Reference(GenomicDataFile) | HGNC VS (extensible) |

This IG replaces this structure with the `analysis.genomicRegion` backbone element:

| Incubator Element | Card. | Type | Description |
|---|---|---|---|
| `analysis.genomicRegion` | 0..* | BackboneElement | One entry per region group |
| `analysis.genomicRegion.type` | 1..1 | code | `studied` \| `called` \| `uncalled` — Binding: genomicstudy-regiontype (extensible) |
| `analysis.genomicRegion.locus` | 0..* | CodeableReference(DocumentReference) | Genomic regions in this group (coded genes or BED file) |
| `analysis.genomicRegion.description` | 0..1 | string | Description scoped to this region group |

Notable structural differences:
- **Region type discrimination**: GRIG used three separate named extension slices (one per type); the Incubator uses a single repeating backbone element with a `type` code discriminator.
- **Gene code representation**: GRIG used a union type (`CodeableConcept | Reference(GenomicDataFile)`) where `GenomicDataFile` is a GRIG-specific profile on `DocumentReference`. The Incubator uses the `CodeableReference(DocumentReference)` data type, which preserves the same dual capability — inline coded concepts (e.g., HGNC gene symbols) via `CodeableReference.concept`, and BED file references via `CodeableReference.reference` — while using the standard FHIR R6 `CodeableReference` type rather than a polymorphic union, and targeting the base `DocumentReference` rather than a GRIG-specific profile.
- **Description scope**: In GRIG the `description` slice was a single string covering the entire regions extension (all region types). In the Incubator, `description` is scoped per `genomicRegion` entry, enabling separate descriptions for studied, called, and uncalled groups.

#### Genomic Source Class — Extension to Native Element

In GRIG STU3, the genomic origin of specimens was captured via the `genomic-source-class` extension (0..1; extensible binding to LOINC LL378-1) applied to the analysis-level `Procedure`.

This IG promotes this concept to a native `analysis.genomicSourceClass` element (`CodeableConcept`, 0..1; extensible binding to LOINC LL378-1). The binding value set and strength are unchanged; only the mechanism of expression (extension vs. native element) differs.

#### Subject Scope Expansion

The GRIG STU3 `genomic-study` profile restricted the `Procedure.subject` element to `Patient | Group` only. The native `GenomicStudy.subject` in this IG supports a broader set of reference targets: `Patient | Group | Device | Practitioner | Medication | Substance | BiologicallyDerivedProduct | NutritionProduct`, reflecting the resource's intended scope beyond human clinical genomics.

---

### MolecularDefinition — Changes from FHIR R6 Ballot 3

MolecularDefinition was introduced in FHIR R6 and was most recently published in R6 ballot 3 (`6.0.0-ballot3`). The version in this IG is currently published against `6.0.0-ballot3` as an interim ballot release targeting FHIR R6, and incorporates the following changes.

#### Simplified Cytogenetic Band Location

In R6 ballot 3, the `cytobandInterval.startCytoband` and `cytobandInterval.endCytoband` elements are full BackboneElements, each containing four granular sub-elements: `arm[x]`, `region[x]`, `band[x]`, and `subBand[x]`. This structure mirrors the hierarchical notation of cytogenetic band designations (e.g., 17q21.31).

This IG simplifies both elements to polymorphic choice types (`startCytoband[x]` and `endCytoband[x]`), reducing the implementation burden while retaining the ability to convey the same information. This change is intended to be addressed in a future ballot.

| Element | R6 Ballot 3 | Incubator |
|---|---|---|
| `cytobandInterval.startCytoband` | BackboneElement with arm[x], region[x], band[x], subBand[x] | Choice type `startCytoband[x]` |
| `cytobandInterval.endCytoband` | BackboneElement with arm[x], region[x], band[x], subBand[x] | Choice type `endCytoband[x]` |

#### Strengthened Terminology Bindings

Several bindings have been strengthened from extensible or unspecified to **required** in order to enforce consistent representation across implementations:

| Element | R6 Ballot 3 Binding | Incubator Binding | Value Set |
|---|---|---|---|
| `moleculeType` | (no required binding) | **required** | moleculardefinition-moleculetype |
| `location.sequenceLocation.coordinateInterval.coordinateSystem.origin` | (no required binding) | **required** | coordinatesystem-origin |
| `location.sequenceLocation.strand` | (no explicit binding) | **required** | moleculardefinition-strand |
| `representation.focus` | (no required binding) | **required** | moleculardefinition-representation-focus |

#### Curated Local Value Sets

The incubator defines dedicated value sets for all key coded elements, published under the `http://hl7.org/fhir/uv/cg-incubator/` canonical. These include curated content for:

- Molecule type (`moleculardefinition-moleculetype`)
- Molecular topology (`moleculardefinition-topology`)
- Strand orientation (`moleculardefinition-strand`)
- Organism (`moleculardefinition-organism`)
- Representation focus (`moleculardefinition-representation-focus`)
- Representation code (`moleculardefinition-representation-code`)
- Literal encoding (`moleculardefinition-literal-encoding`)
- Coordinate system origin (`coordinatesystem-origin`)
- Coordinate system normalization method (`coordinatesystem-normalizationmethod`)

The LOINC LL1040-6 value set is retained for genome assembly build, and LOINC LL5323-2 is retained for coordinate system type, both at extensible binding strength.

#### Terminology Description Refinements

Minor refinements have been made to element short descriptions to improve precision and domain alignment:

- `moleculeType` short description uses **"polypeptide"** instead of "amino acid" for consistency with molecular biology conventions.
- `genomeAssembly.accession` is described as **"NCBI Assembly accession"** (more specific than ballot 3's generic "Accession").
- `genomeAssembly.build` is described as **"Genome assembly build"** (vs ballot 3's "Build number").
- `cytobandInterval.chromosome` is described as **"Human chromosome identifier"** (explicitly scoped to human use).

---

## Binding Comparison: cg-incubator vs. molecular-definition-data-types

The **cg-incubator** IG defines bindings directly on the base `MolecularDefinition` resource StructureDefinition. The **molecular-definition-data-types** (MolDef DT) IG defines bindings on profiled versions of the resource (Allele, Sequence, Variation, Haplotype, Genotype) using FSH. The following analysis compares the two sets of bindings to inform a CG group decision about where CodeSystems and ValueSets should canonically live.

### Binding Map — Element by Element

| Element | cg-incubator VS | cg-incubator Strength | MolDef DT VS | MolDef DT Strength | Notes |
|---|---|---|---|---|---|
| `moleculeType` | `moleculardefinition-moleculetype` | **required** | *(not bound; `type` used instead)* | — | cg-incubator targets R6's dedicated `moleculeType` element; MolDef DT binds `type` for the same purpose |
| `type` | `moleculardefinition-type` | **extensible** | `MoleculeTypeVS` | **required** | Different elements; different VS content (see §Code Source Strategy) |
| `topology` | `moleculardefinition-topology` | **extensible** | `TopologyVS` | **required** | Same concept; different code sources; binding strength differs |
| `location.sequenceLocation.strand` | `moleculardefinition-strand` | **required** | `StrandOrientationVS` | **required** | Same concept; different code sources |
| `location.…coordinateSystem.origin` | `coordinatesystem-origin` | **required** | `CoordinateOriginVS` | **required** | Similar concept; local CS in both; content differs (see §Concept-Level Differences) |
| `location.…coordinateSystem.normalizationMethod` | `coordinatesystem-normalizationmethod` | **extensible** | `NormalizationMethodVS` | **required** | Same concept; local CS in both; binding strength and content differ |
| `location.…coordinateSystem.system` | LOINC `LL5323-2` | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…organism` | `moleculardefinition-organism` → NCBI Taxonomy | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…build` | LOINC `LL1040-6` | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…chromosome` | LOINC `LL2938-0` | **preferred** | *(not bound)* | — | Only in cg-incubator |
| `representation.focus` | `moleculardefinition-representation-focus` | **required** | *(fixed values on slices, no `from` binding)* | — | Same 4 codes defined in both; MolDef DT uses fixed-value slice discriminators rather than a binding |
| `representation.code` | `moleculardefinition-representation-code` → RefSeq + LRG | **example** | *(not bound)* | — | Only in cg-incubator |
| `representation.literal.encoding` | `moleculardefinition-literal-encoding` | **required** | `EncodingsVS` | **required** | Same concept; local CS in both; code granularity differs (see §Encoding) |

*The `coordinateSystem.*` bindings repeat identically for `representation.extracted.*` and `representation.relative.edit.*` in cg-incubator.*

### Code Source Strategy — Major Conceptual Difference

A fundamental divergence between the two IGs is whether coded concepts anchor to external standard terminologies or to locally coined CodeSystems.

| Concept | cg-incubator approach | MolDef DT approach |
|---|---|---|
| Molecule type (DNA/RNA/AA) | **Sequence Ontology** external codes: `SO:0000352`, `SO:0000356`, `SO:0000104` | **Local CodeSystem** `MoleculeType`: `#dna`, `#rna`, `#aa` |
| Topology | **Sequence Ontology**: `SO:0000987` (linear), `SO:0000988` (circular) | **Local CodeSystem** `Topology`: `#linear`, `#linear-discontiguous`, `#circular`, `#branched` |
| Strand | **Sequence Ontology**: `SO:0001030` (forward), `SO:0001031` (reverse) | **Local CodeSystem** `StrandOrientation`: `#forward`, `#reverse` |
| Subtype (`type` element) | **Sequence Ontology** curated subset (extensible) | Not separately modeled — subtype subsumed into `type` |
| Coordinate origin | **Local CodeSystem** (cg-incubator CS) | **Local CodeSystem** (MolDef DT CS) |
| Normalization method | **Local CodeSystem** (cg-incubator CS) | **Local CodeSystem** (MolDef DT CS) |
| Encoding | **Local CodeSystem** (cg-incubator CS) | **Local CodeSystem** (MolDef DT CS) + character-alphabet CSes |
| Representation focus | **Local CodeSystem** (cg-incubator CS) | **Local CodeSystem** (MolDef DT CS) |

**Key tension:** cg-incubator anchors molecule type, topology, and strand to Sequence Ontology. MolDef DT coins local codes — simpler and more controlled, but not interoperable with SO-based systems without a concept map.

### Concept-Level Differences within Shared Domains

#### Coordinate Origin

| Code concept | cg-incubator CS | MolDef DT CS |
|---|---|---|
| Sequence start | ✅ (`#sequence-start`) | ✅ (`#sequence-start`) |
| CDS start | ✅ (`#cds-start`) | ❌ not present |
| Feature start | ✅ (`#feature-start`) | ✅ (`#feature-start`) |
| Feature end | ✅ (`#feature-end`) | ✅ (`#feature-end`) |

`#cds-start` (start of the coding sequence / ATG codon) is defined in cg-incubator but absent from MolDef DT. MolDef DT's `#feature-start` could cover this by convention, but the concepts are in tension.

#### Normalization Method

| Code concept | cg-incubator CS | MolDef DT CS |
|---|---|---|
| Left shift | ✅ (`#left-shift`) | ✅ (`#left-shift`) |
| Right shift | ✅ (`#right-shift`) | ✅ (`#right-shift`) |
| Fully justified | ✅ (`#fully-justified`) | ✅ (`#fully-justified`) |
| No normalization | ✅ (`#no-normalization`) | ❌ not present |

`#no-normalization` is defined in cg-incubator and absent from MolDef DT. Because MolDef DT binds `NormalizationMethodVS` at **required** strength, this concept cannot currently be expressed using MolDef DT profiles.

#### Topology

| Code concept | cg-incubator VS (SO codes) | MolDef DT CS |
|---|---|---|
| Linear | ✅ `SO:0000987` | ✅ `#linear` |
| Circular | ✅ `SO:0000988` | ✅ `#circular` |
| Linear discontiguous | ❌ not in VS | ✅ `#linear-discontiguous` |
| Branched | ❌ not in VS | ✅ `#branched` |

MolDef DT's topology coverage is richer at the code level. cg-incubator's **extensible** binding would allow additional SO codes or local codes to express these concepts, but they are not currently enumerated.

#### Representation Focus

All 4 codes are aligned in both IGs, though they live in separate local CodeSystems with different canonical URLs:

| Code | cg-incubator | MolDef DT |
|---|---|---|
| `allele-state` | ✅ | ✅ |
| `context-state` | ✅ | ✅ |
| `reference-state` | ✅ | ✅ |
| `alternative-state` | ✅ | ✅ |

### Encoding (Literal Representation)

cg-incubator defines encoding *schemes* as 9 category codes. MolDef DT also defines equivalent category codes in the `Encodings` CS, and additionally provides three character-alphabet CodeSystems (`NucleotideDNA`, `NucleotideRNA`, `AminoAcid`) with 8 corresponding ValueSets enabling validation of actual sequence string content.

| Encoding category | cg-incubator code | MolDef DT code | Aligned? |
|---|---|---|---|
| Nucleotide DNA 1-letter unambiguous | `nuc-dna-1-noamb` | `nucleotide-dna-1letter-unambiguous` | ✅ same concept |
| Nucleotide RNA 1-letter unambiguous | `nuc-rna-1-noamb` | `nucleotide-rna-1letter-unambiguous` | ✅ same concept |
| Nucleotide DNA 1-letter with N | `nuc-dna-1-noamb-n` | `nucleotide-dna-1letter-with-n` | ✅ same concept |
| Nucleotide DNA 1-letter ambiguous (IUPAC) | `nuc-dna-1-amb` | `nucleotide-dna-1letter-ambiguous` | ✅ same concept |
| Nucleotide RNA 1-letter ambiguous | `nuc-rna-1-amb` | ❌ not present | cg-incubator only |
| AA 1-letter unambiguous (20 standard) | `aa-1-noamb-20common` | `amino-acid-1letter-unambiguous` | ✅ same concept |
| AA 3-letter unambiguous (20 standard) | `aa-3-noamb-20common` | `amino-acid-3letter-unambiguous` | ✅ same concept |
| AA 1-letter ambiguous | `aa-1-amb` | `amino-acid-1letter-ambiguous` | ✅ same concept |
| AA 3-letter ambiguous | `aa-3-amb` | `amino-acid-3letter-ambiguous` | ✅ same concept |

### Bindings Only in cg-incubator

The following bindings exist in cg-incubator with no equivalent in MolDef DT:

| Element | ValueSet / System | Strength | Comment |
|---|---|---|---|
| `coordinateSystem.system` | LOINC `LL5323-2` | extensible | Coordinate numbering reference system (HGVS, VCF, etc.) |
| `cytobandLocation.genomeAssembly.organism` | NCBI Taxonomy | extensible | Species for genome assembly |
| `cytobandLocation.genomeAssembly.build` | LOINC `LL1040-6` | extensible | Reference genome build (GRCh38, etc.) |
| `cytobandLocation.cytobandInterval.chromosome` | LOINC `LL2938-0` | preferred | Chromosome identifier |
| `representation.code` | RefSeq + LRG | example | Accession-based reference to sequence databases |
| `moleculeType` (R6 element) | Local CS → SO codes | required | R6-specific dedicated element; MolDef DT does not model this separation |
| `type` (subtype element) | SO-based curated subset | extensible | Detailed subtype classes (mRNA, rRNA, genomic DNA, etc.) |

### Experimental Status

An asymmetry exists within MolDef DT: its CodeSystems are marked `experimental = false` (stable), while its ValueSets are marked `experimental = true`. cg-incubator does not explicitly set experimental flags on its terminology artifacts.

### Open Questions for CG Group

| Question | Status |
|---|---|
| **Where should shared CodeSystems live?** | Both IGs define the same logical concepts (strand, topology, coordinate origin, normalization, encoding, representation focus) in separate local CSes with different canonicals. These need to be merged into one canonical location — the MolDef base resource IG, a shared utility IG, or the FHIR core spec. |
| **Should SO be used for molecule type, topology, and strand?** | cg-incubator uses Sequence Ontology; MolDef DT uses local codes. This is a core tension requiring a decision. |
| **Character-alphabet CSes — scope?** | Only MolDef DT defines character-level CodeSystems for sequence validation. Should these be in scope for the base resource IG? |
| **`cds-start` and `no-normalization` gaps** | These concepts are present in cg-incubator but absent from MolDef DT. MolDef DT's `required` binding on normalization method means `no-normalization` cannot currently be expressed in MolDef DT profiles. |
| **Topology coverage** | MolDef DT has `#linear-discontiguous` and `#branched`; cg-incubator does not. The cg-incubator `extensible` binding accommodates these; MolDef DT's `required` binding would need new codes added to accommodate future concepts. |
| **Binding strength for shared elements** | cg-incubator uses `extensible` for topology and normalization method; MolDef DT uses `required` for both. The right strength depends on whether local/SO codes will be permitted. |
| **cytobandLocation bindings** | These are only modeled in cg-incubator; MolDef DT profiles do not cover cytogenomic scope. |