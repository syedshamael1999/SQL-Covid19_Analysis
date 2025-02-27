# 🦠 SQL Queries for COVID-19 Data Analysis  

## 📌 Overview  
This file contains **SQL queries** used to analyze **COVID-19 data**, consisting of **two tables**, namely **Covid_Deaths** and **Covid_Vaccinations**. The queries help derive insights into key trends, mortality rates, vaccination impact, testing efficiency, and continental comparisons. 
These queries were executed using **MS SQL Server** to derive **actionable insights** into the pandemic's impact worldwide. 

---

## 📊 Global Trends  

### Total cases, deaths, and vaccinations over time
```sql
SELECT D.date,
       SUM(TRY_CAST(D.total_cases AS FLOAT)) AS Total_Cases,
	   SUM(TRY_CAST(D.total_deaths AS FLOAT)) AS Total_Deaths,
	   SUM(TRY_CAST(V.total_vaccinations AS FLOAT)) AS Total_Vaccinations
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V
     ON D.iso_code=V.iso_code
	AND D.date=V.date
WHERE D.location = 'World'  -- Focuses only on global data
GROUP BY D.date
ORDER BY D.date
```

### Countries with Highest Cases, Deaths, and Vaccinations (Highest to Lowest)
```sql
SELECT D.location,
       MAX(TRY_CAST(D.total_cases AS FLOAT)) AS Max_Cases,
	   MAX(TRY_CAST(D.total_deaths AS FLOAT)) AS Max_Deaths,
	   MAX(TRY_CAST(V.total_vaccinations AS FLOAT)) AS Max_Vaccinations
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V
     ON D.iso_code = V.iso_code
    AND D.date = V.date
WHERE D.continent IS NOT NULL -- Excludes continents & world-level data
GROUP BY D.location
ORDER BY Max_Cases DESC
```

---

## ⚰️ Mortality Analysis  

### Case fatality rate (CFR) (Total Deaths / Total Cases)
```sql
--NULLIF(..., 0) prevents division by zero by returning NULL if a country has 0 total cases.
--FORMAT(..., 'N2') Formats the result to 2 decimal places
--CONCAT(..., ' %') Adds a % to the formatted number 
SELECT location,
       CONCAT(
	          FORMAT(
	                 MAX(TRY_CAST(total_deaths AS FLOAT))
	                 /NULLIF(MAX(TRY_CAST(total_cases AS FLOAT)), 0)*100 ,
					 'N2'),
					       '%' )AS CFR_PCT
FROM dbo.Covid_Deaths
WHERE continent IS NOT NULL  -- Excludes continents & world-level data
GROUP BY location
ORDER BY MAX(TRY_CAST(total_deaths AS FLOAT))
	          /NULLIF(MAX(TRY_CAST(total_cases AS FLOAT)), 0)*100 DESC  -- Numeric sorting
```

### Deadliest Month (Highest Monthly Deaths)
```sql
--DATEFROMPARTS(YEAR(date), MONTH(date), 1), 'YYYY-MM') makes it easier to format as YYYY-MM
SELECT location,
       FORMAT(DATEFROMPARTS(YEAR(date), MONTH(date), 1), 'yyyy-MM') AS Month,
       SUM(TRY_CAST(new_deaths AS FLOAT)) AS Monthly_Deaths
FROM dbo.Covid_Deaths
WHERE continent IS NOT NULL  -- Excludes continents & world-level data
GROUP BY location, YEAR(date), MONTH(date)
ORDER BY Monthly_Deaths DESC
```
---

## 💉 Vaccination Impact  

### Impact of Vaccination on Deaths Over Time
```sql
SELECT 
    D.location,
    FORMAT(DATEFROMPARTS(YEAR(D.date), MONTH(D.date), 1), 'yyyy-MM') AS Month,
    SUM(TRY_CAST(V.new_vaccinations AS FLOAT)) AS Monthly_New_Vaccinations,
    SUM(TRY_CAST(D.new_deaths AS FLOAT)) AS Monthly_New_Deaths
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V 
     ON D.iso_code = V.iso_code
    AND D.date = V.date
WHERE D.continent IS NOT NULL
  AND D.new_deaths IS NOT NULL
  AND V.new_vaccinations IS NOT NULL
GROUP BY D.location, YEAR(D.date), MONTH(D.date)
ORDER BY D.location, Month;
```

---

## 🧪 Testing & Spread Analysis  

### Testing Rates vs. Case Detection
```sql
SELECT 
    D.location,
    FORMAT(DATEFROMPARTS(YEAR(D.date), MONTH(D.date), 1), 'yyyy-MM') AS Month,
    SUM(TRY_CAST(V.new_tests AS FLOAT)) AS Monthly_Tests,
    SUM(TRY_CAST(D.new_cases AS FLOAT)) AS Monthly_Cases,
    CASE 
        WHEN SUM(TRY_CAST(V.new_tests AS FLOAT)) = 0 THEN NULL
        ELSE (SUM(TRY_CAST(D.new_cases AS FLOAT)) / NULLIF(SUM(TRY_CAST(V.new_tests AS FLOAT)), 0)) * 100 
    END AS Case_Detection_Rate_PCT
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V 
     ON D.iso_code = V.iso_code
    AND D.date = V.date
WHERE D.continent IS NOT NULL
  AND D.new_cases IS NOT NULL
  AND V.new_tests IS NOT NULL
GROUP BY D.location, YEAR(D.date), MONTH(D.date)
ORDER BY D.location, Month;
```

---

## 🌍 Continental Comparison 

### Which continents handled the pandemic better?
```sql
--Lower CFR% is better. Indicates how deadly the virus has been in terms of the number of cases.
--Higher Case_Detection Rate PCT is better. Indicates how effective the testing system is in detecting COVID-19 cases.
--Higher Vaccination Coverage PCT is better. Indicates how many people are vaccinated, reflecting progress in immunity and control of the pandemic.
SELECT 
    D.continent, 
    (SUM(TRY_CAST(D.total_deaths AS FLOAT)) / NULLIF(SUM(TRY_CAST(D.total_cases AS FLOAT)), 0)) * 100 AS CFR_PCT, 
    (SUM(TRY_CAST(D.total_cases AS FLOAT)) / NULLIF(SUM(TRY_CAST(V.total_tests AS FLOAT)), 0)) * 100 AS Case_Detection_Rate_PCT, 
    (SUM(TRY_CAST(V.total_vaccinations AS FLOAT)) / NULLIF(SUM(TRY_CAST(D.population AS FLOAT)), 0)) * 100 AS Vaccination_Coverage_PCT
FROM dbo.Covid_Deaths D
JOIN dbo.Covid_Vaccinations V 
     ON D.iso_code = V.iso_code 
    AND D.date = V.date
WHERE D.continent IS NOT NULL
GROUP BY D.continent
ORDER BY CFR_PCT ASC;
```
