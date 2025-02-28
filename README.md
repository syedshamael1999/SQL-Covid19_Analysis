# 🦠 **COVID-19 Data Analysis**

## 📌 **Overview**  
This project analyzes **COVID-19 data** using **SQL queries** on **MS SQL Server**, focusing on **global trends, mortality rates, vaccination impact, and testing effectiveness**. The data spans from **January 1, 2020** to **December 4, 2021** and includes key metrics such as **cases, deaths, vaccinations, and tests**. The analysis offers insights into **pandemic progression, regional variations, and the effectiveness of interventions**.

## 📂 **Dataset Overview**  
The datasets used in this project are:
- (`Covid_Deaths.xlsx`) - Contains 85172 Rows and 26 Columns
- (`Covid_Vaccinations.xlsx`) - Contains 85172 Rows and 37 Columns

These datasets were imported into **MS SQL** for analysis, and **complex SQL queries** were used to extract meaningful insights from the data.

## 📊 **What the Report Includes**  
The `Covid19-Analysis_Report.md` contains:
- **Global Trends in Cases, Deaths, and Vaccinations**
- **Mortality Analysis & CFR (Case Fatality Rate) Trends**
- **Impact of Vaccinations on Cases & Deaths**
- **Testing Rates vs. Case Detection**
- **Continental Comparison of COVID-19 Metrics**

## 📜 **What's Included?**  
📂 **SQL Queries (`SQL_Queries.md`)** – All SQL scripts used for data retrieval and analysis.  
📓 **SQL Notebook (`COVID_Queries_Notebook.ipynb`)** – SQL notebook with executed queries and results.  
📄 **Full Business Report (`Covid19-Analysis_Report.md`)** – A report of the findings.  
📊 **Datasets** – (`Covid_Deaths.xlsx`) and (`Covid_Vaccinations.xlsx`) - Raw data files used for analysis.  

## 🛠 **Tools Used**  
- **MS SQL Server** – Data querying & analysis.  
- **Azure Data Studio** – SQL notebook creation & execution.  
- **GitHub** – Version control & documentation.  

## 📁 **Repository Structure**  
```plaintext
COVID-19-Data-Analysis/
│── README.md  # Project overview  
│── SQL_Queries.md  # List of SQL queries used in analysis  
│── COVID_Queries_Notebook.ipynb  # SQL notebook with executed queries  
│── Covid19-Analysis_Report.md  # Detailed report with findings  
│── Datasets/  # Raw dataset files  
│   ├── Covid_Deaths.xlsx  
│   ├── Covid_Vaccinations.xlsx  
```
## 📌 **How to Use**  
1️⃣ **Download the datasets**  
   - Download the datasets from the repository:  
     - [`Covid_Deaths.xlsx`](Covid_Deaths.xlsx)  
     - [`Covid_Vaccinations.xlsx`](Covid_Vaccinations.xlsx)  
   - Import them into **MS SQL Server**.

2️⃣ **Run SQL Queries**  
   - Open and execute the SQL queries from the **[`SQL_Queries.md`](SQL_Queries.md)** file.  
   - These queries will help you generate insights from the data.

### Example Query
This query calculates the Case Detection Rate (percentage of tests that result in confirmed cases) by location and month.
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

3️⃣ **Explore the SQL Notebook**  
   - Open the **[`COVID_Queries_Notebook.ipynb`](COVID_Queries_Notebook.ipynb)** to view and execute the queries along with their results.

4️⃣ **Read the Full Report**  
   - Read the detailed findings in the **[`Covid19-Analysis_Report.md`](Covid19-Analysis_Report.md)**.

📄 **[View Full SQL Queries](SQL_Queries.md)** | 📊 **[Read Business Report](Covid19-Analysis_Report.md)** | 📓 **[View SQL Notebook](COVID_Queries_Notebook.ipynb)**
