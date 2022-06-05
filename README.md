# CMM_Challenge
CoverMyMeds challenge held at the Erdös Institute, May of 2022.

# Summary 
* [patient_pay_summary](patient_pay_summary.ipynb) - summary on patient pay;
* [formulary_status_summary](formulary_status_summary.ipynb) - summary on determining formulary status; 
* [patient_pay_prediction](patient_pay_prediction.ipynb) - details on the patient_pay model;
* [formulary_status](Formulary_status.ipynb) - details on the formulary status;
* [similar_drugs_by_diagnosis](similar_drugs_by_diagnosis.ipynb) - TSNE plot visualization of drugs based on similar diagnosis codes;

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

We see higher out-of-pocket pay early in the year compared to later in the year

![](images/payvstime.png)

Although we see this effect more clearly for some BINs than others: 

![](images/deductiblephasebybin.png)

Also, we see that branded drugs tend to be more expensive and to get rejected more often:

![](images/payvsrej.png)

### Our Approach

The main phases of our project:

 + **Data Exploration**: The data consists of around 14 million pharmacy data-billing claims simulated by CoverMyMeds. We researched insurance plans and explored factors impacting the variation of copayments with respect to the time of year, pharmacy and insurance plans.

 + **Copay prediction**: We introduced new features by feature engineering and predicted copayments with an average 4% MAPE.

 + **Formulary prediction**: We predicted formulary status of each medication on an insurance plan based on discount offered on each medication and rejection rates.

 + **Medication clustering**: We clustered drugs based on similar rejection rates and copayment requirements on Type 1 and Type 2 insurance plans, and we clustered drugs based on deductible and coinsurance for Type 3 insurance plans.



## Out-of-Pocket Payments
### Observations
For each combination of (drug, insurance plan, pharmacy) there are either one or two values of patient_pay. 
Generally, we see the higher copay early in the year. This is due to the deductible phase.
### Preprocessing 
Using exploratory analysis we extracted 8 important features to help predict patient pay. We also
truncated duplicates from the training data to improve model performance.

Here is a heatmap of the correlations between the 8 aggregate features.
The high correlation suggests we should use regularized regression or tree-based models. 

![](images/correlationheatmap.png)

### Prediction Models
We used Ridge Regression, Truncated Ridge Regression, Random Forests, and Gradient Boosting models. 
We chose Random Forests as our final model (with a 4% MAPE score and 0.99 explained variance).

### Metrics
We used a variety of metrics such as explained variance(r^2 score), MAE, RMSE and MAPE to compare our models. 
![](images/modelcompare.png)

### Error Analysis
We observed more mismatch between our prediction and actual patient pay for copay-before-deductible entries.
We recommend another model which predicts the copay-before-deductible if the doctor is interested in
knowing out-of-pocket pay before the deductible phase starts

## Formulary Status

Insurance plans determine drug prices based on a *formulary*:
A classification of drugs into tiers which determine how coverage is applied. 
The number of tiers varies from plan to plan, but in general, higher tiers include
non-preferred, branded or specialty drugs, and have higher rejection rates, costs, and deductibles, 
while lower tiers include preferred and generic drugs, and have lower rejection rates, costs, and deductibles. 

Although the above heuristic holds as a general rule, different insurance plans differ in their specifics between different plans. 
Our motivating question is: from the data alone, can we predict drug tiers for each plan? We thought of this as a clustering problem. 

### Three types of Plans
We used a Gaussian mixture model to separate higher drug costs early in the year from lower costs later in the year. 
We assume this effect is due to a deductible phase in the beginning of the year.
But because not every insurance plan has deductibles, there are not always two distinct price clusters:
![](images/payclusters.png)

By scatter-plotting drugs according to their early-year copays and the difference between their early- and late-year copays, 
we identify three main categories of insurance plans: 
![](images/threetypesofplans.png)
These categories are characterized by the following behavior:

**Category 1)** A wide range of copays, and no deductible. This category includes 23 plans

**Category 2)** A small range of copays, and no deductibles. This category includes 7 plans

**Category 3)** A wide range of copays, and a deductible phase that is followed by a coinsurance phase,
corresponding to the x-intercept and the slopes of the rays in the above graph. This category includes 33 plans. 

### Drug Tiers 
#### Type 1
For category 1 insurance plans, we clustered drugs into tiers according to rejection rate and copay, using KMeans. This leads to 4 tiers
![](images/type1tiers.png)
1) Drugs with zero rejection rate.
2) Drugs with about 20% rejection rate, and low copays.
3) Drugs with about 30% rejection rate, and higher copays. 
4) Drugs with significantly higher copays. 

Please note that (with the exception with tier 1) these tiers were determined by KMeans clustering, and here we are simply describing a heuristic interpretation of the clusters found by the clustering algorithm. 
#### Type 2
For category 2 insurance plans, we again clustered drugs into tiers according to rejection rate and copay, using KMeans. This time, this leads to only 3 tiers
![](images/type2tiers.png)
1) Drugs with zero rejection rate.
2) Drugs with about 20% rejection rate, and low copays.
3) Drugs with about 30% rejection rate, and higher copays. 

Again note that (with the exception with tier 1) these tiers were determined by KMeans clustering, and here we are simply describing a heuristic interpretation of the clusters found by the clustering algorithm. 

#### Type 3 
For category 3 insurance plans it is more useful to cluster drugs into tiers using early-year copay and difference between early- and late-year copay
(as opposed to copay & rejection rate). 
![](images/type3tiers.png)
1) Low copays, large proportion of costs payed after deductible (coinsurance)
2) Medium copays, medium proportion of costs payed after deductible (coinsurance)
3) Higher copays, smaller proportion of costs payed after deductible (coinsurance)
4) Highest copays, apparently no coinsurance

The linear boundaries between these clusters were determined by SVM and further classification was done by analyzing rejection rate. 
Although this method of clustering differs from the method used for Types 1 & 2 insurance plans, the following plot shows that this
classification agrees with clustering by rejection rate. The only exception is between tiers 3 and 4, which rejection rate alone apparently do not distinguish. 
![](images/type3tiers-rej.png)

## Next Steps 

 + Make a supplemental model to predict out-of-pocket pay in the deductible phase for the early months of the year.

 + Cluster preferred drugs based on the diagnoses they treat and compare how they differ across insurance plans. Help patients choose insurance plans based on their prescription needs.

 + Identify preferred in-network and out-of-network pharmacies for each insurance plan.

 + Deploy our model in an application that helps patients and doctors compare insurance plans, pharmacies, and medications with cost in mind. 

## References
1) [uspharmacist.com/article/a-pharmacists-primer-on-prescription-discount-cards](https://www.uspharmacist.com/article/a-pharmacists-primer-on-prescription-discount-cards)
2) [bcbsm.com/medicare/help/understanding-plans/pharmacy-prescription-drugs/tiers.html](https://www.bcbsm.com/medicare/help/understanding-plans/pharmacy-prescription-drugs/tiers.html)
3) [goodrx.com/insurance/health-insurance/medication-formulary](https://www.goodrx.com/insurance/health-insurance/medication-formulary)
4) [patientadvocate.org/explore-our-resources/understanding-health-insurance/understanding-drug-tiers/](https://www.patientadvocate.org/explore-our-resources/understanding-health-insurance/understanding-drug-tiers/)
5) [healthinsurance.org/faqs/whats-the-difference-between-prescription-discount-plans-and-prescription-drug-insurance/ ](https://www.healthinsurance.org/faqs/whats-the-difference-between-prescription-discount-plans-and-prescription-drug-insurance/ )
6) [covermymeds.com/main](https://www.covermymeds.com/main/)
