# Predicting Employee Attrition: Project Overview

Built a model for predicting whether or not an employee will leave (86% accuracy, 38% recall on test data) with a interactive tool to help in designing retention interventions for employees that are high-risk and hard to replace
- Utilized a kaggle sample dataset for attrition analysis from ibm
- validated the data and explored features with respect to attrition
- reduced dimensions of compensation, experience, and sentiment features using PCA
- standardized features and compared CV accuracy and recall on four model types: k-nearest neighbors, logistic regression, random forest classifier, and stochastic gradient boosting classifier
- optimized the highest CV performer (logisistic regression) using gridsearch CV
- interpreted model coefficients to understand what features are likely to increase vs decrease attrition risk which employers also have some degree of control over in order to have some retention intervention strategies
- built a interactive visualization to assist in the identification of individuals for retention intervention and the type of intervention strategies that could help lower attrition risk for that individual


## Data Source

[kaggle ibm attrition dataset](https://www.kaggle.com/datasets/yasserh/ibm-attrition-dataset)

## Data Validation

- no missingness, but some features had zero variance and would have been useless to the model
- only 16% of observations represented employees that left, meaning a baseline model of guessing everyone stays would achieve 84% accuracy and 0% recall

## Exploratory Analysis

I analyzed how numeric features were distributed differently for leavers vs stayers and found leavers were on average:
- less experience
- lower paid
- longer commute
- lower sentiment on employee survey values
- traveled more for work
- worked more overtime

![experience](https://github.com/user-attachments/assets/0a724794-0019-4741-9480-c6bd1215fa6f)
![monthlyincom](https://github.com/user-attachments/assets/68d77d95-a119-47eb-ae38-14604ac1e794)
![commute](https://github.com/user-attachments/assets/7ca1732e-9fc1-4a66-a536-85350ab4f19b)
![satisfaction](https://github.com/user-attachments/assets/5d0d8282-9d9b-474c-ba8d-5b9246d48885)
![travel](https://github.com/user-attachments/assets/1c99dc6a-6d6b-44d7-9990-f2c5e388f04b)
![overtime](https://github.com/user-attachments/assets/16f9b863-6927-4ebd-83fe-5f38ad11e020)

Many of the features theoretically were covering similar constructs, so I ran made a correlation matrix to see if there is some redundancy. I found a lot of correlation between features having to do with years experience or some variant on that idea:

![corrplot](https://github.com/user-attachments/assets/5b883db4-12c1-4e77-b957-167f47ecb603)

## Dimension Reduction (PCA)

Because of the redundancy in experience variables I decided to do PCA do see whether we could decorrelate them and use fewer features. Conceptually, there were also a number of features related to compensation and more related to sentiment so I wanted to see if those could also be decorrelated and reduced. All three analyses revelaed good candidates for PCA dimension reduction, overall going from 16 features to 7. Here is a plot showing you can explain 87% of the variance in the 'experience' related features with only the first 2 principle components:

![pca_experience](https://github.com/user-attachments/assets/95b98edc-d142-424f-9a32-64e602b97ee2)

## Feature Engineering, Transformations, and Train/Test Split

- 7 experience related features were reduced to 2 PC's (87% of variance explained)
- 3 compensation related features were reduced to 2 PC's (99% of variance explained)
- 5 sentiment related features were reduced to 3 PC's (80% of variance explained)
- category features were encoded as dummies for sklearn to interpret
- features without variance were removed
- dataset was split randomly in to training (70%) and test (30%) sets
- training and test features standardized (after splitting so as not to 'leak' information from test to training set) using standard scaler (knn requires standardization and it helps with interpretation when there is large difference in scale and variance between numeric features like in our dataset)

## Model Fitting and Evaluation

The following model types were compared on accuracy and recall on 6 fold cross validation scores:

![cv_acc](https://github.com/user-attachments/assets/de175105-6f7d-42d8-8362-5c7272300af6)
![cv_recall](https://github.com/user-attachments/assets/36f4fad8-ccd5-40a4-8685-b0d800b97751)

Logitistic regression was the clear winner when you combine the two metrics. Recall is important in our case because the risk of someone leaving and we didn't identify it or attempt to stop it is greater than the risk of falsely labeling a non-risky person as risky.

A logit model was then tuned using gridsearch cross validation and our best model had an accuracy of 86% (2% increase over guessing everyone stays) and a recall of 38% (38% increase over guessing everyone stays).

## Interpretation

An examination of model coefficients shows which features had the strongest relationships to attrition. I found it helpful to differentiate the features over which employers have some amount of control to alter vs those that cannot or should not be altered, in order to suggest possible interventions for retention. The model suggests the following retention interventions (in order of coef size/strength of relationship):

- cap hours, ensuring employee does not work overtime very often
- pay employee more
- increase employee's stock options
- lower the rate at which employee must travel for work
- allow work from home days or pay for relocation closer to office (distance from home is probably effects of long commutes)

The coefficients on sentiment features are puzzling and would deserve follow up analysis.

![coefs](https://github.com/user-attachments/assets/730444d7-7003-4868-acd0-6339e5e09c12)

## Use Case

Lastly, I built an interactive visualization tool meant to assist in narrowing down the retention intervention pool of employees to ones that are theoretically difficult to replace (high performers, high level, critical job roles). In the tool you can see the leave probabilities produced by the model to look at employees more likely to leave. The tooltip then shows you values for the individual that correspond to rentetion levels our model suggested might lower attrition risk (overtime hours, pay, stock options, travel, commute). Below is a screenshot, but I was able to retain the interactivity in the html version of [my notebook]('https://github.com/christianclayton7/ibm-attrition-prediction/tree/main/Attrition Prediction.html') (scroll to bottom).

<img width="1044" alt="tool screenshot" src="https://github.com/user-attachments/assets/2ad783ce-566e-4f12-a739-5e814e11111e">

The idea is our model ID's risky employees and retention levels and you ID hard to replace employees as well as which retention levers are relevant to each employee. Because our model is observational only, we should track interventions and subsequent outcomes (if left over a certain period of time) to see which interventions have a causal link to attrition.
