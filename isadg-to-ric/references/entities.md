# RiC-CM 1.0 Entity Reference

19 defined entities from RiC-CM 1.0 (November 2023), explained in practical archival terms. RiC-CM numbers entities E01–E22 but E19, E20, and E21 are not defined — see the note at the end of this file.

The entity hierarchy:
```
RiC-E01 Thing
├── RiC-E02 Record Resource
│   ├── RiC-E03 Record Set
│   ├── RiC-E04 Record
│   └── RiC-E05 Record Part
├── RiC-E06 Instantiation
├── RiC-E07 Agent
│   ├── RiC-E08 Person
│   ├── RiC-E09 Group
│   │   ├── RiC-E10 Family
│   │   └── RiC-E11 Corporate Body
│   ├── RiC-E12 Position
│   └── RiC-E13 Mechanism
├── RiC-E14 Event
│   └── RiC-E15 Activity
├── RiC-E16 Rule
│   └── RiC-E17 Mandate
├── RiC-E18 Date
└── RiC-E22 Place
```

---

## Record Resources (E02–E05)

### RiC-E02 Record Resource
**Definition:** Information produced or acquired and retained by an agent in the course of life or work activity.

The parent class. In practice, you rarely instantiate E02 directly — you use E03, E04, or E05. E02 exists to allow statements that apply to any kind of record resource without specifying which level.

**RiC-O class:** `rico:RecordResource`

---

### RiC-E03 Record Set
**Definition:** One or more records grouped together by an agent based on the records sharing one or more attributes or relations.

**The key archival insight:** a Record Set's identity is derived from its members. The fonds exists because of the records in it. The series exists because of the files in it. A Record Set does not itself have a creation date — only its members do (hence `hasAllMembersWithCreationDate` rather than `hasCreationDate`).

A Record Set can:
- contain other Record Sets (fonds > series > file hierarchy)
- contain Records (file > items)
- belong to more than one Record Set (multiple provenances — impossible in ISAD(G))

**RiC-O class:** `rico:RecordSet`  
**Recommended:** `rico:hasRecordSetType` — identifies it as fonds, series, file, etc. (from `ric-rst:` vocabulary). Strongly recommended practice; not formally mandated as a constraint in RiC-CM 1.0.

**Practical examples:** fonds, sub-fonds, series, sub-series, files, collections, accessions, record groups.

---

### RiC-E04 Record
**Definition:** Discrete information content formed and inscribed, at least once, by any method on any carrier in any persistent, recoverable form by an agent in the course of life or work activity.

**The key archival insight:** a Record's identity is derived from the record itself — from its content, not from what contains it. A letter is a record whether it sits in one folder or another.

A Record must have at least one Instantiation (E06). A Record can contain other Records (e.g., a dossier composed of individual letters, where both the dossier and each letter are Records) or Record Parts.

**RiC-O class:** `rico:Record`

**Practical examples:** a letter, a minute, a will, a photograph, a deed, an email, a spreadsheet, a database.

---

### RiC-E05 Record Part
**Definition:** Component of a record with independent information content that contributes to the intellectual completeness of the record.

A Record Part has meaning on its own but is understood fully only in context of its parent Record. It can itself have Record Parts.

**RiC-O class:** `rico:RecordPart`

**Practical examples:** the seal of a charter; an illumination in a manuscript; a signature page; an attachment to an email; a photograph within a dossier that has independent documentary value.

---

## Instantiation (E06)

### RiC-E06 Instantiation
**Definition:** The inscription of information made by an agent on a carrier in any persistent, recoverable form as a means of communicating information through time and space.

**The most important new concept for digital preservation work.** Instantiation is NOT a kind of Record Resource — it is the physical or digital *carrier*. It parallels PREMIS's Intellectual Entity (Record) / Representation (Instantiation) distinction.

One Record can have multiple Instantiations:
- The original parchment charter (E06 Instantiation, carrier type "parchment")
- A microfilm copy (E06 Instantiation, carrier type "microfilm")
- A digitised TIFF (E06 Instantiation, carrier type "optical disk")

Extent, carrier type, physical characteristics, and format/medium information all belong to the Instantiation, not the Record itself.

**RiC-O class:** `rico:Instantiation`  
**Key attributes:** RiC-A04 Carrier Extent, RiC-A05 Carrier Type, RiC-A23 Instantiation Extent, RiC-A31 Physical Characteristics Note  
**Key relations:** `rico:hasOrHadInstantiation` (connects Record or Record Part to its Instantiation); `rico:isOrWasInstantiationOf` (inverse)

---

## Agents (E07–E13)

### RiC-E07 Agent
**Definition:** A thing that performs activities in the world.

The parent class for all agents. Four key subtypes matter most for archival work: Person, Corporate Body, Family, Position. A fifth — Mechanism — covers automated processes and software.

---

### RiC-E08 Person
**Definition:** An individual human being.

**Practical use:** creators, donors, subjects of records, archivists who described the records, officials who authorised disposal. Any individual identified in 3.2.1 Name of creator, 3.2.2 Admin/Biog history, or 3.2.4 Source of acquisition.

**Best practice:** use Wikidata IRIs (`wd:Q12345`) for historical figures where available. For local/private individuals, create local IRIs.

---

### RiC-E09 Group
**Definition:** Two or more agents that act together as an agent.

The parent class for Family and Corporate Body. Use E09 directly only when a group doesn't fit either subtype.

---

### RiC-E10 Family
**Definition:** Two or more persons related by birth, marriage, adoption, or similar social convention, or who present themselves as a family.

**Practical use:** family fonds are common in archives. The family as a whole is the E10 agent, while individual family members are E08 Persons. RiC-A20 Family Type (controlled value: "family", "dynasty", "clan", "tribe").

---

### RiC-E11 Corporate Body
**Definition:** An organized group of persons that acts together as an agent, and that has a recognized legal or social status.

**Practical use:** government departments, religious orders, businesses, universities, archival repositories themselves. The holding institution is an E11 Corporate Body linked via `rico:hasOrHadHolder`.

**RiC-A12 Corporate Body Type** (controlled value): private, public, non-governmental organization, religious order, etc.

---

### RiC-E12 Position
**Definition:** The functional role of a person within a group.

**Critical insight:** a Position exists independently of the person who holds it. The Chancellor of the Exchequer is a position; different persons have held it over centuries. Records created in the exercise of that position belong to the position, not to any individual holder.

**Practical use:** formal official roles (the "Stadtarchiv" position, the "Treasurer" position) where records flow from the function rather than the individual. Links to persons via `rico:isOrWasHolderOfPosition`.

---

### RiC-E13 Mechanism
**Definition:** A process or system created by a person or group that performs an activity.

**Practical use:** software systems, robots, automated workflows. The digitisation scanner, the records management system, the batch OCR process. Used in RiC to attribute actions to automated processes where a human agent isn't directly responsible.

---

## Events and Activities (E14–E15)

### RiC-E14 Event
**Definition:** Something that happens or occurs in time and space.

Used for things that affect record resources, agents, or institutions but are not necessarily purposefully designed activities. Natural disasters, wars, political changes, births and deaths.

**RiC-A18 Event Type:** accession, acquisition, arrangement, birth, description, digitization, earthquake, hurricane, marriage, transfer.

```turtle
<E03_id> rico:isOrWasAffectedBy <Event_Fire1847> .
<Event_Fire1847> rdf:type rico:Event .
<Event_Fire1847> rico:generalDescription "Fire"@en .
# Literal shortcut for event date:
<Event_Fire1847> rico:beginningDate "1847"^^xsd:gYear .
# Or full Date entity: rico:hasBeginningDate <Date_1847>
```

---

### RiC-E15 Activity
**Definition:** The doing of something for an agent designed purpose. A kind of Event.

The purposeful actions of agents: creating records, appraising, describing, transferring, digitising. In a government or corporate context, activities may also be called "functions."

**The key difference from Event:** an Activity is intentional (designed purpose). An earthquake is an Event; the decision to move the archive to safety is an Activity.

Activities can be composed of other activities: "Digitisation project" > "Scanning" + "Quality checking" + "Ingest."

**RiC-A02 Activity Type:** business process, action, task, transaction.

---

## Rules and Mandates (E16–E17)

### RiC-E16 Rule
**Definition:** Conditions that govern the existence, responsibility, or authority of an agent; or the performance of an activity by an agent; or that contribute to the distinct characteristics of things created or managed by an agent.

Rules are pervasive in archives — they govern who can create records, how long records are kept, who can access them, and how they are described.

Rules can be:
- **Unwritten:** social customs, professional expectations
- **Written:** constitutions, legislation, statutes, regulations, policies, professional standards (including ISAD(G) and RiC-CM themselves)

```turtle
<Rule_ISADG> rdf:type rico:Rule .
<Rule_ISADG> rico:name "ISAD(G): General International Standard Archival Description, 2nd ed."@en .
```

---

### RiC-E17 Mandate
**Definition:** Delegation of responsibility or authority by an agent to another agent to perform an activity. A kind of Rule.

A Mandate explicitly confers authority (whereas a Rule may constrain without explicitly granting authority). A Mandate typically limits the place (jurisdiction) and time period.

**Practical examples:** legislation granting a government department authority to maintain certain records; a formal deed of transfer giving an archive custody.

---

## Date (E18)

### RiC-E18 Date
**Definition:** Chronological information associated with an entity that contributes to its identification and contextualization.

Dates are entities, not just strings, because dates are inherently complex: they may be expressed in natural language, in calendar systems, with varying degrees of precision, or with qualifiers of uncertainty.

**Key attributes:**
- **RiC-A19 Expressed Date:** human-readable string (`"March 1842"@en`, `"XVIIe siècle"@fr`)
- **RiC-A42 Date Type:** single date, date range, date set, year, century
- **RiC-A13 Date Qualifier:** circa, approximate, end unknown

**Standard:** digital dates should use ISO 8601 or EDTF (Extended Date/Time Format).

---

## Place (E22)

### RiC-E22 Place
**Definition:** Bounded, named geographic area or region.

Places in RiC include jurisdictions, man-made structures, and natural features. A place may also be a jurisdiction — the bounded area within which an agent has authority to perform activities under rules.

Places are historical entities: they have beginning and ending dates, and their boundaries can change due to events.

**RiC-A11 Coordinates:** ISO 6709 geographic coordinates.

**Best practice:** use Wikidata IRIs (`wd:Q12345`) for places wherever possible.

```turtle
@prefix wd: <https://www.wikidata.org/entity/> .

<Place_Zug> rdf:type rico:Place .
<Place_Zug> owl:sameAs wd:Q68144 .
<Place_Zug> rico:name "Zug"@de .
```

---

## Note on Missing Entities

The entity hierarchy shown above goes E01 through E22, but not all numbers are used — some were reserved or have not been published. The complete active set in RiC-CM 1.0 is: E01, E02, E03, E04, E05, E06, E07, E08, E09, E10, E11, E12, E13, E14, E15, E16, E17, E18, E22. E19, E20, E21 are not defined in RiC-CM 1.0.
