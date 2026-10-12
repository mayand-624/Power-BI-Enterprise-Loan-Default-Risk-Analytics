# Enterprise Loan Default Risk Analytics

### Dashboard Link : Loan Default & Overview - Proj_Dashboard - Power BI

## Problem Statement

This dashboard enables risk analysts and financial stakeholders to dynamically evaluate applicant financial stability, loan characteristics, and repayment behavior. It helps the institution identify demographic vulnerabilities, track year-over-year financial trends, and assess credit risk across various income brackets and loan purposes.

A significant challenge in enterprise reporting is managing large datasets efficiently while maintaining a single source of truth. By utilizing Power BI Dataflows and a Standard Mode Gateway, this project bypasses local desktop compute limitations, centralizing the ETL process in the cloud for over 250,000 historical loan records.

With detailed metrics on default rates—such as unemployed applicants exhibiting the highest default rate (3.39%) compared to full-time employed applicants (2.36%)—the institution can utilize this dashboard to tighten credit requirements for specific employment and credit score bins, ultimately mitigating financial exposure.

### Steps followed

- Step 1 : Loaded the raw dataset (`Loan_default.csv` containing 19 financial attributes and over 250,000 records) into an on-premises SQL Server database via SQL Server Management Studio (SSMS).
- Step 2 : Installed and configured a Power BI "Standard Mode Gateway" to establish a secure, automated bridge between the local SQL Server and the cloud-based Power BI Service.
- Step 3 : Built a centralized ETL pipeline using Power BI Dataflows (Gen1). This offloads heavy data transformations to the cloud, ensuring faster local desktop performance and establishing a single source of truth for all enterprise reports.
- Step 4 : Connected Power BI Desktop to the cloud Dataflow to ingest the cleaned, pre-processed semantic model.
- Step 5 : Engineered categorical demographic segments to group continuous age values.

For creating this new column, the following DAX expression was written:

```
    Age Groups =
    IF('Loan_default'[Age] <= 19, "Teen",
        IF('Loan_default'[Age] <= 39, "Adult",
            IF('Loan_default'[Age] <= 59, "Middle Age Adult",
           "Senior Citizens")))
```

- Step 6 : Engineered financial segments to facilitate root-cause analysis in AI visuals. A calculated column was created using the `SWITCH` function for optimal DAX performance:
    
    ```
      Income Bracket =
      SWITCH(
          TRUE(),
          'Loan_default'[LoanAmount] <= 30000, "Low Income",
          'Loan_default'[LoanAmount] > 30000 && 'Loan_default'[LoanAmount] <= 60000, "Medium Income",
          'Loan_default'[LoanAmount] > 60000, "High Income"
      )
    ```
    
- Step 7 : Created base aggregations using null-safe iterators. To ensure data integrity, `SUMX` was combined with logical checks to filter out blank records before calculating monetary totals.
- Step 8 : Implemented advanced filter context manipulation to prevent external slicers from distorting baseline KPIs. `ALLEXCEPT` was utilized for isolating dimensions.

Following DAX expression was written for calculating average income strictly anchored to employment type:

```
    Average income by Emp Type =
    CALCULATE(
        AVERAGE('Loan_default'[Income]),
        ALLEXCEPT('Loan_default', 'Loan_default'[EmploymentType])
    )
```

Snap of the Average Income by Employment Type visual:

<img width="428" height="287" alt="SS2" src="https://github.com/user-attachments/assets/fa7c6c07-6bbc-4e82-88e8-0c8a63e3555d" />

- Step 9 : Resolved the Matrix visual "Grand Total" skew issue by engineering an "average of averages". Using `AVERAGEX` and `VALUES`, the engine evaluates specific demographic cohorts equally rather than letting a massive demographic skew the total:
    
    ```
      Average Loan by Age Group =
      AVERAGEX(
          VALUES('Loan_default'[Age Groups]),
          AVERAGE('Loan_default'[LoanAmount])
      )
    ```
    
- Step 10 : Designed dynamic Time Intelligence measures to track Year-over-Year (YoY) variances. `SUM` was used for continuous monetary growth, while `COUNTROWS` was used to measure the frequency growth of actual default events.

Following DAX expression was written for calculating YoY monetary variance:

```
    YOY Loan Amount Change =
    DIVIDE(
        CALCULATE(SUM('Loan_default'[LoanAmount]), 'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) -
        CALCULATE(SUM('Loan_default'[LoanAmount]), 'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),
        CALCULATE(SUM('Loan_default'[LoanAmount]), 'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),
    0) * 100
```

Snap of the YoY Loan Amount Change line chart:

<img width="631" height="267" alt="SS3" src="https://github.com/user-attachments/assets/4da33049-6b4e-40f0-9b52-9166ca5c8b54" />


- Step 11 : Designed a three-page interactive UI: *Loan Default & Overview*, *Applicant Demographics & Financial Profile*, and *Financial Risk Metrics*.
- Step 12 : Deployed a Decomposition Tree (AI Visual) to allow stakeholders to dynamically break down macro KPIs into micro-components based on Income Bracket and Employment Type.

Snap of the Decomposition Tree root-cause analysis visual:

<img width="631" height="362" alt="SS4" src="https://github.com/user-attachments/assets/23530cc6-fd8c-4756-bf4c-09414e895af1" />

- Step 13 : Converted the `Loan_Date` column data type to Date/Time to meet Power BI Service engine partitioning requirements.
- Step 14 : Published the report to the Power BI Service workspace.
- Step 15 : Configured an Automated Scheduled Refresh (1:00 AM IST) via the Gateway and instituted an Incremental Refresh policy (Store 5 years, refresh rolling 10 days, complete days only) to optimize cloud compute capacity.

# Dashboard Snapshots

### Page 1: Loan Default & Overview

<img width="1287" height="722" alt="Page1" src="https://github.com/user-attachments/assets/614cea2a-7d50-4020-9ea0-209e8ec1a019" />

### Page 2: Applicant Demographics & Financial Profile

<img width="1287" height="712" alt="Page2" src="https://github.com/user-attachments/assets/ea3d178d-6aad-4d64-a3a3-58d577947e51" />

### Page 3: Financial Risk Metrics

<img width="1285" height="726" alt="Page3" src="https://github.com/user-attachments/assets/c88ae984-dabc-4351-991d-b8be8bcb50dd" />

# Insights

A multi-page report was created on Power BI Desktop & published to Power BI Service.

Following inferences can be drawn from the dashboard:

### [1] Macro Financial Exposure

Total Loan Amount Deployed = $32.58 Billion

High Income Bracket Exposure = $30.72 Billion

Medium Income Bracket Exposure = $1.40 Billion

Low Income Bracket Exposure = $0.46 Billion

```
       Thus, the vast majority of capital risk is concentrated within the High Income applicant bracket.
```

### [2] Default Rates by Employment Status

```
a) Unemployed - 3.39 %
b) Part-time - 3.01 %
c) Self-employed - 2.86 %
d) Full-time - 2.36 %
```

Applicants with full-time employment exhibit the highest reliability in repayment, while unemployed applicants present the highest statistical risk of default.

### [3] Average Income by Employment Type

```
  a) Full-time: $82,890
  b) Self-employed: $82,447
  c) Part-time: $82,389
  d) Unemployed: $82,272
```

These averages are secured against external dashboard filtering (via `ALLEXCEPT`), providing a static baseline for applicant income evaluation.

### [4] Capital Allocation by Loan Purpose

1.1) Home Loans: $6.54 Billion

1.2) Business Loans: $6.52 Billion

1.3) Education Loans: $6.51 Billion

1.4) Auto Loans: $6.50 Billion

1.5) Other Loans: $6.49 Billion

```
     Capital is distributed highly evenly across all major loan categories, ensuring the institution is not over-leveraged in a single consumer sector.
```

### [5] Demographic Loan Distribution

2.1) Teen: Average Loan Amount of $126,674

2.2) Adult: Average Loan Amount of $127,901

2.3) Middle Age Adult: Average Loan Amount of $127,459

2.4) Senior Citizens: Average Loan Amount of $127,355
