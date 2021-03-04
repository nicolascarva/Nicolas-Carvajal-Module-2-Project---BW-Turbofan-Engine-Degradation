# Nicolas-Carvajal-Module-2-Project---BW-Turbofan-Engine-Degradation
 Unit 2 Build Week
Using Python to predict the Remaining Useful Life of a Turbofan Engine
Data:
The data (link to source data by NASA) consists of four datasets, each with one file for train and two files for test. Each time series data is from a different engine (unit), but each dataset can be considered to be from a fleet of engines of the same type (note that units with the same unit no. in the test and train sets are not the same engine). In the training set the faults grow in magnitude until failure (last data point for each unit), in the test set the data is truncated at some point prior to failure, (with the test target variable containing the number of cycles remaining before engine failure).
The Datasets are:
The columns in the dataset are:
None of the data include units (or even a description of what the sensor is measuring), and may be contaminated by sensor noise.
Data Wrangling
Handling multiple datasets and file formats
Since all three data files (train, test, RUL) for each dataset are slightly different the wrangle function uses an if, else statement (receiving 'train', 'test', or 'RUL' as inputs) to apply the correct function according to the data file received.
To import all 4 datasets simultaneously the function also uses a for loop that returns df[n] for the four datasets (n = 1, 2, 3, or 4).
Cleanup
Columns with only one unique value were dropped.
RUL Column
Since the dimensions of test data and RUL (Remaining Useful Life) data are different (RUL only provides one value per unit, the data will be extrapolated so that the dimensions of y_test match the dimensions of X_test. The way the extrapolation is done is by assigning the RUL value to the last data-point for each unit and adding 1 (timecycle) to each row above. 
A simpler code was used to create a target vector for the train data where the last data-point for each unit was set to zero (by definition the RUL at the last data-point for each unit is zero, which represents failure) and each row above was added 1 until reaching the last data point of the prior unit.
RUL Column for the train set, note how all the lines end at 0 (by definition):
RUL vs time (test data) note how all lines end at a RUL value of 0 (failure)RUL Column for the test set, note how all the lines end at different points, which is where the data is truncated (and what the model is attempting to predict):
RUL vs time (train data), note how data in test set is truncated at random points. This is the target vector of the test set, so this column was only used to validate the model.Feature Engineering: Running Average
Since the RUL for each unit likely depends on prior data points, a Running Average (with N samples) column was added for each operational setpoint and sensor using the following code (the initial N rows are filled with the same data as the original column, to avoid NaN):
To visualize the sensors and detect anomalies, all the sensors for the training data (for Unit 1 of Dataset 1) were plotted vs. time. This makes for a good comparison between the raw data and the running average data
Raw data: values vs. time for each operational setpoint or sensor used in the modelThe same plot for the Running Averages gives us a look at cleaner data:
Modeling
All Modeling described was done on Data Set 1.
Baseline
This is a regression problem so the baseline was calculated to be the mean of the RUL column:
Baseline RMSE : 68.879321
Random Forest Regression
The RFR was run on default settings, the results beat the baseline:
RFR RMSE : 44.267385
Logistic Regression
Logistic Regression also beat the baseline although it needed maximum iterations to be increased to 500, in order to converge:
LR RMSE Test : 46.675430
XGBoost
RMSE Test : 49.867291
Optimize
Random Forest Regressor Optimization
A grid search was run for the Random Forest Regressor (RFR), the new parameters found were:
best parameters: {'max_depth': 5, 'n_estimators': 89}
With these parameters the RMSE error was:
RFR RMSE Test : 41.712057
This represents an improvement of 3 cycles over the original RFR and of 27 cycles over the baseline.
Communicate Results
Model Scores
The best model was the RFR after optimizing (44.26 to 41.71 time-cycles RMSE after optimization), all models beat the baseline:
Permutation Importance
The most important feature on the best model (RFR), by far, was 'time'. This makes intuitive sense (the longer the engine has been going, the closer to failure it will be), but this might be a function of the data wrangling resulting in a linear target matrix (as time moves forward for each unit).
To get a different perspective, the time column was dropped (the notebook for the modeling with 'time' dropped can be found here). The results were surprising, because save for the dropped column, otherwise equal models, most of the features in the top-ten importance when we include the time column, do not appear in the top-ten of the model without the time column! In the latter, sens_meas_11, appears twice (as raw data and running average), while sens_meas_12 is the only one to beat it's running average counterpart (while sens_meas_12 does not even appear on the former's top-ten).
Future Work
There are various ideas that could be used in order to further improve the model (beyond hyper-parameter tuning techniques):
Analyze the relationship between datasets to gain insights into what each sensor sensor or setpoint represents (assuming that they are the same sensors across datasets)
The running average n-datapoints could also be tuned to get maximum model performance
FFT's and other noise filtering algorithms
Dropping columns more aggressively (some of the remaining sensors seem flat, at least for some units)
Logarithmic techniques to change data from exponential to linear
Permutation importance - permutation importance experiments also show the innumerable opportunities for experimentation available. While the 'time' column is clearly more important in predicting the RUL, it is possible that dropping it, while using clever feature engineering to maintain its influence on the model, could result in superior model performance (by permitting other important features to have more influence on the model)
