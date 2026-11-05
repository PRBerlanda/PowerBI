# HR – SESMT (EHS): Work Accidents & Absences

Power BI report for the **SESMT** team (*Serviço Especializado em Segurança e Medicina do Trabalho*, the Brazilian occupational health & safety department, equivalent to EHS). It monitors **work accidents, leaves, medical certificates and ICD codes** so the team can find which jobs, units and causes generate the most absences and how they evolve against the previous period.

📄 **Report preview:** [SESMT (EHS).pdf](SESMT%20(EHS).pdf)

> **About the data:** this portfolio version runs on **fully anonymized, fictitious data**. Names of employees, dependents, managers and recruiters, tax IDs, e-mails, phone numbers and addresses were replaced with synthetic values generated with the Python *Faker* library (pt_BR). Company, branch, cost center and site names were replaced with fictitious ones (for example, *Grupo Mendonça*). Registration numbers and internal codes were renumbered consistently across tables, monetary values were scaled by a random factor, sensitive attributes were shuffled between records, and the period was limited to **January 2024 – December 2025**. The figures do not represent any real company, and the source files are not included in this repository.

> **Architecture change.** In production this report held no data of its own: it was a **composite model** reading the *RH – Relações Trabalhistas* semantic model published in Microsoft Fabric (see the `HR-People-Analytics` branch) through **DirectQuery for Power BI semantic models**. For this portfolio version the remote connection was replaced, so the PBIP now refreshes on its own from the same anonymized CSV files used by People Analytics.

---

## Report pages

| Page | What it answers |
|---|---|
| **Work Accidents – Overview** (*Acidentes de Trabalho Geral*) | Number of work accidents, % with leave, monthly accident rate, comparison with the previous period, and breakdowns by body part affected, severity, type (typical / commuting) and laterality. A field parameter switches the analysis axis (month, company, cost center, job type, days absent). |
| **Work Accidents – Table** | Detailed accident log with switchable columns: employee, date, job, accident type, days off, CAT (accident report), severity, body part, laterality, nature and causing agent. |
| **Absences – Detail** (*Ausências Detalhado*) | Number of leaves, days away and average days per leave, by type, severity and laterality, compared with the previous period. |
| **Absences – Overview** (*Ausências Geral*) | Total employees on leave by reason (work accident, sickness benefit, maternity leave, labor leave, retirement), % on leave, monthly comparison and quarterly history, by job, tenure, age range, education, marital status, ethnicity or company. |
| **ICD** (*CID*) | Leaves with and without an ICD code, counts per ICD chapter or code, ICD by company and by leave type, most affected jobs and trend over time. |
| **Medical Certificates** (*Atestados*) | Certificates delivered and days missed (vs. previous year), by company, cost center, location, employee and ICD, and by certificate type. |
| **Absences / Certificates – Table** | Employee-level detail with switchable dimensions (*Categorias*) and metrics (*Valores*). |

All pages share a header with navigation arrows and a filter panel.

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 73: 60 tables imported from the shared CSV files, 2 calculation groups, 10 field parameters and the measure table (originally 61 DirectQuery tables from the shared model) |
| Measures | 54 (48 SESMT-specific + 6 brought from the shared model) |
| Relationships | 72 (13 inactive) |
| Report pages / visuals | 7 / 208 |

**Original architecture.** The report reused the enterprise HR semantic model instead of duplicating it. It brings in only what it needs (employees, leaves, work accidents, ICD, body part, severity, laterality, causing agent, nature, jobs, cost centers, companies, calendar and so on) and adds:
- a local `Medidas` table with SESMT-specific measures, in the folders **Acidentes** (accidents), **Afastamentos** (leaves) and **Outros** (other);
- local **field parameters** for the dynamic axes and dynamic table columns.

**Portfolio version.** The 60 shared tables are imported with Power Query from the anonymized CSV files (folder set by the `PastaBaseDados` parameter). The calculated columns and the two calculation groups that lived in the shared model (job SLA and leadership level, management hierarchy levels, RLS levels, `AFASTAMENTOS`, `MÉDIA E TOTAL`) were recreated locally with the same DAX, and the measures the report used from the shared model (average salary by gender, hourly wage and their dependencies) were copied into the local `Medidas` table.

**Measures (selection)**
- *Accidents:* accident count (current vs. previous year), CATs issued, typical / commuting / disease accidents, light / moderate / severe accidents, causes and body parts.
- *Leaves:* employees on leave (current vs. previous period), days away, average days away, % of accidents with leave, causes that generate the most leaves, leaves with and without ICD, medical certificates.

---

## DAX highlights

```dax
Pct de acidentes com afastamento =
DIVIDE ( [Nº de afastamentos], [Nº de acidentes de trabalho] )
```

```dax
Média de Dias afastados =
CALCULATE (
    AVERAGEX ( ACIDENTE_TRABALHO, ACIDENTE_TRABALHO[DIAS_DEBITADOS] ),
    ACIDENTE_TRABALHO[DIAS_DEBITADOS] > 0
)
```

---

## Techniques used

- **Composite model / DirectQuery for Power BI semantic models** (production): one governed source of truth, extended locally with report-specific measures.
- Conversion of the composite model into a standalone import model for the portfolio, preserving calculated columns, calculation groups and measures.
- Field parameters for dynamic axes and configurable detail tables.
- Period-over-period comparisons (current vs. previous period / year).
- Consistent page header with navigation and filter panel.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Microsoft Fabric (shared semantic model, production) · Power Query (M) · DAX
