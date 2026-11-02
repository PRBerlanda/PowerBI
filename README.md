# HR – SESMT (EHS): Work Accidents & Absences

Power BI report for the **SESMT** team (*Serviço Especializado em Segurança e Medicina do Trabalho*, the Brazilian occupational health & safety department, equivalent to EHS). It monitors **work accidents, leaves, medical certificates and ICD codes** so the team can find which jobs, units and causes generate the most absences and how they evolve against the previous period.

📄 **Report preview:** [SESMT (EHS).pdf](SESMT%20(EHS).pdf)

> The numbers shown in the PDF do not represent the company's actual figures. The data was filtered and is outdated.

> ⚠️ **Live connection.** This report does not hold its own data. It is a **composite model** that reads the *RH – Relações Trabalhistas* semantic model published in Microsoft Fabric (see the `HR-People-Analytics` branch) through **DirectQuery for Power BI semantic models**. Since that workspace is no longer available, the PBIP opens without data; the PDF documents how the report looked in production.

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
| Tables | 73: 61 DirectQuery tables from the shared model, 10 local field parameters and 2 calculation groups |
| Local measures | 48 |
| Relationships | 72 (13 inactive) |
| Report pages / visuals | 7 / 208 |

**Architecture.** The report reuses the enterprise HR semantic model instead of duplicating it. It brings in only what it needs (employees, leaves, work accidents, ICD, body part, severity, laterality, causing agent, nature, jobs, cost centers, companies, calendar and so on) and adds:
- a local `Medidas` table with SESMT-specific measures, in the folders **Acidentes** (accidents), **Afastamentos** (leaves) and **Outros** (other);
- local **field parameters** for the dynamic axes and dynamic table columns.

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

- **Composite model / DirectQuery for Power BI semantic models**: one governed source of truth, extended locally with report-specific measures.
- Field parameters for dynamic axes and configurable detail tables.
- Period-over-period comparisons (current vs. previous period / year).
- Consistent page header with navigation and filter panel.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Microsoft Fabric (shared semantic model) · DAX
