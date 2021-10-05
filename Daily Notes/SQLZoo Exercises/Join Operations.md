## SQLZOO Exercises
### :triangular_flag_on_post: **JOIN** Operations

- The tables contain all matches and goals from UEFA EURO 2012 Football Championship in Poland and Ukraine.
- Exercises areavalable at [SQLZoo](https://sqlzoo.net/wiki/SQL_Tutorial)
- Data Schema :


    ![alt text](https://sqlzoo.net/w/images/c/ce/FootballERD.png)

**Show the matchid and player name for all goals scored by Germany. To identify German players, check for: teamid = 'GER'**

 ````sql
 SELECT matchid,player
 FROM goal 
 WHERE teamid ='GER'; 
````

> Notice that the column **matchid** in the *goal* table corresponds to the **id** column in the *game* table
 
 **Show id, stadium, team1, team2 for just game 1012**
 
 ````sql
 SELECT id,stadium,team1,team2
FROM game
WHERE id=1012;
````

> We can combine the two steps into a single query with a **JOIN** operation.

````sql
SELECT *
FROM game JOIN goal ON (game.id=goal.matchid);
  ````
**Show the player, teamid, stadium and mdate for every German goal.**

````sql
SELECT player,teamid,stadium,mdate
FROM game JOIN goal ON (game.id=goal.matchid)
WHERE teamid='GER';
````

**Show the team1, team2 and player for every goal scored by a player called Mario `player LIKE 'Mario%'`**

````sql
SELECT team1,team2,player
FROM game JOIN goal ON (game.id=goal.matchid)
WHERE player LIKE 'Mario%';
````

**Show player, teamid, coach, gtime for all goals scored in the first 10 minutes `gtime<=10`**

````sql
SELECT player, teamid,coach,gtime
  FROM goal JOIN eteam ON (goal.teamid=eteam.id)
 WHERE gtime<=10;
 ````
 
 **List the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.**
 
 ````sql
 SELECT mdate,teamname
FROM eteam JOIN game ON(eteam.id=game.team1)
WHERE coach='Fernando Santos';
 
 ````
 
**List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'**

 ````sql
SELECT player
FROM game JOIN goal ON(game.id=goal.matchid)
WHERE stadium='National Stadium, Warsaw';
 
 ````
 
 **Show the name of all players who scored a goal against Germany.**
 
 ````sql
 
SELECT DISTINCT player
FROM goal JOIN game ON(goal.matchid=game.id)
WHERE teamid!='GER'AND
      (team1='GER' OR
      team2='GER');
 
 ````


**Show teamname and the total number of goals scored.**

````sql
SELECT teamname, 
       COUNT(*) AS total_goals
FROM eteam JOIN goal ON eteam.id=goal.teamid
GROUP BY teamname
ORDER BY teamname;
````

**Show the stadium and the number of goals scored in each stadium.**

````sql
SELECT stadium,
       COUNT(id) AS game_count
FROM game JOIN goal ON (game.id=goal.matchid)
GROUP BY stadium
ORDER BY stadium;
````

**For every match involving 'POL', show the matchid, date and the number of goals scored.**

````sql
SELECT matchid,
       mdate,
       COUNT(*) AS goal_count
FROM game JOIN goal ON (game.id = goal.matchid) 
WHERE (team1 = 'POL' OR team2 = 'POL')
GROUP BY matchid,mdate;
````


**For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'**

````sql
SELECT matchid,
       mdate,
       COUNT(*) AS goal_count
FROM goal JOIN game ON (goal.matchid=game.id)
WHERE teamid='GER'
GROUP BY matchid,mdate;
````


**List every match with the goals scored by each team.
 If it was a team1 goal then a 1 appears in score1, otherwise there is a 0. You could `SUM` this column to get a count of the goals scored by team1. Sort your result by mdate, matchid, team1 and team2.**
 
 
 ````sql
 SELECT mdate,
       team1,
       SUM(CASE WHEN teamid = team1 THEN 1 ELSE 0 END) 
       AS goal_score1,
       team2,
       SUM(CASE WHEN teamid = team2 THEN 1 ELSE 0 END) 
       AS goal_score2
FROM game JOIN goal ON (game.id = goal.matchid)
GROUP BY mdate, matchid, team1, team2;
````

More **JOIN** Operations on Movie database
 - The database consists of three tables movie , actor and casting 
 - Tables are given below
![movie table schema](https://user-images.githubusercontent.com/17575943/136108263-b3112035-51e3-43ae-818a-061f2b045405.PNG)


**List the films where the yr is 1962 [Show id, title]**
````sql
SELECT id, title
FROM movie
WHERE yr = 1962;
````
**Give year of 'Citizen Kane'**

````sql
SELECT yr
FROM movie
WHERE title = 'Citizen Kane';
````

 **List all of the Star Trek movies, include the id, title and yr
 (all of these movies include the words Star Trek in the title). Order results by year.**

````sql
SELECT id, title, yr
FROM movie
WHERE title LIKE '%Star Trek%'
ORDER BY yr;
````
 **What are the titles of the films with id 11768, 11955, 21191**

````sql
SELECT title
FROM movie
WHERE id IN (11768, 11955, 21191);
````
**What id number does the actor 'Glenn Close' have?**

````sql
SELECT id
FROM actor
WHERE name = 'Glenn Close';
````
**What is the id of the film 'Casablanca'**
````sql
SELECT id
FROM movie
WHERE title = 'Casablanca';
````
**Obtain the cast list for 'Casablanca'. Use the id value that
you obtained in the previous question.**

````sql
SELECT actor.name
FROM actor
JOIN casting
ON casting.actorid = actor.id
WHERE casting.movieid = 11768;
````

**Obtain the cast list for the film 'Alien'**
````sql
SELECT actor.name
FROM actor
JOIN casting
ON casting.actorid = actor.id
JOIN movie
ON movie.id = casting.movieid
WHERE movie.title = 'Alien';
````
**List the films in which 'Harrison Ford' has appeared**
````sql

SELECT movie.title
FROM movie
JOIN casting
ON casting.movieid = movie.id
JOIN actor
ON actor.id = casting.actorid
WHERE actor.name = 'Harrison Ford';
````
 **List the films where 'Harrison Ford' has appeared - but not in the star role. [Note: the ord field of casting gives the position
of the actor. If ord=1 then this actor is in the starring role]**
````sql

SELECT movie.title
FROM movie
JOIN casting
ON casting.movieid = movie.id
JOIN actor
ON actor.id = casting.actorid
WHERE actor.name = 'Harrison Ford'
AND casting.ord != 1;
````
**List the films together with the leading star for all 1962 films.**
````sql
SELECT movie.title, actor.name
FROM movie
JOIN casting
ON casting.movieid = movie.id
JOIN actor
ON actor.id = casting.actorid
WHERE movie.yr = 1962
AND casting.ord = 1;
````
**Which were the busiest years for 'John Travolta', show the year and the number of movies he made each year for any year in which he made at least 2 movies.**
````sql
SELECT movie.yr, COUNT(*)
FROM movie
JOIN casting
ON casting.movieid = movie.id
JOIN actor
ON actor.id = casting.actorid
WHERE actor.name = 'John Travolta'
GROUP BY movie.yr
HAVING COUNT(movie.title) >= 2;
````
**List the film title and the leading actor for all of 'Julie Andrews' films.**
````sql
SELECT DISTINCT m.title, a.name
FROM (SELECT movie.*
      FROM movie
      JOIN casting
      ON casting.movieid = movie.id
      JOIN actor
      ON actor.id = casting.actorid
      WHERE actor.name = 'Julie Andrews') AS m
JOIN (SELECT actor.*, casting.movieid AS movieid
      FROM actor
      JOIN casting
      ON casting.actorid = actor.id
      WHERE casting.ord = 1) as a
ON m.id = a.movieid
ORDER BY m.title;
````
**Obtain a list of actors in who have had at least 30 starring roles.**

````sql
SELECT actor.name
FROM actor
JOIN casting
ON casting.actorid = actor.id
WHERE casting.ord = 1
GROUP BY actor.name
HAVING COUNT(*) >= 30;
````
**List the 1978 films by order of cast list size.**
````sql
SELECT movie.title, COUNT(*)
FROM movie
JOIN casting
ON movie.id = casting.movieid
WHERE movie.yr = 1978
GROUP BY movie.id
ORDER BY COUNT(*) DESC
````
**List all the people who have worked with 'Art Garfunkel'.**
````sql
SELECT a.name
  FROM (SELECT movie.*
          FROM movie
          JOIN casting
            ON casting.movieid = movie.id
          JOIN actor
            ON actor.id = casting.actorid
         WHERE actor.name = 'Art Garfunkel') AS m
  JOIN (SELECT actor.*, casting.movieid
          FROM actor
          JOIN casting
            ON casting.actorid = actor.id
         WHERE actor.name != 'Art Garfunkel') as a
    ON m.id = a.movieid;
````


