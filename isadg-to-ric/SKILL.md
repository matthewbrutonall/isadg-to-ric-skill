---
name: isadg-to-ric
description: "Practical mapping guide for archivists migrating descriptions from ISAD(G) to Records in Contexts (RiC-CM 1.0). Use when the user asks about: converting any ISAD(G) element (reference code, title, dates, level of description, creator, scope and content, conditions of access, extent, language, etc.) to its RiC equivalent; choosing the right RiC entity type (Record Set, Record, Instantiation, Agent, Activity, Place, Date); understanding the structural shift from ISAD(G) hierarchy to RiC graph; migrating a catalogue or AIS from ISAD(G) to RiC; or any question about RiC-CM entities, attributes (RiC-A), or relations (RiC-R). Based on the official ICA crosswalk (RiC-AG 0.1, 2025) and RiC-CM 1.0 (November 2023)."
---

# ISAD(G) to RiC Conversion Guide

## The Core Shift

**ISAD(G) model:** one "unit of description" + up to 26 attributes. The logic is entity–attribute, the same as a spreadsheet row or relational database record. Hierarchy is a strict parent–child tree.

**RiC model:** distinct entity types (Record Resource, Agent, Event, Place, Date, Rule…) each with their own attributes, connected by named relations (subject–predicate–object triples). Any entity can relate to any other — not just up and down a hierarchy.

**Key consequence:** one ISAD(G) unit of description becomes multiple RiC entities. At minimum: the intellectual content (RecordSet or Record) plus, for Records, the physical/digital carrier (Instantiation). Creators, dates, activities, places, and rules each become their own entities too.

## RiC-O: Object Properties vs. Shortcut Datatype Properties

RiC-O 1.1 implements RiC-CM attributes in two ways. Understanding this is essential for writing valid Turtle:

**Full object property pattern** — links to an intermediate resource node:
```turtle
<E04_id> rico:hasOrHadIdentifier <Identifier_001> .
<Identifier_001> rdf:type rico:Identifier .
<Identifier_001> rico:identifier "CH-ABC-123" .
```

**Shortcut datatype property** — takes a literal value directly:
```turtle
<E04_id> rico:identifier "CH-ABC-123" .
```

The shortcut form is simpler and recommended for bulk migrations. **Never use an object property (e.g., `rico:hasBeginningDate`) with a plain literal** — object properties must point to a resource IRI. The corresponding literal shortcut is a different predicate.

### Quick Reference: Object Property vs. Literal Shortcut

| Concept | RiC-O Object Property (→ entity node) | RiC-O Literal Shortcut |
|---|---|---|
| Identifier | `rico:hasOrHadIdentifier` → `rico:Identifier` | `rico:identifier` |
| Name / Title | `rico:hasOrHadName` → `rico:Name` | `rico:name` |
| Beginning date | `rico:hasBeginningDate` → `rico:Date` | `rico:beginningDate` |
| End date | `rico:hasEndDate` → `rico:Date` | `rico:endDate` |
| Creation date | `rico:hasCreationDate` → `rico:Date` | `rico:creationDate` |
| Birth date (Agent) | `rico:hasBirthDate` → `rico:Date` | `rico:birthDate` |
| Death date (Agent) | `rico:hasDeathDate` → `rico:Date` | `rico:deathDate` |
| Language | `rico:hasOrHadLanguage` → `rico:Language` | — (no standard shortcut; Language node required) |
| Carrier type | `rico:hasCarrierType` → `rico:CarrierType` | `rico:type` (generic literal) |
| Record Set Type | `rico:hasRecordSetType` → `ric-rst:` node | — (always a controlled vocabulary IRI) |
| Creator | `rico:hasOrHadCreator` → Agent | — (agents are always IRIs, never literals) |
| Accumulator | `rico:hasOrHadAccumulator` → Agent | — |
| Organic provenance | `rico:hasOrganicProvenance` → Agent | — |

> **Verify predicate names.** RiC-O 1.1 is the current standard; predicate names have shifted across drafts. Always check against the ICA-EGAD RiC-O 1.1 source at https://raw.githubusercontent.com/ICA-EGAD/RiC-O/master/ontology/current-version/RiC-O_1-1.rdf before production use. **RiC-CM attribute numbers** (RiC-A22, etc.) are from RiC-CM 1.0 (November 2023) — verify against the official PDF if in doubt.

## Choosing the Right Entity Type

### Record Set vs Record: the critical distinction

**Record Set (RiC-E03):** A grouping of records. Identity derived from its members — the set exists because of what it contains. Use for fonds, sub-fonds, series, sub-series, and files that contain multiple items. A Record Set *may* have an Instantiation (e.g., a microfilm roll of a whole series) but is not required to.

**Record (RiC-E04):** A single discrete document. Identity derived from the document itself. Use for items that are themselves one document. A Record *must* have at least one Instantiation.

**Record Part (RiC-E05):** A component of a single record with independent information content (e.g., an illumination in a manuscript; the seal of a charter). Must have at least one Instantiation.

**Instantiation (RiC-E06):** The physical or digital carrier — the specific inscription on a specific medium. Extent, carrier type, and physical characteristics all belong here, not on the Record itself.

### Level of description → Entity + Record Set Type

| ISAD(G) Level | RiC Entity | Record Set Type |
|---|---|---|
| Fonds | E03 Record Set | `ric-rst:Fonds` |
| Series | E03 Record Set | `ric-rst:Series` |
| File (contains items) | E03 Record Set | `ric-rst:File` |
| Item (single document) | E04 Record | — |
| Item part | E05 Record Part | — |

**Note on the ric-rst: vocabulary:** `ric-rst:Fonds`, `ric-rst:Series`, `ric-rst:File`, and `ric-rst:Collection` are the four top-level concepts in the published ICA Record Set Types vocabulary (use PascalCase — `ric-rst:Fonds` not `ric-rst:fonds`). Sub-fonds, sub-series, and item may need to be defined locally; verify against the current vocabulary release.

**Recommended, not mandatory:** `rico:hasRecordSetType` is the standard way to express level-of-description for a Record Set, and it is strongly recommended practice. It is not formally mandated as a required constraint in RiC-CM 1.0.

**No direct RiC attribute for "level of description"** as a text string exists. Level is expressed by the entity type choice plus `rico:hasRecordSetType` on Record Sets.

**File level ambiguity:** A "file" can be either a Record Set (a folder holding documents) or a Record (a single-document dossier). If the file contains multiple independent documents → E03 Record Set. If it is one document → E04 Record.

## Three Structural Rules

### Rule 1: Records (not Record Sets) require an Instantiation

A **Record (E04) or Record Part (E05)** must have at least one Instantiation. Think: the document must exist somewhere on some carrier.

A **Record Set (E03)** may have an Instantiation (e.g., a digitised microfilm of an entire series) but is not required to have one.

Connect a Record to its Instantiation with:
```turtle
<Record> rico:hasOrHadInstantiation <Instantiation> .
```

A Record can have multiple Instantiations: the original parchment, a microfilm copy, a digitised TIFF — each a separate E06.

### Rule 2: Creators become Agent entities, not attribute strings

ISAD(G) element 3.2.1 "Name of creator" is free text. In RiC, the creator becomes a Person (E08), Corporate Body (E11), or Family (E10) entity, linked via:

- `rico:hasOrganicProvenance` — general: any agent whose activity resulted in the records
- `rico:hasOrHadCreator` — the agent who created the content
- `rico:hasOrHadAccumulator` — the agent who assembled the records over time

The creator's name becomes a `rico:name` attribute on the agent entity.

### Rule 3: Two date models — literal shortcuts vs. Date entities

**Simple migration:** use the literal shortcut predicates directly:
```turtle
<E03_id> rico:beginningDate "1847"^^xsd:gYear .
<E03_id> rico:endDate "1858"^^xsd:gYear .
```

**Full RiC compliance:** create E18 Date entities and use the object-property form:
```turtle
<E03_id> rico:hasBeginningDate <Date_1847> .
<Date_1847> rdf:type rico:Date .
<Date_1847> rico:expressedDate "1847"@en .
```

**Do not mix:** `rico:hasBeginningDate "literal"` is invalid — `hasBeginningDate` is an object property requiring a Date entity IRI. For a literal date, use `rico:beginningDate`.

## Hierarchy in RiC-O

**ISAD(G)** uses a strict parent–child tree. **RiC** models record-set aggregation with specific inclusion relations:

```turtle
# Child Record Set → parent (preferred for record-set aggregation)
<E03_Series> rico:isOrWasIncludedIn <E03_Fonds> .

# Parent → child (inverse)
<E03_Fonds> rico:includes <E03_Series> .
```

More generic whole-part relations (`rico:isOrWasPartOf`, `rico:hasOrHadMember`) also exist for other contexts. The RiC-O 1.1 record-set aggregation vocabulary is built around `includes` / `isOrWasIncludedIn`.

**Key difference from ISAD(G):** in RiC a Record Set can be part of *more than one* Record Set, enabling cross-institutional shared fonds and records with multiple provenances — impossible to express in ISAD(G).

## Quick Mapping Reference — All 26 ISAD(G) Elements

Based on the official ICA crosswalk (RiC-AG 0.1, October 2025) and RiC-CM 1.0. RiC-CM attribute codes (RiC-A22, etc.) are from RiC-CM 1.0 (November 2023). The RiC-O predicate names appear in `references/crosswalk.md`.

### 3.1 Identity Statement Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.1.1 Reference code | E02/03/04/05/06 | RiC-A22 Identifier — shortcut: `rico:identifier`; full: `rico:hasOrHadIdentifier` |
| 3.1.2 Title | E02/03/04/05/06 | RiC-A30 Name — shortcut: `rico:name`; full: `rico:hasOrHadName` |
| 3.1.3 Dates | E02/03/04/05/06 → E18 Date | Literal shortcuts: `rico:beginningDate`, `rico:endDate`, `rico:creationDate`; Date entities: `rico:hasBeginningDate`, `rico:hasEndDate`; Record Set member dates: `rico:hasAllMembersWithCreationDate` / `rico:hasSomeMembersWithCreationDate` |
| 3.1.4 Level of description | Choose entity type | E03 + `rico:hasRecordSetType ric-rst:Fonds/Series/File`; or E04 Record |
| 3.1.5 Extent and medium | E06 Instantiation | RiC-A04 Carrier Extent (`rico:carrierExtent`); RiC-A05 Carrier Type (`rico:hasCarrierType` → node); RiC-A23 Instantiation Extent; RiC-A31 Physical Characteristics Note |

### 3.2 Context Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.2.1 Name of creator | E02/03/04/05/06 → Agent (E08/09/10/11/12) | `rico:hasOrganicProvenance` / `rico:hasOrHadCreator` / `rico:hasOrHadAccumulator`; agent has `rico:name` |
| 3.2.2 Admin/Biog history | Agent (E08/09/10/11/12) | RiC-A23 History (`rico:history`); or `rico:isOrWasAffectedBy` → E14 Event |
| 3.2.3 Archival history | E02/03/04/05/06 | RiC-A23 History (`rico:history`); or `rico:isOrWasAffectedBy` → E14 Event |
| 3.2.4 Immediate source of acquisition | E02/03/04/05/06 | RiC-A23 History; or `rico:resultsOrResultedFrom` → E15 Activity; or `rico:isOrWasAffectedBy` → E14 Event |

### 3.3 Content and Structure Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.3.1 Scope and content | E03/E04/E05 (RiC-O domain: RecordResource including E02) | RiC-A41 `rico:scopeAndContent` |
| 3.3.2 Appraisal, destruction, scheduling | E02/03/04/05/06 | `rico:isOrWasAffectedBy` → E15 Activity, regulated by `rico:isOrWasRegulatedBy` → E16 Rule; or RiC-A23 History |
| 3.3.3 Accruals | **E03 Record Set only** | RiC-A01 `rico:accruals` |
| 3.3.4 System of arrangement | E02/03/04/05/06 | RiC-A42 `rico:structure` |

### 3.4 Conditions of Access and Use Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.4.1 Conditions governing access | E03/04/05/06 | RiC-A10 `rico:conditionsOfAccess`; also `rico:isOrWasRegulatedBy` → E16 Rule |
| 3.4.2 Conditions governing reproduction | E03/04/05/06 | `rico:conditionsOfUse`; also Rule |
| 3.4.3 Language/scripts of material | E04/E05 direct; E03 via Language node or member relations | RiC-A27 Language — requires Language node: `rico:hasOrHadLanguage` → `rico:Language`; no standard literal shortcut |
| 3.4.4 Physical characteristics and technical requirements | E06 Instantiation | RiC-A10 Conditions of Access; RiC-A31 Physical Characteristics Note |
| 3.4.5 Finding aids | E02/03/04/05/06 | `rico:isOrWasDescribedBy` → E02 Record Resource (the finding aid is itself a Record Resource) |

### 3.5 Allied Materials Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.5.1 Existence and location of originals | E04/E05 | `rico:hasOrHadOriginal` → E04/E05; also `rico:hasOrHadHolder` → Agent |
| 3.5.2 Existence and location of copies | E02/03/04/05 | `rico:hasCopy` → Record Resource; E06 via `rico:hasOrHadDerivedInstantiation`, `rico:migratedInto` |
| 3.5.3 Related units of description | E02/03/04/05/06 | `rico:isOrWasRelatedTo`; `rico:hasGeneticLinkToRecordResource`; `rico:isRecordResourceAssociatedWithRecordResource` |
| 3.5.4 Publication note | E02/03/04/05/06 | `rico:isOrWasDescribedBy` → Record; `rico:isOrWasMainSubjectOf` → Record; `rico:isOrWasSubjectOf` → Record |

### 3.6 Notes Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.6.1 Note | E02/03/04/05/06 | RiC-A43 `rico:generalDescription` — catch-all; use only when no more specific attribute exists |

### 3.7 Description Control Area

| ISAD(G) Element | RiC Entity | RiC-CM Attribute / Relation |
|---|---|---|
| 3.7.1 Archivist's note | E02/03/04/05/06 | RiC-A43 General Description; or `rico:isOrWasDescribedBy` → finding-aid Record Resource |
| 3.7.2 Rules or conventions | E15 Activity | Description Activity `rico:isOrWasRegulatedBy` → E16 Rule |
| 3.7.3 Date(s) of descriptions | E15 Activity | Description Activity `rico:endDate "2023"^^xsd:gYear` (literal) or `rico:hasEndDate` → Date entity |

## Common RiC Prefixes

```turtle
@prefix rico:    <https://www.ica.org/standards/RiC/ontology#> .
@prefix ric-rst: <https://www.ica.org/standards/RiC/vocabularies/recordSetTypes#> .
@prefix ric-dft: <https://www.ica.org/standards/RiC/vocabularies/documentaryFormTypes#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .
@prefix edtf:    <http://id.loc.gov/datatypes/edtf/> .
@prefix wd:      <https://www.wikidata.org/entity/> .
```

Note: `@prefix wd: <https://www.wikidata.org/entity/>` is the correct entity namespace for RDF use (not the `/wiki/` web-page URL). Write `wd:Q27490192` as a bare prefixed name, never `<wd:Q27490192>`.

## Reference Files

- `references/crosswalk.md` — detailed notes, edge cases, Turtle examples, and common pitfalls for all 26 ISAD(G) elements
- `references/entities.md` — definitions and practical archival explanations for all defined RiC-CM entity types
- `references/worked-example.md` — complete worked example: ISAD(G) AIS export → RiC entities and Turtle triples
- `references/nara-ric.md` — US federal records: NARA metadata requirements (accession numbers, GRS disposition authorities, ERA transfer packages, FOIA/Privacy Act restrictions) → RiC mapping
- `references/moreq-european-ric.md` — European EDRMS: MoReq2010 aggregation model, Italian protocollo registration metadata, French SEDA, German XDOMEA → RiC mapping
