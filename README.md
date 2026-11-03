# Power BI Portfolio – Paulo Berlanda

Welcome! This repository gathers **Business Intelligence projects I developed with Power BI** during my career, mostly focused on **Human Resources (People Analytics)**, plus a public-data **E-commerce** case study.

Each project lives in **its own branch**. Switch branches with the selector at the top of the file list, or use the links below. Every branch has a `README.md` describing the report pages, semantic model, data sources, DAX highlights and techniques used, along with a **PDF export** of the report so you can see it without opening Power BI.

---

## 📂 What you will find

### Human Resources (corporate projects)

These applications were built for a large Brazilian company over about two years. In total there were **6 applications, more than 30 dashboards, 800+ DAX measures and 110+ tables**, covering HR and part of Controllership.

| Branch | Project | Highlights |
|---|---|---|
| [`HR-People-Analytics`](https://github.com/PRBerlanda/PowerBI/tree/HR-People-Analytics) | **People Analytics (Labor Relations)**: the flagship HR app | 41 pages, 122 tables, 761 measures, RLS by management hierarchy, calculation groups, severance calculator, model published in Microsoft Fabric |
| [`HR-Benefits-and-Remuneration`](https://github.com/PRBerlanda/PowerBI/tree/HR-Benefits-and-Remuneration) | **Benefits & Remuneration** | Health / dental plan adoption and cost, award programs, salary increases |
| [`HR-Recruitment-and-Selection`](https://github.com/PRBerlanda/PowerBI/tree/HR-Recruitment-and-Selection) | **Recruitment & Selection** | Job openings pipeline, hiring SLA in business days, time to hire, turnover |
| [`HR-Epidemiological-Profile`](https://github.com/PRBerlanda/PowerBI/tree/HR-Epidemiological-Profile) | **Epidemiological Profile** | Occupational health questionnaires and lab exams: habits, comorbidities, medication, psychosocial risk |
| [`HR-SESMT-EHS`](https://github.com/PRBerlanda/PowerBI/tree/HR-SESMT-EHS) | **SESMT (EHS): Accidents & Absences** | Work accidents, leaves, medical certificates and ICD analysis; composite model over the shared Fabric dataset |
| [`HR-SESMT-Safety-Technicians`](https://github.com/PRBerlanda/PowerBI/tree/HR-SESMT-Safety-Technicians) | **SESMT (EHS): Safety Technicians** | Route coverage, visit compliance and checklist scores per technician and unit |

> **About the data:** the figures shown in the HR PDFs **do not represent the company's actual numbers**. The data was filtered and is outdated. Some applications were developed directly in Microsoft Fabric workspaces, so not everything is available here.

### Case study (public data)

| Branch | Project | Highlights |
|---|---|---|
| [`PRBerlanda-Commerce`](https://github.com/PRBerlanda/PowerBI/tree/PRBerlanda-Commerce) | **E-commerce Sales & Operations** | About 100k orders from a public Brazilian marketplace dataset; executive, logistics, customer and product views; 32 measures validated by 176 automated tests. Includes the `.pbix` file. |

---

## 🛠️ Skills demonstrated

- **Data modeling:** star schemas, role-playing dates, slowly changing assignments, daily snapshot tables, composite models (DirectQuery for Power BI semantic models).
- **DAX:** time intelligence, calculation groups, field parameters, point-in-time headcount, retention, turnover, absenteeism, business-day SLAs, Brazilian labor-law calculations (CLT severance, FGTS, 13th salary, vacation).
- **Security & governance:** dynamic row-level security by management hierarchy; shared semantic models in Microsoft Fabric; projects saved as **PBIP** (TMDL + PBIR) for version control.
- **Data integration:** Oracle, Power Platform Dataflows, SharePoint / Excel and Power Query (M).
- **Report design & UX:** menu navigation, bookmarks, drill-through, tooltips, KPI cards with period comparisons, custom themes.

---

## 📬 Contact

Paulo Berlanda, Data Analyst / Power BI Developer.
Open to freelance projects: semantic modeling, reports and Power BI consulting.
