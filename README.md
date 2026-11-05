# HR – SESMT (EHS): Safety Technicians

Power BI report that tracks the field work of the **Occupational Safety Technicians** (*TSTs, Técnicos de Segurança do Trabalho*). Each technician is assigned a **route** of client units and must visit each one at a set frequency and fill in a safety **checklist** that produces a score. The report shows route coverage, visits performed against the schedule and checklist scores per technician and unit.

📄 **Report preview:** [SESMT (EHS) - Safety Technicians.pdf](SESMT%20(EHS)%20-%20Safety%20Technicians.pdf)

> **About the data:** this portfolio version runs on **fully anonymized, fictitious data**. Names of employees, dependents, managers and recruiters, tax IDs, e-mails, phone numbers and addresses were replaced with synthetic values generated with the Python *Faker* library (pt_BR). Company, branch, cost center and site names were replaced with fictitious ones (for example, *Grupo Mendonça*). Registration numbers and internal codes were renumbered consistently across tables, monetary values were scaled by a random factor, sensitive attributes were shuffled between records, and the period was limited to **January 2024 – December 2025**. The figures do not represent any real company, and the source files are not included in this repository.

---

## Report pages

| Page | What it answers |
|---|---|
| **TST Indicators** (*Indicadores TST*) | Average checklist score and number of scored checklists; % of routes covered in the month and the year (routes registered vs. visited vs. unique routes); a gauge against the annual coverage target; cumulative scores and visits over the year; score by unit. A technician selector covers active and terminated technicians. |
| **Units** (*Unidades*) | Searchable list of units served (visits and average score) next to the list of routes **not yet visited**, filtered by state, category, frequency and person in charge. |
| **Tooltip** (hidden) | Report-page tooltip with indicator details. |

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 9 |
| Measures | 24 |
| Calculated columns | 2 |
| Relationships | 11 (2 inactive) |
| Report pages / visuals | 3 / 53 |

**Star schema**
- **Facts:** `Fato_CheckList` (one row per visit: technician, unit, date, score, notes) and `Fato_Programação_Visitas` (scheduled visits).
- **Dimensions:** `Dim_Rotas` (route assignments with a validity period: unit, state, technician, frequency, person in charge, start / end date), `Dim_Unidades`, `Dim_Categoria`, `Dim_Tipo_Empresa`, `Dim_TST's` (technicians) and a `Data` calendar.
- Route assignments are **slowly changing**: a unit can move between technicians over time. The `Data Inicial` / `Data Final` columns are related to the calendar through inactive relationships, and the measures filter routes valid on the selected date.

---

## Data sources

Original sources in production:

| Source | Tables |
|---|---|
| SharePoint / OneDrive (Excel files, combined with *Folder* helper queries) | Checklists, route assignments, visit schedule, unit categories |
| Embedded table | Company type |
| Power Query (derived) | Technicians and units dimensions, built from the fact and route tables |

**Portfolio version.** All 8 import queries now read the anonymized CSV files from a local folder defined by the `PastaBaseDados` parameter (Power Query `Csv.Document`, pt-BR culture); technician and unit names are fictitious. The original connection strings were removed from the project.

---

## DAX highlights

**Routes valid on a date** (slowly changing assignment):

```dax
Contagem Dinâmica de Rotas por TST =
VAR dataMax = MAX ( Data[Data] )
RETURN
    CALCULATE (
        DISTINCTCOUNT ( Dim_Rotas[CHAVE] ),
        OR (
            AND ( Dim_Rotas[Data Inicial] <= dataMax, Dim_Rotas[Data Final] >= dataMax ),
            AND ( Dim_Rotas[Data Inicial] <= dataMax, Dim_Rotas[Data Final] = BLANK () )
        ),
        REMOVEFILTERS ( Data )
    )
```

**Annual coverage target:**

```dax
Pct Meta =
VAR DIV = DIVIDE ( [Rotas Atendidas no Ano (todos TST's)], [Rotas Cadastradas por TST] )
RETURN IF ( DIV <> BLANK (), DIV, 0 )
```

**Monthly visit compliance** (visits done vs. scheduled):

```dax
Indicador Visitas no Mês =
VAR calc = DIVIDE ( [Visitas Realizadas], [Programação de Visitas] )
RETURN IF ( calc <> BLANK (), calc, 0 )
```

---

## Techniques used

- Date-effective (SCD type 2–like) route assignments handled in DAX.
- Excel files on SharePoint combined with Power Query *Folder* helper functions, with the SharePoint path stored as a parameter.
- Gauge against a target, cumulative trend lines and a report-page tooltip.
- Keyword search on units and a list of units not visited.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Power Query (M) · DAX · SharePoint / Excel
