# 🦠 SQL Queries for COVID-19 Data Analysis  

## 📌 Overview  
This file contains **SQL queries** used to analyze **COVID-19 data**, consisting of **two tables**, namely **Covid_Deaths** and **Covid_Vaccinations**. The queries help derive insights into cases, deaths, vaccinations, mortality rates, peak surges, and continent-wise comparisons. 
These queries were executed using **MS SQL Server** to derive **actionable insights** into the pandemic's impact worldwide. 

---

### Data Duration
```sql
SELECT 
    (SELECT MAX(date) FROM Covid_Deaths) AS Max_Date_Deaths_Table,
	(SELECT MIN(date) FROM Covid_Deaths) AS Min_Date_Deaths_Table,
    (SELECT MAX(date) FROM Covid_Vaccinations) AS Max_Date_Vaccination_Table,
    (SELECT MIN(date) FROM Covid_Vaccinations) AS Min_Date_Vaccination_Table
--Findings: dataset contains data Fom January 1, 2020 to April 30, 2021
```
--- 

### 1. Total cases, deaths, and vaccinations over time (Monthly)(Globally)
```sql
SELECT 
       FORMAT(D.date, 'yyyy-MM') AS Month,
       SUM(TRY_CAST(D.total_cases AS FLOAT)) AS Total_Cases,
	   SUM(TRY_CAST(D.total_deaths AS FLOAT)) AS Total_Deaths,
	   SUM(TRY_CAST(V.total_vaccinations AS FLOAT)) AS Total_Vaccinations
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V
     ON D.iso_code=V.iso_code
	AND D.date=V.date
WHERE D.location = 'World'  -- Focuses only on global data
GROUP BY FORMAT(D.date, 'yyyy-MM')
ORDER BY Month
```

---

### 2. Top 10 Countries with Highest Cases, Deaths, and Vaccinations (Highest to Lowest)
```sql
SELECT TOP 10 D.location,
       MAX(TRY_CAST(D.total_cases AS FLOAT)) AS Max_Cases,
	   MAX(TRY_CAST(D.total_deaths AS FLOAT)) AS Max_Deaths,
	   MAX(TRY_CAST(V.total_vaccinations AS FLOAT)) AS Max_Vaccinations
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V
     ON D.iso_code = V.iso_code
    AND D.date = V.date
WHERE D.continent IS NOT NULL -- Excludes continents & world-level data
GROUP BY D.location
ORDER BY Max_Cases DESC -- Sorting by highest cases
```

---

### 3. Top 10 countries with the highest Mortality Rate
```sql
SELECT TOP 10 location AS Country,
       SUM(TRY_CAST(total_deaths AS FLOAT)) 
	   / NULLIF(SUM(TRY_CAST(total_cases AS FLOAT)), 0) * 100 AS Mortality_Rate
FROM dbo.Covid_Deaths
WHERE continent IS NOT NULL  -- Excludes aggregated regions like "World"
GROUP BY location
ORDER BY Mortality_Rate DESC;
```
---

### 4. Country with the highest daily spike cases
```sql
SELECT Top 1 location AS Country, 
       date, 
       MAX(TRY_CAST(new_cases AS FLOAT)) AS Highest_Daily_Cases
FROM dbo.Covid_Deaths
WHERE continent IS NOT NULL  -- Excludes aggregated regions like "World"
GROUP BY location, date
ORDER BY Highest_Daily_Cases DESC;
```
---

### 5. Deadliest Month (Highest Monthly Deaths)
```sql
SELECT FORMAT(D.date, 'yyyy-MM') AS Month,
       SUM(TRY_CAST(D.new_deaths AS FLOAT)) AS Monthly_Deaths
FROM dbo.Covid_Deaths D
WHERE D.new_deaths IS NOT NULL
GROUP BY FORMAT(D.date, 'yyyy-MM')
ORDER BY Monthly_Deaths DESC;
```
---

### 6. Top 2 Countries by percentage of population vaccinated
```sql
SELECT TOP 2 
    D.location AS Country,
    MAX(TRY_CAST(V.people_fully_vaccinated AS FLOAT)) 
    / NULLIF(MAX(TRY_CAST(D.population AS FLOAT)), 0) * 100 AS Vaccination_Percentage
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V 
    ON D.iso_code = V.iso_code 
    AND D.date = V.date
WHERE D.continent IS NOT NULL -- Exclude aggregated regions like 'World'
GROUP BY D.location
ORDER BY Vaccination_Percentage DESC;
```
---

### 7. Which continents handled the pandemic better?
```sql
SELECT 
    D.continent AS Continent,
    SUM(TRY_CAST(D.total_deaths AS FLOAT)) / NULLIF(SUM(TRY_CAST(D.total_cases AS FLOAT)), 0) * 100 AS Mortality_Rate,
    SUM(TRY_CAST(D.total_cases AS FLOAT)) / NULLIF(SUM(TRY_CAST(V.total_tests AS FLOAT)), 0) * 100 AS Test_Positivity_Rate,
    MAX(TRY_CAST(V.people_fully_vaccinated AS FLOAT)) / NULLIF(MAX(TRY_CAST(D.population AS FLOAT)), 0) * 100 AS Vaccination_Coverage
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V 
    ON D.iso_code = V.iso_code 
    AND D.date = V.date
WHERE D.continent IS NOT NULL  -- Exclude 'World' and other aggregate regions
GROUP BY D.continent
ORDER BY Mortality_Rate ASC;  -- Lower mortality = better handling
```


