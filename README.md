# Road Accident Modeling: Severity and Frequency (France, 2019)

Statistical study of road accidents in France, based on the **ONISR 2019** open data (BAAC file) and **INSEE** population data. Team project (Équipe 15) for the ENSIMAG *Data Science* course, 2025-2026.

- **Team:** Rayane Beggar, Ayoub Bkhach, Ayoub Marsouk, Yassir Sabri
- **Stack:** R (`dplyr`, `tidyr`, `ggplot2`, `sf`, `viridis`), R Markdown, LaTeX
- **Report:** [`RAPPORT_PAM.pdf`](RAPPORT_PAM.pdf) (in French) · **Slides:** [`présentation/présentation.pdf`](pr%C3%A9sentation/pr%C3%A9sentation.pdf)

![Accidents per département](Code%20R/accidentR_files/figure-latex/carte_accidents_departements.png)

## Questions

1. Which factors are associated with the **severity** of an accident?
2. Are there recurring **types of accidents**?
3. How does accident **frequency** vary between départements once exposure is taken into account?

## Approach

### 1. Severity score and linear models
- Built a **severity score** from the number of people killed, hospitalised and slightly injured:
  `S = (9 × killed + 3 × hospitalised + 1 × injured) / 13 × 100`
  The weights follow the order of gravity; this is a modelling choice, not a medical measure.
- Compared three linear-model specifications (`lm`):
  1. **full model** with every available variable (lighting, weather, location, road infrastructure, number of vehicles by type);
  2. **stepwise selection** (`step`, both directions) to reduce the complexity;
  3. **final model** with grouped categories (lighting, weather, collision type, speed-limit classes, accident situation), easier to interpret and used as the basis of the analysis.

### 2. Accident typology with k-means
- Categorical variables one-hot encoded, all variables standardised, rows with missing values removed.
- Number of clusters chosen with the **elbow method** on the within-cluster sum of squares; **k = 3** retained.
- `kmeans` with `nstart = 50` and a fixed seed; clusters shown on the first two axes of a **PCA**, with the projected cluster centres.
- Result: general conditions are very homogeneous across the three clusters (daylight, normal weather). The clusters differ mainly by vehicle composition (share of motorcycles, bicycles and trucks), so they are read as descriptive groupings rather than strict categories.

### 3. Frequency analysis per département
- **Frequency = accidents / estimated drivers**, expressed per 1,000 drivers.
- Drivers estimated from INSEE data, joined on the département code, with a constant licence-holder coefficient of **0.65** (stated assumption).
- Départements ranked by frequency: Paris (75) comes first. Results mapped with `sf` and a département GeoJSON.
- **Quasi-Poisson** count models (`glm`) fitted for a high-frequency département (75, Paris) and a low-frequency one (90, Territoire de Belfort) to compare the effect of night-time share, adverse weather, main-road share and speed limits.
- Result: the same factor can be decisive in one département and negligible in another, so the weight of risk factors depends on the local context.

## Limitations
- Severity weights are intuitive; victim age, injury type and length of hospital stay are not in the data.
- Driver counts are an estimate; real use of the road network is not measured.
- A single year (2019) is used, so there is no temporal analysis.

## Repository content

| Path | Content |
|---|---|
| `Code R/accidentR.Rmd` | Full analysis: score, linear models, k-means and PCA, frequency table, map, quasi-Poisson models |
| `Code R/ONISR-2019.csv` | ONISR 2019 accident data |
| `Code R/donnees_departements.csv` | INSEE département data (drivers estimate) |
| `Code R/table_frequence.csv` | Output: accidents and frequency per département |
| `Code R/departements.geojson` | Département boundaries used for the map |
| `rapport.tex`, `RAPPORT_PAM.pdf` | Report (LaTeX source and compiled PDF) |
| `présentation/` | Presentation slides |
| `doc utile/` | Project brief, data description and bibliography |
| `planning.pdf`, `planning.ods` | Project planning |

## Reproduce

1. Install the R packages: `install.packages(c("dplyr", "tidyr", "ggplot2", "sf", "viridis", "rmarkdown"))`.
2. Open `Code R/accidentR.Rmd` in RStudio and set the working directory to `Code R/`, where the CSV files are.
3. Knit the document (PDF or HTML). The map chunk downloads the département GeoJSON if it is missing, so it needs an internet connection the first time.
4. To rebuild the report: `pdflatex rapport.tex` (with BibTeX).

## Author
[Ayoub Bkhach](https://github.com/Bkhach-Ayoub) · [LinkedIn](https://www.linkedin.com/in/bkhach-ayoub/) · [Portfolio](https://bkhach-ayoub.github.io)
