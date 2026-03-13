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

## General Guidance for Future Bindings

1. **Prefer SO for sequence and molecular feature concepts.** SO is the correct home for most vocabulary needed by `MolecularDefinition`. Check SO before coining new codes.
2. **Use `required` only for classification axes that must be stable for search.** Once `required`, adding new codes is a breaking change for existing systems that must validate. Plan the code set carefully.
3. **Use `extensible` for domain-semantic or interpretive concepts.** If implementers will legitimately need codes beyond the curated set, `extensible` is correct. Document the expected extension pattern.
4. **Never use `preferred` for codes where external system coverage is known and good.** `preferred` signals that alternatives are acceptable. If SO provides the right code, `required` or `extensible` (depending on stability needs) is more appropriate than `preferred`, which would allow silent substitution.
5. **Document SO hierarchy limitations explicitly.** As seen with RNA subtypes and topology scope, SO's hierarchy does not always match biological intuition. Always verify parent/child relationships in the local SO CodeSystem before writing a filter-based ValueSet `include`.
6. **Version-pin SO includes.** Each ValueSet pins `<version value="2026-03-02"/>`. When updating, confirm that existing codes have not been deprecated or replaced (check the `obsoletedBy` property in the SO CodeSystem).
