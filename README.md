# Shark-Attacks-Time-Series
This analysis explores Shark Attack incidents which is crucial for understanding the dynamics and risks associated with these events. 

## Introduction

This analysis explores Shark Attack incidents, understanding the dynamics and risks associated with these events. This report provides insight into public a dataset on global shark attacks, sourced from the OpenDatasoft Website. The dataset offers a variety of variables, including the dates of attacks, types of attacks, locations, activities of the individuals involved, their sex, and the fatality outcomes. 

## Executive Summary 

The primary objective of this project is to uncover patterns and trends in shark attacks over time and to forecast the expected number of incidents over the next five years. These insights aim to support regions that frequently experience shark attacks by informing preventative strategies and resource planning.

The analysis employs univariate time series modelling using the ARIMA algorithm. Data was sourced from OpenDataSoft, transformed using Power Query in Excel, and analysed in Power BI and Databricks, with Python as the primary programming language. Key steps included data cleaning, handling missing values, and ensuring consistency across variables.

The focus was on annual shark attack volumes, with statistical techniques applied to ensure stationarity and model accuracy. Forecasts were evaluated using metrics such as Mean Absolute Error (MAE) and Root Mean Squared Error (RMSE), demonstrating a reliable predictive model. This project highlights the importance of data quality, model validation, and the practical value of time series forecasting in public safety and environmental analytics.


## Data

This data is sourced on OpenDataSoft website - https://public.opendatasoft.com/explore/dataset/global-shark-attack/table/?disjunctive.country&amp;disjunctive.area&amp;disjunctive.activity

## Data Infrastructure & Tools 

The dataset is well-structured and easy to interpret. Exporting it to Excel allows for efficient transformation using Power Query, which offers a range of built-in tools for shaping data. Although JSON is also available and flexible, I chose Excel due to its familiarity and accessibility.

After transformation, the data is loaded into Power BI for analysis using visual tools like line charts and heat maps to explore seasonality and trends. Tableau was considered for its advanced visuals but wasn’t used due to cost and lack of access.

The data will be imported into Databricks, where Time Series analysis will be performed. Databricks is a versatile platform that supports data visualisation, modelling, and transformation—all in one place. It also allows the use of multiple programming languages, including Python, SQL, R, and Scala, making it accessible to a wide range of users. For my analysis, I will be writing code using Python due to the benefits such as wide available libraries, easy to use and visual support. 

## Data Engineering  

The dataset is exported to Excel and imported into Power Query for transformation. I displayed all rows to understand its size using the 'Column Distribution' and 'Column Quality' features. These tools helped me quickly see value distributions and identify anomalies and errors. Although they can slow down the process for large datasets, I verified that the dataset contains 6,890 records, which Power Query can easily handle.

I identified and filtered null values in the Date column for further investigation. The Date column is crucial for accurate time-dependent analysis. By filtering the nulls, I found 303 null values. Given the limited data, retaining as many records as possible is important. The Year field, available separately, showed some nulls in the Date column had a year populated. One option was to use the Year to create a default date, e.g., transforming a blank Date with Year 2001 to 01/01/2001. However, I noticed some years were before 1950, which I considered outdated. To avoid skewing the analysis, I removed data before 1950 and filtered the Date column to remove any remaining nulls.
 
The 'Type' column shows the type of shark attack that occurred, such as Provoked, Unprovoked, or Sea Disaster. Although I might not use this column in my final analysis, it could provide insights later. Therefore, I will replace any null values in the 'Invalid' group to ensure the dataset's completeness by eliminating missing values. The completeness data quality dimension is defined as the percentage of data populated, as mentioned on the ICEDQ website. Changing null values to a specific category like "Invalid" would be considered a form of data imputation. This method is known as constant value imputation, where missing values are replaced with a constant value or category, ensuring that the data is complete. 

 
For data protection purposes, I have removed the column that includes the name of the victim, as this information is not relevant to my analysis. This step ensures compliance with privacy regulations and protects sensitive personal information. Additionally, removing unnecessary columns helps streamline the dataset, making the analysis more efficient and focused on the relevant data.

‘Age’ column format was irregular by some entries having whole numbers and decimals which I formatted so they were all the same. This help to keep the data consistent throughout and ensures the data follows and in the correct format. 

Please see ELT process below: 


<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/ETL%20Pipeline.JPG" />


## Data Analytics 

Time series models are designed to identify and learn from temporal patterns in historical data, enabling accurate forecasting of year-on-year trends. Methods such as ARIMA and SARIMA are particularly effective for predicting shark attacks, as they account for both trend and seasonal variations in past occurrences.

 
Load and Prepare Dataset 

*I have used machinelearningmastery.com as a guide to complete my Time Series Analysis. 

Python Code to load and prepare data:

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Importing the Required libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from pyspark.sql.functions import lit, sum as _sum

#### Loading the dataset from the SQL table
Shark_Raw = spark.sql("SELECT * FROM `hive_metastore`.`default`.`shark_attack`")

#### Adding a column with the value 1 to each row
Shark_Raw = Shark_Raw.withColumn("Volume", lit(1))
Shark_Raw = Shark_Raw.filter((Shark_Raw['Year']  >= 2000) & (Shark_Raw['Year'] <= 2023))

#### Display the dataframe
display(Shark_Raw)

#### Group by 'Year' and sum the 'Volume' column
yearly_volume = Shark_Raw.groupBy('Year').agg(_sum('Volume').alias('Total_Volume'))

#### Display the new table
display(yearly_volume)

#### Convert the Spark DataFrame to a Pandas DataFrame
yearly_volume_pd = yearly_volume.toPandas()

#### Ensure the 'Year' column is in datetime format
yearly_volume_pd['Year'] = pd.to_datetime(yearly_volume_pd['Year'], format='%Y')

#### Sort the DataFrame by 'Year'
yearly_volume_pd.sort_values(by='Year', inplace=True)

#### Set 'Year' as the index
yearly_volume_pd.set_index('Year', inplace=True)

#### Plotting the time series data to visualize the trends or patterns
plt.figure(figsize=(10, 6))
plt.plot(yearly_volume_pd.index, yearly_volume_pd['Total_Volume'], label='Yearly Shark Attacks')
plt.title('Shark Attacks Over Time')
plt.xlabel('Year')
plt.ylabel('Total Volume')
plt.legend()

#### Saving the plot as an image
plt.savefig('timeseries_plot.png')
plt.show()

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
The dataset was loaded into Databricks, with most of the initial transformations completed in Power Query. Additional validation steps were incorporated into the code to ensure data cleanliness, as inconsistencies could distort the accuracy of the time series model. A line chart was created using the target variable (Volume) against the independent variable (Year) to verify alignment with the visualisation presented in the dashboard.


<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/Shark%20Attack%20Over%20Time%20Line%20Chart.png" />

Splitting the dataset into train and test is not required on this dataset due to the low volume.

Check for Stationarity

ADF Statistic: –1.51
P-value: 0.53

Since the p-value exceeds the 0.05 significance level, we fail to reject the null hypothesis. This suggests the presence of a unit root, indicating that the time series is non-stationary.

To improve stationarity, the analysis was limited to data from post-1950 to post-2000, focusing on periods where the data-generating process is presumed to be more stable. First-order differencing was applied to remove trends by subtracting each observation from its predecessor, enhancing the model’s ability to detect underlying patterns. Additionally, a logarithmic transformation was used to stabilise variance and promote a more consistent distribution over time.

Appying Log Transformation and First Order Code:

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Importing the Required libraries
import numpy as np
from statsmodels.tsa.stattools import adfuller

#### Apply log transformation to stabilize variance
yearly_volume_pd['Log_Total_Volume'] = np.log(yearly_volume_pd['Total_Volume'])

#### Apply first-order differencing to the log-transformed data
yearly_volume_pd['Log_Differenced'] = yearly_volume_pd['Log_Total_Volume'].diff().dropna()

#### Display the DataFrame with the log-transformed and differenced data
display(yearly_volume_pd)

#### Perform the ADF test on the log-transformed and differenced data 
result_log_diff = adfuller(yearly_volume_pd['Log_Differenced'].dropna())
print(f"ADF Statistic (Log Differenced): {result_log_diff[0]}")
print(f"p-value (Log Differenced): {result_log_diff[1]}")
print("Critical Values:")
for key, value in result_log_diff[4].items():
print(f"   {key}: {value}")

----------------------------------------------------------------------------------------------------------------------------------------
Here is the new summary of the results:
•	ADF Statistic: –6.55 — a strongly negative value, providing strong evidence against the null hypothesis.

•	P-value: 0.0000911 — well below the 0.05 threshold, allowing us to reject the null hypothesis and conclude that the transformed time series is stationary.

## Results 

ARIMA has been used to fit the model to the log-transformed time series data and then making a prediction for the next five years. 

<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/ARIMA%20Prediction%20for%20Next%205%20Years.jpg" />

The code was tweaked to overlay the model's predictions with the actual values on the time series chart. This allows for a clear visual comparison, making it easier to assess the model's accuracy by observing how closely the predicted line aligns with the actual data.

Updated Chart: 

<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/ARIMA%20Prediction%20for%20Next%205%20Years%20with%20Previous%20Prediction%20Overlay.jpg" />


MAE: A metric that tells us the mean absolute difference between the predicted values and the actual values in a dataset. The lower the MAE, the better a model fits a dataset.
 
Interpretation: MAE of 11.93 means that, on average, the model's predictions are off by about 11.93 shark attacks per year.

RMSE: A metric that tells us the square root of the average squared difference between the predicted values and the actual values in a dataset. The lower the RMSE, the better a model fits a dataset.

Interpretation: RMSE of 21.89 means that my model's predictions have an average error of about 21.89 shark attacks per year. RMSE gives more weight to larger errors due to the squaring of differences, making it more sensitive to outliers compared to MAE.

*Information on MAE & RMSE provided by Zach Bobbitt on statology webiste

Residuals of the ARIMA model

<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/Residuals%20of%20the%20ARIMA%20Model.jpg" />
 
Residuals are the difference between the observed value and the values predicted by the model. Residuals appear low and stable after the year 2000, suggesting that the model effectively captures the underlying structure of the time series during this period. However, there seems to be a potential issue around the year 2000, which may indicate a poor model fit in the earlier years. To address this, I could consider refitting the model with a different start date, as this might improve the overall pattern and performance of the model.

Ljung-Box test 

<img width="316" height="316" alt="image" src="https://github.com/Haley1203/Shark-Attacks-Time-Series/blob/main/Ljung%20Box%20Test.jpg" />

In my ARIMA model, residual autocorrelations mostly fall within the 95% confidence bounds, indicating that the model has effectively captured the temporal structure of the data. The residuals resemble white noise, suggesting a good model fit.


## Summary 
In summary, this project successfully applied ARIMA time series modelling to forecast shark attack volumes over the next five years. Through careful data engineering, transformation, and validation, the dataset was prepared for accurate analysis. The model showed strong performance, with low residuals and evaluation metrics indicating reliable predictions. While minor issues were noted around the year 2000, overall results suggest the model effectively captures temporal patterns.

The next step would be to share this information with hot spot areas, but using the map displayed within the appendix. I could also look to implement SARIMA to capture seasonal patterns which both analyses can help toward shark attacks prevention. The forecasts can guide lifeguard staffing, medical readiness, and surveillance efforts in regions expected to see higher shark activity. 


## Referencing & Academic Support 

OpenDataSoft. (n.d.). Global Shark Attack Dataset. Retrieved July 8, 2025, from https://public.opendatasoft.com/explore/dataset/global-shark-attack/table/?disjunctive.country&amp;disjunctive.area&amp;disjunctive.activity

Microsoft. (2025). Power Query documentation. Retrieved July 8, 2025, from https://learn.microsoft.com/en-us/power-query/power-query-what-is-power-query

iceDQ. (n.d.). 6 Data Quality Dimensions: Complete Guide with Examples and Measurement Methods. Retrieved July 8, 2025, from https://icedq.com/6-data-quality-dimensions#what_is_completeness_data_quality_dimension

Neptune.ai. (n.d.). ARIMA and SARIMA: Real-world time series forecasting guide. [online] Available at: https://neptune.ai/blog/arima-sarima-real-world-time-series-forecasting-guide [Accessed 29 Jul. 2025].

Brownlee, J. (2020). How to develop an autoregression forecast model for household electricity consumption. [online] Machine Learning Mastery. Available at: https://machinelearningmastery.com/how-to-develop-an-autoregression-forecast-model-for-household-electricity-consumption/ [Accessed 29 Jul. 2025].

Bobbitt, Z., 2021. MAE vs. RMSE: Which Metric Should You Use? [online] Statology. Available at: https://www.statology.org/mae-vs-rmse/ [Accessed 5 Aug. 2025].

Yurekli, K., Kurunc, A. and Ozturk, F., 2005. Testing the Residuals of an ARIMA Model on the Çekerek Stream Watershed in Turkey. Turkish Journal of Engineering and Environmental Sciences. Available at: https://www.researchgate.net/publication/286644057_Testing_the_Residuals_of_an_ARIMA_Model_on_the_C_ekerek_Stream_Watershed_in_Turkey [Accessed 5 Aug. 2025].






