# Klinik Medinara: Appointment & Patient Analytics

Analysis of a clinic's operational data to find where revenue is leaking and whether its follow-up process is working. Built with Excel (Power Query, Data Model, DAX) for cleaning and analysis, and Power BI for the interactive dashboard.

## Business problems

**1. Where is revenue leaking?**
15.0% of all bookings end as cancelled or no-show, representing about **Rp135.87 million** in revenue at risk. A chi-square test of independence across seven possible drivers (payment method, insurance type, specialization, doctor, weekday, time slot, and month of year) found only **one statistically real driver: payment method** (p = 0.0029). Cash and bank-transfer patients miss appointments noticeably more often than BPJS or private-insurance patients (18.3% and 15.8% vs. 13.9% and 12.5%). Every other factor tested was not significant, so the problem isn't concentrated in any doctor, day, or time slot, it's broad-based and tied to how the visit is paid for.

**2. Is the follow-up process working?**
1,123 diagnoses were flagged "follow-up required." Those patients return at about the same rate as patients who weren't flagged (56.5% vs. 57.2%), and come back within 30 days *less* often (6.3% vs. 8.0%). The follow-up flag has no measurable effect on whether, or how quickly, patients return, the opposite of what a working follow-up system should produce.

## Data

Five related CSV files, loaded into Excel's Data Model:

| Table | Rows | Description |
|---|---|---|
| patients | 2,000 | demographics, city, insurance type, registration date |
| doctors | 25 | specialization, experience, consultation fee |
| appointments | 5,000 | 4,000 completed, 400 cancelled, 350 no-show, 250 rescheduled |
| diagnoses | 3,880 | ICD code, description, severity, follow-up flag |
| prescriptions | 6,000 | medicine, dosage, route, duration |

Data covers 4 June 2021 to 31 January 2025. 2021 and 2025 are partial years.

## Data quality issues found and handled

- **Mixed date formats**: `appt_date` and `prescribed_date` mix three formats (`yyyy-mm-dd`, `dd/mm/yyyy`, `dd-mm-yyyy`) in the same column, day-first. Parsed by text pattern rather than relying on locale auto-detection, which could silently swap day and month.
- **Missing ICD codes**: 698 of 3,880 diagnoses (18%) carry no code, only a vague placeholder description ("Diagnosis sementara," "Gejala umum tidak spesifik," and similar). Labeled "Uncoded" and reported separately rather than dropped.
- **Fees on non-completed visits**: cancelled, no-show, and rescheduled appointments all still carry a fee. Revenue is counted from completed visits only; fees on the other three statuses are reported as revenue at risk, not income.
- **120 completed appointments have no diagnosis logged.** Kept as visits, excluded from diagnosis-level analysis.
- **Other missing values**: blood type missing for 300 patients (15%, filled "Unknown"), phone for 160 (8%, unused), diagnosis notes for 1,552 (40%, unused).
- **Repeated patient names**: 765 names repeat across different patients. Patient IDs are unique and used for all joins; names are never used to identify a patient.

All keys check out clean: no orphaned foreign keys, no duplicate IDs, and every join (patients → appointments → diagnoses → prescriptions) resolves correctly.

## Other findings

- **Revenue and visit volume grow steadily** from mid-2021 through 2024, consistent with a young, growing clinic.
- **BPJS is disproportionately valuable**: 48% of visits, but 61% of revenue, since BPJS patients tend to see higher-fee doctors.
- **Visits rise steadily with age**, from 372 for children (0-11) to 930 for seniors (60+).
- **Jakarta leads by city** with 950 of 4,000 completed visits (24%); the top 4 Jabodetabek cities account for 63%.
- **Severity mix**: mild 60.3%, moderate 30.0%, severe 9.7%.
- **Prescribing**: no single medicine dominates (top 15 all in a 203-222 range); oral is 73.5% of all prescription routes; the general practice (Umum) specialization accounts for 34% of all prescriptions.

## Dashboard (Power BI)

Three pages, built on the same Data Model as the Excel analysis:

1. **Overview** — revenue, visits, missed rate, revenue at risk, and return rate KPIs; monthly revenue trend; visits by age group; visits by city.
2. **Missed Appointment** — missed rate by payment method; the chi-square result (p = 0.0029); missed rate by calendar month (confirmed not significant, p = 0.29); revenue at risk by payment method.
3. **Diagnoses & Follow-up** — return rate and 30-day return rate by follow-up flag; the follow-up finding; top diagnoses; severity mix.

All chart axes are forced to start at 0, so visual height reflects the real scale of the data rather than exaggerating small swings (the Monthly Missed Rate line chart originally auto-scaled from 12%, making a ~5-point swing look dramatic; it's now 0%-based).

Screenshots: `images/page1-overview.png`, `images/page2-missed-appointments.png`, `images/page3-diagnoses-followup.png`

## Repository structure

```
Medinara/
  data/              raw CSVs
  excel/             Medinara_Analysis.xlsx (cleaning, Data Model, DAX, statistical tests)
  powerbi/           Medinara_Dashboard.pbix (3-page interactive dashboard)
  images/            dashboard screenshots
  README.md
```

## Tools

- **Excel**: Power Query (data cleaning), Data Model and relationships, DAX measures, PivotTables, CHISQ.TEST for statistical validation
- **Power BI Desktop**: imported Data Model, 3-page report with slicers, cards, and charts

## Limitations

- The Excel workbook and Power BI file are separate copies after import; a change made in one does not carry over to the other.
- `age_group` is calculated from today's date each time the data refreshes, so age-band boundaries can shift slightly between refreshes.
- Follow-up return rates for visits near the end of the data window (late 2024 - January 2025) are understated, since those patients have had less time to return.
- The appointment fee always equals the doctor's listed consultation fee; there's no price variation by visit type to analyze.
