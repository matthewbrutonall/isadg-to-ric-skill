# ISAD(G) to RiC Conversion Skill for Claude Code

![Archival hierarchy becoming a network of records, people, places, and dates](assets/isadg-to-ric-skill.png)

A practical skill for [Claude Code](https://claude.ai/code) that helps archivists migrate descriptions from **ISAD(G)** to **Records in Contexts (RiC-CM 1.0)** — the new International Council on Archives (ICA) standard for archival description.

This is a focused conversion and migration tool, not a general RiC tutorial. It answers the question every archivist facing RiC actually asks: *"My catalogue is in ISAD(G) — what does each field become in RiC?"*

## Install with your AI agent

Copy and paste this into your agent:

```text
Install the ISAD(G) to RiC skill from https://github.com/matthewbrutonall/isadg-to-ric-skill using your agent's supported skill installation method. Use the packaged skill or keep the complete `isadg-to-ric` directory, including its `references`, together. Confirm the skill is available and tell me how to invoke it. Do not convert any catalogue data yet.
```

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

## Install

```bash
claude skills install https://github.com/matthewbrutonall/isadg-to-ric-skill
```

Or download `isadg-to-ric.skill` from this repository and install it directly.

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
