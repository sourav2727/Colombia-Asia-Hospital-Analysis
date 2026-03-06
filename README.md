 #🏥 Colombia Hospital — Patient Analytics Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-Live%20Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Advanced%20Queries-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)

> A full-stack data analytics project covering data engineering (MySQL), business intelligence (Power BI), and advanced SQL — built to extract operational and clinical insights from hospital patient visit data.

---

## 📌 Project Overview

This project analyzes real-world-style hospital data from **Colombia Hospital**, covering **9,216 patient visits** across multiple departments. The goal was to clean, model, and visualize the data to help hospital management make informed decisions around patient flow, doctor performance, wait times, revenue, and satisfaction.

The project was built end-to-end:

- Raw data loaded into **MySQL** via structured schema
- Business questions answered via **DAX measures in Power BI**
- Advanced analytical queries written in **MySQL** (CTEs, Window Functions, JOINs)
- Findings presented in a **PowerPoint deck** and documented in a **Word report**

---

## 🔗 Live Dashboard

**👉 [Click here to view the Live Power BI Report](https://app.powerbi.com/groups/me/reports/15c3b099-f22c-4059-b7a3-a70b7d26804d/f43b3299ee7742433f7c?experience=power-bi)**

> Explore the fully interactive dashboard — filter by date, department, age group, and more in real time.

---

## 🗂️ Repository Structure

```
colombia-hospital-analytics/
├── SQL_1.sql                # DDL + Data: Doctors table
├── SQL_2.sql                # DDL + Data: Patients table
├── SQL_QUERY.sql            # Analytical SQL queries (Q15–Q21)
├── Colombia_Hospital.pbix   # Power BI dashboard
├── Colombia_Hospital.pptx   # Presentation deck
├── Colombia_Hospital.docx   # Full written report
└── README.md
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| MySQL | Database creation, data ingestion, analytical queries |
| Power BI | Interactive dashboard, DAX measures, data cleaning |
| Power Query | Null handling, data type fixes, Age Group binning |
| DAX | KPI measures — Avg Wait Time, Total Visits, Satisfaction Score |
| MS Word | Written project documentation |
| MS PowerPoint | Stakeholder presentation |

---

## 🗃️ Database Schema

### Doctors Table

| Column | Type | Description |
|--------|------|-------------|
| patient_id | VARCHAR | Foreign key linking to Patients |
| department_referral | VARCHAR | Department the doctor works in |
| Doctor Name | VARCHAR | Full name of the doctor |
| Appointment Fees | INT | Fee charged per appointment |
| Total Bill | INT | Total billing amount per visit |
| Doctor ID | VARCHAR | Unique doctor identifier (e.g., CAH001) |

### Patients Table

| Column | Type | Description |
|--------|------|-------------|
| date | VARCHAR | Visit date and time (DD-MM-YYYY HH:MM) |
| patient_id | VARCHAR | Unique patient identifier |
| patient_gender | VARCHAR | M / F |
| patient_age | INT | Patient age in years |
| patient_sat_score | VARCHAR | Satisfaction score (0-10); ~73% null |
| patient_race | VARCHAR | Patient racial/ethnic background |
| patient_admin_flag | VARCHAR | Whether patient was admitted (TRUE/FALSE) |
| patient_waittime | INT | Wait time before care (minutes) |
| department_referral | VARCHAR | Department visited |

---

## 📊 Power BI Dashboard — Key Questions Answered

| # | Question | Key Finding |
|---|----------|-------------|
| 1 | Data Cleaning | 73% nulls in patient_sat_score — imputed with median (5) |
| 2 | Avg Wait Time | ~36 minutes overall; Neurology highest at ~40 mins |
| 3 | Visits by Department | General Practice dominates — 7,500 visits (81%) |
| 4 | Visits by Age Group | 19–35 age group highest (~2,800 visits) |
| 5 | Null Handling | Median imputation chosen over row deletion |
| 6 | Gender vs Visits | Near-equal split; slight male majority |

**DAX Measures Used:**

```
Avg Wait Time = AVERAGE('Patient_Visits'[patient_waittime])
Total Visits  = COUNTROWS('Patient_Visits')
```

---

## 🔍 Advanced SQL Queries (Q15–Q21)

### Q15 — Top 5 Revenue-Efficient Doctors

Doctors who generated the highest total revenue while serving the fewest unique patients.

```sql
SELECT d.`Doctor ID`, d.`Doctor Name`,
       SUM(d.`Total Bill`)          AS total_revenue,
       COUNT(DISTINCT p.patient_id) AS total_patient
FROM Doctors d
JOIN Patients p ON d.patient_id = p.patient_id
GROUP BY d.`Doctor ID`, d.`Doctor Name`
ORDER BY total_revenue DESC, total_patient ASC
LIMIT 5;
```

### Q16 — Departments with 3 Consecutive Months of Decreasing Wait Time

Uses LAG / LEAD window functions to detect improving departments.

```sql
WITH cte AS (
    SELECT department_referral,
           DATE_FORMAT(STR_TO_DATE(date, '%d-%m-%Y'), '%Y-%m') AS month,
           AVG(patient_waittime) AS avg_wait
    FROM Patients
    GROUP BY department_referral, month
),
cte_1 AS (
    SELECT *,
           LAG(avg_wait)  OVER (PARTITION BY department_referral ORDER BY month) AS previous_month_wait,
           LEAD(avg_wait) OVER (PARTITION BY department_referral ORDER BY month) AS next_month_wait
    FROM cte
)
SELECT DISTINCT department_referral
FROM cte_1
WHERE previous_month_wait > avg_wait
  AND avg_wait > next_month_wait;
```

### Q17 — Male-to-Female Patient Ratio per Doctor (Ranked)

```sql
WITH cte AS (
    SELECT d.`Doctor Name`, d.`Doctor ID`,
           SUM(CASE WHEN p.patient_gender = 'M' THEN 1 ELSE 0 END) /
           SUM(CASE WHEN p.patient_gender = 'F' THEN 1 ELSE 0 END) AS ratio
    FROM Doctors d
    JOIN Patients p ON d.patient_id = p.patient_id
    GROUP BY d.`Doctor Name`, d.`Doctor ID`
)
SELECT doctor_name, doctor_id, ratio,
       RANK() OVER (ORDER BY ratio) AS `rank`
FROM cte;
```

### Q18 — Average Satisfaction Score per Doctor

```sql
SELECT d.`Doctor Name`, d.`Doctor ID`,
       ROUND(AVG(p.patient_sat_score), 2) AS avg_sat_score
FROM Doctors d
JOIN Patients p ON d.patient_id = p.patient_id
GROUP BY d.`Doctor Name`, d.`Doctor ID`;
```

### Q19 — Patient Diversity per Doctor

```sql
SELECT d.`Doctor Name`, d.`Doctor ID`,
       COUNT(DISTINCT p.patient_race) AS diversity,
       COUNT(*)                        AS total_patients_treated
FROM Doctors d
JOIN Patients p ON d.patient_id = p.patient_id
GROUP BY d.`Doctor Name`, d.`Doctor ID`;
```

### Q20 — Male vs Female Billing Ratio

```sql
SELECT
    SUM(CASE WHEN p.patient_gender = 'F' THEN d.`Total Bill` ELSE 0 END) /
    SUM(CASE WHEN p.patient_gender = 'M' THEN d.`Total Bill` ELSE 0 END) AS ratio
FROM Doctors d
JOIN Patients p ON d.patient_id = p.patient_id;
```

### Q21 — Update Satisfaction Score (Capped at 10)

Increases score by 2 for General Practice patients who waited over 30 minutes, without exceeding 10.

```sql
SET SQL_SAFE_UPDATES = 0;

UPDATE Patients
SET patient_sat_score = patient_sat_score + 2
WHERE patient_waittime > 30
  AND department_referral = 'General Practice'
  AND patient_sat_score <= 8;
```

---

## 💡 Key Insights

- **General Practice** handles 81% of all visits — a potential bottleneck risk
- **Neurology and Orthopedics** have the longest average wait times, suggesting understaffing
- **73% of satisfaction scores were missing** — imputed with median (5) to preserve sample integrity
- **Working-age adults (19–50)** account for ~60% of total visits
- **Older patients (66+)** experience longer wait times (~38 mins vs ~33 mins for under-18s)
- No significant racial bias detected in wait time distribution

---

## ⚙️ How to Run

**Step 1 — Set up MySQL Database**

```sql
source SQL_1.sql;
source SQL_2.sql;
```

**Step 2 — Run Analytical Queries**

```sql
source SQL_QUERY.sql;
```

**Step 3 — Open Power BI Dashboard**

Open `Colombia_Hospital.pbix` in Power BI Desktop. If prompted, update the data source connection to point to your local MySQL instance.

---

## 📁 Deliverables

| File | Description |
|------|-------------|
| Colombia_Hospital.pbix | Interactive Power BI dashboard |
| Colombia_Hospital.docx | Detailed written report with all Q&A |
| Colombia_Hospital.pptx | Stakeholder presentation with visuals |
| SQL_1.sql | Doctors table DDL + data |
| SQL_2.sql | Patients table DDL + data |
| SQL_QUERY.sql | Advanced SQL queries (Q15–Q21) |

---

## 👤 Author

**Sourav Jana**  
Aspiring Data Analyst | SQL • Power BI • DAX  
[LinkedIn](https://www.linkedin.com/in/sourav-jana-66b082376/) • [GitHub](https://github.com/sourav2727)

---

*This project was built as a comprehensive analytics case study on hospital operations data. All patient data used is synthetic and for educational purposes only.*
