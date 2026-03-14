# Terminology Considerations

This page documents design decisions, rationale, and guidance for maintainers and implementers regarding the terminology bindings used in this Implementation Guide. It is intended to be a living record updated as new bindings are established or revised.

---

## MolecularDefinition: `moleculeType`, `type`, and `topology`

### Background

Three elements on `MolecularDefinition` carry terminology bindings that required careful vocabulary alignment:

| Element | Cardinality | Datatype | Binding Strength | ValueSet |
|---|---|---|---|---|
| `moleculeType` | 0..1 | CodeableConcept | **required** | [MolecularDefinitionMoleculeType](ValueSet-moleculardefinition-moleculetype.html) |
| `type` | 0..* | CodeableConcept | **extensible** | [MolecularDefinitionType](ValueSet-moleculardefinition-type.html) |
| `topology` | 0..* | CodeableConcept | **extensible** | [MolecularDefinitionTopology](ValueSet-moleculardefinition-topology.html) |

For these three elements, codes are drawn from the **[Sequence Ontology](http://www.sequenceontology.org) (SO)**, the authoritative community-maintained vocabulary for sequence feature types and molecular attributes. SO is used here because it is the recognized external system for molecular biology vocabulary under HL7 governance policies — coining new HL7 codes for concepts that already exist in SO would duplicate authoritative external vocabulary unnecessarily.

Across the broader IG, terminology bindings use a range of external systems (SO, LOINC, NCBI Taxonomy, NCBI RefSeq, NCBI Assembly, INSDC, Ensembl, LRG) alongside locally defined CG CodeSystems for domain-specific concepts with no suitable external home. For terminology server support, see the companion genomics terminology server package.

---

### `moleculeType` — Required Binding

**Codes:** `SO:0000352` (DNA), `SO:0000356` (RNA), `SO:0000104` (polypeptide)

**Binding strength: `required`**

`moleculeType` is the primary search and profiling axis for `MolecularDefinition`. Its definition explicitly states it is "intended to facilitate searching." A `required` binding is necessary so that all conformant systems use the same codes — if systems were free to substitute synonyms, cross-system discovery would fail.

**Design note — "amino acid" vs "polypeptide":**
Earlier versions of the `moleculeType` short description used the label "amino acid." This was updated to "polypeptide" (`SO:0000104`) because:
- `SO:0001237` (`amino_acid`) represents a **single residue** in SO, with 24 children corresponding to glycine, alanine, valine, etc. This is semantically incorrect for a molecule-class descriptor.
- `SO:0000104` (`polypeptide`) — "a sequence of amino acids linked by peptide bonds" — correctly represents what this element intends.

Existing examples and profiles that used the informal label "amino acid" should migrate to `SO:0000104` with display "polypeptide."

**For implementers:** This is a `required` binding. You **must** use one of the three SO codes listed. If a molecule class genuinely cannot be represented by DNA, RNA, or polypeptide (e.g., peptide nucleic acid, oligosaccharide), raise a change request against the IG — do not locally extend this ValueSet.

---

### `type` — Extensible Binding

**Codes:** A curated enumeration of SO terms covering genomic DNA subtypes, cDNA subtypes, and RNA subtypes (mRNA, rRNA, tRNA, snRNA, snoRNA, miRNA, siRNA, lncRNA, antisense RNA, ncRNA, piRNA).

**Binding strength: `extensible`**

`type` carries "arbitrary types, based on domain semantics" per the element definition. This is the textbook use case for `extensible`: the core set of concepts is enumerated, but implementers are free to use additional codes from SO or other systems when no matching code is present.

**Why RNA subtypes are explicitly enumerated (not filter-based):**
SO's RNA subtype concepts (mRNA, tRNA, rRNA, miRNA, etc.) are **not** hierarchical descendants of `SO:0000356` (RNA). They descend from the *transcript feature* branch of SO:
- `mRNA` (SO:0000234) → parent: `mature_transcript` (SO:0000233)
- `tRNA` (SO:0000253) → parent: `sncRNA` (SO:0002247)
- `rRNA` (SO:0000252) → parent: `ncRNA` (SO:0000655)
- `miRNA` (SO:0000276) → parent: `small_regulatory_ncRNA` (SO:0000370)

`SO:0000356` (RNA) is an **attribute concept** in the `polymer_attribute` branch, not a structural parent of transcript types. An `is-a SO:0000356` filter would return zero RNA subtypes. The ValueSet therefore enumerates them explicitly. This is a known limitation of the current SO hierarchy and is documented here so future maintainers do not attempt to switch to a filter-based approach.

**Preventing semantically invalid instances:** Implementers should not repeat in `type` what is already expressed in `moleculeType`. For example, if `moleculeType = SO:0000356` (RNA), do not also set `type = SO:0000356` (RNA). A FHIRPath invariant to enforce this constraint is under consideration for a future release.

**For implementers:** Prefer codes from the enumerated ValueSet. If you use a code outside this set, it must be from SO or another recognized external system, not a locally invented plain string. When using a code outside the set, include both `system` and `code` — do not rely on `text` alone.

---

### `topology` — Extensible Binding

**Codes:** `SO:0000987` (linear), `SO:0000988` (circular)

**Binding strength: `extensible`**

These two codes are the direct children of `SO:0000986` (`topology_attribute`) in SO, which is itself defined as "the attribute of whether a nucleotide polymer is linear or circular." They represent the complete SO coverage for this concept at the time of publication.

**Key limitation — scope of SO topology_attribute:**
`SO:0000986` and its children are explicitly scoped to **nucleotide polymers**. Polypeptide topology (e.g., quaternary structure, coiled-coil) is not represented in the SO `topology_attribute` branch. The `topology` element's definition notes that "more complex entities might be branched or have a quaternary structure." If a profile needs to record polypeptide topology, a locally coined code (or a code from a protein structure vocabulary) is permissible under the `extensible` binding.

**Common topology concepts not yet in SO `topology_attribute`:**
The following biologically meaningful topology states have no code in the SO `topology_attribute` branch and would require locally coined codes:
- `supercoiled` — negatively or positively supercoiled circular DNA (bacteriophage, plasmid)
- `relaxed-circular` / `nicked-circular` — open circular form with at least one strand nick
- `branched` — branched RNA or synthetic branched nucleic acid

If any of these are needed, define them in a CG supplement CodeSystem and submit a request to SO for inclusion in a future release.

**Relationship between `type` and `topology`:**
SO contains molecule-type concepts that encode both type and topology in a single code — for example, `SO:0002292` (`circular_mRNA`) and `SO:0002291` (`circular_ncRNA`). These are children of the *molecule type* branch, not the `topology_attribute` branch.

The recommended pattern in this IG is to keep these dimensions **orthogonal**:
- Use `moleculeType` + `type` to express what the molecule *is* (e.g., RNA → mRNA)
- Use `topology` to express its structural form (e.g., circular)

Avoid using `type = SO:0002292` (circular_mRNA) in place of `type = SO:0000234` (mRNA) + `topology = SO:0000988` (circular). The orthogonal representation supports more precise querying — for example, a search for all circular molecules (`topology = circular`) would not match instances that encoded circularity only in the `type` field.

---

## MolecularDefinition: `strand`

`strand` qualifies a `sequenceLocation` with an orientation, relevant when the interval alone is insufficient to identify a location (e.g., on a double-stranded DNA molecule). The element previously had no binding and only mentioned forward/reverse as examples in free text.

### Design Decision: Sequence Ontology `direction_attribute` codes, Required Binding

SO provides exactly two leaf concepts under `SO:0001029` (direction_attribute) that match this domain:

| SO Code | Display | Equivalent Terms |
|---|---|---|
| `SO:0001030` | `forward` | plus strand, sense strand, Watson strand |
| `SO:0001031` | `reverse` | minus strand, antisense strand, Crick strand |

SO also defines `SO:0002262` (Watson_strand) and `SO:0002263` (Crick_strand), but these are classified under `SO:0000830` (chromosome_part) — they model strands as structural components of a chromosome, not as orientation attributes of a feature. They are therefore semantically inappropriate for an orientation-qualifying element and are excluded.

**Why `required`?** Strand is a binary, closed domain. There is no valid third option for a nucleotide polymer location. A `required` binding ensures that systems can reliably interpret orientation without ambiguity and prevents free-text drift (e.g., "+", "plus", "sense", "watson" all appearing in the wild for the same concept).

The `description` on the binding explicitly maps forward → plus/sense/Watson and reverse → minus/antisense/Crick so that implementers translating from VCF (`+`/`-`), GFF3 (`+`/`-`/`.`), or GA4GH conventions can identify the correct code without consulting SO directly.

---

## MolecularDefinition: `coordinateSystem` Child Elements

The `coordinateSystem` backbone element appears three times within `MolecularDefinition` (under `location.sequenceLocation.coordinateInterval`, `representation.extracted.coordinateInterval`, and `representation.relative.edit.coordinateInterval`). Each instance carries three vocabulary-bearing child elements. The same bindings apply uniformly to all three instances.

### `system` — Extensible Binding (LOINC LL5323-2, preserved)

`system` describes the counting convention: whether positions are 0-based or 1-based, and whether the boundary model is character-based or interval-based. An existing `extensible` binding to the LOINC answer list [LL5323-2](http://loinc.org/vs/LL5323-2) is preserved unchanged. The four LOINC codes map directly to widely-used genomic systems:

| LOINC Code | Display | Real-World Systems |
|---|---|---|
| `LA30100-4` | 0-based interval counting | UCSC BED, GA4GH VRS, SPDI |
| `LA30101-2` | 0-based character counting | Less common; used in some REST APIs |
| `LA30102-0` | 1-based character counting | HGVS (c./g./n./p.), RefSeq accession positions |
| `LA30103-8` | 1-based interval counting | VCF variants (loosely) |

The `bindingName` was corrected from the LOINC answer list ID (`LL5323-2`) to the semantic name `CoordinateSystemType`. No other changes were made to the binding and no new ValueSet is introduced for this element.

**Why preserve LOINC rather than coining CG codes?** The LOINC LL5323-2 answer list is already used in genomics-reporting and other CG IGs. Preserving it maintains cross-IG consistency and avoids fragmenting terminology for the same concept.

### `origin` — Required Binding (CG-defined CodeSystem)

`origin` identifies the reference landmark from which coordinate distances are measured. This concept is entirely domain-specific to clinical genomics; no external vocabulary (LOINC, SO, SNOMED CT) provides suitable codes. A new CG-defined CodeSystem `coordinatesystem-origin` is introduced with four concepts:

| Code | Display | Typical Standard |
|---|---|---|
| `sequence-start` | Sequence Start | Genomic/chromosomal coordinates (NC_, NG_, NT_ accessions) |
| `cds-start` | CDS Start (A of ATG) | HGVS c. (coding DNA) |
| `feature-start` | Feature Start | HGVS exon-relative positions |
| `feature-end` | Feature End | HGVS intronic positions (e.g., c.100+3) |

**Why `required`?** Origin ambiguity is a patient safety concern. Identical numeric coordinates carry entirely different biological meanings depending on origin — for example, the same integer under `cds-start` versus `sequence-start` conventions may differ by hundreds or thousands of bases. A `required` binding ensures that any system receiving a `MolecularDefinition` instance can unambiguously interpret the coordinates without relying on out-of-band context. This is distinct from `normalizationMethod`, where uncertainty is confined to repetitive regions and has less impact on safety-critical interpretation.

**On extensibility:** The `required` binding permits no codes outside this set. If future molecular types require new origin concepts (e.g., mitochondrial origins, protein feature-relative origins), the CodeSystem should be extended rather than using `text`-only fallbacks.

### `normalizationMethod` — Extensible Binding (CG-defined CodeSystem)

`normalizationMethod` captures which alignment convention was applied to resolve positional ambiguity in repetitive sequence regions. Like `origin`, no external vocabulary provides these codes. A new CG-defined CodeSystem `coordinatesystem-normalizationmethod` is introduced with four concepts:

| Code | Display | Typical Standard |
|---|---|---|
| `left-shift` | Left Shift | VCF |
| `right-shift` | Right Shift | HGVS (3' rule for duplications and insertions) |
| `fully-justified` | Fully Justified | VOCA, GA4GH |
| `no-normalization` | No Normalization | As-reported from caller |

**Why `extensible` rather than `required`?** Normalization methodology is evolving and implementation-context-dependent. New algorithmic conventions may emerge (e.g., strand-aware approaches, protein-level normalization). An `extensible` binding ensures common cases are covered with standard codes while permitting new codes via `text` or local CodeSystems for methods not yet enumerated.

**Why record normalization at all?** The same variant can occupy different positions when different normalization methods are applied to a repeat region. Systems that aggregate or compare variants across databases must know the normalization convention to correctly determine equivalence. Recording this in a structured, coded field rather than free text is critical for machine-interpretable genomic data exchange.

---

## MolecularDefinition: `genomeAssembly` Child Elements

The `genomeAssembly` backbone element is nested under `location.cytobandLocation` and identifies the reference genome for cytoband-based coordinates. Three of its four child elements are vocabulary-bearing.

### `organism` — Extensible Binding (NCBI Taxonomy)

`organism` identifies the species whose genome the assembly represents. The authoritative vocabulary is the [NCBI Taxonomy database](https://www.ncbi.nlm.nih.gov/taxonomy), which assigns a numeric taxon ID to every organism with curated sequence data. The FHIR system URI is `http://www.ncbi.nlm.nih.gov/taxonomy`.

The binding is **extensible** because any organism with a sequenced genome is a legitimate value — restricting to an enumerated set would exclude non-human organisms (mouse, rat, zebrafish, etc.) without scientific justification. Commonly used values:

| Taxon ID | Organism |
|---|---|
| `9606` | *Homo sapiens* |
| `10090` | *Mus musculus* |
| `10116` | *Rattus norvegicus* |
| `7955` | *Danio rerio* (zebrafish) |

Note: NCBI Taxonomy does not publish a FHIR CodeSystem. Systems using this binding should include the numeric taxon ID as `code` and the scientific name as `display`.

### `build` — Extensible Binding (LOINC LL1040-6)

`build` records the named genome assembly (e.g., GRCh38, GRCh37). The LOINC answer list [LL1040-6](https://loinc.org/LL1040-6/) is already used for the equivalent `component[reference-sequence-assembly]` element in the genomics-reporting IG. Reusing it here maintains cross-IG consistency.

The binding is **extensible** to accommodate assemblies not yet represented in LL1040-6, most notably T2T-CHM13v2.0 (Telomere-to-Telomere). As new reference assemblies gain adoption, they may be added to LL1040-6 or recorded using a locally defined code with appropriate `display` text.

**Why not NCBI Assembly accession here?** The `build` element is intended for the human-readable assembly name; the corresponding machine-readable accession belongs in the sibling `accession` element. Keeping these separate supports both human-facing display and programmatic lookup.

### `accession` — Example Binding (NCBI Assembly)

`accession` records the versioned machine-readable NCBI Assembly accession for the genome assembly. Unlike `build`, which carries a human-recognizable name, `accession` is intended for programmatic lookup and precise version identification.

NCBI Assembly accessions follow one of two patterns:

| Prefix | Registry | Example | Assembly |
|--------|----------|---------|----------|
| `GCF_` | RefSeq (curated) | `GCF_000001405.40` | GRCh38.p14 |
| `GCF_` | RefSeq (curated) | `GCF_000001405.26` | GRCh38 (initial) |
| `GCF_` | RefSeq (curated) | `GCF_000001405.13` | GRCh37.p13 |
| `GCA_` | GenBank | `GCA_009914755.4` | T2T-CHM13v2.0 |

The conventional FHIR system URI for NCBI Assembly is `http://www.ncbi.nlm.nih.gov/assembly`. There is no enumerable FHIR CodeSystem or ValueSet for NCBI Assembly accessions — the identifier space is open and version-bearing — so a ValueSet-based binding is not appropriate.

The binding strength is **example**, which:
- Documents the expected identifier system to implementers
- Does not constrain the full `CodeableConcept` (allowing text-only or locally coded variants)
- Is consistent with FHIR guidance for open, continuously-updated identifier namespaces

---

## MolecularDefinition: cytobandInterval.chromosome

### `chromosome` — Preferred Binding (LOINC LL2938-0)

`chromosome` identifies which human chromosome a cytoband interval is located on. The binding uses LOINC answer list [LL2938-0](https://loinc.org/LL2938-0/), the standard LOINC answer list for human chromosomes.

| Category | Chromosomes | LOINC code range |
|----------|-------------|------------------|
| Autosomes | 1–22 | LA21254-0 – LA21275-5 |
| Sex chromosomes | X, Y | LA21276-3, LA21277-1 |
| Mitochondrial | M | LA25391-0 |

The binding strength is **preferred** — consistent with how chromosome is bound in the genomics-reporting IG (e.g., `component[chromosome-studied]`). Using the same answer list maintains cross-IG consistency and allows consumers to correlate cytoband locations with sequencing data from both IGs.

**Why preferred, not required?** Chromosome naming conventions occasionally diverge (e.g., "chr1" vs "1" vs "Chromosome 1"), and some jurisdictions or laboratory systems use local codes. `preferred` strongly directs implementers toward LOINC LL2938-0 while accommodating legitimate local practice.

---

## MolecularDefinition: representation.focus

### `representation.focus` — Required Binding (CG CodeSystem)

`representation.focus` classifies the role of a given representation within a `MolecularDefinition` instance. When a single instance carries multiple representations (e.g., both a reference allele and an observed variant), this element is the machine-readable discriminator that tells a consuming system what each representation describes.

The element type is `code` (not `CodeableConcept`), and uses a locally defined CG CodeSystem with four codes:

| Code | Display | Meaning |
|------|---------|----------|
| `allele-state` | Allele State | The specific allele being described at this location |
| `reference-state` | Reference State | The reference sequence at this location (as in a reference genome) |
| `alternative-state` | Alternative State | An alternate sequence at this location (e.g., an observed variant allele) |
| `context-state` | Context State | The surrounding genomic sequence context that frames the allele |

The distinction between `reference-state` and `allele-state` is intentional: `reference-state` always refers to the canonical reference genome sequence, while `allele-state` is the experimentally observed or clinically reported allele (which may or may not differ from the reference). `context-state` is used when a representation captures broader flanking sequence rather than the allele itself — for example, the full genomic region submitted for variant calling.

**Why required?** The element type is `code` (a primitive), and the set of focus roles is closed. Allowing unconstrained codes would undermine the discriminator function of this element. If future use cases require additional focus roles, the CodeSystem and ValueSet will be updated.

---

## MolecularDefinition: representation.code

### `representation.code` — Example Binding (Sequence Identifier Systems)

`representation.code` carries a coded identifier for the molecular entity described by a `representation` instance. The `0..*` cardinality allows the same molecule to be cross-referenced in multiple databases within a single representation.

The dominant use case is a versioned **NCBI RefSeq accession**, consistent with usage in the genomics-reporting IG and the cg-incubator examples. Versioned accessions are strongly preferred over unversioned ones to ensure stable, unambiguous identification of a specific sequence version.

#### Common accession types and their RefSeq prefixes

| Prefix | Sequence type | Example | Meaning |
|--------|--------------|---------|----------|
| `NC_` | Chromosomal genomic | `NC_000010.11` | Chromosome 10, GRCh38.p14 |
| `NG_` | RefSeqGene (genomic) | `NG_008384.3` | CYP2C19 locus |
| `NM_` | mRNA (coding transcript) | `NM_000769.4` | CYP2C19 mRNA |
| `NR_` | Non-coding RNA | `NR_024540.1` | |
| `NP_` | Protein | `NP_000760.1` | CYP2C19 protein |
| `NT_`/`NW_` | Genomic contig/scaffold | `NT_009237.19` | |

#### Identifier systems included in the example binding

| System URI | Database | Notes |
|-----------|----------|-------|
| `http://www.ncbi.nlm.nih.gov/refseq` | NCBI RefSeq | Curated, versioned; primary system for clinical genomics |
| `http://insdc.org` | INSDC (GenBank / EMBL / DDBJ) | International nucleotide sequence collaboration; unversioned IDs also exist |
| `http://www.ensembl.org` | Ensembl | Common in research/computational genomics (ENSG, ENST, ENSP) |
| `http://www.lrg-sequence.org` | LRG | Locus Reference Genomic — stable reference sequences for clinical variant reporting, common in European labs |

**Why example, not extensible or required?** `representation.code` is an open identifier element — any public sequence database producing stable accession identifiers is a valid source. An `example` binding documents the most commonly used systems without falsely constraining a genuinely open identifier space. The `0..*` cardinality further signals that multiple systems may co-exist on a single representation.

**Why not HGVS expressions here?** HGVS expressions (e.g., `NC_000010.11:g.94762681C>T`) are variant notations that incorporate both a reference accession and a variant description. They are valid in `representation.code` as coded values, using a system URI such as `http://varnomen.hgvs.org`, but are typically better expressed in `representation.literal` or via dedicated variant representation patterns.

---

## GenomicStudy: Terminology Bindings

GenomicStudy carries a wider range of terminology bindings than MolecularDefinition, reflecting its role as a study-level metadata resource that bridges clinical lab systems, bioinformatics pipelines, and clinical reporting. The full binding inventory is:

| Element | Strength | ValueSet / System |
|---|---|---|
| `status` | **required** | `genomicstudy-status` (incubator CS) |
| `type` | **example** | `genomicstudy-type` (incubator CS) |
| `analysis.methodType` | **preferred** | `genomicstudy-methodtype` (incubator CS) |
| `analysis.changeType` | **preferred** | `genomicstudy-changetype` (incubator CS, SO-based) |
| `analysis.genomeBuild` | **extensible** | LOINC [LL1040-6](https://loinc.org/LL1040-6/) |
| `analysis.genomicSourceClass` | **extensible** | LOINC [LL378-1](https://loinc.org/LL378-1/) |
| `analysis.regionsStudied` | **extensible** | [hgnc-vs](https://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs) (GRIG) |
| `analysis.regionsCalled` | **extensible** | [hgnc-vs](https://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs) (GRIG) |
| `analysis.regionsUncalled` | **extensible** | [hgnc-vs](https://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs) (GRIG) |
| `analysis.input.type` | **example** | `genomicstudy-dataformat` (incubator CS) |
| `analysis.output.type` | **example** | `genomicstudy-dataformat` (incubator CS) |

---

### `status` — Required Binding (incubator CodeSystem)

`status` is a modifier element. It must be `required` so that systems can reliably interpret whether a study record is valid (`available`), provisional (`registered`), voided (`entered-in-error`), or withdrawn (`cancelled`). Only systems that can enumerate and validate the full set can safely process GenomicStudy resources.

---

### `type` — Example Binding (incubator CodeSystem)

`type` is free-form in clinical practice — laboratory study types vary widely across institutions and assay generations. An `example` binding provides guidance without constraining implementation. Feedback from ballot reviewers is expected to inform whether a `preferred` binding is appropriate in a future version.

---

### `analysis.methodType` — Preferred Binding (incubator CodeSystem)

The `methodType` ValueSet draws from [NCBI-GTR major method category, method category, and primary methodology](https://www.ncbi.nlm.nih.gov/gtr/) lists, supplemented by CG Workgroup discussion. The binding was upgraded from `example` to **preferred** to signal to implementers that this is the expected code space and to encourage convergence, while stopping short of `required` because:

- The GTR list is large (~50+ codes) and not all are relevant to all institutions.
- New assay technologies emerge regularly (e.g., long-read sequencing, optical genome mapping) and the ValueSet evolves accordingly.
- Different laboratories may use proprietary method classifications that do not map cleanly to GTR terms.

**Design note — relationship to GRIG STU3:** The genomics-reporting IG STU3 uses the CodeSystem `http://hl7.org/fhir/uv/genomics-reporting/CodeSystem/genomic-study-method-type-cs`. The incubator uses `http://hl7.org/fhir/uv/cg-incubator/CodeSystem/genomicstudy-methodtype`. The codes are intended to be equivalent — the incubator CS was established before the cross-IG alignment was settled. Future normalization toward a single canonical CS URL should be tracked.

---

### `analysis.changeType` — Preferred Binding (incubator CodeSystem, SO-based)

The `changeType` ValueSet enumerates variant consequence types drawn from [Sequence Ontology](http://www.sequenceontology.org/). The binding was upgraded from `example` to **preferred** for the same rationale as `methodType`.

**Why Sequence Ontology here?** Variant consequence classification is SO's core domain. The dominant codes (SNV, MNV, delins, CNV, gene_fusion, transcript_variant) have stable, widely recognized SO identifiers. Using SO avoids coining incubator-local codes for concepts with established SO terms, consistent with general guidance in the [_General Guidance_](#general-guidance-for-future-bindings) section.

**Why not SNOMED CT?** SNOMED CT has mutation type concepts, but they are less granular at the molecular level, carry licensing encumbrances in some jurisdictions, and are less consistently used in bioinformatics toolchains than SO.

---

### `analysis.genomeBuild` — Extensible Binding (LOINC LL1040-6)

This reuses the same LOINC answer list used throughout the genomics-reporting IG for `component[reference-sequence-assembly]`. Cross-IG consistency is the primary rationale. The binding is **extensible** because:

- New assemblies (e.g., T2T-CHM13v2.0) may not yet appear in LL1040-6.
- Non-human organisms use entirely different assembly naming conventions.

---

### `analysis.genomicSourceClass` — Extensible Binding (LOINC LL378-1)

`genomicSourceClass` records whether the genome analyzed is germline (`LA6683-2`) or somatic (`LA6684-0`). This binding is directly aligned with `component[genomic-source-class]` in the genomics-reporting IG, which uses the same LOINC answer list.

**Why LOINC, not a local code?** The LOINC answer list LL378-1 is already the de facto standard for this field in clinical genomics FHIR exchange, used consistently in GRIG and elsewhere. Reusing it preserves cross-IG interoperability and allows a downstream DiagnosticReport to correlate the study-level source class with observation-level source class without code translation.

**Why extensible, not required?** LL378-1 contains a small number of additional codes (e.g., `LA6685-7` Prenatal, `LA10429-1` Fetal) that may be relevant in specific clinical contexts. `extensible` permits use of the full answer list while still accommodating edge cases.

---

### `analysis.regionsStudied`, `regionsCalled`, `regionsUncalled` — Extensible Binding (HGNC ValueSet, from GRIG)

These three elements share an identical `extensible` binding to the [hgnc-vs](https://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs) ValueSet, which is defined and maintained in the genomics-reporting IG.

#### Datatype: CodeableReference

All three use the `CodeableReference(DocumentReference)` datatype, which has two sides:

- **`concept`** — a `CodeableConcept` bound to the HGNC ValueSet, for expressing a specific gene by its HGNC identifier.
- **`reference`** — a `Reference(DocumentReference)`, for pointing to a BED file describing genomic coordinates.

Both sides may appear within the same analysis, in separate element instances. The binding applies only to the `concept` side; the `reference` side is constrained to `DocumentReference` by the type profile.

#### Why HGNC for the concept side?

[HGNC](https://www.genenames.org/) (HUGO Gene Nomenclature Committee) is the authoritative source of approved human gene symbols and identifiers. Using HGNC gene identifiers (e.g., `HGNC:76` for ABL1) is clearly preferable to using gene symbols alone (which are subject to change) or NCBI Gene IDs (which are database-internal identifiers). HGNC identifiers are stable, publicly available, and already used throughout the genomics-reporting IG for gene-quantified observations.

#### Cross-IG dependency

The `hgnc-vs` ValueSet is defined at `http://hl7.org/fhir/uv/genomics-reporting/ValueSet/hgnc-vs` — it is owned by GRIG, not the incubator. This creates a cross-IG dependency. Implementations using the incubator IG must therefore also include the genomics-reporting package as a dependency for terminology resolution. This dependency is declared in `sushi-config.yaml`.

If this dependency becomes undesirable in a future release (e.g., version skew between the two IGs), the incubator could mint its own equivalent HGNC ValueSet. For now, reuse is preferred to avoid duplication.

#### `regionsUncalled` — new element

`regionsUncalled` has no equivalent in the FHIR R5/R6 core GenomicStudy resource but was present as an extension in the GRIG STU3 Procedure backport profiles. It is introduced here as a first-class element to explicitly document gaps in analytical coverage. Uncallable regions should always be recorded when known, to allow clinical consumers to distinguish a true negative result from an untested region — a distinction critical for variant interpretation.

---

### `analysis.input.type` and `analysis.output.type` — Example Binding (incubator CodeSystem)

File format codes (BAM, CRAM, FASTQ, VCF, MAF, BED, etc.) are drawn from the [Integrative Genomics Viewer (IGV) documentation](https://software.broadinstitute.org/software/igv/FileFormats) by Broad Institute. The binding is **example** because:

- New bioinformatics file formats emerge regularly.
- Different pipeline toolchains may use proprietary or extended format variants not covered by IGV's list.
- The primary use of this element is human-readable labelling of file references, not machine-executable dispatch logic.

---

### Unbound terminology-bearing elements

Two elements in GenomicStudy carry `CodeableConcept` values but have no formal ValueSet binding in the current StructureDefinition:

| Element | Suggested External System | Notes |
|---|---|---|
| `analysis.performer.role` | HL7 v3 ParticipationType (`http://terminology.hl7.org/CodeSystem/v3-ParticipationType`) | Code `PRF` (Performer) used in examples. A binding to `http://hl7.org/fhir/ValueSet/performer-role` is a candidate for a future version. |
| `analysis.device.function` | LOINC | Code `LA26398-0` (Sequencing) used in examples. No standard VS for device function exists in the CG domain; a future binding should be considered once common patterns stabilize. |

Implementers should use `http://terminology.hl7.org/CodeSystem/v3-ParticipationType` for performer roles and LOINC for device function in the interim.

---

## General Guidance for Future Bindings

1. **Prefer an established external vocabulary before coining new CG codes.** Check SO for molecular/sequence feature concepts, LOINC for clinical lab and observation concepts, and NCBI systems (Taxonomy, RefSeq, Assembly) for biological entity identifiers. Only define a local CG CodeSystem when no suitable external vocabulary covers the concept — as done for `coordinateSystem.origin` and `coordinateSystem.normalizationMethod`.
2. **Use `required` only for classification axes that must be stable for search.** Once `required`, adding new codes is a breaking change for existing systems that must validate. Plan the code set carefully.
3. **Use `extensible` for domain-semantic or interpretive concepts.** If implementers will legitimately need codes beyond the curated set, `extensible` is correct. Document the expected extension pattern.
4. **Never use `preferred` for codes where external system coverage is known and complete.** `preferred` signals that alternatives are acceptable without penalty. If an established external vocabulary provides the right codes, `required` or `extensible` (depending on stability needs) is more appropriate than `preferred`, which would allow silent substitution and fragment interoperability.
5. **Document SO hierarchy limitations explicitly.** As seen with RNA subtypes and topology scope, SO's hierarchy does not always match biological intuition. Always verify parent/child relationships in the local SO CodeSystem before writing a filter-based ValueSet `include`.
6. **Monitor SO for code deprecation.** SO codes can be deprecated or replaced over time. When updating SO-bound ValueSets, confirm that enumerated codes have not been deprecated or replaced (check the `obsoletedBy` property in the SO CodeSystem).
