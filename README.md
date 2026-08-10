# IBM Telco Customer Churn Analysis

## Overview

This project analyzes customer churn using the IBM Telco Customer Churn dataset.

The goal was to understand which customers are more likely to leave, how much monthly revenue is being lost because of churn, and which active customers may need more attention.

I used **Python** for data cleaning and feature engineering, **SQL / BigQuery** for analysis, and **Power BI** to build an interactive dashboard.

---

## Business Questions

This project focuses on three main questions:

1. Which customers are more likely to churn?
2. How much monthly revenue is being lost or is currently at risk?
3. Which individual customers should be prioritized for retention?

---

## Tools Used

- **Python (Pandas)** – data cleaning and feature engineering
- **Google Colab** – Python development
- **Google BigQuery / SQL** – data analysis and creation of reporting views
- **Power BI** – data visualization, DAX measures, and dashboard development

---

## Dataset

The dataset contains **7,043 customers** and includes information such as:

- Customer demographics
- Tenure
- Contract type
- Internet service
- Payment method
- Monthly charges
- Total charges
- Services subscribed
- Churn status

The original dataset contained 21 columns.

After cleaning and feature engineering, the final dataset contained **27 columns**.

---

## Data Cleaning

The dataset was cleaned in Python before being loaded into BigQuery and Power BI.

Main cleaning steps included:

- Checked for duplicate customers
- Checked for missing values
- Converted `TotalCharges` to numeric
- Found 11 blank `TotalCharges` records
- Verified that these customers had zero tenure
- Replaced their `TotalCharges` with 0
- Checked column data types
- Standardized the dataset before analysis

The final cleaned dataset contained:

- **7,043 rows**
- **27 columns**
- **0 missing values**
- **0 duplicate customers**

---

## Feature Engineering

Several new features were created in Python to make the churn analysis more useful.

### Tenure Group

Customers were grouped based on how long they had been with the company:

| Tenure Group | Months |
|---|---:|
| New | 0–12 |
| Established | 13–24 |
| Loyal | 25–48 |
| Veteran | 49+ |

This made it easier to compare churn across different stages of the customer lifecycle.

---

### Service Count

`Service_Count` represents the number of services used by each customer.

The following services were included:

- Phone Service
- Multiple Lines
- Online Security
- Online Backup
- Device Protection
- Tech Support
- Streaming TV
- Streaming Movies

This was used to check whether customers using more services were less likely to leave.

---

### Revenue Lost

`Revenue_Lost` estimates the monthly revenue lost from customers who already churned.

For churned customers:

```text
Revenue Lost = Monthly Charges
```

For active customers:

```text
Revenue Lost = 0
```

Total monthly revenue lost from churned customers was approximately **$139,130.85**.

---

### High Risk Flag

A simple high-risk flag was created based on a combination of customer characteristics.

Customers were flagged when they had:

- Month-to-month contract
- Electronic check payment
- Tenure of 12 months or less

This identified a smaller group of customers with several churn-related characteristics.

---

### Risk Segment

Customers were also grouped into:

- High Risk
- Medium Risk
- Low Risk

This allowed the dashboard to compare churn rates across broader risk groups.

The churn rates showed a clear difference:

| Risk Segment | Churn Rate |
|---|---:|
| High | 53.73% |
| Medium | 23.44% |
| Low | 5.74% |

---

### Revenue at Risk

`At_Risk_Revenue` estimates the monthly revenue associated with active customers identified as high risk.

This helps separate:

- **Revenue Lost** – customers who already left
- **Revenue at Risk** – active customers who may still be retained

---

## SQL Analysis

After cleaning and feature engineering, the data was loaded into BigQuery.

SQL views were created for the main analyses used in Power BI:

- `v_churn_by_risk_segment`
- `v_churn_by_tenure_group`
- `v_churn_by_service_count`
- `v_revenue_summary`
- `v_contract_payment_matrix`
- `v_customer_detail`

These views were used to summarize the data and prepare it for reporting.

---

## Power BI & DAX

Power BI was used to build the final interactive dashboard.

Some of the main DAX measures include:

### Total Customers

```DAX
TotalCustomers =
COUNTROWS(telco_customers_clean)
```

### Churn Rate

```DAX
ChurnRate =
DIVIDE(
    CALCULATE(
        COUNTROWS(telco_customers_clean),
        telco_customers_clean[Churn] = "Yes"
    ),
    COUNTROWS(telco_customers_clean)
)
```

### Active High Risk Customers

```DAX
ActiveHighRiskCustomers =
CALCULATE(
    COUNTROWS(telco_customers_clean),
    telco_customers_clean[Risk_Segment] = "High",
    telco_customers_clean[Churn] = "No"
)
```

### Monthly Revenue

```DAX
TotalMonthlyRevenue =
SUM(telco_customers_clean[MonthlyCharges])
```

### ARPU

Average Revenue Per User:

```DAX
ARPU =
DIVIDE(
    SUM(telco_customers_clean[MonthlyCharges]),
    [TotalCustomers]
)
```

Other measures were created for:

- Revenue Lost
- Revenue at Risk
- Average Monthly Charges
- Average Tenure
- Selected Customer information
- Customer risk factors
- Recommended retention actions

---

# Power BI Dashboard

The final dashboard contains three pages.

## 1. Executive Overview

The first page gives a high-level view of customer churn.

It includes:

- Total Customers
- Churn Rate
- Revenue Lost
- Revenue at Risk
- Active High Risk Customers
- Churn Rate by Risk Segment
- Churn Rate by Tenure Group
- Churn Rate by Service Count

Users can also filter the dashboard by:

- Contract Type
- Internet Service

### Main Findings

**High-risk customers have a much higher churn rate.**

The High Risk segment had a churn rate of approximately **53.7%**, compared with only **5.7%** for Low Risk customers.

**New customers are more likely to churn.**

Customers with 0–12 months of tenure had a churn rate of approximately **47.4%**, while Veteran customers had a churn rate of only **9.5%**.

**Customers using more services generally churn less.**

Customers with higher service counts showed lower churn rates, suggesting that customers using more of the company's services may be more likely to stay.

---

## 2. Revenue at Risk

The second page focuses on the financial impact of churn.

Main KPIs include:

- Monthly Revenue
- Revenue Lost
- Revenue at Risk
- ARPU

The page looks at revenue loss across:

- Payment Method
- Risk Segment
- Tenure Group
- Service Count

The goal of this page is to show where churn is having the largest financial impact instead of looking only at customer counts.

---

## 3. Customer Explorer

The final page allows users to look at individual customers.

Users can search for a Customer ID or select a customer directly from the customer list.

The Customer Profile displays information such as:

- Risk Segment
- Churn Status
- Monthly Charges
- Total Charges
- Tenure
- Service Count
- Contract
- Internet Service
- Payment Method
- Tech Support
- Gender
- Senior Citizen
- Partner
- Dependents

The page also uses DAX to dynamically display customer risk factors.

For example:

```text
⚠ Month-to-month contract
⚠ New customer (12 months or less)
⚠ Electronic check payment
⚠ Fiber optic service
⚠ No tech support
```

Based on these factors, the dashboard also provides simple rule-based retention recommendations, such as:

```text
✓ Offer longer-term contract incentive
✓ Prioritize for early-tenure retention
✓ Encourage automatic payment
✓ Promote tech support service
```

These recommendations are not predictions from a machine learning model. They are simple business rules based on the selected customer's characteristics.

---

## Key Findings

The analysis showed several clear churn patterns:

- **New customers had the highest churn rate**, especially during their first 12 months.
- **High-risk customers had significantly higher churn** than Medium and Low Risk customers.
- **Customers using more services generally had lower churn rates.**
- Month-to-month contracts were strongly associated with churn.
- Electronic check customers showed higher churn compared with other payment methods.
- Fiber optic customers also showed relatively high churn.
- Churn represents a meaningful amount of lost monthly revenue, making customer retention financially important.

---

## Business Recommendations

Based on the analysis:

1. Focus retention efforts on customers during their first year.
2. Prioritize active high-risk customers before they churn.
3. Encourage month-to-month customers to move to longer-term contracts through suitable incentives.
4. Investigate why electronic check customers have higher churn and encourage automatic payment options where appropriate.
5. Promote useful additional services to customers with low service adoption.
6. Use the Customer Explorer to identify individual high-risk customers for targeted retention campaigns.

---

## Limitations

This project has several limitations:

- The dataset does not contain dates, so churn trends over time cannot be analyzed.
- `Revenue_Lost` represents estimated monthly revenue lost based on Monthly Charges, not total lifetime revenue.
- The risk segments and recommended actions are rule-based and are not generated by a machine learning model.
- The dataset does not provide information on customer complaints, network quality, customer satisfaction, or competitor offers, which may also affect churn.

---

## Project Workflow

```text
Raw IBM Telco Dataset
        ↓
Python / Pandas
Data Cleaning & Feature Engineering
        ↓
BigQuery / SQL
Analysis & Reporting Views
        ↓
Power BI
DAX Measures & Interactive Dashboard
        ↓
Business Insights & Retention Recommendations
```

---

---

## Dashboard Preview

### Executive Overview

<img width="1223" height="687" alt="image" src="https://github.com/user-attachments/assets/60b6789a-9942-4a2e-bda8-1d826de8e852" />


### Revenue at Risk

<img width="1222" height="692" alt="image" src="https://github.com/user-attachments/assets/22bc0a99-b3a0-419a-be29-fed9ef7698c8" />


### Customer Explorer

<img width="1225" height="687" alt="image" src="https://github.com/user-attachments/assets/0f808528-c761-4d51-b185-831679803b12" />

---

## Author

**Aquila Rozeus R. Azores**

Licensed Civil Engineer transitioning into Data Analytics, with experience in quantity surveying, cost analysis, data reconciliation, and reporting.
