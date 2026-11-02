# HR – SESMT (EHS): Safety Technicians

Power BI report that tracks the field work of the **Occupational Safety Technicians** (*TSTs, Técnicos de Segurança do Trabalho*). Each technician is assigned a **route** of client units and must visit each one at a set frequency and fill in a safety **checklist** that produces a score. The report shows route coverage, visits performed against the schedule and checklist scores per technician and unit.

📄 **Report preview:** [SESMT (EHS) - Safety Technicians.pdf](SESMT%20(EHS)%20-%20Safety%20Technicians.pdf)

> The numbers shown in the PDF do not represent the company's actual figures. The data was filtered and is outdated.

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

| Source | Tables |
|---|---|
| SharePoint / OneDrive (Excel files, combined with *Folder* helper queries) | Checklists, route assignments, visit schedule, unit categories |
| Embedded table | Company type |
| Power Query (derived) | Technicians and units dimensions, built from the fact and route tables |

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
