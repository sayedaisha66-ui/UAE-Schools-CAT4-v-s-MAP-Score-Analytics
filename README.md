
# UAE Schools CAT4 & MAP Analytics (Power BI, Power Query, Python)

An end-to-end data analytics project built on **simulated** UAE school data. It extracts data from four different file types, cleans it, models it as a star schema, and presents the results in a Power BI dashboard that compares pupils' **ability (CAT4)** with their **attainment and growth (MAP Growth)**.

> **All data is synthetic.** No real pupils, teachers or schools are involved. The MAP benchmarks are illustrative, not official NWEA norms. The findings show the method, not real UAE patterns.

## The question

Are pupils achieving what their ability predicts, and where is growth strongest or weakest across 8 schools in 5 emirates?

- **CAT4** (GL Assessment) measures reasoning ability once a year: Verbal, Quantitative, Non-Verbal and Spatial.
- **MAP Growth** (NWEA) measures attainment in Reading, Language Usage and Maths three times a year (Fall, Winter, Spring).

## At a glance

| | |
|---|---|
| Schools | 8 (Dubai, Abu Dhabi, Sharjah, Ajman, Ras Al Khaimah) |
| Academic years | 2024-25 and 2025-26 |
| Pupils | 2,837 (Years 3 to 10) |
| CAT4 records | 5,054 |
| MAP test records | 43,887 |
| Enrolment records | 5,159 |
| Source formats | Excel, CSV, HTML, JSON |

## Data sources (extraction)

| Source | Format | What it taught |
|---|---|---|
| Student information export (Students, Enrolment, Teachers, Classes, Schools) | Excel, 5 sheets | Header not on row 1, mixed data types |
| CAT4 2024-25 and 2025-26 | CSV | The second year uses a semicolon delimiter, comment lines, decimal commas and different column names |
| MAP Growth 2024-25 and 2025-26 | CSV | The second year has a different schema (numeric student ID, school names instead of IDs) |
| School directory, inspection ratings, MAP benchmarks, stanine bands | HTML tables | Web-table extraction with Power Query |
| Attendance | Nested JSON | Flattening student > year > term records |

## Cleaning and quality (Power Query)

Every issue was logged with a count and a fix. Examples:

- **Keys:** pupil IDs standardised across all sources (case, spaces, hyphens, and an ID rebuilt from a numeric field).
- **Duplicates:** exact duplicates removed; retests resolved by keeping the latest sitting.
- **Invalid values:** scores outside valid ranges (SAS 60 to 140, RIT 100 to 300), text such as `ABS` and `187*`, and impossible test durations were set to null, with rows kept.
- **Inconsistent labels:** gender, nationality, EAL, SEN, term and subject spellings standardised.
- **Dates and numbers:** mixed formats and decimal commas converted using the correct locale.
- **Orphans:** records with no matching pupil were counted first with a Left Anti join, then removed (40 CAT4, 40 MAP, 15 Enrolment).
- **Personal data:** names and phone numbers removed from the model.

## Data model

Star schema with:

- **Facts:** `Fact_CAT4`, `Fact_MAP`, attendance
- **Dimensions:** Students, Schools, Classes, Teachers (through a `Class_Teacher` bridge), `Dim_Year`
- **Reference tables:** MAP benchmarks, SAS-to-stanine bands, CAT4 profile rules

All relationships are one-to-many with single-direction filters. Reference tables with multi-column keys are used through merges, not relationships.

An analysis table (`Analysis_Pupil`) puts each pupil's CAT4 and MAP scores side by side, with Fall-to-Spring growth and the difference from the grade benchmark.

## Dashboard

1. **Front page:** one business question per page, each a navigation button
2. **CAT4 Analysis:** ability profile by school and battery
3. **MAP Analysis:** attainment and growth by school, term, EAL and SEN status
4. **CAT4 vs MAP:** how ability relates to attainment and growth
5. **Key insights**

Screenshots are in `docs/images/`.

## Key findings (synthetic data)

- Average CAT4 Mean SAS is 98.8; Gulf Horizon and Emirates Pearl are highest and Verbal is the weakest battery (97.5).
- Pupils gain about 6.3 RIT points in Maths and 5.7 in Reading and Language Usage between Fall and Spring.
- Ability strongly predicts attainment: high-ability pupils finish about 13 points above the grade benchmark and low-ability pupils about 15 below, with similar growth in every band, so the gap does not close.
- One school (Al Wasl Heritage) shows far higher Reading and Language growth than the rest. This is the kind of result to check with the data owner before acting on it.

## Validation

KPIs were recalculated independently in Python (pandas) and compared with the Power BI cards. An earlier cross-check in MySQL gave a different Verbal average (96.65 against 97.48): the CSV import wizard had turned blank scores into zeros. Fixing how blanks were read resolved it, and it is a good reminder to check what a tool does with missing values.

## Repository structure

```
data/raw/          synthetic source files (Excel, CSV, HTML, JSON)
powerbi/           the .pbix report
python/            validation notebook
docs/              data quality log, data dictionary, project brief (PDF)
docs/images/       dashboard and data model screenshots
```



## Tools and skills

Power Query (M), Power BI, DAX, Python (pandas, Google Colab), MySQL. Skills shown: extraction from several formats, data profiling and cleaning, key standardisation, deduplication, orphan detection, star-schema modelling, DAX measures, derived columns, KPI design, cross-tool validation and documentation.


## Disclaimer

CAT4 is a product of GL Assessment and MAP Growth is a product of NWEA. This project is an independent learning exercise and is not affiliated with either organisation.

