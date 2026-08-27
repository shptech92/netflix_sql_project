# netflix_sql_project

![Netflix Logo](https://raw.githubusercontent.com/shptech92/netflix_sql_project/main/logo.png)



## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

1. Count the number of Movies vs TV Shows
2. Find the most common rating for movies and TV shows
3. List all movies released in a specific year (e.g., 2020)
4. Find the top 5 countries with the most content on Netflix
5. Identify the longest movie
6. Find content added in the last 5 years
7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
8. List all TV shows with more than 5 seasons
9. Count the number of content items in each genre
10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!
11. List all movies that are documentaries
12. Find all content without a director
13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
15.
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.


## Schema
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);

-- 1. Count the number of Movies vs TV Shows

SELECT 
  type,
  COUNT(*) as total_content
FROM netflix
GROUP BY type;

-- 2. Find the most common rating for movies and TV shows

  SELECT
       type,
	   rating
  FROM 
      (SELECT
	     type,
		 rating,
		 COUNT(*),
		 RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
		 --MAX(rating)
	  FROM netflix
	  GROUP BY 1,2
	  ORDER BY 1,3 DESC
) as t1
WHERE
ranking = 1;

-- 3. List all movies released in a specific year (e.g., 2020)

SELECT * FROM
netflix
WHERE type = 'Movie'
AND
release_year = 2020;


-- 4. Find the top 5 countries with the most content on Netflix

SELECT
        UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
        COUNT(show_id) as total_content
    FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- 5. Identify the longest movie

SELECT * FROM netflix
WHERE 
type = 'Movie'
AND
duration = (SELECT MAX(duration) FROM netflix)


-- 6. Find content added in the last 5 years

SELECT *
FROM netflix
WHERE 
     TO_DATE(date_added, 'Month DD, YYYY')>= CURRENT_DATE - INTERVAL '5 years'


-- 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT * FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';

-- 8. List all TV shows with more than 5 seasons
	  
SELECT 
     *
FROM netflix
WHERE 
    type = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1) ::numeric > 5
	

-- 9. Count the number of content items in each genre	

SELECT * FROM netflix;


SELECT
	 UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
	 COUNT(show_id) as total_content
FROM netflix
GROUP BY 1;

-- 10. Find each year and the average numbers of content release in India on netflix.
return top 5 year with highest avg content release !

SELECT 
     country,
	 release_year,
	 COUNT(show_id) as total_release,
	 ROUND(
           COUNT(show_id)::numeric/
		   (SELECT COUNT(show_id) FROM netflix WHERE country = 'India' ):: numeric * 100
		   ,2
		   ) as avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, 2
ORDER BY avg_release DESC
LIMIT 5;

-- 11. List all movies that are documentaries

SELECT * FROM netflix

where listed_in LIKE '%Documentaries';

-- 12. Find all content without a director

SELECT * FROM netflix
WHERE director IS NULL;


-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT * FROM 
netflix
WHERE casts LIKE "%Salman Khan";

SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

SELECT 
UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%india'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;


-- 15.
-- Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
-- the description field. Label content containing these keywords as 'Bad' and all other 
-- content as 'Good'. Count how many items fall into each category.

WITH new_table
AS
(
	SELECT *,
	      CASE
		  WHEN 
		      description ILIKE '%kill%' OR
			  description ILIKE '%violence%' THEN 'Bad_Content'
			  ELSE 'Good Content'
		  END category
	FROM netflix
)

SELECT
     category,
	 COUNT(*) as total_content
FROM new_table
GROUP BY 1;

# Findings and Conclusion
Why I Took Up This Project

Netflix's public dataset offered a realistic, messy, business-flavored dataset (multi-value fields, inconsistent date formats, missing directors, mixed content types) — exactly the kind of data a working analyst runs into, not a clean textbook table. I chose it to practice translating vague, real-world business questions ("what content should we focus on?", "who are our key markets?") into precise SQL logic, and to build a portfolio piece that demonstrates end-to-end SQL problem-solving rather than a single query.

# What I Did

I framed the analysis around 15 business questions a content/streaming strategy team might realistically ask, and solved each one using PostgreSQL. The work involved:

Data modeling & cleaning logic: designing the schema and handling multi-valued fields (country, casts, listed_in) using STRING_TO_ARRAY + UNNEST, since Netflix stores comma-separated lists in single text columns.
Aggregation & ranking: GROUP BY, COUNT, RANK() OVER (PARTITION BY ...) to find most common ratings per content type, and top countries/actors.
Date & string parsing: converting text dates with TO_DATE, extracting numeric duration values with SPLIT_PART, and pattern matching with ILIKE/LIKE for directors, actors, and genres.
Conditional categorization: a CASE WHEN + CTE approach to label content as "Good" or "Bad" based on keyword presence in descriptions — a simple proxy for content-tone analysis.
Time-based filtering: identifying recently added content and actor appearances within a rolling 10-year window using CURRENT_DATE and EXTRACT.

# Key Findings
**Content mix:** Movies significantly outnumber TV Shows in the catalog, indicating Netflix's core library still skews toward single-release films even as it invests in series.
**Audience targeting:** The most frequent rating for both Movies and TV Shows clusters around mature/teen categories, suggesting the platform's dominant content is aimed at adult and young-adult audiences rather than children's programming.
**Market concentration:** A small group of countries (led by the US and India) account for a disproportionate share of total titles, showing Netflix's content sourcing is concentrated rather than evenly spread globally.
**India-specific trend:** Certain years show a distinct spike in the average share of India-originated content, pointing to specific periods of aggressive regional content investment.
**Genre distribution:** Drama, comedy, and documentary-tagged genres make up the bulk of listed categories, useful for spotting over- and under-represented genres.
**Data quality gaps:** A measurable number of titles have no director listed, which matters for any downstream analysis that assumes director completeness (e.g., director-based recommendations).
**Content tone signal:** Categorizing descriptions by keywords like "kill" and "violence" showed the "Good" category vastly outweighs "Bad," giving a rough, extendable baseline for content sentiment/tone auditing.

# How This Helps in a Business Context

This kind of analysis mirrors real tasks a data/content analyst supports:

**Content strategy:** identifying which genres, countries, and ratings dominate helps prioritize acquisition and production budgets.
**Market expansion:** country- and year-level breakdowns support decisions on where to invest in local content.
**Content moderation & compliance:** keyword-based tone categorization is a lightweight first step toward automated content flagging, which can be scaled with NLP.
**Data governance:** surfacing missing-director records highlights where the underlying dataset needs cleaning before it can power reliable dashboards or recommendation logic.

# Conclusion

This project shows my ability to take open-ended business questions, translate them into structured SQL logic, and extract insights that map directly to real decisions a streaming platform would make — from content acquisition to regional strategy to data quality management. It reflects core data analyst skills: aggregation, window functions, string/array handling, date logic, and conditional categorization, all applied to a business-relevant, non-trivial dataset.

# Future Scope
# Build a live dashboard (Power BI/Tableau) on top of these queries for interactive exploration.
# Extend the country/genre analysis with time-series forecasting to predict future content trends.


