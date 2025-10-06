# Spotify SQL Project and Query Optimization

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```


### 1. Data Exploration
 The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 2. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

#### LEVEL-1 Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### LEVEL-2 Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### LEVEL-3 Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 3. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

## 15 Questions(Solved)

### Level-1

1. Retrieve the names of all tracks that have more than 1 billion streams.
```sql
SELECT * FROM spotify
WHERE stream > 1000000000
```
2. List all albums along with their respective artists.
```sql
SELECT DISTINCT album,artist FROM spotify
order by 1
SELECT DISTINCT album FROM spotify
order by 1
```
3. Get the total number of comments for tracks where `licensed = TRUE`.
```sql
SELECT SUM(comments) as total_comments FROM spotify
WHERE licensed= true
```
4. Find all tracks that belong to the album type `single`.
```sql
SELECT track FROM spotify
Where album_type= 'single' 
```
5. Count the total number of tracks by each artist.
```sql
SELE
CT DISTINCT artist,
COUNT (*) as total_no_songs
FROM spotify
GROUP BY artist
order by 2 
```
### Level-2
1. Calculate the average danceability of tracks in each album.
```sql
SELECT album, avg(danceability) as avg_danceability
FROM spotify
GROUP BY 1
ORDER BY 2 DESC
```
2. Find the top 5 tracks with the highest energy values.
```sql
SELECT TRACK, MAX(ENERGY) FROM SPOTIFY
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```
3. List all tracks along with their views and likes where `official_video = TRUE`.
```sql
SELECT track, 
sum(views) as total_views,
sum(likes) as total_likes
from spotify
where official_video='true'
group by 1
order by 2 desc
```
4. For each album, calculate the total views of all associated tracks.**
```sql
SELECT ALBUM, TRACK,
SUM(VIEWS) AS TOTATL_VIEWS
FROM SPOTIFY
GROUP BY 1,2
ORDER BY 3 DESC
```
5. Retrieve the track names that have been streamed on Spotify more than YouTube.**
```sql
SELECT * FROM 
(SELECT
track,
COALESCE(SUM(CASE WHEN most_played_on = 'Youtube'then stream end),0) as streamed_on_youtube,
COALESCE(SUM(CASE WHEN most_played_on = 'Spotify'then stream end),0) as streamed_on_spotify
from spotify
group by 1
) as t1
where
streamed_on_spotify > streamed_on_youtube
AND
streamed_on_youtube <> 0

```
### Level-3
1. Find the top 3 most-viewed tracks for each artist using window functions.**
```sql
WITH ranking_artist
AS
(SELECT artist,track,SUM(VIEWS) as total_views,
DENSE_RANK() OVER(PARTITION BY ARTIST ORDER BY SUM(VIEWS) DESC) AS RANK
FROM spotify
group by 1,2
order by 1,3 desc)
SELECT * FROM ranking_artist
WHERE rank<= 3
```
2. Write a query to find tracks where the liveness score is above the average.**
```sql
SELECT track,liveness
FROM spotify
WHERE liveness > (SELECT AVG(liveness) FROM spotify)
```
3. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   
4. Find tracks where the energy-to-liveness ratio is greater than 1.2.**
```sql
SELECT
    TRACK,
	ENERGY/LIVENESS AS  energy_to_liveness_ratio
FROM spotify
WHERE ENERGY/LIVENESS> 1.2
```
5. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.**
```sql

```
---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools used**: pgAdmin 4 




