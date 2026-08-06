---
sidebarPosition: 60
---

# Import

_Many projects start with imports, and target [Exports](/guide/export), the former are detailed here._

For strategies on migrating whole projects see [Migrate to TaxonWorks](/guide/migrate-to-TaxonWorks). This includes an overview of the many ways that data can be added to TaxonWorks.

::: tip
If you're running TaxonWorks locally (i.e. not in production or on a sandbox) then you'll likely need to manually trigger processing of the background jobs created during import. See [Delayed Jobs](/guide/import#seed-a-project-users-and-some-data-from-the-command-line) for details.
:::

## Batch loaders

There are various batch importers available within the [UI](/about/glossary#ui). These are polished to differing degrees and have various benefits and limitations. The required format, and often an example spreadsheet, is provided in the UI. All batch loaders are two-step, allowing for (and requiring) a preview of results before inserting them into the database.

- To explore available batch loaders click on a Data card in the Hub. If batch loader(s) are available then then the batch load link will be enabled.
- Batch importers largely target tab-separated text files, though this is not exclusively the case.
- Notable batch loaders are found in the TaxonNames, Otus, and Sources data cards, though others exist.
- Explore various batch loaders (each data card highlighted in yellow has associated batch loaders at this writing).

### Try a batch loader

In your test project,

1. Go to the data tab
2. Select the Otu Data card
3. Click “batch load”
4. See instructions in the UI for expected / accepted data types and format.
5. Create your own file or use this [test file](link)
   Header column = otu_name
   Blank lines are skipped
   Tab-delimited format, UTF-8 encoding, Unix line-endings required
6. Browse to your file to select it, click preview
7. If data looks as expected, browse to select that file again and click create.

Batch loaders (as of March 2022) include:

- OTUs [operational taxonomic units](https://docs.taxonworks.org/about/glossary.html#otu-operational-taxonomic-unit)
  - simple batch load
  - data attributes
  - simple batch file
  - OTU with identifier batch load
- collecting events
  - gpx (collecting events with georeferences)
  - castor
- collection objects
  - castor
  - buffered strings
- descriptors
  - qualitative descriptors
  - modify gene descriptor
- sequences
  - Genbank
  - Genbank batch
  - primers
- sources
  - BibTeX
- taxon names
  - simple
  - castor
- asserted distributions
  - simple
- namespaces
  - simple
- sequence relationships
  - primers batch

## Darwin Core Archive (DwC-A) import

### Checklist Data

To upload checklist data, this method supports simple and somewhat more complex taxon name lists. Below you will find examples to guide how to create your own datatset and datasets you can use to try in a sandbox.

### Preparing a Checklist

- Please check the table below for terms (fields) the importer recognizes and whether or not certain fields are required or have dependencies (e. g. formatting, identifiers). This is the **mapping** step.
- Identifiers are required in the following columns for this method to work (`taxonID`, `acceptedNameUsageID`, `parentNameUsageID`).
  - The `originalNameUsageID` column must be present for the dataset to import. The software will generate the numbers for you for this column if you don't fill it out (it duplicates the number in the taxonID column).
- Running the exact same dataset in twice will not duplicate names in the case where
  - a) the `parentNameUsageID` _is null_ AND
  - b) you use the `Settings` option to match on existing names where the `parentNameUsageID` is null.
  - IF `parentNameUsageID` _is null_ and you do not use the `Settings` option, the names will be entered (again) as children of `Root` and will say `[GENUS Unspecified]`. These would need to be cleaned up by hand after import.
- Your data in your spreadsheet first goes through a `Staging` step. You will be able to edit data in each cell at the point, if need be, before you click on `Import`.
- Each name you want to import must have its own record row in your dataset. For example, if you will be including **higher classification** data, each of those higher taxa must have their own row. If not, your higher classification data for each taxa **will not import.**
- Your dataset needs to be in xlsx, comma (csv), or tab-separated (txt, tsv) format.
- For best results for how diacritics are handled (like umlauts or tildas), ensure your data are UTF-8 encoded.

:::tip
- This method imports only. It does not update (e. g. fix typos) on a re-try or add more data to a given exiting object in the database.
- Case of `taxonRank` values doesn't seem to matter.
- IF you want to match on existing names, what matters are [TO BE VERIFIED]: only the `sciName` + `scientificNameAuthorship`. (Note a change to higher classification doesn't seem to matter to making the match).
:::

| Term                                                 | Mapping                                                                                                           |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `taxonID`                                            | REQUIRED - a unique identifier for the taxa in this record row                                                    |
| `parentNameUsageID`                                  | REQUIRED - a unique identifier for asserting the correct parent                                                   |
| `parentNameUsage`                                    |
| `acceptedNameUsageID`                                | REQUIRED - if the name is a valid one, this matches the `taxonID`                                                 |
| `scientificName`                                     | REQUIRED                                                                                                          |
| `kingdom`                                            |
| `class`                                              |
| `order`                                              |
| `family`                                             |
| `genus`                                              | REQUIRED                                                                                                          |
| `subgenus`                                           |
| `specificEpithet`                                    | REQUIRED                                                                                                          |
| `infraspecificEpithet`                               |
| `taxonRank`                                          | REQUIRED - Family, Genus, Tribe, Subtribe, Species, etc (not case sensitive)                                      |
| `scientificNameAuthorship`                           | REQUIRED\* - Must provide IF you want to match on existing names in the db (and same format)                      |
| `originalNameUsageID`                                | REQUIRED - Column must be present. IF all cells empty, software will populate them with taxonID at `Staging` step |
| `nomenclaturalCode`                                  | ICZN, ICN - This can be selected in the importer; does not have to be in the spreadsheet                          |
| `TW:TaxonNameClassification:Latinized:Gender`        | note maps directly to the TW datamodel; see TW:\<data model\>:...                                                 |
| `TW:TaxonNameClassification:Latinized:PartOfSpeech`  | note maps directly to the TW datamodel; see TW:\<data model\>:...                                                 |
| `TW:TaxonNameRelationship:incertae\_sedis\_in\_rank` | note maps directly to the TW datamodel; see TW:\<data model\>:...                                                 |
| `TW:TaxonNameClassification:Iczn:Fossil`             | note maps directly to the TW datamodel; see TW:\<data model\>:...                                                 |

|
| **need to search codebase to see if these are supported on import** |
| taxonomicStatus | valid, incertae sedis, obsolete combination |
| originalNameUsage |
| cultivarEpithet |
| nameAccordingTo |
| nomenclaturalStatus |
| taxonRemarks |
| references |

### The Checklist Importer

What follows are the simplest steps when uploading names into an empty database. It is possible to **match** on existing names in your TW project in the event you are importing children of those names, for example.

1. From the `Task` list select `Darwin Core Archive (DwC-A) import`

#left[DwC-A Checklist Importer Task](https://sfg.taxonworks.org/s/0wsyos[shows the task card labeled DwC-A Import])

2. In the importer interface, enter a `Description` for your dataset

#left[DwC-A Checklist Importer Screen in TaxonWorks](https://sfg.taxonworks.org/s/fh5j3y[shows layout of drag and drop importer with description, nomencaltural code, and dataset type fields])

3. Next, select the `Dataset type`. In this case, `Checklist`
4. Then, select the relevant `Nomenclature code`
5. Once you prepare your dataset, click to upload it by picking or drag and drop the file.

   - Depending on the file type (xlsx, csv, txt, tsv) you will need to verify the **separator** (delimiter) for the fields and strings. With xlsx files, the importer figures this out. With csv (comma) and txt (tab) you will get a pop-up asking you to confirm or pick the correct options.
   - In either of these delimiter pop-ups, after you pick or verify, click `upload`.

   #left[Checklist CSV file delimiter verification](https://sfg.taxonworks.org/s/6dyixc [popup to verify the delimiters used in the csv file to be uploaded])

   #left[Checklist TXT file delimiter verification](https://sfg.taxonworks.org/s/35agr4 [popup to verify the delimiters used in the text file to be uploaded])

6. The software will `Stage` your data now (it will take a few seconds or a bit longer depending on the size of the import).

#left[The DwC-A Importer `Staging` step](https://sfg.taxonworks.org/s/srblkw[showing the data ready to import with editable fields before import]) 7. In the resulting staged view, you can edit data in cells (only before the actual import)

- Note at this point, you can sort on the columns and replace values in all or any of the cells if necessary (you cannot edit the header rows).
  - Note your original dataset is stored permanently, but not with values you change after `Staging`.

7. Next click `Import`

- Names will import and you can click on `Browse` for a given row in your dataset to see the data in TW.

#left[The DwC-A Checklist Importer `Browse` after upload](https://sfg.taxonworks.org/s/25r8ml[shows the have been imported and gives you a button per row to see each imported record])

8. If you get error messages, rows with errors don't upload. You can click where it says `Error` to get the error message.

- For some errors, you can fix them in the spreadsheet and then try to `Import` that row/s again.
- For example, you might discover an error message **unparsed tail** for a given cell. Sometimes, it might indicate their is an encoding (diacritic) issue or a hidden character. Try retyping the value for that cell and then click to try re-import of that errored row.

9. In the `Import` pop-up, note you can select `Retry errored records` where you've changed the data in the relevant cells and then click `Start import`.

#left[DwC-A Checklist Retry errored rows](https://sfg.taxonworks.org/s/z1sc7c[pop up with option to retry import of errored record rows that did not upload])

10. You can always download your original dataset.

#### Sample Datasets

We offer five different example datasets (in various file formats) differing in complexity and source (e. g. one of them is from the DwC-A file from a [Plazi Treatment Bank Treatment](https://treatment.plazi.org/GgServer/summary/FFA0AE675753FFF9FFC5FFF1FFA9530C)). Please use them to try out the DwC-A Checklist Importer and as models for your own dataset tests and uploads.

#### Simplest Basic Checklist

This dataset inserts a genus and 5 species in that genus. We provide this sample dataset in 3 file formats, csv, txt, xlsx. It was used to upload names into an empty project (no records in the database).

- [Basic sample dataset - CSV](/examples/sample-checklist-files/basic-simplest-checklist-20250301.csv)
- [Basic sample dataset - TXT](/examples/sample-checklist-files/basic-simplest-checklist-20250301.txt)
- [Basic sample dataset - XLSX](/examples/sample-checklist-files/basic-simplest-checklist-20250301.xlsx)

::: tip
Import `Settings` did not seem to matter in this case since we were not trying to macth on any existing names in the database.

<!---
see SANDWORM 07, 08 -->

:::

#left[Simple DwC-A Checklist](https://sfg.taxonworks.org/s/t5nljb[image showing fields used and sample data for this simplest upload])

#### A Published Genus with many new species

In this use case, we take advantage of the Darwin Core Archive formatted **treatment** files that Plazi produces when it pulls names out of existing published literature. With these treatment files you need to add or adjust very few fields (term) headers and the identifiers you need are already in place. This dataset adds 300 names, one new genus and 299 new children of that genus. We did test where the validly published genus was also NOT already in the database. We then also tested how to match on an existing Genus already in the database. See the process below.

If you are adding new children to an existing genus in the database, then be sure to

- Use the `Settings` option to match on existing names in the database. Note well that in order to match on existing, the `scientificName` string and `scientificNameAuthorship` in the dataset must match the database.

Here is one simple version (derived from Plazi Treatment Bank taxa.txt from inside the DwC-A file for a given treatment). This file will import 300 names. NOT all fields in this file are imported.

1. From the original taxa.txt file

   - we removed all the synonyms, just leaving new species
   - we added a row for the Genus, _Galeopsomyia_, to match the parent in the TW database
   - in the genus row, we put a `1` for `taxonID`, `acceptedNameUsageID`, and `originalNameUsageID`.
   - in the `parentNameUsageID` column we added a `1` for all the species
   - for the `scientificNameAuthorship` for the genus row, we made sure to match the Author name as it appears in the database.
   - we edited the **combinationAuthor** field to match the paper (there was a parsing error in the Plazi Treatment which has been fixed)

2. Dataset

- [Original zipped treatment containing multiple files](/examples/sample-checklist-files/plazi-sample-checklist/FFA0AE675753FFF9FFC5FFF1FFA9530C.zip)
- [Original taxa.txt file](/examples/sample-checklist-files/plazi-sample-checklist/FFA0AE675753FFF9FFC5FFF1FFA9530C\taxa.txt)
- [Modified taxa.txt file](/examples/sample-checklist-files/plazi-treatment-text-file-modified.txt)

3. So, if you have names to upload, it can pay to check [Plazi Treatment Bank](http://plazi.org/treatmentbank/) to see if they have already parsed the names of interest from that published literature.

To test the entire scenario, have a look at the [Modified taxa.txt file](/examples/sample-checklist-files/plazi-treatment-text-file-modified.txt) and try using it to import (into a sandbox account). Columns not recognized by the importer will be ignored.

**Note** there are other usefule files in the Plazi Treatment DwC-A pkg

- The ([references.txt](/examples/sample-checklist-files/plazi-sample-checklist/FFA0AE675753FFF9FFC5FFF1FFA9530C/references.txt)) that specifies the page numbers for each new taxon name.
- With some work, we could adjust the importer to add or match on an existing source
- We could imaging, on import, adding a citation for that name inside that source on the specific page.
- With the [multimedia.txt](/examples/sample-checklist-files/plazi-sample-checklist/FFA0AE675753FFF9FFC5FFF1FFA9530C/multimedia.txt) data we could link to images (figures) that Plazi processing has deposited in Zenodo as part of creating the treatment.
- Using the [occurrences.txt](/examples/sample-checklist-files/plazi-sample-checklist/FFA0AE675753FFF9FFC5FFF1FFA9530C/occurrences.txt) we could pull in data from the materials examined information for each specimen cited in the treatement.

Meanwhile, you can use `Citations by Source` to easily add the source page numbers provided in the treatment to each citation record in TW.

::: tip
Do check the page numbers that the treament file asserts to ensure the paper was parsed correctly.
:::

#### Bryozoa names from a website

In this example set, we started with names we could see on the web (bryozoa.net) for the [year 2008](http://bryozoa.net/annual/taxa2008.html). The following files differ only in file format. Each will import 171 names. Note that to create this file, we had to create the identifier columns for (`taxonID`, `acceptedNameUsageID`, `parentNameUsageID`, and `originalNameUsageID`). (Some testing suggests that you can leave `originalNameUsageID empty and the upload will work. The column must be present however).

- [Bryozoa 2008 - CSV](/examples/sample-checklist-files/bryozoa-2008names-checklist.csv)
- [Bryozoa 2008 - TSV](/examples/sample-checklist-files/bryozoa-2008names-checklist.tsv)
- [Bryozoa 2008 - XLSX](/examples/sample-checklist-files/bryozoa-2008names-checklist.xlsx)

#### More Complex Checklists

Delving into more complex scenarios (synonyms for example) here are some examples for you to look at as you plan your name upload strategy. Note that [in these datasets](https://github.com/SpeciesFileGroup/taxonworks/tree/development/spec/files/import_datasets/checklists), the names existed in a source database. So the identifiers were from their own database. This set of upload test files comes from work done by the developer who wrote the `Checklist` Importer code.

#### Source data from Checklist Bank

[See recent work](https://github.com/speciesfilegroup/taxonworks/issues/3658) to show how you can use / modify datasets from Checklist Bank for importing in to Taxonworks.

---

### Occurrence Data

To upload occurrence data, TW offers the ability to use a DwC Archive file format. _For occurrences, the importer is presently limited to vouchered specimen data records._

To use this approach you must have your specimen data in a single spreadsheet-style format that can be exported as "CSV".

Preparing for an import follows the following general procedures:

- [Map your data](/guide/import#map-your-data) (provide a column header) for each column of data to be imported
- [Add TaxonWorks data to support your DwC import](/guide/import#add-taxonworks-data-to-support-your-dwc-occurrence-data-import) by creating records that will be used during the import process
- [Configure Settings in the import task](#configure-settings-in-the-import-task) 

::: tip
As part of your process you may need to go back and forth between mapping, adding supporting TW data, and becoming familiar with supported import options
:::

#### Map your data

The DwC importer provides flexibility in importing diverse data. These fall in to several types:

1. DwC terms
2. User customizable data attributes
3. User customizable biocuration classes
4. TaxonWorks' model specific attributes

As headers, these will look like this:

| `catalogNumber`      | `TW:DataAttribute:CollectionObject:color` | `caste`                      | `TW:CollectingEvent:verbatim_collectors` |
| -------------------- | ----------------------------------------- | ---------------------------- | ---------------------------------------- |
| _A DwC term mapping_ | _A user customizable data attribute_      | _A TW biocuration attribute_ | _A TW specific attribute_                |

::: tip
A first step is to go through your data and figure out which column header types you'll need. Start by matching to supported DwC terms, then go on from there.
:::

#### DwC term mapping

When going from DwC, a flat format, to TaxonWorks you're moving your data from rows to Things. We can group the DwC terms into classes to reflect where they end up in TaxonWorks.

Of the terms described below, your occurrence import file is required to provide columns for:
- `occurrenceID`,
- `basisOfRecord`, and 
- `TW:TaxonDetermination:otu_id` and/or `scientificName` - each row must contain one (or both) of these columns; if `TW:TaxonDetermination:otu_id` is present then all other taxon determination columns are ignored.



##### Record-level class

| Term              | Mapping                                                                                                                                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`            | Only `PhysicalObject` is allowed as a value. If the value is empty or the term is not present it is assumed it is a `PhysicalObject`                                                                                                                                                 |
| `institutionCode` | The TW repository with an acronym equal to this value is assigned to the imported specimen                                                                                                                                                                                                                            |
| `collectionCode`  | Paired with `institutionCode` (when present) it is used to select the namespace for `catalogNumber` from the user-defined lookup table you define in the import settings. The value itself is not imported.                                                                                                                                               |
| `basisOfRecord`   | Only `PreservedSpecimen`, `FossilSpecimen` and their GBIF variants `PRESERVED_SPECIMEN` and `FOSSIL_SPECIMEN` are allowed. If the value is empty it is assumed it is a `PreservedSpecimen` (but note *the `basisOfRecord` column must still be present* in that case). |

##### Occurrence class

| Term              | Mapping                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `occurrenceID`    | Must be unique within your import; for reference/filtering purposes preferably universally unique, though that is not required.
| `catalogNumber`   | The identifier value for a Catalog Number local identifier. See [details below](#catalognumber-details).|
| `recordNumber`    | The identifier value for a Record Number local identifier. See [details below](#recordnumber-details).|
| `recordedBy`      | Imported as-is into the verbatim collectors field of the collecting event. Additionally, the value is parsed into unvetted people and assigned as collectors of the Collecting Event. (A given name is used to create a new Person the first time it appears in an import, after that the same Person is reused.)
| `individualCount` | The total number of entities associated with the specimen record (e.g. this record may be for a "lot" containing 6 objects). This field can be empty (no value) but it can't be 0.                                                                                                                                                                                                                                                                                                                   |
| `sex`             | Selects the biocuration class from the "sex" biocuration group to be assigned as biocuration classification for the specimen.                                                                                                                                                                                                                                                                                                                  |
| `preparations`    | Selects an existing preparation having the same name.                                                                                                                                                                                                                                                                                                           |
###### `catalogNumber` details
- **Rows with a `catalogNumber` are marked as Not Ready until their namespace has been resolved.**
- The namespace is matched to a TW namespace as follows:
  - if there's a `TW:Namespace:catalogNumber` value, check it and nothing else
  - if there's only a `collectionCode` value, check the `collectionCode` to namespace mapping in the settings
  - if there's a `collectionCode` and an `institutionCode` value, first check the `institutionCode:collectionCode` to namespace mapping in settings, and then check the `collectionCode` mapping in settings
- 
  - If you select the `Error records when computed identifier will not match catalogNumber` setting then your `catalogNumber` value *must* include its namespaced short name prefix (e.g. `abc123`, not just `123`). If that option is not selected then you can write your identifier value with or without its namespace prefix.
- If you require several records to share the same Catalog Number identifier, you may do so by enabling the `Containerize specimen with existing ones when catalog number already exists` import setting, along with distinct `recordNumber` values.

###### `recordNumber` details

- If not empty, the record is required to have the short name of the Namespace to pair with the `recordNumber` in a separate TW-specific column named `TW:Namespace:RecordNumber`.
  - Do not include namespace prefix with your `recordNumber` value, e.g. if your namespace short name is 'abc' then `recordNumber` should be '123', not 'abc123'.
- When a `recordNumber` is present then multiple rows can have the same `catalogNumber`: objects with the same `catalogNumber` are added to a virtual container to which the Catalog Number identifier is assigned, and then the Record Number identifiers are used to disambiguate objects within that container. 

##### Event class

| Term                | Mapping                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `eventID`           | An identifier for the Collecting Event, either a global identifier or a local event identifier. See [details below](#eventid-details). |
| `fieldNumber`       | The identifier value for a Field Number local identifier. See [details below](#fieldnumber-details). |
| `eventDate`         | The ISO8601-formatted date (YYYY-MM-DD) is split into start year, month, and day collecting event fields. If the value is composed of two dates separated by `/`, then the rightmost date is used as end date and split in the same way as the start date. If this data contradicts dates from other non-empty date-related terms, the record will fail to import.                                                                                                                                                                                                                                                                                                      |
| `eventTime`         | Time is split into start hour, minute, and second values for the collecting event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `startDayOfYear`    | Using `year` and the value for this term, month and day are calculated and stored in start year, month, and day collecting event fields. If the computed value contradicts dates from other non-empty date-related terms the record will fail to import.                                                                                                                                                                                                                                                                                                                                                                                    |
| `endDayOfYear`      | Using `year` and the value for this term, month and day are calculated and stored in end year, month, and day collecting event fields. If the computed value contradicts dates from other non-empty date-related terms the record will fail to import.                                                                                                                                                                                                                                                                                                                                                                                       |
| `year`              | The start date year of the collecting event. If the value contradicts dates from other non-empty date-related terms the record will fail to import.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `month`             | The start date month of the collecting event. If the value contradicts dates from other non-empty date-related terms the record will fail to import.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `day`               | The start date day of the collecting event. If the value contradicts dates from other non-empty date-related terms the record will fail to import.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `verbatimEventDate` | Verbatim date of the collecting event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `habitat`           | Verbatim habitat of the collecting event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `samplingProtocol`  | Verbatim sampling method of the collecting event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `fieldNotes`        | Field notes of the collecting event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

###### `eventID` details

- If not empty, the importer requires a TW-specific column named `TW:Namespace:EventID`, as follows: 
  - option 1: you may specify that the identifier is global by providing a global identifier type string, such as `Identifier::Global::Uuid`, `Identifier::Global::Lsid`, etc. (in this case no namespace is required);
  - option 2: provide the short name of a TW namespace;
  - option 3: if left empty, the importer assigns a dataset-specific namespace with a synthetic name that you can later change.
In cases 2 and 3 an `Identifier::Local::Event` is created.
- If you select the `Error records when computed identifier will not match eventID` setting then your `eventID` value *must* include its namespaced short name prefix (e.g. `abc123`, not just `123`). If that option is not selected then you can write your identifier value with or without its namespace prefix.
- _When an existing TW Collecting Event already has the identifier you define in this way, the importer re-uses it and all other event-related data is ignored (provided `fieldNumber`, if present, resolves to the same Collecting Event)._
- If a Collecting Event is already matched by `fieldNumber`, this identifier must match the same Collecting Event, otherwise the importer will reject the record.


###### `fieldNumber` details

- If not empty, then the short name of the Namespace to use with the identifier is required to be in a TW-specific column named `TW:Namespace:FieldNumber`. 
  - Do not include namespace prefix with your `fieldNumber` value, e.g. if your namespace short name is 'abc' then `fieldNumber` should be '123', not 'abc123'.
- _When an existing Collecting Event already has this identifier, the importer re-uses it and all other event-related data is ignored (provided `eventID`, if present, resolves to the same Collecting Event)._
- If a Collecting Event is already matched by `eventID`, this identifier must match the same Collecting Event, otherwise the importer will reject the record. 

##### Identification class

| Term             | Mapping                                                                                                                                                                                                                                                                            |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `identifiedBy`   | A list of names of people, groups, or organizations who assigned the Taxon to the subject. If possible, separate the values in a list with a vertical bar \| (known as a pipe) with a space on either side (e.g. <code>Theodore Pappenfuss &#124; Robert Macey</code>). Names are matched to TW people (and/or organizations, depending on import settings) and assigned to a Taxon Determination. |
| `dateIdentified` | The date on which the subject was determined as representing the Taxon. Best practice is to use a date that conforms to ISO 8601-1:2019 [see examples](https://dwc.tdwg.org/terms/#dwc:dateIdentified).                                                                             |

##### Taxon class

| Term                       | Mapping                                                                                                                                                                                                                                                                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TW:TaxonDetermination:otu_id` | The id of the TW OTU to assign to the Taxon Determination of the imported Collection Object. _All other Taxon class columns below are __ignored__ when this id is provided._ For existing names this is the preferred and suggested way to import your determinations - see the 'Match OTU by Taxon Name' task in TW.
| `nomenclaturalCode`        | Selects the nomenclatural code for the taxon rank used when creating protonyms - allowed values are `iczn`, `icn`, `icnp`, `icvcn` (case-insensitive). The value itself is not imported. If Taxon Names need to be created then either this field or the corresponding default setting must be provided (this field overrides the default when both exist).                                                                                                                                                                                                                                                                     |
| `kingdom`                  | Creates (unless already present) a protonym at kingdom rank.                                                                                                                                                                                                                                                                                                                           |
| `phylum`                   | Creates (unless already present) a protonym at phylum rank.                                                                                                                                                                                                                                                                                                                            |
| `class`                    | Creates (unless already present) a protonym at class rank.                                                                                                                                                                                                                                                                                                                             |
| `order`                    | Creates (unless already present) a protonym at order rank.                                                                                                                                                                                                                                                                                                                             |
| `family`                   | Creates (unless already present) a protonym at family rank.                                                                                                                                                                                                                                                                                                                            |
| `genus`                    | __Ignored__. Extracted from `scientificName` instead.                                                                                                                                                                                                                                                                                                                                      |
| `subgenus`                 | __Ignored__. Extracted from `scientificName` instead.                                                                                                                                                                                                                                                                                                                                      |
| `specificEpithet`          | __Ignored__. Extracted from `scientificName` instead.                                                                                                                                                                                                                                                                                                                                      |
| `infraspecificEpithet`     | __Ignored__. Extracted from `scientificName` instead.                                                                                                                                                                                                                                                                                                                                      |
| `scientificName`           | One or more protonyms created (only when not present already) with their corresponding ranks and placements.                                                                                                                                                                                                                                                                               |
| `taxonRank`                | The taxon rank of the most specific protonym.                                                                                                                                                                                                                                                                                                                                          |
| `higherClassification`     | One or more protonyms created (only when not present already) with their corresponding ranks and placement. Processed after individual rank fields (`kingdom` through `subtribe`): any name in `higherClassification` that matches a name already established by an individual rank field (matched by string) is assigned that field's explicit rank. For names that appear only in `higherClassification` (not covered by an individual rank field), only family-group names will be created — any name above family-group that doesn't already exist in TW will result in error (it may already exist from a previously-processed row). _Names at genus rank or lower are ignored and extracted from `scientificName` instead._ |
| `scientificNameAuthorship` | Verbatim author of most specific protonym.                                                                                                                                                                                                                                                                                                                                           |

#### TaxonWorks mappings

The DwC importer task includes some TW-specific mappings that are neither DwC core terms nor in any DwC extension term lists but instead: direct mappings to predicates in your project imported as data attributes for collection objects and collecting events; biocuration groups and classes; and as an advanced-use feature you may have direct mappings to model fields.

::: warning
If submitting an actual DwC-A zip file and not a tab-separated text file or spreadsheet, these TW-specific mappings have to be placed as headers in the core table, and not in meta.xml. If you are replacing a mapping from meta.xml, you must make sure to comment it out and also if inserting columns make sure you do the appropriate adjustments to avoid collision. <-- Clarify this
:::

_See [Add TaxonWorks data to support your DwC import](/guide/import#add-taxonworks-data-to-support-your-dwc-occurrence-data-import) for how to create the records referenced in these mappings._

##### Mappings to project predicates

In cases where you need to import predicate values targetting the imported collection object or collecting event you may do so by naming the column with a pattern like `TW:DataAttribute:<target_class>:<predicate_identifier>`.
`<target_class>` may be `CollectionObject` or `CollectingEvent`, and the `<predicate_identifier>` may be either the name of the predicate or its URI. As an example if you have a predicate registered with name `ageInDays` and URI `http://rs.gbif.org/terms/1.0/ageInDays`, both `TW:DataAttribute:CollectionObject:ageInDays` and `TW:DataAttribute:CollectionObject:http://rs.gbif.org/terms/1.0/ageInDays` can be used to refer to the same predicate.

##### Mappings to biocuration groups and classes

The importer is able to map `sex` into the appropriate biocuration group and select the appropriate class according to the value. For additional mappings you may use a special column name pattern to select a biocuration group like `TW::BiocurationGroup:<group_identifier>` where `<group_identifier>` can be the name of the biocuration group or its URI. In addition the values must match an existing biocuration class and you may use either the class's name or URI. For example, if you have a biocuration group registered with name `Caste` and URI `urn:example:ants:caste` and biocuration class with name `Queen` and URI `urn:example:ants:caste:queen`, any combination of the following column header and cell value forms will create the same biocuration classification:

**Column header** (either works):
- `TW::BiocurationGroup:Caste`
- `TW::BiocurationGroup:urn:example:ants:caste`

**Cell value** (either works):
- `Queen`
- `urn:example:ants:caste:queen`

##### Mappings to DwC predicates

Whenever the importer sees that your project has custom attributes for collecting events and/or collection objects that match Darwin Core URI terms (`http://rs.tdwg.org/dwc/terms/<term>`), they will be imported as data attributes, regardless of any existing mapping of the same field. This allows to preserve verbatim dataset values for reference and also to import data from terms not supported by the importer.

##### Direct mapping to TW model fields

This is an advanced mapping and requires knowledge of the underlying TW models. The pattern is `TW:<model_class>:<field>` where model can be either [`CollectionObject`](https://docs.taxonworks.org/develop/Data/models.html#collection-object) or [`CollectingEvent`](https://docs.taxonworks.org/develop/Data/models.html#collecting-event), and `<field>` can be the ones listed below.

| Class              | fields                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CollectionObject` | `buffered_collecting_event`, `buffered_determinations`, `buffered_other_labels`, `total`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `CollectingEvent`  | document_label, elevation_precision, end_date_day, end_date_month, end_date_year, field_notes, formation, group, lithology, max_ma, maximum_elevation, member, min_ma, minimum_elevation, print_label, start_date_day, start_date_month, start_date_year, time_end_hour, time_end_minute, time_end_second, time_start_hour, time_start_minute, time_start_second, verbatim_collectors, verbatim_date, verbatim_datum, verbatim_elevation, verbatim_geolocation_uncertainty, verbatim_habitat, verbatim_label, verbatim_latitude, verbatim_locality, verbatim_longitude, verbatim_method, verbatim_trip_identifier |

### Add TaxonWorks data to support your DwC Occurrence data import

To import your DwC you many need to create several types of things in TaxonWorks. These include [namespaces](Manual/identifiers#namespaces) and [controlled vocabulary terms](Manual/customization#controlled-vocabulary-terms).

#### Namespaces

In the context of the DwC importer namespaces allow TW to

- Assign an Identifier as a CatalogNumber
- Track uniqueness of each object during the import, helping TW to normalize your data, turning it from rows to Things
- Group your Identifiers (and therefore the CollectionObjects they reference) as coming from a specific place

#### Controlled vocabulary terms

There are several kinds of CVTs that may be used in the import process.

::: tip
All CVTs are created and managed via the `Manage controlled vocabulary terms task`.
:::

##### Predicates

Think of Predicates as your custom column headers. Predicates are referenced in DataAttributes. Use a Predicate when you want to assign many different values (have rows with many different values) under one heading.

##### Biocuration classes

Think of biocuration classes as custom attributes for your collection objects, things like 'male', 'pupa', or 'larva'. These let you assign values useful for your curation of your specimens in a controlled way, ensuring problems like 'M.', 'MALE', 'ale' don't happen in what might otherwise be a "Sex" field. [TODO: reference groups?]. This approach is used when your rows have only a few specific values across the dataset.

### Configure Settings in the import task
Click on `Settings` in the upper right of the import task for an occurrence dataset to view the settings panel for that import.

#left[The settings panel](https://sfg.taxonworks.org/s/w8co44)

##### Nomenclatural Code
By default names created by the import are created under this code, but it can be overridden on a row-by-row basis using the `nomenclaturalCode` column.

##### Containerize specimen with existing ones when catalog number already exists
When processing a row, if the row has a catalog number that already exists on another collection object or container of collection objects, then the new collection object is combined with the existing one in a container or added to the existing container, and the catalog number is applied to the container instead of the objects in the container. Note you're required to supply a `recordNumber` in such a row so that the collection objects in a given container can still be uniquely identified.

##### Restrict import to existing nomenclature only
When checked, no new names will be created in TW. It's *not* necessarily true that a given name in the import must exactly match a name already in TW: whether this option is selected or not, we will sometimes attempt to match against multiple gender endings for a given name. If an import name can't be matched, the row is in error.

##### Error records when computed identifier will not match eventID
If you provide an `eventID` and the associated `TW:Namespace:EventID` provides a TW namespace, then the `eventID` must start with the short name of the namespace followed by its separator value. Apply this setting to ensure that your `eventID`s will all be exported exactly as you expect.

##### Error records when computed identifier will not match catalogNumber
If you provide a `catalogNumber` then it must start with the short name of the namespace determined by your `institutionCode:collectionCode` pairing. Apply this setting to ensure that your `catalogNumber`s will all be exported exactly as you expect.

##### Enable searching for Organization name in determinedBy field
When this option is enabled, the search for a name in the `determinedBy` field starts with Organizations and then People; otherwise it only searches People.

##### Also search for Organization alternate name
*Only applies when `Enable searching for Organization name in determinedBy field` is checked.* In that case the `determinedBy` field is also matched against the `alternate_name` field of Organizations.

##### Geographic Area matching settings
If your import includes one or more of the `county`, `stateProvince`, and `country` or `countryCode` DwC fields, then you may want to place requirements on the ways in which those fields match to a TW Geographic Area, using the options below. In general, the rules for matching are:
- county + country isn't allowed (you'd need to include a stateProvince as well), all other combinations are allowed and can match.
- nestings must be geopolitical as in TW, e.g. Illinois, Canada doesn't match even though both names are in TW.
- in general if a set of terms doesn't match, the finest (e.g. county) will be removed and a match attempted again, etc.
- **Warning**: the result is essentially randomly chosen when there's more than one match (mainly an issue at the county level, so including *only* county is more likely to result in a bad match, e.g.).

###### Require geographical area data origin
Require that the matched Geographic Area come from a particular gazetteer.

###### Only search for the finest geographical name provided
Without this option, if your DwC fields are 'Duckville', 'Ontario', 'Canada' and TW doesn't have a 'Duckville' county in Ontario, then your fields will match to the next best Geographic Area, in this case Ontario, Canada. With the option turned on those fields would not match any Geographic Area in this case. (In general a match must match all provided geographic fields with correct geopolitical containments, whether or not this setting is on.)

###### Require that the matched geographic area has a shape
Exactly as stated (not all Geographic Areas in TW include a shape).

###### Error if no geographic area with provided name exists
If no Geographic Area in TW is matched, the row reports an error.

##### Catalog number namespace mapping
**A Row with a `catalogNumber` is marked NotReady until a namespace has been resolved for it.** See [details above](#catalognumber-details) for how to provide a namespace.


### Specification examples

The examples below are "specification specs": each one pairs a minimal, single-purpose input file with an automated test asserting exactly what the importer does with it. They're generated directly from the TaxonWorks test suite, so if the importer's behavior ever changes, these examples (and their automated tests) will be updated together. This differs from the batch loader test files used elsewhere in TaxonWorks' test suite, which tend to be drawn from real (sometimes messy) production datasets and pin down bug fixes rather than illustrate one concept at a time.

Each example follows the same template:
- **Test spreadsheet** / **Test code** &mdash; links to the fixture file and to the RSpec context that asserts this example's behavior (GitHub, plus a local link &mdash; see tip below).
- **Input** &mdash; what must already exist in the project's database before the import (beyond a fresh project) for the scenario to apply, one bullet per item.
- **Settings** &mdash; the DwC importer `Settings` used, if any differ from the defaults.
- **The spreadsheet**, shown with a narrow blank column separating its input columns from a set of shaded outcome columns appended on the right. The `status` column uses the same colors as the row status in the importer's own UI (<span style="color: var(--color-import-imported); font-weight: 600;">Imported</span>, <span style="color: var(--color-import-errored); font-weight: 600;">Errored</span>, <span style="color: var(--color-import-not-ready); font-weight: 600;">NotReady</span>, <span style="color: var(--color-import-unsupported); font-weight: 600;">Unsupported</span>); the remaining outcome columns are counts of what got created (TaxonNames, CollectionObjects, TaxonDeterminations, etc., as relevant to the example) &mdash; so you can tell what happened at a glance, without reading prose.
- **Notes** &mdash; anything about the outcome that doesn't reduce to a single column value (e.g. relationships between the records that got created).

Examples are grouped the same way the term-mapping tables above are: [Record-level class](#record-level-class) first, then [Occurrence class](#occurrence-class), then Event class, and so on &mdash; plus a **Matching** group of its own for cross-cutting matching/disambiguation behavior (nomenclature matching, person matching, containerization, etc.) that doesn't belong to any single term, and a **Settings** group covering the DwC importer's `Settings` toggles.

These examples describe how a conforming DwC occurrence importer is expected to behave, not necessarily everything about how this particular installation currently behaves. Where TaxonWorks is known to fall short of the behavior described, that's called out with a red **danger** notice rather than folded silently into the example as if it were correct.

::: tip
The "locally" links below point at `http://localhost:4747/...`, a tiny static file server rooted at the taxonworks2 checkout (browsers block `http://` pages from linking directly to `file://` paths, so this is the workaround). Start it with `python3 -m http.server 4747 --bind 127.0.0.1` from the repo root. This is a personal dev convenience only, not required to read these docs.
:::

Fixture files: [on GitHub](https://github.com/SpeciesFileGroup/taxonworks/tree/development/spec/files/import_datasets/occurrences/specification) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification">locally</a>.
Their automated assertions: [`occurrence_specification_spec.rb` on GitHub](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>.

#### Minimum required fields

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

#### Minimum required fields, matching by OTU instead

There's a second, mutually exclusive minimal field set: replace `scientificName` with [`TW:TaxonDetermination:otu_id`](#taxon-class), which is matched to an already-existing OTU rather than to nomenclature. This is the recommended path for existing names &mdash; use the `Match OTU by Taxon Name` task in TW to build up the `otu_id`s to use beforehand. The OTU may or may not itself have an associated TaxonName; either way the TaxonDetermination is built directly from the OTU. No new TaxonNames are ever created via this path.

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

**Notes:** Row 1's TaxonDetermination uses OTU 900001 and (through it) its TaxonName. Row 2's TaxonDetermination uses OTU 900002 directly; its `otu.taxon_name` is `nil`, since that OTU has none. Providing `TW:TaxonDetermination:otu_id` makes all other Taxon class columns (`scientificName`, `taxonRank`, `kingdom`&hellip;) ignored if present &mdash; see [Taxon class](#taxon-class).

#### Record level

Covers the [Record-level class](#record-level-class) terms: `type`, `basisOfRecord`, and (a level down) the `BiocurationClass` that `basisOfRecord: FossilSpecimen` requires. `institutionCode` and `collectionCode` are also Record-level terms, but their *resolution* (matching a text value to a `Repository` or `Namespace`, including disambiguation when it's ambiguous) is covered under **Matching** instead &mdash; these two are kept here to the mechanics that don't involve matching.

##### type defaults

The [Record-level class](#record-level-class)'s other value-checked term. Like `basisOfRecord`, a blank `type` defaults to the one accepted value (`PhysicalObject`) rather than erroring. Unlike `basisOfRecord`, the match is case-sensitive and there's no GBIF-style reformatting &mdash; `physicalobject` is rejected exactly like any other unrecognized value.

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

##### basisOfRecord defaults

`basisOfRecord` must be present as a column (it's in the [required field set](#occurrence-data)), but the cell value itself may be blank &mdash; a blank cell defaults to `PreservedSpecimen`, matched case-insensitively, and GBIF's `SCREAMING_SNAKE_CASE` occurrence-download variants (`PRESERVED_SPECIMEN`, `FOSSIL_SPECIMEN`) are reformatted and accepted too. Anything else errors, naming the field.

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

##### FossilSpecimen, without the biocuration class present

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

##### FossilSpecimen, with the biocuration class present

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

#### Occurrence

Covers the [Occurrence class](#occurrence-class) terms: `occurrenceID`, `catalogNumber`, `recordNumber`, `recordedBy`, `individualCount`, `sex`, `preparations`. As with Record level, the *matching/disambiguation* parts of some of these terms (which `Namespace` a `catalogNumber` resolves to, containerizing rows that share a `catalogNumber`, reusing an existing `Person` for `recordedBy`) are covered under **Matching** instead.

##### Duplicate occurrenceID

Yes &mdash; a second row reusing an `occurrenceID` already seen earlier in the same import errors, even if every other field is otherwise valid. `occurrenceID` is assigned as an identifier in a namespace that's auto-created once per import dataset (see [`occurrenceID` mapping](#occurrence-class)) and shared by every row in the file, so a repeat value collides with the row that used it first.

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

##### catalogNumber namespace mechanics

A blank `catalogNumber` needs nothing. A `catalogNumber` paired with an explicit `TW:Namespace:catalogNumber` resolves immediately. A `catalogNumber` with neither that column nor an `institutionCode`/`collectionCode` mapping configured in `Settings` doesn't error &mdash; it stages as <span style="color: var(--color-import-not-ready); font-weight: 600;">NotReady</span> and stays that way until you resolve it. **NotReady rows are never included when the import is run** &mdash; they're simply excluded from processing (not attempted, not failed) until a namespace is set.

**Test spreadsheet:** [`catalog_number_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/catalog_number_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/catalog_number_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `ABC` and delimiter `NONE` (so the computed identifier reads as `ABC123`, not `ABC 123` &mdash; see [Namespaces](#namespaces)).

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

##### recordNumber namespace mechanics

Unlike `catalogNumber`, a `recordNumber` with no way to resolve a namespace doesn't stage as NotReady &mdash; it errors outright at import time, because (unlike `catalogNumber`) there's no `institutionCode`/`collectionCode`-based fallback mapping for `recordNumber`: the `TW:Namespace:recordNumber` column is the only path.

**Test spreadsheet:** [`record_number_namespace.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/record_number_namespace.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/record_number_namespace.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `DEF` and delimiter `NONE` (so the computed identifier reads as `DEF222`, not `DEF 222` &mdash; see [Namespaces](#namespaces)).

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

##### recordedBy

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

##### individualCount

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

##### sex

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

##### preparations

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

#### Matching

Cross-cutting matching/disambiguation behavior that doesn't belong to a single term &mdash; how the importer decides "is this the same thing I've already seen, or something new." As more of these accumulate, expect this group to grow its own Record-level/Occurrence/Event-class subdivisions the same way the sections above did; for now there's one.

##### occurrenceID reused across separate imports

A common real-world workflow: run an import, some rows error, fix the source file, re-run. Does re-running collide with the `occurrenceID`s that already imported successfully the first time? No &mdash; unlike a duplicate `occurrenceID` *within* one import (which errors, see [Occurrence](#occurrence) above), the same `occurrenceID` reused across two *separate* imports doesn't collide at all, because each import gets its own `occurrenceID` namespace (see [`occurrenceID` mapping](#occurrence-class)).

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

##### catalogNumber namespace resolution via institutionCode/collectionCode

Without an explicit `TW:Namespace:catalogNumber` column (see [catalogNumber namespace mechanics](#catalognumber-namespace-mechanics) above), a `catalogNumber`'s Namespace is instead resolved from `institutionCode`/`collectionCode`, matched against a mapping table configured through the DwC importer's `Settings` panel &mdash; not a spreadsheet column. Three mappings can be configured: `institutionCode` + `collectionCode` together (most specific), `collectionCode` alone, and `institutionCode` alone (its own distinct mapping, not a fallback that reuses the `collectionCode`-only one). When a row has both fields and both mappings are configured, the `institutionCode` + `collectionCode` mapping wins.

**Test spreadsheet:** [`catalog_number_namespace_by_institution_collection_code.tsv`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/files/import_datasets/occurrences/specification/catalog_number_namespace_by_institution_collection_code.tsv) &middot; <a href="http://localhost:4747/spec/files/import_datasets/occurrences/specification/catalog_number_namespace_by_institution_collection_code.tsv">locally</a><br>
**Test code:** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

**Input:**
- A Namespace with short name `INHS`, delimiter `NONE`.
- A Namespace with short name `GENERIC`, delimiter `NONE`.
- A Repository with acronym `INHS` &mdash; `institutionCode` is independently matched against `Repository` acronyms too (see [Record-level class](#record-level-class)); without a matching Repository, a row with an `institutionCode` errors regardless of catalogNumber namespace resolution.

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

##### catalogNumber must match its computed identifier verbatim

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

##### Duplicate catalogNumber

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

##### Duplicate recordNumber

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

##### Containers

`catalogNumber` collisions aren't always an error: if the colliding row also has a `recordNumber`, the importer merges the new CollectionObject into a `Container` with the earlier one instead of rejecting it &mdash; each item's `recordNumber` disambiguates it within the container. This is the intended way to import, say, a vial of several specimens logged as separate rows.

A setting, `Containerize specimen with existing ones when catalog number already exists` (see [Settings](#settings) below), additionally allows containerizing a colliding `catalogNumber` when no `recordNumber` is present at all &mdash; a deliberate trade-off some projects want: the items placed in the container this way aren't individually identifiable by identifier afterward, though each remains its own separate record.

`recordNumber` is allowed to repeat (see [Duplicate recordNumber](#duplicate-recordnumber) above), and that holds inside a container too: nothing requires the `recordNumber`s of items sharing a `catalogNumber` to actually be distinct from each other.

**Test code (all scenarios below):** [`occurrence_specification_spec.rb`](https://github.com/SpeciesFileGroup/taxonworks/blob/development/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb) &middot; <a href="http://localhost:4747/spec/models/dataset_record/darwin_core/occurrence_specification_spec.rb">locally</a>

###### Same catalogNumber, different recordNumbers (the intended case)

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

###### Same catalogNumber, same recordNumber

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

###### containerize_dup_cat_no enabled, without a recordNumber

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

###### containerize_dup_cat_no enabled, without a recordNumber, across separate imports

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

###### Spanning separate imports

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

###### Same catalogNumber, no recordNumber, across separate imports

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

###### recordNumber reused across unrelated catalogNumbers

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

#### Settings

Each `Settings` toggle is documented here with the minimal data that makes it meaningful, and what's expected with the option off versus on.

##### Containerize specimen with existing ones when catalog number already exists

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

### Unmapped columns

Column headers that can't be linked via one of the 3 mechanisms are ignored during the import process. This means it's important to do some trial runs in a sandbox, or with a smaller dataset to see that your values are mapping over. The `Browse collection object` task is a good place to check this.

::: tip
You can augment your data after import with batch update functionality inside TW. Carefully planning your overal import process can lead to a more efficient overall approach. Sometimes it's easier to work in spreadsheets, sometimes within a database.
:::

### Occurrence Import FAQ (Frequently Asked Questions)

#### When are CollectionObjects auto-created by an import?
- Always. Every imported row creates a single new Collection Object: a Specimen if `individualCount` is 1, otherwise a Lot.

#### When are CollectingEvents auto-created by an import?
- An existing Collecting Event is reused if it has an existing identifier matched by the row's [`eventID`](#eventid-details) and/or [`fieldNumber`](#fieldnumber-details) fields; otherwise a new one is created.

#### When are names related to Taxon Determination auto-created by an import?
- If you set the `Restrict import to existing nomenclature only` option in Settings, then never. If this is not set, read on.
- If you use the `TW:TaxonDetermination:otu_id` column to match to a name already present in TW, then never.
- If you use the `scientificName` and individual rank columns above genus, those names will be created when not already present.
- If you use the `higherClassification` column, only family-group names will be created as needed (higher rank names must match a rank column or an existing TW name).


## Drag and drop

Drag and drop loading of images and documents are accessible in various places in TW including the [Radial annotator](/about/glossary#radial-annotator), and, notably, `Tasks -> New image`.

## Record by record

When first learning TaxonWorks, entering records one-at-a-time offers you the opportunity to learn about more of the features in TW and get a feel for how you and others experience the UI.

For example, you want to enter a specimen record. You have two Tasks enabling you to do this. Choose to use Comprehensive Specimen Digitization Task or the Simple New Specimen Task.

### Try Simple New Specimen

In your project, try creating a simple new specimen record.

- Note you will need to select a namespace. You may find you need to add a namespace before you can do this TW task. Adding a value for namespace ensures your uploaded data records will be unique inside your TW project and across TW projects. In your project, you may also need more than one namespace. [Use Tommy’s INHS Insect Collection as an example, with 12 different namespaces that effectively group the various collections housed at INHS ENT].

- If you tried the OTU batch loader you can pick one of your OTUs for the name to assign to this specimen.

- Add an image if you wish

- Select the Preparation type for this specimen. You may need to add a new value to the dropdown using the New preparation type task.

## Coming from other software

### Scratchpads

We are in the process of exploring two routes to come from Scratchpads to TaxonWorks.

- The DwC import should work well for occurrence data that is based on collected objects.
- The SFG team has worked with a select number of individual Scratchpad curators to script the process of transferring their data. Contact us if you are interested in what this approach entails. Note that this process takes programming effort that is a limited resource within the SFG.

## Importing shapes (GIS)

### Background
TaxonWorks comes with its own *fixed* set of geographic shapes, called Geographic Areas. These consist solely of geopolitical shapes/boundaries at the county, state/province, and country levels. Frequently though individuals will have a need for more specialized shapes of, for example, a national park in which they study, or a particular water body, or one particular island. For these needs TaxonWorks has Gazetteers, which are individual named-shapes created/imported *as needed by users for their own use cases*. Gazetteers:
* have a name, such as "Mediterranean Sea" or "Awaji Island" (rarely will these be geopolitical names, as opposed to Geographic Areas)
* have a shape/boundary which is either created by the user or imported from a data file (typically a shapefile)
* are project-level objects, as opposed to Geographic Areas which are community data

### Creating Gazetteer shapes

#### The New Gazetteer task
We'll only touch briefly on the New Gazetteer task, which is geared more toward user-drawn shapes:

#left[New Gazetteer Task](https://sfg.taxonworks.org/s/75gj90)

The `name` field is required, `ISO 3166 A2/A3` are optional. You can see here that we've selected 2 of our projects to add the Gazetteer we're creating to.

:::warning
*You can't currently copy shapes between projects after importing* (though that option will be available in the future), so this is your one chance to do so.
:::

Creation options are:
* From Leaflet: draw a shape on a map using your mouse
* WKT coordinates: import a shape using the [well-known-text format](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)
* Enter coordinates of a point
* Add an existing Geographic Area or Gazetteer to your shape, by union or intersection

#### The Import Gazetteers task
The Import Gazetteers task allows you to import many shapes at once with whatever precision the original shapes were created with, via a [shapefile](https://en.wikipedia.org/wiki/Shapefile). Typically in this case the shapefile will be something you or a colleague found online and then downloaded.

:::tip
At this time we only support import of shapefiles. If you find the shapes you would like to import in a different format, you can likely use software such as ArcGIS (commercial), QGis, or GDAL (more advanced) to convert to the shapefile format.
:::

:::warning
We strongly encourage you to perform an initial test import on a *sandbox* to make sure everything turns out as expected; otherwise you may wind up with duplicated or missing shapes if there are issues with the import process and you have to run the import multiple times.
:::

:::tip
Similarly to Taxon Name imports, it can be helpful to examine your gazetteer data prior to attempting an import. If you have GIS software experience with ArcGIS (ESRI) or QGis or similar, it can save time to examine your data to check for issues such as:
* shapes with invalid geometry
* a column that provides names for your shapes (can be called anything, doesn't need to be `name`)
* duplicate names on different shapes
* missing shapes or names
* misspelled names or names with extra information you're not interested in

*Some* of these issues can be dealt with in an ad-hoc manner in TaxonWorks, but in any case will likely be easier to resolve in GIS software made precisely for dealing with such data.
:::

For these instructions we'll go through the process of importing the 8 Biogeographical Realms of the World as provided by a shapefile we can download online. The original data link is [https://data-gis.unep-wcmc.org/portal/home/item.html?id=f196d94a226d430fa214947d51dad35a](https://data-gis.unep-wcmc.org/portal/home/item.html?id=f196d94a226d430fa214947d51dad35a) but *we won't be working directly with that download file* since it has one of the data issues mentioned above. The shapes for Nearctic, Oceanic, and Palearctic each cross the prime-meridian line, so those regions are provided by that shapefile as *two* polygons each, each with the same name.

In our case we'd like to be able to filter by the entire region using just one polygon, not two, so we have two choices:
1) Import that shapefile as is and then use the Create or Edit Gazetteers task to create the union of each of those pairs (followed by deleting the originals)
2) Perform those unions using ArcGIS or QGis or similar and then import that new shapefile instead

We'll go with option 2 here, where we've already created the new shapefile for you, available [here](/examples/shapefiles/biogeographical_regions_of_the_world_2004-eight_shapes.zip). Download that shapefile and unzip it - TaxonWorks doesn't currently support importing the zip file directly.

##### Importing shapefile files
Shapefiles provide their data spread over several different files - the mandatory ones for import into TaxonWorks are the .shp, .shx, .dbf, and .prj files (visit the Wikipedia link above for more on what each of those files contributes).

The easiest way to get our shapefile files into TaxonWorks is to click on the New tab in the 'Shapefile documents' section:

#left[Shapefile documents chooser](https://sfg.taxonworks.org/s/ne2kv9)

Either drag your files into the drop region, or click in the drop region and select your files from the file selector (you can select them all at once by clicking the first one, then pressing the shift key and clicking on the last one). You should now see reds turned to green (as well as the yellow .cpg file, which is optional but should be included when you have one):

#left[Shapefile documents loaded](https://sfg.taxonworks.org/s/h6y457)

Note that your required shapefile documents have been auto-selected for you: they appear below the select in rows with trash can icons.

##### Selecting the Name column
It turns out shapefile data can be thought of a lot like we think of spreadsheet data: as columns and rows. We need to tell TaxonWorks which column has the names of our shapes - to do so click on the 'Select from shapefile fields' (i.e. columns) for the shapefile field containing the Gazetteer names:

#left[Shapefile name field input](https://sfg.taxonworks.org/s/n6egx6)

You'll see the list of all of the shapefile columns that might be your name column:

#left[Shapefile fields](https://sfg.taxonworks.org/s/l3sc5f)

These names were created by whoever made the shapefile, not necessarily with you in mind: in this case the likely choices are Realm or RealmCode, and RealmCode sounds more like a shorthand (it is), so we'll try 'Realm': click 'Realm' and then 'Select name field'.

:::tip
See the preview step described below for checking that the field you selected contains the data you expect.
:::

The name field is required for gazetteers; the next two, Iso 3166 A2 and A3, are not. They're not included by our shapefile, so we'll skip them.

##### Selecting a source to cite your imported gazetteers with
Next up you can enter a Source (the paper in which this data was described, e.g.) to cite each gazetteer you'll create. We'll pass this time.

##### Importing gazetteers to multiple projects
You also have the opportunity to choose which project(s) you'd like to import your shapes into - all of the projects you're currently a member of should be available as options:

#left[Choose project(s) to import to](https://sfg.taxonworks.org/s/ac3mqc)

:::warning
*You can't currently copy shapes between projects after importing* (though that option will be available in the future), so this is your one chance to do so.
:::

##### Previewing your import data
Before importing, you have the opportunity to *check that the choices you made above are actually giving reasonable looking data*. It's always a good idea to at least glance at the preview before importing. Here we see:

#left[shapefile import preview](https://sfg.taxonworks.org/s/48cp0n)

We see the eight expected regions, each with a single shape - that's what we want.

TaxonWorks does not include a shape preview.

:::warning
If we had worked with the original unaltered shapefile instead then at this point we would see Nearctic, Oceanic, and Palearctic each listed twice, each with a count of 2: a red flag that there was an issue requiring our attention.
:::

:::tip
The preview rows are ordered by the `Name` column. The first column of the preview, `Record number`, gives the row number of that name in the shapefile in case you need to make changes to your shapefile.
:::

##### Running the import job
The preview contains the data we're expecting, so click on `Process shapefile` to submit the import request. Imports run in the background: you should now see your job listed in the `Import Job Status` section:

#left[Import Job Status, new job](https://sfg.taxonworks.org/s/3zjk02)

Press the refresh button to track your job's status as the job runs. Very detailed very large shapes have been known to take tens of minutes to import; less detailed shapes can take under a second.

Here we see the job status is `Completed`, and we've imported all 8/8 shapes with no errors in about 2 seconds:

#left[Import Job Status, job completed](https://sfg.taxonworks.org/s/katruo)

##### Checking the results
Click on the Gazetteers link to see a list of some/all of your new Gazetteers, and click one to check the imported shape:

#left[Imported Palearctic shape](https://sfg.taxonworks.org/s/cajgwz)

Note that the Palearctic shape here displays as two pieces, split across the prime meridian, as expected.





