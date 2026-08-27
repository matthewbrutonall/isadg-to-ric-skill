# NARA Metadata Requirements → RiC Mapping (US Federal Records)

This file covers metadata that US federal agencies must capture for permanent records transferred to the National Archives and Records Administration (NARA), and how those fields map to RiC-CM 1.0.

ISAD(G) is rarely used explicitly in US federal records management — agencies follow NARA's own metadata requirements — but many of the same descriptive concepts appear under different names. If you are migrating a US federal record description to RiC, use this file alongside the main ISAD(G) crosswalk.

---

## Context: The NARA Transfer Framework

Federal agencies transfer permanent records to NARA under:
- **General Records Schedules (GRS)** — government-wide disposal authorities
- **Agency-specific records schedules** — approved by NARA for records unique to that agency
- **NARA Universal ERM Requirements** — minimum metadata fields required for electronic records at transfer
- **ERA (Electronic Records Archives)** — NARA's ingest system, which requires a structured transfer package

Records reaching NARA already have an approved disposition authority — meaning appraisal and scheduling happened upstream, in the records management phase, not at the archive. RiC models this via the Rule → Activity → Record Resource chain.

---

## Key NARA Metadata Fields → RiC

### Accession Number

NARA assigns a unique accession number to each transfer (e.g., `NARA-2024-0112-A`).

```turtle
<E03_FederalRecords> rico:hasOrHadIdentifier <Identifier_Accession> .
<Identifier_Accession> rdf:type rico:Identifier .
<Identifier_Accession> rico:identifier "NARA-2024-0112-A" .
<Identifier_Accession> rdfs:comment "NARA accession number"@en .
```

Or using the shortcut: `<E03_id> rico:identifier "NARA-2024-0112-A" .`

### Record Series Title

Maps directly to RiC-A30 Name:
```turtle
<E03_id> rico:name "General Correspondence Files, Office of the Secretary"@en .
<E03_id> rico:hasRecordSetType ric-rst:Series .
```

### Creating Agency / Office of Origin

NARA requires clear identification of the creating agency and sub-unit. This becomes an Agent in RiC:

```turtle
<E03_id> rico:hasOrganicProvenance <Agent_AgencyOffice> .

<Agent_AgencyOffice>
    rdf:type rico:CorporateBody ;
    rico:name "Office of the Secretary, Department of the Interior"@en .
```

For hierarchical agency structures, model each level as a separate E11 Corporate Body with `rico:isOrWasIncludedIn` between them.

### Date Span

```turtle
<E03_id> rico:beginningDate "1985"^^xsd:gYear .
<E03_id> rico:endDate "2003"^^xsd:gYear .
```

### Disposition Authority (GRS or Agency Schedule)

The legal basis for transferring records to NARA — the most important records management metadata. In RiC, this is a Rule (or Mandate) linked to an appraisal/disposition Activity:

```turtle
<E03_id> rico:isOrWasAffectedBy <Activity_Disposition> .

<Activity_Disposition>
    rdf:type rico:Activity ;
    rico:generalDescription "Permanent retention — transfer to NARA"@en ;
    rico:isOrWasRegulatedBy <Mandate_GRS021> .

<Mandate_GRS021>
    rdf:type rico:Mandate ;
    rico:name "General Records Schedule 2.1: Information Systems Records"@en ;
    rico:identifier "GRS 2.1" ;
    rico:name "DAA-GRS-2013-0006"@en .
```

Note: a disposition authority number (e.g., `DAA-GRS-2013-0006`) is an identifier for the Rule/Mandate entity.

### Volume and Extent

Goes on the Instantiation (E06), not the Record Set:
```turtle
<E06_id> rico:carrierExtent "4 cubic feet (6 boxes)"@en .
```

For electronic records:
```turtle
<E06_id> rico:carrierExtent "2.3 GB"@en .
<E06_id> rico:generalDescription "Electronic records in PDF and XLSX formats"@en .
```

### Security Classification and Access Restrictions

```turtle
<E03_id> rico:conditionsOfAccess "Restricted — review required under FOIA before release."@en .
<E03_id> rico:isOrWasRegulatedBy <Rule_FOIA> .

<Rule_FOIA>
    rdf:type rico:Rule ;
    rico:name "Freedom of Information Act (5 U.S.C. § 552)"@en .
```

For classified records:
```turtle
<E06_id> rico:conditionsOfAccess "Classified — SECRET. Declassification review required."@en .
<E06_id> rico:isOrWasRegulatedBy <Rule_EO13526> .

<Rule_EO13526>
    rdf:type rico:Rule ;
    rico:name "Executive Order 13526 — Classified National Security Information"@en .
```

### Privacy Act Coverage

```turtle
<E03_id> rico:conditionsOfAccess "Contains Privacy Act protected information. Access restricted per 5 U.S.C. § 552a."@en .
<E03_id> rico:isOrWasRegulatedBy <Rule_PrivacyAct> .
```

### Custodial History / Transfer Chain

```turtle
<E03_id> rico:history "Created by the Office of the Secretary 1985–2003. Transferred to NARA 2024 under GRS 2.1. Previously held by agency records center 2003–2024."@en .
```

Or modelled as sequential Events/Activities with `rico:isOrWasAffectedBy`.

### Format and Technical Metadata (Electronic Records)

NARA requires technical metadata for electronic transfers (format, software dependency, etc.). This belongs on the Instantiation:

```turtle
<E06_id>
    rdf:type rico:Instantiation ;
    rico:generalDescription "PDF/A-1b, created with Adobe Acrobat 11"@en ;
    rico:conditionsOfAccess "No special software required — PDF/A is an open format."@en ;
    rico:physicalCharacteristicsNote "2,341 files; 2.3 GB total; average file size 1.0 MB."@en .
```

---

## NARA-Specific Entities and Relations Summary

| NARA Concept | RiC Entity / Predicate |
|---|---|
| Accession number | `rico:identifier` on E03 (or full Identifier node) |
| Record group | E03 Record Set with `ric-rst:Fonds` or local type |
| Series | E03 Record Set with `ric-rst:Series` |
| Creating agency | E11 Corporate Body via `rico:hasOrganicProvenance` |
| Disposition authority (GRS) | E17 Mandate via `rico:isOrWasRegulatedBy` on Activity |
| Disposal action | E15 Activity via `rico:isOrWasAffectedBy` |
| Access restriction | `rico:conditionsOfAccess` + E16 Rule (FOIA, Privacy Act, EO13526) |
| Transfer date | E15 Activity (transfer) with `rico:endDate` |
| Extent (physical) | E06 `rico:carrierExtent` |
| Extent (electronic) | E06 `rico:carrierExtent` (GB/TB) + `rico:generalDescription` for format |

---

## NARA Record Group Hierarchy → RiC

NARA organises permanent records into Record Groups → Series → (Sub-series) → Item. These map straightforwardly to RiC:

```turtle
# Record Group
<E03_RG048>
    rdf:type rico:RecordSet ;
    rico:hasRecordSetType ric-rst:Fonds ;
    rico:name "Record Group 48: Records of the Office of the Secretary of the Interior"@en ;
    rico:identifier "RG 48" .

# Series within the Record Group
<E03_Series_Correspondence>
    rdf:type rico:RecordSet ;
    rico:hasRecordSetType ric-rst:Series ;
    rico:name "General Correspondence Files"@en ;
    rico:isOrWasIncludedIn <E03_RG048> .
```

---

## ERA Transfer Package → RiC

NARA's ERA system requires a structured transfer package with a manifest. When converting this to RiC:
- The transfer package manifest becomes an E15 Activity (the transfer event)
- Each logical folder in the package becomes an E03 Record Set
- Each file becomes an E04 Record with an E06 Instantiation (the specific file)
- Technical metadata on the file goes on the Instantiation

---

## Further Reference

- NARA Universal ERM Requirements: https://www.archives.gov/records-mgmt/policy/universalermreqs.html
- NARA Metadata Guidance: https://www.archives.gov/records-mgmt/initiatives/erm-overview.html
- General Records Schedules: https://www.archives.gov/records-mgmt/grs
- RiC-CM 1.0 entities applicable here: E03 Record Set, E04 Record, E06 Instantiation, E11 Corporate Body, E15 Activity, E16 Rule, E17 Mandate
