# RSVP-Movies-Case-Study-SQL-MySQL
Tech Stack: MySQL I have analysed the RSVP Movies database to derive insights on movie performance, production patterns, and trends. Using advanced SQL (joins, subqueries, CTEs), you built queries that identified top-performing genres, directors, actors, and revenue contributors.


USE imdb;

/* Now that you have imported the data sets, let’s explore some of the tables. 
 To begin with, it is beneficial to know the shape of the tables and whether any column has null values.
 Further in this segment, you will take a look at 'movies' and 'genre' tables.*/

-- Segment 1:

-- Q1. Find the total number of rows in each table of the schema?
-- Type your code below:

SELECT table_name, table_rows
FROM information_schema.tables
WHERE table_schema = "imdb";

-- OUTPUT 
/*
director_mapping	3867
genre	14662
movie	6844
names	25182
ratings	7927
role_mapping	15142
*/
--  ------------------------------------------------------------------------------------------------------------------------------

-- Q2. Which columns in the movie table have null values?
-- Type your code below:

select 
#count(*) as total_rows
count(*)-count(id) as id_null
,count(*)-count(title) as title_null 
,count(*)-count('year') as year_null 
,count(*)-count(date_published) as date_published_null 
,count(*)-count(country) as country_null 
,count(*)-count(worlwide_gross_income) as worlwide_gross_income_null 
,count(*)-count(languages) as languages_null 
,count(*)-count(production_company) as production_company_null 
,count(*)-count(duration) as duration_null 
from movie;

-- OUTPUT :
-- null for country=20, worlwide_gross_income=3724,languages=194,production_company=528

--  ------------------------------------------------------------------------------------------------------------------------------

-- Now as you can see four columns of the movie table has null values. Let's look at the at the movies released each year. 
-- Q3. Find the total number of movies released each year? How does the trend look month wise? (Output expected)

/* Output format for the first part:

+---------------+-------------------+
| Year			|	number_of_movies|
+-------------------+----------------
|	2017		|	2134			|
|	2018		|		.			|
|	2019		|		.			|
+---------------+-------------------+


Output format for the second part of the question:
+---------------+-------------------+
|	month_num	|	number_of_movies|
+---------------+----------------
|	1			|	 134			|
|	2			|	 231			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:

select Year ,count(distinct id) As Number_of_movies
  from movie
  group by Year 
  order by year
;
SELECT month(date_published) as month_num, count(distinct id) as number_of_movies
 FROM movie
 group by month(date_published)
 order by month_num
;
  
-- OUTPUT Year
-- Year, Number_of_movies
-- 2017	3052
-- 2018	2944
-- 2019	2001

-- OUTPUT month
-- month_num, number_of_movies
-- 1	804
-- 2	640
-- 3	824
-- 4	680
-- 5	625
-- 6	580
-- 7	493
-- 8	678
-- 9	809
-- 10	801
-- 11	625
-- 12	438

/*The highest number of movies is produced in the month of March.
So, now that you have understood the month-wise trend of movies, let’s take a look at the other details in the movies table. 
We know USA and India produces huge number of movies each year. Lets find the number of movies produced by USA or India for the last year.*/

--  ------------------------------------------------------------------------------------------------------------------------------
  
-- Q4. How many movies were produced in the USA or India in the year 2019??
-- Type your code below:

SELECT country, COUNT(country) AS 'movie produced in India or USA 2019'
FROM movie
WHERE year =2019 and country='USA' OR country = 'India' 
GROUP BY country;

WITH USA_INDIA 
AS
(
select *  
from movie 
where country regexp("USA") or country regexp("INDIA")
)
select count(distinct id) AS "Movie_produced_in_India_or_USA_2019"
from USA_INDIA 
where year = 2019;

-- OUTPUT 
-- Movie_produced_in_India_or_USA_2019
-- 1059

/* USA and India produced more than a thousand movies(you know the exact number!) in the year 2019.
Exploring table Genre would be fun!! 
Let’s find out the different genres in the dataset.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q5. Find the unique list of the genres present in the data set?
-- Type your code below:

Select distinct(genre)
from genre;

-- OUTPUT  
-- # genre
-- Action
-- Adventure
-- Comedy
-- Crime
-- Drama
-- Family
-- Fantasy
-- Horror
-- Mystery
-- Others
-- Romance
-- Sci-Fi
-- Thriller


/* So, RSVP Movies plans to make a movie of one of these genres.
Now, wouldn’t you want to know which genre had the highest number of movies produced in the last year?
Combining both the movie and genres table can give more interesting insights. */

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q6.Which genre had the highest number of movies produced overall?
-- Type your code below:

select genre , count(movie_id) AS Number_of_movies
from genre as g 
left join movie as m
on g.movie_id = m.id
group by genre
order by count(movie_id)  desc 
limit 1
;

-- OUTPUT  
-- #genre  Number_of_movies
-- Drama	4285

-- ANSWER 
--  Drama has highest number of movies produced with 4285 movies.

/* So, based on the insight that you just drew, RSVP Movies should focus on the ‘Drama’ genre. 
But wait, it is too early to decide. A movie can belong to two or more genres. 
So, let’s find out the count of movies that belong to only one genre.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q7. How many movies belong to only one genre?
-- Type your code below:
with Single_genre_movie
AS 
(
Select movie_id,title, count(genre) AS number_of_genre
 from genre as G
 inner join movie as M
 on G.movie_id = M.id
 group by id
having number_of_genre=1
)
select count(distinct movie_id) as number_of_sigle_genere_movies 
from Single_genre_movie ;

-- OUTPUT  
-- number_of_sigle_genere_movies
-- 3289


/* There are more than three thousand movies which has only one genre associated with them.
So, this figure appears significant. 
Now, let's find out the possible duration of RSVP Movies’ next project.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q8.What is the average duration of movies in each genre? 
-- (Note: The same movie can belong to multiple genres.)

/* Output format:

+---------------+-------------------+
| genre			|	avg_duration	|
+-------------------+----------------
|	thriller	|		105			|
|	.			|		.			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:

WITH Single_genre_movie 
AS 
(
SELECT M.id, M.title, G.genre, M.duration, COUNT(G.genre) AS number_of_genre
FROM genre AS G
INNER JOIN movie AS M 
ON G.movie_id = M.id
GROUP BY M.id, G.genre
HAVING number_of_genre = 1
)
SELECT genre, ROUND(AVG(duration)) AS average_duration
FROM Single_genre_movie
GROUP BY genre;

/* Now you know, movies of genre 'Drama' (produced highest in number in 2019) has the average duration of 106.77 mins.
Lets find where the movies of genre 'thriller' on the basis of number of movies.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q9.What is the rank of the ‘thriller’ genre of movies among all the genres in terms of number of movies produced? 
-- (Hint: Use the Rank function)
/* Output format:
+---------------+-------------------+---------------------+
| genre			|		movie_count	|		genre_rank    |	
+---------------+-------------------+---------------------+
|drama			|	2312			|			2		  |
+---------------+-------------------+---------------------+*/
-- Type your code below:

Select genre , count(distinct id) as movie_count , 
RANK()OVER (order by count(distinct id) Desc ) AS genre_rank 
from genre AS G
INNER JOIN movie AS M 
ON G.movie_id = M.id
GROUP BY  G.genre;

/*Thriller movies is in top 3 among all genres in terms of number of movies
 In the previous segment, you analysed the movies and genres tables. 
 In this segment, you will analyse the ratings table as well.
To start with lets get the min and max values of different columns in the table*/

select * from ratings;
-- Segment 2:

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q10.  Find the minimum and maximum values in  each column of the ratings table except the movie_id column?
/* Output format:
+---------------+-------------------+---------------------+----------------------+-----------------+-----------------+
| min_avg_rating|	max_avg_rating	|	min_total_votes   |	max_total_votes 	 |min_median_rating|min_median_rating|
+---------------+-------------------+---------------------+----------------------+-----------------+-----------------+
|		0		|			5		|	       177		  |	   2000	    		 |		0	       |	8			 |
+---------------+-------------------+---------------------+----------------------+-----------------+-----------------+*/
-- Type your code below:
select 
round(min(avg_rating)) AS min_avg_rating,
round(max(avg_rating)) AS max_avg_rating,
min(total_votes) AS min_total_votes,
max(total_votes) AS max_total_votes,
min(median_rating) AS min_median_rating,
max(median_rating) AS min_median_rating
from ratings;

/* So, the minimum and maximum values in each column of the ratings table are in the expected range. 
This implies there are no outliers in the table. 
Now, let’s find out the top 10 movies based on average rating.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q11. Which are the top 10 movies based on average rating?
/* Output format:
+---------------+-------------------+---------------------+
| title			|		avg_rating	|		movie_rank    |
+---------------+-------------------+---------------------+
| Fan			|		9.6			|			5	  	  |
|	.			|		.			|			.		  |
|	.			|		.			|			.		  |
|	.			|		.			|			.		  |
+---------------+-------------------+---------------------+*/
-- Type your code below:
-- It's ok if RANK() or DENSE_RANK() is used too

With Ranked_Ratings
AS
(
Select Title,avg_rating,
RANK() OVER(order by avg_rating desc)  AS movie_rank 
From ratings AS R
Inner Join Movie as M
on M.id = R.movie_id 
WINDOW window_rating AS (order by avg_rating DESC)
)
select * from Ranked_Ratings
where movie_rank <=10;

/* Do you find you favourite movie FAN in the top 10 movies with an average rating of 9.6? If not, please check your code again!!
So, now that you know the top 10 movies, do you think character actors and filler actors can be from these movies?
Summarising the ratings table based on the movie counts by median rating can give an excellent insight.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q12. Summarise the ratings table based on the movie counts by median ratings.
/* Output format:

+---------------+-------------------+
| median_rating	|	movie_count		|
+-------------------+----------------
|	1			|		105			|
|	.			|		.			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:
-- Order by is good to have
select median_rating,count(distinct id) as movie_count 
from ratings as r 
inner join movie as m
ON r.movie_id= m.id
group by median_rating 
order by movie_count; 

/* Movies with a median rating of 7 is highest in number. 
Now, let's find out the production house with which RSVP Movies can partner for its next project.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q13. Which production house has produced the most number of hit movies (average rating > 8)??
/* Output format:
+------------------+-------------------+---------------------+
|production_company|movie_count	       |	prod_company_rank|
+------------------+-------------------+---------------------+
| The Archers	   |		1		   |			1	  	 |
+------------------+-------------------+---------------------+*/
-- Type your code below:

With production_house_hits -- ||This code give us a CTE production_house_hits which has moivie details that have avg_rating > 8||
as
(
select production_company,title,avg_rating,id
from movie as M
inner join ratings as R 
on M.id = R.movie_id
where avg_rating >8
)
select production_company , count(distinct id) as movie_count,
DENSE_RANK()over(Order by count(distinct id) desc)AS prod_company_rank
from production_house_hits
group by production_company
order by movie_count DESC
;
-- -- Ans:  Rank 2 :  Dream Warrior Pictures or National Theatre Live || With null values 21 as rank 1 which can be ignored.

-- It's ok if RANK() or DENSE_RANK() is used too
-- Answer can be Dream Warrior Pictures or National Theatre Live or both

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q14. How many movies released in each genre during March 2017 in the USA had more than 1,000 votes?
/* Output format:

+---------------+-------------------+
| genre			|	movie_count		|
+-------------------+----------------
|	thriller	|		105			|
|	.			|		.			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:

WITH movies_released_March_2017_USA_1000Votes 
AS 
(
SELECT id,year,date_published,country,genre,total_votes
FROM movie AS M
INNER JOIN ratings AS R 
ON M.id = R.movie_id
INNER JOIN genre AS G 
ON M.id = G.movie_id
WHERE
YEAR = 2017
AND month(date_published)=3
AND country regexp 'USA' 
AND total_votes>1000
)
SELECT genre,count(distinct id) as movie_count
FROM movies_released_March_2017_USA_1000Votes
group by genre;
-- Order by movie_count desc;

-- OUTPUT  
-- # genre	movie_count
-- Drama	   24
-- Comedy	   9
-- Action	   8
-- Thriller	   8
-- Sci-Fi	   7
-- Crime	   6
-- Horror	   6
-- Mystery	   4
-- Romance	   4
-- Adventure   3
-- Fantasy	   3
-- Family	   1

--  ------------------------------------------------------------------------------------------------

-- Lets try to analyse with a unique problem statement.
-- Q15. Find movies of each genre that start with the word ‘The’ and which have an average rating > 8?
/* Output format:
+---------------+-------------------+---------------------+
| title			|		avg_rating	|		genre	      |
+---------------+-------------------+---------------------+
| Theeran		|		8.3			|		Thriller	  |
|	.			|		.			|			.		  |
|	.			|		.			|			.		  |
|	.			|		.			|			.		  |
+---------------+-------------------+---------------------+*/
-- Type your code below:

With movies_start_the_rating_8_above
AS
(
Select M.title,R.avg_rating,G.genre,M.id
from movie as M
inner join ratings as R
on R.movie_id = M.id
inner join genre as G
on G.movie_id = M.id
Where avg_rating > 8 
AND M.title like "The%"   -- Selecting movies that commence with 'The' and possess an average rating exceeding 8.
)
Select title,avg_rating,genre from movies_start_the_rating_8_above
order by genre;

-- OUTPUT  
-- # title	                   #avg_rating	     #genre
-- Theeran Adhigaaram Ondru	      8.3	         Action
-- The Irishman	                  8.7	         Crime
-- Theeran Adhigaaram Ondru		  8.3            Crime
-- The Gambinos	                  8.4	         Crime
-- The Blue Elephant 2	          8.8	         Drama
-- The Brighton Miracle		      9.5            Drama
-- The Irishman	                  8.7	         Drama
-- The Colour of Darkness	      9.1	         Drama
-- The Mystery of Godliness:The Sequel	8.5	     Drama
-- The Gambinos	                  8.4	         Drama
-- The King and I	              8.2	         Drama
-- The Blue Elephant 2	          8.8	         Horror
-- The Blue Elephant 2	          8.8	         Mystery
-- The King and I	              8.2	         Romance
-- Theeran Adhigaaram Ondru	      8.3	         Thriller

--  ------------------------------------------------------------------------------------------------------------------------------

-- You should also try your hand at median rating and check whether the ‘median rating’ column gives any significant insights.
-- Q16. Of the movies released between 1 April 2018 and 1 April 2019, how many were given a median rating of 8?
-- Type your code below:

With Movies_01April_2018_2019
AS
(
Select id,date_published,median_rating 
from movie as M
inner join ratings as R
on R.movie_id = M.id
where date_published BETWEEN '2018-04-01' AND '2019-04-01'
AND median_rating = 8
)
Select median_rating ,count(distinct id) AS movie_count from Movies_01April_2018_2019
order by movie_count;

-- OUTPUT
-- # median_rating	movie_count
-- 8	361

-- Ans: median_rating : 8 has movie_count 361

--  ------------------------------------------------------------------------------------------------------------------------------

-- Once again, try to solve the problem given below.
-- Q17. Do German movies get more votes than Italian movies? 
-- Hint: Here you have to find the total number of votes for both German and Italian movies.
-- Type your code below:

with german_summary AS (
SELECT SUM(r.total_votes) AS german_total_votes,
RANK() OVER(ORDER BY SUM(r.total_votes)) AS unique_id
FROM movie AS m
INNER JOIN ratings AS r
ON m.id=r.movie_id
WHERE m.languages LIKE '%German%'
), italian_summary AS (
SELECT SUM(r.total_votes) AS italian_total_votes,
RANK() OVER(ORDER BY sum(r.total_votes)) AS unique_id
FROM movie AS m
INNER JOIN ratings AS r
ON m.id=r.movie_id
WHERE m.languages LIKE '%Italian%'
) SELECT *,
CASE
	WHEN german_total_votes > italian_total_votes THEN 'Yes' ELSE 'No'
    END AS 'German_Movie_Is_Popular_Than_Italian_Movie'
FROM german_summary
INNER JOIN
italian_summary
USING(unique_id); 

/*
Output
unique_id, german_total_votes, italian_total_votes, German_Movie_Is_Popular_Than_Italian_Movie
1			4421525					2559540					Yes

*/
-- Answer is Yes

--  ------------------------------------------------------------------------------------------------------------------------------

/* Now that you have analysed the movies, genres and ratings tables, let us now analyse another table, the names table. 
Let’s begin by searching for null values in the tables.*/

-- Segment 3:

-- Q18. Which columns in the names table have null values??
/*Hint: You can find null values for individual columns or follow below output format
+---------------+-------------------+---------------------+----------------------+
| name_nulls	|	height_nulls	|date_of_birth_nulls  |known_for_movies_nulls|
+---------------+-------------------+---------------------+----------------------+
|		0		|			123		|	       1234		  |	   12345	    	 |
+---------------+-------------------+---------------------+----------------------+*/
-- Type your code below:

SELECT
SUM(CASE WHEN name IS NULL OR name = '' THEN 1 ELSE 0 END) AS name_nulls,
SUM(CASE WHEN height IS NULL OR height = '' THEN 1 ELSE 0 END) AS height_nulls,
SUM(CASE WHEN date_of_birth IS NULL THEN 1 ELSE 0 END) AS date_of_birth_nulls,
SUM(CASE WHEN known_for_movies IS NULL THEN 1 ELSE 0 END) AS known_for_movies_nulls
FROM names;
-- OUTPUT  
-- #name_nulls  #height_nulls  #date_of_birth_nulls  #known_for_movies_nulls
--       0	       17335	           13431	           15226

-- Ans: No null values in name column.

--  ------------------------------------------------------------------------------------------------

/* There are no Null value in the column 'name'.
The director is the most important person in a movie crew. 
Let’s find out the top three directors in the top three genres who can be hired by RSVP Movies.*/

-- Q19. Who are the top three directors in the top three genres whose movies have an average rating > 8?
-- (Hint: The top three genres would have the most number of movies with an average rating > 8.)
/* Output format:

+---------------+-------------------+
| director_name	|	movie_count		|
+---------------+-------------------|
|James Mangold	|		4			|
|	.			|		.			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:

WITH top_3_genres AS
(
           SELECT     genre,
                      Count(m.id)                            AS movie_count ,
                      Rank() OVER(ORDER BY Count(m.id) DESC) AS genre_rank
           FROM       movie                                  AS m
           INNER JOIN genre                                  AS g
           ON         g.movie_id = m.id
           INNER JOIN ratings AS r
           ON         r.movie_id = m.id
           WHERE      avg_rating > 8
           GROUP BY   genre limit 3 )
SELECT     n.NAME            AS director_name ,
           Count(d.movie_id) AS movie_count
FROM       director_mapping  AS d
INNER JOIN genre G
using     (movie_id)
INNER JOIN names AS n
ON         n.id = d.name_id
INNER JOIN top_3_genres
using     (genre)
INNER JOIN ratings
using      (movie_id)
WHERE      avg_rating > 8
GROUP BY   NAME
ORDER BY   movie_count DESC limit 3 ;

-- OUTPUT 
-- # director_name	movie_count
--  James Mangold	4
--  Anthony Russo	3
--  Soubin Shahir	3

-- Answer :Drama,Comedy,Thriller these are the top 3 Genre 
-- Answer :James Mangold,Anthony Russo and Soubin Shahir are the Directors with top three genres whose movies have an average rating > 8

--  ------------------------------------------------------------------------------------------------------------------------------

/* James Mangold can be hired as the director for RSVP's next project. Do you remeber his movies, 'Logan' and 'The Wolverine'. 
Now, let’s find out the top two actors.*/

-- Q20. Who are the top two actors whose movies have a median rating >= 8?
/* Output format:

+---------------+-------------------+
| actor_name	|	movie_count		|
+-------------------+----------------
|Christain Bale	|		10			|
|	.			|		.			|
+---------------+-------------------+ */
-- Type your code below:

With TOP_ACTORS
AS
(
Select N.name as actor_name ,RM.movie_id,RM.name_id,R.median_rating ,category
from role_mapping as RM
inner join ratings as R 
on R.movie_id = RM.movie_id
inner join names as N
on N.id = RM.name_id
where category = 'actor' AND median_rating >=8
)
Select actor_name,count(movie_id) as movie_count from TOP_ACTORS
Group by actor_name
order by movie_count desc
limit 2;

--  OUTPUT:
-- actor_name, movie_count:
-- 'Mammootty', '8'
-- 'Mohanlal', '5'

-- ANSWER : TOP two actors whose movies have a median rating >= 8 are Mammootty and Mohanlal

--  ------------------------------------------------------------------------------------------------------------------------------

/* Have you find your favourite actor 'Mohanlal' in the list. If no, please check your code again. 
RSVP Movies plans to partner with other global production houses. 
Let’s find out the top three production houses in the world.*/

-- Q21. Which are the top three production houses based on the number of votes received by their movies?
/* Output format:
+------------------+--------------------+---------------------+
|production_company|vote_count			|		prod_comp_rank|
+------------------+--------------------+---------------------+
| The Archers		|		830			|		1	  		  |
|	.				|		.			|			.		  |
|	.				|		.			|			.		  |
+-------------------+-------------------+---------------------+*/
-- Type your code below:

With production_company
AS
(
select production_company, sum(total_votes) as vote_count,
rank() over(order by sum(R.total_votes) desc) as prod_comp_rank
from movie as M
inner join ratings as R
on R.movie_id = M.id
Group by production_company
order by prod_comp_rank
)
select production_company, vote_count,prod_comp_rank from production_company
where prod_comp_rank <= 3
order by prod_comp_rank;

-- OUTPUT:
-- production_company, vote_count, prod_comp_rank
-- 'Marvel Studios', '2656967', '1'
-- 'Twentieth Century Fox', '2411163', '2'
-- 'Warner Bros.', '2396057', '3'

-- ANSWER : Top of the list Rank is 'Marvel Studios' with '2656967' votes.

/*Yes Marvel Studios rules the movie world.
So, these are the top three production houses based on the number of votes received by the movies they have produced.

Since RSVP Movies is based out of Mumbai, India also wants to woo its local audience. 
RSVP Movies also wants to hire a few Indian actors for its upcoming project to give a regional feel. 
Let’s find who these actors could be.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q22. Rank actors with movies released in India based on their average ratings. Which actor is at the top of the list?
-- Note: The actor should have acted in at least five Indian movies. 
-- (Hint: You should use the weighted average based on votes. If the ratings clash, then the total number of votes should act as the tie breaker.)

/* Output format:
+---------------+-------------------+---------------------+----------------------+-----------------+
| actor_name	|	total_votes		|	movie_count		  |	actor_avg_rating 	 |actor_rank	   |
+---------------+-------------------+---------------------+----------------------+-----------------+
|	Yogi Babu	|			3455	|	       11		  |	   8.42	    		 |		1	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
+---------------+-------------------+---------------------+----------------------+-----------------+*/
-- Type your code below:

WITH top_actor
     AS (SELECT n.NAME
                AS
                actor_name,
                Sum(r.total_votes)
                AS
                   total_votes,
                Count(DISTINCT rm.movie_id)
                AS
                   movie_count,
                Round(Sum(r.avg_rating * r.total_votes) / Sum(r.total_votes), 2)
                AS
                actor_avg_rating
         FROM   role_mapping rm
                INNER JOIN names n
                        ON rm.name_id = n.id
                INNER JOIN ratings r
                        ON rm.movie_id = r.movie_id
                INNER JOIN movie m
                        ON rm.movie_id = m.id
         WHERE  rm.category = 'actor'
                AND m.country LIKE '%India%'
         GROUP  BY rm.name_id,
                   n.NAME
         HAVING Count(DISTINCT rm.movie_id) >= 5)
SELECT *,
       Rank()
         OVER (
           ORDER BY actor_avg_rating DESC) AS actor_rank
FROM   top_actor; 

-- OUTPUT : 
/*
Vijay Sethupathi	23114	5	8.42	1
Fahadh Faasil	13557	5	7.99	2
Yogi Babu	8500	11	7.83	3
Joju George	3926	5	7.58	4
Ammy Virk	2504	6	7.55	5
Dileesh Pothan	6235	5	7.52	6
Kunchacko Boban	5628	6	7.48	7
Pankaj Tripathi	40728	5	7.44	8
Rajkummar Rao	42560	6	7.37	9
Dulquer Salmaan	17666	5	7.30	10
Amit Sadh	13355	5	7.21	11
Tovino Thomas	11596	8	7.15	12
Mammootty	12613	8	7.04	13
Nassar	4016	5	7.03	14
Karamjit Anmol	1970	6	6.91	15
Hareesh Kanaran	3196	5	6.58	16
Naseeruddin Shah	12604	5	6.54	17
Anandraj	2750	6	6.54	17
Mohanlal	17622	7	6.47	19
Siddique	5953	7	6.43	20
Aju Varghese	2237	5	6.43	20
Prakash Raj	8548	6	6.37	22
Jimmy Sheirgill	3826	6	6.29	23
Biju Menon	1916	5	6.21	24
Mahesh Achanta	2716	6	6.21	24
Suraj Venjaramoodu	4284	6	6.19	26
Abir Chatterjee	1413	5	5.80	27
Sunny Deol	4594	5	5.71	28
Radha Ravi	1483	5	5.70	29
Prabhu Deva	2044	5	5.68	30
Atul Sharma	9604	5	4.78	31
*/

-- ANSWER:Vijay Sethupathi is at the top actor of the list.

-- Top actor is Vijay Sethupathi

--  ------------------------------------------------------------------------------------------------------------------------------

-- Q23.Find out the top five actresses in Hindi movies released in India based on their average ratings? 
-- Note: The actresses should have acted in at least three Indian movies. 
-- (Hint: You should use the weighted average based on votes. If the ratings clash, then the total number of votes should act as the tie breaker.)
/* Output format:
+---------------+-------------------+---------------------+----------------------+-----------------+
| actress_name	|	total_votes		|	movie_count		  |	actress_avg_rating 	 |actress_rank	   |
+---------------+-------------------+---------------------+----------------------+-----------------+
|	Tabu		|			3455	|	       11		  |	   8.42	    		 |		1	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
+---------------+-------------------+---------------------+----------------------+-----------------+*/
-- Type your code below:

WITH top_actress
     AS (SELECT n.NAME
                AS
                actress_name,
                Sum(r.total_votes)
                AS
                   total_votes,
                Count(DISTINCT rm.movie_id)
                AS
                   movie_count,
                Round(Sum(r.avg_rating * r.total_votes) / Sum(r.total_votes), 2)
                AS
                actress_avg_rating
         FROM   role_mapping rm
                INNER JOIN names n
                        ON rm.name_id = n.id
                INNER JOIN ratings r
                        ON rm.movie_id = r.movie_id
                INNER JOIN movie m
                        ON rm.movie_id = m.id
         WHERE  rm.category = 'actress'
                AND m.country like '%India%'
                and m.languages like '%HINDI%'
         GROUP  BY rm.name_id,
                   n.NAME
         HAVING Count(DISTINCT rm.movie_id) >= 3)
SELECT *,
       Rank()
         OVER (
           ORDER BY actress_avg_rating DESC) AS actress_rank
FROM   top_actress
limit 5; 
-- OUTPUT : 
-- actress_name, total_votes, movie_count, actress_avg_rating, actress_rank
-- Taapsee Pannu	18061	3	7.74	1
-- Kriti Sanon	21967	3	7.05	2
-- Divya Dutta	8579	3	6.88	3
-- Shraddha Kapoor	26779	3	6.63	4
-- Kriti Kharbanda	2549	3	4.80	5

-- ANSWER : Taapsee Pannu tops teh list of actresses in Hindi movies released in India based on their average ratings

--  ------------------------------------------------------------------------------------------------------------------------------

/* Taapsee Pannu tops with average rating 7.74. 
Now let us divide all the thriller movies in the following categories and find out their numbers.*/

/* Q24. Select thriller movies as per avg rating and classify them in the following category: 

			Rating > 8: Superhit movies
			Rating between 7 and 8: Hit movies
			Rating between 5 and 7: One-time-watch movies
			Rating < 5: Flop movies
--------------------------------------------------------------------------------------------*/
-- Type your code below:
WITH thriller_movies
     AS (SELECT DISTINCT title,
                         avg_rating
         FROM   movie AS M
                INNER JOIN ratings AS R
                        ON R.movie_id = M.id
                INNER JOIN genre AS G using(movie_id)
         WHERE  genre LIKE 'THRILLER')
SELECT *,
       CASE
         WHEN avg_rating > 8 THEN 'Superhit movies'
         WHEN avg_rating BETWEEN 7 AND 8 THEN 'Hit movies'
         WHEN avg_rating BETWEEN 5 AND 7 THEN 'One-time-watch movies'
         ELSE 'Flop movies'
       END AS avg_rating_category
FROM   thriller_movies;

-- OUTPUT :
-- title, avg_rating, avg_rating_category
-- 'Der müde Tod', '7.7', 'Hit movies'
-- 'Fahrenheit 451', '4.9', 'Flop movies'
-- 'Pet Sematary', '5.8', 'One-time-watch movies'

-- ANSWER : 
-- Safe has the highest  rating movie  which was a Superhit
-- Roofied has the lowest rating movie which was a flop

/* Until now, you have analysed various tables of the data set. 
Now, you will perform some tasks that will give you a broader understanding of the data in this segment.*/

--  ------------------------------------------------------------------------------------------------------------------------------

-- Segment 4:

-- Q25. What is the genre-wise running total and moving average of the average movie duration? 
-- (Note: You need to show the output table in the question.) 
/* Output format:
+---------------+-------------------+---------------------+----------------------+
| genre			|	avg_duration	|running_total_duration|moving_avg_duration  |
+---------------+-------------------+---------------------+----------------------+
|	comdy		|			145		|	       106.2	  |	   128.42	    	 |
|		.		|			.		|	       .		  |	   .	    		 |
|		.		|			.		|	       .		  |	   .	    		 |
|		.		|			.		|	       .		  |	   .	    		 |
+---------------+-------------------+---------------------+----------------------+*/
-- Type your code below:

SELECT genre,
		ROUND(AVG(duration),2) AS avg_duration,
        SUM(ROUND(AVG(duration),2)) OVER(ORDER BY genre ROWS UNBOUNDED PRECEDING) AS running_total_duration,
        AVG(ROUND(AVG(duration),2)) OVER(ORDER BY genre ROWS 10 PRECEDING) AS moving_avg_duration
FROM movie AS m 
INNER JOIN genre AS g 
ON m.id= g.movie_id
GROUP BY genre
ORDER BY genre;

/*
output

Action	112.88	112.88	112.880000
Adventure	101.87	214.75	107.375000
Comedy	102.62	317.37	105.790000
Crime	107.05	424.42	106.105000
Drama	106.77	531.19	106.238000
Family	100.97	632.16	105.360000
Fantasy	105.14	737.30	105.328571
Horror	92.72	830.02	103.752500
Mystery	101.80	931.82	103.535556
Others	100.16	1031.98	103.198000
Romance	109.53	1141.51	103.773636
Sci-Fi	97.94	1239.45	102.415455
Thriller	101.58	1341.03	102.389091
*/
--  ------------------------------------------------------------------------------------------------------------------------------

-- Q26. Which are the five highest-grossing movies of each year that belong to the top three genres? 
-- (Note: The top 3 genres would have the most number of movies.)

/* Output format:
+---------------+-------------------+---------------------+----------------------+-----------------+
| genre			|	year			|	movie_name		  |worldwide_gross_income|movie_rank	   |
+---------------+-------------------+---------------------+----------------------+-----------------+
|	comedy		|			2017	|	       indian	  |	   $103244842	     |		1	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
+---------------+-------------------+---------------------+----------------------+-----------------+*/
-- Type your code below:

-- Top 3 Genres based on most number of movies
WITH top_genres AS
(
           SELECT     genre,
                      Count(m.id)                            AS movie_count ,
                      Rank() OVER(ORDER BY Count(m.id) DESC) AS genre_rank
           FROM       movie                                  AS m
           INNER JOIN genre                                  AS g
           ON         g.movie_id = m.id
           INNER JOIN ratings AS r
           ON         r.movie_id = m.id
           WHERE      avg_rating > 8
           GROUP BY   genre limit 3 )
           ,movie_summary AS
(
           SELECT     genre,
                      year,
                      title AS movie_name,
                      CAST(replace(replace(ifnull(worlwide_gross_income,0),'INR',''),'$','') AS decimal(10)) AS worlwide_gross_income ,
                      DENSE_RANK() OVER(partition BY year ORDER BY CAST(replace(replace(ifnull(worlwide_gross_income,0),'INR',''),'$','') AS decimal(10))  DESC ) AS movie_rank
           FROM       movie                                                                     AS m
           INNER JOIN genre                                                                     AS g
           ON         m.id = g.movie_id
           WHERE      genre IN
                      (
                             SELECT genre
                             FROM   top_genres)
           --  GROUP BY   movie_name
           )
SELECT genre, year, movie_name, concat('$',worlwide_gross_income) as worlwide_gross_income, movie_rank
FROM   movie_summary
WHERE  movie_rank<=5
ORDER BY YEAR;

select genre , count(movie_id) AS Number_of_movies
from genre as g 
left join movie as m
on g.movie_id = m.id
group by genre
order by count(movie_id)  desc 
limit 3
;

-- OUTPUT:
-- # genre	YEAR	movie_name	worlwide_gross_income	movie_rank
/*
'Action', '2017', 'Star Wars: Episode VIII - The Last Jedi', '$1332539889', '1'
'Action', '2017', 'The Fate of the Furious', '$1236005118', '2'
'Comedy', '2017', 'Despicable Me 3', '$1034799409', '3'
'Action', '2017', 'Jumanji: Welcome to the Jungle', '$962102237', '4'
'Comedy', '2017', 'Jumanji: Welcome to the Jungle', '$962102237', '4'
'Action', '2017', 'Spider-Man: Homecoming', '$880166924', '5'
'Action', '2018', 'Avengers: Infinity War', '$2048359754', '1'
'Action', '2018', 'Black Panther', '$1346913161', '2'
'Action', '2018', 'Jurassic World: Fallen Kingdom', '$1308467944', '3'
'Action', '2018', 'The Villain', '$1300000000', '4'
'Action', '2018', 'Incredibles 2', '$1242805359', '5'
'Action', '2019', 'Avengers: Endgame', '$2797800564', '1'
'Drama', '2019', 'Avengers: Endgame', '$2797800564', '1'
'Drama', '2019', 'The Lion King', '$1655156910', '2'
'Action', '2019', 'Spider-Man: Far from Home', '$1131845802', '3'
'Action', '2019', 'Captain Marvel', '$1128274794', '4'
'Comedy', '2019', 'Toy Story 4', '$1073168585', '5'
/*
--  ------------------------------------------------------------------------------------------------------------------------------

-- Finally, let’s find out the names of the top two production houses that have produced the highest number of hits among multilingual movies.
-- Q27.  Which are the top two production houses that have produced the highest number of hits (median rating >= 8) among multilingual movies?
/* Output format:
+-------------------+-------------------+---------------------+
|production_company |movie_count		|		prod_comp_rank|
+-------------------+-------------------+---------------------+
| The Archers		|		830			|		1	  		  |
|	.				|		.			|			.		  |
|	.				|		.			|			.		  |
+-------------------+-------------------+---------------------+*/
-- Type your code below:
SELECT production_company, COUNT(m.id) AS movie_count,
ROW_NUMBER() OVER(ORDER BY count(id) DESC) AS prod_comp_rank
FROM movie AS m 
INNER JOIN ratings AS r 
ON m.id=r.movie_id
WHERE median_rating>=8 AND production_company IS NOT NULL AND POSITION(',' IN languages)>0  -- a movie when its median rating is equal to or greater than 8.
GROUP BY production_company
limit 2;

-- OUTPUT  
-- # production_company	#movie_count  #prod_comp_rank
-- Star Cinema	            7	            1
-- Twentieth Century Fox	4	            2

--  ------------------------------------------------------------------------------------------------

-- Multilingual is the important piece in the above question. It was created using POSITION(',' IN languages)>0 logic
-- If there is a comma, that means the movie is of more than one language


-- Q28. Who are the top 3 actresses based on number of Super Hit movies (average rating >8) in drama genre?
/* Output format:
+---------------+-------------------+---------------------+----------------------+-----------------+
| actress_name	|	total_votes		|	movie_count		  |actress_avg_rating	 |actress_rank	   |
+---------------+-------------------+---------------------+----------------------+-----------------+
|	Laura Dern	|			1016	|	       1		  |	   9.60			     |		1	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
|		.		|			.		|	       .		  |	   .	    		 |		.	       |
+---------------+-------------------+---------------------+----------------------+-----------------+*/
-- Type your code below:

with Actress_summary
as(
SELECT n. name as actress_name , SUM(total_votes) AS total_votes,
		COUNT(rm.movie_id) AS movie_count,

        Round(Sum(avg_rating*total_votes)/Sum(total_votes),2) as Weighted_Average_Rating
FROM names AS n
INNER JOIN role_mapping AS rm
ON n.id = rm.name_id
INNER JOIN ratings AS r
ON r.movie_id = rm.movie_id
INNER JOIN genre AS g
ON r.movie_id = g.movie_id
INNER JOIN movie as m
ON  r.movie_id = m.id
WHERE category = 'actress' AND r.avg_rating > 8 AND genre = 'drama' -- filtering by actresses, drama genre and movies with avg_rating >8
GROUP BY NAME
)

SELECT   *,
         Rank() OVER(ORDER BY movie_count DESC) AS actress_rank
FROM     Actress_summary LIMIT 3;


-- OUTPUT  
-- # actress_name	   #total_votes 	#movie_count	#Weighted_Average_Rating	#actress_rank
-- Parvathy Thiruvothu	 4974		        2                   8.25	                1
-- Susan Brown	          656			    2                   8.94                    1
-- Amanda Lawrence	      656			    2                   8.94                    1

--  ------------------------------------------------------------------------------------------------

/* Q29. Get the following details for top 9 directors (based on number of movies)
Director id
Name
Number of movies
Average inter movie duration in days
Average movie ratings
Total votes
Min rating
Max rating
total movie durations

Format:
+---------------+-------------------+---------------------+----------------------+--------------+--------------+------------+------------+----------------+
| director_id	|	director_name	|	number_of_movies  |	avg_inter_movie_days |	avg_rating	| total_votes  | min_rating	| max_rating | total_duration |
+---------------+-------------------+---------------------+----------------------+--------------+--------------+------------+------------+----------------+
|nm1777967		|	A.L. Vijay		|			5		  |	       177			 |	   5.65	    |	1754	   |	3.7		|	6.9		 |		613		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
|	.			|		.			|			.		  |	       .			 |	   .	    |	.		   |	.		|	.		 |		.		  |
+---------------+-------------------+---------------------+----------------------+--------------+--------------+------------+------------+----------------+

--------------------------------------------------------------------------------------------*/
-- Type you code below:

WITH next_date_published_summary AS
(
           SELECT     d.name_id,
                      NAME,
                      d.movie_id,
                      duration,
                      r.avg_rating,
                      total_votes,
                      m.date_published,
                      Lead(date_published,1) OVER(partition BY d.name_id ORDER BY date_published,movie_id ) AS next_date_published
           FROM       director_mapping                                                                      AS d
           INNER JOIN names                                                                                 AS n
           ON         n.id = d.name_id
           INNER JOIN movie AS m
           ON         m.id = d.movie_id
           INNER JOIN ratings AS r
           ON         r.movie_id = m.id ), top_director_summary AS
(
       SELECT *,
              Datediff(next_date_published, date_published) AS date_difference
       FROM   next_date_published_summary )
SELECT   name_id                       AS director_id,
         NAME                          AS director_name,
         Count(movie_id)               AS number_of_movies,
         Round(Avg(date_difference),2) AS avg_inter_movie_days,
         Round(Avg(avg_rating),2)               AS avg_rating,
         Sum(total_votes)              AS total_votes,
         Min(avg_rating)               AS min_rating,
         Max(avg_rating)               AS max_rating,
         Sum(duration)                 AS total_duration
FROM     top_director_summary
GROUP BY director_id
ORDER BY Count(movie_id) DESC limit 9;

-- OUTPUT
-- # director_id   #director_name #number_of_movies #avg_inter_movie_days #avg_rating	total_votes	min_rating	max_rating	total_duration
-- nm2096009	     Andrew Jones	      5	           190.75	             3.02	     1989	       2.7	       3.2	       432
-- nm1777967	      A.L. Vijay	      5	           176.75	             5.42	     1754	       3.7	        6.9	       613
-- nm0814469	      Sion Sono	          4	          331.00	             6.03	     2972	       5.4	        6.4	       502
-- nm0831321	      Chris Stokes	      4	          198.33	             4.33	     3664	       4.0	        4.6	       352
-- nm0515005	      Sam Liu	          4	         260.33	                 6.23	     28557	       5.8	        6.7	       312
-- nm0001752	      Steven Soderbergh	  4	         254.33	                 6.48	    171684	        6.2	        7.0	       401
-- nm0425364	       Jesse V. Johnson	  4	         299.00	                 5.45	     14778	        4.2	       6.5	       383
-- nm2691863	       Justin Price       4	          315.00	             4.50	     5343	        3.0	       5.8	       346
-- nm6356309	       Özgür Bakar	      4	           112.00	             3.75	     1092	        3.1	       4.9   	   374




