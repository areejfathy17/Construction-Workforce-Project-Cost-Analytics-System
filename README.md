# Construction Workforce & Project Cost Analytics System

**A relational payroll and project-costing engine for a multi-department construction contractor — built on Excel, Power Pivot, and DAX to turn daily attendance into automated payroll, project cost, and workforce intelligence.**

---

## 1. Business Problem

A construction contractor operating across ten departments — Construction, Finishes, Electrical, Plumbing, Stone, Aluminum, Equipment, Carpentry, Paints, and Administration — needed to answer three interlocked questions every pay cycle:

- What did each **project** actually cost in labor, including overtime, this period?
- What does each **department** owe in payroll once bonuses, loans, and deductions are applied?
- Is the **workforce** stable and compliant — attendance, headcount, absence patterns, cost per worker?

These weren't three separate reporting problems. They were one problem viewed from three angles, and the existing process answered them with disconnected spreadsheets rebuilt by hand every month — a process that doesn't scale and doesn't reconcile.

## 2. Solution Overview

A single relational data model where **one daily attendance record simultaneously drives project costing and departmental payroll**, with a DAX calculation layer computing cost and salary automatically, and three purpose-built dashboards reading from that one model.

```
Departments → Projects → Employees → Daily Work Logs → Payroll Transactions → Salary & Cost Engine → Executive Dashboards
```

The **Daily Work log is the operational core** of the system. Every record — employee, project, work days, work hours, OT hours, attendance status — feeds forward into payroll, overtime cost, department expense, and project labor cost, with no duplicate entry and no manual reconciliation between the three views.

## 3. Architecture

The system is not a manual spreadsheet — it's a controlled data-entry architecture sitting on top of a relational model:

- **Master data tables** (Employees, Departments, Projects) are the single source of truth for every lookup used downstream.
- **Validation lists and controlled dropdowns** restrict entry for Departments, Projects, Transaction Types, Payment Methods, and Employee Status — an operator selects an employee ID and every related field (name, department, pay rate, project) populates automatically.
- **Hidden reference tables** back the dropdowns without cluttering the entry sheets.

This matters because it removes the two biggest failure modes of spreadsheet-based systems: inconsistent free-text entry and broken lookups — the model self-corrects at the point of entry instead of during reporting.

## 4. Data Model

Five tables, related through a proper star-schema structure in Power Pivot — not a flat sheet with VLOOKUPs:

[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/power-pivot-data-model.png]

| Table | Role | Key Relationship |
|---|---|---|
| `Master_Employee` | Employee dimension — pay rates, salary components, compliance data | `1 → *` to `Daily_Work`, `1 → *` to `tbl_Transactions` |
| `tbl_Dept` | Department dimension | `1 → *` to `Master_Employee` |
| `tbl_Projects` | Project dimension — client, location, timeline, status | `1 → *` to `Daily_Work` |
| `Daily_Work` | **Fact table** — the operational core: attendance, hours, OT, status | Many-to-one to `Master_Employee` and `tbl_Projects` |
| `tbl_Transactions` | **Fact table** — bonuses, loans, deductions, additions | Many-to-one to `Master_Employee` |

Two fact tables sharing the same employee dimension is what allows payroll (via `tbl_Transactions`) and project cost (via `Daily_Work`) to stay reconciled without ever touching each other directly.

## 5. Core Modules

1. **Employee Master Database** — rates, salary structure, job title, nationality, compliance/residency status
2. **Department Master Database** — bilingual department reference, coded IDs
3. **Project Master Database** — client, location, timeline, status
4. **Daily Work Tracking System** — the attendance and hours engine driving all downstream cost
5. **Payroll Transactions System** — bonuses, loans, deductions, additions per employee
6. **Power Pivot Data Model** — the relational layer connecting all of the above
7. **DAX Calculation Engine** — 20+ measures computing salary, cost, and workforce KPIs in real time
8. **Executive Dashboards** — HR, Payroll, and Project Cost views over the single model

## 6. DAX Showcase

The measures below are the actual calculation logic behind the system, grouped by function.

**Payroll Engine**

```DAX
Net Salary =
[Total Work Cost]
+ [Total Bonus]
+ [Total Addition]
- [Total Deductions]
- [Total Loans]
```

```DAX
Avg Salary =
AVERAGEX(
    FILTER(
        VALUES(Master_Employee[Emp_ID]),
        [Net Salary] > 0
    ),
    [Net Salary]
)
```

```DAX
Max Salary =
MAXX(VALUES(Master_Employee[Emp_ID]), [Net Salary])
```

```DAX
Min Salary =
MINX(
    FILTER(VALUES(Master_Employee[Emp_ID]), [Net Salary] > 0),
    [Net Salary]
)
```

`Net Salary` is the assembly point of the whole engine — it pulls together work-based cost and every transaction type into one figure, computed per employee, per period, on the fly.

**Project & Cost Engine**

```DAX
Project Cost =
SUMX(
    Daily_Work,
    Daily_Work[Days_Work] * RELATED(Master_Employee[Daily_Rate])
    + Daily_Work[OT_Hrs] * RELATED(Master_Employee[OT_Hr_Rate])
)
```

```DAX
OT Cost =
SUMX(
    Daily_Work,
    Daily_Work[OT_Hrs] * RELATED(Master_Employee[OT_Hr_Rate])
)
```

```DAX
Total Work Cost = [Normal Cost] + [OT Cost]
```

```DAX
Cost per Worker = DIVIDE([Project Cost], [Workers Count])
```

`Project Cost` is the measure doing the real work: it iterates every attendance row and pulls each employee's individual daily and overtime rate via `RELATED`, meaning cost is computed at the person-day level, not a department-wide average. This is what lets the same fact table answer both "what did this project cost" and "what does this department owe."

**Workforce Analytics**

```DAX
Employee Count = DISTINCTCOUNT(Master_Employee[Emp_ID])
Workers Count = DISTINCTCOUNT(Daily_Work[Emp_ID])

Absent Days =
CALCULATE(COUNTROWS(Daily_Work), Daily_Work[Emp_Status] = "Absent")

Absence Rate =
DIVIDE(SUM(Daily_Work[Status_Code]), COUNTROWS(Daily_Work))

Total Work Days = SUM(Daily_Work[Days_Work])
Total OT Hours = SUM(Daily_Work[OT_Hrs])
```

**Payroll Transactions**

```DAX
Total Bonus =
CALCULATE(SUM(tbl_Transactions[Amount]), tbl_Transactions[Type_Code] = "Bo_01")

Total Loans =
CALCULATE(SUM(tbl_Transactions[Amount]), tbl_Transactions[Type_Code] = "Lo_01")

Total Deductions =
CALCULATE(SUM(tbl_Transactions[Amount]), tbl_Transactions[Type_Code] = "De_01")

Total Addition =
CALCULATE(SUM(tbl_Transactions[Amount]), tbl_Transactions[Type_Code] = "Ad_01")
```

**Trend Analysis**

```DAX
Previous Month Salary =
CALCULATE([Net Salary], DATEADD(Daily_Work[Month], -1, MONTH))

Difference = [Selected Month Salary] - [Previous Month Salary]
```

`DATEADD` over the attendance calendar gives month-over-month payroll trend without a separate date table maintained by hand — the model's own Month column drives the time intelligence.

## 7. Dashboard Gallery

Three dashboards, one model — each answering a different question for a different audience.

**HR Analytics** — workforce composition: headcount, grade distribution, employment status, department and job-title breakdown, salary bands by role.

[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/HR_Dashboard.png]

**Payroll Analysis** — net salary, salary range, month-over-month trend (via `Previous Month Salary` / `Difference`), top earners, department-level salary distribution, disbursement channel split.

[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/Payroll%20Analysis%20Dashboard.png]

**Project Cost Analysis** — total project cost, workforce assigned, work days and OT hours, cost broken down by project and by department simultaneously — the direct output of the `Project Cost` and `Cost per Worker` measures.

[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/Project_Cost_Dashboard%20.png]

**Data entry & validation layer** — shown as supporting evidence of the controlled-entry architecture (Section 3), not as standalone reporting:

[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/employee-master-data-entry.png]
[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/daily-work-tracking.png]
[https://github.com/areejfathy17/Construction-Workforce-Project-Cost-Analytics-System/blob/main/payroll-transactions-entry.png]

## 8. Technical Highlights

- Relational star-schema model in Power Pivot — five related tables, not flat-sheet reporting
- 20+ DAX measures spanning payroll calculation, cost allocation, workforce KPIs, and time intelligence
- `RELATED()`-driven cost allocation at the person-day level, enabling accurate multi-dimensional rollups (project *and* department) from one fact table
- `DATEADD` time intelligence over a native attendance calendar for month-over-month trend, with no manually maintained date table
- Controlled data-entry architecture (validation lists, dropdowns, hidden reference tables) that prevents bad data at the source rather than cleaning it downstream
- Three independent executive dashboards consuming a single reusable model

## 9. Challenges Solved

- **Cost duplication risk** — eliminated by allocating labor cost from a single fact table (`Daily_Work`) to both project and department views, rather than maintaining separate cost sheets that could drift out of sync.
- **Payroll assembly complexity** — `Net Salary` had to combine work-based cost with four independent transaction types (bonus, addition, deduction, loan) without hardcoding transaction logic into the payroll sheet; solved by filtering `tbl_Transactions` by `Type_Code` inside dedicated measures.
- **Manual month-end trend building** — replaced with `DATEADD`-based time intelligence, so month-over-month payroll comparison updates automatically as new attendance data is added.
- **Data entry error risk** — solved architecturally, through validation lists and auto-populated lookups, rather than through downstream data cleaning.

## 10. Business Impact

- Project and departmental labor cost are available on demand, reconciled from the same source, instead of being rebuilt manually each cycle.
- Payroll calculation — including overtime, bonuses, loans, and deductions — is automated end-to-end from attendance data.
- Workforce risk (absence patterns, headcount shifts, cost per worker) is visible continuously rather than surfacing only at month-end.
- The model is built to extend: adding a department, project, or transaction type requires a new row in a master table, not a rebuild of the reporting layer.


This project reflects a real operating system built for a live business context, sanitized for portfolio use — names, IDs, and figures in the screenshots are illustrative; the data model, relationships, and DAX logic reflect the actual system design.
