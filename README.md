# Performance – Estimated

This project analyzes time estimation performance in logistics processes using data retrieved from AWS Athena

The analysis focuses on the relationship between **estimated overdue legs** and **final package delivery status**, in order to evaluate estimation accuracy and identify which process legs have the greatest impact on package delays.

---

## Objectives

This analysis aims to answer the following questions:

1. Is our time estimation accurate?
2. Which leg has the highest impact on determining package overdue?
3. Is there a strong relationship between overdue legs and overdue packages?
4. What operational actions can be taken based on these findings?

---

## Estimation Logic and Data Model

This project retrieves **estimation metrics and overdue-related flags at package level**.

### Overdue estimation

- `diff_sla_statistics` represents the difference between:
  - total actual travel time, and the 80 percentile of historical travel time.

A package is considered **finally overdue** using the boolean field: `flag_controlled_final_medition_vs_client_sla_overdue`

### Leg-level estimation

For each leg, the P80 travel time is calculated:

- **WH** → P80 travel time for Warehouse leg  
- **TD** → P80 travel time for Transport leg  
- **AD** → P80 travel time for Additional leg  
- **UM** → P80 travel time for Last Mile leg  

If actual_leg_time > P80_leg_time then that leg is flagged as **estimated overdue**.

With this logic, we can determine:
- Whether a package was overdue
- Whether the estimation predicted an overdue
- Which leg(s) were estimated as delayed

The query focuses especially on **packages delivered on time but estimated as overdue**, in order to detect estimation inconsistencies.

---

## Technologies Used

- Python  
- Pandas  
- Matplotlib  
- Seaborn  
- AWS Athena  

---


### 1. Data Preparation

- SQL query execution in Athena
- Loading results into a Pandas DataFrame
- Volume aggregation at package level

---

### 2. Volume Analysis by Estimated Overdue Flags

Packages are grouped by the total number of active overdue flags (`total_flags`).

This step answers:
- How many packages were **estimated as overdue but delivered on time**
- Which legs contribute most to estimating a package as overdue

A **bubble chart** is used where:
- X-axis represents the number of overdue legs
- Bubble size represents package volume
- Color represents the overdue leg

This visualization helps identify which legs are most relevant in determining package delay.

---

### 3. Correlation Analysis

A correlation analysis is performed between overdue flags, including:

- `flag_wh_overdue`
- `flag_td_overdue`
- `flag_ad_overdue`
- `flag_um_overdue`

A correlation matrix is visualized using a heatmap to evaluate dependencies between legs.

---

## 🔍 Key Findings

- A significant number of packages are estimated as overdue without any leg being flagged as overdue, which **is not logically consistent**.
- This indicates that the **statistical estimation logic must be reviewed** by the engineering team.
- Bubble chart analysis shows that **UM and AD legs have lower relevance** in determining final package delay.
- Correlation values suggest that delays in one leg **do not strongly propagate** to other legs.

---

## Next Steps

- Validate and correct the statistical fields used for estimation.
- Redesign the estimation logic to consider **inter-leg dependency**, prioritizing packages delayed in any leg.
- Improve SLA performance by using this model as an early warning system.
- I would add a conection across all legs to prioritize the package delayed in any leg. With this tool, we will have a better SLA performance
---


