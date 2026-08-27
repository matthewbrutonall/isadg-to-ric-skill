# MoReq2010 and European EDRMS → RiC Mapping

This file covers how European EDRMS (Electronic Document and Records Management System) metadata — particularly under MoReq2010 — maps to RiC-CM 1.0, with a separate section on the Italian *protocollo* registration system as a concrete worked example.

European archiving operates under different upstream conditions than the Anglo-American model. Records typically pass through formal registration systems and structured EDRMS environments before reaching the archive. RiC must accommodate this richer upstream metadata.

---

## Context: The European Records Management Stack

European public administrations typically operate with:
- **MoReq2010** (Model Requirements for Electronic Records Management Systems) — a European Commission-backed functional specification defining what an EDRMS must do
- **National implementations** — each EU member state typically has national legislation and standards building on MoReq2010 (e.g., Italy's CAD/DPR 445/2000, France's SIAF requirements, Germany's DOMEA/XDOMEA)
- **Registration systems** — many European traditions require formal registration of official documents at the point of creation (Italy's *protocollo informatico* is the most developed example)
- **Classification schemes (titolari/file plans)** — hierarchical subject/functional classification applied to records at creation, which pre-determines their archival placement

The key difference from ISAD(G)-based archival description: **much of what ISAD(G) captures retrospectively, European EDRMS systems capture prospectively**, at the moment a document is registered. RiC must map both layers.

---

## MoReq2010 Aggregation Model → RiC

MoReq2010 defines a formal aggregation hierarchy:

| MoReq2010 Entity | Description | RiC Entity |
|---|---|---|
| **Service** | Top-level organisational container | E03 Record Set (`ric-rst:Fonds` or local type) |
| **Class** (classification) | Node in the classification scheme / file plan | E03 Record Set (`ric-rst:Series` or local type) |
| **Aggregation** | A grouping of records (roughly = file/dossier) | E03 Record Set (`ric-rst:File`) |
| **Record** | A discrete document unit | E04 Record |
| **Component** | An attachment or version of a record | E05 Record Part or E06 Instantiation |

The MoReq2010 classification scheme (the set of Classes) maps to a hierarchy of E03 Record Sets connected via `rico:isOrWasIncludedIn`. The classification code becomes a `rico:identifier` on each Record Set.

```turtle
# Classification scheme node (MoReq2010 "Class")
<E03_Class_01_03>
    rdf:type rico:RecordSet ;
    rico:name "Contratti e convenzioni"@it ;
    rico:identifier "1.3" ;
    rico:isOrWasIncludedIn <E03_Class_01> .

# Aggregation (dossier) within the class
<E03_Dossier_2024_0045>
    rdf:type rico:RecordSet ;
    rico:hasRecordSetType ric-rst:File ;
    rico:name "Contratto fornitura arredi — 2024"@it ;
    rico:isOrWasIncludedIn <E03_Class_01_03> .
```

### MoReq2010 Metadata Fields → RiC

| MoReq2010 Metadata | RiC Predicate / Entity |
|---|---|
| Record identifier (system ID) | `rico:identifier` on E04 |
| Title | `rico:name` on E04 |
| Classification code | `rico:identifier` on E03 (the class node) |
| Creator (user/agent) | E08 Person via `rico:hasOrHadCreator` |
| Creating organisation | E11 Corporate Body via `rico:hasOrganicProvenance` |
| Date created | `rico:creationDate` on E04 (literal) or `rico:hasCreationDate` → Date entity |
| Date received | `rico:beginningDate` or separate Event |
| Disposal schedule reference | E17 Mandate via `rico:isOrWasRegulatedBy` on Activity |
| Retention period | E15 Activity (appraisal) with `rico:isOrWasRegulatedBy` → Rule |
| Access rights | `rico:conditionsOfAccess` + `rico:isOrWasRegulatedBy` → Rule |
| Status (active/closed/transferred) | `rico:generalDescription` or Event (`rico:isOrWasAffectedBy`) |
| Component/attachment | E05 Record Part or E06 Instantiation |

### Disposal Schedule → RiC

MoReq2010 requires every aggregation to be linked to a disposal schedule. In RiC:

```turtle
<E03_Dossier_id> rico:isOrWasAffectedBy <Activity_Disposal> .

<Activity_Disposal>
    rdf:type rico:Activity ;
    rico:generalDescription "Disposal — transfer to archive after 10 years"@en ;
    rico:isOrWasRegulatedBy <Mandate_DisposalSchedule_Section4> .

<Mandate_DisposalSchedule_Section4>
    rdf:type rico:Mandate ;
    rico:name "Piano di conservazione — Sezione 4: Contratti"@it ;
    rico:identifier "PdC-4.3" .
```

---

## Italian Protocollo Registration → RiC

The Italian *protocollo informatico* is a formal registration system required by law for all official documents in Italian public administrations (DPR 445/2000, AgID Guidelines). Every document sent or received by a public body must be registered in the protocol register (*registro di protocollo*) at the moment of its arrival or departure.

### Protocollo Metadata Fields

| Protocollo Field | Description | RiC Mapping |
|---|---|---|
| **Numero di protocollo** | Unique sequential identifier (e.g., `2024-PROT-001247`) | `rico:identifier` on E06 Instantiation (it identifies the specific registered instance) |
| **Data di protocollo** | Date of registration | `rico:creationDate` on E06 Instantiation (literal) |
| **Mezzo di trasmissione** | Transmission medium (PEC email, paper, hand delivery) | `rico:hasCarrierType` → CarrierType node on E06; or `rico:generalDescription` |
| **Mittente** | Sender (for incoming) | E08 Person or E11 Corporate Body via `rico:hasOrHadCreator` on E04 Record |
| **Destinatario** | Recipient (for outgoing) | Agent linked via a recipient relation (no standard RiC-O predicate — use `rico:generalDescription` or local extension) |
| **Oggetto** | Subject / brief description | `rico:name` or `rico:scopeAndContent` on E04 |
| **Classificazione** | Classification code from *titolario* | `rico:identifier` on the E03 Record Set (class node) |
| **Fascicolo di riferimento** | Dossier/file this document belongs to | `rico:isOrWasIncludedIn` → parent E03 Record Set |
| **Allegati** | Attachments | E05 Record Part or additional E06 Instantiation |

### Why the Protocol Number Goes on the Instantiation

The protocol number identifies the *specific act of registration* of a particular physical/digital object (paper letter, PEC message, etc.) — not the intellectual content of the document itself. The same letter received by two different offices could carry two different protocol numbers. This is why the protocol number belongs on the **E06 Instantiation** (the specific carrier), not the E04 Record (the intellectual content).

```turtle
# E04 Record — the intellectual document itself
<E04_Lettera_SupplierRequest>
    rdf:type rico:Record ;
    rico:name "Richiesta preventivo — forniture informatiche"@it ;
    rico:scopeAndContent "Request for quotation for IT equipment purchase."@en ;
    rico:hasOrHadCreator <Agent_Fornitore_Acme> ;
    rico:isOrWasIncludedIn <E03_Fascicolo_Acquisti2024> ;
    rico:hasOrHadInstantiation <E06_PEC_Registered> .

# E06 Instantiation — the specific PEC message as registered
<E06_PEC_Registered>
    rdf:type rico:Instantiation ;
    rdfs:label "PEC message — protocollo 2024-PROT-001247" ;
    # Protocol number identifies this specific received instance
    rico:identifier "2024-PROT-001247" ;
    rico:creationDate "2024-03-15"^^xsd:date ;
    rico:generalDescription "Received via PEC (Posta Elettronica Certificata)"@it ;
    rico:isOrWasInstantiationOf <E04_Lettera_SupplierRequest> ;
    rico:hasOrHadHolder <Agent_ComuneRoma> .

# Sender (Mittente)
<Agent_Fornitore_Acme>
    rdf:type rico:CorporateBody ;
    rico:name "Acme S.r.l."@it .
```

### Titolario (Classification Scheme) → RiC Record Set Hierarchy

The *titolario di classificazione* is the functional classification scheme applied to all documents. It maps to a pre-existing hierarchy of E03 Record Sets:

```turtle
# Top-level class
<E03_Class_01>
    rdf:type rico:RecordSet ;
    rico:name "Amministrazione generale"@it ;
    rico:identifier "1" .

# Sub-class
<E03_Class_01_04>
    rdf:type rico:RecordSet ;
    rico:name "Contratti e accordi"@it ;
    rico:identifier "1.4" ;
    rico:isOrWasIncludedIn <E03_Class_01> .

# Fascicolo (dossier) under the sub-class
<E03_Fascicolo_Acquisti2024>
    rdf:type rico:RecordSet ;
    rico:hasRecordSetType ric-rst:File ;
    rico:name "Acquisti informatica 2024"@it ;
    rico:identifier "1.4/2024/003" ;
    rico:isOrWasIncludedIn <E03_Class_01_04> ;
    rico:beginningDate "2024-01-01"^^xsd:date .
```

---

## Other National Implementations

### France (SIAF / SEDA)

The French national archives (SIAF) use **SEDA** (Standard d'Échange de Données pour l'Archivage) for electronic transfer packages. SEDA v2.1 defines an XML transfer format with metadata broadly similar to ISAD(G). Key additions:
- `<OriginatingAgency>` — creating body (→ E11 Corporate Body via `rico:hasOrganicProvenance`)
- `<SubmissionAgency>` — transferring body if different (→ E11 Corporate Body via Activity)
- `<ArchivalAgreement>` — the legal agreement governing transfer (→ E17 Mandate)
- `<AppraisalRule>` — retention rule (→ E16 Rule via `rico:isOrWasRegulatedBy`)

### Germany (XDOMEA / DOMEA)

German public bodies use **XDOMEA** (an XML standard) for records transfer. The DOMEA concept defines:
- Aktenplan (file plan) — hierarchy of Aktenzeichen (file codes) — maps to E03 Record Set hierarchy
- Akte (file/dossier) — E03 Record Set (`ric-rst:File`)
- Vorgang (transaction/case) — E03 Record Set (sub-file) or E04 Record
- Dokument (document) — E04 Record
- Aktenzeichen (file code) — `rico:identifier` on the appropriate E03 Record Set

---

## Key Differences from Pure ISAD(G) Migration

| Aspect | ISAD(G) approach | European EDRMS approach |
|---|---|---|
| Description timing | Retrospective (by archivist) | Prospective (at document creation) |
| Creator | Often inferred from fonds | Captured at registration — map directly to Agent |
| Classification | Imposed by archivist post-transfer | Pre-assigned via titolario/Aktenplan — map to E03 hierarchy |
| Registration ID | No equivalent | Goes on E06 Instantiation (protocol number) |
| Disposal authority | Captured in 3.3.2 | Already linked via EDRMS — map to E17 Mandate |
| Transmission metadata | Not captured | Goes on E06 (medium, date, sender/recipient) |

---

## Further Reference

- MoReq2010: https://www.moreq.info/
- Italian protocollo — DPR 445/2000: https://www.normattiva.it/uri-res/N2Ls?urn:nir:stato:decreto.del.presidente.della.repubblica:2000-12-28;445
- AgID linee guida sulla gestione documentale: https://www.agid.gov.it/it/aree-tematiche/gestione-documentale
- SEDA (France): https://redirect.francearchives.fr/seda/
- XDOMEA (Germany): https://www.xdomea.de/
- RiC-CM 1.0 entities applicable here: E03 Record Set, E04 Record, E05 Record Part, E06 Instantiation, E11 Corporate Body, E15 Activity, E16 Rule, E17 Mandate
