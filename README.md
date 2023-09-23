# Survival-Analysis-with-Employee-Turnover
Analyzed employee turnover data encompassing 1400+ employees using Python's KaplanMeierFitter function and Cox-PH model to identify critical employee attributes influencing survival duration and predict future survival probability
# Analysis

**The Kaplan-Meier curve** is used to estimate the survival function from data that are censored, truncated, or have missing values. It shows the probability that a subject will survive up to time t.

### Relationship Examination:
**a. Survival Probability based on Years At Company** 

![download](https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/f402e235-4caa-4f7d-b811-b7f0bea1ecc3)

--> From the graph, it shows that people staying at the company for about 20 years have 70% survival probability. The slope gets steeper at 35 years and goes downward from that point.

We can also plot the same graph with confidence interval with plot_survival_function. 

![download](https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/975c876f-7a26-41a7-b93d-9996f86c9fa7)

The wider the confidence interval the less certainty the model has. The model has high confidence level at the beginning. However, at year 25 forwards, the confidence level starts going down, as shown in the widened confidence interval.

**b. Survival Probability based on the Environmental Satisfaction:**

Since the environmental satisfaction level is rated on the scale of 1 to 4, we need to categorize it into "low" or "high" group. With environmental satisfaction equaling 1 or 2, we categorize it as “low”. With environmental satisfaction equaling 3 or 4, we categorize it as “high”. 

```
Survival Function of Different Groups with KMF:

# Define the low and high satisfaction
Low = ((df.EnvironmentSatisfaction == 1) | (df.EnvironmentSatisfaction == 2))
High = ((df.EnvironmentSatisfaction == 3) | (df.EnvironmentSatisfaction == 4))

# Plot the survival function
ax = plt.subplot()
kmf = KaplanMeierFitter()

kmf.fit(durations=df[Low].YearsAtCompany, event_observed=df[Low].Attrition, label='Low Satisfaction')
kmf.survival_function_.plot(ax=ax)
kmf.fit(durations=df[High].YearsAtCompany, event_observed=df[High].Attrition, label='High Satisfaction')
kmf.survival_function_.plot(ax=ax)
plt.title('Survival Function based on Environmental Satisfaction')
plt.show()
```
![download](https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/da596127-6580-44b9-a440-de42dfa09dfb)

According to this graph, we see that in the same period of time, employee with higher environmental satisfaction tend to have a higher survival probability than those with low environmental satisfaction.

**c. Gender: Female vs Male:**

Similarly, we can plot the survival function with female and male employees. 
![download](https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/e9602d61-0e45-40fc-8f46-48ce1a35df20)

Since from the graph, the difference between survival probability between female and male is not too clear. Therefore, we should run a **log-rank hypothesis test** to verify the hypothesis that there is no difference in survival duration between male and female employees. If the p-value is less than 0.05, then we can reject the null hypothesis indicating male and female survival curves are identical.

```
from lifelines.statistics import logrank_test
output = logrank_test(durations_A = df[male]['YearsAtCompany'],
                      durations_B = df[female]['YearsAtCompany'],
                      event_observed_A = df[male]['Attrition'],
                      event_observed_B = df[female]['Attrition'])
print(output.print_summary)
```
**The results:**
```
<bound method StatisticalResult.print_summary of <lifelines.StatisticalResult: logrank_test>
               t_0 = -1
 null_distribution = chi squared
degrees_of_freedom = 1
         test_name = logrank_test

---
 test_statistic    p  -log2(p)
           1.79 0.18      2.47>
```

Since the p-value is 0.18, we fail to reject the hypothesis. We do not reject the hypothesis of male and female survival curves are identical. 

### COX-PH Model:

**Cox-PH model** is a regression model to examine the relationship between the survival time of individuals and other variables. This model can let us know whether a variable has a high or low impact on the survival duration and predict future survival probabilities of current employees.

First, let's select the columns that we want to examine with. Here are the columns I select:
```
#Cox Proportional-Hazards Model
columns_selected = ['Attrition',
                    'EnvironmentSatisfaction',
                    'JobInvolvement',
                    'JobLevel',
                    'JobSatisfaction',
                    'PercentSalaryHike',
                    'RelationshipSatisfaction',
                    'StockOptionLevel', 
                    'TrainingTimesLastYear', 
                    'YearsAtCompany',
                    ]
df = df[columns_selected]
```

Then, we can fit these columns into the coxPHFitter() function. Remember to specify the duration column and event column.

```
from lifelines import CoxPHFitter 

coxph = CoxPHFitter()
coxph.fit(df, 
          duration_col='YearsAtCompany',
          event_col='Attrition')
```
**Predict Employee Survival Duration**

<img width="500" alt="Screenshot 2023-09-22 185452" src="https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/017970bf-7886-43f5-8dc8-27772f153d98">

From this table, the columns indicate employees's survival probability and the rows indicate the working year. For example, most employee have 100% survival rate during the first year. However, employee 1 has 80% survival during year 10 while employee 4 only has 53% survival in the same year.

**
<img width="900" alt="123456789" src="https://github.com/codingispink/Survival-Analysis-with-Employee-Turnover/assets/138828365/f4b4aff6-7161-40b9-b6a4-f15eb59f3687">

There are a couple of variables to notice. First, let's look at **p value**. With p-value more than 0.05, we know that percent salary hike or relationship satisfaction does not have as much of an impact, or in other word, low impact on the survival duration. On the other hand, other factors such as environmental satisfaction, job level, stock option level... does have a big impact on the survival duration.

The second variable to look at is the exp(coef). In this model, "exp(coef) is the hazard ratio which indicates how much the baseline hazard changes due to one-unit change in the corresponding factor". For example, if job involvement changes one unit, survival time change 53% ([1/0.65]-1).


## ACKNOWLEDGEMENT
[1] Idil Ismiguzel, https://towardsdatascience.com/hands-on-survival-analysis-with-python-270fa1e6fb41

[2] https://lifelines.readthedocs.io/en/latest/Survival%20Regression.html

[3] IBM Attrition Report









