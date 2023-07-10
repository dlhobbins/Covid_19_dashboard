# Covid_19_dashboard
- This was a follow along project that I completed in order to further develop my skills with Big Query and Tableau in addition to beginning to build a portfolio.  
- Downloaded the dataset from from: https://ourworldindata.org/covid-deaths  Accessed on 05/06/2023 (DD,MM,YYYY)
- Opened data set in excel and removed a selection of columns until I had 2 data sets. covid_deaths and covid_vaccinations
- Loaded csv files into Big Query for exploration of the data

## Big Query

### Ran the following queries to explore the dataset  

--Total Cases vs Total Deaths--  
SELECT  
  location,date,total_deaths, total_cases, (total_deaths/total_cases)*100 AS death_percentage  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths`   
ORDER BY  
  1,2  


--Looking at total cases vs population--  
SELECT   
  location,date,population,total_cases, (total_cases/population) *100 AS percentage_population_infected  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths`   
ORDER BY  
  1,2  


--Looking for countries with highest infection rate per capita--
SELECT  
  location,population, MAX(total_cases) AS highest_infection_rate, (MAX(total_cases/population)) *100 AS percent_population_infected  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths`   
GROUP BY  
  location,population  
ORDER BY  
  4 DESC  


--Looking for the country with the highest death rate--  
SELECT    
  location, MAX(total_deaths) AS total_death_count  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths`   
WHERE   
  continent is not null   
GROUP BY  
  location  
ORDER BY  
  total_death_count DESC  


-- Death rates by continent--  
SELECT    
  location, MAX(total_deaths) AS total_death_count  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths`   
WHERE   
  continent is null   
  and location not in ("World","High income","Upper middle income","Lower middle income","European Union","Low income")  
GROUP BY  
  location  
ORDER BY  
  total_death_count DESC  


--Looking at number of people vaccinated--
SELECT   
  dea.continent, dea.location, dea.date, dea.population, vax.new_vaccinations, SUM(vax.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated,  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths` as dea  
JOIN  
  `portfolioproject388206.CovidProject.CovidVax` as vax  
ON dea.location = vax.location  
AND dea.date = vax.date  
WHERE dea.continent is not null  
ORDER BY  
  1,2,3  


- Once this query had run, I was able to create a CTE to perform calculations on the rolling people vaccinated column  

--Using a CTE--  
WITH popvsvac AS  
(   
  SELECT   
  dea.continent, dea.location, dea.date, dea.population, vax.new_vaccinations, SUM(vax.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths` as dea  
JOIN  
  `portfolioproject388206.CovidProject.CovidVax` as vax  
ON dea.location = vax.location  
AND dea.date = vax.date  
WHERE dea.continent is not null  
)  
SELECT  
  *,(rolling_people_vaccinated/population)*100 AS percent_population_vaccinated  
FROM  
  popvsvac  


--Looking for corelation between vax and reduced infection--  
SELECT   
  dea.location,dea.date, dea.new_cases, vax.people_fully_vaccinated, dea.continent  
FROM   
  `portfolioproject388206.CovidProject.CovidDeaths` AS dea  
JOIN  
  `portfolioproject388206.CovidProject.CovidVax` AS vax  
ON dea.location = vax.location and dea.date = vax.date  
WHERE dea.continent is null and dea.location not in ("World","High income","Upper middle income","Lower middle income","European Union","Low income")  
ORDER BY  
  1,2   

## Having explored the data set, I decided to use 4 of the queries to get the tables I would use for the dashboard creation.  

- Global Numbers Table which shows total cases, total deaths and death percentage  
- Total Deaths by Continent  
- Highest Infection Rate by Country  
- Percent of Population Infected by Date

  












