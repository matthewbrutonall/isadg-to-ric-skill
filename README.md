# ISAD(G) to RiC Conversion Skill

![Archival hierarchy becoming a network of records, people, places, and dates](assets/isadg-to-ric-skill.png)

A catalogue description written under **ISAD(G)**, a widely used archival description standard, will need new decisions if you want to represent it in **Records in Contexts (RiC)**. RiC can describe records alongside the people, places, activities, and other relationships around them.

This **agent skill** guides an assistant through those decisions, field by field. It is a reference and working method for archivists planning a migration; it is not a program that automatically rewrites a catalogue.

For example, an ISAD(G) file description might name *Parish Council* as its creator and sit inside a series called *Minutes*. In RiC, the file, its creator, and its relationship to the series can be described explicitly. The skill helps you decide which RiC entities and relationships express those facts. This is an illustrative example; your institution must review its own source data and modelling choices.

## Who might use it?

Archivists, cataloguers, and migration teams who already have ISAD(G) descriptions and want to plan or review a RiC representation. You can use it to work through one record, compare field mappings, or discuss a batch migration before writing import software.

## What it helps you do

- Match ISAD(G) elements such as reference code, title, dates, creator, and access conditions to possible RiC descriptions.
- Decide whether a thing is a record, a group of records, a person or organisation, a date, or another RiC entity.
- Keep important relationships visible, such as a file belonging to a series or a creator being connected to records.
- Read a worked example that starts with an archival export and ends with RiC statements.

The guidance also includes notes for US federal and European records-management contexts. Mappings are decisions to review, not guarantees that every field in your catalogue has one automatic equivalent. RiC guidance and ontology terms evolve, so check the current ICA sources and your local practice before production use.

## Set it up with your AI agent

Copy and paste this into your agent:

```text
Install the ISAD(G) to RiC skill from https://github.com/matthewbrutonall/isadg-to-ric-skill using your agent's supported skill installation method. Use the packaged skill or keep the complete `isadg-to-ric` directory, including its `references`, together. Confirm the skill is available and tell me how to invoke it. Do not convert any catalogue data yet.
```

Once installed, you might ask: “Using the ISAD(G) to RiC skill, show how this file-level description could be represented in RiC. Explain the choices and mark anything that needs local review.”

## What It Covers

- **The conceptual shift** — moving from ISAD(G)'s entity–attribute model (one unit of description, up to 26 attributes, strict hierarchy) to RiC's graph model (distinct entity types connected by named relations)
- **Full element crosswalk** — every ISAD(G) element (reference code, title, dates, level of description, name of creator, scope and content, conditions of access, extent, language, and the rest of areas 3.1–3.7) mapped to its RiC equivalent
- **Entity selection** — choosing the right RiC entity type (Record Set, Record, Instantiation, Agent, Activity, Place, Date, Rule)
- **RiC-O in practice** — object properties vs. shortcut datatype properties, so you can write valid Turtle
- **Structural rules** — how one ISAD(G) unit of description becomes multiple RiC entities, why creators and dates become entities rather than attributes, and how hierarchy is expressed as relations
- **A complete worked example** — an ISAD(G) AIS export converted to RiC entities and Turtle triples
- **Jurisdiction bridges** for migrations that start from records-management metadata rather than ISAD(G) proper:
  - **US federal** — NARA metadata requirements (accession numbers, General Records Schedules, ERA transfers, FOIA/Privacy Act restrictions) → RiC
  - **European EDRMS** — MoReq2010, the Italian *protocollo* registration system, French SEDA, German XDOMEA → RiC

## Sources

Built on the official ICA crosswalk published in **RiC-AG 0.1** (October 2025) and **RiC-CM 1.0** (November 2023), with RiC-O 1.1 predicate names.

## Installation files

The packaged `isadg-to-ric.skill` file and the complete `isadg-to-ric/` directory are both provided. Use your agent's supported skill installation method and keep the reference files with `SKILL.md`.

## Structure

- `isadg-to-ric/SKILL.md` — core guidance: conceptual shift, entity selection, structural rules, the 26-element quick mapping table
- `isadg-to-ric/references/crosswalk.md` — detailed per-element notes, edge cases, Turtle examples, common pitfalls
- `isadg-to-ric/references/entities.md` — the RiC-CM entity types explained in archival terms
- `isadg-to-ric/references/worked-example.md` — a complete ISAD(G) → RiC migration example
- `isadg-to-ric/references/nara-ric.md` — US federal records → RiC
- `isadg-to-ric/references/moreq-european-ric.md` — European EDRMS → RiC

## Disclaimer

RiC is a young and evolving standard. RiC-AG (the Application Guidelines) is still a draft, and RiC-O continues to develop. Treat the mappings here as informed guidance, verify against the current [ICA-EGAD](https://github.com/ICA-EGAD) sources for production work, and adapt to your institution's local practice.

## License

MIT — see [LICENSE](LICENSE).
