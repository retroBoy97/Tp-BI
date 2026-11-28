# Tp-BI — Power BI practicals (TP1 & TP2)

Two Business Intelligence practicals ("TP") done in **Power BI Desktop**. Everything here is French coursework:
the handouts (`Enoncé`) are PDFs, the work lives in `.pbix` / `.pbip` files, and there is no code to build or run.

- **TP1 — Hébergements classés (Paris & IDF)**: beginner. Clean two CSVs in Power Query (TP 1.1), build a
  4-page interactive report (TP 1.2), store file paths in parameters (TP 1.2 annex).
- **TP2 — Ventes « Meubles transcanadiens » 2013–2015**: advanced. Combine a folder of 36 monthly Excel files,
  reshape a messy text inventory report, generate a date table in M, build a 2-fact-table star model and
  10 time-intelligence DAX measures. **Scope agreed with the professor: up to page 40 of the handout.**
  The "Tableau de bord" page (handout pp. 41–42) was added so the measures can be demonstrated.

Both TPs were validated live with the professor on **2026-11-27** (see [Validation](#validation-2026-11-27)).

## Repository layout

```
Tp 1/
  Enoce/Tp 1.1.pdf, Tp 1.2.pdf        handouts (Power Query part / report part + annex)
  Hebergement-paris.csv               source data (atout-france.fr), ';' separated, cp1252
  fr_postal_codes.csv                 postcode → latitude/longitude
  TP1_Hebergement.pbix                single-file report (open this to demo)
  TP1_Hebergement.pbip                Power BI Project: same report as text files ↓
  TP1_Hebergement.Report/             report pages + visuals (PBIR JSON), custom theme
  TP1_Hebergement.SemanticModel/      queries, tables, measures, relationships (TMDL)
Tp2/
  Enoncé.pdf                          99-page handout (we stop at p. 40, + pp. 41–42)
  Sources_de_donnees/Sources_de_donnees/
    Ventes/Ventes_YYYY-MM.xlsx        36 monthly sales files (table "Table1")
    Dim*.xlsx                         dimensions (DimDatesFiscal is not used before p. 73)
    Inventaires.txt                   printed inventory report to reshape
  TP Avancé BI.pbix / .pbip / .Report/ / .SemanticModel/   same structure as TP1
guides/
  guide_TP1.html, guide_TP2.html      study guides (French): every step, why, likely questions
```

## Setup (Windows)

1. **Install Power BI Desktop** from the Microsoft Store (ID `9NTXR16HNW1T`, publisher Microsoft Corporation).
   The files were saved with version **2.159** (Nov 2026) in the new PBIR format: older versions may refuse them.
2. **Clone to a normal Windows path**, e.g. `C:\Users\<you>\Documents\TP-BI`. Power BI cannot reliably read
   files inside WSL (`\\wsl$\...`). OneDrive-synced folders work, but sync can briefly lock files.
3. **Fonts (optional)**: the theme uses **Geist** (body) and **Geist Serif** (titles). Install both on Windows,
   otherwise Power BI silently falls back to a default font.
4. **Enable maps** (TP1): File → Options and settings → Options → Global → Security →
   tick *Use Map and Filled Map visuals*, then **restart Power BI** (it does not apply until restart).
5. **Repoint the data paths** (next section). The report opens without this, because the data is cached in the
   file, but **Refresh** and the Power Query preview fail until the paths are yours.

### Repoint the data paths (TP 1.2 annex / TP2 pp. 12, 34)

Paths are stored in Power Query **parameters**; only their values are machine-specific.
Open the file → **Transform data** → **Manage Parameters** → set *Current value* → OK → **Close & Apply** → **Refresh**.

| File | Parameter | Value |
|---|---|---|
| TP1 | `CheminFichierHébergement` | `<repo>\Tp 1\Hebergement-paris.csv` |
| TP1 | `CheminFichierCodesPostaux` | `<repo>\Tp 1\fr_postal_codes.csv` |
| TP2 | `CheminDossierVentes` | `<repo>\Tp2\Sources_de_donnees\Sources_de_donnees\Ventes` |
| TP2 | `CheminDossierFichiersSources` | `<repo>\Tp2\Sources_de_donnees\Sources_de_donnees` (no trailing `\`) |

> **Note on the path parameters — do this first after cloning.**
> The committed values point to the machine the files were last saved on
> (`C:\Users\kacem\OneDrive\Documents\TP-BI\...`). Earlier versions pointed to `C:\Users\MSI\Downloads\...` and
> `C:\Users\Dell\Downloads\...`. On any other PC, **Refresh** fails with
> `DataSource.NotFound: Could not find a part of the path …` until you change the four values above to your own paths.
> The report still opens and filters correctly before that, because the data is cached inside the file.
> Change only the parameter *values*: never type a path back into a query's `Source` step. That is the whole
> point of the parameters (the annex's answer to "why use parameters?").

In the `.pbip` you can also edit the values directly in `*.SemanticModel/definition/expressions.tmdl`
(close Power BI first).

## `.pbix` vs `.pbip`

- **`.pbix`**: one binary file. Easiest to open, send, or hand in.
- **`.pbip`** (Power BI Project): the same report saved as text — report JSON under `*.Report/`, model as
  TMDL under `*.SemanticModel/`. Diffable in git, and editable by hand or by an agent such as Claude Code.
  Create one with File → Save as → *Power BI project files (*.pbip)*.
- `.gitignore` excludes `**/.pbi/cache.abf` (the data cache, ~25 MB for TP2) and `localSettings.json`. After a
  fresh clone, open the `.pbip` and click **Refresh** to rebuild the cache.
- To regenerate the `.pbix` from the `.pbip`: open the `.pbip` → File → Save as → *.pbix*.
  **Keep both in sync**: an outdated `.pbix` next to an updated `.pbip` is an easy way to demo the wrong file.

## TP1 — what the file contains

Details, step by step, in `guides/guide_TP1.html`.

- **Power Query**: `Hébergement temporaire` (CSV cleanup: `-` → empty before converting to numbers, split
  "3 étoiles", `LOCALISATION` address for the map, year) and `fr_postal_codes` (keep postcode + lat/long), both
  in the *Tables temporaires* group → `Hébergement` (left outer merge, `Regroupement` Paris / Hors Paris,
  room-count bands + `Ordre Intervalles` sort column, URL cleanup).
- **Model**: measure `NOMBRE DE CHAMBRES MOYEN`, bins of 25 rooms, data categories (Address, Web URL, Image URL).
- **Report pages**: *Evaluation des établissements*, *Etude des évaluations* (drill-down Année → Regroupement →
  Typologie → intervalle), *Recherche d'informations*, *Recherche d'établissements* (page filter excluding
  PARC RÉSIDENTIEL DE LOISIRS).

**Changed during the handoff:**
- Parameter values repointed (the parameters themselves already existed).
- New layout on all pages (same visuals, nothing removed) and custom theme `ThemeGeist.json`.
- Pages renamed to the handout titles.
- Map: bubble colour by number of stars (TP 1.2 p. 3).
- Year slicer no longer filters the line chart (TP 1.2 pp. 5–6).
- Year slicer switched to a **dropdown** (asked by the professor).

## TP2 — what the file contains

Details, step by step, in `guides/guide_TP2.html`.

- **Parameters**: `CheminDossierVentes`, `CheminDossierFichiersSources` (combined parameter:
  `CheminDossierFichiersSources & "\DimProduit.xlsx"`), `DateDébut` 2013-01-01, `DateFin` 2015-12-31, `NbJours`.
- **Ventes**: *Combine files* from the folder (helper queries *Exemple de fichier*, *Transformer le fichier*).
- **Inventaires**: `Lines.FromBinary` on the text report, then index + conditional column + fill down to rebuild
  the date (`15/12` → 2015-12-31, end of month), the store number and the inventory category; split / remove
  errors to get `NoDept` and the amount.
- **DimDates**: `List.Dates(DateDébut, NbJours, #duration(1,0,0,0))`, French month names (`"fr-FR"`), sorted by `Mois`.
- **Model**: star schema with 2 fact tables (Ventes, Inventaires) and 10 relationships, all `*:1`.
- **Measures (table Ventes, p. 39)**: `VentesBrutes$ = SUMX(Ventes, RELATED(DimProduit1[Prix unitaire]) * Ventes[Unités vendues])`,
  plus `…AnPrec` (`DATEADD(-1, YEAR)`), `…YTD` (`DATESYTD`), `NbTransactions` (`COUNTROWS`), and the
  differences in $ and % (`DIVIDE(…, …, 1)`). The original Ventes columns are hidden.
- **Page "Tableau de bord"**: year slicer, month slicer, matrix of the 10 measures by department.

**Changed during the handoff:**
- Parameter values repointed. The helper query `Exemple de fichier` had a hard-coded path; it now uses `CheminDossierVentes`.
- **The 10 measures were missing**: added them, with formats.
- `Inventaires.NoMagasin` / `NoDept` converted to whole numbers to match the dimension keys.
- French month names, sorted by month number.
- Dashboard page built, with the same theme as TP1.

**Verified against the source Excel files** (and the handout screenshot p. 42), for 2014 + janvier:

| Measure (Total row) | Value |
|---|---|
| `VentesBrutes$` | 70,797,919 |
| `VentesBrutes$AnPrec` (= January 2013) | 117,698,709 |
| `Écart$VentesBrutesAnPrec` | -46,900,790 |
| `NbTransactions` / `NbTransactionsAnPrec` | 33,128 / 32,426 |
| `VentesBrutes$YTD` with février selected | 150,802,875 (Jan + Feb) |

## Validation 2026-11-27

Format: open the files, demo the interactions (slicers, PARIS / Hors Paris, drill-down), and answer questions on
*how* each step was done. The PDF write-up was not graded. TP2 only needed to go up to page 40.

The professor asked, live:
- **"How do you stop a slicer from filtering a chart?"** Select the slicer → **Format → Edit interactions** →
  click **⊘ (None)** above the target chart. Interactions are set **per source visual**: changing the Année
  slicer does not affect what Regroupement or Typologie do.
- **"Turn the date slicer on page 1 (TP1) into a dropdown."** Select the Année slicer → Format visual →
  Visual → Slicer settings → Options → Style: **Dropdown**.
- **TP2: she only asked what level we reached.** The answer was "around page 42" and that was it: no demo or
  questions on TP2 content. Still be ready to show the measures on the Tableau de bord page.

Likely questions and short French answers are in section 6 of `guides/guide_TP1.html` and section 9 of `guides/guide_TP2.html`.

## Teammates: make the design your own

Each of us is validated individually on our own file, so **don't present this exact layout**. Change the design
yourself before your validation, and keep every visual and feature (the professor checks the features, not the look):

- **Theme** (fonts, colours, card style): View → Themes → *Customize current theme*, or edit
  `*.Report/StaticResources/RegisteredResources/ThemeGeist.json` in the `.pbip` (close Power BI first).
- **Layout**: move and resize visuals, change the page background, swap a bar chart for a column chart, rename titles.
- **Slicer styles**: list / dropdown / tiles (Format visual → Slicer settings → Style).

Doing it yourself is also good practice: you will be asked *how* things were done, and changing the formatting is
the fastest way to learn where every option lives.

## Lessons learned / gotchas

- **Paths are per machine.** Every teammate's file pointed to their own `Downloads` folder (`C:\Users\MSI\...`,
  `C:\Users\Dell\...`). Parameters make the fix a single value change.
- **"Load … Waiting for other queries…" never finishes** when one source is missing. Click *Close*, don't apply
  pending changes, fix the paths first.
- **Map visual disabled**: enable it in Options → Security, then restart. Microsoft is retiring the Bing "Map"
  visual; if it misbehaves, convert it to **Azure Maps** (same fields).
- **"Color saturation" is gone** from the Map field wells in recent versions. Use Format → Bubbles → Colors →
  **fx** → Gradient based on the field.
- **Edit interactions is easy to apply to the wrong visual.** The icons sit above every visual; check the result
  (move the slicer) before saving.
- **Relationships need matching data types** on both sides (text vs whole number broke the Inventaires keys).
- **Handout inconsistency (TP2 p. 42)**: the screenshot shows `Écart%` = 60.15 %, but the formula on p. 39 gives
  -39.8 % (= écart / année précédente). 60.15 % is sales ÷ previous-year sales. Our file follows the formula.
- **Without a year filter, YTD shows only 2015**: `DATESYTD` uses the last year in the filter context.
- **Month names follow the Windows locale** unless you pass a culture: `Date.MonthName([Date], "fr-FR")`.

### Editing the `.pbip` by hand (or with Claude Code)

- **Close Power BI first.** If it is open on the same project, its next save overwrites your edits.
  From WSL: `powershell.exe -NoProfile -Command "Get-Process PBIDesktop"`.
- **Back up** the `*.Report` / `*.SemanticModel` folders before editing.
- **TMDL files use CRLF and tabs**; keep them that way.
- **Lineage tags matter.** Turning a loaded table into a non-loaded query broke opening with
  *"Lineage tag … was not found"*: it was still referenced in `.pbi/unappliedChanges.json` and
  `diagramLayout.json`. Make that kind of change in Power BI itself (right-click query → untick *Enable load*).
- **`.pbi/unappliedChanges.json`** holds pending Power Query edits that override the TMDL queries in the editor.
  A stale one re-introduced the old paths. Delete it only after checking that it matches the model.
- **Custom theme**: `*.Report/StaticResources/RegisteredResources/ThemeGeist.json`, registered in
  `definition/report.json` (`themeCollection.customTheme` + a `CustomTheme` resource item). Textboxes set
  their font explicitly in their text runs; everything else takes it from the theme.
- From WSL, `explorer.exe <file>` opens a file with its Windows default app.

## Known gaps (not done, or done differently from the handout)

TP1:
- CATÉGORIE, MENTION, TÉLÉPHONE, COURRIEL and TYPE DE SÉJOUR are not removed.
- Capacity / emplacement / unités columns stay text.
- Latitude/longitude are merged but never expanded; the map uses the `LOCALISATION` address instead.
- The last room band shows `150` instead of `150+`.
- Only one of the two measures exists (average stars is missing).
- `fr_postal_codes` is still loaded.
- The line chart title is the default "Count of ADRESSE".

TP2:
- Tables are named `DimBannière1`, `DimProduit1`… instead of `DimBannières`, `DimProduits`.
- `NbJours` is still loaded as a table (p. 29 asks to disable loading: do it in Power BI).
- Pages after 42 (unconnected measure table, hierarchies, comparables, inventory needs, 4-4-5 fiscal calendar)
  are out of scope.
