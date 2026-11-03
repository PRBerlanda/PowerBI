# HR – Recruitment & Selection

Power BI report built for the **Recruitment & Selection (R&S) team** to manage job requisitions end to end. It tracks openings in progress, filled and cancelled positions, time to hire in **business days** and compliance with the team's **hiring SLA**, alongside headcount, admissions, terminations and turnover.

📄 **Report preview:** [RH - Recrutamento & Seleção.pdf](RH%20-%20Recrutamento%20%26%20Sele%C3%A7%C3%A3o.pdf)

> The numbers shown in the PDF do not represent the company's actual figures. The data was filtered and is outdated.

---

## Report pages

| Page | What it answers |
|---|---|
| **Menu** | Landing page with navigation buttons to every section. |
| **General Summary** (*Resumo Geral*) | Executive KPIs for the period: openings worked, opened, in progress and closed; % closed within / outside the SLA; average days to hire (excluding apprentices); headcount, admissions, terminations and turnover. |
| **Openings Summary** (*Resumo de Vagas*) | Openings in progress, opened, finished and cancelled, broken down by a user-selected dimension (field parameter) and by recruiter. |
| **SLA Target** (*Meta SLA*) | Gauge of % of openings closed within the deadline against the **90% target**, per recruiter and team. |
| **R&S Indicator** (*Indicador ReS*) | Monthly trend of openings closed within and outside the deadline, average business days to close an opening, and the flow of opened, finished and cancelled openings. |
| **NVR** | Openings balance (worked − closed), the previous month's balance, year-to-date openings worked and the ratio of closed to opened, set against headcount and turnover. |
| **Openings Table** (*Tabela de Vagas*) | Detailed list of requisitions with dynamic columns (field parameter) and filters for opening type, status, process stage and job type. |

Every page shares slicers for year, month, company, branch, cost center, job, job type, recruiter and opening type.

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 18 business tables (6 field parameters), plus 16 auto date/time tables |
| Measures | 47 |
| Calculated columns | 5 |
| Relationships | 35 (10 inactive) |
| Report pages / visuals | 7 / 198 |

**Main tables**
- `Vagas`: fact table of job requisitions (requisition, stage, status, requisition / forecast / closing / cancellation dates, recruiter, salary, replaced employee, PwD flag).
- `Quadro de Vagas`: planned vs. filled positions per sector and job.
- `Colaboradores` and `Afastamento`: employees and leaves, used for headcount and turnover.
- Dimensions: `Cargos` (jobs), `Centro de Custo` (cost centers), `Empresas` (companies), `Filiais Ativas` (active branches), `Escala` (work schedules), `Feriados` (holidays) and the `Calendario` calendar.

**Business rules modeled**
- **SLA per job type.** The calculated column `Cargos[PRAZO_CARGO]` sets a deadline by job level from 15 days for operational roles to 35 days for managers and executives.
- **Business-day lead time.** `Vagas[PRAZO_CONTRATACAO]` uses `NETWORKDAYS` with the holiday table to flag each requisition as *Dentro do Prazo* (within deadline) or *Fora do Prazo* (outside deadline).
- **Voluntary vs. involuntary terminations.** `Colaboradores[DESC_DESLIGAMENTO]` maps termination reason codes into groups.

---

## Data sources

| Source | Tables |
|---|---|
| Oracle data warehouse (HR system) | Requisitions, planned positions, employees, leaves, jobs, schedules, branches |
| Power Platform Dataflows | Cost centers |
| Embedded tables | Companies, holidays |

---

## DAX highlights

**Average time to hire.** The requisition date is a role-playing date, so the inactive relationship is enabled on demand:

```dax
Media dias contratação =
CALCULATE (
    AVERAGE ( Vagas[DIAS_CONTRATACAO] ),
    USERELATIONSHIP ( Calendario[Data], Vagas[DT_REQUISICAO] )
)
```

**SLA compliance.** Openings closed within the deadline divided by openings worked in the period:

```dax
Pct Finalizadas Dentro do Período (SLA) = DIVIDE ( [Dentro do Prazo], [Vagas Trabalhadas] )
Meta SLA = 0.90
```

**Openings worked.** An opening counts as worked in the period if it was open at any point of the period:

```dax
Vagas Trabalhadas =
CALCULATE (
    [Vagas Geral],
    OR (
        AND ( Vagas[DT_REQUISICAO] <= MAX ( Calendario[Data] ),
              Vagas[DT_ENCERRAMENTO] >= MIN ( Calendario[Data] ) ),
        AND ( Vagas[DT_REQUISICAO] <= MAX ( Calendario[Data] ),
              ISBLANK ( Vagas[DT_ENCERRAMENTO] ) )
    )
)
```

---

## Techniques used

- Role-playing dates (requisition, closing, admission, termination) through inactive relationships and `USERELATIONSHIP`.
- Business-day calculations with `NETWORKDAYS` and a holiday table.
- Field parameters for dynamic breakdowns and a configurable detail table.
- Time intelligence: `TOTALYTD` and `PREVIOUSMONTH`.
- Menu and bookmark-based navigation.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Power Query (M) · DAX · Oracle · Power Platform Dataflows
