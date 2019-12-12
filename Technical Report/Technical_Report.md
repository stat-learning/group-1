Abstract
--------

The aim of this research project is to predict the amount of time spent
in a 911 dispatcher's queue before emergency services were deployed,
based on data published by the Portland Police Bureau. We combined
Census data with the police report data in order to have a more holistic
look at the population accessing emergency services. Our early results
show that the two strongest predictors outside of priority of call and
call type (both set by the dispatcher), were mean income and race. The
queue time was lower for those living in a neighborhood with a higher
mean income. To predict, we used a 10-fold cross validation on 6 models:
multiple linear regression, ridge regression, lasso, bagged trees,
random forest, and boosted forest models. For further research, we
recommended a more robust data set, inclusion of spatial and temporal
variables, and comparing difference between years.

 

Introduction
------------

The Portland Police Bureau (PPB) serves the greater Portland, Oregon
metropolitan area. Within Portland, there are 95+ neighborhoods that
differ from each other greatly in terms of income, race, and percentage
of population below the poverty line. We predicted that income and race
would have effects on the performance of the PPB, which we assessed
through dispatch time. We looked to see if we could predict response
time of the PPB using income and race of the neighborhoods, as well as
variables provided by the PPB that ranked the priority of the call and
classified the reason for calling, such as crime or traffic complaint.

 

The Data
--------

Our data is a combination of two different data sets. Census data from
2010 was taken from the City of Portland website
(<https://www.portlandoregon.gov/civic/56897>) and data about police
response time from 2012 was taken from the Portland Police Bureau
(<https://www.portlandoregon.gov/police/76454>.). Rather than using
total response time, we are using the time spent in the queue, that is,
the amount of time a caller waited in the police dispatch queue before
an officer was dispatched to them. The total response time includes
travel time once the officer was dispatched. As there is no record of
how far the officer had to travel, there was no way to standardize the
total response time, so we used the time spent in the queue as a more
standardized response time.

 

The unit of observation in this combined data set is a call to the
Portland Police Bureau (PPB). The number of observations is 194,044. To
predict the time in queue of the PPB we are looking at priority of the
crime as rated by the PPB, racial makeup of the neighborhood (in
percentage of white population), population density, and average income
in the neighborhood.

 

Location data is majority of data that is missing in observations with
missing data. This is addressed by PPB on the website. If "the incident
occurred outside of the boundaries of the Portland neighborhoods or at a
location that could not be assigned to a specific address in the system
(e.g., Portland, near Washington Park, on the streetcar, etc.)" (PPB)
there will be no data for location.

 

Population and economic data came from Portland Monthly:
(<https://www.pdxmonthly.com/articles/2016/4/1/real-estate-2016-the-city>)

 \#\#Exploratory Data Analysis

 

 

### Time in Queue for PPB calls

There were categorical variables in the data that coded the priority of
the call and the category of the call. Priority was low, medium, or
high, so we recoded the variable to be 0, 0.5, and 1, respectively.
Category was recoded to be 1 for the "crime" category and 0 for all
other categories, including Traffic, Disorder, and Assist. We assumed
that crime calls may be given priority over other kinds of calls.  

(see r chunks: Data Wrangling)

 

As a preliminary visualization of all of the data, a histogram was
generated for time spent in the queue. This histogram is heavily skewed
right, with a majority of the calls spending less time in the queue than
the mean.

  (see r chunks: Call Frequency)

    ggplot(data=calls, aes(x=TimeInQueue_sec)) + 
      geom_histogram(binwidth=500) +
      labs(x = "Time in Queue", y="Frequency" , title="Frequency of Time in Queue", 
           caption="Number of Observations = 154247, Dashed Line represents Mean of 
           the Call Times") +
      geom_vline(aes(xintercept=mean(TimeInQueue_sec)),
                 color="blue", linetype="dashed", size=1)

![](Technical_Report_files/figure-markdown_strict/Call%20Frequency-1.png)

    summary(calls$TimeInQueue_sec)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##     0.0    37.0   114.0   660.5   686.0 14359.0

 

Plotting bivariate correlations was our first step in exploring the
data. For each, the entire data set was plotted without priority first
and then also plotted with the data faceted by priority. Faceting by
priority allowed us to see how important priority was to response time
as well as see that generally, those with lower income spent longer
times in the queue when controlling for the priority of the call.

 

(see r chunks: Bivariate Predictor Race, Bivariate Predictor Race and
Priority, Predictor Mean Income, Predictor Mean Income and Priority,
transforming variables) ![Frequency plot of time spent in queue by
percentage of white
population](Technical_Report_files/figure-markdown_strict/Bivariate%20Predictor%20Race-1.png)

 

![Frequency plot of time spent in queue by percentage of white
population faceted by
priority](Technical_Report_files/figure-markdown_strict/Bivariate%20Predictor%20Race%20and%20Priority-1.png)

 

![Frequency Plot of Time Spent in Queue Based on Mean
Income](Technical_Report_files/figure-markdown_strict/Predictor%20Mean%20Income-1.png)

 

![Frequency plot of time spent in queue based on mean income faceted by
priority](Technical_Report_files/figure-markdown_strict/Predictor%20Mean%20Income%20and%20Priority-1.png)

 

![](Technical_Report_files/figure-markdown_strict/transforming%20variables-1.png)![](Technical_Report_files/figure-markdown_strict/transforming%20variables-2.png)

 

Principal Component Analysis
----------------------------

We ran principal component analysis in an attempt of dimension
reduction. Our strongest principal component only accounted for 30% of
the variability, and we found that the number of principal components we
needed to account for a majority of the variance was around 4. Our
normal data set only utilizes 5 numerical variables, so we felt that the
dimension reduction was unnecessary.

 

(see r chunks: PCA model building, Plotting PC1 and PC2, Scree plot)

    ## Importance of components:
    ##                           PC1    PC2    PC3     PC4     PC5     PC6
    ## Standard deviation     1.8414 1.3734 1.1103 0.99096 0.93418 0.91700
    ## Proportion of Variance 0.3082 0.1715 0.1121 0.08927 0.07934 0.07644
    ## Cumulative Proportion  0.3082 0.4797 0.5918 0.68106 0.76040 0.83684
    ##                            PC7     PC8     PC9    PC10      PC11
    ## Standard deviation     0.86061 0.77646 0.52282 0.42174 5.743e-15
    ## Proportion of Variance 0.06733 0.05481 0.02485 0.01617 0.000e+00
    ## Cumulative Proportion  0.90417 0.95898 0.98383 1.00000 1.000e+00

    ## List of 5
    ##  $ sdev    : num [1:11] 1.841 1.373 1.11 0.991 0.934 ...
    ##  $ rotation: num [1:11, 1:11] 0.0837 -0.4986 -0.4727 -0.2326 -0.0135 ...
    ##   ..- attr(*, "dimnames")=List of 2
    ##   .. ..$ : chr [1:11] "X" "ResponseTime_sec" "TimeInQueue_sec" "TravelTime_sec" ...
    ##   .. ..$ : chr [1:11] "PC1" "PC2" "PC3" "PC4" ...
    ##  $ center  : Named num [1:11] 7.71e+04 1.10e+03 6.61e+02 4.44e+02 6.93e-01 ...
    ##   ..- attr(*, "names")= chr [1:11] "X" "ResponseTime_sec" "TimeInQueue_sec" "TravelTime_sec" ...
    ##  $ scale   : Named num [1:11] 4.45e+04 1.46e+03 1.26e+03 5.73e+02 1.14e-01 ...
    ##   ..- attr(*, "names")= chr [1:11] "X" "ResponseTime_sec" "TimeInQueue_sec" "TravelTime_sec" ...
    ##  $ x       : num [1:154234, 1:11] 1.253 1.265 -0.496 1.77 1.64 ...
    ##   ..- attr(*, "dimnames")=List of 2
    ##   .. ..$ : chr [1:154234] "1" "2" "3" "4" ...
    ##   .. ..$ : chr [1:11] "PC1" "PC2" "PC3" "PC4" ...
    ##  - attr(*, "class")= chr "prcomp"

 

![Plotting on first two principal component along with the
bearings.](Technical_Report_files/figure-markdown_strict/Plotting%20PC1%20and%20PC2-1.png)

 

![A screeplot to show the amount of variance accounted for by each
principal
component.](Technical_Report_files/figure-markdown_strict/Screeplot-1.png)

 

Modeling
--------

We tried a basic non-cross validated linear model just to see how the
skew of the variables would impact the diagnostic graphs. Upon seeing
the non-random residuals and highly non-linear QQ plot, we applied log
transformations to some of the variables. The linear model using the
transformed variables still had problems in the diagnostic graphs, but
they were improved from the non-transformed model. When we built a
cross-validated linear model, we decided to use these transformed
variables.

 

(see r chunk: linear regression, Plot of Residuals, transformed model,
figs2 )

    ## 
    ## Call:
    ## lm(formula = TimeInQueue_sec ~ Pct.White + Std_pop + Income.Std + 
    ##     priority_dummy + category_dummy, data = calls_dummy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1634.4  -533.5  -105.2   180.0 13168.7 
    ## 
    ## Coefficients:
    ##                 Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)     1502.924     22.279   67.460  < 2e-16 ***
    ## Pct.White       -451.016     28.606  -15.766  < 2e-16 ***
    ## Std_pop            8.719      1.980    4.403 1.07e-05 ***
    ## Income.Std        14.816      5.383    2.753  0.00591 ** 
    ## priority_dummy -1283.701      6.965 -184.320  < 2e-16 ***
    ## category_dummy   350.753      6.740   52.038  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1094 on 154228 degrees of freedom
    ## Multiple R-squared:  0.2438, Adjusted R-squared:  0.2437 
    ## F-statistic:  9942 on 5 and 154228 DF,  p-value: < 2.2e-16

 

![Residual Plots for Non-Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/Plot%20of%20Residuals-1.png)![Residual
Plots for Non-Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/Plot%20of%20Residuals-2.png)![Residual
Plots for Non-Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/Plot%20of%20Residuals-3.png)![Residual
Plots for Non-Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/Plot%20of%20Residuals-4.png)

 

    ## 
    ## Call:
    ## lm(formula = log_queue ~ Pct.White + log_income + priority_dummy + 
    ##     category_dummy, data = calls_dummy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -6.0349 -0.8047 -0.0168  0.8127  5.5361 
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)     6.6455115  0.0195351  340.183  < 2e-16 ***
    ## Pct.White      -0.3977650  0.0265922  -14.958  < 2e-16 ***
    ## log_income      0.0068571  0.0008448    8.117 4.81e-16 ***
    ## priority_dummy -2.8572870  0.0075449 -378.705  < 2e-16 ***
    ## category_dummy  0.3231592  0.0072980   44.280  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.186 on 154229 degrees of freedom
    ## Multiple R-squared:  0.5378, Adjusted R-squared:  0.5378 
    ## F-statistic: 4.486e+04 on 4 and 154229 DF,  p-value: < 2.2e-16

 

![Residual Plots for Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/figs2-1.png)![Residual
Plots for Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/figs2-2.png)![Residual
Plots for Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/figs2-3.png)![Residual
Plots for Transformed Loinear
Regression](Technical_Report_files/figure-markdown_strict/figs2-4.png)

 

We tested six different models: three regression-based models and three
algorithmic models. Using 10-fold cross-validation, we fit multiple
linear regression, ridge regression, lasso, bagged trees, random forest,
and boosted forest models. The caretEnsemble package tested the
parameters for all of these models and chose the best performing model
of each type. Since many of our variables were skewed, even after
transformation, we anticipated the algorithmic approach to be more
appropriate. However, since we wanted to engage in inference regarding
time in the queue, linear-based models would provide more interpretable
information.

 

(see r chunks: caret, model results )

 

    ##     LM GLMNET RANGER TREEBAG  GBM
    ## 1 1.19   1.19   1.15    1.16 1.15

 

Our data set was large enough that we could tune the models through
10-fold CV on training set and then test the models into a test set. The
test MSEs for the best model of each type is shown below.

 

(see r chunks: test MSEs, test MSE summary, variable importance )

 

    ##     LM GLMNET RANGER TREEBAG  GBM
    ## 1 1.42   1.42   1.33    1.35 1.33

 

The boosted random forest model has the lowest test MSE. We used
variable importance to infer how much each variable was impacting the
time in the queue. Unsurprisingly, priority and category were
overwhelmingly the most importance.

 

![Summary of the Model with Least Mean Squared
Error](Technical_Report_files/figure-markdown_strict/variable%20importance-1.png)

    ##                           var rel.inf
    ## priority_dummy priority_dummy  98.553
    ## category_dummy category_dummy   0.843
    ## Pct.White           Pct.White   0.367
    ## log_income         log_income   0.236

 

We tried a boosted random forest model without priority and then without
priority and category to compare the race and income variables.
Interestingly, when we removed priority, income switched places with
race as the more important of the two variables, indicating that
priority may have been capturing some of the importance of income in the
original model.

 

(see r chunks: without priority, summary of no priority, without
priority, summary of model without priority)

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9807            -nan     0.1000    0.0613
    ##      2        2.9308            -nan     0.1000    0.0494
    ##      3        2.8903            -nan     0.1000    0.0400
    ##      4        2.8580            -nan     0.1000    0.0327
    ##      5        2.8315            -nan     0.1000    0.0265
    ##      6        2.8099            -nan     0.1000    0.0213
    ##      7        2.7928            -nan     0.1000    0.0173
    ##      8        2.7788            -nan     0.1000    0.0140
    ##      9        2.7673            -nan     0.1000    0.0114
    ##     10        2.7580            -nan     0.1000    0.0092
    ##     20        2.7236            -nan     0.1000    0.0011
    ##     40        2.7147            -nan     0.1000    0.0002
    ##     60        2.7124            -nan     0.1000   -0.0000
    ##     80        2.7114            -nan     0.1000    0.0000
    ##    100        2.7108            -nan     0.1000    0.0000
    ##    120        2.7103            -nan     0.1000    0.0000
    ##    140        2.7098            -nan     0.1000   -0.0000
    ##    150        2.7096            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9783            -nan     0.1000    0.0615
    ##      2        2.9282            -nan     0.1000    0.0499
    ##      3        2.8881            -nan     0.1000    0.0403
    ##      4        2.8549            -nan     0.1000    0.0329
    ##      5        2.8281            -nan     0.1000    0.0264
    ##      6        2.8068            -nan     0.1000    0.0216
    ##      7        2.7892            -nan     0.1000    0.0175
    ##      8        2.7746            -nan     0.1000    0.0142
    ##      9        2.7632            -nan     0.1000    0.0114
    ##     10        2.7540            -nan     0.1000    0.0094
    ##     20        2.7182            -nan     0.1000    0.0012
    ##     40        2.7110            -nan     0.1000    0.0001
    ##     60        2.7088            -nan     0.1000    0.0000
    ##     80        2.7076            -nan     0.1000    0.0000
    ##    100        2.7067            -nan     0.1000   -0.0000
    ##    120        2.7061            -nan     0.1000   -0.0000
    ##    140        2.7056            -nan     0.1000   -0.0000
    ##    150        2.7054            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9780            -nan     0.1000    0.0618
    ##      2        2.9269            -nan     0.1000    0.0502
    ##      3        2.8858            -nan     0.1000    0.0405
    ##      4        2.8533            -nan     0.1000    0.0328
    ##      5        2.8260            -nan     0.1000    0.0263
    ##      6        2.8049            -nan     0.1000    0.0214
    ##      7        2.7871            -nan     0.1000    0.0173
    ##      8        2.7730            -nan     0.1000    0.0141
    ##      9        2.7619            -nan     0.1000    0.0113
    ##     10        2.7524            -nan     0.1000    0.0093
    ##     20        2.7166            -nan     0.1000    0.0013
    ##     40        2.7092            -nan     0.1000    0.0002
    ##     60        2.7072            -nan     0.1000   -0.0001
    ##     80        2.7059            -nan     0.1000   -0.0000
    ##    100        2.7048            -nan     0.1000    0.0000
    ##    120        2.7040            -nan     0.1000   -0.0000
    ##    140        2.7035            -nan     0.1000   -0.0001
    ##    150        2.7033            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9792            -nan     0.1000    0.0610
    ##      2        2.9298            -nan     0.1000    0.0498
    ##      3        2.8892            -nan     0.1000    0.0402
    ##      4        2.8564            -nan     0.1000    0.0323
    ##      5        2.8305            -nan     0.1000    0.0262
    ##      6        2.8097            -nan     0.1000    0.0213
    ##      7        2.7922            -nan     0.1000    0.0173
    ##      8        2.7783            -nan     0.1000    0.0140
    ##      9        2.7671            -nan     0.1000    0.0114
    ##     10        2.7580            -nan     0.1000    0.0092
    ##     20        2.7233            -nan     0.1000    0.0011
    ##     40        2.7141            -nan     0.1000    0.0002
    ##     60        2.7118            -nan     0.1000    0.0000
    ##     80        2.7107            -nan     0.1000    0.0000
    ##    100        2.7100            -nan     0.1000   -0.0000
    ##    120        2.7094            -nan     0.1000    0.0000
    ##    140        2.7089            -nan     0.1000   -0.0000
    ##    150        2.7087            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9777            -nan     0.1000    0.0615
    ##      2        2.9282            -nan     0.1000    0.0502
    ##      3        2.8869            -nan     0.1000    0.0404
    ##      4        2.8550            -nan     0.1000    0.0327
    ##      5        2.8281            -nan     0.1000    0.0267
    ##      6        2.8065            -nan     0.1000    0.0216
    ##      7        2.7888            -nan     0.1000    0.0175
    ##      8        2.7746            -nan     0.1000    0.0143
    ##      9        2.7629            -nan     0.1000    0.0114
    ##     10        2.7534            -nan     0.1000    0.0093
    ##     20        2.7174            -nan     0.1000    0.0011
    ##     40        2.7102            -nan     0.1000    0.0001
    ##     60        2.7078            -nan     0.1000    0.0001
    ##     80        2.7064            -nan     0.1000   -0.0000
    ##    100        2.7052            -nan     0.1000   -0.0000
    ##    120        2.7044            -nan     0.1000    0.0000
    ##    140        2.7039            -nan     0.1000   -0.0000
    ##    150        2.7036            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9766            -nan     0.1000    0.0621
    ##      2        2.9265            -nan     0.1000    0.0501
    ##      3        2.8859            -nan     0.1000    0.0407
    ##      4        2.8524            -nan     0.1000    0.0327
    ##      5        2.8253            -nan     0.1000    0.0264
    ##      6        2.8042            -nan     0.1000    0.0213
    ##      7        2.7869            -nan     0.1000    0.0175
    ##      8        2.7729            -nan     0.1000    0.0141
    ##      9        2.7611            -nan     0.1000    0.0116
    ##     10        2.7521            -nan     0.1000    0.0093
    ##     20        2.7155            -nan     0.1000    0.0012
    ##     40        2.7080            -nan     0.1000    0.0001
    ##     60        2.7057            -nan     0.1000   -0.0000
    ##     80        2.7040            -nan     0.1000   -0.0000
    ##    100        2.7032            -nan     0.1000   -0.0000
    ##    120        2.7023            -nan     0.1000    0.0000
    ##    140        2.7017            -nan     0.1000   -0.0001
    ##    150        2.7014            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9818            -nan     0.1000    0.0616
    ##      2        2.9325            -nan     0.1000    0.0500
    ##      3        2.8927            -nan     0.1000    0.0407
    ##      4        2.8594            -nan     0.1000    0.0330
    ##      5        2.8327            -nan     0.1000    0.0268
    ##      6        2.8118            -nan     0.1000    0.0217
    ##      7        2.7939            -nan     0.1000    0.0177
    ##      8        2.7795            -nan     0.1000    0.0141
    ##      9        2.7680            -nan     0.1000    0.0116
    ##     10        2.7584            -nan     0.1000    0.0093
    ##     20        2.7234            -nan     0.1000    0.0011
    ##     40        2.7137            -nan     0.1000    0.0001
    ##     60        2.7112            -nan     0.1000    0.0000
    ##     80        2.7102            -nan     0.1000    0.0000
    ##    100        2.7095            -nan     0.1000   -0.0000
    ##    120        2.7090            -nan     0.1000   -0.0000
    ##    140        2.7085            -nan     0.1000   -0.0000
    ##    150        2.7084            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9804            -nan     0.1000    0.0621
    ##      2        2.9293            -nan     0.1000    0.0503
    ##      3        2.8884            -nan     0.1000    0.0409
    ##      4        2.8548            -nan     0.1000    0.0329
    ##      5        2.8282            -nan     0.1000    0.0266
    ##      6        2.8063            -nan     0.1000    0.0216
    ##      7        2.7887            -nan     0.1000    0.0176
    ##      8        2.7743            -nan     0.1000    0.0143
    ##      9        2.7625            -nan     0.1000    0.0116
    ##     10        2.7529            -nan     0.1000    0.0093
    ##     20        2.7172            -nan     0.1000    0.0012
    ##     40        2.7097            -nan     0.1000    0.0002
    ##     60        2.7075            -nan     0.1000   -0.0000
    ##     80        2.7062            -nan     0.1000    0.0000
    ##    100        2.7053            -nan     0.1000   -0.0000
    ##    120        2.7047            -nan     0.1000   -0.0001
    ##    140        2.7042            -nan     0.1000   -0.0000
    ##    150        2.7039            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9805            -nan     0.1000    0.0628
    ##      2        2.9298            -nan     0.1000    0.0509
    ##      3        2.8884            -nan     0.1000    0.0413
    ##      4        2.8550            -nan     0.1000    0.0334
    ##      5        2.8286            -nan     0.1000    0.0270
    ##      6        2.8064            -nan     0.1000    0.0219
    ##      7        2.7888            -nan     0.1000    0.0177
    ##      8        2.7739            -nan     0.1000    0.0146
    ##      9        2.7622            -nan     0.1000    0.0118
    ##     10        2.7525            -nan     0.1000    0.0096
    ##     20        2.7154            -nan     0.1000    0.0013
    ##     40        2.7078            -nan     0.1000    0.0002
    ##     60        2.7054            -nan     0.1000   -0.0000
    ##     80        2.7041            -nan     0.1000   -0.0000
    ##    100        2.7031            -nan     0.1000    0.0000
    ##    120        2.7025            -nan     0.1000   -0.0001
    ##    140        2.7017            -nan     0.1000   -0.0000
    ##    150        2.7015            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9809            -nan     0.1000    0.0610
    ##      2        2.9312            -nan     0.1000    0.0498
    ##      3        2.8912            -nan     0.1000    0.0402
    ##      4        2.8587            -nan     0.1000    0.0324
    ##      5        2.8323            -nan     0.1000    0.0263
    ##      6        2.8110            -nan     0.1000    0.0213
    ##      7        2.7939            -nan     0.1000    0.0172
    ##      8        2.7802            -nan     0.1000    0.0139
    ##      9        2.7687            -nan     0.1000    0.0114
    ##     10        2.7596            -nan     0.1000    0.0092
    ##     20        2.7252            -nan     0.1000    0.0011
    ##     40        2.7156            -nan     0.1000    0.0002
    ##     60        2.7132            -nan     0.1000    0.0000
    ##     80        2.7122            -nan     0.1000    0.0000
    ##    100        2.7115            -nan     0.1000   -0.0000
    ##    120        2.7110            -nan     0.1000   -0.0000
    ##    140        2.7106            -nan     0.1000   -0.0000
    ##    150        2.7104            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9792            -nan     0.1000    0.0617
    ##      2        2.9287            -nan     0.1000    0.0501
    ##      3        2.8881            -nan     0.1000    0.0405
    ##      4        2.8559            -nan     0.1000    0.0325
    ##      5        2.8291            -nan     0.1000    0.0265
    ##      6        2.8080            -nan     0.1000    0.0217
    ##      7        2.7908            -nan     0.1000    0.0176
    ##      8        2.7769            -nan     0.1000    0.0143
    ##      9        2.7652            -nan     0.1000    0.0115
    ##     10        2.7557            -nan     0.1000    0.0095
    ##     20        2.7194            -nan     0.1000    0.0012
    ##     40        2.7120            -nan     0.1000    0.0002
    ##     60        2.7098            -nan     0.1000    0.0000
    ##     80        2.7084            -nan     0.1000   -0.0000
    ##    100        2.7075            -nan     0.1000   -0.0000
    ##    120        2.7066            -nan     0.1000   -0.0000
    ##    140        2.7062            -nan     0.1000   -0.0000
    ##    150        2.7059            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9798            -nan     0.1000    0.0622
    ##      2        2.9293            -nan     0.1000    0.0502
    ##      3        2.8888            -nan     0.1000    0.0410
    ##      4        2.8559            -nan     0.1000    0.0332
    ##      5        2.8288            -nan     0.1000    0.0267
    ##      6        2.8071            -nan     0.1000    0.0219
    ##      7        2.7889            -nan     0.1000    0.0176
    ##      8        2.7744            -nan     0.1000    0.0142
    ##      9        2.7628            -nan     0.1000    0.0114
    ##     10        2.7531            -nan     0.1000    0.0092
    ##     20        2.7172            -nan     0.1000    0.0013
    ##     40        2.7099            -nan     0.1000    0.0000
    ##     60        2.7076            -nan     0.1000    0.0000
    ##     80        2.7064            -nan     0.1000   -0.0000
    ##    100        2.7055            -nan     0.1000   -0.0001
    ##    120        2.7047            -nan     0.1000   -0.0000
    ##    140        2.7041            -nan     0.1000   -0.0000
    ##    150        2.7039            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9795            -nan     0.1000    0.0613
    ##      2        2.9302            -nan     0.1000    0.0497
    ##      3        2.8898            -nan     0.1000    0.0403
    ##      4        2.8576            -nan     0.1000    0.0327
    ##      5        2.8312            -nan     0.1000    0.0266
    ##      6        2.8097            -nan     0.1000    0.0213
    ##      7        2.7918            -nan     0.1000    0.0174
    ##      8        2.7776            -nan     0.1000    0.0141
    ##      9        2.7661            -nan     0.1000    0.0113
    ##     10        2.7566            -nan     0.1000    0.0091
    ##     20        2.7225            -nan     0.1000    0.0011
    ##     40        2.7135            -nan     0.1000    0.0002
    ##     60        2.7111            -nan     0.1000    0.0000
    ##     80        2.7101            -nan     0.1000    0.0000
    ##    100        2.7095            -nan     0.1000    0.0000
    ##    120        2.7090            -nan     0.1000   -0.0000
    ##    140        2.7085            -nan     0.1000   -0.0000
    ##    150        2.7083            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9795            -nan     0.1000    0.0625
    ##      2        2.9291            -nan     0.1000    0.0502
    ##      3        2.8877            -nan     0.1000    0.0410
    ##      4        2.8550            -nan     0.1000    0.0331
    ##      5        2.8282            -nan     0.1000    0.0267
    ##      6        2.8063            -nan     0.1000    0.0217
    ##      7        2.7881            -nan     0.1000    0.0176
    ##      8        2.7741            -nan     0.1000    0.0143
    ##      9        2.7625            -nan     0.1000    0.0116
    ##     10        2.7532            -nan     0.1000    0.0095
    ##     20        2.7170            -nan     0.1000    0.0013
    ##     40        2.7097            -nan     0.1000    0.0000
    ##     60        2.7076            -nan     0.1000    0.0000
    ##     80        2.7061            -nan     0.1000    0.0001
    ##    100        2.7052            -nan     0.1000   -0.0000
    ##    120        2.7046            -nan     0.1000   -0.0000
    ##    140        2.7038            -nan     0.1000    0.0001
    ##    150        2.7036            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9781            -nan     0.1000    0.0623
    ##      2        2.9277            -nan     0.1000    0.0504
    ##      3        2.8867            -nan     0.1000    0.0408
    ##      4        2.8532            -nan     0.1000    0.0332
    ##      5        2.8264            -nan     0.1000    0.0267
    ##      6        2.8042            -nan     0.1000    0.0218
    ##      7        2.7863            -nan     0.1000    0.0175
    ##      8        2.7718            -nan     0.1000    0.0141
    ##      9        2.7606            -nan     0.1000    0.0115
    ##     10        2.7511            -nan     0.1000    0.0093
    ##     20        2.7153            -nan     0.1000    0.0013
    ##     40        2.7078            -nan     0.1000   -0.0000
    ##     60        2.7055            -nan     0.1000    0.0000
    ##     80        2.7041            -nan     0.1000   -0.0000
    ##    100        2.7029            -nan     0.1000   -0.0000
    ##    120        2.7023            -nan     0.1000   -0.0000
    ##    140        2.7017            -nan     0.1000   -0.0000
    ##    150        2.7015            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9802            -nan     0.1000    0.0621
    ##      2        2.9293            -nan     0.1000    0.0504
    ##      3        2.8880            -nan     0.1000    0.0408
    ##      4        2.8551            -nan     0.1000    0.0330
    ##      5        2.8282            -nan     0.1000    0.0268
    ##      6        2.8063            -nan     0.1000    0.0216
    ##      7        2.7884            -nan     0.1000    0.0175
    ##      8        2.7739            -nan     0.1000    0.0141
    ##      9        2.7628            -nan     0.1000    0.0115
    ##     10        2.7530            -nan     0.1000    0.0092
    ##     20        2.7187            -nan     0.1000    0.0011
    ##     40        2.7091            -nan     0.1000    0.0002
    ##     60        2.7066            -nan     0.1000    0.0000
    ##     80        2.7056            -nan     0.1000    0.0000
    ##    100        2.7049            -nan     0.1000    0.0000
    ##    120        2.7044            -nan     0.1000    0.0000
    ##    140        2.7040            -nan     0.1000    0.0000
    ##    150        2.7038            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9783            -nan     0.1000    0.0630
    ##      2        2.9273            -nan     0.1000    0.0510
    ##      3        2.8856            -nan     0.1000    0.0415
    ##      4        2.8521            -nan     0.1000    0.0337
    ##      5        2.8250            -nan     0.1000    0.0271
    ##      6        2.8032            -nan     0.1000    0.0222
    ##      7        2.7861            -nan     0.1000    0.0179
    ##      8        2.7710            -nan     0.1000    0.0147
    ##      9        2.7593            -nan     0.1000    0.0119
    ##     10        2.7495            -nan     0.1000    0.0096
    ##     20        2.7124            -nan     0.1000    0.0012
    ##     40        2.7050            -nan     0.1000    0.0001
    ##     60        2.7028            -nan     0.1000    0.0000
    ##     80        2.7015            -nan     0.1000   -0.0000
    ##    100        2.7006            -nan     0.1000    0.0000
    ##    120        2.6999            -nan     0.1000   -0.0001
    ##    140        2.6994            -nan     0.1000    0.0000
    ##    150        2.6992            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9789            -nan     0.1000    0.0636
    ##      2        2.9271            -nan     0.1000    0.0513
    ##      3        2.8851            -nan     0.1000    0.0416
    ##      4        2.8513            -nan     0.1000    0.0339
    ##      5        2.8241            -nan     0.1000    0.0272
    ##      6        2.8022            -nan     0.1000    0.0221
    ##      7        2.7845            -nan     0.1000    0.0180
    ##      8        2.7698            -nan     0.1000    0.0146
    ##      9        2.7579            -nan     0.1000    0.0118
    ##     10        2.7481            -nan     0.1000    0.0096
    ##     20        2.7108            -nan     0.1000    0.0013
    ##     40        2.7033            -nan     0.1000    0.0001
    ##     60        2.7009            -nan     0.1000    0.0000
    ##     80        2.6993            -nan     0.1000   -0.0000
    ##    100        2.6982            -nan     0.1000   -0.0000
    ##    120        2.6975            -nan     0.1000   -0.0000
    ##    140        2.6970            -nan     0.1000   -0.0000
    ##    150        2.6968            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9801            -nan     0.1000    0.0614
    ##      2        2.9308            -nan     0.1000    0.0502
    ##      3        2.8900            -nan     0.1000    0.0407
    ##      4        2.8577            -nan     0.1000    0.0330
    ##      5        2.8317            -nan     0.1000    0.0269
    ##      6        2.8101            -nan     0.1000    0.0219
    ##      7        2.7923            -nan     0.1000    0.0177
    ##      8        2.7780            -nan     0.1000    0.0144
    ##      9        2.7665            -nan     0.1000    0.0117
    ##     10        2.7568            -nan     0.1000    0.0094
    ##     20        2.7217            -nan     0.1000    0.0012
    ##     40        2.7120            -nan     0.1000    0.0002
    ##     60        2.7095            -nan     0.1000    0.0000
    ##     80        2.7086            -nan     0.1000   -0.0000
    ##    100        2.7078            -nan     0.1000   -0.0000
    ##    120        2.7073            -nan     0.1000   -0.0000
    ##    140        2.7069            -nan     0.1000    0.0000
    ##    150        2.7067            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9793            -nan     0.1000    0.0622
    ##      2        2.9280            -nan     0.1000    0.0502
    ##      3        2.8869            -nan     0.1000    0.0408
    ##      4        2.8529            -nan     0.1000    0.0330
    ##      5        2.8261            -nan     0.1000    0.0268
    ##      6        2.8040            -nan     0.1000    0.0215
    ##      7        2.7865            -nan     0.1000    0.0175
    ##      8        2.7724            -nan     0.1000    0.0142
    ##      9        2.7606            -nan     0.1000    0.0115
    ##     10        2.7511            -nan     0.1000    0.0093
    ##     20        2.7154            -nan     0.1000    0.0012
    ##     40        2.7081            -nan     0.1000    0.0001
    ##     60        2.7062            -nan     0.1000   -0.0000
    ##     80        2.7049            -nan     0.1000    0.0000
    ##    100        2.7039            -nan     0.1000   -0.0000
    ##    120        2.7034            -nan     0.1000   -0.0000
    ##    140        2.7026            -nan     0.1000   -0.0000
    ##    150        2.7025            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9785            -nan     0.1000    0.0629
    ##      2        2.9275            -nan     0.1000    0.0508
    ##      3        2.8861            -nan     0.1000    0.0411
    ##      4        2.8531            -nan     0.1000    0.0335
    ##      5        2.8252            -nan     0.1000    0.0270
    ##      6        2.8030            -nan     0.1000    0.0217
    ##      7        2.7857            -nan     0.1000    0.0175
    ##      8        2.7712            -nan     0.1000    0.0143
    ##      9        2.7597            -nan     0.1000    0.0116
    ##     10        2.7503            -nan     0.1000    0.0094
    ##     20        2.7135            -nan     0.1000    0.0013
    ##     40        2.7059            -nan     0.1000    0.0002
    ##     60        2.7037            -nan     0.1000    0.0001
    ##     80        2.7024            -nan     0.1000   -0.0000
    ##    100        2.7015            -nan     0.1000   -0.0000
    ##    120        2.7007            -nan     0.1000   -0.0000
    ##    140        2.7002            -nan     0.1000   -0.0001
    ##    150        2.7000            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9799            -nan     0.1000    0.0615
    ##      2        2.9300            -nan     0.1000    0.0497
    ##      3        2.8897            -nan     0.1000    0.0402
    ##      4        2.8571            -nan     0.1000    0.0326
    ##      5        2.8311            -nan     0.1000    0.0264
    ##      6        2.8098            -nan     0.1000    0.0217
    ##      7        2.7926            -nan     0.1000    0.0174
    ##      8        2.7786            -nan     0.1000    0.0142
    ##      9        2.7673            -nan     0.1000    0.0116
    ##     10        2.7580            -nan     0.1000    0.0094
    ##     20        2.7227            -nan     0.1000    0.0011
    ##     40        2.7133            -nan     0.1000    0.0002
    ##     60        2.7109            -nan     0.1000    0.0000
    ##     80        2.7100            -nan     0.1000    0.0000
    ##    100        2.7093            -nan     0.1000    0.0000
    ##    120        2.7088            -nan     0.1000    0.0000
    ##    140        2.7084            -nan     0.1000   -0.0000
    ##    150        2.7082            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9790            -nan     0.1000    0.0620
    ##      2        2.9289            -nan     0.1000    0.0504
    ##      3        2.8881            -nan     0.1000    0.0409
    ##      4        2.8546            -nan     0.1000    0.0329
    ##      5        2.8282            -nan     0.1000    0.0269
    ##      6        2.8067            -nan     0.1000    0.0218
    ##      7        2.7886            -nan     0.1000    0.0176
    ##      8        2.7736            -nan     0.1000    0.0143
    ##      9        2.7618            -nan     0.1000    0.0115
    ##     10        2.7522            -nan     0.1000    0.0093
    ##     20        2.7170            -nan     0.1000    0.0012
    ##     40        2.7097            -nan     0.1000    0.0001
    ##     60        2.7076            -nan     0.1000   -0.0001
    ##     80        2.7062            -nan     0.1000    0.0001
    ##    100        2.7051            -nan     0.1000    0.0001
    ##    120        2.7045            -nan     0.1000   -0.0000
    ##    140        2.7040            -nan     0.1000   -0.0000
    ##    150        2.7037            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9790            -nan     0.1000    0.0625
    ##      2        2.9268            -nan     0.1000    0.0504
    ##      3        2.8858            -nan     0.1000    0.0408
    ##      4        2.8530            -nan     0.1000    0.0330
    ##      5        2.8263            -nan     0.1000    0.0268
    ##      6        2.8042            -nan     0.1000    0.0216
    ##      7        2.7867            -nan     0.1000    0.0174
    ##      8        2.7726            -nan     0.1000    0.0142
    ##      9        2.7608            -nan     0.1000    0.0115
    ##     10        2.7514            -nan     0.1000    0.0093
    ##     20        2.7150            -nan     0.1000    0.0012
    ##     40        2.7074            -nan     0.1000    0.0001
    ##     60        2.7050            -nan     0.1000    0.0001
    ##     80        2.7039            -nan     0.1000   -0.0000
    ##    100        2.7029            -nan     0.1000   -0.0000
    ##    120        2.7022            -nan     0.1000    0.0000
    ##    140        2.7015            -nan     0.1000   -0.0000
    ##    150        2.7013            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9787            -nan     0.1000    0.0610
    ##      2        2.9290            -nan     0.1000    0.0493
    ##      3        2.8890            -nan     0.1000    0.0400
    ##      4        2.8565            -nan     0.1000    0.0323
    ##      5        2.8306            -nan     0.1000    0.0262
    ##      6        2.8092            -nan     0.1000    0.0213
    ##      7        2.7913            -nan     0.1000    0.0172
    ##      8        2.7776            -nan     0.1000    0.0139
    ##      9        2.7663            -nan     0.1000    0.0113
    ##     10        2.7570            -nan     0.1000    0.0092
    ##     20        2.7229            -nan     0.1000    0.0011
    ##     40        2.7132            -nan     0.1000    0.0002
    ##     60        2.7107            -nan     0.1000    0.0000
    ##     80        2.7098            -nan     0.1000    0.0000
    ##    100        2.7091            -nan     0.1000    0.0000
    ##    120        2.7086            -nan     0.1000   -0.0000
    ##    140        2.7081            -nan     0.1000   -0.0000
    ##    150        2.7079            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9785            -nan     0.1000    0.0620
    ##      2        2.9278            -nan     0.1000    0.0504
    ##      3        2.8869            -nan     0.1000    0.0405
    ##      4        2.8535            -nan     0.1000    0.0329
    ##      5        2.8275            -nan     0.1000    0.0266
    ##      6        2.8058            -nan     0.1000    0.0217
    ##      7        2.7879            -nan     0.1000    0.0177
    ##      8        2.7735            -nan     0.1000    0.0141
    ##      9        2.7623            -nan     0.1000    0.0115
    ##     10        2.7527            -nan     0.1000    0.0095
    ##     20        2.7167            -nan     0.1000    0.0012
    ##     40        2.7090            -nan     0.1000    0.0002
    ##     60        2.7067            -nan     0.1000    0.0001
    ##     80        2.7054            -nan     0.1000   -0.0000
    ##    100        2.7044            -nan     0.1000    0.0000
    ##    120        2.7036            -nan     0.1000    0.0000
    ##    140        2.7029            -nan     0.1000   -0.0000
    ##    150        2.7026            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9782            -nan     0.1000    0.0622
    ##      2        2.9282            -nan     0.1000    0.0508
    ##      3        2.8874            -nan     0.1000    0.0412
    ##      4        2.8537            -nan     0.1000    0.0331
    ##      5        2.8272            -nan     0.1000    0.0269
    ##      6        2.8047            -nan     0.1000    0.0219
    ##      7        2.7869            -nan     0.1000    0.0176
    ##      8        2.7723            -nan     0.1000    0.0143
    ##      9        2.7607            -nan     0.1000    0.0116
    ##     10        2.7512            -nan     0.1000    0.0094
    ##     20        2.7145            -nan     0.1000    0.0011
    ##     40        2.7071            -nan     0.1000    0.0001
    ##     60        2.7044            -nan     0.1000    0.0000
    ##     80        2.7031            -nan     0.1000    0.0000
    ##    100        2.7021            -nan     0.1000   -0.0001
    ##    120        2.7013            -nan     0.1000   -0.0001
    ##    140        2.7007            -nan     0.1000   -0.0001
    ##    150        2.7004            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9798            -nan     0.1000    0.0612
    ##      2        2.9290            -nan     0.1000    0.0498
    ##      3        2.8885            -nan     0.1000    0.0402
    ##      4        2.8565            -nan     0.1000    0.0323
    ##      5        2.8304            -nan     0.1000    0.0264
    ##      6        2.8094            -nan     0.1000    0.0215
    ##      7        2.7920            -nan     0.1000    0.0173
    ##      8        2.7776            -nan     0.1000    0.0140
    ##      9        2.7661            -nan     0.1000    0.0113
    ##     10        2.7567            -nan     0.1000    0.0091
    ##     20        2.7224            -nan     0.1000    0.0011
    ##     40        2.7130            -nan     0.1000    0.0002
    ##     60        2.7105            -nan     0.1000    0.0001
    ##     80        2.7096            -nan     0.1000    0.0000
    ##    100        2.7088            -nan     0.1000   -0.0000
    ##    120        2.7084            -nan     0.1000   -0.0000
    ##    140        2.7079            -nan     0.1000   -0.0000
    ##    150        2.7077            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9797            -nan     0.1000    0.0623
    ##      2        2.9294            -nan     0.1000    0.0503
    ##      3        2.8881            -nan     0.1000    0.0411
    ##      4        2.8549            -nan     0.1000    0.0332
    ##      5        2.8285            -nan     0.1000    0.0270
    ##      6        2.8064            -nan     0.1000    0.0218
    ##      7        2.7889            -nan     0.1000    0.0177
    ##      8        2.7742            -nan     0.1000    0.0144
    ##      9        2.7626            -nan     0.1000    0.0116
    ##     10        2.7530            -nan     0.1000    0.0095
    ##     20        2.7166            -nan     0.1000    0.0012
    ##     40        2.7093            -nan     0.1000    0.0000
    ##     60        2.7071            -nan     0.1000   -0.0000
    ##     80        2.7058            -nan     0.1000    0.0000
    ##    100        2.7049            -nan     0.1000   -0.0000
    ##    120        2.7043            -nan     0.1000    0.0000
    ##    140        2.7039            -nan     0.1000   -0.0000
    ##    150        2.7036            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9788            -nan     0.1000    0.0629
    ##      2        2.9283            -nan     0.1000    0.0509
    ##      3        2.8875            -nan     0.1000    0.0406
    ##      4        2.8540            -nan     0.1000    0.0334
    ##      5        2.8269            -nan     0.1000    0.0269
    ##      6        2.8051            -nan     0.1000    0.0218
    ##      7        2.7874            -nan     0.1000    0.0177
    ##      8        2.7732            -nan     0.1000    0.0144
    ##      9        2.7617            -nan     0.1000    0.0117
    ##     10        2.7524            -nan     0.1000    0.0097
    ##     20        2.7145            -nan     0.1000    0.0012
    ##     40        2.7076            -nan     0.1000    0.0001
    ##     60        2.7050            -nan     0.1000   -0.0000
    ##     80        2.7036            -nan     0.1000    0.0000
    ##    100        2.7026            -nan     0.1000   -0.0000
    ##    120        2.7020            -nan     0.1000   -0.0001
    ##    140        2.7015            -nan     0.1000   -0.0000
    ##    150        2.7013            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        2.9790            -nan     0.1000    0.0628
    ##      2        2.9290            -nan     0.1000    0.0507
    ##      3        2.8877            -nan     0.1000    0.0410
    ##      4        2.8546            -nan     0.1000    0.0333
    ##      5        2.8275            -nan     0.1000    0.0272
    ##      6        2.8059            -nan     0.1000    0.0219
    ##      7        2.7880            -nan     0.1000    0.0178
    ##      8        2.7736            -nan     0.1000    0.0146
    ##      9        2.7612            -nan     0.1000    0.0118
    ##     10        2.7514            -nan     0.1000    0.0095
    ##     20        2.7152            -nan     0.1000    0.0013
    ##     40        2.7073            -nan     0.1000    0.0000
    ##     60        2.7049            -nan     0.1000    0.0000
    ##     80        2.7037            -nan     0.1000   -0.0000
    ##    100        2.7027            -nan     0.1000    0.0000
    ##    120        2.7019            -nan     0.1000   -0.0000
    ##    140        2.7015            -nan     0.1000   -0.0000
    ##    150        2.7012            -nan     0.1000   -0.0001

 

![Summary of Boosted Random Forest Model without including priority as a
predictor.](Technical_Report_files/figure-markdown_strict/summary%20of%20no%20priority-1.png)

    ##                           var rel.inf
    ## category_dummy category_dummy   92.68
    ## log_income         log_income    4.92
    ## Pct.White           Pct.White    2.40

 

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0390            -nan     0.1000    0.0005
    ##      2        3.0386            -nan     0.1000    0.0003
    ##      3        3.0382            -nan     0.1000    0.0004
    ##      4        3.0379            -nan     0.1000    0.0003
    ##      5        3.0376            -nan     0.1000    0.0003
    ##      6        3.0374            -nan     0.1000    0.0003
    ##      7        3.0372            -nan     0.1000    0.0002
    ##      8        3.0369            -nan     0.1000    0.0002
    ##      9        3.0367            -nan     0.1000    0.0002
    ##     10        3.0366            -nan     0.1000    0.0001
    ##     20        3.0355            -nan     0.1000    0.0000
    ##     40        3.0340            -nan     0.1000   -0.0000
    ##     60        3.0332            -nan     0.1000   -0.0000
    ##     80        3.0325            -nan     0.1000    0.0000
    ##    100        3.0319            -nan     0.1000    0.0000
    ##    120        3.0315            -nan     0.1000   -0.0000
    ##    140        3.0310            -nan     0.1000    0.0000
    ##    150        3.0309            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0385            -nan     0.1000    0.0009
    ##      2        3.0378            -nan     0.1000    0.0007
    ##      3        3.0371            -nan     0.1000    0.0007
    ##      4        3.0365            -nan     0.1000    0.0005
    ##      5        3.0359            -nan     0.1000    0.0005
    ##      6        3.0355            -nan     0.1000    0.0002
    ##      7        3.0352            -nan     0.1000    0.0002
    ##      8        3.0349            -nan     0.1000    0.0002
    ##      9        3.0345            -nan     0.1000    0.0004
    ##     10        3.0343            -nan     0.1000    0.0001
    ##     20        3.0325            -nan     0.1000    0.0001
    ##     40        3.0301            -nan     0.1000    0.0000
    ##     60        3.0281            -nan     0.1000   -0.0000
    ##     80        3.0272            -nan     0.1000   -0.0000
    ##    100        3.0262            -nan     0.1000   -0.0001
    ##    120        3.0254            -nan     0.1000    0.0000
    ##    140        3.0247            -nan     0.1000   -0.0000
    ##    150        3.0245            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0383            -nan     0.1000    0.0013
    ##      2        3.0373            -nan     0.1000    0.0010
    ##      3        3.0364            -nan     0.1000    0.0008
    ##      4        3.0356            -nan     0.1000    0.0006
    ##      5        3.0350            -nan     0.1000    0.0006
    ##      6        3.0345            -nan     0.1000    0.0004
    ##      7        3.0340            -nan     0.1000    0.0004
    ##      8        3.0336            -nan     0.1000    0.0003
    ##      9        3.0332            -nan     0.1000    0.0003
    ##     10        3.0328            -nan     0.1000    0.0003
    ##     20        3.0308            -nan     0.1000    0.0000
    ##     40        3.0281            -nan     0.1000    0.0000
    ##     60        3.0264            -nan     0.1000   -0.0000
    ##     80        3.0252            -nan     0.1000   -0.0001
    ##    100        3.0241            -nan     0.1000   -0.0001
    ##    120        3.0234            -nan     0.1000   -0.0000
    ##    140        3.0229            -nan     0.1000   -0.0000
    ##    150        3.0228            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0416            -nan     0.1000    0.0006
    ##      2        3.0411            -nan     0.1000    0.0004
    ##      3        3.0407            -nan     0.1000    0.0004
    ##      4        3.0405            -nan     0.1000    0.0003
    ##      5        3.0401            -nan     0.1000    0.0003
    ##      6        3.0398            -nan     0.1000    0.0003
    ##      7        3.0396            -nan     0.1000    0.0002
    ##      8        3.0394            -nan     0.1000    0.0002
    ##      9        3.0392            -nan     0.1000    0.0001
    ##     10        3.0390            -nan     0.1000    0.0001
    ##     20        3.0379            -nan     0.1000   -0.0000
    ##     40        3.0366            -nan     0.1000   -0.0000
    ##     60        3.0356            -nan     0.1000    0.0000
    ##     80        3.0350            -nan     0.1000    0.0000
    ##    100        3.0344            -nan     0.1000    0.0000
    ##    120        3.0339            -nan     0.1000   -0.0001
    ##    140        3.0336            -nan     0.1000   -0.0000
    ##    150        3.0334            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0412            -nan     0.1000    0.0009
    ##      2        3.0403            -nan     0.1000    0.0007
    ##      3        3.0397            -nan     0.1000    0.0006
    ##      4        3.0392            -nan     0.1000    0.0005
    ##      5        3.0388            -nan     0.1000    0.0004
    ##      6        3.0384            -nan     0.1000    0.0004
    ##      7        3.0380            -nan     0.1000    0.0004
    ##      8        3.0376            -nan     0.1000    0.0003
    ##      9        3.0374            -nan     0.1000    0.0002
    ##     10        3.0371            -nan     0.1000    0.0001
    ##     20        3.0352            -nan     0.1000    0.0001
    ##     40        3.0328            -nan     0.1000    0.0001
    ##     60        3.0314            -nan     0.1000   -0.0000
    ##     80        3.0298            -nan     0.1000   -0.0001
    ##    100        3.0288            -nan     0.1000   -0.0000
    ##    120        3.0280            -nan     0.1000    0.0000
    ##    140        3.0276            -nan     0.1000   -0.0001
    ##    150        3.0274            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0407            -nan     0.1000    0.0012
    ##      2        3.0397            -nan     0.1000    0.0010
    ##      3        3.0389            -nan     0.1000    0.0007
    ##      4        3.0382            -nan     0.1000    0.0006
    ##      5        3.0377            -nan     0.1000    0.0005
    ##      6        3.0373            -nan     0.1000    0.0004
    ##      7        3.0369            -nan     0.1000    0.0005
    ##      8        3.0366            -nan     0.1000    0.0002
    ##      9        3.0363            -nan     0.1000    0.0003
    ##     10        3.0357            -nan     0.1000    0.0003
    ##     20        3.0333            -nan     0.1000    0.0001
    ##     40        3.0306            -nan     0.1000    0.0000
    ##     60        3.0286            -nan     0.1000   -0.0000
    ##     80        3.0274            -nan     0.1000   -0.0000
    ##    100        3.0267            -nan     0.1000   -0.0000
    ##    120        3.0260            -nan     0.1000    0.0000
    ##    140        3.0255            -nan     0.1000   -0.0000
    ##    150        3.0253            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0410            -nan     0.1000    0.0004
    ##      2        3.0404            -nan     0.1000    0.0006
    ##      3        3.0399            -nan     0.1000    0.0005
    ##      4        3.0395            -nan     0.1000    0.0004
    ##      5        3.0392            -nan     0.1000    0.0003
    ##      6        3.0389            -nan     0.1000    0.0002
    ##      7        3.0386            -nan     0.1000    0.0003
    ##      8        3.0383            -nan     0.1000    0.0002
    ##      9        3.0381            -nan     0.1000    0.0002
    ##     10        3.0380            -nan     0.1000    0.0001
    ##     20        3.0367            -nan     0.1000    0.0000
    ##     40        3.0353            -nan     0.1000    0.0000
    ##     60        3.0345            -nan     0.1000   -0.0001
    ##     80        3.0338            -nan     0.1000   -0.0000
    ##    100        3.0331            -nan     0.1000    0.0000
    ##    120        3.0327            -nan     0.1000   -0.0000
    ##    140        3.0323            -nan     0.1000   -0.0000
    ##    150        3.0322            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0405            -nan     0.1000    0.0010
    ##      2        3.0397            -nan     0.1000    0.0008
    ##      3        3.0390            -nan     0.1000    0.0007
    ##      4        3.0384            -nan     0.1000    0.0006
    ##      5        3.0379            -nan     0.1000    0.0005
    ##      6        3.0374            -nan     0.1000    0.0004
    ##      7        3.0371            -nan     0.1000    0.0003
    ##      8        3.0366            -nan     0.1000    0.0004
    ##      9        3.0363            -nan     0.1000    0.0002
    ##     10        3.0360            -nan     0.1000    0.0003
    ##     20        3.0339            -nan     0.1000    0.0001
    ##     40        3.0314            -nan     0.1000    0.0001
    ##     60        3.0298            -nan     0.1000    0.0001
    ##     80        3.0284            -nan     0.1000    0.0000
    ##    100        3.0274            -nan     0.1000   -0.0000
    ##    120        3.0266            -nan     0.1000    0.0000
    ##    140        3.0260            -nan     0.1000   -0.0000
    ##    150        3.0257            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0401            -nan     0.1000    0.0013
    ##      2        3.0390            -nan     0.1000    0.0011
    ##      3        3.0382            -nan     0.1000    0.0009
    ##      4        3.0375            -nan     0.1000    0.0006
    ##      5        3.0370            -nan     0.1000    0.0006
    ##      6        3.0365            -nan     0.1000    0.0005
    ##      7        3.0360            -nan     0.1000    0.0004
    ##      8        3.0357            -nan     0.1000    0.0003
    ##      9        3.0352            -nan     0.1000    0.0003
    ##     10        3.0348            -nan     0.1000    0.0003
    ##     20        3.0321            -nan     0.1000    0.0002
    ##     40        3.0290            -nan     0.1000    0.0001
    ##     60        3.0271            -nan     0.1000   -0.0000
    ##     80        3.0258            -nan     0.1000    0.0000
    ##    100        3.0251            -nan     0.1000   -0.0000
    ##    120        3.0246            -nan     0.1000   -0.0001
    ##    140        3.0240            -nan     0.1000   -0.0001
    ##    150        3.0238            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0431            -nan     0.1000    0.0005
    ##      2        3.0426            -nan     0.1000    0.0004
    ##      3        3.0422            -nan     0.1000    0.0003
    ##      4        3.0419            -nan     0.1000    0.0003
    ##      5        3.0415            -nan     0.1000    0.0003
    ##      6        3.0412            -nan     0.1000    0.0002
    ##      7        3.0410            -nan     0.1000    0.0003
    ##      8        3.0407            -nan     0.1000    0.0002
    ##      9        3.0406            -nan     0.1000    0.0001
    ##     10        3.0404            -nan     0.1000    0.0001
    ##     20        3.0393            -nan     0.1000    0.0001
    ##     40        3.0379            -nan     0.1000    0.0000
    ##     60        3.0370            -nan     0.1000    0.0000
    ##     80        3.0363            -nan     0.1000   -0.0000
    ##    100        3.0358            -nan     0.1000   -0.0000
    ##    120        3.0353            -nan     0.1000   -0.0000
    ##    140        3.0348            -nan     0.1000   -0.0000
    ##    150        3.0347            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0426            -nan     0.1000    0.0010
    ##      2        3.0419            -nan     0.1000    0.0005
    ##      3        3.0413            -nan     0.1000    0.0006
    ##      4        3.0406            -nan     0.1000    0.0006
    ##      5        3.0400            -nan     0.1000    0.0005
    ##      6        3.0397            -nan     0.1000    0.0004
    ##      7        3.0392            -nan     0.1000    0.0004
    ##      8        3.0389            -nan     0.1000    0.0002
    ##      9        3.0385            -nan     0.1000    0.0002
    ##     10        3.0382            -nan     0.1000    0.0003
    ##     20        3.0361            -nan     0.1000    0.0001
    ##     40        3.0338            -nan     0.1000    0.0000
    ##     60        3.0324            -nan     0.1000   -0.0000
    ##     80        3.0313            -nan     0.1000   -0.0000
    ##    100        3.0303            -nan     0.1000   -0.0000
    ##    120        3.0293            -nan     0.1000   -0.0000
    ##    140        3.0288            -nan     0.1000   -0.0001
    ##    150        3.0286            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0421            -nan     0.1000    0.0013
    ##      2        3.0411            -nan     0.1000    0.0010
    ##      3        3.0403            -nan     0.1000    0.0008
    ##      4        3.0396            -nan     0.1000    0.0007
    ##      5        3.0390            -nan     0.1000    0.0005
    ##      6        3.0385            -nan     0.1000    0.0004
    ##      7        3.0381            -nan     0.1000    0.0003
    ##      8        3.0376            -nan     0.1000    0.0005
    ##      9        3.0373            -nan     0.1000    0.0002
    ##     10        3.0370            -nan     0.1000    0.0001
    ##     20        3.0343            -nan     0.1000    0.0000
    ##     40        3.0313            -nan     0.1000   -0.0001
    ##     60        3.0295            -nan     0.1000    0.0000
    ##     80        3.0283            -nan     0.1000    0.0000
    ##    100        3.0276            -nan     0.1000   -0.0000
    ##    120        3.0271            -nan     0.1000   -0.0000
    ##    140        3.0267            -nan     0.1000   -0.0001
    ##    150        3.0266            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0403            -nan     0.1000    0.0006
    ##      2        3.0398            -nan     0.1000    0.0005
    ##      3        3.0395            -nan     0.1000    0.0001
    ##      4        3.0392            -nan     0.1000    0.0003
    ##      5        3.0389            -nan     0.1000    0.0004
    ##      6        3.0386            -nan     0.1000    0.0003
    ##      7        3.0383            -nan     0.1000    0.0003
    ##      8        3.0381            -nan     0.1000    0.0002
    ##      9        3.0378            -nan     0.1000    0.0002
    ##     10        3.0377            -nan     0.1000    0.0002
    ##     20        3.0364            -nan     0.1000    0.0000
    ##     40        3.0350            -nan     0.1000    0.0000
    ##     60        3.0340            -nan     0.1000    0.0000
    ##     80        3.0333            -nan     0.1000   -0.0000
    ##    100        3.0327            -nan     0.1000   -0.0000
    ##    120        3.0323            -nan     0.1000   -0.0000
    ##    140        3.0320            -nan     0.1000   -0.0001
    ##    150        3.0317            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0399            -nan     0.1000    0.0009
    ##      2        3.0390            -nan     0.1000    0.0009
    ##      3        3.0384            -nan     0.1000    0.0006
    ##      4        3.0377            -nan     0.1000    0.0006
    ##      5        3.0371            -nan     0.1000    0.0005
    ##      6        3.0366            -nan     0.1000    0.0003
    ##      7        3.0362            -nan     0.1000    0.0003
    ##      8        3.0360            -nan     0.1000    0.0002
    ##      9        3.0357            -nan     0.1000    0.0002
    ##     10        3.0354            -nan     0.1000    0.0002
    ##     20        3.0334            -nan     0.1000    0.0002
    ##     40        3.0308            -nan     0.1000    0.0000
    ##     60        3.0290            -nan     0.1000    0.0000
    ##     80        3.0278            -nan     0.1000    0.0001
    ##    100        3.0268            -nan     0.1000   -0.0000
    ##    120        3.0262            -nan     0.1000   -0.0000
    ##    140        3.0256            -nan     0.1000    0.0001
    ##    150        3.0253            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0395            -nan     0.1000    0.0013
    ##      2        3.0385            -nan     0.1000    0.0011
    ##      3        3.0376            -nan     0.1000    0.0008
    ##      4        3.0369            -nan     0.1000    0.0007
    ##      5        3.0362            -nan     0.1000    0.0006
    ##      6        3.0358            -nan     0.1000    0.0003
    ##      7        3.0355            -nan     0.1000    0.0002
    ##      8        3.0352            -nan     0.1000    0.0001
    ##      9        3.0348            -nan     0.1000    0.0004
    ##     10        3.0345            -nan     0.1000    0.0003
    ##     20        3.0319            -nan     0.1000    0.0001
    ##     40        3.0290            -nan     0.1000    0.0000
    ##     60        3.0271            -nan     0.1000    0.0000
    ##     80        3.0260            -nan     0.1000    0.0000
    ##    100        3.0250            -nan     0.1000   -0.0001
    ##    120        3.0243            -nan     0.1000   -0.0000
    ##    140        3.0239            -nan     0.1000   -0.0001
    ##    150        3.0237            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0400            -nan     0.1000    0.0006
    ##      2        3.0394            -nan     0.1000    0.0004
    ##      3        3.0390            -nan     0.1000    0.0003
    ##      4        3.0387            -nan     0.1000    0.0003
    ##      5        3.0385            -nan     0.1000    0.0003
    ##      6        3.0382            -nan     0.1000    0.0002
    ##      7        3.0380            -nan     0.1000    0.0002
    ##      8        3.0378            -nan     0.1000    0.0001
    ##      9        3.0376            -nan     0.1000    0.0002
    ##     10        3.0374            -nan     0.1000    0.0001
    ##     20        3.0363            -nan     0.1000    0.0001
    ##     40        3.0349            -nan     0.1000    0.0000
    ##     60        3.0340            -nan     0.1000   -0.0000
    ##     80        3.0333            -nan     0.1000   -0.0000
    ##    100        3.0328            -nan     0.1000    0.0000
    ##    120        3.0323            -nan     0.1000   -0.0000
    ##    140        3.0318            -nan     0.1000   -0.0000
    ##    150        3.0316            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0396            -nan     0.1000    0.0010
    ##      2        3.0388            -nan     0.1000    0.0008
    ##      3        3.0383            -nan     0.1000    0.0005
    ##      4        3.0377            -nan     0.1000    0.0006
    ##      5        3.0372            -nan     0.1000    0.0005
    ##      6        3.0367            -nan     0.1000    0.0004
    ##      7        3.0363            -nan     0.1000    0.0004
    ##      8        3.0360            -nan     0.1000    0.0002
    ##      9        3.0358            -nan     0.1000    0.0002
    ##     10        3.0355            -nan     0.1000    0.0002
    ##     20        3.0332            -nan     0.1000    0.0002
    ##     40        3.0308            -nan     0.1000    0.0001
    ##     60        3.0293            -nan     0.1000    0.0000
    ##     80        3.0278            -nan     0.1000    0.0000
    ##    100        3.0269            -nan     0.1000   -0.0001
    ##    120        3.0260            -nan     0.1000    0.0000
    ##    140        3.0254            -nan     0.1000    0.0000
    ##    150        3.0252            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0391            -nan     0.1000    0.0013
    ##      2        3.0382            -nan     0.1000    0.0010
    ##      3        3.0374            -nan     0.1000    0.0008
    ##      4        3.0367            -nan     0.1000    0.0006
    ##      5        3.0362            -nan     0.1000    0.0004
    ##      6        3.0357            -nan     0.1000    0.0005
    ##      7        3.0353            -nan     0.1000    0.0005
    ##      8        3.0348            -nan     0.1000    0.0003
    ##      9        3.0344            -nan     0.1000    0.0003
    ##     10        3.0341            -nan     0.1000    0.0002
    ##     20        3.0317            -nan     0.1000    0.0002
    ##     40        3.0285            -nan     0.1000    0.0001
    ##     60        3.0267            -nan     0.1000   -0.0000
    ##     80        3.0255            -nan     0.1000   -0.0001
    ##    100        3.0247            -nan     0.1000   -0.0001
    ##    120        3.0240            -nan     0.1000    0.0000
    ##    140        3.0235            -nan     0.1000   -0.0001
    ##    150        3.0234            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0426            -nan     0.1000    0.0003
    ##      2        3.0420            -nan     0.1000    0.0006
    ##      3        3.0416            -nan     0.1000    0.0005
    ##      4        3.0412            -nan     0.1000    0.0004
    ##      5        3.0409            -nan     0.1000    0.0002
    ##      6        3.0406            -nan     0.1000    0.0003
    ##      7        3.0404            -nan     0.1000    0.0002
    ##      8        3.0402            -nan     0.1000    0.0002
    ##      9        3.0399            -nan     0.1000    0.0002
    ##     10        3.0398            -nan     0.1000    0.0001
    ##     20        3.0387            -nan     0.1000    0.0000
    ##     40        3.0374            -nan     0.1000    0.0000
    ##     60        3.0366            -nan     0.1000   -0.0000
    ##     80        3.0359            -nan     0.1000    0.0000
    ##    100        3.0353            -nan     0.1000    0.0000
    ##    120        3.0349            -nan     0.1000   -0.0000
    ##    140        3.0344            -nan     0.1000    0.0000
    ##    150        3.0343            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0420            -nan     0.1000    0.0010
    ##      2        3.0412            -nan     0.1000    0.0008
    ##      3        3.0404            -nan     0.1000    0.0006
    ##      4        3.0399            -nan     0.1000    0.0006
    ##      5        3.0394            -nan     0.1000    0.0005
    ##      6        3.0389            -nan     0.1000    0.0004
    ##      7        3.0385            -nan     0.1000    0.0003
    ##      8        3.0383            -nan     0.1000    0.0002
    ##      9        3.0379            -nan     0.1000    0.0003
    ##     10        3.0377            -nan     0.1000    0.0002
    ##     20        3.0359            -nan     0.1000   -0.0000
    ##     40        3.0334            -nan     0.1000    0.0001
    ##     60        3.0319            -nan     0.1000   -0.0000
    ##     80        3.0308            -nan     0.1000   -0.0000
    ##    100        3.0298            -nan     0.1000    0.0000
    ##    120        3.0290            -nan     0.1000   -0.0000
    ##    140        3.0283            -nan     0.1000   -0.0000
    ##    150        3.0279            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0418            -nan     0.1000    0.0013
    ##      2        3.0409            -nan     0.1000    0.0008
    ##      3        3.0401            -nan     0.1000    0.0008
    ##      4        3.0394            -nan     0.1000    0.0006
    ##      5        3.0387            -nan     0.1000    0.0007
    ##      6        3.0382            -nan     0.1000    0.0006
    ##      7        3.0376            -nan     0.1000    0.0004
    ##      8        3.0374            -nan     0.1000    0.0002
    ##      9        3.0370            -nan     0.1000    0.0003
    ##     10        3.0367            -nan     0.1000    0.0003
    ##     20        3.0346            -nan     0.1000    0.0001
    ##     40        3.0316            -nan     0.1000    0.0001
    ##     60        3.0299            -nan     0.1000    0.0000
    ##     80        3.0285            -nan     0.1000   -0.0000
    ##    100        3.0275            -nan     0.1000   -0.0000
    ##    120        3.0270            -nan     0.1000   -0.0000
    ##    140        3.0266            -nan     0.1000    0.0000
    ##    150        3.0264            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0398            -nan     0.1000    0.0005
    ##      2        3.0393            -nan     0.1000    0.0004
    ##      3        3.0389            -nan     0.1000    0.0003
    ##      4        3.0386            -nan     0.1000    0.0003
    ##      5        3.0383            -nan     0.1000    0.0003
    ##      6        3.0381            -nan     0.1000    0.0002
    ##      7        3.0378            -nan     0.1000    0.0002
    ##      8        3.0376            -nan     0.1000    0.0002
    ##      9        3.0374            -nan     0.1000    0.0002
    ##     10        3.0372            -nan     0.1000    0.0001
    ##     20        3.0361            -nan     0.1000    0.0001
    ##     40        3.0348            -nan     0.1000    0.0000
    ##     60        3.0339            -nan     0.1000    0.0000
    ##     80        3.0332            -nan     0.1000    0.0000
    ##    100        3.0327            -nan     0.1000   -0.0000
    ##    120        3.0321            -nan     0.1000   -0.0000
    ##    140        3.0317            -nan     0.1000   -0.0000
    ##    150        3.0314            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0396            -nan     0.1000    0.0009
    ##      2        3.0387            -nan     0.1000    0.0008
    ##      3        3.0380            -nan     0.1000    0.0006
    ##      4        3.0375            -nan     0.1000    0.0006
    ##      5        3.0370            -nan     0.1000    0.0005
    ##      6        3.0367            -nan     0.1000    0.0004
    ##      7        3.0363            -nan     0.1000    0.0004
    ##      8        3.0360            -nan     0.1000    0.0002
    ##      9        3.0356            -nan     0.1000    0.0004
    ##     10        3.0353            -nan     0.1000    0.0002
    ##     20        3.0333            -nan     0.1000    0.0001
    ##     40        3.0308            -nan     0.1000   -0.0000
    ##     60        3.0287            -nan     0.1000    0.0000
    ##     80        3.0278            -nan     0.1000   -0.0000
    ##    100        3.0269            -nan     0.1000   -0.0001
    ##    120        3.0261            -nan     0.1000   -0.0001
    ##    140        3.0254            -nan     0.1000   -0.0000
    ##    150        3.0251            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0391            -nan     0.1000    0.0013
    ##      2        3.0382            -nan     0.1000    0.0009
    ##      3        3.0373            -nan     0.1000    0.0008
    ##      4        3.0366            -nan     0.1000    0.0007
    ##      5        3.0360            -nan     0.1000    0.0006
    ##      6        3.0355            -nan     0.1000    0.0005
    ##      7        3.0349            -nan     0.1000    0.0003
    ##      8        3.0345            -nan     0.1000    0.0003
    ##      9        3.0343            -nan     0.1000    0.0001
    ##     10        3.0340            -nan     0.1000    0.0002
    ##     20        3.0316            -nan     0.1000    0.0000
    ##     40        3.0286            -nan     0.1000    0.0001
    ##     60        3.0266            -nan     0.1000    0.0000
    ##     80        3.0256            -nan     0.1000    0.0000
    ##    100        3.0248            -nan     0.1000   -0.0000
    ##    120        3.0241            -nan     0.1000   -0.0001
    ##    140        3.0236            -nan     0.1000   -0.0000
    ##    150        3.0234            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0411            -nan     0.1000    0.0005
    ##      2        3.0406            -nan     0.1000    0.0004
    ##      3        3.0403            -nan     0.1000    0.0003
    ##      4        3.0400            -nan     0.1000    0.0002
    ##      5        3.0397            -nan     0.1000    0.0003
    ##      6        3.0394            -nan     0.1000    0.0003
    ##      7        3.0392            -nan     0.1000    0.0002
    ##      8        3.0390            -nan     0.1000    0.0002
    ##      9        3.0388            -nan     0.1000    0.0001
    ##     10        3.0386            -nan     0.1000    0.0001
    ##     20        3.0375            -nan     0.1000    0.0001
    ##     40        3.0364            -nan     0.1000   -0.0000
    ##     60        3.0355            -nan     0.1000   -0.0000
    ##     80        3.0348            -nan     0.1000    0.0000
    ##    100        3.0343            -nan     0.1000    0.0000
    ##    120        3.0338            -nan     0.1000   -0.0000
    ##    140        3.0334            -nan     0.1000   -0.0000
    ##    150        3.0332            -nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0406            -nan     0.1000    0.0009
    ##      2        3.0399            -nan     0.1000    0.0007
    ##      3        3.0394            -nan     0.1000    0.0005
    ##      4        3.0389            -nan     0.1000    0.0005
    ##      5        3.0384            -nan     0.1000    0.0004
    ##      6        3.0379            -nan     0.1000    0.0005
    ##      7        3.0375            -nan     0.1000    0.0003
    ##      8        3.0371            -nan     0.1000    0.0003
    ##      9        3.0369            -nan     0.1000    0.0002
    ##     10        3.0367            -nan     0.1000    0.0001
    ##     20        3.0348            -nan     0.1000    0.0000
    ##     40        3.0325            -nan     0.1000   -0.0000
    ##     60        3.0306            -nan     0.1000    0.0001
    ##     80        3.0293            -nan     0.1000   -0.0000
    ##    100        3.0283            -nan     0.1000   -0.0000
    ##    120        3.0279            -nan     0.1000   -0.0000
    ##    140        3.0272            -nan     0.1000   -0.0000
    ##    150        3.0270            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0404            -nan     0.1000    0.0012
    ##      2        3.0395            -nan     0.1000    0.0010
    ##      3        3.0390            -nan     0.1000    0.0003
    ##      4        3.0381            -nan     0.1000    0.0007
    ##      5        3.0375            -nan     0.1000    0.0006
    ##      6        3.0370            -nan     0.1000    0.0005
    ##      7        3.0367            -nan     0.1000    0.0003
    ##      8        3.0362            -nan     0.1000    0.0004
    ##      9        3.0358            -nan     0.1000    0.0003
    ##     10        3.0355            -nan     0.1000    0.0003
    ##     20        3.0332            -nan     0.1000    0.0000
    ##     40        3.0302            -nan     0.1000   -0.0000
    ##     60        3.0283            -nan     0.1000    0.0001
    ##     80        3.0270            -nan     0.1000   -0.0000
    ##    100        3.0262            -nan     0.1000   -0.0000
    ##    120        3.0256            -nan     0.1000   -0.0001
    ##    140        3.0254            -nan     0.1000   -0.0001
    ##    150        3.0252            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0367            -nan     0.1000    0.0005
    ##      2        3.0362            -nan     0.1000    0.0004
    ##      3        3.0359            -nan     0.1000    0.0004
    ##      4        3.0355            -nan     0.1000    0.0003
    ##      5        3.0352            -nan     0.1000    0.0003
    ##      6        3.0350            -nan     0.1000    0.0002
    ##      7        3.0348            -nan     0.1000    0.0002
    ##      8        3.0346            -nan     0.1000    0.0002
    ##      9        3.0344            -nan     0.1000    0.0001
    ##     10        3.0343            -nan     0.1000    0.0002
    ##     20        3.0332            -nan     0.1000    0.0000
    ##     40        3.0319            -nan     0.1000    0.0000
    ##     60        3.0310            -nan     0.1000   -0.0000
    ##     80        3.0303            -nan     0.1000   -0.0000
    ##    100        3.0297            -nan     0.1000   -0.0000
    ##    120        3.0292            -nan     0.1000    0.0000
    ##    140        3.0288            -nan     0.1000   -0.0000
    ##    150        3.0286            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0365            -nan     0.1000    0.0008
    ##      2        3.0357            -nan     0.1000    0.0006
    ##      3        3.0349            -nan     0.1000    0.0007
    ##      4        3.0344            -nan     0.1000    0.0006
    ##      5        3.0339            -nan     0.1000    0.0004
    ##      6        3.0335            -nan     0.1000    0.0004
    ##      7        3.0331            -nan     0.1000    0.0003
    ##      8        3.0329            -nan     0.1000    0.0002
    ##      9        3.0325            -nan     0.1000    0.0003
    ##     10        3.0322            -nan     0.1000    0.0002
    ##     20        3.0305            -nan     0.1000    0.0000
    ##     40        3.0276            -nan     0.1000    0.0000
    ##     60        3.0261            -nan     0.1000   -0.0000
    ##     80        3.0248            -nan     0.1000    0.0000
    ##    100        3.0238            -nan     0.1000    0.0001
    ##    120        3.0229            -nan     0.1000   -0.0001
    ##    140        3.0222            -nan     0.1000   -0.0000
    ##    150        3.0220            -nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0361            -nan     0.1000    0.0012
    ##      2        3.0352            -nan     0.1000    0.0010
    ##      3        3.0344            -nan     0.1000    0.0006
    ##      4        3.0336            -nan     0.1000    0.0007
    ##      5        3.0329            -nan     0.1000    0.0004
    ##      6        3.0324            -nan     0.1000    0.0004
    ##      7        3.0321            -nan     0.1000    0.0003
    ##      8        3.0317            -nan     0.1000    0.0003
    ##      9        3.0314            -nan     0.1000    0.0002
    ##     10        3.0310            -nan     0.1000    0.0004
    ##     20        3.0285            -nan     0.1000    0.0001
    ##     40        3.0254            -nan     0.1000   -0.0000
    ##     60        3.0235            -nan     0.1000   -0.0000
    ##     80        3.0222            -nan     0.1000    0.0000
    ##    100        3.0211            -nan     0.1000   -0.0001
    ##    120        3.0206            -nan     0.1000   -0.0000
    ##    140        3.0201            -nan     0.1000   -0.0001
    ##    150        3.0200            -nan     0.1000   -0.0001
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        3.0397            -nan     0.1000    0.0013
    ##      2        3.0389            -nan     0.1000    0.0009
    ##      3        3.0381            -nan     0.1000    0.0008
    ##      4        3.0372            -nan     0.1000    0.0007
    ##      5        3.0368            -nan     0.1000    0.0004
    ##      6        3.0363            -nan     0.1000    0.0005
    ##      7        3.0358            -nan     0.1000    0.0004
    ##      8        3.0353            -nan     0.1000    0.0003
    ##      9        3.0349            -nan     0.1000    0.0003
    ##     10        3.0346            -nan     0.1000    0.0004
    ##     20        3.0323            -nan     0.1000    0.0001
    ##     40        3.0295            -nan     0.1000    0.0001
    ##     60        3.0274            -nan     0.1000    0.0000
    ##     80        3.0263            -nan     0.1000    0.0000
    ##    100        3.0256            -nan     0.1000   -0.0001
    ##    120        3.0249            -nan     0.1000   -0.0001
    ##    140        3.0245            -nan     0.1000   -0.0000
    ##    150        3.0243            -nan     0.1000   -0.0001

 

![Summary of Boosted Random Forest Model without including priority and
call
type](Technical_Report_files/figure-markdown_strict/Summary%20of%20Model%20Without%20Priority-1.png)

    ##                   var rel.inf
    ## log_income log_income    57.7
    ## Pct.White   Pct.White    42.3

 

Looking at our best linear regression and penalized regression models,
we get more insights for inference. In the non-penalized model, all of
the variables are significant at the 0.001 significance level. However,
the coefficient for priority is much, much higher than the other
coefficients. The penalized regression reduced the coefficients of all
of the variables but did not make any of them zero, so we cannot fully
discount any of the four variables.  

    ## 
    ## Call:
    ## lm(formula = .outcome ~ ., data = dat)
    ## 
    ## Residuals:
    ##    Min     1Q Median     3Q    Max 
    ## -6.036 -0.804 -0.017  0.811  5.535 
    ## 
    ## Coefficients:
    ##                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)     6.639560   0.020584  322.56  < 2e-16 ***
    ## Pct.White      -0.389505   0.028017  -13.90  < 2e-16 ***
    ## log_income      0.006940   0.000891    7.79  6.7e-15 ***
    ## category_dummy  0.325191   0.007685   42.32  < 2e-16 ***
    ## priority_dummy -2.855008   0.007944 -359.40  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.19 on 138805 degrees of freedom
    ## Multiple R-squared:  0.538,  Adjusted R-squared:  0.538 
    ## F-statistic: 4.04e+04 on 4 and 138805 DF,  p-value: <2e-16

    ## 5 x 1 sparse Matrix of class "dgCMatrix"
    ##                       1
    ## (Intercept)     6.60803
    ## Pct.White      -0.35320
    ## log_income      0.00587
    ## category_dummy  0.32004
    ## priority_dummy -2.84212

 

Discussion
----------

  Unfortunately we were limited by the amount of information contained
in the data. More descriptive information, like the day and time of the
call, would have been very helpful in determining the volume of calls at
any given time, which likely has a strong relationship to time in the
queue.

\#\#References 2010 Census Data for Portland Neighborhoods | The City of
Portland, Oregon. (n.d.). Retrieved December 11, 2019, from The City Of
Portland Oregon website: <https://www.portlandoregon.gov/civic/56897>
DeNies, R. (2016, April 1). Portland Neighborhoods by the Numbers 2016:
The City. Retrieved December 11, 2019, from Portland Monthly website:
<https://www.pdxmonthly.com/articles/2016/4/1/real-estate-2016-the-city>
Dispatched Calls | The City of Portland, Oregon. (n.d.). Retrieved
December 11, 2019, from The City Of Portland Oregon website:
<https://www.portlandoregon.gov/police/76454>  

R Code
======

We have included all of the r chunks below, in order of use in the
document. In the document, the chunks will be referenced by the name
included in the r chunk heading.

 

    if(!require(ggfortify)){
        install.packages("ggforitfy")
    }
    library(ggfortify)
    if(!require(glmnet)){
        install.packages("glmnet")
    }
    library(glmnet)
    if(!require(MLmetrics)){
        install.packages("MLmetrics")
    }
    library(MLmetrics)
    library(tree)
    if(!require(caret)){
        install.packages("caret")
    }
    library(caret)
    if(!require(randomForest)){
        install.packages("randomForest")
    }
    library(randomForest)
    if(!require(gbm)){
        install.packages("gbm")
    }
    library(gbm)
    if(!require(bst)){
        install.packages("bst")
    }
    library(bst)
    if(!require(caretEnsemble)){
        install.packages("caretEnsemble")
    }
    library(caretEnsemble)
    if(!require(tidyverse)){
        install.packages("tidyverse")
    }
    library(tidyverse)
    if(!require(dplyr)){
        install.packages("dplyr")
    }
    library(dplyr)
    if(!require(ggplot2)){
        install.packages("ggplot2")
    }
    library(ggplot2)

 

    calls <- read.csv("https://raw.githubusercontent.com/stat-learning/group-1/master/Data/calls.csv")

    priority_dummy <- NULL
    priority_dummy[calls$Priority == "Low"] = 0
    priority_dummy[calls$Priority == "Medium"] = 0.5
    priority_dummy[calls$Priority == "High"] = 1

    category_dummy <- NULL
    category_dummy[calls$FinalCallGroup == "Traffic"] = 0
    category_dummy[calls$FinalCallGroup == "Disorder"] = 0
    category_dummy[calls$FinalCallGroup == "Community Policing"] = 0
    category_dummy[calls$FinalCallGroup == "Assist"] = 0
    category_dummy[calls$FinalCallGroup == "Other"] = 0
    category_dummy[calls$FinalCallGroup == "Alarm"] = 0
    category_dummy[calls$FinalCallGroup == "Civil"] = 0
    category_dummy[calls$FinalCallGroup == "Crime"] = 1

    calls_dummy <- mutate(calls, priority_dummy)
    calls_dummy <- mutate(calls_dummy, category_dummy)

 

    ggplot(data=calls, aes(x=TimeInQueue_sec)) + 
      geom_histogram(binwidth=500) +
      labs(x = "Time in Queue", y="Frequency" , title="Frequency of Time in Queue", 
           caption="Number of Observations = 154247, Dashed Line represents Mean of 
           the Call Times") +
      geom_vline(aes(xintercept=mean(TimeInQueue_sec)),
                 color="blue", linetype="dashed", size=1)
    summary(calls$TimeInQueue_sec)

 

    ggplot(data = calls, mapping = aes(x = Pct.White,
                                       y = TimeInQueue_sec)) +
    geom_point(alpha = 0.1)+
    labs(x = "Percentage of White Residents", y = "Time Spent in Queue")

 

    #same as above but faceted by priority
    ggplot(data = calls, mapping = aes(x = Pct.White,
                                       y = TimeInQueue_sec)) +
    geom_point(alpha = 0.1) +
    facet_wrap(~Priority, ncol = 3) +
    labs(x = "Percentage of White Residents", y = "Time Spent in Queue")

 

    ggplot(data = calls, mapping = aes(x = Income.Std,
                                       y = TimeInQueue_sec)) +
    geom_point(alpha = 0.1)+
    labs(x = "Mean Income", y = "Time Spent in Queue")

 

    #same as above but faceted by priority
    ggplot(data = calls, mapping = aes(x = Income.Std,
                                       y = TimeInQueue_sec)) +
    geom_point(alpha = 0.1) +
    facet_wrap(~Priority, ncol = 3) +
    labs(x = "Mean Income", y = "Time Spent in Queue")

 

    income_positive <- (calls_dummy$Income.Std + 1.353877)
    log_income <- log(income_positive)
    hist(log_income)

    log_queue <- log(calls_dummy$TimeInQueue_sec)
    hist(log_queue)

    calls_dummy <- add_column(calls_dummy, log_income, .after = 9)
    calls_dummy <- add_column(calls_dummy, log_queue, .after = 10)

    #remove negative infinity variables
    calls_dummy <- calls_dummy[!is.infinite(log_queue),]

 

    # creating PCA model
    numCalls<-calls_dummy[, -c(2:5)]
    calls.PCA<-prcomp(na.omit(numCalls), center=TRUE, scale.=TRUE)
    summary(calls.PCA)
    str(calls.PCA)

 

    #plotting PCA
    ggplot2::autoplot(calls.PCA,
                      alpha=0.05, loadings=TRUE, loadings.label=TRUE,
                      data=numCalls, loadings.label.repel=TRUE) +
    ggtitle(label = "Principal Component Analysis for 911 Queue Time")

 

    screeplot(calls.PCA, type="lines")
    #based on the scree plot, we should use 3 PC's. 
    PC1<-calls.PCA$x[,1]
    PC2<-calls.PCA$x[,2]
    PC3<-calls.PCA$x[,3]

 

    #without tranformed variables
    lm_base <- lm(TimeInQueue_sec ~ Pct.White +
                Std_pop +
                Income.Std +
                priority_dummy +
                category_dummy,
              data = calls_dummy)
    summary(lm_base)

 

    #Checking the residual graphs
    plot(lm_base)

 

    # with transformed variables
    lm_transformed <- lm(log_queue ~ Pct.White +
                log_income +
                priority_dummy +
                category_dummy,
              data = calls_dummy)
    summary(lm_transformed)

 

    #Checking the residual graphs - these look a lot better
    plot(lm_transformed)

 

    set.seed(23489)
    train_index <- sample(1:nrow(calls_dummy), 0.9 * nrow(calls_dummy))
    calls_train <- calls_dummy[train_index, ]
    calls_test <- calls_dummy[-train_index, ]

    preds <- c(7, 9:10, 14:15)
    train_preds <- as.matrix(calls_train[,preds])
    test_preds <- as.matrix(calls_test[,preds])

    fit_control <- trainControl(## 10-fold CV
                               method = "cv",
                               number = 10,
                               allowParallel = TRUE)

    model_list <- caretList(log_queue ~ Pct.White + log_income + category_dummy +
                              priority_dummy,
                            trControl = fit_control,
                            data=calls_train,
                            methodList = c("lm", "glmnet", "ranger", 
                                           "gbm", "treebag"),
                            tuneList = NULL,
                            continue_on_fail = FALSE)

    best_lm <- model_list$lm$finalModel

    best_glmnet <- model_list$glmnet$finalModel

    best_rf <- model_list$ranger$finalModel

    best_bagged_trees <- model_list$treebag$finalModel

    best_boosted_forest <- model_list$gbm$finalModel

    options(digits = 3)
    model_results <- data.frame(
     LM = min(model_list$lm$results$RMSE),
     GLMNET = min(model_list$glmnet$results$RMSE),
     RANGER = min(model_list$ranger$results$RMSE),
     TREEBAG = min(model_list$treebag$results$RMSE),
     GBM = min(model_list$gbm$results$RMSE)
     )

 

    print(model_results)

 

    lm_yhats <- predict(model_list$lm, newdata = calls_test)
    best_lm_mse <- MSE(lm_yhats, calls_test$log_queue)


    glmnet_yhats <- predict(model_list$glmnet,
                            newdata = calls_test)
    best_glmnet_mse <- MSE(glmnet_yhats, calls_test$log_queue)


    rf_yhats <- predict(model_list$ranger, newdata = calls_test)
    best_rf_mse <- MSE(rf_yhats, calls_test$log_queue)

    bagged_yhats <- predict(model_list$treebag,newdata = calls_test)
    best_bagged_mse <- MSE(bagged_yhats, calls_test$log_queue)
      
    boosted_yhats <- predict(model_list$gbm, newdata = calls_test)
    best_boosted_mse <- MSE(boosted_yhats, calls_test$log_queue)


    options(digits = 3)
    test_MSE <- data.frame(
     LM = best_lm_mse,
     GLMNET = best_glmnet_mse,
     RANGER = best_rf_mse,
     TREEBAG = best_bagged_mse,
     GBM = best_boosted_mse
     )

 

    print(test_MSE)

 

    summary(best_boosted_forest)

 

    no_priority <- caretList(log_queue ~ Pct.White + log_income + category_dummy,
                            trControl = fit_control,
                            data=calls_train,
                            methodList = c("gbm"),
                            tuneList = NULL,
                            continue_on_fail = FALSE)

 

    summary(no_priority$gbm$finalModel)

 

    no_p_or_c <- caretList(log_queue ~ Pct.White + log_income,
                            trControl = fit_control,
                            data=calls_train,
                            methodList = c("gbm"),
                            tuneList = NULL,
                            continue_on_fail = FALSE)

 

    summary(no_p_or_c$gbm$finalModel)

 

    summary(best_lm)

    coef(best_glmnet, 0.00254)
