# Covid-19_Data_Visualization

## Introduction

After exploring the data in the Covid-19 Data exploration project it is time to get some vizualizations of this data and try to show how each country is affected and how the world reacted to this virus.

### Vizualization Queries 

#### Covid-19 Death Vizualization Queries 

##### 1. Total cases wit h total deaths and there percentage

    USE CovidProject

    SELECT  SUM(new_cases) AS 'Total Cases' ,
    SUM(CAST(new_deaths AS INT))  AS 'Total Deaths',
    (SUM(CAST(new_deaths AS INT))/ SUM(new_cases) )*100 AS 'Death Percentage'
    
    FROM CovidDeaths
    WHERE continent is not null

##### 2. Highest death ranked by the Continents 

    SELECT location AS 'Continent',
    MAX(cast(total_deaths as INT)) AS Highest_Death_Counts
    
    FROM CovidDeaths
    WHERE continent is null AND location NOT IN ('International','European Union','World','Low income','Lower middle income','High income','Upper middle income' )
    GROUP BY location
    ORDER BY Highest_Death_Counts DESC

##### 3. Looking at Countries with highest infection rate compared to population

    SELECT location, population,
    MAX(total_cases) AS Highest_Infection_Count,
    MAX((total_cases/population))*100 AS Infected_Percentage
    
    FROM CovidDeaths
    GROUP BY location,population
    ORDER BY Infected_Percentage DESC


##### 4. Looking at Countries with highest infection rate compared to population with date

    SELECT location,date, population,
    MAX(total_cases) AS Highest_Infection_Count,
    MAX((total_cases/population))*100 AS Infected_Percentage
    
    FROM CovidDeaths
    GROUP BY location,population,date
    ORDER BY Infected_Percentage DESC

##### 5. Total Population vs Total Vaccination

    SELECT CD.continent , CD.location , CD.date, CD.population, CV.new_vaccinations,
    SUM (CONVERT(BIGINT, CV.new_vaccinations)) OVER (PARTITION BY CD.location ORDER BY CD.location, CD.date) AS 'Total Amout of Vaccination Per Location'
    FROM CovidDeaths AS CD
    JOIN CovidVaccinations AS CV
         ON CD.location = CV.location 
	     AND CD.date = CV.date
	     WHERE CD.continent is not null
	     ORDER BY CD.continent , CD.location 

##### 6. Total Vaccinated Population by location Percentage

    WITH pvv (continent, location, date, population, new_vaccinations , Total_Amout_of_Vaccination_Per_Location)
    AS (
    SELECT CD.continent , CD.location , CD.date, CD.population, CV.new_vaccinations,
    SUM (CONVERT(BIGINT, CV.new_vaccinations)) OVER (PARTITION BY CD.location ORDER BY CD.location, CD.date) AS Total_Amout_of_Vaccination_Per_Location
   
    FROM CovidDeaths AS CD
    JOIN CovidVaccinations AS CV
         ON CD.location = CV.location 
	     AND CD.date = CV.date
	     WHERE CD.continent is not null
       )

    SELECT *, (Total_Amout_of_Vaccination_Per_Location/population)*100 AS Percentage_Vaccinated
    
    FROM pvv

##### 7. Total Vac Per

    CREATE VIEW  percent_population_vaccinated_View AS

    SELECT CD.continent , CD.location , CD.date, CD.population, CV.new_vaccinations,
    SUM (CONVERT(BIGINT, CV.new_vaccinations)) OVER (PARTITION BY CD.location ORDER BY CD.location, CD.date) AS Total_Amout_of_Vaccination_Per_Location
    
    FROM CovidDeaths AS CD
    JOIN CovidVaccinations AS CV
        ON CD.location = CV.location 
	   AND CD.date = CV.date
	   WHERE CD.continent is not null

    WITH TotalVac (Location, Population, Total_Vac, percentage_of_population_per_location)
    AS (
    SELECT Location, SUM(Population) AS Population, 
    SUM(Total_Amout_of_Vaccination_Per_Location) AS Total_Vac,
    (SUM(Total_Amout_of_Vaccination_Per_Location)/SUM(population))*100 AS percentage_of_population_per_location
   
    FROM percent_population_vaccinated
    GROUP BY Location
       )

    SELECT*
    
    FROM TotalVac
    WHERE percentage_of_population_per_location <100
    ORDER BY percentage_of_population_per_location DESC

##### 8.TOTAl percatne vaccinated by Continent

    SELECT CD.continent , 
    SUM(CONVERT(BIGINT,CD.population)) AS Total_Population,
    SUM(CONVERT(BIGINT , CV.New_vaccinations)) AS Total_Vaccinations,
    SUM((CONVERT(FLOAT, CV.New_vaccinations))/(CONVERT(FlOAT,CD.population))) AS Percentage_Vaccinated

    FROM CovidDeaths AS CD
    JOIN CovidVaccinations AS CV
         ON CD.location = CV.location 
	     AND CD.date = CV.date
             WHERE CD.continent IS NOT NULL
    GROUP BY CD.continent
    ORDER BY Total_Vaccinations DESC
