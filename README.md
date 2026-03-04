# 🏢 Palmoria Group — HR Analytics & Gender Pay Equity Analysis

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Pages](https://img.shields.io/badge/Dashboard%20Pages-3-blue?style=for-the-badge)

> *"The future of gender equality in Palmoria lies in your hands."*
> — Mr. Yunus Shofoluwe, CHRO, Palmoria Group

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Business Context](#-business-context)
- [Dataset Summary](#-dataset-summary)
- [Data Preparation](#-data-preparation)
- [DAX Measures & Calculated Columns](#-dax-measures--calculated-columns)
- [Dashboard Pages](#-dashboard-pages)
- [Key Findings](#-key-findings)
- [Compliance Status](#-compliance-status)
- [Recommendations](#-recommendations)
- [Tools Used](#-tools-used)
- [Project Structure](#-project-structure)

---

## 📌 Project Overview

This project delivers a full HR analytics solution for **Palmoria Group**, a Nigerian manufacturing company with 946 employees across 3 locations and 12 departments. The analysis was commissioned by the CHRO to surface gender-related HR issues — particularly around workforce composition, performance ratings, and salary equity — before they escalate into legal or reputational risks.

The deliverable is a 3-page interactive Power BI dashboard with glassmorphism design, supported by this documentation and an executive insight report for leadership.

---

## 🏭 Business Context

| Stakeholder | Role |
|---|---|
| Mr. Ayodeji Chukwuma | CEO — commissioned the gender equity review |
| Mr. Yunus Shofoluwe | CHRO — assigned the analysis task |
| Mr. Gamma | Internal advisor — provided analytical pointers |
| HR Analytics Consultant | Built the dashboard and generated insights |

### The Four Analytical Questions

1. **Workforce Composition** — What is the gender distribution across the organisation, by region and by department?
2. **Performance Ratings** — Are ratings distributed fairly across genders?
3. **Salary Equity** — Is there a gender pay gap? Which departments and regions are most affected?
4. **Regulatory Compliance** — Does Palmoria meet the new $90,000 minimum salary regulation? How are salaries distributed across $10,000 pay bands, by region?

---

## 📊 Dataset Summary

| Table | Columns | Rows | Description |
|---|---|---|---|
| `Palmoria Group Emp-Data` | 6 original + 3 added | 946 | Core employee records |
| `Bonus Mapping` | 6 | 12 | Department-level bonus rate lookup by rating |

### Original Columns — Employee Data
`Name` · `Gender` · `Department` · `Location` · `Salary` · `Rating`

### Original Columns — Bonus Mapping
`Department` · `Very Poor` · `Poor` · `Average` · `Good` · `Very Good`

---

## 🧹 Data Preparation

All cleaning was performed in **Power Query** before loading into the Power BI data model.

### Step 1 — Gender Column
```
Issue:   Some employees left gender blank (null)
Fix:     Replaced all null values with "Others"
Reason:  Ensures every employee is counted in gender-based analysis
         without making assumptions about identity
```

### Step 2 — Department Column
```
Issue:   Some records contained "NULL" as the department value
Fix:     Removed all rows where Department = "NULL"
Reason:  Null department records cannot be meaningfully categorised
         and would distort department-level analysis
```

### Step 3 — Salary Column
```
Issue:   Some employees had no salary value (null)
Fix:     Removed all rows where Salary = null
Reason:  These employees are no longer with the company;
         including them would skew pay analysis
```

> **Result after cleaning:** 946 valid employee records across 12 departments and 3 locations.

---

## ⚙️ DAX Measures & Calculated Columns

### 1. Salary Band *(Calculated Column)*
Groups each employee's salary into a $10,000 bracket for distribution analysis.

```dax
Salary Band = 
SWITCH(
    TRUE(),
    [Salary] >= 10000  && [Salary] <= 20000,  "10,000-20,000",
    [Salary] >= 20001  && [Salary] <= 30000,  "20,000-30,000",
    [Salary] >= 30001  && [Salary] <= 40000,  "30,000-40,000",
    [Salary] >= 40001  && [Salary] <= 50000,  "40,000-50,000",
    [Salary] >= 50001  && [Salary] <= 60000,  "50,000-60,000",
    [Salary] >= 60001  && [Salary] <= 70000,  "60,000-70,000",
    [Salary] >= 70001  && [Salary] <= 80000,  "70,000-80,000",
    [Salary] >= 80001  && [Salary] <= 90000,  "80,000-90,000",
    [Salary] >= 90001  && [Salary] <= 100000, "90,000-100,000",
    [Salary] >= 100001,                        "100K+",
    "Null"
)
```

---

### 2. Bonus *(Calculated Column)*
Looks up each employee's bonus rate from the `Bonus Mapping` table based on their department and rating, then calculates the bonus amount.

```dax
Bonus = 
VAR _dept = 'Palmoria Group Emp-Data'[Department]
VAR _rating = 'Palmoria Group Emp-Data'[Rating]
VAR _bonusRate = 
    SWITCH(
        _rating,
        "Very Poor", LOOKUPVALUE('Bonus Mapping'[Very Poor], 'Bonus Mapping'[Department], _dept),
        "Poor",      LOOKUPVALUE('Bonus Mapping'[Poor],      'Bonus Mapping'[Department], _dept),
        "Average",   LOOKUPVALUE('Bonus Mapping'[Average],   'Bonus Mapping'[Department], _dept),
        "Good",      LOOKUPVALUE('Bonus Mapping'[Good],      'Bonus Mapping'[Department], _dept),
        "Very Good", LOOKUPVALUE('Bonus Mapping'[Very Good], 'Bonus Mapping'[Department], _dept),
        0
    )
RETURN [Salary] * _bonusRate
```

---

### 3. Total Pay *(Calculated Column)*
Combines base salary and bonus for each employee's complete compensation.

```dax
Total Pay = 'Palmoria Group Emp-Data'[Salary] + 'Palmoria Group Emp-Data'[Bonus]
```

---

### 4. SumTotalPay *(Measure)*
Aggregates total compensation across all employees or any filtered context.

```dax
SumTotalPay = SUM('Palmoria Group Emp-Data'[Total Pay])
```

---

### 5. TotalPayoutbyLocation *(Measure)*
Calculates total payout iteratively — used in location-level visuals where the Location field acts as the row context.

```dax
TotalPayoutbyLocation = SUMX('Palmoria Group Emp-Data', [Salary] + [Bonus])
-- Note: "Location" is used as the X-axis iterator when building this visual
```

---

### 6. CompanywidePayout *(Measure)*
Returns the total compensation bill across the entire company.

```dax
CompanywidePayout = SUMX('Palmoria Group Emp-Data', [Salary] + [Bonus])
```

---

### 7. Compliance Status *(Calculated Column)*
Flags each employee as compliant or below the regulatory $90,000 minimum.

```dax
Compliance Status = IF('Palmoria Group Emp-Data'[Salary] >= 90000, "Compliant", "Below Minimum")
```

---

## 📱 Dashboard Pages

The dashboard uses a **Glassmorphism design** — backgrounds were built in PowerPoint and imported into Power BI as custom canvas backgrounds. All three pages share consistent Location, Gender, and Rating slicers for cross-filtering.

<img width="887" height="501" alt="Screenshot 2026-03-04 081446" src="https://github.com/user-attachments/assets/51c1bab8-8b64-4d33-aea6-81cd4a18e583" />

<img width="1256" height="707" alt="Screenshot 2026-03-03 121749" src="https://github.com/user-attachments/assets/ce6a4ffb-57c2-4c1a-ab42-0632b7090dd5" />

<img width="1258" height="707" alt="Screenshot 2026-03-03 120050" src="https://github.com/user-attachments/assets/42df39ee-414a-4e48-aee5-1d81a27342ff" />

---

### Page 1 — Workforce Composition

| Chart | Type | Fields Used |
|---|---|---|
| Employee Headcount by Gender | Clustered Bar | Gender, Count of Name |
| Number of Employees per Department | Stacked Bar | Department, Count of Name, Gender (legend) |
| Staff Gender Distribution Across Cities | Stacked Bar | Location, Count of Name, Gender (legend) |
| KPI Cards | Card | Employees, Locations, Departments |

**Purpose:** Establish the demographic baseline. Understand how gender is distributed across the whole organisation, by city, and by department.

---

### Page 2 — Performance Rating Analysis

| Chart | Type | Fields Used |
|---|---|---|
| Performance Rating by Gender | Clustered Bar | Rating, Count of Name, Gender (legend) |
| Least Performing Department | Bar (filtered) | Department, Count of Name — filtered to "Very Poor" only |

**Purpose:** Identify whether performance ratings are distributed equitably between genders and surface which departments have the most performance concerns.

---

### Page 3 — Salary Analysis

| Chart | Type | Fields Used |
|---|---|---|
| Total Salary Paid to Each Gender (by City) | Clustered Bar | Location, Average Salary, Gender (legend) |
| Total Salary Spend per Department | Clustered Bar | Department, SumTotalPay, Gender (legend) |
| Employees Earning over $90K | Donut | Compliance Status, Count of Name |
| How Many Employees Earn in Each Pay Range by City | Stacked Bar | Salary Band, Count of Name, Location (legend) |
| KPI Cards | Card | Total Payout ($71.92M), Total Bonus Payout ($2.2M), Locations |

**Purpose:** Quantify the gender pay gap at company, city, and department level. Assess regulatory compliance against the $90,000 minimum salary rule.

---

## 🔍 Key Findings

### 👥 1. Workforce Composition

Palmoria employs **946 people** across **3 cities** and **12 departments**, with a near-even gender split overall: **49% Male · 47% Female · 4% Others**.

| City | Women | Men | Others | Total |
|---|---|---|---|---|
| Kaduna | 165 | 182 | 14 | 361 |
| Abuja | 158 | 159 | 18 | 335 |
| Lagos | 118 | 124 | 8 | 250 |

- **Abuja** is the most gender-balanced office — almost exactly equal men and women
- **Kaduna** has the widest gap (17 more men than women)
- Every department has a mix of men and women — but headcount balance alone does not guarantee equal opportunity at senior levels

---

### ⭐ 2. Performance Ratings

Performance ratings tell a surprising story — **women are outperforming men at every level**.

| Rating | Women | Men | Others | Total |
|---|---|---|---|---|
| Very Good | 49 | 36 | 5 | 90 |
| Good | 89 | 82 | 9 | 180 |
| Average | 190 | 212 | 18 | 420 |
| Poor | 58 | 70 | 3 | 131 |
| Very Poor | 20 | 31 | 3 | 54 |
| Not Yet Rated | 35 | 34 | 2 | 71 |

**Key takeaways:**
- 🟢 Women receive more **"Very Good"** ratings: **49 women vs 36 men**
- 🔴 Men account for **57% of all "Very Poor"** ratings (31 men vs 20 women)
- ⚠️ **44% of the company is rated "Average"** — a company-wide performance culture concern, not a gender issue
- ⚠️ **71 employees have never been rated** — a critical process gap affecting pay and promotion decisions

**Most underperforming departments** (highest "Very Poor" counts):

| Department | Very Poor Ratings |
|---|---|
| Services | 8 |
| Human Resources | 7 |
| Marketing | 7 |
| Product Management | 6 |
| Research | 6 |

> ⚠️ Notably, the **HR department itself** — responsible for supporting Palmoria's people — ranks among the most underperforming teams.

---

### 💰 3. Gender Pay Gap

This is the most critical finding. **Women outperform men in ratings but are paid less in every city and every department.**

#### Company-Wide Pay Split

| Gender | Headcount | Total Salary |
|---|---|---|
| Men | 465 (49%) | $35.81M (50%) |
| Women | 441 (47%) | $32.88M (46%) |
| Others | 40 (4%) | $106.41K |

Women represent 47% of the workforce but receive only 44% of total salary — a 3% gap that cannot be explained by headcount alone.

#### Pay Gap by City

| City | Women Avg Salary | Men Avg Salary | Gap |
|---|---|---|---|
| Lagos | ~$74,000 | ~$77,000 | $3,000 more for men |
| Kaduna | ~$72,000 | ~$75,000 | $3,000 more for men |
| Abuja | ~$70,000 | ~$73,000 | $3,000 more for men |

The same **$3,000 average gap exists in all three cities** — confirming this is a systemic, company-wide pattern, not an isolated location issue.

#### Pay Gap by Department

Men's salaries exceed women's salaries in virtually every department. The highest-impact cases by total spend:

| Department | Total Spend | Men | Women | Note |
|---|---|---|---|---|
| Product Management | ~$6.5M | $3.6M | $2.9M | Largest dept — small % gap = large $ gap |
| Legal | ~$6.3M | $3.5M | $2.3M | Second highest spend |
| Business Development | ~$6.2M | $2.9M | $3.0M | ✅ Women lead here |

#### Bonus Distribution

Palmoria's **$2.2M bonus pool** breaks down as Men **$1.03M** · Women **$1.06M** — bonuses are performance-driven and women's higher ratings mean they receive marginally more, which partially offsets the base salary gap. However, a formal gender audit of bonus distribution is still required before the next cycle.

> 🔑 **The core contradiction:** Women are performing better and being paid less. This is not coincidence — it is a systemic pattern requiring correction.

---

## ⚖️ Compliance Status

A recent regulation requires all manufacturing companies to pay a minimum of **$90,000 per employee.**

```
✅ Compliant (≥$90K)   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  30.87%  →  292 employees
❌ Below Minimum        ████████████████████████████  69.13%  →  654 employees
```

**Palmoria is non-compliant for 69% of its workforce.**

#### Salary Distribution by Pay Band & City

| Pay Band | Abuja | Kaduna | Lagos | Total | Status |
|---|---|---|---|---|---|
| $100K+ | 68 | 79 | 55 | 202 | ✅ |
| $90,000–$100,000 | 29 | 32 | 29 | 90 | ✅ |
| $80,000–$90,000 | 39 | 39 | 30 | 108 | ❌ |
| $70,000–$80,000 | 35 | 48 | 34 | 117 | ❌ |
| $60,000–$70,000 | 36 | 37 | 26 | 99 | ❌ |
| $50,000–$60,000 | 37 | 41 | 18 | 96 | ❌ |
| $40,000–$50,000 | 45 | 33 | 27 | 105 | ❌ |
| $30,000–$40,000 | 37 | 39 | 25 | 101 | ❌ |
| $20,000–$30,000 | 9 | 13 | 6 | 28 | ❌ |

**City-level compliance insights:**
- **Kaduna** has the most high earners (79 above $100K) but also 48 employees in the $70K–$80K band — the quickest wins for compliance
- **Abuja** has 45 employees in the $40K–$50K band — the biggest cluster of deeply underpaid staff requiring the largest increases
- **Lagos** has the fewest employees but is proportionally concentrated in mid-to-lower pay bands with the fewest top earners

> ⚠️ **Most urgent:** 28 employees earn between $20,000–$30,000 — less than one-third of the legal minimum. The **lowest salary on record is $28,000**.

---

## ✅ Recommendations

### 🔴 Immediate 
1. **Salary compliance plan** — Prioritise the 28 employees earning under $30K, then work through all 654 below $90K with a phased budget plan developed by the CFO and CHRO
2. **Pay equity audit** — Compare average male vs female salary within each department and city; correct unjustified gaps in the next pay cycle
3. **Bonus audit** — Produce a gender breakdown of the $2.2M bonus payout before the next bonus cycle

### 🟡 Short-Term 
4. **Rate all 71 unrated employees** before any pay or promotion decisions are made for them
5. **Performance improvement plans** for Services, HR, and Marketing — modelled on the practices of top-performing departments (Support and Engineering)
6. **Investigate "Others" pay anomaly** — undisclosed-gender employees earn the highest average in Lagos and Kaduna; understand and document why

### 🟢 Ongoing
7. **Gender-conscious hiring in Kaduna** — intentionally target female candidates in shortlists for new roles
8. **Leadership audit** — verify gender balance in senior roles within Product Management and Legal, the two largest and highest-paid departments

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Data modelling, DAX, visualisation |
| **Power Query (M)** | Data cleaning and transformation |
| **DAX** | Calculated columns and measures |
| **Microsoft PowerPoint** | Dashboard background design (Glassmorphism) |
| **Microsoft Excel** | Source data format |

---


## 👤 Author

**Gbenovie Obhoo**
Data Analytics Consultant
📧 Gbenovieobhoo@gmail.com

---

## 📄 License

This project was developed for analytical and educational purposes for Palmoria Group internal use.

---

*"A company that sees its data clearly is a company that can act with confidence."*
