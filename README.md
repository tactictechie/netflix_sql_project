# Netflix Movies and Shows Analysis using SQL
![Netflix Logo](https://github.com/tactictechie/netflix_sql_project/blob/main/Logo/netflix_logo.png)

**Author:** Shivang Sagwaliya


## Project Overview

## Objectives

* Analyze the distribution of content types (Movies vs TV Shows).
* Identify the most common ratings for movies and TV shows.
* Explore content by release year, country, and duration.
* Extract genre and cast-based insights (top actors, top countries).
* Categorize content by keywords in descriptions (e.g., `kill`, `violence`).
* Provide ready SQL queries that answer common business questions.

---

## Dataset

* **Source ([original dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)):** Kaggle — Netflix Movies and TV Shows dataset.
* **Working table:** `netflix_data` (see schema and SQL below).

---

## Table Schema & SQL Code 

```sql
CREATE TABLE netflix_data(
	show_id			 nvarchar(50) NULL,
	type			 nvarchar(50) NULL,
	title			 varchar(max) NULL,
	director		 varchar(max) NULL,
	cast			 varchar(max) NULL,
	country			 varchar (max) NULL,
	date_added		 date  NULL,
	release_year	 date  NULL,
	rating			 nvarchar(50) NULL,
    duration		 nvarchar(50) NULL,
	listed_in		 nvarchar(100) NULL,
	description		nvarchar (250) NULL
)  

-- 1. Count the number of Movies vs TV Shows

SELECT type , COUNT(title) AS title
FROM netflix_data
GROUP BY type;


-- 2.Find the most common rating for movies and TV shows
WITH rating_Count AS (
	SELECT
	type ,
	rating,
	COUNT(rating) AS rating_cnt
	FROM netflix_data
	GROUP BY type , rating
	),
rating_rank AS (
SELECT
type,
rating,
rating_cnt ,
ROW_NUMBER() OVER(PARTITION BY type ORDER BY rating_cnt DESC) AS rn
FROM rating_count
)

 SELECT type ,
 rating
 FROM rating_rank
 WHERE rn =1
;

-- 3. List all movies released in a specific year (e.g., 2020)

SELECT * FROM netflix_data
WHERE type = 'movie' AND release_year = 2020

-- 4. Find the top 5 countries with the most content on Netflix

SELECT  country
FROM  ( SELECT  TOP (5) country , COUNT(country) AS cnt
		FROM netflix_data
		GROUP BY country
		ORDER BY COUNT(country) DESC) t);


-- 5. Identify the longest movie
WITH cte AS (
SELECT TOP 1 title, CAST (REPLACE(duration,'Min','') AS INT) AS duration
FROM netflix_data
WHERE type = 'Movie'
ORDER BY  CAST (REPLACE(duration,'Min','') AS INT) DESC
)
SELECT  title ,
CONCAT(duration,' ', 'Min') AS duration
FROM cte
;

-- 6. Find content added in the last 5 years

SELECT *
FROM netflix_data
WHERE YEAR(date_added) >= YEAR(GETDATE()) - 5;

-- 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT * FROM netflix_data
WHERE director LIKE '%Rajiv Chilaka%';

-- 8. List all TV shows with more than 5 seasons

SELECT *
FROM netflix_data
WHERE type = 'TV Show'
  AND CAST(REPLACE(REPLACE(duration, ' Seasons', ''), ' Season', '') AS INT) > 5;

-- 9. Count the number of content items in each genre
SELECT
    TRIM(T.value) AS genre,
    COUNT(*) AS total_content
FROM
    netflix_data
CROSS APPLY
    STRING_SPLIT(listed_in, ',') AS T
GROUP BY
    TRIM(T.value);

-- 10.Find each year and the  number of content release in India on netflix.

SELECT
    YEAR(release_year) AS release_year,
    COUNT(date_added) AS content_count
FROM
    netflix_data
CROSS APPLY
    STRING_SPLIT(country, ',') AS T
WHERE
    TRIM(T.value) = 'India'
GROUP BY
    YEAR(release_year)
ORDER BY
    YEAR(release_year) DESC;


-- 11. List all movies that are documentaries
SELECT show_id , title ,
TRIM(T.value) AS type
FROM netflix_data
CROSS APPLY
STRING_SPLIT(listed_in,',') AS T
WHERE TRIM(T.value) ='Documentaries'
AND type= 'Movie';

-- 12. Find all content without a director

SELECT *
FROM netflix_data
WHERE director IS NULL;

-- 13. Find the top 10 actors who have appeared in the highest number of movies produced in India.

WITH cte AS(
    SELECT TRIM(T.value) AS actors
    FROM netflix_data
    CROSS APPLY
    STRING_SPLIT(cast,',') AS T
    WHERE country LIKE '%India%')
SELECT TOP (10) actors,
COUNT(actors) AS total_appearance
FROM cte
GROUP BY actors
ORDER BY total_appearance DESC;

/*
14.Categorize the content based on the presence of the keywords 'kill' and 'violence' in
the description field. Label content containing these keywords as 'Bad' and all other
content as 'Good'. Count how many items fall into each category.
*/


    SELECT *,
    CASE WHEN LOWER(description) LIKE '%kill%' OR LOWER(description) LIKE '%violence%' OR  LOWER(description) LIKE '%violent%' THEN 'Bad'
         ELSE 'Good'
         END AS category
    FROM netflix_data;
```

---

## Business Questions Covered

This README and the included SQL address the following business questions (15 total):

1. Count Movies vs TV Shows
2. Most common rating by type
3. Movies released in a specific year
4. Top 5 countries by content volume
5. Longest movie
6. Content added in the last 5 years
7. Content by director (example: `Rajiv Chilaka`)
8. TV shows with > 5 seasons
9. Genre counts
10. Yearly content releases in India
11. Documentary movies
12. Content without directors
13. Top 10 actors in Indian-produced movies
14. Content categorization based on keywords

---

## How to Use

1. Load the Kaggle dataset into your SQL Server (or target RDBMS) and create the `netflix_data` table using the schema above.
2. Run the queries in the order presented to reproduce the analysis.
3. Adjust any `WHERE` filters or `TOP` limits to match your reporting needs.

---

## Notes & Assumptions

* The `release_year` column in the schema is a `date` type — some queries use `YEAR(release_year)` to extract the year.
* The `country`, `cast`, `director`, and `listed_in` columns often contain comma-separated values; queries use `STRING_SPLIT` / `CROSS APPLY` / `TRIM` or `UNNEST` logic to normalize them.
* Date parsing and functions (e.g., `GETDATE()`, `YEAR()`) are written for SQL Server syntax.

---

## License

For educational and analytical purposes. © Shivang Sagwaliya

---

If you want, I can also:

* Produce a downloadable `README.md` file for your repo,
* Add a short `CONTRIBUTING.md`, or
* Generate sample results (tables/visuals) using a subset of the dataset you provide.

Tell me which one you want next.

