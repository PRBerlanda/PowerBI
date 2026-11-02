# HR – Benefits and Remuneration

Power BI report that monitors **health and dental plan adoption, employee awards and salary increases**. It was built for the HR team as a monthly follow-up tool, comparing every indicator with the previous month and the previous year.

📄 **Report preview:** [Benefits and Remuneration.pdf](Benefits%20and%20Remuneration.pdf)

> The numbers shown in the PDF do not represent the company's actual figures. The data was filtered and is outdated.

---

## Report pages

| Page | What it answers |
|---|---|
| **Benefits** (*Benefícios*) | How many lives (employees + dependents) are covered by the health and dental plans? How many new enrollments happened in the period? What is the monthly cost split between the employee and the company? How has the 6‑month moving average evolved? |
| **Awards** (*Premiações*) | How many employees received each award program (Refer a Friend, *Conte Comigo*, Effectiveness Bonus, Eyeglasses Voucher)? What was the amount paid compared with the previous month and the previous year, and which cost centers received the most? |
| **Salary Increases** (*Aumentos Salariais*) | How many raises were granted, what share of the workforce they cover, the total and average increase, the gender split, and the distribution by cost center. Includes a chart/table toggle. |

**UX features**
- Bookmark-driven navigation (chart ↔ table views, filter panel).
- **Field parameters** let users switch the breakdown dimension (age range, salary group, gender…) and the metric (active plans, enrollments, raise count, average raise) without leaving the page.
- KPI cards with sparklines and *previous month / previous year* comparisons.

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 43 (11 of them are field parameters, 1 is a calculation group) |
| Measures | 166 |
| Calculated columns | 15 |
| Relationships | 45 (14 inactive, enabled on demand with `USERELATIONSHIP`) |
| Report pages / visuals | 3 / 121 |

**Star schema.** The central dimension is `COLABORADORES` (employees). It is linked to the fact tables `BENEFICIOS` (plan enrollments for employees and dependents), `FICHA_FINANCEIRA` (payroll events), `HISTORICO_SALARIO` (salary history), `AFASTAMENTO` (leaves) and `VAGAS` (job openings). It is also linked to lookup dimensions such as `CARGO` (job), `CENTRO_CUSTO` (cost center), `EMPRESA` (company), `ESCALA` (work schedule), `SINDICATO` (union), `FAIXA_ETARIA` (age range), `GRAU_INSTRUCAO` (education) and `TEMPO_CASA` (tenure). A single `DATA` calendar (generated in Power Query from the `Data_Inicial` / `Data_Final` parameters) drives all time intelligence.

**Measures** are kept in a dedicated `Medidas` table and organized into display folders:
- `Benefícios\Colaboradores` and `Benefícios\Dependentes`: counts and costs for health and dental plans.
- `Financeiro\…`: one folder per award program, each with value, count, previous month, previous year and % of winners.
- `Salarios\Aumentos` and `Salarios\Diferença Entre Salários`: raise counts, MoM, YoY, running totals and the difference between the current and previous salary.
- `Vagas`: opened, finished and cancelled job requisitions in the period.
- `Headcount`: point-in-time headcount, with and without employees on leave.

---

## Data sources

| Source | Tables |
|---|---|
| Oracle data warehouse (HR schema) | Employees, jobs, benefits, leaves, job openings, hierarchy, schedules |
| Power Platform Dataflows | Payroll events, cost centers, salary / job / cost center history, sectors |
| SharePoint (Excel) | Salary structure (grades and ranges) |
| Embedded tables (Enter Data) | Small lookups: union, employee status, contract type, age range, benefit plan category |

---

## DAX highlights

**Point-in-time headcount.** This counts everyone admitted on or before the last date in context who had not yet left by that date:

```dax
Headcount =
VAR maxData = MAX ( DATA[Data] )
RETURN
    CALCULATE (
        [Contagem de Colaboradores],
        OR (
            AND ( COLABORADORES[DATA_ADMISSAO] <= maxData, COLABORADORES[DATA_DEMISSAO] > maxData ),
            AND ( COLABORADORES[DATA_ADMISSAO] <= maxData, ISBLANK ( COLABORADORES[DATA_DEMISSAO] ) )
        )
    )
```

**Previous salary.** This finds the salary in force before the most recent change, which is used to measure raise size:

```dax
Salario Anterior =
VAR UltSal = [Data Salario Anterior]
RETURN
    CALCULATE (
        [Salario Atual],
        HISTORICO_SALARIO[DATA_INICIO] = UltSal,
        ALLEXCEPT ( DATA, DATA[Mês] )
    )
```

---

## Techniques used

- Star schema with role-playing dates through inactive relationships and `USERELATIONSHIP`.
- Field parameters for dynamic axes and dynamic metrics.
- A calculation group scaffold for reusable time logic.
- A DAX calculated table (`HISTORICO_CENTRO_CUSTOS`) that expands cost center history into one row per employee per day.
- Time intelligence: MoM, YoY, running totals and a 6‑month moving average.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Power Query (M) · DAX · Oracle · Power Platform Dataflows · SharePoint
