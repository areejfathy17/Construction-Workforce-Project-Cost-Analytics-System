# Construction Workforce & Project Cost Analytics System

**An end-to-end workforce, payroll, and project cost management system built with Excel, Power Pivot, and DAX, transforming daily attendance records into automated payroll calculations, labor cost allocation, and executive analytics.**

---

## Project Snapshot

| Metric | Value |
|----------|----------|
| Departments Managed | 10 |
| Connected Tables | 5 |
| DAX Measures | 20+ |
| Executive Dashboards | 3 |
| Business Functions | Workforce, Payroll, Project Costing |
| Platform | Excel + Power Pivot + DAX |

---

# Dashboard Gallery

## HR Analytics Dashboard

![HR Dashboard](HR_Dashboard.png)

---

## Payroll Analysis Dashboard

![Payroll Dashboard](Payroll%20Analysis%20Dashboard.png)

---

## Project Cost Analysis Dashboard

![Project Cost Dashboard](Project_Cost_Dashboard.png)

---

# Data Model

The solution is built on a centralized relational Power Pivot model that connects workforce, projects, attendance, payroll transactions, and cost calculations into a single source of truth.

![Power Pivot Data Model](power-pivot-data-model.png)

### Core Tables

| Table | Purpose |
|---------|---------|
| Master_Employee | Employee master records, salary structure, rates, job details |
| tbl_Dept | Department hierarchy and reference data |
| tbl_Projects | Project information, client, location, status |
| Daily_Work | Attendance, working days, overtime hours |
| tbl_Transactions | Bonuses, deductions, loans, additions |

### Key Relationships

```text
Departments
      │
      ▼
Master_Employee
      │
      ├──────────────► Transactions
      │
      ▼
Daily_Work ◄──────── Projects
```

The **Daily_Work** table acts as the operational core of the system, driving both payroll calculations and project labor costing.

---

# Business Problem

The company managed employees, projects, attendance, overtime, payroll adjustments, and project costs through disconnected spreadsheets.

This created challenges in:

- Manual payroll calculations
- Project cost estimation
- Workforce performance monitoring
- Tracking bonuses, deductions, and loans
- Producing management reports across departments

---

# Solution Overview

I designed and developed a complete operational management system that integrates:

- Employee Management
- Department Management
- Project Management
- Daily Workforce Tracking
- Payroll Processing
- Labor Cost Allocation
- Executive Reporting

All modules are connected through a centralized Power Pivot model and automated DAX calculation engine.

---

# System Architecture

```text
Departments
      ↓
Projects
      ↓
Employees
      ↓
Daily Work Logs
      ↓
Payroll Transactions
      ↓
Salary & Cost Engine
      ↓
Executive Dashboards
```

The system was designed around controlled data-entry workflows rather than manual spreadsheet editing.

---

# Data Entry & Automation Layer

## Employee Master

![Employee Master](employee-master-data-entry.png)

Centralized employee database containing:

- Employee information
- Department assignment
- Salary structure
- Daily rates
- OT rates
- Job title
- Employment status

---

## Daily Work Tracking

![Daily Work](daily-work-tracking.png)

The operational core of the system.

Tracks:

- Attendance
- Work days
- Work hours
- Overtime hours
- Project allocation

Used to automatically calculate:

- Employee salaries
- Overtime payments
- Project labor costs
- Department expenses

---

## Payroll Transactions

![Transactions](payroll-transactions-entry.png)

Tracks:

- Bonuses
- Loans
- Deductions
- Additional payments

All transactions automatically impact final payroll calculations.

---

# DAX Calculation Engine

The business logic is powered by 20+ DAX measures built on top of the Power Pivot model.

### Net Salary

```DAX
Net Salary =
[Total Work Cost]
+ [Total Bonus]
+ [Total Addition]
- [Total Deductions]
- [Total Loans]
```

### Project Cost

```DAX
Project Cost =
SUMX(
    Daily_Work,
    Daily_Work[Days_Work] *
    RELATED(Master_Employee[Daily_Rate])
    +
    Daily_Work[OT_Hrs] *
    RELATED(Master_Employee[OT_Hr_Rate])
)
```

### OT Cost

```DAX
OT Cost =
SUMX(
    Daily_Work,
    Daily_Work[OT_Hrs] *
    RELATED(Master_Employee[OT_Hr_Rate])
)
```

### Previous Month Salary

```DAX
Previous Month Salary =
CALCULATE(
    [Net Salary],
    DATEADD(
        Daily_Work[Month],
        -1,
        MONTH
    )
)
```

### Additional KPIs

- Employee Count
- Workers Count
- Avg Salary
- Max Salary
- Min Salary
- Absence Rate
- Cost per Worker
- Total Work Days
- Total OT Hours
- Payroll Trends
- Bonus Analysis
- Deduction Analysis

---

# Technical Highlights

- Relational Power Pivot Data Model
- Workforce & Payroll Automation
- Project Cost Allocation Engine
- Advanced DAX Calculations
- Dynamic KPI Reporting
- Data Validation Architecture
- Lookup-driven Data Entry
- Multi-Dashboard Reporting
- Time Intelligence Analysis

---

# Challenges Solved

### Payroll Automation

Eliminated manual salary calculations by combining attendance, overtime, bonuses, deductions, and loans into a centralized payroll engine.

### Project Cost Allocation

Automatically allocated workforce costs to projects using actual attendance and overtime records.

### Data Quality

Implemented validation lists and controlled dropdowns to reduce data-entry errors and maintain consistency across departments.

### Reporting Efficiency

Replaced manual reporting processes with real-time executive dashboards powered by a single data model.

---

# Business Impact

- Automated payroll calculations
- Automated labor cost allocation
- Reduced manual reporting effort
- Improved payroll accuracy
- Centralized workforce and project data
- Enabled real-time workforce analytics
- Improved management visibility across projects and departments

---

## Tech Stack

**Microsoft Excel • Power Pivot • DAX • Power Query • Data Modeling • Pivot Tables • Interactive Dashboard Design**
