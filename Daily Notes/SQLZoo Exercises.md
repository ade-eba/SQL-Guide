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






