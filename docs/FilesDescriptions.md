# `docs/` — File Descriptions

**Purpose of this file.** A single orientation page for the `docs/` folder of the
**Cyclamen** project. It explains *what each document is*, *what question it answers*,
*who or what consumes it*, and *how the documents relate to one another*. It is written to be
read both by people joining the project and by LLM agents that need to pick the right
source document before answering a question or generating code.

**Maintenance rule.** Any file added to `docs/` should get an entry here in the same
change. If a description and the actual file disagree, the file wins — fix the description.

---

## 1. Domain context in one paragraph

The **Survey of Israel** (SoI / מרכז למיפוי ישראל, "מפ"י") receives cadastral plans from
licensed surveyors as **CAD files (DWG/DXF)**. The structure of those files is fixed by a
national specification called **Hanit 1.4** (חני"ת 1.4) — it dictates which *layers*,
*blocks* and *block attributes* must exist, per **process type** (תצ"ר, תת"ג, ה"ק / הסדר,
תיקון 97ב', פסק דין, רישום ראשון, תשא"ג, 3D / תמ"ר …). A submitted file is first
**validated** against a battery of automated checks, and only if it passes is its content
**migrated** into the National Cadaster **ESRI Geodatabase**. The documents in this folder
are the machine-readable specifications behind those three stages.

```
   Surveyor
      |  submits
      v
  DWG / DXF  --drawn according to-->  Hanit 1.4.gsheet  (+ Hanit 1.4 Metadata.gdoc)
      |                                    ^
      |                                    | same layer/block/attribute vocabulary
      v                                    |
  VALIDATION  --rules defined by----->  Tests.gsheet
      |
      |  pass
      v
  MIGRATION   --mapping defined by--->  Geodatabase Population.gsheet
      |                                        |  also defines the schema itself
      v                                        v
  InProcess* staging  --on registration-->  Recorded cadaster classes
      \_______________________  _______________________/
                              \/
        National Cadaster ESRI Geodatabase  (31 datasets, 27 domains)
```

The three primary spreadsheets share one **controlled vocabulary** — layer names
(`C1602_X`, `C1740`, `TASHAG_TABLE` …), block names (`C1603`, `C1744_HEADER` …) and
attribute names (`PARCEL_NAME`, `GUSH`, `CALC_AREA` …). That shared vocabulary is what lets
them be joined, cross-checked, and code-generated from.

---

## 2. At a glance

| File                                                                                                                                          | Stage           | Answers the question                                                  | Status                                                  |
| --------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------- | ------------------------------------------------------- |
| [Hanit 1.4.gsheet](https://docs.google.com/spreadsheets/d/1gCmvDhN8UGxfcnymE1nY6_35nLJqgoHLXYgrb72BqO4/edit)                                   | Authoring       | *How must the DWG be drawn?*                                        | Active —**verified**                             |
| [Hanit 1.4 Metadata.gdoc](https://docs.google.com/document/d/1UfwWQA9pkRNk9o1m35etap82OAnBRlQnY99SLtu2uO8/edit?tab=t.0#heading=h.3i740ep2glsa) | Authoring       | *What does each column of the above mean?*                          | Active —**verified**                             |
| [Tests.gsheet](https://docs.google.com/spreadsheets/d/1xAP9iNyEAopjPXgsLjLECWgxLYmJ0IPjBUSrgP_C66w/edit?gid=0#gid=0)                           | Validation      | *Is this submitted DWG acceptable?*                                 | **Early draft** — 8 of 14 checks written         |
| [Geodatabase Population.gsheet](https://docs.google.com/spreadsheets/d/1RD6Cyj8yDmQryHTWtnjanY7poYZRDdTztGhDtBt33V8/edit?gid=0#gid=0)          | Migration       | *What is the GDB schema, and where does each CAD value land in it?* | Schema**complete**; mapping **10% written** |
| [Hanit 1.4 Format/](<Hanit%201.4%20Format/>)                                                                                                   | Source material | *What does the officially published spec say?*                      | Reference                                               |
| [Older References/](<Older%20References/>)                                                                                                     | Prior art       | *How was this specified before?*                                    | Archive                                                 |

> **Note on `.gsheet` / `.gdoc` files.** These are Google Drive pointers, not local data.
> They hold no content on disk and cannot be opened by local tools (`cat`, `pandas`,
> `openpyxl`). Open them in Google Sheets/Docs, or export to `.xlsx`/`.csv` first if you
> need to process them programmatically.

> **Note on `.url` shortcut files.** `Hanit 1.4.url`, `Tests.url` and
> `Geodatabase Population.url` are plain-text Windows Internet Shortcuts
> (`[InternetShortcut]` / `URL=`) pointing at the edit URL of the matching Sheet — the same
> format already used by `Hanit 1.4 Format/Hanit page at SoI.url`. They are a local
> convenience for opening a sheet in a browser without going through the Drive file-stream
> integration. Unlike a `.lnk`, a `.url` file holds nothing but the target URL — no browser
> install path, no machine-specific content — so it is portable across machines. They carry
> no content of their own and are **not** a fourth source or source of truth; the three
> Sheets listed above remain authoritative. There is no corresponding `.url` for
> `Hanit 1.4 Metadata.gdoc` — only the three Sheets have one, not the Doc.

---

## 3. Primary documents

### 3.1 `Hanit 1.4.gsheet` — the CAD authoring specification

**Link**:
https://docs.google.com/spreadsheets/d/1gCmvDhN8UGxfcnymE1nY6_35nLJqgoHLXYgrb72BqO4/edit?usp=sharing

**What it is?** The working, table-form rendition of the **Hanit 1.4** specification: the
rules and guidelines for *how to draw (create) a cadastral plan as a DWG file*, differentiated
by the **process type** of the plan. It is the normative machine-readable schema for the whole
pipeline — not prose, but five joined tables.

**Sheet inventory** (verified against the live workbook):

| Tab                   | Rows      | Grain                                  | Columns                                                                                                                                                |
| --------------------- | --------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Layers`            | 74        | one CAD layer                          | `Process Type`, `Layer Name`, `Layer Description`, `Layer Content`, `Geometry Type`, `Mandatory`, `Condition`                            |
| `Blocks`            | 76        | one block                              | `Containing layer`, `Block Name`, `Content Description`, `Occurrence Rule`                                                                     |
| `Blocks Attributes` | 438       | one attribute of one block (71 blocks) | `Block Name`, `Attribute Name`, `Attribute Description`, `Data Type`, `Expected Values`, `Length\Precision`, `Allow Null`, `Condition` |
| `Codebook`          | 78        | one code of one domain (15 domains)    | `Name`, `Code`, `Value`                                                                                                                          |
| `GDB Process Types` | 16 +`*` | one process type                       | `Process Type Code`, `Process Type Description`, `Comment`                                                                                       |

**How process types are expressed.** Unlike the legacy matrices in `Older References/`, process
type is **not** one column per type. It is a single `Process Type` column on `Layers` holding a
comma-separated list of **numeric codes** drawn from `GDB Process Types` — e.g.
`1, 3, 4, 6, 9, 11, 15` — or `*` meaning *all plans* (כל התכניות). Any parser must split that
list and resolve the codes.

**Controlled vocabularies actually in use:**

- **`Layer Content`** — `Geometry` (15), `Info Block` (43), `Coordinates Block` (11), `Mixed` (5).
  This is the layer's *role*, and it is the distinction the migration depends on: geometry
  layers become Feature Class shapes, block layers become attribute rows.
- **`Geometry Type`** — populated only for geometry-bearing layers: `Closed 2D Polyline`,
  `2D Point`, `3D Point`, `3D Polyline`, `Closed 3D Polyline`, `Solid`, `Mesh`, `3D Surface`,
  `Line`, and comma-separated combinations where several are permitted.
- **`Mandatory`** — `True` (31), `False` (40), `Conditioanl` (3, *sic* — see §6.7).
- **`Occurrence Rule`** (Blocks) — how many instances of the block may appear on its layer:
  `1`, `1+`, `3+`, `0+`, `Total of 1`.
- **`Data Type`** (Attributes) — `String` (206), `Integer` (153), `Float` (53), `Date` (21).
- **`Allow Null`** (Attributes) — `True` / `False` / `Conditioanl`, blank where unspecified.

**The `Condition` columns carry real logic, not only prose.** Some are formal and
machine-readable — `C1602_X` reads
`{ProcessType: Value} {1: True, 3: True, 4: False, 6: True, 9: True, 11: True}`, i.e. whether
the layer is mandatory *varies per process type*. Others are Hebrew business rules, e.g.
`C1603.PARCEL_SOURCE` — *"mandatory for temporary base parcels in the existing state (stage 0);
parcel number in square brackets"*.

**`Expected Values` is the domain-binding column.** It holds either a literal constraint
(`> 0`, `1`, `/`) or a reference into the Codebook written as `Codebook['NAME']`. That
reference is the join from an attribute to its permitted code list.

**Codebook domains (15).** `PARCEL_STATUS`, `CLASS`, `MARK`, `SOURCE`, `REGISTER_STATUS`,
`DISC_CODE`, `POINT_DEBUG`, `PROCESS_TYPE`, `GRID`, `TARGET_VALUE_SOURCE`, `SCALE`,
`SUMMARIZE_ROW`, `DIFF_CHECK`, `GRID_NAME`, `RELATIVE_POSITION`.

**Process types — and the definition of Cyclamen's scope.** The `GDB Process Types` tab is the
authoritative enumeration, and its `Comment` column also **states what this project covers**:

| Code  | Process type                                                                      | In Cyclamen scope?              |
| ----- | --------------------------------------------------------------------------------- | ------------------------------- |
| 1     | תכנית לצרכי רישום (תצ"ר)                                        | ✅                              |
| 2     | תכנית מרחבית לצרכי רישום                                     | ❌*Not relevant for Cyclamen* |
| 3     | פסק דין                                                                     | ✅                              |
| 4     | תשריט תיעוד גבולות (תת"ג)                                      | ✅                              |
| 5     | קדסטר מבוסס קואורדינטות                                      | ❌*Retired Process Type*      |
| 6     | תיקון הסדר לפי סעיף 97ב                                          | ✅                              |
| 7     | תיקון תכנית לצרכי רישום                                       | ❌*Not in this Scope*         |
| 8     | תיקון/עדכון בעקבות פניית ציבור                          | ❌*Not in this Scope*         |
| 9     | הסדר מקרקעין                                                           | ✅                              |
| 10    | רישום ראשון בשטח לא מוסדר                                    | ✅                              |
| 11    | תכנית לצרכי רישום בשטח לא מסודר                         | ✅                              |
| 12    | תיקון תצ"ר בשטח לא מסודר                                       | ❌*Not in this Scope*         |
| 13    | תיקון רישום שטח וגבולות בשטח לא מוסדר              | ✅                              |
| 14    | תיקון/עדכון בעקבות פניית ציבור בשטח לא מוסדר | ❌*Not in this Scope*         |
| 15    | תשריט תיעוד גבולות להסדר מקרקעין                      | ✅                              |
| 16    | עריכה חופשית                                                           | ❌*Not relevant for Cyclamen* |
| `*` | כל התכניות — applies to all of the above                                | —                              |

**Cyclamen's in-scope process types are therefore 1, 3, 4, 6, 9, 10, 11, 13, 15** — nine of sixteen.

**Why it matters.** It is the contract with the surveyor. Every rule in `Tests.gsheet` is
ultimately an assertion that this contract was honoured, and every mapping in
`Geodatabase Population.gsheet` starts from a name defined here.

**Relationship to the published spec.** This workbook is a *restructuring*, not a copy, of
`Hanit 1.4 Format/Format Table.xlsx`. The published table has two sheets keyed on
layer → block → attribute, carrying a **visibility** (נראה/בלתי נראה) column and a `LINETYPE`
column. This workbook drops both, promotes blocks to their own tab with an occurrence rule, and
adds `Data Type`, `Expected Values`, `Allow Null`, `Condition`, the `Codebook` and the
process-type enumeration. **Where they disagree, this workbook is the project's working truth
and `Format Table.xlsx` is the published source** — but the divergence is worth checking rather
than assuming either is stale.

---

### 3.2 `Hanit 1.4 Metadata.gdoc` — the data dictionary for §3.1

**Link**: https://docs.google.com/document/d/1UfwWQA9pkRNk9o1m35etap82OAnBRlQnY99SLtu2uO8/edit?usp=sharing

**What it is.** The **metadata / schema description** for `Hanit 1.4.gsheet`: a Hebrew document
that walks the workbook tab by tab and defines the meaning and permitted values of **every
column**. Verified against the live document — it covers exactly the five tabs described in
§3.1, in the same order, with no drift.

**Structure.** One section per tab, each opening with a short statement of the tab's purpose and
followed by a two-column `Column Name` / `Description` table.

**Definitions it supplies that are not recoverable from the workbook itself.** This is the
reason to read it before writing any parser:

- **The `X` suffix in layer names is the *action number*.** Not a generic placeholder: it is an
  integer from 1 upward identifying the action within the plan. `C1602_1` is the 2D parcel-border
  layer *after the first action*. `C1602_0` is therefore the existing (pre-action) state — which
  is what makes the iteration in §3.4's `InProcessParcels2D` mapping ("loop through 0 state to
  n state of actions") intelligible.
- **`Layer Content` values, precisely.** `Geometry` = geometric entities only. `Info Blocks` =
  blocks carrying alphanumeric attributes where **placement is irrelevant**. `Coordinates Block` =
  blocks carrying attributes where **placement is meaningful** (a parcel border point, say).
  `Mixed` = both blocks and geometry permitted. The Info/Coordinates split is a positional
  semantics distinction, not a content one, and nothing in the workbook states it.
- **`Length\Precision` is type-dependent.** For `Integer` it is the **digit count** (a limit of 1
  permits 1–9). For `Float` it is the number of **decimal places**. For `String` it is the
  **character count**, with spaces and niqqud each counting as a character.
- **`Occurrence Rule` full vocabulary** — `1` (exactly once), `0+` (optional, any number),
  `1+` / `2+` / `3+` (at least that many), and `Total of 1` = *one block in total on that layer*,
  for layers where several block types are permissible but only one may actually appear (the
  surveyor-declaration block in a תצ"ר or תת"ג).
  Note `2+` is documented here but unused in the current workbook.
- **`Condition` is scoped.** It applies **only** where `Mandatory` (Layers) or `Allow Null`
  (Attributes) is `Conditional`. A `Condition` cell on a row that is plainly `True`/`False` is
  not a rule to evaluate.
- **Empty `Expected Values` means unconstrained** — any value is acceptable. It does not mean
  "not yet specified".

> Filename note: this is a Google **Doc** (`.gdoc`), not a Sheet. Earlier drafts referred to it
> as `Hanit 1.4 Metadata.gsheet`. Export it with `?format=docx`, not `?format=xlsx`.

---

### 3.3 `Tests.gsheet` — the validation / QA rule catalogue

**Link**: https://docs.google.com/spreadsheets/d/1xAP9iNyEAopjPXgsLjLECWgxLYmJ0IPjBUSrgP_C66w/edit?usp=sharing

**What it is.** The catalogue of **every automated check** applied to a submitted DWG before it
is accepted. Each row is one check, defined declaratively so it can be implemented, tracked and
reported against. It is the intended single consolidated list superseding the fragmented
predecessors in [`Older References/`](#42-older-references--prior-art).

**Status: early. 8 checks defined out of 14 numbered slots.** IDs 9–14 are empty placeholders,
and ID `14.0` appears twice. Treat this workbook as a *design in progress*, not a specification
to build against — but note that the **column design is settled** and the eight written checks
establish the conventions clearly.

**Sheet inventory** (verified against the live workbook):

| Tab                   | Rows                       | Grain            | Columns                                                                                                                                                                         |
| --------------------- | -------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Tests`             | 8 defined + 6 placeholders | one check        | `ID`, `Name`, `ProcessType`, `Category`, `Sevirity`, `Description`, `Process Type Context`, `Dependencies`, `Message`, `Message Formatting`, `Guidelines` |
| `GDB Process Types` | 16 +`*`                  | one process type | `Process Type Code`, `Process Type Description`, `Comment`                                                                                                                |

The `GDB Process Types` tab is a **duplicate** of the tab of the same name in `Hanit 1.4.gsheet`
— identical content, maintained separately in both workbooks. See §7.

**Column semantics.**

- **`ProcessType`** — the same convention as §3.1: a comma-separated list of numeric codes
  (`1, 11`) or `*` for all plans. **Not** one column per process type; the legacy applicability
  matrix shape from `Older References/רקפת/` was *not* carried forward.
- **`Category`** — the check taxonomy. In use so far: `Format` (5), `Geometric` (2)… with the
  predecessor workbooks suggesting `Topology`, `Logic` and others will follow.
- **`Sevirity`** *(sic — misspelled in the header)* — `Fatal`, `Critical`, `Info`. This is a
  **three-level scale and a new vocabulary**; it does not match the `Critical`/`Stop`/`Report`
  or `קריטית`/`דיווח` grading of the predecessor workbooks, and there is no `Importance` (`A`–`D`)
  column. Do not assume the old severity semantics map onto these names.
- **`Process Type Context`** — a machine-readable per-process-type parameter table, letting one
  check row cover many process types with different operands. Two shapes are in use:
  `{Process Type: Block} {1: C1642, 2: C1742, 3: C1635, …}` and
  `{Process Type: Action Type} {1: [חלוקה, איחוד, העברה], …}`.
  This is the mechanism that replaces the legacy per-type columns.
- **`Dependencies`** — the ID of a check that must pass first (check 6 depends on 4; check 7 on 5).
  Checks therefore form a DAG, and a runner must respect the ordering rather than evaluating
  rows independently.
- **`Message`** / **`Message Formatting`** — the Hebrew text shown to the surveyor, plus a legend
  naming each interpolated placeholder (`X: Process Type, Y: Block`). Placeholders appear both
  bare (`שכבת חובה Layer חסרה`) and brace-delimited (`בלוק {Block} חסר`) — **the two styles are
  not yet consistent**, which matters for anything doing the substitution.
- **`Guidelines`** — remedial advice for the surveyor: how to fix the failure, not what it is.

**The eight defined checks**, as a picture of the intended coverage:

| ID | Check                                                                      | Process types | Category  | Severity |
| -- | -------------------------------------------------------------------------- | ------------- | --------- | -------- |
| 1  | Declared process type matches the drawing (via a signature block per type) | `*`         | Format    | Fatal    |
| 2  | Plan boundary does not cross the borders of Israel                         | `*`         | Geometric | Critical |
| 3  | Work area lies more than 500 m from the national border                    | `*`         | Geometric | Info     |
| 4  | All mandatory (and conditionally mandatory) layers are present             | `*`         | Format    | Fatal    |
| 5  | Each layer holds only its permitted`Layer Content` type                  | `*`         | Format    | Fatal    |
| 6  | Each layer holds the required block count per`Occurrence Rule`           | `*`         | Format    | Fatal    |
| 7  | Block attribute values conform to`Blocks Attributes`                     | `*`         | Format    | Critical |
| 8  | A תצ"ר carries at least one complete action table                       | `1, 11`     | Format    | Critical |

**Note how tightly this binds to §3.1.** Checks 4, 5, 6 and 7 are each a direct assertion over
one tab of `Hanit 1.4.gsheet` — `Layers.Mandatory`, `Layers.Layer Content`, `Blocks.Occurrence Rule`, `Blocks Attributes` respectively, and the `Message Formatting` column names those tabs and
columns explicitly. The validator is, in large part, an interpreter of the Hanit workbook.

**Two checks reach outside the DWG.** Checks 2 and 3 compare against a separate internal
national-boundary file, described in the sheet as *"a separate in-system file intended for this
check"*. Comparison against בנק"ל data — prominent in the predecessor workbooks — has **not** yet
appeared here.

**Why it matters.** This is the executable definition of "valid submission". Severity is
operationally load-bearing: `Fatal` / `Critical` reject, `Info` only warns.

---

### 3.4 `Geodatabase Population.gsheet` — the Geodatabase schema and CAD → GDB mapping

**Link**: https://docs.google.com/spreadsheets/d/1RD6Cyj8yDmQryHTWtnjanY7poYZRDdTztGhDtBt33V8/edit?usp=sharing

**What it is.** Two things in one workbook, and the distinction matters:

1. **A complete definition of the target Geodatabase** — every table, every field, every coded
   domain. This part is finished.
2. **The CAD → GDB population mapping** — which DWG layer, block and attribute feeds each field.
   This part covers **61 of 603 fields (10%)** and is actively being written.

It is organised **GDB-first**: one row per *target field*, with the CAD source attached as
columns. This inverts the predecessor `Dictionary` sheet in `Older References/דקל/`, which was
CAD-first (`CAD Block Layer | CAD Block | … | GIS Table | GIS Field`). Do not expect the old
shape.

**Sheet inventory** (verified against the live workbook):

| Tab               | Rows | Grain                               | Columns                                                                                                                                                                                                                          |
| ----------------- | ---- | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Tables`        | 31   | one GDB table                       | `Name`, `Alias`, `Dataset Type`, `Geometry Type`, `Description`                                                                                                                                                        |
| `Field Mapping` | 603  | one field of one table              | `Dataset`, `Name`, `Alias`, `Type`, `Nullable`, `Domain`, `Default Value`, `Length`, `Population`, `Relevant CAD Layer`, `Block Name\Geometry Type`, `Population Description`, `Overwrite Description` |
| `Domains`       | 237  | one code of one domain (27 domains) | `Domain Name`, `Code`, `Description`                                                                                                                                                                                       |

**The target Geodatabase: 31 datasets** — 27 Feature Classes (10 Polygon, 7 Polyline, 6 Point,
4 Multi Patch) and 4 Tables. They fall into five families:

| Family                      | Datasets                                                                                                                                                                                                                      | Role                                                                                                       |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Recorded cadaster** | `Blocks`, `BlocksFronts`, `Parcels2D`, `Parcels2DFronts`, `BorderPoints`, `Parcels3D`, `BorderPoints3D`, `Substractions`, `ProjectedParcels3D`, `ProjectedSubstractions`                                  | The authoritative registered state                                                                         |
| **In-process**        | `InProcessParcels2D`, `InProcessFronts`, `InProcessBorderPoints`, `InProcessParcels3D`, `InProcessBorderPoints3D`, `InProcessSubstractions`, `InProcessProjectedParcels3D`, `InProcessProjectedSubstractions` | The submitted plan's content, staged before recording                                                      |
| **Process control**   | `CadasterProcessBorders`, `CadasterRecordsBorders`, `CPBStatusAndDates`, `SequenceActions`, `TaskHistory`                                                                                                           | Which process, its status, its sequence of actions                                                         |
| **Validation output** | `GDB_ValidationPointErrors`, `GDB_ValidationLineErrors`, `GDB_ValidationPolygonErrors`, `GDB_ValidationObjectErrors`                                                                                                  | Where check failures are written back, geometry included                                                   |
| **Adjustment**        | `AdjustmentPoints`, `AdjustmentLines`, `AdjustmentVectors`, `ConnectionsLines`                                                                                                                                        | Coordinate-adjustment support —**three are marked "הטבלה אינה בשימוש"** (not in use) |

The **In-process → Recorded** split is the central architectural fact: a validated DWG lands in
the `InProcess*` classes carrying a `CPBUniqueID` back to its `CadasterProcessBorders` row, and is
promoted to the recorded classes only on registration. `Recorded` is a Boolean field on the
in-process classes, and the mapping specifies it as **constant 0 on load**.

**Scope answered: the GDB is 2D *and* 3D.** `Parcels2D` / `Parcels3D`, `BorderPoints` /
`BorderPoints3D`, and the `Substractions` (גריעות) Multi Patch classes all coexist. This
supersedes the earlier inference in §4.2 that the target was 3D-only.

**How a population rule is written.** The `Population` flag marks a field as CAD-fed; the mapping
is then carried across three columns:

- **`Relevant CAD Layer`** — per-process-type, in the same brace notation as §3.3:
  `{Process Type: CAD Layer} : {[1: C1602_X], [3: C1602_X], [4: C1602_0], [9: C1602_1], …}`
- **`Block Name\Geometry Type`** — the block (`C1642\C1644\C1645`) or the CAD geometry
  (`Polyline (Closed)`) supplying the value.
- **`Population Description`** — the transformation, in prose but precise. These are real ETL
  instructions, not notes. Examples:
  - *"Aggregated closed polygon geometry of all polygons found in the parcels layer according to
    the layers mapping. Multiparts allowed."* (`CadasterProcessBorders.Shape`)
  - *"The GUID value returned when querying PF.Blocks with `RetiredByRecord Is Null And BlockNumber = <GUSH_NUM> And SubBlockNumber = <SubBlockNumber>`. If sub block number doesn't
    exist, use `SubBlockNumber = 0`."* (`BlockUniqueID`, a **lookup join**, not a copy)
  - *"A text-stripping function must handle sub-block numbers greater than zero. If a sub-block
    number is present, extract the first integer before the slash (/) character."* (`BlockNumber`)
  - *"{ProcessType: Status} : {[1: 1], [3: 11], [4: 1], [9: 91], …}"* (`Status`, a constant that
    varies by process type)
  - Plain constants: `GeodeticNetwork` = 3, `DataSource` = 5, `IsTax` = 0.

**Which datasets have mapping so far** — six: `CadasterProcessBorders`, `CPBStatusAndDates`,
`SequenceActions`, `InProcessParcels2D`, `InProcessFronts`, `InProcessBorderPoints`. That is
precisely the **2D submission path**, which tells you the build order. 34 of the 61 flagged
fields still have an empty `Population Description`.

**Domains: 27, all clean.** Every domain referenced by a field is defined, and every defined
domain is used — no dangling references in either direction (contrast §6.7, where the Hanit
Codebook has one). `ProcessType` (16 codes) mirrors the process-type enumeration; `ProcessStatus`
(33) and `PF_AreaUnits` (66) are the largest. **There is no column linking a GDB domain to its
Hanit `Codebook` counterpart, and none is wanted** — the two are independent vocabularies by
design; see §7 (Resolved).

**`Overwrite Description`** governs re-ingest: whether a reload preserves an existing value. Only
one field currently sets it (*"Ignore to maintain the current value"*).

**Why it matters.** This is the single source of truth for the migration code, and — via the
`Tables`/`Field Mapping`/`Domains` triple — for creating the Geodatabase schema in the first
place. Without it a DWG is just geometry; with it, the DWG becomes rows in the cadastral database.

---

## 4. Supporting material

### 4.1 `Hanit 1.4 Format/` — the published specification

The upstream, authoritative material that §3.1 and §3.2 are derived from. **Treat these as
read-only source; do not edit them to record project decisions.**

| Item                                             | Description                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Hanit 1.4 document.pdf`                       | The officially published Hanit 1.4 specification.                                                                                                                                                                                                                                                                    |
| `Hanit 1.4 document.docx`                      | Editable copy of the same. Covers the spec's goals and principles (geographic data delivered as DWG/DXF per DXF 2004; no bare`TEXT` entities — captions go through dedicated blocks; 1 mm tolerance; no dependency on specific CAD software or versions), then a section per layer family for 2D and 3D cadaster. |
| `Format Table.xlsx`                            | The spec's two normative tables as a spreadsheet:*רשימת שכבות ובלוקים* (layers and blocks) and *רשימת שדות בבלוקים* (block attributes). The direct ancestor of `Hanit 1.4.gsheet`.                                                                                            |
| `CAD Blocks/mifratML_TEMPLETE_CADASTRE_2D.zip` | Official 2D CAD template with the block definitions pre-built.                                                                                                                                                                                                                                                       |
| `CAD Blocks/mifratML_TEMPLETE_CADASTRE_3D.zip` | Official 3D CAD template.                                                                                                                                                                                                                                                                                            |
| `Hanit page at SoI.url`                        | Shortcut to the spec's home page:[https://www.gov.il/he/pages/mml1-4](https://www.gov.il/he/pages/mml1-4)                                                                                                                                                                                                             |

### 4.2 `Older References/` — prior art

Earlier, superseded specifications kept for traceability. **Do not implement from these** —
use them to understand history, recover rationale, or verify that a rule was not dropped by
accident while consolidating into `Tests.gsheet` and `Geodatabase Population.gsheet`.

**`Older References/דקל/`** — the *Dekel* line, oriented to **3D cadaster (תמ"ר)**:

- `Migration tests dictionary 5.6.24.xlsx` — the most complete predecessor and the closest
  structural template for today's files. Six sheets: `Layers & Blocks`, `Blocks Attributes`,
  `Checks`, **`Dictionary`** (the CAD → GIS crosswalk, direct ancestor of
  `Geodatabase Population.gsheet`), `Values` (controlled vocabularies) and `LUTs` (10
  coded-value domains). Target Feature Classes and tables include `CadasterProcessBorders3D`,
  `CPBStatusAndDates`, `Parcels3D`, `Substractions`, `SequenceActions`, `BorderPoints3D`.
- `Migration tests dictionary 12.3.26.xlsx` — later revision, `Checks` sheet only (~170 checks).
- `Advanced tests dictionary 12.3.26.xlsx` — advanced cross-entity logical checks.
- `Advanced Comparison checks dictionary 12.3.26.xlsx` — the above plus a **`Comparison Checks`**
  sheet validating the submission against existing national data (בנק"ל).

**`Older References/רקפת/`** — the *Rakefet* line, oriented to **2D process types**:

- `רקפת - איפיון טבלה.xlsx` — check specification table: existing check ID, check type,
  severity (old and revised), characterisation, description, participating layers / blocks /
  fields, and old versus new error message text. Notably includes the mapping from **legacy
  check numbers** to the new numbering — the key to migrating without losing coverage.
- `מטריצת בדיקות לטופוקד…xlsx` — the **applicability matrix**: one row per check, one column
  per process type (תצ"ר, תת"ג, הסדר, 97ב, פסק דין, תת"ג להסדר, תצ"ר בשטח לא מוסדר), marked
  `X` where the check applies. Conceptually the ancestor of applicability in `Tests.gsheet`,
  though **the shape was not carried forward** — `Tests.gsheet` uses a numeric code list and the
  `Process Type Context` brace notation instead. Useful as a coverage checklist, not as a template.

---

## 5. Glossary

| Term                               | Meaning                                                                                                                                                                           |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hanit / חני"ת**        | The national CAD specification for cadastral maps submitted to SoI. Current version 1.4.                                                                                          |
| **SoI / מפ"י**            | Survey of Israel, המרכז למיפוי ישראל — the national mapping and cadastre authority.                                                                              |
| **Gush / גוש**            | Cadastral block — the primary land-registration grouping.                                                                                                                        |
| **Helka / חלקה**         | Parcel — a single registered land unit within a *block*.                                                                                                                      |
| **תצ"ר**                  | Parcellation plan; a*dynamic* process, it changes parcel boundaries.                                                                                                            |
| **תת"ג**                  | Block boundary determination plan (*static*).                                                                                                                                   |
| **ה"ק / הסדר**         | Land settlement / settlement of title.                                                                                                                                            |
| **תיקון 97ב'**         | An amendment-97b correction plan.                                                                                                                                                 |
| **פסק דין**            | Court-judgment-driven cadastral change.                                                                                                                                           |
| **רישום ראשון**    | First registration.                                                                                                                                                               |
| **תשא"ג**                | Correction of registered area and boundaries.                                                                                                                                     |
| **תמ"ר**                  | 3D cadastral plan — three-dimensional parcels and subtractions.                                                                                                                  |
| **גריעה / Subtraction** | A volume removed from a 3D parcel.                                                                                                                                                |
| **בנק"ל**                | The national cadastral data bank, used as the comparison baseline.                                                                                                                |
| **Feature Class**            | An ESRI Geodatabase table holding geometry — the migration target for CAD layers.                                                                                                |
| **LUT / Domain**             | Lookup table — a coded-value domain in the Geodatabase.`Geodatabase Population.gsheet` calls these *Domains*; the CAD-side equivalent is the Hanit *Codebook*.             |
| **CPB**                      | `CadasterProcessBorders` — the polygon representing one cadastral process. Its GUID (`CPBUniqueID`) is the foreign key that ties every in-process feature to its submission. |
| **In-process**               | The`InProcess*` Feature Classes: a validated submission staged in the Geodatabase but not yet registered. Promoted to the recorded classes on registration.                     |
| **Block / Block Attribute**  | CAD constructs. Hanit carries all textual and tabular data as block attributes rather than free`TEXT` entities.                                                                 |

---

## 6. Conventions for working with these documents

1. **Names are the join keys.** Layer, block and attribute names are the shared identifiers
   across all three primary spreadsheets. Never rename one in isolation.
2. **Process types are numeric codes.** In `Hanit 1.4.gsheet` and `Tests.gsheet` alike, the
   process-type column holds a comma-separated code list (or `*` for all), resolved against the
   `GDB Process Types` tab. The legacy workbooks in `Older References/` instead use one column
   per process type — do not copy that shape forward without converting.
3. **Brace notation is the shared per-process-type idiom.** All three primary workbooks express
   "this value varies by process type" the same way:
   `{Process Type: <Thing>} {1: …, 3: …, 4: …}` (with a `:` before the braces and `[…]` around
   each entry in `Geodatabase Population.gsheet`). It appears in `Condition`, `Process Type Context`, `Relevant CAD Layer` and `Population Description`. Write one parser for it.
4. **Placeholders are part of the name.** `X`, `NameP3D`, `NameG`, `NameP` in layer names are
   variables, not literals, and must be expanded. Per §3.2, **`X` is the action number** within
   the plan — `C1602_0` is the existing state, `C1602_1` the state after the first action.
5. **Hebrew is content, not decoration.** Descriptions, conditions and error messages are
   Hebrew and user-facing. Read and write these files as UTF-8.
6. **`Condition` cells are mixed-mode.** Some hold a machine-readable expression
   (`{ProcessType: Value} {1: True, …}`), others free Hebrew prose. Detect the formal shape
   before attempting to parse; never assume a `Condition` cell is executable.
7. **`.gsheet` / `.gdoc` are pointers.** They hold no bytes on this Drive mount and cannot be
   read by local tools — the failure is at the filesystem driver, so sharing settings do not
   help. Fetch a link-shared file directly instead, minding the type:
   `https://docs.google.com/spreadsheets/d/<ID>/export?format=xlsx` for a Sheet, and
   `https://docs.google.com/document/d/<ID>/export?format=docx` for a Doc.
   **Keep the URLs in this file** — they are the only record of the file IDs.
8. **Known data-quality issues in `Hanit 1.4.gsheet`** (verified 2026-09-07 — fix at source rather
   than working around them in code; see each item for its current status):
   - `Conditioanl` is misspelled in both the `Mandatory` column (3 rows) and the `Allow Null`
     column (3 rows). A parser matching `Conditional` will silently miss them.
   - The Codebook reference prefix is spelled two ways — `CodeBook[…]` (44 uses) and
     `Codebook[…]` (18 uses). Match case-insensitively.
   - ~~**Broken reference:** `C1603.PARCEL_STATUS` points to `Codebook['PARCEL_SOURE']`, a domain
     that does not exist in the `Codebook` tab.~~ **Fixed at source in `Hanit 1.4.gsheet`** per the
     user's recorded decision (2026-10; `ClaryfingQuestions.html` Q2.4, and `CLAUDE.md`
     convention 9). Corrected in the same edit: every attribute whose `Expected Values` references
     a Codebook domain now has `Allow Null = False` (`ClaryfingQuestions.html` Q2.8). Both fixes
     are recorded from the user's decision and have **not** been re-verified against live sheet
     content. At the time of the 2026-09-07 reading, all other 61 references resolved and all 15
     defined domains were referenced at least once.
   - Two unresolved questions are embedded in the sheet itself rather than tracked elsewhere:
     `C1603.PARCEL_STATUS` — *"verify with Shimon whether the field may be left empty"*; and
     layer `C1609_0`, whose condition notes the rule is not implementable as stated because it
     requires checking border-point blocks and classification before front-line existence can
     be evaluated.
9. **Severity drives behaviour**, and the vocabulary changed. `Tests.gsheet` grades checks
   `Fatal` / `Critical` / `Info` — the first two reject the submission, `Info` only warns. The
   predecessor workbooks used `Critical`/`Stop`/`Report` and `קריטית`/`דיווח`; do not assume the
   old names map cleanly onto the new three. Note the header is misspelled `Sevirity`.

---

## 7. Open questions

Items to confirm and fold back into this document.

**Resolved** (verified by reading all four primary Google files, 2026-09-07, except where an item
cites a later user decision):

- [X] Sheet and column inventory of all four primary documents — §3.1–§3.4 are now verified
  against live content, not inferred.
- [X] Cyclamen's process-type scope — codes **1, 3, 4, 6, 9, 10, 11, 13, 15**; the remaining
  seven are marked *Not relevant for Cyclamen*, *Retired* or *Not in this Scope*. This also
  confirms **Cyclamen is the project's own name**, used inside the specification.
- [X] Does `Hanit 1.4 Metadata.gdoc` still match the five-tab structure? **Yes** — it covers
  exactly those five tabs, in order, with no drift.
- [X] Does the target Geodatabase extend beyond 3D to the 2D cadastral Feature Classes?
  **Yes** — 31 datasets covering both, and the mapping written so far is entirely the 2D path.
- [X] Does `Tests.gsheet` use the legacy one-column-per-process-type matrix? **No** — it uses the
  numeric code list plus the `Process Type Context` brace notation.
- [X] **Is a crosswalk needed between the Hanit `Codebook` (15 domains) and the GDB `Domains` (27)?**
  **No — and one must not be built.** Per the user's recorded decision
  (`ClaryfingQuestions.html` Q2.4), the two belong to different schemas with different structures
  and are **independent vocabularies**; do not join or equate them. Several pairs look as though
  they correspond — `CLASS`→`Class`, `MARK`→`MarkCode`, `SOURCE`→`DataSource`,
  `PROCESS_TYPE`→`ProcessType`, `GRID`→`GeodeticNetwork`, `RELATIVE_POSITION`→`RelativePosition` —
  but the resemblance is conceptual only and the code values are **not** guaranteed identical.
  Treat it as a trap, not a mapping. See `CLAUDE.md` convention 8.

**Still open — content questions:**

- [ ] **`Tests.gsheet` is 8 checks of a planned 14+.** — *tracked as Q3.1.* The predecessors hold
  ~170 (`Migration tests dictionary`) and a full 2D matrix (`רקפת`). What is the intended final
  size, and is there a migration plan from the legacy check numbering — the
  `רקפת - איפיון טבלה.xlsx` old→new mapping — so coverage is not silently dropped?
- [ ] **`Tests.gsheet` has no `ID 14` deduplication and no `Importance` column.** Was the
  `Mandatory`/`A`–`D` importance grading from the predecessors deliberately dropped in favour
  of the three-level `Sevirity`, or is it still to be added?
- [ ] **Comparison against בנק"ל is absent from `Tests.gsheet`.** — *tracked as Q10.4.* The
  predecessor workbooks devote a whole sheet to it (gush validation, base-parcel reuse). In scope
  for Cyclamen?
- [ ] **`Geodatabase Population.gsheet` has unresolved cells.** — *tracked as G2.*
  `CadasterProcessBorders` rows for process types **3 (פסק דין)** and **6 (תיקון הסדר 97ב)** carry
  `??????` in place of a block name, across `ProcessName`, `ProcessNumber`, `ProcessYear` and
  `SurveyorLicenseID`. Which blocks supply these for those two process types?
- [ ] **34 of the 61 `Population=True` fields have an empty `Population Description`**
  — *tracked as Q4.4.* Is a blank there "straight copy from the named block attribute", or "not
  yet written"? The two need distinguishing before anyone codes from this sheet.
- [ ] Reconcile the divergence between `Hanit 1.4.gsheet` and the published
  `Format Table.xlsx` (dropped `visibility` and `LINETYPE` columns) — deliberate, or drift?
- [ ] Confirm whether *דקל* is a separate SoI effort (3D / תמ"ר migration) and how it relates
  to Cyclamen — §4.2 currently states this as an inference from the folder contents.

**Still open — process questions:**

- [ ] **`GDB Process Types` is duplicated** in `Hanit 1.4.gsheet` and `Tests.gsheet`, maintained
  by hand in both. Which is authoritative, and how is drift prevented? — *tracked as G3.*
- [ ] **Three adjustment Feature Classes are marked "הטבלה אינה בשימוש"** (`AdjustmentVectors`,
  `AdjustmentLines`, `ConnectionsLines`). Retain in the schema, or retire? — *tracked as G1.*
- [ ] Record the versioning and change-control process for the four primary Google files.
