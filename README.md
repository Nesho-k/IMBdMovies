# IMBd Movies Analysis

### Project Overview

This data analysis project aims to provide insights into the performance of movies with their ratings over the years. By analyzing various aspects of the movies data, we seek to identify trends and make data-driven recommendations for futur movies.

### Data Source 

Movies Data: The primary dataset used for this analysis is the "imdb_top_1000.csv" file, containing detailed information about movies Poster Link's, Title's, Released Year's, Certificate's, Runtime's, Genre's, IMDB Rating's, Overview's, Meta score's, Director's, Star1's, Star2's, Star3's, Star4's, No_of_votes's, Gross's 

### Tools 

- Excel : Data Cleaning
- SQL : Data Cleaning, Data Exploration and Data Analysis

### Data Cleaning part. 1 

In the initial data preparation phase that we did in excel, we performed the following tasks:

1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting

### Data Cleaning part. 2

Unfortunately, we had some problems when we import the dataset from Excel to SQL, so we had to clean it again on SQL. First, we took columns that are going to be the most interesting, in that respect we are going to create a temp table :

```sql
DROP TABLE IF EXISTS #IMDb_top_1000
CREATE TABLE #IMDb_top_1000
(Title nvarchar(255),
Year float,
Runtime nvarchar(255), 
Genre nvarchar(255),
Rating float,
Director nvarchar(255),
Star1 nvarchar(255),
Star2 nvarchar(255),
Star3 nvarchar(255),
Star4 nvarchar(255),
No_of_Votes float,
Gross float
)

INSERT INTO #IMDb_top_1000
SELECT DISTINCT Series_Title, Released_Year, Runtime, Genre, IMDB_Rating, Director, Star1, Star2, Star3, Star4, No_of_Votes, Gross
FROM PortfolioProject..['imdb_top_1000 raw$']

SELECT * FROM #IMDb_top_1000

```




