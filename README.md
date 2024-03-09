# IMBd Movies Analysis

### Project Overview

This data analysis project aims to provide insights into the performance of movies with their ratings over the years. By analyzing various aspects of the movies data, we seek to identify trends and make data-driven recommendations for futur movies dorectors and help them in their decisions.

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

Unfortunately, we had some problems when we import the dataset from Excel to SQL, so we had to clean it again on SQL. First, we took columns that are going to be the most interesting which are Title, Released Year, Runtime, Genre, Director, Star 1, Star 2, Star 3, Star 4, No of Votes and Gross, in that respect we are going to create a temp table :

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

#### Missing Data

When we look at the data, we see that a lot of values are missing from Gross and indeed :

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Gross is null

```

There is about 197 missing values out of 1000. We are gonna go further, pour voir à qquelle période ils manquent le plus de données :

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Gross is null AND Year < '1960'

SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Year < '1960'

SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Gross is null AND Year between '1960' and '1980'

SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Year between '1960' and '1980'
```

There is 61 missing values out of 126 which is a lot. Contre 46 pour 157.  So, pour le reste du projet on va principalement s'intéresser aux films sortis après 1960 (l'année 1960a été choisi au hasard), car avant cela il y a trop de données manquantes. 


### Data Expolration 

In this part, we are gonna explore the data to answer key questions : 
  1. What is the average rating for each genre ?
  2. What Director has the best Ratings ?
  3. What genre brings the most revenue ?
  4. Most popular genre over the years ?
  5. Evolution gross au fil des années ?
  6. Notes des films au fil des années ?
  7. Durée des films au fil des années ?



### 1. What is the average rating for each genre ?

```
SELECT Genre, AVG(Rating) AS AVGRate
From #IMDb_top_1000
GROUP BY Genre 
ORDER BY 2 DESC
```



### 2. What Director has the best Ratings  ?

```
SELECT IMBD.Director, AVG(Rating) as AvgRating
FROM #IMDb_top_1000 IMBD
GROUP BY IMBD.Director
ORDER BY 2 DESC
```



### 3. What genre brings the most revenue ?

```
SELECT IMDB.Genre, AVG(Gross)
FROM #IMDb_top_1000 AS IMDB 
GROUP BY IMDB.Genre
ORDER BY 2 DESC
```



### 4. Most popular genre over the years ?

```
WITH FindMax AS (
	SELECT Year, Genre, SUM(NO_of_Votes) AS SumVotes
	FROM #IMDb_top_1000
	GROUP BY Year, Genre
)
SELECT fmax.Year, fmax.SumVotes, fmax.Genre
FROM FindMax fmax
JOIN (
	SELECT Year, Max(SumVotes) AS MaxVotes
	FROM FindMax
	GROUP BY Year
) FindGenre 
ON fmax.Year = FindGenre.Year AND fmax.SumVotes = FindGenre.MaxVotes
```



### 4. Bonus : with the title 

```
WITH FindMax AS (
	SELECT Year, Genre, Title, Sum(No_of_Votes) AS SumVotes
	FROM #IMDb_top_1000
	GROUP BY Year, Genre, Title
),
FindGenre AS (
	SELECT YEAR, Max(SumVotes) AS MaxVotes
	FROM FindMax
	Group by Year
)
SELECT fmax.Year, fmax.SumVotes, fmax.Genre, fmax.Title
FROM FindMax AS fmax
JOIN FindGenre
ON fmax.Year = FindGenre.Year AND fmax.SumVotes = FindGenre.MaxVotes
```



### 5. Evolution gross au fil des années ?

```
SELECT AVG(Gross)
FROM #IMDb_top_1000

SELECT COUNT(*) 
FROM (
	SELECT Year, Gross
	FROM #IMDb_top_1000
	WHERE Gross > 100000000 AND Year BETWEEN '2000' and '2020'
) AS Comp1


SELECT COUNT(*) 
FROM (
	SELECT Year, Gross
	FROM #IMDb_top_1000
	WHERE Gross > 100000000 AND Year BETWEEN '1980' and '2000'
) AS Comp2


SELECT COUNT(*) 
FROM (
	SELECT Year, Gross
	FROM #IMDb_top_1000
	WHERE Gross > 100000000 AND Year BETWEEN '1960' and '1980'
) AS Comp3
```



### 6. Notes des films au fil des années ?

```
SELECT COUNT(*) 
FROM (
	SELECT Year, Rating 
	FROM #IMDb_top_1000
	WHERE Rating > 7.0 AND Year BETWEEN '2000' and '2020'
) AS Comp1

SELECT COUNT(*) 
FROM (
	SELECT Year, Rating 
	FROM #IMDb_top_1000
	WHERE Rating > 7.0 AND Year BETWEEN '1980' and '2000'
) AS Comp2

SELECT COUNT(*) 
FROM (
	SELECT Year, Rating 
	FROM #IMDb_top_1000
	WHERE Rating > 7.0 AND Year BETWEEN '1960' and '1980'
) AS Comp3
```

#### Les nouveaux films sont-ils vraiment meilleurs que les anciens ? 

```
SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '2000' and '2020'

SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '1980' and '2000'

SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '1960' and '1980'
```



### 7. Durée des films au fil des années ?

```
UPDATE #IMDb_top_1000
SET Runtime = REPLACE(Runtime, 'min', '')
WHERE Runtime LIKE '%min%'

SELECT * FROM #IMDb_top_1000

SELECT AVG(convert(int,Runtime)) as AvgTime1
FROM #IMDb_top_1000
WHERE Year BETWEEN '2000' and '2020'

SELECT AVG(convert(int,Runtime)) as AvgTime2
FROM #IMDb_top_1000
WHERE Year BETWEEN '1980' and '2000'

SELECT AVG(convert(int,Runtime)) as AvgTime3
FROM #IMDb_top_1000
WHERE Year BETWEEN '1960' and '1980'
```




 






