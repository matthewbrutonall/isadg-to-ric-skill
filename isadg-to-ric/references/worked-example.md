# Worked Example: ISAD(G) AIS Export → RiC Turtle

This example shows how a typical ISAD(G) archival information system (AIS) export maps to RiC entities and RDF triples. It is modelled on real migration work from multiple Swiss GLAM institutions (Spec, 2023), adapted as a generic example.

---

## Starting Point: ISAD(G) AIS Export (CSV)

A city archive exports the following unit of description from its AIS:

| Field | Value |
|---|---|
| ID (software ID) | 413398 |
| Verzeichnungsstufe (Level) | Dossier (= File) |
| Titel (Title) | Neugasse 2, Verwaltungsgebäude — Baugesuche |
| Signatur (Reference code) | S.13–1.457 |
| Zeitraum (Date range) | 1970–1985 |
| Form und Inhalt (Scope and content) | Building permit applications for the municipal administration building at Neugasse 2, including structural plans and correspondence. |
| Archivalienart (Content type) | Schriftgut (Written materials) |
| Frist (Access conditions) | Open (no restriction) |
| Überlieferungsform (Representation type) | Original |
| Umfang (Extent/carrier) | 1 Ordner (1 binder) |
| Superordinate (from reference code hierarchy) | ID 413350 (the series "Baubewilligungen") |

---

## Step 1: Identify RiC Entities

One ISAD(G) unit of description becomes multiple RiC entities:

| Entity | Type | IRI |
|---|---|---|
| The intellectual content of the file | E03 Record Set | `<E03_413398>` |
| The physical binder | E06 Instantiation | `<E06_413398_001>` |
| The holding institution | E11 Corporate Body | `wd:Q27490192` (Wikidata entity for Stadtarchiv Zug) |
| The superordinate series | E03 Record Set | `<E03_413350>` |

Note: `@prefix wd: <https://www.wikidata.org/entity/>` is the correct RDF namespace for Wikidata entities (not the `/wiki/` web-page URL). Write `wd:Q27490192` as a bare prefixed name — NOT `<wd:Q27490192>`.

---

## Step 2: Map to RiC Triples (Turtle format)

This uses Approach A (literal shortcuts) for dates. For full Date entities, see Step 4.

```turtle
@prefix rico:    <https://www.ica.org/standards/RiC/ontology#> .
@prefix ric-rst: <https://www.ica.org/standards/RiC/vocabularies/recordSetTypes#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .
@prefix wd:      <https://www.wikidata.org/entity/> .

# ──────────────────────────────────────────────
# E03 Record Set (intellectual content of the file)
# ──────────────────────────────────────────────

<E03_413398>
    rdf:type rico:RecordSet ;
    rdfs:label "Neugasse 2, Verwaltungsgebäude — Baugesuche" ;

    # Reference code (Signatur) — shortcut literal
    rico:identifier "S.13–1.457" ;

    # Software ID
    rico:identifier "413398" ;

    # Title — shortcut literal
    rico:name "Neugasse 2, Verwaltungsgebäude — Baugesuche"@de ;

    # Level: Dossier = File (PascalCase)
    rico:hasRecordSetType ric-rst:File ;

    # Date range — use shortcut predicates for literal values
    rico:beginningDate "1970"^^xsd:gYear ;
    rico:endDate "1985"^^xsd:gYear ;

    # Scope and content (RiC-A41)
    rico:scopeAndContent "Building permit applications for the municipal administration building at Neugasse 2, including structural plans and correspondence."@en ;

    # System of arrangement (RiC-A42)
    rico:structure "Chronological."@en ;

    # Holder
    rico:hasOrHadHolder wd:Q27490192 ;

    # Hierarchy: this file is included in the series
    # Use isOrWasIncludedIn for record-set aggregation in RiC-O 1.1
    rico:isOrWasIncludedIn <E03_413350> ;

    # Connection to instantiation
    rico:hasOrHadInstantiation <E06_413398_001> .


# ──────────────────────────────────────────────
# E06 Instantiation (the physical binder)
# ──────────────────────────────────────────────

<E06_413398_001>
    rdf:type rico:Instantiation ;
    rdfs:label "Instantiation of: Neugasse 2, Verwaltungsgebäude — Baugesuche" ;

    # Representation type (informal literal note; for controlled vocab use rico:hasRepresentationType → node)
    rico:generalDescription "Original"@de ;

    # Carrier extent
    rico:carrierExtent "1 Ordner"@de ;

    # Access conditions
    rico:conditionsOfAccess "Open"@en ;

    # Inverse link
    rico:isOrWasInstantiationOf <E03_413398> ;

    # Holder
    rico:hasOrHadHolder wd:Q27490192 .


# ──────────────────────────────────────────────
# Holding institution (E11 Corporate Body)
# ──────────────────────────────────────────────

wd:Q27490192
    rdfs:label "Stadtarchiv Zug"@de .


# ──────────────────────────────────────────────
# Superordinate series (referenced but not fully described here)
# ──────────────────────────────────────────────

<E03_413350>
    rdf:type rico:RecordSet ;
    rico:hasRecordSetType ric-rst:Series ;
    rico:name "Baubewilligungen"@de ;
    rico:includes <E03_413398> .
```

---

## Step 3: Add Creator (if known)

If the AIS also exports a creator name — e.g., "Stadtpolizei Zug (Bauamt), 1890–1972" — this becomes an Agent entity:

```turtle
<E03_413398>
    rico:hasOrHadAccumulator <Agent_Bauamt> .

<Agent_Bauamt>
    rdf:type rico:CorporateBody ;
    rico:name "Bauamt der Stadtpolizei Zug"@de ;
    # Literal shortcut for date on Agent — use rico:endDate, not rico:hasEndDate
    rico:endDate "1972"^^xsd:gYear .
```

Use `rico:hasOrHadCreator` when the agent's role is clearly intellectual authorship; `rico:hasOrHadAccumulator` when the agent assembled records over time (records management context); `rico:hasOrganicProvenance` for the most general creator/custodian relation.

---

## Step 4: Full Date Entities (optional upgrade)

For richer linked data, upgrade date literals to Date entities. Use the object-property predicates (`rico:hasBeginningDate`, `rico:hasEndDate`) when the object is a Date entity IRI:

```turtle
@prefix edtf: <http://id.loc.gov/datatypes/edtf/> .

<E03_413398>
    rico:hasBeginningDate <Date_1970> ;
    rico:hasEndDate <Date_1985> .

<Date_1970>
    rdf:type rico:Date ;
    rico:expressedDate "1970"@en ;
    rico:normalizedDateValue "1970"^^edtf:EDTF ;
    rico:dateQualifier "year"@en .

<Date_1985>
    rdf:type rico:Date ;
    rico:expressedDate "1985"@en ;
    rico:normalizedDateValue "1985"^^edtf:EDTF ;
    rico:dateQualifier "year"@en .
```

Declare `@prefix edtf: <http://id.loc.gov/datatypes/edtf/>` when using EDTF typed literals.

---

## Step 5: Description Activity (optional — elements 3.7.1–3.7.3)

To record *how* the description was made, use `rico:isOrWasAffectedBy` — NOT `rico:isOrWasDescribedBy` (which connects to a finding-aid Record Resource, not an Activity):

```turtle
<E03_413398>
    rico:isOrWasAffectedBy <DescriptionAct_2023> .

<DescriptionAct_2023>
    rdf:type rico:Activity ;
    rico:generalDescription "Archival description"@en ;
    rico:isOrWasPerformedBy <Agent_Archivist_JB> ;
    rico:isOrWasRegulatedBy <Rule_ISADG> ;
    # Literal shortcut for activity date
    rico:endDate "2023"^^xsd:gYear .

<Rule_ISADG>
    rdf:type rico:Rule ;
    rico:name "ISAD(G): General International Standard Archival Description, 2nd ed."@en .
```

---

## Mapping Decision Summary

| ISAD(G) Field | Became | RiC predicate used |
|---|---|---|
| ID | Identifier on E03 | `rico:identifier "413398"` (shortcut literal) |
| Verzeichnungsstufe (Dossier) | E03 Record Set with type | `rdf:type rico:RecordSet` ; `rico:hasRecordSetType ric-rst:File` |
| Titel | Name on E03, label on E06 | `rico:name` ; `rdfs:label` |
| Signatur | Identifier on E03 | `rico:identifier "S.13–1.457"` |
| Zeitraum | Beginning/End dates | `rico:beginningDate` ; `rico:endDate` (literal shortcuts) |
| Form und Inhalt | Scope and content on E03 | `rico:scopeAndContent` |
| Archivalienart | Content type note | `rico:generalDescription` (no direct RiC predicate maps) |
| Frist | Conditions of access on E06 | `rico:conditionsOfAccess` |
| Überlieferungsform | Representation type on E06 | `rico:generalDescription` (informal); formal: `rico:hasRepresentationType` → node |
| Umfang | Carrier extent on E06 | `rico:carrierExtent` |
| Hierarchy (from reference code) | Inclusion relation | `rico:isOrWasIncludedIn <E03_413350>` |
| Connection to physical carrier | Instantiation link | `rico:hasOrHadInstantiation <E06_413398_001>` |
| Holding institution | E11 Corporate Body + holder | `rico:hasOrHadHolder wd:Q27490192` |

---

## Practical Notes for AIS Migrations

**Date predicates — two levels:**
- For plain literals: use shortcut predicates (`rico:beginningDate`, `rico:endDate`, `rico:creationDate`)
- For Date entity IRIs: use object predicates (`rico:hasBeginningDate`, `rico:hasEndDate`, `rico:hasCreationDate`)
- Never use an object predicate with a plain literal value — this is a range violation

**Hierarchy reconstruction:** Most AIS exports don't include explicit parent–child keys. Reconstruct hierarchy from the reference code structure (e.g., "A.1/2" is included in "A.1" which is included in "A"). Create the `rico:isOrWasIncludedIn` triple manually or via script.

**IRIs:** Use the AIS's internal ID as the basis for locally created IRIs. Prefix with entity type: `<E03_413398>` for a Record Set, `<E06_413398_001>` for its first Instantiation. The sequential suffix `_001` allows for future additional instantiations.

**Holder:** Always state the holding institution explicitly on both the Record Set and the Instantiation. For items held elsewhere, create a separate Agent entity for that institution.

**Graph database tools:**
- **OntoRefine** (free, by Ontotext) — GUI-based CSV to RDF mapping, excellent for AIS exports
- **YARRRML/RML** — declarative mapping language for CSV/JSON/XML to RDF
- **OpenRefine** — data cleaning before RDF transformation
- **Turtle files** load directly into triple stores (GraphDB, Apache Jena Fuseki, Virtuoso)
