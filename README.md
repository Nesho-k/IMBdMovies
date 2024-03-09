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



#### 5. Nombre de film ayant un gross supérieur à 100m au fil des années 

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

Sachant que l'on sait que le budget pour la production d'un film augmente de plus en plus au fil des années, on remarque qu'il y a de plus en plus de films avec un gross supérieur à 100m. L'augmentation du budget n'est donc pas inutile au contraire.   


#### 6. Notes des films au fil des années ?

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

D'après la question précédente, la note moyenne des films n'a pas tellement évolué au fil des années. Néanmoins, un critère est à prendre en compte, le nombre de votes : 

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

Il y a beaucoup plus de votes pour les films récents qu'anciens, il y en a 10 fois plus pour les films sortis entre 2000 et 2020, qu'entre 1960 et 1980. Ainsi sachant qu'il est plus difficile d'avoir une bonne note lorsque le nombre de votes est plus faible, on pourrait penser qu'il y a plus de bons films à l'époque que maintenant. Toutefois ce constat est à prendre avec des pincettes, d'autres facteurs doivent également être pris en compte. 



#### 7. Durée des films au fil des années ?

Nous devons tout d'abord enlever la partie 'min' dans la colonne Runtime puis convertir les données en int. 

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

La durée moyenne d'un film n'a donc pas évoluer au fil des années.


### Limitation 

Pour la totalité de nos analyses nous avons utiliser la fonction moyenne (AVG), plutôt que la médianne puisqu'il n'existe pas de fonction prédéfini alors que cette dernière est plus pertinente. On peut donc vérifier si les données sont assez homogènes, et ainsi utiliser la moyenne (pour ne pas avoir des trucs abbérants). Pour cela, regardons pour chaque colonne, le nombre de d'éléments inférieur à la moyenne. Sacahnt qu'il y a 1000 valeurs pour chaque colonne, si il y environ 500 valeur en dessous de la moyenne alors la fonction moyenne est pertinente pour cette colonne. 

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

CCL :

### Résultats / conclusion 


 






