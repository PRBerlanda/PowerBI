# HR – Epidemiological Profile (Health Questionnaires)

Power BI report that consolidates the **occupational health questionnaires** answered by employees during medical appointments, together with the results of **occupational lab exams**. The goal is to give the occupational health team an epidemiological view of the workforce: lifestyle habits, chronic conditions, medication use, psychosocial risk factors and exam results, broken down by company, cost center, job and age range.

📄 **Report preview:** [Epidemiological Profile.pdf](Epidemiological%20Profile.pdf)

> **About the data:** this portfolio version runs on **fully anonymized, fictitious data**. Names of employees, dependents, managers and recruiters, tax IDs, e-mails, phone numbers and addresses were replaced with synthetic values generated with the Python *Faker* library (pt_BR). Company, branch, cost center and site names were replaced with fictitious ones (for example, *Grupo Mendonça*). Registration numbers and internal codes were renumbered consistently across tables, monetary values were scaled by a random factor, sensitive attributes were shuffled between records, and the period was limited to **January 2024 – December 2025**. The figures do not represent any real company, and the source files are not included in this repository.

---

## Report pages

All pages share the same filter bar (cost center, location, appointment type, questionnaire, company, year and month) and a side navigation menu.

| Page | Content |
|---|---|
| **Overview** (*Visão Geral*) | Total appointments and people interviewed, fitness for duty (fit / unfit), monthly trend, and breakdown by company, job, appointment category (admission, periodic, return to work, dismissal…), BMI class and age range. |
| **Central Occupational Physician** (*Central*) | Answers from the central medical team: surgeries, work accidents, disorders, alcohol / drug / tobacco use, cholesterol, heart disease, blood pressure, diabetes, physical activity, children and pregnancy history. |
| **Health Data** (*Dados de Saúde*) | Side-by-side comparison of the two questionnaire providers (central team vs. outsourced clinic) for alcohol, drugs, tobacco, physical activity, social security leave, work accidents, PwD status and second job. |
| **Medical History** (*Histórico Médico*) | Previous surgeries (abdominal, cosmetic, eye, orthopedic), fractures by body region and family history (cancer, diabetes, heart disease, hypertension). |
| **Medication Use** (*Uso de Medicamentos*) | Continuous medication, antidepressants / anxiolytics, antihypertensives, anti-inflammatories, thyroid medication, insulin, supplements and vitamins. |
| **Comorbidities** (*Comorbidades*) | Allergy, anxiety, cancer, depression, diabetes, heart disease, epilepsy, hypertension, obesity, rheumatism and rhinitis, each with its treatment status. |
| **Psychosocial Assessment** (*Avaliação Psicossocial*) | Clear responsibilities, sufficient breaks, group leisure, unreachable deadlines, positive feedback, unusual tiredness, sleep hours and emotionally demanding work. |
| **Exams** (*Exames*) | Exams performed, normal and altered results, % altered by exam type (clinical, stool culture, CBC, parasitology, VDRL…), split by company and unit. |

---

## Semantic model

| Item | Count |
|---|---|
| Tables | 24 (3 field parameters) |
| Measures | 16 |
| Relationships | 14 |
| Report pages / visuals | 8 / 337 |

**Modeling approach.** Questionnaires arrive in a long (key/value) format: one row per *person × date × question × answer*. Two sources (`QUESTIONARIO_CENTRAL` and `QUESTIONARIO_FARIAS`) are standardized in Power Query and appended into a single fact table, `QUESTIONARIOS`. The BMI is derived in helper queries (`IMC_CENTRAL`, `IMC_FARIAS`) and classified by `DESC_IMC`. Each visual filters the fact table to one question (`PERGUNTAS` / `BASE_PERGUNTAS`), so new questions can be added without changing the model.

Supporting dimensions: `COLABORADORES` (employees), `CARGO` and `HISTORICO_CARGOS` (job at the date of the appointment, built as a DAX calculated table), `CENTRO_CUSTO` (cost center), `EMPRESA` (company), `LOCAL` (location), `FAIXA_ETARIA` (age range), `TIPO_RELATORIO` (report type), `TIPO_QUESTIONARIO` (questionnaire type) and a `DATA` calendar. The exams are a separate fact table (`EXAMES`) loaded from an Excel file on SharePoint.

---

## Data sources

Original sources in production:

| Source | Tables |
|---|---|
| Oracle data warehouse | Jobs, companies |
| Power Platform Dataflows | Employees, cost centers, locations, job history, central questionnaire |
| SharePoint (Excel) | Outsourced clinic questionnaire, lab exams |
| Embedded tables | Questions, BMI classes, age ranges, report / questionnaire types |

**Portfolio version.** All 19 import queries now read the anonymized CSV files from a local folder defined by the `PastaBaseDados` parameter (Power Query `Csv.Document`, pt-BR culture). The original connection strings were removed from the project.

---

## DAX highlights

**Appointments.** A person can answer several questionnaires, so an appointment is a distinct combination of *person × date × questionnaire*:

```dax
Atendimentos =
COUNTX (
    SUMMARIZECOLUMNS (
        QUESTIONARIOS[CPF],
        QUESTIONARIOS[DATA_QUESTIONARIO],
        QUESTIONARIOS[COD_QUESTIONARIO],
        "contagem", COUNTROWS ( QUESTIONARIOS )
    ),
    [contagem]
)
```

**Share of altered exams:**

```dax
PCT Exames Alterados = DIVIDE ( [Soma de Exames Alterados], [Soma de Exames] )
```

---

## Techniques used

- Unpivoted questionnaire model (EAV pattern) that unifies two providers in one fact table.
- Power Query append and standardization of heterogeneous sources.
- Field parameters to switch the breakdown dimension (company / BMI / age range; unit / exam / group).
- Button and side-menu navigation between thematic pages.
- Saved as **PBIP** (TMDL + PBIR) for source control.

## Tech stack

Power BI Desktop · Power Query (M) · DAX · Oracle · Power Platform Dataflows · SharePoint / Excel
