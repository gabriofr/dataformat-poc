# SKDE export – recommended JSON structure

Sample files implementing (and in two places amending) the architecture from the ChatGPT
discussion, grounded in the current exports `autisme_barn.json` and `psa_rb1.json`.

## Files

```
shared/
  dimensions.json          Shared dimension registry (year, kjønn, alder, measure)
  category-sets.json       Category registry with stable IDs, levels and parentId hierarchy
  category-mappings.json   Explicit, curated mappings between category sets
indicators/
  autisme_barn.json        Per-indicator bundle: metadata + charts + tidy datasets
  psa_rb1.json             Same structure for the PSA export
state/
  example-selection-state.json   Global UI state, kept outside definitions and data
```

## What changed vs. the current exports

| Current export | Problem | New structure |
|---|---|---|
| `psa_rb1`: columns `tot2021`, `tot2022`, `tot2023` + per-chart `annualVar`/`annualVarLabels` | Year dimension encoded in column *names*; labels duplicated per chart | Tidy observations with a `time` field; `year` defined once in `shared/dimensions.json` |
| `psa_rb1`: `datasett_1` (area) and `datasett_2` (borhf) as separate datasets | Same data duplicated per geographic level, no machine-readable link between them | One dataset with observations at both levels of the shared `geo` hierarchy; charts pick a presentation level |
| `autisme_barn`: `data[view][year][level][area][variable][measure]` nesting keyed by display-ish strings (`Helse_SørØst`) | Names used as identifiers; deep nesting hard to filter/validate | Flat observations referencing stable category IDs (`helse_sorost`) |
| `autisme_barn` `views[].variables` | Categories defined inside each view, no identity across indicators | Shared category sets (`behandler`, `sektor`) referenced by ID |
| Mixed locale keys: `no`, `nb`, `label_no`/`label_en` | Inconsistent | `{ "nb": …, "en": … }` everywhere (BCP 47; `psa_rb1` already used `nb`) |
| Presentation config (`format`, `jenks`, colors) mixed with data | — | Stays in chart definitions — charts describe *how to visualize*, never own state or labels |

## Where this deviates from the ChatGPT thread (deliberately)

1. **Selection storage (its §3, "Approach B"):** the thread recommends storing
   expanded *leaf* category IDs. SKDE rates are age/sex-adjusted, so a parent value
   (RHF, national) is **not** derivable from its children — observations must exist at
   every level, and leaves are *not* the atomic unit of the data. The samples therefore
   store selected IDs at any level (its "Approach A") and resolve parent/child display
   (including partial selection) through the shared hierarchy. Same reason `meanOfYears`
   period rows are pipeline-computed observations, not client-side aggregates.
2. **Observations separate dimensions from measures** instead of one
   `values: Record<string, …>` bag: dimension fields (`geo`, `year`, `kjonn`, `category`)
   are top-level filter keys; `values` holds only the named, precomputed measures
   declared in the bundle's `measures` section. This keeps observations schema-validatable
   and makes the `measure` dimension a simple lookup, never a client-side calculation.

Everything else follows the thread: shared registries with stable IDs, charts as views
referencing dimensions/category sets, tidy data, explicit (never name-based) category
mappings, and global state where *not using* a selection never *deletes* it.
