**Olympics SQL Data Engineering Portfolio**

**Overview**

This repository contains a comprehensive collection of SQL queries written to analyze historical Olympic Games data. The exercises utilize a relational database schema (til_data_engineering.olympics) to extract insights about athletes, national committees, historic editions, and sporting events. The queries demonstrate a clear progression from basic data retrieval to highly advanced analytical window functions and complex table joins.

**Dataset Schema Context**

The queries interact with the following tables within the olympics schema:

```athletes:``` Competitor demographics (DOB, height, weight, status).

```nocs:``` National Olympic Committees and country mapping.

```editions:``` Historical data on host cities, seasons, and participant/medal totals.

```sport_groups, disciplines, events:``` The hierarchy of Olympic sports.

```results:``` Granular records of athlete event participation and medals won.

**Skills Demonstrated**

**Basic & String Manipulation:** ```SELECT```, ```WHERE```, ```LIKE```, ```LEFT()```, ```RIGHT()```, ```CONCAT()```, ```UPPER()```, ```LOWER()```, ```LENGTH()```.

**Date Math & Arithmetic:** ```DATEDIFF()```, ```DATE_PART()```, simple arithmetic calculations, and ratios.

**Conditional Logic:** ```CASE WHEN``` statements for custom categorizations and demographic ratios.

**Aggregations:** ```COUNT()```, ```SUM()```, ```AVG()```, ```MIN()```, ```MAX()```, combined with ```GROUP BY``` and ```HAVING```.

**Advanced Joins:** Multi-table relational mapping, Anti-Joins (IS NULL).

**Advanced Analytics:** Common Table Expressions (CTEs), Subqueries, and Window Functions (```RANK()```, ```DENSE_RANK()```, ```ROW_NUMBER()```, ```LAG()```, ```OVER(PARTITION BY...)```, Cumulative Sums, Rolling Averages).

A few examples of the questions included in this section are listed below ...

**🟢 Beginner Level (Data Retrieval & Filtering)**

1. Historic Female Competitors: Retrieve names and birth dates of deceased female competitors, sorted oldest to youngest.

```
SELECT 
    full_name,
    sex,
    date_of_birth
FROM til_data_engineering.olympics.athletes
WHERE sex = 'Female' 
  AND status = 'Dead'
ORDER BY date_of_birth ASC;
```

2. Pattern Matching (NOCs): Find countries starting and ending with 'A' (case-insensitive).

```
SELECT country_name
FROM til_data_engineering.olympics.nocs
WHERE LOWER(LEFT(country_name, 1)) = 'a' 
  AND LOWER(RIGHT(country_name, 1)) = 'a';
```

3. Basic Arithmetic: Calculate the difference between total participants and total medals across all editions.

```
SELECT
    SUM(total_participants),
    SUM(total_medals),
    SUM(total_participants) - SUM(DISTINCT total_medals) AS difference
FROM til_data_engineering.olympics.editions;
```

4. Filtering & Sorting: Identify active competitors weighing exactly 75 kg.

```
SELECT 
    full_name,
    country_of_birth,
    weight_kg
FROM til_data_engineering.olympics.athletes
WHERE weight_kg = 75 
  AND status = 'Alive'
ORDER BY full_name DESC;
```

5. Simple Aggregation: Sum of total medals across all Winter editions.

```
SELECT SUM(total_medals)
FROM til_data_engineering.olympics.editions
WHERE season = 'Winter';
```

6. Formatting & Deduplication: Retrieve unique seasons from sports groups, capitalized.

```
SELECT UPPER(season)
FROM til_data_engineering.olympics.sport_groups
GROUP BY season;
```

7. BMI Calculation: Calculate BMI for active athletes.

```
SELECT 
    full_name,
    ROUND(weight_kg / POWER(height_cm / 100, 2), 2) AS bmi
FROM til_data_engineering.olympics.athletes;
```

(Additional Beginner Queries Included in Repository: Range Filtering, Date Extractions, NULL Filtering, and String Concatenation)

**🟡 Intermediate Level (Joins, Aggregations & Logic)**

1. Longest Lived Competitors: Top 10 competitors by lifespan (excluding living/unknown).

```
SELECT 
    full_name,
    DATEDIFF(years, date_of_birth, date_of_death) AS lifespan
FROM til_data_engineering.olympics.athletes
WHERE status != 'Alive' 
  AND date_of_death IS NOT NULL
ORDER BY lifespan DESC
LIMIT 10;
```

2. Multi-Entity Aggregation (Medal-Heavy Disciplines): Disciplines awarding strictly more than 500 medals historically.

```
SELECT 
    d.discipline_name,
    COUNT(r.medal) AS total_medal
FROM til_data_engineering.olympics.disciplines AS d
JOIN til_data_engineering.olympics.events AS e ON d.discipline_id = e.discipline_id 
JOIN til_data_engineering.olympics.results AS r ON r.event_id = e.event_id
GROUP BY d.discipline_name
HAVING total_medal > 500
ORDER BY total_medal DESC;
```

3. Conditional Averages (Physical Attributes): Average height/weight of male medalists.

```
SELECT 
    sex,
    ROUND(AVG(height_cm), 1) AS avg_height,
    ROUND(AVG(weight_kg), 1) AS avg_weight
FROM til_data_engineering.olympics.athletes AS a
LEFT JOIN til_data_engineering.olympics.results AS r ON a.athlete_id = r.athlete_id
WHERE medal IS NOT NULL 
  AND height_cm IS NOT NULL 
  AND weight_kg IS NOT NULL
GROUP BY sex;
```

4. Demographic Ratios (CASE Statements): Proportion of female competitors per country.

```
SELECT 
    country_of_birth,
    ROUND((COUNT(CASE WHEN sex = 'Female' THEN 1 END) / COUNT(*)) * 100) AS perc_of_total
FROM athletes
GROUP BY country_of_birth
HAVING COUNT(*) > 50
ORDER BY perc_of_total DESC;
```

5. Custom Categorization (Weight Classes):

```
SELECT 
    full_name,
    weight_kg,
    CASE 
        WHEN weight_kg < 65 THEN 'Lightweight'
        WHEN weight_kg >= 65 AND weight_kg <= 90 THEN 'Middleweight'
        WHEN weight_kg > 90 THEN 'Heavyweight'
    END AS weight_cat
FROM til_data_engineering.olympics.athletes
WHERE weight_kg IS NOT NULL;
```


🔴 Advanced Level (Window Functions, CTEs & Complex Joins)


1. Cross-Seasonal Versatility (HAVING COUNT DISTINCT): Competitors with medals in both Summer and Winter editions.

```
SELECT 
    full_name,
    COUNT(medal) AS total_medals
FROM til_data_engineering.olympics.results AS r 
JOIN til_data_engineering.olympics.editions AS e ON r.edition_id = e.edition_id
JOIN til_data_engineering.olympics.athletes AS a ON a.athlete_id = r.athlete_id
WHERE medal IS NOT NULL 
GROUP BY full_name
HAVING COUNT(DISTINCT season) = 2;
```

2. Category Maximums (ROW_NUMBER): Edition year of highest medal count per country.
```
WITH cte AS (
    SELECT 
        noc.country_name,
        ed.edition_year,
        COUNT(res.medal) AS total_medal_count
    FROM til_data_engineering.olympics.nocs AS noc
    JOIN til_data_engineering.olympics.results AS res ON noc.noc = res.noc_id
    JOIN til_data_engineering.olympics.editions AS ed ON res.edition_id = ed.edition_id
    GROUP BY country_name, edition_year
),
cte2 AS (    
    SELECT 
        country_name,
        edition_year,  
        total_medal_count,
        ROW_NUMBER() OVER (PARTITION BY country_name ORDER BY total_medal_count DESC, edition_year DESC) AS max_medals
    FROM cte 
)
SELECT 
    country_name,
    edition_year,
    total_medal_count
FROM cte2
WHERE max_medals = 1 
ORDER BY total_medal_count DESC;
```

3. Year-over-Year Growth (LAG Window Function): Participant growth tracking.

```
SELECT 
    edition_year,
    season,
    total_participants,
    LAG(total_participants, 1) OVER (
        PARTITION BY season 
        ORDER BY edition_year ASC
    ) AS previous_participant_count
FROM til_data_engineering.olympics.editions;
```

4. Advanced Window Aggregation & Percentages: 2012 Medal percentage share by country.
```
WITH q1 AS (
    SELECT 
        country_name,
        COUNT(medal) AS total_medals
    FROM til_data_engineering.olympics.nocs AS nocs
    LEFT JOIN til_data_engineering.olympics.results AS res ON nocs.noc = res.noc_id
    LEFT JOIN til_data_engineering.olympics.editions AS ed ON res.edition_id = ed.edition_id
    WHERE edition_year = 2012
    GROUP BY country_name
)
SELECT 
    country_name,
    (total_medals / SUM(total_medals) OVER ()) * 100 AS perc_of_total
FROM q1
ORDER BY perc_of_total DESC;
```

5. Rolling Calculations (ROWS BETWEEN): Track historical medal bloat in Summer games using a rolling average of 3 editions.
```
SELECT 
    edition_year,
    ROUND(AVG(total_medals) OVER (
        ORDER BY edition_year ASC 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 1) AS rolling_medal_avg
FROM editions
WHERE season = 'Summer'
ORDER BY edition_year ASC;
```

6. Category Dominance (RANK): The single most successful discipline for every National Olympic Committee.
```
SELECT 
    country_name,
    RANK() OVER (PARTITION BY country_name ORDER BY COUNT(medal) DESC) AS dominant_cat,
    discipline_name,
    COUNT(medal) AS total_medals
FROM nocs AS n 
JOIN results AS r ON n.noc = r.noc_id
JOIN editions AS ed ON r.edition_id = ed.edition_id
JOIN events AS e ON e.event_id = r.event_id
JOIN disciplines AS d ON d.discipline_id = e.discipline_id
GROUP BY country_name, discipline_name
QUALIFY dominant_cat = 1
ORDER BY country_name;
```
