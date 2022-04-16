# Prediction_of_Hospital_Readmission_Project
This was a project in which I used a massive amount of hospital-level inpatient outcomes data publically availble from the CMS/Medicare website to predict readmission rates at the hospital level using machine learning. 
Beyond the CMS/Medicare data, I also incorporated demographic data from the US Census Bureau in the development of features capturing the broader context in which hospitals operate.
The best model I was able to develop performed better than very simple models, or simply using average readmission rate across all hospitals to predict rates at a given hospital. However, readmission rates are tightly bunched within a percent or
two at the large majority of hospitals, and this mere fact makes it challenging to subtly differentiate even with large amounts of data and efficient algorithms. Even still, some
interesting insights were gleaned about factors (eg, case mix, respiratory status of patients at intake and post-surgery) potentially important in instantiating even these small (think fractions of a percent to a few percent) differences seen across hospitals
with relatively similar profiles. 
