# **Impacto del Covid-19 en América Latina**


[**Ver el dashboard en Power BI en línea**](https://bit.ly/3Knt7HP)

[![alt text](https://imgur.com/SqfOdqO.png)](https://bit.ly/3Knt7HP)

* * *
## _Contenido_

* **[0. Problema de Negocio](#Problema-de-Negocio)**
* **[1. Extracción y Limpieza de Datos](#Extracción-y-Limpieza-de-Datos)**
* **[2. Creación de vistas](#Creación-de-vistas)**
* **[3. Resultados de Análisis](#Resultados-de-Análisis)**

***


# Problema de Negocio

Según Comité sobre el COVID-19 del CONCYTEC, hasta el 23 de noviembre del 2021 Perú presentó 5977 muertes por millón de habitantes, la cifra más alta en todo el mundo.

### ❓ **<u>Preguntas de análisis</u>**: 
* ¿Cuál fue la evolución de la tasa de mortalidad en Perú en comparación con otros países de América Latina?
* ¿Factores como la incidencia de diabetes y la edad promedio influyeron en la tasa de mortalidad de los países?
* ¿Qué impacto tuvo la vacunación en Perú sobre la tasa de mortalidad en comparación con otros países?


### 🎯 **<u>Objetivo</u>**<br>
Analizar la evolución del COVID-19 en Perú y el comportamiento de a tasa de mortalidad frente a otros países latinoamericanos.
<br>

| Alcance de estudio | Tipo de Análisis | Tools used |
| --- | :-: | --: |
| Longitudinal: Desde inicio de la pandemia al 27 de Diciembre del 2021 | Exploratorio | SQL- Power BI - Markdown |

# Extracción y Limpieza de Datos

Los datos usados para el análisis fueron extraídos de  [_Our World in Data_](https://ourworldindata.org/covid-deaths), al 27 de diciembre de 2021 en dos tablas denominadas:

- Tabla "Latin\_deaths": Casos, muertes, prevalencia de diabetes, edad promedio, casos y muertes por millón, casos nuevos por día.
- Tabla "Vacu\_latin": Datos de vacunación, población vacunada completa.

Primero, vamos a limpiar el conjunto de dato eliminando las columnas que no son de nuestro interés. <br>
¡Hecha un vistazo al código!


```sql
ALTER TABLE Latin_deaths DROP COLUMN continent, total_cases_per_million, 
total_deaths_per_million, gdp_per_capita, cardiovasc_death_rate, 
hospital_beds_per_thousand, human_development_index
```

# Creación de vistas

Debido a los más de 88,000 registros se crearon vistas con la función **CREATE VIEW**, las cuales mostrarán el conjunto de datos específico que nos permita analizar cada una de las preguntas a detalle. 

📍<u>Evolución de la tasa de mortalidad en Perú</u><br>
La tasa de mortalidad fue calculada respecto al total de casos, sin embargo, también fue necesario calcular la tasa de contagio por población para contrastar la proporción de muertos respecto al número total de habitantes. Los cálculos se muestran para cada fecha desde la fecha del Caso 0 en cada país.

¡Veamos el script!



```sql
CREATE VIEW DIATasaContagPorPob AS
  SELECT location, date, population, total_cases, total_deaths
    ,(total_deaths/total_cases)* 100 AS per_tasamortalidad
    ,(total_cases/population)* 100 AS perc_contagiados
  FROM PORTFOLIOProject..LatinDeaths
  ORDER by total_cases desc
```

Luego, se filtró el total de muertos por país *location* y por número de habitantes *population* mediante función agregada para ver un resumen de la información de la vista anterior, ya que en ésta solo se muestran la cifras máximas hasta el corte del 27 de diciembre.


```sql
CREATE VIEW RESUMEN AS
  SELECT location, population,MAX(cast(total_deaths as int)) AS totmuertos
  FROM PORTFOLIOProject..LatinDeaths
  GROUP BY location, population
```

📍<u>Indicadores de incidencia de diabetes y la edad promedio</u>  
Para esta vista solo se utilizaron funciones agregadas, pues los valores incidencia de diabetes y edad promedio son el mismo para todos los registros hasta la fecha de corte.  
Se calculó el promedio de ambas variables y se agruparon con **GROUP BY** por locación y número de habitantes como la anterior vista.


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

📍<u>Vacunación y mortalidad</u><br>
Es necesario unir con **JOIN** la información de las tablas *LatinDeaths* y *vacu_latin* para contrastar la tasa de mortalidad con el número de nuevos vacunados por día *new_vaccinations*. En este caso se utilizó la función **OVER(PARTITION BY)** en lugar de **GROUP BY** pues ésta última se limita a mostrar los atributos por los que se agrupa y excluye variables relevantes para nuestro análisis como *date y population*. <br> ¡Veamos el script!


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

# Resultados de Análisis

Para una mejor comprensión de los datos desarrollé un Dashboard dinámico en Power BI.  
Pruébalo [**AQUÍ**](https://bit.ly/3Knt7HP)

✔️ **¿Cuál fue la evolución de la tasa de mortalidad en Perú en comparación con otros países de América Latina?** <BR>
Como se observa en el gráfico, el Perú es el país Latinoamericano con la mayor tasa de mortalidad en promedio (8.9%), los que le siguen son Ecuador (6.2%) y Paraguay (3.6%), es decir aproximadamente 9 de cada 100 casos peruanos de COVID-19 han fallecido en el Perú. 

<center><img src="https://imgur.com/RxjADml.png" height="300" /></center>

Respecto a la población total de 33.36 millones, 6 de cada 1000 peruanos en promedio (0.595%) han fallecido por COVID-19 hasta la fecha.

✔️ **¿Factores como la incidencia de diabetes y la edad promedio influyeron en la tasa de mortalidad de los países?** <BR>
No se observa una influencia de la incidencia de diabetes y la edad promedio de la población sobre la tasa de mortalidad en los países Latinoamericanos. El gráfico de dispersión no muestra tendencia alguna entre Prevalencia de diabetes y Tasa de Mortalidad, ni entre la Edad Promedio y la Tasa de Mortalidad, por lo que se descarta la probable influencia de estas variables.
<center><img src="https://imgur.com/6cia7uJ.png" height="280" /></center>

Por ejemplo, si consideramos a la edad como un factor determinante de la mortalidad, se puede invalidar esta premisa al evaluar el caso de Uruguay: Con una edad promedio superior al resto de países presenta una de las menores tasas de mortalidad, pues aproximadamente 2 de cada 1000 uruguayos fallecen por COVID-19.

<center><img src="https://imgur.com/dczdULI.png" height="280" /></center>

✔️ **¿Qué impacto tuvo la vacunación en Perú sobre la tasa de mortalidad en comparación con otros países?** <BR>
La tasa de mortalidad antes del 9 de febrero (fecha de inicio de vacunación) fue de 9.14%, hasta fines del 2021 esta cifra disminuyó a 8.89%. Es decir, que 3 de cada 1000 contagiados (-0.25%) dejaron de morir luego de la vacunación. 

<u>ANTES DE VACUNACIÓN</u><BR>
<img src="https://imgur.com/9zAGzjn.png" width="350" />

<u>DESPUÉS DE VACUNACIÓN</u><BR>
<img src="https://imgur.com/1FoQ0yy.png" width="350" />











* * *

### 📌 **CONOCE MÁS DE MIS PROYECTOS [AQUÍ](https://lu-emperatriz.github.io/)**

Lucero Emperatriz.

 <a href="https://www.linkedin.com/in/lucero-sovero/"><img src="https://imgur.com/p58yPZr.png" height="auto" width="50" style="border-radius:50%"></a>
