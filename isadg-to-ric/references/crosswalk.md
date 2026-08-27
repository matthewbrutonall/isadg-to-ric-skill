# ISAD(G) → RiC-CM Crosswalk: Detailed Notes

Based on the official ICA crosswalk spreadsheet (`crosswalk--ica_standards_to_ric-cm.xlsx`, published in RiC-AG 0.1, October 2025) and RiC-CM 1.0.

**Important — object properties vs. literal shortcuts.** In RiC-O 1.1, many attributes have two predicate forms: an object property (range = a resource node, e.g., `rico:hasBeginningDate` → `rico:Date`) and a literal shortcut (range = a typed literal, e.g., `rico:beginningDate "1847"^^xsd:gYear`). Never use an object property with a plain literal value — this produces a range violation in any RiC-compliant RDF environment. The examples in this file use shortcut predicates where the value is a literal and object properties where the value is an entity node. See SKILL.md for the complete object/shortcut reference table.

**RiC-CM attribute numbers** are from RiC-CM 1.0 (November 2023). Verify against the official PDF if in doubt — numbers shifted between the 2021 drafts and the 1.0 release.

---

## 3.1 Identity Statement Area

### 3.1.1 Reference code(s)

**RiC mapping:** RiC-A22 Identifier. Literal shortcut: `rico:identifier`. Full object-property form: `rico:hasOrHadIdentifier` → `rico:Identifier` node.

```turtle
@prefix rico: <https://www.ica.org/standards/RiC/ontology#> .

<E03_ABC123> rico:identifier "CH-ABC-123" .
```

The reference code typically applies to the Record Set (E03) or Record (E04), but can also apply to the Instantiation (E06) if the code refers specifically to the physical item.

**Edge case — two types of identifier in the same export:**

For typed identifier nodes (richer linked data):
```turtle
<E03_id> rico:hasOrHadIdentifier <Identifier_SW> .
<Identifier_SW> rdf:type rico:Identifier .
<Identifier_SW> rico:identifier "413398" .
<Identifier_SW> rdfs:comment "Local software ID"@en .

<E03_id> rico:hasOrHadIdentifier <Identifier_REF> .
<Identifier_REF> rdf:type rico:Identifier .
<Identifier_REF> rico:identifier "A-1/23" .
<Identifier_REF> rdfs:comment "Archival reference code"@en .
```

This distinction matters when merging data from multiple institutions.

---

### 3.1.2 Title

**RiC mapping:** RiC-A30 Name. Literal shortcut: `rico:name`. Full object-property form: `rico:hasOrHadName` → `rico:Name` node.

Always tag with `@lang` to support multilingual descriptions:

```turtle
<E03_id>
    rico:name "Building permits, Zug municipal administration"@en ;
    rico:name "Baubewilligungen, Stadtverwaltung Zug"@de ;
    rdfs:label "Building permits, Zug municipal administration"@en .
```

Note: `rdfs:label` requires a complete triple — subject, predicate, object. If you add it inside a semicolon chain, the subject is inherited. If it is its own statement, write the subject explicitly.

---

### 3.1.3 Dates

This is the most complex ISAD(G) element in RiC. Choose the approach based on your needs:

**Approach A — Literal shortcuts (fastest migration):**
```turtle
<E03_id> rico:beginningDate "1847"^^xsd:gYear .
<E03_id> rico:endDate "1858"^^xsd:gYear .
```
Use the shortcut datatype predicates (`rico:beginningDate`, `rico:endDate`, `rico:creationDate`) for plain literal dates. The object-property forms (`rico:hasBeginningDate`, `rico:hasEndDate`) require a Date entity IRI as the object — using them with a literal is a range violation.

**Approach B — Full RiC Date entity (full compliance):**
```turtle
@prefix edtf: <http://id.loc.gov/datatypes/edtf/> .

<E03_id> rico:hasBeginningDate <Date_1847> .
<E03_id> rico:hasEndDate <Date_1858> .

<Date_1847> rdf:type rico:Date .
<Date_1847> rico:expressedDate "1847"@en .
<Date_1847> rico:normalizedDateValue "1847"^^edtf:EDTF .
<Date_1847> rico:dateQualifier "year"@en .
```

Declare `@prefix edtf: <http://id.loc.gov/datatypes/edtf/>` when using EDTF typed literals.

**Which date predicate to use:**

| Situation | Literal shortcut | Object property (→ Date entity) |
|---|---|---|
| Start of date range | `rico:beginningDate` | `rico:hasBeginningDate` |
| End of date range | `rico:endDate` | `rico:hasEndDate` |
| Single creation date | `rico:creationDate` | `rico:hasCreationDate` |
| General date association | — | `rico:isAssociatedWithDate` |
| All members of Record Set share creation date | — | `rico:hasAllMembersWithCreationDate` |
| Some members of Record Set share creation date | — | `rico:hasSomeMembersWithCreationDate` |
| Most members of Record Set share creation date | — | `rico:hasMostMembersWithCreationDate` |

**Important:** `hasAllMembersWithCreationDate` and similar relations exist because a Record Set does not itself have a creation date — only its member Records do. These relations express a generalisation about the members. They are object properties; check RiC-O 1.1 for the canonical predicate names.

**Uncertain dates:** Add a `rico:dateQualifier` (RiC-A13) with value "circa", "approximate", "end unknown", etc.

---

### 3.1.4 Level of description

**No direct RiC equivalent.** This is the most conceptually important change.

In ISAD(G), "level of description" is an attribute. In RiC, this information is encoded via:
1. **Entity type:** Record Set (E03) vs Record (E04) vs Record Part (E05)
2. **Record Set Type:** `rico:hasRecordSetType` with a value from the `ric-rst:` controlled vocabulary

```turtle
@prefix ric-rst: <https://www.ica.org/standards/RiC/vocabularies/recordSetTypes#> .

<E03_id> rdf:type rico:RecordSet .
<E03_id> rico:hasRecordSetType ric-rst:Fonds .
```

Use PascalCase: `ric-rst:Fonds`, `ric-rst:Series`, `ric-rst:File`, `ric-rst:Collection`. The published ICA vocabulary currently defines these four top-level concepts. Sub-fonds, sub-series, item as Record Set Type may need to be defined locally — verify the current vocabulary release.

**Decision rule:**
- Does this thing contain other things? → Record Set (E03)
- Is this thing itself a single document? → Record (E04)
- Is this part of a single document with independent content? → Record Part (E05)

**File level ambiguity:** A "file" in ISAD(G) can be either a Record Set (a folder containing documents) or a Record (a single-document dossier). If the file contains multiple independent documents, use E03. If it is one document, use E04.

---

### 3.1.5 Extent and medium

**RiC mapping:** Moves from the Record Resource to its Instantiation(s).

| ISAD(G) content | RiC attribute | Entity |
|---|---|---|
| Number of items (boxes, folders, sheets) | RiC-A04 Carrier Extent (`rico:carrierExtent`) | E06 Instantiation |
| Medium/format (paper, parchment, film) | RiC-A05 Carrier Type — use `rico:hasCarrierType` → CarrierType node; literal fallback: `rico:type` | E06 Instantiation |
| Size of the content itself (words, pages) | RiC-A23 Instantiation Extent | E06 Instantiation |
| Physical condition notes | RiC-A31 Physical Characteristics Note | E06 Instantiation |
| Arrangement/structure of the content | RiC-A42 Structure (`rico:structure`) | E02/03/04/05 |

**Carrier type note:** `rico:carrierType` is not a standard RiC-O predicate. Use `rico:hasCarrierType` pointing to a CarrierType resource, or `rico:type` as a generic literal if a CarrierType node is not practical.

---

## 3.2 Context Area

### 3.2.1 Name of creator

The biggest structural change in moving to RiC. The creator is no longer an attribute string — they become an Agent entity.

```turtle
# The Record Set
<E03_id> rico:hasOrHadCreator <Agent_JosefWeber> .

# The Agent entity
<Agent_JosefWeber> rdf:type rico:Person .
<Agent_JosefWeber> rico:name "Weber-Strebel, Josef Maria"@de .
<Agent_JosefWeber> rico:birthDate "1842"^^xsd:gYear .
<Agent_JosefWeber> rico:deathDate "1933"^^xsd:gYear .
```

For full Date entities (Approach B), use `rico:hasBirthDate` / `rico:hasDeathDate` → Date node.

**Three creator relations:**
- `rico:hasOrganicProvenance`: most general — any agent whose activities created or resulted in the records
- `rico:hasOrHadCreator`: the agent who intellectually created the content
- `rico:hasOrHadAccumulator`: the agent who assembled the records (may be different from creator)

These can apply simultaneously: e.g., a religious order (E11 Corporate Body) as accumulator + individual members (E08 Persons) as creators.

**Practical tip — if the creator name contains dates (e.g., "Friedrich Dürrenmatt, 1921–1990"):** split the field during data cleaning: `name`, `birthDate`, `deathDate`.

**IRIs for agents:** Use Wikidata IRIs where possible (`wd:Q12345` with `@prefix wd: <https://www.wikidata.org/entity/>`). For local figures not in Wikidata, create local IRIs.

---

### 3.2.2 Administrative/Biographical history

**RiC mapping:** RiC-A23 History. Two paths:

**Path A (simple — single text block):**
```turtle
<Agent_id> rico:history "Founded in 1847 as the Cantonal Archive of Zug..."@en .
```

**Path B (structured — Event model):**
```turtle
<Agent_id> rico:isOrWasAffectedBy <Event_Founded1847> .
<Event_Founded1847> rdf:type rico:Event .
<Event_Founded1847> rico:generalDescription "Founding of the institution"@en .
<Event_Founded1847> rico:beginningDate "1847"^^xsd:gYear .
```

Path B is more powerful (queryable, linkable) but requires more work. For bulk migrations, start with Path A.

---

### 3.2.3 Archival history

Same two paths as 3.2.2, but applied to the Record Resource entities (E02/03/04/05/06) rather than Agent entities.

RiC-A23 History on a Record Resource "may cover the entire history and include information about the history of origination, responsibility, property, custody, control, arrangement, description, and management of the record resource."

---

### 3.2.4 Immediate source of acquisition

Three encoding options:

**Option A:** Include in RiC-A23 History on the Record Resource.

**Option B:** Model as an Activity:
```turtle
<E03_id> rico:resultsOrResultedFrom <Activity_Accession2022> .
<Activity_Accession2022> rdf:type rico:Activity .
<Activity_Accession2022> rico:generalDescription "Accession"@en .
<Activity_Accession2022> rico:isOrWasPerformedBy <Agent_Donor> .
```

**Option C:** Model as an Event:
```turtle
<E03_id> rico:isOrWasAffectedBy <Event_Transfer2022> .
<Event_Transfer2022> rdf:type rico:Event .
<Event_Transfer2022> rico:generalDescription "Transfer"@en .
<Event_Transfer2022> rico:hasOrHadParticipant <Agent_Donor> .
```

---

## 3.3 Content and Structure Area

### 3.3.1 Scope and content

**RiC mapping:** RiC-A41 `rico:scopeAndContent`. Applies to E03 Record Set, E04 Record, E05 Record Part (the crosswalk targets these concrete subclasses). In RiC-O 1.1 the domain is RecordResource (E02), which includes E03/04/05 as subclasses — so the attribute applies across the board. It does **not** apply to E06 Instantiation.

```turtle
<E03_id> rico:scopeAndContent "Contains correspondence and administrative records relating to..."@en .
```

---

### 3.3.2 Appraisal, destruction and scheduling

```turtle
<E03_id> rico:isOrWasAffectedBy <Activity_Appraisal2015> .
<Activity_Appraisal2015> rdf:type rico:Activity .
<Activity_Appraisal2015> rico:generalDescription "Appraisal"@en .
<Activity_Appraisal2015> rico:isOrWasRegulatedBy <Rule_RetentionSchedule> .
<Rule_RetentionSchedule> rdf:type rico:Rule .
<Rule_RetentionSchedule> rico:name "Retention schedule section 4.2"@en .
```

For simpler migrations, use RiC-A23 History.

---

### 3.3.3 Accruals

**Only on Record Set (E03).** Not applicable to Records or Instantiations.

```turtle
<E03_id> rico:accruals "Accruing. Annual transfers expected from the municipal planning office."@en .
```
Suggested values: "Accruing", "Non-accruing", "Unknown".

---

### 3.3.4 System of arrangement

RiC-A42 Structure (`rico:structure`) applies to all Record Resource entities and to Instantiation.

```turtle
<E03_id> rico:structure "Arranged chronologically by year of decision, then alphabetically by applicant name."@en .
```

---

## 3.4 Conditions of Access and Use Area

### 3.4.1 Conditions governing access

**RiC mapping:** RiC-A10 Conditions of Access.

```turtle
<E03_id> rico:conditionsOfAccess "Closed for 30 years under data protection legislation."@en .
```

For structured access rules, also link to a Rule entity:
```turtle
<E03_id> rico:isOrWasRegulatedBy <Rule_DataProtection> .
<Rule_DataProtection> rdf:type rico:Rule .
<Rule_DataProtection> rico:name "General Data Protection Regulation (GDPR)"@en .
```

`rico:hasOrHadRuleType` is an object property requiring a RuleType resource. For a simple textual type note, use `rico:generalDescription` on the Rule instead.

Note: RiC-A10 Conditions of Access applies to Record Resources AND Instantiations (e.g., "Requires VHS player to access tape").

---

### 3.4.2 Conditions governing reproduction

```turtle
<E03_id> rico:conditionsOfUse "Permission of copyright owner required."@en .
```

Also links to Rule entity (same pattern as access conditions).

---

### 3.4.3 Language/scripts of material

**RiC mapping:** RiC-A27 Language. In RiC-O 1.1, language is modelled via object properties to Language nodes — there is no standard literal shortcut.

**For E04 Record and E05 Record Part** — use `rico:hasOrHadLanguage`:
```turtle
<E04_id> rico:hasOrHadLanguage <Language_Latin> .
<Language_Latin> rdf:type rico:Language .
<Language_Latin> rdfs:label "Latin"@en .
```

**For E03 Record Set** — the language is formally a property of the *members*, not the set itself. Use member-level relations:
```turtle
<E03_id> rico:hasOrHadAllMembersWithLanguage <Language_German> .
<Language_German> rdf:type rico:Language .
<Language_German> rdfs:label "German"@en .
```

**Pragmatic shortcut (for bulk migration where Language nodes are impractical):** Some implementations use `rico:language "German"@en` directly, treating it as an informal literal note. This is not formally correct per RiC-O but may be acceptable as a transitional measure while building out Language entities. Mark these triples as provisional if doing so.

Verify predicate names (`hasOrHadAllMembersWithLanguage` etc.) against RiC-O 1.1 — member-level language relations may be named differently in the published ontology.

Use a controlled vocabulary for language values (ISO 639 codes or similar).

---

### 3.4.4 Physical characteristics and technical requirements

Applies to **E06 Instantiation**:

```turtle
<E06_id> rico:physicalCharacteristicsNote "VHS tape format. Requires VHS player. Tape shows signs of deterioration."@en .
<E06_id> rico:conditionsOfAccess "Requires conversion to digital format before consultation."@en .
```

---

### 3.4.5 Finding aids

The finding aid is itself a Record Resource — it is a record about records.

```turtle
<E03_id> rico:isOrWasDescribedBy <FindingAid_ABC> .
<FindingAid_ABC> rdf:type rico:RecordResource .
<FindingAid_ABC> rico:name "Repertorium der Stadtarchiv Zug 1847–1990"@de .
```

**Important:** `rico:isOrWasDescribedBy` connects a Record Resource to another Record Resource (the finding aid). Do NOT use it to connect to an Activity — see 3.7 below.

---

## 3.5 Allied Materials Area

### 3.5.1–3.5.2 Originals and copies

**Originals:**
```turtle
<E04_Copy> rico:hasOrHadOriginal <E04_Original> .
<E04_Original> rico:hasOrHadHolder <Agent_NationalArchives> .
```

**Copies (Instantiation level):**
```turtle
<E06_Original> rico:hasOrHadDerivedInstantiation <E06_Digitised> .
<E06_Digitised> rico:migratedInto <E06_Preservation> .
```

---

### 3.5.3 Related units of description

Three relations of increasing specificity:
- `rico:isOrWasRelatedTo` — any relationship (most general)
- `rico:hasGeneticLinkToRecordResource` — records share a common origin or were created in relation to each other
- `rico:isRecordResourceAssociatedWithRecordResource` — documentary/intellectual association

---

## 3.6 Notes Area

### 3.6.1 Note

`rico:generalDescription` (RiC-A43) is the catch-all. It applies to any entity (domain: Thing). **Always prefer more specific attributes** — use General Description only for content that genuinely doesn't fit elsewhere.

---

## 3.7 Description Control Area

### 3.7.1–3.7.3 Archivist's note, Rules, Dates of description

These elements describe the *act of description* — an Activity in RiC terms.

**Common error:** do NOT use `rico:isOrWasDescribedBy` to connect to an Activity. That predicate connects a Record Resource to its finding-aid (3.4.5). For the description activity, use `rico:isOrWasAffectedBy`:

```turtle
<E03_id> rico:isOrWasAffectedBy <DescriptionActivity_2023> .

<DescriptionActivity_2023>
    rdf:type rico:Activity ;
    rico:generalDescription "Archival description"@en ;
    rico:isOrWasPerformedBy <Agent_Archivist> ;
    rico:isOrWasRegulatedBy <Rule_ISADG> ;
    rico:endDate "2023"^^xsd:gYear .

<Rule_ISADG>
    rdf:type rico:Rule ;
    rico:name "ISAD(G): General International Standard Archival Description, 2nd ed."@en .
```

For the activity end date, `rico:endDate "2023"^^xsd:gYear` is the literal shortcut. For a full Date entity, use `rico:hasEndDate <Date_2023>`.

For simpler implementations, RiC-A43 General Description on the Record Resource suffices for the archivist's note:
```turtle
<E03_id> rico:generalDescription "Described by J. Brown, 2023, following ISAD(G) 2nd ed."@en .
```

---

## Common Pitfalls

### Object properties with literal values
`rico:hasBeginningDate`, `rico:hasEndDate`, `rico:hasOrHadName`, `rico:hasOrHadIdentifier`, `rico:hasOrHadLanguage` are object properties — their object must be a resource IRI, not a plain literal. Using them with a literal is a range violation. Use the corresponding shortcut predicates (`rico:beginningDate`, `rico:endDate`, `rico:name`, `rico:identifier`) for literal values. Language has no standard literal shortcut — model via Language nodes.

### Scope and content on Instantiation
`rico:scopeAndContent` (RiC-A41) cannot be used on E06 Instantiation. For physical notes, use `rico:physicalCharacteristicsNote` (RiC-A31) or `rico:conditionsOfAccess` (RiC-A10).

### Language on Instantiation
`rico:hasOrHadLanguage` (RiC-A27) applies to the intellectual content (E03/04/05) — not to E06 Instantiation. The language of the *content* is a property of the Record, not the carrier.

### Accruals on Records
`rico:accruals` (RiC-A01) applies only to E03 Record Set. A Record cannot accrue — only a set can.

### isOrWasDescribedBy for Activities
`rico:isOrWasDescribedBy` connects a Record Resource to another Record Resource (a finding aid). It cannot be used to link to an Activity. See 3.7 above.

### ric-rst: vocabulary case
Use PascalCase: `ric-rst:Fonds`, `ric-rst:Series`, `ric-rst:File`, `ric-rst:Collection`. Lowercase variants are invalid.

### Wikidata IRIs in Turtle
Use `@prefix wd: <https://www.wikidata.org/entity/>` — the entity IRI, not the `/wiki/` web-page URL. Write `wd:Q27490192` as a bare prefixed name, NOT `<wd:Q27490192>` (angle brackets around a prefixed name is a syntax error).

### carrierType as a literal predicate
`rico:carrierType` is not a standard RiC-O predicate for the carrier type attribute. Use `rico:hasCarrierType` → CarrierType resource, or `rico:type` as a generic literal fallback.

### Hierarchy in RiC-O 1.1
The current RiC-O record-set aggregation vocabulary uses `includes` and `isOrWasIncludedIn` for hierarchical containment between Record Resources:
```turtle
# Child → parent
<E03_Series> rico:isOrWasIncludedIn <E03_Fonds> .
# Parent → child
<E03_Fonds> rico:includes <E03_Series> .
```
More generic whole-part relations (`rico:isOrWasPartOf`, `rico:hasOrHadMember`) also exist. Avoid the older forms `rico:isPartOf` and `rico:hasMember` (without the `isOrWas` / `hasOrHad` prefix). Verify the canonical predicates against RiC-O 1.1.

### Holder is on both RecordSet and Instantiation
Assign `rico:hasOrHadHolder` to both the Record Set (intellectual custody) and the Instantiation (physical custody). They are often the same institution, but for copies or deposits, they may differ.

### Crosswalk typo — RiC-A43
The official ICA crosswalk spreadsheet has a typo on element 3.6.1: it reads "RiCA43" instead of "RiC-A43". This is a formatting error in the source document.
