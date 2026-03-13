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

All three ValueSets draw their primary codes from the **[Sequence Ontology](http://www.sequenceontology.org) (SO)**, the authoritative community-maintained vocabulary for sequence feature types and molecular attributes. For terminology server support, see the companion genomics terminology server package.

---

### Design Decision: Single Authoritative External System (SO)

Rather than creating a CG-specific CodeSystem mirroring SO concepts, this IG binds directly to SO. The rationale:

- SO is a recognized external system under HL7 vocabulary governance policies. Coining new HL7 codes for concepts that already exist in SO would violate the principle of not duplicating authoritative external vocabulary.
- SO is actively maintained and versioned. Binding to it allows the IG to benefit from SO additions without requiring IG republication.

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

## General Guidance for Future Bindings

1. **Prefer SO for sequence and molecular feature concepts.** SO is the correct home for most vocabulary needed by `MolecularDefinition`. Check SO before coining new codes.
2. **Use `required` only for classification axes that must be stable for search.** Once `required`, adding new codes is a breaking change for existing systems that must validate. Plan the code set carefully.
3. **Use `extensible` for domain-semantic or interpretive concepts.** If implementers will legitimately need codes beyond the curated set, `extensible` is correct. Document the expected extension pattern.
4. **Never use `preferred` for codes where external system coverage is known and good.** `preferred` signals that alternatives are acceptable. If SO provides the right code, `required` or `extensible` (depending on stability needs) is more appropriate than `preferred`, which would allow silent substitution.
5. **Document SO hierarchy limitations explicitly.** As seen with RNA subtypes and topology scope, SO's hierarchy does not always match biological intuition. Always verify parent/child relationships in the local SO CodeSystem before writing a filter-based ValueSet `include`.
6. **Monitor SO for code deprecation.** SO codes can be deprecated or replaced over time. When updating SO-bound ValueSets, confirm that enumerated codes have not been deprecated or replaced (check the `obsoletedBy` property in the SO CodeSystem).
