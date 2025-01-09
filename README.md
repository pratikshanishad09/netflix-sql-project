# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/pratikshanishad09/netflix-sql-project/blob/main/netflix.avif)

## Overview
This is my first ever project which involves a comprehensive analysis of Netflix Movies and TV Shows using SQL. 

## Objective 
> Analyze the distribution of content types[Movies and TV Shows].
> Identify the most common ratings for movies and TV shows.
> List and analyze content based on release years, countries, and durations.
> Explore and categorizes content based on specific criteria and keywords.

## Dataset 
The data for this project is sourced from the kaggle dataset:
 > Dataset Link:- (https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema
```sql
--Netflix Project--

DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
 show_id VARCHAR(6),
 type VARCHAR(10),
 title VARCHAR(150),
 director VARCHAR(208),
 casts VARCHAR(1000),	
 country VARCHAR(150),	
 date_added VARCHAR(150),	
 release_year INT,	
 rating VARCHAR(10),	
 duration VARCHAR(15), 	
 listed_in VARCHAR(100),	
 description VARCHAR(250)
);

SELECT * FROM netflix;

SELECT 
   COUNT(*) as total_content 
FROM netflix;

SELECT 
   DISTINCT type 
FROM netflix;
```

## -- 15 Business problems--

## -- 1. Count the number of Movies vs TV shows
```sql
SELECT
   type,
   count(*) as content 
FROM netflix
GROUP BY type
```

## --Find the most common rating for the moving and series
```sql
SELECT 
   type,
   rating
FROM 
(
SELECT 
  type,
  rating,
  COUNT(*),
  RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
  FROM netflix
GROUP BY 1,2
) as t1
WHERE 
   ranking = 1
```

## -- 3. List all the movies released in a specific year(e.g...2020)
```sql
--filter 2020
--movies

SELECT * FROM netflix
WHERE 
    type = 'Movie'
    AND
	release_year = 2020
```

## --4. Find the top 5 countries with the most content on netflix
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5

SELECT 
    UNNEST(STRING_TO_ARRAY(country, ',')) as new_country
FROM netflix
```

## --5. Identify the longest movie
```sql
SELECT * FROM netflix
WHERE 
    type = 'Movie'
	AND 
	duration = (SELECT MAX(duration) FROM netflix)
```

## --6. Find the content added in the last 5 years
```sql
SELECT * FROM netflix
WHERE 
    TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'

SELECT CURRENT_DATE - INTERVAL '5 years'
```

## --7. Find all the movies/TV shows by director 'Rajiv chikala'
```sql
SELECT* FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%'
```

## -- 8.List all TV shows with more than 5 seasons
```sql
SELECT 
    * 
FROM netflix
WHERE
     type = 'TV Show'
	 AND
	 SPLIT_PART(duration, ' ', 1)::numeric > 5 
```

## -- 9. Count the number of content items in each genre
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id)
FROM netflix
GROUP BY 1
```

### -- 10. Find each year and the average numbers of content release by India on netflix, return top 5 year with highest avg content release
```sql
total content 333/972

SELECT 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year,
	COUNT(*) as yearly_content,
	ROUND(
	COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100, 2)
	as avg_content_per_year
FROM netflix
WHERE country = 'India'
GROUP BY 1
```

## -- 11. List all movies that are documentries
```sql
SELECT * FROM netflix
WHERE
    listed_in ILIKE '%documentaries%'
```

## -- 12.Find all the content without a director
```sql
SELECT * FROM netflix
WHERE 
    director IS NULL
```

## -- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years.
```sql
SELECT * FROM netflix
WHERE
    casts ILIKE '%Salman Khan%' 
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

## -- 14.Find the top 10 actors who have appeared in the highest number of movies produced in india.
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

## -- 15.categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field,. Label content containing these keyowrds as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
```sql
WITH new_table
AS
(
SELECT 
    *,
	CASE
	WHEN description ILIKE '%kill%'OR
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


