# Motor Insurance Frequency, Severity, and Pure Premium Analysis in Excel

## Overview

This project is an Excel-based actuarial analysis of a motor insurance portfolio using the `freMTPL2` dataset. It was built as a portfolio project to demonstrate practical actuarial thinking in a simple spreadsheet workflow.

The analysis focuses on three core pricing concepts:

- **Frequency**: how often claims occur
- **Severity**: how large claims are when they occur
- **Pure Premium**: expected observed claim cost per unit of exposure

The project segments results by **age band** and **geographic area**, and presents them through pivot-based analysis and a dashboard.

---

## Project Goal

The goal of this project was to build a clean, interpretable Excel workflow that:

- combines policy-level and claim-level insurance data
- measures claim frequency across segments
- measures claim severity across segments
- estimates pure premium across segments
- presents results in a dashboard format

Rather than trying to build a full production pricing model, the project focuses on the core actuarial logic behind loss-cost analysis.

---

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

---

## Data Preparation

To make the data usable for pivot-based analysis, I built a consolidated `Raw_Data` sheet at the policy level.

The main fields used in the final table include:

- `IDpol`
- `Exposure`
- `ClaimNb`
- `Total_Claim_Amount`
- `Age_Band`
- `Area`

### Aggregating claim amount to the policy level

Since the severity file contains one row per claim, total claim amount had to be aggregated to the policy level. I used `SUMIF` to sum all claim amounts associated with each policy:

```excel
=SUMIF(freMTPL2sev!A:A, A2, freMTPL2sev!B:B)
