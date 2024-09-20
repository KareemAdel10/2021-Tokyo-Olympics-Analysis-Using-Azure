
--1. Distribution of Male and Female Athletes Across Disciplines
```
SELECT Discipline, 
       SUM(Female) AS Female_Athletes, 
       SUM(Male) AS Male_Athletes, 
       SUM(Total) AS Total_Athletes 
FROM EntriesGender
GROUP BY Discipline
ORDER BY Total_Athletes DESC;
```

----
--2. Top 5 Countries that earned Total Medal Count
```
SELECT
    TOP 5
    [Team/NOC],
    Gold + Silver + Bronze as Total_medals
FROM 
    Medals
```
----
--3. Coaches by Country and Discipline
```
SELECT 
    NOC, 
    Discipline,
    count(*) as Total_coaches
FROM 
    Coaches
GROUP BY    
    NOC, 
    Discipline
```
----
--4. Countries with the Highest Number of Disciplines Coached
```
SELECT
    NOC,
    count(DISTINCT Discipline) as Total_Coached_Disciplines
FROM
    Coaches
GROUP BY
    NOC
ORDER by
    Total_Coached_Disciplines DESC
```

----
--5. Relationship Between Athletes and Coaches by Discipline
```
SELECT a.Discipline, 
       COUNT(DISTINCT a.Name) AS Number_of_Athletes, 
       COUNT(DISTINCT c.Name) AS Number_of_Coaches, 
       COUNT(DISTINCT a.Name)/COUNT(DISTINCT c.Name) AS Athlete_to_Coach_Ratio
FROM Atheletes a
JOIN Coaches c ON a.Discipline = c.Discipline AND a.NOC = c.NOC
GROUP BY a.Discipline
ORDER BY Athlete_to_Coach_Ratio DESC;
```
----
--6. Countries with Equal Representation in Male and Female Entries
```
SELECT Discipline
From EntriesGender
Where Female = Male
```
----
--7. Total members in each Team
```
SELECT NOC,
      count(distinct name) + count(distinct coach_name) as Team_capacity
FROM Atheletes
GROUP BY NOC
ORDER BY Team_capacity desc
```
