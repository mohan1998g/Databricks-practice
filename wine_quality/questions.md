Since you're using the Wine Quality dataset in Databricks, it's a great dataset to practice everything from basic DataFrame operations to advanced PySpark transformations, window functions, and feature engineering.

Level 1: Basic DataFrame Operations

Count the total number of records in the dataset.

Display the schema of the DataFrame.

Find the number of columns.

Show the first 10 records.

Find rows where quality > 7.

Count wines for each quality rating.

Find distinct quality values.

Sort wines by alcohol in descending order.

Find wines with the highest and lowest alcohol content.

Display wines having residual sugar greater than 10.

Level 2: Aggregations

Find average alcohol content.

Find minimum, maximum, and average pH.

Calculate average quality.

Find average alcohol grouped by quality.

Find average sulphates grouped by quality.

Find the number of wines for each quality score.

Find quality-wise maximum alcohol.

Find fixed acidity statistics:

Min
Max
Avg

Find average density by quality.

Which quality rating appears most frequently?

Level 3: Filtering & Conditions

Find wines with alcohol greater than dataset average.

Find wines where volatile acidity < 0.3.

Find wines with:

alcohol > 12
quality >= 7

Find wines having pH between 3.0 and 3.5.

Find wines with sulphates > 1.

Classify wines as:

Good      -> quality >= 7
Average   -> quality 5-6
Poor      -> quality < 5

Count wines in each category.
Level 4: Column Transformations
Create a new column:
alcohol_percentage = alcohol * 100

Create acidity ratio:
fixed_acidity / volatile_acidity

Create a column:
high_alcohol = "Yes" if alcohol > 12 else "No"


Round density to 3 decimal places.

Convert quality into categorical labels.

Create a quality_score:

quality * alcohol

Create a quality bucket:
Low
Medium
High

Level 5: Window Functions

Rank wines by alcohol.

Find top 5 wines with highest alcohol content.

Rank wines within each quality group by alcohol.

Find cumulative average alcohol.

Assign row numbers ordered by alcohol.

Find wine with highest alcohol in every quality category.

Find second highest alcohol wine for each quality.

Calculate moving average alcohol (window of 3 rows).

Level 6: Advanced Aggregations
Find correlation between:
Alcohol and Quality
Sulphates and Quality
Density and Quality


Which feature has highest correlation with quality?

Find median alcohol content.

Calculate percentile values:

25th
50th
75th

Find standard deviation of alcohol.

Find variance of acidity columns.

Calculate skewness for alcohol.

Calculate kurtosis for residual sugar.

Level 7: SQL Practice in Databricks

Create a temporary view:

df.createOrReplaceTempView("wine")


Practice SQL queries:

Find average alcohol by quality.

Find top 10 wines by alcohol.

Find quality distribution.

Find wines with alcohol above average.

Find percentage contribution of each quality category.

Find top quality-wise alcohol wine.

Find cumulative count of wines by quality.

Level 8: Feature Engineering
Create binary target:
1 -> quality >= 7
0 -> otherwise


Normalize alcohol column.

Standardize acidity columns.

Create interaction feature:

alcohol * sulphates

Create total acidity:
fixed_acidity + volatile_acidity + citric_acid

Create density category:
Low Density
Medium Density
High Density

Level 9: Mini Project Questions

What are the characteristics of high-quality wines?

Which features differ most between poor and good wines?

Does alcohol content increase with quality?

Are higher sulphates associated with better quality?

Which quality has highest average alcohol?

Build a summary report showing:

Quality
Count
Avg Alcohol
Avg pH
Avg Sulphates
Avg Density

Identify outliers in alcohol content using IQR.
Level 10: Interview-Oriented PySpark Challenges

Find duplicate rows.

Remove duplicates.

Add a surrogate key using:

monotonically_increasing_id()


Cache the DataFrame and compare execution times.

Explain the physical plan using:

df.explain(True)


Repartition the DataFrame by quality and compare performance.

Persist the DataFrame using different storage levels.

Write the transformed data into:

Delta Table
Parquet
CSV
Perform data quality checks:
Null counts
Duplicate counts
Invalid values
Build a complete ETL pipeline using PySpark and Delta Lake.
Bonus Challenge (Real Interview Level)

Implement this complete workflow:

Read Wine Data
↓
Data Cleaning
↓
EDA
↓
Feature Engineering
↓
Quality Categorization
↓
Window Analysis
↓
Correlation Analysis
↓
Outlier Detection
↓
Aggregate Reporting
↓
Write to Delta Table


If you can solve Questions 1-80, you'll cover about 90% of the PySpark DataFrame concepts typically asked in Databricks/Cognizant/MNC interviews.
