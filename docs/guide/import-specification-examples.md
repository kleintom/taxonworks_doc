---
sidebarPosition: 61
pageClass: wide-page
---

# Import Specification Examples

_Worked examples of Darwin Core Occurrence import behavior, generated from and verified against the TaxonWorks test suite. See [Import](/guide/import) for the general import guide, including [DwC term mapping](/guide/import#dwc-term-mapping) and [Configure Settings in the import task](/guide/import#configure-settings-in-the-import-task)._

The examples below are "specification specs": each one pairs a minimal, single-purpose input file with an automated test asserting exactly what the importer does with it. They're generated directly from the TaxonWorks test suite, so if the importer's behavior ever changes, these examples (and their automated tests) will be updated together. This differs from the batch loader test files used elsewhere in TaxonWorks' test suite, which tend to be drawn from real (sometimes messy) production datasets and pin down bug fixes rather than illustrate one concept at a time.

Each example follows the same template:
- **Test spreadsheet** / **Test code** &mdash; links to the fixture file and to the RSpec context that asserts this example's behavior (GitHub, plus a local link &mdash; see tip below).
- **Input** &mdash; what must already exist in the project's database before the import (beyond a fresh project) for the scenario to apply, one bullet per item.
- **Settings** &mdash; the DwC importer `Settings` used, if any differ from the defaults.
- **The spreadsheet**, shown with a narrow blank column separating its input columns from a set of shaded outcome columns appended on the right. The `status` column uses the same colors as the row status in the importer's own UI (<span style="color: var(--color-import-imported); font-weight: 600;">Imported</span>, <span style="color: var(--color-import-errored); font-weight: 600;">Errored</span>, <span style="color: var(--color-import-not-ready); font-weight: 600;">NotReady</span>, <span style="color: var(--color-import-unsupported); font-weight: 600;">Unsupported</span>); the remaining outcome columns are counts of what got created (TaxonNames, CollectionObjects, TaxonDeterminations, etc., as relevant to the example) &mdash; so you can tell what happened at a glance, without reading prose.
- **Notes** &mdash; anything about the outcome that doesn't reduce to a single column value (e.g. relationships between the records that got created).

Examples are grouped the same way the [DwC term mapping](/guide/import#dwc-term-mapping) tables are: [Record-level class](/guide/import#record-level-class) first, then [Occurrence class](/guide/import#occurrence-class), then Event class, and so on &mdash; plus a **Matching** group of its own for cross-cutting matching/disambiguation behavior (nomenclature matching, person matching, containerization, etc.) that doesn't belong to any single term, and a **Settings** group covering the DwC importer's `Settings` toggles.

These examples describe how a conforming DwC occurrence importer is expected to behave, not necessarily everything about how this particular installation currently behaves. Where TaxonWorks is known to fall short of the behavior described, that's called out with a red **danger** notice rather than folded silently into the example as if it were correct.

::: tip
The "locally" links below point at `http://localhost:4747/...`, a tiny static file server rooted at the taxonworks2 checkout (browsers block `http://` pages from linking directly to `file://` paths, so this is the workaround). Start it with `python3 -m http.server 4747 --bind 127.0.0.1` from the repo root. This is a personal dev convenience only, not required to read these docs.
:::

Fixture files: [on GitHub](https://github.com/SpeciesFileGroup/taxonworks/tree/development/spec/files/import_datasets/occurrences/specification) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification">locally</a>.
Their automated assertions: [`occurrence_specification_spec.rb` on GitHub](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>.

## Minimum required fields

The smallest file the importer will accept &mdash; just `occurrenceID`, `basisOfRecord`, and `scientificName`.

**Test spreadsheet:** [`minimum_required_fields.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/minimum_required_fields.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/minimum_required_fields.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults, so new nomenclature may be created).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** Neither `Orotettix` (genus) nor `andeanus` (species) exist yet, so both TaxonNames are created, with the species nested under the genus. The TaxonDetermination links the new CollectionObject to the new species.

## Minimum required fields, matching by OTU instead

There's a second, mutually exclusive minimal field set: replace `scientificName` with [`TW:TaxonDetermination:otu_id`](/guide/import#taxon-class), which is matched to an already-existing OTU rather than to nomenclature. This is the recommended path for existing names &mdash; use the `Match OTU by Taxon Name` task in TW to build up the `otu_id`s to use beforehand. The OTU may or may not itself have an associated TaxonName; either way the TaxonDetermination is built directly from the OTU. No new TaxonNames are ever created via this path.

**Test spreadsheet:** [`minimum_required_fields_otu_id.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/minimum_required_fields_otu_id.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/minimum_required_fields_otu_id.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- OTU 900001, with an associated TaxonName (a species).
- OTU 900002, with no associated TaxonName (a "nomenclature-less" OTU &mdash; it has only a name of its own, e.g. `Unidentified sp.`).

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>TW:TaxonDetermination:otu_id</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>900001</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>900002</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's TaxonDetermination uses OTU 900001 and (through it) its TaxonName. Row 2's TaxonDetermination uses OTU 900002 directly; its `otu.taxon_name` is `nil`, since that OTU has none. Providing `TW:TaxonDetermination:otu_id` makes all other Taxon class columns (`scientificName`, `taxonRank`, `kingdom`&hellip;) ignored if present &mdash; see [Taxon class](/guide/import#taxon-class).

## Record level

Covers the [Record-level class](/guide/import#record-level-class) terms: `type`, `basisOfRecord`, and (a level down) the `BiocurationClass` that `basisOfRecord: FossilSpecimen` requires. `institutionCode` and `collectionCode` are also Record-level terms, but their *resolution* (matching a text value to a `Repository` or `Namespace`, including disambiguation when it's ambiguous) is covered under **Matching** instead &mdash; these two are kept here to the mechanics that don't involve matching.

### type defaults

The [Record-level class](/guide/import#record-level-class)'s other value-checked term. Like `basisOfRecord`, a blank `type` defaults to the one accepted value (`PhysicalObject`) rather than erroring. Unlike `basisOfRecord`, the match is case-sensitive and there's no GBIF-style reformatting &mdash; `physicalobject` is rejected exactly like any other unrecognized value.

**Test spreadsheet:** [`type_defaults.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/type_defaults.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/type_defaults.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>type</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>PhysicalObject</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-003</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>physicalobject</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
<tr>
  <td>spec-004</td>
  <td>PreservedSpecimen</td>
  <td>Schistocerca gregaria</td>
  <td>StillImage</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Rows 3 and 4 both get the same message: `type: "Only 'PhysicalObject' or empty allowed"`. Row 4's value, `StillImage`, isn't a typo or nonsense &mdash; it's a real term from the [DCMI Type Vocabulary](https://www.dublincore.org/specifications/dublin-core/dcmi-type-vocabulary/2010-10-11/) that `dwc:type` is defined against, and it's still rejected: TaxonWorks only accepts the one term meaningful for vouchered specimen records.

### basisOfRecord defaults

`basisOfRecord` must be present as a column (it's in the [required field set](/guide/import#occurrence-data)), but the cell value itself may be blank &mdash; a blank cell defaults to `PreservedSpecimen`, matched case-insensitively, and GBIF's `SCREAMING_SNAKE_CASE` occurrence-download variants (`PRESERVED_SPECIMEN`, `FOSSIL_SPECIMEN`) are reformatted and accepted too. Anything else errors, naming the field.

**Test spreadsheet:** [`basis_of_record_defaults.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/basis_of_record_defaults.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/basis_of_record_defaults.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td><em>(blank)</em></td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-003</td>
  <td>PRESERVED_SPECIMEN</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-004</td>
  <td>preservedspecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-005</td>
  <td>Foo</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Rows 2&ndash;4 create no new TaxonNames because row 1 already created `Orotettix andeanus`; every row after the first just reuses it (a `scientificName` reuse detail, not specific to `basisOfRecord`). Row 5's error is `basisOfRecord: "Only 'PreservedSpecimen', 'FossilSpecimen' or blank is allowed."`. `FossilSpecimen` (and its GBIF variant) is accepted too, but requires a `BiocurationClass` already present in the project &mdash; covered next.

### FossilSpecimen, without the biocuration class present

`FossilSpecimen` needs one specific thing to already exist in the project: a `BiocurationClass` whose `uri` is `http://rs.tdwg.org/dwc/terms/FossilSpecimen`. Without it, the row errors rather than silently falling back to `PreservedSpecimen` or skipping the classification.

**Test spreadsheet:** [`fossil_specimen.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/fossil_specimen.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/fossil_specimen.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no `BiocurationClass` with the DwC fossil URI, no pre-existing nomenclature.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>FossilSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** The error is `basisOfRecord: "Biocuration class http://rs.tdwg.org/dwc/terms/FossilSpecimen is not present in project"`. This check runs before anything else in the row, so nothing is created &mdash; not even the TaxonNames that would otherwise come from `scientificName`.

### FossilSpecimen, with the biocuration class present

Same spreadsheet as above, but the project already has the required `BiocurationClass`.

**Test spreadsheet:** [`fossil_specimen.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/fossil_specimen.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/fossil_specimen.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A `BiocurationClass` with `uri` `http://rs.tdwg.org/dwc/terms/FossilSpecimen` (any name).

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>FossilSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** The CollectionObject is classified with the matching `BiocurationClass` &mdash; the row's `basisOfRecord: FossilSpecimen` becomes a biocuration classification, not a separate field on the record.

## Occurrence

Covers the [Occurrence class](/guide/import#occurrence-class) terms: `occurrenceID`, `catalogNumber`, `recordNumber`, `recordedBy`, `individualCount`, `sex`, `preparations`. As with Record level, the *matching/disambiguation* parts of some of these terms (which `Namespace` a `catalogNumber` resolves to, containerizing rows that share a `catalogNumber`, reusing an existing `Person` for `recordedBy`) are covered under **Matching** instead.

### Duplicate occurrenceID

Yes &mdash; a second row reusing an `occurrenceID` already seen earlier in the same import errors, even if every other field is otherwise valid. `occurrenceID` is assigned as an identifier in a namespace that's auto-created once per import dataset (see [`occurrenceID` mapping](/guide/import#occurrence-class)) and shared by every row in the file, so a repeat value collides with the row that used it first.

**Test spreadsheet:** [`duplicate_occurrence_id.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_occurrence_id.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/duplicate_occurrence_id.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Row 2's error is `identifier: "spec-001 already taken"`. Row 2's CollectionObject and TaxonDetermination never actually persist &mdash; the entire row is undone when it errors, hence 0, not 1, in those columns above.

::: danger
Row 2 also reports a second, spurious message: `identifier_object: "is invalid"`. This shouldn't be there &mdash; it's noise left over from how the rejection is currently implemented, not a second thing wrong with the row. Ignore it; the real problem is the one named above.
:::

### catalogNumber namespace mechanics

A blank `catalogNumber` needs nothing. A `catalogNumber` paired with an explicit `TW:Namespace:catalogNumber` resolves immediately. A `catalogNumber` with neither that column nor an `institutionCode`/`collectionCode` mapping configured in `Settings` doesn't error &mdash; it stages as <span style="color: var(--color-import-not-ready); font-weight: 600;">NotReady</span> and stays that way until you resolve it. **NotReady rows are never included when the import is run** &mdash; they're simply excluded from processing (not attempted, not failed) until a namespace is set.

**Test spreadsheet:** [`catalog_number_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/catalog_number_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/catalog_number_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `ABC` and delimiter `NONE` (so the computed identifier reads as `ABC123`, not `ABC 123` &mdash; see [Namespaces](/guide/import#namespaces)).

**Settings:** None (all defaults) &mdash; no `institutionCode`/`collectionCode` &rarr; Namespace mapping configured.

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>TW:Namespace:catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td><em>(blank)</em></td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>123</td>
  <td>ABC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">ABC123</td>
</tr>
<tr>
  <td>spec-003</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>456</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-not-ready); font-weight: 600;">NotReady</span></td>
  <td class="outcome-col">&mdash;</td>
  <td class="outcome-col">&mdash;</td>
  <td class="outcome-col">&mdash;</td>
  <td class="outcome-col">&mdash;</td>
</tr>
</tbody>
</table>

**Notes:** Row 3's `&mdash;` cells mean "never attempted," not "attempted and created zero" &mdash; only rows in a `Ready` (or, when retrying, `Errored`) state are ever processed by an import run. Only row 2 creates a CatalogNumber identifier (in namespace `ABC`); row 1 has no `catalogNumber` at all, so no identifier is created for it either. The identifier's value is the namespace's short name plus the `catalogNumber` value, joined by the namespace's delimiter.

### recordNumber namespace mechanics

Unlike `catalogNumber`, a `recordNumber` with no way to resolve a namespace doesn't stage as NotReady &mdash; it errors outright at import time, because (unlike `catalogNumber`) there's no `institutionCode`/`collectionCode`-based fallback mapping for `recordNumber`: the `TW:Namespace:recordNumber` column is the only path.

**Test spreadsheet:** [`record_number_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/record_number_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/record_number_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `DEF` and delimiter `NONE` (so the computed identifier reads as `DEF222`, not `DEF 222` &mdash; see [Namespaces](/guide/import#namespaces)).

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>recordNumber</th>
  <th>TW:Namespace:recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>111</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>222</td>
  <td>DEF</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">DEF222</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is on `TW:Namespace:recordNumber: "Namespace not found"` &mdash; named for the companion column, not `recordNumber` itself.

### recordedBy

Creates one unvetted `Person` per name, and always writes the raw column value into `verbatim_collectors` regardless of how many names it parses into. Names are separated with `" | "` &mdash; the same pipe-list convention used across several DwC terms in this importer (e.g. `identifiedBy`).

**Test spreadsheet:** [`recorded_by.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/recorded_by.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/recorded_by.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing People.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 4.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>recordedBy</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">People created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>Jane Smith</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>John Doe | Mary Jones</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">2</td>
</tr>
</tbody>
</table>

**Notes:** All 3 People created are `Person::Unvetted`. Row 2's `verbatim_collectors` is stored as the full raw string `"John Doe | Mary Jones"`, not split.

### individualCount

`individualCount` is the total number of specimens the record represents. A blank value defaults to `1`. `1` creates a `Specimen`; anything greater creates a `Lot` instead. `0` or a negative value is not valid and must be rejected.

**Test spreadsheet:** [`individual_count.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/individual_count.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/individual_count.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>individualCount</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-003</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>5</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-004</td>
  <td>PreservedSpecimen</td>
  <td>Schistocerca gregaria</td>
  <td>0</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
<tr>
  <td>spec-005</td>
  <td>PreservedSpecimen</td>
  <td>Locusta migratoria</td>
  <td>-1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Rows 1 and 2 create a `Specimen`; row 3 creates a `Lot` (any `individualCount` &gt; 1 does). Rows 4 and 5 are correctly rejected.

::: danger
The rejection message for rows 4 and 5 is currently `total: ["Must be positive.", "total must be > 1"]` &mdash; it names the internal `total` field rather than `individualCount` (what you actually typed), and lists two overlapping complaints for the one value. It should instead name `individualCount` directly and say, plainly, that it can't be zero or negative.
:::

### sex

Unlike `basisOfRecord: FossilSpecimen`'s `BiocurationClass`, `sex` doesn't require anything to pre-exist: an unrecognized value **auto-creates** a `BiocurationGroup` named "Sex" (first time only) and a `BiocurationClass` for that value. A repeated value reuses the same class (matched case-insensitively) rather than duplicating it. The one hard restriction: a `sex` value may only be a single word.

**Test spreadsheet:** [`sex.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/sex.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/sex.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing "Sex" BiocurationGroup or BiocurationClass.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>sex</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>male</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>male</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-003</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>unknown sex</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** After row 1, exactly one `BiocurationGroup` ("Sex", `uri: http://rs.tdwg.org/dwc/terms/sex`) and one `BiocurationClass` ("male") exist; row 2 reuses both rather than creating duplicates. Row 3's error is `sex: "Only single-word controlled vocabulary supported at this time."`.

### preparations

The mirror image of `sex`: `preparations` must match an existing `PreparationType` by name (case-insensitively) &mdash; nothing is auto-created. This is issue [#4883](https://github.com/SpeciesFileGroup/taxonworks/issues/4883) row 13.

**Test spreadsheet:** [`preparations.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/preparations.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/preparations.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A PreparationType named `pinned`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>preparations</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>pinned</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>spec-002</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>spread</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Row 2's error is `preparations: "Unknown preparation \"spread\". If it is correct please add it to preparation types and retry."` No new `PreparationType` is created &mdash; only 1 exists after import (the pre-existing `pinned`), not 2.

## Collecting Event

A CollectingEvent is created for (or shared by, see [Matching](#matching)) every imported row; there is no minimum set of Event-class fields required to create one.

### eventID namespace mechanics

Unlike `catalogNumber`, an `eventID` with no `TW:Namespace:eventID` column value doesn't stage as NotReady, and doesn't error either: a default Namespace private to this import dataset is created automatically and used for every such row, so the `eventID` &mdash; Identifier::Local::Event mapping still works out of the box. A `TW:Namespace:eventID` value, when given, is used instead of the default.

**Test spreadsheet:** [`event_id_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_id_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_id_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `EVT` and delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Event<wbr>identifier value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">eventID:100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>200</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">EVT200</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's identifier value shows the shape of the default namespace: a fixed `eventID` prefix joined with `:`. The default namespace is scoped to this import dataset &mdash; a second import with the same blank `TW:Namespace:eventID` column reuses the same default namespace, so `eventID` values remain comparable within a dataset even when it's never explicitly configured (see [eventID reused across separate imports](#eventid-reused-across-separate-imports)).

### fieldNumber namespace mechanics

Unlike `eventID`, `fieldNumber` has no default-namespace fallback: `TW:Namespace:fieldNumber` is the only path to a namespace, and a `fieldNumber` given without one errors outright at import time, the same way `recordNumber` does.

**Test spreadsheet:** [`field_number_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/field_number_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/field_number_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `FLD` and delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>fieldNumber</th>
  <th>TW:Namespace:fieldNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">FieldNumber<wbr>identifier value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>200</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">FLD200</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is `TW:Namespace:fieldNumber: "Namespace not found"` &mdash; naming the missing companion column, not `fieldNumber` itself.

### Event date

A CollectingEvent's start and end dates can come from `eventDate` (a single ISO 8601 date, or a `/`-separated range), from the individual `year`/`month`/`day` columns, or from `year` combined with `startDayOfYear`/`endDayOfYear`. These sources are cross-checked against each other where they overlap, not merged silently.

#### eventDate: single value vs. a range

A single `eventDate` (e.g. `1983-10-25`) sets only the start date. A range, expressed as two ISO 8601 dates separated by `/` (e.g. `2020-11-30/2020-12-04`), sets both the start and end date.

**Test spreadsheet:** [`event_date_single_and_range.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_date_single_and_range.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_date_single_and_range.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventDate</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">start date</th>
  <th class="outcome-header">end date</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1983-10-25</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1983-10-25</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>2020-11-30/2020-12-04</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">2020-11-30</td>
  <td class="outcome-col">2020-12-04</td>
</tr>
</tbody>
</table>

**Notes:** The second date in a range may also be abbreviated, omitting the higher-order elements it shares with the first (for example `2020-11-30/12-04` or `2020-11-30/04`), per the [ISO 8601 time interval](https://en.wikipedia.org/wiki/ISO_8601#Time_intervals) convention.

#### year, month, and day columns as an alternative to eventDate

When no `eventDate` is given, `year`, `month`, and `day` populate the start date directly.

**Test spreadsheet:** [`event_date_year_month_day.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_date_year_month_day.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_date_year_month_day.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 2.5em;">
  <col style="width: 3em;">
  <col style="width: 2.5em;">
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>year</th>
  <th>month</th>
  <th>day</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">start date</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1999</td>
  <td>7</td>
  <td>4</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1999-07-04</td>
</tr>
</tbody>
</table>

**Notes:** `year`, `month`, and `day` may each be given independently; any subset populates just those parts of the start date.

#### eventDate conflicts with year, month, and/or day

When both `eventDate` and one or more of `year`/`month`/`day` are given, they must agree; a mismatch on any of the three is rejected rather than one silently overriding the other.

**Test spreadsheet:** [`event_date_conflict.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_date_conflict.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_date_conflict.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 2.5em;">
  <col style="width: 3em;">
  <col style="width: 2.5em;">
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventDate</th>
  <th>year</th>
  <th>month</th>
  <th>day</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1999-07-04</td>
  <td>1999</td>
  <td>7</td>
  <td>5</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** `day: 5` disagrees with the `4` implied by `eventDate: "1999-07-04"`. The row errors with `eventDate: "Conflicting values. Please check year, month, and day match eventDate"`, and no CollectingEvent is created.

#### verbatimEventDate

`verbatimEventDate` is stored as-is, independent of whether `eventDate` and/or `year`/`month`/`day` are also given and successfully parsed.

**Test spreadsheet:** [`event_date_verbatim.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_date_verbatim.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_date_verbatim.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventDate</th>
  <th>verbatimEventDate</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1999-07-04</td>
  <td>summer of '99</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** The CollectingEvent's start date parses to `1999-07-04` as usual, and its verbatim date is stored as the literal string `summer of '99`, unrelated to whether that string is itself a parseable date.

#### startDayOfYear and endDayOfYear

`startDayOfYear`/`endDayOfYear` express the date as an ordinal day count within `year` (day 1 = January 1st) rather than a calendar month/day; each requires `year` to be given, since an ordinal day is meaningless without knowing the year.

**Test spreadsheet:** [`event_start_end_day_of_year.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_start_end_day_of_year.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_start_end_day_of_year.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 2.5em;">
  <col style="width: 4em;">
  <col style="width: 4em;">
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>year</th>
  <th>startDayOfYear</th>
  <th>endDayOfYear</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">start date</th>
  <th class="outcome-header">end date</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>2000</td>
  <td>60</td>
  <td>65</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">2000-02-29</td>
  <td class="outcome-col">2000-03-05</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>2000</td>
  <td>60</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">2000-02-29</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-c</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td><em>(blank)</em></td>
  <td><em>(blank)</em></td>
  <td>60</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
</tbody>
</table>

**Notes:** 2000 is a leap year, so day 60 is February 29th (not March 1st, as it would be in a non-leap year) &mdash; row a's day 65 correspondingly lands on March 5th. Row c errors with `endDayOfYear: "Missing year value"`, the same message `startDayOfYear` gives under the same condition.

### eventTime

Like `eventDate`, `eventTime` accepts either a single value or a `/`-separated range, and the second value in a range may omit the higher-order components it shares with the first (e.g. `10:15:30/14:45:00` or the abbreviated `10:15:30/14:45`).

#### eventTime: single value vs. a range

A single `eventTime` (e.g. `10:15:30`) sets only the start time. A range sets both the start and end time.

**Test spreadsheet:** [`event_time_single_and_range.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_time_single_and_range.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_time_single_and_range.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventTime</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">start time</th>
  <th class="outcome-header">end time</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>10:15:30</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">10:15:30</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>10:15:30/14:45:00</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">10:15:30</td>
  <td class="outcome-col">14:45:00</td>
</tr>
</tbody>
</table>

**Notes:** `minute` and `second` are each independently optional, at both the start and end of a range &mdash; `10`, `10:15`, and `10:15:30` are all accepted on their own.

#### Malformed eventTime

An `eventTime` value that doesn't fit the expected shape (a single time, or two times separated by `/`) is expected to error, naming the value that couldn't be parsed &mdash; the same way an unparseable `eventDate` does.

::: danger
TaxonWorks does not currently do this. Instead, the row imports successfully, and all of the CollectingEvent's time fields are left unset, with nothing in the row's status or messages indicating that the `eventTime` value was unusable. If you're relying on imported time-of-day data, this is worth checking for directly (e.g. reviewing CollectingEvents with no time fields set despite an `eventTime` column value being present) rather than assuming an `Imported` status means the value was understood.
:::

**Test spreadsheet:** [`event_time_malformed.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_time_malformed.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_time_malformed.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

#### eventTime with an out-of-range component

A value with the right shape but an out-of-range component (e.g. minute `75`) is a different situation from a malformed value &mdash; here, unlike above, the row is correctly rejected.

**Test spreadsheet:** [`event_time_out_of_range.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_time_out_of_range.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_time_out_of_range.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventTime</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>10:75</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is `time_start_minute: ["not in range", "must be an integer between 0 and 59"]` &mdash; `75` isn't a valid minute. No CollectingEvent is created.

::: danger
The same value is flagged twice in slightly different words, and the row's error data also includes a spurious `collection_objects: "is invalid"` &mdash; noise left over from how the rejection cascades through the associated CollectionObject, the same kind of thing already noted for [Duplicate occurrenceID](#duplicate-occurrenceid). Ignore the extra messages; the real problem is `time_start_minute` being out of range.
:::

### Geographic data

Geographic Location terms only apply when a row creates a *new* CollectingEvent &mdash; when a row instead reuses an existing one (matched by `eventID`/`fieldNumber`, see [Collecting Event](#collecting-event) above), its geographic data is left as whatever the CollectingEvent already had, and any Location-class columns on that row are ignored.

#### country, stateProvince, county resolve to a GeographicArea

`county`, `stateProvince`, and `country` are matched together against TaxonWorks' GeographicArea gazetteer data, most specific first.

**Test spreadsheet:** [`geographic_area_country_state_county.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_country_state_county.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_country_state_county.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- GeographicAreas `Champaign` (county) &rarr; `Illinois` (state) &rarr; `United States` (country), correctly nested.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 6em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>country</th>
  <th>stateProvince</th>
  <th>county</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">GeographicArea matched</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>United States</td>
  <td>Illinois</td>
  <td>Champaign</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">Champaign</td>
</tr>
</tbody>
</table>

**Notes:** The row's CollectingEvent is linked to the county-level GeographicArea, the most specific of the three terms provided.

#### country, stateProvince, county: recursive fallback vs. exact match only

If the full combination of `county` + `stateProvince` + `country` doesn't match any GeographicArea, the finest term (`county`) is dropped and the remaining combination is tried again, and so on, until either something matches or no terms are left. The `Only search for the finest geographical name provided` setting turns this fallback off, requiring the full combination given to match exactly.

**Test spreadsheet:** [`geographic_area_recursive_fallback.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_recursive_fallback.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_recursive_fallback.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- GeographicAreas `Champaign` (county) &rarr; `Illinois` (state) &rarr; `United States` (country). The row's `county` value (`Nonexistent County`) matches none of them.

**Off (default):**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 6em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>country</th>
  <th>stateProvince</th>
  <th>county</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">GeographicArea matched</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>United States</td>
  <td>Illinois</td>
  <td>Nonexistent County</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">Illinois</td>
</tr>
</tbody>
</table>

`county` doesn't match, so it's dropped and `stateProvince` + `country` is tried, matching `Illinois`.

**On:** with the setting enabled, the same row instead imports with no GeographicArea matched at all &mdash; the full `county` + `stateProvince` + `country` combination is tried exactly once, and since it doesn't match, nothing is matched. This isn't an error by itself; see the next section for the setting that makes an unmatched combination an error.

#### Error if no geographic area with the provided name exists

By default, when nothing matches (whether via the recursive search exhausting every combination, or a single exact-match attempt under the setting above), the row still imports &mdash; simply with no GeographicArea linked to its CollectingEvent. The `Error if no geographic area with provided name exists` setting makes that same situation an error instead.

**Test spreadsheet:** [`geographic_area_no_match.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_no_match.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_no_match.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None &mdash; `Nowhereland`, `Nowhere State`, and `Nowhere County` don't match any GeographicArea in the project, at any level.

**Off (default):**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>country</th>
  <th>stateProvince</th>
  <th>county</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>Nowhereland</td>
  <td>Nowhere State</td>
  <td>Nowhere County</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**On:** the same row instead errors, with `country, stateProvince, county: "GeographicArea with location levels county:Nowhere County, state_province:Nowhere State, country:Nowhereland not found."` &mdash; no CollectionObject or CollectingEvent persists.

#### Require geographical area data origin

TaxonWorks' GeographicArea gazetteer data can come from more than one source (TDWG, Natural Earth, a project's own custom entries, etc.), tracked per-GeographicArea as its `data_origin`. This setting restricts matching to GeographicAreas from one specific source.

**Test spreadsheet:** [`geographic_area_data_origin.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_data_origin.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_data_origin.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- GeographicAreas `Champaign` (county) &rarr; `Illinois` (state) &rarr; `United States` (country), all with `data_origin: "Test Data"`.

**On, set to the `data_origin` actually used (`Test Data`):** the row imports and matches `Champaign`, same as [country, stateProvince, county resolve to a GeographicArea](#country-stateprovince-county-resolve-to-a-geographicarea) above.

**On, set to any other `data_origin`:** the row imports, matching no GeographicArea &mdash; the same "silent non-match" outcome as an unmatched name, just reached by source filtering instead. Not an error by itself; combine with `Error if no geographic area with provided name exists` above if that's not what you want.

#### countryCode as a fallback for country

When `country` is blank but `countryCode` is present, a 2-letter (ISO 3166-1 alpha-2) or 3-letter (alpha-3) code is resolved to its country name first, then matched the same way `country` would be.

**Test spreadsheet:** [`geographic_area_country_code.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_country_code.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_country_code.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A GeographicArea `United States`, with `iso_3166_a2: "US"` and `iso_3166_a3: "USA"`, `data_origin: "country_names_and_code_elements"`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 6em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>countryCode</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">GeographicArea matched</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>US</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">United States</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>USA</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">United States</td>
</tr>
</tbody>
</table>

**Notes:** Both the 2-letter and 3-letter forms resolve to the same country. `countryCode` resolution specifically requires a GeographicArea with `data_origin: "country_names_and_code_elements"` &mdash; ordinary gazetteer entries from other sources aren't consulted at this step, regardless of the `Require geographical area data origin` setting above.

#### Unrecognized countryCode

A `countryCode` that doesn't match any known country is expected to error, naming the unrecognized value.

::: danger
TaxonWorks does not currently do this. Resolving `countryCode` looks up a GeographicArea and immediately calls `.name` on the result with no check that anything was found. For an unrecognized code, the lookup returns nothing, and the row fails with a raw internal exception (`undefined method 'name' for nil`) rather than a normal import error &mdash; the row's status is `Failed` (not `Errored`), and its error data is a Ruby exception message and stack trace instead of a message naming the problem. If you see a row with status `Failed` after an import, this is one of the ways that can happen; check the `countryCode` column for typos.
:::

**Test spreadsheet:** [`geographic_area_country_code_unrecognized.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_country_code_unrecognized.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_country_code_unrecognized.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

#### Require that the matched geographic area has a shape

A GeographicArea can optionally have a geographic shape (a polygon boundary) attached; this setting restricts matching to only GeographicAreas that do.

::: danger
Leaving this setting unconfigured &mdash; its normal, default state &mdash; does not mean "don't care whether the matched GeographicArea has a shape," as the setting's own name implies. It silently does the *opposite*: matching is restricted to GeographicAreas that do **not** have a shape, and a GeographicArea that does have one is never matched by default, at any level, unless a shapeless coarser ancestor happens to exist to fall back to. Since real, imported gazetteer data normally does have shapes, this can mean geographic matching silently fails (or falls back to the wrong, coarser GeographicArea) far more often than expected, for any project that hasn't explicitly turned this setting on. Confirmed by direct comparison: the identical GeographicArea, with a shape attached, is matched successfully when the setting is explicitly set to `true`, and never matched when the setting is left unset.
:::

**Test spreadsheet:** [`geographic_area_has_shape.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_has_shape.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_has_shape.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A GeographicArea `United States`, with a shape attached.

**Settings:** None (all defaults) &mdash; expected to match regardless, since nothing was asked to require a shape one way or the other.

#### decimalLatitude / decimalLongitude

`decimalLatitude` and `decimalLongitude`, when both present, are stored verbatim on the CollectingEvent and additionally create a Georeference, alongside `geodeticDatum` and `coordinateUncertaintyInMeters`.

**Test spreadsheet:** [`geographic_area_lat_long.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_lat_long.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_lat_long.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>decimalLatitude</th>
  <th>decimalLongitude</th>
  <th>geodeticDatum</th>
  <th>coordinateUncertaintyInMeters</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>40.11</td>
  <td>-88.20</td>
  <td>WGS84</td>
  <td>50</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** The CollectingEvent's `verbatim_latitude`, `verbatim_longitude`, and `verbatim_datum` store `40.11`, `-88.20`, and `WGS84` respectively, unmodified. `coordinateUncertaintyInMeters` is stored two different ways: as `50m` (with a unit suffix) on the CollectingEvent's own `verbatim_geolocation_uncertainty`, and as the plain number `50` on the separately-created Georeference's `error_radius`. A Georeference is only created when both latitude and longitude are present; see next.

#### decimalLatitude without decimalLongitude

`decimalLatitude` and `decimalLongitude` are expected together; providing only one is rejected rather than accepted as a partial coordinate.

**Test spreadsheet:** [`geographic_area_lat_without_long.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_lat_without_long.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_lat_without_long.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>decimalLatitude</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>40.11</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is `verbatim_longitude: "can't be blank"`. No CollectingEvent or Georeference is created.

#### coordinateUncertaintyInMeters must be an integer

**Test spreadsheet:** [`geographic_area_coordinate_uncertainty_non_integer.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/geographic_area_coordinate_uncertainty_non_integer.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/geographic_area_coordinate_uncertainty_non_integer.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>decimalLatitude</th>
  <th>decimalLongitude</th>
  <th>coordinateUncertaintyInMeters</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>40.11</td>
  <td>-88.20</td>
  <td>not-a-number</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is `coordinateUncertaintyInMeters: "Non-integer value"`. This check runs before the CollectingEvent is created, so no CollectingEvent exists afterward either, even though `decimalLatitude`/`decimalLongitude` were themselves valid.

## Matching

Cross-cutting matching/disambiguation behavior that doesn't belong to a single term &mdash; how the importer decides "is this the same thing I've already seen, or something new."

### occurrenceID reused across separate imports

A common real-world workflow: run an import, some rows error, fix the source file, re-run. Does re-running collide with the `occurrenceID`s that already imported successfully the first time? No &mdash; unlike a duplicate `occurrenceID` *within* one import (which errors, see [Occurrence](#occurrence) above), the same `occurrenceID` reused across two *separate* imports doesn't collide at all, because each import gets its own `occurrenceID` namespace (see [`occurrenceID` mapping](/guide/import#occurrence-class)).

**Test spreadsheet:** [`occurrence_id_reuse_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/occurrence_id_reuse_a.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/occurrence_id_reuse_a.tsv">locally</a>, [`occurrence_id_reuse_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/occurrence_id_reuse_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/occurrence_id_reuse_b.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

Each row also has its own `catalogNumber` and `recordNumber` (differing between the two imports), to confirm reusing `occurrenceID` doesn't have knock-on effects for those other identifiers either &mdash; see [Duplicate catalogNumber](#duplicate-catalognumber) and [Duplicate recordNumber](#duplicate-recordnumber) below for what happens when *those* actually collide.

**Input:**
- A Namespace with short name `CATD` (delimiter `NONE`), for `catalogNumber`.
- A Namespace with short name `RECD` (delimiter `NONE`), for `recordNumber`.

**Settings:** None (all defaults), for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>A</td>
  <td>spec-shared</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CATD700</td>
  <td class="outcome-col">RECDRA</td>
</tr>
<tr>
  <td>B</td>
  <td>spec-shared</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CATD800</td>
  <td class="outcome-col">RECDRB</td>
</tr>
</tbody>
</table>

**Notes:** Both rows use `occurrenceID: spec-shared`, but they're in two separate import datasets (two separate spreadsheets, `A` and `B`, run independently), each producing its own `Identifier` namespace &mdash; contrast with [Duplicate occurrenceID](#duplicate-occurrenceid), where the collision is specifically because both rows share one namespace by being in the *same* import. `catalogNumber` and `recordNumber`, by contrast, use a real project Namespace you create yourself (not an auto-generated per-import one) &mdash; whether *those* collide across imports depends on whether their values actually collide, not on which import they came from. See below.

### catalogNumber namespace resolution via institutionCode/collectionCode

Without an explicit `TW:Namespace:catalogNumber` column (see [catalogNumber namespace mechanics](#catalognumber-namespace-mechanics) above), a `catalogNumber`'s Namespace is instead resolved from `institutionCode`/`collectionCode`, matched against a mapping table configured through the DwC importer's `Settings` panel &mdash; not a spreadsheet column. Three mappings can be configured: `institutionCode` + `collectionCode` together (most specific), `collectionCode` alone, and `institutionCode` alone (its own distinct mapping, not a fallback that reuses the `collectionCode`-only one). When a row has both fields and both mappings are configured, the `institutionCode` + `collectionCode` mapping wins.

**Test spreadsheet:** [`catalog_number_namespace_by_institution_collection_code.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/catalog_number_namespace_by_institution_collection_code.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/catalog_number_namespace_by_institution_collection_code.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `INHS`, delimiter `NONE`.
- A Namespace with short name `GENERIC`, delimiter `NONE`.
- A Repository with acronym `INHS` &mdash; `institutionCode` is independently matched against `Repository` acronyms too (see [Record-level class](/guide/import#record-level-class)); without a matching Repository, a row with an `institutionCode` errors regardless of catalogNumber namespace resolution.

**Settings:** the catalog number namespace mapping table configured with `collectionCode` `ENT` &rarr; Namespace `GENERIC`, `institutionCode` `INHS` + `collectionCode` `ENT` &rarr; Namespace `INHS`, and `institutionCode` `INHS` alone (no `collectionCode`) &rarr; Namespace `INHS`.

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>institutionCode</th>
  <th>collectionCode</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>INHS</td>
  <td>ENT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">INHS100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>200</td>
  <td><em>(blank)</em></td>
  <td>ENT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">GENERIC200</td>
</tr>
<tr>
  <td>occ-c</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>300</td>
  <td>INHS</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">INHS300</td>
</tr>
</tbody>
</table>

**Notes:** Row 1 resolves via the more specific `institutionCode` + `collectionCode` mapping (`INHS100`). Row 2 has no `institutionCode`, so that mapping doesn't apply to it &mdash; it falls back to the `collectionCode`-only mapping (`GENERIC200`). Row 3 has no `collectionCode` at all; it resolves via the separate `institutionCode`-alone mapping (`INHS300`) &mdash; not via row 2's `collectionCode`-only mapping, which requires a `collectionCode` value to match against.

### catalogNumber must match its computed identifier verbatim

A namespace's short name plus a row's `catalogNumber` value together compute an identifier (e.g. Namespace `ABC` + `catalogNumber` `100` &rarr; `ABC100`). By default, the `catalogNumber` cell can be written with or without that prefix &mdash; both `100` and `ABC100` resolve to the same identifier. The `Error records when computed identifier will not match catalogNumber` setting tightens this: with it on, the cell value must already include the prefix exactly, or the row errors.

**Test spreadsheet:** [`catalog_number_verbatim_match.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/catalog_number_verbatim_match.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/catalog_number_verbatim_match.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `ABC`, delimiter `NONE`.

**Off (default):**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>TW:Namespace:catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>ABC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">ABC100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>ABC200</td>
  <td>ABC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">ABC200</td>
</tr>
</tbody>
</table>

Both the bare value (`100`) and the already-prefixed value (`ABC200`) import fine, computing to `ABC100` and `ABC200` respectively.

**On:**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>TW:Namespace:catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>ABC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>ABC200</td>
  <td>ABC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">ABC200</td>
</tr>
</tbody>
</table>

Row 1's error is `catalogNumber: "Computed catalog number ABC100 will not match verbatim 100. Verify the mapped namespace and namespace delimiter are correct."` Row 2 already included the prefix, so it's unaffected by the setting.

### Duplicate catalogNumber

Unlike `occurrenceID`, `catalogNumber`'s Namespace is a real one you create and reuse on purpose (via `TW:Namespace:catalogNumber`, or an `institutionCode`/`collectionCode` mapping) &mdash; so a repeated `catalogNumber` value in that Namespace collides *whether or not* it's in the same import. Rows 1&ndash;2 below are one import (two rows); rows 3&ndash;4 are two separate imports (one row each) &mdash; both pairs behave identically.

**Test spreadsheet:** [`duplicate_catalog_number_same_import.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_catalog_number_same_import.tsv), [`duplicate_catalog_number_import_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_catalog_number_import_a.tsv), [`duplicate_catalog_number_import_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_catalog_number_import_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col style="width: 5em;">
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>TW:Namespace:catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">same<br>import</td>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>100</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td rowspan="2">separate<br>imports</td>
  <td>occ-a1</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>200</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT200</td>
</tr>
<tr>
  <td>occ-b1</td>
  <td>PreservedSpecimen</td>
  <td>Schistocerca gregaria</td>
  <td>200</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
</tbody>
</table>

**Notes:** Both second rows error with `catalogNumber: "Is already in use"`. See [Containers](#containers) below for what happens when a `recordNumber` is added to disambiguate instead of erroring.

### Duplicate recordNumber

Same shape as above, but with the opposite outcome. A `recordNumber` value is not required to be unique &mdash; it represents a collector's own field number, which can legitimately repeat across different collectors or contexts. A repeated `recordNumber`, whether in the same import or a separate one, is expected to import without complaint, exactly like the two cases below.

This holds even when a `recordNumber` is being used to disambiguate items that share a `catalogNumber`: nothing requires those `recordNumber`s to actually be distinct from each other either &mdash; see [Containers](#containers) below.

**Test spreadsheet:** [`duplicate_record_number_same_import.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_record_number_same_import.tsv), [`duplicate_record_number_import_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_record_number_import_a.tsv), [`duplicate_record_number_import_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_record_number_import_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `REC`, delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col style="width: 5em;">
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>recordNumber</th>
  <th>TW:Namespace:recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">same<br>import</td>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>300</td>
  <td>REC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">REC300</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>300</td>
  <td>REC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">REC300</td>
</tr>
<tr>
  <td rowspan="2">separate<br>imports</td>
  <td>occ-a1</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>400</td>
  <td>REC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">REC400</td>
</tr>
<tr>
  <td>occ-b1</td>
  <td>PreservedSpecimen</td>
  <td>Schistocerca gregaria</td>
  <td>400</td>
  <td>REC</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">REC400</td>
</tr>
</tbody>
</table>

**Notes:** All four rows import; both pairs create two separate RecordNumber identifiers carrying the identical value. See [Containers](#containers) directly below for the one situation where a repeated `recordNumber` value is actually a problem.

### Collecting Event

Whether a row shares an existing CollectingEvent or creates a new one is decided entirely by `eventID` and/or `fieldNumber` matching an existing identifier &mdash; never by comparing other Event- or Location-class field values (see [No eventID or fieldNumber given](#no-eventid-or-fieldnumber-given) below).

#### eventID reused across separate imports

Reusing an `eventID` value across two separate imports only shares the underlying CollectingEvent when both imports resolve it through the same Namespace. Left to the default (no `TW:Namespace:eventID` column value), each import gets its own private Namespace (see [eventID namespace mechanics](#eventid-namespace-mechanics)), so the same `eventID` value in two different imports resolves to two different, unrelated CollectingEvents. An explicit, shared `TW:Namespace:eventID` avoids this, the same way an explicit shared Namespace lets `catalogNumber`/`recordNumber` values do the same (see [catalogNumber namespace resolution via institutionCode/collectionCode](#catalognumber-namespace-resolution-via-institutioncode-collectioncode)).

**Test spreadsheet:** [`event_id_reuse_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_id_reuse_a.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_id_reuse_a.tsv">locally</a>, [`event_id_reuse_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_id_reuse_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_id_reuse_b.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `EVT`, delimiter `NONE`.

**Settings:** None (all defaults), for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 6em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Collecting<wbr>Event shared with</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">A</td>
  <td>occ-a1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col"><em>(nothing, new CE)</em></td>
</tr>
<tr>
  <td>occ-a2</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>200</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col"><em>(nothing, new CE)</em></td>
</tr>
<tr>
  <td rowspan="2">B</td>
  <td>occ-b1</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col"><em>(nothing, new CE)</em></td>
</tr>
<tr>
  <td>occ-b2</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>200</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">occ-a2</td>
</tr>
</tbody>
</table>

**Notes:** All four rows import, and 3 CollectingEvents are created in total. `occ-a1` and `occ-b1` share the identical `eventID` value `100`, but each is left to its import's own default Namespace, so they resolve to two separate CollectingEvents. `occ-a2` and `occ-b2` share `eventID` value `200` through the same explicit `EVT` Namespace in both imports, so `occ-b2` reuses `occ-a2`'s CollectingEvent instead of creating a new one.

#### eventID must match its computed identifier verbatim

The `eventID` analog of [catalogNumber must match its computed identifier verbatim](#catalognumber-must-match-its-computed-identifier-verbatim): a Namespace's short name plus a row's `eventID` value together compute an identifier (e.g. Namespace `EVT` + `eventID` `100` &rarr; `EVT100`). By default, the `eventID` cell can be written with or without that prefix. The `Error records when computed identifier will not match eventID` setting tightens this: with it on, the cell value must already include the prefix exactly, or the row errors.

This only applies to a row with an *explicit* `TW:Namespace:eventID` &mdash; it has no effect on a row left to the default per-import Namespace (see [eventID namespace mechanics](#eventid-namespace-mechanics)), since there's no explicit namespace prefix to check the value against.

**Test spreadsheet:** [`event_id_verbatim_match.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_id_verbatim_match.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_id_verbatim_match.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `EVT`, delimiter `NONE`.

**Off (default):**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Event<wbr>identifier value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">EVT100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>EVT200</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">EVT200</td>
</tr>
<tr>
  <td>occ-c</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">eventID:100</td>
</tr>
</tbody>
</table>

Both the bare value (`100`) and the already-prefixed value (`EVT200`) import fine under the explicit `EVT` Namespace, computing to `EVT100` and `EVT200` respectively. Row 3, left to the default Namespace, is unaffected either way.

**On:**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Event<wbr>identifier value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>EVT200</td>
  <td>EVT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">EVT200</td>
</tr>
<tr>
  <td>occ-c</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>100</td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">eventID:100</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's error is `eventID: "Computed Event EVT100 will not match verbatim 100. Verify the namespace delimiter is correct."` Row 3 still imports even with the setting on, since it has no explicit `TW:Namespace:eventID` for the value to be checked against.

#### fieldNumber reused across separate imports

Unlike `eventID`, `fieldNumber` has no default-namespace fallback (see [fieldNumber namespace mechanics](#fieldnumber-namespace-mechanics)) &mdash; a `TW:Namespace:fieldNumber` value is always required, and is necessarily shared across imports that use the same Namespace. Reusing a `fieldNumber` value (in the same Namespace) across separate imports therefore always shares the CollectingEvent.

**Test spreadsheet:** [`field_number_reuse_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/field_number_reuse_a.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/field_number_reuse_a.tsv">locally</a>, [`field_number_reuse_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/field_number_reuse_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/field_number_reuse_b.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `FLD`, delimiter `NONE`.

**Settings:** None (all defaults), for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 6em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>fieldNumber</th>
  <th>TW:Namespace:fieldNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Collecting<wbr>Event shared with</th>
</tr>
</thead>
<tbody>
<tr>
  <td>A</td>
  <td>occ-a1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>500</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col"><em>(nothing, new CE)</em></td>
</tr>
<tr>
  <td>B</td>
  <td>occ-b1</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>500</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">occ-a1</td>
</tr>
</tbody>
</table>

**Notes:** Both rows import; only 1 CollectingEvent and 1 FieldNumber identifier exist afterward.

#### eventID and fieldNumber refer to inconsistent collecting events

When a row supplies both `eventID` and `fieldNumber`, both must agree about which CollectingEvent is meant. Two distinct inconsistencies are possible, and both are rejected rather than silently resolved one way or the other.

**fieldNumber does not match a previously established collecting event:**

**Test spreadsheet:** [`event_field_number_partial_mismatch.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_field_number_partial_mismatch.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_field_number_partial_mismatch.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `EVT`, delimiter `NONE`, for `eventID`.
- A Namespace with short name `FLD`, delimiter `NONE`, for `fieldNumber`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th>fieldNumber</th>
  <th>TW:Namespace:fieldNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>event-1</td>
  <td>EVT</td>
  <td>1</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>event-1</td>
  <td>EVT</td>
  <td>2</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

Row 1 establishes a CollectingEvent identified by both `eventID: "event-1"` and `fieldNumber: "1"`. Row 2 reuses the same `eventID`, correctly resolving to that same CollectingEvent, but pairs it with a `fieldNumber` (`"2"`) never seen before &mdash; an inconsistent claim about the same event. Row 2 errors with `eventID/fieldNumber: "does not match previous definition of collecting event"`, and only 1 CollectingEvent exists afterward.

**eventID and fieldNumber each already belong to a different, previously established collecting event:**

**Test spreadsheet:** [`event_field_number_conflicting_ce.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/event_field_number_conflicting_ce.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/event_field_number_conflicting_ce.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** Same as above &mdash; `EVT` and `FLD` Namespaces.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>eventID</th>
  <th>TW:Namespace:eventID</th>
  <th>fieldNumber</th>
  <th>TW:Namespace:fieldNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>event-1</td>
  <td>EVT</td>
  <td><em>(blank)</em></td>
  <td><em>(blank)</em></td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td><em>(blank)</em></td>
  <td><em>(blank)</em></td>
  <td>1</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-c</td>
  <td>PreservedSpecimen</td>
  <td>Melanoplus femurrubrum</td>
  <td>event-1</td>
  <td>EVT</td>
  <td>1</td>
  <td>FLD</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

Rows 1 and 2 each independently establish their own CollectingEvent &mdash; row 1's identified by `eventID: "event-1"`, row 2's by `fieldNumber: "1"`. Row 3 supplies both together, but they now point at two different, already-established CollectingEvents. Row 3 errors with `eventID/fieldNumber: "eventId and fieldNumber refer to different collecting events"`, and both pre-existing CollectingEvents are left untouched &mdash; TaxonWorks never merges them.

#### No eventID or fieldNumber given

Without an `eventID` or `fieldNumber` to match against, a new CollectingEvent is created for every row, however similar its other Event- and Location-class data is to a previous row's. A conforming importer is not expected to infer Event identity from field values alone &mdash; only from an explicit identifier.

**Test spreadsheet:** [`no_event_field_number_identical_locality.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/no_event_field_number_identical_locality.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/no_event_field_number_identical_locality.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:** None.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>verbatimLocality</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>Mount Kilimanjaro</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>Mount Kilimanjaro</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** Both rows share the identical `verbatimLocality`, but 2 separate CollectingEvents are created &mdash; one per row. If you need rows sharing real-world Event data to share a single CollectingEvent, give them a common `eventID` and/or `fieldNumber`.

### Containers

`catalogNumber` collisions aren't always an error: if the colliding row also has a `recordNumber`, the importer merges the new CollectionObject into a `Container` with the earlier one instead of rejecting it &mdash; each item's `recordNumber` disambiguates it within the container. This is the intended way to import, say, a vial of several specimens logged as separate rows.

A setting, `Containerize specimen with existing ones when catalog number already exists` (see [Settings](#settings) below), additionally allows containerizing a colliding `catalogNumber` when no `recordNumber` is present at all &mdash; a deliberate trade-off some projects want: the items placed in the container this way aren't individually identifiable by identifier afterward, though each remains its own separate record.

`recordNumber` is allowed to repeat (see [Duplicate recordNumber](#duplicate-recordnumber) above), and that holds inside a container too: nothing requires the `recordNumber`s of items sharing a `catalogNumber` to actually be distinct from each other.

**Test code (all scenarios below):** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

#### Same catalogNumber, different recordNumbers (the intended case)

**Test spreadsheet:** [`container_different_record_numbers.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_different_record_numbers.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/container_different_record_numbers.tsv">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.
- A Namespace with short name `REC`, delimiter `NONE`.

**Settings:** None (all defaults) &mdash; `Containerize specimen with existing ones when catalog number already exists` is **not** required when a `recordNumber` is present.

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>500</td>
  <td>R1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT500</td>
  <td class="outcome-col">RECR1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>500</td>
  <td>R2</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT500</td>
  <td class="outcome-col">RECR2</td>
</tr>
</tbody>
</table>

**Notes:** One CatalogNumber identifier (shared), two RecordNumber identifiers (`R1`, `R2`), one `Container` holding both CollectionObjects. This is the healthy, disambiguated case.

#### Same catalogNumber, same recordNumber

A row can reuse both the `catalogNumber` and the `recordNumber` of an item already in a container &mdash; for example, two specimens from the same collecting event (same `recordNumber`, i.e. the same field number) that end up placed in the same physical lot (same `catalogNumber`).

**Test spreadsheet:** [`container_duplicate_record_number.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_duplicate_record_number.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/container_duplicate_record_number.tsv">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.
- A Namespace with short name `REC`, delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>600</td>
  <td>R1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT600</td>
  <td class="outcome-col">RECR1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>600</td>
  <td>R1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT600</td>
  <td class="outcome-col">RECR1</td>
</tr>
</tbody>
</table>

**Notes:** Both rows land in the same `Container`, and both RecordNumber identifiers carry the identical value `R1`. There's no way to distinguish the two items by identifier alone afterward &mdash; if that matters for your data, use distinct `recordNumber`s per item, as in the previous example.

#### containerize_dup_cat_no enabled, without a recordNumber

`Containerize specimen with existing ones when catalog number already exists` (see [Settings](#settings) below) lets a colliding `catalogNumber` containerize even when neither row has a `recordNumber` at all. This is the setting's whole purpose &mdash; without it, the same scenario errors instead (see [Duplicate catalogNumber](#duplicate-catalognumber) above).

**Test spreadsheet:** [`container_no_record_number_setting_enabled.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_no_record_number_setting_enabled.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/container_no_record_number_setting_enabled.tsv">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.

**Settings:** `Containerize specimen with existing ones when catalog number already exists` enabled.

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1100</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>1100</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1100</td>
</tr>
</tbody>
</table>

**Notes:** Neither row has a `recordNumber`, yet both import and land in the same `Container` &mdash; zero RecordNumber identifiers are created. This is the trade-off of enabling the setting: the two items in the container aren't individually identifiable by identifier afterward, though each remains its own separate CollectionObject record.

#### containerize_dup_cat_no enabled, without a recordNumber, across separate imports

The same scenario as above, but as two separate imports rather than two rows in one import &mdash; behaves identically, since the setting, the Namespace, and the resulting `Container` are all ordinary project-level records with nothing tying them to a particular import.

**Test spreadsheet:** [`container_no_record_number_setting_enabled_across_imports_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_no_record_number_setting_enabled_across_imports_a.tsv), [`container_no_record_number_setting_enabled_across_imports_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_no_record_number_setting_enabled_across_imports_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.

**Settings:** `Containerize specimen with existing ones when catalog number already exists` enabled, for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>A</td>
  <td>occ-g1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1400</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1400</td>
</tr>
<tr>
  <td>B</td>
  <td>occ-g2</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>1400</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1400</td>
</tr>
</tbody>
</table>

**Notes:** One `Container` holds both CollectionObjects, exactly as in the same-import version, with zero RecordNumber identifiers created.

#### Spanning separate imports

The disambiguated (recordNumber-present) case from above works identically across two separate imports, not just within one &mdash; a container can grow every time you re-import into the same project.

**Test spreadsheet:** [`container_spans_imports_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_spans_imports_a.tsv), [`container_spans_imports_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_spans_imports_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.
- A Namespace with short name `REC`, delimiter `NONE`.

**Settings:** None (all defaults), for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>A</td>
  <td>occ-e1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>900</td>
  <td>RE1</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT900</td>
  <td class="outcome-col">RECRE1</td>
</tr>
<tr>
  <td>B</td>
  <td>occ-e2</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>900</td>
  <td>RE2</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT900</td>
  <td class="outcome-col">RECRE2</td>
</tr>
</tbody>
</table>

**Notes:** One `Container` holds both CollectionObjects, despite them coming from two entirely separate import runs. This is the intended, disambiguated version of cross-import containerization &mdash; both items keep distinct `recordNumber`s.

#### Same catalogNumber, no recordNumber, across separate imports

The rejection from [Duplicate catalogNumber](#duplicate-catalognumber) holds across imports too: without a `recordNumber`, a colliding `catalogNumber` is expected to error rather than silently containerize, even when the collision spans two separate import runs.

**Test spreadsheet:** [`container_no_record_number_across_imports_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_no_record_number_across_imports_a.tsv), [`container_no_record_number_across_imports_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/container_no_record_number_across_imports_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.

**Settings:** None (all defaults), for both imports.

<table class="spec-table">
<colgroup>
  <col style="width: 3.5em;">
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>A</td>
  <td>occ-f1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1000</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1000</td>
</tr>
<tr>
  <td>B</td>
  <td>occ-f2</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>1000</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
</tbody>
</table>

**Notes:** Import B's row errors with `catalogNumber: "Is already in use"`, exactly as it would within a single import. No `Container` is created.

#### recordNumber reused across unrelated catalogNumbers

A `recordNumber` value repeated on rows with two genuinely *different* `catalogNumber` values doesn't trigger any containerization &mdash; containerization is driven entirely by a `catalogNumber` collision, never by `recordNumber` alone.

**Test spreadsheet:** [`record_number_reused_across_catalog_numbers.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/record_number_reused_across_catalog_numbers.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/record_number_reused_across_catalog_numbers.tsv">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.
- A Namespace with short name `REC`, delimiter `NONE`.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>recordNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
  <th class="outcome-header">Record<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>1200</td>
  <td>RSAME</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1200</td>
  <td class="outcome-col">RECRSAME</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>1300</td>
  <td>RSAME</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT1300</td>
  <td class="outcome-col">RECRSAME</td>
</tr>
</tbody>
</table>

**Notes:** Two separate CatalogNumber identifiers, no `Container`. Reusing a `recordNumber` by itself is harmless: containerization is driven only by a `catalogNumber` collision, and these two `catalogNumber` values genuinely differ. The row where both a `catalogNumber` *and* a `recordNumber` collide is the one documented above, under "Same catalogNumber, same recordNumber."

### Name matching

How the importer decides a row's `scientificName` refers to a TaxonName it's already created (or that already existed), rather than creating a duplicate. Starting with the simplest possible case; expect this group to grow, since nomenclature matching is the most involved part of the importer.

#### Same scientificName imported twice

**Test spreadsheet:** [`scientific_name_matched_same_import.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_matched_same_import.tsv), [`scientific_name_matched_import_a.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_matched_import_a.tsv), [`scientific_name_matched_import_b.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_matched_import_b.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A fresh project &mdash; no pre-existing nomenclature is required.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col style="width: 5em;">
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>import</th>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">same<br>import</td>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td rowspan="2">separate<br>imports</td>
  <td>occ-a1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b1</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** The first row (or first import) creates the `Orotettix` genus and `andeanus` species TaxonNames. The second row (or second import) matches both rather than creating duplicates &mdash; 0 TaxonNames created, and both CollectionObjects' TaxonDeterminations reference the same species TaxonName. This holds identically whether the two rows are in the same import or two separate imports, since TaxonNames, like Namespaces and Identifiers, are ordinary project-level records with nothing tying them to a particular import.

#### scientificName already exists in the project

The same matching applies when the genus and species already existed before the import even started &mdash; entered by a curator directly, or from an earlier, unrelated import. This is the same [minimum required fields](#minimum-required-fields) example, but with the nomenclature already in place.

**Test spreadsheet:** [`minimum_required_fields.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/minimum_required_fields.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/minimum_required_fields.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Orotettix` (genus) and `andeanus` (species), already present.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>spec-001</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** Compare with [Minimum required fields](#minimum-required-fields) above, where the identical spreadsheet creates 2 new TaxonNames in a fresh project. Here, the row's TaxonDetermination references the pre-existing species TaxonName directly &mdash; matching happens the same way whether the name was created moments earlier by this same importer or already existed beforehand.

#### scientificName omits a subgenus present in the project

A `scientificName` doesn't have to spell out every rank to match. If the project already has the species placed under a subgenus (`Genus (Subgenus) species`), a `scientificName` giving just `Genus species` &mdash; omitting the subgenus &mdash; still matches it.

**Test spreadsheet:** [`scientific_name_missing_subgenus.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_missing_subgenus.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/scientific_name_missing_subgenus.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Camponotus` (genus) &rarr; `Tanaemyrmex` (subgenus) &rarr; `americanus` (species), already present.

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Camponotus americanus</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** No new TaxonNames are created &mdash; the row's TaxonDetermination matches straight through to `americanus`, nested under `Tanaemyrmex`. This example has only one subgenus to consider; when a genus has *multiple* subgenera and more than one contains a same-named species, matching this way can become ambiguous &mdash; see next.

#### Ambiguous subgenus homonym, no disambiguating information

If a genus has two subgenera that each contain a species with the identical name (e.g. `Camponotus (Tanaemyrmex) americanus (Mayr, 1862)` and `Camponotus (Myrmentoma) americanus (Emery, 1893)`), a `scientificName` of just `Camponotus americanus`, with nothing to tell the two apart, is ambiguous. It's expected to error, naming the candidates.

::: danger
TaxonWorks does not currently do this. Instead, the row imports successfully, and its TaxonDetermination silently matches the bare genus (`Camponotus`) &mdash; not either species, and not even the subgenus. Nothing in the row's status or messages indicates that anything was lost; the only sign is that the determination is coarser than the `scientificName` you provided. If you're relying on species-level determinations, this is worth checking for directly (e.g. reviewing determinations left at genus rank) rather than assuming an `Imported` status means the full name resolved.
:::

**Test spreadsheet:** [`scientific_name_ambiguous_subgenus_homonym.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_ambiguous_subgenus_homonym.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/scientific_name_ambiguous_subgenus_homonym.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Camponotus` (genus) &rarr; `Tanaemyrmex` (subgenus) &rarr; `americanus` (Mayr, 1862), and `Camponotus` &rarr; `Myrmentoma` (subgenus) &rarr; `americanus` (Emery, 1893).

**Settings:** None (all defaults).

See [next](#ambiguous-subgenus-homonym-disambiguated-by-scientificnameauthorship) for how providing `scientificNameAuthorship` resolves this same ambiguous data correctly.

#### Ambiguous subgenus homonym, disambiguated by scientificNameAuthorship

The same ambiguous data as above, but with `scientificNameAuthorship` provided. It can include just the author (`Emery`), or the author and year together (`Mayr, 1862`) &mdash; either is enough to disambiguate here, since the two candidate species have different authors.

**Test spreadsheet:** [`scientific_name_disambiguated_by_author_year.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_disambiguated_by_author_year.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/scientific_name_disambiguated_by_author_year.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Camponotus` (genus) &rarr; `Tanaemyrmex` (subgenus) &rarr; `americanus` (Mayr, 1862), and `Camponotus` &rarr; `Myrmentoma` (subgenus) &rarr; `americanus` (Emery, 1893).

**Settings:** None (all defaults).

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>scientificNameAuthorship</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Camponotus americanus</td>
  <td>Mayr, 1862</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Camponotus americanus</td>
  <td>Emery</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

**Notes:** Row 1's TaxonDetermination correctly resolves to the `Mayr, 1862` species; row 2's to the `Emery` species &mdash; each matched via a different `subgenus`, despite neither row's `scientificName` mentioning a subgenus at all. No new TaxonNames are created for either row.

#### scientificNameAuthorship with a typo matches no existing candidate

Matching `scientificNameAuthorship` against an existing name's author is an exact string match, not a fuzzy one. A single-character typo is therefore indistinguishable from a genuinely different author: it does not match the intended candidate, and does not error either &mdash; it creates a new, unwanted homonym species instead.

**Test spreadsheet:** [`scientific_name_author_typo.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_author_typo.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/scientific_name_author_typo.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Camponotus` (genus) &rarr; `Tanaemyrmex` (subgenus) &rarr; `americanus` (Mayr, 1862), and `Camponotus` &rarr; `Myrmentoma` (subgenus) &rarr; `americanus` (Emery, 1893).

**Settings:** None (all defaults). See [Restrict import to existing nomenclature only](#restrict-import-to-existing-nomenclature-only) for how this same input is rejected instead, with that setting enabled.

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>scientificNameAuthorship</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Camponotus americanus</td>
  <td>Emory</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
</tr>
</tbody>
</table>

::: warning
`Emory` is a one-letter typo of the existing `Emery, 1893` species' author. A conforming importer cannot distinguish a typo from a genuinely new, different author on the same name &mdash; both are, correctly, treated as "no existing match," and a new species TaxonName `Camponotus americanus (Emory)` is created directly under the genus `Camponotus`, alongside the two pre-existing subgenus-nested species. This is the standard, intended behavior of unmatched-name handling (see [Minimum required fields](#minimum-required-fields)), not specific to this ambiguous-homonym scenario &mdash; it is called out here because a typo is easy to introduce and easy to miss precisely in cases like this one, where multiple identically-spelled species already exist.
:::

**Notes:** Verify author spelling carefully against the target project's existing nomenclature before import, particularly when multiple identically-spelled species already exist. Projects that want mismatches like this one rejected instead of silently creating a new name can enable [Restrict import to existing nomenclature only](#restrict-import-to-existing-nomenclature-only).

## Settings

Each `Settings` toggle is documented here with the minimal data that makes it meaningful, and what's expected with the option off versus on.

### Containerize specimen with existing ones when catalog number already exists

Only meaningful for a row whose `catalogNumber` collides with an existing one and which has no `recordNumber` &mdash; if a `recordNumber` is present, containerization already happens regardless of this setting (see [Containers](#containers)).

**Test spreadsheet:** [`duplicate_catalog_number_same_import.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/duplicate_catalog_number_same_import.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/duplicate_catalog_number_same_import.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `CAT`, delimiter `NONE`.

**Off (default):**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>catalogNumber</th>
  <th>TW:Namespace:catalogNumber</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
  <th class="outcome-header">Catalog<wbr>Number value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Orotettix andeanus</td>
  <td>100</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-imported); font-weight: 600;">Imported</span></td>
  <td class="outcome-col">2</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">1</td>
  <td class="outcome-col">CAT100</td>
</tr>
<tr>
  <td>occ-b</td>
  <td>PreservedSpecimen</td>
  <td>Sphenarium purpurascens</td>
  <td>100</td>
  <td>CAT</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col"><em>(none)</em></td>
</tr>
</tbody>
</table>

Row 2 errors with `catalogNumber: "Is already in use"` &mdash; a colliding `catalogNumber` with nothing to disambiguate it is rejected by default.

**On:** with the setting enabled, row 2 above instead imports and is containerized alongside row 1 &mdash; that's the entire purpose of the setting, an opt-in trade-off: the two items end up sharing one `Container` with no `recordNumber` on either to tell them apart afterward. See [Containers &gt; containerize_dup_cat_no enabled, without a recordNumber](#containerize-dup-cat-no-enabled-without-a-recordnumber) for the full worked example.

### Restrict import to existing nomenclature only

By default, a `scientificName` (with or without `scientificNameAuthorship`) that doesn't match any existing TaxonName creates a new one. With this setting enabled, importing is restricted to matching against nomenclature that already exists in the project: an unmatched name errors the row instead of creating anything.

**Test spreadsheet:** [`scientific_name_author_typo.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/scientific_name_author_typo.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/scientific_name_author_typo.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- TaxonNames `Camponotus` (genus) &rarr; `Tanaemyrmex` (subgenus) &rarr; `americanus` (Mayr, 1862), and `Camponotus` &rarr; `Myrmentoma` (subgenus) &rarr; `americanus` (Emery, 1893).
- The same row as [scientificNameAuthorship with a typo matches no existing candidate](#scientificnameauthorship-with-a-typo-matches-no-existing-candidate): `scientificName: "Camponotus americanus"`, `scientificNameAuthorship: "Emory"` &mdash; a typo of the existing `Emery, 1893` species, matching neither existing candidate.

**Off (default):** the row imports, creating a new species TaxonName. See the full worked example at [scientificNameAuthorship with a typo matches no existing candidate](#scientificnameauthorship-with-a-typo-matches-no-existing-candidate).

**On:**

<table class="spec-table">
<colgroup>
  <col>
  <col>
  <col>
  <col>
  <col style="width: 1em;">
  <col style="width: 4.5em;">
  <col style="width: 5em;">
  <col style="width: 5.5em;">
  <col style="width: 5.5em;">
</colgroup>
<thead>
<tr>
  <th>occurrenceID</th>
  <th>basisOfRecord</th>
  <th>scientificName</th>
  <th>scientificNameAuthorship</th>
  <th class="col-spacer">&nbsp;</th>
  <th class="outcome-header">status</th>
  <th class="outcome-header">Taxon<wbr>Names created</th>
  <th class="outcome-header">Collection<wbr>Objects created</th>
  <th class="outcome-header">Taxon<wbr>Determinations created</th>
</tr>
</thead>
<tbody>
<tr>
  <td>occ-a</td>
  <td>PreservedSpecimen</td>
  <td>Camponotus americanus</td>
  <td>Emory</td>
  <td class="col-spacer">&nbsp;</td>
  <td class="outcome-col"><span style="color: var(--color-import-errored); font-weight: 600;">Errored</span></td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
  <td class="outcome-col">0</td>
</tr>
</tbody>
</table>

Row 1 errors with `scientificName: "Protonym americanus not found with that name and/or classification. Importing new names is disabled by import settings."` &mdash; no new TaxonName is created, and the pre-existing `Mayr, 1862` and `Emery, 1893` species are left untouched.

