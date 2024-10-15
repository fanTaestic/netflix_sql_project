# Netflix Movies and TV shows Data Analysis using SQL
![Netflix Logo](https://github.com/fanTaestic/netflix_sql_project/blob/main/Netflix%20frenar%C3%A1%20a%20los%20usuarios%20que%20comparten%20la%20contrase%C3%B1a.jpeg)

## Overview
The project involves a comprehensive analysis on Netflix movies and TV shows data using SQL. The goal is to extract valuable insights and answers various Business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solution, finding and conclusions.

## Objectives
- Analyze the distribution of content types (movies vs TV Shows).
- Identify the most common ratings for movies and TV Shows.
- List and analyze content based on release years, countries and durations.
- Explore and categorized content based on specific criteria and keywords.

## Dataset
The data for this project is sourced from the Kaggle dataset.
Dataset Link: https://www.kaggle.com/datasets/shivamb/netflix-shows

## Schema
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
   show_id	VARCHAR(6),
   type VARCHAR(10),
   title VARCHAR(150),
   director	VARCHAR(208),
   casts VARCHAR(1000),
   country VARCHAR(150),
   date_added VARCHAR(50),
   release_year	INT,
   rating VARCHAR(10),
   duration	VARCHAR(15),
   listed_in VARCHAR(100),
   description VARCHAR(250)
);
```

## 15 BUSINESS PROBLEMS AND SOLUTIONS

### 1. COUNT THE NUMBER OF MOVIES VS TV SHOWS
```sql
SELECT type,
COUNT(*) AS TOTAL_CONTENT
FROM netflix
GROUP BY type
```

### 2. FIND THE MOST COMMON RATING FOR MOVIES AND TV SHOWS
```sql
SELECT 
	   type,
	   rating
FROM 
	(SELECT 
	   type,
	   rating,
	   COUNT(*),
	   RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1, 2
) as t1
WHERE 
	ranking = 1
```
  
### 3. LIST ALL MOVIES RELEASED IN A SPECIFIC YEAR (EG., 2020)
   --FILTER DATA FROM 2020 AND FOR MOVIES ONLY
```sql   
SELECT * FROM NETFLIX
WHERE 
	type = 'Movie'
	and release_year = 2020
```

### 4. FIND THE TOP 5 COUNTRIES WITH THE MOST CONTENT ON NETFLIX
```sql
SELECT 
	country,
	COUNT(show_id) as total_content
FROM NETFLIX
GROUP BY 1

-- split countries by comma 
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country
FROM netflix

-- merging both codes
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5 --TOP 5 COUNTRIES
```

### 5. IDENTIFY THE LONGEST MOVIE
```sql
SELECT * FROM netflix
WHERE 
	type = 'Movie'
	and
	duration= (Select max(duration)FROM netflix)
```

### 6. FIND CONTENT ADDED IN THE LAST 5 YEARS
```sql
SELECT *,
	TO_DATE(date_added, 'Month DD, YYYY')   --to convert into actual date
FROM netflix 

SELECT CURRENT_DATE - INTERVAL '5 years' --FUNCTION THAT RETURNS 5 YEARS OLD DATE

SELECT *
FROM netflix 
WHERE 
	TO_DATE(date_added, 'Month DD, YYYY')>= CURRENT_DATE - INTERVAL '5 years'
```	

### 7. FIND ALL THE MOVIES/TV SHOWS BY DIRECTOR 'RAJIV CHILAKA'!
```sql
SELECT * FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%'    --ILIKE for case sensitive data
```

### 8. LIST ALL TV SHOWS WITH MORE THAN 5 SEASONS
```sql
SELECT *
	--SPLIT_PART(duration, ' ', 1) as sessions
FROM netflix
WHERE 
	type = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::numeric > 5 
```

### 9. COUNT THE NUMBER OF CONTENT ITEMS IN EACH GENRE 
```sql
SELECT
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
```

### 10. FIND EACH YEAR AND THE AVERAGE NUMBERS OF CONTENT RELEASE BY INDIA ON NETFLIX. RETURN TOP 5 YEAR WITH HIGHEST AVG CONTENT RELEASE.
```sql
SELECT
	EXTRACT(YEAR FROM TO_DATE(date_added, 'MONTH, DD, YYYY')) as year,
	COUNT(*)
FROM NETFLIX
WHERE country = 'India'
GROUP BY 1

-- AVERAGE (333/972 EG.)

SELECT
	EXTRACT(YEAR FROM TO_DATE(date_added, 'MONTH, DD, YYYY')) as year,
	COUNT(*) as yearly_content,
	ROUND(
	COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 
	100, 2) as avg_content_per_year
	--total count of content from India
FROM NETFLIX
WHERE country = 'India'
GROUP BY 1
```

### 11. List all the movies that are documentaries
```sql
SELECT * FROM netflix
WHERE
	listed_in ILIKE '%documentaries%'
```

### 12. Find all content without a director 
```sql
SELECT * FROM netflix
WHERE
	director IS NULL
```

### 13. Find how many movies actor Salman Khan appeared in last 10 years
```sql
SELECT * FROM netflix
WHERE
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT 
--show_id, casts
UNNEST(STRING_TO_ARRAY(casts, ',')) AS actors,
COUNT(*) AS total_content
FROM netflix 
WHERE country ILIKE '%India%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

```sql
WITH new_table
AS 
(
SELECT 
*,
	CASE 
	WHEN description ILIKE '%kill%' OR 
		description ILIKE '%violence%' THEN 'Bad_Content'

		ELSE 'Good_Content'
	END category
FROM netflix
)
SELECT
	category,
	COUNT(*) as total_content
FROM new_table
GROUP BY 1
```
