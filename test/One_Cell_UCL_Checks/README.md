# One-cell checks: PnET cohorts as other extensions see them

Small scenarios, all text, on a one-cell landscape (30 m) with one species, `querrubr` (red oak; longevity 100, fire tolerance 2). Each checks one way that another extension reads or changes PnET's cohorts through the Universal Cohort Library, or one PnET setting. The PnET parameters are the `querrubr` rows of Tool-Docker-Apptainer's PnET test scenario (`tests/TestPnET_AllExtension`); the climate file is the one from `deploy/examples/OneCellSim_Harvest` in Extension-Output-Biomass-PnET.

Run a case from this folder, e.g. in the LANDIS-II Docker image:

```sh
docker run --rm -v "$PWD":/sim -w /sim --entrypoint dotnet <image> \
  /opt/landis-ii/Core-Model-v8-LINUX/build/Release/Landis.Console.dll scenario-partial.txt
```

Every case writes Output Biomass Community (`community-input-file-<year>.csv`) and PnET's site output (`Output/Site1/Site.csv`).

## Cases

| Scenario | What it checks | Expected |
|---|---|---|
| `scenario-partial.txt` | Biomass Harvest takes 50 % of the one cohort (age 70) at year 10 | the harvest event log shows 1 site and 1 cohort partially harvested; the cohort's year-10 biomass is well below `scenario-nocut.txt`'s |
| `scenario-nocut.txt` | the same without Biomass Harvest | baseline |
| `scenario-none.txt` | a prescription whose age range (1-5) holds no cohort | nothing harvested, and litter as in `scenario-nocut.txt` |
| `scenario-fire.txt` | one severity-1 fire at year 10 (weather in `weather.csv`) | the cohort, at 70 % of its longevity, survives: in the damage table the 85 % row kills it only at severity - tolerance >= 0 |
| `scenario-merge.txt` | initial cohorts aged 5 and 8; `CohortBinSize` not set (it defaults to `Timestep`, 10) | both are within one bin, so they are merged |
| `scenario-merge-apart.txt` | initial cohorts aged 5 and 15; `CohortBinSize` not set | two cohorts, 5 and 15 |
| `scenario-merge-bin20.txt` | initial cohorts aged 5 and 15; `CohortBinSize 20` | both within one bin, so they are merged |
| `scenario-par.txt` | Output-PnET's sub-canopy PAR map (keyword `SubCanopyPAR`, as in the Output-PnET v1.0 guide) | the map holds July's sub-canopy PAR (the July value in `Output/Site1/Site.csv`) |
| `scenario-par-new.txt` | as `scenario-par.txt` with the keyword `SubcanopyPAR` | as `scenario-par.txt` |

## Results, 2026-11-04

Tool-Docker-Apptainer's UCLv2 image at `ghcr.io/landis-ii-foundation/landis-ii-v8-uclv2-release@sha256:89fb537774dcaeb8ca97f9dbbc6e3189c384332e36fcbfdb302d07799c065cd5` (PnET-Succession 6.1.2, Library-PnET-Cohort 2.4.1, Universal Cohort Library 2.0.1), alone and with these pull requests built and mounted over the image's DLLs: [Library-Universal-Cohort #7](https://github.com/LANDIS-II-Foundation/Library-Universal-Cohort/pull/7), [Library-PnET-Cohort #17](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/pull/17), [Extension-PnET-Succession #37](https://github.com/LANDIS-II-Foundation/Extension-PnET-Succession/pull/37) and [Extension-Output-Biomass-PnET #24](https://github.com/LANDIS-II-Foundation/Extension-Output-Biomass-PnET/pull/24) ("PRs"). "PRs + `ICohort.Data`" adds an explicit implementation of the Universal Cohort Library's `ICohort.Data` to PnET's `Cohort`, as suggested on [Library-PnET-Cohort #18](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/issues/18).

| Scenario | 2.4.1 | PRs | PRs + `ICohort.Data` |
|---|---|---|---|
| `scenario-partial.txt` | 0 sites, 0 cohorts harvested; year-10 biomass 125 | same | 1 site, 1 cohort harvested (0.036 Mg); year-10 biomass 34 |
| `scenario-nocut.txt` | year-10 biomass 125; litter in Dec 2009 33.73 g DW m-2 | same | same |
| `scenario-none.txt` | nothing harvested; litter 31.73 | same | same |
| `scenario-fire.txt` | fuel type 0 (Dynamic Fuels sees no cohorts), severity 3; cohort killed | fuel type 8 (D1), severity 1; cohort killed | fuel type 8, severity 1; cohort survives (age 80, biomass 125) |
| `scenario-merge.txt` | stops: "The given key '5' was not present in the dictionary" (`SiteCohorts.cs` line 490) | two cohorts (5, 8) | |
| `scenario-merge-apart.txt` | one cohort (5) | two cohorts (5, 15) | |
| `scenario-merge-bin20.txt` | stops, as `scenario-merge.txt` | one cohort (5) | |
| `scenario-par.txt` / `-new.txt` | map 851 (July 2009 in `Site.csv`: 851.9) | map 256 (December 2009: 256) | |

Biomass values are Output Biomass Community's `CohortBiomass` for the one cohort.

Issues: partial harvest and fire, [Library-PnET-Cohort #18](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/issues/18); the merge cases, [#19](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/issues/19) (and `CohortBinSize` in [#17](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/pull/17)); dead pools, [#20](https://github.com/LANDIS-II-Foundation/Library-PnET-Cohort/issues/20); sub-canopy PAR, [Extension-Output-Biomass-PnET #24](https://github.com/LANDIS-II-Foundation/Extension-Output-Biomass-PnET/pull/24).
