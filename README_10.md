# Healthcare Data Governance and Cleaning

**Files**
| File | Purpose |
|---|---|
| `raw_healthcare_patients_sample.csv` | Untouched source data (10 rows) |
| `clean_patients.py` | Reproducible validation + cleaning script |
| `clean_healthcare_patients.csv` | Cleaned output (BP split into SystolicBP / DiastolicBP) |
| `data_dictionary.csv` | Field types, ranges, privacy classification |
| `exception_log.csv` | Every rule hit with severity and action |

Run: `pip install pandas && python clean_patients.py`

## Why this sample counts as de-identified
- No direct identifiers: no name, address, phone, email, DOB, MRN, or exact visit dates.
- `PatientID` is a synthetic sequential key (PAT-201…210) with no embedded personal data.
- Age is the only quasi-identifier-style field with detail; at 10 synthetic rows with no geography or dates, re-identification risk is low. The dataset is also labelled synthetic training data by its publisher.
- Real-world caution: if this were live data, I'd generalise age to bands, check for small cells (e.g. one patient with Hypothyroidism), and store any re-identification key separately with access control (HIPAA Safe Harbor / expert-determination thinking).

## Field classification
- **Pseudonymous ID:** PatientID
- **Quasi-identifiers:** Age, Gender
- **Sensitive health data:** Diagnosis, PrimaryMedication, Readmitted
- **Clinical measures:** DosageMg, TreatmentDurationDays, BloodPressure, CholesterolLevel

## Validation results
| Check | Rule | Result |
|---|---|---|
| Types | Numeric fields parse as numbers | Pass (0 failures) |
| Missing values | No blanks in any field | Pass (0 missing) |
| Duplicates | PatientID unique | Pass (10 unique IDs) |
| ID format | `PAT-###` | Pass |
| Ranges | Age 0-120, Dose 0.001-2000 mg, Duration 1-730 days, Cholesterol 80-400 | Pass |
| Categories | Gender, Readmitted in allowed lists | Pass |
| Blood pressure | `SYS/DIA` format, plausible, systolic > diastolic | Pass |

## Cleaning rules (no invented values)
1. Trim whitespace and standardise case on categorical text only.
2. Remove only *exact* duplicate rows; same-ID conflicting rows are kept for review.
3. Out-of-range, wrongly typed, or malformed values are set to null and logged, never imputed or "corrected" by guess.
4. Split BloodPressure into two numeric columns (a format change, not a value change).
5. Clinical-consistency issues are flagged for review, not edited.

## Exception log summary
This sample was already clean: **1 informational entry**. PAT-210 Levothyroxine `0.05` mg is valid (= 50 mcg) but the drug is normally labelled in micrograms, so a unit-column is recommended to prevent misreading as an overdose or a typo. No values were changed.

## Limitations and next steps
- 10 rows is too small for statistical conclusions.
- `CholesterolLevel` doesn't state which lipid measure (total/LDL) it is; confirm with the source.
- Dosage unit is assumed mg for all drugs; add an explicit unit column.
- The script is built to catch problems on larger or messier files, so test it by injecting errors (blank, duplicate ID, Age=250) to prove the rules fire.
