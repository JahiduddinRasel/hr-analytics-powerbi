
# HR Analytics — Employee Attrition Dashboard (Power BI)

An end-to-end Power BI project analyzing employee attrition using the IBM HR Analytics dataset (1,470 employees, 35 attributes). The project covers data cleaning, star-schema modeling, DAX measures, and an interactive two-page executive dashboard.

---

## 📊 Project Objective

Identify the key drivers behind employee attrition and give HR leadership a clear, interactive view of:

- Who is leaving (demographics, department, role)
- Why they are leaving (satisfaction, work-life balance, overtime, tenure)
- Where the highest-risk areas are (department & job role)

---

## 🗂️ Dataset

| Property | Value |
|---|---|
| **Source** | IBM HR Analytics Employee Attrition & Performance |
|**Link**   |  https://www.kaggle.com/code/mostafaaazaza/ibm-hr-analytics-employee-attrition-performance?select=WA_Fn-UseC_-HR-Employee-Attrition.csv|
| **Rows** | 1,470 employees |
| **Columns** | 35 |
| **Missing values** | 0 |
| **Duplicates** | 0 |
| **Format** | CSV (raw), transformed inside Power Query |

---

## 🛠️ Tools & Skills Used

| Area | Tools / Techniques |
|---|---|
| Data cleaning | Power Query (M) |
| Modeling | Star schema, role-playing dimensions |
| Calculations | DAX (measures, calculated columns) |
| Visualization | Power BI Desktop |
| Design | Dark theme, custom color palette, navigation buttons |

---

## 🧱 Data Model — Star Schema

The raw flat file was decomposed into a clean star schema:

```
                       dim_employee
                             │
   dim_env_satisfaction      │      dim_job
   dim_job_satisfaction ──── fact_hr ──── dim_education
   dim_rel_satisfaction      │      dim_work_life_balance
   dim_job_involvement       │
                       (measures)
```

### Fact Table
**`fact_hr`** — Contains foreign keys and numeric metrics only.

| Column | Type |
|---|---|
| employee_number | FK → dim_employee |
| job_role_id | FK → dim_job |
| education | FK → dim_education |
| environment_satisfaction | FK → dim_env_satisfaction |
| job_satisfaction | FK → dim_job_satisfaction |
| relationship_satisfaction | FK → dim_rel_satisfaction |
| job_involvement | FK → dim_job_involvement |
| work_life_balance | FK → dim_work_life_balance |
| attrition | Text (Yes / No) |
| monthly_income, daily_rate, hourly_rate, monthly_rate | Numeric |
| percent_salary_hike, performance_rating | Numeric |
| years_at_company, years_in_current_role, years_since_last_promotion, years_with_curr_manager | Numeric |
| total_working_years, training_times_last_year, num_companies_worked, stock_option_level | Numeric |

### Dimension Tables

| Table | Grain | Key |
|---|---|---|
| `dim_employee` | 1 row per employee | employee_number |
| `dim_job` | 1 row per unique job combination | job_role_id |
| `dim_education` | 5 education levels | education_id |
| `dim_env_satisfaction` | 4 satisfaction levels | satisfaction_id |
| `dim_job_satisfaction` | 4 satisfaction levels | satisfaction_id |
| `dim_rel_satisfaction` | 4 satisfaction levels | satisfaction_id |
| `dim_job_involvement` | 4 satisfaction levels | satisfaction_id |
| `dim_work_life_balance` | 4 balance levels | balance_id |

### Relationships

| From (1) | To (*) | Cross-filter |
|---|---|---|
| dim_employee[employee_number] | fact_hr[employee_number] | Single |
| dim_job[job_role_id] | fact_hr[job_role_id] | Single |
| dim_education[education_id] | fact_hr[education] | Single |
| dim_env_satisfaction[satisfaction_id] | fact_hr[environment_satisfaction] | Single |
| dim_job_satisfaction[satisfaction_id] | fact_hr[job_satisfaction] | Single |
| dim_rel_satisfaction[satisfaction_id] | fact_hr[relationship_satisfaction] | Single |
| dim_job_involvement[satisfaction_id] | fact_hr[job_involvement] | Single |
| dim_work_life_balance[balance_id] | fact_hr[work_life_balance] | Single |

> All relationships are single-direction (Dimension → Fact) to prevent circular filters and improve performance.

---

## 🔧 Power Query Transformation Steps

1. **Automated Folder Ingestion :** 
   - Connected Power Query directly to a raw data source folder (`/data/`) using the **Folder Connector**.
   - Configured automated combining logic so any future monthly HR data exports dropped into the folder are automatically parsed, transformed, and appended into the data model without manual intervention.
2. Renamed all columns to `snake_case` for consistency.
3. Updated data types for every column.
4. Removed `employee_count` column (constant value `1`).
5. Created `dim_job` by duplicating the raw table, keeping only job-related columns, removing duplicates, and adding an Index column as `job_role_id`.
6. Merged `dim_job` into `fact_hr` on 4 columns, expanded `job_role_id`, removed original job columns from `fact_hr`.
7. Created `dim_employee` by referencing the raw table and keeping personal demographic columns.
8. Created lookup dimensions using **Enter Data**:
   - `dim_education` (1 → Below College, 2 → College, 3 → Bachelor, 4 → Master, 5 → Doctor)
   - `dim_satisfaction` (1 → Low, 2 → Medium, 3 → High, 4 → Very High)
   - `dim_work_life_balance` (1 → Bad, 2 → Good, 3 → Better, 4 → Best)
9. Handled the role-playing dimension by duplicating `dim_satisfaction` via DAX New Table to create:
   - `dim_env_satisfaction`
   - `dim_job_satisfaction`
   - `dim_rel_satisfaction`
   - `dim_job_involvement`
10. Applied **Close & Apply**.

### Key Decisions

- Used **Duplicate** only for `dim_job` (which needs to merge back into the fact table) to avoid circular dependency errors in Power Query.
- Used **Reference** for `dim_employee` — lighter and faster.
- Chose **separate role-playing dims** instead of `USERELATIONSHIP` in DAX — simpler and reduces measure complexity.

---

## 🧮 DAX Measures

All measures live in a dedicated table named `_measures`.

### Core KPIs
| Measure | Description |
|---|---|
| `total_employees` | Total count of rows in fact_hr |
| `attrition_count` | Count of employees who left (Yes) |
| `active_employees` | Count of employees who stayed (No) |
| `attrition_rate` | attrition_count / total_employees |

### Compensation & Tenure
| Measure | Description |
|---|---|
| `avg_monthly_income` | Average monthly income |
| `avg_salary_hike` | Average percent salary hike |
| `avg_age` | Average employee age |
| `avg_years_at_company` | Average tenure at company |
| `avg_years_since_promotion` | Average years since last promotion |

### Overtime Impact
| Measure | Description |
|---|---|
| `overtime_attrition_rate` | Attrition rate for employees who work overtime |

### Calculated Columns
| Table | Column | Description |
|---|---|---|
| dim_employee | `age_group` | Bins age into Under 25, 25-34, 35-44, 45-54, 55+ |
| dim_employee | `age_group_sort` | Numeric sort helper for age_group |

### Sort-By-Column Applied

| Column | Sorted By |
|---|---|
| dim_employee[age_group] | age_group_sort |
| dim_work_life_balance[balance_level] | balance_id |
| dim_satisfaction[satisfaction_level] (all copies) | satisfaction_id |
| dim_education[education_level] | education_id |

---

## 🧭 Report Structure

### Page 1 — Executive Overview
- 5 KPI cards: Total Employees, Attrition Count, Attrition Rate, Active Employees, Avg Monthly Income
- Attrition by Department (Donut)
- Attrition by Age Group (Bar)
- Attrition by Job Role (Horizontal Bar)
- Attrition by Education Level (Column)
- Attrition by Gender (Donut)
- Avg Income by Job Role (Treemap)

### Page 2 — Detailed Analysis
- 5 KPI cards: Avg Age, Avg Years at Company, Avg Salary Hike, Overtime Attrition Rate, Avg Years Since Promotion
- Environment Satisfaction Impact (Stacked Bar)
- Job Satisfaction Impact (Stacked Bar)
- Attrition Rate by Work-Life Balance (Column)
- Attrition Trend by Years at Company (Area)
- Monthly Income vs Tenure (Scatter)
- Department & Role Breakdown (Matrix with conditional formatting)

### Interactivity
- Slicers (synced across both pages): Department, Education Field, Overtime
- Navigation buttons: Overview ↔ Detailed Analysis

---

## 💡 Key Insights

- Overall attrition rate: **16.1%**
- Highest-attrition roles: **Sales Representative (39.8%)**, **Laboratory Technician (23.9%)**, **Human Resources (23.1%)**
- Employees working **overtime** show significantly higher attrition (~30%)
- Employees rating **work-life balance as "Bad"** show attrition above **31%**
- Attrition is heavily concentrated in employees with **less than 2 years** at the company
- Lower monthly income + shorter tenure = highest attrition cluster

---

## 🎯 Business Recommendations

1. Redesign onboarding and career-path support for the first 2 years of tenure.
2. Investigate compensation structure for Sales Representatives and Lab Technicians.
3. Review overtime workload policies; consider caps and rotation.
4. Create early-warning HR alerts for employees with low satisfaction + overtime + short tenure.
5. Improve promotion cadence — average years since last promotion is over 2 years for many employees.

---

## 📁 Repository Structure

```
hr-analytics-powerbi
├── README.md
├── HR_Analytics_Dashboard.pbix
├── hr_employee_attrition.csv
├── page1_overview.png
└── page2_detailed.png
```

---

## 🚀 How to Reproduce

1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/hr-analytics-powerbi.git
   ```
2. Open `HR_Analytics_Dashboard.pbix` in **Power BI Desktop**.
3. If prompted, point the data source to `hr_employee_attrition.csv`.
4. Click **Refresh** to reload the model.

---

## 🧾 Data Dictionary (Key Fields)

| Field | Description |
|---|---|
| attrition | Whether the employee left (Yes / No) |
| education | 1 = Below College, 2 = College, 3 = Bachelor, 4 = Master, 5 = Doctor |
| environment_satisfaction | 1 = Low, 2 = Medium, 3 = High, 4 = Very High |
| job_satisfaction | Same scale as above |
| relationship_satisfaction | Same scale as above |
| job_involvement | Same scale as above |
| work_life_balance | 1 = Bad, 2 = Good, 3 = Better, 4 = Best |
| performance_rating | 1 = Low, 2 = Good, 3 = Excellent, 4 = Outstanding |
| over_time | Whether employee works overtime (Yes / No) |
| monthly_income | Monthly salary in USD |
| years_at_company | Total years at current company |

---

## 👤 Author

**Your Name**
Data Analyst | Power BI • DAX • Data Modeling
[LinkedIn](https://linkedin.com/in/your-handle) · [Portfolio](https://your-portfolio.com)

---

## 🏷️ Tags

`Power BI` `DAX` `Data Modeling` `Star Schema` `HR Analytics` `Employee Attrition` `Business Intelligence`
