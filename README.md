# SQL-resources
SELECT * FROM [Porfolio1].[dbo].[Data1]
SELECT * FROM [Porfolio1].[dbo].[Data2]

--number of rows in dataset
SELECT COUNT(*) FROM [Porfolio1].[dbo].[Data1]
SELECT COUNT(*) FROM [Porfolio1].[dbo].[Data2]

--Data from Jharkhand và Bihar
SELECT * 
FROM [Porfolio1].[dbo].[Data1]
WHERE State IN ('Jharkhand','Bihar')

--Sum the population of Indian from column population in data2
SELECT SUM(Population) AS population
FROM [Porfolio1].[dbo].[Data2]

--Average growth of population
SELECT AVG(Growth)*100 AS average_growth_in_percentage
FROM [Porfolio1].[dbo].[Data1]

--Average growth of population by each State
SELECT State, AVG(Growth)*100 AS average_growth_in_percentage
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State

--Average sex ratio
SELECT State, ROUND(AVG(Sex_ratio),0) AS average_sex_ratio
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
ORDER BY average_sex_ratio DESC

--Average literacy 
SELECT State, ROUND(AVG(Literacy),0) AS average_literacy
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
HAVING ROUND(AVG(Literacy),0) > 90
ORDER BY average_literacy DESC

--Top 3 state show highest growth ratio
SELECT TOP 3.State, AVG(Growth)*100 AS average_growth_in_percentage
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
ORDER BY average_growth_in_percentage DESC

--Bottom 3 state that have lowest sex ratio
SELECT TOP 3.State, ROUND(AVG(Sex_ratio),0) AS average_sex_ratio
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
ORDER BY average_sex_ratio ASC

--Create temporary table(#Top_3)
DROP TABLE IF EXISTS #TOP_3
CREATE TABLE #TOP_3
(
State NVARCHAR(255),
top_state FLOAT
)
INSERT INTO #TOP_3
SELECT State, ROUND(AVG(Literacy),0) AS average_literacy
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
ORDER BY average_literacy DESC

SELECT TOP 3 *
FROM #TOP_3
ORDER BY #TOP_3.top_state DESC

--Create temporary table(#Bottom_3)
DROP TABLE IF EXISTS #BOTTOM_3
CREATE TABLE #BOTTOM_3
(
State NVARCHAR(255),
bottom_state FLOAT
)
INSERT INTO #BOTTOM_3
SELECT State, ROUND(AVG(Literacy),0) AS average_literacy
FROM [Porfolio1].[dbo].[Data1]
GROUP BY State
ORDER BY average_literacy ASC

SELECT TOP 3 *
FROM #BOTTOM_3
ORDER BY #BOTTOM_3.bottom_state ASC

--Union operater
SELECT * 
FROM (SELECT TOP 3 * FROM #BOTTOM_3 ORDER BY #BOTTOM_3.bottom_state ASC) a
UNION
SELECT *
FROM (SELECT TOP 3 * FROM #TOP_3 ORDER BY #TOP_3.top_state DESC) b

--State which has name start with letter A
SELECT DISTINCT State FROM [Porfolio1].[dbo].[Data1]
WHERE LOWER(State) LIKE 'a%'

--Joining both tables
SELECT a.District, a.State, a.Sex_Ratio/1000, b.Population 
FROM [Porfolio1].[dbo].[Data1] a 
INNER JOIN [Porfolio1].[dbo].[Data2] b 
ON a.District = b.District

--Fomulas + total males and females
SELECT d.State, SUM(d.males) total_males, SUM(d.females) total_female FROM 
(SELECT c.District, c.State, ROUND(c.population/(c.sex_ratio+1),0) males, ROUND((c.population*c.sex_ratio)/(c.sex_ratio+1),0) females 
FROM 
(SELECT a.District, a.State, a.Sex_Ratio/1000 sex_ratio, b.Population FROM [Porfolio1].[dbo].[Data1] a INNER JOIN [Porfolio1].[dbo].[Data2] b ON a.District = b.District) c) d
GROUP BY State

--Total literacy rate
select c.state,sum(literate_people) total_literate_pop,sum(illiterate_people) total_lliterate_pop from 
(select d.district,d.state,round(d.literacy_ratio*d.population,0) literate_people,
round((1-d.literacy_ratio)* d.population,0) illiterate_people from
(select a.district,a.state,a.literacy/100 literacy_ratio,b.population from [Porfolio1].[dbo].[Data1] a 
inner join [Porfolio1].[dbo].[Data2] b on a.district=b.district) d) c
group by c.state

-- population in previous census
select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from [Porfolio1].[dbo].[Data1] a inner join [Porfolio1].[dbo].[Data2] b on a.district=b.district) d) e
group by e.state)m

-- population vs area

select (g.total_area/g.previous_census_population)  as previous_census_population_vs_area, (g.total_area/g.current_census_population) as 
current_census_population_vs_area from
(select q.*,r.total_area from (

select '1' as keyy,n.* from
(select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from [Porfolio1].[dbo].[Data1] a inner join [Porfolio1].[dbo].[Data2] b on a.district=b.district) d) e
group by e.state)m) n) q inner join (

select '1' as keyy,z.* from (
select sum(area_km2) total_area from [Porfolio1].[dbo].[Data2])z) r on q.keyy=r.keyy)g

--window function
--OUTPUT top 3 districts from each state with highest literacy rate
select a.* from
(select district,state,literacy,rank() over(partition by state order by literacy desc) rnk from [Porfolio1].[dbo].[Data1]) a
where a.rnk in (1,2,3) order by state
