# IMBd Movies Analysis

## Table of contents 

- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning part.1](#data-cleaning-part-1)
- [Data Cleaning part.2](#data-cleaning-part-2)
- [Data Exploration](#data-exploration)
- [Limitations](#Limitations)
- [Results](#Results)

### Project Overview

This data analysis project aims to provide insights into the performance of movies over the years. By analyzing various aspects of the movie data, we seek to identify trends and make data-driven recommendations for future movie directors to help them in their decisions

### Data Source 

Movies Data : The primary dataset used for this analysis is the 'imdb_top_1000.csv' file, containing detailed information about movie posters, titles, release years, certificates, runtimes, genres, IMDB ratings, overviews, Metascores, directors, stars, number of votes, and gross.

### Tools 

- Excel : Data Cleaning
- SQL : Data Cleaning, Data Exploration and Data Analysis

### Data Cleaning part.1 

In the initial data preparation phase that we did in excel, we performed the following tasks:

1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting

### Data Cleaning part.2

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

There is about 197 missing values out of 1000. We will go further to see during which period there is the most missing data.

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

There is 61 missing values out of 126 which is a lot against 46 out of 157.  So, for the rest of the project, we will primarily focus on films released after 1960 (the year 1960 was chosen randomly), as there are too many missing data before that


### Data Expolration 

In this part, we are gonna explore the data to answer key questions : 
  1. What is the average rating for each genre ?
  2. What Director has the best Ratings ?
  3. What genre brings the most revenue ?
  4. Number of films with gross revenue exceeding 100m over the years ?
  5. Evolution of gross over the years ?
  6. Film Ratings over the years ?
  7. Film Runtimes over the years?



#### 1. What is the average rating for each genre ?

```
SELECT Genre, AVG(Rating) AS AVGRate
From #IMDb_top_1000
GROUP BY Genre 
ORDER BY 2 DESC
```

|Genre|AvgRate|
|-----|-------|
|Animation, Drama, War|8,5|
|Action, Sci-Fi|8,4|
|Drama, Musical|8,4|


#### 2. What Director has the best Ratings  ?

```
SELECT IMBD.Director, AVG(Rating) as AvgRating
FROM #IMDb_top_1000 IMBD
GROUP BY IMBD.Director
ORDER BY 2 DESC
```

|Director|AvgRating|
|--------|---------|
|Frank Darabont|8,95|
|Irvin Kershner|8,7|
|Lana Wachowski|8,7|
|George Lucas|8,6|



#### 3. What genre brings the most revenue ?

```
SELECT IMDB.Genre, AVG(Gross) AS AvgGross
FROM #IMDb_top_1000 AS IMDB 
GROUP BY IMDB.Genre
ORDER BY 2 DESC
```

|Genre|AvgGross|
|--------|---------|
|Family, Sci-Fi|435 110 554|
|Action, Adventure, Fantasy|352 723 505,16|
|Action, Adventure, Family|301 959 197|


#### 4. Most popular genre over the years ?

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



#### 5. Number of films with gross revenue exceeding 100m over the years
```
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


Given that we know the budget for film production is increasing over the years, we observe that there are more and more films with a gross revenue exceeding 100m. The increase in budget is therefore not useless; on the contrary, it seems to be beneficial. 


#### 6. Film Ratings over the years ?


```
SELECT AVG(Rating)
FROM #IMDb_top_1000
WHERE Year between '2000' and '2020'
```

7,91

```
SELECT AVG(Rating)
FROM #IMDb_top_1000
WHERE Year between '1980' and '2000'
```

7,96	

```
SELECT AVG(Rating)
FROM #IMDb_top_1000
WHERE Year between '1960' and '1980'
```

7,98

#### Attention

Based on the previous question, the average rating of films hasn't changed much over the years. However, a criteria should be consider : the number of votes

```
SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '2000' and '2020'
```

158 884 033

```
SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '1980' and '2000'
```

82 582 142

```
SELECT Sum(No_of_Votes)
FROM #IMDb_top_1000
WHERE Year between '1960' and '1980'
```

27 816 959

There are significantly more votes for recent films than for older ones; there are 10 times more votes for films released between 2000 and 2020 than between 1960 and 1980. In that respect, a large number of votes can often lead to a more accurate evaluation of the quality of a film, making the ratings of recent films more precise.



#### 7. Film Runtimes over the years?

We first need to remove the 'min' part from the Runtime column and then convert the data to integers.

```
UPDATE #IMDb_top_1000
SET Runtime = REPLACE(Runtime, 'min', '')
WHERE Runtime LIKE '%min%'

SELECT AVG(convert(int,Runtime)) as AvgTime1
FROM #IMDb_top_1000
WHERE Year BETWEEN '2000' and '2020'
```

125

```
SELECT AVG(convert(int,Runtime)) as AvgTime2
FROM #IMDb_top_1000
WHERE Year BETWEEN '1980' and '2000'
```

122

```
SELECT AVG(convert(int,Runtime)) as AvgTime3
FROM #IMDb_top_1000
WHERE Year BETWEEN '1960' and '1980'
```

124

The average duration of a film has not changed over the years.



### Limitations 

For all our analyses, we have used the average function (AVG) rather than the median, since there is no predefined function, even though the median is more appropriate. We can therefore verify if the data is homogeneous to see if the mean is an appropriate function. To do this, let's look at each column and count the number of elements below the mean. Knowing that there are 1000 values for each column, if there are approximately 500 values below the mean, then it is suitable.

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE (convert(int,Runtime)) < (SELECT AVG(convert(int,Runtime)) FROM #IMDb_top_1000)
```

538

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Gross < (SELECT AVG(Gross) FROM #IMDb_top_1000)
```

568

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE No_of_Votes < (SELECT AVG(No_of_Votes) FROM #IMDb_top_1000)
```

669

```
SELECT COUNT(*)
FROM #IMDb_top_1000
WHERE Rating < (SELECT AVG(Rating) FROM #IMDb_top_1000)
```

537

Thus, except for the number of votes, the mean is an appropriate function.



### Results 

Based on the analysis, we recommend the following actions:

- A movie with a duration of about 2 hours
- The genre Action, Sci-Fi
- Directed by George Lucas, Frank Darabont, Irvin Kershner, or Lana Wachowski


 






