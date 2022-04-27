# **Impact of Covid-19 in Latin America**

[**View the dashboard in Power BI online**](https://bit.ly/3OGgn2c)

[![alt text](https://imgur.com/SqfOdqO.png)](https://bit.ly/3OGgn2c)

***

## *Table of Contents*

* **[0. Business Problem](#Business-Problem)**
* **[1. Data Extraction and Cleaning](#Data-Extraction-and-Cleaning)**
* **[2. Creating Views](#Creating-Views)**
* **[3. Analysis & Results](#Analysis-&-Results)**

***

# Business Problem

According to Committee on COVID-19 of CONCYTEC, until November 23, 2021 Peru presented 5977 deaths per million inhabitants, the highest rate worldwide.

### ❓ **<ins>Analysis questions</ins>**:

* What was the evolution of the mortality rate in Peru compared to other Latin American countries?
* Did factors such as the incidence of diabetes and average age influence the countries' mortality rate?
* What was the impact of vaccination in Peru on the mortality rate compared to other countries?

### 🎯 **<ins>Objective</ins>**. 

To analyze the evolution of COVID-19 in Peru and the behavior of the mortality rate compared to other Latin American countries.<br/>

| Scope of the study | Type of analysis | Tools used |
| --- | :-: | --: |
| Longitudinal: From the beginning of the pandemic to December 27, 2021. |    Exploratory   | SQL- Power BI - Markdown |

# Data Extraction and Cleaning

The data used for the analysis were extracted from [*Our World in Data*](https://ourworldindata.org/covid-deaths), as of December 27, 2021 in two tables named:

* Table "Latin_deaths": cases, deaths, prevalence of diabetes, average age, cases and deaths per million, new cases per day.
* Table "Vacu_latin": Vaccination data, full vaccinated population.

First, let's clean the dataset by removing the columns that are not of interest to us.<br/>Take a look at the code!

```sql
ALTER TABLE Latin_deaths DROP COLUMN continent, total_cases_per_million, 
total_deaths_per_million, gdp_per_capita, cardiovasc_death_rate, 
hospital_beds_per_thousand, human_development_index
```

# Creating views

Due to the 88,000+ records we created views with the **CREATE VIEW** function, which will display the specific dataset allowing us to analyze each of the questions in detail.

📍<ins>Evolution of Mortality rate in Peru</ins><br/>The mortality rate was calculated with respect to the total number of cases; however, it was also necessary to calculate the infection rate per population to contrast the proportion of deaths with respect to the total number of inhabitants. The calculations are shown for each date from the date of Case 0 in each country.

Let's check the code!

```sql
CREATE VIEW DIATasaContagPorPob AS
  SELECT location, date, population, total_cases, total_deaths
    ,(total_deaths/total_cases)* 100 AS per_tasamortalidad
    ,(total_cases/population)* 100 AS perc_contagiados
  FROM PORTFOLIOProject..LatinDeaths
  ORDER by total_cases desc
```

Next, we filtered the total deaths by country *location* and by number of *population* by aggregate function to see a summary of the information in the previous view, since this one only shows the maximum numbers up to the December 27 cutoff.

```sql
CREATE VIEW RESUMEN AS
  SELECT location, population,MAX(cast(total_deaths as int)) AS totmuertos
  FROM PORTFOLIOProject..LatinDeaths
  GROUP BY location, population
```

📍<ins>Indicators of diabetes incidence and average age</ins><br/>
Only aggregate functions were used for this view, since the values for diabetes incidence and average age are the same for all records up to the cutoff date.<br/>The average of both variables was calculated and grouped with **GROUP BY** by location and number of inhabitants as the previous view.

```sql
CREATE VIEW RESTasaMortPorPob AS
  SELECT location, population, AVG(ROUND(median_age,0)) AS EdadProm
    , ROUND(AVG(diabetes_prevalence),2) AS Diabetes
    , MAX(cast(total_deaths as int)) AS totalDeath 
    ,MAX(total_deaths/population)*100 AS perc_tasamortali
  FROM PORTFOLIOProject..LatinDeaths
  GROUP BY location, population
  ORDER BY perc_tasamortali desc
```

📍<ins>Vaccination and mortality</ins><br/>
It is necessary to join with **JOIN** the information from the *LatinDeaths* and *vacu_latin* tables to contrast the mortality rate with the number of new vaccinations per day *new_vaccinations*. In this case we used the function **OVER(PARTITION BY)** instead of **GROUP BY** because the latter is limited to show the attributes by which the groups are grouped and excludes relevant variables for our analysis such as *date and population*.<br/>Let's see the script!

```sql
CREATE VIEW PobVaccinated AS 
	  SELECT dea.location, dea.date, dea.population
		  ,vac.new_vaccinations
		  ,CAST(people_vaccinated_per_hundred as float) AS perc_pob_vac
		  ,SUM(CAST(vac.new_vaccinations as float)) 
		  OVER(PARTITION BY dea.location ORDER BY dea.location, dea.date) as total_vac
	  FROM PORTFOLIOProject..LatinDeaths dea
	  JOIN PORTFOLIOProject..vacu_latin vac
	  ON dea.location = vac.location AND dea.date = vac.date
```

# Analysis & Results

For a better understanding of the data I developed a dynamic Dashboard in Power BI.<br/>
Try it out [**here**](https://bit.ly/3OGgn2c).

✔️ **What was the evolution of the mortality rate in Peru compared to other Latin American countries?**<br/>
As shown in the graph, Peru is the Latin American country with the highest mortality rate on average (8.9%), those following are Ecuador (6.2%) and Paraguay (3.6%), i.e. approximately 9 out of every 100 Peruvian COVID-19 cases have died in Peru.

<center><img src="https://imgur.com/RBad5w3.png" height="300"/></center>

With respect to the total population of 33.36 million, 6 out of every 1000 Peruvians on average (0.595%) have died from COVID-19 to date.

✔️ **Did factors such as the incidence of diabetes and average age influence the countries' mortality rate?**<br/>No influence of diabetes incidence and average age of the population on the mortality rate in Latin American countries is observed. The scatter plot does not show any trend between Diabetes Prevalence and Mortality Rate, nor between Average Age and Mortality Rate, so the probable influence of these variables is ruled out.

<center><img src="https://imgur.com/YSUBrxM.png" height="280"/></center>

For example, if we consider age as a determinant of mortality, this premise can be invalidated when evaluating the case of Uruguay: With an average age higher than the rest of the countries, it presents one of the lowest mortality rates, since approximately 2 out of every 1000 Uruguayans die from COVID-19.

<center><img src="https://imgur.com/jZYhG6a.png" height="280"/></center>

✔️ **What impact did vaccination in Peru have on the mortality rate compared to other countries?**<br/>The mortality rate before February 9 (vaccination start date) was 9.14%, by the end of 2021 this figure decreased to 8.89%. In other words, 3 out of every 1000 infected (-0.25%) stopped dying after vaccination.

*BEFORE VACCINATION*<br/>
Vaccination in Peru started on 07 Feb. 2021.<br/>
![](https://imgur.com/9zAGzjn.png)

*AFTER VACCINATION*<br/>

![](https://imgur.com/1FoQ0yy.png)

***

### **📌 Discover more of my projects [HERE](https://lu-emperatriz.github.io/)**.

[![](https://imgur.com/p58yPZr.png)](https://www.linkedin.com/in/lucero-sovero/)
