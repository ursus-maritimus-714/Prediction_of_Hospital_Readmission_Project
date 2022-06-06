# Prediction_of_Hospital_Readmission_Project
# Introduction 
In 2012, the Centers for Medicare and Medicaid Services (CMS) began reducing Medicare payments for subsection(d) hospitals with excess readmissions within 30 days of discharge, under the Hospital Readmission Reduction Program (HRRP). CMS has a proprietary formula for determining excess readmission rate (RR). Total readmissions for cases involving a specific set of conditions for which initial admission occured (heart attack, heart failure, pneumonia, chronic obstructive pulmonary disease, hip/knee replacement, and coronary artery bypass graft surgery) at a given hospital are divided by an expected RR that is based upon RR at hospitals deemed to be similar by CMS.

In the hypothetical scenario driving this project, we have been approached by the leadership of a medium-large sized (~2,700 inpatient cases in the previous year) public hospital in New Jersey, Community Medical Center (CMC), for assistance. In their most recent review from CMS, CMC was flagged as having an unacceptably high readmission rate (17%), and was told that they needed to reduce this rate by at least 1% over the next year to avoid Medicare payment reduction. CMC asked us to help them conduct their own analysis of factors most related to RR, and to potentially assist them in coming up with a data-driven plan to avoid a quality of care-reducing penalty.

# Methods and Approach
## Data Collection
I took two general approaches in data collection for this project. The first was to obtain as much data as possible at the individual hospital level, with the idea being that quantitative accounting of what transpires in the hospital during inpatient (and, in some cases, outpatient) stays from admission to release will have a lot to say about whether or not patients return to the hospital. For this "Within Hospital" data, a number of csv files(>10) were obtained from the CMS website data repository (https://data.cms.gov/provider-data/sites/default/files/resources/). This "Within Hospital" data included:

***Proportions of initial admissions due to different reasons, including those in CMS’ own formula (heart failure, COPD, hip-knee replacement, heart attack, pneumonia, COPD, coronary artery bypass graft surgery)
***Rates of different types of infections during inpatient stay
***CMS computed Patient Safety and Adverse events rates during inpatient stay
***HCAHPS-Related Features (Patient Survey Star ratings from inpatient stay)
***Various other measures from other aspects of hospital care (e.g., emergency department efficiency metrics and hospital worker vaccination rates)

The second data collection approach was focused on the collection of demographic and socioeconomic data at the country level. The rationale being that these types of data ("County-Level Demographic and Socioeconomic Data") would be useful for the development of features that provide context to "Within Hospital" data. From the Bureau of Economic Analysis data archive (https://www.bea.gov/sites/default/files/), I obtained csv files containing county level data on median income. From the US Census Bureau data archive (https://www2.census.gov/programs-surveys/popest/tables/2010-2019/counties/) I obtained data on county level population and median age. 

The "County-Level Demographic and Socioeconimic Data" sources required quite a bit a cleaning and standardization (especially ensuring county names are consistent across sources, and that same-named counties stayed with their apporpriate state) before merging with the "Within Hospital" data tables (many, many Left Joins in this project). Once cleaned and merged, these data souces were used to create several contextualizing features for each hospital. For example, "population per hospital in county" gave each hospital in the sample a de facto measure of theoretical overall patient burden. 

## Exploratory Data Analysis
Prior to the modeling stage, the relationship between individual putative predictive features (~50) and the target feature of Readmission Rate (RR) were explored visually. Some categorial features (e.g., ER Case Volume) were numerically encoded prior to this stage as well using one-hot encoding. During the EDA, it was discovered that many of the HCAHPS-Related Features (patient ratings) were highly correlated with each other and their presence in the data set was consequently reduced to several aggregate features capturing patient's overall sentiments about their care. This reduced concerns for multicollinearity in the first modeling stage (linear modeling). 

The strongest correlations to target feature in the EDA were all negative correlations and were as follows:
* % Initial Inpatient Admissions Due to Hip/Knee Replacement (-.22)
* Overall Hospital Rating (Patient Survey Star Rating from Inpatient Stay) (-.19)
* % Initial Inpatient Admissions Due to Acute Myocardial Infarction (-.18)
* % of Healthcare Workers Vaccinated by Hospital (-.15)
* Also of note were the fact that both the % of patients with COPD at admission and % of patients having respiratory failure post-surgery had negative correlations (both ~-.10) with RR. These features would pop up as important in the best model, and would feature in the client recommendation (see below) 

## Modeling
Overall, four predictive regression models were created using scikitlearn (scikit-learn 1.1.1): Linear, Random Forest, Gradient Boosting, and HistGradient Boosting. For all models, a 70/30 training-test split and 5-fold training set cross validation were enacted. Additionally, hyperparameter grid search optimization was employed per model as warranted (for example, for HistGradient boost, the grid search was conducted for imputation type, scaler type, learning rate, maximum iterations and maximum tree depth). By RMSE on training set cross validation, the best model turned out to be Random Forest (0.748, 0.029), followed by Gradient Boosting (0.752,0.025), HistGradient Boosting (0.756, 0.031), and Linear Regression (0.779, 0.023). Test set validation predictions were within ~1% of training set predictions for each model, indicating that overfitting to the training set was not a major concern. A data quality analysis also indicted that the sample size was not hampering prediction quality. Importantly, the best model (and all full models) performed substantially better than a "dummy" model whereby RR was predicted simply by training set mean (RMSE 0.825). 

With regard to feature importance in the best model, the most important (by a substantial margin) was % of Hip-Knee Replacements in the overall hospital inpatient case mix. This was followed by % acute myocardial infarction in the overall hospital inpatient case mix, % of patients with respiratory problems following surgery, % patients with pneumonia following surgery, and % patients with COPD listed as a reason for initial admission.

## Business Modeling and Client Recommendation
As a final step, best model was run on the entire dataset save for the client hospital (CMC). RMSE of the training set was slightly improved on the entire sample (0.739, 0.05) relative to the training set cross validation in the previous step. The client hospital was shown to have a predicted RR of 15.45% with the best model, which is well below the 16.80% actual RR. I therefore recommended to the client that "watch and wait" is a viable strategy over the next year as variance alone could account for the observed RR that was of concern to CMS. 

However, I also ran a simulation in which CMC increased their %Hip-Knee Replacements in the case mix to that of the sample median and simultaneously also decreased % of patients with respiratory failure after inpatient surgery to the sample median. In both of these areas, CMC was an outlier in the direction associated with higher RR. Both of these features were also among the few most important in the best model. In this simulation, we found that CMC could expect a RR reduction 32% of the way to the overall desired reduction. Addition of one more specialist for Hip-Knee Replacement surgery might be sufficient to boost that procedure sufficiently in the case mix. Indeed many hospitals at the low end of RR in the sample, boast a MAJORITY of cases as this type of elective surgery. As for reduction of respiratory failure post surgery, CMC has a higher than expected % of initial admissions with COPD as a listed cause. It was also evident in the EDA stage that COPD is POSITIVELY correlated to increased respiratory failure rate post surgery. Furthermore, COPD rate was one of the most important features in best model construction. Given that CMC itself is high in both COPD % at intake and postoperative respiratory failure rate, extra vigilance toward COPD patients undergoing surgery might go a long way toward reducing the latter at the hospital. 

## Future Directions
One potential way to improve the model would be to conduct unsupervised learning upfront to cluster hospitals, and then to fit a more bespoke model to each cluster. Currently, the ~2,700 hospitals in the sample (a minimum of 200 inpatient cased per year was used as a highpass filter during the cleaning and wrangling stage) are all modeled together. While this hetereogeneity may be a strength in some respects, it may also be a hindrance as, for example, public vs private or urban vs rural or large vs small hospitals, may systematically vary in some important ways that average out when attempting to model them all together.

Additionally, one major frustration of this project was the unavailabilty of detailed information about facilities, monetary and people resources on a per-hospital level. The county-level socioeconomic and demographic data I brought in from the US Census and Bureau of Economic Analysis were fairly crude proxies for direct knowledge of the resourcing at individual hospitals or, ideally, resourcing related to individual procedure types at individual hospital. Likewise, I generated some features from outpatient services (e.g., average waiting time for patients in the Emergency Department, % of staff vaccinated) as proxies for unavailable data more closely related to inpatient services. Perhaps direct queries to individual hospitals can yield at least some of this direct resourcing data not available from the CMS data repository. 
 


     




