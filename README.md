# CardioX Hypertension Trial — Clinical Data Analysis Project

End-to-end Clinical Data Management project simulating the workflow of a Clinical Data Analyst at a pharmaceutical company / CRO: data cleaning, database design, SQL analysis, and Power BI dashboarding for a simulated Phase III hypertension trial.

## Project overview

This project replicates the operational workflow of a Clinical Data Analyst receiving raw, messy trial data from multiple hospital sites and taking it through to management-ready insights. It follows the real industry pipeline: **Excel (clean & validate) → MySQL (structure & query) → Power BI (visualize & report)**.

## Business problem

A sponsor has completed a Phase III trial testing two treatments (Drug A, Drug B) against a Placebo for hypertension across 50 sites. Raw patient data submitted by sites contains duplicates, missing values, inconsistent date formats, and physiologically invalid entries. As the Clinical Data Analyst, the task is to clean and validate this data, structure it in a relational database, and produce dashboards that let management assess enrollment, safety, and treatment effectiveness.

## Study background

- **Disease:** Hypertension (high blood pressure)
- **Trial phase:** Phase III (large-scale, multi-site efficacy and safety trial)
- **Study design:** Randomized, 3-arm (Drug A / Drug B / Placebo)
- **Patients:** 1,000
- **Sites:** 50 (grouped into 4 regions for this project: North, South, East, West)

**Note on dataset authenticity:** True patient-level clinical trial data is never publicly released due to HIPAA/GDPR and sponsor confidentiality requirements. This project uses a realistic, publicly available **synthetic** dataset (Kaggle: "Hypothetical Phase III Clinical Trial Dataset") that mirrors the structure, fields, and data quality issues of a real trial, so the cleaning/analysis workflow is representative of real CDA work. This is disclosed transparently rather than presented as real patient data.

## Dataset description

Source: Kaggle — Hypothetical Phase III Clinical Trial Dataset
Raw file: `data/CardioX_Raw_Data.csv` (1,000 rows, as submitted by sites, uncleaned)

## Data dictionary

| Column | Type | Description |
|---|---|---|
| Subject_ID | INT (PK) | Unique patient identifier |
| Site_ID | INT | Site where patient was enrolled (1–50) |
| Age | INT | Patient age in years |
| Gender | VARCHAR | Male / Female |
| Enrollment_Date | DATE | Date patient enrolled in the trial |
| Treatment_Group | VARCHAR | Drug A / Drug B / Placebo (randomized arm) |
| Adverse_Events | INT | Count of adverse events reported by the patient |
| Dropout | INT (0/1) | 1 = patient discontinued before study completion |
| Systolic_BP | INT | Systolic blood pressure (mmHg) |
| Diastolic_BP | INT | Diastolic blood pressure (mmHg) |
| Cholesterol_Level | INT | Cholesterol level (mg/dL) |

## Data quality issues found (Excel cleaning phase)

| Issue | Finding |
|---|---|
| Duplicate Patient IDs | None found (verified via COUNTIF) |
| Inconsistent date formats | Mixed MDY/DMY across sites; standardized to DMY, later converted to ISO (YYYY-MM-DD) for SQL import |
| Implausible ages | 12 patients below protocol minimum age of 18 — flagged for query |
| Physiologically invalid BP | 9 rows where Diastolic BP > Systolic BP — flagged for query |
| Missing Treatment Group | None found |
| Gender inconsistency | None found — clean Male/Female values only |

## Excel work

- Duplicate detection (COUNTIF)
- Date format standardization (Text to Columns, TEXT formula)
- Range/logic validation (Age, Systolic/Diastolic BP) via MIN/MAX and helper-column flags
- Conditional formatting to visually flag invalid rows
- XLOOKUP / INDEX-MATCH for patient record lookup
- COUNTIFS / AVERAGEIFS for group-level metrics (e.g., adverse event counts and average BP by treatment group)
- PivotTable + PivotChart for cross-validation of treatment group summaries
- Exported cleaned, formula-free data to CSV for database import

File: `excel/Cleaned_Clinical_Data.xlsx`

## SQL database design

Database: `cardiox_trial` (MySQL)

**Tables:**
- `Patients` — one row per patient (11 columns, see data dictionary)
- `Sites` — Site_ID to Region mapping (Region labels synthetically assigned for demonstration, since source data had no site metadata)

**Relationships:** `Patients.Site_ID` → `Sites.Site_ID` (many-to-one)

Script: `sql/cardiox_sql_queries.sql` (contains both table creation and analysis queries)

## SQL analysis — key queries

1. Filter patients by treatment group and adverse event status (WHERE)
2. Patient count and average Systolic BP by treatment group (GROUP BY)
3. Patient-to-site region lookup (INNER JOIN)
4. Dropout count and rate by region (JOIN + GROUP BY)
5. Adverse event rate by age group and treatment group (CASE + GROUP BY)

## Power BI dashboard

**Page 1 — Patient Enrollment Dashboard**
- Card visuals: Total Patients, Dropout Rate %, Total Dropouts, Average Systolic BP
- Line chart: Enrollment over time
- Donut chart: Gender split
- Clustered column chart: Patients by Treatment Group
- Table: Region-level enrollment, dropout, and adverse event summary
- Slicer: Treatment Group filter (interactive, cross-filters all visuals)

Note: Power BI was connected to data exported from MySQL (rather than a live MySQL connector) due to a known MySQL/Power BI driver compatibility issue (`Internal connection fatal error`) encountered on this machine. Data lineage is preserved: Excel → MySQL → CSV export → Power BI.

File: `powerbi/CardioX_Enrollment_Dashboard.pbix`

## Business insights

1. **195 patients (~63%) in the Drug A group** reported at least one adverse event, the highest rate among all three arms.
2. **Average Systolic BP** was similar across all groups (Drug A: 120.1, Drug B: 119.4, Placebo: 119.2) — Drug A showed no clear BP reduction advantage over Placebo in this dataset.
3. **East Region had the highest dropout rate (18.9%)**, nearly 6 percentage points higher than the best-performing North Region (13.1%) — a candidate for site-level retention review.
4. **Drug B showed the lowest average adverse event rate across every age group** (Under 40, 40–60, Over 60), while Drug A consistently showed the highest — suggesting a more favorable safety profile for Drug B relative to Drug A in this dataset.
5. **The 40–60 age group showed the highest adverse event rates** across all three treatments compared to Under-40 and Over-60 groups.

## Recommendations

- Investigate East Region's site-level retention practices given its elevated dropout rate.
- Flag Drug A's higher adverse event rate for further safety review, particularly in the 40–60 age bracket.
- Re-verify the 12 sub-18 patient records and 9 Diastolic>Systolic records with source sites before database lock.

## Challenges encountered

- Mixed MDY/DMY date formats across sites required standardization before database import.
- A MySQL/Power BI live-connector driver bug required switching to a CSV-export workaround.
- Synthetic dataset lacked true site/region metadata, requiring a documented placeholder assignment for demonstration purposes.

## Future improvements

- Add a Treatment Effectiveness and Adverse Event dashboard page.
- Add DAX measures for date intelligence (e.g., days-to-dropout).
- Expand the schema with Visits/Lab Results tables if a richer dataset becomes available.

## Repository structure

```
cardiox-clinical-data-project/
├── README.md
├── data/
│   ├── CardioX_Raw_Data.csv
│   └── Cleaned_Clinical_Data_for_SQL.csv
├── excel/
│   └── Cleaned_Clinical_Data.xlsx
├── sql/
│   └── cardiox_sql_queries.sql
├── powerbi/
│   ├── CardioX_Enrollment_Dashboard.pbix
│   └── screenshots/
│       └── enrollment_dashboard.png
```

*(Data dictionary and business insights are included as sections within this README rather than as separate files.)*

## Tools used

Excel (data cleaning & validation) · MySQL (relational database & analysis) · Power BI (dashboarding)
