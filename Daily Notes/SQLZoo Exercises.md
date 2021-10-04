## SQLZOO Exercises
### :triangular_flag_on_post: **JOIN** Operations

- The tables contain all matches and goals from UEFA EURO 2012 Football Championship in Poland and Ukraine.
- The data is available (mysql format) at [SQLZoo](http://sqlzoo.net/euro2012.sql)
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
FROM game JOIN goal ON (id=matchid)
  ````
