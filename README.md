# Motor Insurance Frequency, Severity, and Pure Premium Analysis in Excel


Full Workbook link: https://docs.google.com/spreadsheets/d/14PEGcqZe_PAD7uomSUmpDdDPf0isI7Ku/edit?usp=sharing&ouid=108592561068055681868&rtpof=true&sd=true

Short description: Developed an Excel-based motor insurance pricing project that combines policy-level and claim-level data to analyze claim frequency, observed severity, and pure premium across rating segments.

## Overview

This project is an Excel-based actuarial analysis of a motor insurance portfolio using the `freMTPL2` dataset: https://www.kaggle.com/datasets/karansarpal/fremtpl2-french-motor-tpl-insurance-claims & https://www.kaggle.com/datasets/floser/fremtpl2sev. 

It was built as a portfolio project to demonstrate practical actuarial thinking in a simple spreadsheet workflow.

The analysis focuses on three core pricing concepts:

- **Frequency**: how often claims occur
- **Severity**: how large claims are when they occur
- **Pure Premium**: expected observed claim cost per unit of exposure

The project segments results by **age band** and **geographic area**, and presents them through pivot-based analysis and a dashboard.

## Project Goal

The goal of this project was to build a clean, interpretable Excel workflow that:

- combines policy-level and claim-level insurance data
- measures claim frequency across segments
- measures claim severity across segments
- estimates pure premium across segments
- presents results in a dashboard format

Rather than trying to build a full production pricing model, the project focuses on the core actuarial logic behind loss-cost analysis.

## Dataset

The project uses the `freMTPL2` motor insurance data, which comes in two related parts.

### Frequency data

The frequency dataset contains one row per policy. Key fields used in this project include:

- `IDpol`
- `Exposure`
- `ClaimNb`
- `Area`

### Severity data

The severity dataset contains one row per claim. Key fields used include:

- `IDpol`
- `ClaimAmount`

Because one policy can generate multiple claims, the severity data has a one-to-many relationship with the policy-level frequency data. That relationship had to be handled before any meaningful analysis could be done.

## Data Preparation

To make the data usable for pivot-based analysis, I built a consolidated `Raw_Data` sheet at the policy level.

The main fields used in the final table include:

- `IDpol`
- `Exposure`
- `ClaimNb`
- `Total_Claim_Amount`
- `Age_Band`
- `Area`

### Area

The dataset includes a categorical variable `Area`, labeled from A to F, representing different geographic segments within the portfolio. While the exact mapping of these categories is not provided, they can be interpreted as proxies for geographic risk factors (e.g., urban vs. rural environments or varying traffic conditions).

In this analysis, `Area` is used as a segmentation variable to evaluate how claim frequency and severity differ across geographic groups.

### Aggregating claim amount to the policy level

Since the severity file contains one row per claim, total claim amount had to be aggregated to the policy level. I used `SUMIF` to sum all claim amounts associated with each policy:

```excel
=SUMIF(freMTPL2sev!A:A, A2, freMTPL2sev!B:B)
```

This step is important because a lookup formula such as `XLOOKUP` would only return one matching claim and would not correctly handle multiple claims for the same policy.

### Creating age bands

I grouped policyholder ages into broader rating-style bands for segmentation. These bands were then used in the pivot tables and dashboard.

Example age groups:

- `<25`
- `25–35`
- `35–45`
- `45–55`
- `55–65`
- `>65`

## Frequency Analysis

Claim frequency was defined as:

```text
Frequency = Total Reported Claims / Total Exposure
```

To calculate this, I built a pivot table using:

- **Rows**: `Age_Band`
- **Columns**: `Area`
- **Values**:
  - Sum of `ClaimNb`
  - Sum of `Exposure`

I then calculated frequency outside the pivot as:

```text
Frequency = Sum(ClaimNb) / Sum(Exposure)
```

This distinction matters. The correct approach is to divide aggregated claims by aggregated exposure, rather than averaging row-level claim ratios.

## Severity Analysis

Observed claim severity was defined as:

```text
Severity = Total Observed Claim Amount / Severity-Eligible Claim Count
```

During the data preparation process, I found that some policies had positive claim counts but zero aggregated claim amount. In other words, some reported claims did not appear to have usable severity information in the claim-amount data.

To avoid understating average claim size, I created a helper field that only counts claims with observed claim amount:

```excel
=IF([@Total_Claim_Amount]>0, [@ClaimNb], 0)
```

This produced a `Severity_Eligible_ClaimNb` field, which was then used as the denominator for severity.

The severity pivot table used:

- **Rows**: `Age_Band`
- **Columns**: `Area`
- **Values**:
  - Sum of `Total_Claim_Amount`
  - Sum of `Severity_Eligible_ClaimNb`

Severity was then calculated as:

```text
Severity = Sum(Total_Claim_Amount) / Sum(Severity_Eligible_ClaimNb)
```

This means the severity measure in this project should be interpreted as **observed severity**, based only on claims with usable amount data.

## Pure Premium Analysis

Pure premium was defined as:

```text
Pure Premium = Total Observed Claim Amount / Total Exposure
```

This represents expected observed claim cost per unit of exposure.

In a fully aligned dataset, pure premium is often expressed as:

```text
Pure Premium = Frequency × Severity
```

However, that was not the cleanest approach here. In this project, frequency includes all reported claims, while severity excludes claims without usable claim amount data. Because those two measures are based on different claim universes, I calculated pure premium directly instead of forcing the decomposition.

The pure premium pivot table used:

- **Rows**: `Age_Band`
- **Columns**: `Area`
- **Values**:
  - Sum of `Total_Claim_Amount`
  - Sum of `Exposure`

Pure premium was then calculated as:

```text
Pure Premium = Sum(Total_Claim_Amount) / Sum(Exposure)
```

This is the most defensible measure of expected observed loss cost given the available data.

## Dashboard

I created a dashboard to summarize the portfolio-level and segment-level results.

The dashboard includes:

- overall frequency
- overall observed severity
- overall pure premium
- age-level summary metrics
- a pure premium matrix by age band and area

The goal of the dashboard is to make the segmentation story easy to interpret at a glance.

## Key Findings

Several useful patterns emerged from the analysis.

- Claim frequency varies across age bands and geographic areas, indicating that risk is not evenly distributed across the portfolio.
- Claim severity does not move perfectly with claim frequency, suggesting that claim frequency and claim size are influenced by different underlying factors.
- Pure premium provides the clearest segment-level pricing view because it combines both claim incidence and claim size into a single loss-cost measure.

Overall, the project shows why segment-level analysis is more informative than relying on a single portfolio-wide average.

## Excel Techniques Demonstrated

This project uses a number of practical Excel techniques relevant to actuarial and analytical work:

- pivot tables for aggregation
- `GETPIVOTDATA` for dynamic retrieval of segmented results
- `SUMIF` for one-to-many claim aggregation
- lookup logic for banding continuous variables
- helper columns for handling imperfect data
- dashboard design and KPI presentation

## Assumptions and Limitations

This is a simplified portfolio project, so the results should be interpreted with several limitations in mind.

### 1. Severity is based on observed claim amounts

Some reported claims had no associated claim amount in the severity data. These were excluded from the severity denominator.

### 2. Pure premium is not full charged premium

The project estimates expected observed claim cost only. It does not include expense load, profit margin, reinsurance cost, taxes, or other pricing components that would be included in the final premium charged to a policyholder.

### 3. Segmentation is limited

The project focuses only on age band and area. A production pricing model would likely consider additional variables and interactions.

### 4. This is an Excel portfolio project

The purpose of the project is to demonstrate pricing logic, aggregation discipline, and analytical communication rather than to build a production-ready actuarial model.

## Why This Project Matters

This project demonstrates how core actuarial pricing ideas can be translated into a practical Excel workflow.

It shows how to:

- work with policy-level and claim-level data together
- handle one-to-many relationships correctly
- construct frequency, severity, and pure premium with consistent denominators
- communicate actuarial results in a clear business-oriented format

For me, the project was also a useful exercise in turning raw insurance data into a structured, interpretable analysis without relying on specialized actuarial software.

## File Structure

- `Raw_Data`: consolidated policy-level analysis table
- `Frequency_Analysis`: frequency by age band and area
- `Severity_Analysis`: observed severity by age band and area
- `Pure_Premium`: observed pure premium by age band and area
- `Dashboard`: summary KPIs and segment-level results

## Possible Next Steps

A few natural extensions of this project would be:

- adding more rating variables such as vehicle age or bonus-malus class
- building segment relativities
- testing alternative treatments for missing severity data
- replicating the workflow in Python or R
- extending the analysis into a simple generalized linear modeling exercise
