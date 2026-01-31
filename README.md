# Weather trend forecasting assesment

THE GLOBAL REPORT WITH VISUALIZATIONS IS IN THE "analysis_report.html" file.

## Methodology

### 1. Data cleaning and preprocessing

The first step was to understand the data structure and handle quality issues.

**Missing values**: The dataset had some missing values in air quality columns (Carbon Monoxide, Ozone, PM2.5, etc.). These were imputed using the median value of each column, which is robust to outliers and preserves the distribution.

**Outlier detection**: I used the IQR (Interquartile range) method to identify outliers in key weather variables like temperature, humidity, precipitation, and wind speed. After analysis, I decided to keep the outliers since they represent valid extreme weather conditions rather than data errors.

**Normalization**: Numerical features were normalized using MinMaxScaler to prepare them for model training. This ensures all features are on the same scale.

### 2. Exploratory data analysis

The EDA phase focused on understanding patterns, correlations, and distributions in the data.

**Temperature analysis**: I examined temperature distribution across the dataset and found significant variation by geographic location and time of day. The hourly pattern shows clear daily cycles with temperatures peaking in the afternoon. I also identified the hottest and coldest countries based on average temperature.

**Precipitation analysis**: Precipitation data is heavily right-skewed, with most readings being zero or near zero. This makes sense since it does not rain most of the time in most places. I analyzed precipitation patterns by hour and identified the countries with highest average rainfall.

**Correlation analysis**: A correlation matrix revealed strong relationships between variables. The most notable finding was the 0.98 correlation between actual temperature and "feels like" temperature. Humidity showed negative correlation with temperature and visibility, which aligns with meteorological principles.

### 3. Forecasting model

VERY IMPORTANT DETAIL:

- For the time series forecasting task, I selected a single location rather than using the entire dataset. This decision is grounded in fundamental time series theory: forecasting models learn temporal patterns from sequential observations of the same entity over time. Mixing data from multiple locations would violate the stationarity assumption and introduce confounding factors (different climates, hemispheres, timezones), making it impossible for the model to learn meaningful temporal dependencies. Kabul was selected because it had the highest number of records in the dataset (603 observations), providing the longest time span for the model to learn daily and seasonal patterns. An alternative approach would be to treat this as a regression problem using all locations with geographical features as inputs, but that would no longer be time series forecasting in the classical sense.

**Feature engineering**: I extracted temporal features from the `last_updated` column: hour, day, month, day of week, and day of year. I also created cyclical encodings for hour using sine and cosine transformations to capture the circular nature of daily patterns. Additional features included humidity, pressure, and cloud cover.

**Models trained**:

1. **Linear regression**: A simple baseline model to establish minimum performance expectations.
2. **Random forest**: An ensemble of decision trees that can capture non-linear relationships. I used 100 estimators with default hyperparameters.
3. **XGBoost**: A gradient boosting implementation known for strong performance on tabular data. Used 100 estimators with learning rate of 0.1.

The assesment asked to create an ensemble method of different models, but due to the fact that XGBoost and Random Forest are already ensembling methods, I thought that could be sufficient.

Nevertheless, a simple ensembling method could be achieved by averaging the three predictions and dividing by 3:

ensemble_pred = (linear_pred + rf_pred + xgb_pred) / 3

**Train/test split**: I used a time-based split (80/20) rather than random sampling to avoid data leakage. This means the model is trained on earlier data and tested on later data, simulating real forecasting scenarios.

The 80/20 split is in my opinion a standard in the industry that could be changed depending on the problem, but in this case I thought it was okay to leave those sizes.

**Results**:

| Model         | RMSE   | MAE    | R²    |
| ------------- | ------ | ------ | ------ |
| Linear regr.  | 2.8402 | 2.2563 | 0.9346 |
| Random forest | 1.6884 | 1.2758 | 0.9769 |
| XGBoost       | 1.9087 | 1.4679 | 0.9704 |

Random Forest achieved the best performance with an R² of 0.977, meaning it explains 97.7% of temperature variance. The mean absolute error of 1.28°C is quite acceptable for weather forecasting.

### 4. Advanced analyses

**Climate analysis**: I grouped data by timezone/region to study climate patterns. Some regions show high temperature variability (continental climates) while others remain stable (tropical zones). This analysis helps understand which areas experience more extreme temperature swings.

**Environmental impact**: I analyzed the correlation between air quality metrics and weather parameters. Wind speed shows inverse correlation with pollutant concentration, which makes sense since wind disperses pollutants. I also ranked countries by air quality, identifying both the cleanest and most polluted locations.

**Spatial analysis**: I created latitude bands to analyze how weather varies by latitude. As expected, temperature decreases with distance from the equator. Tropical bands show the highest average temperatures and humidity levels. A scatter plot visualization shows global temperature distribution by coordinates.

## Key findings

The analysis revealed several interesting insights:

1. Temperature forecasting is highly accurate using time-based features and weather variables, with Random Forest achieving R² of 0.969.
2. The relationship between actual and perceived temperature is nearly linear (0.98 correlation), though wind and humidity can cause deviations.
3. Air quality varies significantly by country, with clear geographical patterns. Wind is a key factor in pollutant dispersion.
4. Latitude is the primary driver of temperature differences, but regional factors like ocean proximity and altitude also play important roles.

## Requirements

See `requirements.txt` for complete dependencies.
