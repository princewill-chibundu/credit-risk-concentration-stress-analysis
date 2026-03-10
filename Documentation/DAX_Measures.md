# DAX Measures Documentation

## Credit Risk Concentration & Stress Sensitivity Analysis  
### Power BI Measure Reference

This document lists the key DAX measures used to build the Executive and Operational dashboards for the **Credit Risk Concentration & Stress Sensitivity Analysis** project.

The measures below represent the final calculations that drive the visuals used in both dashboards.

---

# 1. Core Portfolio Measures

### Total Portfolio Value

```DAX
Total Portfolio Value =
SUM(loans[Loan Amount])
```

### Total Loans

```DAX
Total Loans =
COUNT(loans[Loan ID])
```

### Performing Exposure

```DAX
Performing Exposure =
[Total Portfolio Value] - [Exposure at Risk]
```

---

# 2. Risk Monitoring Measures

### Delinquent Loans

```DAX
Delinquent Loans =
CALCULATE(
    COUNT(loans[Loan ID]),
    loans[Loan Status Standard] = "DELINQUENT"
)
```

### Defaulted Loans

```DAX
Defaulted Loans =
CALCULATE(
    COUNT(loans[Loan ID]),
    loans[Loan Status Standard] = "DEFAULT"
)
```

### Charged Off Loans

```DAX
Charged Off Loans =
CALCULATE(
    COUNT(loans[Loan ID]),
    loans[Loan Status Standard] = "CHARGED OFF"
)
```

---

# 3. Portfolio Risk Ratios

### Delinquency Ratio

```DAX
Delinquency Ratio =
DIVIDE([Delinquent Loans], [Total Loans])
```

### Default Rate

```DAX
Default Rate =
DIVIDE([Defaulted Loans], [Total Loans])
```

### Charge-Off Ratio

```DAX
Charge-Off Ratio =
DIVIDE([Charged Off Loans], [Total Loans])
```

### Total Risk Ratio

```DAX
Total Risk Ratio =
[Delinquency Ratio] +
[Default Rate] +
[Charge-Off Ratio]
```

---

# 4. Exposure at Risk Measures

### Delinquent Exposure

```DAX
Delinquent Exposure =
CALCULATE(
    SUM(loans[Loan Amount]),
    loans[Loan Status Standard] = "DELINQUENT"
)
```

### Default Exposure

```DAX
Default Exposure =
CALCULATE(
    SUM(loans[Loan Amount]),
    loans[Loan Status Standard] = "DEFAULT"
)
```

### Exposure at Risk

```DAX
Exposure at Risk =
[Delinquent Exposure] +
[Default Exposure]
```

### Exposure at Risk %

```DAX
Exposure at Risk % =
DIVIDE([Exposure at Risk], [Total Portfolio Value])
```

---

# 5. Credit Band Concentration Measures

### Good Band Exposure

```DAX
Good Band Exposure =
CALCULATE(
    [Exposure at Risk],
    loans[Credit Score Band (Ordered)] = "3 - Good (670–739)"
)
```

### Mid-Tier Risk Share %

```DAX
Mid-Tier Risk Share % =
DIVIDE(
    [Good Band Exposure],
    [Exposure at Risk]
)
```

These measures support the **Risk Concentration by Credit Band** visuals used in both dashboards.

---

# 6. Stress Simulation Measures

A **What-If parameter (Shock %)** was implemented to simulate deterioration within the mid-tier credit segment.

### Exposure at Risk (Shocked – Good Band Only)

```DAX
Exposure at Risk (Shocked – Good Band Only) =
VAR Shock = [Shock % Value]
VAR IsGoodBand =
    SELECTEDVALUE(loans[Credit Score Band (Ordered)]) = "3 - Good (670–739)"
RETURN
IF(
    IsGoodBand,
    [Exposure at Risk] * (1 + Shock),
    [Exposure at Risk]
)
```

### Incremental Exposure Under Stress Scenario

```DAX
Incremental Exposure Under Stress Scenario =
SUMX(
    VALUES(loans[Credit Score Band (Ordered)]),
    [Exposure at Risk (Shocked – Good Band Only)]
)
-
[Exposure at Risk]
```

### Stress Sensitivity (% of Portfolio)

```DAX
Stress Sensitivity (% of Portfolio) =
DIVIDE(
    [Incremental Exposure Under Stress Scenario],
    [Total Portfolio Value]
)
```

These measures power the **Stress Sensitivity analysis** used in both dashboards.

---

# 7. Provision Impact Estimate

### Estimated Incremental Provision Requirement

```DAX
Estimated Incremental Provision Requirement =
[Incremental Exposure Under Stress Scenario] * [LGD Proxy]
```

This provides a simplified estimate of additional provisioning requirements under simulated deterioration.

---

# Dashboards Using These Measures

## Executive Dashboard

Primary decision-support measures:

- Total Portfolio Value  
- Exposure at Risk %  
- Mid-Tier Risk Share %  
- Delinquency Ratio  
- Default Rate  
- Charge-Off Ratio  
- Incremental Exposure Under Stress Scenario  
- Estimated Incremental Provision Requirement  

This page supports **strategic risk oversight and portfolio sensitivity evaluation**.

---

## Operational Dashboard

Primary monitoring measures:

- Delinquency Ratio  
- Default Rate  
- Charge-Off Ratio  
- Exposure at Risk %  
- Stress Sensitivity (% of Portfolio)  
- Exposure at Risk (Shocked – Good Band Only)

This page supports **diagnostic monitoring and scenario testing by operational managers**.

---

# Notes

- Loan status values were standardized in **Power Query** before DAX measures were created.
- Stress simulation was implemented using a **Power BI What-If parameter (Shock %)**.
- Stress testing was intentionally applied to the **mid-tier credit band**, where the analysis identified structural concentration risk.
