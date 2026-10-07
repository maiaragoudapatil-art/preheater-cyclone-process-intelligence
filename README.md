
# Preheater Cyclone Process Intelligence

## About the Project

This project focuses on identifying abnormal operating periods in a cyclone preheater using historical sensor data.
The dataset contains around 377,000 observations collected every 5 minutes from 2017 to 2020. It includes six process variables covering temperature, material temperature, and draft measurements.
The main goal was to find periods where the overall process behaviour looked unusual and could be worth investigating further.
Since the dataset did not contain labelled abnormal events, I approached this as an unsupervised anomaly detection problem.


## What I Did

I followed the project in four main stages:

1. Prepared and checked the sensor data
2. Created features to capture process behaviour over time
3. Used Isolation Forest to detect unusual observations
4. Grouped the observations into meaningful abnormal periods and analyzed them


## 1. Data Preparation

I started by understanding the structure and quality of the raw data.

The timestamp column contained different date-time formats, so I standardized it and checked whether the data followed the expected 5-minute sampling interval.

I also checked:

- Missing values
- Duplicate timestamps
- Time gaps
- Invalid sensor values
- System/status messages
- Unusual temperature and draft readings

Some sensor columns contained values such as `Not Connect`, `I/O Timeout`, `Configure`, and `Comm Fail`. I kept these separate from the actual numeric sensor readings rather than treating them as process values.

The dataset had a relatively small amount of missing numeric data, so the usable data remained very high.


## 2. Understanding the Process

Before applying a machine learning model, I explored how the sensors behaved.

I looked at sensor distributions, correlations, trends, and different operating conditions.

Some sensors showed very strong relationships. For example, inlet draft and outlet gas draft had a correlation of about 0.995, while the two main gas temperature measurements had a correlation of about 0.991.

This showed that the process variables are closely related.

I also found that some values that initially looked unusual could actually occur during valid operating conditions. Because of this, I avoided using simple fixed thresholds to label anomalies.

## 3. Feature Engineering

The six original sensor readings alone do not tell the complete story.

For example, a temperature value might be normal by itself, but a sudden drop over 30 minutes could be important.

To capture this behaviour, I created additional features.

### Change Features

I calculated sensor changes over:

- 5 minutes
- 30 minutes
- 60 minutes

These features help capture short-term and longer-term process changes.

### Rolling Statistics

I also calculated 60-minute rolling statistics.

The rolling mean represents the recent process level, while the rolling standard deviation represents how much the process is fluctuating.

This gives the model some context about what was happening around each observation.

### Process Relationships

I also created features based on relationships between related temperature and draft variables.

Overall, the feature-engineered dataset contained 44 columns. After removing redundant features, 36 features were used by the final anomaly detection model.


## 4. Anomaly Detection

For anomaly detection, I used **Isolation Forest** from Scikit-learn.

I chose Isolation Forest mainly because the dataset did not contain labelled abnormal events.

A supervised model would require examples of known normal and abnormal behaviour, which were not available.

Isolation Forest works by isolating unusual observations from the rest of the data. Observations that are easier to isolate are considered more unusual.

The model was configured with:

- 200 trees
- Random state = 42
- Automatic contamination setting

The model produced an anomaly score for each observation. In my implementation, a higher score represents more unusual behaviour.

## 5. Selecting Abnormal Observations

Instead of choosing an arbitrary anomaly-score value, I used the 99th percentile of the score distribution as the threshold.

The selected threshold was approximately:

"0.5977"

This initially identified "3,748 candidate anomaly observations".

However, I did not treat every individual point as an abnormal event.
A single unusual point could simply be noise or a temporary sensor issue.

## 6. Finding Sustained Abnormal Periods

To make the results more meaningful, I grouped consecutive anomaly observations together.
I required at least **6 consecutive observations** to consider something an abnormal period.
Since each observation represents 5 minutes:
"6 observations × 5 minutes = 30 minutes"

So the final result only includes abnormal behaviour that continued for at least 30 minutes.
This reduced isolated spikes and helped focus the analysis on sustained process behaviour.


## Results

The final analysis identified:

| Metric | Result |
|---|---:|
| Total observations | 377,719 |
| Features used by model | 36 |
| Isolation Forest trees | 200 |
| Anomaly threshold | 0.5977 |
| Candidate anomaly observations | 3,748 |
| Minimum abnormal duration | 30 minutes |
| Final abnormal periods | 252 |
| Highest anomaly score | 0.7406 |
| Total detected duration | 8 days 3 hours 55 minutes |

Abnormal periods were detected across all four years.

### Year-wise Results

| Year | Abnormal Periods |
|---|---:|
| 2017 | 80 |
| 2018 | 102 |
| 2019 | 49 |
| 2020 | 21 |

2018 had the highest number of detected abnormal periods and the highest total detected duration.

This does not mean these were confirmed equipment failures. They represent periods where the sensor data showed unusual operating behaviour.


## Strongest Detected Period

The strongest period identified by the model occurred on:
"27 November 2018, 05:40 – 06:30"
It lasted for approximately "50 minutes".
The mean anomaly score was "0.7030", with a maximum score of "0.7406".
During this period, several variables changed at the same time.
For example:

- Inlet gas temperature decreased from around 911 to 750
- Gas outlet temperature decreased from around 866 to 716
- Material temperature decreased from around 952 to 548
- Draft values shifted from strongly negative values towards zero or positive values

The important point here is that the model did not flag just one sensor. It detected a coordinated change across multiple process variables.

This makes the period a good candidate for further engineering investigation.


## Visualizations

I created separate visualizations to understand both the process and the model results.

These include:

- Overall temperature trends
- Overall draft trends
- Sensor distributions
- Sensor correlation heatmap
- Rolling mean
- Rolling standard deviation
- 5, 30 and 60-minute changes
- Draft relationship plots
- Anomaly-score distribution
- Strongest abnormal period

The visualizations were used to understand whether the detected events made sense from a process perspective rather than relying only on the model output.


## Why Isolation Forest?

I considered the nature of the problem before selecting the model.
"K-Means" is mainly designed for clustering, so it is not as directly suited to identifying anomalies.
"Random Forest" is a supervised algorithm and would require labelled abnormal events.
An "Autoencoder" could also be used, but it would introduce more complexity than necessary for this project.
Isolation Forest provided a relatively simple and suitable unsupervised approach for identifying unusual multivariate behaviour.

## Key Takeaway

The main idea behind this project was to move beyond simply finding unusual sensor values.

Instead, I tried to identify **sustained changes in the overall process behaviour** by combining sensor relationships, time-based changes, rolling statistics, and unsupervised anomaly detection.

The final output gives engineers a list of candidate abnormal periods that can be investigated further using plant records, alarms, maintenance information, and operational knowledge.

"The detected periods are indicators of unusual process behaviour, not confirmed equipment failures."

---

## Project Structure
preheater-cyclone-process-intelligence/
│
├── 1_data_preprocessing.ipynb
├── 2_feature_engineering.ipynb
├── 3_anomaly_detection.ipynb
├── 4_visualizations.ipynb
│
├── cyclone_data.csv
├── cleaned_sensor_data.csv
├── feature_engineered_data.csv
├── detected_abnormal_periods.csv
│
└── README.md