## Notes

### Encodings

Molecular sequences are represented using numerous encodings, which are not always explicitly specified. The `representation.literal.encoding` attribute captures this information directly, so that implementors can validate the content of messages and computationally determine how a particular sequence should be interpreted.

The examples below illustrate different encodings, which could be used to create terms for this attribute. They are based on the IUPAC symbols for nucleotide and amino acid sequences.

- [IUPAC symbols for nucleotide sequences](https://www.bioinformatics.org/sms/iupac.html)
- [IUPAC symbols for amino acid sequences (1-letter code)](https://iupac.qmul.ac.uk/AminoAcid/AA1n2.html#AA1)
- [IUPAC symbols for nucleotide sequences (IUBMB)](https://iubmb.qmul.ac.uk/misc/naseq.html#tab1)
- [IUPAC symbols for amino acid sequences (3-letter code)](https://iupac.qmul.ac.uk/AminoAcid/A2021.html)

#### Nucleotide Symbols (1-letter, no ambiguity, DNA residues)

| Symbol | Meaning | Origin of designation |
|--------|---------|----------------------|
| G | Guanine | G |
| A | Adenine | A |
| T | Thymine | T |
| C | Cytosine | C |

#### Nucleotide, 1-letter, no ambiguity, RNA residues

| Symbol | Meaning | Origin of designation |
|--------|---------|----------------------|
| G | Guanine | G |
| A | Adenine | A |
| U | Uracil | U |
| C | Cytosine | C |

#### Nucleotide Symbols (1-letter, no ambiguity except N, DNA residues)

| Symbol | Meaning | Origin of designation |
|--------|---------|----------------------|
| G | Guanine | G |
| A | Adenine | A |
| T | Thymine | T |
| C | Cytosine | C |
| N | G or A or T or C | aNy |

#### Nucleotide Symbols (1-letter, with ambiguity, DNA residues)

| Symbol | Meaning | Origin of designation |
|--------|---------|----------------------|
| G | Guanine | G |
| A | Adenine | A |
| T | Thymine | T |
| C | Cytosine | C |
| R | G or A | puRine |
| Y | T or C | pYrimidine |
| M | A or C | aMino |
| K | G or T | Keto |
| S | G or C | Strong interaction (3 H bonds) |
| W | A or T | Weak interaction (2 H bonds) |
| H | A or C or T | not-G, H follows G in the alphabet |
| B | G or T or C | not-A, B follows A |
| V | G or C or A | not-T (not-U), V follows U |
| D | G or A or T | not-C, D follows C |
| N | G or A or T or C | aNy |

#### Amino Acid Symbols (1-letter, no ambiguity, 20 common)

| Symbol | Amino acid |
|--------|-----------|
| A | alanine |
| C | cysteine |
| D | aspartic acid |
| E | glutamic acid |
| F | phenylalanine |
| G | glycine |
| H | histidine |
| I | isoleucine |
| K | lysine |
| L | leucine |
| M | methionine |
| N | asparagine |
| P | proline |
| Q | glutamine |
| R | arginine |
| S | serine |
| T | threonine |
| V | valine |
| W | tryptophan |
| Y | tyrosine |

#### Amino Acid Symbols (3-letter, no ambiguity, 20 common)

| Symbol | Amino acid |
|--------|-----------|
| Ala | alanine |
| Cys | cysteine |
| Asp | aspartic acid |
| Glu | glutamic acid |
| Phe | phenylalanine |
| Gly | glycine |
| His | histidine |
| Ile | isoleucine |
| Lys | lysine |
| Leu | leucine |
| Met | methionine |
| Asn | asparagine |
| Pro | proline |
| Gln | glutamine |
| Arg | arginine |
| Ser | serine |
| Thr | threonine |
| Val | valine |
| Trp | tryptophan |
| Tyr | tyrosine |

#### Amino Acid Symbols (1-letter, with ambiguity)

| Symbol | Amino acid |
|--------|-----------|
| A | alanine |
| B | aspartic acid or asparagine |
| C | cysteine |
| D | aspartic acid |
| E | glutamic acid |
| F | phenylalanine |
| G | glycine |
| H | histidine |
| I | isoleucine |
| K | lysine |
| L | leucine |
| M | methionine |
| N | asparagine |
| P | proline |
| Q | glutamine |
| R | arginine |
| S | serine |
| T | threonine |
| U | selenocysteine |
| V | valine |
| W | tryptophan |
| X | unknown or 'other' amino acid |
| Y | tyrosine |
| Z | glutamic acid or glutamine |

#### Amino Acid Symbols (3-letter, with ambiguity)

| Symbol | Amino acid |
|--------|-----------|
| Ala | alanine |
| Asx | aspartic acid or asparagine |
| Cys | cysteine |
| Asp | aspartic acid |
| Glu | glutamic acid |
| Phe | phenylalanine |
| Gly | glycine |
| His | histidine |
| Ile | isoleucine |
| Lys | lysine |
| Leu | leucine |
| Met | methionine |
| Asn | asparagine |
| Pro | proline |
| Gln | glutamine |
| Arg | arginine |
| Ser | serine |
| Thr | threonine |
| Sec | selenocysteine |
| Val | valine |
| Trp | tryptophan |
| Xaa | unknown or 'other' amino acid |
| Tyr | tyrosine |
| Glx | glutamic acid or glutamine |

### Molecular Representations

The Molecular Definition resource supports several different methods for representing a molecule. Some of the elements described below may apply only to sequences, and different elements may be added to support other types of molecular concepts.

Native representations: The literal, code, and resolvable are native representations, meaning they represent a sequence "as-is" without any additional computation.

Derived representations: The extracted, concatenated, repeated, and relative representations are derived representations, meaning they require one or more computational operations to be performed to create the sequence that is being represented.

#### Literal

The literal element can be used to represent a sequence as a string of characters. By convention, nucleotide sequences are expressed 5' to 3' and protein sequences are expressed N to C terminus. The encoding element can optionally be used to specify the encoding used for the sequence literal. The encoding can be important in disambiguating sequences that share alphabets (for example, ATG might represent a translation start codon in DNA, but it could also represent a peptide containing 3 amino acids).

#### Code

The `representation.code` element (`0..*`) can be used to identify a molecular entity by a coded accession or identifier from a known sequence database. The `0..*` cardinality allows the same molecule to be cross-referenced using identifiers from multiple databases within a single representation.

The `system`, `code`, and `display` elements of the `Coding` datatype should be used to fully identify the sequence: `system` carries the database URI, `code` carries the accession identifier, and `display` carries a human-readable description.

The most common use case in clinical genomics is a **versioned NCBI RefSeq accession** (system: `http://www.ncbi.nlm.nih.gov/refseq`). Versioned accessions (those including a dot-version suffix, e.g., `NC_000010.11`) are strongly preferred over unversioned ones to ensure stable, unambiguous identification of a specific sequence version. Commonly used accession types:

| Prefix | Type | Example |
|--------|------|---------|
| `NC_` | Chromosomal genomic | `NC_000010.11` |
| `NG_` | RefSeqGene | `NG_008384.3` |
| `NM_` | mRNA transcript | `NM_000769.4` |
| `NR_` | Non-coding RNA | `NR_024540.1` |
| `NP_` | Protein | `NP_000760.1` |

Other recognized sequence identifier systems include INSDC/GenBank (`http://insdc.org`), Ensembl (`http://www.ensembl.org`), and LRG (`http://www.lrg-sequence.org`). An example binding documenting these systems is defined in the [Terminology Considerations](terminology-considerations.html) page.

Note that `representation.code` does not guarantee that the repository is publicly accessible or that the referenced sequence can be retrieved — it only identifies the sequence using a code that can be exchanged. A private biobank accession that follows a known system scheme is a valid use of this element. For publicly accessible files, prefer `representation.resolvable` (which implies the content can be retrieved).

#### Resolvable

The resolvable element can be used to represent a sequence by reference, but it also implies that the sequence is accessible and SHOULD be resolvable (although a security layer may be present). This element makes use of the Document Reference resource, which contains the content.attachment element. The Attachment datatype can be used to represent sequences that are captured as a formatted file (using .contentType and .data) or as a URL (using .contentType and .url).

#### Extracted

The extracted element can be used to represent a sequence that is derived from another, longer sequence. The startingMolecule element refers to the "parent" sequence, and is itself an instance of Molecular Definition (with its own representation). The coordinateInterval element specifies a precise interval on the "parent" sequence, which is to be extracted (conceptually or literally) and optionally reverse-complemented. This element provides a way to conveniently reference regions of very long molecules (e.g., chromosomes) without requiring either the "parent" or the extracted sequence to be serialized. Conceptually, this representation is the inverse operation of the concatenated representation.

#### Concatenated

The concatenated element can be used to represent a sequence that is comprised of other sequences that are concatenated together to form the intended sequence. Each sequenceElement is specified as an instance of Molecular Definition (and each has its own representation). The order of concatenation is explicitly defined using the ordinalIndex element. Conceptually, this representation is the inverse operation of the extracted representation.

#### Repeated

The repeated element can be used to represent a sequence that is comprised of a sequence motif that is repeated a specified number of times. The sequenceMotif is an instance of Molecular Definition (and has its own representation), and copyCount specifies the number of times the motif is copied in tandem. Conceptually, this representation is a special case of the concatenated representation, where each element is an identical copy of a given motif.

#### Relative

The relative element can be used to represent a sequence in relation to another sequence, where the difference between the two sequences can be expressed as an ordered series of edit operations. This representation can be used to conveniently represent minor but meaningful differences between long or complex sequences (e.g., HLA alleles). Algorithmically, the relative representation defines a sequence by beginning with a startingMolecule (an instance of Molecular Definition) and performing at least one edit operation on it. Each edit operation is performed in order and includes replacing the sequence (the replacedMolecule) at a defined coordinateInterval with the sequence specified by the replacementMolecule. The resulting sequence after all edits have been performed is the sequence referenced by this representation element.

Note that the edits specified in this representation are operations and NOT variations. Variations are defined as a specific comparison between two states (a reference and an alternative), and while they are sometimes called "changes" and therefore they might be confused for edit operations, they are semantically distinct concepts.

### Combining Representations

Since the derived representations (extracted, concatenated, repeated, and relative) each reference Molecular Definition, representations can be combined to support complex use cases. For example:

- An extracted representation can use as its startingMolecule a chromosome sequence that is specified using an accession number (represented as a code).
- A repeated representation can define the sequenceMotif using a literal.
- A concatenated representation for an assembled contig can include each sequenceElement as an attached, formatted file via resolvable.
- A relative representation can specify the startingMolecule using a code, and the replacementMolecule for each edit could be defined using a literal.

It is possible to create arbitrarily deep structures using derived representations, and while there might be rationale for doing so implementations should avoid overly-complex representation structures.

### Equivalence and Identity

Every representation, regardless of its complexity, can be resolved to a literal. Two instances of MolecularDefinition are considered equivalent if they define the same entity. For molecular sequences, this means that for two instances of MolecularDefinition to be equivalent they must resolve to the same literal sequence. Two instances are identical if their serializations are identical: they must contain the same elements, and each corresponding element must have the same value.

### Profiling MolecularDefinition

#### Support for Molecular Concepts

The Molecular Definition resource supports several profiles that represent molecular concepts:

- Sequence: a primary sequence
- Allele: a Sequence at a Location on a larger, contextual Sequence
- Variation: a defined comparison between a specified reference Allele and an alternative Allele, both at a given Location on a larger, contextual Sequence

In addition, profiles have been drafted to represent the concepts of Haplotype and Genotype, although they have not been exercised as deeply as the profiles listed above. Finally, preliminary work has demonstrated that the Molecular Definition resource could be used to represent concepts related to structural variation, including Adjacency and Fusion. It is anticipated that profiles to support these concepts will be developed over time.

#### Modular Semantics and Schemas

The MolecularDefinition resource is an abstract resource that provides building blocks for creating semantically robust, computable structures that define molecular entities. The two most complex backbone elements, location and representation, support the concept of molecular sequences but they might not be relevant to other types of entities. Conversely, other entities may require different backbone elements. As such, it is expected that these high-level backbone elements will serve as modular schemas that can be profiled as needed for a given molecular entity. Profiling could include constraints on cardinality (e.g., the Sequence profile has 0..0 location, while Allele has 1..1 location) and slicing.

#### Slicing the Representation Element

The `representation` backbone element provides a series of methods for specifying the value of a sequence. As a result, the entire structure can be used any time a sequence is referenced, and this is accomplished by slicing on `representation.focus`. The `focus` element uses a `required` binding to the `MolecularDefinitionRepresentationFocus` CodeSystem; its four codes classify the role of each representation slice.

The current sequence-based profiles of MolecularDefinition define representation slices as follows:

| Profile | Cardinality | `representation.focus` code | Semantic meaning |
|---------|-------------|------------------------------|-----------------|
| Sequence | 1..1 | *(none — focus omitted)* | The primary sequence of the molecule |
| Allele | 1..1 | `allele-state` | The sequence of the allele at the specified location |
| Allele | 0..1 | `context-state` | The surrounding genomic context at the specified location |
| Variation | 1..1 | `reference-state` | The sequence defined as the reference allele |
| Variation | 1..1 | `alternative-state` | The sequence defined as the alternate allele |
| Variation | 0..1 | `context-state` | The surrounding genomic context at the specified location |
