# HR – People Analytics (Labor Relations)

The flagship HR application: a single Power BI report and semantic model covering the **whole employee lifecycle** (headcount, admissions, terminations, retention, turnover, absenteeism, leaves, diversity, payroll, benefits, labor lawsuits, time and attendance, vacations, job openings and training), with **row-level security by management hierarchy** and a built-in **severance calculator**.

The semantic model was published to a Microsoft Fabric workspace and reused as a shared dataset by other reports (for example, the SESMT / EHS report connects to it through DirectQuery).

📄 **Report preview:** [People Analytics.pdf](People%20Analytics.pdf)
🗺️ **Semantic model diagrams:** [Semantic Model.pdf](Semantic%20Model.pdf)

> **About the data:** this portfolio version runs on **fully anonymized, fictitious data**. Names of employees, dependents, managers and recruiters, tax IDs, e-mails, phone numbers and addresses were replaced with synthetic values generated with the Python *Faker* library (pt_BR). Company, branch, cost center and site names were replaced with fictitious ones (for example, *Grupo Mendonça*). Registration numbers and internal codes were renumbered consistently across tables, monetary values were scaled by a random factor, sensitive attributes were shuffled between records, and the period was limited to **January 2024 – December 2025**. The figures do not represent any real company, and the source files are not included in this repository.

---

## Report pages

The report opens on a personalized **menu** ("Hello, *user*!") that groups 40 pages into five areas.

| Area | Pages |
|---|---|
| **People** | Overview · Headcount · Admissions · Terminations · Retention · Leaves (*Afastamento*) · Diversity · Tenure & Age · Hierarchy · Employee Lookup · Birthdays & Commemorative Dates |
| **Indicators** | Absenteeism · Turnover · Salary Equity · Labor Lawsuits · Lawsuit Costs · Indicators (monthly KPI pack) |
| **Remuneration & Benefits** | Remuneration · Payroll Ledger (*Ficha Financeira*) · Open / Summarized Payroll · Monthly Position · Remuneration & Benefits Analysis · Raises & Transfers · Results Structure |
| **Personnel Administration** | Vacations · Time Clock (*Ponto*) · Hour Bank · Time & Hour Bank Detail · Reports · Monthly Submissions · **Severance Calculator** |
| **Other** | Job Openings (summary and detail) · User Guide |

Several analyses have a hidden *drill-through / detail* page (Headcount, Leaves, Tenure, Salary Equity, Job Openings, Time & Hour Bank).

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 122 (36 field parameters, 4 calculation groups) |
| Measures | **761**, organized in display folders |
| Calculated columns | 39 |
| Relationships | 130 (31 inactive) |
| Security roles | 2 (`Geral`, `Gestores`) |
| Report pages / visuals | 41 / ~1,760 |

**Model layout.** The model is documented in diagram tabs (see *Semantic Model.pdf*): Time Clock History, Moves & Raises History, Payroll Ledger, Hierarchy, Work Accidents, Hour Bank, Leaves, Job Openings, Lawsuits, Training, Time Clock and Benefits.

**Core tables**
- **Employee hub:** `COLABORADORES`, `HISTORICO` (daily employee snapshot used for point-in-time headcount), `HIERARQUIA`, `DEPENDENTE`.
- **Facts:** `FICHA_FINANCEIRA` (payroll events), `BENEFICIOS`, `AFASTAMENTO` (leaves), `ACIDENTE_TRABALHO` (work accidents), `PONTO` (time clock), `BANCO_HORAS` (hour bank), `FERIAS` and `SALDO_FERIAS` (vacations), `VAGAS` (job openings), `PROCESSOS` and `PROCESSOS_CUSTOS_MENSAIS` (lawsuits), `FATURAMENTO` (revenue), plus history tables for salary, job and cost center.
- **Cost allocation tables** (`RATEIO_CD`, `RATEIO_PCD_APRENDIZ`, `RATEIO_PCP`, `RATEIOS_DIVERSOS`, `RATEIO_TERCEIROS_PJ`) that spread shared payroll costs across cost centers.
- **Conformed dimensions:** `DATA`, `CARGO`, `CENTRO_CUSTO`, `EMPRESA`, `FILIAL`, `LOCAL`, `ESCALA`, `SINDICATO`, `GENERO`, `ETINIA`, `GERACOES`, `FAIXA_ETARIA`, `TEMPO_CASA`, `CID`, and more.

**Calculation groups**
- `INTELIGÊNCIA TEMPORAL`: 33 time-intelligence items (MTD, QTD, YTD, PM, PQ, PY, MoM / QoQ / YoY with and without %, MAT / PYMAT / MATG, 6‑ and 12‑month moving averages, ranking).
- `CALCULOS TEMPORAIS`: Current, Cumulative, Growth, Moving Average, Percent.
- `AFASTAMENTOS`: switches between leaves / medical certificates / absences, *in the period* vs. *running*.
- `MÉDIA E TOTAL`: average vs. total.

**Measure folders (selection):** Headcount · Admissions & Terminations · Retention (fixed and variable windows) · Absenteeism & Turnover · Leaves · Accidents · Salaries (nominal, average, gender gap) · Payroll events & encargos (employer charges) · Cost allocation · Benefits (health, dental, pharmacy, transportation, meal) · Severance (13th salary, vacation, FGTS, notice period, stability) · Lawsuits · Training (66 measures) · Job openings · KPI indicators (cost per capita, benefit share and more) · PLR (profit sharing).

---

## Security (RLS)

The `Gestores` (managers) role filters the `RLS` table by the signed-in user's UPN against up to six levels of the management hierarchy. Each manager sees only their own organization; the `Geral` role has full access.

```dax
-- table permission on RLS
[user principal name] = 'RLS'[NIVEL 1 E-MAIL] ||
[user principal name] = 'RLS'[NIVEL 2 E-MAIL] ||
...
[user principal name] = 'RLS'[NIVEL 6 E-MAIL]
```

---

## Data sources

Original sources in production:

| Source | Tables |
|---|---|
| Oracle data warehouse (HR / payroll system) | Employees, dependents, jobs, companies, branches, schedules, benefits, work accidents, time clock, job openings |
| Power Platform Dataflows | Daily employee history, payroll ledger, leaves, vacations, hour bank, salary / job / cost center history, hierarchy and RLS, cost centers |
| SharePoint (Excel) | Lawsuits and lawsuit costs, revenue, payroll event catalog, ICD codes, holidays, cost allocation inputs |
| Embedded tables / Power Query lists | About 40 small lookups (status codes, gender, generations, severity…) |
| DAX calculated tables | 36 field parameters, 4 calculation groups, measure table |

**Portfolio version.** All 81 import queries now read the anonymized CSV files from a local folder defined by the `PastaBaseDados` parameter (Power Query `Csv.Document`, pt-BR culture); incremental refresh was removed. The original connection strings were removed from the project.

---

## DAX highlights

**Point-in-time headcount**, from the daily history snapshot (excluding status 7, terminated):

```dax
Headcount =
VAR maxData = MAX ( 'DATA'[Data] )
RETURN
    CALCULATE (
        DISTINCTCOUNT ( 'COLABORADORES'[CPF] ),
        'HISTORICO'[DATA_CALENDARIO] = maxData,
        HISTORICO[COD_SITUACAO] <> 7
    )
```

**Absenteeism rate**, from payroll hour events:

```dax
Absenteismo =
DIVIDE (
    [Horas Faltas FF] + [Horas Atestados FF],
    [Horas Faltas FF] + [Horas Atestados FF] + [Horas Trabalhadas FF]
)
```

**Retention.** The methodology changed at a cut-off date and the measure switches automatically:

```dax
Retenção Geral =
VAR maxData   = MAX ( 'DATA'[DATA] )
VAR dataCorte = DATE ( 2024, 12, 31 )
RETURN
    SWITCH ( TRUE (), dataCorte >= maxData, [Retenção Fixa 3 Meses], [Retenção Variável 3 meses] )
```

**Severance calculator: FGTS fine (40%).** This is only evaluated when a single employee is selected:

```dax
FGTS Multa =
IF (
    ISFILTERED ( COLABORADORES[MATRICULA_NOME] ),
    ( [FGTS Saldo] + [FGTS dias Trabalhados] + [FGTS decimo terceiro sal. deposito] + [FGTS Aviso Prévio] ) * 0.40
)
```

---

## Techniques used

- Enterprise-scale star schema (120+ tables) shared as a **Fabric semantic model** and reused by other reports.
- Dynamic **row-level security** driven by the management hierarchy (`USERPRINCIPALNAME`).
- Calculation groups for reusable time intelligence and for switching analysis modes.
- 36 field parameters for user-driven axes, metrics and table columns.
- Daily snapshot table for accurate point-in-time headcount.
- Cost allocation logic across cost centers in DAX.
- A severance calculator implementing Brazilian labor law rules (CLT): 13th salary, vacation + ⅓, FGTS + 40% fine, notice period and stability.
- Personalized landing page, menu navigation, bookmarks and drill-through pages.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Microsoft Fabric · Power Query (M) · DAX · Oracle · Power Platform Dataflows · SharePoint / Excel · Python (data anonymization)
