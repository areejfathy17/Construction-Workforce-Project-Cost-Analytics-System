# Construction Workforce & Project Cost Analytics System

**An automated labor-cost engine that converts daily attendance into real-time project costing, departmental payroll, and workforce analytics — built entirely on Excel + Power Pivot + DAX.**

---

## 1. The Business Problem

A multi-department construction contractor (Construction, Finishes, Electrical, Plumbing, Stone, Aluminum, Equipment, Carpentry, Paints, Administration) needed to answer three questions every month that were previously handled through disconnected spreadsheets:

1. **What did each project actually cost in labor** — including overtime — this month?
2. **What does each department owe in payroll**, once bonuses, loans, and deductions are applied?
3. **Is the workforce compliant and stable** — sponsorship status, residency validity, absence patterns, headcount by grade and nationality?

The challenge wasn't the math — it was that all three answers had to come from **the same source of truth**, updated daily, without manual re-entry at month-end.

## 2. The Solution

A relational data model where a single daily attendance record simultaneously drives **project costing** and **departmental payroll** — with a DAX layer computing cost, and three purpose-built dashboards consuming one model.

```
Departments → Projects → Employees → Daily Work Logs → Payroll Transactions → Cost & Salary Engine → Dashboards
```

The **Daily Work Log** is the operational core: every row (employee × project × day) carries hours worked, overtime hours, and attendance status, and DAX measures convert that into Normal Cost and OT Cost — which then roll up two ways at once: to the **project** it was worked on, and to the **department** the employee belongs to.

## 3. Why This Is More Than a Payroll Sheet

| Feature | Why it matters |
|---|---|
| **Star-schema data model in Power Pivot** | Five related tables (Master_Employee, tbl_Dept, tbl_Projects, Daily_Work, tbl_Transactions) with proper 1-to-many relationships — not a single flat sheet with VLOOKUPs. |
| **Dual-path cost allocation** | One attendance record feeds both project costing and department payroll without duplicate entry. |
| **Multi-channel payroll disbursement** | Employees are paid via Bank, Cash, or Mobile Wallet — the model reconciles total payroll cost across all three, reflecting real disbursement complexity on a labor-intensive site. |
| **Compliance monitoring built into the employee master** | Residency/permit expiry is tracked with a countdown and conditional-formatting alert, alongside a multi-nationality workforce breakdown — the kind of operational risk tracking that's easy to skip and expensive to ignore. |
| **Bilingual architecture (Arabic / English)** | Every table, label, and dashboard is dual-language by design, not translated after the fact — built for a bilingual operations team. |
| **Three dashboards, one model** | HR Analytics, Payroll Analysis, and Project Cost Analysis are independent views over a single reusable data model, not three separate builds. |

## 4. Data Model

The core of the system — five tables connected through a proper relational structure in Power Pivot:

`[Screenshot: Power Pivot Diagram View — assets/screenshots/01_data_model.png]`

- **Master_Employee** *(dimension)* — one row per employee: pay rates, salary components, job title, nationality, compliance dates, department.
- **tbl_Dept** *(dimension)* — department reference table, bilingual, with department codes used across all fact tables.
- **tbl_Projects** *(dimension)* — project reference table: client, location, timeline, status.
- **Daily_Work** *(fact table)* — the operational core: daily hours, overtime, attendance status, linked to both employee and project.
- **tbl_Transactions** *(fact table)* — bonuses, loans, and deductions, linked to employee.

Relationships: `Master_Employee (1) → Daily_Work (*)`, `tbl_Dept (1) → Daily_Work (*)`, `tbl_Projects (1) → Daily_Work (*)`, `Master_Employee (1) → tbl_Transactions (*)`.

## 5. Dashboards

### HR Analytics
Workforce composition at a glance — headcount, grade distribution, employment status, sponsorship status, department and job-title breakdowns, and salary bands by role.

`[Screenshot: assets/screenshots/02_hr_dashboard.png]`

### Payroll Analysis
Net salary, salary range and average, month-over-month payroll trend, top earners, department-level salary distribution, and disbursement split across Bank / Cash / Mobile Wallet.

`[Screenshot: assets/screenshots/03_payroll_analysis.png]`

### Project Cost Analysis
Total project cost, workforce assigned per project, work days and OT hours, and a full cost breakdown by project **and** by department — the direct output of the daily-work cost engine.

`[Screenshot: assets/screenshots/04_project_cost_dashboard.png]`

## 6. Key DAX Measures

The measures below sit on top of the data model and are what convert raw attendance into decision-ready numbers. Grouped by what they answer:

**Cost Engine**
- `Net Salary` — final take-home pay after allowances, bonuses, loans, and deductions
- `Project Cost` — total labor cost allocated to a project
- `OT Cost` / `Total Work Cost` — overtime and combined labor cost
- `Cost per Worker` — normalized cost for cross-project comparison

**Workforce Metrics**
- `Workers Count` — active headcount by project/department/period
- `Absence Rate` — attendance reliability by employee, project, or department

**Payroll Trend Analysis**
- `Avg Salary`, `Max Salary`, `Min Salary` — payroll distribution
- `Previous Month Salary` / `Payroll Trends` — period-over-period comparison for cost control

*(Formula-level detail available in `docs/dax-measures.md` — populated with exact expressions on request.)*

## 7. System Inputs (Data Entry Layer)

The dashboards are read-only outputs of a structured entry system. The two screens below represent the entry layer's design discipline — bilingual headers, coded IDs, filterable structure — rather than the full table set:

`[Screenshot: Daily Work Log entry screen — assets/screenshots/05_daily_work_entry.png]`
`[Screenshot: Master Employee record screen — assets/screenshots/06_master_employee_entry.png]`

Departments, Projects, and Transactions use the same design pattern (coded IDs, bilingual labels, structured columns) and feed the same model — they're summarized in `docs/data-dictionary.md` rather than repeated here.

## 8. Tech Stack

`Excel` · `Power Pivot` · `DAX` · `Power Query (data prep)` · Relational data modeling · Bilingual (AR/EN) UI design

## 9. Repository Structure

```
Construction-Workforce-Cost-Analytics/
├── README.md
├── assets/
│   └── screenshots/
│       ├── 01_data_model.png
│       ├── 02_hr_dashboard.png
│       ├── 03_payroll_analysis.png
│       ├── 04_project_cost_dashboard.png
│       ├── 05_daily_work_entry.png
│       └── 06_master_employee_entry.png
├── docs/
│   ├── data-dictionary.md      # table/field reference for all 5 tables
│   ├── dax-measures.md         # full measure list with DAX expressions
│   └── business-logic.md       # cost allocation & payroll calculation rules
└── HR_Labor_Cost_System.xlsx   # sanitized/sample workbook
```

## 10. Notes on This Repository

This project was built for a real operating context and has been sanitized for portfolio use — employee names, IDs, and figures shown in screenshots are illustrative. The data model, relationships, and DAX logic reflect the actual system design.
