# CMM_Challenge
CoverMyMeds challenge held at the Erdös Institute, May of 2022.

## Overview
### Problem Context

Companies like CoverMyMeds have access to a large quantity of pharmacy transaction data. Our goal is to extrapolate from this data information about the expenses that patients should expect to see at the pharmacy, which is relevant to the patients as well as healthcare providers.

The expenses that patients actually pay for prescription drugs are determined by complicated interactions between drug manufacturers, insurance companies, and pharmacies. While official coverage information from insurance companies, as well as information about manufacturer discount programs, can be difficult to track down, data about individual transactions is abundant. For example, the dataset provided by CoverMyMeds for this project is simulated to model such real-world data sets. 

The purpose of our project is to predict patients’ out-of-pocket pay and drugs’ formulary status for each insurance plan, directly from a large database of transaction data.
This helps patients save money on prescriptions, and also helps doctors to make an informed decision about medication costs when prescribing medications. 

### The Dataset

![](images/datahead.png)

A glimpse of the data:
 + 13.9 million entries
 + 114 drugs
 + 133 diagnosis codes
 + 61 insurance plans
 + 7.8% rejected claims

### Our Approach

The main phases of our project:

 + **Data Exploration**: The data consists of around 14 million pharmacy data-billing claims simulated by CoverMyMeds. Researched insurance plans and explored factors impacting the variation of copayments with respect to the time of year, pharmacy and insurance plans.

 + **Copay prediction**: Introduced new features by feature engineering and predicted copayments with an average 4% MAPE.

 + **Formulary prediction**: Predicted formulary status of each medication on an insurance plan based on discount offered on each medication and rejection rates.

 + **Medication clustering**: Clustered drugs based on similar rejection rates and copayment requirements on Type 1 and Type 2 insurance plans. Clustered drugs based on deductible and coinsurance for Type 3 insurance plans.



## Out-of-Pocket Payments
### Observations
### Preprocessing 
### Prediction Models
### Error Analysis
### Recommendations

## Formulary Status
### Deductible Phase
### Three types of Plans
### Drug Tiers 
#### Type 1
#### Type 2
#### Type 3 

## Next Steps 

## References
