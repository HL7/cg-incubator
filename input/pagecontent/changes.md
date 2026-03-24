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

## Terminology Alignment: cg-incubator and molecular-definition-data-types

> **Note:** Neither this IG nor the molecular-definition-data-types IG has been officially released. The comparison below is a point-in-time snapshot of two actively developing IGs. The current state of each IG is what matters; how terminology was previously defined is not relevant.

The **cg-incubator** IG defines bindings directly on the base `MolecularDefinition` resource StructureDefinition. The **molecular-definition-data-types** (MolDef DT) IG defines bindings on profiled versions of the resource (Allele, Sequence, Variation, Haplotype, Genotype) using FSH.

**Intended relationship:** The MolDef DT IG is expected to eventually declare the Incubator IG as a dependency, at which point the MolDef DT profiles will inherit all terminology resources (CodeSystems, ValueSets) and bindings defined here. Under that model, the Incubator is the canonical source of truth for all `MolecularDefinition`-related terminology, and the MolDef DT IG will reference rather than duplicate those resources. The current state of both IGs has been aligned in anticipation of that dependency.

### Alignment Decisions (March 2025)

| Topic | Decision |
|---|---|
| **SO vs. local codes** | Drop Sequence Ontology codes for `moleculeType`, `topology`, `strand`, and `type`. Replace with locally-defined CodeSystems aligned 1-1 with MolDef DT codes. |
| **Code content (union rule)** | Where the two IGs previously had partial overlap, the Incubator CodeSystems carry the **superset** of both IGs' codes (e.g., topology includes all 4 codes: `#linear`, `#linear-discontiguous`, `#circular`, `#branched`; normalizationMethod includes `#no-normalization`). |
| **Experimental flag** | All locally-defined terminology resources in this IG are marked `experimental = true`. |
| **CodeSystem home** | The Incubator IG is the canonical home for all `MolecularDefinition`-related CodeSystems and ValueSets. The MolDef DT IG will reference these resources as a dependency rather than defining its own. |
| **Topology and normalizationMethod binding strength** | Both are promoted to **required** (matching MolDef DT). |
| **Character-alphabet CodeSystems** | `NucleotideDNA`, `NucleotideRNA`, and `AminoAcid` CodeSystems and their corresponding ValueSets are added to the Incubator. |
| **cytobandLocation bindings** | Retained in Incubator. |

### Binding Map — Current State

| Element | cg-incubator VS | cg-incubator Strength | MolDef DT VS | MolDef DT Strength | Notes |
|---|---|---|---|---|---|
| `moleculeType` | `moleculardefinition-moleculetype` | **required** | *(not bound; `type` used instead)* | — | cg-incubator targets R6's dedicated `moleculeType` element; MolDef DT uses `type` for the same purpose |
| `type` | `moleculardefinition-type` | **extensible** | `MoleculeTypeVS` | **required** | Local CS in both IGs; content is a superset; different binding element names |
| `topology` | `moleculardefinition-topology` | **required** ✅ | `TopologyVS` | **required** | Local CS in both; Incubator CS is superset (4 codes) |
| `location.sequenceLocation.strand` | `moleculardefinition-strand` | **required** | `StrandOrientationVS` | **required** | Local CS in both; aligned codes |
| `location.…coordinateSystem.origin` | `coordinatesystem-origin` | **required** | `CoordinateOriginVS` | **required** | Local CS in both; Incubator CS is superset (`#cds-start` incubator-only) |
| `location.…coordinateSystem.normalizationMethod` | `coordinatesystem-normalizationmethod` | **required** ✅ | `NormalizationMethodVS` | **required** | Local CS in both; Incubator CS is superset (`#no-normalization` incubator-only) |
| `location.…coordinateSystem.system` | LOINC `LL5323-2` | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…organism` | `moleculardefinition-organism` → NCBI Taxonomy | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…build` | LOINC `LL1040-6` | **extensible** | *(not bound)* | — | Only in cg-incubator |
| `location.cytobandLocation.…chromosome` | LOINC `LL2938-0` | **preferred** | *(not bound)* | — | Only in cg-incubator |
| `representation.focus` | `moleculardefinition-representation-focus` | **required** | *(fixed values on slices)* | — | Same 4 codes in both; MolDef DT uses fixed-value slice discriminators |
| `representation.code` | `moleculardefinition-representation-code` → RefSeq + LRG | **example** | *(not bound)* | — | Only in cg-incubator |
| `representation.literal.encoding` | `moleculardefinition-literal-encoding` | **required** | `EncodingsVS` | **required** | Local CS in both; Incubator CS is superset (`#nuc-rna-1-amb` incubator-only) |

*The `coordinateSystem.*` bindings apply identically for `representation.extracted.*` and `representation.relative.edit.*` in both IGs.*

### Code Source Strategy

Both IGs use locally-defined CodeSystems for all molecule-class coded elements. The Incubator CodeSystems are the canonical definitions; MolDef DT will reference them directly once the dependency is established.

| Concept | cg-incubator approach | MolDef DT approach | Status |
|---|---|---|---|
| Molecule type (DNA/RNA/AA) | **Local CS** `moleculardefinition-moleculetype`: `#dna`, `#rna`, `#aa` | **Local CS** `MoleculeType`: `#dna`, `#rna`, `#aa` | ✅ Aligned |
| Topology | **Local CS** `moleculardefinition-topology`: `#linear`, `#circular`, `#linear-discontiguous`, `#branched` | **Local CS** `Topology`: same 4 codes | ✅ Aligned (union) |
| Strand | **Local CS** `moleculardefinition-strand`: `#forward`, `#reverse` | **Local CS** `StrandOrientation`: `#forward`, `#reverse` | ✅ Aligned |
| Subtype (`type` element) | **Local CS** `moleculardefinition-type`: DNA/RNA subtypes | **Local CS** `MoleculeType` (same element, different R6 shape) | ✅ Aligned in approach |
| Coordinate origin | **Local CS** (Incubator) | **Local CS** (MolDef DT) | ✅ Aligned in approach; Incubator is superset |
| Normalization method | **Local CS** (Incubator) | **Local CS** (MolDef DT) | ✅ Aligned in approach; Incubator is superset |
| Encoding | **Local CS** (Incubator) | **Local CS** (MolDef DT) + character-alphabet CSes | ✅ Aligned in approach; Incubator now has char-alphabet CSes too |
| Representation focus | **Local CS** (Incubator) | **Local CS** (MolDef DT) | ✅ Aligned |

### Concept-Level Coverage — Superset Codes in Incubator

#### Coordinate Origin

| Code concept | cg-incubator CS | MolDef DT CS |
|---|---|---|
| Sequence start | ✅ `#sequence-start` | ✅ `#sequence-start` |
| CDS start | ✅ `#cds-start` (Incubator-only) | ❌ not present |
| Feature start | ✅ `#feature-start` | ✅ `#feature-start` |
| Feature end | ✅ `#feature-end` | ✅ `#feature-end` |

`#cds-start` is retained in the Incubator CodeSystem as an incubator-only concept (union rule). MolDef DT's `required` binding means it cannot currently be expressed in MolDef DT profiles.

#### Normalization Method

| Code concept | cg-incubator CS | MolDef DT CS |
|---|---|---|
| Left shift | ✅ `#left-shift` | ✅ `#left-shift` |
| Right shift | ✅ `#right-shift` | ✅ `#right-shift` |
| Fully justified | ✅ `#fully-justified` | ✅ `#fully-justified` |
| No normalization | ✅ `#no-normalization` (Incubator-only) | ❌ not present |

`#no-normalization` is retained in the Incubator CodeSystem (union rule). The Incubator binding is now also `required`, which means implementations would need to use this code when no normalization is applied.

#### Topology

| Code concept | cg-incubator CS | MolDef DT CS |
|---|---|---|
| Linear | ✅ `#linear` | ✅ `#linear` |
| Circular | ✅ `#circular` | ✅ `#circular` |
| Linear discontiguous | ✅ `#linear-discontiguous` | ✅ `#linear-discontiguous` |
| Branched | ✅ `#branched` | ✅ `#branched` |

All 4 topology codes are now present in both IGs (union achieved).

#### Representation Focus

All 4 codes are aligned in both IGs:

| Code | cg-incubator | MolDef DT |
|---|---|---|
| `allele-state` | ✅ | ✅ |
| `context-state` | ✅ | ✅ |
| `reference-state` | ✅ | ✅ |
| `alternative-state` | ✅ | ✅ |

### Encoding (Literal Representation) and Character-Alphabet CodeSystems

The Incubator now defines character-alphabet CodeSystems (`nucleotide-dna`, `nucleotide-rna`, `amino-acid`) matching those in MolDef DT, enabling validation of sequence string content character-by-character.

| Encoding category | cg-incubator VS | MolDef DT VS | Notes |
|---|---|---|---|
| Nucleotide DNA 1-letter unambiguous | `nucleotide-dna-1letter-unambiguous` | `NucleotideDNA1LetterUnambiguous` | ✅ Aligned |
| Nucleotide RNA 1-letter unambiguous | `nucleotide-rna-1letter-unambiguous` | `NucleotideRNA1LetterUnambiguous` | ✅ Aligned |
| Nucleotide DNA 1-letter with N | `nucleotide-dna-1letter-with-n` | `NucleotideDNA1LetterWithN` | ✅ Aligned |
| Nucleotide DNA 1-letter ambiguous (IUPAC) | `nucleotide-dna-1letter-ambiguous` | `NucleotideDNA1LetterAmbiguous` | ✅ Aligned |
| Nucleotide RNA 1-letter ambiguous | *(not defined)* | *(not defined)* | Omitted in both |
| AA 1-letter unambiguous (20 standard) | `amino-acid-1letter-unambiguous` | `AminoAcid1LetterUnambiguous` | ✅ Aligned |
| AA 3-letter unambiguous (20 standard) | `amino-acid-3letter-unambiguous` | `AminoAcid3LetterUnambiguous` | ✅ Aligned |
| AA 1-letter ambiguous | `amino-acid-1letter-ambiguous` | `AminoAcid1LetterAmbiguous` | ✅ Aligned |
| AA 3-letter ambiguous | `amino-acid-3letter-ambiguous` | `AminoAcid3LetterAmbiguous` | ✅ Aligned |

The encoding *category* codes in `moleculardefinition-literal-encoding` reference these ValueSets to indicate which character alphabet is valid for a given sequence string.

### Experimental Status

All locally-defined CodeSystems and ValueSets in the Incubator are marked `experimental = true`, reflecting their pre-release status. Both this IG and the MolDef DT IG are under active development and have not been formally published. Terminology content should be considered stable enough to implement against but is subject to change before an official release.

### Outstanding Items

| Item | Status |
|---|---|
| **MolDef DT dependency on Incubator** | Pending — tooling support for cross-IG terminology dependencies must be confirmed before the formal dependency declaration can be made. |
| **`#cds-start` and `#no-normalization` in MolDef DT** | These concepts are defined in the Incubator but not yet present in MolDef DT profiles. When MolDef DT inherits Incubator terminology, these codes will become expressible in MolDef DT profiles automatically. |

