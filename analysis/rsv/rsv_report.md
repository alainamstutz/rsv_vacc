# Descriptive Data Quality Assessment of RSV Vaccination Records

The structure of this report mirrors `analysis/covid/covid_report.md` in [opensafely/vaccines-data-quality](https://github.com/opensafely/vaccines-data-quality).

## 1. Aims

- Describe RSV vaccination records in OpenSAFELY-TPP at the event level
- Flag and quantify recording issues in dates, products, eligibility, same-day records, dose repeats (as far as applicable) and agreement between data sources

------------------------------------------------------------------------

## 2. Overview of Data Quality Framework

Six domains will be assessed:

1.  Impossible dates
2.  Non-routine NHS programme products
3.  Multiple vaccinations recorded on the same day
4.  Repeat doses (every pregnancy, other repeats)
5.  Eligibility on the vaccination date
6.  Agreement between data sources

- All issues are flagged and quantified
- Differences to the COVID-19 and flu vaccine data quality work (I think):
  - Year-round programme, so, no campaigns
  - One lifetime dose for older adults and other at-risk groups, one dose per/in every pregnancy
  - Eligibility widened on 2026-04-01 and again on 2026-09-01.
  - Maternity vaccinations reach GP records through the Record a Vaccination Service (RAVS)

The main policy source: The Green Book: <https://www.gov.uk/government/publications/respiratory-syncytial-virus-the-green-book>

------------------------------------------------------------------------

## 3. Definitions of Flags

Before constructing any flags:

- Records with missing vaccination dates (`vax_date`) are excluded
- Raw product names are harmonised using `vax_product_lookup`
- Programme phases are assigned from `vax_date` using `phase_info`, see below table with all programme phase info/dates
- Possible maternal record: female aged 14-50 on `vax_date` (using the preg algo at a later stage), with product Abrysvo or unspecified. Arexvy and mRESVIA have no pregnancy indication and are not licensed under 18.

#### Data sources

| Source | Selection |
|---|---|
| [`vaccinations`](https://docs.opensafely.org/ehrql/reference/schemas/tpp/#vaccinations) and [reference table](https://reports.opensafely.org/reports/opensafely-tpp-database-reference-values/#VaccinationReference-Table) | Target disease (column name: VaccinationContent) ==`"HUMAN RESPIRATORY SYNCYTIAL VIRUS"` |
| [`clinical_events`](https://docs.opensafely.org/ehrql/reference/schemas/tpp/#clinical_events) | [RSVADMIN_COD](https://www.opencodelists.org/codelist/nhsd-primary-care-domain-refsets/rsvadmin_cod/20260630/) (2 codes) |
| [`medications`](https://docs.opensafely.org/ehrql/reference/schemas/tpp/#medications) | RSV vaccine dm+d codelist? TBD. Check: <https://www.opencodelists.org/codelist/opensafely/bnf-chapter-14-immunological-products-and-vaccines-dmd/59ae8754/> |
| `practice_registrations`, `ons_deaths` | Registered and alive on the vaccination date |

#### Programme phases

| phase_label | phase_start_date | NHS programme: older adults | NHS programme: pregnancy | Outside the NHS programme |
|---|---|---|---|---|
| Pre-licence | 1900-01-01 | None | None | No licensed product. Trial products only. |
| Pre-programme | 2023-07-10 | None | None | - Arexvy from 2023-07-10 (approval date source see below)<br>- Abrysvo from 2023-11-29 (approval date source see below) |
| Launch | 2024-09-01 | Abrysvo for:<br>- routine cohort, from the 75th birthday<br>- catch-up cohort, all year, including those turning 80 | Abrysvo from week 28, every pregnancy | - Abrysvo, Arexvy<br>- mRESVIA from 2025-02-28 (approval date source see below)<br>(only Abrysvo licensed for pregnancy) |
| Year two | 2025-09-01 | Abrysvo for:<br>- routine cohort, from the 75th birthday<br>- catch-up cohort, until the 80th birthday | Abrysvo from week 28, every pregnancy | Abrysvo, Arexvy, mRESVIA<br>(only Abrysvo licensed for pregnancy) |
| Year two, adding 80+ and care homes | 2026-04-01 | Abrysvo for:<br>- everyone aged 75+, no upper age limit<br>- adult residents of care homes for older adults, any age | Abrysvo from week 28, every pregnancy | Abrysvo, Arexvy, mRESVIA<br>(only Abrysvo licensed for pregnancy) |
| Year three, adding at-risk 65-74y | 2026-09-01 | Abrysvo for:<br>- everyone aged 75+ and care home residents, as above<br>- aged 65-74 with chronic respiratory disease or immunosuppression | Abrysvo from week 28, every pregnancy | Abrysvo, Arexvy, mRESVIA<br>(only Abrysvo licensed for pregnancy) |

- Each phase ends the day before the next starts. The last phase ends at `end_date`.
- **Routine cohort:** turning 75 on or after 2024-09-01, i.e. born September 1949 or later.
- **Catch-up cohort:** aged 75-79 on 2024-08-31, i.e. born September 1944 to August 1949.
- From 2026-04-01 the upper age limit is removed
- **Licences/Indication (Green Book, June 2026):** Abrysvo for pregnancy and for adults 60+ or 18-59 at increased risk. Arexvy for adults 60+ or 50-59 at increased risk. mRESVIA for adults 60+ or 18-59 at increased risk. Only Abrysvo is licensed in pregnancy (weeks 28-36)!

------------------------------------------------------------------------

### 3.1 Impossible Dates

#### A. Pre-licence date

- **Definition:** `vax_date < 2023-07-10`, the first UK licence of an RSV vaccine.
- **Interpretation:** Error or trial participation

#### B. Pre-programme date

- **Definition:** `2023-07-10 <= vax_date < 2024-09-01`
- **Interpretation:** Private vaccination or trial participation.

------------------------------------------------------------------------

### 3.2 Product Mismatches

#### Product lookup and approval dates

| code | TPP `product_name` | UK approval | website |
|---|---|---|---|
| abrysvo | Abrysvo vaccine powder and solvent for solution for injection 0.5ml vials (Pfizer) | 2023-11-29 | [news blog](https://pmlive.com/pharma_news/pfizers_rsv_vaccine_granted_mhra_approval_to_protect_infants_and_older_adults_1504343) (exact date a bit unsure) |
| arexvy | Arexvy vaccine inj 0.5ml vials (GlaxoSmithKline UK Ltd) | 2023-07-10 | [GSK](https://www.gsk.com/en-gb/media/press-releases/medicines-and-healthcare-products-regulatory-agency-authorises-gsk-s-arexvy-the-first-respiratory-syncytial-virus-rsv-vaccine-for-older-adults/) press release |
| mresvia | ??? (not found yet in TPP [reference table](https://reports.opensafely.org/reports/opensafely-tpp-database-reference-values/#VaccinationReference-Table)) | 2025-02-28 | [Moderna](https://www.accessnewswire.com/newsroom/en/healthcare-and-pharmaceutical/moderna-receives-medicines-and-healthcare-products-regulatory-agency-m-993322) press release |
| unspecified | Respiratory Syncytial Virus (RSV) vaccine | n/a | TPP [reference table](https://reports.opensafely.org/reports/opensafely-tpp-database-reference-values/#VaccinationReference-Table) |

#### Product flags

| Flag | Definition | Interpretation |
|---|---|---|
| Unspecified product | `vax_product == "unspecified"` | Incomplete coding, possibly from an external source (e.g. maternity or pharmacy records?). May still be a programme vaccine, most likely Abrysvo? |
| Non-NHS programme product | arexvy or mresvia from 2024-09-01 | Vaccine given outside the programme, or product miscoded. Still counts as an RSV vaccination. |
| Before approval | `vax_date` before the product's UK approval date | Error or trial participation |
| Child | Age under 18 on `vax_date`, unless a possible maternal record | Administration error, or an infant antibody recorded as a vaccine? Arexvy and mRESVIA under 18 are always flagged, Abrysvo could indicate pregnancy? |

------------------------------------------------------------------------

### 3.3 Multiple Vaccinations on the Same Day

Grouped by `patient_id`, `vax_date` and `vax_product` (or not), as in the COVID-19 report

#### A. Same-day same-product

- **Definition:** Same patient, date and product, multiple entries
- **Interpretation:** Highly likely duplicate

#### B. Same-day mixed-product

- **Definition:** Same patient and date, different RSV products, for example abrysvo and "unspecified".

- **Interpretation:** Likely one dose recorded through two routes, such as the maternity (via RAVS) and a GP entry.

- Same-day vaccines against other diseases (flu, COVID-19, pertussis) are not flagged.

------------------------------------------------------------------------

### 3.4 Repeat Doses

- **Older adults and other eligible adults, including the immunosuppressed:** one dose only. The Green Book says revaccination is not currently recommended, so any second RSV record is flagged (`flag_repeat_dose`).
- **Possible maternal records:** one dose per pregnancy, so a second dose is expected only in a later pregnancy. Two records less than 180 days apart cannot be separate pregnancies and are flagged (`flag_repeat_same_pregnancy`).
- Only records on different days are compared. Same-day records are handled in 3.3 (deduplicated)
- Intervals reported in 3 bins of 1-27, 28-179 and 180+ days. Short intervals suggest one dose recorded twice with different dates. Longer ones rather suggest a real second dose, e.g. private then NHS?

------------------------------------------------------------------------

### 3.5 Eligibility on the Vaccination Date

| eligibility_status | Definition |
|---|---|
| eligible_age | Age rule for the phase met (see above) |
| possible_maternal | Possible maternal record, just based on female aged 14-50y (more complex stuff re preg algo for later) |
| possible_at_risk | Aged 65-74 from 2026-09-01, and a chronic respiratory disease or immunosuppression codelists (from JCVI work?) |
| possible_care_home | Aged 18-74 from 2026-04-01 and a care home code (or address matching) |
| not_eligible | None of the above |

- One person can fulfill several eligibility criteria

- **Interpretation not eligible:** Private vaccination, misdated record, missing eligibility coding, ...?

------------------------------------------------------------------------

### 3.6 Agreement Between Data Sources

Follows the flu pipeline:

- **Sources:** vaccinations table, clinical events (RSVADMIN_COD), medications (RSV dm+d).
- **Unit:** person per phase.
- **Source combinations:** table, snomed and drug, alone or in any combination.
- **Date agreement:** exact-day agreement between the vaccinations table and each other source. Differences binned as 0, 1-3, 4-7, 8-14 and 15+ days
- **Declined or contraindicated codes:** [RSVDEC_COD](https://www.opencodelists.org/codelist/nhsd-primary-care-domain-refsets/rsvdec_cod/20260630/) or [RSVCON_COD](https://www.opencodelists.org/codelist/nhsd-primary-care-domain-refsets/rsvcon_cod/20260630/) on or after a vaccination date counted as conflicts and added as separate flag

------------------------------------------------------------------------

## 4. Overview of All Flag Types

Non-interval flags are converted into long format:

- `flag_pre_licence_date`
- `flag_pre_programme_date`
- `flag_unspecified_product`
- `flag_non_programme_product`
- `flag_before_approval`
- `flag_child`
- `flag_not_eligible`
- `flag_same_day_same_product`
- `flag_same_day_mixed_product`

Only records with `flag_value == TRUE` are retained in the long-format table.

Repeat-dose flags from 3.4 (`flag_repeat_dose`, `flag_repeat_same_pregnancy`) are summarised separately with their interval bins. Same for source agreement (3.6).

------------------------------------------------------------------------

## 5. Summary Tables (similar to the covid report)

- All counts are midpoint 6 rounded

### 5.1 Non-interval summaries

- **Table 0** `count_record_flag_count.csv`: flags per record. Contents: `flag_count`, `n_records`, `denom_records_total`.
- **Table 1** `count_overall_noninterval_flags.csv`: each flag across all records. Contents: `n_records`, `n_patients`, total denominators.
- **Table 2** `count_phase_product_noninterval_flags.csv`: phase by product by flag, with records and patients active on the vaccination date as denominators.
- **Table 3** `count_phase_eligibility.csv`: phase by `eligibility_status` by age band by sex.

### 5.2 Repeat-dose summaries

- **Table 4** `count_repeat_doses.csv`: group (possible maternal, other) by interval bin (1-27, 28-179, 180+ days).
- **Table 5** `count_repeat_product_transition.csv`: previous product by current product by interval bin.

### 5.3 Source agreement summaries

- **Table 6** `table_rsv_sources.csv`: source combinations by phase by age band, plus UpSet counts.
- **Table 7** `table_date_agreement.csv`: date differences between sources, binned as outlined in 3.6.
- **Table 8** `table_vax_by_week_source.csv`: weekly counts by source.

Think of more pregnancy-specific summaries/insights?

------------------------------------------------------------------------

## 6. Main Descriptive Results

Summaries for each of the 6 domain (see Chapter 2)

------------------------------------------------------------------------

## 7. Potential Extensions and TBD

- **Double-check exact UK approval date for Abrysvo!**
- **Add codelists:** Care home, immunocompromised and CRD, and DM+D codelists
- **Pertussis in pregnancy:** same checks, as a comparator? but not covered by ethics approval.. same for mAbs...
- **Pregnancy (OpenPregnosis algo):** timing against week 28 and delivery (14+, 10-13 and 0-9 days before, after delivery), repeat doses within one pregnancy, arexvy or mresvia in pregnancy, doses outside any pregnancy.
- **Benchmarking:** compare coverage with UKHSA reports by region?
- **Confirm event-level data permission**

------------------------------------------------------------------------

## 8. ...

------------------------------------------------------------------------