# Cyclamen

A specification repository for the **Survey of Israel** (SoI / מפ"י) CAD → Geodatabase
pipeline. There is no source code here — the deliverable is a set of machine-readable
specifications that will drive a future validation + migration engine:

```
Surveyor submits DWG/DXF  →  VALIDATION  →  MIGRATION  →  National Cadaster ESRI Geodatabase
        drawn per Hanit 1.4                 (Tests)       (Geodatabase Population)
```

The engine processes one DWG file at a time: all validation checks run first, and
migration (conversion + GDB insertion) proceeds only if every Critical/Fatal check passes.
On insertion failure the engine rolls back; a file can always be reprocessed.

## The four primary documents

These `.gsheet` / `.gdoc` files are Google Drive pointers — open them in Google
Sheets/Docs, or see [CLAUDE.md](CLAUDE.md) for how to export them to local `.xlsx`/`.docx`.

| Document | Role | Status |
|---|---|---|
| [Hanit 1.4](https://docs.google.com/spreadsheets/d/1gCmvDhN8UGxfcnymE1nY6_35nLJqgoHLXYgrb72BqO4/edit?usp=sharing) | How a DWG must be drawn — layers, blocks, attributes, codebook | Verified |
| [Hanit 1.4 Metadata](https://docs.google.com/document/d/1UfwWQA9pkRNk9o1m35etap82OAnBRlQnY99SLtu2uO8/edit?usp=sharing) | Data dictionary for the above | Verified |
| [Tests](https://docs.google.com/spreadsheets/d/1xAP9iNyEAopjPXgsLjLECWgxLYmJ0IPjBUSrgP_C66w/edit?usp=sharing) | The validation rule catalogue | Early draft — 8 of 14 checks |
| [Geodatabase Population](https://docs.google.com/spreadsheets/d/1RD6Cyj8yDmQryHTWtnjanY7poYZRDdTztGhDtBt33V8/edit?usp=sharing) | GDB schema and the CAD → GDB mapping | Schema complete; mapping ~10% |

These three spreadsheets share one controlled vocabulary — layer names (`C1602_X`),
block names (`C1603`), attribute names (`PARCEL_NAME`) — which is what lets them be
joined and eventually code-generated from.

## Where to go next

- **[CLAUDE.md](CLAUDE.md)** — the full project brief: architecture, conventions that
  bite, source hierarchy, and the fetch procedure for the Google files.
- **[docs/FilesDescriptions.md](docs/FilesDescriptions.md)** — the orientation page for
  `docs/`: verified sheet/column inventory of every file, and the open content questions.
- **[ClaryfingQuestions.html](ClaryfingQuestions.html)** — open and answered questions
  about the *future engine's* behavior (one DWG at a time vs. batch, rollback semantics,
  which DWG library, where the GDB lives).

## Repository layout

```
docs/
  Hanit 1.4.gsheet, Hanit 1.4 Metadata.gdoc, Tests.gsheet, Geodatabase Population.gsheet
  Hanit 1.4 Format/      — published upstream spec (read-only source)
  Older References/      — superseded prior art, not to be implemented from
  FilesDescriptions.md   — orientation page for docs/
answers/                 — exported/auto-saved answers from ClaryfingQuestions.html
CLAUDE.md                — project instructions for Claude Code
ClaryfingQuestions.html  — the engine-behavior Q&A bank
```
