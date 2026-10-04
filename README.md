<div align="center">

<img src="assets/banner.svg" alt="Auditing a Public Road Accident Dataset from Bangladesh" width="100%">

<a href="https://github.com/nishatkh/bangladesh-road-accident-audit">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=500&size=20&pause=1200&color=2F81F7&center=true&vCenter=true&width=760&lines=Checking+the+data+before+fitting+the+model;47%2C676+accidents.+31+variables.+18+tests.;Classifier+AUC+of+.504%3A+indistinguishable+from+chance;The+file+is+unsuitable+for+risk-factor+inference" alt="Typing animation summarising the findings">
</a>

<br>

![Records](https://img.shields.io/badge/records-47%2C676-2f81f7?style=for-the-badge)
![Variables](https://img.shields.io/badge/variables-31-2f81f7?style=for-the-badge)
![AUC](https://img.shields.io/badge/test%20AUC-.504-f0883e?style=for-the-badge)
![Max effect](https://img.shields.io/badge/max%20Cram%C3%A9r's%20V-.026-f0883e?style=for-the-badge)
![Python](https://img.shields.io/badge/python-pandas%20%7C%20scipy%20%7C%20scikit--learn-3776ab?style=for-the-badge)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Key Findings](#key-findings)
- [Research Questions](#research-questions)
- [Analysis Pipeline](#analysis-pipeline)
- [Data](#data)
- [Method](#method)
- [Results](#results)
- [Data Quality Audit](#data-quality-audit)
- [Audit Checklist for Crash Data](#audit-checklist-for-crash-data)
- [Recommended Fields for Future Collection](#recommended-fields-for-future-collection)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Repository Structure](#repository-structure)
- [Citation](#citation)
- [Author](#author)

---

## Overview

Road crash records decide where enforcement and engineering money goes, so their quality has to be checked before any model is fitted. This project examines a public Kaggle file of Bangladesh road accidents using exploratory analysis, chi-square tests of association, and supervised classification.

The study began as an attempt to find risk factors for fatal accidents and ended as an audit of the data. Every road, light, weather, and calendar variable turned out to be unrelated to fatality, and a tuned classifier performed at chance. Internal consistency checks suggest the file contains synthetic or heavily edited records, so its null results say nothing about real Bangladeshi roads.

> A classifier with an impressive score on these data would have looked like a finding and would have been wrong. Checking a dataset before modeling it is the more valuable result.

<div align="center">
  <img src="assets/pipeline.svg" alt="Analysis pipeline" width="100%">
</div>

---

## Key Findings

| Area | Result |
| --- | --- |
| Sample | 47,676 accidents after removing 4 exact duplicates (47,680 raw rows, 31 variables) |
| Outcome | 66.8% of accidents recorded at least one death |
| Volume trend | Recorded accidents rose 98.8% from 2007 to 2021 (about 2,500 to 4,900), adding 123.6 per year |
| Fatal share | Stayed between 65.8% and 67.5% across all 15 years |
| Association tests | 0 of 18 variables significant at .05; largest Cramér's V = .026 (hour of day) |
| Classifier | Tuned model reached AUC = .504 on 9,536 held-out accidents |
| Verdict | The file is unsuitable for risk-factor inference |

<div align="center">
  <img src="assets/trend.svg" alt="Volume rose while fatal share stayed flat" width="85%">
</div>

---

## Research Questions

1. How do the volume and fatality of recorded accidents vary across years and calendar periods?
2. Is fatality associated with any recorded road, light, weather, or calendar variable?
3. Can a supervised classifier predict whether an accident is fatal from those variables?
4. Do internal consistency checks support using the file for the first three questions?

The answers to questions 2 and 3 were negative, and the answer to question 4 pointed to serious problems with the file.

---

## Analysis Pipeline

| Stage | What was done | Output |
| --- | --- | --- |
| 1. Load and clean | Normalised column names, trimmed whitespace, dropped exact duplicates | 47,676 accidents, no missing values |
| 2. Descriptive analysis | Counts by death class, year, month, hour, light, weather, road type | Figures 1 to 4 in the paper |
| 3. Association testing | Pearson chi-square plus Cramér's V for 18 categorical variables | Table A1 |
| 4. Classification | Logistic regression, random forest, histogram gradient boosting; randomized search with 15 draws | Test AUC = .504 |
| 5. Data quality audit | Cross-checked dates, categories, narratives, and labels | Table 4 indicators |

---

## Data

| Property | Value |
| --- | --- |
| Source | Kaggle: *Bangladesh Road Accident Dataset 2007-2024* by mdnahidurrahmankh |
| File loaded | `Modified_Accident_Dataset final.xlsx` |
| Rows | 47,680 raw, 47,676 after duplicate removal |
| Columns | 31 |
| Actual year range | 2007 to 2021 (the title states 2024) |

Variables by topic:

| Topic | Variables |
| --- | --- |
| Identification | `accident_serial_no`, `id` |
| Time | `date`, `time`, `year`, `month` |
| Place | `location`, `longitude_and_latitude`, `place_characteristic` |
| People and vehicles | `driver_age`, `vehicle_info`, `vehicle_count`, `victim_category` |
| Outcome | `death_count`, `death_info`, `injury_count`, `injury_info`, `accident_intensity` |
| Narrative | `accident_narrative` |
| Road | `road_surface_type`, `road_nature`, `road_classification`, `type_of_road_connection_point`, `road_surface_condition`, `junction`, `traffic_control`, `trafic_condition` |
| Light and weather | `light_condition`, `lighting`, `weather_x`, `weather_y` |

The `_x` and `_y` suffixes are what pandas adds when two tables are merged, which suggests the file combines two sources.

---

## Method

**Outcome definition.** An accident is fatal when `death_count > 0`. Severity classes were 0, 1, 2 to 4, and 5 or more deaths.

**Statistical analysis.** Each categorical variable with 2 to 30 categories was cross-tabulated with the fatal indicator and tested with a Pearson chi-square test at alpha = .05. Because large samples detect tiny differences, effect size was measured with Cramér's V, which equals Cohen's w for two-column tables (.10 small, .30 medium, .50 large).

**Predictive modeling.**

| Setting | Value |
| --- | --- |
| Features | Calendar, road, light, and weather variables; outcome-related columns excluded to prevent leakage |
| Split | 80/20, stratified on outcome; 9,536 test accidents |
| Models compared | Logistic regression, random forest, histogram-based gradient boosting |
| Validation | 5-fold stratified cross-validation on AUC and macro F1 |
| Class handling | Balanced class weights |
| Tuning | Randomized search, 15 parameter draws |
| Seed | 42 |

**Software.** pandas, SciPy, scikit-learn, matplotlib, seaborn.

---

## Results

### Severity distribution

Accidents with 0, 1, and 2 to 4 deaths each make up about one third of the file, and no accident has five or more deaths. Real severity counts normally fall as deaths per crash rise.

<div align="center">
  <img src="assets/death-classes.svg" alt="Accidents by number of deaths" width="85%">
</div>

### Calendar patterns

- September had the most accidents (about 4,480) and February the fewest (about 3,550).
- The fatal proportion peaked in May at 68.5%; all other months sat near 66% to 68%.
- Nearly every time value converted to hour 23, so hour-of-day and day-part results carry no information.
- The date column mixes ISO timestamps and day-month-year strings and failed to parse, so weekday and season were not derived.

### Road, light, and weather conditions

| Condition | Fatal proportion range |
| --- | --- |
| Light (dusk, day, night) | 66.5% to 67.3% |
| Weather (clear, sunny, rainy, foggy) | 66.2% to 67.0% |

Category sizes were also suspiciously even: 15,735 to 15,989 accidents per light category and 11,821 to 12,039 per weather category.

### Association with fatality

None of the 18 variables was significant. The ten strongest:

| Variable | Categories | Cramér's V | p |
| --- | --- | --- | --- |
| `hour` | 24 | .0257 | .117 |
| `trafic_condition` | 25 | .0201 | .741 |
| `month` | 12 | .0159 | .364 |
| `junction` | 9 | .0145 | .268 |
| `type_of_road_connection_point` | 7 | .0134 | .198 |
| `road_classification` | 4 | .0126 | .056 |
| `lighting` | 4 | .0117 | .087 |
| `year` | 15 | .0114 | .961 |
| `traffic_control` | 6 | .0110 | .330 |
| `weather_y` | 4 | .0105 | .157 |

All effects fall below .10, the lower bound of a small effect. Even `accident_intensity`, which should track deaths, had V = .003 (p = .907). The hour result carries an extra caveat because that column collapsed into one value.

### Classification

| Class | Precision | Recall | F1 | Support |
| --- | --- | --- | --- | --- |
| Not fatal | .33 | .53 | .41 | 3,164 |
| Fatal | .67 | .47 | .55 | 6,372 |
| Accuracy | | | .49 | 9,536 |
| Macro average | .50 | .50 | .48 | 9,536 |
| Weighted average | .56 | .49 | .50 | 9,536 |

AUC = .504. Precision simply matches the class base rates, and accuracy sits below the .668 a rule predicting "fatal" for everything would reach. This is not a weak model: tuning cannot recover signal the inputs do not hold.

---

## Data Quality Audit

Flat results could mean fatality is truly independent of conditions, but for light and road conditions that seemed unlikely, so the file itself was examined.

| Indicator | Evidence |
| --- | --- |
| Even outcome classes | 0, 1, and 2 to 4 deaths held 33.2%, 33.5%, and 33.3%; the 5+ class was empty |
| Even category sizes | Near-identical counts across light and weather categories |
| Conflicting dates | First row: date 2022-06-05, year 2007, month September |
| Collapsed hours | Nearly every time converted to hour 23 |
| Repeated descriptions | First five rows share driver age 26 and the same 50-year-old death description, including rows with zero deaths |
| Fields and narrative disagree | Row 1 gives 06.35 am and a car-pedestrian collision; the narrative describes about 11:45 AM and a loaded cargo truck |
| Unrelated outcome variable | `accident_intensity` unrelated to fatality (V = .003) |
| Incomplete coverage | Data ends in 2021, not 2024 |
| Inconsistent labels | "Two-Lane" and "Two-lane" stored as separate values |
| Modified source | File name contains the word "Modified" |

No single indicator is decisive, but together they suggest many values were assigned independently of one another. Values drawn at random from a short list produce near-equal counts, whereas real traffic piles up in some conditions and thins out in others. How the file was produced could not be verified.

---

## Audit Checklist for Crash Data

These steps are cheap and would have flagged the problems at the start.

- [ ] Plot the outcome distribution first; real severity counts fall as deaths per crash rise
- [ ] Compare date-like columns with each other; year, month, and full date should agree
- [ ] Count rows per category; near-identical sizes across unrelated categories suggest random assignment
- [ ] Test each variable against the outcome with an effect size as well as a p value
- [ ] Read a sample of full rows, narrative included, to see whether fields agree
- [ ] Compare yearly totals with an official source before drawing conclusions

---

## Recommended Fields for Future Collection

A crash file that could support safety decisions would record:

| Group | Fields |
| --- | --- |
| Time | Verified date and 24-hour time |
| Place | District and division, checked coordinates |
| Participants | Vehicle types and road user types |
| Road | Road class and speed limit |
| Outcome | Consistent injury scale; outcome after a fixed period such as 30 days |
| Provenance | Reporting agency |

With these fields, location risk tiers and time-window risk classes become possible. A planned district risk classification in this study produced no output because the file has no district or division variable, only free-text locations.

---

## Limitations

- The study rests on one file whose origin could not be verified, so every finding describes the file, not Bangladeshi roads.
- The outcome is a binary fatal indicator; injury severity and number of deaths were not modeled.
- Free-text columns (narrative, location, coordinates) were not mined.
- Hour and day-part results are unreliable, and weekday and season results do not exist.
- The classifier had a modest tuning budget of 15 draws, though an AUC at chance makes a larger budget unlikely to change the conclusion.
- Classification results come from an earlier notebook version, and per-family cross-validation scores were not retained, so the winning model family cannot be named.
- Association tests show relationships, not cause, even in a clean file.

---

## Future Work

1. Obtain a verified crash dataset, for example from police records, the Bangladesh Road Transport Authority, or university research units, and repeat the analysis.
2. Compare yearly totals in this file against official statistics to measure how far the records diverge.
3. On clean data, run the district-level risk classification, time-window classification, and severity models described in the paper.

---

## Repository Structure

Adjust to match your actual layout.

```text
.
|-- README.md
|-- assets/                  animated SVGs used in this README
|   |-- banner.svg
|   |-- pipeline.svg
|   |-- trend.svg
|   `-- death-classes.svg
|-- paper/
|   `-- bangladesh_road_accident_article.pdf
|-- notebooks/               analysis notebook
`-- data/                    place the Kaggle file here (not redistributed)
```

---

## Citation

```bibtex
@misc{nishat_road_accident_audit,
  author = {Nishat, MD. Nahidur Rahman Khan},
  title  = {Auditing a Public Road Accident Dataset from Bangladesh:
            Exploratory Analysis, Association Testing, and Fatality Classification},
  note   = {Department of Computing and Information System,
            Daffodil International University, Dhaka-1216, Bangladesh},
  year   = {2026}
}
```

Dataset: mdnahidurrahmankh. *Bangladesh Road Accident Dataset 2007-2024* [Data set]. Kaggle.

---

## Author

**MD. Nahidur Rahman Khan Nishat**
Department of Computing and Information System, Daffodil International University, Dhaka-1216, Bangladesh
GitHub: [@nishatkh](https://github.com/nishatkh)

---

<div align="center">
  <sub>Findings describe the audited file only and should not be read as evidence about road risk in Bangladesh.</sub>
</div>
